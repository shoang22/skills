---
name: github-board
description: "Operations on this repo's GitHub Projects board, the issue tracker the engineering skills share: find the board, file issues onto it, move cards between columns, set Priority and Size, list cards by column, link sub-issues and blocking edges. Use whenever a skill says to publish to, read from, or update the board."
user-invocable: false
---

# GitHub Board

The **board** is the GitHub Projects (v2) project linked to this repo, and it is the repo's issue tracker. Every **card** is a real repo issue, never a draft item: drafts can't carry labels, comments, sub-issues or blocking links. A card's **column** (its `Status` value) is its triage state. Nothing about the board is written into the repo, so find it fresh every session.

Every operation uses the `gh` CLI, run from inside a clone of the repo so `gh` infers the repo from `git remote`.

## Preflight

Run once per session, before the first operation. Stop at the first failure and tell the user the fix:

1. `gh --version` fails: `gh` is missing. Point the user at https://github.com/cli/cli#installation (on Debian or Ubuntu, GitHub's own apt repository; the snap package is unsupported).
2. `gh auth status` shows no login, or its token scopes lack `project`: tell the user to run `gh auth login`, then `gh auth refresh -s project`.
3. [Find the board](#find-the-board). No linked board: tell the user to run `/setup-matt-pocock-skills`.

Preflight is done when you hold the board's **owner**, **number**, **title** and **URL**.

## Find the board

```bash
gh repo view --json nameWithOwner,projectsV2
```

Read the linked boards (number, title, URL) from `projectsV2`, ignoring closed ones. The board's owner is the repo owner.

- One linked board: use it.
- Several: use the one whose title equals the repo name. If none matches, ask the user which.
- None: preflight fails at step 3.

## Board shape

What `/setup-matt-pocock-skills` provisions, and what every operation below assumes.

Columns, in order:

| Column | Colour | Means |
| --- | --- | --- |
| `Needs triage` | `GRAY` | Nobody has evaluated it yet |
| `Needs info` | `YELLOW` | Waiting on the reporter for more information |
| `Ready for agent` | `BLUE` | Fully specified, ready for an AFK agent |
| `Ready for human` | `PINK` | Specified, but needs a person |
| `In progress` | `ORANGE` | Someone is working on it |
| `In review` | `PURPLE` | A pull request is open for it |
| `Done` | `GREEN` | Closed. GitHub's built-in "Item closed" rule moves closed issues here |

Fields besides `Status`:

- `Priority`: single-select `P0`, `P1`, `P2`.
- `Size`: single-select `XS`, `S`, `M`, `L`, `XL`, a rough effort estimate for sprint planning.
- `Sprint`: an iteration field. People assign sprints during sprint planning; no skill sets it.

Views:

| View | Layout | Filter |
| --- | --- | --- |
| `Current sprint` | board | `sprint:@current` |
| `Backlog` | table | `is:open no:sprint` |
| `Triage` | table | `status:"Needs triage","Needs info"` |

Labels: `bug` and `enhancement` (the category roles), and `spec`.

Where each kind of issue lives:

- A **spec**, filed by `/to-spec`, is an issue labelled `spec` and kept **off** the board.
- A **ticket**, filed by `/to-tickets`, is a sub-issue of its spec and sits on the board.
- Every other issue joins the board when it's triaged.

`wontfix` has no column. Close the issue as "not planned" and its card moves to `Done`.

## Operations

`<n>` is an issue number, `<board>` the board number, `<owner>` its owner, `"<title>"` its title.

**File an issue onto the board.** Create blockers first, so their numbers exist when you reference them:

```bash
gh issue create --title "..." --project "<title>" --body-file - \
  [--label bug] [--parent <spec>] [--blocked-by <n>,<n>] <<'EOF'
...body...
EOF
```

Then set its column and fields. For a spec, drop `--project` and pass `--label spec`.

**Add an existing issue to the board.** `gh issue edit <n> --add-project "<title>"`.

**Move a card or set a field.** The issue must already be on the board:

```bash
gh project item-edit <board> --owner <owner> --url <issue-url> --field Status --value "Ready for agent"
```

The same shape sets `--field Priority --value P1` and `--field Size --value M`. Pass `--clear` in place of `--value` to empty a field.

**List cards.** `--query` takes the board's filter syntax (`status:"Needs info"`, `is:open`, `-label:spec`, `no:status`, `sprint:@current`):

```bash
gh project item-list <board> --owner <owner> --format json -L 1000 --query 'status:"Needs triage"'
```

**List open issues that aren't on the board.**

```bash
gh issue list --state open -L 500 --json number,title,labels,createdAt,projectItems
```

Keep the issues with no `projectItems` entry for this board, or an entry with no status. Skip issues labelled `spec`.

**Read an issue.** `gh issue view <n> --comments` for the conversation, and `gh issue view <n> --json title,body,labels,state,stateReason,parent,subIssues,blockedBy,projectItems` for its structure.

**Comment.** `gh issue comment <n> --body-file -`.

**Close.** `gh issue close <n> --reason completed`, or `--reason "not planned"` for `wontfix`. Add `--comment "..."` to record why.

**Link a sub-issue after the fact.** `gh issue edit <spec> --add-sub-issue <n>`.

**Add a blocking edge after the fact.** `gh issue edit <n> --add-blocked-by <blocker>`. A ticket is **unblocked** when every issue in its `blockedBy` is closed.
