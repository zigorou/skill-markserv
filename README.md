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
```

Or just ask naturally:

- "markserv 起動して"
- "この README をプレビューしたい"
- "markserv 止めて"

## Requirements

- Node.js (for `npx markserv`)
- Python 3 (for port availability check)
