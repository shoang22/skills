# Plan: GitHub Projects board as the only issue tracker

This fork (`shoang22/skills`) turns mattpocock/skills into the `monopoly-skills` Claude Code plugin. The engineering skills stop reading a per-repo `docs/agents/issue-tracker.md`. The tracker becomes a GitHub Projects v2 **board** that `/setup-matt-pocock-skills` provisions and every other skill finds at run time.

Every decision below was settled with the user in a grilling session on 2026-09-13. Treat them as fixed. Anything marked **verify** is a fact to check against GitHub before relying on it. If a verify check fails in a way that breaks a decision, stop and ask the user.

Work on branch `github-project-board`. Merge to `main` only after the test pass in [Verification](#verification). Delete this file in the merge commit. The new ADR keeps the reasoning.

## Vocabulary

- **Board**: the GitHub Projects v2 project linked to the repo. Owned by the repo's owner, titled after the repo.
- **Card**: a real repo issue on the board. Never a draft item. Drafts can't carry labels, comments, sub-issues or blocking links.
- **Column**: an option of the board's `Status` field. The column is the only record of triage state. There are no triage labels.
- **Spec**: the issue `/to-spec` files. Labelled `spec`, kept off the board.
- **Ticket**: an issue `/to-tickets` files. A sub-issue of its spec, on the board.

## Decisions

### Tracker

1. GitHub Projects v2 is the only tracker. Remove the GitLab, local-markdown and "Other" options.
2. Setup commits nothing to the target repo. No `docs/agents/`, no `## Agent skills` block in `CLAUDE.md` or `AGENTS.md`. The one exception is a README line, `Project board: <url>`, which setup adds only with the user's OK. It's for humans, and no skill reads it.
3. Skills find the board at run time from the repo's linked projects (GraphQL `repository.projectsV2`). One linked board: use it. Several: use the one whose title equals the repo name, else ask the user. None: tell the user to run `/setup-matt-pocock-skills`.
4. Pull-request triage is removed. The old "PRs as a request surface" flag goes with it.
5. `wayfinder` is removed from the fork.

### Board shape (Scrum)

6. `Status` columns, in this order: `Needs triage`, `Needs info`, `Ready for agent`, `Ready for human`, `In progress`, `In review`, `Done`.
7. `wontfix` means closing the issue as "not planned". GitHub's built-in rule then moves the card to `Done`.
8. Fields besides `Status`:
   - `Sprint`, an iteration field. Setup asks the length each run. Default 14 days, first sprint starting next Monday. Offer to align sprints with the user's course milestones.
   - `Priority`, single-select `P0`, `P1`, `P2`.
   - `Size`, single-select `XS`, `S`, `M`, `L`, `XL`.
9. Views:
   - `Current sprint`: board layout, filtered to the current sprint, columns from `Status`.
   - `Backlog`: table of open cards with no sprint, sorted by `Priority` then `Size`.
   - `Triage`: table of cards in `Needs triage` or `Needs info`.
   - No roadmap view.
10. Labels: `bug` and `enhancement` (category), `spec`.
11. Specs stay off the board. Tickets are sub-issues of their spec, so the board's built-in `Parent issue` field groups stories by spec and the spec shows `Sub-issue progress`.

### Who moves cards

12. `Done` comes from GitHub's built-in "Item closed" and "Pull request merged" rules.
13. `/to-tickets` files tickets into `Ready for agent`.
14. `/triage` moves cards between the triage columns.
15. `In progress` and `In review` are moved by hand, by the human or agent doing the work.
16. No skill touches `Sprint`. Humans assign sprints in sprint planning.
17. Issues filed in the GitHub web UI don't reach the board unless the user enables auto-add, which is UI-only. `/triage` covers the gap (decision 24).

### `/setup-matt-pocock-skills`

18. It runs in this order:
    1. Check that `gh` is installed and `gh auth status` lists the `project` scope. If `gh` is missing, stop and print the install step. If the scope is missing, stop and print `gh auth refresh -s project`.
    2. Confirm `git remote` points at GitHub. Read the repo's owner, visibility and linked projects.
    3. Present the findings. Get the user's confirmation before creating anything on GitHub.
    4. Reuse the linked board per decision 3, or create one. A new board is owned by the repo owner, titled after the repo, matches the repo's visibility, and is linked with `gh project link`.
    5. Ensure the columns, fields, views and labels from decisions 6 to 10 exist.
    6. Check that the "Item closed" workflow is enabled. Warn if it isn't; the API can't enable it.
    7. Offer the README line. Print the board URL and a note on enabling auto-add in the UI.
19. Setup is safe to re-run. It adds what's missing. It never deletes, renames or reorders existing columns, fields, views or labels, and it leaves an existing `Sprint` field alone.
20. Keep the skill name `setup-matt-pocock-skills` so upstream diffs stay small.

### New skill: `github-board`

21. Lives at `skills/engineering/github-board/`. It's the single source of truth for board operations and holds the `gh` checks from step 18.1. Every skill that touches the board calls it through the Skill tool. Other skills can invoke it; hide it from the slash menu if Claude Code's frontmatter supports that (**verify** the field name in the Claude Code skills docs).
22. Operations to cover: discover the board, resolve field and option IDs, create an issue on the board with `Status`/`Priority`/`Size`, read an issue, list cards by column or sprint, list open repo issues that aren't on the board, comment, set a field, close as completed or not planned, add a sub-issue, add a blocked-by link. Start from the `gh` recipes in the current `setup-matt-pocock-skills/issue-tracker-github.md` (issue CRUD, sub-issues, native dependencies via `repos/<owner>/<repo>/issues/<n>/dependencies/blocked_by` with the blocker's database id) and drop its PR and wayfinder sections.

### Changes to existing skills

23. `to-spec`: publish the spec through `github-board` as an issue labelled `spec`, not placed on the board. It no longer applies `ready-for-agent`.
24. `triage`:
    - The column replaces the state label. Remove the label-mapping paragraph and every "for a PR" delta.
    - "Show what needs attention" lists three buckets: open issues not on the board or with no `Status` (skip `spec` issues), `Needs triage`, and `Needs info` with reporter activity since the last triage notes. Triaging an off-board issue adds it to the board.
    - Moving a card to `Ready for agent` or `Ready for human` also sets `Priority` and `Size`. Skills ask the user for `Priority`; they never guess it.
    - Keep the calls to `grilling` and `domain-modeling`.
    - Update `triage/AGENT-BRIEF.md`, which still cites `gh issue list --label needs-triage` and closing "with a `wontfix` label".
25. `to-tickets`: remove the local-files branch and its template. The quiz step proposes a `Size` per ticket. Publish each approved ticket through `github-board` as a sub-issue of the spec (when the source is an issue), with native blocking links, into `Ready for agent`, with the approved `Size`. `Priority` stays blank unless the user gives one.
26. `code-review`: fetch issues referenced in commit messages with `gh issue view <n> --comments`. Replace the `docs/agents/issue-tracker.md` pointer with a plain "needs `gh`" check.
27. `domain-modeling`: absorb the consumer rules from `setup-matt-pocock-skills/domain.md`. Read `CONTEXT.md`, or `CONTEXT-MAP.md` and each relevant per-context `CONTEXT.md`, plus `docs/adr/` and `src/<context>/docs/adr/`. Proceed silently when they're absent. Use the glossary's terms. Flag ADR conflicts explicitly. The layout (single- or multi-context) is detected by whether `CONTEXT-MAP.md` exists.

### Files removed

28. From `skills/engineering/setup-matt-pocock-skills/`: `issue-tracker-github.md`, `issue-tracker-gitlab.md`, `issue-tracker-local.md`, `triage-labels.md`, `domain.md`.
29. `skills/engineering/wayfinder/`, the whole `docs/` tree, and `.agents/writing-docs.md`. The fork has no human-facing docs pages: each `SKILL.md` is the documentation (decided 2026-09-14, replacing the earlier plan to re-sync the pages).

### Docs

30. Update every remaining doc that describes the old tracker: `CONTEXT.md` (the Issue tracker and triage role definitions), `.agents/adr/0001-explicit-setup-pointer-only-for-hard-dependencies.md`, `.out-of-scope/mainstream-issue-trackers-only.md`, the root `README.md` (install and setup sections), `skills/engineering/README.md`, and `skills/engineering/ask-matt/SKILL.md` (the wayfinder route and "Custom issue trackers also work").
31. Add an ADR at the next free number in `.agents/adr/`: "GitHub Projects is the only tracker; no committed tracker config". Record why: the user wants the tracker to be real GitHub infrastructure for a Scrum team, not a markdown file in each repo.

### Plugin and personal skills

32. `.claude-plugin/plugin.json` and `marketplace.json`: name `monopoly-skills`, repository `shoang22/skills`. Check `scripts/sync-plugin-version.mjs` before bumping versions by hand.
33. The plugin ships exactly 13 skills:
    - Board: `setup-matt-pocock-skills`, `github-board`, `to-spec`, `to-tickets`, `triage`, `code-review`, `domain-modeling`.
    - Personal: `grilling`, `grill-me`, `writing-for-agents`, `teach` (from `skills/productivity/`), `unslop`, `technical-writing` (from `skills/pstack/`).
34. `teach` ships with the user's "Writing" section, which invokes `unslop`. Merge branch `teach-unslop` into `github-project-board`.
35. To add another upstream skill later, add its path to `plugin.json`. Never install upstream `mattpocock-skills` alongside this plugin.
36. Add `*:Zone.Identifier` to `.gitignore`. Windows attaches these files to downloads.
37. Never run `scripts/link-skills.sh`. It `rm -rf`s any real directory in `~/.claude/skills` that shares a name with a repo skill.

## Status

Done on the old machine (Ubuntu 20.04) before this file was pushed:

- [x] Branch `teach-unslop` holds the `teach` edits (commit `eb3da77`).
- [x] `skills/pstack/unslop/` and `skills/pstack/technical-writing/` committed on this branch.
- [x] This plan committed on this branch.

To do on the new machine:

- [x] Install `gh` from GitHub's apt repository, never the snap. Run `gh auth login`, then `gh auth refresh -s project`. (Done on 20.04 after all; implementation happened there.)
- [x] Merge `teach-unslop` into this branch.
- [x] Run the **verify** checks in [Facts](#facts) that the implementation depends on. The ones still marked **verify** need the test board.
- [x] Implement decisions 1 to 37. The root `README.md` became a short fork README rather than an edit of upstream's (the user's call, 2026-09-14).
- [x] Throwaway repo created: private `shoang22/board-test`, cloned at `~/workspace/toys/board-test`. From there, `claude --plugin-dir ~/workspace/toys/skills` loads the plugin without installing it.
- [ ] Pass [Verification](#verification).
- [ ] Merge to `main`, deleting this file. Install with `/plugin marketplace add shoang22/skills`, then install `monopoly-skills`.

## Facts

Researched on 2026-09-13. Sources are the GitHub docs pages named in each item.

- **Views.** The GraphQL API has `createProjectV2View` and `updateProjectV2View` (GraphQL reference, Projects). Layouts are `BOARD_LAYOUT`, `TABLE_LAYOUT`, `ROADMAP_LAYOUT`. Community threads saying views can't be created through the API are out of date. Verified by schema introspection on 2026-09-14: create takes `name`, `layout` and `configuration`; update also takes `filter`; `ProjectV2ViewConfigurationInput` only holds `visibleFieldIds`. Grouping, sort and a board's column field can't be set, so setup prints them as manual steps. **Verify** on the test board that a new board-layout view uses `Status` for its columns by default.
- **Sprint field.** `createProjectV2Field` accepts `dataType: ITERATION` with `iterationConfiguration { duration (days), startDate, iterations }`, where each iteration is `{ title, startDate, duration }`. `gh project field-create` only supports `TEXT`, `SINGLE_SELECT`, `DATE`, `NUMBER`, so the sprint field needs `gh api graphql`. **Verify** on the test board what an empty `iterations` list produces.
- **Editing columns.** Verified by schema introspection: `updateProjectV2Field`'s `singleSelectOptions` overwrite the existing options. An option sent with its `id` keeps its identity, so cards keep their value; `name`, `color` and `description` are required on every option. Setup sends every existing option back with its `id`.
- **Workflows.** No mutation creates or enables a workflow; only `deleteProjectV2Workflow` exists, and workflows can be read. "Item closed → Done" and "Pull request merged → Done" are on by default on a new board. Auto-add and auto-archive are UI-only.
- **Copying a board.** `copyProjectV2` and `gh project copy` copy views, custom fields and workflows (except auto-add), but not items or repo links. Not used. It's the fallback if view configuration through the API proves too limited: keep one template board and copy it.
- **Hierarchy fields.** Projects have built-in `Parent issue` and `Sub-issue progress` fields. A view can group or filter by `Parent issue`.
- **Issue commands.** Verified on `gh` 2.100.0: `gh issue create` takes `--project`, `--parent` and `--blocked-by`; `gh issue edit` takes `--add-project`, `--add-sub-issue` and `--add-blocked-by`; `gh issue close --reason "not planned"` exists; `gh project item-edit --url <issue> --field <name> --value <value>` sets a field by name, with no IDs.
- **Filters.** Verified in "Filtering projects": `iteration:@current`, `no:<field>`, `status:"Needs triage","Needs info"`, `-label:x`, `is:open`. **Verify** on the test board that a field named `Sprint` filters as `sprint:@current` and `no:sprint`.
- **Hidden skill.** Verified in the Claude Code skills docs: `user-invocable: false` hides a skill from the `/` menu and keeps it model-invocable.

## Verification

Run this in a **fresh** Claude Code session, not the one that wrote the skills, so each skill is judged on its text alone rather than on what its author already knows. Load the plugin from the fork first. Create a throwaway GitHub repo under the user's account; ask the user before creating it. Done means every item below holds:

- [ ] Setup on a fresh repo creates and links the board with all seven columns in order, the three fields, three views and three labels. The only manual steps it prints are auto-add and anything the view-configuration **verify** check ruled out.
- [ ] A second setup run changes nothing and reports that everything exists.
- [ ] `/to-spec` files a `spec` issue that is not on the board.
- [ ] `/to-tickets` on that spec files sub-issues with blocking links, each on the board in `Ready for agent` with its approved `Size`.
- [ ] An issue opened in the web UI shows up in `/triage`'s first bucket. Triaging it to `Ready for human` adds it to the board and sets `Priority` and `Size`.
- [ ] `/triage` closing an issue as wontfix leaves it closed as "not planned", with the card in `Done`.
- [ ] `/code-review` on a commit that references `#<n>` reads that issue.
- [ ] Grep for `issue-tracker`, `issue tracker`, `triage-labels`, `triage label`, `docs/agents`, `wayfinder`, `glab` and `.scratch/` across the repo. Every hit is either updated or intentionally kept.
- [ ] The plugin installed from the fork lists exactly the 13 skills in decision 33.

Then run setup on the user's XR-ProVA repo, once it has a GitHub remote.
