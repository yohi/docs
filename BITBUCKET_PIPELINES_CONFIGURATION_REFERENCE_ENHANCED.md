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

## ⚡ グローバルオプション

### リソース配分とタイムアウト設定

#### 基本オプション
```yaml
options:
  docker: true        # 全ステップでDockerサービス有効化
  max-time: 120       # 最大実行時間（分）
  size: 2x           # リソースサイズ
  runtime:
    cloud:
      version: "3"
      atlassian-ip-ranges: true
      arch: arm
```

#### リソースサイズ詳細表

| サイズ | CPU | メモリ | ディスク | 料金倍率 | 適用場面 |
|--------|-----|--------|----------|-----------|----------|
| 1x     | 2   | 4GB    | 64GB     | 1x        | 軽量ビルド・テスト |
| 2x     | 4   | 8GB    | 64GB     | 2x        | 標準ビルド・並列テスト |
| 4x     | 8   | 16GB   | 256GB    | 4x        | 大規模ビルド・重いテスト |
| 8x     | 16  | 32GB   | 256GB    | 8x        | エンタープライズ級 |
| 16x    | 32  | 64GB   | 256GB    | 16x       | 超大規模プロジェクト |

### 使用例とベストプラクティス

```yaml
# プロジェクト規模別推奨設定
options:
  # 小～中規模プロジェクト
  size: 2x
  max-time: 60
  docker: false  # 必要時のみ有効化

# または大規模プロジェクト
options:
  size: 4x
  max-time: 120
  docker: true
  runtime:
    cloud:
      version: "3"
      atlassian-ip-ranges: true
```

---

## 🔄 並列ステップオプション

### 基本並列実行

```yaml
pipelines:
  default:
    - step:
        name: 🏗️ ビルド
        script:
          - npm run build
        artifacts:
          - dist/**
    
    - parallel:
        fail-fast: true
        steps:
          - step:
              name: 🧪 単体テスト
              script:
                - npm run test:unit
          - step:
              name: 🔍 リント
              script:
                - npm run lint
          - step:
              name: 🔒 セキュリティスキャン
              script:
                - npm audit
```

### 高度な並列戦略

#### マトリックステスト
```yaml
- parallel:
    steps:
      - step:
          name: 🧪 Node.js 16
          image: node:16
          script:
            - npm test
      - step:
          name: 🧪 Node.js 18
          image: node:18
          script:
            - npm test
      - step:
          name: 🧪 Node.js 20
          image: node:20
          script:
            - npm test
```

#### 条件付き並列実行
```yaml
- parallel:
    steps:
      - step:
          name: 🎨 フロントエンドテスト
          condition:
            changesets:
              includePaths:
                - "frontend/**"
          script:
            - cd frontend && npm test
      
      - step:
          name: 🔧 バックエンドテスト
          condition:
            changesets:
              includePaths:
                - "backend/**"
          script:
            - cd backend && python -m pytest
```

---

## 🏗️ 言語・フレームワーク別完全ガイド

### 🟨 JavaScript/TypeScript エコシステム

#### Node.js + npm
```yaml
image: node:18-alpine

definitions:
  caches:
    nodemodules: node_modules
    npm-cache: ~/.npm

pipelines:
  default:
    - step:
        name: 📦 依存関係インストール
        caches:
          - nodemodules
          - npm-cache
        script:
          - npm ci
        artifacts:
          - node_modules/**
    
    - parallel:
        steps:
          - step:
              name: 🧪 テスト + カバレッジ
              script:
                - npm run test:coverage
                - npm run test:e2e
              artifacts:
                - coverage/**
          
          - step:
              name: 🔍 コード品質
              script:
                - npm run lint
                - npm run type-check
                - npm audit --audit-level=high
```

#### React + TypeScript プロジェクト
```yaml
image: node:18

definitions:
  caches:
    nodemodules: node_modules
    cypress: ~/.cache/Cypress

pipelines:
  default:
    - step:
        name: 🏗️ ビルドとテスト
        caches:
          - nodemodules
          - cypress
        script:
          - npm ci
          - npm run build
          - npm run test:unit
          - npm run test:integration
        artifacts:
          - build/**
          - coverage/**
    
    - step:
        name: 🎭 E2Eテスト
        caches:
          - cypress
        script:
          - npx cypress run --record --key $CYPRESS_RECORD_KEY
        artifacts:
          - cypress/screenshots/**
          - cypress/videos/**
```

#### Next.js プロジェクト
```yaml
image: node:18

pipelines:
  default:
    - step:
        name: 🚀 Next.js ビルドとテスト
        caches:
          - nodemodules
        script:
          - npm ci
          - npm run build
          - npm run test
          - npm run lint
        artifacts:
          - .next/**
          - out/**
```

### 🐍 Python エコシステム

#### Django + PostgreSQL
```yaml
image: python:3.11

definitions:
  caches:
    pip: ~/.cache/pip
    poetry: ~/.cache/pypoetry
  services:
    postgres:
      image: postgres:13-alpine
      variables:
        POSTGRES_DB: testdb
        POSTGRES_USER: testuser
        POSTGRES_PASSWORD: testpass
    redis:
      image: redis:6-alpine

pipelines:
  default:
    - step:
        name: 🐍 Django フルスイート
        caches:
          - pip
        services:
          - postgres
          - redis
        script:
          # 依存関係インストール
          - pip install --upgrade pip
          - pip install -r requirements.txt
          - pip install -r requirements-dev.txt
          
          # データベースセットアップ
          - python manage.py migrate
          - python manage.py collectstatic --noinput
          
          # テスト実行
          - python manage.py test
          - coverage run --source='.' manage.py test
          - coverage xml
          
          # コード品質
          - flake8 .
          - black --check .
          - isort --check-only .
          - bandit -r . -f json -o bandit-report.json
          
          # セキュリティチェック
          - safety check
          - python manage.py check --deploy
        artifacts:
          - coverage.xml
          - bandit-report.json
```

#### FastAPI プロジェクト
```yaml
image: python:3.11-slim

pipelines:
  default:
    - step:
        name: ⚡ FastAPI テストとビルド
        caches:
          - pip
        script:
          - pip install -r requirements.txt
          - pip install pytest pytest-cov
          - pytest --cov=app --cov-report=xml
          - uvicorn app.main:app --host 0.0.0.0 --port 8000 &
          - sleep 5
          - curl -f http://localhost:8000/health
        artifacts:
          - coverage.xml
```

### ☕ Java エコシステム

#### Spring Boot + Maven
```yaml
image: maven:3.8-openjdk-17

definitions:
  caches:
    maven: ~/.m2/repository

pipelines:
  default:
    - step:
        name: ☕ Spring Boot ビルドとテスト
        caches:
          - maven
        script:
          - mvn clean compile
          - mvn test
          - mvn package -DskipTests
          - mvn spring-boot:run &
          - sleep 30
          - curl -f http://localhost:8080/actuator/health
        artifacts:
          - target/**
```

#### Gradle プロジェクト
```yaml
image: gradle:7-jdk17

definitions:
  caches:
    gradle: ~/.gradle/caches
    gradle-wrapper: ~/.gradle/wrapper

pipelines:
  default:
    - step:
        name: 🔧 Gradle ビルドとテスト
        caches:
          - gradle
          - gradle-wrapper
        script:
          - ./gradlew clean build test
          - ./gradlew jacocoTestReport
          - ./gradlew check
        artifacts:
          - build/reports/**
          - build/libs/**
```

### 🚀 Go言語

#### Go プロジェクト完全例
```yaml
image: golang:1.21-alpine

definitions:
  caches:
    go-modules: ~/go/pkg/mod
    go-build: ~/.cache/go-build

pipelines:
  default:
    - step:
        name: 🚀 Go ビルドとテスト
        caches:
          - go-modules
          - go-build
        script:
          # 依存関係ダウンロード
          - go mod download
          - go mod verify
          
          # コード品質チェック
          - go fmt ./...
          - go vet ./...
          - |
            if [ -n "$(gofmt -l .)" ]; then
              echo "Go code is not formatted:"
              gofmt -d .
              exit 1
            fi
          
          # テスト実行
          - go test -v -race -coverprofile=coverage.out ./...
          - go tool cover -html=coverage.out -o coverage.html
          
          # ビルド（複数アーキテクチャ）
          - GOOS=linux GOARCH=amd64 go build -o bin/app-linux-amd64 .
          - GOOS=linux GOARCH=arm64 go build -o bin/app-linux-arm64 .
          - GOOS=windows GOARCH=amd64 go build -o bin/app-windows-amd64.exe .
          - GOOS=darwin GOARCH=amd64 go build -o bin/app-darwin-amd64 .
        artifacts:
          - bin/**
          - coverage.out
          - coverage.html
```

### 💎 Ruby

#### Rails プロジェクト
```yaml
image: ruby:3.1

definitions:
  caches:
    bundler: vendor/bundle
  services:
    postgres:
      image: postgres:13
      variables:
        POSTGRES_PASSWORD: password
    redis:
      image: redis:6

pipelines:
  default:
    - step:
        name: 💎 Rails テストスイート
        caches:
          - bundler
        services:
          - postgres
          - redis
        script:
          # 環境設定
          - bundle config set --local path 'vendor/bundle'
          - bundle install
          
          # データベースセットアップ
          - RAILS_ENV=test bundle exec rails db:create
          - RAILS_ENV=test bundle exec rails db:migrate
          
          # テスト実行
          - bundle exec rspec
          - bundle exec rails test
          
          # コード品質
          - bundle exec rubocop
          - bundle exec brakeman --no-pager
          
          # セキュリティ監査
          - bundle audit --update
        artifacts:
          - coverage/**
```

### 🐘 PHP

#### Laravel プロジェクト
```yaml
image: php:8.2-cli

definitions:
  caches:
    composer: ~/.composer/cache
    vendor: vendor
  services:
    mysql:
      image: mysql:8.0
      variables:
        MYSQL_DATABASE: testdb
        MYSQL_ROOT_PASSWORD: secret

pipelines:
  default:
    - step:
        name: 🐘 Laravel テストとビルド
        caches:
          - composer
          - vendor
        services:
          - mysql
        script:
          # PHP拡張インストール
          - apt-get update && apt-get install -y libzip-dev
          - docker-php-ext-install zip pdo_mysql
          
          # Composer インストール
          - curl -sS https://getcomposer.org/installer | php
          - mv composer.phar /usr/local/bin/composer
          
          # 依存関係インストール
          - composer install --no-dev --optimize-autoloader
          
          # 環境設定
          - cp .env.testing .env
          - php artisan key:generate
          
          # データベースセットアップ
          - php artisan migrate --force
          
          # テスト実行
          - php artisan test
          - ./vendor/bin/phpunit --coverage-html coverage
          
          # コード品質
          - ./vendor/bin/phpcs
          - ./vendor/bin/phpstan analyse
        artifacts:
          - coverage/**
          - vendor/**
```

---

## 💼 企業レベル実装例

### 🏢 マイクロサービスアーキテクチャ

```yaml
image: node:18

definitions:
  caches:
    nodemodules: node_modules
  services:
    postgres:
      image: postgres:13
      variables:
        POSTGRES_DB: microservices_db
        POSTGRES_USER: admin
        POSTGRES_PASSWORD: secret
    redis:
      image: redis:6-alpine
    mongodb:
      image: mongo:4.4

pipelines:
  default:
    # 並列サービスビルド
    - parallel:
        fail-fast: true
        steps:
          - step:
              name: 🔧 API Gateway
              script:
                - cd services/api-gateway
                - npm ci
                - npm run build
                - npm run test
                - npm run security-scan
              artifacts:
                - services/api-gateway/dist/**
          
          - step:
              name: 👤 User Service
              services:
                - postgres
              script:
                - cd services/user-service
                - npm ci
                - npm run test:integration
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
    
    # 統合テストとコンテナビルド
    - step:
        name: 🔬 統合テスト
        services:
          - postgres
          - mongodb
          - redis
        script:
          - npm run test:integration:all
          - npm run test:contract
    
    - step:
        name: 🐳 コンテナイメージビルド
        services:
          - docker
        script:
          # 各サービスのDockerイメージビルド
          - docker build -t api-gateway:$BITBUCKET_BUILD_NUMBER services/api-gateway
          - docker build -t user-service:$BITBUCKET_BUILD_NUMBER services/user-service
          - docker build -t order-service:$BITBUCKET_BUILD_NUMBER services/order-service
          - docker build -t payment-service:$BITBUCKET_BUILD_NUMBER services/payment-service
          
          # セキュリティスキャン
          - docker run --rm -v /var/run/docker.sock:/var/run/docker.sock \
              aquasec/trivy image api-gateway:$BITBUCKET_BUILD_NUMBER
```

### 🏭 エンタープライズJavaプロジェクト

```yaml
image: maven:3.8-openjdk-17

definitions:
  caches:
    maven: ~/.m2/repository
  services:
    oracle:
      image: gvenzl/oracle-xe:latest
      variables:
        ORACLE_PASSWORD: secret
    sonarqube:
      image: sonarqube:community

pipelines:
  default:
    - step:
        name: 🏗️ Maven ビルドと品質ゲート
        size: 4x
        caches:
          - maven
        services:
          - oracle
          - sonarqube
        script:
          # 依存関係解決
          - mvn dependency:resolve
          
          # コンパイルとテスト
          - mvn clean compile
          - mvn test
          - mvn integration-test
          
          # 品質分析
          - mvn sonar:sonar \
              -Dsonar.projectKey=$BITBUCKET_REPO_SLUG \
              -Dsonar.host.url=$SONAR_HOST_URL \
              -Dsonar.login=$SONAR_TOKEN
          
          # セキュリティチェック
          - mvn org.owasp:dependency-check-maven:check
          
          # パッケージング
          - mvn package -DskipTests
          
          # 品質ゲート確認
          - |
            quality_gate_status=$(curl -s -u $SONAR_TOKEN: \
              "$SONAR_HOST_URL/api/qualitygates/project_status?projectKey=$BITBUCKET_REPO_SLUG" \
              | jq -r '.projectStatus.status')
            if [ "$quality_gate_status" != "OK" ]; then
              echo "Quality gate failed: $quality_gate_status"
              exit 1
            fi
        artifacts:
          - target/**
          - dependency-check-report.html
```

---

## ⚠️ トラブルシューティング強化版

### 🔍 診断とデバッグツール

#### システム情報収集
```yaml
- step:
    name: 🔍 システム診断
    script:
      - echo "=== システム情報 ==="
      - uname -a
      - cat /etc/os-release
      - df -h
      - free -h
      - nproc
      
      - echo "=== 環境変数 ==="
      - env | sort
      
      - echo "=== ネットワーク ==="
      - ping -c 3 google.com
      - nslookup bitbucket.org
      
      - echo "=== Docker情報 ==="
      - docker --version || echo "Docker not available"
      - docker images || echo "Cannot list images"
```

### ❌ よくあるエラーと解決策

#### メモリ不足エラー
```yaml
# 症状: ENOMEM: not enough memory
# 解決策1: サイズ増加
options:
  size: 4x  # 2x → 4x に増加

# 解決策2: メモリ使用量監視
- step:
    script:
      - echo "=== ビルド前メモリ ==="
      - free -h
      - npm ci
      - echo "=== ビルド後メモリ ==="
      - free -h
      - npm run build
```

#### Docker認証エラー
```yaml
# 症状: pull access denied
# 解決策: 認証情報の確認とデバッグ
- step:
    script:
      - echo "Docker認証テスト"
      - echo $DOCKER_PASSWORD | docker login --username $DOCKER_USERNAME --password-stdin
      - docker pull private-registry.com/image:tag
```

#### 並列ステップでのアーティファクト問題
```yaml
# 症状: アーティファクトが見つからない
# 解決策: 明示的なダウンロード制御
- parallel:
    steps:
      - step:
          name: テスト1
          artifacts:
            download: true  # 明示的にダウンロード
          script:
            - ls dist/  # 前のアーティファクトを確認
      
      - step:
          name: テスト2（アーティファクト不要）
          artifacts:
            download: false
          script:
            - npm run lint
```

#### キャッシュが効かない問題
```yaml
# 解決策: キャッシュデバッグ
- step:
    name: キャッシュ診断
    caches:
      - nodemodules
    script:
      - echo "=== キャッシュ前 ==="
      - ls -la node_modules/ || echo "node_modules not found"
      - du -sh node_modules/ || echo "node_modules size: 0"
      
      - npm ci
      
      - echo "=== キャッシュ後 ==="
      - ls -la node_modules/
      - du -sh node_modules/
```

### 🚨 エラーハンドリングパターン

#### 段階的エラー処理
```yaml
- step:
    name: 堅牢なビルドステップ
    script:
      - set -e  # エラー時即座に終了
      
      # 必須処理（失敗時は即座に終了）
      - npm ci
      
      # オプション処理（失敗しても続行）
      - npm run optional-task || echo "Optional task failed, continuing..."
      
      # 条件付き処理
      - |
        if npm test; then
          echo "✅ テスト成功"
          npm run build
        else
          echo "❌ テスト失敗、詳細を出力"
          npm test -- --verbose
          exit 1
        fi
    
    after-script:
      # 成功・失敗に関わらず実行
      - echo "ビルド完了時刻: $(date)"
      - du -sh node_modules/ || true
```

---

## 📚 リソース・参照リンク集

### 🔧 基本・設定
- [Bitbucket Pipelines設定リファレンス](https://support.atlassian.com/bitbucket-cloud/docs/bitbucket-pipelines-configuration-reference/)
- [グローバルオプション](https://support.atlassian.com/bitbucket-cloud/docs/global-options/)
- [ステップオプション](https://support.atlassian.com/bitbucket-cloud/docs/step-options/)
- [並列ステップオプション](https://support.atlassian.com/bitbucket-cloud/docs/parallel-step-options/)

### 🐳 実行環境・コンテナ
- [Dockerイメージオプション](https://support.atlassian.com/bitbucket-cloud/docs/docker-image-options/)
- [Runtime v3有効化](https://support.atlassian.com/bitbucket-cloud/docs/enable-and-use-runtime-v3/)
- [データベース・サービスコンテナ](https://support.atlassian.com/bitbucket-cloud/docs/databases-and-service-containers/)

### 🚀 デプロイメント・統合
- [AWSデプロイガイド](https://support.atlassian.com/bitbucket-cloud/docs/deploy-on-aws-using-bitbucket-pipelines-openid-connect/)
- [Pipes統合](https://support.atlassian.com/bitbucket-cloud/docs/use-pipes-in-bitbucket-pipelines/)
- [デプロイメント](https://support.atlassian.com/bitbucket-cloud/docs/deployments/)

### 🔒 セキュリティ・認証
- [変数とシークレット](https://support.atlassian.com/bitbucket-cloud/docs/variables-and-secrets/)
- [SSH鍵の使用](https://support.atlassian.com/bitbucket-cloud/docs/using-ssh-keys-in-bitbucket-pipelines/)

### ⚡ 最適化・効率化
- [キャッシュ依存関係](https://support.atlassian.com/bitbucket-cloud/docs/cache-dependencies/)
- [アーティファクト使用](https://support.atlassian.com/bitbucket-cloud/docs/use-artifacts-in-steps/)
- [Gitクローン動作](https://support.atlassian.com/bitbucket-cloud/docs/git-clone-behavior/)

### 📊 監視・テスト
- [テスト](https://support.atlassian.com/bitbucket-cloud/docs/testing/)
- [パイプライン監視](https://support.atlassian.com/bitbucket-cloud/docs/view-your-pipeline-history-and-build-reports/)

---

## 📝 まとめ

この Enhanced Edition では、Bitbucket Pipelinesの包括的なガイドを提供しました。

### 🎯 主要な改善点
1. **視覚的理解**: アーキテクチャ図とフロー図による直感的な理解
2. **言語別完全ガイド**: Go、Ruby、PHP等の詳細な実装例
3. **企業レベル例**: マイクロサービス、エンタープライズJavaの実践例
4. **強化されたトラブルシューティング**: 具体的なエラー解決策
5. **体系化されたリンク集**: 機能別・カテゴリ別の整理

### 💡 活用のポイント
- **段階的導入**: 基本→応用→企業レベルの順で習得
- **プロジェクト特化**: 言語・フレームワークに応じた最適化
- **継続的改善**: トラブルシューティング情報の活用

このドキュメントを活用して、効率的で堅牢なCI/CDパイプラインを構築してください。