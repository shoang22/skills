# Issue trackers other than GitHub Projects

This fork supports exactly one issue tracker: a GitHub Projects board linked to the repo. Requests to support GitLab, Linear, Jira, local markdown files, or any other tracker are out of scope, however mainstream the tool.

## Why this is out of scope

The engineering skills lean on things only GitHub provides: Projects columns and iteration fields, native sub-issues and blocking links, and the `gh` CLI that drives all of it. Fixing the tracker is what let the per-repo config files go away and the board operations move into one skill, `github-board`. A second tracker would bring back both. [ADR 0003](../.agents/adr/0003-github-projects-only-tracker.md) records the decision.

Upstream [mattpocock/skills](https://github.com/mattpocock/skills) supports GitHub, GitLab, local markdown, and trackers described in prose. Use it if you need one of those.

## Prior requests

- Upstream #99: "Add dex as an issue tracker backend", rejected there under upstream's mainstream-only rule before this fork narrowed support to GitHub Projects.
