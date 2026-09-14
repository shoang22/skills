# Explicit `/setup-matt-pocock-skills` pointer only for hard dependencies

Engineering skills depend on a GitHub Projects board provisioned by `/setup-matt-pocock-skills` (see [ADR 0003](./0003-github-projects-only-tracker.md)). Some skills cannot meaningfully function without it: they publish issues onto the board or move cards between its columns. Others only use the repo's domain docs to sharpen output (vocabulary, ADR awareness) and degrade gracefully without them.

We split these into **hard-dependency** and **soft-dependency** skills:

- **Hard dependency** (`to-tickets`, `to-spec`, `triage`): call the Skill tool with "github-board" and run its preflight, with an explicit one-liner: _"If it reports no board, tell the user to run `/setup-matt-pocock-skills`."_ Without the board, output is wrong, not just fuzzy.
- **Soft dependency** (`diagnose`, `tdd`, `improve-codebase-architecture`): reference "the project's domain glossary" and "ADRs in the area you're touching" in vague prose only. If the docs aren't there, the skill still works; output is just less sharp.

The split keeps soft-dependency skills token-light and avoids cargo-culting the setup pointer into places where it isn't load-bearing.
