# The canonical install block

One install story, one wording. `README.md` and `.changeset/*` must say **this** and nothing else. Change it here first, then propagate.

`monopoly-skills` is not in Claude Code's official marketplace. It installs from this fork's own marketplace, `.claude-plugin/marketplace.json`, so the marketplace is added first.

## Claude Code: the plugin

<canonical-block name="claude-code">

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

</canonical-block>

## Codex, and other agents: skills.sh

The plugin is Claude Code only. Everywhere else, [skills.sh](https://skills.sh/shoang22/skills) copies editable skill files into the project. Use the whole-set form on `README.md`:

<canonical-block name="skills-sh-whole-set">

```bash
npx skills@latest add shoang22/skills
```

Pick the skills you want, and which coding agents to install them on. **The installer lets you choose which skills to take: make sure `setup-matt-pocock-skills` and `github-board` are among them.**

</canonical-block>

…and the single-skill form wherever one skill is named on its own.

<canonical-block name="skills-sh-one-skill">

```bash
npx skills@latest add shoang22/skills --skill=<name>
```

```bash
npx skills@latest update <name>
```

</canonical-block>

`skills@latest` is the pinned spelling in all three.

## The two routes are exclusive

The plugin is a managed, read-only bundle you subscribe to. skills.sh writes files you own and edit. Installing both leaves the user with every skill twice: always say "pick one".

## Not alongside upstream

Upstream's `mattpocock-skills` plugin ships its own `triage`, `to-spec`, `to-tickets` and `setup-matt-pocock-skills`, wired to the per-repo tracker files this fork removed. Installing both gives two of each; always say "not alongside `mattpocock-skills`".
