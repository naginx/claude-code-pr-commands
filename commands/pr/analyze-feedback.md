---
description: Pull Requestのレビューコメントとフィードバックを分析する際に使用します。PRフィードバックの分析、レビューコメントの分類、フィードバックへの対応アクションプランの作成、PRレビュー対応の整理を行いたい場合に役立ちます。PRフィードバック、レビューコメント、PR対応計画に関する要求でトリガーされます。
arguments:
  - name: pr-url
    description: "Pull RequestのURL（例: https://github.com/owner/repo/pull/123）"
    required: false
    default: ""
    argument-hint: "PR URLを入力してください。未指定の場合は対話的に取得します。"
allowed-tools:
  - Bash
  - Read
  - Write
---

# PR Feedback分析

Pull Requestのレビューコメントを分析し、タイプと優先度別に分類して、対応アクションプランをMarkdownドキュメントとして出力する。

## ワークフロー

### ステップ1: PR URLの取得

ユーザーからPR URLを取得し、owner、repo、PR番号を抽出する。

URLが提供されていない場合：

```
分析したいPull RequestのURLを入力してください。
形式: https://github.com/{owner}/{repo}/pull/{number}
```

### ステップ2: GitHub APIでPRデータを取得

```bash
# PR基本情報
gh pr view {number} --repo {owner}/{repo} --json title,body,state,author,reviews,comments

# レビューコメント
gh api repos/{owner}/{repo}/pulls/{number}/reviews

# Issue/Discussionコメント
gh api repos/{owner}/{repo}/issues/{number}/comments

# ファイル単位のレビューコメント
gh api repos/{owner}/{repo}/pulls/{number}/comments
```

### ステップ3: フィードバックの分類

#### タイプ別

| タイプ | 説明 |
|--------|------|
| Bug | 実装のバグや論理エラー |
| Improvement | コードの改善や最適化の提案 |
| Style | コードスタイルやフォーマット |
| Architecture | 設計やアーキテクチャの懸念 |
| Performance | パフォーマンスの懸念や最適化 |
| Security | セキュリティの懸念事項 |
| Documentation | ドキュメントやコメントの不足 |
| Testing | テストの不足や改善 |
| Question | 説明や確認が必要な質問 |
| Praise | 良い実装への称賛 |

#### 優先度別

| 優先度 | 説明 | 対象 |
|--------|------|------|
| Must Fix | マージ前に必須 | バグ、セキュリティ、重大な設計問題 |
| Should Fix | 対応を強く推奨 | パフォーマンス問題、重要な改善提案 |
| Nice to Have | 対応は任意 | スタイル、軽微な改善、ドキュメント |
| Info | 対応不要 | 質問への回答、称賛 |

### ステップ4: アクションプランの生成

以下の形式でレポートを生成する：

```markdown
# PR Feedback Analysis Report

## サマリー

| 項目 | 値 |
|------|-----|
| タイトル | {title} |
| 作成者 | {author} |
| 状態 | {state} |
| URL | {url} |
| 総コメント数 | {total_comments} |
| 必須対応 | {must_fix_count} |
| 推奨対応 | {should_fix_count} |

## フィードバック一覧

### Must Fix

| # | タイプ | レビュワー | ファイル | 内容 | 対応案 |
|---|--------|-----------|----------|------|--------|
| 1 | Bug | @{reviewer} | {file_path}:{line} | {comment} | {action} |

### Should Fix

| # | タイプ | レビュワー | ファイル | 内容 | 対応案 |
|---|--------|-----------|----------|------|--------|
| 1 | Improvement | @{reviewer} | {file_path} | {comment} | {action} |

### Nice to Have

| # | タイプ | レビュワー | 内容 | 対応案 |
|---|--------|-----------|------|--------|
| 1 | Style | @{reviewer} | {comment} | {action} |

## アクションプラン

### Phase 1: 必須対応
- [ ] {task1}
- [ ] {task2}

### Phase 2: 推奨対応
- [ ] {task3}
- [ ] {task4}

### Phase 3: 任意対応
- [ ] {task5}

## レビュワーへの回答案

### @{reviewer1}
> ご指摘いただいた{対応内容}を修正しました。コミット: {hash}

### @{reviewer2}
> ご指摘の点について、{理由}のため現状維持としました。

## 対応進捗

| 優先度 | 総数 | 対応済 | 残り |
|--------|------|--------|------|
| Must Fix | {n} | {done} | {remain} |
| Should Fix | {n} | {done} | {remain} |
| Nice to Have | {n} | {done} | {remain} |

## マージ前チェックリスト

- [ ] 必須項目に対応済み
- [ ] CI/CDがグリーン
- [ ] レビュワーの承認を取得
- [ ] コンフリクトを解決済み
```

### ステップ5: GitHub上でのコメント対応（任意）

```bash
# コメントに返信
gh api repos/{owner}/{repo}/pulls/{number}/comments/{comment_id}/replies \
  --method POST \
  --field body="{reply_text}"

# PRレビューに返信
gh pr review {number} --repo {owner}/{repo} --comment --body "{response}"
```

## エラーハンドリング

以下のエラーケースを想定し、適切に対処してください：

### GitHub API認証エラー

**症状**: `gh: authentication required` エラーが発生

**対処法**:
```bash
# 認証状態を確認
gh auth status

# 認証をリフレッシュ
gh auth refresh

# 必要に応じて再ログイン
gh auth login
```

### GitHub APIレート制限

**症状**: `API rate limit exceeded` エラーが発生

**対処法**:
```bash
# レート制限の状態を確認
gh api rate_limit

# レート制限に達した場合は、ユーザーに待機時間を伝える
# 通常は1時間以内にリセットされる
```

### PRが見つからない

**症状**: `pull request not found` エラーが発生

**対処法**:
- PR番号が正しいか確認
- リポジトリ名（owner/repo）が正しいか確認
- プライベートリポジトリの場合は適切な権限があるか確認

### レビューコメントが取得できない

**症状**: レビューコメントが空、または取得できない

**対処法**:
- PRにレビューコメントが存在するか確認
- レビューがまだ実施されていない場合は、ユーザーに伝える
- Issueコメントとレビューコメントを区別して取得する

### コメントの分類が困難

**症状**: コメントのタイプや優先度が判断できない

**対処法**:
- コメントの文脈を確認
- レビュワーの意図を推測し、ユーザーに確認を求める
- 不明確な場合は「Question」タイプとして分類
