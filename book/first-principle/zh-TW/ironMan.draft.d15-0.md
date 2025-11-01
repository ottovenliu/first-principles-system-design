# Day 15 | CI/CD 全自動化實作 - GitHub Actions × CodePipeline × CodeBuild : 持續整合部署流水線與任務分割管理

在現代軟體開發中，我們不僅要關注程式碼的品質，更要確保整個交付流程的可靠性。還記得我們在 <版本控制策略(PRReview strategy)> 中所提到的 `想像一下，每一個 PR 就像是一份「產品提案」，它必須經過一個嚴謹的「評審審查程序」，才能被納入公司的正式產品線（合併到主分支）。...`嗎?

```mermaid
flowchart TD
    A[開發者提交 PR] --> B{自動化檢查}
    B -->|失敗| C[修復問題]
    C --> A
    B -->|通過| D[指派審核者]

    subgraph "個人端"
      P1[開發者自我檢查]
      P2[本地測試執行]
      P3[代碼格式化]
      P4[提交訊息規範確認]

      P1 --> P2
      P2 --> P3
      P3 --> P4
      P4 --> A
    end

    subgraph "審核端"
      D[指派審核者] --> E[技術主管審核]
      D --> F[資深工程師審核]
      D --> G[產品負責人審核]

      E --> H{技術審核通過?}
      F --> I{代碼品質審核通過?}
      G --> J{業務邏輯審核通過?}

      H -->|否| K[要求修改]
      I -->|否| K
      J -->|否| K
      K --> A

      H -->|是| L{所有審核都通過?}
      I -->|是| L
      J -->|是| L

      L -->|否| M[等待其他審核]
      L -->|是| N[最終批准]
    end

    N --> O{衝突檢查}
    O -->|有衝突| P[解決衝突]
    P --> A
    O -->|無衝突| Q[合併到目標分支]

    Q --> R[自動部署]
    Q --> S[通知相關人員]

    style A fill:#e3f2fd
    style Q fill:#e8f5e8
    style K fill:#ffebee
    style R fill:#f3e5f5
```

在這個章節中我們在流程圖中稍微提到了一下 **自動化檢查與自動部署(CI/CD)** ，但當時在這個流程環節中我們要怎麼去執行沒有特別多說，而是專注於分析 Git ，或者說開發成果的階段性品質管理中，但在這個環節中我們可以減少每一次在本地端提交的耗時，將一些與 `發布環境功能測試` 、 `系統間整合測試` 、 `e2e測試` 的環節同樣應用在一個獨立運行的環境自動化去檢測，這就是 **CI（Continuous Integration，持續整合）** 最主要的功能之一。

這個流程免除了我們每一次的本地端執行測試的成本，也盡可能地僅少開發與發佈環境差異(開發過經驗中也是有遇到過在本地端測試不過而在商品端無恙的情境，相反的狀況也有) ，同時每一個執行的 Task 與 Job 也是一個 **商業邏輯實現** 的 **積極性保護** ，所以今天我們將深入探討如何透過 GitHub Actions、AWS CodePipeline 和 CodeBuild 建立一個企業級的 CI/CD 自動化流水線，並特別關注：

- **Jobs 的碎片化管理**：如何合理分割和組織 CI/CD 任務
- **Task 版控管理**：如何對 CI/CD 流程進行版本控制
- **參數抽離管理**：如何管理不同環境的配置參數
- **業務審核門檻**：如何在自動化流程中加入人工審核機制

讓我們先回顧一下沒有 CI/CD 時代的軟體部署困境：

**情境一：手動部署的恐懼**

```
週五下午 5 點...

開發者：「我要部署新功能到生產環境了」
同事們：「等等！週五不要部署！」
開發者：「為什麼？」
同事們：「因為如果出問題，我們就要加班到週末...」

**問題核心**：
- 部署過程依賴人工操作，容易出錯
- 缺乏標準化流程，每次部署都是冒險
- 回滾困難，出問題影響範圍大
- 團隊對部署缺乏信心
```

**### **情境二：環境不一致的災難\*\*

```
開發者：「奇怪，在我的電腦上明明可以運行...」
測試人員：「但是在測試環境就是有 bug」
運維工程師：「生產環境又跟測試環境不一樣...」
產品經理：「什麼時候才能修好？客戶在等著呢！」

**問題核心**：
- 開發、測試、生產環境配置不一致
- 依賴管理混亂，版本衝突頻繁
- 缺乏自動化測試，問題發現太晚
- 環境建置耗時，影響開發效率
```

**情境三：部署流程的黑盒子**

```
新人工程師：「我要怎麼部署我的程式碼？」
資深工程師：「嗯...讓我想想...」
「首先你要 SSH 到伺服器...」
「然後 git pull...」
「接著重新編譯...」
「記住要重啟 nginx...」
「還有清除快取...」
「對了，別忘了備份資料庫...」

新人：「😵‍💫」

**問題核心**：
- 部署知識集中在少數人身上
- 流程複雜且缺乏文檔化
- 新人學習成本高
- 關鍵人員離職風險大
```

有沒有發現這些軟體交付的痛點跟我們在 <Infrastructure as Code : Terraform 基礎設施代碼化與版本管控> 章節中的情境很像? 當我們要進行檢測與部屬的時候沒有一個 **穩定的環境** 與 **固定的商業邏輯測試流程**，很容易讓開發團隊每次部署都如履薄冰。**CI（Continuous Integration，持續整合）** 與 **CD（Continuous Deployment/Delivery，持續部署/交付）** 最重要且為什麼必須的原因就在於建立一個穩定且固定的商業邏輯驗證交付流程，**`積極性的保護既有商業邏輯不被程式碼異動所破壞汙染`** 。

## CI/CD 核心概念與價值

```
保護既有商業邏輯不被程式碼異動所破壞汙染
```

### 什麼是 CI/CD？

**CI（Continuous Integration，持續整合）**：開發者頻繁地將程式碼變更合併到主分支，每次合併都會觸發自動化的建置和測試流程。

**CD（Continuous Deployment/Delivery，持續部署/交付）**：透過自動化流程，將通過測試的程式碼自動部署到各個環境，甚至直接部署到生產環境。

### CI/CD 的核心價值

```mermaid
graph TD
    A[CI/CD 核心價值] --> B[快速反饋<br/>Fast Feedback]
    A --> C[降低風險<br/>Risk Reduction]
    A --> D[提高效率<br/>Efficiency Boost]
    A --> E[品質保證<br/>Quality Assurance]

    B --> B1[早期發現問題]
    B --> B2[快速修復缺陷]

    C --> C1[小批次發布]
    C --> C2[快速回滾能力]

    D --> D1[自動化取代手工]
    D --> D2[並行處理流程]

    E --> E1[自動化測試覆蓋]
    E --> E2[一致性環境]
```

想像一下：`從「手工作坊」到「自動化工廠」的轉變`

過去的軟體開發就像一個手工作坊：

- **手工製作**：每個產品（軟體版本）都需要工匠（開發者）親自製作
- **品質不穩定**：依賴工匠的技術和當天的狀態
- **產量有限**：一個工匠一天只能製作有限的產品
- **知識集中**：技術掌握在少數工匠手中

而 CI/CD 就像建立了一條自動化的生產線：

- **標準化流程**：每個步驟都有明確的標準和自動化檢查
- **品質保證**：每個產品都經過相同的品質檢測流程
- **高效產出**：機器不會疲倦，可以 24/7 持續工作
- **知識共享**：流程被編碼化，任何人都可以理解和改進

## GitHub Actions 深度實戰

GitHub Actions 是 GitHub 提供的 CI/CD 平台，讓我們可以直接在程式碼倉庫中定義和執行自動化工作流程。

### GitHub Actions 核心概念

```mermaid
graph LR
    A[GitHub Actions 架構] --> B[Workflow<br/>工作流程]
    A --> C[Job<br/>作業]
    A --> D[Step<br/>步驟]
    A --> E[Action<br/>動作]

    B --> B1[.github/workflows/*.yml]
    B --> B2[觸發條件定義]

    C --> C1[運行環境選擇]
    C --> C2[並行執行支援]

    D --> D1[具體執行指令]
    D --> D2[環境變數設定]

    E --> E1[可重用的程式碼單元]
    E --> E2[Marketplace 生態]
```

### Jobs 碎片化管理策略

在企業級 CI/CD 流程中，合理的 Jobs 分割是至關重要的。我們需要平衡執行效率和可維護性：

#### 1. 基礎 Jobs 分割原則

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
  # 第一層：基礎檢查（快速失敗）
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

  # 第二層：測試執行（並行處理）
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

  # 第三層：安全性和品質檢查
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

  # 第四層：建置和部署準備
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
          # 檢查關鍵檔案
          test -f dist/index.html || { echo "❌ index.html not found"; exit 1; }
          test -f dist/version.txt || { echo "❌ version.txt not found"; exit 1; }

          # 檢查建置大小
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

  # 第五層：部署到測試環境
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

  # 第六層：業務審核門檻
  approval-gate:
    name: "Production Approval"
    runs-on: ubuntu-latest
    needs: [build, deploy-staging]
    if: github.ref == 'refs/heads/main'
    timeout-minutes: 1440 # 24 hours timeout

    environment:
      name: production-approval

    steps:
      - name: Request production deployment approval
        run: |
          echo "🔍 Production deployment approval requested"
          echo "📋 Build version: ${{ needs.build.outputs.build-version }}"
          echo "🌐 Staging URL: https://staging.myapp.com"
          echo "⏰ Approval timeout: 24 hours"

  # 第七層：生產環境部署
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

          for i in {1..5}; do
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

#### 2. 企業級 Jobs 模組化與跨領域引用

除了在單一 Workflow 檔案中分割 Jobs，為了符合企業級的共用標準與可維護性，我們可以將屬於特定領域 (Domain) 或功能的 Jobs 獨立切割成不同的可重用工作流程 (Reusable Workflows)。當有需要時，主流程就可以像函式庫一樣引用並執行這些獨立的 Jobs。

這個做法最大的好處在於，它讓不同系統間的 Domain 交互邏輯得以互相驗證。例如，當 `user-service` 的 CI 流程更新後，`order-service` 的 CI 流程可以透過引用 `user-service` 的測試 Job，來確保這次的變更沒有破壞到訂單流程中與使用者相關的功能，從而避免「在我的 Domain 沒問題，但卻影響到其他 Domain」的窘境。

**情境：使用者服務 (User Service) 的變更，需要確保訂單服務 (Order Service) 不受影響**

首先，我們為 `user-service` 定義一個可重用的 CI 工作流程，其中包含一個專門用來驗證核心功能的 Job。

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
          # 執行針對核心功能的整合測試
          pnpm run test:core-integration
          echo "summary=User core logic validated successfully" >> $GITHUB_OUTPUT

      - name: Create test summary
        id: test-summary
        run: echo "summary=✅ User Service core validation passed" >> $GITHUB_OUTPUT
```

接著，在 `order-service` 的主 CI/CD 流程中，我們就可以引用上述的 Job 來進行跨領域的整合驗證。

```yaml
# .github/workflows/order-service-ci.yml
name: "Order Service CI/CD"

on:
  push:
    branches: [main]
  pull_request:

jobs:
  # 執行訂單服務自己的測試
  test-order-service:
    name: "Test Order Service"
    runs-on: ubuntu-latest
    steps:
      - name: Checkout Order Service code
        uses: actions/checkout@v4
      # ... (Run order service's own tests) ...
      - name: Run order tests
        run: pnpm run test

  # 引用 User Service 的 CI Job 進行整合驗證
  cross-validate-with-user-service:
    name: "Cross-Domain Validation (User Service)"
    needs: test-order-service
    uses: ./.github/workflows/reusable-user-service-ci.yml
    with:
      node-version: "18"

  # 只有在自己的測試和跨領域驗證都通過後，才進行建置
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
      # ... (Build steps) ...
      - name: Build application
        run: pnpm run build
```

當然，若是要再將顆粒度縮得更小至 `Step` 也是可以實作的

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

透過這種方式，我們將 CI/CD 的任務從「單一專案的品質保證」提升到了「跨專案、跨領域的系統穩定性保護」，這在微服務或複雜系統架構中至關重要。

#### 3. 可重用的 Action 建立

為了提高代碼重用性，我們建立自定義的 Action：

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

### Task 版控管理策略

#### 1. Workflow 版本化配置

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

          # 版本相容性檢查
          case "${{ env.WORKFLOW_VERSION }}" in
            v1.*)
              echo "⚠️ Using legacy workflow version"
              ;;
            v2.*)
              echo "✅ Using current workflow version"
              ;;
            *)
              echo "❌ Unknown workflow version"
              exit 1
              ;;
          esac
```

#### 2. Pipeline 設定檔案管理

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

#### 3. 動態 Pipeline 載入

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
          # 根據分支選擇配置檔案
          if [[ "${{ github.ref }}" == "refs/heads/main" ]]; then
            CONFIG_FILE=".github/pipeline-configs/production.yml"
          elif [[ "${{ github.ref }}" == "refs/heads/develop" ]]; then
            CONFIG_FILE=".github/pipeline-configs/staging.yml"
          else
            CONFIG_FILE=".github/pipeline-configs/development.yml"
          fi

          # 載入並輸出配置
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

#### 4. 跨領域可重用 Workflow 的版控

當我們將 Jobs 模組化為可重用的 Workflow (如 `reusable-user-service-ci.yml`) 後，對這些共用 Workflow 進行版本控制就變得至關重要。這確保了當一個核心服務 (如 `user-service`) 的 CI 流程發生變更時，不會意外地破壞依賴它的其他服務 (如 `order-service`) 的建置流程。

**情境：`user-service` 的 CI 流程從 v1 升級到 v2，但 `order-service` 暫時仍需使用 v1**

我們可以透過 Git 的標籤 (Tag) 來對可重用的 Workflow 進行版控。

首先，`user-service` 的 `reusable-user-service-ci.yml` 檔案本身不需要特別標示版本。它的版本是透過其所在的 Git 倉庫的標籤來管理的。當我們認為 `reusable-user-service-ci.yml` 達到一個穩定狀態時，就在該 commit 上打上一個版本標籤，例如 `v1.0` 或 `v2.0`。

接著，在 `order-service` 的 CI 流程中，我們可以明確指定要使用的 `user-service` CI 的版本。

```yaml
# .github/workflows/order-service-ci.yml
name: "Order Service CI/CD"

on:
  push:
    branches: [main]
  pull_request:

jobs:
  test-order-service:
    # ... (執行訂單服務自己的測試) ...
    name: "Test Order Service"
    runs-on: ubuntu-latest
    steps:
      - name: Checkout Order Service code
        uses: actions/checkout@v4
      - name: Run order tests
        run: pnpm run test

  # 引用 User Service CI 的 v1.0 版本進行驗證
  cross-validate-with-user-service-v1:
    name: "Cross-Domain Validation (User Service @v1.0)"
    needs: test-order-service
    # 透過 @<tag> 的語法來鎖定可重用 Workflow 的版本
    uses: my-org/user-service/.github/workflows/reusable-user-service-ci.yml@v1.0
    with:
      node-version: "18"

  # 當團隊準備好時，可以建立一個新的 PR 來升級到 v2.0
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

這種策略將「Task 版控」的概念從單一專案內部，擴展到了跨專案、跨服務的企業級維度，實現了真正意義上的 CI/CD 流程即代碼 (Pipeline as Code) 的版本化管理。

### 參數抽離管理策略

#### 1. 環境變數分層管理

```yaml
# .github/environments/production.yml
variables:
  # 公開變數
  NODE_ENV: "production"
  API_VERSION: "v2"
  BUILD_TARGET: "production"
  CACHE_TTL: "3600"

  # CDN 配置
  CDN_DOMAIN: "cdn.myapp.com"
  ASSET_PREFIX: "/static"

  # 功能開關
  FEATURE_NEW_UI: "true"
  FEATURE_ANALYTICS: "true"
  FEATURE_AB_TESTING: "true"

secrets:
  # 敏感資訊（需在 GitHub Secrets 中設定）
  - AWS_ACCESS_KEY_ID
  - AWS_SECRET_ACCESS_KEY
  - SENTRY_DSN
  - ANALYTICS_API_KEY
  - DATABASE_CONNECTION_STRING
```

#### 2. 動態環境變數注入

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
          # 根據目標環境載入配置
          case "${{ github.ref }}" in
            refs/heads/main)
              ENV_FILE=".github/environments/production.yml"
              ;;
            refs/heads/develop)
              ENV_FILE=".github/environments/staging.yml"
              ;;
            *)
              ENV_FILE=".github/environments/development.yml"
              ;;
          esac

          # 載入環境變數
          ENV_VARS=$(yq '.variables' $ENV_FILE)

          # 輸出為 JSON 格式
          echo "config=$ENV_VARS" >> $GITHUB_OUTPUT

          # 設定當前 job 的環境變數
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
          # 從上一個 job 獲取環境配置
          ENV_CONFIG='${{ needs.setup-environment.outputs.env-config }}'
          echo "$ENV_CONFIG" | jq -r 'to_entries | .[] | "\(.key)=\(.value)"' >> $GITHUB_ENV

      - name: Build application
        run: |
          echo "🏗️ Building with environment: $NODE_ENV"
          echo "🌐 API Version: $API_VERSION"
          echo "🎯 Build Target: $BUILD_TARGET"

          # 建立環境資訊檔案
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

          # 執行建置
          npm run build
```

#### 3. 秘密參數安全管理

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
          # 引用所需的秘密
          AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          SENTRY_DSN: ${{ secrets.SENTRY_DSN }}
          SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK_URL }}

        run: |
          # 驗證必要的秘密是否存在
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
          # 測試 AWS 憑證有效性
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

## AWS CodePipeline 與 CodeBuild 企業級整合

對於大型企業環境，我們需要更強大的 CI/CD 基礎設施。AWS CodePipeline 提供了更完整的企業級功能。

### CodePipeline 完整流程設計

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

  # 第一階段：源碼獲取
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

  # 第二階段：程式碼品質檢查
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

  # 第三階段：測試執行
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

  # 第四階段：建置
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

  # 第五階段：部署到測試環境
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

  # 第六階段：業務審核
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

  # 第七階段：生產環境部署
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
      name            = "DeployToProduction"
      category        = "Deploy"
      owner           = "AWS"
      provider        = "S3"
      version         = "1"
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

  # 第八階段：部署後驗證
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

# Pipeline 失敗處理
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

### CodeBuild 專案進階配置

#### 1. 多階段建置專案

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

      # 安裝 Node.js 相關工具
      - npm install -g pnpm@8.15.0
      - npm install -g npm-check-updates

      # 安裝 Python 相關工具
      - pip install --upgrade pip
      - pip install boto3 requests

      # 安裝其他工具
      - curl -sSfL https://raw.githubusercontent.com/anchore/syft/main/install.sh | sh -s -- -b /usr/local/bin

  pre_build:
    commands:
      - echo "🔍 Pre-build validation started"

      # 環境變數驗證
      - |
        REQUIRED_VARS=("ENVIRONMENT" "BUILD_NUMBER" "COMMIT_SHA")
        for var in "${REQUIRED_VARS[@]}"; do
          if [ -z "${!var}" ]; then
            echo "❌ Required environment variable $var is not set"
            exit 1
          fi
        done
        echo "✅ All required environment variables are set"

      # 依賴安裝和緩存
      - echo "📦 Installing dependencies..."
      - pnpm install --frozen-lockfile

      # 依賴安全檢查
      - echo "🔒 Running dependency security audit..."
      - pnpm audit --audit-level=moderate

      # 生成建置資訊
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

      # 程式碼品質檢查
      - echo "🔍 Running code quality checks..."
      - pnpm run lint:check
      - pnpm run format:check
      - pnpm run type-check

      # 單元測試
      - echo "🧪 Running unit tests..."
      - pnpm run test:unit --coverage --reporter=junit --outputFile=test-results.xml

      # 建置應用程式
      - echo "📦 Building application..."
      - pnpm run build

      # 建置後驗證
      - echo "✅ Post-build validation..."
      - |
        # 檢查建置檔案
        if [ ! -f "dist/index.html" ]; then
          echo "❌ Main entry file not found"
          exit 1
        fi

        # 檢查建置大小
        BUNDLE_SIZE=$(du -sk dist | cut -f1)
        echo "📊 Bundle size: ${BUNDLE_SIZE}KB"

        if [ $BUNDLE_SIZE -gt 51200 ]; then  # 50MB
          echo "⚠️ Bundle size is large: ${BUNDLE_SIZE}KB"
        fi

        # 檢查關鍵資源
        ASSET_COUNT=$(find dist -name "*.js" -o -name "*.css" | wc -l)
        echo "📄 Assets generated: $ASSET_COUNT files"

      # 產生 SBOM (Software Bill of Materials)
      - echo "📋 Generating SBOM..."
      - syft packages dir:. -o spdx-json > sbom.spdx.json

      # 建置 Docker 映像檔（如果需要）
      - |
        if [ -f "Dockerfile" ]; then
          echo "🐳 Building Docker image..."
          docker build -t ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${IMAGE_REPO_NAME}:${BUILD_NUMBER} .
          docker build -t ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${IMAGE_REPO_NAME}:latest .
        fi

  post_build:
    commands:
      - echo "🎯 Post-build phase started"

      # 測試覆蓋率檢查
      - |
        if [ -f "coverage/lcov.info" ]; then
          COVERAGE=$(lcov --summary coverage/lcov.info 2>&1 | grep "lines" | cut -d' ' -f4 | cut -d'%' -f1)
          echo "📊 Test coverage: ${COVERAGE}%"
          
          if (( $(echo "$COVERAGE < 80" | bc -l) )); then
            echo "⚠️ Test coverage below threshold: ${COVERAGE}% < 80%"
          fi
        fi

      # 推送 Docker 映像檔（如果需要）
      - |
        if [ -f "Dockerfile" ]; then
          echo "🚢 Pushing Docker image..."
          aws ecr get-login-password --region $AWS_DEFAULT_REGION | docker login --username AWS --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com
          docker push ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${IMAGE_REPO_NAME}:${BUILD_NUMBER}
          docker push ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${IMAGE_REPO_NAME}:latest
        fi

      # 部署準備
      - echo "🎁 Preparing deployment package..."
      - cp build-info.json dist/
      - tar -czf deployment-package.tar.gz -C dist .

      # 建置報告
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
    files:
      - "coverage/lcov.info"
    file-format: "CLOVERXML"

cache:
  paths:
    - "node_modules/**/*"
    - ".pnpm-store/**/*"
    - "/root/.cache/pip/**/*"
```

#### 2. 環境特定的建置配置

```terraform
# CodeBuild 專案 - 環境特定配置
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

    # 敏感環境變數
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

# 環境配置變數
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

## 業務審核門檻實作

最終在企業環境中，生產部署必須經過一個嚴謹的 **業務邏輯確認審核**，才能被納入公司的正式產品發布，我們接下來將實作一個的結合 **同儕審核門檻** 、 **業務審核門檻** 、 **品質審核門檻** 三方驗證的審核機制流：

```mermaid
graph TD
    subgraph "智慧審核與通知 Smart Review & Notification"
        A[部署請求觸發<br/>Deployment Request Triggered] --> B{分析變更風險與類型<br/>Analyze Change Risk & Type}
        B --> C{決定審核路徑<br/>Determine Approval Path}
        C --> D[建立審核記錄<br/>Create Approval Record<br/>using DynamoDB]
        D --> E[發送多通道審核通知<br/>Send Multi-Channel Notifications<br/>Slack and Email]
    end

    subgraph "多層級審核流程 Multi-Level Approval Gates"
        E --> F{需要技術審核?<br/>Technical Review Required?}
        F -->|是 Yes| G[技術主管審核<br/>Technical Lead Review<br/>via Slack/UI]
        F -->|否 No| J{需要業務審核?<br/>Business Review Required?}
        G --> H{通過?<br/>Approved?}
        H -->|否 No| Z[審核失敗<br/>Approval Failed]
        H -->|是 Yes| J
        J -->|是 Yes| K[產品經理審核<br/>Product Manager Review<br/>via Slack/UI]
        J -->|否 No| P{需要安全審核?<br/>Security Review Required?}
        K --> L{通過?<br/>Approved?}
        L -->|否 No| Z
        L -->|是 Yes| P
        P -->|是 Yes| Q[安全團隊審核<br/>Security Team Review<br/>via Slack/UI]
        P -->|否 No| T[所有審核完成<br/>All Approvals Complete]

        Q --> R{通過?<br/>Approved?}
        R -->|否 No| Z
        R -->|是 Yes| T
    end

    subgraph "最終授權與執行 Final Authorization & Execution"
        T --> U[最終部署授權<br/>Final Deployment Authorization]
        U --> V[✅ 觸發生產部署<br/>Trigger Production Deployment]
        Z --> W[❌ 通知部署失敗與原因<br/>Notify Failure and Reason]
    end

    style A fill:#e3f2fd,stroke:#333,stroke-width:2px
    style V fill:#e8f5e9,stroke:#333,stroke-width:2px
    style W fill:#ffebee,stroke:#333,stroke-width:2px
    style Z fill:#ffebee,stroke:#333,stroke-width:2px
```

### 1. 多層級審核流程

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
  # 第一層：技術審核
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

  # 第二層：業務審核
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
          echo "============================"
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

  # 第三層：安全審核（高風險變更）
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
          echo "============================"
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

  # 最終審核：部署授權
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

### 2. 智慧審核規則

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

          # 預設審核需求
          REQUIRES_TECHNICAL="true"
          REQUIRES_BUSINESS="false"
          REQUIRES_SECURITY="false"
          REQUIRES_EMERGENCY="false"

          echo "📊 Analyzing change requirements..."
          echo "Change Type: $CHANGE_TYPE"
          echo "Risk Level: $RISK_LEVEL"
          echo "Affected Components: $AFFECTED_COMPONENTS"

          # 風險等級判斷
          case "$RISK_LEVEL" in
            "low")
              REQUIRES_BUSINESS="false"
              REQUIRES_SECURITY="false"
              ;;
            "medium")
              REQUIRES_BUSINESS="true"
              REQUIRES_SECURITY="false"
              ;;
            "high"|"critical")
              REQUIRES_BUSINESS="true"
              REQUIRES_SECURITY="true"
              ;;
          esac

          # 變更類型判斷
          case "$CHANGE_TYPE" in
            "hotfix"|"emergency")
              REQUIRES_EMERGENCY="true"
              REQUIRES_BUSINESS="false"
              REQUIRES_SECURITY="false"
              ;;
            "security-patch")
              REQUIRES_SECURITY="true"
              ;;
            "feature")
              REQUIRES_BUSINESS="true"
              ;;
          esac

          # 受影響組件判斷
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

          # 輸出結果
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

### 3. 審核通知與追蹤

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
        """建立審核請求"""
        approval_id = f"approval-{deployment_info['build_number']}-{int(datetime.now().timestamp())}"

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

        # 儲存到 DynamoDB
        self.approval_table.put_item(Item=approval_record)

        # 發送通知
        self.send_approval_notifications(approval_record)

        return approval_id

    def send_approval_notifications(self, approval_record):
        """發送審核通知"""

        # Email 通知
        self.send_email_notification(approval_record)

        # Slack 通知
        self.send_slack_notification(approval_record)

        # Teams 通知
        self.send_teams_notification(approval_record)

    def send_slack_notification(self, approval_record):
        """發送 Slack 通知"""
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
        """處理審核回應"""

        # 獲取審核記錄
        response = self.approval_table.get_item(Key={'approval_id': approval_id})
        approval_record = response.get('Item')

        if not approval_record:
            return {'error': 'Approval record not found'}

        if approval_record['status'] != 'pending':
            return {'error': 'Approval already completed or expired'}

        # 更新審核狀態
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

        # 檢查是否所有必要審核都已完成
        required_approvals = set(approval_record['required_approvals'])
        completed_approvals = set([a['type'] for a in approval_record['completed_approvals']])

        if decision == 'approve' and required_approvals.issubset(completed_approvals):
            approval_record['status'] = 'approved'
            approval_record['approved_at'] = datetime.now().isoformat()

            # 觸發部署流程
            self.trigger_deployment(approval_record)

        # 更新資料庫記錄
        self.approval_table.put_item(Item=approval_record)

        # 發送狀態更新通知
        self.send_status_update(approval_record, approval_type, approver, decision)

        return {'status': approval_record['status']}

    def trigger_deployment(self, approval_record):
        """觸發生產部署"""

        # 通過 GitHub API 觸發部署 workflow
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
        """發送狀態更新通知"""

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

        # 發送 Slack 更新
        webhook_url = os.environ.get('SLACK_WEBHOOK_URL')
        requests.post(webhook_url, json={"text": message})

# Lambda 函數處理器
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

## 建構現代化軟體交付流水線

CI/CD 全自動化實作為現代軟體開發帶來了根本性的變革。我們從傳統手動部署的痛點出發，深入探討了完整的自動化流水線建置，特別關注了企業級需求：

### 核心價值重申

**技術層面的轉變**：

- **Jobs 碎片化管理**：合理分割任務，提高執行效率和可維護性
- **Task 版控管理**：對 CI/CD 流程進行版本化管理，確保可追蹤性
- **參數抽離管理**：環境配置分離，支援多環境部署
- **業務審核門檻**：在自動化中融入人工審核，平衡效率與風險

**組織層面的影響**：

- **效率與品質並重**：自動化提升效率，審核機制保證品質
- **風險可控的敏捷**：快速交付與風險控制的平衡
- **團隊協作最佳化**：明確的流程和責任分工
- **持續改進文化**：數據驅動的流程最佳化

### 實施關鍵成功因素

#### 1. 漸進式自動化策略

- 從簡單任務開始自動化
- 逐步增加複雜度和覆蓋範圍
- 建立監控和告警機制
- 持續最佳化流程效率

#### 2. 智慧化審核機制

- 基於風險等級的動態審核
- 自動化的審核流程管理
- 多通道的通知和追蹤
- 審核歷史的完整記錄

#### 3. 企業級整合能力

- 與現有系統的無縫整合
- 多雲環境的支援能力
- 安全性和合規性考量
- 可擴展的架構設計

### 從手動部署到智慧化交付的演進

**第一階段：手動部署** → **第二階段：自動化流水線** → **第三階段：智慧化交付**

我們正在進入第三階段，其特點包括：

- **智慧決策**：基於歷史數據和實時監控的自動決策
- **自適應流程**：根據專案特性和團隊習慣自動調整
- **預測性維護**：提前識別和預防潛在問題
- **持續學習**：從每次部署中學習並最佳化流程

### 未來展望：AI 驅動的 CI/CD

- **智慧測試選擇**：AI 分析程式碼變更，智慧選擇測試策略
- **自動問題診斷**：機器學習模型自動分析失敗原因
- **預測性部署**：基於風險評估的部署時機最佳化
- **智慧資源管理**：動態調整運算資源以最佳化成本

**CI/CD 讓我們不再是「部署程式碼的操作員」，而是「設計智慧交付系統的架構師」。**

這就是 CI/CD 全自動化的真正價值：它不僅讓軟體交付變得可靠、快速、可預測，更重要的是建立了一個能夠持續學習和改進的智慧化系統。在這個快速變化的技術環境中，掌握現代 CI/CD 實踐已經成為每個軟體團隊的核心競爭力。

透過完整的 CI/CD 流水線，我們實現了從程式碼提交到生產部署的全程自動化，同時保持了必要的人工審核和風險控制。這種平衡讓我們既能享受自動化帶來的效率提升，又能確保業務的安全性和穩定性。這就是現代軟體工程的核心：**技術與業務的完美融合**。
