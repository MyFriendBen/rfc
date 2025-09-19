# RFC 0004: Multi-Tenant Analytics Platform for MFB Screening Data

Status: Draft
Author: Katie Brey
Created: 2025-09-17
PR: TODO

## Summary

We propose an MFB Analytics Platform that replaces hard-to-maintain materialized views with a developer-friendly dbt-based data transformation pipeline. The new system provides reliable, testable analytics for global MFB insights and secure multi-tenant dashboards for white-label partners like North Carolina. Built with dbt, Grafana, and Terraform, it implements PostgreSQL row-level security to automatically enforce data isolation without application-level filtering.

## Background

MyFriendBen currently uses a set of [materialized views](https://github.com/MyFriendBen/data-queries) to provide analytics data for Grafana dashboards. These views aggregate screening data from the Django application's PostgreSQL database for our Looker Studio dashbaords. However, our current approach has significant maintainability and scalability challenges.

The materialized views are difficult to modify because they have complex interdependencies - changing one view often requires dropping and recreating multiple dependent views in the correct order. There's no systematic way to test data transformations, making it risky to implement changes. Version control of the view definitions is manual and error-prone. When views need to be recreated, developers must manually determine the dependency order and execute SQL scripts in sequence.

These limitations are particularly problematic as analytics requirements evolve. Global analytics need to show cross-tenant trends while white-label partners like North Carolina require secure, isolated access to only their data. The current materialized view approach lacks automated data isolation mechanisms, and makes it difficult to make updates to our analytics logic as we grow.

## Proposal

Replace materialized views with a dbt-based data transformation pipeline that provides dependency management, automated testing, and version control. Implement PostgreSQL row-level security to enable both global analytics (priority) and secure multi-tenant access for white-label partners.

Dbt defines data transformations as code with automatic dependency resolution, built-in testing capabilities, and git-based version control. For multi-tenancy, PostgreSQL RLS policies automatically filter data based on database user credentials, eliminating the need for application-level filtering and reducing the risk of data leakage.

## Implementation

The platform consists of three main components working together to provide secure, multi-tenant analytics:

**1. dbt Data Transformation Layer**

- Stages raw Django model data into clean, analytics-ready tables
- Implements row-level security policies automatically via post-hooks
- Provides macros for creating tenant-specific database users with appropriate RLS settings
- Maintains data lineage and documentation for all transformations

**2. Grafana Visualization Layer**

- Provides interactive dashboards for screening analytics
- Uses tenant-specific database connections to enforce data isolation
- Supports both global and tenant-specific dashboard views
- Configurable through JSON templates with variable substitution

**3. Terraform Infrastructure Layer**

- Automates provisioning of Grafana datasources and dashboards
- Manages tenant configurations and database credentials securely
- Enables easy addition of new tenants through configuration updates
- Maintains infrastructure state for reproducible deployments

### Technical Details

**Row-Level Security Implementation**: Each analytics table includes a post-hook that calls the `setup_white_label_rls()` macro. This macro creates PostgreSQL policies that automatically filter rows based on the current user's `rls.white_label_id` setting. Admin users are created with `BYPASSRLS` privilege to access all data.

**Tenant User Management**: The `create_rls_user()` macro automates creation of database users with appropriate permissions and RLS settings. Regular users get their `white_label_id` set as a user-level configuration, while admin users bypass RLS entirely.

### Code Examples

```sql
-- dbt model with RLS enabled
{{
  config(
    materialized='table',
    post_hook="{{ setup_white_label_rls(this.name) }}"
  )
}}

SELECT * FROM {{ ref('stg_screens') }}
```

```bash
# Create tenant user with RLS
dbt run-operation create_rls_user --vars '{"username": "nc", "password": "secure_password", "white_label_access": 1}'
```

## User Experience

**Administrators**:

- Access global dashboards showing cross-tenant analytics
- Manage tenant access through Terraform configuration updates

**State Partners (Tenant Users)**:

- Access Grafana dashboards using tenant-specific credentials
- View analytics filtered automatically to their state's data

**Developers**:

- Add new tenants by updating `terraform.tfvars` and running `terraform apply`
- Create analytics tables using dbt with automatic RLS via post-hooks
- Extend dashboards by modifying JSON templates in the `grafana/dashboards/` directory
- Connect to local grafana instance via docker, allowing for easy dashboard development and testing without needing to provide credentials.
