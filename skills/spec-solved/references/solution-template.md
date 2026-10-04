# Solution.md template

Translate the headings into the user's language. Remove a section only if it truly doesn't apply, and say so in one line rather than leaving it silently empty.

```markdown
# Solution: <requirement title>

> Status: Explore | Brainstorm | Plan | Implement | Verify | Done
> Last updated: <date>

## 1. Requirement check
Original requirement:
> <quoted verbatim>

| ID | Requirement | Source |
|----|-------------|--------|
| R1 | ... | user-confirmed / found-in-code / assumption |

## 2. Constraints
- Technical: <stack, existing structure, conventions found in code>
- Business: <rules decided by the user>
- Out of scope: <explicitly excluded>

## 3. Decisions & assumptions
| # | Question | Decision | Source |
|---|----------|----------|--------|
| D1 | ... | ... | user-confirmed / assumption (user delegated) |

## 4. Proposals & trade-offs
### Option A: <name> (recommended)
- How it covers R-IDs:
- Pros / Cons:
### Option B: <name>
...
**Chosen:** <option> — <reason>

## 5. Plan
| Step | Description | Covers | Verified by |
|------|-------------|--------|-------------|
| 1 | ... | R1, R2 | unit test `...` |

## 6. Implementation summary
- Files changed / created:
- Where the business rules live:

## 7. Verification
| ID | Result | Evidence |
|----|--------|----------|
| R1 | ✅ / ⚠️ / ❌ | test name, command output, or worked example |

## 8. Discovered exceptions
| # | Exception | Impact | Resolution |
|---|-----------|--------|------------|
| E1 | ... | ... | ... |
```
