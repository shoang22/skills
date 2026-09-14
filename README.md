# monopoly-skills

A fork of [mattpocock/skills](https://github.com/mattpocock/skills) that runs the spec, ticket and triage flow on a GitHub Projects board. It ships as the `monopoly-skills` Claude Code plugin. The original skills are Matt Pocock's; this fork changes how they track work.

## What's different from upstream

- **The issue tracker is a GitHub Projects board, set up for Scrum.** `/setup-matt-pocock-skills` creates it and commits nothing to your repo. The other skills find the board at run time.
- **A card's column is its triage state:** `Needs triage`, `Needs info`, `Ready for agent`, `Ready for human`, then `In progress`, `In review` and `Done`. The board also has `Priority`, `Size` and `Sprint` fields, and `Current sprint`, `Backlog` and `Triage` views.
- **Specs stay off the board** under a `spec` label. The tickets cut from a spec are its sub-issues, on the board, with native blocking links.
- **Removed:** GitLab, local-markdown and custom trackers, pull-request triage, `wayfinder`, and upstream's per-skill docs pages. Each `SKILL.md` is the documentation.

[ADR 0003](./.agents/adr/0003-github-projects-only-tracker.md) explains why.

## Install

In Claude Code:

```bash
claude plugin marketplace add shoang22/skills
claude plugin install monopoly-skills@monopoly-skills
```

Or, from inside a session:

```
/plugin marketplace add shoang22/skills
/plugin install monopoly-skills@monopoly-skills
```

Pull later changes with `/plugin marketplace update monopoly-skills`.

On Codex and other agents, use skills.sh instead:

```bash
npx skills@latest add shoang22/skills
```

Pick the skills you want, and which coding agents to install them on. **The installer lets you choose which skills to take: make sure `setup-matt-pocock-skills` and `github-board` are among them.**

Pick one route: installing both gives you every skill twice. Don't install upstream's `mattpocock-skills` plugin alongside this one either. It ships its own `triage`, `to-spec`, `to-tickets` and setup, wired to tracker files this fork removed.

## Set up a repo

You need the `gh` CLI, logged in with the `project` scope:

```bash
gh auth login
gh auth refresh -s project
```

Then run `/setup-matt-pocock-skills` in the repo. It finds or creates the board, adds whatever is missing, asks how long your sprints are, and lists the few settings GitHub only lets you change by hand. Running it again is safe.

## The flow

| Step | Who | What happens on the board |
| --- | --- | --- |
| Settle the idea | `/grill-me` | Nothing yet |
| Write it down | `/to-spec` | A `spec` issue, kept off the board |
| Slice it | `/to-tickets` | Tickets as sub-issues of the spec, in `Ready for agent`, each with a `Size` |
| Plan the sprint | You | Drag tickets from `Backlog` into the sprint |
| Build | You or an agent | Move the card to `In progress`, then `In review`; closing the issue moves it to `Done` |
| Review | `/code-review` | Reads the ticket and its parent spec |
| Incoming issues | `/triage` | Moves them through the triage columns, adding any that aren't on the board |

## Skills in the plugin

The board flow:

- **[setup-matt-pocock-skills](./skills/engineering/setup-matt-pocock-skills/SKILL.md)**: Provision the repo's GitHub Projects board. Run once per repo; safe to re-run.
- **[github-board](./skills/engineering/github-board/SKILL.md)**: Every board operation, shared by the other board skills. Hidden from the `/` menu.
- **[to-spec](./skills/engineering/to-spec/SKILL.md)**: Turn the current conversation into a spec issue.
- **[to-tickets](./skills/engineering/to-tickets/SKILL.md)**: Break a spec or plan into tracer-bullet tickets on the board.
- **[triage](./skills/engineering/triage/SKILL.md)**: Move incoming issues through the triage columns.
- **[code-review](./skills/engineering/code-review/SKILL.md)**: Review a diff against the repo's standards and the originating spec.
- **[domain-modeling](./skills/engineering/domain-modeling/SKILL.md)**: Build and sharpen the project's `CONTEXT.md` glossary and ADRs.

Personal:

- **[grilling](./skills/productivity/grilling/SKILL.md)**: Interview the user about a plan until every branch of the design tree is resolved.
- **[grill-me](./skills/productivity/grill-me/SKILL.md)**: Get grilled about a plan or design, with no repo needed.
- **[teach](./skills/productivity/teach/SKILL.md)**: Learn a concept over several sessions, using the current directory as a teaching workspace.
- **[writing-for-agents](./skills/productivity/writing-for-agents/SKILL.md)**: Write documents agents consume: skills, `AGENTS.md`/`CLAUDE.md`, pointed-at docs.
- **[unslop](./skills/pstack/unslop/SKILL.md)**: Cut AI tells from any writing.
- **[technical-writing](./skills/pstack/technical-writing/SKILL.md)**: A layered technical-writing standard for docs, READMEs, PR descriptions and commit messages.

Upstream's other skills are still in `skills/`, unchanged. Add a skill's path to `.claude-plugin/plugin.json` to ship it. The bucket READMEs list everything: [engineering](./skills/engineering/README.md), [productivity](./skills/productivity/README.md), [pstack](./skills/pstack/README.md).

## License

MIT, as upstream. See [LICENSE](./LICENSE).
