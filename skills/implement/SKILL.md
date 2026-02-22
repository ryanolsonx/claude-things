---
name: implement
description: |
  Implements a feature from an approved plan. Use this when the user runs /implement, says "start implementing", "implement the plan", "let's build it", or any variation indicating they're ready to move from planning to code.

  The user may append extra instructions after /implement — read and honor them alongside the plan. Examples: "/implement use the existing error handler pattern", "/implement write tests for each phase".

  This skill reads the current plan from context, implements every task phase by phase, marks each task complete in the plan document as it goes, and runs typecheck continuously to catch issues early.
---

# Implement

Time to build. The plan has been approved and the todo list is ready. Work through it completely — phase by phase, task by task — until every checkbox is ticked. Don't stop early, don't ask for permission to continue between phases. The plan is the contract.

## Before Starting

### 1. Read any extra instructions

The user may have appended instructions after `/implement`. Extract and honor them throughout. They take precedence over defaults when there's a conflict.

Examples of what to watch for:
- Technology choices: "use bun", "use the existing logger"
- Scope additions: "write tests for each phase"
- Style constraints: "follow the existing error handling pattern"

### 2. Find and read the plan

Look in this conversation's context for the `.claude/plans/` path from the most recent `/deepplan`, `/deepplan-revised`, or `/create-todolist` run. Read the full plan — understand the phases, the file map, the approach, and every task before writing a line of code.

### 3. Find the typecheck command

Look in `package.json` for the typecheck script (commonly `tsc --noEmit`, `tsc`, or a script named `typecheck`, `type-check`, or `types`). You'll run this repeatedly throughout implementation.

---

## Implementation Loop

Work through each phase in order. Within each phase, work through each task in order.

### For each task:

1. **Implement it** — write clean, focused code that does exactly what the task describes. Refer back to the plan's approach and code snippets to stay aligned with the intended design.

2. **No noise** — don't add comments explaining what the code does, don't add JSDoc/TSDoc blocks unless they were in the plan, don't use `any` or `unknown` types. Types should be specific and correct.

3. **Run typecheck** — after completing a task (or a coherent group of related changes), run the typecheck command. Fix any errors before moving to the next task. A clean typecheck after each task means errors stay small and attributable.

4. **Mark complete** — edit the plan document to change `- [ ]` to `- [x]` for that task. This keeps the plan document live as a progress tracker.

### After each phase:

Mark the phase header as complete in the plan document (e.g., add `✓` to the heading, or note it's done). Run typecheck one more time as a phase-level checkpoint.

---

## Code Quality

A few things that matter throughout:

- **No `any` or `unknown`** — if you don't know the type, figure it out from the surrounding code, the plan's type definitions, or the existing codebase. Reach for `never` or a proper union before reaching for `any`.
- **No unnecessary comments** — the code should be readable on its own. Don't narrate what it does; make it clear enough that narration isn't needed.
- **Stay consistent** — match the patterns, naming conventions, and idioms of the existing codebase. If the plan references existing utilities or services, use them.
- **Keep typecheck green** — don't accumulate type errors and plan to fix them at the end. Fix them as they appear.

---

## When You're Done

Once every task in every phase is marked complete:

1. Run a final typecheck to confirm everything is clean.
2. Summarize what was built — a brief paragraph covering what was implemented and any notable decisions made during the work.
3. Note anything that came up during implementation that might be worth the user knowing (unexpected complexity, a pattern that was reused, a decision point where you made a judgment call).
