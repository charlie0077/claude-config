# Global Rules

## Auto-learn
When I correct your approach, immediately save the lesson to the project's `code-style.md` in memory. Don't wait.

## Package Manager
- Never use npm — always use bun (`bun add`, `bun install`, `bunx`, etc.)
- When adding a new dependency, always install the latest version (e.g. `bun add <pkg>@latest`)

## Code Style (all projects)
- No hardcoded magic numbers — use named constants
- Extract reusable logic into shared util functions
- Keep solutions concise; don't over-engineer
- Prefer auto-detection over manual config when possible
- Memory files should only contain rules and preferences, not implementation details
