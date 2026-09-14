# Verify/Check Mode for `setup-matt-pocock-skills`

This project will not add a dedicated verify/check mode (or a separate verify skill) for `setup-matt-pocock-skills`.

## Why this is out of scope

A second skill (or a `--verify` flag) for checking whether a repo's GitHub Projects board still has every column, field, view and label the skills expect would duplicate work the existing setup skill already does: its explore step reports exactly that before it writes anything.

The intended workflow is: **run `/setup-matt-pocock-skills` and tell it to verify your current setup.** The skill is prompt-driven, so the maintainer can scope it to a verification pass ("don't create anything, just report what's missing from the board") without needing a separate code path. Adding a flag or a sibling skill would split the surface area of a feature that's already expressible through the natural-language entry point.

Keeping board provisioning to a single skill also avoids the maintenance cost of two skills drifting from each other when the board shape evolves.

## Prior requests

- #106: Feature request: verify/check mode for setup-matt-pocock-skills
