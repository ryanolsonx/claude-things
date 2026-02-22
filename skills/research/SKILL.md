---
name: research
description: |
  Deep codebase research skill. Use this whenever the user asks to research, investigate, understand, or explore how something works in their codebase — even if they just say "how does X work?" or "understand Y for me". The skill finds all relevant files and directories, reads them thoroughly to understand how they connect, and produces a persistent markdown research report saved to the project-local .claude/research/ directory.

  Trigger phrases: "/research <topic>", "research how X works", "investigate X", "deep dive into X", "understand X", "explore how X is implemented".
---

# Research

Given a topic, deeply explore the codebase to understand how it works and produce a thorough, structured research report saved as a markdown file.

The report serves as a durable artifact — a verification surface for any planning or implementation that follows. If the research is wrong, everything downstream will be too. That's why depth matters here.

## Workflow

### 1. Parse the topic

Extract the research subject from the user's message. Convert it to a URL-friendly slug for the filename (lowercase, hyphens for spaces, drop punctuation).

Examples:
- `/research task scheduling flow` → slug: `task-scheduling-flow`
- `/research how auth middleware works` → slug: `auth-middleware`
- `/research the payment webhook pipeline` → slug: `payment-webhook-pipeline`

### 2. Determine the output path

The report goes in the **project-local** `.claude/research/` directory — that means relative to the current working directory, not `~/.claude/`.

Filename format: `MM-DD-<slug>.md` using today's date.

Example: `.claude/research/02-22-task-scheduling-flow.md`

Create the directory if it doesn't exist:
```bash
mkdir -p .claude/research
```

### 3. Search for relevant files and directories

Cast a wide net first, then narrow. Use multiple search strategies in parallel:

- **Glob** for files matching the topic (by name, path segment, or extension pattern)
- **Grep** for key terms, function names, class names, types, and imports related to the topic
- **Directory listing** to understand structural organization around relevant areas

Don't stop at the first hit. Think about what else might be involved — config files, types, tests, entry points, utilities, middleware. Search for terms you discover during reading, not just the original topic.

### 4. Read deeply and trace the connections

This is the most important step. Read the relevant files thoroughly — not skimming, not just the exports. Understand:

- How execution flows (what calls what, in what order)
- Where state lives and how it moves
- What the key abstractions are and why they exist
- How components depend on each other
- What the non-obvious parts are

Follow imports and references. If file A imports from file B, read file B too. If there's a config file that shapes behavior, read it. If there are types that define the shape of data, understand them.

### 5. Write the report

Write the full research report to the output path. Use this structure:

```
# Research: <Topic>
*<date> · `.claude/research/<filename>`*

## Summary
2–4 sentences: what this is, the core mechanism, and why it's designed this way (if apparent).

## How It Works
Narrative walkthrough of the system — flow, phases, triggers, key decision points.
Write this so someone unfamiliar with the codebase can follow along.
Include code snippets for non-obvious or critical logic.

## Key Components
For each significant file or module involved:

### `path/to/file` (or ComponentName)
- **Role**: What it does in this system
- **Key exports / functions**: The most important ones and what they do
- **Notes**: Anything non-obvious, surprising, or important to know

## Data & Control Flow
How data enters, transforms, and exits. What triggers what.
A sequence or bullet trace works well here.

## Relevant Files & Directories
Complete reference list for future work. Annotate each entry.

| Path | Purpose |
|------|---------|
| `src/scheduler/` | Directory — contains all scheduling logic |
| `src/scheduler/index.ts` | Entry point, exports the public API |
| `src/scheduler/worker.ts` | Background worker that processes the queue |
| `src/types/jobs.ts` | Type definitions for job payloads |

## Connections & Dependencies
What this system touches outside itself — other modules, external services, shared state.

## Key Patterns & Gotchas
Important things to know before modifying this code. Surprises, implicit assumptions, things that look one way but work another.
```

### 6. Tell the user where to find it

After writing, output a short message:

```
Research complete. Report saved to `.claude/research/<filename>`.
```

## What makes good research

- **Breadth first, then depth**: Find everything relevant before reading any of it deeply. Missing a file is worse than spending extra time on one.
- **Follow the threads**: Don't stop at surface-level exports. Trace into dependencies. Read the types. Understand the config.
- **Explain the why, not just the what**: "This uses a queue because the underlying API is rate-limited" is more valuable than "this uses a queue."
- **Make the file list useful**: The files table is for future reference when someone is planning or implementing. Include everything that would be relevant to touch, not just what was most interesting to read.
- **Be honest about gaps**: If something is unclear or you couldn't find a definitive answer, say so in the report. An honest gap is more useful than a confident guess.
