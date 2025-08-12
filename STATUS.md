# RFC Status Definitions

This document defines the canonical statuses for RFCs and their lifecycle.

## Status Flow

```
Draft → Discussion → Final Comment Period → Accepted → Implemented
                ↓                      ↓         ↓
            Withdrawn              Rejected  Superseded
```

## Status Definitions

### Draft
- **Description**: Initial proposal being written and refined
- **Duration**: No limit
- **Next Steps**: Move to Discussion when ready for feedback
- **PR State**: Open, marked as draft

### Discussion
- **Description**: Open for community feedback and iteration
- **Duration**: Typically 2-3 weeks
- **Next Steps**: Move to FCP when consensus is near
- **PR State**: Open, ready for review

### Final Comment Period (FCP)
- **Description**: Last opportunity for feedback before decision
- **Duration**: 7 days minimum
- **Next Steps**: Accept or Reject based on feedback
- **PR State**: Open, labeled with `rfc-fcp`
- **Requirements**: 
  - Announcement in relevant channels
  - No blocking concerns unresolved

### Accepted
- **Description**: Approved for implementation
- **Duration**: Until implemented
- **Next Steps**: Begin implementation
- **PR State**: Merged
- **Requirements**:
  - Approved by required reviewers
  - FCP completed without blocking concerns

### Implemented
- **Description**: Fully implemented and shipped
- **Duration**: Permanent
- **Next Steps**: May be superseded by future RFCs
- **PR State**: Merged
- **Requirements**:
  - Implementation complete
  - Documentation updated
  - Tests passing

### Rejected
- **Description**: Not accepted after review
- **Duration**: Permanent
- **Location**: Moved to `archive/rejected/`
- **PR State**: Closed
- **Reasons**:
  - Does not align with project goals
  - Better alternatives exist
  - Unresolved blocking concerns

### Withdrawn
- **Description**: Withdrawn by the author
- **Duration**: Permanent
- **Location**: Moved to `archive/withdrawn/`
- **PR State**: Closed
- **Reasons**:
  - Author no longer pursuing
  - Superseded by events
  - Needs fundamental rework

### Superseded
- **Description**: Replaced by a newer RFC
- **Duration**: Permanent
- **Location**: Moved to `archive/superseded/`
- **Note**: Links to superseding RFC
- **Requirements**:
  - New RFC explicitly supersedes this one
  - New RFC is accepted

## Status Transitions

### Moving to FCP
Requirements:
- Major concerns addressed
- Design is complete
- Implementation plan exists
- Key stakeholders have reviewed

### Accepting an RFC
Requirements:
- FCP completed (minimum 7 days)
- No unresolved blocking concerns
- Approved by required reviewers
- Implementation commitment exists

### Rejecting an RFC
Conditions:
- Blocking concerns cannot be resolved
- Consensus cannot be reached
- Proposal conflicts with project direction

## Special Cases

### Experimental RFCs
- May skip FCP for experimental features
- Marked with `experimental` label
- Re-evaluated after trial period

### Emergency RFCs
- For critical security or stability issues
- Shortened timeline (24-48 hour FCP)
- Requires special approval

### Informational RFCs
- Document existing practices
- May have simplified process
- Focus on accuracy over consensus

## Roles and Responsibilities

### Author
- Write and refine proposal
- Address feedback
- Drive consensus
- Coordinate implementation

### Reviewers
- Provide constructive feedback
- Identify issues and concerns
- Suggest improvements
- Vote on acceptance

### Maintainers
- Guide process
- Make final decisions
- Ensure process is followed
- Resolve conflicts

## Timeline Guidelines

- **Draft**: 1-2 weeks for initial writing
- **Discussion**: 2-3 weeks for feedback
- **FCP**: 7 days minimum
- **Total**: 4-6 weeks typical, varies by complexity

## Appeals Process

If an RFC is rejected and the author disagrees:
1. Author may request reconsideration with new information
2. Escalate to project leadership if needed
3. Final decision rests with project maintainers