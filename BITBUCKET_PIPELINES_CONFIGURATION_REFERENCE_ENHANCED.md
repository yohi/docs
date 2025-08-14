# 🚀 Bitbucket Pipelines 詳細リファレンス - Enhanced Edition

このドキュメントは、Bitbucket Pipelinesの完全なリファレンスガイドです。基本概念から企業レベルの実装例まで、実用的な観点で解説しています。

## 🎯 概要とアーキテクチャ

### Bitbucket Pipelinesとは

**Bitbucket Pipelines**は、Bitbucket Cloudに統合されたクラウドベースのCI/CDサービスです。コード変更を検知して自動的にビルド、テスト、デプロイメントを実行します。

#### 🏗️ システムアーキテクチャ

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Git Push      │ ──▷│  Bitbucket      │ ──▷│   Pipelines     │
│   Pull Request  │    │  Repository     │    │   Execution     │
│   Tag Creation  │    │                 │    │                 │
└─────────────────┘    └─────────────────┘    └─────────────────┘
                                  │
                                  ▼
                       ┌─────────────────┐
                       │  AWS Cloud      │
                       │  Docker         │
                       │  Containers     │
                       └─────────────────┘
```

#### ⚡ 動作フロー

```yaml
# 基本的な実行フロー
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

#### 🌟 主要特徴

- **統合性**: Bitbucket Cloudとネイティブ統合
- **コンテナベース**: Dockerコンテナで実行
- **設定ファイル**: `bitbucket-pipelines.yml`で定義
- **スケーラビリティ**: 自動スケーリング対応
- **セキュリティ**: 環境分離とシークレット管理
- **クラウドネイティブ**: AWS上で動作する分散実行基盤

#### 💻 実行環境

- **OS**: Ubuntu 20.04 LTS (デフォルト)
- **アーキテクチャ**: x86_64、ARM64（Runtime v3）
- **メモリ**: 4GB (1x) ～ 64GB (16x)
- **CPU**: 2 vCPU ～ 32 vCPU
- **ディスク**: 64GB ～ 256GB

### 🧩 基本概念

#### パイプライン（Pipeline）
- **定義**: 一連のビルド・テスト・デプロイプロセス
- **種類**: デフォルト、ブランチ別、タグ別、プルリクエスト、カスタム
- **実行コンテキスト**: ブランチ、プルリクエスト、タグ、手動トリガー

#### ステップ（Step）
- **実行単位**: 独立したDockerコンテナで実行
- **リソース**: CPU、メモリ、ディスク容量を指定可能
- **タイムアウト**: デフォルト120分、最大2時間まで設定可能
- **状態管理**: 成功、失敗、停止の状態を持つ

#### ステージ（Stage）
- **並列実行**: 複数のステップを同時実行
- **依存関係**: ステージ間の実行順序制御
- **リソース効率**: 並列度によるビルド時間短縮

#### アーティファクト（Artifact）
- **ビルド成果物**: ファイル、パッケージ等の出力物
- **ステップ間共有**: 後続ステップでの利用可能
- **永続化**: 指定期間の保存

## 📑 目次

1. [⚙️ 基本設定と構造](#⚙️-基本設定と構造)
2. [🔧 キャッシュとサービスコンテナー定義](#🔧-キャッシュとサービスコンテナー定義)
3. [🐳 Dockerイメージオプション](#🐳-dockerイメージオプション)
4. [🚀 Runtime v3の有効化と使用](#🚀-runtime-v3の有効化と使用)
5. [📥 Gitクローンの動作](#📥-gitクローンの動作)
6. [⚡ グローバルオプション](#⚡-グローバルオプション)
7. [🔄 並列ステップオプション](#🔄-並列ステップオプション)
8. [🎯 パイプライン開始条件](#🎯-パイプライン開始条件)
9. [📊 ステージオプション](#📊-ステージオプション)
10. [🔩 ステップオプション](#🔩-ステップオプション)
11. [🔐 変数とシークレット管理](#🔐-変数とシークレット管理)
12. [🚀 デプロイメント](#🚀-デプロイメント)
13. [🏗️ 言語・フレームワーク別完全ガイド](#🏗️-言語フレームワーク別完全ガイド)
14. [🔒 セキュリティ・認証](#🔒-セキュリティ認証)
15. [⚡ パフォーマンス最適化](#⚡-パフォーマンス最適化)
16. [🧪 テスト・品質保証](#🧪-テスト品質保証)
17. [🔄 統合・拡張](#🔄-統合拡張)
18. [💼 企業レベル実装例](#💼-企業レベル実装例)
19. [⚠️ トラブルシューティング](#⚠️-トラブルシューティング)
20. [📚 リソース・参照リンク集](#📚-リソース参照リンク集)

---

## ⚙️ 基本設定と構造

### パイプライン設定ファイルの基本構造

```yaml
# bitbucket-pipelines.yml の完全構造
image: atlassian/default-image:latest

# グローバルオプション
options:
  docker: true
  max-time: 120
  size: 2x

# Gitクローン設定
clone:
  depth: full
  lfs: true

# 定義セクション（再利用可能な設定）
definitions:
  caches:
    # キャッシュ定義
  services:
    # サービスコンテナー定義
  steps:
    # 再利用可能ステップ定義

# パイプライン定義
pipelines:
  default:
    # デフォルトパイプライン
  branches:
    # ブランチ別パイプライン
  pull-requests:
    # プルリクエスト時パイプライン
  tags:
    # タグ別パイプライン
  custom:
    # カスタム（手動）パイプライン
```

### 最小設定例

```yaml
# 最も基本的な設定
image: node:18

pipelines:
  default:
    - step:
        name: Build and Test
        script:
          - npm install
          - npm test
```

### 実用的な基本設定例

```yaml
image: node:18

options:
  docker: true
  max-time: 90

definitions:
  caches:
    nodemodules: node_modules
  services:
    postgres:
      image: postgres:13
      variables:
        POSTGRES_DB: testdb
        POSTGRES_PASSWORD: secret

pipelines:
  default:
    - step:
        name: 🔨 ビルドとテスト
        caches:
          - nodemodules
        services:
          - postgres
        script:
          - npm ci
          - npm run build
          - npm test
        artifacts:
          - dist/**
          - coverage/**

  branches:
    main:
      - step:
          name: 🚀 本番デプロイ
          deployment: production
          trigger: manual
          script:
            - npm run deploy:prod

  pull-requests:
    '**':
      - step:
          name: 🔍 PR検証
          script:
            - npm run lint
            - npm run test:ci
```

---

## 🔧 キャッシュとサービスコンテナー定義

### キャッシュの詳細設定

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

#### カスタムキャッシュ戦略

```yaml
definitions:
  caches:
    # Node.js エコシステム
    node-modules: node_modules
    npm-cache: ~/.npm
    yarn-cache: ~/.cache/yarn
    pnpm-store: ~/.pnpm-store

    # Python エコシステム
    pip-packages: ~/.cache/pip
    pipenv-venv: ~/.local/share/virtualenvs
    poetry-venv: ~/.cache/pypoetry

    # Java エコシステム
    maven-deps: ~/.m2/repository
    gradle-deps: ~/.gradle/caches
    gradle-wrapper: ~/.gradle/wrapper

    # PHP エコシステム
    composer-vendor: vendor
    composer-cache: ~/.composer/cache

    # Ruby エコシステム
    bundler-gems: vendor/bundle
    gem-cache: ~/.gem

    # Go エコシステム
    go-modules: ~/go/pkg/mod
    go-build: ~/.cache/go-build

    # .NET エコシステム
    nuget-packages: ~/.nuget/packages
    dotnet-tools: ~/.dotnet/tools

    # Frontend ツール
    cypress-cache: ~/.cache/Cypress
    playwright-cache: ~/.cache/ms-playwright
```

### サービスコンテナーの詳細設定

#### データベースサービス

##### PostgreSQL
```yaml
definitions:
  services:
    postgres:
      image: postgres:13-alpine
      variables:
        POSTGRES_DB: testdb
        POSTGRES_USER: testuser
        POSTGRES_PASSWORD: testpass
        POSTGRES_HOST_AUTH_METHOD: trust
      ports:
        - "5432:5432"
    
    postgres-with-extensions:
      image: postgis/postgis:13-3.1-alpine
      variables:
        POSTGRES_DB: gisdb
        POSTGRES_USER: gisuser
        POSTGRES_PASSWORD: gispass
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
    
    mariadb:
      image: mariadb:10.6
      variables:
        MARIADB_DATABASE: testdb
        MARIADB_USER: testuser
        MARIADB_PASSWORD: testpass
        MARIADB_ROOT_PASSWORD: rootpass
      ports:
        - "3306:3306"
```

##### MongoDB
```yaml
definitions:
  services:
    mongodb:
      image: mongo:4.4-bionic
      variables:
        MONGO_INITDB_ROOT_USERNAME: admin
        MONGO_INITDB_ROOT_PASSWORD: password
        MONGO_INITDB_DATABASE: testdb
      ports:
        - "27017:27017"
```

#### キャッシュ・メッセージングサービス

##### Redis
```yaml
definitions:
  services:
    redis:
      image: redis:6.2-alpine
      ports:
        - "6379:6379"
    
    redis-with-config:
      image: redis:6.2-alpine
      variables:
        REDIS_PASSWORD: secretpass
      ports:
        - "6379:6379"
```

##### RabbitMQ
```yaml
definitions:
  services:
    rabbitmq:
      image: rabbitmq:3.9-management-alpine
      variables:
        RABBITMQ_DEFAULT_USER: guest
        RABBITMQ_DEFAULT_PASS: guest
      ports:
        - "5672:5672"
        - "15672:15672"
```

##### Elasticsearch
```yaml
definitions:
  services:
    elasticsearch:
      image: docker.elastic.co/elasticsearch/elasticsearch:7.15.0
      variables:
        discovery.type: single-node
        ES_JAVA_OPTS: "-Xms512m -Xmx512m"
        xpack.security.enabled: "false"
      ports:
        - "9200:9200"
```

---

## 🐳 Dockerイメージオプション

### イメージの種類と認証

#### パブリックイメージ
```yaml
# 基本的なパブリックイメージ
image: node:18
image: python:3.9
image: openjdk:11
image: php:8.1-cli
image: golang:1.19
```

#### プライベートレジストリ認証

##### Docker Hub プライベートリポジトリ
```yaml
image:
  name: mycompany/private-image:latest
  username: $DOCKER_HUB_USERNAME
  password: $DOCKER_HUB_PASSWORD
```

##### AWS ECR
```yaml
image:
  name: 123456789012.dkr.ecr.us-east-1.amazonaws.com/my-repo:latest
  aws:
    access-key: $AWS_ACCESS_KEY
    secret-key: $AWS_SECRET_KEY
    
# OIDC認証
image:
  name: 123456789012.dkr.ecr.us-east-1.amazonaws.com/my-repo:latest
  aws:
    oidc-role: arn:aws:iam::123456789012:role/my-role
```

##### Google Container Registry
```yaml
image:
  name: gcr.io/my-project/my-image:latest
  username: _json_key
  password: $GCR_JSON_KEY
```

##### Azure Container Registry
```yaml
image:
  name: myregistry.azurecr.io/myapp:latest
  username: $ACR_USERNAME
  password: $ACR_PASSWORD
```

#### ユーザー指定実行
```yaml
image:
  name: my-image:latest
  run-as-user: 1000  # 特定のユーザーIDで実行
```

### マルチステージイメージ使用例

```yaml
pipelines:
  default:
    - step:
        name: 🏗️ ビルド環境
        image: node:18-alpine
        script:
          - npm ci
          - npm run build
        artifacts:
          - dist/**
    
    - step:
        name: 🧪 テスト環境
        image: node:18
        script:
          - npm run test
    
    - step:
        name: 🐳 本番イメージビルド
        image: docker:20
        services:
          - docker
        script:
          - docker build -t myapp:$BITBUCKET_BUILD_NUMBER .
```

---

## 🚀 Runtime v3の有効化と使用

### Runtime v3の特徴と有効化

#### 全体有効化
```yaml
options:
  runtime:
    cloud:
      version: "3"
```

#### ステップ別有効化
```yaml
- step:
    runtime:
      cloud:
        version: "3"
    script:
      - echo "Runtime v3で実行"
```

### 新機能の活用

#### マルチアーキテクチャビルド
```yaml
- step:
    name: 🏗️ マルチアーチビルド
    runtime:
      cloud:
        version: "3"
    services:
      - docker
    script:
      # Buildxセットアップ
      - docker buildx create --name multiarch-builder --use
      - docker buildx inspect --bootstrap
      
      # マルチプラットフォームビルド
      - |
        docker buildx build \
          --platform=linux/amd64,linux/arm64 \
          --tag myapp:latest \
          --push .
```

#### 特権コンテナーの使用
```yaml
- step:
    name: 🔧 特権操作
    runtime:
      cloud:
        version: "3"
    services:
      - docker
    script:
      # 特権が必要な操作
      - docker run --privileged alpine:latest sh -c "echo test"
      - docker run --privileged --rm -v /var/run/docker.sock:/var/run/docker.sock alpine
```

#### リモートキャッシュ活用
```yaml
- step:
    name: 📦 キャッシュ最適化ビルド
    runtime:
      cloud:
        version: "3"
    services:
      - docker
    script:
      # S3キャッシュの使用
      - |
        docker buildx build \
          --cache-from type=s3,region=us-east-1,bucket=my-cache-bucket \
          --cache-to type=s3,region=us-east-1,bucket=my-cache-bucket \
          --tag myapp:$BITBUCKET_BUILD_NUMBER .
      
      # レジストリキャッシュの使用
      - |
        docker buildx build \
          --cache-from type=registry,ref=myregistry/mycache:latest \
          --cache-to type=registry,ref=myregistry/mycache:latest \
          --tag myapp:$BITBUCKET_BUILD_NUMBER .
```

### Runtime v3限定機能

#### ARM64アーキテクチャ
```yaml
options:
  runtime:
    cloud:
      version: "3"
      arch: arm

pipelines:
  default:
    - step:
        script:
          - uname -m  # aarch64を出力
          - echo "ARM64環境で実行中"
```

---

## 📥 Gitクローンの動作

### クローン設定オプション

#### 基本設定
```yaml
clone:
  depth: full    # 完全クローン（全履歴）
  # または
  depth: 50      # シャロークローン（指定した深度）
  lfs: true      # Git LFS有効化
  enabled: true  # クローン有効化（デフォルト）
```

#### クローン無効化
```yaml
clone:
  enabled: false  # Gitクローンを完全に無効化

pipelines:
  default:
    - step:
        name: カスタムクローン
        script:
          # 独自のクローン処理
          - git clone --depth 1 $BITBUCKET_GIT_SSH_ORIGIN .
          - git fetch --depth 10
```

### ステップ別クローン設定

```yaml
pipelines:
  default:
    - step:
        name: 🔄 完全履歴必要
        clone:
          depth: full
          lfs: true
        script:
          - git log --oneline | wc -l
          - git lfs ls-files
    
    - step:
        name: ⚡ 高速クローン
        clone:
          depth: 1
        script:
          - echo "最新コミットのみ"
    
    - step:
        name: 🚫 クローン不要
        clone:
          enabled: false
        script:
          - echo "Gitリポジトリなしで実行"
```

### セルフホストランナー用設定

```yaml
# SSL検証スキップ（セルフホスト環境用）
clone:
  skip-ssl-verify: true
```

### 大容量リポジトリ最適化

```yaml
clone:
  depth: 50
  lfs: false  # LFSを無効化して高速化

pipelines:
  default:
    - step:
        name: 📊 リポジトリ情報
        script:
          - echo "リポジトリサイズ: $(du -sh .)"
          - echo "コミット数: $(git rev-list --count HEAD)"
          - echo "ブランチ: $BITBUCKET_BRANCH"
```

---

*この文書は継続的に更新されています。次のセクション（グローバルオプション）以降についても同様の品質で詳細化します。*