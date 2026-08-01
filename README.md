# Moshi skill

Standalone agent skill repository for Moshi remote coding best practices on any host OS.

## Included skill

- `moshi-best-practices`: Easy Pair host setup, host readiness, **Herdr as the preferred agent multiplexer** (keep tmux installed for launcher / recent-project flows), project session bootstrap, `MOSHI_CLIENT` adaptations, and `moshi-hook` (including the packaged `moshi DIR` tmux launcher)

## Install with `skills`

From this checkout (directory name may be `skill` or a clone named `moshi-skill`):

```bash
npx skills add .
# or, if the checkout directory is named moshi-skill:
npx skills add ./moshi-skill
```

Install the skill from the repo:

```bash
npx skills add . --skill moshi-best-practices
```

List discoverable skills without installing:

```bash
npx skills add . --list
```

When this repo is published on GitHub, the same layout can be installed with the owner/repo path that matches the remote:

```bash
npx skills add <owner>/<repo>
```
