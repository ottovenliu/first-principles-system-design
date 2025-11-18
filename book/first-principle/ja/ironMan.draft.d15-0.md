# Day 15 | 完全なCI/CD自動化実装 - GitHub Actions × CodePipeline × CodeBuild: 継続的インテグレーションとデプロイメントパイプラインおよびタスクセグメンテーション管理

現代のソフトウェア開発において、コードの品質だけでなく、配信プロセス全体の信頼性を確保する必要があります。<バージョン管理戦略(PRレビュー戦略)>で述べた内容を覚えていますか：`各PRを「製品提案」のように想像してください。厳格な「レビュープロセス」を経て初めて、会社の公式製品ライン(メインブランチへのマージ)に組み込まれることができます...`？

```mermaid
flowchart TD
    A[開発者がPRを提出] --> B{自動チェック}
    B -->|失敗| C[問題を修正]
    C --> A
    B -->|成功| D[レビュアーを割り当て]

    subgraph "開発者側"
      P1[開発者セルフチェック]
      P2[ローカルテストを実行]
      P3[コードをフォーマット]
      P4[コミットメッセージ規約を確認]

      P1 --> P2
      P2 --> P3
      P3 --> P4
      P4 --> A
    end

    subgraph "レビュアー側"
      D[レビュアーを割り当て] --> E[テックリードレビュー]
      D --> F[シニアエンジニアレビュー]
      D --> G[プロダクトオーナーレビュー]

      E --> H{技術レビューに合格?}
      F --> I{コード品質レビューに合格?}
      G --> J{ビジネスロジックレビューに合格?}

      H -->|いいえ| K[変更を要求]
      I -->|いいえ| K
      J -->|いいえ| K
      K --> A

      H -->|はい| L{すべてのレビューに合格?}
      I -->|はい| L
      J -->|はい| L

      L -->|いいえ| M[他のレビューを待機中]
      L -->|はい| N[最終承認]
    end

    N --> O{コンフリクトチェック}
    O -->|コンフリクトあり| P[コンフリクトを解決]
    P --> A
    O -->|コンフリクトなし| Q[ターゲットブランチにマージ]

    Q --> R[自動デプロイ]
    Q --> S[ステークホルダーに通知]

    style A fill:#e3f2fd
    style Q fill:#e8f5e8
    style K fill:#ffebee
    style R fill:#f3e5f5
```

この章では、フローチャートで**自動チェックと自動デプロイ(CI/CD)**について簡単に触れました。しかし、その時点では、このプロセスの実行方法については詳しく説明せず、Gitつまり開発成果の段階的品質管理の分析に焦点を当てました。しかし、このプロセスの一部では、ローカル提出ごとに費やす時間を削減し、`リリース環境機能テスト`、`システム間統合テスト`、`e2eテスト`などのステップを独立して実行される自動化環境で検出することができます。これが**CI(継続的インテグレーション)**の主な機能の1つです。

このプロセスは、毎回ローカルでテストを実行するコストを排除し、開発環境とリリース環境の違いを可能な限り最小限に抑えます(私の開発経験では、ローカルではテストが失敗したが製品側では問題なかった、またはその逆の状況に遭遇したことがあります)。同時に、実行されるすべてのTaskとJobは、**ビジネスロジック実装**の**アクティブな保護**でもあります。そこで今日は、GitHub Actions、AWS CodePipeline、CodeBuildを通じてエンタープライズレベルのCI/CD自動化パイプラインを構築する方法を深く掘り下げ、特に以下に焦点を当てます：

- **Jobsの断片化管理**: CI/CDタスクを合理的に分割し整理する方法。
- **タスクバージョン管理**: CI/CDプロセスをバージョン管理する方法。
- **パラメータ抽象化管理**: 異なる環境の設定パラメータを管理する方法。
- **ビジネスレビューゲート**: 自動化プロセスに手動レビューメカニズムを追加する方法。

まず、CI/CD時代以前のソフトウェアデプロイメントの困難を振り返りましょう：

**シナリオ1: 手動デプロイメントの恐怖**

```
金曜日、午後5時...

開発者: 「新機能を本番環境にデプロイしようとしています。」
同僚: 「待って！金曜日にデプロイしないで！」
開発者: 「なぜですか？」
同僚: 「何か問題が起きたら、週末ずっと残業しなきゃいけなくなるから...」

**核心的な問題**:
- デプロイメントプロセスが手動操作に依存し、エラーが発生しやすい。
- 標準化されたプロセスがなく、すべてのデプロイメントがリスクになる。
- ロールバックが困難で、問題の影響が大きい。
- チームはデプロイメントに自信がない。
```

**シナリオ2: 一貫性のない環境の災害**

```
開発者: 「おかしいな、私のマシンでは正常に動作するのに...」
テスター: 「でもテスト環境にはバグがあります。」
運用エンジニア: 「そして本番環境はテスト環境と異なります...」
プロダクトマネージャー: 「いつ修正できますか？お客様が待っています！」

**核心的な問題**:
- 開発、テスト、本番環境の設定が一貫していない。
- 依存関係管理が混乱し、バージョンの競合が頻繁に発生。
- 自動化されたテストがなく、問題の発見が遅すぎる。
- 環境セットアップに時間がかかり、開発効率に影響。
```

**シナリオ3: デプロイメントプロセスのブラックボックス**

```
新人エンジニア: 「コードをデプロイするにはどうすればいいですか？」
シニアエンジニア: 「うーん...考えさせてください...」
「まず、サーバーにSSHする必要があります...」
「それからgit pullして...」
「それから再コンパイルして...」
「nginxを再起動することを忘れずに...」
「あと、キャッシュもクリアして...」
「あ、データベースのバックアップも忘れずに...」

新人: 😵‍💫

**核心的な問題**:
- デプロイメント知識が少数の人に集中している。
- プロセスが複雑でドキュメントが不足している。
- 新しいチームメンバーの学習コストが高い。
- キーパーソンが退職するとリスクが高い。
```

これらのソフトウェア配信の痛点が、<Infrastructure as Code: Terraformでインフラストラクチャをコード化しバージョン管理する>の章のシナリオと非常に似ていることに気づきましたか？**安定した環境**と**固定されたビジネスロジックテストプロセス**なしでテストとデプロイメントを行う必要がある場合、開発チームはすべてのデプロイメントで薄氷を踏む思いをしやすくなります。**CI(継続的インテグレーション)**と**CD(継続的デプロイメント/デリバリー)**が必要な最も重要な理由は、安定した固定されたビジネスロジック検証と配信プロセスを確立し、**`既存のビジネスロジックがコード変更によって破壊されるのをアクティブに保護する`**ことです。

## CI/CDの核心概念と価値

```
既存のビジネスロジックがコード変更によって破壊されるのを保護する。
```

### CI/CDとは何か？

**CI(継続的インテグレーション)**: 開発者が頻繁にコード変更をメインブランチにマージし、各マージが自動化されたビルドとテストプロセスをトリガーします。

**CD(継続的デプロイメント/デリバリー)**: 自動化されたプロセスを通じて、テストに合格したコードを自動的にさまざまな環境にデプロイし、さらには本番環境に直接デプロイします。

### CI/CDの核心価値

```mermaid
graph TD
    A[CI/CDの核心価値] --> B[迅速なフィードバック]
    A --> C[リスク削減]
    A --> D[効率向上]
    A --> E[品質保証]

    B --> B1[早期の問題検出]
    B --> B2[迅速な欠陥修正]

    C --> C1[小バッチリリース]
    C --> C2[迅速なロールバック機能]

    D --> D1[自動化が手動作業に取って代わる]
    D --> D2[並列プロセス処理]

    E --> E1[自動化されたテストカバレッジ]
    E --> E2[一貫した環境]
```

想像してください：`「手作業工房」から「自動化工場」への変革。`

過去のソフトウェア開発は手作業工房のようなものでした：

- **手作り**: 各製品(ソフトウェアバージョン)には職人(開発者)が手作業で作成する必要がありました。
- **不安定な品質**: その日の職人のスキルと状態に依存していました。
- **限られた生産量**: 職人が1日に生産できる製品の数は限られていました。
- **集中した知識**: スキルは少数の職人が保有していました。

そしてCI/CDは自動化された生産ラインを構築するようなものです：

- **標準化されたプロセス**: すべてのステップに明確な基準と自動化されたチェックがあります。
- **品質保証**: すべての製品が同じ品質検査プロセスを経ます。
- **高効率**: 機械は疲れず、24時間365日稼働できます。
- **知識の共有**: プロセスがコード化されているため、誰でも理解し改善できます。

## GitHub Actionsの深い実践

GitHub ActionsはGitHubが提供するCI/CDプラットフォームで、コードリポジトリで直接自動化されたワークフローを定義し実行することができます。

### GitHub Actionsの核心概念

```mermaid
graph LR
    A[GitHub Actionsアーキテクチャ] --> B[Workflow]
    A --> C[Job]
    A --> D[Step]
    A --> E[Action]

    B --> B1[.github/workflows/*.yml]
    B --> B2[トリガー条件の定義]

    C --> C1[Runner環境の選択]
    C --> C2[並列実行のサポート]

    D --> D1[特定の実行コマンド]
    D --> D2[環境変数の設定]

    E --> E1[再利用可能なコードユニット]
    E --> E2[Marketplaceエコシステム]
```

### Job断片化管理戦略

エンタープライズレベルのCI/CDプロセスにおいて、合理的なjob分割は非常に重要です。実行効率とメンテナンス性のバランスを取る必要があります：

#### 1. 基本的なJob分割の原則

```yaml
# .github/workflows/ci-cd.yml
name: "Frontend CI/CD Pipeline"

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]
  workflow_dispatch:

env:
  NODE_VERSION: "18"
  PNPM_VERSION: "8.15.0"

jobs:
  # 第1層: 基本チェック(早期失敗)
  code-quality:
    name: "Code Quality Check"
    runs-on: ubuntu-latest
    timeout-minutes: 10

    outputs:
      cache-key: ${{ steps.cache-key.outputs.key }}

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Generate cache key
        id: cache-key
        run: |
          echo "key=node-${{ env.NODE_VERSION }}-${{ hashFiles('**/pnpm-lock.yaml') }}" >> $GITHUB_OUTPUT

      - name: Setup Node.js and dependencies
        uses: ./.github/actions/setup-node
        with:
          node-version: ${{ env.NODE_VERSION }}
          pnpm-version: ${{ env.PNPM_VERSION }}
          cache-key: ${{ steps.cache-key.outputs.key }}

      - name: Lint and format check
        run: |
          pnpm run lint:check
          pnpm run format:check
          pnpm run type-check

  # 第2層: テスト実行(並列処理)
  unit-tests:
    name: "Unit Tests"
    runs-on: ubuntu-latest
    needs: code-quality
    timeout-minutes: 15

    strategy:
      matrix:
        node-version: [16, 18, 20]

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup Node.js and dependencies
        uses: ./.github/actions/setup-node
        with:
          node-version: ${{ matrix.node-version }}
          pnpm-version: ${{ env.PNPM_VERSION }}
          cache-key: ${{ needs.code-quality.outputs.cache-key }}

      - name: Run unit tests
        run: pnpm run test:unit --coverage

      - name: Upload coverage
        if: matrix.node-version == '18'
        uses: codecov/codecov-action@v4
        with:
          token: ${{ secrets.CODECOV_TOKEN }}

  integration-tests:
    name: "Integration Tests"
    runs-on: ubuntu-latest
    needs: code-quality
    timeout-minutes: 20

    services:
      postgres:
        image: postgres:15
        env:
          POSTGRES_PASSWORD: postgres
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup Node.js and dependencies
        uses: ./.github/actions/setup-node
        with:
          node-version: ${{ env.NODE_VERSION }}
          pnpm-version: ${{ env.PNPM_VERSION }}
          cache-key: ${{ needs.code-quality.outputs.cache-key }}

      - name: Setup test database
        run: |
          pnpm run db:migrate:test
          pnpm run db:seed:test
        env:
          DATABASE_URL: postgresql://postgres:postgres@localhost:5432/test_db

      - name: Run integration tests
        run: pnpm run test:integration
        env:
          DATABASE_URL: postgresql://postgres:postgres@localhost:5432/test_db

  e2e-tests:
    name: "E2E Tests"
    runs-on: ubuntu-latest
    needs: code-quality
    timeout-minutes: 30

    strategy:
      matrix:
        browser: [chromium, firefox]
        shard: [1/4, 2/4, 3/4, 4/4]

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup Node.js and dependencies
        uses: ./.github/actions/setup-node
        with:
          node-version: ${{ env.NODE_VERSION }}
          pnpm-version: ${{ env.PNPM_VERSION }}
          cache-key: ${{ needs.code-quality.outputs.cache-key }}

      - name: Install Playwright
        run: npx playwright install --with-deps ${{ matrix.browser }}

      - name: Build application
        run: pnpm run build:test

      - name: Run E2E tests
        run: |
          pnpm run start:test &
          npx wait-on http://localhost:3000
          npx playwright test --project=${{ matrix.browser }} --shard=${{ matrix.shard }}

      - name: Upload test results
        if: failure()
        uses: actions/upload-artifact@v4
        with:
          name: e2e-results-${{ matrix.browser }}-${{ matrix.shard }}
          path: playwright-report/

  # 第3層: セキュリティと品質チェック
  security-scan:
    name: "Security Scan"
    runs-on: ubuntu-latest
    needs: code-quality
    timeout-minutes: 15

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Run security audit
        run: |
          npx audit-ci --config .audit-ci.json

      - name: Run SAST scan
        uses: github/super-linter@v4
        env:
          DEFAULT_BRANCH: main
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          VALIDATE_JAVASCRIPT_ES: true
          VALIDATE_TYPESCRIPT_ES: true

      - name: Run Trivy vulnerability scanner
        uses: aquasecurity/trivy-action@master
        with:
          scan-type: "fs"
          scan-ref: "."
          format: "sarif"
          output: "trivy-results.sarif"

      - name: Upload security scan results
        uses: github/codeql-action/upload-sarif@v3
        with:
          sarif_file: "trivy-results.sarif"

  # 第4層: ビルドとデプロイメント準備
  build:
    name: "Build Application"
    runs-on: ubuntu-latest
    needs: [unit-tests, integration-tests, e2e-tests, security-scan]
    timeout-minutes: 10

    outputs:
      build-version: ${{ steps.version.outputs.version }}

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Generate version
        id: version
        run: |
          VERSION=$(date +%Y%m%d-%H%M%S)-${GITHUB_SHA:0:7}
          echo "version=$VERSION" >> $GITHUB_OUTPUT
          echo "::notice::Build version: $VERSION"

      - name: Setup Node.js and dependencies
        uses: ./.github/actions/setup-node
        with:
          node-version: ${{ env.NODE_VERSION }}
          pnpm-version: ${{ env.PNPM_VERSION }}
          cache-key: ${{ needs.code-quality.outputs.cache-key }}

      - name: Build application
        run: |
          pnpm run build
          echo ${{ steps.version.outputs.version }} > dist/version.txt
        env:
          VITE_BUILD_VERSION: ${{ steps.version.outputs.version }}
          VITE_BUILD_TIME: ${{ github.event.head_commit.timestamp }}
          VITE_COMMIT_SHA: ${{ github.sha }}

      - name: Validate build
        run: |
          # キーファイルをチェック
          test -f dist/index.html || { echo "❌ index.html not found"; exit 1; }
          test -f dist/version.txt || { echo "❌ version.txt not found"; exit 1; }

          # ビルドサイズをチェック
          BUNDLE_SIZE=$(du -sk dist | cut -f1)
          echo "Bundle size: ${BUNDLE_SIZE}KB"

          if [ $BUNDLE_SIZE -gt 20480 ]; then
            echo "⚠️ Bundle size warning: ${BUNDLE_SIZE}KB > 20MB"
          fi

      - name: Upload build artifacts
        uses: actions/upload-artifact@v4
        with:
          name: build-${{ steps.version.outputs.version }}
          path: dist/
          retention-days: 7

  # 第5層: ステージング環境へのデプロイ
  deploy-staging:
    name: "Deploy to Staging"
    runs-on: ubuntu-latest
    needs: build
    if: github.ref == 'refs/heads/develop'
    timeout-minutes: 10

    environment:
      name: staging
      url: https://staging.myapp.com

    steps:
      - name: Download build artifacts
        uses: actions/download-artifact@v4
        with:
          name: build-${{ needs.build.outputs.build-version }}
          path: dist/

      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: ap-northeast-1

      - name: Deploy to S3
        run: |
          aws s3 sync dist/ s3://${{ secrets.STAGING_S3_BUCKET }} --delete
          aws cloudfront create-invalidation --distribution-id ${{ secrets.STAGING_CLOUDFRONT_ID }} --paths "/*"

      - name: Run deployment verification
        run: |
          sleep 30
          HTTP_CODE=$(curl -s -o /dev/null -w "%{http_code}" https://staging.myapp.com)
          if [ $HTTP_CODE -ne 200 ]; then
            echo "❌ Deployment verification failed (HTTP $HTTP_CODE)"
            exit 1
          fi
          echo "✅ Staging deployment verified"

  # 第6層: ビジネスレビューゲート
  approval-gate:
    name: "Production Approval"
    runs-on: ubuntu-latest
    needs: [build, deploy-staging]
    if: github.ref == 'refs/heads/main'
    timeout-minutes: 1440 # 24時間タイムアウト

    environment:
      name: production-approval

    steps:
      - name: Request production deployment approval
        run: |
          echo "🔍 Production deployment approval requested"
          echo "📋 Build version: ${{ needs.build.outputs.build-version }}"
          echo "🌐 Staging URL: https://staging.myapp.com"
          echo "⏰ Approval timeout: 24 hours"

  # 第7層: 本番デプロイメント
  deploy-production:
    name: "Deploy to Production"
    runs-on: ubuntu-latest
    needs: [build, approval-gate]
    timeout-minutes: 15

    environment:
      name: production
      url: https://myapp.com

    steps:
      - name: Download build artifacts
        uses: actions/download-artifact@v4
        with:
          name: build-${{ needs.build.outputs.build-version }}
          path: dist/

      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: ap-northeast-1

      - name: Backup current version
        run: |
          BACKUP_TIMESTAMP=$(date +%Y%m%d-%H%M%S)
          aws s3 sync s3://${{ secrets.PRODUCTION_S3_BUCKET }} s3://${{ secrets.BACKUP_S3_BUCKET }}/${BACKUP_TIMESTAMP}/
          echo "BACKUP_PATH=${BACKUP_TIMESTAMP}" >> $GITHUB_ENV

      - name: Deploy to production
        run: |
          aws s3 sync dist/ s3://${{ secrets.PRODUCTION_S3_BUCKET }} --delete
          aws cloudfront create-invalidation --distribution-id ${{ secrets.PRODUCTION_CLOUDFRONT_ID }} --paths "/*"

      - name: Production health check
        run: |
          echo "Waiting for CDN propagation..."
          sleep 60

          for i in {1..5};
          do
            HTTP_CODE=$(curl -s -o /dev/null -w "%{http_code}" https://myapp.com)
            if [ $HTTP_CODE -eq 200 ]; then
              echo "✅ Production health check passed (attempt $i)"
              break
            elif [ $i -eq 5 ]; then
              echo "❌ Production health check failed after 5 attempts"
              echo "🔄 Rolling back..."
              aws s3 sync s3://${{ secrets.BACKUP_S3_BUCKET }}/${{ env.BACKUP_PATH }}/ s3://${{ secrets.PRODUCTION_S3_BUCKET }} --delete
              aws cloudfront create-invalidation --distribution-id ${{ secrets.PRODUCTION_CLOUDFRONT_ID }} --paths "/*"
              exit 1
            else
              echo "⏳ Health check attempt $i failed, retrying..."
              sleep 30
            fi
          done

      - name: Notify deployment success
        if: success()
        uses: 8398a7/action-slack@v3
        with:
          status: success
          channel: "#deployment"
          text: |
            🚀 Production deployment successful!
            📦 Version: ${{ needs.build.outputs.build-version }}
            🌐 URL: https://myapp.com
            👤 Deployed by: ${{ github.actor }}
        env:
          SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK_URL }}

      - name: Notify deployment failure
        if: failure()
        uses: 8398a7/action-slack@v3
        with:
          status: failure
          channel: "#deployment"
          text: |
            ❌ Production deployment failed!
            📦 Version: ${{ needs.build.outputs.build-version }}
            🔄 Automatic rollback initiated
            👤 Attempted by: ${{ github.actor }}
        env:
          SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK_URL }}
```

#### 2. エンタープライズレベルのJobモジュール化とクロスドメイン参照

単一のワークフローファイル内でjobsを分割するだけでなく、エンタープライズレベルの共有標準とメンテナンス性のために、特定のドメインまたは機能に属するjobsを異なる再利用可能なワークフローに分離することができます。必要に応じて、メインワークフローはライブラリのようにこれらの独立したjobsを参照し実行できます。

このアプローチの最大の利点は、異なるシステム間のドメインインタラクションロジックを相互に検証できることです。たとえば、`user-service`のCIプロセスが更新された場合、`order-service`のCIプロセスは、`user-service`のテストjobを参照することで、この変更が注文プロセスのユーザー関連機能を破壊していないことを確認できます。これにより「私のドメインでは問題ないが、他のドメインに影響する」というジレンマを回避できます。

**シナリオ: User Serviceの変更がOrder Serviceに影響しないことを確認する必要がある。**

まず、`user-service`の再利用可能なCIワークフローを定義します。これには、コア機能を検証するための専用jobが含まれています。

```yaml
# .github/workflows/reusable-user-service-ci.yml
name: "Reusable User Service CI"

on:
  workflow_call:
    inputs:
      node-version:
        required: false
        type: string
        default: "18"
    outputs:
      test-results-summary:
        description: "A summary of the user service test results"
        value: ${{ jobs.validate-user-core.outputs.summary }}

jobs:
  validate-user-core:
    name: "Validate User Service Core Logic"
    runs-on: ubuntu-latest
    outputs:
      summary: ${{ steps.test-summary.outputs.summary }}

    steps:
      - name: Checkout User Service code
        uses: actions/checkout@v4
        with:
          repository: "my-org/user-service"
          ref: "main"

      # ... (Setup Node.js, install dependencies) ...

      - name: Run user service core tests
        id: core-tests
        run: |
          # コア機能の統合テストを実行
          pnpm run test:core-integration
          echo "summary=User core logic validated successfully" >> $GITHUB_OUTPUT

      - name: Create test summary
        id: test-summary
        run: echo "summary=✅ User Service core validation passed" >> $GITHUB_OUTPUT
```

次に、`order-service`のメインCI/CDワークフローで、上記のjobをクロスドメイン統合検証のために参照できます。

```yaml
# .github/workflows/order-service-ci.yml
name: "Order Service CI/CD"

on:
  push:
    branches: [main]
  pull_request:

jobs:
  # order service自身のテストを実行
  test-order-service:
    name: "Test Order Service"
    runs-on: ubuntu-latest
    steps:
      - name: Checkout Order Service code
        uses: actions/checkout@v4
      # ... (order serviceの独自テストを実行) ...
      - name: Run order tests
        run: pnpm run test

  # User ServiceのCIジョブを参照して統合検証
  cross-validate-with-user-service:
    name: "Cross-Domain Validation (User Service)"
    needs: test-order-service
    uses: ./.github/workflows/reusable-user-service-ci.yml
    with:
      node-version: "18"

  # 自身のテストとクロスドメイン検証の両方が合格した後にのみビルド
  build:
    name: "Build Order Service"
    runs-on: ubuntu-latest
    needs: [test-order-service, cross-validate-with-user-service]
    steps:
      - name: Display validation results
        run: |
          echo "Order Service tests passed."
          echo "User Service validation summary: ${{ needs.cross-validate-with-user-service.outputs.test-results-summary }}"
      - name: Checkout code
        uses: actions/checkout@v4
      # ... (ビルドステップ) ...
      - name: Build application
        run: pnpm run build
```

もちろん、粒度を`Step`レベルに縮小したい場合も可能です。

```yaml
steps:
  - task: SonarCloudAnalyze@3
    inputs:
      jdkversion: "JAVA_HOME_21_X64"
    condition: succeededOrFailed()
  - task: SonarCloudPublish@3
    inputs:
      pollingTimeoutSec: "300"
    condition: succeededOrFailed()
```

このアプローチにより、CI/CDのタスクを「単一プロジェクトの品質保証」から「プロジェクトとドメインを横断したシステム安定性保護」に昇華させることができます。これはマイクロサービスや複雑なシステムアーキテクチャにおいて非常に重要です。

#### 3. 再利用可能なActionsの作成

コードの再利用性を向上させるために、カスタムアクションを作成します：

```yaml
# .github/actions/setup-node/action.yml
name: "Setup Node.js Environment"
description: "Setup Node.js with pnpm and dependencies caching"

inputs:
  node-version:
    description: "Node.js version"
    required: true
    default: "18"
  pnpm-version:
    description: "pnpm version"
    required: true
    default: "8.15.0"
  cache-key:
    description: "Cache key for dependencies"
    required: true

runs:
  using: "composite"
  steps:
    - name: Setup Node.js
      uses: actions/setup-node@v4
      with:
        node-version: ${{ inputs.node-version }}

    - name: Setup pnpm
      uses: pnpm/action-setup@v3
      with:
        version: ${{ inputs.pnpm-version }}

    - name: Get pnpm store directory
      id: pnpm-cache
      shell: bash
      run: echo "STORE_PATH=$(pnpm store path)" >> $GITHUB_OUTPUT

    - name: Setup pnpm cache
      uses: actions/cache@v4
      with:
        path: ${{ steps.pnpm-cache.outputs.STORE_PATH }}
        key: ${{ inputs.cache-key }}
        restore-keys: |
          ${{ runner.os }}-pnpm-store-

    - name: Install dependencies
      shell: bash
      run: pnpm install --frozen-lockfile
```

### タスクバージョン管理戦略

#### 1. ワークフローのバージョン管理設定

```yaml
# .github/workflows/ci-cd-v2.yml
name: "CI/CD Pipeline v2.0"

on:
  workflow_call:
    inputs:
      environment:
        required: true
        type: string
      workflow-version:
        required: false
        type: string
        default: "v2.0"

env:
  WORKFLOW_VERSION: ${{ inputs.workflow-version }}
  TARGET_ENVIRONMENT: ${{ inputs.environment }}

jobs:
  version-check:
    name: "Workflow Version Check"
    runs-on: ubuntu-latest

    steps:
      - name: Validate workflow version
        run: |
          echo "🔄 Running CI/CD Pipeline ${{ env.WORKFLOW_VERSION }}"
          echo "🎯 Target Environment: ${{ env.TARGET_ENVIRONMENT }}"

          # バージョン互換性チェック
          case "${{ env.WORKFLOW_VERSION }}" in
            v1.*)
              echo "⚠️ Using legacy workflow version"
              ;;;
            v2.*)
              echo "✅ Using current workflow version"
              ;;;
            *)
              echo "❌ Unknown workflow version"
              exit 1
              ;;;
          esac
```

#### 2. パイプライン設定ファイル管理

```yaml
# .github/pipeline-configs/production.yml
version: "2.0"
environment: "production"

stages:
  code-quality:
    enabled: true
    timeout: 10
    node-versions: [18, 20]

  testing:
    unit-tests:
      enabled: true
      timeout: 15
      parallel-matrix: true
    integration-tests:
      enabled: true
      timeout: 20
      services: [postgres, redis]
    e2e-tests:
      enabled: true
      timeout: 30
      browsers: [chromium, firefox]
      sharding: 4

  security:
    audit: true
    sast: true
    vulnerability-scan: true

  deployment:
    strategy: "blue-green"
    health-check:
      enabled: true
      retries: 5
      interval: 30
    rollback:
      auto: true
      timeout: 300

  notifications:
    slack:
      success: "#deployment"
      failure: "#alerts"
    email:
      - devops@company.com
```

#### 3. 動的パイプラインロード

```yaml
# .github/workflows/dynamic-pipeline.yml
name: "Dynamic Pipeline Loader"

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  load-config:
    name: "Load Pipeline Configuration"
    runs-on: ubuntu-latest

    outputs:
      config: ${{ steps.config.outputs.config }}
      version: ${{ steps.config.outputs.version }}

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Load pipeline configuration
        id: config
        run: |
          # ブランチに基づいて設定ファイルを選択
          if [[ "${{ github.ref }}" == "refs/heads/main" ]]; then
            CONFIG_FILE=".github/pipeline-configs/production.yml"
          elif [[ "${{ github.ref }}" == "refs/heads/develop" ]]; then
            CONFIG_FILE=".github/pipeline-configs/staging.yml"
          else
            CONFIG_FILE=".github/pipeline-configs/development.yml"
          fi

          # 設定をロードして出力
          CONFIG=$(cat $CONFIG_FILE | yq -o=json)
          VERSION=$(echo $CONFIG | jq -r '.version')

          echo "config=$CONFIG" >> $GITHUB_OUTPUT
          echo "version=$VERSION" >> $GITHUB_OUTPUT
          echo "📄 Loaded configuration from: $CONFIG_FILE"
          echo "🏷️ Pipeline version: $VERSION"

  execute-pipeline:
    name: "Execute Pipeline"
    needs: load-config
    uses: ./.github/workflows/ci-cd-v2.yml
    with:
      environment: ${{ fromJson(needs.load-config.outputs.config).environment }}
      workflow-version: ${{ needs.load-config.outputs.version }}
    secrets: inherit
```

#### 4. クロスドメイン再利用可能ワークフローのバージョン管理

jobsを再利用可能なワークフロー（`reusable-user-service-ci.yml`など）にモジュール化する場合、これらの共有ワークフローをバージョン管理することが重要になります。これにより、コアサービス（`user-service`など）のCIプロセスが変更されたときに、それに依存する他のサービス（`order-service`など）のビルドプロセスを誤って破壊しないことが保証されます。

**シナリオ: `user-service`のCIプロセスがv1からv2にアップグレードされたが、`order-service`は一時的にv1を使い続ける必要がある。**

再利用可能なワークフローをバージョン管理するためにGitタグを使用できます。

まず、`user-service`自体の`reusable-user-service-ci.yml`ファイルには特別なバージョンタグは不要です。そのバージョンは、それが存在するGitリポジトリのタグによって管理されます。`reusable-user-service-ci.yml`が安定した状態に達したと判断したら、そのコミットにバージョンタグを作成します。たとえば、`v1.0`や`v2.0`です。

次に、`order-service`のCIワークフローで、使用する`user-service`のCIのバージョンを明示的に指定できます。

```yaml
# .github/workflows/order-service-ci.yml
name: "Order Service CI/CD"

on:
  push:
    branches: [main]
  pull_request:

jobs:
  test-order-service:
    # ... (order service自身のテストを実行) ...
    name: "Test Order Service"
    runs-on: ubuntu-latest
    steps:
      - name: Checkout Order Service code
        uses: actions/checkout@v4
      - name: Run order tests
        run: pnpm run test

  # User Service CI v1.0を検証のために参照
  cross-validate-with-user-service-v1:
    name: "Cross-Domain Validation (User Service @v1.0)"
    needs: test-order-service
    # @<tag>構文を使用して再利用可能ワークフローのバージョンをロック
    uses: my-org/user-service/.github/workflows/reusable-user-service-ci.yml@v1.0
    with:
      node-version: "18"

  # チームの準備が整ったら、新しいPRを作成してv2.0にアップグレード可能
  # cross-validate-with-user-service-v2:
  #   name: "Cross-Domain Validation (User Service @v2.0)"
  #   needs: test-order-service
  #   uses: my-org/user-service/.github/workflows/reusable-user-service-ci.yml@v2.0
  #   with:
  #     node-version: '20'

  build:
    name: "Build Order Service"
    runs-on: ubuntu-latest
    needs: [test-order-service, cross-validate-with-user-service-v1]
    steps:
      - name: Display validation results
        run: |
          echo "Order Service tests passed."
          echo "User Service validation summary: ${{ needs.cross-validate-with-user-service-v1.outputs.test-results-summary }}"
      - name: Checkout code
        uses: actions/checkout@v4
      - name: Build application
        run: pnpm run build
```

この戦略により、「タスクバージョン管理」の概念を単一プロジェクト内から、プロジェクトとサービスを横断するエンタープライズレベルの次元に拡張し、真のPipeline as Codeのバージョン管理を実現します。

### パラメータ抽象化管理戦略

#### 1. 環境変数の階層化管理

```yaml
# .github/environments/production.yml
variables:
  # 公開変数
  NODE_ENV: "production"
  API_VERSION: "v2"
  BUILD_TARGET: "production"
  CACHE_TTL: "3600"

  # CDN設定
  CDN_DOMAIN: "cdn.myapp.com"
  ASSET_PREFIX: "/static"

  # 機能フラグ
  FEATURE_NEW_UI: "true"
  FEATURE_ANALYTICS: "true"
  FEATURE_AB_TESTING: "true"

secrets:
  # 機密情報(GitHub Secretsに設定する必要があります)
  - AWS_ACCESS_KEY_ID
  - AWS_SECRET_ACCESS_KEY
  - SENTRY_DSN
  - ANALYTICS_API_KEY
  - DATABASE_CONNECTION_STRING
```

#### 2. 動的環境変数インジェクション

```yaml
# .github/workflows/env-management.yml
name: "Environment Management"

jobs:
  setup-environment:
    name: "Setup Environment Variables"
    runs-on: ubuntu-latest

    outputs:
      env-config: ${{ steps.env-setup.outputs.config }}

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup environment configuration
        id: env-setup
        run: |
          # ターゲット環境に基づいて設定をロード
          case "${{ github.ref }}" in
            refs/heads/main)
              ENV_FILE=".github/environments/production.yml"
              ;;;
            refs/heads/develop)
              ENV_FILE=".github/environments/staging.yml"
              ;;;
            *)
              ENV_FILE=".github/environments/development.yml"
              ;;;
          esac

          # 環境変数をロード
          ENV_VARS=$(yq '.variables' $ENV_FILE)

          # JSON形式で出力
          echo "config=$ENV_VARS" >> $GITHUB_OUTPUT

          # 現在のjobに環境変数を設定
          yq '.variables | to_entries | .[] | "\(.key)=\(.value)"' $ENV_FILE >> $GITHUB_ENV

  build-with-env:
    name: "Build with Environment"
    needs: setup-environment
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Apply environment configuration
        run: |
          # 前のjobから環境設定を取得
          ENV_CONFIG='${{ needs.setup-environment.outputs.env-config }}'
          echo "$ENV_CONFIG" | jq -r 'to_entries | .[] | "\(.key)=\(.value)"' >> $GITHUB_ENV

      - name: Build application
        run: |
          echo "🏗️ Building with environment: $NODE_ENV"
          echo "🌐 API Version: $API_VERSION"
          echo "🎯 Build Target: $BUILD_TARGET"

          # ビルド情報ファイルを作成
          cat > build-info.json << EOF
          {
            "environment": "$NODE_ENV",
            "apiVersion": "$API_VERSION",
            "buildTarget": "$BUILD_TARGET",
            "buildTime": "$(date -u +%Y-%m-%dT%H:%M:%SZ)",
            "gitCommit": "$GITHUB_SHA",
            "gitBranch": "$GITHUB_REF_NAME"
          }
          EOF

          # ビルドを実行
          npm run build
```

#### 3. シークレットパラメータの安全な管理

```yaml
# .github/workflows/secrets-management.yml
name: "Secrets Management"

jobs:
  validate-secrets:
    name: "Validate Required Secrets"
    runs-on: ubuntu-latest

    steps:
      - name: Check required secrets
        env:
          # 必要なsecretsを参照
          AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          SENTRY_DSN: ${{ secrets.SENTRY_DSN }}
          SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK_URL }}

        run: |
          # 必要なsecretsが存在することを検証
          MISSING_SECRETS=()

          [ -z "$AWS_ACCESS_KEY_ID" ] && MISSING_SECRETS+=("AWS_ACCESS_KEY_ID")
          [ -z "$AWS_SECRET_ACCESS_KEY" ] && MISSING_SECRETS+=("AWS_SECRET_ACCESS_KEY")
          [ -z "$SENTRY_DSN" ] && MISSING_SECRETS+=("SENTRY_DSN")
          [ -z "$SLACK_WEBHOOK_URL" ] && MISSING_SECRETS+=("SLACK_WEBHOOK_URL")

          if [ ${#MISSING_SECRETS[@]} -gt 0 ]; then
            echo "❌ Missing required secrets:"
            printf "  - %s\n" "${MISSING_SECRETS[@]}"
            exit 1
          fi

          echo "✅ All required secrets are available"

      - name: Test AWS credentials
        run: |
          # AWS認証情報の有効性をテスト
          aws sts get-caller-identity || {
            echo "❌ AWS credentials are invalid"
            exit 1
          }
          echo "✅ AWS credentials are valid"
        env:
          AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          AWS_DEFAULT_REGION: ap-northeast-1
```

## AWS CodePipelineとCodeBuildとのエンタープライズレベル統合

大規模なエンタープライズ環境では、より強力なCI/CDインフラストラクチャが必要です。AWS CodePipelineは、より完全なエンタープライズレベルの機能を提供します。

### CodePipelineフルプロセス設計

```yaml
# infrastructure/codepipeline.tf
resource "aws_codepipeline" "main" {
  name     = "${var.project_name}-pipeline"
  role_arn = aws_iam_role.codepipeline_role.arn

  artifact_store {
    location = aws_s3_bucket.pipeline_artifacts.bucket
    type     = "S3"

    encryption_key {
      id   = aws_kms_key.pipeline_key.arn
      type = "KMS"
    }
  }

  # ステージ1: ソース
  stage {
    name = "Source"

    action {
      name             = "SourceAction"
      category         = "Source"
      owner            = "ThirdParty"
      provider         = "GitHub"
      version          = "1"
      output_artifacts = ["source_output"]

      configuration = {
        Owner  = var.github_owner
        Repo   = var.github_repo
        Branch = var.github_branch
        OAuthToken = var.github_token
        PollForSourceChanges = false
      }
    }
  }

  # ステージ2: コード品質
  stage {
    name = "CodeQuality"

    action {
      name             = "LintAndFormat"
      category         = "Test"
      owner            = "AWS"
      provider         = "CodeBuild"
      version          = "1"
      input_artifacts  = ["source_output"]
      output_artifacts = ["quality_output"]

      configuration = {
        ProjectName = aws_codebuild_project.code_quality.name
      }
    }

    action {
      name             = "SecurityScan"
      category         = "Test"
      owner            = "AWS"
      provider         = "CodeBuild"
      version          = "1"
      input_artifacts  = ["source_output"]
      output_artifacts = ["security_output"]

      configuration = {
        ProjectName = aws_codebuild_project.security_scan.name
      }

      run_order = 2
    }
  }

  # ステージ3: テスティング
  stage {
    name = "Testing"

    action {
      name             = "UnitTests"
      category         = "Test"
      owner            = "AWS"
      provider         = "CodeBuild"
      version          = "1"
      input_artifacts  = ["source_output"]
      output_artifacts = ["unit_test_output"]

      configuration = {
        ProjectName = aws_codebuild_project.unit_tests.name
      }
    }

    action {
      name             = "IntegrationTests"
      category         = "Test"
      owner            = "AWS"
      provider         = "CodeBuild"
      version          = "1"
      input_artifacts  = ["source_output"]
      output_artifacts = ["integration_test_output"]

      configuration = {
        ProjectName = aws_codebuild_project.integration_tests.name
      }

      run_order = 2
    }

    action {
      name             = "E2ETests"
      category         = "Test"
      owner            = "AWS"
      provider         = "CodeBuild"
      version          = "1"
      input_artifacts  = ["source_output"]
      output_artifacts = ["e2e_test_output"]

      configuration = {
        ProjectName = aws_codebuild_project.e2e_tests.name
      }

      run_order = 3
    }
  }

  # ステージ4: ビルド
  stage {
    name = "Build"

    action {
      name             = "BuildApplication"
      category         = "Build"
      owner            = "AWS"
      provider         = "CodeBuild"
      version          = "1"
      input_artifacts  = ["source_output"]
      output_artifacts = ["build_output"]

      configuration = {
        ProjectName = aws_codebuild_project.build.name
        EnvironmentVariables = jsonencode([
          {
            name  = "ENVIRONMENT"
            value = "staging"
          },
          {
            name  = "BUILD_NUMBER"
            value = "#{codepipeline.PipelineExecutionId}"
          },
          {
            name  = "COMMIT_SHA"
            value = "#{SourceVariables.CommitId}"
          }
        ])
      }
    }
  }

  # ステージ5: ステージングへのデプロイ
  stage {
    name = "DeployStaging"

    action {
      name            = "DeployToS3"
      category        = "Deploy"
      owner           = "AWS"
      provider        = "S3"
      version         = "1"
      input_artifacts = ["build_output"]

      configuration = {
        BucketName = aws_s3_bucket.staging.bucket
        Extract    = "true"
      }
    }

    action {
      name             = "InvalidateCloudFront"
      category         = "Invoke"
      owner            = "AWS"
      provider         = "Lambda"
      version          = "1"

      configuration = {
        FunctionName = aws_lambda_function.invalidate_cache.function_name
        UserParameters = jsonencode({
          distribution_id = aws_cloudfront_distribution.staging.id
          paths = ["/*"]
        })
      }

      run_order = 2
    }

    action {
      name             = "StagingTests"
      category         = "Test"
      owner            = "AWS"
      provider         = "CodeBuild"
      version          = "1"
      input_artifacts  = ["source_output"]

      configuration = {
        ProjectName = aws_codebuild_project.staging_tests.name
        EnvironmentVariables = jsonencode([
          {
            name  = "TEST_URL"
            value = "https://${aws_cloudfront_distribution.staging.domain_name}"
          }
        ])
      }

      run_order = 3
    }
  }

  # ステージ6: ビジネス承認
  stage {
    name = "ApprovalGate"

    action {
      name     = "ProductionApproval"
      category = "Approval"
      owner    = "AWS"
      provider = "Manual"
      version  = "1"

      configuration = {
        NotificationArn    = aws_sns_topic.deployment_approval.arn
        CustomData         = "Please review staging deployment and approve for production. Staging URL: https://${aws_cloudfront_distribution.staging.domain_name}"
        ExternalEntityLink = "https://${aws_cloudfront_distribution.staging.domain_name}"
      }
    }
  }

  # ステージ7: 本番環境へのデプロイ
  stage {
    name = "DeployProduction"

    action {
      name             = "BackupProduction"
      category         = "Invoke"
      owner            = "AWS"
      provider         = "Lambda"
      version          = "1"

      configuration = {
        FunctionName = aws_lambda_function.backup_production.function_name
        UserParameters = jsonencode({
          source_bucket = aws_s3_bucket.production.bucket
          backup_bucket = aws_s3_bucket.backup.bucket
          timestamp = "#{codepipeline.PipelineExecutionId}"
        })
      }
    }

    action {
      name             = "DeployToProduction"
      category         = "Deploy"
      owner            = "AWS"
      provider         = "S3"
      version          = "1"
      input_artifacts = ["build_output"]

      configuration = {
        BucketName = aws_s3_bucket.production.bucket
        Extract    = "true"
      }

      run_order = 2
    }

    action {
      name             = "InvalidateProductionCache"
      category         = "Invoke"
      owner            = "AWS"
      provider         = "Lambda"
      version          = "1"

      configuration = {
        FunctionName = aws_lambda_function.invalidate_cache.function_name
        UserParameters = jsonencode({
          distribution_id = aws_cloudfront_distribution.production.id
          paths = ["/*"]
        })
      }

      run_order = 3
    }

    action {
      name             = "ProductionHealthCheck"
      category         = "Invoke"
      owner            = "AWS"
      provider         = "Lambda"
      version          = "1"

      configuration = {
        FunctionName = aws_lambda_function.health_check.function_name
        UserParameters = jsonencode({
          url = "https://${aws_cloudfront_distribution.production.domain_name}"
          retries = 5
          interval = 30
        })
      }

      run_order = 4
    }
  }

  # ステージ8: デプロイ後検証
  stage {
    name = "PostDeployment"

    action {
      name             = "ProductionTests"
      category         = "Test"
      owner            = "AWS"
      provider         = "CodeBuild"
      version          = "1"
      input_artifacts  = ["source_output"]

      configuration = {
        ProjectName = aws_codebuild_project.production_tests.name
        EnvironmentVariables = jsonencode([
          {
            name  = "TEST_URL"
            value = "https://${aws_cloudfront_distribution.production.domain_name}"
          }
        ])
      }
    }

    action {
      name             = "NotifySuccess"
      category         = "Invoke"
      owner            = "AWS"
      provider         = "Lambda"
      version          = "1"

      configuration = {
        FunctionName = aws_lambda_function.notify_deployment.function_name
        UserParameters = jsonencode({
          status = "success"
          pipeline_execution_id = "#{codepipeline.PipelineExecutionId}"
          commit_id = "#{SourceVariables.CommitId}"
        })
      }

      run_order = 2
    }
  }

  tags = var.tags
}

# パイプライン失敗処理
resource "aws_cloudwatch_event_rule" "pipeline_failure" {
  name = "${var.project_name}-pipeline-failure"

  event_pattern = jsonencode({
    source      = ["aws.codepipeline"]
    detail-type = ["CodePipeline Pipeline Execution State Change"]
    detail = {
      state = ["FAILED"]
      pipeline = [aws_codepipeline.main.name]
    }
  })
}

resource "aws_cloudwatch_event_target" "pipeline_failure_lambda" {
  rule      = aws_cloudwatch_event_rule.pipeline_failure.name
  target_id = "PipelineFailureLambdaTarget"
  arn       = aws_lambda_function.handle_pipeline_failure.arn
}
```

### 高度なCodeBuildプロジェクト設定

#### 1. マルチステージビルドプロジェクト

```yaml
# buildspec/multi-stage-build.yml
version: 0.2

phases:
  install:
    runtime-versions:
      nodejs: 18
      python: 3.9
      docker: 20
    commands:
      - echo "🚀 Starting multi-stage build process"
      - echo "📦 Installing build dependencies..."

      # Node.js関連ツールのインストール
      - npm install -g pnpm@8.15.0
      - npm install -g npm-check-updates

      # Python関連ツールのインストール
      - pip install --upgrade pip
      - pip install boto3 requests

      # その他のツールをインストール
      - curl -sSfL https://raw.githubusercontent.com/anchore/syft/main/install.sh | sh -s -- -b /usr/local/bin

  pre_build:
    commands:
      - echo "🔍 Pre-build validation started"

      # 環境変数の検証
      - |
        REQUIRED_VARS=("ENVIRONMENT" "BUILD_NUMBER" "COMMIT_SHA")
        for var in "${REQUIRED_VARS[@]}"; do
          if [ -z "${!var}" ]; then
            echo "❌ Required environment variable $var is not set"
            exit 1
          fi
        done
        echo "✅ All required environment variables are set"

      # 依存関係のインストールとキャッシング
      - echo "📦 Installing dependencies..."
      - pnpm install --frozen-lockfile

      # 依存関係のセキュリティチェック
      - echo "🔒 Running dependency security audit..."
      - pnpm audit --audit-level=moderate

      # ビルド情報の生成
      - |
        BUILD_INFO=$(cat << EOF
        {
          "buildNumber": "$BUILD_NUMBER",
          "environment": "$ENVIRONMENT",
          "commitSha": "$COMMIT_SHA",
          "buildTime": "$(date -u +%Y-%m-%dT%H:%M:%SZ)",
          "nodeVersion": "$(node --version)",
          "pnpmVersion": "$(pnpm --version)",
          "platform": "$(uname -a)"
        }
        EOF
        )
        echo "$BUILD_INFO" > build-info.json
        echo "📄 Build info generated"

  build:
    commands:
      - echo "🏗️ Build phase started"

      # コード品質チェック
      - echo "🔍 Running code quality checks..."
      - pnpm run lint:check
      - pnpm run format:check
      - pnpm run type-check

      # ユニットテスト
      - echo "🧪 Running unit tests..."
      - pnpm run test:unit --coverage --reporter=junit --outputFile=test-results.xml

      # アプリケーションのビルド
      - echo "📦 Building application..."
      - pnpm run build

      # ビルド後の検証
      - echo "✅ Post-build validation..."
      - |
        # ビルドファイルをチェック
        if [ ! -f "dist/index.html" ]; then
          echo "❌ Main entry file not found"
          exit 1
        fi

        # ビルドサイズをチェック
        BUNDLE_SIZE=$(du -sk dist | cut -f1)
        echo "📊 Bundle size: ${BUNDLE_SIZE}KB"

        if [ $BUNDLE_SIZE -gt 51200 ]; then  # 50MB
          echo "⚠️ Bundle size is large: ${BUNDLE_SIZE}KB"
        fi

        # 主要リソースをチェック
        ASSET_COUNT=$(find dist -name "*.js" -o -name "*.css" | wc -l)
        echo "📄 Assets generated: $ASSET_COUNT files"

      # SBOM(Software Bill of Materials)の生成
      - echo "📋 Generating SBOM..."
      - syft packages dir:. -o spdx-json > sbom.spdx.json

      # Dockerイメージのビルド(必要な場合)
      - |
        if [ -f "Dockerfile" ]; then
          echo "🐳 Building Docker image..."
          docker build -t ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${IMAGE_REPO_NAME}:${BUILD_NUMBER} .
          docker build -t ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${IMAGE_REPO_NAME}:latest .
        fi

  post_build:
    commands:
      - echo "🎯 Post-build phase started"

      # テストカバレッジチェック
      - |
        if [ -f "coverage/lcov.info" ]; then
          COVERAGE=$(lcov --summary coverage/lcov.info 2>&1 | grep "lines" | cut -d' ' -f4 | cut -d'%' -f1)
          echo "📊 Test coverage: ${COVERAGE}%"
          
          if (( $(echo "$COVERAGE < 80" | bc -l) )); then
            echo "⚠️ Test coverage below threshold: ${COVERAGE}% < 80%"
          fi
        fi

      # Dockerイメージのプッシュ(必要な場合)
      - |
        if [ -f "Dockerfile" ]; then
          echo "🚢 Pushing Docker image..."
          aws ecr get-login-password --region $AWS_DEFAULT_REGION | docker login --username AWS --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com
          docker push ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${IMAGE_REPO_NAME}:${BUILD_NUMBER}
          docker push ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${IMAGE_REPO_NAME}:latest
        fi

      # デプロイパッケージの準備
      - echo "🎁 Preparing deployment package..."
      - cp build-info.json dist/
      - tar -czf deployment-package.tar.gz -C dist .

      # ビルドレポート
      - |
        echo "📊 Build Report" > build-report.txt
        echo "===============" >> build-report.txt
        echo "Build Number: $BUILD_NUMBER" >> build-report.txt
        echo "Environment: $ENVIRONMENT" >> build-report.txt
        echo "Commit: $COMMIT_SHA" >> build-report.txt
        echo "Bundle Size: ${BUNDLE_SIZE}KB" >> build-report.txt
        echo "Assets Count: $ASSET_COUNT" >> build-report.txt
        if [ -n "$COVERAGE" ]; then
          echo "Test Coverage: ${COVERAGE}%" >> build-report.txt
        fi
        echo "Build Time: $(date)" >> build-report.txt

      - |
        if [ $CODEBUILD_BUILD_SUCCEEDING -eq 1 ]; then
          echo "✅ Build completed successfully"
        else
          echo "❌ Build failed"
          exit 1
        fi

artifacts:
  files: 
    - "**/*"
  base-directory: dist
  name: build-$BUILD_NUMBER

secondary-artifacts:
  BuildReport:
    files:
      - build-report.txt
      - build-info.json
      - test-results.xml
      - sbom.spdx.json
    name: build-artifacts-$BUILD_NUMBER

  Coverage:
    files:
      - "coverage/**/*"
    name: coverage-$BUILD_NUMBER

reports:
  unit-tests:
    files:
      - "test-results.xml"
    file-format: "JUNITXML"

coverage-reports:
  coverage-reports:
    files:
      - "coverage/lcov.info"
    file-format: "CLOVERXML"

cache:
  paths:
    - "node_modules/**/*"
    - ".pnpm-store/**/*"
    - "/root/.cache/pip/**/*"
```

#### 2. 環境別ビルド設定

```terraform
# CodeBuildプロジェクト - 環境別設定
resource "aws_codebuild_project" "build" {
  for_each = var.environments

  name          = "${var.project_name}-build-${each.key}"
  description   = "Build project for ${each.key} environment"
  service_role  = aws_iam_role.codebuild_role.arn

  artifacts {
    type = "CODEPIPELINE"
  }

  environment {
    compute_type                = each.value.compute_type
    image                      = "aws/codebuild/standard:7.0"
    type                       = "LINUX_CONTAINER"
    image_pull_credentials_type = "CODEBUILD"
    privileged_mode            = true

    dynamic "environment_variable" {
      for_each = merge(
        var.common_env_vars,
        each.value.env_vars,
        {
          ENVIRONMENT = each.key
          BUILD_TARGET = each.value.build_target
          API_ENDPOINT = each.value.api_endpoint
        }
      )
      content {
        name  = environment_variable.key
        value = environment_variable.value
      }
    }

    # 機密性の高い環境変数
    dynamic "environment_variable" {
      for_each = each.value.secret_env_vars
      content {
        name  = environment_variable.key
        value = environment_variable.value
        type  = "PARAMETER_STORE"
      }
    }
  }

  source {
    type = "CODEPIPELINE"
    buildspec = templatefile("${path.module}/buildspec/${each.value.buildspec_file}", {
      environment = each.key
      node_version = each.value.node_version
      build_commands = each.value.build_commands
    })
  }

  vpc_config {
    vpc_id = each.value.vpc_id
    subnets = each.value.subnet_ids
    security_group_ids = [aws_security_group.codebuild[each.key].id]
  }

  logs_config {
    cloudwatch_logs {
      group_name  = aws_cloudwatch_log_group.codebuild[each.key].name
      stream_name = "build-log"
    }

    s3_logs {
      status   = "ENABLED"
      location = "${aws_s3_bucket.build_logs.bucket}/build-logs/${each.key}"
    }
  }

  tags = merge(var.tags, {
    Environment = each.key
    Purpose     = "Build"
  })
}

# 環境設定変数
variable "environments" {
  description = "Environment-specific configurations"
  type = map(object({
    compute_type     = string
    build_target     = string
    api_endpoint     = string
    node_version     = string
    buildspec_file   = string
    vpc_id          = string
    subnet_ids      = list(string)
    env_vars        = map(string)
    secret_env_vars = map(string)
    build_commands  = list(string)
  }))

  default = {
    development = {
      compute_type     = "BUILD_GENERAL1_SMALL"
      build_target     = "development"
      api_endpoint     = "https://api-dev.myapp.com"
      node_version     = "18"
      buildspec_file   = "development.yml"
      vpc_id          = "vpc-dev123"
      subnet_ids      = ["subnet-dev1", "subnet-dev2"]
      env_vars = {
        DEBUG_MODE = "true"
        LOG_LEVEL  = "debug"
      }
      secret_env_vars = {
        DEV_API_KEY = "/myapp/dev/api-key"
      }
      build_commands = [
        "npm run build:dev",
        "npm run test:unit"
      ]
    }

    staging = {
      compute_type     = "BUILD_GENERAL1_MEDIUM"
      build_target     = "staging"
      api_endpoint     = "https://api-staging.myapp.com"
      node_version     = "18"
      buildspec_file   = "staging.yml"
      vpc_id          = "vpc-staging123"
      subnet_ids      = ["subnet-staging1", "subnet-staging2"]
      env_vars = {
        DEBUG_MODE = "false"
        LOG_LEVEL  = "info"
      }
      secret_env_vars = {
        STAGING_API_KEY = "/myapp/staging/api-key"
      }
      build_commands = [
        "npm run build:staging",
        "npm run test:unit",
        "npm run test:integration"
      ]
    }

    production = {
      compute_type     = "BUILD_GENERAL1_LARGE"
      build_target     = "production"
      api_endpoint     = "https://api.myapp.com"
      node_version     = "18"
      buildspec_file   = "production.yml"
      vpc_id          = "vpc-prod123"
      subnet_ids      = ["subnet-prod1", "subnet-prod2"]
      env_vars = {
        DEBUG_MODE = "false"
        LOG_LEVEL  = "warn"
        OPTIMIZE   = "true"
      }
      secret_env_vars = {
        PROD_API_KEY    = "/myapp/production/api-key"
        SENTRY_DSN      = "/myapp/production/sentry-dsn"
        ANALYTICS_KEY   = "/myapp/production/analytics-key"
      }
      build_commands = [
        "npm run build:production",
        "npm run test:unit",
        "npm run test:integration",
        "npm run test:e2e"
      ]
    }
  }
}
```

## ビジネスレビューゲートの実装

最後に、エンタープライズ環境では、本番デプロイメントは会社の公式製品リリースに組み込まれる前に、厳格な**ビジネスロジック確認レビュー**を経る必要があります。ここでは、**Peer Review Gate**、**Business Review Gate**、**Quality Review Gate**を組み合わせたレビューメカニズムを実装します。

```mermaid
graph TD
    subgraph "スマートレビューと通知"
        A[デプロイリクエストがトリガー] --> B{変更リスクとタイプを分析}
        B --> C{承認パスを決定}
        C --> D[DynamoDBを使用して<br/>承認記録を作成]
        D --> E[マルチチャネル通知を送信<br/>SlackとEmail]
    end

    subgraph "多層承認ゲート"
        E --> F{技術レビューが必要?}
        F -->|はい| G[テクニカルリードレビュー<br/>Slack/UI経由]
        F -->|いいえ| J{ビジネスレビューが必要?}
        G --> H{承認された?}
        H -->|いいえ| Z[承認失敗]
        H -->|はい| J
        J -->|はい| K[プロダクトマネージャーレビュー<br/>Slack/UI経由]
        J -->|いいえ| P{セキュリティレビューが必要?}
        K --> L{承認された?}
        L -->|いいえ| Z
        L -->|はい| P
        P -->|はい| Q[セキュリティチームレビュー<br/>Slack/UI経由]
        P -->|いいえ| T[すべての承認完了]

        Q --> R{承認された?}
        R -->|いいえ| Z
        R -->|はい| T
    end

    subgraph "最終承認と実行"
        T --> U[最終デプロイメント承認]
        U --> V[✅ 本番デプロイメントをトリガー]
        Z --> W[❌ 失敗と理由を通知]
    end

    style A fill:#e3f2fd,stroke:#333,stroke-width:2px
    style V fill:#e8f5e9,stroke:#333,stroke-width:2px
    style W fill:#ffebee,stroke:#333,stroke-width:2px
    style Z fill:#ffebee,stroke:#333,stroke-width:2px
```

### 1. 多層承認プロセス

```yaml
# .github/workflows/approval-workflow.yml
name: "Production Deployment Approval"

on:
  workflow_call:
    inputs:
      deployment-version:
        required: true
        type: string
      staging-url:
        required: true
        type: string

jobs:
  # 第1層: 技術レビュー
  technical-review:
    name: "Technical Review"
    runs-on: ubuntu-latest

    environment:
      name: technical-approval

    steps:
      - name: Technical checklist validation
        run: |
          echo "🔍 Technical Review Checklist"
          echo "=============================="
          echo "📦 Version: ${{ inputs.deployment-version }}"
          echo "🌐 Staging URL: ${{ inputs.staging-url }}"
          echo ""
          echo "Please verify:"
          echo "✓ All tests are passing"
          echo "✓ Code review completed"
          echo "✓ Security scan passed"
          echo "✓ Performance benchmarks met"
          echo "✓ Documentation updated"
          echo ""
          echo "Approved by: Technical Lead"

  # 第2層: ビジネスレビュー
  business-review:
    name: "Business Review"
    runs-on: ubuntu-latest
    needs: technical-review

    environment:
      name: business-approval

    steps:
      - name: Business validation
        run: |
          echo "💼 Business Review Checklist"
          echo "==========================="
          echo "📦 Version: ${{ inputs.deployment-version }}"
          echo "🌐 Staging URL: ${{ inputs.staging-url }}"
          echo ""
          echo "Please verify:"
          echo "✓ Feature requirements met"
          echo "✓ User acceptance testing completed"
          echo "✓ Business logic validated"
          echo "✓ Compliance requirements satisfied"
          echo "✓ Customer impact assessed"
          echo ""
          echo "Approved by: Product Manager"

  # 第3層: セキュリティレビュー(高リスク変更の場合)
  security-review:
    name: "Security Review"
    runs-on: ubuntu-latest
    needs: business-review
    if: contains(github.event.head_commit.message, '[security-review]')

    environment:
      name: security-approval

    steps:
      - name: Security validation
        run: |
          echo "🛡️ Security Review Checklist"
          echo "==========================="
          echo "📦 Version: ${{ inputs.deployment-version }}"
          echo "🌐 Staging URL: ${{ inputs.staging-url }}"
          echo ""
          echo "Please verify:"
          echo "✓ Security vulnerability scan passed"
          echo "✓ Authentication/Authorization tested"
          echo "✓ Data privacy compliance verified"
          echo "✓ Infrastructure security reviewed"
          echo "✓ Third-party dependencies audited"
          echo ""
          echo "Approved by: Security Team"

  # 最終承認: デプロイメント承認
  deployment-authorization:
    name: "Deployment Authorization"
    runs-on: ubuntu-latest
    needs: [technical-review, business-review]
    if: always() && (needs.technical-review.result == 'success' && needs.business-review.result == 'success')

    environment:
      name: deployment-authorization

    outputs:
      approved: ${{ steps.authorization.outputs.approved }}
      approver: ${{ steps.authorization.outputs.approver }}

    steps:
      - name: Final deployment authorization
        id: authorization
        run: |
          echo "🚀 Final Deployment Authorization"
          echo "================================="
          echo "📦 Version: ${{ inputs.deployment-version }}"
          echo "🌐 Staging URL: ${{ inputs.staging-url }}"
          echo "✅ Technical Review: Approved"
          echo "✅ Business Review: Approved"

          if [ "${{ needs.security-review.result }}" == "success" ]; then
            echo "✅ Security Review: Approved"
          fi

          echo ""
          echo "🎯 Ready for production deployment"
          echo "approved=true" >> $GITHUB_OUTPUT
          echo "approver=${{ github.actor }}" >> $GITHUB_OUTPUT
```

### 2. スマート承認ルール

```yaml
# .github/workflows/smart-approval.yml
name: "Smart Approval Logic"

on:
  workflow_call:
    inputs:
      change-type:
        required: true
        type: string
      risk-level:
        required: true
        type: string
      affected-components:
        required: true
        type: string

jobs:
  determine-approval-path:
    name: "Determine Approval Requirements"
    runs-on: ubuntu-latest

    outputs:
      requires-technical: ${{ steps.approval-logic.outputs.requires-technical }}
      requires-business: ${{ steps.approval-logic.outputs.requires-business }}
      requires-security: ${{ steps.approval-logic.outputs.requires-security }}
      requires-emergency: ${{ steps.approval-logic.outputs.requires-emergency }}

    steps:
      - name: Analyze change requirements
        id: approval-logic
        run: |
          CHANGE_TYPE="${{ inputs.change-type }}"
          RISK_LEVEL="${{ inputs.risk-level }}"
          AFFECTED_COMPONENTS="${{ inputs.affected-components }}"

          # デフォルトの承認要件
          REQUIRES_TECHNICAL="true"
          REQUIRES_BUSINESS="false"
          REQUIRES_SECURITY="false"
          REQUIRES_EMERGENCY="false"

          echo "📊 Analyzing change requirements..."
          echo "Change Type: $CHANGE_TYPE"
          echo "Risk Level: $RISK_LEVEL"
          echo "Affected Components: $AFFECTED_COMPONENTS"

          # リスクレベルの判定
          case "$RISK_LEVEL" in
            "low")
              REQUIRES_BUSINESS="false"
              REQUIRES_SECURITY="false"
              ;;;
            "medium")
              REQUIRES_BUSINESS="true"
              REQUIRES_SECURITY="false"
              ;;;
            "high"|"critical")
              REQUIRES_BUSINESS="true"
              REQUIRES_SECURITY="true"
              ;;;
          esac

          # 変更タイプの判定
          case "$CHANGE_TYPE" in
            "hotfix"|"emergency")
              REQUIRES_EMERGENCY="true"
              REQUIRES_BUSINESS="false"
              REQUIRES_SECURITY="false"
              ;;;
            "security-patch")
              REQUIRES_SECURITY="true"
              ;;;
            "feature")
              REQUIRES_BUSINESS="true"
              ;;;
          esac

          # 影響を受けるコンポーネントの判定
          if [[ "$AFFECTED_COMPONENTS" == *"authentication"* ]] || \
             [[ "$AFFECTED_COMPONENTS" == *"payment"* ]] || \
             [[ "$AFFECTED_COMPONENTS" == *"user-data"* ]]; then
            REQUIRES_SECURITY="true"
          fi

          if [[ "$AFFECTED_COMPONENTS" == *"database"* ]] || \
             [[ "$AFFECTED_COMPONENTS" == *"infrastructure"* ]]; then
            REQUIRES_SECURITY="true"
            REQUIRES_BUSINESS="true"
          fi

          # 結果を出力
          echo "requires-technical=$REQUIRES_TECHNICAL" >> $GITHUB_OUTPUT
          echo "requires-business=$REQUIRES_BUSINESS" >> $GITHUB_OUTPUT
          echo "requires-security=$REQUIRES_SECURITY" >> $GITHUB_OUTPUT
          echo "requires-emergency=$REQUIRES_EMERGENCY" >> $GITHUB_OUTPUT

          echo ""
          echo "📋 Approval Requirements:"
          echo "Technical Review: $REQUIRES_TECHNICAL"
          echo "Business Review: $REQUIRES_BUSINESS"
          echo "Security Review: $REQUIRES_SECURITY"
          echo "Emergency Process: $REQUIRES_EMERGENCY"

  conditional-approvals:
    name: "Execute Required Approvals"
    needs: determine-approval-path
    uses: ./.github/workflows/conditional-approval.yml
    with:
      requires-technical: ${{ needs.determine-approval-path.outputs.requires-technical }}
      requires-business: ${{ needs.determine-approval-path.outputs.requires-business }}
      requires-security: ${{ needs.determine-approval-path.outputs.requires-security }}
      requires-emergency: ${{ needs.determine-approval-path.outputs.requires-emergency }}
```

### 3. 承認通知と追跡

```python
# scripts/approval-notification.py
import json
import boto3
import requests
from datetime import datetime, timedelta

class ApprovalNotificationManager:
    def __init__(self):
        self.sns = boto3.client('sns')
        self.dynamodb = boto3.resource('dynamodb')
        self.approval_table = self.dynamodb.Table('approval-tracking')

    def create_approval_request(self, deployment_info):
        """承認リクエストを作成"""
        approval_id = f"approval-${deployment_info['build_number']}-${int(datetime.now().timestamp())}"

        approval_record = {
            'approval_id': approval_id,
            'deployment_version': deployment_info['version'],
            'staging_url': deployment_info['staging_url'],
            'requester': deployment_info['requester'],
            'created_at': datetime.now().isoformat(),
            'expires_at': (datetime.now() + timedelta(hours=24)).isoformat(),
            'status': 'pending',
            'required_approvals': deployment_info['required_approvals'],
            'completed_approvals': [],
            'change_summary': deployment_info['change_summary']
        }

        # DynamoDBに保存
        self.approval_table.put_item(Item=approval_record)

        # 通知を送信
        self.send_approval_notifications(approval_record)

        return approval_id

    def send_approval_notifications(self, approval_record):
        """承認通知を送信"""

        # Email通知
        self.send_email_notification(approval_record)

        # Slack通知
        self.send_slack_notification(approval_record)

        # Teams通知
        self.send_teams_notification(approval_record)

    def send_slack_notification(self, approval_record):
        """Slack通知を送信"""
        webhook_url = os.environ.get('SLACK_WEBHOOK_URL')

        required_approvals = approval_record['required_approvals']
        approval_buttons = []

        for approval_type in required_approvals:
            approval_buttons.append({
                "type": "button",
                "text": {
                    "type": "plain_text",
                    "text": f"Approve {approval_type.title()}"
                },
                "action_id": f"approve_{approval_type}",
                "value": approval_record['approval_id'],
                "style": "primary"
            })

        message = {
            "blocks": [
                {
                    "type": "header",
                    "text": {
                        "type": "plain_text",
                        "text": "🚀 Production Deployment Approval Required"
                    }
                },
                {
                    "type": "section",
                    "fields": [
                        {
                            "type": "mrkdwn",
                            "text": f"*Version:* {approval_record['deployment_version']}"
                        },
                        {
                            "type": "mrkdwn",
                            "text": f"*Requester:* {approval_record['requester']}"
                        },
                        {
                            "type": "mrkdwn",
                            "text": f"*Staging URL:* <{approval_record['staging_url']}|View Staging>"
                        },
                        {
                            "type": "mrkdwn",
                            "text": f"*Expires:* {approval_record['expires_at']}"
                        }
                    ]
                },
                {
                    "type": "section",
                    "text": {
                        "type": "mrkdwn",
                        "text": f"*Change Summary:*\n{approval_record['change_summary']}"
                    }
                },
                {
                    "type": "section",
                    "text": {
                        "type": "mrkdwn",
                        "text": f"*Required Approvals:* {', '.join(required_approvals)}"
                    }
                },
                {
                    "type": "actions",
                    "elements": approval_buttons + [
                        {
                            "type": "button",
                            "text": {
                                "type": "plain_text",
                                "text": "Reject"
                            },
                            "action_id": "reject_deployment",
                            "value": approval_record['approval_id'],
                            "style": "danger"
                        }
                    ]
                }
            ]
        }

        response = requests.post(webhook_url, json=message)
        return response.status_code == 200

    def process_approval_response(self, approval_id, approval_type, approver, decision):
        """承認レスポンスを処理"""

        # 承認記録を取得
        response = self.approval_table.get_item(Key={'approval_id': approval_id})
        approval_record = response.get('Item')

        if not approval_record:
            return {'error': 'Approval record not found'}

        if approval_record['status'] != 'pending':
            return {'error': 'Approval already completed or expired'}

        # 承認ステータスを更新
        if decision == 'approve':
            approval_record['completed_approvals'].append({
                'type': approval_type,
                'approver': approver,
                'timestamp': datetime.now().isoformat(),
                'decision': 'approved'
            })
        else:
            approval_record['status'] = 'rejected'
            approval_record['rejected_by'] = approver
            approval_record['rejected_at'] = datetime.now().isoformat()

        # すべての必要な承認が完了したかチェック
        required_approvals = set(approval_record['required_approvals'])
        completed_approvals = set([a['type'] for a in approval_record['completed_approvals']])

        if decision == 'approve' and required_approvals.issubset(completed_approvals):
            approval_record['status'] = 'approved'
            approval_record['approved_at'] = datetime.now().isoformat()

            # デプロイメントプロセスをトリガー
            self.trigger_deployment(approval_record)

        # データベース記録を更新
        self.approval_table.put_item(Item=approval_record)

        # ステータス更新通知を送信
        self.send_status_update(approval_record, approval_type, approver, decision)

        return {'status': approval_record['status']}

    def trigger_deployment(self, approval_record):
        """本番デプロイメントをトリガー"""

        # GitHub API経由でデプロイメントワークフローをトリガー
        github_token = os.environ.get('GITHUB_TOKEN')
        repo_owner = os.environ.get('GITHUB_REPO_OWNER')
        repo_name = os.environ.get('GITHUB_REPO_NAME')

        headers = {
            'Authorization': f'Bearer {github_token}',
            'Accept': 'application/vnd.github.v3+json'
        }

        workflow_data = {
            'ref': 'main',
            'inputs': {
                'deployment_version': approval_record['deployment_version'],
                'approval_id': approval_record['approval_id'],
                'approved_by': ', '.join([a['approver'] for a in approval_record['completed_approvals']])
            }
        }

        url = f"https://api.github.com/repos/{repo_owner}/{repo_name}/actions/workflows/production-deploy.yml/dispatches"
        response = requests.post(url, json=workflow_data, headers=headers)

        return response.status_code == 204

    def send_status_update(self, approval_record, approval_type, approver, decision):
        """ステータス更新通知を送信"""

        status_emoji = "✅" if decision == "approve" else "❌"
        status_text = "approved" if decision == "approve" else "rejected"

        if approval_record['status'] == 'approved':
            message = f"🚀 Deployment approved and triggered!\n"
            message += f"Version: {approval_record['deployment_version']}\n"
            message += f"All required approvals completed."
        elif approval_record['status'] == 'rejected':
            message = f"❌ Deployment rejected by {approver}\n"
            message += f"Version: {approval_record['deployment_version']}\n"
            message += f"Reason: {approval_type} approval denied."
        else:
            remaining_approvals = set(approval_record['required_approvals']) - set([a['type'] for a in approval_record['completed_approvals']])
            message = f"{status_emoji} {approval_type.title()} {status_text} by {approver}\n"
            message += f"Version: {approval_record['deployment_version']}\n"
            message += f"Remaining approvals needed: {', '.join(remaining_approvals)}"

        # Slack更新を送信
        webhook_url = os.environ.get('SLACK_WEBHOOK_URL')
        requests.post(webhook_url, json={"text": message})

# Lambda関数ハンドラ
def lambda_handler(event, context):
    manager = ApprovalNotificationManager()

    if event.get('action') == 'create_approval':
        approval_id = manager.create_approval_request(event['deployment_info'])
        return {'approval_id': approval_id}

    elif event.get('action') == 'process_response':
        result = manager.process_approval_response(
            event['approval_id'],
            event['approval_type'],
            event['approver'],
            event['decision']
        )
        return result

    else:
        return {'error': 'Unknown action'}
```

## 現代のソフトウェア配信パイプラインの構築

完全なCI/CD自動化は、現代のソフトウェア開発に根本的な変化をもたらします。私たちは従来の手動デプロイメントの痛点から始め、完全な自動化パイプラインの構築を深く掘り下げ、特にエンタープライズレベルの要件に焦点を当てました：

### コア価値の再確認

**技術的変革**:

- **Job断片化管理**: タスクを合理的に分割し、実行効率とメンテナンス性を向上させる。
- **タスクバージョン管理**: CI/CDプロセスをバージョン管理し、追跡可能性を確保する。
- **パラメータ抽象化管理**: 環境設定を分離し、マルチ環境デプロイメントをサポートする。
- **ビジネスレビューゲート**: 手動レビューを自動化に統合し、効率とリスクのバランスを取る。

**組織的影響**:

- **効率と品質のバランス**: 自動化により効率が向上し、レビューメカニズムにより品質が保証される。
- **リスク管理された俊敏性**: 迅速な配信とリスク管理のバランス。
- **チームコラボレーションの最適化**: 明確なプロセスと責任の分担。
- **継続的改善の文化**: データ駆動型のプロセス最適化。

### 実装の成功要因

#### 1. 段階的自動化戦略

- シンプルなタスクから自動化を開始する。
- 段階的に複雑さとカバレッジを増やす。
- 監視とアラートメカニズムを確立する。
- プロセス効率を継続的に最適化する。

#### 2. スマートレビューメカニズム

- リスクレベルに基づく動的レビュー。
- レビュープロセスの自動管理。
- マルチチャネル通知と追跡。
- レビュー履歴の完全な記録。

#### 3. エンタープライズレベル統合機能

- 既存システムとのシームレスな統合。
- マルチクラウド環境のサポート。
- セキュリティとコンプライアンスの考慮。
- スケーラブルなアーキテクチャ設計。

### 手動デプロイからスマートデリバリーへの進化

**フェーズ1: 手動デプロイメント** → **フェーズ2: 自動化パイプライン** → **フェーズ3: スマートデリバリー**

私たちは現在第3フェーズに入っており、その特徴は：

- **スマート意思決定**: 履歴データとリアルタイム監視に基づく自動決定。
- **適応型プロセス**: プロジェクトの特性とチームの習慣に基づく自動調整。
- **予測的メンテナンス**: 潜在的な問題を事前に特定して防止。
- **継続的学習**: 各デプロイメントから学習しプロセスを最適化。

### 今後の展望: AI駆動型CI/CD

- **スマートテスト選択**: AIがコード変更を分析し、テスト戦略をインテリジェントに選択。
- **自動問題診断**: 機械学習モデルが失敗原因を自動分析。
- **予測的デプロイメント**: リスク評価に基づくデプロイメントタイミングの最適化。
- **スマートリソース管理**: コンピューティングリソースを動的に調整してコストを最適化。

**CI/CDにより、私たちは単なる「コードをデプロイする作業者」ではなく、「スマートデリバリーシステムを設計するアーキテクト」になることができます。**

これが完全なCI/CD自動化の真の価値です：ソフトウェア配信を信頼性が高く、迅速で、予測可能にするだけでなく、より重要なことに、継続的に学習し改善できるインテリジェントシステムを確立します。この急速に変化する技術環境において、現代のCI/CDプラクティスをマスターすることは、すべてのソフトウェアチームの中核的能力となっています。

完全なCI/CDパイプラインを通じて、コード提出から本番デプロイメントまでの完全な自動化を実現しながら、必要な手動レビューとリスク管理を維持します。このバランスにより、自動化の効率向上を享受しながら、ビジネスのセキュリティと安定性を確保できます。これが現代のソフトウェアエンジニアリングの核心です：**技術とビジネスの完璧な融合**。
```
