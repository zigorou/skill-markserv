# skill-markserv

A [Claude Code](https://claude.com/claude-code) skill for managing a local [markserv](https://github.com/markserv/markserv) instance — start and stop Markdown preview servers with automatic port selection.

## Features

- Start markserv on a file or directory with auto port selection
- Stop running markserv instances
- Handles multiple concurrent instances

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

## Requirements

- Node.js (for `npx markserv`)
- Python 3 (for port availability check)
