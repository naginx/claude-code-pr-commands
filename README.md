# Claude Code PR Commands

PRレビュー業務を効率化するClaude Code / Cursorスラッシュコマンド集です。

## コマンド一覧

| コマンド | 目的 | 実行タイミング |
|---------|------|----------------|
| `/git:commit` | 論理単位でのコミット作成 | 実装完了時 |
| `/pr:create` | PR説明文の生成・更新 | PR作成時・更新時 |
| `/pr:assist-review` | PRの構造理解支援 | レビュー開始前 |
| `/pr:analyze-feedback` | フィードバック分析・対応案生成 | レビューコメント対応時 |

## インストール

### Claude Code

`~/.claude/commands/` ディレクトリにコマンドファイルを配置します。

```bash
# リポジトリをクローン
git clone https://github.com/naginx/claude-code-pr-commands.git

# コマンドをコピー
cp -r claude-code-pr-commands/commands/* ~/.claude/commands/
```

### Cursor

Cursorの「基本設定 > Cursor Settings > Rules and Commands」から「Import Claude Commands」を有効にすることで、`~/.claude/commands` に配置したコマンドをCursorからも利用できます。

## 使い方

```bash
# 変更を論理単位でコミット
/git:commit

# コミット後にpushも実行
/git:commit -p

# 現在のブランチからPR作成
/pr:create

# 既存PRの説明文を更新
/pr:create https://github.com/owner/repo/pull/123

# レビュー準備（PRの構造を理解）
/pr:assist-review https://github.com/owner/repo/pull/123

# フィードバック対応（レビューコメントの分析・対応案生成）
/pr:analyze-feedback https://github.com/owner/repo/pull/123
```

## 前提条件

- [GitHub CLI (gh)](https://cli.github.com/) がインストールされていること
- `gh auth login` で認証済みであること

## 各コマンドの詳細

### `/git:commit`

変更を論理的な単位に分割し、適切なコミットメッセージで個別にコミットします。

**機能:**
- 変更を機能単位・変更種別・依存関係で分類
- 各単位ごとに適切なコミットメッセージを生成
- `-p` オプションでpushも実行可能

### `/pr:create`

PRテンプレートに基づいてPR説明文を自動生成します。

**機能:**
- リポジトリの `.github/pull_request_template.md` を読み込み
- 変更内容を分析してテンプレートに沿った説明文を生成
- 新規PRはドラフトとして作成
- 既存PRの説明文更新にも対応

### `/pr:assist-review`

PRの構造を図式化し、レビュー観点を整理します。

**機能:**
- Mermaid図でコード構造・処理フローを可視化
- 修正前後の挙動比較表を作成
- レビュー観点をカテゴリ・優先度別に整理
- 調査結果を `PR_Review_{pr_number}.md` として出力

### `/pr:analyze-feedback`

レビューコメントを分析し、対応アクションプランを生成します。

**機能:**
- レビューコメントをタイプ別・優先度別に分類
- 各コメントに対する対応案を生成
- フェーズ別のアクションプランを作成
- レビュワーへの回答案を自動生成

## ディレクトリ構造

```
commands/
├── git/
│   └── commit.md      # /git:commit
└── pr/
    ├── create.md          # /pr:create
    ├── assist-review.md   # /pr:assist-review
    └── analyze-feedback.md # /pr:analyze-feedback
```

## ライセンス

MIT License

Copyright (c) 2025 naginx
