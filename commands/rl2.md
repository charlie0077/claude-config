Run an iterative review + fix loop using only Codex CLI agent until the code is clean.

## Instructions

You are the driver of this loop. Do NOT delegate to shell scripts. Use your own tools (Bash, Read, Edit, Write) to fix issues directly.

### Progress tracking

Use TaskCreate/TaskUpdate throughout to give the user visibility:

1. Create a parent task: "Review-fix loop – Codex only (max 5 iterations)" → mark `in_progress` immediately.
2. For each iteration, create tasks for each phase and update their status as you go:
   - "Iteration N: codex review (3 agents)" → `in_progress` when started, `completed` when done
   - "Iteration N: fix issues" → `in_progress` when fixing, `completed` when done
3. Mark the parent task `completed` when the loop ends.

### Loop (max 5 iterations)

For each iteration:

1. **Run 3 identical Codex reviews in parallel (all in background)**:
   - Execute `codex review $ARGUMENTS` 3 times using Bash with `run_in_background: true`, all in a single message so they run concurrently. Default to `--uncommitted` if no arguments are provided. The `--uncommitted` flag scopes the review to the git diff only — do NOT pass flags that would review the entire project.
   - All 3 run the same command — the goal is independent redundancy so issues are less likely to be missed.
   - Wait for all to complete. Do NOT poll with `block: false` — simply wait for the background task notifications.
   - Once all are done, call `TaskOutput` with `block: true` and `timeout: 600000` for each task to retrieve the final output (all three calls in parallel).

2. **Extract issues only — do NOT dump raw output**:
   - Do NOT print raw reviewer output to the main conversation — it bloats context across iterations.
   - Instead, immediately parse each agent's output into structured issues (file, line, severity, description) and discard the raw text.
   - **Filter out noise**: Discard anything that is a style preference, naming opinion, missing comment/doc suggestion, hypothetical "consider doing X" suggestion, or anything that wouldn't break production. Only keep real bugs, actual security vulnerabilities, logic errors, race conditions, data loss risks, or clear correctness problems. It is completely OK to end up with zero issues after filtering.

3. **Collect and merge results**: Deduplicate issues — if multiple agents flag the same thing, count it once. Tag each issue with how many agents flagged it: `[1/3]`, `[2/3]`, or `[3/3]`. Issues flagged by more agents should be treated as higher confidence.

4. **Parse the output**: Distinguish between:
   - **Actual issues**: specific file/line references with bug descriptions, security warnings, or concrete fix suggestions.
   - **No issues**: phrases like "no issues", "looks clean", "no changes", "nothing to review", or a response that only describes the code without flagging problems.

5. **If issues are found**:
   - Print a **compact** merged issue table: `| # | Confidence | File | Line | Severity | Description |`
   - Track issues across iterations — if the same issue reappears after a fix attempt, flag it as unresolved rather than retrying the same fix.
   - Fix every reported issue yourself using Read and Edit tools. Do NOT over-engineer — make minimal, targeted fixes.
   - After fixing, print a **brief fix summary** (one line per fix: file, what changed). Do NOT repeat the full issue descriptions.
   - Do NOT commit — let the user decide when to commit.
   - Print issue trend so far, e.g. `Issues: 5 → 2 → ...`
   - Then **ALWAYS continue to the next iteration** (go back to step 1). Do NOT stop early — you must verify the fixes are clean by running the reviewers again. Do NOT skip the verification iteration because you "think" the issues are fixed or "unlikely to reappear."

6. **If no issues are found from all 3 agents**:
   - Tell the user the code is clean and stop the loop.

### Stopping conditions

ONLY these conditions can stop the loop — nothing else:

- All 3 agents report no issues in the same iteration → stop, tell the user the code is clean.
- Max 5 iterations reached → stop, summarize any remaining unresolved issues.
- Same issue persists after 3 fix attempts → stop, explain what you tried and why it's not working.
- Issues that cannot be fixed (e.g., architectural, out of scope) → stop, list them for the user to handle manually.

Do NOT stop for any other reason. "Issues were fixed and probably won't reappear" is NOT a valid reason to stop — you must verify by running the next iteration.

### Final summary

When the loop ends (for any reason), print a report:

```
## Review-Fix Loop Summary (Codex Only)
- Iterations: N
- Issues found:  X (3/3: A, 2/3: B, 1/3: C)
- Issues fixed:  Y
- Issues remaining: Z
- Trend: 5 → 2 → 0
- Files modified: list of files
```

If there are remaining issues, list each one with file/line so the user can address them.

### Important rules

- Always show the user the merged issue table and fix summary at each iteration.
- Do NOT print raw reviewer output to the main conversation — parse it into the issue table and discard. This prevents context from growing across iterations.
- Keep fixes minimal — only change what the reviewer flagged.
- Do NOT add unnecessary refactoring, comments, or type annotations.
- Do NOT commit or stage changes. The user controls git.
