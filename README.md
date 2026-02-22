# claude-things

Just Claude Code things: skills, agents, hooks, etc that I've come up with or copied from the internet.

## Skills

Skills live in `skills/` and are symlinked to `~/.claude/skills` so Claude Code picks them up automatically.

These skills implement a **plan-first development workflow** inspired by [Boris Tane's approach](https://boristane.com/blog/how-i-use-claude-code/): research → plan → annotate → implement → audit. Nothing gets built until a plan is reviewed and approved.

---

### `/research <topic>`

Deep codebase research. Finds all relevant files and directories, reads them thoroughly to understand how they connect, and saves a structured report.

```
/research task scheduling flow
/research how auth middleware works
/research the payment webhook pipeline
```

**Output:** `.claude/research/MM-DD-<topic>.md`

The report includes: summary, how it works, key components, data/control flow, a full file reference table, and gotchas. The file table is especially useful — hand it back to `/deepplan` so the plan knows exactly which files to touch.

---

### `/deepplan <name> <description> to achieve <goal>`

Writes a detailed implementation plan before any code is written. Draws from any matching research file in `.claude/research/` automatically.

```
/deepplan user-auth JWT-based authentication to achieve secure stateless login
/deepplan email-queue send transactional emails on key events to achieve user engagement
```

**Output:** `.claude/plans/MM-DD-<name>.md`

The plan includes: overview, approach + trade-offs, step-by-step implementation with code snippets and file paths, a file map, open questions, and out-of-scope items.

After it's written: **open it in your editor, add inline notes/corrections**, then run `/deepplan-revised`.

---

### `/deepplan-revised`

You've added inline notes to the plan. This reads them, addresses every one, updates the document in place, and removes the annotations. Repeat as many times as needed.

Notes can be in any format — HTML comments, blockquotes, bold markers, plain text. Claude finds them all.

**Output:** Same plan file, revised and cleaned up.

After each revision: review again, add more notes if needed, run `/deepplan-revised` again — or move on when satisfied.

---

### `/create-todolist`

Converts the approved plan into a phased `- [ ]` checklist appended to the plan document. Run this when you're happy with the plan and ready to hand it off for implementation.

```
/create-todolist
```

**Output:** `## Todo List` section added to the current plan, organized by phase with granular tasks.

---

### `/implement [extra instructions]`

Implements everything in the plan. Works phase by phase, task by task, marks each `- [x]` as it goes, and runs typecheck continuously. Doesn't stop until all tasks are done.

```
/implement
/implement use the existing error handler pattern
/implement write tests for each phase
```

Rules baked in: no `any`/`unknown` types, no unnecessary comments or JSDoc, typecheck must stay green throughout.

---

### `/followed-plan`

Post-implementation audit. Runs as a subagent, reads the plan, gets the git diff, and reports on whether implementation matched the plan.

```
/followed-plan
```

**Output:** A report with:
- **Verdict** — Followed / Mostly Followed / Deviated
- **Plan coverage** — which tasks were implemented (✓/✗/~)
- **Unexpected changes** — diff changes not in the plan
- **Deviations** — where approach differed from the plan
- **Missing items** — plan tasks with no evidence in the diff

---

## Full Workflow

```
/research <topic>          → understand the codebase first
/deepplan <name> ...       → write the plan
  (annotate in editor)
/deepplan-revised          → address your notes  ← repeat until satisfied
/create-todolist           → convert to task list
/implement                 → build it all
/followed-plan             → verify it matched the plan
```
