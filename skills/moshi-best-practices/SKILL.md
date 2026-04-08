---
name: moshi-best-practices
description: Use when preparing or debugging a Mac for Moshi remote coding. Trigger this for SSH or Mosh readiness, Remote Login, non-interactive shell PATH issues, tmux defaults, creating a tmux project session rooted at a chosen directory, installing Moshi agent hooks for Claude Code or Codex CLI, or offering the optional `moshi DIR` shell helper.
---

# Moshi Best Practices

Use this skill to make a Mac feel good in Moshi with the fewest safe changes.

## Rules

- Inspect before editing.
- Prefer direct config edits over wrapper scripts.
- Verify every outcome after changing it.
- For `moshi <dir>`, use a shell function named `moshi`, not a literal alias. Aliases cannot take directory arguments safely.

## 1. Host Readiness

Target outcome:

- Remote Login is on if the user wants Moshi access.
- `tmux` and `mosh-server` are installed.
- both resolve in the current shell and in the login shell's non-interactive mode.
- at least one tmux session exists so the Moshi selector can appear.

Inspect:

```bash
dscl . -read "/Users/$USER" UserShell
systemsetup -getremotelogin || true
command -v tmux || true
command -v mosh-server || true
tmux list-sessions 2>/dev/null || true
LOGIN_SHELL="$(dscl . -read "/Users/$USER" UserShell | awk '{print $2}')"
"$LOGIN_SHELL" -c 'command -v tmux'
"$LOGIN_SHELL" -c 'command -v mosh-server'
```

Common fixes:

- enable Remote Login
- `brew install tmux mosh`
- if Homebrew binaries are missing in non-interactive zsh, add `/opt/homebrew/bin:/usr/local/bin` to `~/.zshenv`
- create `tmux new-session -d -s main` if no sessions exist

Verify:

```bash
command -v tmux
command -v mosh-server
"$LOGIN_SHELL" -c 'command -v tmux && command -v mosh-server'
tmux list-sessions
```

Then ask the user to reconnect from Moshi. Expected result: the tmux selector appears, and Mosh can work instead of plain SSH when configured.

## 2. tmux Environment

Use these defaults unless the user wants something different:

```tmux
set -g history-limit 100000
set -g mouse on
set -g set-titles on
set -g set-titles-string "#I: #W"
set -g base-index 1
setw -g pane-base-index 1
set -g renumber-windows on
```

Workflow:

- inspect `~/.tmux.conf`
- update overlapping settings instead of appending duplicates
- reload with `tmux source-file ~/.tmux.conf`

Verify:

```bash
tmux show -g history-limit
tmux show -g mouse
tmux show -g set-titles-string
tmux show -g base-index
tmux show -gw pane-base-index
tmux show -g renumber-windows
```

## 3. tmux Project Session

When creating a new session:

- read the current working directory
- ask one concise question: should the session start from here?
- if the answer is no, ask for the directory
- default the session name to the directory basename
- create the session detached
- use the chosen directory for every initial window with `tmux ... -c <dir>`

Recommended windows:

1. `agent`
2. `review`
3. `tests`
4. `servers`
5. `misc`

Pattern:

```bash
tmux new-session -d -s "$SESSION" -c "$DIR" -n agent
tmux new-window -t "$SESSION":2 -c "$DIR" -n review
tmux new-window -t "$SESSION":3 -c "$DIR" -n tests
tmux new-window -t "$SESSION":4 -c "$DIR" -n servers
tmux new-window -t "$SESSION":5 -c "$DIR" -n misc
```

Verify:

```bash
tmux has-session -t "$SESSION"
tmux list-windows -t "$SESSION"
tmux display-message -p -t "$SESSION":1 "#{pane_current_path}"
```

Then ask the user to reconnect in Moshi. Expected result: the session is visible in the tmux selector.

## 4. Optional `moshi <dir>` Helper

Do not install this silently. Ask the user first if they want it.

If yes:

- install a shell function named `moshi` in the correct shell startup file
- make it accept a directory argument, defaulting to `$PWD`
- name the tmux session from the directory basename
- create the standard detached session layout only if the session does not already exist
- attach to the session afterward

Use the exact function from `references/moshi-shell-function.md`.

Verify:

```bash
type moshi
```

If the user wants a non-interactive check, create a temporary directory and run `moshi <dir>` from another terminal, or inspect the function body without invoking it.

## 5. Agent Hooks

Use `moshi-hooks`, not hand-written config, unless the user explicitly wants manual edits.

Core commands:

```bash
bunx moshi-hooks setup
bunx moshi-hooks token <YOUR_TOKEN>
```

Optional integrations:

```bash
bunx moshi-hooks setup --local
bunx moshi-hooks setup .
bunx moshi-hooks setup --codex
bunx moshi-hooks setup --opencode
```

Verify:

```bash
test -f ~/.config/moshi/token
rg -n "moshi-hooks" ~/.claude/settings.json ~/.claude/settings.local.json .claude/settings.json .claude/settings.local.json 2>/dev/null
rg -n 'moshi-hooks|notify =' ~/.codex/config.toml 2>/dev/null
```

Final verification:

- run a short real agent task
- confirm Moshi receives a push notification or Live Activity update
