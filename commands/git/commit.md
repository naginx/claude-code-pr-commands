---
description: 変更を適切な単位毎にコミットします。-p オプションでpushも実行します。
arguments:
  - name: push
    description: コミット後にpushを実行するかどうか
    required: false
    default: false
    argument-hint: "-p または --push を指定するとコミット後に自動的にpushします"
allowed-tools:
  - Bash
  - Read
  - Grep
---

# コミットコマンド

変更を論理的な単位に分割し、適切なコミットメッセージで個別にコミットします。

## 引数の解析

引数: $ARGUMENTS

- `-p` または `--push`: コミット後にpushを実行

## 実行手順

### 1. 現在の変更を確認

以下のコマンドを並列実行して状態を把握：
- `git status` - 変更ファイル一覧
- `git diff` - ステージングされていない変更
- `git diff --cached` - ステージング済みの変更
- `git log --oneline -5` - 直近のコミットスタイル確認

### 2. 変更の分類

変更を以下の観点で論理的な単位に分類：

| 分類基準 | 例 |
|----------|-----|
| 機能単位 | 同じ機能に関連するファイル群 |
| 変更種別 | fix / feat / refactor / style / chore / docs |
| 依存関係 | 他の変更に依存する/しない |

### 3. コミットの実行

各単位ごとに以下を実行：

1. 関連ファイルをステージング: `git add <files>`
2. コミットメッセージを作成（以下の形式）:
   ```
   <type>: <簡潔な説明>

   [必要に応じて詳細説明]

   🤖 Generated with [Claude Code](https://claude.com/claude-code)

   Co-Authored-By: Claude <noreply@anthropic.com>
   ```

#### コミットタイプ

| タイプ | 用途 |
|--------|------|
| feat | 新機能追加 |
| fix | バグ修正 |
| refactor | リファクタリング（機能変更なし） |
| style | コードスタイル変更（フォーマット等） |
| chore | ビルド、設定、依存関係等 |
| docs | ドキュメントのみの変更 |
| test | テストの追加・修正 |

### 4. Push（オプション）

引数に `-p` または `--push` が含まれる場合：
- `git push` を実行

### 5. 結果報告

コミット完了後、以下を報告：
- 作成したコミット一覧（ハッシュとメッセージ）
- pushした場合はその結果

## 注意事項

- 変更がない場合は何もせず終了
- 機密情報（.env、credentials等）が含まれていないか確認
- コミット前に内容を確認し、不要なファイルは除外
