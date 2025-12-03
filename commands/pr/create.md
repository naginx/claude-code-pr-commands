---
description: PRテンプレートに基づいてPull Request説明文を生成・更新する際に使用します。ユーザーがpull_request_template.mdに従ってPR説明文を作成したい、既存のPR説明文を更新したい、またはGitHub APIを使ってPRテンプレートを適用したい場合に役立ちます。
arguments:
  - name: pr-url
    description: 既存PRのURL（更新時）またはリポジトリ情報（新規作成時）
    required: false
    default: ""
    argument-hint: "PR URLまたはowner/repo形式で入力。未指定の場合は対話的に取得します。"
  - name: base-branch
    description: ベースブランチ名
    required: false
    default: "develop"
allowed-tools:
  - Bash(gh:*)
  - Bash(git:*)
  - Bash(cat:*)
  - Read
  - Write
  - Glob
---

# PR作成

このコマンドは、PR URLまたはリポジトリ情報（owner/repo/PR番号）を取得し、PRの変更内容を分析してプロジェクトのPRテンプレートに基づいた説明文を生成・更新します。

## ブランチ作成時の命名規則

PR作成前にブランチを作成する場合、チームの命名規則に従ってください。

<!-- チームのブランチ命名規則をここに記載してください -->

## ステップ1：モード判定

引数の有無と既存PRの存在により、**新規作成モード**または**既存PR更新モード**を判定します。

### 判定フロー

```bash
# 1. 現在のブランチを取得
current_branch=$(git branch --show-current)

# 2. 現在のブランチで既存PRがあるか確認
gh pr view "$current_branch" --json number,url 2>/dev/null
```

| 条件 | モード | 動作 |
|------|--------|------|
| 引数なし + PRが存在しない | 新規作成 | 現在のブランチから `develop`（デフォルト）に向けたPRを新規作成 |
| 引数なし + PRが既に存在 | 既存PR更新 | 現在のブランチのPR説明文を更新 |
| PR URLを指定 | 既存PR更新 | 指定されたPRの説明文を更新 |

### 新規作成モードの場合

```bash
# リポジトリ情報を取得
gh repo view --json nameWithOwner --jq '.nameWithOwner'
```

### 既存PR更新モードの場合

以下の形式を受け付けます：
- 引数なし（現在のブランチにPRが存在する場合）
- GitHub PR URL: `https://github.com/{owner}/{repo}/pull/{number}`
- リポジトリ情報: `{owner}/{repo}` + PR番号

## ステップ2：PRテンプレートの確認

リポジトリからPRテンプレートを取得します（GitHub公式の配置場所）：

1. `.github/pull_request_template.md`
2. `.github/PULL_REQUEST_TEMPLATE/`（複数テンプレート用ディレクトリ）

テンプレートが見つからない場合は、以下のメッセージを表示して終了：

```text
⚠️ PRテンプレートが見つかりません。

リポジトリに `.github/pull_request_template.md` を作成してから再度実行してください。
```

## ステップ3：PR変更内容の分析

GitHub APIを使用してPR情報を取得・分析：

```bash
# PR情報の取得
gh pr view {number} --repo {owner}/{repo} --json title,body,state,author,files,commits,additions,deletions,changedFiles

# 変更ファイルの詳細取得
gh pr diff {number} --repo {owner}/{repo}

# コミットメッセージの取得
gh pr view {number} --repo {owner}/{repo} --json commits
```

分析する内容：
- 変更の種類（新機能、バグ修正、リファクタリング）
- 影響範囲（変更ファイル、モジュール）
- テストの追加・修正
- ドキュメントの更新
- 破壊的変更の有無

## ステップ3.5：PR説明文の整合性検証（既存PRの場合）

既存のPR説明文がある場合、**実際のコード変更との整合性を必ず検証**する。

### 検証項目

1. **変更内容セクションとdiffの照合**
   - PR説明文に記載された各変更項目が、実際のdiffに存在するか確認
   - diffに含まれる主要な変更が、PR説明文に漏れなく記載されているか確認

2. **コミット履歴の確認（複数コミットがある場合は特に重要）**
   - 途中のコミットで変更が取り消し・修正されていないか確認
   - 最終的なコード状態とPR説明文の記載が一致しているか確認
   - 例：「AをBに変更」→「BをAに戻す」のような履歴がないか

3. **不整合発見時の対応**
   - PR説明文に記載されているが実際のdiffにない変更 → 削除を提案
   - diffにあるがPR説明文に記載されていない変更 → 追加を提案

### 検証の実行例

```bash
# PR説明文の「変更内容」セクションを抽出
gh pr view {number} --repo {owner}/{repo} --json body --jq '.body' | grep -A20 "### 変更内容"

# 実際のdiffと照合
gh pr diff {number} --repo {owner}/{repo}

# コミット履歴で変更の取り消しがないか確認
gh pr view {number} --repo {owner}/{repo} --json commits --jq '.commits[].messageHeadline'
```

**重要**: 「PR説明文に書かれている」≠「正しい」。必ず実際のコードと照合すること。

## ステップ4：PR説明文の生成

リポジトリのPRテンプレートに従って、分析した変更内容を埋め込みます。

## ステップ5：PR説明文の更新

### 新規PRの場合

**重要**: 新規PRは**必ずドラフト状態**で作成してください。`--draft`フラグは省略不可です。レビュー準備完了後にユーザーが手動でReady for reviewに変更します。

```bash
# PRの作成（必ず --draft を付けること）
gh pr create --draft \
  --title "{title}" \
  --body "{generated_description}" \
  --repo {owner}/{repo} \
  --base {base_branch} \
  --head {head_branch}
```

### 既存PRの更新

`gh pr edit`コマンドはGitHub Projects Classicの廃止に伴うGraphQLエラーで失敗することがあるため、`gh api`を使用します。
参考: [Issue #11983](https://github.com/cli/cli/issues/11983)

```bash
# PR説明文を一時ファイルに保存してから更新
cat > /tmp/pr_body.md << 'EOF'
{generated_description}
EOF

gh api repos/{owner}/{repo}/pulls/{number} -X PATCH -F body=@/tmp/pr_body.md
```

### 更新後の確認

```bash
# 更新が正しく反映されたか確認
gh pr view {number} --repo {owner}/{repo} --json body --jq '.body' | head -30
```

---
