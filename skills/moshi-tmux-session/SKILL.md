---
name: moshi-tmux-session
description: Use when the user wants a new tmux session or project workspace that Moshi can attach to cleanly. Trigger this when starting a fresh project inside tmux, wrapping a new agent run in tmux, or creating a predictable session and window layout rooted at a chosen working directory.
---

# Moshi tmux Session

This skill is for creating or preparing a project session. It is not the place to tune `~/.tmux.conf`.

Create tmux sessions in detached mode by default so you do not take over the user's current terminal unexpectedly.

## Ask Before Creating

When the user has not already specified a directory:

1. Read the current working directory.
2. Ask one concise question: whether the new tmux session should start from that directory.
3. If the user says no, ask for the directory to use.

When the user has not already specified a session name:

- default to a normalized name derived from the directory basename
- if that name already exists, ask whether to reuse it or create a different session

## Recommended Session Layout

Use the chosen directory for every initial window with `tmux ... -c <dir>`.

Preferred initial windows:

1. `agent`
2. `review`
3. `tests`
4. `servers`
5. `misc`

Example command pattern:

```bash
tmux new-session -d -s "$SESSION" -c "$DIR" -n agent
tmux new-window -t "$SESSION":2 -c "$DIR" -n review
tmux new-window -t "$SESSION":3 -c "$DIR" -n tests
tmux new-window -t "$SESSION":4 -c "$DIR" -n servers
tmux new-window -t "$SESSION":5 -c "$DIR" -n misc
```

If the session already exists, do not destroy it. Inspect it first and either reuse it or create a new name with the user's approval.

## Verification

Run:

```bash
tmux has-session -t "$SESSION"
tmux list-windows -t "$SESSION"
tmux display-message -p -t "$SESSION":1 "#{pane_current_path}"
```

The first window path should match the chosen directory.

## Final User Prompt

After creating or preparing the session, ask the user to reconnect to the host from Moshi.

Expected result:

- the tmux selector appears during reconnect
- the new session is visible in the selector
- after attaching, the user can experience the full Moshi flow with tmux shortcuts, scrolling, and stable window layout

## Notes

- This skill is intentionally interactive because the session root directory matters.
- Prefer the current working directory as the default suggestion when it is sensible for the task.
