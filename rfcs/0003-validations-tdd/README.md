# RFC 0003: Validations for TDD

Status: Draft
Author: Katie Brey, Sonali Bedge
Created: 2025-09-17
PR: #XXX

## Summary

[Validations](https://github.com/Gary-Community-Ventures/benefits-api/wiki/Validations) are how MFB detects regressions to eligibility results. When an admin tests a program and confirms it's working correctly, they can save that state as a validation. Later, when someone runs the validate command, MFB compares the current program behavior against these saved validations and alerts you if anything has changed unexpectedly.

These validations are very effective at catching regressions, but we want to extend their value to the development process, where engineers currently have to manually run test cases through the UI. SMEs provide test scenarios in written form (household characteristics, income, benefits, etc.), but engineers must then manually click through the screener UI to recreate these scenarios during development and testing. This same manual screener process is repeated again when SMEs create the official validations in staging, essentially duplicating the same work multiple times throughout the development lifecycle. Additionally, since validations are currently created after development is complete, we may discover issues only after significant development effort has been invested, making fixes more costly and time-consuming.

This workflow creates significant inefficiencies: engineers spend substantial time navigating UI forms during each development iteration, making comprehensive testing less likely, while SMEs must recreate scenarios they've already documented, creating deployment bottlenecks.

We would like to extend validation functionality to enable engineers to run test cases programmatically during development, eliminating the manual screener navigation that currently slows down both development iteration and validation creation.

## Background

### Current Process

**Step 1: Test Case Definition**  
Subject Matter Experts (SMEs) provide desired logic and informal test scenarios in written form. These typically include household characteristics such as:

- Household composition (number of people, relationships, ages)
- Income sources and amounts
- Existing benefit enrollments
- Geographic location and other eligibility factors

**Step 2: Initial Development**  
Engineers implement the new program's eligibility logic and benefit calculation rules in their local development environment based on the SME requirements.

**Step 3: Manual Testing via UI**  
Engineers manually recreate each SME-provided test scenario by clicking through the screener interface, entering all household details, income information, and benefit history to verify their implementation produces the expected results.

**Step 4: Iterative Development**  
As engineers refine the program logic, they must repeatedly navigate through the entire screener UI for each test case to validate their changes. This creates a significant time burden, especially when testing multiple scenarios or making frequent adjustments.

**Step 5: Staging Validation**  
Once code is merged to staging, SMEs manually execute the same test scenarios through the screener interface to create official validations. These validations serve as regression detection benchmarks for future changes.

### Problem Statement

Our current workflow is heavily dependent on manual UI interaction for both developers and SMEs. This creates several inefficiencies:

For Developers:

- Significant time overhead clicking through screener forms for each test iteration
- Difficulty testing edge cases and boundary conditions due to manual setup burden
- Reduced likelihood of comprehensive testing during active development
- Slower feedback loops when refining eligibility logic

For SMEs:

- Duplicate effort recreating the same scenarios they originally documented
- Limited ability to provide structured, reusable test cases

### Desired Solution

We need a streamlined approach that:

- Enables faster developer testing by reducing manual screener navigation during development iterations
- Empowers SMEs to create structured, reusable test cases that can be programmatically executed across development, staging, and production environments
- Bridges the gap between written requirements and executable tests to reduce translation errors and duplication of effort

## Proposal

### Overview

SMEs create test cases by specifying screener inputs and expected outputs in JSON, which are attached to a ticket before implementation begins. An engineer runs a script to import these test cases as validations that can be used to verify program logic during development, as well as in staging and production environments.

### Suggested Process

**Step 1: Test Case Definition**  
Subject Matter Expert (SMEs) creates JSON files that capture screen inputs and expected estimated values. One file can contain multiple test cases, as programs likely require several scenarios.

Imagine a program called “new program” that requires the household to have a child under the age of 2\.

Example structure:

\[  
{  
 "test_id": "nc_new_program_child_under_2_001",  
 "household": { ...payload mapped to ScreenSerializer... },  
 "expected_results": { "eligibility": true, "benefit_amount": 400 }  
}  
{  
 "test_id": "nc_new_program_no_children_001",  
 "household": { ...payload mapped to ScreenSerializer... },  
 "expected_results": { "eligibility": false, "benefit_amount": 0 }  
}

\]

Note on test_id: The test_id is only used as metadata to help the json writer build the test cases. This proposal does not include saving this field to the database, although it is mentioned in the [idea to mitigate the duplicate validation risk](#risks).

**Step 2: Engineer sets up the program**  
Engineer creates the new program in their local environment. The program must exist in order to create validations for it. For this example, the external name of the program is new_program.

**Step 3: Import Validations from JSON**  
Engineer runs import_validations script to create validations from the SME test cases. The script expects the path to the JSON file, and the external name of the program. The external name is unique across white labels, so we do not need to pass the white label id.

python manage.py import_validations test-cases-from-SME.json new_program

The import_validations script will:

1. Validate the arguments: run the json file through a schema validator and ensure a program with a matching external_name exists. Enforce a limit on the number of screens that can be created (10 at a time).
2. Loop through each test case in the JSON file. For each test case:
   1. Create a screen record from the household ScreenSerializer payload (for an existing example of this, see the pull_screen.py script)
      1. The screen will be set to is_test \= true so that they cannot show up in analytics.
   2. Create a validation record with a foreign key to the associated screen_id. Populate the validation’s program name, eligible boolean, and value from the json inputs.
3. Output a summary including the results of which inputs passed/failed and links to the screens that were created.

**Step 4: Iterative Development and Validation**  
Engineer implements the new program's eligibility logic and benefit calculation rules in their local development environment based on the SME requirements.

Engineer runs the validate script to run the test cases:

python manage.py validate new_program

If any of the test cases fail, they will be listed in the output of the script. The engineer will continue iterating on logic until all test cases pass.

**Step 5: Staging/Production Validation**  
After the code is approved and merged to staging, in the Heroku terminal of the staging environment, an engineer will run the import_validations script with the same SME-provided JSON test cases.

**\[Heroku terminal\]**  
python manage.py import_validations test-cases-from-SME.json new_program

To get the validations to production, an engineer can either pull the validations from staging or run the import_validations script (this can be determined by what fits best into the current process)

### Risks {#risks}

**Duplicate validations:** If a json file is imported multiple times, we could end up with duplicate validations.

Ideas:

- Require unique ‘test_id” in each test case (unique constraint). In the import script ensure that the test_id field is unique so that the same test cases cannot be recreated.

**Incorrect validations:** If a user passes the wrong program name, we could end up with validations in the wrong program, without a good way to clean it up.

Ideas:

- Improve the validations view in django admin to allow us to more easily manage and delete validations.

**Database Performance Impacts:** As we scale and use this heavily, we could end up with a lot of test screens in the database. This could cause performance issues when fetching a real user's results. This could be avoided with more traditional testing approaches that run outside of production data.

Ideas:

- Don’t keep the test cases in the database, but instead generate screens dynamically at runtime, passing in the json files.
- Use a separate database table for validation screens.
- Before creating a new screen, check that the same household doesn’t already exist in a test screen (could even hash the screen if we want this to be fast). Reuse the screen for the new validation.
