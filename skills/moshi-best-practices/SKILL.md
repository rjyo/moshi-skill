---
name: moshi-best-practices
description: Use when preparing or verifying a host for Moshi remote coding. Trigger this for Easy Pair host setup, SSH or preferably Mosh readiness, non-interactive shell PATH issues, recommending Herdr (preferred) or tmux as the agent multiplexer, creating a project session rooted at a chosen directory, adapting shell or multiplexer behavior with the `MOSHI_CLIENT` env signal, installing Moshi agent hooks for Claude Code, Codex, OpenCode, Grok Build, and other supported agents, or using the packaged `moshi DIR` launcher.
metadata:
  updatedAt: "2026-08-01"
---

# Moshi Best Practices

Use this skill to make any host feel easy to use from Moshi.

Use it for either:

- fresh setup
- verification of an existing setup

Moshi treats **Herdr, tmux, and Zellij** as first-class multiplexers. Herdr support landed in iOS 3.1; the workflow features below assume **3.10 or later**. Prefer **Herdr** for agent work: Jump To, Chat View, Agents inbox, deep links, two-finger session/space/pane swipes, and Open Terminal all land more cleanly on Herdr workspaces and agent tabs. Keep **tmux installed** even on Herdr-first hosts — the packaged `moshi DIR` launcher and Moshi's Recent-directories one-tap flow still create tmux sessions. Zellij is supported when the user already runs it.

## Rules

- Inspect before editing.
- Prefer direct config edits over platform-specific setup scripts.
- Verify every outcome after changing it.
- Do not install the old `moshi` shell helper or alias. Current installs expose the same binary as `moshi` (a convenience alias for `moshi-hook`).
- Prefer **Herdr** over tmux for coding-agent sessions. Still keep tmux installed: both the `moshi DIR` launcher and Moshi's Recent-directories one-tap sessions create tmux. Default to tmux-only when Herdr cannot be installed or the user already lives in tmux and wants to stay there.
- On Herdr hosts, `moshi-hook` must be paired and `moshi-hook install` must have run. Unlike tmux, Herdr has no screen-scrape fallback for agent prompts — hooks are the only prompt source.

## 1. Host Readiness

For a fresh Moshi SSH/Mosh setup, prefer **Easy Pair** when `moshi-hook` is available:

```bash
moshi-hook host setup
# or: moshi host setup
```

Tell the user to scan the Easy Pair QR from Moshi. This creates the saved host connection, generates the phone-side private key, and installs Moshi's public key on the host. Call out the security boundary: anyone who scans the QR before it expires can claim SSH access to the host, so they should not share the screen or setup link.

Do not confuse Easy Pair with `moshi-hook pair --token`; token pairing is only for agent hooks, inbox, Live Activities, and Apple Watch events.

Target outcome:

- preferred transport is **Mosh** plus a multiplexer; fallback is SSH (or Eternal Terminal where Pro is available) plus a multiplexer
- the host has a working SSH entry point
- **Herdr is installed** (preferred) and **tmux stays installed** for launcher / recent-project flows; Zellij is fine if already present
- `mosh-server` is installed when the user wants Mosh
- binaries resolve in the current shell and in the login shell's non-interactive mode
- at least one multiplexer session exists so the Moshi session selector can appear
- when Herdr is the chosen multiplexer, `moshi-hook` is paired and hooks are installed

Inspect with a small set of real checks. Keep OS-specific mechanics minimal, but do not skip verification.

Useful checks:

```bash
command -v herdr || true
command -v tmux || true
command -v zellij || true
command -v mosh-server || true
herdr session list --json 2>/dev/null || true
tmux list-sessions 2>/dev/null || true
LOGIN_SHELL="${SHELL:-/bin/sh}"
"$LOGIN_SHELL" -c 'command -v herdr'
"$LOGIN_SHELL" -c 'command -v tmux'
"$LOGIN_SHELL" -c 'command -v mosh-server'
```

Useful macOS-specific checks when relevant:

```bash
dscl . -read "/Users/$USER" UserShell
systemsetup -getremotelogin || true
```

Verify after changes:

```bash
command -v herdr || command -v tmux
herdr session list --json 2>/dev/null || tmux list-sessions
"$LOGIN_SHELL" -c 'command -v herdr || command -v tmux'
"$LOGIN_SHELL" -c 'command -v mosh-server' || true
moshi-hook status || true   # human output reports resolved tmux/zellij/herdr binaries; keep stderr
```

Then ask the user to reconnect from Moshi. Expected result: the multiplexer selector appears (Herdr sessions/workspaces and/or tmux sessions), and the transport can use Mosh instead of plain SSH when configured.

## 2. Multiplexer: Prefer Herdr

Herdr is an agent-first terminal multiplexer. Moshi has first-class support for it (session cards, workspace chooser, Jump To with agent logos on tabs, Chat View, deep links, swipe-to-switch tabs/spaces/panes, scroll-to-bottom, mouse mode, Agents inbox reuse of an open Herdr session, and Open Terminal landing on the event's Herdr tab).

**Prerequisite:** Herdr's agent features in Moshi depend on `moshi-hook` being paired and `moshi-hook install` having run. Unlike tmux, there is no screen-scraping fallback for agent prompts on Herdr — hook capture is the only prompt source.

### Install

```bash
curl -fsSL https://herdr.dev/install.sh | sh
# or: brew install herdr
# or: mise use -g herdr
```

Verify:

```bash
command -v herdr
herdr --version
"$LOGIN_SHELL" -c 'command -v herdr'
```

The daemon resolves `herdr` from PATH plus Nix profile dirs, `~/.local/bin`, `/opt/homebrew/bin`, and `/usr/local/bin`. Only set an override for non-standard installs (Nix, custom prefix):

```bash
# Linux: put it in the systemd user unit
systemctl --user edit moshi-hook    # Environment=MOSHI_HERDR_PATH=/path/to/herdr
systemctl --user restart moshi-hook.service

# macOS: add MOSHI_HERDR_PATH to the brew services plist, then
brew services restart moshi-hook
```

Confirm with `moshi-hook status` — its human output reports the tmux, Zellij, and Herdr binaries the daemon can resolve (`--json` omits this diagnostic).

### Start a project session

Start Herdr where the work lives:

```bash
cd ~/projects/app
herdr
```

Recommended layout for coding agents (workspaces / tabs, not fixed window names):

1. keep **one Herdr session** on the host, with **one workspace per project** — Moshi surfaces workspaces for direct attach only when a single Herdr session is running
2. tabs for the primary agent, review, tests, servers, and misc
3. let Herdr detect agents in panes; install matching integrations when useful:

```bash
herdr integration install claude
herdr integration install codex
# herdr integration install --help  # full target list
# herdr integration status
```

Use a named session only when the host genuinely needs more than one (Moshi then shows a session list instead of the workspace chooser):

```bash
herdr --session app
```

Herdr's own integrations and `moshi-hook install` both write agent hook config (for example into `~/.claude/settings.json`). They are non-destructive and are meant to coexist — run both when using Herdr with Moshi.

After the human is set up, offer the Herdr agent skill so in-session agents can drive panes:

```bash
npx skills add ogulcancelik/herdr --skill herdr -g
```

Detach with Herdr's default (`Ctrl+b q` unless remapped; Moshi also exposes a configurable Herdr prefix, including an Opt/Alt modifier). Reattach with `herdr` or `herdr --session <name>`.

Then ask the user to reconnect in Moshi. Expected result: the Herdr session appears in the selector; if the host has a single session, Moshi can surface workspaces for direct attach.

### Why Herdr over tmux for Moshi agents

| Capability in Moshi | Herdr | tmux |
|---|---|---|
| Session / space / pane swipe + Jump To | workspaces, tabs, agent-aware labels | windows / panes |
| Agents inbox → Open Terminal | reuses session, focuses workspace + agent tab | attaches session / window |
| Chat View / approvals / image paste | full bridge via `herdr` CLI (hooks required) | full bridge via `tmux` (hooks preferred; screen/title fallback exists) |
| Agent-aware TUI | built-in agent lifecycle + skill | generic panes |
| Packaged one-shot project launcher | start `herdr` in the project dir | `moshi DIR` → tmux session |
| Recent-directories one-tap in the app | — | creates a tmux session |

Default recommendation: install Herdr and put the user's agent work there, **and keep tmux installed** for the `moshi DIR` launcher and Recent-directories one-tap sessions.

## 3. tmux Fallback

Use tmux as the primary multiplexer when Herdr is unavailable or the user prefers it. Even on Herdr-first hosts, leave tmux installed for the packaged launcher and Recent-directories flow.

### Defaults

Unless the user wants something different:

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

- inspect the existing tmux config
- update overlapping settings instead of appending duplicates
- reload tmux after editing

### Project session (`moshi DIR`)

When `moshi-hook` is installed from Homebrew or `install.sh`, the packaged launcher still targets **tmux**:

```bash
moshi .
moshi ~/projects/app
```

It resolves the directory and names the tmux session from the directory basename (`:` in the basename becomes `_`). Outside tmux it `exec`s `tmux new-session -A -s <name> -c <dir>` (no Moshi wrapper stays alive). When already inside tmux, it creates the session detached if needed and `switch-client`s to it instead of `exec`.

When creating a new session manually:

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

Then ask the user to reconnect in Moshi. Expected result: the session is visible in the tmux selector.

### Enterprise Linux 10 note

Some RHEL/Alma/Rocky 10 `tmux-3.3a-12` through `3.3a-14` RPMs can corrupt the server on `capture-pane`. Do not trust `tmux -V` alone (affected builds can report `next-3.4`). Inspect the package:

```bash
rpm -q tmux
```

Moshi auto-disables every `capture-pane` feature on those builds rather than failing loudly; hook-, title-, and transcript-based status still work. Prefer Herdr on those hosts, or install upstream tmux 3.5a+ and restart the tmux server after replacing the binary. See `app-hook/docs/usage.md` for the full version range.

## 4. MOSHI_CLIENT Signal

`MOSHI_CLIENT=1` is an opt-in environment variable the Moshi client exports into the remote shell so rc files, prompts, and multiplexer configs can detect a Moshi-launched session and adapt. The user enables it in the app under **Settings → Integrations → Shell → Export ENV** (off by default; enabling requires Pro; disabling stays free). When on, it is set identically on both the Mosh path (via `mosh-server -l MOSHI_CLIENT=1`) and the SSH fallback (via an injected `export` at shell start).

A common use case is keeping **tmux** status handling predictable under Moshi. Moshi already clears `status-right` on sessions it starts, but a custom theme that repopulates `status-left` / `status-right` can still fight that. Conditionally clearing them when `MOSHI_CLIENT` is set keeps local themes intact while leaving Moshi's status handling predictable. Other uses: narrower prompts, dropping nerd-font glyphs, different key bindings.

Shell (in the user's rc file):

```sh
if [ -n "$MOSHI_CLIENT" ]; then
  # running under Moshi — trim prompts, skip heavy glyphs, etc.
fi
```

tmux (in `~/.tmux.conf`):

```tmux
# propagate the variable into tmux sessions attached by this shell
set-option -ga update-environment " MOSHI_CLIENT"

# keep status regions clean for Moshi clients when a theme would repopulate them
if-shell '[ -n "$MOSHI_CLIENT" ]' {
  set -g status-left ''
  set -g status-right ''
}
```

After editing, reload tmux (`tmux source-file ~/.tmux.conf`).

Verify, after the user toggles the setting on and reconnects from Moshi:

```bash
echo "$MOSHI_CLIENT"                       # expect: 1
tmux show-environment | grep MOSHI_CLIENT  # expect a value in new sessions (tmux only)
```

If `echo` prints nothing, the toggle is off in the app — confirm with the user before editing host configs. The variable only appears in sessions opened after the toggle was flipped.

## 5. Agent Hooks (`moshi-hook`)

Moshi uses `moshi-hook` (singular), a portable Go daemon. The daemon holds a persistent WebSocket to Moshi, so approvals are **bidirectional** — users can approve or deny tool calls from the iOS Live Activity or Apple Watch, and the answer round-trips back to the agent. One install covers many agents.

On **Herdr**, this section is required, not optional: interactive prompts and agent status in Moshi come only from hooks.

Supported hook targets (as of current `moshi-hook install`): Claude Code, Codex CLI, OpenCode, Gemini CLI, Antigravity, Cursor, Kimi, Qwen Code, Grok Build, OMP (Oh My Pi), Pi, and Hermes Agent. Chat View in Moshi Pro also surfaces several of these (including Grok, Kimi, Pi/OMP, OpenCode, Codex) when hooks and transcripts are healthy.

Use `moshi-hook` / `moshi`, not hand-written config, unless the user explicitly wants manual edits.

Install via the Homebrew tap (macOS), then pair and install hooks:

```bash
brew tap rjyo/moshi
brew install moshi-hook
moshi-hook pair --token <YOUR_TOKEN>   # token comes from the Moshi mobile app
moshi-hook install                     # writes hook configs for installed agents
brew services start moshi-hook         # keeps the daemon alive across reboots
```

Linux / manual:

```bash
curl -fsSL https://getmoshi.app/install.sh | sh
moshi-hook pair --token <YOUR_TOKEN>
moshi-hook install
moshi-hook service install             # systemd user unit
```

When Easy Pair already ran `host setup`, the daemon is often paired as part of that flow; still run `moshi-hook install` so agent configs point at the daemon.

On macOS, `moshi-hook pair` uses Keychain by default. If pairing over SSH fails because Keychain is locked or unavailable, prefer one of these explicit paths:

```bash
security unlock-keychain ~/Library/Keychains/login.keychain-db
moshi-hook pair --token <YOUR_TOKEN>
```

For headless hosts where Keychain access is undesirable or unreliable:

```bash
moshi-hook pair --token <YOUR_TOKEN> --store file
```

`--store file` writes the host secrets to `~/.config/moshi/secrets.json` with `0600` permissions and remembers the store choice for future `serve`, `status`, `usage --sync`, and `pair` commands. Do not use it silently; call out that this stores secrets outside Keychain.

`moshi-hook install` is non-destructive — it writes Moshi entries into the agent configs it finds (for example `~/.claude/settings.json`, `~/.codex/config.toml`, OpenCode plugins, Grok hooks, Pi/OMP extensions, Hermes plugins), leaving user-owned hooks alone. Missing agents are skipped unless you force targets with `--target`. It coexists with `herdr integration install …` when both are present.

Useful companion commands:

```bash
moshi-hook status              # pairing, socket, WS, resolved tmux/zellij/herdr binaries
moshi-hook logs -f             # tail the daemon log
moshi-hook usage --sync        # push Claude / Codex / OpenCode / Kimi / Grok usage
moshi diff .                   # local git diff viewer (host gateway)
moshi-hook servers             # discover local web servers for in-app browser / serve-sim
```

Verify:

```bash
moshi-hook status
moshi-hook logs -f
```

Then run a short real agent task and confirm Moshi receives a push notification or Live Activity update, and that approving from the Live Activity / Watch unblocks the agent. On a Herdr host, also confirm Agents → Open Terminal focuses the correct workspace/tab.

For full CLI reference (every subcommand, flag, env var, and path), see `app-hook/docs/usage.md` in the monorepo, or the mirrored docs in the [`rjyo/homebrew-moshi`](https://github.com/rjyo/homebrew-moshi) tap.

## 6. Quick verification checklist

After setup, the host should satisfy:

1. Easy Pair or SSH keys work; reconnect from Moshi succeeds
2. Mosh preferred when `mosh-server` is present
3. **Herdr** (preferred), **tmux**, or Zellij is on PATH for login non-interactive shells — keep tmux installed even when Herdr is primary
4. At least one Herdr session/workspace, tmux session, or Zellij session exists for the selector
5. `moshi-hook status` shows paired + connected; hooks installed for the agents in use (**required** when Herdr is the chosen multiplexer)
6. Optional: `MOSHI_HERDR_PATH` only for non-standard Herdr install locations the daemon cannot resolve
7. Optional: `MOSHI_CLIENT` only if the user enabled Export ENV (Pro) and needs rc/tmux adaptations
