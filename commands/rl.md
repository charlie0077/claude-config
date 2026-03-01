Run a full review + fix pipeline: /simplify + /rl1 in parallel, then /rl2.

## Instructions

### Progress tracking

Use TaskCreate/TaskUpdate throughout:

1. Create a parent task: "Full review pipeline: (simplify + rl1) → rl2" → mark `in_progress` immediately.
2. Create three child tasks:
   - "Phase 1a: simplify (code simplification)"
   - "Phase 1b: rl1 (Claude Code review loop)"
   - "Phase 2: rl2 (Codex review loop)" — this task name MUST contain "rl2" so that `/rl1` can detect it and chain into `/rl2` automatically.

### Execution

1. **Launch simplify + rl1 in parallel**: In a **single message**:
   - Launch a background Agent (subagent_type: `general-purpose`, `run_in_background: true`) for simplify. Prompt: "Invoke the /simplify skill using the Skill tool: Skill(simplify). Do not do anything else." This runs the native system `/simplify` skill which launches its own 3 review agents (reuse, quality, efficiency) and fixes issues directly.
   - Call `Skill(rl1)` with $ARGUMENTS in the foreground. This takes over the conversation and runs the full review loop.
   - The simplify agent runs in the background in parallel while rl1 runs.

2. **Automatic rl2 handoff**: When rl1 finishes, it will check TaskList, find the pending "Phase 2: rl2" task, and automatically call `Skill(rl2)`. No manual intervention needed.

3. When rl2 completes, mark parent task `completed` and print the final summary.

### Final summary

After ALL phases complete, print a combined report:

```
## Full Review Pipeline Summary ((simplify + rl1) → rl2)

### Phase 1a: Simplify
- (summarize simplify results — files modified, what changed)

### Phase 1b: Claude Code
- (summarize rl1 results)

### Phase 2: Codex
- (summarize rl2 results)

### Combined
- Total issues found: X
- Total issues fixed: Y
- Issues remaining: Z
- Files modified: list of files
```

If there are remaining issues, list each one with file/line so the user can address them.
