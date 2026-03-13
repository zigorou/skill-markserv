# skill-markserv

ローカルの [markserv](https://github.com/markserv/markserv) インスタンスを管理する [Claude Code](https://claude.com/claude-code) スキルです。自動ポート選択機能付きで Markdown プレビューサーバーの起動・停止を行います。

## 機能

- ファイルまたはディレクトリに対して markserv を起動（ポート自動選択）
- 実行中の markserv インスタンスを停止
- 複数インスタンスの同時管理

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

## 動作要件

- Node.js（`npx markserv` の実行に必要）
- Python 3（ポート疎通確認に使用）
