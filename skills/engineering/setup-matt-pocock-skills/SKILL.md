---
name: setup-matt-pocock-skills
description: "Provision this repo's GitHub Projects board for the engineering skills: triage columns, Priority, Size and Sprint fields, views, and labels. Run once per repo before first use of the other engineering skills; safe to re-run."
disable-model-invocation: true
---

# Setup Matt Pocock's Skills

Provision the GitHub Projects **board** that the engineering skills use as their issue tracker. Setup commits nothing to this repo: the other skills find the board at run time. The only file it may touch is the README, and only to add a board link the user asked for.

This is a prompt-driven skill, not a deterministic script. Explore, present what you found, confirm with the user, then write. Everything it creates lives on GitHub where collaborators can see it, so get the user's yes before the first write.

## Process

### 1. Load the board reference

Call the Skill tool with "github-board". Run its preflight steps 1 and 2 (the `gh` checks) and stop if either fails. Its **Board shape** section is the target this skill provisions: every column, field, view and label, with names, colours and filters.

Preflight step 3 (finding a board) is what this skill fixes, so a missing board is expected here.

### 2. Explore

- `git remote -v`: does it point at GitHub? If not, stop and tell the user that these skills only support GitHub Projects.
- `gh repo view --json nameWithOwner,owner,isPrivate,projectsV2`: the repo, its owner, its visibility, and its linked boards.
- Pick the board with github-board's "Find the board" rule. No linked board means this run creates one.
- For a reused board, read what already exists: the `Status` options, other fields, views and workflows (the query in step 4b), and `gh label list --json name`.

Exploration is done when you can say, for every item in Board shape, whether it already exists.

### 3. Present and confirm

Summarise, then ask in one message:

- The board: reuse `<title>` (with its URL), or create `<repo name>`, owned by `<owner>`, with the repo's visibility.
- What will be added: the missing columns, fields, views and labels. On a reused board, name the existing columns that will stay as they are.
- The sprint length, when no `Sprint` field exists yet: "2-week sprints starting Monday <date>?" Offer to line sprint boundaries up with dates the user already works to, such as course milestones or release dates.

Wait for a yes.

### 4. Provision

Create only what's missing. Never delete, rename or reorder anything that existed before this run, because the user may have built on it. The one exception is the seed columns of a board this run just created (4b).

**4a. The board.** When creating one:

```bash
gh project create --owner <owner> --title "<repo name>" --format json   # note the number and url
gh project edit <number> --owner <owner> --visibility PRIVATE            # or PUBLIC, matching the repo
gh project link <number> --owner <owner> --repo <owner>/<repo>
```

**4b. Columns.** Read the board's node ID, then its `Status` field, fields, views and workflows:

```bash
gh project view <number> --owner <owner> --format json   # "id" is the board's node ID
gh api graphql -f query='
query($id: ID!) { node(id: $id) { ... on ProjectV2 {
  field(name: "Status") { ... on ProjectV2SingleSelectField { id options { id name color description } } }
  fields(first: 50) { nodes { ... on ProjectV2FieldCommon { name dataType } } }
  views(first: 50) { nodes { id name layout filter } }
  workflows(first: 20) { nodes { name enabled } }
} } }' -f id=<board node ID>
```

`updateProjectV2Field` replaces the whole option list with the one you send. An option sent with its `id` keeps its cards; one sent without an `id` is created; one left out is deleted, and its cards lose their column. Build the list accordingly:

- **A board this run created** has GitHub's seed columns `Todo`, `In Progress` and `Done`, and no cards. Send the seven Board shape columns in order, reusing `In Progress`'s `id` for `In progress` and `Done`'s `id` for `Done`. Leave `Todo` out.
- **A reused board**: send every existing option with its `id`, `name`, `color` and `description` unchanged and in its current order, then append each Board shape column that has no case-insensitive match.

Every option needs a `color` (from Board shape) and a `description` (its "Means" text). The option list is awkward to pass as flags, so send the mutation as a JSON body written to a temporary file outside the repo:

```bash
gh api graphql --input body.json
```

```json
{
  "query": "mutation($field: ID!, $options: [ProjectV2SingleSelectFieldOptionInput!]!) { updateProjectV2Field(input: {fieldId: $field, singleSelectOptions: $options}) { projectV2Field { ... on ProjectV2SingleSelectField { options { id name } } } } }",
  "variables": {
    "field": "<Status field ID>",
    "options": [
      { "name": "Needs triage", "color": "GRAY", "description": "Nobody has evaluated it yet" },
      { "id": "<existing option ID>", "name": "In progress", "color": "ORANGE", "description": "Someone is working on it" }
    ]
  }
}
```

**4c. Priority and Size.** For each missing field:

```bash
gh project field-create <number> --owner <owner> --name Priority --data-type SINGLE_SELECT --single-select-options "P0,P1,P2"
gh project field-create <number> --owner <owner> --name Size --data-type SINGLE_SELECT --single-select-options "XS,S,M,L,XL"
```

**4d. Sprint.** `gh project field-create` can't make iteration fields, so use GraphQL. `<start>` is the first sprint's start date (`YYYY-MM-DD`) and `<days>` the confirmed sprint length:

```bash
gh api graphql -f query='
mutation($project: ID!, $start: Date!, $days: Int!) {
  createProjectV2Field(input: {projectId: $project, dataType: ITERATION, name: "Sprint",
    iterationConfiguration: {startDate: $start, duration: $days, iterations: []}}) {
    projectV2Field { ... on ProjectV2IterationField { id } } } }' \
  -f project=<board node ID> -f start=<start> -F days=<days>
```

Leave an existing `Sprint` field alone, whatever its length.

**4e. Views.** For each missing view, create it, then set its filter from Board shape:

```bash
gh api graphql -f query='
mutation($project: ID!, $name: String!, $layout: ProjectV2ViewLayout!) {
  createProjectV2View(input: {projectId: $project, name: $name, layout: $layout}) { projectV2View { id } } }' \
  -f project=<board node ID> -f name="Current sprint" -f layout=BOARD_LAYOUT

gh api graphql -f query='
mutation($view: ID!, $filter: String!) {
  updateProjectV2View(input: {viewId: $view, filter: $filter}) { clientMutationId } }' \
  -f view=<view ID> -f filter='sprint:@current'
```

Use `TABLE_LAYOUT` for `Backlog` and `Triage`. The API can't set a view's grouping or sort order; those go in the manual steps in step 5.

**4f. Labels.** For each Board shape label that `gh label list` didn't show:

```bash
gh label create spec --color 5319E7 --description "A spec; its tickets are sub-issues on the board"
gh label create bug --color D73A4A --description "Something isn't working"
gh label create enhancement --color A2EEEF --description "New feature or request"
```

**4g. Workflows.** In the 4b query result, check the workflow named `Item closed`. If it's disabled, warn the user that closed issues won't move to `Done` until they enable it in the board's workflow settings. The API can read workflows but can't enable them.

Provisioning is done when every column, field, view and label in Board shape exists on the board.

### 5. Done

- Offer to add one line to the README, `Project board: <board URL>`, so collaborators can find the board. Edit the README only on a yes.
- Give the user the board URL and the manual steps the API can't do:
  - In `Backlog`, sort by `Priority`, then `Size`.
  - Optionally, group `Backlog` or `Current sprint` by `Parent issue` to see the tickets under each spec.
  - To put issues filed in the web UI onto the board automatically, enable the **Auto-add to project** workflow with a filter that excludes specs, such as `is:issue is:open -label:spec`. Without it, `/triage` still finds them.
  - On a reused board, drag any appended columns into the Board shape order.
- Tell the user which skills now use the board: `/to-spec`, `/to-tickets` and `/triage`, with `/code-review` reading linked issues. Re-running setup is safe: it only adds what's missing.
