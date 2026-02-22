---
name: create-todolist
description: |
  Adds a detailed, phased todo list to the current feature plan. Use this when the user runs /create-todolist, or says "create a todo list", "add tasks to the plan", "break this into tasks", or anything indicating they're ready to turn an approved plan into actionable work items.

  This is the step after /deepplan and /deepplan-revised — once the plan is approved, this converts it into a granular checklist appended directly to the plan document. Do not implement anything.
---

# Create Todo List

The plan has been reviewed and approved. Now convert it into a detailed, actionable todo list appended to the plan document. This is a planning step — nothing gets implemented.

## Workflow

### 1. Find the plan

Look in the conversation context for the `.claude/plans/` path from the most recent `/deepplan` or `/deepplan-revised` run. That's the file to update. If it's not in context, check `.claude/plans/` for the most recently modified `.md` file.

### 2. Read the plan

Read it fully. You need to understand every implementation step, its dependencies, and the full scope of work before writing tasks. The todo list should reflect the plan's structure — not a generic breakdown invented from scratch.

### 3. Append the todo list

Add a `## Todo List` section to the end of the plan document. Organize it into phases that mirror the plan's implementation steps. Within each phase, break the work into granular, concrete tasks — small enough that completing one feels like clear progress.

Use markdown checkboxes:

```markdown
## Todo List

### Phase 1: {name from Implementation Step 1}
- [ ] {specific, concrete task}
- [ ] {specific, concrete task}
- [ ] {specific, concrete task}

### Phase 2: {name from Implementation Step 2}
- [ ] {specific, concrete task}
- [ ] {specific, concrete task}

### Phase 3: {name from Implementation Step 3}
- [ ] {specific, concrete task}
- [ ] {specific, concrete task}
```

Good tasks are specific enough that there's no ambiguity about when they're done. "Add JWT middleware" is a task. "Work on auth" is not.

If the plan has cross-cutting concerns (types, tests, config), add them as their own phase rather than scattering them across others.

### 4. Tell the user

After writing:

```
Todo list added to `.claude/plans/{filename}`.
```

Then list the phases so they can see the structure at a glance — no need to reproduce every task, just the phase names and task counts.

Finish with:

```
When you're ready to start implementing, just say the word.
```
