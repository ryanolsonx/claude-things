---
name: followed-plan
description: |
  Post-implementation audit that compares what was actually built (via git diff) against the plan that guided the implementation. Use this when the user runs /followed-plan, says "did we follow the plan?", "check the implementation against the plan", or "audit what was built".

  Runs as a subagent — spawn it after /implement completes. It reads the plan, gets the git diff, and reports on plan adherence: what was followed, what was missed, and what deviated unexpectedly.
---

# Followed Plan

An independent audit of whether the implementation matches the plan. The goal is to catch gaps, unexpected scope creep, or approach deviations before they become problems — while the context is still fresh and corrections are cheap.

This runs as a subagent so it brings fresh eyes without the implementation's context coloring the evaluation.

## Workflow

### 1. Find the plan

Look in the conversation context for the `.claude/plans/` path from the most recent `/implement` or `/deepplan` run. Read the full plan — the implementation steps, file map, approach, and todo list (if present).

### 2. Get the git diff

Figure out what actually changed. Check the working tree state first:

```bash
git status
```

Then get the full diff of implementation changes:

- If there are uncommitted changes: `git diff HEAD` captures both staged and unstaged
- If the working tree is clean (changes were committed): use `git diff HEAD~1` or find the relevant commit range from `git log --oneline -10`
- If uncertain, run both and use whichever has content

Read the diff carefully. You're looking at: which files were touched, what was added, what was removed, and what the changes actually do.

### 3. Compare plan vs. reality

Go through the plan systematically and assess each element:

**For each implementation step / todo item in the plan:**
- Was it implemented? Look for evidence in the diff (new files, modified functions, added logic)
- Was the approach consistent with what the plan described?
- Were the right files touched (compare against the plan's File Map)?

**Look at the diff for things outside the plan:**
- Files changed that aren't in the plan's File Map
- Logic added that wasn't described in any implementation step
- Removals or refactors that weren't mentioned

### 4. Write the report

Produce a clear, honest assessment. Don't soften real gaps or invent problems that aren't there.

```
## Plan Adherence Report

**Verdict**: [Followed / Mostly Followed / Deviated]

### Plan Coverage
What the plan called for and whether it was implemented:

✓ Phase 1: {name} — implemented as described
✓ {specific task} — found in diff at src/foo/bar.ts
✗ {specific task} — no evidence in diff
~ {specific task} — implemented but approach differed (see Deviations)

### Unexpected Changes
Changes in the diff that weren't in the plan:
- `src/some/file.ts` — modified but not in the File Map; {what changed}
- (or: "None — diff matches plan scope exactly")

### Deviations
Where implementation diverged from the plan's described approach:
- {plan said X, diff shows Y, here's why that matters or doesn't}
- (or: "None significant")

### Missing Items
Plan tasks with no evidence of implementation:
- {task name} — expected in {file}, not found
- (or: "None — all tasks accounted for")

### Summary
1–3 sentences on overall fidelity. Flag anything that warrants follow-up.
```

Be specific. "Phase 2 was implemented" is less useful than "the JWT middleware was added to `src/middleware/auth.ts` as planned, but the token expiry logic described in Step 2.3 isn't present in the diff."

### 5. Return results

Report back to the main conversation with the full audit. If there are gaps or deviations, be concrete about what's missing so the user knows exactly what to address.
