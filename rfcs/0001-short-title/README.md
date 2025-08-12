# RFC 0001: Example RFC

Status: Accepted
Author: Jane Doe
Created: 2025-01-15
PR: #1

## Summary

This is an example RFC that demonstrates the format and structure of proposals in this repository. It serves as a reference implementation showing authors what a completed RFC should look like, including the level of detail expected and proper formatting conventions.

## Background

The RFC (Request for Comments) process is a critical component of open-source project governance, providing a structured way to propose, discuss, and document significant changes. However, many contributors struggle with understanding what constitutes a well-written RFC without concrete examples to reference.

This RFC addresses the need for a reference implementation by providing a complete example that demonstrates all required sections, appropriate detail levels, and formatting conventions. Prior discussions in issue #42 highlighted that new contributors often submit incomplete RFCs or struggle with understanding the expected scope and detail. By having this example RFC in the repository, we establish clear expectations and reduce the iteration cycles needed for new proposals.

## Proposal

We propose maintaining this example RFC as a permanent fixture in the repository to serve as a reference implementation. The example demonstrates all required sections with appropriate placeholder content that illustrates the expected level of detail without being overly prescriptive about specific technical implementations.

### Abandoned Ideas

**Template-only approach**: We considered only providing a bare template without example content, but this was rejected because it doesn't give authors enough guidance on the appropriate level of detail and tone for each section.

## Implementation

The implementation involves creating and maintaining this example RFC with representative content for each section. The RFC will be stored in the `rfcs/0001-example/` directory and referenced from both the template and contributing documentation.

### Technical Details

- Location: `rfcs/0001-example/README.md`
- Status: Permanently maintained as "Accepted"
- Updates: Synchronized with template changes
- Cross-references: Linked from CONTRIBUTING.md and template

### Code Examples

```markdown
## Summary

[Your 1-2 paragraph overview here, explaining the goal 
without deep technical details]
```

## User Experience

New RFC authors will have a clear reference showing how to structure their proposals, what level of detail to include, and how to format different sections.

### Before

Authors would copy the bare template and often struggle with understanding what content belongs in each section, leading to multiple review cycles.

### After

Authors can reference this example RFC to understand expectations, resulting in higher-quality initial submissions and fewer review iterations.

## Security Considerations

This RFC has no security implications as it only provides documentation and examples.

## Performance Impact

No performance impact as this is purely documentation.

## Backwards Compatibility

This RFC introduces no breaking changes and is fully backwards compatible with existing RFC processes.

## Testing Strategy

The example RFC will be validated against the template structure in CI to ensure it remains synchronized with template updates.

## Documentation

- Update CONTRIBUTING.md to reference this example
- Link from the RFC template
- Include in onboarding documentation

## Rollout Plan

1. **Phase 1**: Merge this example RFC
2. **Phase 2**: Update all documentation to reference it
3. **Phase 3**: Monitor new RFC submissions for quality improvements

## Open Questions

- Should we maintain multiple examples for different types of RFCs (features, breaking changes, process)?
- How often should the example be updated to reflect best practices?

## Alternatives Considered

**Multiple examples**: We considered maintaining several example RFCs for different scenarios but decided a single comprehensive example would be less confusing for new contributors.

**External documentation**: Keeping examples in separate documentation was considered but rejected to keep everything in one place.

## References

- [Rust RFC Process](https://github.com/rust-lang/rfcs)
- [React RFCs](https://github.com/reactjs/rfcs)
- [Python PEP Process](https://www.python.org/dev/peps/)