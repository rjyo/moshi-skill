# Moshi skill

Standalone agent skill repository for Moshi remote coding best practices on any host OS.

## Included skill

- `moshi-best-practices`: host readiness, tmux defaults, tmux session bootstrap, optional `moshi DIR` shell helper, and `moshi-hook`

## Install with `skills`

From a local checkout:

```bash
npx skills add ./moshi-skill
```

Install the skill from the repo:

```bash
npx skills add ./moshi-skill --skill moshi-best-practices
```

List discoverable skills without installing:

```bash
npx skills add ./moshi-skill --list
```

When this repo is pushed to GitHub, the same layout can be installed with:

```bash
npx skills add <owner>/moshi-skill
```
