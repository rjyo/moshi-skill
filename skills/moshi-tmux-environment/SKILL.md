---
name: moshi-tmux-environment
description: Use when configuring tmux itself for the best Moshi experience. Trigger this when tmux scrolling does not work well on mobile, window numbering starts at zero, titles are not helpful in Moshi, or the user wants the recommended `~/.tmux.conf` defaults for agent-driven workflows.
---

# Moshi tmux Environment

This skill configures the global tmux defaults. It is about `~/.tmux.conf`, not about creating a project session.

Do not generate helper scripts. Inspect the existing config, merge carefully, and verify the active tmux server picks up the new values.

## Desired Defaults

The recommended Moshi-oriented defaults are:

```tmux
set -g history-limit 100000
set -g mouse on
set -g set-titles on
set -g set-titles-string "#I: #W"
set -g base-index 1
setw -g pane-base-index 1
set -g renumber-windows on
```

## Workflow

1. Inspect `~/.tmux.conf` first.
2. If the file already contains overlapping settings, update the existing values instead of appending conflicting duplicates.
3. Apply the recommended defaults unless the user explicitly wants a different convention.
4. Reload the config with `tmux source-file ~/.tmux.conf`.

## Verification

Run:

```bash
tmux source-file ~/.tmux.conf
tmux show -g history-limit
tmux show -g mouse
tmux show -g set-titles
tmux show -g set-titles-string
tmux show -g base-index
tmux show -gw pane-base-index
tmux show -g renumber-windows
```

Manual Moshi verification:

- reconnect through Moshi and attach to a tmux session
- confirm swipe scrolling works
- confirm window numbering starts at `1`
- confirm the titles shown in Moshi are stable and useful

## Notes

- Moshi can enable mouse mode when attaching through the tmux selector, but the config should still set `mouse on` so the behavior is stable everywhere.
- This skill does not create or restructure sessions. Use `$moshi-tmux-session` for that.
