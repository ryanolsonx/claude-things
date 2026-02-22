---
name: deepplan
description: |
  Feature planning skill that follows a structured plan-first workflow. Use this for two commands:

  /deepplan {feature name} {feature description} to achieve {goal} — writes a thorough implementation plan (file paths, code snippets, trade-offs) to the project-local .claude/plans/MM-DD-{feature-name}.md. The plan is the control surface. Nothing gets built until the plan is approved.

  /deepplan-revised — the user has reviewed the plan from the most recent /deepplan and added inline notes/corrections. Read that plan, address every note, update the document in place. Do not implement anything.

  Trigger on: "/deepplan", "/deepplan-revised", "plan this feature", "write a plan for", "I updated the plan", "I added notes to the plan".
---

# DeepPlan

Planning before coding. The plan is the control surface — a durable artifact that captures intent, approach, and trade-offs before any code is written. It's also the annotation surface: the user reviews it, adds inline corrections and domain knowledge, and sends it back. This cycle repeats until the plan is solid enough to implement confidently.

Two commands, two phases of the same loop:

- **`/deepplan`** — write the initial plan
- **`/deepplan-revised`** — address the user's inline notes and update the plan

---

## /deepplan — Writing the Initial Plan

### 1. Parse the arguments

The command format is: `/deepplan {feature name} {feature description} to achieve {goal}`

Extract:
- **Feature name** — short identifier, used for the filename slug
- **Feature description** — what the feature is / does
- **Goal** — what it achieves / why it exists

Slugify the feature name for the filename: lowercase, hyphens for spaces, drop punctuation.

Examples:
- `/deepplan user-auth JWT-based authentication to achieve secure stateless login` → slug: `user-auth`
- `/deepplan email-queue send transactional emails on key events to achieve user engagement` → slug: `email-queue`

### 2. Check for existing research

Look in `.claude/research/` for any files related to the feature. If found, read them before writing the plan — they contain the relevant file map, architecture understanding, and key patterns that make the plan accurate. Good research makes for a plan you can actually execute.

### 3. Determine the output path

Save to the **project-local** `.claude/plans/` directory (relative to cwd, not `~/.claude/`).

Filename format: `MM-DD-{slug}.md` using today's date.

```bash
mkdir -p .claude/plans
```

### 4. Write the plan

A good plan is concrete enough that two engineers would implement it the same way. Vague plans defeat the whole purpose. Use actual file paths, function signatures, type shapes, and representative code snippets wherever they clarify intent.

Use this structure:

```
# Plan: {Feature Name}
*{date} · Goal: {goal}*

## Overview
2–4 sentences: what this feature does, how it fits into the existing system,
and the core implementation approach.

## Approach
The chosen strategy and why. If meaningful alternatives were considered,
briefly note them and why they were set aside. This is where trade-offs live.

## Implementation Steps

### 1. {Step name}
**Files:** `src/foo/bar.ts`, `src/types/baz.ts`

What this step does, with enough detail that there's no ambiguity.

```ts
// Representative shape of what gets written — not final code
interface Baz {
  id: string
  createdAt: Date
}
```

### 2. {Step name}
...

## File Map
Complete list of files to create or modify:

| File | Action | Purpose |
|------|--------|---------|
| `src/auth/jwt.ts` | Create | JWT encode/decode utilities |
| `src/middleware/auth.ts` | Modify | Add JWT verification middleware |

## Open Questions
Decisions that need input, or things that were unclear during planning.

## Out of Scope
What this plan explicitly does not cover, to prevent scope creep.
```

### 5. Tell the user

After writing, output exactly:

```
Plan saved to `.claude/plans/{filename}`.

Open it in your editor, add any inline notes or corrections, then run /deepplan-revised when ready.
```

Follow with a 2–3 sentence summary of the approach so they know what to look for when they open it.

---

## /deepplan-revised — Addressing Inline Notes

The user has read the plan, added inline annotations, and is sending it back. Your job: address every note, update the plan in place, and produce a clean revised document. Do not write any code or touch any file outside `.claude/plans/`.

### 1. Find the plan

The plan path is in this conversation — look for the `.claude/plans/` path that was output when `/deepplan` ran. Use that exact file. If for some reason it's not in context, look for the most recently modified `.md` in `.claude/plans/`.

### 2. Read the entire plan first

Read it fully before making any edits. Get the whole picture before touching anything — one note's fix can affect another section, and catching that before you start is much cleaner than patching afterward.

### 3. Find and address every note

Notes are anything the user added that looks like an annotation rather than original plan content — corrections, questions, constraints, domain knowledge, redirections. They might appear in any form:

- HTML comments: `<!-- NOTE: use postgres not sqlite -->`
- Blockquotes: `> NOTE: this approach won't work because...`
- Brackets: `[NOTE: prefer the existing UserService here]`
- Bold markers: `**NOTE:** we already have this in utils/auth.ts`
- Plain inline text that's clearly a personal addition

For each note:
1. Understand what's being corrected or added
2. Update the relevant section of the plan to reflect it
3. Remove the note itself — the goal is a clean revised plan, not a plan with annotations still in it

If a note raises a question you can't resolve from the codebase or context, add it to "Open Questions" rather than guessing.

### 4. Update the document

Rewrite the plan file with all notes incorporated and all annotation markers removed. The result should read like it was written this way from the start — the user's knowledge absorbed into the plan, no trace of the back-and-forth.

### 5. Tell the user

After updating, output:

```
I added a few notes to the document, address all the notes and update the document accordingly. don't implement yet.

Changes made:
- [one bullet per note: what it said → what changed]
```

Then invite the next step:

```
Open it again if you want another pass. When you're satisfied, say the word
and we'll turn it into a task list and start implementing.
```

The change summary is important — it confirms the notes landed correctly and lets the user verify without re-reading the whole plan.
