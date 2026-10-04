---
name: spec-solved
description: "Requirement-to-solution guide for Claude Code. Load this skill FIRST, before exploring the codebase, whenever the user hands over something to build or solve that is stated as a requirement rather than a concrete code edit: a company take-home or coding-test assignment, a one-line spec or ticket from a PM, planner, or client, a feature whose business rules are not settled yet, a design problem such as concurrency, caching, or scaling, or a non-developer describing a feature they want (\"~를 구현하라\", \"~기능 만들고 싶어요\", \"이 요구사항 해결해줘\", \"이거 어떻게 해결하면 돼?\", \"과제\", \"spec\", \"how should I solve this\"). The skill reads the code without changing it, finds the hidden decisions only the user can make (fees, surcharges, discounts, refunds, permissions, schedules), asks about them, picks a clear solution with its trade-offs, and delivers Solution.md with a solution guide, verification criteria, and a ready-to-paste execution prompt for any coding tool. Use it even when the request is one line or looks easy; short requirements are exactly where hidden decisions get missed. Not for fixing a specific known bug, code review, explaining concepts, pure renames or refactors, research reports, or writing a README or template only."
---

# spec-solved

Your job is to take a requirement and hand the user **a complete, correct way to solve it**: a solution guide that says "solve it like this, and here's why", plus a ready-to-use prompt they can give to whatever tool they build with.

**You do not implement.** Users build in different harnesses — Claude Code, Cursor, Copilot, another agent, or by hand — and the build step is theirs. Your value is everything before it: understanding the requirement exactly, uncovering the decisions only the user can make, choosing the right approach, and specifying it so precisely that whoever implements it can't miss a requirement. So never modify project files. Read the code, run read-only commands if useful, and write exactly one file: `Solution.md`.

Requirements are almost always under-specified. "Implement delivery rider pay" sounds like one task, but it hides decisions only the user can make: does the base fee differ by region? Is there a weather surcharge? How does distance affect pay? A solution built on silent guesses looks complete but is wrong. So the core of this skill is: **surface hidden decisions, get them answered, choose a clear solution, and trace every requirement through to how it will be verified.**

## Principles

- **Follow the user's language.** Write all questions, chat output, Solution.md, and the execution prompt in the language the user writes in.
- **Same goal for everyone.** Detect whether the user is a developer (mentions frameworks, files, APIs, code) or not. Adjust *how* you ask and explain — plain words and concrete examples for non-developers, technical terms for developers — but never lower the bar. Both get the same stages and the same Solution.md structure.
- **Questions are neutral; solutions are decisive.** Business rules and policies belong to the user, so ask about them without nudging (see Explore). Technical solutions are where your expertise counts, so state one clearly ("Use a Redis distributed lock for this"), explain why, and show the alternatives you rejected and their trade-offs.
- **Never present a guess as a fact.** Every piece of information has a source: `사용자 확인` / `user-confirmed`, `코드 확인` / `found-in-code`, or `가정` / `assumption`. Assumptions are always marked and listed so the user can overturn them.
- **Trace by ID.** Each requirement gets an ID (R1, R2, …). The plan, the solution guide, the execution prompt, and the verification criteria all refer back to those IDs. A requirement with no plan step or no verification criterion is a gap in the solution — fix it before handing over.

## The five stages

### 1. Explore — define and confirm the requirement

1. Restate the requirement in your own words.
2. If there is a codebase in the working directory, read it now (structure, stack, related modules, conventions, tests, infrastructure such as databases or caches already in use). What you find shapes both the constraints and the solution — a solution that ignores the existing stack is wrong even if it's technically sound. If there is no code, say so and move on.
3. Decompose the requirement into atomic requirements R1…Rn. Include implicit ones (e.g., "pay must never be negative").
4. Hunt for **hidden decisions** — rules the user must choose. Read `references/question-guide.md` for the categories to check and how to phrase questions.
5. Ask the user. Batch questions (max ~4–5 per round) and put the most consequential first. If more remain, ask them in the next round.
6. **Ask neutrally — no recommendations.** The moment an option is labeled "recommended", people tend to pick it without thinking about what they actually want, and the answer stops reflecting their real policy. So present options side by side with equal weight: for each, say concretely what happens if they choose it, and always leave room for their own answer. Don't order options by preference or add "recommended" / "(default)" labels. In Claude Code, the AskUserQuestion tool works well for this.
7. If the user explicitly says "you decide", ask what matters most to them (e.g., simplicity, fairness, cost) if it isn't already clear, then choose based on that goal, state your choice and reason, and record it as an assumption. If they skip a question without delegating, ask it again rather than filling it in.

**Gate 1:** Show the confirmed requirement list (R-IDs) and constraints, and ask the user to confirm. Do not proceed on an unconfirmed requirement list — a wrong list makes everything after it wrong.

**When the user asks how to solve a problem** ("이거 어떻게 해결하면 돼?", "how do I fix the double booking?"), lead with the answer. They came for a direction, and a reply that is only questions feels like no answer. In your first reply, state the solution direction you'd take and why ("Make the assignment a single conditional UPDATE in PostgreSQL so only the first request succeeds; no Redis lock needed"). If a pending answer would change it, show the branches ("first-come → X; server picks the rider → Y"). Mark it as a direction, not the final guide. Then ask only the questions that could change it, and write the full Solution.md after Gate 1.

### 2. Brainstorm — find the right solution

Consider 2–3 genuinely different approaches (not one approach and two strawmen). For each, state how it covers the R-IDs and its trade-offs (complexity, performance, consistency, operational cost, fit with the existing stack, time to build). Then **pick one and say so plainly**, giving the reasons in terms of the user's confirmed goals and the codebase you read. If only one approach is reasonable, say that instead of inventing alternatives.

### 3. Plan — break the solution into build steps

Write the steps someone would follow to build the chosen solution. Each step names the files or modules it touches (from Explore), the R-IDs it covers, and how it will be verified. Check coverage before moving on: every R-ID appears in at least one step and has a verification criterion.

**Gate 2:** Show the chosen approach and the plan in a few lines and ask the user to confirm before writing the full guide.

### 4. Solve — write the solution guide and the execution prompt

Write `Solution.md` using `references/solution-template.md`. Two parts matter most:

- **Solution guide** — explains *how to solve it*: the chosen approach and why, how it fits the existing code, where business rules should live (one clearly named place, since those values change most), the edge cases and how to handle each, and **short key snippets** for the parts that are easy to get wrong (e.g., the lock acquisition, the rounding rule, the boundary check). Snippets illustrate; they are not a full implementation, and they go in Solution.md, never into project files.
- **Execution prompt** — a self-contained prompt the user pastes into their own tool. Assume the reader has none of this conversation's context and doesn't know which tool will run it. See the template for its structure. Keep it tool-neutral: describe *what* to build and *how to check it*, not which tool commands to run.

### 5. Verify — make sure the solution is right before handing it over

Two checks, both required:

1. **Verification criteria for the user.** For each R-ID, write how to confirm it's met once built: a test scenario with concrete inputs and expected output (e.g., "Gangnam, rainy, 4.2 km → 3,500 + 500 + 600 = 4,600원"), including boundary cases.
2. **Self-check of the solution.** Before handing over, re-read the original requirement word by word and confirm:
   - every R-ID appears in the plan, the guide, the execution prompt, and the verification criteria;
   - every decision in the guide matches what the user confirmed (no silently changed values);
   - every file, class, or field you mention actually exists in the code you read (or is clearly marked as new);
   - the worked examples compute correctly.
   Record the result in Solution.md. If something can't be resolved, mark it ⚠️ and say what's needed.

## Handing over

In the chat, keep it short: the chosen solution in one or two sentences, where Solution.md is, and the execution prompt in a copyable code block so the user can paste it right away. For non-developers, add one line on how to use it (e.g., "Paste this into Claude Code, Cursor, or give it to your developer").

If a `Solution.md` already exists and isn't from this work, ask before overwriting.

## Scaling to the task

A one-line change doesn't need three brainstormed approaches or a question round. When the request is small and its literal meaning is clear (e.g., "change this error message to X"), skip the gates and Brainstorm, and hand over a short Solution.md: what to change and where (file and line found in the code), a short execution prompt, and how to verify. If you notice an adjacent issue (e.g., the same message is also shown in a case where the new wording is inaccurate), don't block on it — include it as an exception with the possible ways to handle it.

Use the full flow when the request hides decisions that change behavior the user cares about, such as policies, money, permissions, data, concurrency, or performance.
