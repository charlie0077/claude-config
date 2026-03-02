Run a full review + fix pipeline: /simplify + /rl1 in parallel.

## Instructions

### Progress tracking

Use TaskCreate/TaskUpdate throughout:

1. Create a parent task: "Full review pipeline: simplify + rl1" → mark `in_progress` immediately.
2. Create two child tasks:
   - "Phase 1a: simplify (code simplification)"
   - "Phase 1b: rl1 (Claude Code review loop)"

### Execution

1. **Launch simplify + rl1 in parallel**: In a **single message**:
   - Launch a background Agent (subagent_type: `general-purpose`, `run_in_background: true`) for simplify. Prompt: "Invoke the /simplify skill using the Skill tool: Skill(simplify). Do not do anything else." This runs the native system `/simplify` skill which launches its own 3 review agents (reuse, quality, efficiency) and fixes issues directly.
   - Call `Skill(rl1)` with $ARGUMENTS in the foreground. This takes over the conversation and runs the full review loop.
   - The simplify agent runs in the background in parallel while rl1 runs.

2. When rl1 completes, mark parent task `completed` and print the final summary.

### Final summary

After ALL phases complete, print a combined report:

```
## Full Review Pipeline Summary (simplify + rl1)

### Phase 1a: Simplify
- (summarize simplify results — files modified, what changed)

### Phase 1b: Claude Code
- (summarize rl1 results)

### Combined
- Total issues found: X
- Total issues fixed: Y
- Issues remaining: Z
- Files modified: list of files
```

If there are remaining issues, list each one with file/line so the user can address them.
