# Moshi skill

Standalone agent skill repository for Moshi remote host setup and tmux workflows.

## Included skills

- `moshi-mac-setup`: entry skill for host setup, SSH or Mosh readiness, and agent hooks
- `moshi-tmux-environment`: global tmux configuration for Moshi-friendly scrolling, titles, and numbering
- `moshi-tmux-session`: project or agent session bootstrap rooted at a chosen directory

## Install with `skills`

From a local checkout:

```bash
npx skills add ./moshi-skill
```

Install only one skill from the repo:

```bash
npx skills add ./moshi-skill --skill moshi-mac-setup
```

List discoverable skills without installing:

```bash
npx skills add ./moshi-skill --list
```

When this repo is pushed to GitHub, the same layout can be installed with:

```bash
npx skills add <owner>/moshi-skill
```
