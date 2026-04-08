# `moshi` Shell Function

Use this when the user explicitly wants a `moshi <dir>` helper.

Choose the correct startup file for the active shell. For zsh, prefer `~/.zshrc` unless the user already keeps shell functions elsewhere.

```bash
moshi() {
  local dir="${1:-$PWD}"
  if [[ ! -d "$dir" ]]; then
    echo "Directory not found: $dir" >&2
    return 1
  fi

  local abs
  abs="$(cd "$dir" && pwd)"

  local session
  session="$(basename "$abs" | tr -cs '[:alnum:]_-' '-')"
  session="${session#-}"
  session="${session%-}"
  [[ -n "$session" ]] || session="main"

  if ! tmux has-session -t "$session" 2>/dev/null; then
    tmux new-session -d -s "$session" -c "$abs" -n agent
    tmux new-window -t "$session":2 -c "$abs" -n review
    tmux new-window -t "$session":3 -c "$abs" -n tests
    tmux new-window -t "$session":4 -c "$abs" -n servers
    tmux new-window -t "$session":5 -c "$abs" -n misc
  fi

  tmux attach -t "$session"
}
```

Verification:

```bash
type moshi
```
