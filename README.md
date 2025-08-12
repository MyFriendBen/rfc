# RFC (Request for Comments) Repository

This repository contains RFCs for proposing and discussing significant changes to our project.

## What is an RFC?

An RFC (Request for Comments) is a formal proposal for making substantial changes to the project. RFCs provide a consistent and controlled path for introducing new features, significant refactors, or architectural decisions.

## When to use an RFC

Use an RFC when proposing:
- New features or APIs
- Breaking changes
- Architectural decisions
- Process changes
- Any change that significantly affects users or developers

## RFC Lifecycle

1. **Draft** - Initial proposal being written
2. **Discussion** - Open for community feedback
3. **Final Comment Period** - Last call for feedback (typically 7 days)
4. **Accepted** - Approved and ready for implementation
5. **Implemented** - Completed and shipped
6. **Rejected** - Not accepted (moved to archive)
7. **Withdrawn** - Withdrawn by author (moved to archive)
8. **Superseded** - Replaced by another RFC (moved to archive)

## How to Participate

### As an Author
1. Copy the template from `rfcs/0000-template/`
2. Create a new directory `rfcs/NNNN-short-title/`
3. Fill out the template
4. Submit a pull request
5. Engage with feedback during discussion

### As a Reviewer
- Comment on pull requests
- Ask questions for clarification
- Provide constructive feedback
- Help improve proposals

## Directory Structure

- `rfcs/` - All active and accepted RFCs
- `archive/` - Historical RFCs (rejected, withdrawn, superseded)
- `decisions/` - Lightweight ADRs for day-to-day choices
- `index.csv` - Machine-readable index of all RFCs
- `STATUS.md` - Detailed status definitions

## Quick Links

- [Contributing Guide](CONTRIBUTING.md)
- [RFC Template](rfcs/0000-template/README.md)
- [Status Definitions](STATUS.md)
- [RFC Index](index.csv)