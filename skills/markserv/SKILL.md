---
name: markserv
description: Start, stop, or check the status of a local Markdown preview server (markserv). Use this skill when the user wants to preview markdown files in the browser, start markserv, stop markserv, shut down the preview server, check what markserv instances are running, or mentions wanting to see rendered markdown locally. Also trigger when the user says things like "open this doc in the browser", "preview this README", "kill the preview server", or "what port is markserv on".
---

# markserv — Markdown Preview Server

Manage a local markserv instance for previewing Markdown files in the browser with live reload.

## How it works

markserv renders Markdown as GitHub-styled HTML with live-reload. This skill handles starting it on an available port and stopping it when done.

## Starting markserv

When the user wants to preview markdown:

1. Determine the target: use the file or directory the user specifies, or fall back to the current working directory
2. Find an available port starting from 8080, incrementing until one is free on both IPv4 and IPv6 (markserv binds to both):
   ```bash
   for port in $(seq 8080 8099); do
     if python3 -c "
   import socket
   s4=socket.socket(socket.AF_INET,socket.SOCK_STREAM); s4.bind(('127.0.0.1',${port})); s4.close()
   s6=socket.socket(socket.AF_INET6,socket.SOCK_STREAM); s6.bind(('::1',${port})); s6.close()
   " 2>/dev/null; then
       echo "$port"
       break
     fi
   done
   ```
3. Start markserv in the background using `npx`:
   ```bash
   npx markserv <target> -p <port> -l <livereload-port> -b false
   ```
   - Use the Bash tool with `run_in_background: true`
   - Set livereload port to `main_port + 27649` (to avoid collisions with other instances)
   - Use `-b false` to suppress auto-opening the browser
4. Tell the user the URL: `http://localhost:<port>`

## Checking status

When the user runs `/markserv status` or asks what markserv instances are running:

1. Run the same `ps aux` command used for stopping:
   ```bash
   ps aux | grep '[n]px markserv\|[n]pm exec markserv' | grep -v grep
   ```
2. Parse each line to extract:
   - PID
   - Port (`-p <port>`)
   - Root directory / target path (the positional argument before the flags)
3. Present a formatted list, for example:
   ```
   PID    Port   Root
   12345  8080   /Users/you/project
   12678  8081   /Users/you/project/docs
   ```
   Include the URL `http://localhost:<port>` for each entry so the user can click it directly.
4. If no instances are running, say so clearly.

## Stopping markserv

When the user wants to stop the preview server:

1. List running markserv instances with their details using `ps aux`. This gives port and target info that `pgrep` alone does not:
   ```bash
   ps aux | grep '[n]px markserv\|[n]pm exec markserv' | grep -v grep
   ```
2. Parse the output to extract port (`-p <port>`) and target (file/directory) for each instance
3. If **multiple instances** are running, present a list showing port and target for each, and ask the user which to stop (or offer "all")
4. If **one instance**, stop it directly
5. Stop by killing the process tree (the npm/npx parent spawns a child `markserv` process):
   ```bash
   kill <parent-pid>
   ```
6. Confirm to the user that the server has been stopped

## Example interactions

**Starting:**
- "start markserv" → start on cwd, auto-select port
- "I want to preview this README" → start on README.md
- "open docs/ with markserv" → start on docs/
- "launch markserv on a different port" → find next available port

**Status:**
- "markserv status" → list all running instances with port and root dir
- "what port is markserv running on?" → show status
- "is markserv running?" → show status

**Stopping:**
- "stop markserv" → stop the running instance
- "close the preview server" → stop markserv
- "shut down markserv" → stop markserv
- "stop all markserv instances" → stop all instances
