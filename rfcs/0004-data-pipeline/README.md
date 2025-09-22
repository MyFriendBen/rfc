# RFC 0004: Multi-Tenant Analytics Platform for MFB Screening Data

Status: Draft
Author: Katie Brey
Created: 2025-09-17
PR: TODO

## Summary

We propose an MFB Analytics Platform that replaces hard-to-maintain materialized views with a engineer-friendly dbt-based data transformation pipeline. The new system provides reliable, testable analytics for global MFB insights and secure multi-tenant dashboards for white-label partners like North Carolina. Built with dbt, Grafana, and Terraform, it implements PostgreSQL row-level security to automatically enforce data isolation without application-level filtering.

## Background

MyFriendBen currently uses a set of [materialized views](https://github.com/MyFriendBen/data-queries) to provide analytics data. These views aggregate screening data from the Django application's PostgreSQL database for our Looker Studio dashboards. However, our current approach has significant maintainability and scalability challenges.

The materialized views are difficult to modify because they have complex interdependencies - changing one view often requires dropping and recreating multiple dependent views in the correct order. There's no systematic way to test data transformations, making it risky to implement changes. Version control of the view definitions is manual and error-prone. When views need to be recreated, engineers must manually determine the dependency order and execute SQL scripts in sequence.

These limitations are particularly problematic as analytics requirements evolve. Global analytics need to show analytics across white labels, while white-label partners like North Carolina require secure, isolated access to only their data. The current materialized view approach lacks automated data isolation mechanisms, and makes it difficult to make updates to our analytics logic as we grow.

## Proposal

Replace materialized views with a dbt-based data transformation pipeline that provides dependency management, automated testing, and version control. Implement PostgreSQL row-level security to enable both global analytics (priority) and secure multi-tenant access for white-label partners.

Dbt defines data transformations as code with automatic dependency resolution, built-in testing capabilities, and git-based version control. For multi-tenancy, PostgreSQL RLS policies automatically filter data based on database user credentials, eliminating the need for application-level filtering and reducing the risk of data leakage.

Grafana provides a flexible, open-source dashboarding solution that integrates well with PostgreSQL and supports multi-tenant access through separate database connections. Terraform automates the provisioning of Grafana datasources and dashboards, making it easy to manage configurations for multiple tenants.

### User Experience

**MFB Administrators**:

- Access global dashboards showing cross-tenant analytics
- Manage tenant access through Terraform configuration updates

**State Partners (Tenant Users)**:

- Access Grafana dashboards using tenant-specific credentials
- View analytics filtered automatically to their state's data

**Engineers**:

- Create analytics tables using dbt with automatic RLS via post-hooks
- Extend dashboards by modifying JSON templates in the `grafana/dashboards/` directory
- Add new tenants by updating `terraform.tfvars` and running `terraform apply`
- Connect to local grafana instance via docker, allowing for easy dashboard development and testing without needing to provide credentials.

### Implementation

The platform consists of three main components working together to provide secure, multi-tenant analytics:

**1. dbt Data Transformation Layer**

- Transforms raw Django model data into clean, analytics-ready tables
- Uses staging, intermediate, and mart models to structure transformations
- Built-in dependency management to ensure correct build order
- Includes automated tests to validate data quality and transformation logic
- Implements row-level security policies automatically via post-hooks
- Provides macros for creating tenant-specific database users with appropriate RLS settings
- Maintains data lineage and documentation for all transformations

**2. Grafana Visualization Layer**

- Provides interactive dashboards for screening analytics
- Uses tenant-specific database connections to enforce data isolation
- Configurable through JSON templates with variable substitution

**3. Terraform Infrastructure Layer**

- Automates provisioning of Grafana datasources and dashboards
- Manages tenant configurations and database credentials securely
- Enables easy addition of new tenants through configuration updates
- Maintains infrastructure state for reproducible deployments

### Technical Details

**Model Stucture**:
Dbt uses a layered approach with staging models to clean raw data, intermediate models for complex transformations, and mart models for final analytics tables. Each layer builds on the previous one, ensuring modularity and maintainability.

**Row-Level Security Implementation**: Each analytics table includes a post-hook that calls the `setup_white_label_rls()` macro. This macro creates PostgreSQL policies that automatically filter rows based on the current user's `rls.white_label_id` setting. Admin users are created with `BYPASSRLS` privilege to access all data.

**Tenant User Management**: The `create_rls_user()` macro automates creation of database users with appropriate permissions and RLS settings. Regular users get their `white_label_id` set as a user-level configuration, while admin users bypass RLS entirely.

**Materialization Strategy**: dbt has several [materialition strategies](https://docs.getdbt.com/docs/build/materializations). By default, models are regular views. Some pieces of advice from dbt:

- Generally start with views for your models, and only change to another materialization when you notice performance problems.
- Use the table materialization for any models being queried by BI tools, to give your end user a faster experience
- Consider materialized views for use cases where incremental models are sufficient, but you would like the data platform to manage the incremental logic and refresh.

Based on these pieces of advice, I would suggest we use views for staging and intermediate models, and tables for final marts that will be queried by Grafana.

### Code Examples

#### Models

A dbt staging model to provide unique screens, `stg_screens.sql`:

```sql
-- dbt staging model for unique screens
{{
  config(
    materialized='view'
  )
}}

-- Remove duplicates: Some records can share the same `uuid` due to pulling validations.
-- We keep a single row per `uuid`, choosing the most recent `submission_date`
-- (tie-broken by highest `id`). This guarantees unique uuids for downstream
-- models and satisfies the uniqueness tests.
WITH filtered AS (
    SELECT
        id,
        uuid,
        completed,
        submission_date,
        start_date,
        white_label_id,
        household_size,
        household_assets,
        housing_situation,
        zipcode,
        county,
        is_test,
        is_test_data
    FROM {{ source('django_apps', 'screener_screen') }}
    WHERE
        -- Only include completed screeners
        completed = true
        -- Filter out test data (check both is_test and is_test_data)
        AND (is_test = false OR is_test IS NULL)
        AND (is_test_data = false OR is_test_data IS NULL)
        -- Only include records with submission dates
        AND submission_date IS NOT NULL
), deduped AS (
    SELECT
        *,
        ROW_NUMBER() OVER (
            PARTITION BY uuid
            ORDER BY submission_date DESC, id DESC
        ) AS row_num
    FROM filtered
)

SELECT
    id,
    uuid,
    completed,
    submission_date,
    start_date,
    white_label_id,
    household_size,
    household_assets,
    housing_situation,
    zipcode,
    county,
    is_test,
    is_test_data,
    -- Add date parts for easier aggregation
    DATE(submission_date) as submission_date_only,
    EXTRACT(YEAR FROM submission_date) as submission_year,
    EXTRACT(MONTH FROM submission_date) as submission_month,
    EXTRACT(DAY FROM submission_date) as submission_day,
    EXTRACT(DOW FROM submission_date) as submission_day_of_week
FROM deduped
WHERE row_num = 1

SELECT * FROM {{ ref('stg_screens') }}
```

with a schema.yml file to define tests:

```yaml
models:
  - name: stg_screens
    description: "Staging model for screener data, cleaned and filtered"
    columns:
      - name: id
        description: "Primary key from screener_screen"
        tests:
          - not_null
          - unique
```

#### RLS for white-label data isolation

Macro to set up RLS policies, `setup_white_label_rls.sql`:

```sql
{% macro setup_white_label_rls(table_name, white_label_column='white_label_id', schema_name=none) %}

  {% set full_table_name %}
    {% if schema_name %}{{ schema_name }}.{{ table_name }}{% else %}{{ target.schema }}.{{ table_name }}{% endif %}
  {% endset %}

  {% set policy_name %}rls_white_label_{{ table_name }}{% endset %}

  -- Enable RLS on the table
  ALTER TABLE {{ full_table_name }} ENABLE ROW LEVEL SECURITY;

  -- Drop existing policy if it exists
  DROP POLICY IF EXISTS {{ policy_name }} ON {{ full_table_name }};

  -- Create RLS policy that filters by user's white label setting
  CREATE POLICY {{ policy_name }}
    ON {{ full_table_name }}
    FOR ALL
    TO PUBLIC
    USING (
      {{ white_label_column }} = COALESCE(
        NULLIF(current_setting('rls.white_label_id', true), '')::integer,
        -999999  -- Deny access if no white_label_id is set
      )
    );

  -- Grant necessary permissions
  GRANT SELECT ON {{ full_table_name }} TO PUBLIC;

{% endmacro %}
```

Configure RLS on a mart model, `mart_screen_eligibility.sql`:

```sql
-- dbt mart table model with RLS enabled
{{
  config(
    materialized='table',
    description='Mart model summarizing benefit eligibility for each completed screen',
    post_hook="{{ setup_white_label_rls(this.name) }}"
  )
}}
```

```bash
# Create tenant user with RLS
dbt run-operation create_rls_user --vars '{"username": "nc", "password": "secure_password", "white_label_access": 1}'
```

#### Materialization configuration

Dbt materialization configuration in `dbt_project.yml`:

```yaml
# configure dbt materialization strategies to view by default, marts as tables.
models:
  benefits_dbt:
    +materialized: view
    marts:
      +materialized: table
```

#### Terraform dashboards

```
# Global dashboard
resource "grafana_dashboard" "global" {
  depends_on = [grafana_data_source.global_postgres]

  config_json = templatefile("${path.module}/../grafana/dashboards/global.json.tpl", {
    datasource_uid = grafana_data_source.global_postgres.uid
  })
  overwrite = true
}

# Tenant-specific dashboards
resource "grafana_dashboard" "tenant_dashboards" {
  for_each = var.tenants
  depends_on = [grafana_data_source.tenant_postgres]

  config_json = templatefile("${path.module}/../grafana/dashboards/tenant.json.tpl", {
    tenant_name         = each.value.name
    tenant_display_name = each.value.display_name
    datasource_uid      = grafana_data_source.tenant_postgres[each.key].uid
  })
  overwrite = true
}
```
