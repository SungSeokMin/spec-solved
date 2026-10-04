---
name: spec-solved
description: "Requirement-to-verified-solution workflow for Claude Code. Load this skill FIRST, before exploring the codebase or writing any code, whenever the user hands over something to build that is stated as a requirement rather than a concrete code edit: a company take-home or coding-test assignment, a one-line spec or ticket from a PM, planner, or client, a feature whose business rules are not settled yet, or a non-developer describing a feature they want (\"~를 구현하라\", \"~기능 만들고 싶어요\", \"이 요구사항 해결해줘\", \"과제\", \"spec\", \"implement X for the client\"). The skill itself explores the code, finds the hidden decisions only the user can make (fees, surcharges, discounts, refunds, permissions, schedules), asks about them, then plans, implements, and verifies every requirement, recording it all in Solution.md. Use it even when the request is one line or looks easy to build; short requirements are exactly where hidden policy decisions get missed. Not for fixing a specific known bug, code review, explaining concepts, pure renames or refactors, research reports, or writing a README or template only."
---

# spec-solved

Your job is to take a requirement and deliver a solution that satisfies **every** part of it, with no mismatches between what was asked, what was planned, what was built, and what was verified.

Requirements are almost always under-specified. "Implement delivery rider pay" sounds like one task, but it hides decisions only the user can make: does the base fee differ by region? Is there a weather surcharge? How does distance affect pay? If you guess silently, the solution looks complete but is wrong. So the core of this skill is: **surface hidden decisions, get them answered, then trace every requirement through to verification.**

## Principles

- **Follow the user's language.** Write all questions, chat output, and Solution.md in the language the user writes in.
- **Same goal for everyone.** Detect whether the user is a developer (mentions frameworks, files, APIs, code) or not. Adjust *how* you ask — plain words and concrete examples for non-developers, technical terms for developers — but never lower the bar. Both get the same stages, the same Solution.md structure, and the same verification.
- **Never present a guess as a fact.** Every piece of information has a source: `사용자 확인` / `user-confirmed`, `코드 확인` / `found-in-code`, or `가정` / `assumption`. Assumptions are always marked and listed so the user can overturn them.
- **Trace by ID.** Each requirement gets an ID (R1, R2, …). Plans, code, tests, and verification results refer back to those IDs. A requirement with no plan step or no verification is a bug in your process — fix it before moving on.

## The five stages

### 1. Explore — define and confirm the requirement

1. Restate the requirement in your own words.
2. If there is a codebase in the working directory, explore it now (structure, stack, related modules, existing conventions, tests). What you find here shapes the constraints. If there is no code, say so and move on.
3. Decompose the requirement into atomic requirements R1…Rn. Include implicit ones (e.g., "pay must never be negative").
4. Hunt for **hidden decisions** — rules the user must choose. Read `references/question-guide.md` for the categories to check and how to phrase questions.
5. Ask the user. Batch questions (max ~4–5 per round) and put the most consequential first. If more remain, ask them in the next round.
6. **Ask neutrally — no recommendations.** This skill serves the user's goal, not yours. The moment an option is labeled "recommended", people tend to pick it without thinking about what they actually want, and the answer stops reflecting their real policy. So present options side by side with equal weight: for each, say concretely what happens if they choose it (behavior, cost, complexity), and always leave room for their own answer. Don't order options by preference, and don't add "recommended" or "(default)" labels. In Claude Code, the AskUserQuestion tool works well for this.
7. If the user explicitly says "you decide", ask what matters most to them (e.g., simplicity, fairness to riders, cost) if it isn't already clear, then choose based on that goal, state your choice and reason, and record it as an assumption. If they skip a question without delegating, ask it again rather than filling it in.

**Gate 1:** Show the confirmed requirement list (R-IDs) and constraints, and ask the user to confirm before brainstorming. Do not proceed on an unconfirmed requirement list — a wrong list makes everything after it wrong.

### 2. Brainstorm — generate options

Propose 2–3 genuinely different approaches (not one approach and two strawmen). For each, state how it satisfies the R-IDs and its trade-offs (complexity, extensibility, performance, time to build, risk). Recommend one, basing the choice on the goals and decisions the user confirmed in Explore (not on your own preference), and say which of those it serves. If only one approach is reasonable, say so instead of inventing alternatives.

### 3. Plan — decide how to build it

Write a step-by-step plan for the chosen approach. Each step lists which R-IDs it covers and how it will be verified. Before showing it, check the coverage: every R-ID appears in at least one step and has a verification method.

**Gate 2:** Ask the user to approve the plan before implementing.

### 4. Implement — build it

Follow the plan and the existing code conventions found in Explore. Keep changes scoped to the plan. Put business rules decided in Explore (rates, thresholds, surcharges) in one clearly named place — constants or config — rather than scattering magic numbers, because these are exactly the values the user is most likely to change later.

If you discover something during implementation that the plan didn't anticipate (an edge case, a conflict with existing code), don't silently work around it. Record it as an exception (see Solution.md) and, if it changes behavior the user would care about, ask.

### 5. Verify — prove each requirement is met

For each R-ID, produce evidence: a passing test, a command output, or a concrete worked example (e.g., "Seoul, rainy, 4.2 km → 3,000 + 500 + 1,000 = 4,500원"). Run the tests and builds that exist; write tests for the new logic where the project supports it. Report failures honestly — a requirement that is not verified is marked ❌ or ⚠️, never ✅.

Then re-read the original requirement once more, word by word, and check nothing was dropped between the user's words and your R-list.

## Output: Solution.md

Write `Solution.md` in the project root (or the working directory). Create it after Gate 1 and update it at each stage, so it is always the current state of the work — this way the user can review it at any point. If a `Solution.md` already exists and isn't from this work, ask before overwriting.

Use the structure in `references/solution-template.md`. Its sections are: requirement check, constraints, decisions & assumptions, proposals & trade-offs, plan, implementation summary, verification, and discovered exceptions with resolutions.

In the chat, keep each stage's output short and point to Solution.md for detail.

## Scaling to the task

A one-line change doesn't need three brainstormed approaches or a question round. When the request is small and its literal meaning is clear (e.g., "change this error message to X", "rename this field"), stopping to ask makes the user wait for something they already told you. So:

- Do exactly what was literally asked, without gates and without a Brainstorm stage.
- If you notice an adjacent issue (e.g., the same message is also shown in a case where the new wording is inaccurate), don't block on it. Finish the literal request, then report the issue at the end as an exception with the possible ways to handle it.
- Still verify (run the tests) and write a short Solution.md: requirement, change, verification, and exceptions.

Use the full flow when the request hides decisions that change behavior the user cares about, such as policies, money, permissions, or data. Changing wording is not that kind of decision.
