---
name: markserv
description: Start or stop a local Markdown preview server (markserv). Use this skill when the user wants to preview markdown files in the browser, start markserv, stop markserv, shut down the preview server, or mentions wanting to see rendered markdown locally. Also trigger when the user says things like "open this doc in the browser", "preview this README", or "kill the preview server".
---

# markserv — Markdown Preview Server

Manage a local markserv instance for previewing Markdown files in the browser with live reload.

## How it works

markserv renders Markdown as GitHub-styled HTML with live-reload. This skill handles starting it on an available port and stopping it when done.

## Starting markserv

When the user wants to preview markdown:

1. Determine the target: use the file or directory the user specifies, or fall back to the current working directory
2. Find an available port starting from 8080, incrementing until one is free:
   ```bash
   python3 -c "import socket; s=socket.socket(); s.bind(('',PORT)); s.close()" 2>/dev/null
   ```
3. Start markserv in the background using `npx`:
   ```bash
   npx markserv <target> -p <port> -l <livereload-port> -b false
   ```
   - Use the Bash tool with `run_in_background: true`
   - Set livereload port to `main_port + 27649` (to avoid collisions with other instances)
   - Use `-b false` to suppress auto-opening the browser
4. Record the PID from the background task for later shutdown
5. Tell the user the URL: `http://localhost:<port>`

## Stopping markserv

When the user wants to stop the preview server:

1. Find running markserv processes:
   ```bash
   pgrep -f markserv
   ```
2. If multiple instances are running, list them with their ports and let the user choose which to stop
3. If only one instance, stop it directly:
   ```bash
   kill <pid>
   ```
4. Confirm to the user that the server has been stopped

## Port selection

Always check if a port is available before using it. Start from 8080 and try incrementally (8081, 8082, ...) until a free port is found. This avoids conflicts when the user already has other services running.

```bash
for port in $(seq 8080 8099); do
  if python3 -c "import socket; s=socket.socket(); s.bind(('',${port})); s.close()" 2>/dev/null; then
    echo "$port"
    break
  fi
done
```

## Example interactions

**Starting:**
- "markserv 起動して" → start on cwd, auto-select port
- "この README をプレビューしたい" → start on README.md
- "docs/ を markserv で見たい" → start on docs/
- "別ポートで markserv 立ち上げて" → find next available port

**Stopping:**
- "markserv 止めて" → stop the running instance
- "プレビューサーバー閉じて" → stop markserv
- "markserv シャットダウン" → stop markserv
