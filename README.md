# Contribution [1]: [Issue Title]

**Contribution Number:** 1  
**Student:** Suhas Ramesh Vittal  
**Issue:** https://github.com/traceloop/opentelemetry-mcp-server/issues/11 
**Status:** Phase I  In Progress

---

## Why I Chose This Issue

I chose this issue since it also follows a critical pattern seen in data engineering projects. It is a good example of data aggregation to avoid sort in Apache Spark execution.

---

## Understanding the Issue


### Problem Description

This issue aims to reduce the number of tokens consumed by the observability data when queried by the AI agents.
For example, if you query 100 traces, you get 100 objects each repeating the same field names like "model", "provider", 
"count" over and over. That's a lot of wasted tokens.


### Expected Behavior
Return this

{
  "columns": ["model", "provider", "count"],
  "rows": [["gpt-4", "openai", 48], ["gpt-3.5", "openai", 12]]
}


### Current Behavior
 Instead of returning this:

[
  {"model": "gpt-4", "provider": "openai", "count": 48},
  {"model": "gpt-3.5", "provider": "openai", "count": 12}
]

<!--

### Affected Components

[Which parts of the codebase are involved?]

---

## Reproduction Process

### Environment Setup

[Notes on setting up your local development environment - challenges you faced, how you solved them]

### Steps to Reproduce

1. [Step 1]
2. [Step 2]
3. [Observed result]

### Reproduction Evidence

- **Commit showing reproduction:** [Link to commit in your fork]
- **Screenshots/logs:** [If applicable]
- **My findings:** [What you discovered during reproduction]

---

## Solution Approach

### Analysis

[Your analysis of the root cause - what's causing the issue?]

### Proposed Solution

[High-level description of your fix approach]

### Implementation Plan

Using UMPIRE framework (adapted):

**Understand:** [Restate the problem]

**Match:** [What similar patterns/solutions exist in the codebase?]

**Plan:** [Step-by-step implementation plan]
1. [Modify file X to do Y]
2. [Add function Z]
3. [Update tests]

**Implement:** [Link to your branch/commits as you work]

**Review:** [Self-review checklist - does it follow the project's contribution guidelines?]

**Evaluate:** [How will you verify it works?]

---

## Testing Strategy

### Unit Tests

- [ ] Test case 1: [Description]
- [ ] Test case 2: [Description]
- [ ] Test case 3: [Description]

### Integration Tests

- [ ] Integration scenario 1
- [ ] Integration scenario 2

### Manual Testing

[What you tested manually and results]

---

## Implementation Notes

### Week [X] Progress

[What you built this week, challenges faced, decisions made]

### Week [Y] Progress

[Continue documenting as you work]

### Code Changes

- **Files modified:** [List]
- **Key commits:** [Links to important commits]
- **Approach decisions:** [Why you chose certain approaches]

---

## Pull Request

**PR Link:** [GitHub PR URL when submitted]

**PR Description:** [Draft or final PR description - much of the content above can be adapted]

**Maintainer Feedback:**
- [Date]: [Summary of feedback received]
- [Date]: [How you addressed it]

**Status:** [Awaiting review / Iterating / Approved / Merged]

---

## Learnings & Reflections

### Technical Skills Gained

[What you learned technically]

### Challenges Overcome

[What was hard and how you solved it]

### What I'd Do Differently Next Time

[Reflection on your process]

---

## Resources Used

- [Link to helpful documentation]
- [Tutorial or Stack Overflow post that helped]
- [GitHub issues or discussions that helped]
-->
