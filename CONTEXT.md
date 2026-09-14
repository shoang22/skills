# Matt Pocock Skills

A collection of agent skills (slash commands and behaviors) loaded by Claude Code. Skills are organized into buckets. The engineering skills share one GitHub Projects **Board** per repo, provisioned by `/setup-matt-pocock-skills` and found at run time; nothing about it is committed to the repo.

## Language

**Issue tracker**:
The tool that hosts a repo's issues. In this fork it is always a **Board**. Skills like `to-tickets`, `to-spec`, and `triage` read from and write to it.
_Avoid_: backlog manager, backlog backend, issue host

**Board**:
The GitHub Projects (v2) project linked to a repo, holding its **Issues** as cards in **Columns**, with `Priority`, `Size` and `Sprint` fields for planning. The `github-board` skill is the single source of truth for operating it.
_Avoid_: project (ambiguous with the repo itself), kanban

**Column**:
One option of a **Board**'s `Status` field. A card's column is its only record of **Triage role**; there are no triage labels.

**Issue**:
A single tracked unit of work inside an **Issue tracker**: a bug, task, spec, or slice produced by `to-tickets`.
_Avoid_: ticket, except for the slices `to-tickets` produces

**Triage role**:
A canonical state of an **Issue** during triage: `Needs triage`, `Needs info`, `Ready for agent`, `Ready for human`, or `wontfix`. The first four are **Columns**; `wontfix` closes the issue as "not planned".

## Relationships

- An **Issue tracker** holds many **Issues**
- A **Board** has one **Column** per open **Triage role**, plus `In progress`, `In review` and `Done`
- An **Issue** on the **Board** sits in one **Column** at a time
- A spec **Issue** stays off the **Board**; the tickets `to-tickets` cuts from it are its sub-issues and sit on the **Board**

## Flagged ambiguities

- "backlog" was previously used to mean both the *tool* hosting issues and the *body of work* inside it. Resolved: the tool is the **Issue tracker**. The **Board**'s `Backlog` view (open issues with no sprint) is a view name, not a domain term.
- "backlog backend" / "backlog manager". Resolved: collapsed into **Issue tracker**.
