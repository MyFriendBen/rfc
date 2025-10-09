# RFC 0001: Automated Staging Deployment and Feature Flag Integration
Status: Draft
Author: Josh Mejia
Created: 2025-10-09
PR: #XXX

## Summary

This RFC proposes modernizing our deployment process by eliminating the dev branch, implementing automated staging deployments, and potentially integrating LaunchDarkly (or similar) for feature flag management (we currently have a custom feature flagging solution). The primary goal is to reduce manual deployment overhead while maintaining code quality and enabling safer and faster feature releases through progressive rollouts.

The implementation will be phased, starting with automated staging deployments triggered by feature branch merges to main, followed by production deployment automation via GitHub releases, and finally full LaunchDarkly integration for both frontend (React/TypeScript) and backend (Django) applications.

## Background

Our current deployment process involves manual steps that create bottlenecks and potential for human error. Developers must manually deploy to staging after merging to the dev branch, manually execute post-deployment scripts, and manually deploy to production from the main branch.

The current workflow creates several pain points: the dev branch often accumulates work-in-progress commits that complicate release preparation, manual deployments are time-consuming and error-prone.

Faster QA cycles on feature development will improve development velocity and minimize bottlenecks. The team has also expressed interest in reducing the manual overhead of deployments.

## Proposal

We propose implementing automated deployments for our current workflow initially, then gradually simplifying our branching model and leveraging feature flag management across both frontend and backend repositories.

The core approach involves first automating our existing dev branch workflow to gain confidence in the deployment automation, then transitioning to a simplified feature branch → main model, using GitHub releases to trigger production deployments, and leveraging feature flags to safely deploy incomplete or experimental features. This creates a low-risk migration path to a streamlined pipeline where code flows predictably from development to production with appropriate gates and controls.

### Open Questions

- How do we prevent merging to main from interfering with release prep?
  - Lowest-lift: Slack message asking the team to hold off on merging until the release is out? (what happens if someone doesn't see the Slack message?)
  - Have the release coordinator block merges at the repo admin level?
    - A little annoying but still a lower lift solution than existing manual deploys

### Alternatives Considered

- We considered maintaining the dev branch with automated deployments, but this would perpetuates unnecessary complexity.
- We also considered a flow that utilizes release branches

## Implementation

The implementation consists of four phases: automated staging deployment with current workflow, branching model simplification, production deployment automation, and comprehensive feature flag integration.

### Technical Details

**Phase 1a: Automated Staging Deployment**
- Configure GitHub Actions workflow triggered on pushes to dev branch
- Deploy both frontend and backend to respective staging environments on Heroku
- Execute post-deployment scripts automatically
- Implement notification system for deployment status
- Maintain existing feature branch → dev → main workflow

**Phase 1b: Simplified Branching Model**
- After automation is proven reliable, eliminate dev branch
- Update GitHub Actions to trigger on pushes to main branch
- Transition to feature branch → main workflow
- Establish main branch protection rules

**Phase 2: Production Deployment via Releases**
- Extend GitHub Actions to trigger on release publication
- Deploy to production environments with appropriate safeguards
- Implement rollback capabilities through release management

**Phase 3: LaunchDarkly Integration (if needed)**
- Configure LaunchDarkly for both repositories
- Implement feature flag evaluation in React frontend
- Integrate feature flag evaluation in Django backend
- Establish feature flag governance and lifecycle management
