# 📚 Bitbucket Pipelines 完全リファレンスガイド

## 1. 🎯 概要と基本概念

### 1.1 Bitbucket Pipelinesとは
**Bitbucket Pipelines**は、Bitbucket Cloudに統合されたCI/CDサービスです。リポジトリ内の設定ファイルに基づいて、コードの自動ビルド、テスト、デプロイを実行します。

**主要特徴:**
- クラウドベースのコンテナ実行環境
- YAMLベースの設定（`bitbucket-pipelines.yml`）
- Dockerコンテナによる隔離された実行環境
- リポジトリルートに配置される設定ファイル

### 1.2 動作原理
```yaml
# 基本的な動作フロー
1. コードプッシュ/プルリクエスト
   ↓
2. パイプライントリガー
   ↓
3. Dockerコンテナ起動
   ↓
4. ステップ実行
   ↓
5. 結果レポート
```

## 2. ⚙️ 設定ファイル構造

### 2.1 bitbucket-pipelines.yml基本構造
```yaml
# 最小構成例
image: node:16

pipelines:
  default:
    - step:
        name: Build and Test
        script:
          - npm install
          - npm test
```

### 2.2 主要設定要素

#### **グローバルオプション**
```yaml
# グローバル設定
image: atlassian/default-image:3
options:
  docker: true
  size: 2x
  max-time: 60

definitions:
  services:
    postgres:
      image: postgres:13
  caches:
    npm: $HOME/.npm
```

#### **パイプライン種別**
```yaml
pipelines:
  default:           # デフォルトパイプライン
    - step: ...
  
  branches:          # ブランチ別パイプライン
    master:
      - step: ...
    develop:
      - step: ...
    
  pull-requests:     # プルリクエスト用
    '**':
      - step: ...
      
  custom:            # 手動実行パイプライン
    deploy-production:
      - step: ...
      
  tags:              # タグプッシュ時
    release-*:
      - step: ...
```

## 3. 🔧 コア機能詳細

### 3.1 ステップオプション
```yaml
- step:
    name: ステップ名
    image: カスタムイメージ:タグ     # ステップ固有のイメージ
    script:                          # 実行コマンド
      - echo "実行するコマンド"
    caches:                         # キャッシュ利用
      - node
      - docker
    services:                       # サービスコンテナ
      - postgres
      - redis
    artifacts:                      # アーティファクト保存
      - target/**
      - reports/**
    after-script:                   # 後処理（必ず実行）
      - echo "クリーンアップ"
    size: 2x                        # リソースサイズ
    max-time: 30                    # タイムアウト（分）
    deployment: production          # デプロイメント環境
    trigger: manual                 # 手動トリガー
```

### 3.2 並列実行
```yaml
pipelines:
  default:
    - parallel:
        - step:
            name: Unit Tests
            script:
              - npm run test:unit
        - step:
            name: Integration Tests
            script:
              - npm run test:integration
        - step:
            name: Lint
            script:
              - npm run lint
```

### 3.3 ステージ実行
```yaml
pipelines:
  default:
    - stage:
        name: Build Stage
        steps:
          - step:
              name: Build Frontend
              script:
                - npm run build:frontend
          - step:
              name: Build Backend
              script:
                - npm run build:backend
    - stage:
        name: Test Stage
        trigger: manual
        steps:
          - step:
              name: Run Tests
              script:
                - npm test
```

## 4. 🚀 デプロイメント機能

### 4.1 デプロイメント環境
```yaml
pipelines:
  branches:
    master:
      - step:
          name: Deploy to Production
          deployment: production
          script:
            - echo "Deploying to production"
            
# 環境定義
deployments:
  production:
    environment-type: production
  staging:
    environment-type: staging
  test:
    environment-type: test
```

### 4.2 主要デプロイメントパターン

#### **AWS デプロイメント**
```yaml
# S3デプロイ
- step:
    name: Deploy to S3
    script:
      - pipe: atlassian/aws-s3-deploy:1.1.0
        variables:
          AWS_ACCESS_KEY_ID: $AWS_ACCESS_KEY_ID
          AWS_SECRET_ACCESS_KEY: $AWS_SECRET_ACCESS_KEY
          AWS_DEFAULT_REGION: 'us-east-1'
          S3_BUCKET: 'my-bucket'
          LOCAL_PATH: 'dist'

# EKSデプロイ
- step:
    name: Deploy to EKS
    script:
      - pipe: atlassian/aws-eks-kubectl-run:2.2.0
        variables:
          AWS_ACCESS_KEY_ID: $AWS_ACCESS_KEY_ID
          AWS_SECRET_ACCESS_KEY: $AWS_SECRET_ACCESS_KEY
          AWS_DEFAULT_REGION: $AWS_DEFAULT_REGION
          CLUSTER_NAME: 'my-cluster'
          KUBECTL_COMMAND: 'apply'
          RESOURCE_PATH: './k8s'
```

#### **その他のプラットフォーム**
```yaml
# Google Cloud
- pipe: atlassian/google-app-engine-deploy:0.7.3

# Azure
- pipe: microsoft/azure-web-apps-deploy:1.0.3

# Heroku
- pipe: atlassian/heroku-deploy:1.1.4

# Firebase
- pipe: atlassian/firebase-deploy:1.1.0
```

## 5. 🔐 変数とシークレット管理

### 5.1 変数の種類と優先順位
```yaml
# 優先順位（高→低）
1. ステップ変数
2. パイプライン変数  
3. デプロイメント変数
4. リポジトリ変数
5. ワークスペース変数
6. デフォルト変数
```

### 5.2 変数の定義方法
```yaml
pipelines:
  default:
    - step:
        script:
          - echo $MY_VARIABLE        # リポジトリ変数
          - export CUSTOM_VAR=value   # ローカル変数
        variables:
          - name: STEP_VAR
            default: default_value
            allowed-values:           # 手動実行時の選択肢
              - option1
              - option2
```

### 5.3 セキュアな変数
```yaml
# Bitbucket UIで設定（マスクされる）
- セキュア変数は自動的にログでマスク
- 4文字以上の値のみマスク可能
- Base64エンコードされた値も自動マスク
```

## 6. 🎯 高度な機能

### 6.1 Docker機能
```yaml
options:
  docker: true

pipelines:
  default:
    - step:
        script:
          # Dockerコマンド実行
          - docker build -t my-app .
          - docker run my-app
        services:
          - docker
```

### 6.2 キャッシュ管理
```yaml
definitions:
  caches:
    npm: ~/.npm
    maven: ~/.m2/repository
    pip: ~/.cache/pip
    docker: /var/lib/docker
    custom-cache: ./my-cache-dir

pipelines:
  default:
    - step:
        caches:
          - npm
          - custom-cache
        script:
          - npm install
```

### 6.3 アーティファクト管理
```yaml
pipelines:
  default:
    - step:
        name: Build
        artifacts:
          - dist/**          # 次のステップに渡される
          - reports/**
          download: false    # ダウンロード可能にしない
          paths:
            - target/**.jar
    - step:
        name: Deploy
        script:
          - ls dist/         # 前のステップのアーティファクトを利用
```

### 6.4 サービスコンテナ
```yaml
definitions:
  services:
    postgres:
      image: postgres:13
      variables:
        POSTGRES_DB: testdb
        POSTGRES_USER: test
        POSTGRES_PASSWORD: test
    redis:
      image: redis:6
    elasticsearch:
      image: elasticsearch:7.9.3
      memory: 512
      variables:
        discovery.type: single-node

pipelines:
  default:
    - step:
        services:
          - postgres
          - redis
        script:
          - psql -h localhost -U test -d testdb
```

### 6.5 SSH認証
```yaml
pipelines:
  default:
    - step:
        script:
          # リポジトリSSHキー使用
          - ssh user@server.com "deploy command"
          # 複数のSSHキー管理
          - mkdir -p ~/.ssh
          - echo "$CUSTOM_SSH_KEY" > ~/.ssh/custom_key
          - chmod 600 ~/.ssh/custom_key
          - ssh -i ~/.ssh/custom_key user@server.com
```

## 7. 🔄 ランタイムと実行環境

### 7.1 Runtime v3（最新）
```yaml
options:
  runtime:
    cloud:
      atlassian-ip-ranges: true    # Atlassian IPレンジ使用

pipelines:
  default:
    - step:
        runtime:
          cloud:
            size: 4x                # 大規模リソース
```

### 7.2 セルフホストランナー
```yaml
pipelines:
  default:
    - step:
        runs-on:
          - self.hosted
          - linux.shell
        script:
          - echo "セルフホストランナーで実行"
```

## 8. 🧩 Pipes（再利用可能コンポーネント）

### 8.1 Pipeの使用
```yaml
script:
  - pipe: atlassian/slack-notify:2.0.0
    variables:
      WEBHOOK_URL: $SLACK_WEBHOOK
      MESSAGE: "デプロイ完了"
      
  - pipe: atlassian/aws-cloudformation-deploy:0.10.0
    variables:
      AWS_ACCESS_KEY_ID: $AWS_ACCESS_KEY_ID
      AWS_SECRET_ACCESS_KEY: $AWS_SECRET_ACCESS_KEY
      AWS_DEFAULT_REGION: 'us-east-1'
      STACK_NAME: 'my-stack'
      TEMPLATE: 'template.yml'
```

### 8.2 カスタムPipe作成
```yaml
# pipe.yml
name: Custom Pipe
image: alpine:3.12
variables:
  - name: PARAM1
    default: default_value
script:
  - echo "Custom pipe execution"
```

## 9. 🎯 ベストプラクティス

### 9.1 パフォーマンス最適化
```yaml
# 1. キャッシュの活用
caches:
  - node
  - docker

# 2. 並列実行
- parallel:
    - step: ...
    - step: ...

# 3. 条件付き実行
- step:
    condition:
      changesets:
        includePaths:
          - "src/**"

# 4. アーティファクトの最小化
artifacts:
  - dist/*.min.js    # 必要最小限のファイルのみ
```

### 9.2 セキュリティ
```yaml
# 1. セキュア変数の使用
script:
  - echo $SECURE_VAR    # UIで設定

# 2. ブランチ制限
pipelines:
  branches:
    master:
      - step:
          deployment: production

# 3. 手動承認
- step:
    trigger: manual
    deployment: production
```

### 9.3 エラーハンドリング
```yaml
- step:
    script:
      - set -e              # エラー時に停止
      - command1
      - command2 || true    # エラーを無視
    after-script:          # 必ず実行
      - cleanup_command
```

## 10. 🔍 トラブルシューティング

### 10.1 一般的な問題と解決策

#### **ビルド時間の最適化**
```yaml
# 問題: ビルドが遅い
# 解決策:
1. キャッシュの活用
2. 並列実行の利用
3. 不要なステップの削除
4. イメージサイズの最小化
```

#### **メモリ不足**
```yaml
# 問題: メモリ不足エラー
# 解決策:
options:
  size: 2x    # または 4x, 8x
  
services:
  postgres:
    memory: 1024    # サービスメモリ制限
```

#### **アーティファクト問題**
```yaml
# 問題: アーティファクトが見つからない
# 解決策:
artifacts:
  - "**/*.jar"      # ワイルドカード使用
  - "!**/test/**"   # 除外パターン
```

## 11. 📊 YAML高度な機能

### 11.1 YAMLアンカー
```yaml
definitions:
  steps:
    - &build-step
      step:
        name: Build
        script:
          - npm install
          - npm run build

pipelines:
  branches:
    master:
      - <<: *build-step
        deployment: production
    develop:
      - <<: *build-step
        deployment: staging
```

### 11.2 Globパターン
```yaml
pipelines:
  branches:
    'feature/*':        # feature/で始まるブランチ
      - step: ...
    'release-*':        # release-で始まるブランチ
      - step: ...
    '**':              # 全てのブランチ
      - step: ...
```

## 12. 🔄 移行ガイド

### 12.1 他のCI/CDからの移行

#### **Jenkinsからの移行**
```yaml
# Jenkinsfile相当
pipelines:
  default:
    - stage:
        name: Build
        steps:
          - step:
              name: Compile
              script:
                - mvn compile
```

#### **GitHub Actionsからの移行**
```yaml
# GitHub Actions相当
pipelines:
  pull-requests:    # on: pull_request相当
    '**':
      - step:
          name: Test
          script:
            - npm test
```

## 13. 🌐 統合機能

### 13.1 Jira統合
```yaml
- step:
    script:
      # Jiraチケット自動更新
      - echo "Deployed to production [JIRA-123]"
```

### 13.2 Slack通知
```yaml
- pipe: atlassian/slack-notify:2.0.0
  variables:
    WEBHOOK_URL: $SLACK_WEBHOOK
    MESSAGE: |
      :rocket: デプロイ完了
      ブランチ: $BITBUCKET_BRANCH
      コミット: $BITBUCKET_COMMIT
```

### 13.3 外部サービス連携
```yaml
# OpenID Connect (OIDC)
- step:
    oidc: true
    script:
      - export AWS_WEB_IDENTITY_TOKEN_FILE=$(pwd)/web-identity-token
      - echo $BITBUCKET_STEP_OIDC_TOKEN > $AWS_WEB_IDENTITY_TOKEN_FILE
      - aws sts assume-role-with-web-identity
```

## 14. 📈 モニタリングとレポート

### 14.1 テストレポート
```yaml
- step:
    script:
      - npm test -- --coverage
    artifacts:
      - coverage/**
      - test-results/**
    after-script:
      - pipe: atlassian/bitbucket-upload-file:0.3.2
        variables:
          FILENAME: "coverage/index.html"
```

### 14.2 ビルド統計
```yaml
# パイプライン実行時間、成功率、失敗率などは
# Bitbucket UIのInsightsセクションで確認可能
```

## 15. 🔮 動的パイプライン

### 15.1 動的生成
```yaml
pipelines:
  default:
    - step:
        name: Generate Pipeline
        script:
          - python generate_pipeline.py > generated-pipeline.yml
    - step:
        name: Execute Generated Pipeline
        script:
          - cat generated-pipeline.yml
```

## 16. 📝 参照URL一覧

### 公式ドキュメント
- [Bitbucket Pipelines メインページ](https://support.atlassian.com/bitbucket-cloud/docs/build-test-and-deploy-with-pipelines/)
- [Bitbucket Pipelines スタートガイド](https://support.atlassian.com/bitbucket-cloud/docs/get-started-with-bitbucket-pipelines/)
- [設定リファレンス](https://support.atlassian.com/bitbucket-cloud/docs/bitbucket-pipelines-configuration-reference/)
- [デプロイメントガイド](https://support.atlassian.com/bitbucket-cloud/docs/access-pipelines-deployment-guides/)

### 機能別ドキュメント
- [Docker イメージオプション](https://support.atlassian.com/bitbucket-cloud/docs/docker-image-options/)
- [ビルド環境としてのDockerイメージ](https://support.atlassian.com/bitbucket-cloud/docs/use-docker-images-as-build-environments/)
- [データベースとサービスコンテナ](https://support.atlassian.com/bitbucket-cloud/docs/databases-and-service-containers/)
- [Git クローン動作](https://support.atlassian.com/bitbucket-cloud/docs/git-clone-behavior/)
- [Runtime v3の有効化と使用](https://support.atlassian.com/bitbucket-cloud/docs/enable-and-use-runtime-v3/)

### 高度な機能
- [並列ステップオプション](https://support.atlassian.com/bitbucket-cloud/docs/parallel-step-options/)
- [子パイプラインステップオプション](https://support.atlassian.com/bitbucket-cloud/docs/child-pipeline-step-options/)
- [動的パイプライン](https://support.atlassian.com/bitbucket-cloud/docs/dynamic-pipelines/)
- [パイプライン開始条件](https://support.atlassian.com/bitbucket-cloud/docs/pipeline-start-conditions/)
- [ステップオプション](https://support.atlassian.com/bitbucket-cloud/docs/step-options/)
- [ステージオプション](https://support.atlassian.com/bitbucket-cloud/docs/stage-options/)

### セキュリティと変数
- [変数とシークレット](https://support.atlassian.com/bitbucket-cloud/docs/variables-and-secrets/)
- [Bitbucket PipelinesでのSSHキー使用](https://support.atlassian.com/bitbucket-cloud/docs/using-ssh-keys-in-bitbucket-pipelines/)

### 最適化とキャッシュ
- [キャッシュ管理](https://support.atlassian.com/bitbucket-cloud/docs/cache-dependencies/)
- [キャッシュとサービスコンテナ定義](https://support.atlassian.com/bitbucket-cloud/docs/cache-and-service-container-definitions/)
- [アーティファクトの使用](https://support.atlassian.com/bitbucket-cloud/docs/use-artifacts-in-steps/)

### Pipesと統合
- [Bitbucket PipelinesでのPipes使用](https://support.atlassian.com/bitbucket-cloud/docs/use-pipes-in-bitbucket-pipelines/)
- [統合](https://support.atlassian.com/bitbucket-cloud/docs/integrations/)

### ユーティリティ機能
- [YAMLアンカー](https://support.atlassian.com/bitbucket-cloud/docs/yaml-anchors/)
- [Globパターン](https://support.atlassian.com/bitbucket-cloud/docs/use-glob-patterns-on-the-pipelines-yaml-file/)
- [リポジトリへのプッシュバック](https://support.atlassian.com/bitbucket-cloud/docs/push-back-to-your-repository/)
- [パイプライン設定の共有](https://support.atlassian.com/bitbucket-cloud/docs/share-pipelines-configurations/)

### 実行環境
- [ランナー](https://support.atlassian.com/bitbucket-cloud/docs/runners/)
- [パイプライントリガー](https://support.atlassian.com/bitbucket-cloud/docs/pipeline-triggers/)
- [デプロイメント](https://support.atlassian.com/bitbucket-cloud/docs/deployments/)

### テストと品質
- [テスト](https://support.atlassian.com/bitbucket-cloud/docs/testing/)

### 移行ガイド
- [Bitbucket Pipelinesへの移行](https://support.atlassian.com/bitbucket-cloud/docs/migrate-to-bitbucket-pipelines/)
