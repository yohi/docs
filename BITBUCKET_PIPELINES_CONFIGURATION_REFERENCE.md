# Bitbucket Pipelines 設定リファレンス - 完全ガイド

このドキュメントは、Bitbucket Pipelinesの各設定オプションについての包括的なリファレンスです。

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
- Bitbucket Pipelinesのデフォルトキャッシュは非推奨

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

#### **max-time**
```yaml
options:
  max-time: 60  # 最大実行時間（分）
```

#### **size**
```yaml
options:
  size: 2x      # リソースサイズ（1x, 2x, 4x, 8x, 16x）
```

#### **runtime**
```yaml
options:
  runtime:
    cloud:
      atlassian-ip-ranges: true
      arch: arm
```

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

### 使用例
```yaml
pipelines:
  default:
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

## 🔗 関連リンク

- [Bitbucket Pipelines公式ドキュメント](https://support.atlassian.com/bitbucket-cloud/docs/bitbucket-pipelines-configuration-reference/)
- [YAMLアンカーの使用](https://support.atlassian.com/bitbucket-cloud/docs/yaml-anchors/)
- [グロブパターンの使用](https://support.atlassian.com/bitbucket-cloud/docs/use-glob-patterns-on-the-pipelines-yaml-file/)
- [変数とシークレット](https://support.atlassian.com/bitbucket-cloud/docs/variables-and-secrets/)
