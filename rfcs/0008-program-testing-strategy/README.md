# RFC 0008: Program Testing Strategy

Status: Discussion
Author: Patrick Wey
Created: 2025-10-15
PR: [#6](https://github.com/MyFriendBen/rfc/pull/6)

## Summary

This RFC aims to establish an approach, including standards, tooling, and processes, for testing new benefit programs. This will ensure reliability and maintainability as we scale program additions across multiple states and white label configurations.

## Background

As we scale our screener, program additions are one of our primary feature development activities. Currently, testing for new programs is ad-hoc and inconsistent, leading to several challenges:

**Current State**: Programs are tested manually during development and QA. Limited automated test coverage exists for eligibility calculations, value computations, and program-specific business logic. This creates risk when modifying shared code or refactoring program logic.

**Problems**: Without systematic testing, we lack confidence in initial releases and subsequent changes. Regressions in program eligibility go undetected until production. Manual testing effort scales linearly with program count, becoming unsustainable.

**Motivation**: Implementing a comprehensive testing strategy now—before further scaling—will compound benefits: faster development cycles, reduced production bugs, and easier refactoring. This RFC establishes testing as a first-class requirement for program development.

## Proposal

Implement a testing strategy that emphasizes unit testing (Unit > Integration > E2E) and with specific standards for backend and frontend code. Use fixture-based testing to isolate external API dependencies (PolicyEngine) and enable fast, reliable test execution.

## Implementation

### Backend Testing

**Unit Tests**: Test program eligibility and value calculation logic in isolation.

**Integration Tests**: Test API endpoints, multi-model interactions, and service layer operations. Use `vcrpy` fixtures for PolicyEngine calls. Verify correct request formation and response handling.

### Frontend Testing

**Unit Tests**: 
There isn't much (if any) FE code added per program, but there are a few core behaviors that we should add tests for as a one-off task:
- Test pure business logic functions for income calculations (`calcIncomeStreamAmount`, `calcMemberYearlyIncome`, `calcTotalIncome`) and value formatting (`programValue`, `calculateTotalValue`).

**Integration Tests**: Test english translations, program configuration, and "current benefit" flows. Use Playwright with HAR recording for backend API fixtures to enable offline testing.

### E2E Testing

Ues existing patterns to write a limited number smoke tests per white label configuration. Tests run against dev/staging databases without fixtures to catch integration issues. Implement a periodic cleanup job to remove test records.

### Code Coverage

**Tools**:
- Backend: `coverage.py` for Python coverage reporting
- Frontend: Istanbul/nyc (existing)
- CI: CodeCov (free for open source) for tracking trends and (optionally) README badges

## Rollout Plan

## Phase 1
- Ensure all tests run in CI
    * Add GH workflow to run BE test suite
    * Add GH workflow to run FE unit tests
    * Fix/Comment out existing test failures
- Add coverage tooling
    * Add `coverage.py` for BE coverage reporting
    * Add GH workflow to upload BE coverage report to CodeCov
    * Add GH workflow to upload FE coverage report to CodeCov
- Add TX programs using testing strategy
    * Add `vcrpy` for API fixture generation
    * Add fixture refresh automation

## Phase 2
- Enforce testing coverage requirement for new code
- Add unit tests for core FE logic (not program/white-label specific)
- Translate validations into tests (using TX as a guide)
- Implement E2E test data cleanup job

## Open Questions

1. **Fixture refresh cadence**: Nightly vs. weekly? Weekly reduces API load but may miss breaking changes sooner.
2. **Test data cleanup frequency**: How often should we purge E2E test records from dev/staging databases?
3. **Phase 2 coverage threshold**: What's realistic for new code and a codebase-wide minimum?
4. **Reflect**: Do we want to keep this around? I say no. It's expensive and we now have exclusively developers who don't know the tool maintaining these no-code tests.

## References

- Testing Pyramid: https://martinfowler.com/articles/practical-test-pyramid.html
- vcrpy documentation: https://vcrpy.readthedocs.io/
- Playwright HAR recording: https://playwright.dev/docs/network#record-and-replay-network-traffic
- Django testing best practices: https://docs.djangoproject.com/en/stable/topics/testing/
