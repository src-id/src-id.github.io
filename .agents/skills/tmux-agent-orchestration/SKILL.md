---
name: tmux-agent-orchestration
description: Use when controlling coding agents in tmux sidecars.
version: 1.0.0
author: Hermes Agent
license: MIT
platforms: [linux, macos, windows]
metadata:
  hermes:
    tags: [tmux, coding-agent, verification, orchestration]
    related_skills: [claude-code, codex, opencode, verification-before-completion]
---

# tmux Agent Orchestration

## Trigger

Use when a coding agent is already running inside tmux, user asks to inspect/control it, or verification/build commands need clean terminal control while agent TUI stays open.

## Workflow

1. List sessions and panes before acting:
   ```bash
   tmux list-sessions
   tmux list-panes -a -F '#{session_name}:#{window_index}.#{pane_index} #{pane_current_path} #{pane_current_command}'
   ```
2. Capture relevant pane:
   ```bash
   tmux capture-pane -t <session> -p -S -120
   ```
3. Send long prompts through tmux buffer, not shell quoting:
   ```bash
   python3 - <<'PY'
   import subprocess
   prompt = '''...'''.strip()
   subprocess.run(['tmux', 'set-buffer', '-t', '<session>', prompt], check=True)
   subprocess.run(['tmux', 'paste-buffer', '-t', '<session>'], check=True)
   subprocess.run(['tmux', 'send-keys', '-t', '<session>', 'Enter'], check=True)
   PY
   ```
4. For deterministic checks, create sidecar shell in same repo:
   ```bash
   tmux new-session -d -c /path/to/repo -s <project>-terminal bash
   ```
5. Run format/lint/typecheck/build in sidecar or Hermes terminal, not inside busy Claude TUI, when user wants controllable logs.
6. Before claiming pass, read actual command output and exit code.

## Git hygiene pattern

- Check `git status --short` before edits/commits.
- If user asks to ignore local source/reference dump with spaces in name, add explicit anchored rule:
  ```gitignore
  /Tukang Digital - Koperasi/
  ```
- Verify ignored folder disappears from `git status --short`.
- Do not stage unrelated untracked docs/dumps unless user asks.

## Pitfalls

- Do not trust agent report alone. Verify with real commands.
- Do not run verification through Claude if Claude auto mode/classifier blocks Bash; use sidecar terminal.
- Do not commit huge Prettier churn without telling user it is broad formatting.
- Do not touch `.env`, secrets, credentials, or private data when repo targets public static hosting.

## References

- `references/ksu-lidia-sidecar-verification.md` — session-derived example for KSU Lidia tmux sidecar, Prettier/typecheck/build loop, and ignored local folder.
