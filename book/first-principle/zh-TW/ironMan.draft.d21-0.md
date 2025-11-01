# Day 21 | 性能測試與負載壓力測試 - 系統性能基準測試與瓶頸分析

完成了《驗證與品質保證》階段的前三個主題後，我們進入最後一個關鍵環節：**性能測試與負載壓力測試**。

現在，我們將從品保 (QA) 角度重新審視它，目標是 **「建立系統性能基準 (Baseline)」** ，並 **「找出系統的崩潰點 (Breaking Point)」** 。這是一個更嚴謹、更科學的過程，<需求確認 × 系統設計起點(二)：領域邊界與基礎需求確認>中我們學習了如何從用戶行為推導 RPS 需求，從中我們了解到 `系統是商業邏輯的實踐` 的這個核心設計理念。我們要探討的不是一個簡單的技術主題，而是一種工程哲學，我們必須建立一個核心默契： **`性能不是一個稍後添加的功能，而是一種根本的、不可妥協的架構屬性`** ，就像摩天大樓的地基一樣，一個性能不佳的系統不只是「慢」，它在本質上就是一個 `無法滿足商業邏輯` 的 `「損壞」` 系統。

今天我們將這些理論轉化為具體的測試場景和性能基準。

經過前面三天對於驗收準則、UX 測試和可測試系統設計的討論，我們已經確保了系統的**功能正確性**、**用戶體驗品質**和**代碼品質**。今天，我們要探討最後一個品質維度：

> **「在真實的負載條件下，我們的系統能否持續穩定地提供服務？」**

這個問題關乎系統的 **「可靠性」**和 **「可擴展性」** 。如果說前面的測試是確保我們建造了一台 **「功能正確」** 、 **「開起來舒適」** 的車，那麼性能測試就是確保這台車能夠「在高速公路上穩定高速行駛」，並且我們清楚知道它的「極限在哪裡」。

我們會聊聊 `如何設計一個完整的性能測試計畫` 、 `如何定義性能指標 (例如：P95 延遲、最大 QPS)` 、 `如何設計測試場景（例如：尖峰流量、壓力測試、耐力測試）` ，以及最重要的，`如何分析測試結果、定位性能瓶頸` 並根據既有數據資料 `建立起未來的預測模型` 。我們將從性能測試的戰略 **「為何」（商業價值）** 開始，深入探討戰術性的 **「如何」（工具與技術）** ，最終達到預測性的 **「未來如何」（容量規劃與成本優化）** 。

## 性能測試的戰略價值：從猜測到預測

想像一位土木工程師設計一座橋樑，他不會憑空猜測橋樑是否能承受交通流量；他會運用材料科學（ `性能指標` ）、交通流量模型（ `工作負載模型` ）和 `壓力測試` ，來預測其在負載下的行為。我們作為系統架構師，必須採用同樣嚴謹的工程紀律。性能測試不是在開發週期結束時的一個儀式性檢查，而是一個貫穿始終的科學探索過程，旨在將系統行為從 `「猜測」` 轉變為 `「預測」` 。

一個規劃良好的性能測試策略能夠消除臆測，讓團隊在問題觸及終端使用者之前，就能衡量系統的速度、穩定性和可擴展性，若缺乏此策略，企業將面臨緩慢的響應時間、尖峰時段的系統崩潰、緊急修復帶來的高昂成本，以及品牌聲譽的損害(就像我們之前說的騎士集團慘案) - 這代表了一種根本性的思維轉變。 `傳統的品質保證（QA）` 通常是發現那些已經被寫入程式碼的錯誤，而正確執行的性能工程，則提供數據來預防一整類錯誤被部署。 這就像 **消防隊** 與 **消防法規稽查員** 的區別：前者撲滅已發生的火災，後者透過執行建築規範來預防火災的發生，性能工程的目標是讓災難性的失敗變得不可想像，而不僅僅是可恢復。

在討論具體的測試技術之前，我們需要先理解性能測試在現代系統設計中的戰略地位。

### 從「功能驗證」到「容量規劃」的思維升級

**傳統思維：功能驗證導向**

在傳統的測試思維中，性能測試常被視為功能測試的「附加驗證」：

```
傳統性能測試思維：
「功能都測試通過了，現在來看看性能如何」
「測試一下系統能不能撐住 100 個用戶同時使用」
「跑個壓力測試，確保不會當機就好」

結果：
✗ 缺乏系統性的測試計劃
✗ 測試場景不貼近真實使用情況
✗ 無法預測系統在不同負載下的表現
```

**現代思維：容量規劃導向**

而在現代的系統思維中，性能測試是 **「容量規劃」** 和 **「風險管理」** 的核心工具。如我們在<需求確認 × 系統設計起點(二)：領域邊界與基礎需求確認>所述，我們需要從業務增長模式推導技術容量需求：

```
現代性能測試思維：
「基於業務需求，我們的系統需要支撐多少負載？」
「在不同的負載模式下，系統的表現如何變化？」
「我們的瓶頸在哪裡？如何預測擴容需求？」

結果：
✓ 建立科學的性能基準
✓ 制定可預測的擴容策略
✓ 建立系統性能的預測模型
```

傳統上，許多團隊將性能測試視為一種技術驗證任務，其目標是確保系統在上線前不會崩潰。然而，這種觀點極大地低估了其戰略重要性。在現代的系統思維中，性能測試是 **「容量規劃」** 和 **「風險管理」** 的核心工具，一個成熟的組織會將性能工程視為一項戰略性的商業智慧活動，它直接影響收入、客戶忠誠度和運營效率 - **商業價值根植於使用者行為** 。緩慢的性能會直接損害使用者體驗，導致挫敗感和用戶流失，這並非空談，而是有具體數據支持的商業現實。電子商務巨頭亞馬遜（Amazon）曾報告，僅僅 `1` 秒的頁面加載延遲，每年就可能使其 `損失 16 億美元` 的銷售額。 另一項數據顯示 `88%` 的美國消費者對 `性能不佳` 的網站和移動應用 `持有負面印象` 。

因此，我們必須學會將抽象的技術指標轉化為具體的人類情感和財務術語。透過<需求確認 × 系統設計起點(二)：領域邊界與基礎需求確認>、<跨團隊協作設計：技術文件、OpenAPI、共用契約 : API 文檔化與團隊協作標準建立> 與<UX 測試與可用性驗證：從觀察使用者行為到修正設計 - 易用性測試(Usability testing)與用戶體驗優化>，的 `情境討論` 、 `使用者操作` 與邊界設定，我們讓這條價值鏈清晰可見：

- 技術指標：延遲（Latency, ms）
- 使用者情感：挫敗感（Frustration）
- 使用者行為：購物車放棄率（Cart Abandonment Rate）
- 財務影響：收入損失（Lost Revenue）

同樣地，另一條價值鏈也存在：

- 技術指標：系統穩定性（Uptime %）
- 使用者情感：信任感（Trust）
- 使用者行為：客戶保留率（Retention Rate）
- 財務影響：客戶終身價值（Customer Lifetime Value, CLV）

這引出了一個更深層次的觀點： `性能測試` 本身就是一種 **商業智慧工具(BI)** 。IBM 提出的「價值模型」（value model）概念，為量化這種聯繫提供了框架，該模型建議創建公式，將使用者行為指標（如購物車放棄率）作為輸入，並產生財務影響作為輸出。在測試過程中，我們可以直接衡量性能對使用者態度的影響，例如透過收集 `淨推薦值（Net Promoter Score, NPS）` 或 `客戶滿意度（Customer Satisfaction, CSAT）` 等指標 。

因此，性能測試的結果不僅僅是技術數據點，它們是一個 **應用程式預測性經濟模型的輸入** 。測試的產出不再是簡單的「通過/失敗」，而是更具洞察力的結論，例如：「在當前的性能水平下，我們預測購物車放棄率為 75%，這在尖峰時段意味著每小時潛在的收入損失為 $X 美元。」我們能夠用數據驅動的方式，清晰地闡述特定工程投資的回報率（ROI）。

### 性能測試的四個戰略目標

在開發過程的早期階段投資性能測試，可以透過避免發布後的高昂修復成本，顯著提高投資回報率 。它能夠及早發現瓶頸，並促進持續改進的文化 。

這就是所謂的 **「左移」（Shift-Left）** 原則在性能領域的應用。透過將性能測試整合到持續整合/持續交付（CI/CD）流程中，我們創建了一個 **快速的回饋循環** 。這個循環不僅僅是為了捕捉性能衰退，更重要的是，它能為架構決策提供資訊。當我們看到提交的變更導致性能下降 5% 時，我們需要更深入地了解系統的性能特徵，從而在未來寫出更高質量的程式碼。這將性能從開發週期末端的一個關卡，轉變為貫穿整個開發過程的持續對話。而一個規劃良好的性能測試策略能夠消除臆測，讓團隊在問題觸及終端使用者之前，就能衡量系統的速度、穩定性和可擴展性，若缺乏此策略，企業將面臨緩慢的響應時間、尖峰時段的系統崩潰、緊急修復帶來的高昂成本，以及品牌聲譽的損害。這也是為什麼我們會需要這四個面向組成的進程 `建立性能基準 (Performance Baseline) => 發現系統瓶頸 (Bottleneck Identification) => 驗證容量規劃 (Capacity Validation) => 建立監控預警 (Monitoring & Alerting)`

**1. 建立性能基準 (Performance Baseline)**

為系統建立可量化的性能標準，作為未來比較和優化的參照點。

```
基準指標範例：
- 平均響應時間：< 200ms
- P95 響應時間：< 500ms
- P99 響應時間：< 1000ms
- 最大併發用戶數：1000
- 每秒事務處理數 (TPS)：500
- 錯誤率：< 0.1%
```

**2. 發現系統瓶頸 (Bottleneck Identification)**

系統性地找出限制系統性能的關鍵因素，為優化提供明確方向。

```
常見瓶頸類型：
- CPU 密集運算
- 記憶體不足
- 資料庫查詢效率
- 網路頻寬限制
- 第三方服務依賴
- 應用程式邏輯缺陷
```

**3. 驗證容量規劃 (Capacity Validation)**

驗證系統的實際容量是否符合業務需求，並為未來擴容提供依據。

```
容量規劃問題：
「雙 11 活動期間，預期流量會是平常的 10 倍，現有系統能否支撐？」
「如果用戶數量增長 50%，我們需要增加多少資源？」
「在什麼情況下需要觸發自動擴容？」
```

**4. 建立監控預警 (Monitoring & Alerting)**

基於測試結果，建立有效的性能監控體系和預警機制。

```
預警策略：
- 當 P95 響應時間 > 400ms 時，發送警告
- 當錯誤率 > 0.05% 時，發送警告
- 當 CPU 使用率 > 80% 時，準備擴容
- 當可用記憶體 < 20% 時，立即擴容
```

基於測試結果，建立有效的性能監控體系和預警機制。例如：「當 P95 響應時間 > 400ms 時，發送警告」。部分內容我們在<開發者體驗（DX）優化：內部工具與排錯設計>的排錯設計中有說到，詳細內容我們會在未來的<可觀測性三大支柱：從監控到回答未知問題>詳細聊聊。

## 性能測試的分類與場景設計

理解了性能測試的戰略價值後，我們需要深入探討其執行方法。

不同類型的性能測試並非一張需要逐項勾選的清單，而是一套科學方法的工具箱，每種方法都旨在回答一個關於系統行為的特定、關鍵問題，包括 `負載測試` 、 `壓力測試` 、 `尖峰測試` 和 `耐力/浸泡測試` ，為了更好地理解它們，我們應將這些定義抽象為它們為業務所回答的核心問題：

- **負載測試（Load Test）** ：回答「我們的系統能否在預期的日常尖峰流量下，維持良好的使用者體驗？」 => 這個測試旨在驗證系統在已知的、正常狀態下的表現 。
- **壓力測試（Stress Test）** ：回答「我們系統的崩潰點在哪裡？它如何失效？」 => 這個測試的重點是理解系統的失效模式——是優雅地降級，還是災難性地崩潰 。
- **尖峰測試（Spike Test）** ：回答「我們能否承受一次突發的、大規模的流量激增，例如病毒式行銷活動或新品發布？」 => 這個測試檢驗系統的彈性和恢復能力 。
- **浸泡測試（Soak / Endurance Test）** ：回答「我們的系統在長時間運行下是否穩定？是否存在緩慢燃燒的問題，如記憶體洩漏或資源耗盡？」 => 這個測試驗證系統的長期可靠性 。

### 性能測試的四種核心類型

**1. 負載測試 (Load Testing)**

**目的**：驗證系統在預期負載下的性能表現

**特徵**：

- 模擬正常業務負載
- 持續時間：30 分鐘到 2 小時
- 用戶數量：預期的正常用戶數量

```javascript
// K6 負載測試範例
import http from "k6/http";
import { check, sleep } from "k6";
import { Rate } from "k6/metrics";

// 自定義指標
export let errorRate = new Rate("errors");

export let options = {
  stages: [
    { duration: "5m", target: 100 }, // 5分鐘內逐漸增加到100用戶
    { duration: "30m", target: 100 }, // 維持100用戶30分鐘
    { duration: "5m", target: 0 }, // 5分鐘內逐漸減少到0
  ],
  thresholds: {
    http_req_duration: ["p(95)<500"], // 95%的請求必須在500ms內完成
    http_req_failed: ["rate<0.01"], // 錯誤率必須低於1%
    errors: ["rate<0.05"], // 自定義錯誤率低於5%
  },
};

export default function () {
  // 模擬用戶瀏覽商品
  let response = http.get("https://api.example.com/products");

  check(response, {
    "status is 200": (r) => r.status === 200,
    "response time < 300ms": (r) => r.timings.duration < 300,
  }) || errorRate.add(1);

  sleep(Math.random() * 3 + 1); // 1-4秒的隨機停留

  // 模擬查看商品詳情
  if (response.status === 200) {
    let products = response.json();
    if (products.length > 0) {
      let productId = products[0].id;
      let detailResponse = http.get(
        `https://api.example.com/products/${productId}`
      );

      check(detailResponse, {
        "product detail status is 200": (r) => r.status === 200,
      }) || errorRate.add(1);
    }
  }

  sleep(Math.random() * 2 + 1); // 1-3秒的隨機停留
}
```

**2. 壓力測試 (Stress Testing)**

**目的**：找出系統的極限承載能力和崩潰點

**特徵**：

- 逐步增加負載直到系統失效
- 觀察系統在高壓下的行為
- 驗證系統的錯誤處理和恢復能力

```javascript
// K6 壓力測試範例
export let options = {
  stages: [
    { duration: "5m", target: 100 }, // 正常負載
    { duration: "5m", target: 200 }, // 增加到2倍負載
    { duration: "5m", target: 500 }, // 增加到5倍負載
    { duration: "5m", target: 1000 }, // 增加到10倍負載
    { duration: "5m", target: 1500 }, // 極限測試
    { duration: "5m", target: 0 }, // 恢復測試
  ],
  thresholds: {
    http_req_duration: ["p(95)<1000"], // 放寬響應時間要求
    http_req_failed: ["rate<0.1"], // 容忍較高錯誤率
  },
};

export default function () {
  let response = http.get("https://api.example.com/products");

  // 記錄詳細的性能數據
  console.log(
    `VUs: ${__VU}, Iteration: ${__ITER}, Response Time: ${response.timings.duration}ms`
  );

  check(response, {
    "status is not 5xx": (r) => r.status < 500, // 關注系統是否完全崩潰
  });

  sleep(1);
}
```

**3. 尖峰測試 (Spike Testing)**

**目的**：驗證系統在突發流量下的表現

**特徵**：

- 短時間內大幅增加負載
- 模擬突發事件（如促銷活動、新聞熱點）
- 測試自動擴容機制

```javascript
// K6 尖峰測試範例
export let options = {
  stages: [
    { duration: "2m", target: 100 }, // 正常負載
    { duration: "1m", target: 1000 }, // 快速增加到10倍負載
    { duration: "3m", target: 1000 }, // 維持高負載
    { duration: "1m", target: 100 }, // 快速降回正常負載
    { duration: "2m", target: 100 }, // 恢復期觀察
  ],
  thresholds: {
    http_req_duration: ["p(95)<800"],
    http_req_failed: ["rate<0.05"],
  },
};
```

**4. 耐力測試 (Endurance Testing)**

**目的**：驗證系統在長時間運行下的穩定性

**特徵**：

- 長時間持續負載（通常 8-24 小時）
- 發現記憶體洩漏等長期問題
- 驗證系統的穩定性和可靠性

```javascript
// K6 耐力測試範例
export let options = {
  stages: [
    { duration: "30m", target: 200 }, // 逐漸增加到目標負載
    { duration: "8h", target: 200 }, // 維持負載8小時
    { duration: "30m", target: 0 }, // 逐漸降低負載
  ],
  thresholds: {
    http_req_duration: ["p(95)<500"],
    http_req_failed: ["rate<0.01"],
    checks: ["rate>0.99"], // 檢查通過率必須大於99%
  },
};

export default function () {
  // 更複雜的業務流程模擬
  simulateUserJourney();
  sleep(Math.random() * 5 + 2); // 2-7秒的隨機停留
}

function simulateUserJourney() {
  // 登入
  let loginResponse = http.post("https://api.example.com/auth/login", {
    username: "testuser",
    password: "testpass",
  });

  if (loginResponse.status === 200) {
    let token = loginResponse.json().token;
    let headers = { Authorization: `Bearer ${token}` };

    // 瀏覽商品
    http.get("https://api.example.com/products", { headers });

    // 加入購物車
    http.post(
      "https://api.example.com/cart/items",
      {
        productId: 1,
        quantity: 1,
      },
      { headers }
    );

    // 結帳
    http.post(
      "https://api.example.com/orders",
      {
        paymentMethod: "credit_card",
      },
      { headers }
    );
  }
}
```

我們簡單總結這些測試類型的差異與應用場景。

表 1：性能測試類型比較指南

| **測試類型**     | **主要目標**                                           | **負載模式**                                     | **持續時間**             | **典型應用場景**                                     |
| ---------------- | ------------------------------------------------------ | ------------------------------------------------ | ------------------------ | ---------------------------------------------------- |
| **負載測試**     | 驗證在預期峰值負載下的性能                             | 穩定、持續的預期峰值負載                         | 中等（例如 1-2 小時）    | 驗證系統是否滿足日常營運的服務等級協議（SLA）        |
| **壓力測試**     | 找到系統的極限（崩潰點）並測試其恢復能力               | 負載逐漸增加，直至超過系統容量                   | 中等到較長               | 確定系統的最大容量，了解系統在極端壓力下的行為       |
| **尖峰測試**     | 評估系統應對突發流量激增的能力                         | 負載在極短時間內急劇增加和減少                   | 短暫（例如 5-15 分鐘）   | 模擬「黑色星期五」搶購、熱門新聞事件或病毒式行銷活動 |
| **浸泡測試**     | 識別長時間運行下的性能衰退問題（如記憶體洩漏）         | 穩定、中等強度的負載                             | 長時間（例如 8-72 小時） | 確保關鍵業務系統（如 ERP）能夠 24/7 穩定運行         |
| **容量測試**     | 確定系統在不違反性能目標的情況下能處理的最大使用者數量 | 負載逐步增加，直至性能指標（如響應時間）超出閾值 | 中等                     | 進行容量規劃，為未來的業務增長做準備                 |
| **可擴展性測試** | 評估系統在增加硬體資源時性能的提升程度                 | 在不同硬體配置下運行多輪負載測試                 | 多輪，每輪時間中等       | 驗證系統的水平擴展或垂直擴展能力，規劃基礎設施投資   |

**現實模擬的藝術：從需求到場景**

有效的測試案例必須反映真實的使用情況 。這需要深入理解需求、識別關鍵功能、從使用者視角思考，並概述先決條件和預期結果，這個過程是連接商業需求和詳細測試案例的橋樑。要記住:

> 一個測試場景是關於 `使用者` 試圖 `達成某個目標` 的故事。

一個糟糕的場景是：

```
「向 /products/123 發送 1000 個 GET 請求。」
```

一個好的場景則是：

```
「
1. 模擬 1000 名使用者，
2. 他們登入系統，
3. 搜索『跑鞋』，
4. 瀏覽三個產品頁面，
5. 將其中一個加入購物車，
6. 然後進入結帳流程。
」
```

這需要對使用者旅程有深刻的理解，這也是為什麼會如此強調 <需求確認 × 系統設計起點(二)：領域邊界與基礎需求確認>、<跨團隊協作設計：技術文件、OpenAPI、共用契約 : API 文檔化與團隊協作標準建立> 與<UX 測試與可用性驗證：從觀察使用者行為到修正設計 - 易用性測試(Usability testing)與用戶體驗優化> 這三個主題與內容。

這背後隱藏著一個更為深刻的原理：`工作負載模型（Workload Modeling` ）是一門 **`預測性的行為科學`** 。

要`創建` 真實的場景，首先需要 `理解` 使用者的行為模式 。

這通常涉及分析生產環境的數據（如果可用），或與業務利益相關者合作，定義關鍵的使用者旅程及其相對頻率（例如，80% 的使用者在瀏覽，15% 在搜索，5% 在購買）。

這些數據被用來建立一個「工作負載模型」，這是一個對生產流量的抽象數學表示。為了使模擬不那麼機械化、更接近人類行為，需要引入「思考時間」（Think Time）和「步調」（Pacing）等概念 。

讓我們來看一個具體的例子，將理論付諸實踐。

```markdown
## AWS 應用舉例：為無伺服器電商的限時搶購設計尖峰測試

- 場景：一個為期 `1` 小時的熱門商品限時搶購活動，預計將帶來 `10` 倍於平時的流量。該應用程式使用 AWS Lambda、API Gateway 和 DynamoDB 構建。

- 測試設計：
  - 工作負載模型：測試場景將模擬使用者在 `5` 分鐘內迅速增加到基準值的 `10` 倍，將該負載維持 `1` 小時，然後逐漸減少。使用者旅程將高度集中在商品詳情頁和結帳流程上。

成功標準：在整個 1 小時的尖峰期間，「加入購物車」API 呼叫的 p95 響應時間必須維持在 500 毫秒以下，錯誤率低於 0.1%。這個標準將一個技術測量指標與一個清晰的業務目標（成功的促銷活動）緊密地聯繫在一起。
```

在這個案例中，我們測試的不僅僅是「應用程式」本身，而是具體測試 `API Gateway` 的擴展性極限（帳戶級別的節流限制）、 `AWS Lambda` 的冷啟動和並發行為，以及 `DynamoDB` 表（用於商品目錄和訂單）的預置吞吐量或隨需容量模式的響應能力。

因此，性能場景設計 `不能` 僅僅是一項腳本編寫任務，它是一種應用行為科學的實踐。我們正在建立一個大規模使用者行為的預測模型，性能預測的準確性，與這個行為模型的逼真度成正比。這意味著，在場景設計中，最關鍵的技能不是編碼，而是 **同理心** 和 **分析能力** ，能夠設身處地為使用者著想，並將他們的目標轉化為可重現、可衡量的腳本。這需要跨團隊的緊密合作，包括開發人員、QA、產品經理和業務分析師 。

## 壓力測試工具選擇與示例

在確定了「為何」測試以及「測試什麼」之後，我們現在轉向「如何」執行的問題，選擇合適的工具至關重要，因為工具不僅決定了測試的執行效率，更深層次地，它反映並影響著一個團隊的工程文化。

業界主流的開源性能測試工具有 JMeter、Gatling 和 k6。它們在語言、架構、資源效率和目標受眾方面存在顯著差異 。

### 工具特性對比

| **特性**         | **Apache JMeter**                       | **Gatling**                        | **k6 (by Grafana Labs)**                             |
| ---------------- | --------------------------------------- | ---------------------------------- | ---------------------------------------------------- |
| **核心哲學**     | GUI 驅動，功能全面，適合 QA             | 測試即程式碼，高性能，適合開發者   | 開發者體驗優先，DevOps 原生，輕量級                  |
| **腳本語言**     | GUI（XML 存儲），支持 Groovy, BeanShell | Scala DSL (領域特定語言)           | JavaScript (ES6)                                     |
| **架構**         | 基於執行緒，每個虛擬使用者一個執行緒    | 異步，事件驅動 (Akka, Netty)       | 基於事件循環 (Go 語言核心)                           |
| **資源消耗**     | 較高                                    | 低                                 | 非常低                                               |
| **學習曲線**     | GUI 模式下較低，進階功能較陡峭          | 需要 Scala 知識，對開發者較友好    | 對熟悉 JavaScript 的開發者非常友好                   |
| **CI/CD 整合**   | 可整合，但通常需要額外配置              | 原生支持，易於整合                 | 設計初衷即為 CI/CD，整合極其簡單                     |
| **報告**         | 基本的 HTML 報告，可透過外掛擴展        | 非常詳細且美觀的交互式 HTML 報告   | 命令行輸出，可輕鬆整合到 Grafana, Datadog 等平台     |
| **生態系統**     | 極其龐大，擁有大量第三方外掛            | 較小，但穩定增長                   | 快速增長，透過 xk6 擴展                              |
| **分散式測試**   | 原生支持主從模式                        | 商業版提供，開源版需手動設置       | 原生不支持，推薦使用 k6 Cloud 或 Kubernetes Operator |
| **最適合的團隊** | 傳統 QA 團隊，需要廣泛協議支持的企業    | 追求高性能和程式碼化測試的開發團隊 | 實踐 DevOps 和「左移」測試的現代工程團隊             |

**Apache JMeter：資深元老**

JMeter 採用圖形化使用者介面（GUI）驅動，這使得非程式設計師也能輕鬆上手。它基於 Java，擁有龐大而成熟的外掛生態系統，使其功能極其強大和通用。然而，這也使其在運行時相對資源密集 。JMeter 的設計哲學是提供一個全面的、基於 UI 的測試建構環境，非常適合傳統的、有專職 QA 團隊的組織結構。

**Gatling：性能純粹主義者**

Gatling 採用「測試即程式碼」（Test-as-Code）的方法，使用 Scala 語言的領域特定語言（DSL）來編寫腳本。它建立在一個高性能的異步架構之上，能夠非常高效地產生負載。其生成的美觀且詳細的 HTML 報告是其一大特色 。Gatling 的哲學是開發者中心的，將性能測試視為應用程式原始碼的一部分，是後端和自動化工程師的理想選擇。

**k6 (by Grafana Labs)：DevOps 原生**

k6 使用現代 JavaScript (ES6) 編寫腳本，其核心則由 Go 語言編寫以確保高性能。它輕量、命令行優先，專為輕鬆整合到 CI/CD 流程而設計 。k6 的哲學是「左移」，賦予開發人員能力，將性能測試作為其日常工作流程的一部分來編寫和運行。

> **工具的選擇不僅僅是一個技術決策，它更是一種工程文化的體現。**

這些工具的主要使用者群體不同：JMeter 面向 QA 分析師，Gatling 面向後端/自動化工程師，而 k6 則面向開發人員、SRE 和 DevOps 工程師，這些角色對應著不同的組織結構和開發方法論。傳統的瀑布式或孤島式組織通常設有專職的 QA 團隊，他們傾向於使用像 JMeter 這樣的 GUI 工具，現代的 DevOps 或敏捷組織則強調跨功能團隊和開發人員對其程式碼品質的擁有權（「你構建，你運行」）；這些團隊偏愛能夠無縫整合到現有開發者工具鏈（程式碼編輯器、Git、CI/CD）中的工具，這使得 k6 和 Gatling 成為自然之選 。

採用像 k6 這樣的工具可以成為推動 DevOps 轉型的催化劑，但如果缺乏開發人員主人翁精神的文化基礎，這種嘗試也可能失敗。反之，強迫一個以開發者為中心的團隊使用一個笨重的 UI 工具，會製造摩擦並降低生產力。在提供工具建議時，必須評估組織的工程文化和目標。問題不應是「哪個工具最好？」，而應是「哪個工具最符合我們團隊現在的工作方式，或者我們期望他們未來的工作方式？」

### AWS 投資交易系統的實戰測試場景

基於<需求確認 × 系統設計起點(二)：領域邊界與基礎需求確認>的精確 RPS 推導方法，我們設計真實的業務場景測試：

**用戶行為模式分析**：

| 用戶類型   | 日查詢次數 | 日交易次數 | RPS/用戶 | 占比 |
| ---------- | ---------- | ---------- | -------- | ---- |
| 普通投資者 | 2-5        | 0-2        | 0.17     | 80%  |
| 活躍交易者 | 20-50      | 5-15       | 1.74     | 15%  |
| 高頻交易者 | 3600/小時  | 60/小時    | 60       | 5%   |

**綜合 RPS 計算**：

- 總 RPS = (0.8 × 0.17 + 0.15 × 1.74 + 0.05 × 60) × total_users
- = **3.397 × total_users**

**AWS 服務選型的 RPS 臨界點**：

| RPS 範圍       | 推薦架構             | 成本特點             |
| -------------- | -------------------- | -------------------- |
| < 100 RPS      | API Gateway + Lambda | 按需付費，啟動成本低 |
| 100-1000 RPS   | ALB + ECS/EC2        | 平衡成本與性能       |
| 1000-10000 RPS | ALB + Auto Scaling   | 可預測擴容           |
| > 10000 RPS    | Custom LB + 多 AZ    | 高可用性優先         |

**時間分佈模式的測試設計**：

```javascript
// 投資交易系統的真實負載模擬
export let options = {
  scenarios: {
    // 開盤前突發流量 (10倍基準負載)
    pre_market: {
      executor: "ramping-vus",
      startTime: "0s",
      stages: [
        { duration: "30s", target: 1000 }, // 快速攀升
        { duration: "30m", target: 1000 }, // 開盤前高峰
        { duration: "30s", target: 200 }, // 快速回落
      ],
      exec: "tradingScenario",
    },

    // 交易時段穩定負載 (5倍基準負載)
    trading_hours: {
      executor: "constant-vus",
      startTime: "31m",
      duration: "6h",
      vus: 500,
      exec: "tradingScenario",
    },

    // 收盤後逐漸減少
    after_market: {
      executor: "ramping-vus",
      startTime: "7h31m",
      stages: [
        { duration: "2h", target: 100 }, // 逐漸減少
        { duration: "1h", target: 0 }, // 完全停止
      ],
      exec: "tradingScenario",
    },
  },
  thresholds: {
    // 嚴格的交易系統性能要求
    "http_req_duration{scenario:pre_market}": ["p(95)<100"], // 開盤前要求極低延遲
    "http_req_duration{scenario:trading_hours}": ["p(95)<200"], // 交易時段穩定性能
    http_req_failed: ["rate<0.001"], // 錯誤率必須極低
  },
};

export function tradingScenario() {
  group("investment_trading_flow", function () {
    // 1. 查詢持倉 (高頻操作)
    let portfolioResponse = http.get(
      `${__ENV.BASE_URL}/api/portfolios/${user_id}`
    );
    check(portfolioResponse, {
      "portfolio query < 50ms": (r) => r.timings.duration < 50,
    });

    sleep(0.1); // 100ms思考時間

    // 2. 市場數據查詢 (實時價格)
    let marketResponse = http.get(`${__ENV.BASE_URL}/api/market/quotes/AAPL`);
    check(marketResponse, {
      "market data < 30ms": (r) => r.timings.duration < 30,
    });

    // 3. 風險評估 (計算密集)
    if (Math.random() < 0.1) {
      // 10%的用戶進行交易
      let riskResponse = http.post(`${__ENV.BASE_URL}/api/risk/calculate`, {
        portfolio_id: user_id,
        proposed_trade: {
          symbol: "AAPL",
          quantity: 100,
          side: "buy",
        },
      });

      check(riskResponse, {
        "risk calculation < 500ms": (r) => r.timings.duration < 500,
      });
    }

    sleep(Math.random() * 10 + 5); // 5-15秒的隨機間隔
  });
}
```

現行(2025)，AWS 提供了一個名為「Distributed Load Testing on AWS」的解決方案，該方案利用 AWS Fargate 或 Amazon ECS 等服務，自動化地配置一個可擴展的負載產生器集群，而且這個解決方案原生支持 JMeter、k6 和 Locust。

這個 AWS 解決方案使得大規模測試變得普及化。過去，要模擬數百萬虛擬使用者需要昂貴的專用硬體或複雜的手動設置；現在，我們只需將測試腳本打包在一個容器中，AWS 解決方案就會處理在數百甚至數千個 Fargate 任務上運行該容器的協調工作。這使我們能夠從多個 AWS 區域模擬全球使用者流量，從而獲得更真實的測試結果 。這是一個利用雲端解決傳統測試難題的典型範例：測試基礎設施本身成為瓶頸。我們實質上是為了一次測試，臨時創建了一個無伺服器的超級計算機，並在測試結束後立即銷毀它，真正實現了按使用付費 。

## 性能指標定義與分析方法

性能指標存在於多個層次，從 `底層的資源利用率（CPU、記憶體）` ，到 `中層的應用程式性能（響應時間、吞吐量）` ，再到 `頂層的業務關鍵績效指標（KPIs）（收入、滿意度）` 。我們可以將這些指標組織成一個概念性的金字塔模型，這個金字塔模型不僅是一個分類系統，更是一個強大的診斷工具。它提供了一條結構化的、自上而下的診斷路徑，讓我們能夠將一個高層次的業務問題追溯到其底層的技術根源：

**底層（基礎設施指標）** ：CPU 利用率、記憶體使用量、磁碟 I/O、網路頻寬。

- 這些是系統運行的基礎資源。它們是潛在問題的領先指標（Leading Indicators）。例如，持續高企的 CPU 利用率預示著系統即將達到處理能力的上限。

**中層（應用程式性能指標 - APM）**：平均響應時間、p95/p99 延遲、吞吐量（每秒請求數/交易數）、錯誤率。

- 這些指標直接描述了使用者感知的性能。它們是應用程式如何高效利用底層基礎設施資源的直接結果。

**頂層（業務 KPIs）**：轉換率、使用者參與度、客戶流失率、每使用者平均收入（ARPU）、客戶支援工單量。

- 這些是衡量業務成功的滯後指標（Lagging Indicators），它們直接受到中層應用程式性能指標的影響。

我們來順一下由上而下的模擬流程:

> 1. 一個業務問題在金字塔頂層被觀察到（例如，「昨天我們的轉換率下降了 10%」）。
>
> 2. 分析師隨即檢查金字塔中層。他們發現，轉換率的下降與結帳服務的 p99 延遲急劇上升在時間上高度相關 。
>
> 3. 這引導他們深入底層並檢查結帳服務的基礎設施指標
>
> 4. 發現資料庫伺服器的 CPU 利用率在同一時期達到了 100%。

反之，這個模型也讓我們能夠預測一個在底層觀察到的技術問題可能帶來的業務影響。

> 1. 如果這個記憶體洩漏問題繼續下去
>
> 2. 我們預測系統將在 4 小時後崩潰
>
> 3. 這將導致 Y 美元的收入損失

因此，一個有效的性能監控和分析平台（如 APM 工具）必須能夠關聯金字塔所有層次的數據，將一個業務交易、執行它的應用程式程式碼以及運行它的基礎設施聯繫起來，才能發揮其最大價值。

### 核心性能指標體系

在開始測試之前，必須清晰地定義性能驗收標準並建立一個基準線 。這些目標應該是可衡量的（遵循 S.M.A.R.T. 原則），並與業務目標保持一致。我們應該做到

> **定義「好」的標準**

孤立的測試結果，如「響應時間為 200 毫秒」，本身是沒有意義的 - 這是好是壞？

這個答案取決於由 `服務等級目標（Service Level Objective, SLO）` 所定義的上下文和 `基準線（Baseline）` 所建立的門檻， `SLO` 是一個對性能指標的精確、可衡量的目標（例如，「99% 的登入請求應在 300 毫秒內完成」）- 它是關於「好」的標準的正式約定; `基準線（Baseline）` 則是在正常條件下對性能的一次測量，它成為所有未來測試的參考點，沒有 SLO 和基準線，性能測試只是在產生沒有洞察的數字。

為了將工程師的語言轉化為商業邏輯的語言，我們來歸納整理一個技術性能指標與業務 KPIs 之間的對應關係。

| **技術性能指標 (中層)** | **潛在的業務影響 (頂層)**                    | **業務 KPI 示例**                                 |
| ----------------------- | -------------------------------------------- | ------------------------------------------------- |
| **高響應時間/延遲**     | 使用者挫敗感增加，放棄操作                   | 購物車放棄率、頁面跳出率、任務完成率              |
| **低吞吐量**            | 系統無法處理尖峰流量，導致使用者被拒絕服務   | 銷售損失、新使用者註冊失敗率                      |
| **高錯誤率**            | 功能不可用，使用者體驗差，數據可能丟失       | 客戶滿意度 (CSAT) 下降、客戶支援工單量增加        |
| **系統不穩定/低可用性** | 損害品牌信任度，使用者流向競爭對手           | 客戶流失率 (Churn Rate) 上升、淨推薦值 (NPS) 下降 |
| **資源利用率過高**      | 基礎設施成本增加，擴展能力受限               | 營運成本佔收入百分比、每使用者基礎設施成本        |
| **快速的恢復時間**      | 故障對使用者的影響時間縮短，增強了系統的韌性 | 服務等級協議 (SLA) 達標率、平均修復時間 (MTTR)    |

### 性能分析的科學方法

為了從測試結果中提取有意義的洞察，我們需要採用統計學分析方法。以下是一個使用 Python 進行性能數據分析的範例腳本示意，主要是計算關鍵指標、檢測性能衰退並識別潛在瓶頸。

**統計分析方法**

```python
# 性能數據分析腳本
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns
from scipy import stats

class PerformanceAnalyzer:
    def __init__(self, data_file):
        self.data = pd.read_csv(data_file)
        self.prepare_data()

    def prepare_data(self):
        """準備和清理數據"""
        # 轉換時間戳
        self.data['timestamp'] = pd.to_datetime(self.data['timestamp'])

        # 移除異常值（使用 IQR 方法）
        Q1 = self.data['response_time'].quantile(0.25)
        Q3 = self.data['response_time'].quantile(0.75)
        IQR = Q3 - Q1

        lower_bound = Q1 - 1.5 * IQR
        upper_bound = Q3 + 1.5 * IQR

        self.data_cleaned = self.data[
            (self.data['response_time'] >= lower_bound) &
            (self.data['response_time'] <= upper_bound)
        ]

    def calculate_percentiles(self):
        """計算各項百分位數指標"""
        response_times = self.data_cleaned['response_time']

        percentiles = {
            'P50 (Median)': np.percentile(response_times, 50),
            'P75': np.percentile(response_times, 75),
            'P90': np.percentile(response_times, 90),
            'P95': np.percentile(response_times, 95),
            'P99': np.percentile(response_times, 99),
            'P99.9': np.percentile(response_times, 99.9),
        }

        return percentiles

    def analyze_throughput(self):
        """分析吞吐量趨勢"""
        # 按分鐘分組計算 TPS
        self.data_cleaned['minute'] = self.data_cleaned['timestamp'].dt.floor('T')
        tps_by_minute = self.data_cleaned.groupby('minute').size()

        return {
            'average_tps': tps_by_minute.mean(),
            'max_tps': tps_by_minute.max(),
            'min_tps': tps_by_minute.min(),
            'tps_std': tps_by_minute.std(),
        }

    def detect_performance_degradation(self):
        """檢測性能劣化"""
        # 將數據分為兩半進行比較
        midpoint = len(self.data_cleaned) // 2
        first_half = self.data_cleaned.iloc[:midpoint]['response_time']
        second_half = self.data_cleaned.iloc[midpoint:]['response_time']

        # 使用 t-test 檢驗性能是否有顯著差異
        t_stat, p_value = stats.ttest_ind(first_half, second_half)

        first_half_p95 = np.percentile(first_half, 95)
        second_half_p95 = np.percentile(second_half, 95)

        degradation_percentage = ((second_half_p95 - first_half_p95) / first_half_p95) * 100

        return {
            't_statistic': t_stat,
            'p_value': p_value,
            'is_significant': p_value < 0.05,
            'first_half_p95': first_half_p95,
            'second_half_p95': second_half_p95,
            'degradation_percentage': degradation_percentage,
        }

    def identify_bottlenecks(self):
        """識別性能瓶頸"""
        correlations = {}

        if 'cpu_usage' in self.data.columns:
            correlations['cpu_vs_response_time'] = self.data['cpu_usage'].corr(
                self.data['response_time']
            )

        if 'memory_usage' in self.data.columns:
            correlations['memory_vs_response_time'] = self.data['memory_usage'].corr(
                self.data['response_time']
            )

        if 'db_query_time' in self.data.columns:
            correlations['db_vs_response_time'] = self.data['db_query_time'].corr(
                self.data['response_time']
            )

        # 找出最強相關性
        strongest_correlation = max(correlations.items(), key=lambda x: abs(x[1]))

        return {
            'correlations': correlations,
            'strongest_bottleneck': strongest_correlation,
        }

    def generate_performance_report(self):
        """生成完整的性能報告"""
        report = {
            'test_summary': {
                'total_requests': len(self.data),
                'valid_requests': len(self.data_cleaned),
                'error_rate': (len(self.data) - len(self.data_cleaned)) / len(self.data),
                'test_duration': (
                    self.data['timestamp'].max() - self.data['timestamp'].min()
                ).total_seconds(),
            },
            'percentiles': self.calculate_percentiles(),
            'throughput': self.analyze_throughput(),
            'degradation_analysis': self.detect_performance_degradation(),
            'bottleneck_analysis': self.identify_bottlenecks(),
        }

        return report

    def plot_performance_trends(self):
        """繪製性能趨勢圖"""
        fig, axes = plt.subplots(2, 2, figsize=(15, 10))

        # 響應時間趨勢
        axes[0, 0].plot(self.data_cleaned['timestamp'], self.data_cleaned['response_time'])
        axes[0, 0].set_title('Response Time Trend')
        axes[0, 0].set_xlabel('Time')
        axes[0, 0].set_ylabel('Response Time (ms)')

        # 響應時間分布
        axes[0, 1].hist(self.data_cleaned['response_time'], bins=50, alpha=0.7)
        axes[0, 1].set_title('Response Time Distribution')
        axes[0, 1].set_xlabel('Response Time (ms)')
        axes[0, 1].set_ylabel('Frequency')

        # TPS 趨勢
        tps_data = self.data_cleaned.groupby(
            self.data_cleaned['timestamp'].dt.floor('T')
        ).size()
        axes[1, 0].plot(tps_data.index, tps_data.values)
        axes[1, 0].set_title('Throughput Trend (TPS)')
        axes[1, 0].set_xlabel('Time')
        axes[1, 0].set_ylabel('Transactions per Second')

        # 錯誤率趨勢
        error_data = self.data.groupby(
            self.data['timestamp'].dt.floor('T')
        )['status_code'].apply(lambda x: (x != 200).mean())
        axes[1, 1].plot(error_data.index, error_data.values * 100)
        axes[1, 1].set_title('Error Rate Trend')
        axes[1, 1].set_xlabel('Time')
        axes[1, 1].set_ylabel('Error Rate (%)')

        plt.tight_layout()
        plt.savefig('performance_analysis.png', dpi=300, bbox_inches='tight')
        plt.show()

# 使用範例
analyzer = PerformanceAnalyzer('performance_test_results.csv')
report = analyzer.generate_performance_report()
analyzer.plot_performance_trends()

print("Performance Analysis Report:")
print(f"P95 Response Time: {report['percentiles']['P95']:.2f}ms")
print(f"Average TPS: {report['throughput']['average_tps']:.2f}")
print(f"Error Rate: {report['test_summary']['error_rate']:.4f}")
```

## 瓶頸分析與性能優化策略

瓶頸是系統中任何限制整體效率的組件，在這裡，資源需求超過了其供應能力。還記得我們在<高併發與限流設計：如何避免資源瓶頸>與<料庫設計哲學:需求解析、技術選型與 Schema 設計策略>中不斷地說明與強調資料的應用嗎?要記住一個最核心的概念

```
需求( require ) => 行為(conduct) => 影響(effect)
```

當瓶頸發生時就代表著我們的 `商業邏輯的實踐` 出現了無法執行的狀況，這對於我們的系統來說是一個嚴重的缺失 - 代表著核心商業邏輯的失敗。

雖然常見的瓶頸發生在 CPU、記憶體、磁碟 I/O、網路和資料庫等層面，但在所有潛在的瓶頸中，`資料庫` 往往是系統的性能重心，問題包括低效的查詢、缺失的索引、鎖競爭和連接池耗盡等。一個緩慢的資料庫查詢會產生連鎖反應，發起呼叫的應用程式執行緒現在被阻塞，在等待期間佔用著記憶體和 CPU 核心。如果大量執行緒都在等待資料庫，應用程式伺服器的連接池將被耗盡，並開始拒絕新的請求，此時，應用程式伺服器的 CPU 看似很高，但實際上可能只是在等待的執行緒之間進行無效的上下文切換。

因此，那些表現為 `「Web 伺服器 CPU 過高` 」或 `「應用程式記憶體耗盡」` 的問題，通常只是緩慢、掙扎的資料庫所引發的症狀。

這意味著，在開始瓶頸分析時，`資料庫` 幾乎總是應該成為首要懷疑對象，優化一個頻繁執行的查詢，往往能對整個系統的性能產生不成比例的巨大正面影響，甚至解決看似發生在其他層次的瓶頸。

我們必須像醫生診斷疾病一樣來處理瓶頸分析，從症狀追溯到根本原因：

> 1. 觀察症狀：高響應時間、低吞吐量、高錯誤率（對應我們性能金字塔的中層）。
>
> 2. 建立假設：例如，「瓶頸很可能在資料庫，因為『獲取使用者個人資料』這個交易是最慢的。」
>
> 3. 測試假設：使用專門的工具從被懷疑的組件中收集證據。
>
> 4. 識別根源：例如，「『users』表在『email』欄位上缺少索引，導致每次登入都觸發全表掃描。」
>
> 5. 應用補救措施：為該欄位添加索引。
>
> 6. 驗證修復：重新運行性能測試，確認瓶頸已經消除，並且沒有引入新的瓶頸。

根據<高併發與限流設計：如何避免資源瓶頸>的分層監控方法，以下是完整的瓶頸檢測指標：

| 層級         | 指標名稱                            | 說明                                         | 常見檢測工具/方法                          |
| ------------ | ----------------------------------- | -------------------------------------------- | ------------------------------------------ |
| **應用層**   | 吞吐量 (Throughput / RPS/QPS)       | 每秒可處理的請求數，衡量系統承載能力         | JMeter, k6, Locust, New Relic              |
|              | 響應延遲 (Response Latency)         | 請求從進入到回應的耗時，通常關注 P50/P95/P99 | APM (Datadog, New Relic), OpenTelemetry    |
|              | 錯誤率 (Error Rate)                 | HTTP 4xx/5xx 比例，反映應用健壯性            | APM, ELK, Sentry                           |
|              | 併發連線數 (Concurrent Connections) | 同時處理的使用者/會話數量                    | 系統監控 (Prometheus, Grafana)             |
|              | 任務排隊長度 (Queue Length)         | Thread pool、任務隊列積壓狀況                | Micrometer, RabbitMQ/Kafka metrics         |
|              | 資源等待時間 (Wait Time)            | DB 連線池、API Gateway 排隊耗時              | APM Trace, pgbouncer stats                 |
| **資料庫層** | 查詢延遲 (Query Latency)            | 單次 SQL 查詢或交易的耗時                    | MySQL Slow Query Log, pg_stat_statements   |
|              | 每秒查詢數 (QPS/TPS)                | 資料庫吞吐量                                 | MySQL performance_schema, Postgres metrics |
|              | 慢查詢比率 (Slow Query %)           | 超過閾值的查詢比例                           | 慢查詢日誌分析, pt-query-digest            |
|              | 索引命中率 (Index Hit Ratio)        | 查詢是否有效利用索引                         | EXPLAIN, pg_stat_user_indexes              |
|              | 快取命中率 (Cache Hit Ratio)        | DB buffer pool / Redis/Memcached 命中率      | MySQL InnoDB metrics, Redis INFO           |
|              | 鎖等待/死鎖 (Lock Waits/Deadlocks)  | 交易衝突造成的等待或死鎖                     | MySQL Performance Schema, pg_locks         |

**1. 應用層瓶頸**

```javascript
// K6 腳本：應用層性能測試
export default function () {
  group("application_layer_analysis", function () {
    // 測試不同 API 端點的性能
    let endpoints = [
      "/api/products", // 簡單查詢
      "/api/products/search", // 複雜搜尋
      "/api/orders", // 事務處理
      "/api/reports/analytics", // 數據分析
    ];

    endpoints.forEach((endpoint) => {
      let response = http.get(`${__ENV.BASE_URL}${endpoint}`);

      // 記錄各端點的性能
      console.log(`${endpoint}: ${response.timings.duration}ms`);

      check(response, {
        [`${endpoint} responds within SLA`]: (r) => r.timings.duration < 500,
      });
    });

    sleep(1);
  });
}
```

**2. 資料庫瓶頸分析**

```sql
-- 資料庫性能監控查詢
-- PostgreSQL 範例

-- 找出最慢的查詢
SELECT
    query,
    calls,
    total_time,
    mean_time,
    max_time,
    rows
FROM pg_stat_statements
ORDER BY mean_time DESC
LIMIT 10;

-- 找出最頻繁的查詢
SELECT
    query,
    calls,
    total_time / calls as avg_time_ms
FROM pg_stat_statements
ORDER BY calls DESC
LIMIT 10;

-- 檢查索引使用情況
SELECT
    schemaname,
    tablename,
    indexname,
    idx_scan,
    idx_tup_read,
    idx_tup_fetch
FROM pg_stat_user_indexes
WHERE idx_scan = 0;

-- 檢查鎖定情況
SELECT
    blocked_locks.pid AS blocked_pid,
    blocked_activity.usename AS blocked_user,
    blocking_locks.pid AS blocking_pid,
    blocking_activity.usename AS blocking_user,
    blocked_activity.query AS blocked_statement
FROM pg_catalog.pg_locks blocked_locks
JOIN pg_catalog.pg_stat_activity blocked_activity ON blocked_activity.pid = blocked_locks.pid
JOIN pg_catalog.pg_locks blocking_locks ON blocking_locks.locktype = blocked_locks.locktype
JOIN pg_catalog.pg_stat_activity blocking_activity ON blocking_activity.pid = blocking_locks.pid
WHERE NOT blocked_locks.granted;
```

**3. 基礎設施瓶頸監控**

```yaml
# CloudWatch 自定義監控腳本
# AWS CLI 範例

# CPU 使用率監控
aws cloudwatch put-metric-data \
  --namespace "Performance/Test" \
  --metric-data MetricName=CPUUtilization,Value=85.5,Unit=Percent,Timestamp=2025-09-23T10:00:00Z

# 記憶體使用率監控
aws cloudwatch put-metric-data \
  --namespace "Performance/Test" \
  --metric-data MetricName=MemoryUtilization,Value=78.2,Unit=Percent,Timestamp=2025-09-23T10:00:00Z

# 網路吞吐量監控
aws cloudwatch put-metric-data \
  --namespace "Performance/Test" \
  --metric-data MetricName=NetworkIn,Value=1024000,Unit=Bytes,Timestamp=2025-09-23T10:00:00Z
```

**運用 AWS 服務進行主動優化（AWS 性能工具包）**

就像我們在<高併發與限流設計：如何避免資源瓶頸>與<料庫設計哲學:需求解析、技術選型與 Schema 設計策略>中提到的策略與實戰模擬，AWS 提供了一系列強大的服務，可以幫助我們系統性地診斷和解決性能瓶頸。

我們在這邊也是簡單的回憶一下有哪些常見的工具可以進行優化

**使用 Amazon RDS Performance Insights 進行資料庫調優**

- 功能：RDS Performance Insights 是一個資料庫性能監控功能，它能將資料庫負載視覺化，幫助使用者快速識別哪些 SQL 語句是導致高 CPU 或鎖等待等瓶頸的元兇 。
- 應用：這個工具是我們驗證「資料庫瓶頸」假設的主要武器。它的儀表板直接按負載大小列出排名最前的 SQL 查詢。我們不再需要猜測，而是有了一個精確的視覺化指南，告訴我們應該優化哪個查詢。它使得資料庫調優變得普及化，讓非 DBA 專家也能識別問題 。

**使用 Amazon ElastiCache 降低延遲**

- 功能：ElastiCache 是一個全託管的記憶體內快取服務（支持 Redis 或 Memcached 引擎），它能為數據訪問提供微秒級的延遲，從而分擔主資料庫的讀取請求負載 。 -應用：在我們優化了查詢之後，下一步是盡可能避免執行這些查詢。對於那些頻繁讀取但很少變更的數據（例如，使用者個人資料、商品目錄），我們可以利用 ElastiCache 實現一個快取層。常見的快取策略包括延遲載入（Lazy Loading）和穿透寫入（Write-Through） 。延遲載入是指應用程式先查詢快取，如果快取中沒有數據（快取未命中），再從資料庫讀取，並將結果寫入快取。穿透寫入則是在每次寫入資料庫的同時，也將數據寫入快取。這種策略能直接解決資料庫的讀取瓶頸，並顯著降低資料庫成本 。

**使用 Application Load Balancer (ALB) 進行高效流量管理**

- 功能：ALB 在 OSI 模型的第七層（應用層）運作，能夠基於請求內容（如 HTTP 標頭或 URL 路徑）智能地將傳入的 HTTP/HTTPS 流量分發到多個後端目標（如 EC2 實例、容器或 Lambda 函數）。它會執行健康檢查，並自動將流量從不健康的實例上轉移開 。 -應用：ALB 是我們的第一道防線。它透過分散負載來確保可擴展性，並透過處理故障轉移來提高彈性。我們可以根據請求路徑配置路由規則（例如，/api/_ 的請求轉到微服務 A，/images/_ 的請求轉到微服務 B），這對於微服務架構至關重要。在性能測試期間，其健康檢查功能尤為關鍵；如果某個伺服器在負載下開始失效，ALB 會將其從服務輪換中移除，從而防止級聯故障的發生。

## 建立性能預測模型與未來成本優化

最後的最後，有了既有的測試數據資料與成果後，我們來把性能測試產生的原始數據，轉化為一個對未來增長的預測模型，以及一個具體的、可操作的 AWS 雲端成本優化計劃。

這是我們從測量現在走向預測未來的關鍵一步，一次壓力測試的輸出給了我們一個至關重要的數據點：

> 「一個 m5.large 實例可以在響應時間超過我們的 500 毫秒 SLO 之前，處理 1,000 個並發使用者。」

有了這個資訊，我們就能回答關鍵的業務問題：

> 「市場部門預計新產品發布後將有 5,000 名使用者。我們需要多少基礎設施？」

答案是：至少需要 `5` 個 `m5.large` 實例。

「我們計劃進入歐洲市場，這將在明年使我們的使用者基礎翻倍。我們的基礎設施路線圖是什麼？」這個問題將觸發一個長期的容量規劃過程 。

整合<快取策略的哲學：時間、空間與一致性的權衡藝術>中的成本建模思維，我們就能建立性能測試的投資回報率分析框架。

### 性能優化的 ROI 計算框架

借鑑<快取策略的哲學：時間、空間與一致性的權衡藝術>的成本建模方法，我們將性能測試的投資轉化為量化的商業價值：

**實際案例：中型 SaaS 系統的性能測試 ROI**：

| 成本項目         | 金額 (USD/年) | 說明                    |
| ---------------- | ------------- | ----------------------- |
| **測試基礎設施** | $12,000       | K6 Cloud + AWS 測試環境 |
| **工程時間**     | $40,000       | 2 個月初期設置 + 維護   |
| **工具授權**     | $8,000        | 監控工具 + APM 平台     |
| **總成本**       | **$60,000**   | 第一年總投資            |

| 效益項目         | 價值 (USD/年) | 計算依據                       |
| ---------------- | ------------- | ------------------------------ |
| **避免宕機損失** | $87,600       | 99.5% → 99.9% 可用性提升       |
| **轉換率提升**   | $36,000       | 響應時間改善 15% → 轉換率+1.5% |
| **容量優化節省** | $24,000       | 精確容量規劃，避免過度配置     |
| **總效益**       | **$147,600**  | 年度商業價值                   |

**ROI 計算結果**：

- **ROI** = ($147,600 - $60,000) / $60,000 = **146%**
- **投資回收期** = $60,000 / ($147,600 / 12) = **4.9 個月**

**不同規模系統的成本效益分析**：

```yaml
# 小型系統 (< 10K 用戶)
SmallScale:
  投資: "$15K/年"
  效益: "$45K/年"
  ROI: "200%"

# 中型系統 (10K-100K 用戶)
MediumScale:
  投資: "$60K/年"
  效益: "$148K/年"
  ROI: "146%"

# 大型系統 (100K+ 用戶)
LargeScale:
  投資: "$200K/年"
  效益: "$650K/年"
  ROI: "225%"
```

### 將容量轉化為 AWS 雲端成本

我們的容量計劃（「我們需要 5 個 m5.large 實例」）現在可以直接輸入到 AWS 定價計算器中。我們可以預測出每月的帳單金額。這將性能工程師轉變為財務規劃和預算制定過程中的關鍵角色 。

然而，一個更為精密的分析揭示了性能測試與成本優化之間更深層次的聯繫：`工作負載的性能特徵決定了最優的 AWS 定價模型`。

不同類型的性能測試揭示了不同的工作負載特徵: `負載測試` 顯示了 **穩定、可預測** 的基線負載， `尖峰測試` 顯示了 **短暫、巨大且不可預測** 的突發負載， `浸泡測試` 則顯示了 **長時間、持續** 的負載。

AWS 為計算資源提供了不同的定價模型，每種模型都針對不同的使用模式進行了優化：

- 隨需（On-Demand）（按小時計費，靈活）
- 節省計劃（Savings Plans）/預留實例（Reserved Instances）（承諾使用 1-3 年以換取折扣，適用於穩定工作負載）
- 競價實例（Spot Instances）（對空閒計算容量出價以獲得巨大折扣，但可能被中斷）。

因此，工作負載的性能特徵與最具成本效益的 AWS 定價模型之間存在直接的對應關係：

- 穩定的基線負載（來自負載/浸泡測試）：這非常適合使用 AWS Savings Plans。我們知道這部分量是 24/7 都需要的，因此可以做出承諾以換取深度折扣。
- 可預測的每日尖峰（例如，辦公時間）：這可以透過 Savings Plans 覆蓋基礎負載，並結合自動擴展（Auto-Scaling）與隨需實例來處理尖峰流量。
- 負載產生器集群本身：用於運行大規模、短時壓力測試的基礎設施，是競價實例的完美應用場景。這項工作是容錯的（如果一個負載產生器被終止，另一個可以替代它）且是臨時的。這可以將測試成本降低高達 90%。

一個成熟的性能工程實踐不僅僅產出一個容量計劃，它還產出一個成本優化的採購策略。測試結果提供了所需的數據，讓我們能夠自信地選擇正確的定價模型組合，從而超越單純的隨需策略，實現高度優化的財務架構。

在 CI/CD 的背景下，我們同樣必須優化測試流程本身的成本，這包括最小化冗餘的測試任務、明智地使用並行化，以及將測試安排在非尖峰時段執行 。

同時，我們應及時清理測試過程中創建的臨時資源，以下是一些具體的、可操作的建議：

- 合理調整測試環境規模：非生產環境通常被過度配置。使用 Amazon CloudWatch 指標來分析其實際使用情況，並對其進行縮減 。
- 自動化關機：使用 AWS Instance Scheduler 或 Amazon EventBridge 等服務，設定非生產和測試環境在非工作時間自動關閉，這可以節省高達 70% 的成本 。
- 擁抱無伺服器進行測試：使用 AWS Fargate 或 Lambda 作為負載產生基礎設施。這符合基於消耗的模型，你只需為測試運行期間使用的計算時間付費，閒置時成本為零 。
- 實施生命週期策略：自動刪除 S3 中舊的測試結果和 ECR 中舊的容器映像，以防止儲存成本的無謂增長 。
- 監控與告警：使用 AWS Budgets 設定預算告警。當測試基礎設施的成本超過預定閾值時，你會收到通知，從而避免意外的帳單 。

最終，性能測試與負載壓力測試不只是技術驗證，更是**業務風險管理**和**容量規劃**的科學工具。

從戰略價值到成本優化，性能工程是一個完整的、跨學科的領域。它始於一個簡單的前提： `理解我們的系統在壓力下的行為`，但它的影響遠不止於此:

- 從技術到業務的橋樑：性能工程將抽象的技術指標（如延遲和吞吐量）轉化為具體的業務成果（如收入和客戶滿意度）
  - 它為技術投資提供了量化的商業理由。
- 從被動反應到主動預測：透過嚴謹的測試和建模，我們將系統的未來行為從一個未知數變為一個可以預測和規劃的變數。
  - 這使我們能夠為業務增長、市場活動和不可預見的流量高峰做好準備。
- 從成本中心到利潤驅動器：一個高效的性能優化策略不僅能提升使用者體驗，還能直接降低營運成本。
  - 透過將工作負載特徵與 AWS 的定價模型相匹配，性能測試數據成為實現雲端財務優化（FinOps）的關鍵輸入。

我們必須認識到，掌握性能工程不僅僅是學會使用幾種工具，它是要培養一種系統性的思維方式，一種將 **使用者** 、 **程式碼** 、 **基礎設施** 和 **業務目標** 視為一個相互關聯的整體的能力當我們能夠系統性地測量、分析和預測系統性能時，我們就能夠：

1. **主動預防性能問題**，而不是被動應對
2. **科學制定擴容策略**，避免過度投資或資源不足
3. **建立可預測的服務品質**，為業務決策提供技術保障
4. **持續優化系統架構**，在成本與性能之間找到最佳平衡點

正如<快取策略的哲學：時間、空間與一致性的權衡藝術>所強調的，**`技術決策必須通過 ROI 分析來驗證其商業價值`**。性能測試提供的數據是這種分析的基礎，讓我們能夠量化投資回報。

在當今雲原生的環境中，系統的性能表現直接影響用戶體驗和業務成果。一個擁有完整性能測試體系的團隊，能夠在快速增長的業務需求面前保持技術優勢，確保系統始終能夠支撐業務的成功。

> 關鍵要點：
>
> - **戰略定位**：將性能測試從功能驗證升級為容量規劃工具
> - **科學方法**：建立系統性的測試場景和指標體系，基於<{# Day 2-2 | 需求確認 × 系統設計起點(二)：領域邊界與基礎需求確認}>的用戶行為分析
> - **工具選擇**：根據團隊特性選擇合適的測試工具
> - **數據分析**：用統計方法深度分析性能數據
> - **預測建模**：建立數學模型預測未來的性能需求
> - **成本效益**：整合<{# Day 10 | 快取策略的哲學：時間、空間與一致性的權衡藝術}>的 ROI 思維進行投資決策
>
> ### **性能測試的目標不是找出系統能承受多少負載，而是建立可預測、可管理的服務品質基準。**
