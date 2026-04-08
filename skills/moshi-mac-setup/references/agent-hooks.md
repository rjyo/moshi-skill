# Agent Hooks

Use this reference when the host is reachable but Moshi is not receiving agent lifecycle events yet.

## Target Outcome

- `moshi-hooks` is installed or runnable through `bunx`
- a valid Moshi token is saved at `~/.config/moshi/token`
- the relevant agent integration is installed:
  - Claude Code hooks
  - Codex CLI notify
  - OpenCode plugin
- a real agent event reaches Moshi and produces a push or Live Activity update

## Token Source

Get the token from the Moshi app first. The app UI currently shows:

```bash
bunx moshi-hooks setup
bunx moshi-hooks token <YOUR_TOKEN>
```

If `moshi-hooks` is installed globally, `moshi-hooks token <YOUR_TOKEN>` is equivalent.

## Setup Commands

### Claude Code

User scope:

```bash
bunx moshi-hooks setup
```

User-local scope:

```bash
bunx moshi-hooks setup --local
```

Project scope from the current directory:

```bash
bunx moshi-hooks setup .
```

Project-local scope from the current directory:

```bash
bunx moshi-hooks setup --local .
```

### Codex CLI

```bash
bunx moshi-hooks setup --codex
```

This writes a `notify` entry to `~/.codex/config.toml`.

### OpenCode

```bash
bunx moshi-hooks setup --opencode
```

This writes `.opencode/plugins/moshi-hooks.ts` in the selected project.

### Save the token

```bash
bunx moshi-hooks token <YOUR_TOKEN>
```

## Verification

Verify the token:

```bash
test -f ~/.config/moshi/token
cat ~/.config/moshi/token
```

Verify Claude hook registration:

```bash
rg -n "moshi-hooks" ~/.claude/settings.json ~/.claude/settings.local.json .claude/settings.json .claude/settings.local.json 2>/dev/null
```

Verify Codex notify registration:

```bash
rg -n 'moshi-hooks|notify =' ~/.codex/config.toml 2>/dev/null
```

Verify OpenCode plugin registration:

```bash
test -f .opencode/plugins/moshi-hooks.ts && rg -n "moshi-hooks" .opencode/plugins/moshi-hooks.ts
```

Functional verification:

- run a short real agent task that reaches a supported lifecycle event
- for Claude Code, `Stop`, `Notification`, `PreToolUse`, and `PostToolUse` are the important live events
- for Codex, verify a turn-complete notification path
- confirm that Moshi receives the event as a push notification or Live Activity update

## Working Rules

- Prefer the supported `moshi-hooks` CLI over manually writing JSON or TOML.
- Do not hardcode temporary `bunx` cache paths into config files.
- Preserve existing hooks from other tools.
- If the user only wants Claude Code, do not install Codex or OpenCode integrations unless asked.
