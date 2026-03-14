# skill-markserv

ローカルの [markserv](https://github.com/markserv/markserv) インスタンスを管理する [Claude Code](https://claude.com/claude-code) スキルです。自動ポート選択機能付きで Markdown プレビューサーバーの起動・停止を行います。

## 機能

- ファイルまたはディレクトリに対して markserv を起動（ポート自動選択）
- 実行中の markserv インスタンスを停止
- 複数インスタンスの同時管理
- セッション終了時の自動停止（`SessionEnd` フックでオプトイン）

## インストール

```bash
claude plugin install zigorou/skill-markserv
```

または `zigorou-marketplace` 経由:

```bash
claude plugin install skill-markserv@zigorou-marketplace
```

## 使い方

```
/markserv              # カレントディレクトリでプレビュー開始
/markserv stop         # 実行中のインスタンスを停止
/markserv status       # 実行中インスタンスの一覧（ポート・root dir・URL）
```

自然な言葉でも操作できます:

- "markserv 起動して"
- "この README をプレビューしたい"
- "markserv 止めて"

## セッション終了時の自動停止

Claude Code のセッション終了時に markserv を自動停止するには、`~/.claude/settings.json` に `SessionStart` と `SessionEnd` の両フックを追加します。セッション固有の PID ファイルを使うため、**他のセッションで起動した markserv には影響しません**。

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

仕組み：

- **`SessionStart`**: session_id を `$CLAUDE_ENV_FILE` 経由で `CLAUDE_SESSION_ID` 環境変数として設定し、セッション内のすべての Bash ツール実行から参照できるようにします。
- **`SessionEnd`**: stdin から session_id を読み取り、`/tmp/markserv-cc-<session_id>.pid` に記録された PID を kill してファイルを削除します。**このセッションで起動したプロセスのみ**が対象です。

> **補足:** `SessionEnd` フックはセッションが完全に閉じる前に実行されるため、`kill` は確実に動作します。デフォルトのタイムアウトは 1.5 秒ですが、`kill` は即座に完了するため問題ありません。

## 動作要件

- Node.js（`npx markserv` の実行に必要）
- Python 3（ポート疎通確認に使用）
