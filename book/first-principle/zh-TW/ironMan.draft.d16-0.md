# Day 16 | Dev / Staging / Prod 多環境治理與架構策略: AWS 多環境配置管理與部署策略

在先前兩個主題 <Infrastructure as Code : Terraform 基礎設施代碼化與版本管控> 與 <CI/CD 全自動化實作 - GitHub Actions × CodePipeline × CodeBuild>中，我們分別講述了 `**「讓基礎設施變得可預測、可重複、可版本化」**` 的重要性與 **`保護既有商業邏輯不被程式碼異動所破壞汙染`** 的積極性。兩者同樣都是為了解決 `要進行檢測與部屬的時候沒有一個 穩定的環境 與 固定的商業邏輯驗證` 的議題並確保我們 **`商業邏輯驗證`** 能夠在穩定的土壤中成長茁壯成為參天大樹(我說的不是台灣某銀行)，簡單聊聊環境的重要性與對應痛點的解決方案後。接下來，我們將要討論 - **`我需要什麼環境?`** 。

前陣子有幸跟朋友一同探店，前去台北晶華酒店的 Impromptu by Paul Lee 品嘗 2025 夏季風味，那是一個結合開放式廚房與座位區的獨特設計，用餐人士可以簡單且透明的看到主廚與他的團隊夥伴是怎麼將主廚對於 2025 的夏季用食材、香料與選酒呈現一幅美輪美奐、錦羅織景的絕妙風采。我個人非常喜歡這次的菜單設計，可以感覺到清泉瀝瀝的清朗與乾脆俐落的風華。

但話說回來，能夠摘下米其林星的佳餚是怎麼被設計出來的? 有個新的概念是件讓所有老饕興奮與歡欣鼓舞的消息，但假如直接在客人面前的料理台上邊做邊給客人吃，萬一結果失敗了呢？鹽放多了、火候錯了，輕則賠上口碑，重則客人吃壞肚子，明天就上新聞頭條餐廳名譽徹底掃地。

那我們退一步，在餐廳的「主廚房」後場，正式出餐前找個角落試做怎麼樣?

這是一個好方法，我們避免了第一時間客人立即品嘗到實驗性的菜餚，然而，雖然客人看不見，但我們可能會打亂廚房的既有正常運作。我們佔用了正在備料的砧板、需要醬料時發現被其他廚師用完了，甚至實驗油煙可能會影響到旁邊正在精心烹調的出餐。

這樣說來，在資金到位、環境允許的狀況下，我們在一個完全獨立的「研發廚房 (Test Kitchen)」裡去嘗試實現我們新的概念好像是一個最優解。 在這裡，我們有自己全套的鍋碗瓢盆、食材、烤箱，可以盡情地實驗、失敗、重來，完全不用擔心影響到外面的餐廳營運。

這就是 **開發環境 (Development / Dev)** 。它是一個完全隔離的沙盒 (Sandbox)，讓每一位開發者可以自由地撰寫、修改、測試自己的程式碼，而不用擔心「弄壞」任何東西或影響到任何人。它避免了我們在 **「主廚房」後場(共用環境)** 中 影響到正在測試其他功能的同事，反之亦然。最重要的是極大程度的避免在生產環境 (Production / Prod) 直接寫程式碼，任何一個小失誤，比如一個錯誤的資料庫指令，都可能導致所有使用者服務中斷、資料遺失。 **`這是絕對、絕對禁止的行為。`**

在正式端上餐桌前，我們在我們自己的 **研發廚房(開發環境)** 把新菜色做出來了，味道驚為天人。下一步呢？直接把它放上菜單，賣給最重要的美食評論家嗎？

假如我們頭上沒有一個躲在廚師帽裡的大廚，以及端上桌的是法式燉菜的話(料理鼠王是個很好看的作品，有閒暇時間的話可以看看)，這是一個非常冒險的舉動。

在正式推出前，我們需要一個 **「最終演練」** 的場所，這個場所，我們稱之為 「模擬廚房 (Staging Kitchen)」。 它的配置、爐具、動線，甚至連盤子的品牌，都必須和正式的「主廚房」一模一樣。 在這裡，我們要模擬真實的出餐流程 : 服務生要跑一次完整的點單上菜流程、廚師團隊要模擬在尖峰時段的壓力下能否依照設想流程穩定地做出這道菜，最後我們要測試這道新菜和其他經典菜色一起出餐時，會不會有流程上的衝突。

這就是 **預備環境 (Staging / Pre-production)** 的重要性，我們必須有一個環境全面且各種情境的去進行測試確保我們的商業邏輯是符合需求與品質認可。

現在，我們可以將軟體開發的生命週期抽象化為一個清晰的「進程 (Progression)」：

```python

Dev => Staging => Prod

```

- 開發環境 (Dev): The Test Kitchen

  - 目的： 功能開發、單元測試、快速迭代。
  - 特性： 混亂是常態。允許頻繁修改、部署、甚至推倒重來。開發者擁有較高的權限。資料通常是假的或經過匿名化的。
  - 核心關注點： 開發效率。

- 預備環境 (Staging): The Staging Kitchen

  - 目的： 整合測試、使用者驗收測試 (UAT)、效能壓力測試。模擬真實上線的所有情境。
  - 特性： 環境必須極度逼近生產環境。配置、作業系統、網路規則、資料庫版本等都應一致。部署流程也應該和正式上線的流程完全相同。資料可以是生產環境的複製品（需經過去敏化處理）。
  - 核心關注點： 穩定性與一致性 (Consistency)。這是上線前最後的品質防線。

- 生產環境 (Prod): The Michelin-Starred Restaurant

  - 目的： 服務真實使用者，創造價值。
  - 特性： 極度穩定、安全、效能高。任何變更都必須經過嚴格的審批和標準化的流程。權限控管最為嚴格，通常只有自動化工具或極少數授權人員才能進行部署。
  - 核心關注點： 可靠性 (Reliability) 與安全 (Security)。

這個 **`Dev => Staging => Prod`** 的單向流動，就是軟體品質保證的 `核心動線` 。程式碼像水一樣，只能從低環境流向高環境，絕不逆流，每一次流動都是一次 **`「品質閘門 (Quality Gate)」`** 的檢驗，不論是為了 **`保護既有商業邏輯不被程式碼異動所破壞汙染`** 的 **CI（Continuous Integration，持續整合）** 流程中的 **`自動化檢查`** 又或是 <版本控制策略(PRReview strategy)> 中所提到的 **同儕審核門檻** 、 **業務審核門檻** 、 **品質審核門檻** 三方驗證，一旦 **`商業邏輯的實踐任一條件未達標`** 就必須立刻退回與緊急修復，我們必須有一個深刻且鮮明的認知 - **`系統是商業抽象邏輯的實現`**。

在現代軟體開發中，多環境管理是確保應用程式穩定性和可靠性的關鍵。接下來我們將深入探討如何在 AWS 上建構和管理 Dev/Staging/Prod 多環境架構，重點關注 Docker 容器化、ECS 叢集管理、EKS/K8S 編排等核心技術。

> 1. 多環境架構設計原則 - 環境隔離策略與 IaC 實踐
> 2. Docker 容器化策略 - 多階段建構與環境特定配置
> 3. Amazon ECS 叢集管理 - 服務定義與自動擴縮配置
> 4. Amazon EKS 與 Kubernetes 管理 - 叢集配置、應用部署與 Ingress 設定
> 5. CI/CD 管道整合 - GitHub Actions 工作流程與藍綠部署
> 6. 配置管理與秘密管理 - AWS Secrets Manager 與 K8s ConfigMap/Secret
> 7. 監控與日誌管理 - CloudWatch 與 Prometheus/Grafana 整合
> 8. 安全性最佳實踐 - 網路安全與 RBAC 配置
> 9. 成本優化策略 - 資源自動化管理與 Spot Instance 整合
> 10. 災難恢復與備份 - 跨區域備份與資料庫容錯移轉

## 1. 多環境架構設計原則

### 1.1 環境隔離策略

我們上述談到，要把餐廳（我們的系統）分成「研發廚房 (Dev)」、「模擬廚房 (Staging)」和「旗艦總店 (Prod)」，我們來談談如何實際「劃定地界」並確保這三塊地的「建築規範」是一致的。

```python
Production Environment (生產環境)
├── 高可用性配置
├── 自動擴縮容
├── 完整監控告警
└── 嚴格的部署流程

Staging Environment (預發布環境)
├── 生產環境鏡像
├── 完整功能測試
├── 效能測試環境
└── 整合測試驗證

Development Environment (開發環境)
├── 快速部署
├── 開發者友好
├── 資源成本優化
└── 彈性配置調整
```

首先最簡單的方式當然是在給每個廚房上鎖，並給具有對應職能需求的人專屬鑰匙。先鎖好門，避免有些人亂竄，自己人不小心調到參數、配置錯誤導致服務癱瘓不說，至少也要確保連鑰匙都沒有的小偷強盜來進都進不來，所以我們的第一步核心原則很簡單: **`最小化爆炸半徑 (Minimize Blast Radius)`**。 意思是，當一個環境發生災難時（比如被駭客入侵、配置錯誤導致服務癱瘓），絕對不能波及到其他環境。

在 AWS 上，實現這個目標的黃金標準，叫做 **`多帳號策略 (Multi-Account Strategy)`**。

** 1. AWS Organizations：雲端集團總部 **

**`AWS Organizations`** 就是我們的「集團總部」，藉由他我們可以針對不同的 AWS 帳號進行管理與基礎政策的制定。就如同集團下的不同事業體有財務、業務、法務、人資、開發、維護一般，我們可以透過建立一個 `DevOU` 和一個 `ProdOU` 當作事業體的概念。 `Account-Dev`(部門) 放在 `DevOU`(事業體) 裡； `Account-Staging` 和 `Account-Prod` 則放在 `ProdOU` 裡。然後我們可以設定 **`實施集團法規 (Service Control Policies, SCPs)`** 限制旗下帳號 **能** 做什麼或 **不能** 做什麼。這就像是集團下達的「最高行政命令」，當事業體政策與集團總部出現衝突時，會優先且強制依循 `SCPs` 的規範。並且 `AWS Organizations` 還有一個好處，所有事業體的帳單與花費成本都匯總到總部，方便監控與管理。

`SCPs` 是從根源上防止了人為失誤或惡意行為，這比事後補救要有效得多。它定義了每個環境的「行為憲法」與「物理定律」。例如:

- 在 DevOU 底下的所有帳號：
  - 禁止 建立連到外部網路的資料庫 (RDS public access = false)。
  - 禁止 關閉 CloudTrail (稽核日誌)，確保所有操作都有紀錄。
  - 限制 只能在亞太區域（如東京、新加坡）建立資源，避免有人不小心把資源開到昂貴的歐美區域
- 在 ProdOU 底下的所有帳號：
  - 除了 特定管理員角色外，禁止 任何人刪除資料庫或 S3 儲存桶。

** 2. IAM 權限：不同身份的「通行證」 **

帳號隔離後，人要怎麼進去工作呢？答案是 IAM 角色 (IAM Roles)，而不是給每個人在每個帳號都開一組帳密 (IAM User)。

- 概念： 在我們的主帳號 (Management Account) 建立 IAM User，然後定義好不同的 IAM Role (角色)，例如 DeveloperRole, QARole, OperatorRole。再讓這些 User 可以「切換身份 (Assume Role)」到其他帳號去執行任務。
- 比喻： 我們是公司的員工（IAM User），公司給我們不同的工作證（IAM Role）。
  - 拿著「開發人員證」，我們可以刷卡進入 Account-Dev 的所有房間。
  - 拿著「品保人員證」，我們可以進入 Account-Staging，但只能觀察和記錄，不能改動設備。
  - 只有「總店經理證」，才能在審批後，進入 Account-Prod 進行操作。

這種方式讓權限集中管理，安全且清晰。

### 1.2 基礎設施即程式碼 (IaC)

我們確保了環境之間有銅牆鐵壁，但如何保證「模擬廚房」和「旗艦總店」的內部裝潢、管線、爐具都一模一樣？這就是 `基礎設施即程式碼 (Infrastructure as Code, IaC)` 的威力所在，按照 `藍圖(IaC)` 我們可以快速建置出一模一樣的架構與環境出來。我們在<Infrastructure as Code : Terraform 基礎設施代碼化與版本管控>有提到藍圖的重要性，它讓基礎設施變得 `可預測`、`可重複`、`可進化`，讓我們繼續用 Terraform 為例。

```
terraform/
├── modules/
│   ├── vpc/
│   │   ├── main.tf
│   │   └── variables.tf
│   └── ec2_instance/
│       ├── main.tf
│       └── variables.tf
│
└── environments/
    ├── dev/
    │   ├── main.tf
    │   └── terraform.tfvars
    ├── staging/
    │   ├── main.tf
    │   └── terraform.tfvars
    └── prod/
        ├── main.tf
        └── terraform.tfvars
```

- modules/ 存放我們標準化的建築模組。
- environments/ 存放每個環境的「施工指令」。

在 `environments/dev/main.tf` 裡，我們用 fake code 呈現：

```yaml
# 引用 VPC 模組
module "my_vpc" {
  source = "../../modules/vpc"
  cidr_block = var.vpc_cidr
}

# 引用 EC2 模組
module "web_server" {
  source      = "../../modules/ec2_instance"
  instance_type = var.instance_type
  vpc_id      = module.my_vpc.id
}
```

注意到了嗎？這份 `main.tf` 在 dev, staging, prod 裡會長得幾乎一樣。真正的差別在於旁邊的 terraform.tfvars 參數檔：

- `environments/dev/terraform.tfvars`

```
vpc_cidr      = "10.10.0.0/16"
instance_type = "t3.micro"  // 開發環境用小機器省錢
```

- `environments/prod/terraform.tfvars`

```
vpc_cidr      = "10.20.0.0/16"
instance_type = "m5.large"  // 生產環境用大機器扛流量
```

當我們要部署 Dev 環境時，就進入 `environments/dev`資料夾執行 `terraform apply`。Terraform 會自動讀取 `dev` 的參數，套用到共用的模組，蓋出一個符合開發規格的環境，部署 `Prod` 也同理。

這樣就完美達成了我們 IaC 的目標：

1. **一致性 (Consistency)** ： 所有環境的「架構」都來自同一份藍圖（模組）。
2. **可追蹤性 (Traceability)** ： 所有的基礎設施變更，都必須透過修改程式碼來完成，每一次的修改都記錄在版本控制系統（如 Git）裡，可以審查、可以追溯。
3. **可重現性 (Reproducibility)**： 即使整個環境被摧毀，我們也能用同一份程式碼，快速地重新蓋出一個一模一樣的環境。

## 2. Docker 容器化策略

我們已經用 IaC 畫好了三間廚房的建築藍圖，現在，我們要來標準化我們的「做菜工具」了。這就是 Docker 容器化策略要解決的問題。

我們一定聽過，甚至親身經歷過這個經典場景：

> 開發者 A： 「這個功能在我電腦上跑是好的啊！怎麼上線就壞了？」

這個「在我電腦上是好的 (It works on my machine)」的問題，就是 Docker 要根治的頑疾。問題的根源在於，開發者的電腦、測試伺服器、生產伺服器，這三者的「環境」存在著細微但致命的差異：

- 作業系統版本不同 (Ubuntu 20.04 vs 22.04)
- 依賴函式庫版本不同 (Python 3.8 vs 3.9, OpenSSL v1 vs v3)
- 環境變數設定不同
- 硬體參數不同
- 系統路徑或檔案權限不同

甚至全部都相同但就是有一台測試通過、一台不通過，OS 相同 windows(沒有偏見，但我的這是最有可能出現 **`底層差異`** 的地方)，我們只能放乖乖、聖水或是讚美歐姆彌賽亞來乞求機魂大悅順利執行。

**`Docker`** 的核心思想就是將應用程式連同它的「整個執行環境」打包帶走。

與其給廚師一份「法式紅酒燉牛肉」的食譜，讓他用自己廚房裡的鍋子、爐具、調味料去做（這很可能因為每間廚房的差異而味道走樣），我們不如直接把 做好的半成品、搭配特製的醬汁、以及一個精準控溫的「標準化加熱盒」 一起交給他，唯一要做的，就是按下按鈕。

這個「標準化加熱盒」，就是 **Docker 容器 (Container)** ，這個盒子裡裝著：

1. 我們的應用程式碼 (The dish itself)
2. 執行時期 (Runtime)，例如 Node.js v18 或 Java 17 (The specific cooking sauce)
3. 所有系統層級的函式庫與工具 (The special salt and pepper)

無論這個「盒子」被拿到 `Dev` 環境的微波爐，還是 `Prod` 環境的頂級烤箱（只要它們都是合格的 Docker Host），加熱後拿出來的菜，味道都保證一模一樣。這就是容器化帶來的 `可攜性 (Portability)` 與 `一致性 (Consistency)` 。

### 2.1 多階段建構最佳化

我們把 Dockerfile 的建構過程分成兩個階段，就像一個流水線：

- 階段一 (Builder Stage): 一個寬敞、工具齊全的「加工廠」。
- 階段二 (Final Stage): 一個乾淨、輕便的「配送盒」。

```dockerfile
# Dockerfile.multi-stage
# Build stage
FROM node:18-alpine AS builder
WORKDIR /app
# 只複製建構所需的 package.json
COPY package*.json ./
# 只安裝生產環境必要的依賴
RUN npm ci --only=production

# Development stage
FROM builder AS development
RUN npm ci
COPY . .
EXPOSE 3000
CMD ["npm", "run", "dev"]

# Production stage
FROM node:18-alpine AS production
WORKDIR /app
COPY --from=builder /app/node_modules ./node_modules
COPY . .
EXPOSE 3000
USER node
CMD ["npm", "start"]
```

這樣做的好處是顯而易見的：

- 最終映像檔極小： 它只包含了 node_modules (production), dist 和 package.json。沒有原始碼，沒有開發工具。
- 安全性更高： 攻擊面被縮減到最小。套件被攻擊或是內嵌木馬的消息雖然不常，但還是有所耳聞。編譯工具 (如 gcc, npm) 和開發函式庫，這些都可能成為駭客攻擊的漏洞。所以生產環境應該只包含 `「最少必要」` 的執行元件。
- 建構流程清晰： 將「如何建構」和「如何執行」的關注點分離。

### 2.2 環境特定配置管理

一個盒子，如何適應不同廚房？

我們已經有了一個標準化的「加熱盒」，但現在有個新問題：

> 在 Dev 環境，這道菜需要連到 dev-database；在 Prod 環境，它需要連到 prod-database。
>
> 但我們的盒子（Docker Image）只有一個，要怎麼辦？

```yaml
# docker-compose.dev.yml
version: '3.8'
services:
  app:
    build:
      context: .
      target: development
    environment:
      - NODE_ENV=development
      - LOG_LEVEL=debug
    volumes:
      - .:/app
      - /app/node_modules
    ports:
      - "3000:3000"

# docker-compose.prod.yml
version: '3.8'
services:
  app:
    build:
      context: .
      target: production
    environment:
      - NODE_ENV=production
      - LOG_LEVEL=error
    restart: unless-stopped
    ports:
      - "80:3000"
```

首先我們要記住，我們必須遵守一個鐵律、一個絕對法則

> #### **一次建構，到處執行 (Build once, run anywhere)**

我們不能把密封好的便當盒因為不同環境的需求，就直接拆開，這樣妥妥的違反了 **`映像檔不變性 (Image Immutability)`** ，我們絕對不能為不同環境建構不同的 `Image`（例如 `my-app:dev` , `my-app:prod` ），這會摧毀我們前面建立的所有 **`一致性保證`** 。

```
從 Dev 測試通過的那個 Image，必須是原封不動的同一個 Image 被部署到 Staging 和 Prod。
```

但假如我們真的有 `外部注入` 的需求呢? 我們該怎麼辦?

所以我們的「標準加熱盒」上，預留了幾個插槽，像是「資料庫連線插槽」、「API 金鑰插槽」。盒子本身不帶這些東西，而是等它被放到特定廚房的指定位置時，由廚房的「自動化手臂」（容器編排系統）把對應的插頭插上去。

又或者就像是一些日本泡麵一樣，熱水孔與倒水孔是不同的孔位，分別對應各自不同的用途。我們可以用熱水、熱茶(也不錯，我試過)甚至是熱奶茶(???，看過但尊重)倒入跟排出，但不變的是裡面的主體(麵體跟調料)沒有被變動。

所以回頭看一下剛剛的 yaml 示意，我們發現有一些端倪

```yaml
# docker-compose.dev.yml
version: '3.8'
services:
  app:
    build:
      context: .
      target: development
    environment:
      - NODE_ENV=development
      - LOG_LEVEL=debug
    volumes:
      - .:/app
      - /app/node_modules
    ports:
      - "3000:3000"

# docker-compose.prod.yml
version: '3.8'
services:
  app:
    build:
      context: .
      target: production
    environment:
      - NODE_ENV=production
      - LOG_LEVEL=error
    restart: unless-stopped
    ports:
      - "80:3000"
```

```yaml
# docker-compose.dev.yml
    environment:
      - NODE_ENV=development
      - LOG_LEVEL=debug

# docker-compose.prod.yml
    environment:
      - NODE_ENV=production
      - LOG_LEVEL=error
```

我們可以透過宣告 **`環境變數`** 來設定不同的接口，這是最標準、最符合十二要素應用程式 (12-Factor App) 理念的方法 - **應用程式碼不應該寫死任何配置，而是從環境變數中讀取**。

容器啟動時 (由 ECS/Kubernetes 負責)時，我們會如下執行:

- 在 Dev 環境啟動： `docker run -e DATABASE_HOST=dev.db.internal ... my-app`
- 在 Prod 環境啟動： `docker run -e DATABASE_HOST=prod.db.internal ... my-app`

而對於更複雜的配置（例如一整個 `settings.json` ），可以由容器編排系統將對應環境的設定檔，在容器啟動時「掛載」到容器內的特定路徑（例如 `/etc/config/settings.json`），我們的應用程式只需要固定去讀取這個路徑的檔案即可。

最後有一個更進階、更安全的方式。**`啟動時從配置中心拉取 (Fetching from Config Service)`** 容器啟動時，應用程式會利用被賦予的身份（例如 AWS ECS Task Role），去呼叫一個外部服務（如 AWS Secrets Manager 或 Parameter Store）來取得它需要的資料庫密碼、API Key 等。這樣連環境變數都不會明文出現，安全性最高。

## 3. Amazon ECS 叢集管理

我們已經蓋好了標準化的廚房 (IaC)，也設計好了標準化的「半成品料理包」 (Docker Image)。現在，我們需要一個智能、高效的「總廚系統」來管理整個後場的運作。這個系統需要知道：

- 一道菜（我們的應用程式）的標準烹飪流程是什麼？
- 尖峰時段（高流量）要同時開幾個爐子來做這道菜？
- 如果某個爐子壞了（伺服器故障），要如何自動換一個新的爐子繼續烹飪？

這個「總廚系統」，就是 **`Amazon ECS (Elastic Container Service)`**。

### 3.1 ECS 服務定義

要理解 ECS，我們要先認識它的 4 個核心組件: `ECS Cluster (叢集) - 「廚房本身」` 、 `Task Definition (任務定義) - 「標準作業程序 (SOP) 卡」` 、 `Task (任務) - 「大火快炒的爐位」` 、 `Service (服務) - 「不知疲倦的總廚」`。

1. ECS Cluster (叢集) - 「廚房本身」

- 核心概念：一個邏輯上的分組，用來容納我們的所有容器化應用。我們可以把它想像成 `Dev Kitchen Worktop`, `Staging Kitchen Worktop`, `Prod Kitchen Worktop`，就像是跟其他人說我的廚房工作區域有幾坪一樣，尚未涉及到實際裡面的布置，它本身不花錢，只是一個管理的範圍邊界。
- 重點： 叢集需要有「算力」來運作，就像廚房需要有爐子。算力來源有兩種模式：

  - **EC2 模式**： 我們自己買來一批「爐子」（EC2 雲端伺服器），把它們註冊到我們的廚房裡。我們對爐子的品牌、型號有完全的控制權可以自由調校，但也要自己負責維護和管理。
  - **Fargate 模式 (Serverless - 業界主流趨勢)**： 我們不買爐子，要做菜時，直接跟 AWS 說：「我需要一個火力 500W、佔地 1 平方公尺的爐位」，AWS 會自動從他們龐大的資源池裡，變出一個符合我們要求的爐位給我們用，用完即焚。
  - 對於絕大多數新的應用，優先選擇 Fargate。它讓我們從繁瑣的伺服器管理中解放出來，更專注於我們的應用程式本身。

2. Task Definition (任務定義) - 「標準作業程序 (SOP) 卡」

- 概念： 這是 ECS 最核心的藍圖。它是一份 JSON 格式的說明書，詳細定義了「如何烹飪一道菜」。

- 重點：就像是一張給廚師看的傻瓜式 SOP 卡，上面精確地寫著：
  - image: 要用哪個版本的「半成品料理包」 (Docker Image URL)。
  - cpu & memory: 需要分配多少「火力」和「空間」給這個任務。
  - environment: 需要注入哪些「調味料」 (環境變數，例如 DATABASE_URL)。
  - secrets: 需要從「保險櫃」(AWS Secrets Manager) 裡取用哪些「秘方」 (API Key, 資料庫密碼)。
  - logConfiguration: 烹飪過程的紀錄（日誌）要送到哪裡 (例如送到 CloudWatch Logs)。
  - taskRoleArn: 給予這位廚師什麼樣的「權限卡」(IAM Role)，讓它可以去跟其他 AWS 服務互動（例如讀取 S3 上的檔案）。

3. Task (任務) - 「大火快炒的爐位」

- 概念： 任務定義的「執行中實例 (Running Instance)」。當 ECS 根據我們的 SOP 卡實際啟動一個容器後，那個正在運行的容器就是一個 Task， 我們可以透過設定多少個 Task 來決定同時間有多少灶口在大鍋炒

4. Service (服務) - 「不知疲倦的總廚」

- 概念： Service 是 ECS 的大腦和管家。它的職責是維持我們想要的狀態。
- 重點：確保廚房的一切都按照我們所想要的執行秩序運行下去
  - `desiredCount: { N }` ：「我希望永遠有 `{ N }` 份『法式紅酒燉牛肉』在火上烹飪，不多也不少。」
  - 透過 `Health Check` ，總廚會不斷巡視所有爐子是不是正常的。如果他發現其中一份因為爐子故障了(`HC fail`)，他會立刻把這份丟掉，並馬上開一個新的爐子上，確保開火中的爐口的數量永遠是 3。
  - 負載平衡 (Load Balancing): 當有爐點燃時（新的 Task 啟動），總廚會自動通知前台的服務生（Application Load Balancer），告訴他現在多了一個出餐口，可以把客人的訂單（流量）也送過來。

**實作範例**

```json
{
  "family": "app-service",
  "networkMode": "awsvpc",
  "requiresCompatibilities": ["FARGATE"],
  "cpu": "256",
  "memory": "512",
  "executionRoleArn": "arn:aws:iam::account:role/ecsTaskExecutionRole",
  "taskRoleArn": "arn:aws:iam::account:role/ecsTaskRole",
  "containerDefinitions": [
    {
      "name": "app",
      "image": "your-registry/app:${ENVIRONMENT}",
      "portMappings": [
        {
          "containerPort": 3000,
          "protocol": "tcp"
        }
      ],
      "environment": [
        {
          "name": "NODE_ENV",
          "value": "${ENVIRONMENT}"
        }
      ],
      "secrets": [
        {
          "name": "DATABASE_URL",
          "valueFrom": "arn:aws:secretsmanager:region:account:secret:${ENVIRONMENT}/database-url"
        }
      ],
      "logConfiguration": {
        "logDriver": "awslogs",
        "options": {
          "awslogs-group": "/ecs/${ENVIRONMENT}/app",
          "awslogs-region": "us-west-2",
          "awslogs-stream-prefix": "ecs"
        }
      }
    }
  ]
}
```

#### ECS 多環境下的 ECS 實踐

簡單說明了 ECS 的意涵之後，那我們的 `Dev Kitchen Worktop`, `Staging Kitchen Worktop`, `Prod Kitchen Worktop` 策略在 ECS 層級如何體現？

- **叢集隔離**： 遵循多帳號策略，我們會在 `Account-Dev` 裡建立 `dev-cluster`，在 `Account-Prod` 裡建立 `prod-cluster`。這是最徹底的隔離。
- **SOP 卡 (Task Definition) 的管理**： 我們依然使用 IaC (Terraform) 來管理任務定義。我們會有一份共用的 `task-definition.tf` 模板，然後為不同環境傳入不同的參數。

  - `dev.tfvars`: `cpu = 256 (1/4 vCPU)`, `memory = 512 (MB)`, `API_KEY_SECRET_ARN = "arn:aws:secretsmanager:ap-northeast-1:DEV_ACCOUNT_ID..."`
  - `prod.tfvars`: `cpu = 1024 (1 vCPU)`, `memory = 2048 (MB)`, `API_KEY_SECRET_ARN = "arn:aws:secretsmanager:ap-northeast-1:PROD_ACCOUNT_ID..."`

  - 注意囉，我們 `image` 的參數會指向 **同一個 Docker Image** ，但注入的 `CPU` 、 `記憶體` 和 `Secrets` 依舊來自不同的環境配置，維持了我們的 **`一致性`** 與 **`不變性`** 。

- **總廚指令 (Service) 的管理**： 同樣用 IaC 定義。

  - `dev` 環境的 Service，desiredCount 可能只是 `1` ，而且可能不設定自動擴縮，以節省成本。
  - `prod` 環境的 Service，desiredCount 可能是 `3` 起跳，並且會配置接下來要講的自動擴縮策略。

### 3.2 ECS 自動擴縮配置

我們的餐廳不能只靠固定 3 位廚師。有時候是有規律的，像是週二下午可能一位就夠了，但週五晚上可能需要 10 位才能應付源源不絕的訂單這種固定的規律模型，有時又會突然人潮爆量到需要 20 位廚師來 handle。

這時候手動去增減廚師數量太慢且不切實際，所以我們需要設置一個 **`「客流-爐口監控連動系統」( ECS Service Auto Scaling )`** ，讓我們能根據「客流量」自動增減人手從而最佳化我們的成本考量。

- 核心思想： 監控一個指標，並根據這個指標的變化，自動調整 Service 的 desiredCount。
- 最常用的指標 (Metrics)：
  - CPU 利用率 (CPU Utilization)： 「如果廚師們的平均忙碌程度超過 70%，就增加人手。」這是最常見的指標。
  - 記憶體利用率 (Memory Utilization)： 「如果廚房的平均空間佔用超過 75%，就擴大廚房（增加 Task）。」
  - ALB 請求數 (Request Count Per Target)： 這是最能反映真實用戶壓力的指標。「如果平均每個出餐口每分鐘要處理的訂單超過 500 張，就立刻增加出餐口。」
- 設定閾值： 我希望這個`ALBRequestCountPerTarget`指標維持在 500 左右。
- 設定邊界：
  - min_capacity = 2：無論如何，廚房裡至少要有 2 位廚師待命。
  - max_capacity = 20：最多就加到 20 位，再多可能我的成本就爆了，或者後端資料庫也撐不住了。

運作流程大致如下:

- **擴展 (Scale Out)**: 當週五晚上 7 點流量湧入，平均請求數飆升到 800。Auto Scaling 偵測到後，立刻自動提高 Service 的 `desiredCount` (例如從 2->3, 3->5...)，ECS 就會啟動新的 Task。新的 Task 自動註冊到 ALB，分攤流量，直到平均請求數降回 500 左右。
- **縮減 (Scale In)**: 到了晚上 10 點，客人陸續離開，平均請求數掉到 100。Auto Scaling 發現後，會逐步減少 `desiredCount`，ECS 就會優雅地終止掉多餘的 Task，將資源釋放，為我們節省成本。
- **排程(Period)**: 假如我們已經知道每週五晚上 7 點有 **固定流量湧入** 且 在 **晚上 11 點** 客人散的差不多了的話，那我們可以固定化的設置在 `19:00-23:00` 這個區建置固定的 Task `{ N }`數來面對流量，這又能更進一步地減少成本花費 - 監聽也是要錢的。

```yaml
# ecs-auto-scaling.yml
Resources:
  ServiceScalingTarget:
    Type: AWS::ApplicationAutoScaling::ScalableTarget
    Properties:
      MaxCapacity: !Ref MaxCapacity
      MinCapacity: !Ref MinCapacity
      ResourceId: !Sub "service/${ClusterName}/${ServiceName}"
      RoleARN: !Sub "arn:aws:iam::${AWS::AccountId}:role/aws-service-role/ecs.application-autoscaling.amazonaws.com/AWSServiceRoleForApplicationAutoScaling_ECSService"
      ScalableDimension: ecs:service:DesiredCount
      ServiceNamespace: ecs

  ServiceScalingPolicy:
    Type: AWS::ApplicationAutoScaling::ScalingPolicy
    Properties:
      PolicyName: !Sub "${ServiceName}-scaling-policy"
      PolicyType: TargetTrackingScaling
      ScalingTargetId: !Ref ServiceScalingTarget
      TargetTrackingScalingPolicyConfiguration:
        PredefinedMetricSpecification:
          PredefinedMetricType: ECSServiceAverageCPUUtilization
        TargetValue: 70.0
```

## 4. Amazon EKS 與 Kubernetes 管理

人力有時而窮，味蕾也會出現疲乏。有時候要吸引更多的人而開更多的 **米其林餐廳** 不一定是更好的選擇。或許是口味考量，或許是預算的考量，當我們的服務進行業務邏輯的擴張時，一直在同一家餐廳裡不斷增長新的菜色會讓管理成本與複雜度逐漸增加，同時，也有可能開了一新店後，大家都指點某一個菜品(例如米其林主廚做的起司漢堡與吃完後自動爆炸的餐廳)，其他服務根本沒用上 - 這不是廠商的疏失，這無異是成本上的浪費。

如果我們接下來的目標是開一個根據不同客群擁有數十個不同品牌（微服務）、未來還可能把分店開到 Google 或 Microsoft 商場（多雲）的餐飲集團，那麼我們就需要一個業界通用的、標準化的「集團營運系統」。這個系統，就是 Kubernetes (常簡稱為 K8s)。

Amazon EKS (Elastic Kubernetes Service)，就是 AWS 提供的、幫我們把這個複雜營運系統中最核心、最燒腦的「集團總部管理室」（控制平面）給託管起來的服務。=

### 4.1 EKS 叢集配置 — 搭建我們的美食廣場

如果說 ECS Cluster 是一間獨立餐廳的後廚，那麼 EKS Cluster 就是整個美食廣場的場地和基礎設施，其中最重要的就是 **「美食廣場管理辦公室」(控制平面 - Control Plane)** 與 **「美食攤位的鋪位」(數據平面 - Data Plane / Worker Nodes)**。

- 控制平面 (Control Plane) - 「美食廣場管理辦公室」

  - 概念： 這是 Kubernetes 的大腦。它負責接收指令、調度所有攤位（應用程式）的開設位置、監控所有攤位的營運狀況、儲存整個廣場的佈局圖等。這部分極度複雜且至關重要，必須保證 7x24 小時高可用。
  - EKS 的價值： 我們不用自己去蓋這個管理辦公室，也不用去聘請保全、IT 人員來維護它。AWS EKS 會提供一個高可用、安全、自動化升級的管理辦公室給我們。這是我們付錢給 EKS 最主要的原因。我們只需要專心經營我們的美食攤位。

- 數據平面 (Data Plane) / Worker Nodes - 「美食攤位的鋪位」

  - 概念： 這是實際「烹飪」的地方，也就是我們的應用程式容器實際運行的伺服器。
  - 在 EKS 中，主要有兩種形式： - EC2 Managed Node Groups： 和 ECS 的 EC2 模式類似。我們向廣場管理處租下一排「標準鋪位」(EC2 instances)。我們可以選擇鋪位的規格（例如 `m5.large` ），但鋪位的電力、網路、消防等都由 AWS 協助管理，例如自動化的節點升級和替換。這是最常見的模式。 - Fargate Profiles： 和 ECS Fargate 模式一樣。我們連鋪位都不用租。我們直接跟管理處說：「我要開一個臨時的章魚燒攤位，需要 1vCPU 和 2GB 記憶體的空間。」EKS 會自動在廣場的某個角落給我們變出一個符合要求的空間，用完就收走。非常適合無狀態應用或需要應對突發流量的場景。

多環境策略： 原則不變。我們會在 `Account-Dev` 裡建立一個 `dev-eks-cluster` ，在 `Account-Prod` 裡建立 `prod-eks-cluster`。IaC (Terraform) 依然是我們用來標準化定義這些叢集配置的工具。

```yaml
# eks-cluster.yaml
apiVersion: eksctl.io/v1alpha5
kind: ClusterConfig

metadata:
  name: app-cluster-${ENVIRONMENT}
  region: us-west-2
  version: "1.28"

nodeGroups:
  - name: worker-nodes
    instanceType: ${INSTANCE_TYPE}
    minSize: ${MIN_SIZE}
    maxSize: ${MAX_SIZE}
    desiredCapacity: ${DESIRED_SIZE}
    privateNetworking: true
    labels:
      environment: ${ENVIRONMENT}
    tags:
      Environment: ${ENVIRONMENT}
      Project: "multi-env-demo"

addons:
  - name: vpc-cni
  - name: coredns
  - name: kube-proxy
  - name: aws-load-balancer-controller
```

### 4.2 Kubernetes 應用部署

Kubernetes 的世界裡，描述「如何經營一個攤位」有一套標準的詞彙和文件（YAML 檔）。這比 ECS 的 Task Definition 更為細緻和強大。

1. Pod - 「一個獨立的烹飪工作站」

- 概念： Kubernetes 中最小的部署單元。它可以包含一個或多個緊密關聯的容器。
- 比喻： Pod 不僅僅是那口鍋 (Container)，它還包括了鍋子旁邊的一小塊工作台、獨立的水槽和電源插座 (獨立的網路 IP 位址)，以及一個專用的小冰箱 (儲存空間 Volume)。Pod 內的容器可以共享這個小環境。
- 關鍵特性： Pod 是有生命週期的 (Ephemeral)，它們隨時可能被銷毀和重建。我們絕不能直接依賴某一個 Pod 的存在。

2. Deployment - 「連鎖品牌加盟手冊」

- 概念： Deployment 是一個更高層級的控制器，它的核心職責是確保指定數量的、一模一樣的 Pod 副本正在運行。
- 比喻： 我們是「A+ 炸雞排」的品牌方。我們的 Deployment YAML 就是我們的加盟手冊，上面寫著：
  - replicas: 3：我要求在美食廣場裡，必須時刻保持 3 個「A+ 炸雞排」的烹飪工作站 (Pod) 在運作。
  - template: 手冊裡詳細描述了標準工作站的規格（用哪個 Docker Image、需要多少 CPU/Memory 等）。
  - 自我修復： 如果 Deployment 發現只有 2 個工作站在運作，它會立刻自動建立第 3 個。
  - 滾動更新 (Rolling Update)： 當我們要推出新口味（更新 Docker Image）時，Deployment 會非常聰明地：先開一個新的工作站 -> 等新站準備就緒 -> 才關掉一個舊的站。這樣逐步替換，確保我們的攤位服務不中斷。

3. Service - 「攤位的叫號器與內部專線」

- 概念： Pods 是會不斷生滅的，它們的 IP 位址也一直在變。Service 提供了一個穩定不變的內部網路端點，來代表一組功能相同的 Pods。
- 比喻： 我們的 3 個炸雞排工作站 (Pods) 可能在廣場的不同位置，而且還可能隨時換位。客人不可能追著跑。於是，我們在美食廣場的服務台註冊了一個固定的攤位號碼「B-52」 (Service)。
- 運作方式：當美食廣場裡的其他攤位（例如「好喝飲料店」Pod）想訂購炸雞排時，它不需要知道任何一個炸雞排 Pod 的實際位置，它只需要呼叫「B-52」這個內部專線，Kubernetes 的網路系統就會自動把請求轉接到一個當前健康的炸雞排 Pod，這很好地確保了服務之間通訊的穩定性，是微服務架構的基石。

```yaml
# k8s/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app-deployment
  namespace: ${ENVIRONMENT}
  labels:
    app: demo-app
    environment: ${ENVIRONMENT}
spec:
  replicas: ${REPLICA_COUNT}
  selector:
    matchLabels:
      app: demo-app
  template:
    metadata:
      labels:
        app: demo-app
        environment: ${ENVIRONMENT}
    spec:
      containers:
        - name: app
          image: your-registry/app:${IMAGE_TAG}
          ports:
            - containerPort: 3000
          env:
            - name: NODE_ENV
              value: ${ENVIRONMENT}
            - name: DATABASE_URL
              valueFrom:
                secretKeyRef:
                  name: database-secret
                  key: url
          resources:
            requests:
              memory: "128Mi"
              cpu: "100m"
            limits:
              memory: "256Mi"
              cpu: "200m"
          livenessProbe:
            httpGet:
              path: /health
              port: 3000
            initialDelaySeconds: 30
            periodSeconds: 10
          readinessProbe:
            httpGet:
              path: /ready
              port: 3000
            initialDelaySeconds: 5
            periodSeconds: 5
---
apiVersion: v1
kind: Service
metadata:
  name: app-service
  namespace: ${ENVIRONMENT}
spec:
  selector:
    app: demo-app
  ports:
    - port: 80
      targetPort: 3000
  type: ClusterIP
```

### 4.3 Ingress 與 Load Balancer 配置

現在，攤位都開好了，內部員工也可以互相點餐了。但最重要的問題：外面的顧客要怎麼走進來，並找到我們的攤位？

一個簡單粗暴的方法是給每個攤位都開一個直通外面的大門 (Service type LoadBalancer)。這會為每個服務都建立一個獨立的 AWS 負載平衡器，在有數十個服務時，成本會非常驚人。

更優雅、更經濟的做法是使用 **`Ingress`**。 `Ingress` 是管理外部流量進入叢集的 API 物件，它提供基於 HTTP/HTTPS 的路由規則，這就像是整個美食廣場只有一個總入口，而在入口處有一塊巨大的電子導覽看板，上面寫著：

- 想去「A+ 炸雞排」( `host: food.com, path: /chicken` ) -> 請走 B 區 52 號 (導向 `chicken-service` )
- 想去「C- 披薩屋」( `host: food.com, path: /pizza` ) -> 請走 C 區 03 號 (導向 `pizza-service` )

構成我們的大門( Ingress )體系的兩個關鍵組件：

1. **Ingress Controller - 「入口的警衛與導覽員」** : 它是一個實際在叢集中運行的程式 (本身也是 Pod)。它負責監聽我們定義的 Ingress 規則，並實際去設定那個位於總入口的 AWS Application Load Balancer (ALB)。在 EKS 環境中，最常用的就是 AWS Load Balancer Controller。
2. **Ingress Resource - 「我們寫的那份導覽圖規則」** : 這是一個 YAML 檔案，我們向 Kubernetes 提交這個檔案，告訴 Ingress Controller 我們想要的路由規則。

Ingress 的巨大優勢就是我們可以用一個 ALB，來作為數十甚至數百個後端服務的流量入口，透過不同的域名或 URL 路徑，將流量精準分發給對應的 Service。這極大地節省了成本，並簡化了外部流量的管理，基德再也不會找上門了

```yaml
# k8s/ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: app-ingress
  namespace: ${ENVIRONMENT}
  annotations:
    kubernetes.io/ingress.class: "alb"
    alb.ingress.kubernetes.io/scheme: internet-facing
    alb.ingress.kubernetes.io/target-type: ip
    alb.ingress.kubernetes.io/certificate-arn: ${CERTIFICATE_ARN}
    alb.ingress.kubernetes.io/ssl-redirect: "443"
spec:
  rules:
    - host: ${ENVIRONMENT}.example.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: app-service
                port:
                  number: 80
```

### ECS vs. EKS

我們今天(?)從宏觀的叢集搭建，到微觀的應用部署，再到流量的入口管理，完整地走了一遍 Kubernetes 的核心路徑。這套體系雖然複雜，但它正是支撐起當今大規模微服務應用的骨架。

最後的最後我們來簡單比較一下兩者的差別:

> ### ECS 是條理分明的精緻餐廳：
>
> AWS 幫我們定義好了很多規則，我們只需要專注於菜色（我們的應用），整合度高，學習曲線平緩。

> ### EKS/Kubernetes 是生機勃勃的國際美食廣場：
>
> 它提供了一套功能極其強大、靈活、且廠商中立的「營運標準」。我們需要學習它的一整套術語和概念（Pod, Deployment, Service, Ingress...），但回報是無與倫比的彈性、擴展性和活躍的生態圈。

## 5. CI/CD 管道整合

有鑑於我們在 <CI/CD 全自動化實作 - GitHub Actions × CodePipeline × CodeBuild> 中已經說過了關於 CI/CD 的整合與流程概念，我們接下來簡單的說一下流程並且整理出碎片化的 Jobs，然後進入到結合<Infrastructure as Code : Terraform 基礎設施代碼化與版本管控> 、<CI/CD 全自動化實作 - GitHub Actions × CodePipeline × CodeBuild> 、的多環境應用 **`藍綠部屬`** 環節

### 5.1 GitHub Actions 工作流程

重新複習一下，CI/CD 流程最重要的理念就是 `建立一個穩定且固定的商業邏輯驗證交付流程，` **`積極性的保護既有商業邏輯不被程式碼異動所破壞汙染`** `

每當有程式碼被推送（Push）到程式碼倉庫（例如 GitHub），流水線就自動啟動、它會抓取最新的程式碼並執行編譯、執行單元測試、程式碼品質掃描等一系列「品質檢查」。這就像是食品工廠的「品管環節」，每當有新的原料送達，品管員會立刻抽樣檢驗，確保這批原料沒有問題，不會污染整個生產線 - CI 的目標是 **儘早發現問題** 。當 CI 品管環節全部通過後，流水線會繼續下一步：打包（建立 Docker Image）、推送至倉庫（ECR）、然後自動觸發對 Dev -> Staging -> Prod 環境的部署，CD 的目標是讓發布成為一個 **日常** 、 **穩定** 且 **可追蹤**的事件，就像品管合格的原料，會被自動送上生產線，加工、烹飪、真空包裝，最後由輸送帶直接送到貨車上，配送至各大門市。

```yaml
# .github/workflows/deploy.yml
name: Multi-Environment Deployment

on:
  push:
    branches: [release, develop]
  pull_request:
    branches: [release]

env:
  AWS_REGION: us-west-2
  ECR_REPOSITORY: demo-app

jobs:
  build:
    runs-on: ubuntu-latest
    outputs:
      image-tag: ${{ steps.build.outputs.image-tag }}
    steps:
      - uses: actions/checkout@v3

      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v2
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: ${{ env.AWS_REGION }}

      - name: Login to Amazon ECR
        id: login-ecr
        uses: aws-actions/amazon-ecr-login@v1

      - name: Build and push Docker image
        id: build
        run: |
          IMAGE_TAG=${GITHUB_SHA:0:8}
          docker build -t $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG .
          docker push $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG
          echo "image-tag=$IMAGE_TAG" >> $GITHUB_OUTPUT

  deploy-dev:
    if: github.ref == 'refs/heads/develop'
    needs: build
    runs-on: ubuntu-latest
    environment: development
    steps:
      - name: Deploy to Development
        run: |
          # ECS deployment
          aws ecs update-service \
            --cluster dev-cluster \
            --service app-service \
            --task-definition app-service:LATEST

  deploy-staging:
    if: github.ref == 'refs/heads/main'
    needs: build
    runs-on: ubuntu-latest
    environment: staging
    steps:
      - name: Deploy to Staging
        run: |
          # Kubernetes deployment
          kubectl set image deployment/app-deployment \
            app=$ECR_REGISTRY/$ECR_REPOSITORY:${{ needs.build.outputs.image-tag }} \
            -n staging

  deploy-prod:
    if: github.ref == 'refs/heads/main'
    needs: [build, deploy-staging]
    runs-on: ubuntu-latest
    environment: production
    steps:
      - name: Deploy to Production
        run: |
          # Blue-Green deployment
          ./scripts/blue-green-deploy.sh ${{ needs.build.outputs.image-tag }}
```

而根據我們在 <CI/CD 全自動化實作 - GitHub Actions × CodePipeline × CodeBuild> 中所說的，<企業級 Jobs 模組化與跨領域引用> 當我們需要部署的時候，我們可以抽出共同流程(Jobs)將其流水線化抽出管理，並根據當前分支的行為進行自動化部屬

```yaml
# build
parameters:
  - name: ngCliVersion
    type: string
    default: latest
    values:
      - latest
      - 16
      - 12
      - 11
jobs:
  build:
    runs-on: ubuntu-latest
    outputs:
      image-tag: ${{ steps.build.outputs.image-tag }}
    steps:
      - uses: actions/checkout@v3

      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v2
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: ${{ env.AWS_REGION }}

      - name: Login to Amazon ECR
        id: login-ecr
        uses: aws-actions/amazon-ecr-login@v1

      - name: Build and push Docker image
        id: build
        run: |
          IMAGE_TAG=${GITHUB_SHA:0:8}
          docker build -t $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG .
          docker push $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG
          echo "image-tag=$IMAGE_TAG" >> $GITHUB_OUTPUT
```

```yaml
#  deploy-dev
parameters:
  - name: ngCliVersion
    type: string
    default: latest
    values:
      - latest
      - 16
      - 12
      - 11
jobs:
  deploy-dev:
    if: github.ref == 'refs/heads/develop'
    needs: build
    runs-on: ubuntu-latest
    environment: development
    steps:
      - name: Deploy to Development
        run: |
          # ECS deployment
          aws ecs update-service \
            --cluster dev-cluster \
            --service app-service \
            --task-definition app-service:LATEST
```

```yaml
#  deploy-staging
parameters:
  - name: ngCliVersion
    type: string
    default: latest
    values:
      - latest
      - 16
      - 12
      - 11

jobs:
  deploy-staging:
    if: github.ref == 'refs/heads/main'
    needs: build
    runs-on: ubuntu-latest
    environment: staging
    steps:
      - name: Deploy to Staging
        run: |
          # Kubernetes deployment
          kubectl set image deployment/app-deployment \
            app=$ECR_REGISTRY/$ECR_REPOSITORY:${{ needs.build.outputs.image-tag }} \
            -n staging
```

```yaml
#  deploy-prod
parameters:
  - name: ngCliVersion
    type: string
    default: latest
    values:
      - latest
      - 16
      - 12
      - 11

jobs:
  deploy-prod:
    if: github.ref == 'refs/heads/main'
    needs: [build, deploy-staging]
    runs-on: ubuntu-latest
    environment: production
    steps:
      - name: Deploy to Production
        run: |
          # Blue-Green deployment
          ./scripts/blue-green-deploy.sh ${{ needs.build.outputs.image-tag }}
```

抽出完的流程會像是這樣

```yaml
# .github/workflows/deploy.yml
name: Multi-Environment Deployment

on:
  push:
    branches: [release, develop]
  pull_request:
    branches: [release]

env:
  AWS_REGION: us-west-2
  ECR_REPOSITORY: demo-app

jobs:
  - template: jobs/build.yml@templates
  - template: jobs/deploy-dev.yml@templates
  - template: jobs/deploy-staging.yml@templates
    parameters:
      Version: ${{ variables.Version }}
      env: $(env)
      hasPreview: true
```

### 5.2 藍綠部署:單體實例(ECS/EC2)與叢集(EKS)

#### 單體實例(ECS/EC2)的藍綠部屬

在上述我們簡單的複習了一下 CI/CD 流程與 `Jobs` 獨立切分之後，我們來了解多環境部屬的實際應用 - **`藍綠部屬`** 。

現行系統開發中有兩大策略脈絡，其一是根據 `降流直至服務徹底降溫` 之後再部署的 **滾動更新**，而另一個則是 `零停機時間` 的發布策略 - **藍綠部署 (Blue/Green Deployment)** 。滾動更新就像是在 F1 賽車時，只有進入到維修站並處於完全靜止( `降流直至服務徹底降溫` ) 的狀態下才能允許 `快速進行維修與更新`，在維修站中要與倒數的分秒進行賽跑，一旦出現失誤都將導致衝出維修站的時間被延宕，甚至如果開上賽道才發現第二個輪胎時發現新輪胎有問題，車子已經處於不穩定的狀態，要換回來必須要再次進入到維修站才能進行調整 - 而更糟的狀況是我們要在高速行駛中的 **`賽道上緊急進行搶修`** 。

而 **`藍綠部署`** 則提供了一種更安全、更優雅的哲學，我們不在行駛中的賽車（ `藍色環境` ）上動手。我們在旁邊的車道，準備一台一模一樣的、但安裝了新輪胎的新車（ `綠色環境` ）。我們讓這台新車發動、預熱、檢查所有儀表板，當我們 100% 確認新車完美無瑕時(這當然是最理想的狀況，哈哈)，我們逐步將壓力切換到新車上。舊車則在旁邊逐步降載，萬一新車有問題，我們可以再 **`瞬間切換回來`** 。

- 執行步驟：

  1. 初始狀態： 假設我們當前的生產環境是 V1，我們稱之為藍色環境。所有用戶流量都通過 Application Load Balancer (ALB) 指向藍色環境的 Target Group。

  2. 部署綠色環境：

  - CI/CD 流水線啟動。
  - 它會建立一套全新的、獨立的基礎設施（例如新的 ECS Service 或 K8s Deployment），部署上軟體的 V2 版本，這就是綠色環境。此時，綠色環境正在運行，但沒有任何真實的用戶流量。

  3. 驗證綠色環境：

  - 這是藍綠部署最關鍵的優勢。在切換流量之前，我們可以對綠色環境進行完整的、隔離的自動化測試。例如，從流水線內部發送模擬請求到綠色環境的內部端點，驗證其核心功能是否正常、API 是否回傳正確。

  4. 流量切換 (The Switch)：

  - 當所有測試通過（通常還會加上一個「手動審批」的步驟），CI/CD 流水線會執行一個單一、原子性的操作：修改 ALB Listener 的規則，將 **自定義** (視業務需求，但 20%會是一個常用的比例) 的流量從指向藍色 Target Group，改為指向綠色 Target Group，這個切換對用戶來說是瞬時的，他們絲毫感覺不到服務的中斷。

  5. 待命與下線：

  - 切換後，綠色環境逐步接手正式成為新的生產環境。
  - 舊的藍色環境並不會馬上被銷毀，它會作為一個「熱備份」在那裡待命。如果上線後幾分鐘或幾小時，監控系統發現 V2 版本有嚴重問題，我們可以執行「一鍵回滾」- 也就是再執行一次流量切換，把流量指回藍色環境，實現秒級災難恢復。
  - 當確認新版本穩定運行一段時間後，CI/CD 流水線才會去執行最後的清理工作，銷毀藍色環境以節省成本。

AWS CodeDeploy 這個服務與 ECS 和 EC2 深度整合，可以將上述複雜的藍綠部署流程自動化，我們只需要做一些簡單的配置即可。

我們剛剛討論的這套 **`「以 IaC 管理環境狀態、以 CI/CD 管理部署生命週期、逐步驗證環境差異」`** 的分工協作模式，正是當前雲端原生（Cloud-Native）領域中，討論度最高的、公認的 **最佳化實現方案 (Optimal Implementation)** 和 **黃金標準 (Gold Standard)** 。

「最佳化」並非指它是「最簡單」或「最快速建立」的方案，而是指它在幾個關鍵的、往往相互衝突的工程目標之間，取得了最優雅的平衡。首先，它在 **「穩定性」** 與 **「敏捷性」** 之間取得平衡，在過去想讓系統穩定，就意味著要減少變更，部署週期可能是數月一次。想要快速迭代（敏捷），又常常會犧牲穩定性，導致線上頻繁出錯。而現代化雲原生作法透過了 IaC 保障了「穩定性」的基石，我們的基礎設施不再是個黑盒子，每一次變更都有紀錄、有審核 (Code Review)、可追溯。這杜絕了因「手動誤操作」或「環境不一致」導致的大部分穩定性問題 - 基礎設施變得像岩石一樣穩固。CI/CD 提供了「敏捷性」的引擎，在穩固的基石上自動化流水線讓應用程式的部署變得極快且低風險。開發者可以每天進行數十次部署，而不用擔心會搞垮底層設施，藍綠部署等策略更是為這種高速迭代提供了頂級的安全網。

其次，需求團隊（Require）和維運團隊（Maintain）之間常常存在鴻溝。需求者想快速兌換新功能，維運者想穩定既有功能，互相指責、效率低下是常態。但 `「以 IaC 管理環境狀態、以 CI/CD 管理部署生命週期、逐步驗證環境差異」` 在 **「職責清晰」** 與 **「團隊協作」** 之間取得平衡，透過清晰的界線 (Clear Boundary)我們的分工模式提供了一個極其清晰的「契約」:

- 平台/維運團隊 (Platform/Ops Team)： 他們的核心產出是穩定、可靠、標準化的基礎設施平台（由 Terraform 定義）。他們像城市規劃者，負責建好水、電、瓦斯管線和標準化的建築地基。
- 應用開發團隊 (Application/Dev Team)： 他們的核心產出是高品質的應用程式（封裝在 Docker 裡），以及安全可靠的部署流程（由 GitHub Actions Workflow 定義）。他們像是在標準地基上蓋房子的建築隊，只需要專注於房子本身，而不用擔心水電是怎麼來的。

平台團隊提供穩定的平台後，應用團隊就可以透過 CI/CD 管道進行「自助式部署」，而不需要每次都去麻煩平台團隊，極大地提升了開發自主性和效率。

最後則是在 **「可控性」** 與 **「複雜性」** 之間取得平衡，現代雲端系統的確非常複雜，動輒數百個資源和設定。雖然初學有門檻，但 IaC 是管理這種複雜度的唯一有效方法，它將龐雜的雲端資源，轉化為人類可讀、機器可執行的結構化文本。我們可以審視、測試、模組化我們的基礎設施，就像對待任何軟體一樣。同時 CI/CD 隱藏了複雜性，對於大多數開發者來說，他們不需要了解藍綠部署背後複雜的 ALB 規則切換細節。他們只需要知道「當我把程式碼合併到 main 分支，並在 Pipeline 點下『批准』按鈕後，我的新功能就會安全上線」- CI/CD 管道將複雜的部署過程，封裝成一個簡單、可靠的自動化流程。

#### 叢集(EKS)的藍綠部屬

現在我們知道了藍綠部署的「哲學」與在 ECS 和 EC2 層級上的應用，現在就來看看 Kubernetes 這位「美食廣場總管」是如何用它手裡的工具，來具體執行這個精密的流量切換操作的。

與 ECS 不同，Kubernetes 本身沒有一個叫做 blue-green-deployment 的現成資源。但是，它提供了一系列更靈活、更強大的基礎元件（我們稱之為 primitives），讓我們可以像組合樂高一樣，搭建出比 AWS CodeDeploy 更靈活的藍綠部署流程，我們將重點介紹業界最主流、最能體現 Kubernetes 設計哲學的兩種實現方式：**`Service 層切換`** 和 **`Ingress 層切換`** 。

複習一下，Kubernetes 裡的一個關鍵思想： **`Deployment/Pods (應用實例)`** 是實際在運行的「攤位」，它們是 **會變動**、 **會生滅** 的。而 **`Service/Ingress (服務入口)`** 是提供給內外部顧客的「固定攤位號碼」或「美食廣場入口」，它們應該是 **穩定不變的** 。K8s 藍綠部署的精髓，就在於保持「服務入口」不變，只改變它背後指向的「應用實例」。

##### Service Selector 切換 (最經典的 K8s 做法)

這是最能體現 Kubernetes 核心概念的方法，透過修改 Service 的 selector（標籤選擇器）來瞬間切換流量。依照我們之前在討論 EKS 時使用的案例 - 美食廣場，美食廣場的「B-52 叫號器」(Service) 本身不動，我們只是在它的後台系統裡，把服務人員從「藍色制服團隊」瞬間切換到「綠色制服團隊」。

示意 Steps:

1. 初始狀態 (藍色環境運行中):
   - 我們有一個 `Deployment-Blue` ，它掌管著 `v1` 版本的 Pods。這些 Pods 身上都貼著標籤 `version: blue`。
   ```yaml
   # deployment-blue.yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
   name: myapp-blue
   spec:
   replicas: 3
   template:
     metadata:
       labels:
         app: myapp
         version: blue # 關鍵標籤
     spec:
       containers:
         - name: myapp
           image: my-app:v1
   ```
   - 我們有一個穩定的 `Service`，它的 `selector` 只會尋找身上貼有 `version: blue` 標籤的 `Pods`，並將流量轉發給它們。
   ```yaml
   # service.yaml
   apiVersion: v1
   kind: Service
   metadata:
   name: myapp-service # 這是穩定不變的入口
   spec:
   selector:
     app: myapp
     version: blue # 目前指向藍色
   ports:
   - protocol: TCP
    port: 80
    targetPort: 8080
   ```
   此時，所有流量都流向 `v1` 的 `Pods`。
2. 部署綠色環境:
   - CI/CD 管道會建立一個全新的 `Deployment-Green`，它掌管著 `v2` 版本的 `Pods`，這些 `Pods` 的標籤是 `version: green`。
   ```yaml
   # deployment-green.yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
   name: myapp-green
   spec:
   replicas: 3
   template:
     metadata:
       labels:
         app: myapp
         version: green # 關鍵標籤
     spec:
       containers:
         - name: myapp
           image: my-app:v2
   ```
   - 部署完成後，v2 的 Pods 已經在叢集中健康運行，但因為 myapp-service 的 selector 仍然是 version: blue，所以綠色環境完全沒有接收到任何正式流量。
3. 驗證綠色環境
   - 在此時，我們可以建立一個僅供內部測試用的 Service，例如 myapp-green-test-service，它的 selector 指向 version: green。CI/CD 管道可以透過這個內部 Service 的位址，對綠色環境進行完整的自動化測試。
4. 執行流量切換 (The Switch)
   - 當一切準備就緒，CI/CD 管道會執行一個單一、原子性的指令：
   ```bash
   kubectl patch service myapp-service -p '{"spec":{"selector":{"version":"green"}}}'
   ```
   - `Kubernetes` 的網路核心會瞬間更新路由規則。所有之後流向 `myapp-service` 這個穩定入口的流量，都會被立刻導向 `version: green` 的 `Pods`。切換完成！
5. 回滾與清理
   - 如果 `v2` 版本出現問題，回滾操作同樣簡單快速：
   ```bash
   kubectl patch service myapp-service -p '{"spec":{"selector":{"version":"blue"}}}'
   ```
   - 確認 `v2` 穩定運行後，CI/CD 管道就可以安全地刪除舊的藍色環境：
   ```bash
   kubectl delete deployment myapp-blue
   ```

##### Ingress 層切換 (更適合微服務與 Canary 發布)

這種方法更上一層樓，它不動 Service，而是修改 Ingress 的路由規則。 這次，「藍色團隊」和「綠色團隊」有各自獨立的內部叫號器 ( `service-blue` 和 `service-green` )。我們去修改的是美食廣場總入口的導覽看板 (Ingress)，將上面「A+ 炸雞排」的指向，從「藍色櫃檯」改成「綠色櫃檯」。

示意 Steps:

1. 初始狀態：
   - 有 `deployment-blue` 和 `deployment-green`。
   - 有兩個 `Service：service-blue` 指向 `version: blue` 的 `Pods，service-green` 指向 `version: green` 的 `Pods`。
   - 有一個 `Ingress` 資源，其規則指向 `service-blue`。
   ```yaml
   # ingress.yaml
   apiVersion: networking.k8s.io/v1
   kind: Ingress
   metadata:
     name: myapp-ingress
   spec:
     rules:
       - host: myapp.example.com
         http:
           paths:
             - path: /
               pathType: Prefix
               backend:
                 service:
                   name: service-blue # 指向藍色服務
                   port:
                     number: 80
   ```
2. 流量切換：
   - CI/CD 管道執行指令來修改 `Ingress` 資源，將 `backend.service.name` 從 `service-blue` 改為 `service-green`。
   ```bash
   # 簡易範例，實務上會用 kubectl patch 或 apply -f
   # 修改 ingress.yaml 中的 service name 後執行 apply
   kubectl apply -f ingress.yaml
   ```
   - Ingress Controller (如 AWS Load Balancer Controller) 偵測到變更後，會去更新 ALB 的轉發規則，將流量導向 service-green。

這種模式是實現金絲雀發布 (Canary Release) 的基礎，我們可以配置 Ingress，將 95% 的流量發送到 service-blue，5% 的流量發送到 service-green，實現小規模灰度驗證。也因為 Service 層的變動較少，更符合其「穩定入口」的定位，職責更清晰。

我們現在掌握的這套結合 `IaC、容器化、CI/CD 和高級部署策略` 的思維模型，不僅僅是一套技術操作指南，它是一種 **現代軟體工程的哲學** 。它是 Google、Netflix、Amazon 等頂級科技公司能夠每週進行數千次部署，同時依然保持系統高度穩定性的核心秘訣。因為它並非單純地解決一個技術問題，而是在解決一個系統性工程問題，它深刻理解到軟體交付不僅僅是寫程式碼，還包括了測試、部署、維運、團隊協作、風險管控等一系列環節。

在未來的職業生涯中，無論工具如何演進（從 Terraform 到 OpenTofu，從 GitHub Actions 到 GitLab CI），這背後關於 `狀態與流程分離` 、 `聲明式與命令式協作` 的核心思想，將會長期適用，並成為構建高品質軟體系統的堅實基礎。工具會隨著時代演進，但設計哲學是邏輯的推導辯證過程，就像

> 《荀子·儒效》：“千舉萬變，其道一也。”

又或是我較喜歡的莊子說的

> 《莊子·天下》：“不離於宗，謂之天人。”

## 6. 配置管理與秘密管理

今天(?)的最後，我們來說說關於 `環境參數` 、 `授權公鑰` 與 `連線端口`該怎麼管理吧

到目前為止，我們的自動化美食廣場已經非常先進了：有標準化的攤位藍圖 (IaC)、標準化的料理包 (Docker)、高效的總管 (ECS/EKS)，還有一條全自動的配送流水線 (CI/CD)。

但現在，我們要處理一個極度敏感且重要的問題： **每個攤位的「獨家秘方」(Secrets) 要放在哪裡？**

我們總不能把「A+ 炸雞排」的獨門醃料配方，用一張紙條貼在 `Dockerfile` 或 `Git` 倉庫的牆上吧？這絕對是安全上的災難。所以接下來我們要學習蟹老闆的精神，死死捂好我們的機密資料，將「配置 (Configuration)」與「秘密 (Secrets)」從我們的應用程式碼中分離出來，並在需要時才「注入」給對的應用。

**K8s 的原生解決方案 — ConfigMap & Secret**

Kubernetes 提供了兩種原生的資源，來處理這個問題。

1. ConfigMap - 「公開的菜單與公告」

   - 用途： 用於儲存非敏感性的配置資料。
   - 比喻： 想像一下美食廣場的公告欄。上面貼著各種公開資訊：
     - API_URL: https://api.internal.mycompany.com (後端 API 的位址)
     - FEATURE_FLAG_NEW_UI: "true" (是否啟用新介面的功能開關)
     - LOG_LEVEL: "info" (日誌紀錄的詳細等級)
   - 特性：
     - 以鍵值對 (key-value pairs) 的形式儲存純文字資料。
     - 非常靈活，可以透過兩種主要方式被 Pod 使用：
       1. 注入為環境變數 (Environment Variables)： 這是最常見的方式。
       2. 掛載為檔案 (Mounted as a file)： 當配置內容較大或格式複雜（例如一個 nginx.conf 檔案）時，可以將整個 ConfigMap 掛載到 Pod 的檔案系統中。
   - 重點： ConfigMap 是純文字，絕不能用來存放任何敏感資訊。

2. Secret - 「鎖在普通抽屜裡的秘方」

   - 用途： 用於儲存敏感性資料。
   - 比喻： 這就像是攤位店長放在自己辦公室一個普通抽屜裡的「秘方筆記本」。上面可能寫著：
     - DATABASE_PASSWORD: "mypassword123"
     - STRIPE*API_KEY: "sk_live*..."
   - 特性：
     - 同樣以鍵值對的形式儲存。
     - 與 ConfigMap 最大的不同是，Kubernetes 儲存 Secret 的值時，會對其進行 Base64 編碼。
       - **Base64 不是加密！** 它只是一種編碼方式，任何人只要拿到編碼後的字串，都可以輕鬆地解碼回原文。
       1. 注入為環境變數 (Environment Variables)： 這是最常見的方式。
       2. 掛載為檔案 (Mounted as a file)： 當配置內容較大或格式複雜（例如一個 nginx.conf 檔案）時，可以將整個 ConfigMap 掛載到 Pod 的檔案系統中。
   - 重點： Kubernetes Secret 的安全性主要依賴於 RBAC (Role-Based Access Control)。我們可以設定嚴格的權限，規定只有特定的 Pod 或管理員才有權限「讀取」這個 Secret 物件。它防君子不防小人，主要防止的是資訊的意外洩漏。

```yaml
# k8s/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
  namespace: ${ENVIRONMENT}
data:
  NODE_ENV: ${ENVIRONMENT}
  LOG_LEVEL: ${LOG_LEVEL}
  API_BASE_URL: ${API_BASE_URL}
---
apiVersion: v1
kind: Secret
metadata:
  name: database-secret
  namespace: ${ENVIRONMENT}
type: Opaque
data:
  url: ${DATABASE_URL_BASE64}
```

但對於一個嚴肅的生產環境，單純使用 K8s Secret 是不夠的。就像我們剛剛說的 `防君子不防小人，主要防止的是資訊的意外洩漏`，這就像把秘方鎖在一個誰都可以輕易撬開的抽屜裡。我們還缺少了：

- 靜態加密 (Encryption at Rest)： 秘方在資料庫（etcd）中只是編碼，不是加密。
- 存取稽核 (Audit Trail)： 誰在什麼時候偷看了秘方？(有人偷看了罷?)我們無從得知。
- 自動輪換 (Automatic Rotation)： 為了安全，密碼應該定期更換。手動更換既繁瑣又容易出錯。

**AWS 的頂級保險櫃 — Secrets Manager**

為了解決上述問題，雲端服務商提供了專門的「秘密管理服務」。在 AWS，這個服務就是 AWS Secrets Manager。

- 用途： 集中式地儲存、管理、稽核、輪換你所有的敏感秘密。
- 比喻： 這不再是一個普通的抽屜，而是一個銀行級的頂級保險櫃。
- 核心優勢：
  1. 強加密： 所有秘密在儲存時，都使用 AWS KMS (Key Management Service) 進行了強大的加密。
  2. 精細的 IAM 權限控制： 你可以透過 IAM Policy，精確到「哪個應用（哪個 IAM Role）只能讀取哪個秘密的哪個版本」。
  3. 完整的稽核紀錄： 與 CloudTrail 深度整合。每一次對秘密的讀取、修改、刪除操作，都會被詳細地記錄下來，供安全團隊審計。
  4. 自動輪換： 這是 Secrets Manager 的王牌功能。它可以與 RDS 等服務整合，實現資料庫密碼的全自動定期輪換，而你的應用程式完全不需要修改程式碼或重啟。

```yaml
# terraform/secrets.tf
resource "aws_secretsmanager_secret" "database_url" {
name = "${var.environment}/database-url"
description = "Database connection string for ${var.environment}"

tags = {
Environment = var.environment
Project     = "multi-env-demo"
}
}

resource "aws_secretsmanager_secret_version" "database_url" {
secret_id     = aws_secretsmanager_secret.database_url.id
secret_string = jsonencode({
url = var.database_url
})
}
```

**最佳實踐:讓 K8s Pod 直接從 AWS 保險櫃取貨**

現在，我們有了 K8s 的應用 (Pod)，也有了 AWS 的保險櫃 (Secrets Manager)。問題是，如何讓 Pod 安全地拿到保險櫃裡的秘方，而不是透過不可靠的人工傳遞？

這就要用到一個關鍵的「橋樑」技術：IAM Roles for Service Accounts (IRSA)，以及一個重要的工具：AWS Secrets and Configuration Provider (ASCP)。

就像我們不會把保險櫃的密碼告訴攤位廚師（Pod）。相反的，我們給這位廚師頒發一張特殊的 **「授權員工卡」(Service Account with an IAM Role)** 。當廚師需要秘方時，他拿著這張卡去刷銀行的特定閘門（ASCP），銀行系統驗證卡片權限後，會自動把今天需要用的那一頁秘方，臨時放到他面前一個專用的、有鎖的盒子里（掛載為 Pod 內的檔案）。廚師用完後盒子就銷毀。

運作流程：

1.  設定 IRSA：

    - 在 AWS IAM 中，建立一個 Role，授權它可以讀取 my-app/db-password 這個 Secret。
    - 在 EKS 中，建立一個 Kubernetes ServiceAccount。
    - 將這兩者「綁定」，告訴 EKS：「凡是使用這個 ServiceAccount 的 Pod，都可以扮演那個 IAM Role 的身份。」

2.  在 Deployment 中指定 ServiceAccount：

    - 在你的 deployment.yaml 中，明確指定 spec.template.spec.serviceAccountName 為你剛剛建立的 ServiceAccount。

3.  使用 ASCP 掛載秘密：

    1.  你需要先在 EKS 叢集中安裝 AWS Secrets and Configuration Provider 這個附加元件。
    2.  然後，在你的 deployment.yaml 中，定義一個 Volume，告訴 ASCP：

             1. 我要掛載一個秘密。
             2. 秘密的名稱是 my-app/db-password (在 Secrets Manager 中的名稱)。
             3. 請把它掛載到 Pod 裡的 /etc/secrets/db-password 這個路徑。

4.  應用程式讀取秘密： - 你的應用程式啟動後，它不再從環境變數讀取密碼，而是直接去讀取本地檔案 /etc/secrets/db-password 的內容。

這個方案的巨大優勢：

- 最高安全性： 秘密從未以明文形式出現在 Kubernetes 的任何資源（YAML、環境變數）中。它從 AWS Secrets Manager 加密傳輸，直接掛載到 Pod 的記憶體檔案系統中。
- 零程式碼耦合： 你的應用程式不知道也不關心秘密是來自 AWS 還是其他地方，它只需要從一個固定的檔案路徑讀取即可。
- 動態更新： 當你在 Secrets Manager 中更新了秘密的值，ASCP 可以近乎即時地在 Pod 內更新掛載的檔案內容，應用程式可以動態讀取到最新的值。

最後我們建立了一個層次分明的配置與秘密管理策略：

> ConfigMap： 用於存放那些公開無妨的應用程式配置。

> K8s Secret： 作為一個基礎的秘密管理工具，主要靠 RBAC 保護，適用於一些敏感度不高的場景。

AWS Secrets Manager + IRSA + ASCP 是在 EKS 上管理高敏感性秘密的黃金標準和最佳實踐。它提供了端到端的加密、稽核與自動輪換能力，是構建安全可靠系統的必要一環。

接下來讓我們暫且賣個關子，將 **監控與日誌管理** 、 **安全性最佳實踐** 、**災難恢復與備份** 這三個主題放在未來 **發佈與運營監控(安全與合規)** 這個階段的時候詳細談談，我們今天(?) 最主要討論的是 **軟體開發與持續整合 - Dev / Staging / Prod 多環境治理與架構策略**。 而明天我們將討論的是 **軟體開發與持續整合** 最後一個內容 - 開發者體驗（DX）優化：內部工具與排錯設計。
