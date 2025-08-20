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

> **⚠️ 重要な制限事項**
> - **現在ベータ版**: Cursor CLI は現在ベータ版として提供されています
> - **Enterprise制限**: Enterprise プランでは現在利用できません
> - **フィードバック募集中**: 機能改善のためのフィードバックを積極的に収集中
>
> 参考: [Cursor CLI Overview](https://docs.cursor.com/en/cli/overview)

### 主要機能
- **コード分析**: 既存コードベースの理解と分析
- **コード生成**: AI支援による新機能実装
- **コード修正**: 自動リファクタリングとバグ修正
- **ドキュメント生成**: 自動ドキュメント作成・更新
- **CI/CD統合**: GitHub Actions等との連携
- **対話式エージェント**: ターミナル内でのAI対話
- **MCP統合**: Model Context Protocol による拡張

### 使用ケース
- 自動コードレビュー
- ドキュメント自動更新
- CI障害の自動修正
- セキュリティ監査の自動化
- 多言語対応の自動化
- リアルタイムコード修正
- 規模の大きなリファクタリング

---

## インストールとセットアップ

### 基本インストール

#### macOS, Linux, Windows (WSL)
```bash
# 一括インストール
curl https://cursor.com/install -fsS | bash
```

参考: [Installation Guide](https://docs.cursor.com/en/cli/installation)

#### インストール後の検証
```bash
# バージョン確認
cursor-agent --version
```

### PATH設定（重要）

インストール後、以下のPATH設定が必要です：

#### Bash環境
```bash
echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
```

#### Zsh環境
```bash
echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.zshrc
source ~/.zshrc
```

#### GitHub Actions環境
```bash
echo "$HOME/.cursor/bin" >> $GITHUB_PATH
```

### 自動更新システム

Cursor CLI はデフォルトで自動更新が有効になっています：

```bash
# 手動更新（どちらでも可）
cursor-agent update
cursor-agent upgrade

# 自動更新は常時有効（最新版を自動取得）
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

# 対話式モード開始
cursor-agent
```

---

## 基本的な使用方法

### 基本コマンド構文
```bash
cursor-agent [オプション] "プロンプト"
```

### 対話式セッション内のスラッシュコマンド

対話式モードでは、以下のスラッシュコマンドが利用可能です：

| コマンド              | 説明                         | 使用例                         |
| --------------------- | ---------------------------- | ------------------------------ |
| `/model <model>`      | AIモデルの設定・一覧表示     | `/model gpt-4`                 |
| `/auto-run [state]`   | 自動実行の切り替え           | `/auto-run off`                |
| `/new-chat`           | 新しいチャットセッション開始 | `/new-chat`                    |
| `/vim`                | Vimキーバインドの切り替え    | `/vim`                         |
| `/help [command]`     | ヘルプ表示                   | `/help model`                  |
| `/feedback <message>` | フィードバック送信           | `/feedback 素晴らしい機能です` |
| `/resume <chat>`      | 指定チャットの再開           | `/resume my-project`           |
| `/copy-req-id`        | 最後のリクエストIDをコピー   | `/copy-req-id`                 |
| `/logout`             | サインアウト                 | `/logout`                      |
| `/quit`               | セッション終了               | `/quit`                        |

#### スラッシュコマンド使用例
```bash
# 対話式セッション開始
cursor-agent

# セッション内でモデル変更
/model claude-3-sonnet

# 自動実行を無効化
/auto-run off

# 新しいチャットを開始
/new-chat

# 前のチャットに戻る
/resume project-analysis
```

### カスタムルールとコンテキスト設定

Cursor CLI では、IDEと同じルールシステムをサポートしており、エージェントの動作をカスタマイズできます。

#### Rules システム

Cursor CLI は以下のルールファイルを自動的に読み込み、適用します：

```bash
# ルールディレクトリ
.cursor/rules/
├── javascript.md      # JavaScript固有のルール
├── python.md         # Python固有のルール
├── general.md        # 全般的なルール
└── security.md       # セキュリティルール

# プロジェクトルートファイル
AGENT.md              # エージェント専用ルール
CLAUDE.md             # Claude AI専用ルール
```

#### ルールファイル例

**`.cursor/rules/javascript.md`**
```markdown
# JavaScript開発ルール

## コーディング規約
- ES6+ の構文を優先して使用
- async/await を Promise.then() より優先
- const/let を使用し、var は禁止

## ベストプラクティス
- 関数は純粋関数として設計
- エラーハンドリングを必ず実装
- TypeScript型注釈を積極的に活用

## 禁止事項
- console.log は本番コードに残さない
- eval() の使用禁止
- グローバル変数の濫用禁止
```

**`AGENT.md`**
```markdown
# プロジェクト固有のエージェントルール

## このプロジェクトについて
このプロジェクトは React + TypeScript + Express.js のフルスタックアプリケーションです。

## 開発方針
- テスト駆動開発（TDD）を実践
- コンポーネントは小さく、再利用可能に設計
- APIは RESTful な設計に従う

## ファイル構造
```
src/
├── components/       # React コンポーネント
├── hooks/           # カスタムフック
├── services/        # API サービス
├── utils/           # ユーティリティ関数
└── types/           # TypeScript 型定義
```

## 注意事項
- 依存関係の追加時は必ずチームに相談
- セキュリティに関わる変更は慎重に実施
```

#### ルール適用の確認

```bash
# ルールが適用されているかの確認
cursor-agent -p "現在適用されているルールを教えてください"

# 特定のファイルタイプに対するルールの確認
cursor-agent -p "JavaScript ファイルに適用されるルールは何ですか？"
```

#### MCP統合によるルール拡張

MCP設定と組み合わせることで、より高度なルール適用が可能：

```json
{
  "mcpServers": {
    "linting-rules": {
      "command": "node",
      "args": ["linting-mcp-server.js"],
      "env": {
        "RULES_DIR": ".cursor/rules"
      }
    }
  }
}
```

参考: [Using Agent in CLI - Rules](https://docs.cursor.com/en/cli/using#rules)

### 対話式モードの操作

対話式セッションでは、以下のキーバインドと機能が利用できます：

#### ナビゲーション操作
```bash
Arrow-Up/Arrow-Down    # メッセージ履歴の閲覧・選択
Arrow-Left/Arrow-Right # ファイル間の切り替え（レビュー時）
```

#### レビューと編集操作
```bash
Ctrl+R                 # 変更内容のレビュー画面表示
I                      # フォローアップ指示の追加
```

#### コンテキスト管理
```bash
@                      # ファイル・フォルダーのコンテキスト選択
/compress              # コンテキストウィンドウの圧縮（要約化）
```

#### 対話式セッション例
```bash
# セッション開始
cursor-agent

# プロンプト入力
> "src/components/ の React コンポーネントをリファクタリング"

# 変更提案後のレビュー
[Ctrl+R] # レビュー画面表示
[Arrow-Left/Right] # ファイル切り替え
[I] # 追加指示: "TypeScript の型安全性も向上させて"

# コンテキスト追加
> "@src/utils/" # utils ディレクトリをコンテキストに追加

# コンテキスト圧縮
> "/compress" # 長いコンテキストを要約
```

#### セッション管理機能
```bash
# 履歴操作
Arrow-Up               # 前の入力を呼び出し
Arrow-Down             # 次の入力を呼び出し

# チャット管理
/new-chat              # 新しいセッション開始
/resume project-name   # 名前付きセッション再開
```

#### プロンプト最適化のコツ

**明確な意図表示**
```bash
# ✅ 良い例
> "do not write any code - 計画段階なのでファイル編集は行わない"
> "セキュリティ観点でコードレビューを実施し、報告書を作成"

# ❌ 悪い例
> "コードを見て"
> "何か問題があれば直して"
```

**段階的アプローチ**
```bash
# 1段階目: 分析
> "do not write any code - まずプロジェクト構造を分析"

# 2段階目: 計画
> "上記分析に基づいて改善計画を立案"

# 3段階目: 実装
> "計画に従ってコードを修正・実装"
```

参考: [Using Agent in CLI](https://docs.cursor.com/en/cli/using)

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

### コマンド承認システム

**安全性確保のため、Cursor CLI はターミナルコマンド実行前に承認を求めます。**

#### 対話式モードでの承認
```bash
# 例：エージェントがgitコマンド実行を提案
cursor-agent

> "gitの状態を確認して、適切なブランチにいるか教えて"

# エージェントの提案
Agent: git status コマンドを実行してリポジトリの状態を確認します。

# 承認プロンプト表示
Execute command: git status
[Y/N]: Y  # Y で承認、N で拒否

# 承認後にコマンド実行
On branch main
Your branch is up to date with 'origin/main'.
nothing to commit, working tree clean
```

#### 承認対象コマンド

以下のようなシステム操作が承認対象となります：

```bash
# Git操作
git add, git commit, git push, git pull
git checkout, git merge, git rebase

# ファイルシステム操作
rm, mv, cp, mkdir, rmdir
chmod, chown

# パッケージ管理
npm install, npm uninstall
pip install, pip uninstall
apt install, yum install

# ネットワーク操作
curl, wget, ssh, scp

# システム操作
sudo, systemctl, service
```

#### 承認回避（非対話モード）

非対話モードでは自動承認されますが、以下の制限があります：

```bash
# --force フラグで自動承認
cursor-agent -p --force "依存関係を更新してビルド実行"

# 権限制限による安全性確保
cursor-agent -p --config restricted.json "安全な操作のみ実行"
```

#### 承認ログの確認

```bash
# 実行されたコマンドの履歴確認
cursor-agent -p "最近実行したコマンドを教えて"

# 承認・拒否の履歴
ls ~/.cursor/logs/command-approvals.log
```

参考: [Using Agent in CLI - Command Approval](https://docs.cursor.com/en/cli/using#command-approval)

---

## ヘッドレスモード

ヘッドレスモードは、スクリプトや自動化での使用に最適化されたモードです。`--print`（`-p`）フラグを使用して非対話式での実行が可能です。

参考: [Using Headless CLI](https://docs.cursor.com/en/cli/headless)

### プリントモードの動作

```bash
# 読み取り専用分析
cursor-agent -p "コードの問題点を分析"

# ファイル変更を伴う操作（--force必須）
cursor-agent -p --force "ESLintルールに従ってコードを修正"
```

### ファイル変更における重要な違い

#### --force フラグの必要性
```bash
# ❌ ファイル変更されない（提案のみ）
cursor-agent -p "JSDoc コメントを追加"

# ✅ 実際にファイル変更が実行される
cursor-agent -p --force "JSDoc コメントを追加"

# バッチ処理での実際のファイル変更
find src/ -name "*.js" | while read file; do
  cursor-agent -p --force "Add comprehensive JSDoc comments to $file"
done
```

#### 非対話モードの権限

**重要**: 非対話モード（`--print`）では、Cursor が**完全な書き込みアクセス権**を持ちます：

```bash
# 対話モード：各操作で承認が必要
cursor-agent "ファイルを修正"
# → Y/N の承認プロンプト表示

# 非対話モード：自動実行（承認不要）
cursor-agent -p --force "ファイルを修正"
# → 即座に実行、承認プロンプトなし
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

#### 3. リアルタイム進捗追跡（高度版）

**公式例ベース**: [Headless CLI - Real-time Progress](https://docs.cursor.com/en/cli/headless#real-time-progress-tracking)

```bash
#!/bin/bash
# stream-progress.sh - 高度なリアルタイム進捗追跡

echo "🚀 Starting stream processing..."

# 進捗状態管理変数
accumulated_text=""
tool_count=0
start_time=$(date +%s)

cursor-agent -p --force --output-format stream-json \
  "Analyze this project structure and create a summary report in analysis.txt" | \
  while IFS= read -r line; do

    type=$(echo "$line" | jq -r '.type // empty')
    subtype=$(echo "$line" | jq -r '.subtype // empty')

    case "$type" in
      "system")
        if [ "$subtype" = "init" ]; then
          model=$(echo "$line" | jq -r '.model // "unknown"')
          echo "🤖 Using model: $model"
        fi
        ;;

      "assistant")
        # ストリーミングテキストデルタの蓄積
        content=$(echo "$line" | jq -r '.message.content[0].text // empty')
        accumulated_text="$accumulated_text$content"

        # ライブ進捗表示
        printf "\r📝 Generating: %d chars" ${#accumulated_text}
        ;;

      "tool_call")
        if [ "$subtype" = "started" ]; then
          tool_count=$((tool_count + 1))

          # ツール情報の抽出
          if echo "$line" | jq -e '.tool_call.writeToolCall' > /dev/null 2>&1; then
            path=$(echo "$line" | jq -r '.tool_call.writeToolCall.args.path // "unknown"')
            echo -e "\n🔧 Tool #$tool_count: Creating $path"
          elif echo "$line" | jq -e '.tool_call.readToolCall' > /dev/null 2>&1; then
            path=$(echo "$line" | jq -r '.tool_call.readToolCall.args.path // "unknown"')
            echo -e "\n📖 Tool #$tool_count: Reading $path"
          fi

        elif [ "$subtype" = "completed" ]; then
          # ツール結果の抽出と表示
          if echo "$line" | jq -e '.tool_call.writeToolCall.result.success' > /dev/null 2>&1; then
            lines=$(echo "$line" | jq -r '.tool_call.writeToolCall.result.success.linesCreated // 0')
            size=$(echo "$line" | jq -r '.tool_call.writeToolCall.result.success.fileSize // 0')
            echo "   ✅ Created $lines lines ($size bytes)"
          elif echo "$line" | jq -e '.tool_call.readToolCall.result.success' > /dev/null 2>&1; then
            lines=$(echo "$line" | jq -r '.tool_call.readToolCall.result.success.totalLines // 0')
            echo "   ✅ Read $lines lines"
          fi
        fi
        ;;

      "result")
        duration=$(echo "$line" | jq -r '.duration_ms // 0')
        end_time=$(date +%s)
        total_time=$((end_time - start_time))

        echo -e "\n\n🎯 Completed in ${duration}ms (${total_time}s total)"
        echo "📊 Final stats: $tool_count tools, ${#accumulated_text} chars generated"
        ;;
    esac
  done
```

#### 4. 出力フォーマット別の最適化例

**コードベース検索** - `text`フォーマット
```bash
#!/bin/bash
cursor-agent -p --output-format text "What does this codebase do?"
```

**構造化分析** - `json`フォーマット
```bash
#!/bin/bash
result=$(cursor-agent -p --output-format json "Code quality assessment")
status=$(echo "$result" | jq -r '.status')
files_modified=$(echo "$result" | jq -r '.metadata.files_modified')
echo "Status: $status, Files: $files_modified"
```

**バッチ処理** - `--force`での実ファイル変更
```bash
#!/bin/bash
find src/ -name "*.js" | while read file; do
  cursor-agent -p --force "Add comprehensive JSDoc comments to $file"
done
```

---

## CI/CD統合

### GitHub Actions統合

参考: [GitHub Actions Integration](https://docs.cursor.com/en/cli/github-actions)

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

### 自律性レベルの選択

**重要**: エージェントの自律性レベルを選択できます。

#### Full Autonomy Approach（完全自律）
エージェントにgit操作、API呼び出し、外部連携の完全制御を委譲。シンプルだが、より多くの信頼が必要。

```yaml
- name: Update docs (full autonomy)
  env:
    CURSOR_API_KEY: ${{ secrets.CURSOR_API_KEY }}
    GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
  run: |
    cursor-agent -p "You have full access to git, GitHub CLI, and PR operations.
    Handle the entire docs update workflow including commits, pushes, and PR comments."
```

**Full Autonomy の特徴:**
- ✅ シンプルなセットアップ
- ✅ エンドツーエンドの自動化
- ⚠️ 予測不可能な動作の可能性
- ⚠️ 監査証跡の複雑性

#### Restricted Autonomy Approach（制限付き自律・推奨）

**本番CI/CDワークフローでは、権限ベース制限付きのこのアプローチを推奨。**複雑な分析・ファイル変更は任せつつ、重要操作は決定論的に処理。

```yaml
- name: Generate docs updates (restricted)
  env:
    CURSOR_API_KEY: ${{ secrets.CURSOR_API_KEY }}
  run: |
    cursor-agent -p "IMPORTANT: Do NOT create branches, commit, push, or post PR comments.
    Only modify files in the working directory. A later workflow step handles publishing."

- name: Publish docs branch (deterministic)
  run: |
    # 決定論的git操作はCIで処理
    git checkout -B "docs/${{ github.head_ref }}"
    git add -A
    git commit -m "docs: update for PR"
    git push origin "docs/${{ github.head_ref }}"

- name: Post PR comment (deterministic)
  run: |
    # 決定論的PRコメントはCIで処理
    gh pr comment ${{ github.event.pull_request.number }} --body "Docs updated"
```

**Restricted Autonomy の特徴:**
- ✅ 予測可能で制御された動作
- ✅ 明確な監査証跡
- ✅ インテリジェントな分析 + 決定論的実行
- ⚠️ より複雑なワークフロー設定

#### 権限ベース制限

CLI レベルで制限を強制：

```yaml
- name: Setup restricted permissions
  run: |
    cat > .cursor/permissions.json << EOF
    {
      "permissions": {
        "allow": [
          "Read(**/*.md)",
          "Write(docs/**/*)",
          "Shell(grep)",
          "Shell(find)"
        ],
        "deny": [
          "Shell(git)",
          "Shell(gh)",
          "Write(.env*)",
          "Write(package.json)"
        ]
      }
    }
    EOF

- name: Run with restrictions
  run: |
    cursor-agent -p "Documentation update" --config .cursor/permissions.json
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

Cursor CLIは以下の要件を満たす**あらゆるCI/CDシステム**で動作します：

#### 基本要件
- **シェルスクリプト実行** (bash, zsh等)
- **環境変数サポート** APIキー設定用
- **インターネット接続** Cursor API接続用

参考: [GitHub Actions - Other CI systems](https://docs.cursor.com/en/cli/github-actions#other-ci-systems)

#### 対応システム例

**GitLab CI/CD**
```yaml
stages:
  - setup
  - cursor-automation

cursor-job:
  stage: cursor-automation
  image: ubuntu:latest
  before_script:
    - curl https://cursor.com/install -fsS | bash
    - export PATH="$HOME/.cursor/bin:$PATH"
  script:
    - cursor-agent -p "Automated task" --model gpt-4
  variables:
    CURSOR_API_KEY: $CURSOR_API_KEY
```

**Jenkins Pipeline**
```groovy
pipeline {
    agent any
    environment {
        CURSOR_API_KEY = credentials('cursor-api-key')
    }
    stages {
        stage('Setup Cursor CLI') {
            steps {
                sh 'curl https://cursor.com/install -fsS | bash'
                sh 'export PATH="$HOME/.cursor/bin:$PATH"'
            }
        }
        stage('Run Automation') {
            steps {
                sh 'cursor-agent -p "Jenkins automation task" --model gpt-4'
            }
        }
    }
}
```

**Azure DevOps**
```yaml
trigger:
- main

pool:
  vmImage: 'ubuntu-latest'

variables:
  CURSOR_API_KEY: $(cursor-api-key)

steps:
- bash: |
    curl https://cursor.com/install -fsS | bash
    export PATH="$HOME/.cursor/bin:$PATH"
    cursor-agent -p "Azure DevOps automation" --model gpt-4
  displayName: 'Run Cursor Agent'
```

**CircleCI**
```yaml
version: 2.1

jobs:
  cursor-automation:
    docker:
      - image: cimg/base:stable
    steps:
      - checkout
      - run:
          name: Install Cursor CLI
          command: |
            curl https://cursor.com/install -fsS | bash
            echo 'export PATH="$HOME/.cursor/bin:$PATH"' >> $BASH_ENV
      - run:
          name: Run Automation
          command: cursor-agent -p "CircleCI automation" --model gpt-4

workflows:
  automation:
    jobs:
      - cursor-automation
```

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

### 2. CI障害自動修正（公式Cookbook版）

**公式例ベース**: [Auto Fix CI Failures](https://docs.cursor.com/en/cli/cookbook/auto-fix-ci)

```yaml
name: Auto Fix CI Failures

on:
  workflow_run:
    workflows: [Test]  # 監視するワークフロー名を更新
    types: [completed]

permissions:
  contents: write
  pull-requests: write
  actions: read

jobs:
  attempt-fix:
    if: >-
      ${{ github.event.workflow_run.conclusion == 'failure' && github.event.workflow_run.name != 'Auto Fix CI Failures' }}
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repository
        uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Install Cursor CLI
        run: |
          curl https://cursor.com/install -fsS | bash
          echo "$HOME/.cursor/bin" >> $GITHUB_PATH

      - name: Configure git identity
        run: |
          git config user.name "Cursor Agent"
          git config user.email "cursoragent@cursor.com"

      - name: Fix CI failure
        env:
          CURSOR_API_KEY: ${{ secrets.CURSOR_API_KEY }}
          MODEL: gpt-5
          GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          BRANCH_PREFIX: ci-fix
        run: |
          cursor-agent -p "You are operating in a GitHub Actions runner.

          The GitHub CLI is available as \`gh\` and authenticated via \`GH_TOKEN\`. Git is available. You have write access to repository contents and can comment on pull requests, but you must not create or edit PRs directly.

          # Context:
          - Repo: ${{ github.repository }}
          - Owner: ${{ github.repository_owner }}
          - Workflow Run ID: ${{ github.event.workflow_run.id }}
          - Workflow Run URL: ${{ github.event.workflow_run.html_url }}
          - Fix Branch Prefix: ${{ env.BRANCH_PREFIX }}

          # Goal:
          - Implement an end-to-end CI fix flow driven by the failing PR, creating a separate persistent fix branch and proposing a quick-create PR back into the original PR's branch.

          # Requirements:
          1) Identify the PR associated with the failed workflow run and determine its base and head branches. Let HEAD_REF be the PR's head branch (the contributor/origin branch).
          2) Maintain a persistent fix branch for this PR head using the Fix Branch Prefix from Context. Create it if missing, update it otherwise, and push changes to origin.
          3) Attempt to resolve the CI failure by making minimal, targeted edits consistent with the repo's style. Keep changes scoped and safe.
          4) You do NOT have permission to create PRs. Instead, post or update a single natural-language PR comment (1–2 sentences) that briefly explains the CI fix and includes an inline compare link to quick-create a PR.

          # Inputs and conventions:
          - Use \`gh api\`, \`gh run view\`, \`gh pr view\`, \`gh pr diff\`, \`gh pr list\`, \`gh run download\`, and git commands as needed to discover the failing PR and branches.
          - Avoid duplicate comments; if a previous bot comment exists, update it instead of posting a new one.
          - If no actionable fix is possible, make no changes and post no comment.

          # Deliverables when updates occur:
          - Pushed commits to the persistent fix branch for this PR head.
          - A single natural-language PR comment on the original PR that includes the inline compare link above.
          " --force --model "$MODEL" --output-format=text
```

### 3. セキュリティ監査自動化（Secret Audit）

**公式Cookbook**: [Auto Secret Audit](https://docs.cursor.com/en/cli/cookbook/auto-secret-audit)

```bash
#!/bin/bash
# auto-secret-audit.sh - 自動秘密情報監査

echo "🔒 Starting automated security audit..."

cursor-agent -p --force "
Perform comprehensive security audit focused on secret detection and remediation:

1. **Secret Detection**:
   - Scan all source files for hardcoded API keys, passwords, tokens
   - Check .env files are properly gitignored
   - Identify database connection strings in code
   - Detect JWT secrets, encryption keys, certificates

2. **Dependency Vulnerabilities**:
   - Audit package.json / requirements.txt / Gemfile vulnerabilities
   - Identify outdated packages with known CVEs
   - Check for malicious dependencies

3. **Code Security Patterns**:
   - SQL injection vulnerability analysis
   - XSS vulnerability detection
   - Path traversal issues
   - Unsafe deserialization patterns

4. **Configuration Security**:
   - Security headers validation
   - HTTPS enforcement checks
   - CORS policy review
   - Authentication/authorization patterns

5. **Automated Remediation**:
   - Move detected secrets to environment variables
   - Add .env* to .gitignore if missing
   - Update vulnerable dependencies where safe
   - Apply security header configurations

6. **Security Report Generation**:
   - Create security-audit-$(date +%Y%m%d).md with findings
   - Categorize issues by severity (Critical/High/Medium/Low)
   - Provide actionable remediation steps
   - Generate compliance checklist

Focus on immediate security risks and provide automated fixes where possible.
" --model gpt-4 --output-format text

echo "✅ Security audit completed - check security-audit-*.md for details"
```

### 4. 多言語対応自動化（Translate Keys）

**公式Cookbook**: [Auto Translate Keys](https://docs.cursor.com/en/cli/cookbook/auto-translate-keys)

```bash
#!/bin/bash
# auto-translate-keys.sh - 自動翻訳キー管理

echo "🌍 Starting automated translation key management..."

cursor-agent -p --force "
Perform comprehensive internationalization (i18n) key translation and management:

1. **Translation Structure Analysis**:
   - Scan src/locales/, i18n/, public/locales/ directories
   - Identify base language (usually en.json or English keys)
   - Map existing translation files and supported languages
   - Detect missing translation keys across languages

2. **New Key Detection**:
   - Find newly added keys in base language files
   - Scan source code for hardcoded strings that need translation
   - Identify unused translation keys for cleanup
   - Check for key naming consistency and conventions

3. **Intelligent Translation**:
   - Translate missing keys to target languages: ja, zh, ko, es, fr, de, pt, it
   - Maintain context awareness for UI elements vs general text
   - Preserve technical terms and brand names consistently
   - Handle pluralization rules per language
   - Respect cultural nuances and local conventions

4. **File Management**:
   - Update language files (ja.json, zh.json, ko.json, etc.)
   - Maintain JSON structure consistency and formatting
   - Sort keys alphabetically or by logical grouping
   - Validate JSON syntax and structure

5. **Quality Assurance**:
   - Check placeholder consistency ({name}, {count}, etc.)
   - Validate character length constraints for UI elements
   - Ensure context appropriateness of translations
   - Detect potential translation conflicts or ambiguities

6. **Translation Report**:
   - Create translation-report-$(date +%Y%m%d).md
   - List newly translated keys
   - Report coverage statistics per language
   - Highlight keys requiring manual review
   - Generate translation completion metrics

Focus on maintaining translation quality while ensuring consistent user experience across all supported languages.
" --model gpt-4 --output-format text

echo "✅ Translation key management completed - check translation-report-*.md"
```

---

## 設定とパラメータ

### グローバルオプション

全てのコマンドで使用可能なオプション：

| オプション                 | 説明                                         | 使用例                          |
| -------------------------- | -------------------------------------------- | ------------------------------- |
| `-v, --version`            | バージョン番号を表示                         | `cursor-agent --version`        |
| `-a, --api-key <key>`      | API キー指定（CURSOR_API_KEY環境変数でも可） | `cursor-agent --api-key sk-xxx` |
| `-p, --print`              | コンソール出力モード（全ツールアクセス含む） | `cursor-agent -p "コード分析"`  |
| `--output-format <format>` | 出力形式: text, json, stream-json            | `--output-format json`          |
| `-b, --background`         | バックグラウンドモード                       | `cursor-agent -b`               |
| `--fullscreen`             | フルスクリーンモード有効                     | `cursor-agent --fullscreen`     |
| `--resume [chatId]`        | チャットセッション再開                       | `--resume my-chat`              |
| `-m, --model <model>`      | 使用モデル指定                               | `-m gpt-4`                      |
| `-f, --force`              | コマンド強制実行（明示的拒否以外）           | `cursor-agent -f -p "修正実行"` |
| `-h, --help`               | ヘルプ表示                                   | `cursor-agent --help`           |

### 利用可能なコマンド

| コマンド          | 説明                     | 使用例                  |
| ----------------- | ------------------------ | ----------------------- |
| `login`           | Cursor認証               | `cursor-agent login`    |
| `logout`          | サインアウト・認証クリア | `cursor-agent logout`   |
| `status`          | 認証状態確認             | `cursor-agent status`   |
| `mcp`             | MCPサーバー管理          | `cursor-agent mcp list` |
| `update\|upgrade` | 最新版にアップデート     | `cursor-agent update`   |
| `ls`              | チャット一覧表示         | `cursor-agent ls`       |
| `resume`          | 最新チャット再開         | `cursor-agent resume`   |
| `help [command]`  | コマンドヘルプ           | `cursor-agent help mcp` |

### 引数とプロンプト

コマンド未指定時は対話式チャットモードが開始されます。初期プロンプトも指定可能：

```bash
# 対話式モード
cursor-agent

# 初期プロンプト付き対話式モード
cursor-agent "プロジェクトを分析してください"

# 非対話式モード
cursor-agent -p "README.mdを生成"
```

### MCP (Model Context Protocol) 管理

MCPサーバーを管理するための専用コマンド群：

| サブコマンド              | 説明                                          | 使用例                                  |
| ------------------------- | --------------------------------------------- | --------------------------------------- |
| `login <identifier>`      | .cursor/mcp.jsonで設定されたMCPサーバーに認証 | `cursor-agent mcp login my-server`      |
| `list`                    | 設定済みMCPサーバーと状態一覧                 | `cursor-agent mcp list`                 |
| `list-tools <identifier>` | 特定MCPサーバーの利用可能ツール一覧           | `cursor-agent mcp list-tools my-server` |

#### MCP設定例

`.cursor/mcp.json`設定ファイル例：
```json
{
  "mcpServers": {
    "my-server": {
      "command": "node",
      "args": ["my-mcp-server.js"],
      "env": {
        "API_KEY": "your-api-key"
      }
    },
    "database-server": {
      "command": "python",
      "args": ["-m", "database_mcp"],
      "env": {
        "DB_CONNECTION": "postgresql://user:pass@localhost/db"
      }
    }
  }
}
```

#### MCP使用例
```bash
# MCPサーバー一覧確認
cursor-agent mcp list

# 特定サーバーに認証
cursor-agent mcp login database-server

# 利用可能ツール確認
cursor-agent mcp list-tools database-server

# MCPツールを活用したプロンプト
cursor-agent -p "database-serverのツールを使ってユーザーテーブルを分析"
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

Cursor CLIは2つの認証方法をサポートしています：ブラウザ認証（推奨）とAPIキー認証。

#### 1. ブラウザ認証（推奨）

最も簡単で安全な認証方法です：

```bash
# ブラウザフローでログイン
cursor-agent login

# 認証状態確認
cursor-agent status

# サインアウトと認証情報クリア
cursor-agent logout
```

**ブラウザ認証の流れ：**
1. `cursor-agent login` コマンド実行
2. デフォルトブラウザが自動で開く
3. Cursorアカウントでの認証プロンプト
4. 認証完了後、認証情報がローカルに安全に保存

**ブラウザ認証の利点：**
- セキュアな認証フロー
- APIキー管理不要
- ローカル認証情報の自動管理
- 簡単なログイン・ログアウト

#### 2. APIキー認証

自動化、スクリプト、CI/CD環境での使用に最適：

##### ステップ1: APIキー生成
Cursorダッシュボードで生成：
1. Cursorアプリケーションを開く
2. 設定 (Settings) → Integrations → User API Keys
3. 新しいAPIキーを生成

##### ステップ2: APIキー設定

**オプション1: 環境変数（推奨）**
```bash
export CURSOR_API_KEY=your_api_key_here
cursor-agent "ユーザー認証を実装"
```

**オプション2: コマンドライン引数**
```bash
cursor-agent --api-key your_api_key_here "ユーザー認証を実装"
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

### 認証状態の確認

現在の認証状態を確認できます：

```bash
cursor-agent status
```

**表示情報：**
- 認証されているかどうか
- アカウント情報
- 現在のエンドポイント設定

### トラブルシューティング

認証関連の一般的な問題と解決方法：

#### 認証エラー
```bash
# "認証されていません" エラー
cursor-agent login
# または
export CURSOR_API_KEY=your_api_key

# SSL証明書エラー（開発環境）
cursor-agent --insecure "プロンプト"

# エンドポイントエラー
cursor-agent --endpoint https://custom-api.cursor.com "プロンプト"
```

#### 認証状態のリセット
```bash
# 完全なログアウト
cursor-agent logout

# 設定ファイルの手動削除
rm -rf ~/.cursor/auth

# 再認証
cursor-agent login
```

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

### 高度な使用パターン

#### セッション管理

```bash
# チャット履歴の確認
cursor-agent ls

# 特定プロジェクト用のチャット再開
cursor-agent resume project-analysis

# チャットID指定での再開
cursor-agent --resume chat-abc123

# バックグラウンドモードでの開始
cursor-agent -b --fullscreen
```

#### カスタム設定ファイル

`.cursor/config.json` での高度な設定：
```json
{
  "default_model": "gpt-4",
  "output_format": "stream-json",
  "timeout": 600,
  "max_tokens": 8000,
  "temperature": 0.3,
  "working_directory": "./src",
  "auto_run": false,
  "vim_mode": true,
  "permissions": {
    "allow": [
      "Read(**/*.{js,ts,py,md})",
      "Write(docs/**/*)",
      "Shell(git status)",
      "Shell(npm test)"
    ],
    "deny": [
      "Shell(rm -rf)",
      "Write(.env*)",
      "System(*)"
    ]
  },
  "mcp_servers": {
    "database": {
      "command": "python",
      "args": ["-m", "db_mcp_server"],
      "env": {
        "DATABASE_URL": "${DATABASE_URL}"
      }
    }
  }
}
```

#### 統合開発ワークフロー

**1. コードレビュー自動化**
```bash
#!/bin/bash
# comprehensive-review.sh

# ブランチ差分分析
cursor-agent -p "
git diff origin/main..HEAD の変更を分析し、以下を実行：

1. **コード品質評価**:
   - TypeScript/JavaScript: ESLint, Prettier準拠確認
   - Python: Black, isort, flake8準拠確認
   - 複雑度分析（サイクロマティック複雑度）

2. **セキュリティスキャン**:
   - SQLインジェクション脆弱性
   - XSS脆弱性
   - ハードコード化された秘密情報
   - 依存関係の脆弱性

3. **パフォーマンス分析**:
   - N+1クエリ問題
   - 不効率なアルゴリズム
   - メモリリーク可能性

4. **テスト要件**:
   - 新機能のテストケース提案
   - エッジケース特定
   - モックオブジェクト提案

詳細レポートを review-$(date +%Y%m%d-%H%M%S).md に出力
" --force --model gpt-4
```

**2. ドキュメント生成パイプライン**
```bash
#!/bin/bash
# doc-pipeline.sh

# APIドキュメント自動生成
cursor-agent -p "
プロジェクト全体を分析し、以下のドキュメントを生成：

1. **API仕様書** (api-spec.md):
   - OpenAPI 3.0形式
   - 全エンドポイントの詳細
   - リクエスト/レスポンス例
   - 認証方法

2. **開発者ガイド** (dev-guide.md):
   - セットアップ手順
   - 開発ワークフロー
   - コーディング規約
   - デバッグガイド

3. **デプロイメントガイド** (deployment.md):
   - 環境構築
   - CI/CDパイプライン
   - 監視設定
   - トラブルシューティング

4. **ユーザーガイド** (user-guide.md):
   - 機能概要
   - 使用方法
   - FAQ
   - サポート情報

既存のドキュメントは保持し、新情報のみ追加・更新
" --force --model gpt-4

# 翻訳パイプライン実行
cursor-agent -p "
生成されたドキュメントを以下言語に翻訳：
- 日本語 (docs/ja/)
- 中国語 (docs/zh/)
- スペイン語 (docs/es/)
- フランス語 (docs/fr/)

技術用語の一貫性維持、文化的適応を考慮
" --force --model gpt-4
```

**3. 自動テスト生成**
```bash
#!/bin/bash
# test-generator.sh

cursor-agent -p "
現在のコードベースに対して包括的なテストスイートを生成：

1. **単体テスト** (tests/unit/):
   - 全関数・メソッドのテスト
   - エッジケースを含む
   - モックオブジェクト適切使用
   - カバレッジ90%以上を目標

2. **統合テスト** (tests/integration/):
   - API エンドポイントテスト
   - データベース統合テスト
   - 外部サービス統合テスト

3. **E2Eテスト** (tests/e2e/):
   - ユーザーシナリオベース
   - クロスブラウザテスト
   - モバイル対応テスト

4. **パフォーマンステスト** (tests/performance/):
   - 負荷テスト
   - ストレステスト
   - メモリ使用量テスト

既存テストフレームワーク（Jest, Cypress等）に準拠
" --force --model gpt-4
```

#### 継続的品質改善

**品質メトリクス監視**
```bash
#!/bin/bash
# quality-metrics.sh

# 週次品質レポート
cursor-agent -p "
過去1週間のコードベース変更を分析し、品質メトリクスレポートを生成：

## メトリクス:
- コード複雑度の変化
- テストカバレッジの変化
- 技術的負債の増減
- セキュリティスコア
- パフォーマンス指標

## 分析項目:
- 改善された領域
- 悪化した領域
- 推奨アクション
- 次週の目標設定

レポートファイル: quality-report-$(date +%Y-W%U).md
Slackチャンネル投稿用サマリーも生成
" --force --model gpt-4 --output-format json

# Slack通知（オプション）
if [ -n "$SLACK_WEBHOOK_URL" ]; then
  summary=$(cat quality-report-*.md | head -20)
  curl -X POST -H 'Content-type: application/json' \
    --data "{\"text\":\"📊 週次品質レポート\n\`\`\`$summary\`\`\`\"}" \
    "$SLACK_WEBHOOK_URL"
fi
```

---

## まとめ

このガイドでは、Cursor CLIの包括的な使用方法を生成AI向けに最適化された形式で説明しました。公式ドキュメント11つのURL（overview, installation, using, slash-commands, parameters, authentication, permissions, output-format, headless, github-actions, cookbook）のすべての情報を統合し、実践的な使用例を豊富に提供しています。

### 📋 **網羅されている主要機能**
1. **対話式スラッシュコマンド**: `/model`, `/auto-run`, `/new-chat`, `/vim`, `/help`, `/feedback`, `/resume`, `/copy-req-id`, `/logout`, `/quit`
2. **包括的パラメータ**: グローバルオプション、コマンド、引数、MCP管理
3. **二重認証システム**: ブラウザ認証（推奨）とAPIキー認証
4. **MCP統合**: Model Context Protocol サーバー管理とツール活用
5. **高度な権限制御**: 詳細なパーミッション設定と環境別構成
6. **多様な出力フォーマット**: text, json, stream-json での用途別最適化
7. **CI/CD統合**: GitHub Actions, Bitbucket Pipelines, GitLab, Jenkins, Azure DevOps, CircleCI
8. **エンタープライズ機能**: セッション管理、設定ファイル、統合ワークフロー
9. **Rules システム**: .cursor/rules, AGENT.md, CLAUDE.md によるカスタマイズ
10. **対話式操作**: キーバインド、レビュー機能、コンテキスト管理
11. **コマンド承認**: ターミナルコマンド実行時の安全性確保
12. **ヘッドレスモード**: スクリプト・自動化での非対話実行
13. **自律性レベル**: Full Autonomy vs Restricted Autonomy アプローチ
14. **公式Cookbook**: CI修正、ドキュメント更新、セキュリティ監査、翻訳自動化

### 🔑 **重要なポイント**
1. **セキュリティファースト**: ブラウザ認証優先、権限制御の適切な設定
2. **自動化重視**: CI/CDパイプラインでの効果的な活用
3. **MCP活用**: 外部システム統合によるCursor拡張
4. **自律性レベル選択**: Full vs Restricted Autonomy での適切な制御
5. **コマンド承認**: 対話式での安全性確保、非対話式での自動実行
6. **Rules システム**: プロジェクト固有のカスタマイズ
7. **エラーハンドリング**: 堅牢なスクリプト設計とリトライ機能
8. **パフォーマンス最適化**: 効率的な処理パターンとバッチ実行
9. **監視とロギング**: 運用時の可視性確保と品質メトリクス
10. **セッション管理**: チャット履歴とコンテキスト継続性
11. **統合開発**: コードレビュー、ドキュメント生成、テスト自動化
12. **出力フォーマット活用**: text, json, stream-json の適切な選択

### 💡 **実装時の注意事項**
- **認証**: ブラウザ認証を優先、CI/CDではAPIキー使用
- **権限**: 最小権限の原則に従い、環境別設定を実装
- **自律性レベル**: 本番では Restricted Autonomy を推奨
- **コマンド承認**: 対話式は手動承認、非対話式は --force で自動承認
- **Rules活用**: .cursor/rules, AGENT.md, CLAUDE.md でプロジェクト固有設定
- **エラー処理**: リトライ機能とタイムアウト設定を含む
- **ログとメトリクス**: 構造化ログと品質メトリクスを適切に収集
- **セッション管理**: プロジェクト別チャット継続性の活用
- **MCP統合**: 外部ツールとサービスの効果的な統合
- **出力フォーマット**: 用途に応じた text/json/stream-json の選択
- **段階的テスト**: ドライランとデバッグモードでの検証

### 🚀 **次のステップ**
1. **基本セットアップ**: `cursor-agent login` でブラウザ認証
2. **Rules設定**: `.cursor/rules/`, `AGENT.md`, `CLAUDE.md` でプロジェクト固有カスタマイズ
3. **権限設定**: プロジェクトに応じた`.cursor/config.json`作成
4. **CI/CD統合**: Restricted Autonomy での自動化実装
5. **MCP拡張**: 必要に応じて外部サービス統合
6. **Cookbook活用**: 公式例を参考にした実践的ワークフロー構築
7. **監視実装**: 品質メトリクスと継続的改善システム構築

このガイドを参考に、Cursor CLIを効果的に活用した包括的な開発自動化ソリューションを構築してください。
