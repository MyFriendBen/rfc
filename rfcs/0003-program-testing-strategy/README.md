# RFC 0003: Program Testing Strategy

Status: Draft
Author: Patrick Wey
Created: 2025-10-15
PR: #XXX

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

**Unit Tests**: Test program eligibility and value calculation logic in isolation. Record PolicyEngine API responses with `vcrpy` for deterministic, fast tests without external dependencies.

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

**Enforcement**:
- Phase 1: Report coverage without enforcement thresholds. Establish baseline and identify gaps.
- Phase 2: Enforce minimum coverage thresholds for new code (e.g. target: 90% for program logic). Codebase-wide threshold TBD based on Phase 1 data.

## User Experience

### Before
Developers add programs without standardized testing requirements. Bugs discovered in production or during manual QA cycles.

### After
Developers follow testing checklist when adding programs:
1. Write unit tests for eligibility/value logic
3. Add/Edit integration tests for API endpoints
4. Add/Edit E2E smoke test for program white label
5. CI validates coverage meets thresholds before merge

## Rollout Plan

**Phase 1**:
- Set up coverage tooling (`coverage.py`, CodeCov integration)
- Create developer documentation
- Establish baseline coverage metrics
- Implement fixture refresh automation
- Add test data cleanup job
- Begin adding tests to one example program as reference
- Implement coverage reporting (no enforcement)

**Phase 2**:
- Require testing for all new programs
- Evaluate baseline metrics and set Phase 2 thresholds
- Enforce coverage requirements

**Success Metrics**:
- Coverage trending upward month-over-month
- New programs ship with (e.g.) 90%+ test coverage

## Open Questions

1. **Fixture refresh cadence**: Nightly vs. weekly? Weekly reduces API load but may miss breaking changes sooner.
2. **Test data cleanup frequency**: How often should we purge E2E test records from dev/staging databases?
3. **Phase 2 coverage threshold**: What's realistic for codebase-wide minimum? 80%?
4. **Retrofit priority**: Should we prioritize adding tests to high-traffic programs even if not actively modifying them?
5. **Validations**: How do our validations play into this? IMO they still have a place as staging/production smoke tests configurable by non-technical users, but there's probably a lot of overlap
6. **Reflect**: Do we want to keep this around? I say no. It's expensive and we now have exclusively developers who don't know the tool maintaining these no-code tests.

## References

- Testing Pyramid: https://martinfowler.com/articles/practical-test-pyramid.html
- vcrpy documentation: https://vcrpy.readthedocs.io/
- Playwright HAR recording: https://playwright.dev/docs/network#record-and-replay-network-traffic
- Django testing best practices: https://docs.djangoproject.com/en/stable/topics/testing/
