# GitHub Projects is the only issue tracker; no committed tracker config

Upstream, `/setup-matt-pocock-skills` wrote the repo's tracker choice into `docs/agents/issue-tracker.md`, alongside `triage-labels.md` and `domain.md`, and every engineering skill read those files at run time. That let one skill set serve GitHub Issues, GitLab, local markdown, or any tracker described in prose.

This fork drops that design.

## Decision

- The only issue tracker is a **GitHub Projects (v2) board** linked to the repo.
- Setup commits nothing to the repo. It provisions the board on GitHub, and every skill finds the board at run time from the repo's linked projects. The single optional file change is a README link, made only when the user asks.
- A card's `Status` column is its triage state. There are no triage labels and no label-mapping file.
- The board is shaped for Scrum: triage columns plus `In progress`, `In review` and `Done`; a `Sprint` iteration field; `Priority` and `Size` fields; `Current sprint`, `Backlog` and `Triage` views.
- Specs stay off the board, labelled `spec`. The tickets cut from a spec are its sub-issues, on the board, with native blocking links.
- The model-invoked `github-board` skill is the single source of truth for board shape and operations. `setup-matt-pocock-skills`, `to-spec`, `to-tickets` and `triage` all reach it through the Skill tool.
- Removed with the old design: the GitLab, local-markdown and "other" templates, the triage-label mapping, the "PRs as a request surface" flag with PR triage, and `wayfinder`, whose map lived in the removed tracker file.

## Why

The fork's owner plans work with a team on GitHub, in sprints. They wanted the tracker to be real GitHub infrastructure the whole team sees and drags cards around on, not a markdown file in each repo describing a tracker that lives elsewhere.

The per-repo file was also a second source of truth that could disagree with the tracker it described: it named triage labels that setup never created, so `gh` rejected them on first use. Once the tracker is fixed, the recipes for driving it are identical in every repo, so they belong in a skill rather than a per-repo file.

## Consequences

- The engineering skills need `gh`, logged in with the `project` scope. `github-board`'s preflight checks this and tells the user the fix.
- Teams on any other tracker can't use these skills; see [.out-of-scope/mainstream-issue-trackers-only.md](../../.out-of-scope/mainstream-issue-trackers-only.md).
- Some board setup has no API: a view's grouping and sort order, and enabling workflows such as auto-add. Setup prints those as manual steps. Issues filed in the web UI stay off the board until someone triages them, so `/triage` lists open issues that aren't on the board.
- Upstream changes to the removed files, or to the tracker sections of the edited skills, will conflict on merge. Keep this fork's side.
