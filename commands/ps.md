# Server Status

Check all tmux sessions on the Hetzner server and report what's happening.

## Arguments

`$ARGUMENTS` is optional. If provided, it's the server name (default: `agent1`).

## Execution

1. Get the server IP using `hcloud`:
   ```bash
   hcloud server list -o columns=name,ipv4 -o noheader | grep "<server_name>" | awk '{print $2}'
   ```

2. SSH to the server and capture all tmux sessions:
   ```bash
   ssh -i ~/.ssh/hetzner agent@<IP> 'tmux list-sessions -F "#{session_name}" 2>&1'
   ```

3. For each session, capture the last 15 lines of output:
   ```bash
   ssh -i ~/.ssh/hetzner agent@<IP> 'for s in <session_list>; do echo "=== $s ==="; tmux capture-pane -t "$s" -p -S -15 2>&1; echo; done'
   ```

4. Analyze each session and report a concise summary table:

   | Session | Type | Status | Details |
   |---------|------|--------|---------|
   | pp | pipeline | idle | 7/7 done |
   | pi | issues | running | #8 (3m) |
   | ... | ... | ... | ... |

   **Type**: `pipeline` (pp-*), `issues` (pi-*), `flow` (pl-*), or `worker` (pi-*-N)

   **Status**:
   - `idle` — polling, nothing to do
   - `running` — actively processing
   - `stuck` — waiting on prompt, auth error, or no progress
   - `dead` — session exists but process exited (shell prompt visible)

   For `stuck` sessions, explain what they're stuck on (trust prompt, auth, etc.)

5. Print a one-line summary at the end: total sessions, how many running, how many idle, how many stuck/dead.
