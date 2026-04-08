# Host Setup

Use this reference when the machine itself is not ready for Moshi yet.

## Target Outcome

- Remote Login is enabled, or the user explicitly chose not to change it.
- `tmux` and `mosh-server` are installed.
- `tmux` and `mosh-server` resolve from:
  - the current interactive shell
  - the login shell in non-interactive mode
  - `ssh localhost`, when local SSH auth is available
- At least one tmux session exists, so the Moshi selector has something to show.

## Inspect First

Run the smallest set of checks needed to understand the current state:

```bash
sw_vers -productVersion
dscl . -read "/Users/$USER" UserShell
systemsetup -getremotelogin || true
command -v brew || true
command -v tmux || true
command -v mosh-server || true
tmux list-sessions 2>/dev/null || true
```

Then probe the login shell directly:

```bash
LOGIN_SHELL="$(dscl . -read "/Users/$USER" UserShell | awk '{print $2}')"
"$LOGIN_SHELL" -c 'command -v tmux'
"$LOGIN_SHELL" -c 'command -v mosh-server'
```

If `ssh localhost` is already set up without interactive prompts, also check:

```bash
ssh -o BatchMode=yes -o ConnectTimeout=5 localhost 'command -v tmux'
ssh -o BatchMode=yes -o ConnectTimeout=5 localhost 'command -v mosh-server'
ssh -o BatchMode=yes -o ConnectTimeout=5 localhost 'tmux list-sessions -F "#{session_name}" 2>/dev/null'
```

## Common Fixes

### 1. Enable Remote Login

Preferred:

- System Settings -> General -> Sharing -> Remote Login -> On

CLI alternative:

```bash
sudo systemsetup -setremotelogin on
```

### 2. Install tmux and Mosh

```bash
brew install tmux mosh
```

### 3. Fix the non-interactive PATH

On zsh-based macOS systems, the common failure mode is that Homebrew paths only exist in `~/.zprofile`, so `ssh` and Mosh bootstrap shells cannot find `tmux` or `mosh-server`.

Preferred zsh fix:

```bash
export PATH="/opt/homebrew/bin:/usr/local/bin:$PATH"
```

Place that in `~/.zshenv` if it is missing. Do not add duplicate exports.

If the login shell is not `zsh`, adapt the startup file to the actual shell instead of writing `~/.zshenv`.

### 4. Ensure there is at least one tmux session

If no sessions exist yet:

```bash
tmux new-session -d -s main
```

Use `main` unless the user already has a naming convention.

## Verification

The setup is not done until these checks pass:

```bash
command -v tmux
command -v mosh-server
"$LOGIN_SHELL" -c 'command -v tmux && command -v mosh-server'
tmux list-sessions
```

If batch SSH is available, also verify:

```bash
ssh -o BatchMode=yes -o ConnectTimeout=5 localhost 'command -v tmux && command -v mosh-server'
ssh -o BatchMode=yes -o ConnectTimeout=5 localhost 'tmux list-sessions -F "#{session_name}"'
```

Manual Moshi verification:

- reconnect from Moshi with transport set to `Auto` or `Mosh`
- expect the tmux selector to appear
- after connecting, expect the transport badge to show `Mosh` instead of plain `SSH` when Mosh succeeded

## Stop Conditions

Stop and report instead of guessing when:

- the login shell is unusual and you cannot identify the correct startup file safely
- administrator access is required but unavailable
- the machine policy forbids Remote Login or package installation
