# skill-markserv

A [Claude Code](https://claude.com/claude-code) skill for managing a local [markserv](https://github.com/markserv/markserv) instance — start and stop Markdown preview servers with automatic port selection.

## Features

- Start markserv on a file or directory with auto port selection
- Stop running markserv instances
- Handles multiple concurrent instances
- Auto-stop on session end (opt-in via `SessionEnd` hook)

## Installation

```bash
claude plugin install zigorou/skill-markserv
```

Or via the `zigorou-marketplace`:

```bash
claude plugin install skill-markserv@zigorou-marketplace
```

## Usage

```
/markserv              # Start preview on current directory
/markserv stop         # Stop running instance
/markserv status       # Show running instances (port, root dir, URL)
```

Or just ask naturally:

- "start markserv"
- "I want to preview this README"
- "stop markserv"

## Auto-stop on session end

To automatically stop markserv when a Claude Code session ends — without affecting instances started in other sessions — add both a `SessionStart` and a `SessionEnd` hook to your `~/.claude/settings.json`:

```json
{
  "hooks": {
    "SessionStart": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "INPUT=$(cat); SESSION_ID=$(echo \"$INPUT\" | python3 -c \"import sys,json; print(json.load(sys.stdin)['session_id'])\"); echo \"export CLAUDE_SESSION_ID=$SESSION_ID\" >> \"$CLAUDE_ENV_FILE\""
          }
        ]
      }
    ],
    "SessionEnd": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "SESSION_ID=$(cat | python3 -c \"import sys,json; print(json.load(sys.stdin)['session_id'])\"); PID_FILE=\"/tmp/markserv-cc-${SESSION_ID}.pid\"; [ -f \"$PID_FILE\" ] && xargs kill < \"$PID_FILE\" 2>/dev/null; rm -f \"$PID_FILE\""
          }
        ]
      }
    ]
  }
}
```

How it works:

- **`SessionStart`**: writes `export CLAUDE_SESSION_ID=<id>` into `$CLAUDE_ENV_FILE`, making the session ID available to every Bash tool call in the session.
- **`SessionEnd`**: reads the session ID from stdin, kills all PIDs recorded in `/tmp/markserv-cc-<session_id>.pid`, then removes the file. Only processes started in **this session** are affected.

> **Note:** `SessionEnd` runs before the session fully closes, so `kill` works reliably. The default hook timeout is 1.5 s, which is more than enough for a simple `kill`.

## Requirements

- Node.js (for `npx markserv`)
- Python 3 (for port availability check)
