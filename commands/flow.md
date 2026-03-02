# Phase-by-Phase Build

You are executing a phased build pipeline. This command implements phases from PLAN.md one at a time, using subagents to keep context clean.

## Arguments

`$ARGUMENTS` contains an optional phase range: `<start> <end>` (e.g., `3 5` runs phases 3 through 5).

Parse the arguments:
- If two numbers are given, use them as start and end phase (inclusive).
- If one number is given, run only that phase.
- If empty, run all phases.

## Setup

1. Read `PLAN.md` to discover the total number of phases and their names. Validate the requested range and determine the default range when no arguments are given.
2. Read `CLAUDE.md` (if it exists) to collect project conventions for forwarding to subagents.
3. Record the current commit hash (`git rev-parse HEAD`) for the end summary.

## Progress Tracking

Before starting, create a task for each phase in the range using TaskCreate (e.g., "Implement Phase 1 — Foundation"). As you work through phases, update each task: set to `in_progress` when starting, `completed` when committed.

---

## Per-Phase Pipeline

For **each phase** in the requested range, execute these 3 steps in order:

### Step 1: Implement (Subagent)

Mark the phase task as `in_progress`. Launch a **general-purpose Agent** with this prompt:

> You are implementing Phase N (<phase name>) of a build pipeline.
>
> Project conventions: <include relevant rules from CLAUDE.md>
>
> Read `PLAN.md` and implement everything described in Phase N. Create/edit all files and install all dependencies. Ensure the code is complete and builds successfully.

Wait for the subagent to finish before proceeding.

### Step 2: Review + Fix (Subagent)

Launch a **general-purpose Agent** with this prompt:

> You are reviewing a project after Phase N (<phase name>) was implemented. Run the `/rl` skill to review and fix code quality, bugs, and style issues. Let it complete fully.

Wait for the subagent to finish before proceeding.

### Step 3: Git Commit

Stage all changes and commit:
```
git add -A
git commit -m "feat: implement Phase N — <phase name>"
```
Use a descriptive message. If there's nothing to commit, skip.

Mark the phase task as `completed`, then immediately proceed to the next phase.

---

## End Summary

After all phases are complete, print a summary:

- List each phase that was built and its status (committed / skipped)
- Run `git diff --stat <initial-commit-hash>..HEAD` to show total files changed
- Run the project's test command and include the final results
