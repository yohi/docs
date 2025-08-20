# Bitbucket Pipelines 包括的リファレンス

## 目次
1. [概要](#概要)
2. [基本設定](#基本設定)
3. [設定リファレンス](#設定リファレンス)
4. [実行環境](#実行環境)
5. [デプロイメント](#デプロイメント)
6. [高度な機能](#高度な機能)
7. [セキュリティ・認証](#セキュリティ認証)
8. [最適化・効率化](#最適化効率化)
9. [テスト・品質保証](#テスト品質保証)
10. [統合・拡張](#統合拡張)
11. [トラブルシューティング](#トラブルシューティング)
12. [参照URL](#参照url)

---

## 概要

### Bitbucket Pipelinesとは
- **統合CI/CDサービス**: Bitbucket Cloudに組み込まれた継続的インテグレーション・デプロイサービス
- **クラウドコンテナ実行**: コード変更に基づいて自動的にクラウド上でコンテナを作成・実行
- **YAML設定ベース**: `bitbucket-pipelines.yml`ファイルでパイプライン定義
- **Docker基盤**: Linuxベース、Dockerイメージを使用した実行環境

### 基本概念
- **Pipeline**: 一連のビルド・テスト・デプロイプロセス
- **Step**: パイプライン内の個別の実行単位
- **Stage**: 複数のステップをグループ化した単位
- **Pipe**: 再利用可能な事前定義されたコンポーネント
- **Artifact**: ビルド成果物（ファイル、パッケージ等）

---

## 基本設定

### ファイル構造
```yaml
# bitbucket-pipelines.yml (リポジトリルートに配置)
image: node:18

pipelines:
  default:
    - step:
        name: Build and Test
        script:
          - npm install
          - npm test
  branches:
    main:
      - step:
          name: Deploy to Production
          deployment: production
          script:
            - npm run build
            - npm run deploy
```

### 基本要素
- **image**: デフォルトのDockerイメージ指定
- **pipelines**: パイプライン定義のルートセクション
- **default**: デフォルトブランチでの実行内容
- **branches**: 特定ブランチでの実行内容
- **step**: 個別の実行ステップ
- **script**: 実行するコマンド列

---

## 設定リファレンス

### グローバルオプション
```yaml
# 全体設定
image: ubuntu:20.04
clone:
  depth: full
  lfs: true
options:
  max-time: 120
  size: 2x
```

### パイプライン開始条件
```yaml
pipelines:
  # プルリクエスト時
  pull-requests:
    '**':
      - step:
          name: PR Validation
          script:
            - echo "Validating PR"
  
  # タグ時
  tags:
    'release-*':
      - step:
          name: Release Build
          script:
            - echo "Building release"
  
  # 手動トリガー
  custom:
    release-to-staging:
      - step:
          name: Deploy to Staging
          deployment: staging
          script:
            - echo "Deploying to staging"
```

### ステップオプション
```yaml
step:
  name: "Build Application"
  image: node:18
  size: 2x
  max-time: 30
  caches:
    - node
  artifacts:
    - dist/**
  services:
    - postgres
  script:
    - npm install
    - npm run build
  condition:
    changesets:
      includePaths:
        - "src/**"
```

---

## 実行環境

### Dockerイメージ設定
```yaml
# デフォルトイメージ
image: node:18

pipelines:
  default:
    - step:
        # ステップ固有のイメージ
        image: python:3.9
        script:
          - python --version
    
    - step:
        # カスタムイメージ
        image: 
          name: myregistry/myimage:latest
          username: $DOCKER_USERNAME
          password: $DOCKER_PASSWORD
        script:
          - echo "Using custom image"
```

### サービスコンテナ
```yaml
definitions:
  services:
    postgres:
      image: postgres:13
      environment:
        POSTGRES_DB: testdb
        POSTGRES_USER: testuser
        POSTGRES_PASSWORD: testpass
    redis:
      image: redis:6
    mysql:
      image: mysql:8.0
      environment:
        MYSQL_DATABASE: testdb
        MYSQL_ROOT_PASSWORD: rootpass

pipelines:
  default:
    - step:
        name: Test with Database
        services:
          - postgres
          - redis
        script:
          - echo "Testing with services"
```

### ランタイム設定
```yaml
# Runtime v3の有効化
options:
  runtime:
    cloud:
      runtime: v3

# カスタムランナー
pipelines:
  default:
    - step:
        runs-on:
          - 'linux.medium'
        script:
          - echo "Running on custom runner"
```

---

## デプロイメント

### AWS デプロイメント
```yaml
# S3デプロイ
- step:
    name: Deploy to S3
    deployment: production
    script:
      - pipe: atlassian/aws-s3-deploy:1.1.0
        variables:
          AWS_ACCESS_KEY_ID: $AWS_ACCESS_KEY_ID
          AWS_SECRET_ACCESS_KEY: $AWS_SECRET_ACCESS_KEY
          AWS_DEFAULT_REGION: us-west-2
          S3_BUCKET: my-website-bucket
          LOCAL_PATH: dist

# EKS (Kubernetes) デプロイ
- step:
    name: Deploy to EKS
    deployment: production
    script:
      - pipe: atlassian/aws-eks-kubectl-run:2.2.0
        variables:
          AWS_ACCESS_KEY_ID: $AWS_ACCESS_KEY_ID
          AWS_SECRET_ACCESS_KEY: $AWS_SECRET_ACCESS_KEY
          AWS_DEFAULT_REGION: us-west-2
          CLUSTER_NAME: my-cluster
          KUBECTL_COMMAND: 'apply'
          RESOURCE_PATH: 'k8s/'
```

### その他プラットフォーム
```yaml
# Heroku デプロイ
- step:
    name: Deploy to Heroku
    deployment: production
    script:
      - pipe: atlassian/heroku-deploy:1.1.1
        variables:
          HEROKU_API_KEY: $HEROKU_API_KEY
          HEROKU_APP_NAME: my-app

# Firebase デプロイ
- step:
    name: Deploy to Firebase
    script:
      - pipe: atlassian/firebase-deploy:1.0.0
        variables:
          FIREBASE_TOKEN: $FIREBASE_TOKEN
          PROJECT_ID: my-project

# npm パッケージ公開
- step:
    name: Publish to npm
    script:
      - pipe: atlassian/npm-publish:0.3.1
        variables:
          NPM_TOKEN: $NPM_TOKEN
```

---

## 高度な機能

### 並列実行
```yaml
pipelines:
  default:
    - parallel:
        - step:
            name: Unit Tests
            script:
              - npm run test:unit
        - step:
            name: Lint
            script:
              - npm run lint
        - step:
            name: Security Scan
            script:
              - npm audit
    
    - step:
        name: Integration Tests
        script:
          - npm run test:integration
```

### 動的パイプライン
```yaml
pipelines:
  default:
    - step:
        name: Generate Pipeline
        script:
          - echo "Generating dynamic pipeline"
          - echo "image: node:18" > generated-pipeline.yml
          - echo "pipelines:" >> generated-pipeline.yml
          - echo "  default:" >> generated-pipeline.yml
          - echo "    - step:" >> generated-pipeline.yml
          - echo "        script:" >> generated-pipeline.yml
          - echo "          - echo 'Dynamic step'" >> generated-pipeline.yml
        artifacts:
          - generated-pipeline.yml
    
    - step:
        name: Run Dynamic Pipeline
        script:
          - pipe: atlassian/trigger-pipeline:5.0.0
            variables:
              BITBUCKET_USERNAME: $BITBUCKET_USERNAME
              BITBUCKET_APP_PASSWORD: $BITBUCKET_APP_PASSWORD
              PIPELINE_FILE: generated-pipeline.yml
```

### 子パイプライン
```yaml
# メインパイプライン
pipelines:
  default:
    - step:
        name: Trigger Child Pipeline
        script:
          - pipe: atlassian/trigger-pipeline:5.0.0
            variables:
              BITBUCKET_USERNAME: $BITBUCKET_USERNAME
              BITBUCKET_APP_PASSWORD: $BITBUCKET_APP_PASSWORD
              REPOSITORY: myteam/child-repo
              REF_TYPE: branch
              REF_NAME: main
              CUSTOM_PIPELINE_NAME: child-pipeline
```

---

## セキュリティ・認証

### 変数とシークレット
```yaml
# リポジトリ変数の使用
pipelines:
  default:
    - step:
        name: Use Variables
        script:
          - echo "Environment: $ENVIRONMENT"
          - echo "API URL: $API_URL"
          - echo "Secret: $SECRET_TOKEN" # セキュアな変数

# ステップレベル変数
- step:
    name: Custom Variables
    script:
      - export CUSTOM_VAR="value"
      - echo $CUSTOM_VAR
```

### SSH認証
```yaml
pipelines:
  default:
    - step:
        name: SSH Access
        script:
          # SSH鍵の設定（Bitbucket設定で事前に登録）
          - ssh-keyscan -H github.com >> ~/.ssh/known_hosts
          - git clone git@github.com:user/private-repo.git
          
          # カスタムSSH鍵
          - echo $CUSTOM_SSH_KEY | base64 -d > ~/.ssh/custom_key
          - chmod 600 ~/.ssh/custom_key
          - ssh -i ~/.ssh/custom_key user@server
```

### OpenID Connect (OIDC)
```yaml
# AWS OIDC認証
- step:
    name: Deploy with OIDC
    oidc: true
    script:
      - export AWS_ROLE_ARN="arn:aws:iam::123456789012:role/BitbucketRole"
      - export AWS_WEB_IDENTITY_TOKEN_FILE="/opt/atlassian/pipelines/agent/tmp/web-identity-token"
      - aws sts get-caller-identity
      - aws s3 ls
```

---

## 最適化・効率化

### キャッシュ設定
```yaml
definitions:
  caches:
    nodemodules: node_modules
    gradlewrapper: ~/.gradle/wrapper
    gradlecache: ~/.gradle/caches
    custom-cache: ~/.custom

pipelines:
  default:
    - step:
        name: Build with Cache
        caches:
          - nodemodules
          - gradlewrapper
        script:
          - npm install  # node_modulesがキャッシュされる
          - ./gradlew build  # Gradleキャッシュを使用
```

### アーティファクト管理
```yaml
pipelines:
  default:
    - step:
        name: Build
        script:
          - npm run build
        artifacts:
          - dist/**
          - build/reports/**
    
    - step:
        name: Deploy
        script:
          - ls -la dist/  # 前のステップのアーティファクトを使用
          - aws s3 sync dist/ s3://my-bucket/
```

### Git クローン最適化
```yaml
clone:
  depth: 50        # 浅いクローン
  lfs: true        # Git LFS有効化
  enabled: false   # クローンを無効化（カスタムクローン時）

pipelines:
  default:
    - step:
        name: Custom Clone
        script:
          - git clone --depth 1 $BITBUCKET_GIT_SSH_ORIGIN .
```

### YAML アンカー
```yaml
definitions:
  steps:
    - step: &build-step
        name: Build
        image: node:18
        caches:
          - node
        script:
          - npm install
          - npm run build
    
    - step: &test-step
        name: Test
        image: node:18
        caches:
          - node
        script:
          - npm install
          - npm test

pipelines:
  default:
    - <<: *build-step
    - <<: *test-step
  
  branches:
    develop:
      - <<: *build-step
      - <<: *test-step
      - step:
          name: Deploy to Dev
          script:
            - echo "Deploying to development"
```

---

## テスト・品質保証

### テスト実行
```yaml
pipelines:
  default:
    - step:
        name: Unit Tests
        script:
          - npm run test:unit
          - npm run test:coverage
        artifacts:
          - coverage/**
          - test-results.xml
    
    - step:
        name: Integration Tests
        services:
          - postgres
        script:
          - npm run test:integration
          - npm run test:e2e
```

### テストレポート
```yaml
# JUnit形式のテストレポート
- step:
    name: Test with Reports
    script:
      - npm test -- --reporter=mocha-junit-reporter
    artifacts:
      - test-results.xml
    # Bitbucketがテスト結果を自動的に解析・表示
```

### 品質ゲート
```yaml
- step:
    name: Quality Gate
    script:
      - npm run lint
      - npm run test:coverage
      - npm run security:scan
      # カバレッジ閾値チェック
      - if [ $(cat coverage/coverage-summary.json | jq '.total.lines.pct') -lt 80 ]; then exit 1; fi
```

---

## 統合・拡張

### Pipes の活用
```yaml
# 定義済みPipes
- step:
    name: Slack Notification
    script:
      - pipe: atlassian/slack-notify:2.1.0
        variables:
          WEBHOOK_URL: $SLACK_WEBHOOK
          MESSAGE: 'Build completed successfully!'

# カスタムPipe
- step:
    name: Custom Pipe
    script:
      - pipe: docker://myregistry/custom-pipe:latest
        variables:
          CUSTOM_VAR: $CUSTOM_VALUE
```

### 外部サービス連携
```yaml
# Jira連携
- step:
    name: Update Jira
    script:
      - pipe: atlassian/jira-transition:1.0.0
        variables:
          JIRA_BASE_URL: $JIRA_BASE_URL
          JIRA_USER_EMAIL: $JIRA_USER_EMAIL
          JIRA_API_TOKEN: $JIRA_API_TOKEN
          ISSUE_KEYS: 'PROJ-123,PROJ-124'
          TRANSITION_NAME: 'Done'

# SonarQube連携
- step:
    name: SonarQube Analysis
    script:
      - pipe: sonarsource/sonarqube-scan:1.0.0
        variables:
          SONAR_HOST_URL: $SONAR_HOST_URL
          SONAR_TOKEN: $SONAR_TOKEN
```

### セルフホストランナー
```yaml
# カスタムランナー設定
pipelines:
  default:
    - step:
        runs-on:
          - self.hosted
          - linux
          - large
        script:
          - echo "Running on self-hosted runner"
          - docker --version
          - kubectl version
```

---

## トラブルシューティング

### デバッグ・ログ設定
```yaml
- step:
    name: Debug Step
    script:
      - set -x  # コマンドの詳細出力
      - env | sort  # 環境変数の確認
      - df -h  # ディスク使用量
      - free -m  # メモリ使用量
      - docker images  # Dockerイメージ一覧
```

### エラーハンドリング
```yaml
- step:
    name: Error Handling
    script:
      - set -e  # エラー時即座に終了
      - |
        if ! npm test; then
          echo "Tests failed, sending notification"
          curl -X POST $SLACK_WEBHOOK -d '{"text":"Tests failed!"}'
          exit 1
        fi
```

### 条件付き実行
```yaml
- step:
    name: Conditional Step
    condition:
      changesets:
        includePaths:
          - "src/**"
        excludePaths:
          - "docs/**"
    script:
      - echo "Source code changed, running build"
      - npm run build
```

### Globパターン
```yaml
# ブランチパターン
branches:
  'feature/*':
    - step:
        name: Feature Build
        script:
          - npm run build:dev
  
  'release/**':
    - step:
        name: Release Build
        script:
          - npm run build:prod

# ファイルパターン（条件付き実行）
condition:
  changesets:
    includePaths:
      - "app/**/*.js"
      - "src/**/*.{ts,tsx}"
    excludePaths:
      - "**/*.md"
      - "docs/**"
```

---

## 参照URL

### 基本ドキュメント
- [Build, test, and deploy with Pipelines](https://support.atlassian.com/bitbucket-cloud/docs/build-test-and-deploy-with-pipelines/)
- [Get started with Bitbucket Pipelines](https://support.atlassian.com/bitbucket-cloud/docs/get-started-with-bitbucket-pipelines/)
- [Configure your first pipeline](https://support.atlassian.com/bitbucket-cloud/docs/configure-your-first-pipeline/)

### 設定・構成
- [Bitbucket Pipelines configuration reference](https://support.atlassian.com/bitbucket-cloud/docs/bitbucket-pipelines-configuration-reference/)
- [Global options](https://support.atlassian.com/bitbucket-cloud/docs/global-options/)
- [Pipeline start conditions](https://support.atlassian.com/bitbucket-cloud/docs/pipeline-start-conditions/)
- [Step options](https://support.atlassian.com/bitbucket-cloud/docs/step-options/)
- [Stage options](https://support.atlassian.com/bitbucket-cloud/docs/stage-options/)
- [Use glob patterns on the Pipelines yaml file](https://support.atlassian.com/bitbucket-cloud/docs/use-glob-patterns-on-the-pipelines-yaml-file/)
- [YAML anchors](https://support.atlassian.com/bitbucket-cloud/docs/yaml-anchors/)

### 実行環境
- [Docker image options](https://support.atlassian.com/bitbucket-cloud/docs/docker-image-options/)
- [Use Docker images as build environments](https://support.atlassian.com/bitbucket-cloud/docs/use-docker-images-as-build-environments/)
- [Databases and service containers](https://support.atlassian.com/bitbucket-cloud/docs/databases-and-service-containers/)
- [Enable and use Runtime v3](https://support.atlassian.com/bitbucket-cloud/docs/enable-and-use-runtime-v3/)
- [Runners](https://support.atlassian.com/bitbucket-cloud/docs/runners/)

### デプロイメント
- [Access Pipelines deployment guides](https://support.atlassian.com/bitbucket-cloud/docs/access-pipelines-deployment-guides/)
- [Deployments](https://support.atlassian.com/bitbucket-cloud/docs/deployments/)
- [Deploy on AWS using Bitbucket Pipelines OpenID Connect](https://support.atlassian.com/bitbucket-cloud/docs/deploy-on-aws-using-bitbucket-pipelines-openid-connect/)

### 高度な機能
- [Parallel step options](https://support.atlassian.com/bitbucket-cloud/docs/parallel-step-options/)
- [Child pipeline step options](https://support.atlassian.com/bitbucket-cloud/docs/child-pipeline-step-options/)
- [Dynamic pipelines](https://support.atlassian.com/bitbucket-cloud/docs/dynamic-pipelines/)
- [Pipeline triggers](https://support.atlassian.com/bitbucket-cloud/docs/pipeline-triggers/)

### セキュリティ・認証
- [Variables and secrets](https://support.atlassian.com/bitbucket-cloud/docs/variables-and-secrets/)
- [Using SSH keys in Bitbucket Pipelines](https://support.atlassian.com/bitbucket-cloud/docs/using-ssh-keys-in-bitbucket-pipelines/)

### 最適化
- [Cache dependencies](https://support.atlassian.com/bitbucket-cloud/docs/cache-dependencies/)
- [Cache, service container, and export pipelines definitions](https://support.atlassian.com/bitbucket-cloud/docs/cache-and-service-container-definitions/)
- [Use artifacts in steps](https://support.atlassian.com/bitbucket-cloud/docs/use-artifacts-in-steps/)
- [Git clone behavior](https://support.atlassian.com/bitbucket-cloud/docs/git-clone-behavior/)

### テスト・品質
- [Testing](https://support.atlassian.com/bitbucket-cloud/docs/testing/)

### 統合・拡張
- [Use pipes in Bitbucket Pipelines](https://support.atlassian.com/bitbucket-cloud/docs/use-pipes-in-bitbucket-pipelines/)
- [Integrations](https://support.atlassian.com/bitbucket-cloud/docs/integrations/)
- [Share Pipelines Configurations](https://support.atlassian.com/bitbucket-cloud/docs/share-pipelines-configurations/)

### その他
- [Push back to your repository](https://support.atlassian.com/bitbucket-cloud/docs/push-back-to-your-repository/)
- [Migrate to Bitbucket Pipelines](https://support.atlassian.com/bitbucket-cloud/docs/migrate-to-bitbucket-pipelines/)
