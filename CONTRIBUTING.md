# Contributing to RFCs

## Quick Start for Authors

### 1. Before You Begin
- Check existing RFCs to avoid duplication
- Consider discussing your idea in an issue first
- Small changes may not need an RFC

### 2. Writing Your RFC
```bash
# Copy the template
cp -r rfcs/0000-template rfcs/NNNN-your-title

# Edit the RFC
cd rfcs/NNNN-your-title
vim README.md
```

Replace `NNNN` with the next available number.

### 3. RFC Template Structure
- **Title**: Clear, descriptive title
- **Summary**: One paragraph explanation
- **Motivation**: Why is this change needed?
- **Detailed Design**: Technical details
- **Drawbacks**: What are the downsides?
- **Alternatives**: What other options were considered?
- **Unresolved Questions**: What needs further discussion?

### 4. Submit Your RFC
1. Create a pull request with your RFC
2. The PR title should be "RFC NNNN: Your Title"
3. Tag relevant stakeholders for review
4. Engage with feedback constructively

## Quick Start for Reviewers

### What to Look For
- **Clarity**: Is the proposal clear and well-explained?
- **Completeness**: Are all sections adequately filled out?
- **Impact**: What are the effects on users and developers?
- **Alternatives**: Have alternatives been fairly considered?
- **Implementation**: Is the implementation plan realistic?

### How to Review
1. Read the entire RFC before commenting
2. Ask clarifying questions
3. Suggest improvements constructively
4. Consider the broader impact
5. Vote with reactions on the PR:
   - 👍 Approve
   - 👎 Request changes
   - 👀 Acknowledge (no strong opinion)

## RFC Process Timeline

- **Week 1-2**: Initial discussion and feedback
- **Week 3**: Address feedback, refine proposal
- **Week 4**: Final Comment Period (FCP)
- **After FCP**: Decision by maintainers

## Tips for Success

### For Authors
- Be specific but not overly detailed
- Include code examples where helpful
- Address concerns proactively
- Be responsive to feedback
- Keep discussions focused

### For Reviewers
- Be respectful and constructive
- Focus on the proposal, not the person
- Provide specific suggestions
- Acknowledge good ideas
- Help improve the proposal

## Getting Help

- Questions? Open an issue with the "question" label
- Need clarification? Ask in the RFC's PR
- Process issues? Contact maintainers listed in CODEOWNERS