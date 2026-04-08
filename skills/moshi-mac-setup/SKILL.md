---
name: moshi-mac-setup
description: Use when preparing or debugging a macOS machine for Moshi remote coding. Trigger this when enabling Remote Login, fixing SSH or Mosh reachability, making `tmux` and `mosh-server` visible to non-interactive shells, configuring Moshi agent hooks for Claude Code, Codex CLI, or OpenCode, or deciding which Moshi tmux skill should handle the next step.
---

# Moshi Mac Setup

This is the entry skill for Moshi host preparation.

Keep this skill outcome-oriented:

- inspect first
- make direct, idempotent changes
- verify the end state after each change
- do not create wrapper shell scripts unless the user explicitly asks for automation

## Scope Selection

Use the smallest relevant slice of this setup:

- Base host reachability, Remote Login, Homebrew PATH, `tmux`, `mosh-server`: read `references/host-setup.md`
- Agent event delivery into Moshi push notifications and Live Activity: read `references/agent-hooks.md`
- Global tmux defaults for scrolling, titles, and numbering: use `$moshi-tmux-environment`
- Creating a tmux session for a project or new agent run: use `$moshi-tmux-session`

## Working Rules

1. Inspect the current machine state before editing files or enabling services.
2. Prefer small direct edits to the real config files over generating helper scripts.
3. If the login shell is not `zsh`, adapt the PATH fix to the active shell instead of blindly writing `~/.zshenv`.
4. When tmux session creation depends on a working directory, ask the user before creating it. Default to the current working directory if the user agrees.
5. After each setup area, run the verification commands from the relevant reference or tmux skill.
6. If a required change needs administrator access or conflicts with an existing local convention, stop and report the blocker clearly.
