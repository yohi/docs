# Bitbucket Pipelines 詳細リファレンス - 完全ガイド

このドキュメントは、Bitbucket Pipelinesの各設定オプションについての包括的なリファレンスです。基本概念から高度な実装例まで、実用的な観点で解説しています。

## 🚀 はじめに

### Bitbucket Pipelinesとは
Bitbucket Pipelinesは、Bitbucket Cloudに統合されたクラウドベースのCI/CDサービスです。

#### 特徴
- **統合性**: Bitbucket Cloudとネイティブ統合
- **コンテナベース**: Dockerコンテナで実行
- **設定ファイル**: `bitbucket-pipelines.yml`で定義
- **スケーラビリティ**: 自動スケーリング対応
- **セキュリティ**: 環境分離とシークレット管理

#### 実行環境
- **OS**: Ubuntu 20.04 LTS (デフォルト)
- **アーキテクチャ**: x86_64、ARM64（Runtime v3）
- **メモリ**: 4GB (1x) ～ 64GB (16x)
- **CPU**: 2 vCPU ～ 32 vCPU
- **ディスク**: 64GB ～ 256GB

### 基本概念
- **Pipeline**: 一連のビルド・テスト・デプロイプロセス
- **Step**: パイプライン内の個別の実行単位
- **Stage**: 複数のステップをグループ化した単位
- **Pipe**: 再利用可能な事前定義されたコンポーネント
- **Artifact**: ビルド成果物（ファイル、パッケージ等）

### パイプライン設定ファイルの基本構造

```yaml
# bitbucket-pipelines.yml の基本構造
image: atlassian/default-image:latest

options:
  # グローバルオプション

clone:
  # Gitクローン設定

definitions:
  # 再利用可能な定義

pipelines:
  # パイプライン定義
```

## 📑 目次

1. [キャッシュとサービスコンテナー定義](#1-キャッシュとサービスコンテナー定義)
2. [子パイプラインステップオプション](#2-子パイプラインステップオプション)
3. [Dockerイメージオプション](#3-dockerイメージオプション)
4. [Runtime v3の有効化と使用](#4-runtime-v3の有効化と使用)
5. [Gitクローンの動作](#5-gitクローンの動作)
6. [グローバルオプション](#6-グローバルオプション)
7. [並列ステップオプション](#7-並列ステップオプション)
8. [パイプライン開始条件](#8-パイプライン開始条件)
9. [ステージオプション](#9-ステージオプション)
10. [ステップオプション](#10-ステップオプション)
11. [変数とシークレット管理](#11-変数とシークレット管理)
12. [デプロイメント](#12-デプロイメント)
13. [高度な機能](#13-高度な機能)
14. [セキュリティ・認証](#14-セキュリティ認証)
15. [最適化・効率化](#15-最適化効率化)
16. [テスト・品質保証](#16-テスト品質保証)
17. [統合・拡張](#17-統合拡張)
18. [実践的な設定例とベストプラクティス](#18-実践的な設定例とベストプラクティス)
19. [トラブルシューティング](#19-トラブルシューティング)
20. [パフォーマンス最適化](#20-パフォーマンス最適化)

---

## 1. キャッシュとサービスコンテナー定義

### 概要

`definitions`セクションでキャッシュとサービスコンテナーを定義し、パイプライン間で再利用可能にします。

### キーオプション

#### **caches**

```yaml
definitions:
  caches:
    my-cache: vendor/bundle
    another-cache: ~/.npm
```

#### 事前定義キャッシュ

```yaml
caches:
  - docker         # /var/lib/docker
  - node           # ~/.npm と ~/.cache/yarn
  - npm            # ~/.npm
  - yarn           # ~/.cache/yarn
  - pip-cache      # ~/.cache/pip
  - composer       # ~/.composer/cache
  - gradle         # ~/.gradle/caches
  - maven          # ~/.m2/repository
  - sbt            # ~/.ivy2/cache と ~/.sbt
  - dotnetcore     # ~/.nuget/packages
```

#### カスタムキャッシュ設定

```yaml
definitions:
  caches:
    # Node.js プロジェクト用
    node-modules: node_modules
    npm-cache: ~/.npm
    yarn-cache: ~/.cache/yarn

    # Python プロジェクト用
    pip-packages: ~/.cache/pip
    pipenv-venv: ~/.local/share/virtualenvs

    # Java プロジェクト用
    maven-deps: ~/.m2/repository
    gradle-deps: ~/.gradle/caches

    # PHP プロジェクト用
    composer-vendor: vendor
    composer-cache: ~/.composer/cache

    # Ruby プロジェクト用
    bundler-gems: vendor/bundle
    gem-cache: ~/.gem

    # .NET プロジェクト用
    nuget-packages: ~/.nuget/packages

    # Go プロジェクト用
    go-modules: ~/go/pkg/mod
    go-build: ~/.cache/go-build
```

#### **services**

```yaml
definitions:
  services:
    postgres:
      image: postgres:12
      variables:
        POSTGRES_DB: mydb
        POSTGRES_USER: myuser
        POSTGRES_PASSWORD: mypassword
    redis:
      image: redis:6.0
```

#### データベースサービスの詳細設定

##### PostgreSQL

```yaml
definitions:
  services:
    postgres:
      image: postgres:13
      variables:
        POSTGRES_DB: testdb
        POSTGRES_USER: testuser
        POSTGRES_PASSWORD: testpass
        POSTGRES_HOST_AUTH_METHOD: trust
      ports:
        - "5432:5432"
```

##### MySQL

```yaml
definitions:
  services:
    mysql:
      image: mysql:8.0
      variables:
        MYSQL_DATABASE: testdb
        MYSQL_USER: testuser
        MYSQL_PASSWORD: testpass
        MYSQL_ROOT_PASSWORD: rootpass
        MYSQL_ALLOW_EMPTY_PASSWORD: "yes"
      ports:
        - "3306:3306"
```

##### MongoDB

```yaml
definitions:
  services:
    mongodb:
      image: mongo:4.4
      variables:
        MONGO_INITDB_ROOT_USERNAME: admin
        MONGO_INITDB_ROOT_PASSWORD: password
        MONGO_INITDB_DATABASE: testdb
      ports:
        - "27017:27017"
```

#### キャッシュサービス

##### Redis

```yaml
definitions:
  services:
    redis:
      image: redis:6.2-alpine
      ports:
        - "6379:6379"
```

##### Memcached

```yaml
definitions:
  services:
    memcached:
      image: memcached:1.6-alpine
      ports:
        - "11211:11211"
```

#### メッセージキューサービス

##### RabbitMQ

```yaml
definitions:
  services:
    rabbitmq:
      image: rabbitmq:3-management
      variables:
        RABBITMQ_DEFAULT_USER: guest
        RABBITMQ_DEFAULT_PASS: guest
      ports:
        - "5672:5672"
        - "15672:15672"
```

##### Apache Kafka

```yaml
definitions:
  services:
    zookeeper:
      image: confluentinc/cp-zookeeper:latest
      variables:
        ZOOKEEPER_CLIENT_PORT: 2181
        ZOOKEEPER_TICK_TIME: 2000

    kafka:
      image: confluentinc/cp-kafka:latest
      variables:
        KAFKA_BROKER_ID: 1
        KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
        KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://localhost:9092
        KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1
      ports:
        - "9092:9092"
```

#### 検索サービス

##### Elasticsearch

```yaml
definitions:
  services:
    elasticsearch:
      image: docker.elastic.co/elasticsearch/elasticsearch:7.14.0
      variables:
        discovery.type: single-node
        ES_JAVA_OPTS: "-Xms512m -Xmx512m"
      ports:
        - "9200:9200"
```

### 言語別キャッシュ戦略

#### Node.js プロジェクト

```yaml
definitions:
  caches:
    node-modules: node_modules
    npm-global: ~/.npm

pipelines:
  default:
    - step:
        name: Node.js ビルド
        caches:
          - node-modules
          - npm-global
        script:
          - npm ci
          - npm run build
          - npm test
```

#### Python プロジェクト

```yaml
definitions:
  caches:
    pip-cache: ~/.cache/pip
    pipenv-deps: ~/.local/share/virtualenvs

pipelines:
  default:
    - step:
        name: Python ビルド
        image: python:3.9
        caches:
          - pip-cache
        script:
          - pip install -r requirements.txt
          - python -m pytest
```

#### Java プロジェクト

```yaml
definitions:
  caches:
    maven-cache: ~/.m2/repository
    gradle-cache: ~/.gradle/caches

pipelines:
  default:
    - step:
        name: Java ビルド (Maven)
        image: maven:3.8-openjdk-11
        caches:
          - maven-cache
        script:
          - mvn clean compile test

    - step:
        name: Java ビルド (Gradle)
        image: gradle:7-jdk11
        caches:
          - gradle-cache
        script:
          - gradle clean build test
```

#### PHP プロジェクト

```yaml
definitions:
  caches:
    composer-cache: ~/.composer/cache
    vendor-cache: vendor

pipelines:
  default:
    - step:
        name: PHP ビルド
        image: php:8.0-cli
        caches:
          - composer-cache
          - vendor-cache
        script:
          - composer install --no-dev --optimize-autoloader
          - vendor/bin/phpunit
```

### 使用例

```yaml
definitions:
  caches:
    node-modules: node_modules
    composer: ~/.composer
  services:
    mysql:
      image: mysql:8.0
      variables:
        MYSQL_DATABASE: testdb
        MYSQL_ROOT_PASSWORD: secret

pipelines:
  default:
    - step:
        name: ビルドとテスト
        caches:
          - node-modules
        services:
          - mysql
        script:
          - npm install
          - npm test
```

---

## 2. 子パイプラインステップオプション

### 概要
子パイプラインは、メインパイプライン内から別のパイプラインをトリガーする機能です。

### 主要プロパティ

#### **type: pipeline**
```yaml
- step:
    type: pipeline
    custom: my-custom-pipeline
```

#### **custom**
- 実行するカスタムパイプラインの名前を指定
- **必須**: `type: pipeline`使用時

### 制限事項
以下のプロパティは子パイプラインステップでは使用できません：
- `deployment`
- `stage`
- `size`
- `runtime`
- `output-variables`
- `caches`
- `artifacts`
- `runs-on`
- `image`
- `clone`
- `oidc`
- `services`

### 使用例
```yaml
definitions:
  pipelines:
    custom:
      build-and-deploy:
        - step:
            name: ビルド
            script:
              - echo "ビルド実行"
        - step:
            name: デプロイ
            script:
              - echo "デプロイ実行"

pipelines:
  default:
    - step:
        name: 前処理
        script:
          - echo "前処理"
    - step:
        type: pipeline
        custom: build-and-deploy
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

---

## 3. Dockerイメージオプション

### 概要
パブリック・プライベートDockerイメージをビルド環境として使用できます。

### イメージの種類

#### **パブリックイメージ**
```yaml
image: node:16
# または
image: atlassian/default-image:latest
```

#### **プライベートイメージ（認証付き）**

```yaml
image:
  name: my-registry.com/my-image:latest
  username: $DOCKER_USERNAME
  password: $DOCKER_PASSWORD
```

#### **Docker Hub プライベートリポジトリ**

```yaml
image:
  name: mycompany/private-image:latest
  username: $DOCKER_HUB_USERNAME
  password: $DOCKER_HUB_PASSWORD
```

#### **Azure Container Registry**

```yaml
image:
  name: myregistry.azurecr.io/myapp:latest
  username: $ACR_USERNAME
  password: $ACR_PASSWORD
```

#### **Google Container Registry**

```yaml
image:
  name: gcr.io/my-project/my-image:latest
  username: _json_key
  password: $GCR_JSON_KEY
```

#### **AWS ECRイメージ**

```yaml
image:
  name: 123456789012.dkr.ecr.us-east-1.amazonaws.com/my-repo:latest
  aws:
    access-key: $AWS_ACCESS_KEY
    secret-key: $AWS_SECRET_KEY
```

#### **AWS ECR with OIDC**
```yaml
image:
  name: 123456789012.dkr.ecr.us-east-1.amazonaws.com/my-repo:latest
  aws:
    oidc-role: arn:aws:iam::123456789012:role/my-role
```

### オプション

#### **run-as-user**
```yaml
image:
  name: my-image:latest
  run-as-user: 1000
```

### 使用例
```yaml
image: node:16

pipelines:
  default:
    - step:
        name: Node.jsでビルド
        script:
          - npm install
          - npm run build
    - step:
        name: Pythonでテスト
        image: python:3.9
        script:
          - pip install pytest
          - pytest
```

---

## 4. Runtime v3の有効化と使用

### 概要
Runtime v3は、マルチアーキテクチャイメージと高度なDocker機能を提供します。

### 有効化方法

#### **全ステップで有効化**
```yaml
options:
  runtime:
    cloud:
      version: "3"
```

#### **個別ステップで有効化**
```yaml
- step:
    runtime:
      cloud:
        version: "3"
```

### 新機能

#### **特権コンテナー**
```yaml
- step:
    runtime:
      cloud:
        version: "3"
    services:
      - docker
    script:
      - docker run --privileged alpine:3 sh -c "echo test"
```

#### **マルチアーキテクチャビルド**
```yaml
- step:
    runtime:
      cloud:
        version: "3"
    services:
      - docker
    script:
      - docker buildx create --name multiarch-builder --use
      - docker buildx build --platform=linux/amd64,linux/arm64 .
```

#### **リモートキャッシュ**
```yaml
- step:
    runtime:
      cloud:
        version: "3"
    services:
      - docker
    script:
      - docker buildx build --cache-to type=s3,bucket=my-bucket .
```

### 制限事項
- Docker CLIは自動でマウントされません
- デフォルトキャッシュは利用可能ですが、ファイルベースのカスタムキャッシュキーの使用を検討してください

### カスタムランナー設定
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

## 5. Gitクローンの動作

### 概要
パイプラインでのGitクローン動作をカスタマイズできます。

### 主要オプション

#### **depth**
```yaml
clone:
  depth: full    # 完全クローン
  # または
  depth: 50      # シャロークローン（デフォルト）
```

#### **enabled**
```yaml
clone:
  enabled: false  # クローンを無効化
```

#### **lfs**
```yaml
clone:
  lfs: true      # Git LFSを有効化
```

#### **skip-ssl-verify**（セルフホストランナー用）
```yaml
clone:
  skip-ssl-verify: true
```

### 使用例
```yaml
clone:
  depth: full
  lfs: true

pipelines:
  default:
    - step:
        name: 完全履歴でクローン
        script:
          - git log --oneline | wc -l
    - step:
        name: クローン無効
        clone:
          enabled: false
        script:
          - echo "Gitリポジトリなしで実行"
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

---

## 6. グローバルオプション

### 概要
すべてのパイプラインに適用されるグローバル設定です。

### 主要オプション

#### **docker**

```yaml
options:
  docker: true  # 全ステップでDockerサービスを有効化
```

- **用途**: Dockerコマンドの実行、イメージのビルド
- **制限**: 追加リソース消費
- **注意点**: 必要な場合のみ有効化

#### **max-time**

```yaml
options:
  max-time: 60  # 最大実行時間（分）
```

- **デフォルト**: 120分
- **最大値**: 120分
- **最小値**: 1分
- **適用範囲**: パイプライン全体

#### **size**

```yaml
options:
  size: 2x      # リソースサイズ（1x, 2x, 4x, 8x, 16x）
```

- **1x**: 4GB RAM, 通常のビルド用
- **2x**: 8GB RAM, 大きなビルドやテスト用
- **4x以上**: 非常に大きなビルド、並列処理用
- **コスト**: 2xは1xの約2倍のビルドミニッツを消費

#### **runtime**

```yaml
options:
  runtime:
    cloud:
      atlassian-ip-ranges: true
      arch: arm
```

- **Runtime v2**: レガシー（非推奨）
- **Runtime v3**: 改善されたパフォーマンスとセキュリティ
- **移行**: 段階的に移行推奨

### リソース配分表

| サイズ | CPU | メモリ | ボリューム |
|--------|-----|--------|-----------|
| 1x     | 2   | 4GB    | 64GB      |
| 2x     | 4   | 8GB    | 64GB      |
| 4x     | 8   | 16GB   | 256GB     |
| 8x     | 16  | 32GB   | 256GB     |
| 16x    | 32  | 64GB   | 256GB     |

### 使用例
```yaml
options:
  docker: true
  max-time: 120
  size: 2x
  runtime:
    cloud:
      atlassian-ip-ranges: true

pipelines:
  default:
    - step:
        name: グローバル設定適用
        script:
          - docker --version
          - echo "2x メモリで実行"
```

---

## 7. 並列ステップオプション

### 概要
複数のステップを同時に実行してビルド時間を短縮できます。

### 基本構文
```yaml
pipelines:
  default:
    - step:
        name: ビルド
        script:
          - echo "ビルド"
    - parallel:
        steps:
          - step:
              name: テスト1
              script:
                - echo "テスト1"
          - step:
              name: テスト2
              script:
                - echo "テスト2"
```

### オプション

#### **fail-fast**
```yaml
- parallel:
    fail-fast: true  # いずれかが失敗したら全て停止
    steps:
      - step:
          name: テストA
          script: # ...
      - step:
          name: テストB
          script: # ...
```

#### **個別のfail-fast設定**
```yaml
- parallel:
    fail-fast: true
    steps:
      - step:
          name: 重要なテスト
          script: # ...
      - step:
          name: オプショナルなテスト
          fail-fast: false  # このステップの失敗は無視
          script: # ...
```

### デフォルト変数
- `BITBUCKET_PARALLEL_STEP`: ステップのインデックス（0から開始）
- `BITBUCKET_PARALLEL_STEP_COUNT`: 並列ステップの総数

### 制限事項
- アーティファクトは同じ並列グループ内では共有されない
- ワークスペースあたりの同時実行数制限あり
  - Standard/Premium: 600ステップ
  - Free: 10ステップ

### 並列実行の最適化

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

### 使用例
```yaml
pipelines:
  default:
    - step:
        name: ビルド
        script:
          - npm run build
        artifacts:
          - dist/**
    - parallel:
        fail-fast: true
        steps:
          - step:
              name: 単体テスト
              script:
                - npm run test:unit
          - step:
              name: 統合テスト
              script:
                - npm run test:integration
          - step:
              name: リント
              fail-fast: false
              script:
                - npm run lint
```

---

## 8. パイプライン開始条件

### 概要
パイプラインをトリガーする条件を定義します。

### トリガータイプ

#### **branches**
```yaml
pipelines:
  branches:
    main:
      - step:
          name: メインブランチデプロイ
          script:
            - echo "本番デプロイ"
    develop:
      - step:
          name: 開発環境デプロイ
          script:
            - echo "開発環境デプロイ"
    feature/*:
      - step:
          name: 機能テスト
          script:
            - echo "機能テスト"
```

#### **pull-requests**
```yaml
pipelines:
  pull-requests:
    '**':
      - step:
          name: PRテスト
          script:
            - npm test
    feature/*:
      - step:
          name: 機能PR検証
          script:
            - npm run test:feature
```

#### **tags**
```yaml
pipelines:
  tags:
    v*:
      - step:
          name: リリース
          script:
            - echo "バージョンタグでリリース"
    release-*:
      - step:
          name: リリース候補
          script:
            - echo "リリース候補ビルド"
```

#### **custom**（手動トリガー）
```yaml
pipelines:
  custom:
    deploy-staging:
      - step:
          name: ステージングデプロイ
          script:
            - echo "手動でステージングデプロイ"
    emergency-hotfix:
      - step:
          name: 緊急修正
          script:
            - echo "緊急修正デプロイ"
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
# changeset条件はステップ内で使用する
```

### 使用例
```yaml
pipelines:
  default:
    - step:
        name: フロントエンドビルド
        condition:
          changesets:
            includePaths:
              - "app/**/*.js"
              - "src/**/*.{ts,tsx}"
            excludePaths:
              - "**/*.md"
              - "docs/**"
        script:
          - npm run build
    - step:
        name: デフォルトビルド
        script:
          - npm install
          - npm test

  branches:
    main:
      - step:
          name: 本番デプロイ
          deployment: production
          script:
            - npm run deploy:prod
    develop:
      - step:
          name: ステージングデプロイ
          deployment: staging
          script:
            - npm run deploy:staging

  pull-requests:
    '**':
      - step:
          name: PR検証
          script:
            - npm run test:ci
            - npm run lint

  tags:
    'v*.*.*':
      - step:
          name: リリース
          script:
            - npm run build:prod
            - npm publish

  custom:
    manual-deploy:
      - variables:
          - name: ENVIRONMENT
            default: staging
      - step:
          name: 手動デプロイ
          script:
            - echo "デプロイ先: $ENVIRONMENT"
```

---

## 9. ステージオプション

### 概要
複数のステップを論理的にグループ化し、共通のプロパティを共有できます。

### 基本構文
```yaml
pipelines:
  default:
    - stage:
        name: ビルドとテスト
        steps:
          - step:
              name: ビルド
              script:
                - npm run build
          - step:
              name: テスト
              script:
                - npm test
```

### 主要オプション

#### **deployment**
```yaml
- stage:
    name: 本番デプロイ
    deployment: production
    steps:
      - step:
          name: デプロイ
          script:
            - kubectl apply -f k8s/
      - step:
          name: ヘルスチェック
          script:
            - curl -f http://app.example.com/health
```

#### **environment**
```yaml
- stage:
    name: ステージング環境
    environment: staging
    steps:
      - step:
          name: 環境テスト
          script:
            - echo $STAGING_URL
```

#### **trigger**
```yaml
- stage:
    name: 手動承認が必要
    trigger: manual
    steps:
      - step:
          name: 本番リリース
          script:
            - echo "手動承認後実行"
```

#### **condition**
```yaml
- stage:
    name: フロントエンド変更時のみ
    condition:
      changesets:
        includePaths:
          - "src/frontend/**"
    steps:
      - step:
          name: フロントエンドテスト
          script:
            - npm run test:frontend
```

#### **on-fail**
```yaml
- stage:
    name: 失敗時の動作
    on-fail:
      strategy: retry
      maxRetryCount: 3
    steps:
      - step:
          name: 不安定なテスト
          script:
            - npm run test:flaky
```

### 制限事項
- ステージあたり最大100ステップ（Standard/Premium）、10ステップ（Free）
- 並列ステージは未サポート
- ステージ内に並列ステップは含められない
- 手動トリガーステップは含められない

### 使用例
```yaml
pipelines:
  default:
    - stage:
        name: 品質チェック
        condition:
          changesets:
            includePaths:
              - "src/**"
              - "tests/**"
        steps:
          - step:
              name: コードフォーマット
              script:
                - npm run format:check
          - step:
              name: リント
              script:
                - npm run lint
          - step:
              name: 単体テスト
              script:
                - npm run test:unit

    - stage:
        name: ステージングデプロイ
        deployment: staging
        trigger: manual
        steps:
          - step:
              name: ビルドとデプロイ
              script:
                - npm run build
                - npm run deploy:staging
          - step:
              name: 統合テスト
              script:
                - npm run test:integration

    - stage:
        name: 本番デプロイ
        deployment: production
        environment: production
        trigger: manual
        steps:
          - step:
              name: 本番リリース
              script:
                - npm run deploy:production
          - step:
              name: スモークテスト
              script:
                - npm run test:smoke
```

---

## 10. ステップオプション

### 概要
パイプラインの実行単位であるステップの詳細設定オプションです。

### 基本プロパティ

#### **script**（必須）
```yaml
- step:
    script:
      - echo "Hello World"
      - npm install
      - npm test
```

#### **name**
```yaml
- step:
    name: ビルドとテスト
    script:
      - npm run build
```

#### **image**
```yaml
- step:
    image: node:16
    script:
      - npm --version
```

### リソース管理

#### **size**
```yaml
- step:
    size: 2x  # メモリとCPUを倍増
    script:
      - npm run build:large
```

#### **max-time**
```yaml
- step:
    max-time: 30  # 30分でタイムアウト
    script:
      - npm run test:long
```

#### **runtime**
```yaml
- step:
    runtime:
      cloud:
        atlassian-ip-ranges: true
        arch: arm
    script:
      - echo "ARM環境で実行"
```

### キャッシュとアーティファクト

#### **caches**
```yaml
- step:
    caches:
      - node
      - npm
    script:
      - npm install
```

#### **artifacts**
```yaml
- step:
    script:
      - npm run build
    artifacts:
      - dist/**
      - build/static/**
```

#### **artifact options**
```yaml
- step:
    script:
      - npm run lint
    artifacts:
      download: false  # 前のアーティファクトをダウンロードしない
      paths:
        - lint-results.xml
```

### サービスとDocker

#### **services**
```yaml
- step:
    services:
      - postgres
      - redis
    script:
      - npm run test:integration
```

#### **docker（グローバル無効時）**
```yaml
- step:
    services:
      - docker
    script:
      - docker build -t my-app .
      - docker run my-app npm test
```

### セルフホストランナー

#### **runs-on**
```yaml
- step:
    runs-on:
      - 'self.hosted'
      - 'linux.shell'
    script:
      - echo "セルフホストランナーで実行"
```

### フロー制御

#### **trigger**
```yaml
- step:
    name: 手動承認が必要
    trigger: manual
    script:
      - kubectl apply -f prod-deployment.yaml
```

#### **condition**
```yaml
- step:
    name: フロントエンド変更時のみ
    condition:
      changesets:
        includePaths:
          - "frontend/**"
        excludePaths:
          - "frontend/docs/**"
    script:
      - npm run test:frontend
```

#### **on-fail**
```yaml
- step:
    name: 不安定なテスト
    on-fail:
      strategy: retry
      maxRetryCount: 3
    script:
      - npm run test:flaky

- step:
    name: オプショナルなタスク
    on-fail:
      strategy: ignore
    script:
      - npm run optional-task
```

### デプロイメント

#### **deployment**
```yaml
- step:
    name: 本番デプロイ
    deployment: production
    script:
      - kubectl apply -f k8s/production/
```

#### **environment**
```yaml
- step:
    name: 環境依存テスト
    environment: staging
    script:
      - echo $STAGING_DATABASE_URL
```

### 高度なオプション

#### **after-script**
```yaml
- step:
    script:
      - npm test
    after-script:
      - echo "テスト完了"
      - npm run cleanup
```

#### **fail-fast**
```yaml
- step:
    fail-fast: false  # 並列グループ内で失敗しても他を止めない
    script:
      - npm run optional-test
```

#### **output-variables**
```yaml
- step:
    script:
      - echo "BUILD_NUMBER=$(date +%s)" >> $BITBUCKET_PIPELINES_VARIABLES_PATH
    output-variables:
      - BUILD_NUMBER
```

#### **concurrency-group**
```yaml
- step:
    concurrency-group: deploy-production
    script:
      - kubectl apply -f k8s/
```

### Pipes

#### **pipe**
```yaml
- step:
    script:
      - pipe: atlassian/aws-s3-deploy:1.1.0
        variables:
          AWS_ACCESS_KEY_ID: $AWS_ACCESS_KEY_ID
          AWS_SECRET_ACCESS_KEY: $AWS_SECRET_ACCESS_KEY
          S3_BUCKET: my-bucket
          LOCAL_PATH: dist
```

### 完全な使用例
```yaml
options:
  size: 2x
  docker: true

definitions:
  caches:
    npm-cache: ~/.npm
  services:
    postgres:
      image: postgres:13
      variables:
        POSTGRES_DB: testdb
        POSTGRES_PASSWORD: secret

pipelines:
  default:
    - step:
        name: ビルドとキャッシュ
        image: node:16
        size: 4x
        caches:
          - npm-cache
        script:
          - npm ci
          - npm run build
          - npm run lint
        artifacts:
          - dist/**
          - coverage/**
        after-script:
          - echo "ビルド完了: $(date)"

    - parallel:
        fail-fast: true
        steps:
          - step:
              name: 単体テスト
              services:
                - postgres
              script:
                - npm run test:unit
              artifacts:
                download: true
                paths:
                  - test-results.xml

          - step:
              name: 統合テスト
              max-time: 45
              condition:
                changesets:
                  includePaths:
                    - "src/api/**"
              script:
                - npm run test:integration
              on-fail:
                strategy: retry
                maxRetryCount: 2

    - step:
        name: セキュリティスキャン
        trigger: manual
        script:
          - pipe: atlassian/snyk-security-scan:0.3.0
            variables:
              SNYK_TOKEN: $SNYK_TOKEN

    - step:
        name: 本番デプロイ
        deployment: production
        environment: production
        trigger: manual
        runs-on:
          - 'self.hosted'
          - 'production'
        script:
          - kubectl apply -f k8s/production/
          - kubectl rollout status deployment/app
        after-script:
          - echo "デプロイ完了"
```

---

## 11. 変数とシークレット管理

### 環境変数の種類

#### システム提供変数

```bash
# Bitbucket提供の環境変数
BITBUCKET_BRANCH            # 現在のブランチ名
BITBUCKET_BUILD_NUMBER      # ビルド番号
BITBUCKET_CLONE_DIR         # クローンディレクトリ
BITBUCKET_COMMIT            # コミットハッシュ
BITBUCKET_REPO_SLUG         # リポジトリスラッグ
BITBUCKET_REPO_OWNER        # リポジトリオーナー
BITBUCKET_REPO_FULL_NAME    # フルリポジトリ名
BITBUCKET_WORKSPACE         # ワークスペース名
BITBUCKET_PROJECT_KEY       # プロジェクトキー
BITBUCKET_PR_ID             # プルリクエストID
BITBUCKET_PR_DESTINATION_BRANCH # PRの宛先ブランチ
BITBUCKET_TAG               # タグ名（タグビルド時）
BITBUCKET_DEPLOYMENT_ENVIRONMENT # デプロイメント環境
```

#### カスタム変数の設定

```yaml
pipelines:
  default:
    - variables:
        - name: DATABASE_URL
          default: postgres://localhost:5432/mydb
        - name: API_KEY
          default: ""
    - step:
        name: 変数を使用
        script:
          - echo "Database: $DATABASE_URL"
          - echo "API Key: $API_KEY"
```

#### 環境変数の使用例

```yaml
pipelines:
  default:
    - step:
        name: 環境変数テスト
        script:
          - echo "ビルド番号: $BITBUCKET_BUILD_NUMBER"
          - echo "ブランチ: $BITBUCKET_BRANCH"
          - echo "コミット: $BITBUCKET_COMMIT"
          - echo "リポジトリ: $BITBUCKET_REPO_FULL_NAME"
```

### シークレットとセキュリティ

#### Repository Variables設定

1. **リポジトリ設定** → **Repository variables**
2. **環境変数**と**Secured変数**を設定
3. **Secured変数**はログに出力されない

#### Workspace Variables

1. **ワークスペース設定** → **Workspace variables**
2. **複数リポジトリで共有**可能
3. **リポジトリ変数より優先度低**

#### Deployment Variables

```yaml
- step:
    name: 本番デプロイ
    deployment: production
    script:
      - echo "Production URL: $PRODUCTION_URL"
      - echo "API Key: $PRODUCTION_API_KEY"
```

### 変数の使用例
```yaml
# リポジトリ変数の使用
pipelines:
  default:
    - step:
        name: Use Variables
        script:
          - echo "Environment: $ENVIRONMENT"
          - echo "API URL: $API_URL"
          - echo "Secret token: set" # セキュアな変数の存在確認のみ

# ステップレベル変数
- step:
    name: Custom Variables
    script:
      - export CUSTOM_VAR="value"
      - echo $CUSTOM_VAR
```

---

## 12. デプロイメント

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

## 13. 高度な機能

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

## 14. セキュリティ・認証

### SSH認証
```yaml
pipelines:
  default:
    - step:
        name: SSH Access
        script:
          # SSH設定のディレクトリとファイル準備
          - mkdir -p ~/.ssh
          - chmod 700 ~/.ssh
          - ssh-keyscan -H github.com >> ~/.ssh/known_hosts
          - chmod 644 ~/.ssh/known_hosts
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

## 15. 最適化・効率化

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

## 16. テスト・品質保証

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

## 17. 統合・拡張

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

---

## 18. 実践的な設定例とベストプラクティス

### 完全なCI/CDパイプライン例

#### Node.js フルスタックアプリケーション

```yaml
image: node:18

options:
  max-time: 120
  size: 2x

definitions:
  caches:
    node-modules: node_modules
    npm-cache: ~/.npm
  services:
    postgres:
      image: postgres:13
      variables:
        POSTGRES_DB: testdb
        POSTGRES_USER: testuser
        POSTGRES_PASSWORD: testpass
    redis:
      image: redis:6-alpine

pipelines:
  default:
    - step:
        name: 📦 依存関係インストール
        caches:
          - node-modules
          - npm-cache
        script:
          - npm ci
        artifacts:
          - node_modules/**

    - parallel:
        fail-fast: true
        steps:
          - step:
              name: 🧪 単体テスト
              caches:
                - node-modules
              script:
                - npm run test:unit
                - npm run coverage
              artifacts:
                - coverage/**
                - test-results.xml

          - step:
              name: 🔍 コード品質チェック
              caches:
                - node-modules
              script:
                - npm run lint
                - npm run type-check
                - npm run audit

          - step:
              name: 🏗️ ビルド
              caches:
                - node-modules
              script:
                - npm run build
              artifacts:
                - dist/**

    - step:
        name: 🔬 統合テスト
        services:
          - postgres
          - redis
        script:
          - npm run test:integration
          - npm run test:e2e

  branches:
    develop:
      - step:
          name: 📦 依存関係インストール
          caches:
            - node-modules
          script:
            - npm ci

      - parallel:
          steps:
            - step:
                name: 🧪 テスト
                script:
                  - npm run test
            - step:
                name: 🏗️ ビルド
                script:
                  - npm run build
                artifacts:
                  - dist/**

      - step:
          name: 🚀 ステージングデプロイ
          deployment: staging
          script:
            - pipe: atlassian/aws-s3-deploy:1.1.0
              variables:
                AWS_ACCESS_KEY_ID: $AWS_ACCESS_KEY_ID
                AWS_SECRET_ACCESS_KEY: $AWS_SECRET_ACCESS_KEY
                S3_BUCKET: $STAGING_S3_BUCKET
                LOCAL_PATH: dist

    main:
      - step:
          name: 📦 依存関係インストール
          caches:
            - node-modules
          script:
            - npm ci

      - step:
          name: 🏗️ プロダクションビルド
          script:
            - npm run build:prod
          artifacts:
            - dist/**

      - step:
          name: 🔐 セキュリティスキャン
          script:
            - pipe: atlassian/snyk-security-scan:0.3.0
              variables:
                SNYK_TOKEN: $SNYK_TOKEN
                LANGUAGE: javascript

      - step:
          name: 🚀 本番デプロイ
          deployment: production
          trigger: manual
          script:
            - pipe: atlassian/aws-s3-deploy:1.1.0
              variables:
                AWS_ACCESS_KEY_ID: $AWS_ACCESS_KEY_ID
                AWS_SECRET_ACCESS_KEY: $AWS_SECRET_ACCESS_KEY
                S3_BUCKET: $PRODUCTION_S3_BUCKET
                LOCAL_PATH: dist
            - pipe: atlassian/aws-cloudfront-invalidate:0.6.0
              variables:
                AWS_ACCESS_KEY_ID: $AWS_ACCESS_KEY_ID
                AWS_SECRET_ACCESS_KEY: $AWS_SECRET_ACCESS_KEY
                DISTRIBUTION_ID: $CLOUDFRONT_DISTRIBUTION_ID

  pull-requests:
    '**':
      - step:
          name: 📦 依存関係インストール
          caches:
            - node-modules
          script:
            - npm ci

      - parallel:
          fail-fast: false
          steps:
            - step:
                name: 🧪 テスト
                script:
                  - npm run test
                  - npm run test:coverage

            - step:
                name: 🔍 コード品質
                script:
                  - npm run lint
                  - npm run type-check

            - step:
                name: 🏗️ ビルド確認
                script:
                  - npm run build

  tags:
    'v*.*.*':
      - step:
          name: 📦 依存関係インストール
          caches:
            - node-modules
          script:
            - npm ci

      - step:
          name: 🏗️ リリースビルド
          script:
            - npm run build:prod
          artifacts:
            - dist/**

      - step:
          name: 📋 リリースノート生成
          script:
            - npm run generate-changelog
            - git tag -l --format='%(contents)' $BITBUCKET_TAG > release-notes.md
          artifacts:
            - release-notes.md

      - step:
          name: 🚀 リリースデプロイ
          deployment: production
          script:
            - echo "リリース $BITBUCKET_TAG をデプロイ"
            - npm publish
```

#### Python Djangoアプリケーション

```yaml
image: python:3.11

options:
  max-time: 90
  size: 2x

definitions:
  caches:
    pip-cache: ~/.cache/pip
  services:
    postgres:
      image: postgres:13
      variables:
        POSTGRES_DB: testdb
        POSTGRES_USER: testuser
        POSTGRES_PASSWORD: testpass
    redis:
      image: redis:6-alpine

pipelines:
  default:
    - step:
        name: 🐍 環境セットアップ
        caches:
          - pip-cache
        script:
          - pip install --upgrade pip
          - pip install -r requirements.txt
          - pip install -r requirements-dev.txt

    - parallel:
        steps:
          - step:
              name: 🧪 単体テスト
              services:
                - postgres
                - redis
              script:
                - python manage.py test
                - coverage run --source='.' manage.py test
                - coverage xml
              artifacts:
                - coverage.xml

          - step:
              name: 🔍 コード品質
              script:
                - flake8 .
                - black --check .
                - isort --check-only .
                - bandit -r . -f json -o bandit-report.json
              artifacts:
                - bandit-report.json

          - step:
              name: 🔒 セキュリティチェック
              script:
                - safety check
                - pip-audit

    - step:
        name: 🚀 ステージングデプロイ
        deployment: staging
        script:
          - echo "Djangoアプリをステージングにデプロイ"
          - python manage.py collectstatic --noinput
          - python manage.py migrate --check

  branches:
    main:
      - step:
          name: 🐍 環境セットアップ
          caches:
            - pip-cache
          script:
            - pip install -r requirements.txt

      - step:
          name: 🧪 フルテストスイート
          services:
            - postgres
            - redis
          script:
            - python manage.py test
            - python manage.py check --deploy

      - step:
          name: 🚀 本番デプロイ
          deployment: production
          trigger: manual
          script:
            - echo "本番環境へデプロイ"
            - python manage.py collectstatic --noinput
            - python manage.py migrate
```

### マイクロサービス アーキテクチャ

```yaml
image: node:18

definitions:
  caches:
    node-modules: node_modules
  services:
    postgres:
      image: postgres:13
      variables:
        POSTGRES_DB: microservices_db
        POSTGRES_USER: user
        POSTGRES_PASSWORD: password
    redis:
      image: redis:6-alpine
    mongodb:
      image: mongo:4.4

pipelines:
  default:
    - parallel:
        fail-fast: true
        steps:
          - step:
              name: 🔧 API Gateway Service
              script:
                - cd services/api-gateway
                - npm ci
                - npm run test
                - npm run build
              artifacts:
                - services/api-gateway/dist/**

          - step:
              name: 👤 User Service
              services:
                - postgres
              script:
                - cd services/user-service
                - npm ci
                - npm run test
                - npm run build
              artifacts:
                - services/user-service/dist/**

          - step:
              name: 🛒 Order Service
              services:
                - mongodb
              script:
                - cd services/order-service
                - npm ci
                - npm run test
                - npm run build
              artifacts:
                - services/order-service/dist/**

          - step:
              name: 💳 Payment Service
              services:
                - redis
              script:
                - cd services/payment-service
                - npm ci
                - npm run test
                - npm run build
              artifacts:
                - services/payment-service/dist/**

    - step:
        name: 🐳 Dockerイメージビルド
        services:
          - docker
        script:
          - docker build -t api-gateway:$BITBUCKET_BUILD_NUMBER services/api-gateway
          - docker build -t user-service:$BITBUCKET_BUILD_NUMBER services/user-service
          - docker build -t order-service:$BITBUCKET_BUILD_NUMBER services/order-service
          - docker build -t payment-service:$BITBUCKET_BUILD_NUMBER services/payment-service

    - step:
        name: 🔬 統合テスト
        services:
          - postgres
          - mongodb
          - redis
        script:
          - npm run test:integration
```

---

## 19. トラブルシューティング

### よくある問題と解決策

#### 1. メモリ不足エラー

**症状**: `ENOMEM: not enough memory` エラー

**解決策**:
```yaml
options:
  size: 2x  # または 4x

# または個別ステップで
- step:
    size: 2x
    script:
      - npm run build
```

#### 2. ビルド時間の超過

**症状**: `Build failed: Maximum allowed time exceeded`

**解決策**:
```yaml
options:
  max-time: 120  # 最大120分

# キャッシュを活用
definitions:
  caches:
    node-modules: node_modules

pipelines:
  default:
    - step:
        caches:
          - node-modules
        script:
          - npm ci  # npm install より高速
```

#### 3. Docker イメージプル失敗

**症状**: `Error response from daemon: pull access denied`

**解決策**:
```yaml
image:
  name: private-registry.com/image:tag
  username: $DOCKER_USERNAME
  password: $DOCKER_PASSWORD
```

#### 4. 並列ステップでのアーティファクト共有問題

**症状**: 前のステップのアーティファクトが見つからない

**解決策**:
```yaml
- step:
    name: ビルド
    script:
      - npm run build
    artifacts:
      - dist/**

- parallel:
    steps:
      - step:
          name: テスト1
          # 並列ステップは前のアーティファクトを自動継承
          script:
            - ls dist/  # アーティファクトが利用可能
      - step:
          name: テスト2
          artifacts:
            download: false  # アーティファクトが不要な場合
          script:
            - npm run lint
```

#### 5. キャッシュが効かない

**症状**: 毎回依存関係をダウンロード

**解決策**:
```yaml
definitions:
  caches:
    # 正しいキャッシュパスを指定
    node-cache: node_modules
    npm-cache: ~/.npm

pipelines:
  default:
    - step:
        caches:
          - node-cache
          - npm-cache
        script:
          - npm ci  # package-lock.jsonに基づく確定的なインストール
```

### デバッグのベストプラクティス

#### 1. ログ出力の改善

```yaml
- step:
    name: デバッグ情報出力
    script:
      - echo "=== 環境情報 ==="
      - node --version
      - npm --version
      - echo "=== ディスク使用量 ==="
      - df -h
      - echo "=== メモリ使用量 ==="
      - free -h
      - echo "=== プロセス一覧 ==="
      - ps aux
```

#### 2. 条件付きデバッグ

```yaml
- step:
    script:
      - |
        if [ "$DEBUG" = "true" ]; then
          set -x  # コマンドを表示
          env | sort  # 環境変数を表示
        fi
      - npm run build
```

#### 3. エラー時の詳細情報収集

```yaml
- step:
    script:
      - npm test || (echo "テスト失敗時の詳細情報"; cat test-results.log; exit 1)
    after-script:
      - echo "=== 実行完了時刻: $(date) ==="
      - echo "=== 最終ディスク使用量 ==="
      - df -h
```

### デバッグ・ログ設定
```yaml
- step:
    name: Debug Step
    services:
      - docker
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
      - npm test
    after-script:
      - |
        if [ $BITBUCKET_EXIT_CODE -ne 0 ]; then
          echo "Tests failed, sending notification"
        fi
      - pipe: atlassian/slack-notify:2.1.0
        variables:
          WEBHOOK_URL: $SLACK_WEBHOOK
          MESSAGE: "Pipeline status: $BITBUCKET_EXIT_CODE"
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

---

## 20. パフォーマンス最適化

### ビルド時間短縮のテクニック

#### 1. 効果的なキャッシュ戦略

```yaml
definitions:
  caches:
    # 複数のキャッシュを組み合わせ
    node-modules: node_modules
    npm-cache: ~/.npm
    cypress-cache: ~/.cache/Cypress

pipelines:
  default:
    - step:
        name: 依存関係の最適化インストール
        caches:
          - node-modules
          - npm-cache
        script:
          # package-lock.jsonが変更された場合のみ再インストール
          - |
            if [ ! -f node_modules/.package-lock.json ] || ! cmp -s package-lock.json node_modules/.package-lock.json; then
              npm ci
              cp package-lock.json node_modules/.package-lock.json
            else
              echo "依存関係は最新です"
            fi
```

#### 2. 並列化の最適化

```yaml
- parallel:
    fail-fast: true
    steps:
      # 短時間で完了するタスクを先に
      - step:
          name: 🔍 Lint (高速)
          script:
            - npm run lint

      # 中程度の時間がかかるタスク
      - step:
          name: 🧪 単体テスト (中程度)
          script:
            - npm run test:unit

      # 時間がかかるタスクを最後に
      - step:
          name: 🏗️ ビルド (低速)
          script:
            - npm run build
```

#### 3. 条件付き実行の活用

```yaml
- step:
    name: フロントエンドテスト
    condition:
      changesets:
        includePaths:
          - "frontend/**"
          - "package.json"
          - "package-lock.json"
    script:
      - npm run test:frontend

- step:
    name: バックエンドテスト
    condition:
      changesets:
        includePaths:
          - "backend/**"
          - "requirements.txt"
    script:
      - python -m pytest
```

#### 4. Dockerレイヤーキャッシング

```yaml
- step:
    name: 最適化されたDockerビルド
    services:
      - docker
    script:
      # マルチステージビルドでキャッシュ効率化
      - |
        cat > Dockerfile.optimized << 'EOF'
        # ベースイメージ
        FROM node:18-alpine as base
        WORKDIR /app

        # 依存関係のインストール（変更頻度低）
        FROM base as deps
        COPY package*.json ./
        RUN npm ci --only=production

        # 開発依存関係（変更頻度低）
        FROM base as dev-deps
        COPY package*.json ./
        RUN npm ci

        # ビルド（変更頻度高）
        FROM dev-deps as build
        COPY . .
        RUN npm run build

        # 本番イメージ
        FROM base as production
        COPY --from=deps /app/node_modules ./node_modules
        COPY --from=build /app/dist ./dist
        COPY package*.json ./
        EOF
      - docker build -f Dockerfile.optimized -t myapp:$BITBUCKET_BUILD_NUMBER .
```

### リソース使用量の最適化

#### 1. 適切なサイズ選択

```yaml
pipelines:
  default:
    # 軽量なタスクは1x
    - step:
        name: Lint
        size: 1x
        script:
          - npm run lint

    # ビルドやテストは2x
    - step:
        name: ビルドとテスト
        size: 2x
        script:
          - npm run build
          - npm test

    # 大量のメモリが必要なタスクは4x
    - step:
        name: E2Eテスト
        size: 4x
        script:
          - npm run test:e2e
```

#### 2. 効率的なサービス利用

```yaml
definitions:
  services:
    # 軽量なイメージを選択
    postgres:
      image: postgres:13-alpine
    redis:
      image: redis:6-alpine

pipelines:
  default:
    - step:
        name: 単体テスト（DB不要）
        script:
          - npm run test:unit  # サービス未使用

    - step:
        name: 統合テスト（DB必要）
        services:
          - postgres  # 必要な時のみサービス使用
        script:
          - npm run test:integration
```

---

## 🔗 関連リンク

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

---

## 📝 まとめ

このリファレンスでは、Bitbucket Pipelinesの基本設定から高度な実装例まで、実用的な観点で解説しました。

### 重要なポイント

1. **キャッシュ戦略**: 適切なキャッシュ設定でビルド時間を大幅短縮
2. **並列実行**: テストとビルドの並列化でパイプライン実行時間を最適化
3. **条件付き実行**: 変更されたファイルのみを対象とした効率的な実行
4. **セキュリティ**: 環境変数とシークレットの適切な管理
5. **リソース管理**: 必要に応じたサイズとサービスの選択

### ベストプラクティス

- **小さく始めて段階的に拡張**
- **キャッシュを積極的に活用**
- **並列実行でパフォーマンス向上**
- **セキュリティを最優先**
- **継続的な最適化**

Bitbucket Pipelinesを効果的に活用し、高品質なソフトウェアの継続的なデリバリーを実現しましょう。