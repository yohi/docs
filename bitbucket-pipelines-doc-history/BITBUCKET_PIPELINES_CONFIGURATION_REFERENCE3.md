# Bitbucket Pipelines 包括的詳細リファレンス

## 目次
1. [概要](#概要)
2. [基本設定と構造](#基本設定と構造)
3. [設定リファレンス](#設定リファレンス)
4. [実行環境](#実行環境)
5. [言語別設定例](#言語別設定例)
6. [デプロイメント](#デプロイメント)
7. [高度な機能](#高度な機能)
8. [セキュリティ・認証](#セキュリティ認証)
9. [最適化・効率化](#最適化効率化)
10. [テスト・品質保証](#テスト品質保証)
11. [統合・拡張](#統合拡張)
12. [実際のプロジェクト例](#実際のプロジェクト例)
13. [トラブルシューティング](#トラブルシューティング)
14. [移行ガイド](#移行ガイド)
15. [ベストプラクティス](#ベストプラクティス)
16. [参照URL](#参照url)

---

## 概要

### Bitbucket Pipelinesとは

**Bitbucket Pipelines**は、Atlassianが提供するクラウドベースの継続的インテグレーション・継続的デリバリー（CI/CD）サービスです。Bitbucket Cloudに完全統合され、コードの変更を検知して自動的にビルド、テスト、デプロイメントを実行します。

#### 主要な特徴
- **クラウドネイティブ**: AWS上で動作する分散コンテナ実行基盤
- **Docker基盤**: すべての実行環境がDockerコンテナベース
- **スケーラブル**: 需要に応じた自動スケーリング
- **統合性**: Bitbucket、Jira、Confluenceとの深い連携
- **セキュリティ**: エンタープライズグレードのセキュリティ機能

#### アーキテクチャ
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

### 基本概念の詳細

#### パイプライン（Pipeline）
- **定義**: 一連のビルド・テスト・デプロイプロセスの集合
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

---

## 基本設定と構造

### ファイル構造の詳細

#### 基本構造
```yaml
# bitbucket-pipelines.yml
# Bitbucketリポジトリのルートディレクトリに配置

# グローバル設定
image: node:18-alpine
clone:
  depth: full
  lfs: true

# パイプライン定義
pipelines:
  # デフォルトパイプライン（すべてのブランチ）
  default:
    - step:
        name: "ビルドとテスト"
        script:
          - echo "デフォルトパイプライン実行"
          - npm install
          - npm test
        artifacts:
          - node_modules/**
          - dist/**
        caches:
          - node

  # ブランチ別パイプライン
  branches:
    main:
      - step:
          name: "本番デプロイ"
          deployment: production
          trigger: manual
          script:
            - npm run build:prod
            - npm run deploy:prod
    
    develop:
      - step:
          name: "開発環境デプロイ"
          deployment: staging
          script:
            - npm run build:dev
            - npm run deploy:dev
    
    feature/*:
      - step:
          name: "機能ブランチテスト"
          script:
            - npm install
            - npm run test:unit
            - npm run lint

  # プルリクエスト時
  pull-requests:
    '**':
      - step:
          name: "プルリクエスト検証"
          script:
            - npm install
            - npm run test:full
            - npm run security-scan
          condition:
            changesets:
              includePaths:
                - "src/**"
                - "tests/**"

  # タグ別パイプライン
  tags:
    'v*':
      - step:
          name: "リリースビルド"
          script:
            - npm run build:release
            - npm run package
          artifacts:
            - releases/**

  # カスタムパイプライン（手動実行）
  custom:
    hotfix-deploy:
      - variables:
          - name: ENVIRONMENT
            default: staging
            allowed-values:
              - staging
              - production
      - step:
          name: "緊急修正デプロイ"
          deployment: $ENVIRONMENT
          script:
            - echo "緊急修正を$ENVIRONMENTにデプロイ"
            - npm run deploy:$ENVIRONMENT

# 定義セクション（再利用可能な設定）
definitions:
  caches:
    nodemodules: node_modules
    cypress: ~/.cache/Cypress
  
  services:
    postgres:
      image: postgres:13
      environment:
        POSTGRES_DB: testdb
        POSTGRES_USER: testuser
        POSTGRES_PASSWORD: testpass
    
    redis:
      image: redis:6-alpine
      
  steps:
    - step: &build-step
        name: "共通ビルドステップ"
        script:
          - npm install
          - npm run build
        artifacts:
          - dist/**
        caches:
          - nodemodules
```

### 条件付き実行の詳細

#### ファイル変更ベースの条件
```yaml
step:
  name: "条件付きビルド"
  condition:
    changesets:
      # 含めるパス
      includePaths:
        - "src/**/*.{js,ts,tsx}"
        - "package.json"
        - "yarn.lock"
      # 除外パス
      excludePaths:
        - "**/*.md"
        - "docs/**"
        - "*.txt"
  script:
    - npm run build
```

#### 環境変数ベースの条件
```yaml
step:
  name: "環境別デプロイ"
  condition:
    custom:
      - $BITBUCKET_BRANCH == "main"
      - $DEPLOY_ENABLED == "true"
  script:
    - deploy.sh
```

---

## 設定リファレンス

### グローバルオプションの詳細

#### クローン設定
```yaml
clone:
  # クローンの深さ（パフォーマンス向上）
  depth: 50
  
  # Git LFS（Large File Storage）の有効化
  lfs: true
  
  # クローンを無効化（カスタムクローン時）
  enabled: false
  
  # 浅いクローンでのサブモジュール
  submodules: false
```

#### オプション設定
```yaml
options:
  # 最大実行時間（分）
  max-time: 120
  
  # デフォルトコンテナサイズ
  size: 2x
  
  # Dockerレイヤーキャッシュの有効化
  docker: true
  
  # IPv6の有効化
  ipv6: true
```

### ステップオプションの完全リファレンス

```yaml
step:
  # ステップ名
  name: "詳細ステップ設定"
  
  # Dockerイメージ
  image: 
    name: node:18-alpine
    username: $DOCKER_USERNAME
    password: $DOCKER_PASSWORD
    email: $DOCKER_EMAIL
  
  # 実行サイズ（メモリ・CPU）
  size: 2x  # 1x, 2x, 4x, 8x
  
  # 最大実行時間
  max-time: 60
  
  # 実行条件
  condition:
    changesets:
      includePaths:
        - "src/**"
      excludePaths:
        - "**/*.md"
  
  # 実行スクリプト
  script:
    - echo "スクリプト実行"
    - npm install
    - npm test
  
  # 実行後スクリプト（常に実行）
  after-script:
    - echo "クリーンアップ"
    - rm -rf temp/
  
  # アーティファクト（成果物）
  artifacts:
    - dist/**
    - logs/*.log
    - reports/coverage/**
  
  # キャッシュ
  caches:
    - node
    - docker
  
  # サービス
  services:
    - postgres
    - redis
  
  # 環境変数
  env:
    NODE_ENV: production
    API_URL: https://api.example.com
  
  # デプロイメント環境
  deployment: production
  
  # 手動トリガー
  trigger: manual
  
  # 実行するランナー
  runs-on:
    - self.hosted
    - linux
    - large
```

### 並列実行の詳細設定

```yaml
pipelines:
  default:
    # 並列ステップ
    - parallel:
        # 高速失敗（一つでも失敗したら全体を停止）
        fail-fast: true
        
        steps:
          - step:
              name: "単体テスト"
              script:
                - npm run test:unit
          
          - step:
              name: "統合テスト"
              script:
                - npm run test:integration
          
          - step:
              name: "E2Eテスト"
              size: 2x
              script:
                - npm run test:e2e
          
          - step:
              name: "セキュリティスキャン"
              script:
                - npm audit
                - npm run security:scan
    
    # 並列実行後の統合ステップ
    - step:
        name: "レポート統合"
        script:
          - npm run merge-reports
```

---

## 実行環境

### Dockerイメージ設定の詳細

#### 公式イメージの使用
```yaml
# 基本的な使用
image: node:18-alpine

# 複数バージョンのマトリックステスト
definitions:
  steps:
    - step: &test-step
        name: "Node.js テスト"
        script:
          - npm install
          - npm test

pipelines:
  default:
    - parallel:
        steps:
          - step:
              <<: *test-step
              image: node:16
              name: "Node.js 16 テスト"
          - step:
              <<: *test-step
              image: node:18
              name: "Node.js 18 テスト"
          - step:
              <<: *test-step
              image: node:20
              name: "Node.js 20 テスト"
```

#### プライベートレジストリの使用
```yaml
image:
  name: my-registry.example.com/my-app:latest
  username: $REGISTRY_USERNAME
  password: $REGISTRY_PASSWORD
  email: $REGISTRY_EMAIL

# AWS ECRの場合
image:
  name: 123456789012.dkr.ecr.us-west-2.amazonaws.com/my-app:latest
  aws:
    access-key: $AWS_ACCESS_KEY_ID
    secret-key: $AWS_SECRET_ACCESS_KEY
    region: us-west-2
```

### サービスコンテナの詳細設定

#### データベースサービス
```yaml
definitions:
  services:
    # PostgreSQL
    postgres:
      image: postgres:13-alpine
      environment:
        POSTGRES_DB: myapp_test
        POSTGRES_USER: testuser
        POSTGRES_PASSWORD: testpass
        POSTGRES_INITDB_ARGS: "--encoding=UTF-8 --locale=C"
    
    # MySQL
    mysql:
      image: mysql:8.0
      environment:
        MYSQL_DATABASE: myapp_test
        MYSQL_USER: testuser
        MYSQL_PASSWORD: testpass
        MYSQL_ROOT_PASSWORD: rootpass
        MYSQL_CHARACTER_SET_SERVER: utf8mb4
        MYSQL_COLLATION_SERVER: utf8mb4_unicode_ci
    
    # MongoDB
    mongo:
      image: mongo:5.0
      environment:
        MONGO_INITDB_ROOT_USERNAME: root
        MONGO_INITDB_ROOT_PASSWORD: rootpass
        MONGO_INITDB_DATABASE: myapp_test
    
    # Redis
    redis:
      image: redis:6-alpine
      command: ["redis-server", "--appendonly", "yes"]
    
    # Elasticsearch
    elasticsearch:
      image: elasticsearch:7.15.0
      environment:
        discovery.type: single-node
        ES_JAVA_OPTS: "-Xms512m -Xmx512m"
    
    # RabbitMQ
    rabbitmq:
      image: rabbitmq:3.9-management-alpine
      environment:
        RABBITMQ_DEFAULT_USER: guest
        RABBITMQ_DEFAULT_PASS: guest

pipelines:
  default:
    - step:
        name: "フルスタックテスト"
        services:
          - postgres
          - redis
          - elasticsearch
        script:
          # サービス起動待機
          - sleep 10
          
          # 接続テスト
          - pg_isready -h localhost -p 5432
          - redis-cli -h localhost ping
          - curl -f http://localhost:9200/_cluster/health
          
          # アプリケーションテスト
          - npm run test:integration
```

### Runtime v3の詳細

Runtime v3は、Bitbucket Pipelinesの最新実行環境で、パフォーマンスとセキュリティが向上しています。

```yaml
version: "3"

options:
  runtime:
    cloud:
      # リソース制限
      memory: 4096  # MB

pipelines:
  default:
    - step:
        name: "Runtime v3 テスト"
        # ステップレベルでの設定
        runs-on:
          - 'linux.medium'
        script:
          - echo "Runtime v3で実行中"
          - uname -a
          - free -m
```

---

## 言語別設定例

### Node.js プロジェクト

#### 基本設定
```yaml
image: node:18-alpine

definitions:
  caches:
    nodemodules: node_modules
    cypress: ~/.cache/Cypress
    eslint: ~/.eslintcache

pipelines:
  default:
    - step:
        name: "Node.js ビルドとテスト"
        caches:
          - nodemodules
        script:
          # Node.js バージョン確認
          - node --version
          - npm --version
          
          # 依存関係インストール
          - npm ci
          
          # リンティング
          - npm run lint
          
          # 単体テスト
          - npm run test:unit -- --coverage
          
          # ビルド
          - npm run build
          
          # E2Eテスト
          - npm run test:e2e
        artifacts:
          - dist/**
          - coverage/**
          - cypress/screenshots/**
          - cypress/videos/**
        services:
          - postgres

  branches:
    main:
      - step:
          name: "本番デプロイ"
          image: node:18-alpine
          script:
            - npm ci --production
            - npm run build:production
            - npm run deploy
          artifacts:
            - dist/**
```

#### パッケージマネージャー別設定
```yaml
# Yarn使用時
image: node:18-alpine
definitions:
  caches:
    yarn: ~/.yarn/cache

pipelines:
  default:
    - step:
        name: "Yarn ビルド"
        caches:
          - yarn
        script:
          - yarn install --frozen-lockfile
          - yarn test
          - yarn build

# pnpm使用時
image: node:18-alpine
pipelines:
  default:
    - step:
        name: "pnpm ビルド"
        script:
          - npm install -g pnpm
          - pnpm install --frozen-lockfile
          - pnpm test
          - pnpm build
```

### Python プロジェクト

```yaml
image: python:3.11-slim

definitions:
  caches:
    pip: ~/.cache/pip
    pytest: .pytest_cache
  
  services:
    postgres:
      image: postgres:13
      environment:
        POSTGRES_DB: testdb
        POSTGRES_USER: testuser
        POSTGRES_PASSWORD: testpass

pipelines:
  default:
    - step:
        name: "Python テストとリンティング"
        caches:
          - pip
          - pytest
        services:
          - postgres
        script:
          # Python環境設定
          - python --version
          - pip install --upgrade pip setuptools wheel
          
          # 依存関係インストール
          - pip install -r requirements.txt
          - pip install -r requirements-dev.txt
          
          # コード品質チェック
          - flake8 src/ tests/
          - black --check src/ tests/
          - isort --check-only src/ tests/
          - mypy src/
          
          # セキュリティチェック
          - bandit -r src/
          - safety check
          
          # テスト実行
          - pytest tests/ --cov=src --cov-report=xml --cov-report=html
          
          # パッケージビルド
          - python setup.py sdist bdist_wheel
        artifacts:
          - dist/**
          - htmlcov/**
          - coverage.xml

  branches:
    main:
      - step:
          name: "PyPI デプロイ"
          script:
            - pip install twine
            - python setup.py sdist bdist_wheel
            - twine upload dist/*
          condition:
            changesets:
              includePaths:
                - "setup.py"
                - "src/**"
```

### Java/Maven プロジェクト

```yaml
image: maven:3.8.4-openjdk-17

definitions:
  caches:
    maven: ~/.m2/repository
    
  services:
    postgres:
      image: postgres:13
      environment:
        POSTGRES_DB: testdb
        POSTGRES_USER: testuser
        POSTGRES_PASSWORD: testpass

pipelines:
  default:
    - step:
        name: "Maven ビルドとテスト"
        caches:
          - maven
        services:
          - postgres
        script:
          # Java環境確認
          - java -version
          - mvn -version
          
          # 依存関係解決
          - mvn dependency:resolve
          
          # コンパイルとテスト
          - mvn clean compile test
          
          # コード品質チェック
          - mvn checkstyle:check
          - mvn spotbugs:check
          
          # 統合テスト
          - mvn verify
          
          # パッケージ作成
          - mvn package
          
          # SonarQube解析
          - mvn sonar:sonar -Dsonar.projectKey=$SONAR_PROJECT_KEY -Dsonar.host.url=$SONAR_HOST_URL -Dsonar.login=$SONAR_TOKEN
        artifacts:
          - target/*.jar
          - target/site/**

  branches:
    main:
      - step:
          name: "Docker イメージビルド"
          services:
            - docker
          script:
            - mvn package -DskipTests
            - docker build -t my-app:$BITBUCKET_BUILD_NUMBER .
            - docker tag my-app:$BITBUCKET_BUILD_NUMBER my-registry.com/my-app:latest
            - docker push my-registry.com/my-app:latest
```

### Go プロジェクト

```yaml
image: golang:1.21-alpine

definitions:
  caches:
    gomodules: /go/pkg/mod

pipelines:
  default:
    - step:
        name: "Go ビルドとテスト"
        caches:
          - gomodules
        script:
          # Go環境設定
          - go version
          - go env
          
          # 依存関係ダウンロード
          - go mod download
          - go mod verify
          
          # コードフォーマットチェック
          - test -z $(gofmt -l .)
          
          # リンティング
          - go install github.com/golangci/golangci-lint/cmd/golangci-lint@latest
          - golangci-lint run
          
          # テスト実行
          - go test -v -race -coverprofile=coverage.out ./...
          - go tool cover -html=coverage.out -o coverage.html
          
          # ビルド
          - go build -o bin/app ./cmd/app
          
          # クロスコンパイル
          - GOOS=linux GOARCH=amd64 go build -o bin/app-linux-amd64 ./cmd/app
          - GOOS=windows GOARCH=amd64 go build -o bin/app-windows-amd64.exe ./cmd/app
          - GOOS=darwin GOARCH=amd64 go build -o bin/app-darwin-amd64 ./cmd/app
        artifacts:
          - bin/**
          - coverage.html

  branches:
    main:
      - step:
          name: "Docker デプロイ"
          services:
            - docker
          script:
            - docker build -t my-go-app:$BITBUCKET_BUILD_NUMBER .
            - docker push my-registry.com/my-go-app:latest
```

### Ruby プロジェクト

```yaml
image: ruby:3.2-alpine

definitions:
  caches:
    bundler: vendor/bundle
    
  services:
    postgres:
      image: postgres:13
      environment:
        POSTGRES_DB: myapp_test
        POSTGRES_USER: postgres
        POSTGRES_PASSWORD: postgres
    
    redis:
      image: redis:6-alpine

pipelines:
  default:
    - step:
        name: "Ruby テスト"
        caches:
          - bundler
        services:
          - postgres
          - redis
        script:
          # Ruby環境設定
          - ruby -v
          - gem -v
          - bundler -v
          
          # 依存関係インストール
          - bundle config set --local path 'vendor/bundle'
          - bundle install --jobs=4 --retry=3
          
          # データベースセットアップ
          - bundle exec rake db:create db:migrate
          
          # リンティング
          - bundle exec rubocop
          
          # セキュリティチェック
          - bundle exec brakeman
          - bundle audit check --update
          
          # テスト実行
          - bundle exec rspec --format RspecJunitFormatter --out rspec.xml
          
          # Railsアセットプリコンパイル（Rails アプリの場合）
          - bundle exec rake assets:precompile
        artifacts:
          - rspec.xml
          - coverage/**
          - public/assets/**

  branches:
    main:
      - step:
          name: "Heroku デプロイ"
          script:
            - pipe: atlassian/heroku-deploy:1.1.1
              variables:
                HEROKU_API_KEY: $HEROKU_API_KEY
                HEROKU_APP_NAME: $HEROKU_APP_NAME
```

### PHP プロジェクト

```yaml
image: php:8.2-cli

definitions:
  caches:
    composer: ~/.composer/cache
    
  services:
    mysql:
      image: mysql:8.0
      environment:
        MYSQL_DATABASE: laravel_test
        MYSQL_USER: testuser
        MYSQL_PASSWORD: testpass
        MYSQL_ROOT_PASSWORD: rootpass

pipelines:
  default:
    - step:
        name: "PHP テスト"
        caches:
          - composer
        services:
          - mysql
        script:
          # PHP拡張インストール
          - docker-php-ext-install pdo pdo_mysql
          
          # Composerインストール
          - curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer
          
          # PHP環境確認
          - php -v
          - composer -V
          
          # 依存関係インストール
          - composer install --no-interaction --prefer-dist --optimize-autoloader
          
          # コードスタイルチェック
          - ./vendor/bin/phpcs --standard=PSR12 src/
          
          # 静的解析
          - ./vendor/bin/phpstan analyse src/
          
          # テスト実行
          - ./vendor/bin/phpunit --coverage-html coverage/ --coverage-clover coverage.xml
          
          # Laravel固有（Laravelプロジェクトの場合）
          - cp .env.example .env
          - php artisan key:generate
          - php artisan migrate --force
          - php artisan test
        artifacts:
          - coverage/**
          - coverage.xml

  branches:
    main:
      - step:
          name: "デプロイ"
          script:
            - composer install --no-dev --optimize-autoloader
            - php artisan config:cache
            - php artisan route:cache
            - php artisan view:cache
            - ./deploy.sh
```

---

## デプロイメント

### AWS デプロイメント

#### S3 静的ウェブサイト
```yaml
pipelines:
  branches:
    main:
      - step:
          name: "S3 デプロイ"
          script:
            - npm run build
            - pipe: atlassian/aws-s3-deploy:1.1.0
              variables:
                AWS_ACCESS_KEY_ID: $AWS_ACCESS_KEY_ID
                AWS_SECRET_ACCESS_KEY: $AWS_SECRET_ACCESS_KEY
                AWS_DEFAULT_REGION: us-west-2
                S3_BUCKET: my-website-bucket
                LOCAL_PATH: dist
                ACL: public-read
                CACHE_CONTROL: max-age=3600
                EXTRA_ARGS: --delete
                DEBUG: "true"

      - step:
          name: "CloudFront キャッシュ無効化"
          script:
            - pipe: atlassian/aws-cloudfront-invalidate:0.6.0
              variables:
                DISTRIBUTION_ID: $CLOUDFRONT_DISTRIBUTION_ID
                INVALIDATION_PATHS: "/*"
