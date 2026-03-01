Run a codex CLI code review on the current project.

## Steps

1. Check that the current working directory (or the directory specified by the user) is a git repository. If not, tell the user and stop.
2. Run `codex review --uncommitted` using Bash in the background (`run_in_background: true`), then wait for the result using TaskOutput with a timeout of 3600000 milliseconds (1 hour).
3. Parse the review output and present a clean summary to the user:
   - List each issue found with its priority, file path, line number, and description.
   - If no issues are found, tell the user the code looks clean.
4. Ask the user if they want you to fix any of the reported issues.

## Notes

- If the user provides a specific commit SHA or branch as an argument, use `codex review --commit <SHA>` or `codex review --base <branch>` instead of `--commit HEAD`.
- If the user specifies a commit SHA, use `codex review --commit <SHA>`.
- If the user specifies a base branch, use `codex review --base <branch>`.
- Default to `--uncommitted` to review all staged, unstaged, and untracked changes.
