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
# または
/git:commit --push

# 現在のブランチからPR作成（デフォルトはdevelopブランチへ）
/pr:create

# ベースブランチを指定してPR作成
/pr:create --base-branch main

# 既存PRの説明文を更新（現在のブランチにPRが存在する場合）
/pr:create

# 既存PRの説明文を更新（PR URLを指定）
/pr:create https://github.com/{owner}/{repo}/pull/{number}

# レビュー準備（PRの構造を理解）
/pr:assist-review https://github.com/{owner}/{repo}/pull/{number}

# フィードバック対応（レビューコメントの分析・対応案生成）
/pr:analyze-feedback https://github.com/{owner}/{repo}/pull/{number}
```

## 前提条件

- [GitHub CLI (gh)](https://cli.github.com/) がインストールされていること
- `gh auth login` で認証済みであること

## 各コマンドの詳細

### `/git:commit`

変更を論理的な単位に分割し、適切なコミットメッセージで個別にコミットします。

**機能:**
- 変更を機能単位・変更種別・依存関係で分類
- 各単位ごとに適切なコミットメッセージを生成（feat、fix、refactor、style、chore、docs、test）
- `-p` または `--push` オプションでコミット後にpushも実行可能

**引数:**
- `push`（オプション）: `-p` または `--push` を指定するとコミット後に自動的にpushします

### `/pr:create`

PRテンプレートに基づいてPR説明文を自動生成・更新します。

**機能:**
- リポジトリの `.github/pull_request_template.md` を読み込み
- 変更内容を分析してテンプレートに沿った説明文を生成
- 新規PRは**必ずドラフト状態**で作成（レビュー準備完了後に手動でReady for reviewに変更）
- 既存PRの説明文更新にも対応（PR URL指定または現在のブランチから自動検出）
- 既存PR説明文と実際のコード変更の整合性を検証

**引数:**
- `pr-url`（オプション）: 既存PRのURL。未指定の場合は現在のブランチからPRを検出
- `base-branch`（オプション）: ベースブランチ名（デフォルト: `develop`）

**動作モード:**
- **新規作成モード**: 引数なし + 現在のブランチにPRが存在しない場合
- **既存PR更新モード**: 引数でPR URLを指定、または現在のブランチにPRが既に存在する場合

### `/pr:assist-review`

PRの構造を図式化し、レビュー観点を整理します。

**機能:**
- Mermaid図でコード構造・処理フローを可視化（クラス図、シーケンス図、フローチャート、状態遷移図）
- 修正前後の挙動比較表を作成
- レビュー観点をカテゴリ・優先度別に整理（機能性、コード品質、アーキテクチャ、パフォーマンス、セキュリティ、テスト、ドキュメント、UI/UX）
- 調査結果を `PR_Review_{pr_number}.md` としてワークスペースルートに出力

**引数:**
- `pr-url`（オプション）: レビューしたいPRのURL。未指定の場合は対話的に取得

### `/pr:analyze-feedback`

レビューコメントを分析し、対応アクションプランを生成します。

**機能:**
- レビューコメントをタイプ別（Bug、Improvement、Style、Architecture、Performance、Security、Documentation、Testing、Question、Praise）に分類
- 優先度別（Must Fix、Should Fix、Nice to Have、Info）に分類
- 各コメントに対する対応案を生成
- フェーズ別のアクションプランを作成（Phase 1: 必須対応、Phase 2: 推奨対応、Phase 3: 任意対応）
- レビュワーへの回答案を自動生成
- マージ前チェックリストを提供

**引数:**
- `pr-url`（オプション）: 分析したいPRのURL。未指定の場合は対話的に取得

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
