# Rundeck Plugin開発 完全ガイド

## 目次
1. [概要](#1-概要)
2. [プラグインタイプ](#2-プラグインタイプ)
3. [Javaプラグイン開発詳細仕様](#3-javaプラグイン開発詳細仕様)
4. [スクリプトプラグイン開発詳細仕様](#4-スクリプトプラグイン開発詳細仕様)
5. [Groovyプラグイン開発](#5-groovyプラグイン開発)
6. [UI Plugin開発](#6-ui-plugin開発)
7. [国際化（i18n）対応](#7-国際化i18n対応)
8. [プラグイン配置とインストール](#8-プラグイン配置とインストール)
9. [開発ベストプラクティス](#9-開発ベストプラクティス)
10. [テストと品質保証](#10-テストと品質保証)
11. [参考リンク集](#11-参考リンク集)

---

## 1. 概要

Rundeck Pluginは、Rundeckの機能を拡張するためのコンポーネントです。プラグインは既存のRundeckインストールに追加機能を提供し、カスタムワークフローやインテグレーションを可能にします。

Rundeckでは現在3つの主要な開発方法があります：

1. **Javaプラグイン開発**: Jarファイル内に配布されるJavaコードを開発
2. **スクリプトプラグイン開発**: 所望の動作を実装するシェル/システムスクリプトを書き、メタデータとともにzipファイルに格納
3. **Groovyプラグイン開発**: 通知およびログプラグインを実装するGroovyスクリプトを記述

---

## 2. プラグインタイプ

### 2.1 Javaプラグイン開発
- **配布形式**: `.jar`ファイル
- **言語**: Java
- **特徴**: 高性能、豊富な機能、複雑なロジックに適している

### 2.2 スクリプトプラグイン開発  
- **配布形式**: `.zip`ファイル
- **言語**: シェルスクリプト、システムスクリプト
- **特徴**: 簡単な実装、既存スクリプトの活用が可能

### 2.3 Groovyプラグイン開発
- **配布形式**: `.zip`ファイル  
- **言語**: Groovy
- **対象**: Notification、Loggingプラグイン
- **特徴**: Java環境でのスクリプト言語利用

### 2.4 対応サービスタイプ

#### スクリプトプラグインで実装可能なサービス：
- **NodeExecutor**: ノード上でコマンドを実行
- **FileCopier**: ファイルをノードにコピー
- **ResourceModelSource**: ノードリソース情報を提供
- **WorkflowNodeStep**: ワークフローステップを実行
- **RemoteScriptNodeStep**: リモートスクリプトステップを実行

#### Groovyプラグインで対応可能なサービス：
- **Notification Plugin**: 通知プラグイン
- **Streaming Log Reader**: ストリーミングログリーダー
- **Streaming Log Writer**: ストリーミングログライター
- **Execution File Storage**: 実行ファイルストレージ
- **Log Filter**: ログフィルター
- **Content Converter Plugin**: コンテンツコンバータープラグイン

---

## 3. Javaプラグイン開発詳細仕様

### 3.1 基本要件

#### 必須依存関係
```xml
<!-- Maven依存関係 -->
<dependencies>
   <dependency>
      <groupId>org.rundeck</groupId>
      <artifactId>rundeck-core</artifactId>
      <version>5.13.0-20250625</version>
      <scope>compile</scope>
   </dependency>
</dependencies>
```

```groovy
// Gradle依存関係
implementation(group:'org.rundeck', name: 'rundeck-core', version: '5.13.0-20250625')
```

#### ストレージプラグイン用追加依存関係
```xml
<dependency>
   <groupId>org.rundeck</groupId>
   <artifactId>rundeck-storage-api</artifactId>
   <version>5.13.0-20250625</version>
</dependency>
```

### 3.2 プラグインクラス実装

#### 基本構造
```java
@Plugin(name = "my-plugin", service = ServiceNameConstants.NodeExecutor)
public class MyPlugin implements NodeExecutor {
    
    @PluginProperty(title = "Host", description = "Target host")
    private String host;
    
    @PluginProperty(title = "Port", description = "Target port", defaultValue = "22")
    private int port;
    
    @Override
    public NodeExecutorResult executeCommand(ExecutionContext context, 
                                           String[] command, 
                                           INodeEntry node) throws NodeExecutorException {
        // プラグインロジックの実装
        return new NodeExecutorResultImpl(0, "Success", null);
    }
}
```

#### アノテーション仕様

**@Plugin**
- `name`: プラグインの一意識別子
- `service`: サービスタイプ（ServiceNameConstants参照）

**@PluginProperty**
- `title`: ユーザーインターフェースでの表示名
- `description`: プロパティの説明
- `defaultValue`: デフォルト値
- `required`: 必須フラグ（boolean）
- `scope`: スコープ設定

### 3.3 Manifestファイル仕様

#### 必須属性
```
Manifest-Version: 1.0
Rundeck-Plugin-Version: 1.2
Rundeck-Plugin-Archive: true
Rundeck-Plugin-Classnames: com.example.MyPlugin,com.example.AnotherPlugin
Rundeck-Plugin-File-Version: 1.0
```

#### オプション属性
```
Rundeck-Plugin-Libs: lib/somejar-1.2.jar lib/anotherjar-1.3.jar
Rundeck-Plugin-Author: Author Name
Rundeck-Plugin-URL: https://example.com/plugin
Rundeck-Plugin-Date: 2025-01-01T00:00:00Z
```

### 3.4 プラグインプロパティ詳細仕様

#### displayType設定
```java
@PluginProperty(
    title = "Configuration",
    description = "JSON configuration",
    displayType = "CODE"
)
private String config;
```

**利用可能なdisplayType:**
- `TEXT`: 標準テキスト入力
- `PASSWORD`: パスワード入力（マスク表示）
- `BOOLEAN`: チェックボックス
- `SELECT`: セレクトボックス
- `MULTI_SELECT`: 複数選択
- `CODE`: コードエディタ

#### CODE displayType専用設定
```java
@PluginProperty(
    title = "Script",
    description = "Executable script",
    displayType = "CODE",
    codeSyntaxMode = "sh",
    codeSyntaxSelectable = "true"
)
private String script;
```

**対応シンタックスモード:**
- `batchfile`, `diff`, `dockerfile`, `golang`, `groovy`
- `html`, `java`, `javascript`, `json`, `markdown`
- `perl`, `php`, `powershell`, `properties`, `python`
- `ruby`, `sh`, `sql`, `xml`, `yaml`

### 3.5 外部ライブラリ統合

#### ライブラリ包含構造
```
plugin.jar
├── META-INF/
│   └── MANIFEST.MF
├── com/
│   └── example/
│       └── plugin/
│           └── MyPlugin.class
└── lib/
    ├── external-lib-1.0.jar
    └── another-lib-2.0.jar
```

#### Manifest設定
```
Rundeck-Plugin-Libs: lib/external-lib-1.0.jar lib/another-lib-2.0.jar
```

---

## 4. スクリプトプラグイン開発詳細仕様

### 4.1 スクリプトプラグイン概要

スクリプトプラグインは、シェルスクリプトやシステムスクリプトを使用してRundeckの機能を拡張する方法です。Javaの知識なしに、既存のスクリプトを活用してプラグインを開発できます。

### 4.2 プラグイン構造詳細

#### 基本ディレクトリ構造
```
plugin.zip
├── plugin.yaml                    # プラグイン定義ファイル（必須）
├── contents/                      # スクリプトファイル格納（必須）
│   ├── script.sh                  # メインスクリプト
│   ├── helper.py                  # 補助スクリプト
│   └── config/
│       └── default.conf
├── resources/                     # リソースファイル（オプション）
│   ├── i18n/                      # 国際化対応
│   │   ├── messages.properties
│   │   ├── messages_ja.properties
│   │   └── messages_zh.properties
│   ├── templates/                 # テンプレートファイル
│   │   └── config.template
│   └── docs/                      # ドキュメント
│       └── README.md
└── lib/                           # 外部ライブラリ（オプション）
    └── external-tool
```

### 4.3 plugin.yaml詳細仕様

#### 基本メタデータ
```yaml
# プラグイン基本情報
name: my-script-plugin              # プラグイン名（必須）
version: 1.0.0                     # プラグインバージョン（必須）
rundeckPluginVersion: 1.2          # Rundeckプラグインバージョン（必須）
author: Author Name                 # 作成者（オプション）
date: 2025-01-25                   # 作成日（オプション）
url: https://example.com/plugin     # プラグインURL（オプション）
description: |                     # 説明（オプション）
  This plugin provides custom functionality
  for executing scripts on remote nodes.
```

#### プロバイダー定義
```yaml
providers:
  - name: my-node-executor          # プロバイダー名（必須）
    service: NodeExecutor           # サービスタイプ（必須）
    plugin-type: script             # プラグインタイプ（必須）
    script-interpreter: /bin/bash   # インタープリター（必須）
    script-file: script.sh          # スクリプトファイル（必須）
    script-args: ${node.hostname} ${config.port}  # スクリプト引数（オプション）
    
    # プラグイン設定項目
    config:
      - name: host                  # 設定項目名
        title: Target Host          # 表示名
        type: String               # データタイプ
        required: true             # 必須フラグ
        description: "Target hostname or IP address"
        default: localhost         # デフォルト値
        
      - name: port
        title: Port Number
        type: Integer
        required: false
        description: "Connection port number"
        default: 22
        validation: |              # バリデーション（オプション）
          if [ "$1" -lt 1 ] || [ "$1" -gt 65535 ]; then
            echo "Port must be between 1 and 65535"
            exit 1
          fi
          
      - name: use_ssl
        title: Use SSL
        type: Boolean
        required: false
        default: false
        
      - name: auth_method
        title: Authentication Method
        type: Select
        required: true
        default: password
        values:
          - password
          - key
          - certificate
          
      - name: custom_config
        title: Custom Configuration
        type: FreeSelect
        required: false
        values:
          - option1
          - option2
          - option3
        description: "Select predefined option or enter custom value"
```

#### 複数プロバイダー定義例
```yaml
providers:
  # NodeExecutor プロバイダー
  - name: ssh-executor
    service: NodeExecutor
    plugin-type: script
    script-interpreter: /bin/bash
    script-file: ssh-execute.sh
    config:
      - name: ssh_key_path
        title: SSH Key Path
        type: String
        required: true
        
  # FileCopier プロバイダー
  - name: scp-copier
    service: FileCopier
    plugin-type: script
    script-interpreter: /bin/bash
    script-file: scp-copy.sh
    config:
      - name: copy_mode
        title: Copy Mode
        type: Select
        values: [secure, fast, compressed]
        default: secure
```

### 4.4 設定タイプ詳細仕様

#### 利用可能なデータタイプ

| タイプ | 説明 | GUI表示 | 例 |
|--------|------|---------|-----|
| `String` | 文字列 | テキストボックス | hostname, username |
| `Integer` | 整数 | 数値入力 | port, timeout |
| `Long` | 長整数 | 数値入力 | file_size, timestamp |
| `Boolean` | 真偽値 | チェックボックス | enable_ssl, debug_mode |
| `Select` | 選択肢 | セレクトボックス | auth_method, protocol |
| `FreeSelect` | 自由選択 | 入力可能セレクト | custom_values |

#### 高度な設定オプション
```yaml
config:
  - name: complex_config
    title: Complex Configuration
    type: String
    required: true
    description: "Multi-line configuration"
    renderOptions:
      displayType: MULTI_LINE     # 複数行テキスト
      syntax: yaml               # シンタックスハイライト
      
  - name: password_field
    title: Password
    type: String
    required: true
    renderOptions:
      displayType: PASSWORD      # パスワードフィールド
      
  - name: file_path
    title: File Path
    type: String
    required: false
    renderOptions:
      displayType: FILE_SELECT   # ファイル選択
      
  - name: script_content
    title: Script Content
    type: String
    required: false
    renderOptions:
      displayType: CODE_EDITOR   # コードエディタ
      syntax: bash
```

### 4.5 スクリプト実装詳細

#### NodeExecutor スクリプト例
```bash
#!/bin/bash
# ssh-execute.sh

# 環境変数から設定値を取得
HOST=${RD_CONFIG_HOST:-localhost}
PORT=${RD_CONFIG_PORT:-22}
USERNAME=${RD_CONFIG_USERNAME:-$(whoami)}
SSH_KEY=${RD_CONFIG_SSH_KEY_PATH}

# ノード情報から値を取得
NODE_NAME=${RD_NODE_NAME}
NODE_HOSTNAME=${RD_NODE_HOSTNAME}
NODE_USERNAME=${RD_NODE_USERNAME}

# 実行するコマンドを取得
COMMAND="$@"

# ログ出力
echo "Executing on node: ${NODE_NAME} (${NODE_HOSTNAME})"
echo "Command: ${COMMAND}"

# SSH接続でコマンド実行
if [ -n "${SSH_KEY}" ]; then
    # SSH鍵認証
    ssh -i "${SSH_KEY}" \
        -p "${PORT}" \
        -o StrictHostKeyChecking=no \
        -o UserKnownHostsFile=/dev/null \
        "${USERNAME}@${HOST}" \
        "${COMMAND}"
else
    # パスワード認証（非推奨、デモ用）
    sshpass -p "${RD_CONFIG_PASSWORD}" \
        ssh -p "${PORT}" \
        -o StrictHostKeyChecking=no \
        -o UserKnownHostsFile=/dev/null \
        "${USERNAME}@${HOST}" \
        "${COMMAND}"
fi

# 実行結果を返す
exit $?
```

#### FileCopier スクリプト例
```bash
#!/bin/bash
# scp-copy.sh

# 引数: source_file destination_path
SOURCE_FILE="$1"
DEST_PATH="$2"

# 設定値取得
HOST=${RD_CONFIG_HOST}
PORT=${RD_CONFIG_PORT:-22}
USERNAME=${RD_CONFIG_USERNAME}
SSH_KEY=${RD_CONFIG_SSH_KEY_PATH}

# ノード情報
NODE_HOSTNAME=${RD_NODE_HOSTNAME}
NODE_USERNAME=${RD_NODE_USERNAME:-$USERNAME}

# コピー先のフルパス構築
REMOTE_PATH="${NODE_USERNAME}@${NODE_HOSTNAME}:${DEST_PATH}"

# ファイルコピー実行
if [ -n "${SSH_KEY}" ]; then
    scp -i "${SSH_KEY}" \
        -P "${PORT}" \
        -o StrictHostKeyChecking=no \
        -o UserKnownHostsFile=/dev/null \
        "${SOURCE_FILE}" \
        "${REMOTE_PATH}"
else
    sshpass -p "${RD_CONFIG_PASSWORD}" \
        scp -P "${PORT}" \
        -o StrictHostKeyChecking=no \
        -o UserKnownHostsFile=/dev/null \
        "${SOURCE_FILE}" \
        "${REMOTE_PATH}"
fi

RESULT=$?

if [ $RESULT -eq 0 ]; then
    # 成功時：コピー先のパスを標準出力に出力（必須）
    echo "${DEST_PATH}"
else
    echo "File copy failed" >&2
fi

exit $RESULT
```

#### ResourceModelSource スクリプト例
```bash
#!/bin/bash
# resource-model.sh

# 設定値取得
API_URL=${RD_CONFIG_API_URL}
API_TOKEN=${RD_CONFIG_API_TOKEN}

# ノード情報をJSON形式で出力
cat << EOF
{
  "nodes": {
    "web-server-01": {
      "nodename": "web-server-01",
      "hostname": "web01.example.com",
      "username": "rundeck",
      "osFamily": "unix",
      "osName": "Linux",
      "osVersion": "Ubuntu 20.04",
      "tags": ["web", "production"],
      "attributes": {
        "environment": "production",
        "role": "web-server",
        "datacenter": "us-east-1"
      }
    },
    "db-server-01": {
      "nodename": "db-server-01",
      "hostname": "db01.example.com",
      "username": "rundeck",
      "osFamily": "unix",
      "osName": "Linux",
      "osVersion": "Ubuntu 20.04",
      "tags": ["database", "production"],
      "attributes": {
        "environment": "production",
        "role": "database-server",
        "datacenter": "us-east-1"
      }
    }
  }
}
EOF
```

### 4.6 環境変数とコンテキスト

#### 利用可能な環境変数

##### プラグイン設定値
```bash
# plugin.yamlで定義した設定項目
RD_CONFIG_<NAME>        # 設定値（大文字変換）
# 例：
RD_CONFIG_HOST          # config.host の値
RD_CONFIG_PORT          # config.port の値
RD_CONFIG_USE_SSL       # config.use_ssl の値
```

##### ノード情報
```bash
RD_NODE_NAME            # ノード名
RD_NODE_HOSTNAME        # ホスト名
RD_NODE_USERNAME        # ユーザー名
RD_NODE_OSNAME          # OS名
RD_NODE_OSFAMILY        # OSファミリー
RD_NODE_OSVERSION       # OSバージョン
RD_NODE_OSARCH          # アーキテクチャ
RD_NODE_TAGS            # タグ（カンマ区切り）
```

##### ジョブ実行コンテキスト
```bash
RD_JOB_NAME             # ジョブ名
RD_JOB_GROUP            # ジョブグループ
RD_JOB_ID               # ジョブID
RD_JOB_EXECID           # 実行ID
RD_JOB_USERNAME         # 実行ユーザー
RD_JOB_PROJECT          # プロジェクト名
RD_JOB_LOGLEVEL         # ログレベル
```

##### プライベートデータコンテキスト
```bash
# セキュアオプション（job定義での ${option.password} など）
RD_PRIVATE_<NAME>       # プライベートデータ（大文字変換）
# 例：
RD_PRIVATE_PASSWORD     # ${option.password} の値
RD_PRIVATE_API_KEY      # ${option.api_key} の値
```

#### 環境変数使用例
```bash
#!/bin/bash

# デバッグ情報出力
if [ "${RD_JOB_LOGLEVEL}" = "DEBUG" ]; then
    echo "Debug mode enabled"
    echo "Job: ${RD_JOB_NAME} (${RD_JOB_ID})"
    echo "Node: ${RD_NODE_NAME} (${RD_NODE_HOSTNAME})"
    echo "User: ${RD_JOB_USERNAME}"
fi

# セキュアな認証情報の使用
if [ -n "${RD_PRIVATE_API_KEY}" ]; then
    curl -H "Authorization: Bearer ${RD_PRIVATE_API_KEY}" \
         "${RD_CONFIG_API_URL}/status"
fi
```

### 4.7 エラーハンドリングと出力仕様

#### 終了コード
```bash
# 成功
exit 0

# 一般的なエラー
exit 1

# 設定エラー
exit 2

# 接続エラー
exit 3

# 認証エラー
exit 4

# タイムアウト
exit 124
```

#### 出力形式

##### NodeExecutor
```bash
# 標準出力・標準エラー出力はすべてジョブログに記録
echo "Processing node: ${RD_NODE_NAME}"
echo "Error: Connection failed" >&2
```

##### FileCopier
```bash
# 標準出力の最初の行：コピー先ファイルパス（必須）
echo "/path/to/destination/file"
# その他の出力はジョブログに記録
echo "File copied successfully" >&2
```

##### ResourceModelSource
```bash
# 標準出力：JSON/XML形式のリソースデータ
echo '{"nodes": {...}}'
# 標準エラー出力：ログメッセージ
echo "Loading resources from API" >&2
```

### 4.8 テストとデバッグ

#### デバッグ用スクリプト例
```bash
#!/bin/bash
# debug-info.sh

echo "=== Debug Information ==="
echo "Plugin Config:"
env | grep "^RD_CONFIG_" | sort

echo -e "\nNode Information:"
env | grep "^RD_NODE_" | sort

echo -e "\nJob Context:"
env | grep "^RD_JOB_" | sort

echo -e "\nPrivate Data:"
env | grep "^RD_PRIVATE_" | sed 's/=.*/=***MASKED***/' | sort

echo -e "\nArguments:"
for i in "$@"; do
    echo "  Arg: '$i'"
done

echo "=== End Debug Information ==="
```

#### 単体テスト例
```bash
#!/bin/bash
# test-plugin.sh

# テスト環境変数設定
export RD_CONFIG_HOST="test.example.com"
export RD_CONFIG_PORT="2222"
export RD_CONFIG_USERNAME="testuser"
export RD_NODE_NAME="test-node"
export RD_NODE_HOSTNAME="test.example.com"

# プラグインスクリプト実行
./contents/script.sh "echo 'test command'"

# 結果確認
if [ $? -eq 0 ]; then
    echo "Test PASSED"
else
    echo "Test FAILED"
fi
```

### 4.9 パフォーマンスとセキュリティ

#### パフォーマンス最適化
```bash
# タイムアウト設定
timeout 300 ssh "${USERNAME}@${HOST}" "${COMMAND}"

# 並列処理制限
MAX_CONCURRENT=5
job_count=0
for node in "${nodes[@]}"; do
    if [ $job_count -ge $MAX_CONCURRENT ]; then
        wait  # 既存ジョブの完了を待つ
        job_count=0
    fi
    process_node "$node" &
    ((job_count++))
done
wait  # すべてのジョブ完了を待つ
```

#### セキュリティ対策
```bash
# 入力値サニタイゼーション
sanitize_input() {
    echo "$1" | sed 's/[^a-zA-Z0-9._-]//g'
}

HOST=$(sanitize_input "${RD_CONFIG_HOST}")

# 一時ファイルの安全な作成
TEMP_FILE=$(mktemp)
trap 'rm -f "$TEMP_FILE"' EXIT

# 権限設定
chmod 600 "$TEMP_FILE"
```

---

## 5. Groovyプラグイン開発

### 5.1 サポートされるGroovyプラグインタイプ

- **Notification Plugin**: 通知プラグイン
- **Streaming Log Reader**: ストリーミングログリーダー
- **Streaming Log Writer**: ストリーミングログライター
- **Execution File Storage**: 実行ファイルストレージ
- **Log Filter**: ログフィルター
- **Content Converter Plugin**: コンテンツコンバータープラグイン

### 5.2 基本的なGroovyプラグイン構造

```groovy
// Groovy Notification Plugin例
import com.dtolabs.rundeck.plugins.notification.NotificationPlugin

class MyNotificationPlugin implements NotificationPlugin {
    
    static description = [
        name: 'my-notification',
        title: 'My Notification Plugin',
        description: 'Sends notifications via custom method'
    ]
    
    static configuration = [
        webhook_url: [
            title: 'Webhook URL',
            description: 'URL for webhook notifications',
            required: true
        ],
        timeout: [
            title: 'Timeout',
            description: 'Request timeout in seconds',
            type: 'Integer',
            defaultValue: 30
        ]
    ]
    
    boolean postNotification(String trigger, Map executionData, Map config) {
        // 通知ロジックの実装
        def webhookUrl = config.webhook_url
        def timeout = config.timeout as Integer
        
        // HTTP POST request implementation
        return true
    }
}
```

### 5.3 プロパティ検証

```groovy
// 検証ロジック付きプロパティ定義
static configuration = [
    phone_number: [
        title: "Phone number",
        description: "10-digit phone number",
        required: true,
        validation: { value ->
            def cleaned = value.replaceAll(/[^\d]/, '')
            return cleaned ==~ /^\d{10}$/
        }
    ]
]
```

---

## 6. UI Plugin開発

UI PluginはScript Pluginと類似した`ui`プラグインタイプでサポートされています。これらのプラグインは、RundeckのWebインターフェースをカスタマイズするために使用されます。

詳細については、UI Pluginsドキュメントを参照してください。

---

## 7. 国際化（i18n）対応

### 7.1 有効化条件
- **Javaプラグイン**: `Rundeck-Plugin-Version: 1.2`以上
- **スクリプトプラグイン**: `rundeckPluginVersion: 1.2`以上
- **Groovyプラグイン**: Rundeck 2.6.10以降で自動サポート

### 7.2 メッセージファイル構造

#### Javaプラグイン
```
src/main/resources/i18n/
├── messages.properties           # デフォルト（英語）
├── messages_ja.properties        # 日本語
├── messages_ko.properties        # 韓国語
└── messages_zh.properties        # 中国語
```

#### スクリプト/Groovyプラグイン
```
resources/i18n/
├── messages.properties           # デフォルト（英語）
├── messages_ja.properties        # 日本語
├── messages_ko.properties        # 韓国語
├── messages_zh_CN.properties     # 中国語（簡体字）
└── messages_zh_TW.properties     # 中国語（繁体字）
```

### 7.3 メッセージファイル例

#### 基本的なメッセージファイル
```properties
# messages.properties
plugin.title=My Plugin
plugin.description=A sample plugin for demonstration
property.host.title=Host
property.host.description=Target host address
error.connection.failed=Connection to {0} failed
error.authentication.failed=Authentication failed for user {0}

# messages_ja.properties  
plugin.title=マイプラグイン
plugin.description=デモンストレーション用のサンプルプラグイン
property.host.title=ホスト
property.host.description=対象ホストアドレス
error.connection.failed={0} への接続に失敗しました
error.authentication.failed=ユーザー {0} の認証に失敗しました
```

#### スクリプトプラグイン用詳細例
```properties
# messages.properties
plugin.title=SSH Script Executor
plugin.description=Execute commands via SSH connection
config.host.title=Target Host
config.host.description=Hostname or IP address of target server
config.port.title=Port Number
config.port.description=SSH connection port (default: 22)
config.auth_method.title=Authentication Method
config.auth_method.description=Choose authentication method
validation.port.invalid=Port must be between 1 and 65535
validation.host.required=Host is required

# messages_ja.properties
plugin.title=SSH スクリプト実行
plugin.description=SSH接続経由でコマンドを実行します
config.host.title=対象ホスト
config.host.description=対象サーバーのホスト名またはIPアドレス
config.port.title=ポート番号
config.port.description=SSH接続ポート（デフォルト: 22）
config.auth_method.title=認証方式
config.auth_method.description=認証方式を選択してください
validation.port.invalid=ポートは1から65535の間で指定してください
validation.host.required=ホストの指定は必須です
```

---

## 8. プラグイン配置とインストール

### 8.1 配置ディレクトリ
- **Launcher**: `$RDECK_BASE/libext`
- **RPM/DEB**: `/var/lib/rundeck/libext`

### 8.2 利用可能ライブラリ
システムで利用可能なライブラリは以下で確認可能：
```bash
ls /var/lib/rundeck/lib
```

### 8.3 プラグインのロード確認
```bash
# プラグインが正しくロードされているかを確認
grep -i "plugin" /var/log/rundeck/rundeck.log
```

---

## 9. 開発ベストプラクティス

### 9.1 エラーハンドリング

#### Java プラグイン
```java
@Override
public NodeExecutorResult executeCommand(ExecutionContext context, 
                                       String[] command, 
                                       INodeEntry node) throws NodeExecutorException {
    try {
        // プラグインロジック
        return new NodeExecutorResultImpl(0, "Success", null);
    } catch (Exception e) {
        throw new NodeExecutorException("Plugin execution failed: " + e.getMessage(), e);
    }
}
```

#### スクリプトプラグイン
```bash
#!/bin/bash
# エラーハンドリングの例

set -euo pipefail  # エラー時に即座に終了

# 関数でのエラーハンドリング
execute_command() {
    local cmd="$1"
    local retries=3
    local count=0
    
    while [ $count -lt $retries ]; do
        if timeout 30 eval "$cmd"; then
            return 0
        fi
        count=$((count + 1))
        echo "Command failed, retry $count/$retries" >&2
        sleep 5
    done
    
    echo "Command failed after $retries attempts" >&2
    return 1
}
```

### 9.2 ログ出力

#### Java プラグイン
```java
private static final Logger logger = LoggerFactory.getLogger(MyPlugin.class);

public void execute() {
    logger.info("Plugin execution started");
    logger.debug("Debug information: {}", debugInfo);
    logger.error("Error occurred", exception);
}
```

#### スクリプトプラグイン
```bash
# ログレベルに応じた出力
log_debug() {
    if [ "${RD_JOB_LOGLEVEL}" = "DEBUG" ]; then
        echo "[DEBUG] $*" >&2
    fi
}

log_info() {
    echo "[INFO] $*" >&2
}

log_error() {
    echo "[ERROR] $*" >&2
}
```

### 9.3 設定値検証

#### Java プラグイン
```java
@PluginProperty(title = "Port", required = true)
private String port;

private void validateConfiguration() throws PluginException {
    if (port == null || port.trim().isEmpty()) {
        throw new PluginException("Port is required");
    }
    
    try {
        int portNum = Integer.parseInt(port);
        if (portNum < 1 || portNum > 65535) {
            throw new PluginException("Port must be between 1 and 65535");
        }
    } catch (NumberFormatException e) {
        throw new PluginException("Port must be a valid integer");
    }
}
```

#### スクリプトプラグイン
```bash
# 設定値検証関数
validate_config() {
    # ホスト名検証
    if [ -z "${RD_CONFIG_HOST}" ]; then
        echo "Error: Host is required" >&2
        exit 2
    fi
    
    # ポート番号検証
    if ! [[ "${RD_CONFIG_PORT}" =~ ^[0-9]+$ ]] || \
       [ "${RD_CONFIG_PORT}" -lt 1 ] || \
       [ "${RD_CONFIG_PORT}" -gt 65535 ]; then
        echo "Error: Port must be between 1 and 65535" >&2
        exit 2
    fi
    
    # SSH鍵ファイル検証
    if [ -n "${RD_CONFIG_SSH_KEY_PATH}" ] && [ ! -f "${RD_CONFIG_SSH_KEY_PATH}" ]; then
        echo "Error: SSH key file not found: ${RD_CONFIG_SSH_KEY_PATH}" >&2
        exit 2
    fi
}

# メイン処理前に検証を実行
validate_config
```

### 9.4 パフォーマンス考慮事項

#### 共通原則
- 長時間実行処理の進捗表示
- リソースの適切な解放
- メモリ使用量の最適化
- 並行実行時の競合状態回避

#### スクリプト最適化例
```bash
# 並列処理の制御
process_nodes_parallel() {
    local nodes=("$@")
    local max_jobs=5
    local job_count=0
    
    for node in "${nodes[@]}"; do
        if [ $job_count -ge $max_jobs ]; then
            wait  # 一部のジョブ完了を待つ
            job_count=0
        fi
        
        process_single_node "$node" &
        ((job_count++))
    done
    
    wait  # すべてのジョブ完了を待つ
}
```

---

## 10. テストと品質保証

### 10.1 単体テスト例

#### Java プラグイン
```java
@Test
public void testPluginExecution() {
    MyPlugin plugin = new MyPlugin();
    // テスト用の設定
    plugin.setHost("localhost");
    plugin.setPort(22);
    
    ExecutionContext context = mock(ExecutionContext.class);
    INodeEntry node = mock(INodeEntry.class);
    
    NodeExecutorResult result = plugin.executeCommand(context, 
                                                    new String[]{"echo", "test"}, 
                                                    node);
    
    assertEquals(0, result.getResultCode());
    assertEquals("Success", result.getResultData());
}
```

#### スクリプトプラグイン
```bash
#!/bin/bash
# test-suite.sh

# テストフレームワーク関数
assert_equals() {
    local expected="$1"
    local actual="$2"
    local message="$3"
    
    if [ "$expected" = "$actual" ]; then
        echo "PASS: $message"
        return 0
    else
        echo "FAIL: $message (expected: '$expected', actual: '$actual')"
        return 1
    fi
}

# 基本機能テスト
test_basic_execution() {
    export RD_CONFIG_HOST="localhost"
    export RD_CONFIG_PORT="22"
    export RD_CONFIG_USERNAME="testuser"
    
    # プラグインスクリプト実行
    result=$(./contents/script.sh "echo 'test'")
    exit_code=$?
    
    assert_equals "0" "$exit_code" "Basic execution should succeed"
}

# 設定値検証テスト
test_invalid_port() {
    export RD_CONFIG_HOST="localhost"
    export RD_CONFIG_PORT="70000"  # 無効なポート
    
    ./contents/script.sh "echo 'test'" 2>/dev/null
    exit_code=$?
    
    assert_equals "2" "$exit_code" "Invalid port should return exit code 2"
}

# テスト実行
echo "Running plugin tests..."
test_basic_execution
test_invalid_port
echo "Tests completed"
```

### 10.2 統合テスト

#### Docker環境でのテスト
```dockerfile
# Dockerfile.test
FROM rundeck/rundeck:latest

# テスト用プラグインコピー
COPY plugin.zip /home/rundeck/libext/

# テスト用設定
COPY test-config/ /home/rundeck/etc/

# テストスクリプト
COPY test-integration.sh /test-integration.sh
RUN chmod +x /test-integration.sh

CMD ["/test-integration.sh"]
```

```bash
#!/bin/bash
# test-integration.sh

# Rundeckの起動を待つ
wait_for_rundeck() {
    local max_attempts=30
    local attempt=0
    
    while [ $attempt -lt $max_attempts ]; do
        if curl -s http://localhost:4440 > /dev/null; then
            echo "Rundeck is ready"
            return 0
        fi
        echo "Waiting for Rundeck... ($((attempt + 1))/$max_attempts)"
        sleep 10
        ((attempt++))
    done
    
    echo "Rundeck failed to start"
    return 1
}

# プラグインのロード確認
check_plugin_loaded() {
    if grep -q "my-plugin" /home/rundeck/var/logs/rundeck.log; then
        echo "Plugin loaded successfully"
        return 0
    else
        echo "Plugin failed to load"
        return 1
    fi
}

# 統合テスト実行
echo "Starting integration tests..."
wait_for_rundeck
check_plugin_loaded
echo "Integration tests completed"
```

---

## 11. 参考リンク集

### 11.1 公式ドキュメント

#### 基本開発ガイド
- **Plugin Development Overview**
  - URL: https://docs.rundeck.com/docs/developer/01-plugin-development.html
  - 内容: プラグイン開発の基本概念、3つの開発方式（Java/Script/Groovy）の説明

- **Script Plugin Development**
  - URL: https://docs.rundeck.com/docs/developer/02-script-plugin-development.html
  - 内容: スクリプトプラグインの詳細仕様、plugin.yamlの書き方

- **Java Plugin Development**
  - URL: https://docs.rundeck.com/docs/developer/03-java-plugin-development.html
  - 内容: Javaプラグインの開発手順、アノテーション、Maven/Gradle設定

#### サービス別開発ガイド
- **Node Executor Plugins**
  - URL: https://docs.rundeck.com/docs/developer/05-node-executor-plugins.html
  - 内容: ノード実行プラグインの実装方法、SSH接続の代替手段

- **File Copier Plugins**
  - URL: https://docs.rundeck.com/docs/developer/04-file-copier-plugins.html
  - 内容: ファイルコピープラグインの実装、SCP代替の作成方法

- **Resource Model Source Plugins**
  - URL: https://docs.rundeck.com/docs/developer/06-resource-model-source-plugins.html
  - 内容: ノードリソース情報を動的に取得するプラグインの作成

- **Workflow Step Plugins**
  - URL: https://docs.rundeck.com/docs/developer/07-workflow-step-plugins.html
  - 内容: カスタムワークフローステップの実装方法

#### 高度な機能
- **Plugin Localization**
  - URL: https://docs.rundeck.com/docs/developer/plugin-localization.html
  - 内容: プラグインの国際化対応、多言語サポートの実装

- **UI Plugins**
  - URL: https://docs.rundeck.com/docs/developer/ui-plugins.html
  - 内容: WebUIカスタマイズプラグインの開発

- **Notification Plugins**
  - URL: https://docs.rundeck.com/docs/developer/notification-plugins.html
  - 内容: 実行結果通知プラグインの作成方法

### 11.2 プラグイン例とサンプルコード

#### 公式サンプルプラグイン
- **Rundeck Plugin Examples**
  - GitHub: https://github.com/rundeck/rundeck-plugin-examples
  - 内容: 公式のサンプルプラグインコレクション、各サービスタイプの実装例

- **Script Plugin Examples**
  - GitHub: https://github.com/rundeck/rundeck/tree/main/examples/script-plugins
  - 内容: スクリプトプラグインの具体的な実装例

#### コミュニティプラグイン
- **Rundeck Plugins Organization**
  - GitHub: https://github.com/rundeck-plugins
  - 内容: コミュニティが開発した様々なプラグイン集

- **Ansible Plugin**
  - GitHub: https://github.com/rundeck-plugins/ansible-plugin
  - 内容: Ansibleとの統合プラグイン、実用的なスクリプトプラグイン例

- **AWS Plugins**
  - GitHub: https://github.com/rundeck-plugins/rundeck-ec2-nodes-plugin
  - 内容: AWS EC2インスタンス情報を取得するResourceModelSourceプラグイン

### 11.3 開発ツールとテンプレート

#### プラグイン生成ツール
- **Rundeck Plugin Template**
  - GitHub: https://github.com/rundeck/rundeck-plugin-template
  - 内容: プラグイン開発のひな形テンプレート、Gradle設定例

- **Plugin Archetype (Maven)**
  ```bash
  mvn archetype:generate \
    -DarchetypeGroupId=org.rundeck \
    -DarchetypeArtifactId=rundeck-plugin-archetype \
    -DarchetypeVersion=1.0
  ```

#### 開発環境構築
- **Rundeck Docker Development Environment**
  - GitHub: https://github.com/rundeck/docker-zoo
  - 内容: Docker環境でのRundeck開発環境構築

- **Vagrant Development Box**
  - GitHub: https://github.com/rundeck/rundeck-vagrant
  - 内容: Vagrant使用の開発環境セットアップ

### 11.4 API仕様とSDK

#### プラグインAPI仕様
- **Plugin API Documentation**
  - URL: https://docs.rundeck.com/docs/developer/plugin-development-services.html
  - 内容: 各サービスインターフェースの詳細仕様

- **Rundeck Core API JavaDoc**
  - URL: https://rundeck.github.io/rundeck/javadoc/
  - 内容: Rundeck Core APIのJavaDocドキュメント

#### REST API
- **Rundeck API Documentation**
  - URL: https://docs.rundeck.com/docs/api/
  - 内容: REST APIの仕様、プラグインからAPI呼び出しの参考

### 11.5 テストとデバッグ

#### テスト手法
- **Plugin Testing Guide**
  - URL: https://docs.rundeck.com/docs/developer/plugin-testing.html
  - 内容: プラグインのテスト方法、デバッグ手順

- **Integration Testing with Docker**
  - GitHub: https://github.com/rundeck/rundeck/tree/main/test/docker
  - 内容: Docker環境でのプラグイン統合テスト

#### デバッグツール
- **Plugin Development Mode**
  - 設定: `rundeck.plugins.development.mode=true`
  - 効果: プラグインのホットリロード、詳細ログ出力

### 11.6 コミュニティとサポート

#### コミュニティフォーラム
- **Rundeck Community**
  - URL: https://community.rundeck.com/
  - 内容: 質問、ディスカッション、プラグイン共有

- **Rundeck Discord**
  - URL: https://discord.gg/rundeck
  - 内容: リアルタイムチャット、開発者サポート

#### ソースコード
- **Rundeck Main Repository**
  - GitHub: https://github.com/rundeck/rundeck
  - 内容: Rundeckのメインソースコード、プラグインインターフェース

- **Plugin Examples in Core**
  - GitHub: https://github.com/rundeck/rundeck/tree/main/plugins
  - 内容: Rundeck本体に含まれるプラグインの実装例

### 11.7 特定用途向けリソース

#### クラウドプロバイダー統合
- **AWS Plugin Collection**
  - GitHub: https://github.com/rundeck-plugins/rundeck-aws-plugins
  - 内容: AWS各種サービスとの統合プラグイン集

- **Google Cloud Plugin**
  - GitHub: https://github.com/rundeck-plugins/rundeck-google-cloud-plugin
  - 内容: Google Cloud Platform統合プラグイン

- **Azure Plugin**
  - GitHub: https://github.com/rundeck-plugins/rundeck-azure-plugin
  - 内容: Microsoft Azure統合プラグイン

#### DevOpsツール統合
- **Kubernetes Plugin**
  - GitHub: https://github.com/rundeck-plugins/kubernetes
  - 内容: Kubernetesクラスター管理プラグイン

- **Terraform Plugin**
  - GitHub: https://github.com/rundeck-plugins/terraform-plugin
  - 内容: Terraformとの統合プラグイン

- **Jenkins Plugin**
  - GitHub: https://github.com/rundeck-plugins/jenkins-plugin
  - 内容: Jenkins連携プラグイン

#### 監視・通知システム統合
- **Slack Notification Plugin**
  - GitHub: https://github.com/rundeck-plugins/slack-incoming-webhook-plugin
  - 内容: Slack通知プラグインの実装例

- **PagerDuty Plugin**
  - GitHub: https://github.com/rundeck-plugins/pagerduty-notification
  - 内容: PagerDutyとの統合通知プラグイン

- **Datadog Plugin**
  - GitHub: https://github.com/rundeck-plugins/datadog-plugin
  - 内容: Datadog監視との統合プラグイン

### 11.8 学習リソース

#### チュートリアル
- **Plugin Development Tutorial Series**
  - URL: https://docs.rundeck.com/docs/tutorials/plugin-development/
  - 内容: ステップバイステップのプラグイン開発チュートリアル

- **Script Plugin Tutorial**
  - URL: https://docs.rundeck.com/docs/tutorials/script-plugin-development/
  - 内容: スクリプトプラグイン開発の実践的チュートリアル

#### ベストプラクティス
- **Plugin Best Practices**
  - URL: https://docs.rundeck.com/docs/developer/plugin-best-practices.html
  - 内容: プラグイン開発のベストプラクティス、パフォーマンス考慮事項

- **Security Considerations**
  - URL: https://docs.rundeck.com/docs/developer/plugin-security.html
  - 内容: プラグイン開発時のセキュリティ考慮事項

### 11.9 トラブルシューティング

#### よくある問題
- **Plugin Troubleshooting Guide**
  - URL: https://docs.rundeck.com/docs/administration/maintenance/plugin-troubleshooting.html
  - 内容: プラグインのトラブルシューティング手順

- **Common Plugin Issues**
  - URL: https://docs.rundeck.com/docs/developer/plugin-common-issues.html
  - 内容: 頻発する問題とその解決方法

#### ログとデバッグ
- **Plugin Logging Configuration**
  - 設定ファイル: `$RDECK_BASE/server/config/log4j2.properties`
  - 内容: プラグイン用ログ出力の設定方法

### 11.10 リリースと配布

#### プラグインパッケージング
- **Plugin Packaging Guide**
  - URL: https://docs.rundeck.com/docs/developer/plugin-packaging.html
  - 内容: プラグインのパッケージング、配布準備

#### プラグインレジストリ
- **Rundeck Plugin Registry**
  - URL: https://plugins.rundeck.com/
  - 内容: 公式プラグインレジストリ、プラグイン検索・ダウンロード

- **Plugin Submission Guidelines**
  - URL: https://docs.rundeck.com/docs/developer/plugin-submission.html
  - 内容: プラグインをレジストリに登録する手順

---

## まとめ

この完全ガイドは、Rundeckプラグイン開発のあらゆる側面をカバーしています。Java、スクリプト、Groovyの3つの開発方式すべてに対応し、実装例、ベストプラクティス、トラブルシューティング、豊富な参考リンク集を提供しています。

これらの情報を活用することで、生成AIが効率的で高品質なRundeckプラグインを開発できるようになります。プラグインの種類や用途に応じて、適切な開発方式を選択し、このガイドを参考に実装を進めてください。
