# Solution.md template

Translate the headings into the user's language. Remove a section only if it truly doesn't apply, and say so in one line rather than leaving it silently empty.

````markdown
# Solution: <requirement title>

> Status: Explore | Brainstorm | Plan | Solve | Verified
> Last updated: <date>

## 1. Requirement check
Original requirement:
> <quoted verbatim>

| ID | Requirement | Source |
|----|-------------|--------|
| R1 | ... | user-confirmed / found-in-code / assumption |

## 2. Constraints
- Technical: <stack, existing structure, infrastructure, conventions found in code>
- Business: <rules decided by the user>
- Out of scope: <explicitly excluded>

## 3. Decisions & assumptions
| # | Question | Decision | Source |
|---|----------|----------|--------|
| D1 | ... | ... | user-confirmed / assumption (user delegated) |

## 4. Solution
**In one line:** <e.g., "Use a Redis distributed lock keyed by order ID so only one rider can accept an order.">

### Why this approach
<reasons tied to the user's goals and the existing code>

### Alternatives considered
| Option | Pros | Cons | Why not chosen |
|--------|------|------|----------------|

## 5. Plan
| Step | What to build | Where (files/modules) | Covers | Verified by |
|------|---------------|-----------------------|--------|-------------|
| 1 | ... | `app/...` | R1, R2 | V1, V2 |

## 6. Solution guide
- How it fits the existing code
- Where the business rules should live
- Edge cases and how to handle each

Key snippets (illustrative, not a full implementation):
```<language>
...
```

## 7. Verification criteria
| ID | Covers | Scenario (input) | Expected result |
|----|--------|------------------|-----------------|
| V1 | R1 | ... | ... |

## 8. Discovered exceptions
| # | Exception | Impact | How to handle |
|---|-----------|--------|---------------|
| E1 | ... | ... | ... |

## 9. Self-check
- [ ] Every R-ID is covered by the plan, guide, execution prompt, and verification criteria
- [ ] Every decision matches what the user confirmed
- [ ] Every referenced file/class/field exists in the code (or is marked as new)
- [ ] Worked examples compute correctly

## 10. Execution prompt
Paste this into the tool you build with (Claude Code, Cursor, another agent, or hand it to a developer).

```text
<the prompt>
```
````

## Writing the execution prompt

The reader has none of your context and you don't know which tool runs it. Make it self-contained, specific, and tool-neutral. Include, in this order:

1. **Goal** — one or two sentences on what to build and why.
2. **Context** — stack, and the exact files, classes, and fields involved (from Explore), so the implementer doesn't have to rediscover them.
3. **Confirmed rules** — every business decision as concrete values, with R-IDs. No "TBD"; if something is unresolved, it doesn't belong in the prompt yet.
4. **Approach** — the chosen solution and the key design points (where rules live, how edge cases are handled). Include a key snippet only if it prevents a likely mistake.
5. **Steps** — the plan, in order.
6. **Constraints** — what not to change, conventions to follow, out-of-scope items.
7. **Done when** — the verification criteria (V-IDs) as tests or checks the implementer must pass, and what to report back.

Don't include tool-specific commands (slash commands, IDE actions). Describe the outcome and how to check it.
