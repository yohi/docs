# Cursor CLI 包括的ガイド - 生成AI対応版

## 目次
- [概要](#概要)
- [インストールとセットアップ](#インストールとセットアップ)
- [基本的な使用方法](#基本的な使用方法)
- [ヘッドレスモード](#ヘッドレスモード)
- [CI/CD統合](#cicd統合)
- [クックブック例](#クックブック例)
- [設定とパラメータ](#設定とパラメータ)
- [認証とセキュリティ](#認証とセキュリティ)
- [パーミッション制御](#パーミッション制御)
- [出力フォーマット](#出力フォーマット)
- [ベストプラクティス](#ベストプラクティス)
- [トラブルシューティング](#トラブルシューティング)

---

## 概要

Cursor CLIは、AIを活用したコード分析、生成、修正を自動化するためのコマンドラインツールです。スクリプト、CI/CDパイプライン、自動化ワークフローでの使用に最適化されています。

### 主要機能
- **コード分析**: 既存コードベースの理解と分析
- **コード生成**: AI支援による新機能実装
- **コード修正**: 自動リファクタリングとバグ修正
- **ドキュメント生成**: 自動ドキュメント作成・更新
- **CI/CD統合**: GitHub Actions等との連携

### 使用ケース
- 自動コードレビュー
- ドキュメント自動更新
- CI障害の自動修正
- セキュリティ監査の自動化
- 多言語対応の自動化

---

## インストールとセットアップ

### 基本インストール
```bash
# Linux/macOS
curl https://cursor.com/install -fsS | bash

# パスを追加
echo "$HOME/.cursor/bin" >> $GITHUB_PATH  # GitHub Actions用
export PATH="$HOME/.cursor/bin:$PATH"      # ローカル環境用
```

### 認証設定
```bash
# API キーの設定
export CURSOR_API_KEY=your_api_key_here

# 設定ファイルでの認証
# ~/.cursor/config.json
{
  "api_key": "your_api_key_here"
}
```

### 動作確認
```bash
# インストール確認
cursor-agent --version

# 基本動作テスト
cursor-agent -p "Hello, world!"
```

---

## 基本的な使用方法

### 基本コマンド構文
```bash
cursor-agent [オプション] "プロンプト"
```

### 主要オプション
```bash
# プリントモード（非対話式）
cursor-agent -p "コードを分析して"

# ファイル変更を強制実行
cursor-agent -p --force "リファクタリングを実行"

# モデル指定
cursor-agent -p --model gpt-4 "高度な分析を実行"

# 出力フォーマット指定
cursor-agent -p --output-format json "構造化された分析結果を出力"
```

### 基本的な使用例
```bash
# コードベース分析
cursor-agent -p "このプロジェクトの構造を分析して、README.mdを生成"

# バグ修正
cursor-agent -p --force "TypeScriptエラーを修正"

# コードレビュー
cursor-agent -p "最近の変更をレビューして、改善点を指摘"
```

---

## ヘッドレスモード

ヘッドレスモードは、スクリプトや自動化での使用に最適化されたモードです。

### プリントモードの使用
```bash
# 読み取り専用分析
cursor-agent -p "コードの問題点を分析"

# ファイル変更を伴う操作
cursor-agent -p --force "ESLintルールに従ってコードを修正"
```

### スクリプトでの活用例

#### 1. コードベース検索
```bash
#!/bin/bash
# simple-analysis.sh

echo "コードベース分析開始..."

cursor-agent -p --output-format text \
  "このプロジェクトの主要な機能と技術スタックを分析して、
   analysis-report.md に詳細なレポートを作成してください"

if [ $? -eq 0 ]; then
  echo "✅ 分析完了"
else
  echo "❌ 分析失敗"
  exit 1
fi
```

#### 2. 自動コードレビュー
```bash
#!/bin/bash
# auto-code-review.sh

echo "自動コードレビュー開始..."

cursor-agent -p --force --output-format text \
  "最近のコード変更をレビューして以下の観点で評価：
  - コード品質と可読性
  - 潜在的なバグや問題
  - セキュリティ考慮事項
  - ベストプラクティス準拠

  具体的な改善提案を review-report.txt に出力してください"

if [ $? -eq 0 ]; then
  echo "✅ コードレビュー完了"
else
  echo "❌ コードレビュー失敗"
  exit 1
fi
```

#### 3. リアルタイム進捗追跡
```bash
#!/bin/bash
# progress-tracking.sh

echo "🚀 ストリーム処理開始..."

accumulated_text=""
tool_count=0
start_time=$(date +%s)

cursor-agent -p --force --output-format stream-json \
  "プロジェクト構造を分析して summary-report.txt にレポートを作成" | \
  while IFS= read -r line; do

    type=$(echo "$line" | jq -r '.type // empty')
    subtype=$(echo "$line" | jq -r '.subtype // empty')

    case "$type" in
      "system")
        if [ "$subtype" = "init" ]; then
          model=$(echo "$line" | jq -r '.model // "unknown"')
          echo "🤖 使用モデル: $model"
        fi
        ;;

      "assistant")
        content=$(echo "$line" | jq -r '.message.content[0].text // empty')
        accumulated_text="$accumulated_text$content"
        printf "\r📝 生成中: %d文字" ${#accumulated_text}
        ;;

      "tool_call")
        if [ "$subtype" = "started" ]; then
          tool_count=$((tool_count + 1))

          if echo "$line" | jq -e '.tool_call.writeToolCall' > /dev/null 2>&1; then
            path=$(echo "$line" | jq -r '.tool_call.writeToolCall.args.path // "unknown"')
            echo -e "\n🔧 ツール #$tool_count: $path を作成中"
          elif echo "$line" | jq -e '.tool_call.readToolCall' > /dev/null 2>&1; then
            path=$(echo "$line" | jq -r '.tool_call.readToolCall.args.path // "unknown"')
            echo -e "\n📖 ツール #$tool_count: $path を読み取り中"
          fi

        elif [ "$subtype" = "completed" ]; then
          if echo "$line" | jq -e '.tool_call.writeToolCall.result.success' > /dev/null 2>&1; then
            lines=$(echo "$line" | jq -r '.tool_call.writeToolCall.result.success.linesCreated // 0')
            size=$(echo "$line" | jq -r '.tool_call.writeToolCall.result.success.fileSize // 0')
            echo "   ✅ $lines行作成 ($sizeバイト)"
          fi
        fi
        ;;

      "result")
        duration=$(echo "$line" | jq -r '.duration_ms // 0')
        end_time=$(date +%s)
        total_time=$((end_time - start_time))

        echo -e "\n\n🎯 ${duration}ms で完了 (合計 ${total_time}秒)"
        echo "📊 統計: $tool_count ツール, ${#accumulated_text} 文字生成"
        ;;
    esac
  done
```

---

## CI/CD統合

### GitHub Actions統合

#### 基本セットアップ
```yaml
- name: Cursor CLI インストール
  run: |
    curl https://cursor.com/install -fsS | bash
    echo "$HOME/.cursor/bin" >> $GITHUB_PATH

- name: Cursor Agent 実行
  env:
    CURSOR_API_KEY: ${{ secrets.CURSOR_API_KEY }}
  run: |
    cursor-agent -p "プロンプト内容" --model gpt-4
```

#### 完全自動化アプローチ
```yaml
- name: 完全自動ドキュメント更新
  env:
    CURSOR_API_KEY: ${{ secrets.CURSOR_API_KEY }}
    GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
  run: |
    cursor-agent -p "Git、GitHub CLI、PR操作への完全アクセス権限があります。
    コミット、プッシュ、PRコメントを含む、
    ドキュメント更新ワークフロー全体を処理してください。"
```

#### 制限付き自動化アプローチ（推奨）
```yaml
- name: ドキュメント更新（制限付き）
  env:
    CURSOR_API_KEY: ${{ secrets.CURSOR_API_KEY }}
  run: |
    cursor-agent -p "重要: ブランチ作成、コミット、プッシュ、PRコメント投稿は行わないでください。
    作業ディレクトリ内のファイル変更のみを行ってください。
    後続のワークフローステップで公開処理を行います。"

- name: ドキュメントブランチ公開（決定論的）
  run: |
    git checkout -B "docs/${{ github.head_ref }}"
    git add -A
    git commit -m "docs: PR用ドキュメント更新"
    git push origin "docs/${{ github.head_ref }}"

- name: PRコメント投稿（決定論的）
  run: |
    gh pr comment ${{ github.event.pull_request.number }} --body "ドキュメント更新完了"
```

### Bitbucket Pipelines統合
```yaml
definitions:
  steps:
    - step: &install-cursor
        name: Cursor CLI インストール
        script:
          - curl https://cursor.com/install -fsS | bash
          - export PATH="$HOME/.cursor/bin:$PATH"
          - cursor-agent --version

pipelines:
  pull-requests:
    '**':
      - step:
          <<: *install-cursor
          script:
            - export CURSOR_API_KEY=$CURSOR_API_KEY
            - cursor-agent -p "PR差分を分析してドキュメントを更新" --force
```

### その他のCI/CDシステム
Cursor CLIは以下の要件を満たすCI/CDシステムで動作します：
- **シェルスクリプト実行** (bash, zsh等)
- **環境変数** APIキー設定用
- **インターネット接続** Cursor API接続用

---

## クックブック例

### 1. ドキュメント自動更新

#### 完全自動化版
```yaml
name: ドキュメント自動更新

on:
  pull_request:
    types: [opened, synchronize, reopened, ready_for_review]

permissions:
  contents: write
  pull-requests: write

jobs:
  auto-docs:
    if: ${{ !startsWith(github.head_ref, 'docs/') }}
    runs-on: ubuntu-latest
    steps:
      - name: リポジトリチェックアウト
        uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Cursor CLI インストール
        run: |
          curl https://cursor.com/install -fsS | bash
          echo "$HOME/.cursor/bin" >> $GITHUB_PATH

      - name: Git設定
        run: |
          git config user.name "Cursor Agent"
          git config user.email "cursoragent@cursor.com"

      - name: ドキュメント更新
        env:
          MODEL: gpt-4
          CURSOR_API_KEY: ${{ secrets.CURSOR_API_KEY }}
          GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          BRANCH_PREFIX: docs
        run: |
          cursor-agent -p "GitHub Actions ランナーで動作しています。

          GitHub CLIは \`gh\` として利用可能で、\`GH_TOKEN\` で認証済みです。
          Gitも利用可能です。リポジトリコンテンツへの書き込みアクセスと
          プルリクエストへのコメント権限がありますが、PRの作成・編集は禁止されています。

          # コンテキスト:
          - リポジトリ: ${{ github.repository }}
          - オーナー: ${{ github.repository_owner }}
          - PR番号: ${{ github.event.pull_request.number }}
          - ベースRef: ${{ github.base_ref }}
          - ヘッドRef: ${{ github.head_ref }}
          - ドキュメントブランチプレフィックス: ${{ env.BRANCH_PREFIX }}

          # 目標:
          - 元のPRの増分変更に基づくエンドツーエンドのドキュメント更新フローを実装

          # 要件:
          1) 元のPRで何が変更されたかを判断し、複数回のプッシュがある場合は、
             最後に成功したドキュメント更新以降の増分差分を計算
          2) これらの増分変更に基づいて関連ドキュメントのみを更新
          3) コンテキストのドキュメントブランチプレフィックスを使用して、
             このPRヘッド用の永続的なドキュメントブランチを維持。
             存在しない場合は作成、存在する場合は更新し、originに変更をプッシュ
          4) PRを作成する権限はありません。代わりに、ドキュメント更新を簡潔に説明し、
             PR作成用のインライン比較リンクを含む自然言語のPRコメント（1-2文）を投稿または更新

          # 入力と規約:
          - \`gh pr diff\` とgit履歴を使用して変更を検出し、
            最後のドキュメント更新以降の増分範囲を導出
          - PRを直接作成・編集しないでください。上記の比較リンク形式を使用
          - 変更は最小限に抑え、リポジトリスタイルと一貫性を保つ。
            ドキュメント更新が不要な場合は、変更を行わずコメントも投稿しない

          # 更新が発生した場合の成果物:
          - このPRヘッド用の永続的なドキュメントブランチへのプッシュされたコミット
          - 上記のインライン比較リンクを含む、元のPRへの自然言語PRコメント。
            重複投稿を避け、以前のボットコメントがある場合は更新する
          " --force --model "$MODEL" --output-format=text
```

### 2. CI障害自動修正
```yaml
name: CI障害自動修正

on:
  workflow_run:
    workflows: ["メインCI"]
    types: [completed]
    branches: [main, develop]

permissions:
  contents: write
  pull-requests: write
  actions: read

jobs:
  auto-fix-ci:
    if: ${{ github.event.workflow_run.conclusion == 'failure' }}
    runs-on: ubuntu-latest
    steps:
      - name: 失敗したワークフロー情報取得
        id: failed-workflow
        run: |
          echo "run_id=${{ github.event.workflow_run.id }}" >> $GITHUB_OUTPUT
          echo "head_sha=${{ github.event.workflow_run.head_sha }}" >> $GITHUB_OUTPUT

      - name: リポジトリチェックアウト
        uses: actions/checkout@v4
        with:
          ref: ${{ steps.failed-workflow.outputs.head_sha }}
          fetch-depth: 0

      - name: Cursor CLI インストール
        run: |
          curl https://cursor.com/install -fsS | bash
          echo "$HOME/.cursor/bin" >> $GITHUB_PATH

      - name: CI障害自動修正
        env:
          MODEL: gpt-4
          CURSOR_API_KEY: ${{ secrets.CURSOR_API_KEY }}
          GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          FAILED_RUN_ID: ${{ steps.failed-workflow.outputs.run_id }}
        run: |
          cursor-agent -p "CI障害の自動修正を行います。

          # 失敗したワークフロー情報:
          - ワークフロー実行ID: $FAILED_RUN_ID
          - SHA: ${{ steps.failed-workflow.outputs.head_sha }}

          # タスク:
          1) GitHub APIを使用して失敗したワークフローのログを取得・分析
          2) 障害の根本原因を特定（ビルドエラー、テスト失敗、リント問題等）
          3) 修正コードを実装
          4) 修正用ブランチを作成・プッシュ
          5) 修正内容を説明するPRコメントを投稿

          修正は保守的で安全なものに限定してください。
          " --force --model "$MODEL" --output-format=text
```

### 3. セキュリティ監査自動化
```bash
#!/bin/bash
# security-audit.sh

echo "🔒 セキュリティ監査開始..."

cursor-agent -p --force "
セキュリティ監査を実行してください：

1. **依存関係チェック**:
   - package.json / requirements.txt / Gemfile の脆弱性スキャン
   - 古いパッケージバージョンの特定

2. **コードスキャン**:
   - ハードコードされた秘密情報（API キー、パスワード等）の検出
   - SQLインジェクション脆弱性の確認
   - XSS脆弱性の確認

3. **設定チェック**:
   - セキュリティヘッダーの確認
   - HTTPSの強制確認
   - CORS設定の確認

4. **レポート生成**:
   - security-audit-report.md に詳細レポートを出力
   - 重要度別の脆弱性リスト
   - 修正推奨事項

高リスクの問題が見つかった場合は、可能な範囲で自動修正も実行してください。
" --model gpt-4

echo "✅ セキュリティ監査完了"
```

### 4. 多言語対応自動化
```bash
#!/bin/bash
# auto-translate.sh

echo "🌍 多言語対応開始..."

cursor-agent -p --force "
多言語対応の自動化を実行してください：

1. **翻訳対象の特定**:
   - src/locales/ または i18n/ ディレクトリの確認
   - 新しい翻訳キーの検出
   - 不足している言語の特定

2. **自動翻訳**:
   - 英語をベースに日本語、中国語、韓国語、スペイン語、フランス語に翻訳
   - コンテキストを考慮した適切な翻訳
   - 技術用語の一貫性維持

3. **ファイル更新**:
   - 各言語ファイル（ja.json, zh.json, ko.json等）の更新
   - JSON構造の一貫性確保
   - 重複キーの確認・修正

4. **品質チェック**:
   - 翻訳の文脈適合性確認
   - 文字数制限の確認（UI制約考慮）
   - プレースホルダー（{name}等）の整合性確認

翻訳には文化的ニュアンスも考慮してください。
" --model gpt-4

echo "✅ 多言語対応完了"
```

---

## 設定とパラメータ

### 基本パラメータ
```bash
# プリントモード（非対話式）
-p, --print

# ファイル変更の強制実行
--force

# 使用するAIモデルの指定
--model <model_name>
# 利用可能: gpt-3.5-turbo, gpt-4, gpt-4-turbo, claude-3-sonnet

# 出力フォーマット
--output-format <format>
# 利用可能: text, json, stream-json

# 作業ディレクトリ
--working-directory <path>

# 設定ファイル指定
--config <path>

# 詳細ログ出力
--verbose

# ヘルプ表示
--help
```

### 高度なパラメータ
```bash
# タイムアウト設定（秒）
--timeout 300

# 最大トークン数
--max-tokens 4000

# 温度設定（創造性レベル）
--temperature 0.7

# リトライ回数
--max-retries 3

# プロキシ設定
--proxy http://proxy.example.com:8080

# ログレベル
--log-level debug
# 利用可能: debug, info, warn, error

# 並列実行数
--concurrency 5
```

### 設定ファイル例
```json
{
  "default_model": "gpt-4",
  "output_format": "text",
  "timeout": 300,
  "max_tokens": 4000,
  "temperature": 0.7,
  "working_directory": "./",
  "permissions": {
    "allow": [
      "Read(**/*.md)",
      "Write(docs/**/*)",
      "Shell(grep)",
      "Shell(find)"
    ],
    "deny": [
      "Shell(git)",
      "Write(.env*)",
      "Write(package.json)"
    ]
  }
}
```

---

## 認証とセキュリティ

### 認証方法

#### 1. 環境変数による認証
```bash
export CURSOR_API_KEY=your_api_key_here
```

#### 2. 設定ファイルによる認証
```json
{
  "api_key": "your_api_key_here"
}
```

#### 3. CI/CD環境での認証
```yaml
# GitHub Actions
env:
  CURSOR_API_KEY: ${{ secrets.CURSOR_API_KEY }}

# Bitbucket Pipelines
script:
  - export CURSOR_API_KEY=$CURSOR_API_KEY
```

### APIキーの取得
Cursor設定画面から取得可能：
1. Cursorアプリケーションを開く
2. 設定 (Settings) に移動
3. API Keys セクションでキーを生成

### セキュリティベストプラクティス

#### 1. APIキーの管理
```bash
# ❌ 悪い例: ハードコーディング
cursor-agent -p "test" --api-key sk-abc123

# ✅ 良い例: 環境変数使用
export CURSOR_API_KEY=sk-abc123
cursor-agent -p "test"
```

#### 2. 権限の最小化
```json
{
  "permissions": {
    "allow": [
      "Read(src/**/*)",
      "Write(docs/**/*)"
    ],
    "deny": [
      "Shell(*)",
      "Write(.env*)",
      "Write(package.json)"
    ]
  }
}
```

#### 3. ログの管理
```bash
# 機密情報を含むログの無効化
cursor-agent -p "sensitive task" --no-log

# ログファイルの制限
cursor-agent -p "task" --log-file /secure/path/cursor.log
```

---

## パーミッション制御

### パーミッション設定

#### 基本的な権限制御
```json
{
  "permissions": {
    "allow": [
      "Read(**/*.md)",           // Markdownファイル読み取り
      "Write(docs/**/*)",        // docsディレクトリ書き込み
      "Shell(grep)",             // grep コマンド実行
      "Shell(find)",             // find コマンド実行
      "Shell(cat)",              // cat コマンド実行
      "Network(https://api.github.com)" // GitHub API アクセス
    ],
    "deny": [
      "Shell(rm)",               // ファイル削除禁止
      "Shell(git)",              // Git操作禁止
      "Write(.env*)",            // 環境ファイル変更禁止
      "Write(package.json)",     // 依存関係変更禁止
      "Write(Dockerfile)",       // Docker設定変更禁止
      "Network(*)",              // 汎用ネットワークアクセス禁止
      "System(*)"                // システム操作禁止
    ]
  }
}
```

#### 高度な権限制御例

##### 開発環境用
```json
{
  "permissions": {
    "allow": [
      "Read(**/*)",
      "Write(src/**/*)",
      "Write(tests/**/*)",
      "Write(docs/**/*)",
      "Shell(npm)",
      "Shell(yarn)",
      "Shell(git status)",
      "Shell(git diff)",
      "Network(https://registry.npmjs.org)"
    ],
    "deny": [
      "Shell(rm -rf)",
      "Shell(sudo)",
      "Write(.env.production)",
      "Write(package-lock.json)",
      "System(shutdown)",
      "System(reboot)"
    ]
  }
}
```

##### 本番環境用
```json
{
  "permissions": {
    "allow": [
      "Read(src/**/*)",
      "Read(docs/**/*)",
      "Write(logs/**/*)",
      "Shell(grep)",
      "Shell(find)",
      "Shell(cat)"
    ],
    "deny": [
      "Write(src/**/*)",
      "Write(config/**/*)",
      "Shell(git)",
      "Shell(npm)",
      "Shell(yarn)",
      "Network(*)",
      "System(*)"
    ]
  }
}
```

##### CI/CD用
```json
{
  "permissions": {
    "allow": [
      "Read(**/*)",
      "Write(dist/**/*)",
      "Write(coverage/**/*)",
      "Write(test-results/**/*)",
      "Shell(npm test)",
      "Shell(npm run build)",
      "Shell(npm run lint)",
      "Network(https://codecov.io)"
    ],
    "deny": [
      "Write(src/**/*)",
      "Write(.env*)",
      "Shell(npm install)",
      "Shell(git push)",
      "System(*)"
    ]
  }
}
```

### 権限検証

#### 権限テスト
```bash
# 権限設定のテスト
cursor-agent -p "権限設定をテストしてください" --config permissions.json --dry-run

# 特定操作の権限確認
cursor-agent -p "package.json を読み取れますか？" --config restrictive-permissions.json
```

#### 権限違反時の動作
```bash
# 権限違反時のログ例
2024-01-15 10:30:15 [ERROR] Permission denied: Write(package.json)
2024-01-15 10:30:15 [INFO] Skipping forbidden operation
2024-01-15 10:30:15 [INFO] Continuing with allowed operations only
```

---

## 出力フォーマット

### 利用可能な出力フォーマット

#### 1. text（デフォルト）
```bash
cursor-agent -p "コードを分析" --output-format text
```
- 人間が読みやすい形式
- ログやレポート生成に最適
- 色付き出力サポート

#### 2. json
```bash
cursor-agent -p "構造化分析" --output-format json
```
```json
{
  "status": "success",
  "timestamp": "2024-01-15T10:30:15Z",
  "model": "gpt-4",
  "response": {
    "content": "分析結果...",
    "tool_calls": [
      {
        "type": "read_file",
        "args": {"path": "src/main.js"},
        "result": "success"
      }
    ]
  },
  "metadata": {
    "duration_ms": 2500,
    "tokens_used": 1250,
    "files_modified": 3
  }
}
```

#### 3. stream-json
```bash
cursor-agent -p "リアルタイム処理" --output-format stream-json
```
リアルタイムでJSONストリームを出力：
```json
{"type": "system", "subtype": "init", "model": "gpt-4", "timestamp": "2024-01-15T10:30:15Z"}
{"type": "assistant", "message": {"content": [{"text": "分析を開始します..."}]}}
{"type": "tool_call", "subtype": "started", "tool_call": {"readToolCall": {"args": {"path": "src/main.js"}}}}
{"type": "tool_call", "subtype": "completed", "tool_call": {"readToolCall": {"result": {"success": {"totalLines": 150}}}}}
{"type": "result", "duration_ms": 2500, "status": "completed"}
```

### フォーマット別使用例

#### textフォーマット - レポート生成
```bash
#!/bin/bash
# report-generator.sh

cursor-agent -p "プロジェクトの詳細分析レポートを生成" \
  --output-format text > project-analysis.txt

echo "📄 レポートが project-analysis.txt に生成されました"
```

#### jsonフォーマット - 自動化処理
```bash
#!/bin/bash
# json-processor.sh

result=$(cursor-agent -p "コード品質を評価" --output-format json)

status=$(echo "$result" | jq -r '.status')
files_modified=$(echo "$result" | jq -r '.metadata.files_modified')
duration=$(echo "$result" | jq -r '.metadata.duration_ms')

echo "処理状況: $status"
echo "変更ファイル数: $files_modified"
echo "処理時間: ${duration}ms"

if [ "$status" = "success" ]; then
  echo "✅ 処理完了"
else
  echo "❌ 処理失敗"
  exit 1
fi
```

#### stream-jsonフォーマット - 進捗監視
```bash
#!/bin/bash
# progress-monitor.sh

echo "🚀 ストリーミング処理開始..."

cursor-agent -p "大量のファイルを処理" --output-format stream-json | \
while IFS= read -r line; do
  type=$(echo "$line" | jq -r '.type // empty')

  case "$type" in
    "system")
      model=$(echo "$line" | jq -r '.model // "unknown"')
      echo "🤖 モデル: $model"
      ;;

    "assistant")
      content=$(echo "$line" | jq -r '.message.content[0].text // empty')
      echo -n "📝 $content"
      ;;

    "tool_call")
      subtype=$(echo "$line" | jq -r '.subtype // empty')
      if [ "$subtype" = "started" ]; then
        echo -e "\n🔧 ツール実行中..."
      elif [ "$subtype" = "completed" ]; then
        echo "   ✅ 完了"
      fi
      ;;

    "result")
      duration=$(echo "$line" | jq -r '.duration_ms // 0')
      echo -e "\n🎯 全体完了: ${duration}ms"
      ;;
  esac
done
```

---

## ベストプラクティス

### 1. プロンプト設計

#### 効果的なプロンプト例
```bash
# ✅ 良い例: 具体的で構造化された指示
cursor-agent -p "
以下の要件でコードレビューを実行してください：

## 対象:
- 最新のPR差分
- 特に src/ ディレクトリの変更

## 評価項目:
1. **コード品質**: 可読性、保守性、パフォーマンス
2. **セキュリティ**: 脆弱性、入力検証、認証
3. **テスト**: カバレッジ、テストケースの妥当性
4. **ドキュメント**: コメント、README更新の必要性

## 出力形式:
- 問題の重要度を High/Medium/Low で分類
- 具体的な修正提案を含む
- review-report.md ファイルに保存

処理後は概要を標準出力に表示してください。
"

# ❌ 悪い例: 曖昧で具体性に欠ける指示
cursor-agent -p "コードを見て問題があれば直して"
```

#### プロンプトテンプレート
```bash
# コードレビューテンプレート
REVIEW_PROMPT="
# コードレビュー要求

## コンテキスト:
- プロジェクト: {{PROJECT_NAME}}
- ブランチ: {{BRANCH_NAME}}
- 変更範囲: {{CHANGED_FILES}}

## 評価基準:
1. **品質**: {{QUALITY_CRITERIA}}
2. **セキュリティ**: {{SECURITY_CRITERIA}}
3. **パフォーマンス**: {{PERFORMANCE_CRITERIA}}

## 成果物:
- 評価レポート: {{OUTPUT_FILE}}
- 重要度分類: High/Medium/Low
- 修正提案: 具体的なコード例を含む
"

# テンプレート使用例
cursor-agent -p "$REVIEW_PROMPT" \
  --output-format json \
  --model gpt-4
```

### 2. エラーハンドリング

#### 堅牢なスクリプト例
```bash
#!/bin/bash
# robust-cursor-script.sh

set -euo pipefail  # エラー時に即座に終了

# 設定
MAX_RETRIES=3
TIMEOUT=300
LOG_FILE="/tmp/cursor-agent.log"

# ログ関数
log() {
  echo "[$(date '+%Y-%m-%d %H:%M:%S')] $*" | tee -a "$LOG_FILE"
}

# リトライ機能付き実行
run_with_retry() {
  local cmd="$1"
  local attempt=1

  while [ $attempt -le $MAX_RETRIES ]; do
    log "試行 $attempt/$MAX_RETRIES: $cmd"

    if timeout $TIMEOUT $cmd; then
      log "✅ 成功: $cmd"
      return 0
    else
      local exit_code=$?
      log "❌ 失敗 (終了コード: $exit_code): $cmd"

      if [ $attempt -eq $MAX_RETRIES ]; then
        log "🚨 最大リトライ回数に到達。処理を中断します。"
        return $exit_code
      fi

      log "⏳ 10秒後にリトライします..."
      sleep 10
      ((attempt++))
    fi
  done
}

# メイン処理
main() {
  log "🚀 Cursor CLI処理開始"

  # 前提条件チェック
  if [ -z "${CURSOR_API_KEY:-}" ]; then
    log "🚨 CURSOR_API_KEY が設定されていません"
    exit 1
  fi

  if ! command -v cursor-agent &> /dev/null; then
    log "🚨 cursor-agent が見つかりません"
    exit 1
  fi

  # 処理実行
  if run_with_retry "cursor-agent -p 'プロジェクト分析を実行' --force --model gpt-4"; then
    log "🎯 全処理完了"
  else
    log "💥 処理失敗"
    exit 1
  fi
}

# エラー時のクリーンアップ
cleanup() {
  log "🧹 クリーンアップ実行"
  # 一時ファイルの削除など
}

# シグナルハンドラー設定
trap cleanup EXIT INT TERM

# メイン実行
main "$@"
```

### 3. パフォーマンス最適化

#### 効率的な処理パターン
```bash
#!/bin/bash
# optimized-processing.sh

# 並列処理での効率化
process_files_parallel() {
  local files=("$@")
  local max_parallel=4
  local pids=()

  for file in "${files[@]}"; do
    # 並列実行数制限
    while [ ${#pids[@]} -ge $max_parallel ]; do
      for i in "${!pids[@]}"; do
        if ! kill -0 "${pids[$i]}" 2>/dev/null; then
          unset "pids[$i]"
        fi
      done
      pids=("${pids[@]}")  # 配列の再インデックス
      sleep 0.1
    done

    # バックグラウンドで処理実行
    {
      cursor-agent -p "ファイル $file を分析・最適化" \
        --output-format json \
        --working-directory "$(dirname "$file")" \
        > "analysis-$(basename "$file").json"
    } &

    pids+=($!)
  done

  # 全処理の完了待機
  for pid in "${pids[@]}"; do
    wait "$pid"
  done
}

# バッチ処理での効率化
process_batch() {
  local batch_size=10
  local files=(src/**/*.js src/**/*.ts)

  for ((i=0; i<${#files[@]}; i+=batch_size)); do
    local batch=("${files[@]:$i:$batch_size}")
    local batch_list=$(printf '%s\n' "${batch[@]}")

    cursor-agent -p "
    以下のファイル群をバッチ処理してください：

    $batch_list

    各ファイルに対して：
    1. ESLint規則適用
    2. TypeScript型チェック
    3. パフォーマンス最適化
    4. セキュリティスキャン

    結果は batch-result-$((i/batch_size + 1)).json に出力
    " --force --model gpt-4
  done
}

# キャッシュ活用
use_cache() {
  local cache_dir=".cursor-cache"
  local cache_key="$1"
  local cache_file="$cache_dir/$cache_key.json"

  mkdir -p "$cache_dir"

  # キャッシュ確認
  if [ -f "$cache_file" ] && [ $(($(date +%s) - $(stat -c %Y "$cache_file"))) -lt 3600 ]; then
    echo "📋 キャッシュから結果を読み込み: $cache_key"
    cat "$cache_file"
    return 0
  fi

  # 新規処理
  cursor-agent -p "$2" --output-format json > "$cache_file"
  cat "$cache_file"
}
```

### 4. 監視とロギング

#### 包括的ログシステム
```bash
#!/bin/bash
# monitoring-system.sh

# ログ設定
LOG_DIR="/var/log/cursor-cli"
DEBUG_LOG="$LOG_DIR/debug.log"
ERROR_LOG="$LOG_DIR/error.log"
METRICS_LOG="$LOG_DIR/metrics.log"

mkdir -p "$LOG_DIR"

# 構造化ログ関数
log_structured() {
  local level="$1"
  local message="$2"
  local context="${3:-}"

  local timestamp=$(date -u +"%Y-%m-%dT%H:%M:%S.%3NZ")
  local log_entry

  if [ -n "$context" ]; then
    log_entry="{\"timestamp\":\"$timestamp\",\"level\":\"$level\",\"message\":\"$message\",\"context\":$context}"
  else
    log_entry="{\"timestamp\":\"$timestamp\",\"level\":\"$level\",\"message\":\"$message\"}"
  fi

  echo "$log_entry" | tee -a "$DEBUG_LOG"

  if [ "$level" = "ERROR" ]; then
    echo "$log_entry" >> "$ERROR_LOG"
  fi
}

# メトリクス収集
collect_metrics() {
  local start_time="$1"
  local end_time="$2"
  local operation="$3"
  local status="$4"
  local files_processed="${5:-0}"
  local tokens_used="${6:-0}"

  local duration=$((end_time - start_time))

  local metrics="{
    \"timestamp\":\"$(date -u +"%Y-%m-%dT%H:%M:%S.%3NZ")\",
    \"operation\":\"$operation\",
    \"duration_seconds\":$duration,
    \"status\":\"$status\",
    \"files_processed\":$files_processed,
    \"tokens_used\":$tokens_used
  }"

  echo "$metrics" >> "$METRICS_LOG"
}

# アラート機能
send_alert() {
  local severity="$1"
  local message="$2"

  log_structured "ALERT" "$message" "{\"severity\":\"$severity\"}"

  case "$severity" in
    "HIGH")
      # Slack通知、メール送信等
      curl -X POST -H 'Content-type: application/json' \
        --data "{\"text\":\"🚨 HIGH: $message\"}" \
        "$SLACK_WEBHOOK_URL" || true
      ;;
    "MEDIUM")
      # ログのみ
      ;;
  esac
}

# 監視付き実行
monitored_execution() {
  local operation="$1"
  local prompt="$2"

  local start_time=$(date +%s)
  log_structured "INFO" "操作開始: $operation"

  local temp_output=$(mktemp)
  local exit_code=0

  if cursor-agent -p "$prompt" --output-format json > "$temp_output" 2>&1; then
    local end_time=$(date +%s)
    local result=$(cat "$temp_output")

    # メトリクス抽出
    local tokens_used=$(echo "$result" | jq -r '.metadata.tokens_used // 0')
    local files_modified=$(echo "$result" | jq -r '.metadata.files_modified // 0')

    collect_metrics "$start_time" "$end_time" "$operation" "success" "$files_modified" "$tokens_used"
    log_structured "INFO" "操作完了: $operation" "{\"duration\":$((end_time - start_time)),\"files_modified\":$files_modified}"

    echo "$result"
  else
    exit_code=$?
    local end_time=$(date +%s)
    local error_output=$(cat "$temp_output")

    collect_metrics "$start_time" "$end_time" "$operation" "failure" 0 0
    log_structured "ERROR" "操作失敗: $operation" "{\"exit_code\":$exit_code,\"error\":\"$error_output\"}"

    # エラーレベルに応じてアラート
    if [[ "$error_output" =~ "Permission denied" ]]; then
      send_alert "HIGH" "権限エラー: $operation"
    elif [[ "$error_output" =~ "API rate limit" ]]; then
      send_alert "MEDIUM" "API制限: $operation"
    fi

    rm -f "$temp_output"
    return $exit_code
  fi

  rm -f "$temp_output"
}

# 使用例
monitored_execution "code_review" "PRの変更をレビューしてください"
```

---

## トラブルシューティング

### よくある問題と解決方法

#### 1. 認証エラー
```bash
# エラー例
Error: Authentication failed. Invalid API key.

# 解決方法
# API キーの確認
echo $CURSOR_API_KEY

# API キーの再設定
export CURSOR_API_KEY=your_correct_api_key

# 設定ファイルの確認
cat ~/.cursor/config.json
```

#### 2. 権限エラー
```bash
# エラー例
Error: Permission denied: Write(src/main.js)

# 解決方法
# 権限設定の確認
cursor-agent -p "権限設定を表示" --config permissions.json

# 権限設定の修正
{
  "permissions": {
    "allow": ["Write(src/**/*)"]
  }
}
```

#### 3. タイムアウトエラー
```bash
# エラー例
Error: Request timeout after 60 seconds

# 解決方法
# タイムアウト時間を延長
cursor-agent -p "大規模処理" --timeout 300

# 処理を分割
cursor-agent -p "小さな単位で処理" --max-tokens 2000
```

#### 4. API制限エラー
```bash
# エラー例
Error: API rate limit exceeded

# 解決方法
# リトライ間隔を設定
cursor-agent -p "処理" --max-retries 5 --retry-delay 60

# 並列処理数を削減
export CURSOR_MAX_CONCURRENT=2
```

#### 5. ファイルアクセスエラー
```bash
# エラー例
Error: File not found: src/main.js

# 解決方法
# 作業ディレクトリを確認
pwd
ls -la src/

# 相対パスの修正
cursor-agent -p "処理" --working-directory /absolute/path/to/project
```

### デバッグ手法

#### 1. 詳細ログの有効化
```bash
# デバッグモード
cursor-agent -p "処理" --verbose --log-level debug

# ログファイルへの出力
cursor-agent -p "処理" --log-file debug.log 2>&1
```

#### 2. ドライランモード
```bash
# 実際の変更なしでテスト
cursor-agent -p "ファイル変更" --dry-run

# 権限チェックのみ
cursor-agent -p "権限テスト" --check-permissions-only
```

#### 3. 段階的なデバッグ
```bash
#!/bin/bash
# debug-script.sh

# ステップ1: 基本動作確認
echo "ステップ1: 基本動作確認"
cursor-agent -p "Hello, world!" --output-format json

# ステップ2: ファイル読み取りテスト
echo "ステップ2: ファイル読み取りテスト"
cursor-agent -p "README.md を読み取って内容を要約" --output-format json

# ステップ3: ファイル書き込みテスト
echo "ステップ3: ファイル書き込みテスト"
cursor-agent -p "test.txt に現在時刻を書き込み" --force --output-format json

# ステップ4: 複合操作テスト
echo "ステップ4: 複合操作テスト"
cursor-agent -p "プロジェクト構造を分析してレポート生成" --force --output-format json
```

### パフォーマンス問題の診断

#### 1. 処理時間の測定
```bash
#!/bin/bash
# performance-test.sh

# 時間測定
time cursor-agent -p "大規模処理" --output-format json

# 詳細なベンチマーク
{
  echo "開始時刻: $(date)"
  start_time=$(date +%s.%N)

  cursor-agent -p "処理内容" --output-format json

  end_time=$(date +%s.%N)
  duration=$(echo "$end_time - $start_time" | bc)

  echo "終了時刻: $(date)"
  echo "処理時間: ${duration}秒"
}
```

#### 2. メモリ使用量の監視
```bash
#!/bin/bash
# memory-monitor.sh

# メモリ使用量を監視しながら実行
{
  cursor-agent -p "大量ファイル処理" --output-format json &
  pid="$!"

  while kill -0 "$pid" 2>/dev/null; do
    ps -p "$pid" -o pid,ppid,rss,vsz,comm
    sleep 5
  done

  wait "$pid"
}
```

#### 3. トークン使用量の最適化
```bash
# トークン使用量を削減する工夫

# ❌ 非効率: 冗長なプロンプト
cursor-agent -p "
このプロジェクトについて詳細に分析してください。
すべてのファイルを見て、すべての問題を見つけて、
すべての改善案を提示して、すべてのドキュメントを更新してください。
"

# ✅ 効率的: 焦点を絞ったプロンプト
cursor-agent -p "
主要なTypeScriptエラーを特定し、修正してください。
対象: src/ ディレクトリの *.ts ファイル
出力: エラー一覧と修正されたファイル
"
```

---

## まとめ

このガイドでは、Cursor CLIの包括的な使用方法を生成AI向けに最適化された形式で説明しました。

### 重要なポイント
1. **セキュリティファースト**: 権限制御と認証の適切な設定
2. **自動化重視**: CI/CDパイプラインでの効果的な活用
3. **エラーハンドリング**: 堅牢なスクリプト設計
4. **パフォーマンス最適化**: 効率的な処理パターンの採用
5. **監視とロギング**: 運用時の可視性確保

### 実装時の注意事項
- API キーは環境変数で管理
- 権限は最小限に制限
- エラー処理とリトライ機能を実装
- ログとメトリクスを適切に収集
- 段階的なテストとデバッグを実施

このガイドを参考に、Cursor CLIを効果的に活用した自動化ソリューションを構築してください。
