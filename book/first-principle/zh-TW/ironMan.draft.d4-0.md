# Day 4| 用 DDD 建構業務邏輯：從用例到聚合設計

昨天我們完成了抽象建模的四重境界，從概念、行為、資料、架構四個維度理解了系統的本質。但當我們要將這些抽象理解轉化為可執行的業務邏輯時，面臨一個關鍵挑戰：

**如何將領域概念組織成內聚且有意義的業務單元？**

這就是 DDD 實踐的核心問題：**從用例分析到聚合設計的系統化轉換**。今天我們將建立領域模型的靜態結構，為明天深入探討使用者操作情境做好基礎。

## 用例分析：從意圖到邊界的識別

### 重新審視 Day 2 的六個案例

在 Day 2 我們分析了六個典型用戶案例，每個案例背後都有其**核心用例模式**。讓我們從 DDD 的角度重新解構：

**投資交易系統 [I-N-T-J] - 控制型用例模式**

```
核心用例：執行即時交易決策
Primary Actor: 專業交易者
Goal in Context: 在最短時間內完成高質量交易
Scope: 個人投資組合管理系統
Level: 用戶目標級別

主要成功場景：
1. 交易者監控市場數據
2. 識別交易機會
3. 計算風險影響
4. 提交交易指令
5. 確認交易執行
6. 更新投資組合狀態

延伸場景：
2a. 市場數據延遲 → 系統提供延遲警告
3a. 風險超過限額 → 系統要求明確確認
4a. 資金不足 → 系統建議調整或融資
5a. 交易失敗 → 系統進入重試或人工處理
```

從這個用例分析中，我們可以識別出幾個**核心領域概念**：

- **交易者** (Trader) - 有決策能力的主體
- **投資組合** (Portfolio) - 資產配置的狀態容器
- **交易指令** (TradeOrder) - 交易意圖的結構化表達
- **市場數據** (MarketData) - 外部價格信息來源
- **風險評估** (RiskAssessment) - 決策支撐的計算邏輯

**家庭財務系統 [E-N-T-J] - 協調型用例模式**

```
核心用例：管理家庭協作支出
Primary Actor: 家庭財務負責人
Goal in Context: 協調家庭成員的支出行為，維持預算平衡
Scope: 家庭財務管理系統
Level: 業務摘要級別

主要成功場景：
1. 設定家庭預算與權限
2. 家庭成員記錄支出
3. 系統檢查預算狀況
4. 觸發預算警告或調整
5. 生成家庭財務報告

角色交互複雜度：
- 主要決策者：設定規則，監控整體
- 一般成員：執行支出，遵守規則
- 系統協調：確保規則執行，提供透明度
```

領域概念識別：

- **家庭** (Family) - 協作的邊界和規則制定者
- **家庭成員** (FamilyMember) - 有特定權限的參與者
- **預算** (Budget) - 資源分配的約束規則
- **支出記錄** (Expense) - 資源消耗的事實記錄
- **權限管理** (PermissionManagement) - 協作行為的控制機制

### 用例模式的領域特徵

通過六個案例的對比分析，我們發現不同的用例模式對應不同的**領域複雜度特徵**：

| 用例模式          | 主要複雜度 | 核心挑戰     | 設計重點   |
| ----------------- | ---------- | ------------ | ---------- |
| 控制型 (投資交易) | 時間敏感性 | 毫秒級決策   | 狀態一致性 |
| 協調型 (家庭財務) | 多角色協作 | 權限與透明度 | 角色邊界   |
| 確定型 (健康監控) | 數據可靠性 | 連續性監控   | 數據完整性 |
| 適應型 (智慧家居) | 彈性響應   | 漸進式控制   | 優雅降級   |
| 透明型 (包裹追蹤) | 信息同步   | 多源數據整合 | 狀態同步   |
| 創造型 (旅遊分享) | 表達靈活性 | 創意支持工具 | 內容流動性 |

每種模式都會影響我們的聚合設計策略。

## 聚合識別：業務邊界的劃分藝術

### 聚合識別的三個原則

**原則 1：業務不變式邊界**
聚合的邊界應該對應業務中必須保持一致性的概念集合。

**投資交易系統的不變式分析**：

```
Portfolio聚合的不變式：
- 總資產價值 = Σ(持倉價值) + 現金餘額
- 任何交易不能導致資金為負
- 風險敞口不能超過預設限額

Order聚合的不變式：
- 訂單一旦提交不能修改核心參數
- 執行記錄必須與原始指令對應
- 狀態轉換必須符合業務流程
```

**原則 2：事務邊界對應**
一個聚合內的修改應該能在單一事務中完成。

**家庭財務系統的事務分析**：

```
Family聚合：
✓ 可在單一事務中完成：
  - 設定預算額度
  - 調整成員權限
  - 更新家庭基本信息

✗ 不能在單一事務中完成：
  - 統計所有成員的月度支出 (跨多個聚合)
  - 生成年度報告 (數據量太大)
```

**原則 3：業務語言對應**
聚合的命名和結構應該直接對應業務專家的語言。

### 六個案例的聚合設計對比

**案例 1：投資交易系統**

```
聚合設計：
┌─────────────────┐    ┌─────────────────┐
│   Portfolio     │    │      Order      │
│  (投資組合)     │    │    (交易訂單)   │
├─────────────────┤    ├─────────────────┤
│ - portfolioId   │    │ - orderId       │
│ - traderId      │    │ - portfolioId   │
│ - holdings[]    │    │ - tradeOrder    │
│ - cashBalance   │    │ - status        │
│ - riskLimit     │    │ - execution[]   │
└─────────────────┘    └─────────────────┘
        │                       │
        └───────────────────────┘
             領域事件通信

設計邏輯：
- Portfolio管理資產狀態和風險控制
- Order管理交易流程和執行追蹤
- 兩者通過事件保持最終一致性
```

**案例 2：家庭財務系統**

```
聚合設計：
┌─────────────────┐    ┌─────────────────┐
│     Family      │    │     Budget      │
│    (家庭)       │    │    (預算)       │
├─────────────────┤    ├─────────────────┤
│ - familyId      │    │ - budgetId      │
│ - members[]     │    │ - familyId      │
│ - permissions   │    │ - categories[]  │
│ - settings      │    │ - limits        │
└─────────────────┘    │ - period        │
                       └─────────────────┘
              │                 │
              └─────────────────┘
                事件驅動協調

┌─────────────────┐
│    Expense      │
│   (支出記錄)    │
├─────────────────┤
│ - expenseId     │
│ - familyId      │
│ - memberId      │
│ - amount        │
│ - category      │
│ - timestamp     │
└─────────────────┘

設計邏輯：
- Family管理成員和權限結構
- Budget管理預算規則和限額
- Expense記錄實際支出事實
- 三者協作實現預算控制
```

**案例 3：健康監控系統**

```
聚合設計：
┌─────────────────┐    ┌─────────────────┐
│   HealthProfile │    │  DeviceReading  │
│   (健康檔案)    │    │   (設備數據)    │
├─────────────────┤    ├─────────────────┤
│ - userId        │    │ - readingId     │
│ - metrics[]     │    │ - userId        │
│ - thresholds    │    │ - deviceType    │
│ - history       │    │ - value         │
└─────────────────┘    │ - timestamp     │
                       │ - status        │
                       └─────────────────┘

設計邏輯：
- HealthProfile管理長期健康狀態
- DeviceReading管理即時數據收集
- 分離設計支持不同的數據生命週期
```

### 聚合間的交互模式

不同的領域會產生不同的聚合交互模式：

**模式 1：事件驅動協調 (投資交易)**

```
Portfolio聚合 → TradeValidatedEvent → Order聚合
Order聚合 → OrderExecutedEvent → Portfolio聚合
Portfolio聚合 → PortfolioUpdatedEvent → RiskManagement聚合
```

**模式 2：權限驅動協作 (家庭財務)**

```
Family聚合 → 設定權限規則 → Expense聚合檢查權限
Budget聚合 → 設定限額規則 → Expense聚合檢查限額
Expense聚合 → 記錄支出 → Budget聚合更新狀態
```

**模式 3：數據驅動聚合 (健康監控)**

```
Device → 推送數據 → DeviceReading聚合
DeviceReading聚合 → 數據驗證完成 → HealthProfile聚合
HealthProfile聚合 → 閾值檢查 → Alert聚合
```

## 領域服務：跨聚合的業務邏輯

### 識別領域服務的時機

當業務邏輯**不自然地屬於任何單一聚合**時，我們需要領域服務。

**投資交易系統的領域服務案例**：

```typescript
// 風險計算服務：需要Portfolio和MarketData的協作
class RiskCalculationService {
  calculate(portfolio: Portfolio, marketData: MarketData): RiskMetric {
    // 這個邏輯不屬於Portfolio（因為需要外部市場數據）
    // 也不屬於MarketData（因為需要組合特定信息）

    const portfolioValue = portfolio.getTotalValue(marketData);
    const volatility = marketData.calculateVolatility(portfolio.getSymbols());
    const concentration = portfolio.calculateConcentration();

    return new RiskMetric(portfolioValue, volatility, concentration);
  }
}

// 交易執行協調服務：協調Portfolio和Order聚合
class TradeExecutionService {
  async executeTrade(
    portfolio: Portfolio,
    tradeOrder: TradeOrder
  ): Promise<ExecutionResult> {
    // 1. 驗證Portfolio的交易能力
    const validation = portfolio.validateTrade(tradeOrder);
    if (!validation.isValid) {
      return ExecutionResult.rejected(validation.reason);
    }

    // 2. 創建Order聚合管理執行過程
    const order = Order.create(portfolio.getId(), tradeOrder);

    // 3. 提交到市場（外部服務）
    const marketResult = await this.marketService.submit(order);

    // 4. 更新相關聚合狀態
    if (marketResult.isSuccessful) {
      portfolio.applyExecution(marketResult);
      order.markAsExecuted(marketResult);
    }

    return ExecutionResult.fromMarket(marketResult);
  }
}
```

### 領域服務的設計原則

**無狀態性**：領域服務本身不維護狀態，只協調聚合間的交互

**業務導向**：服務的命名和接口應該反映業務概念，而非技術實現

**最小職責**：每個服務只處理一個明確的業務場景

## 為使用者操作情境做準備

### 從領域模型到使用者角色

今天建立的聚合設計，將直接影響明天我們設計使用者操作情境：

**角色與聚合的對應關係**：

```
投資交易系統：
- 交易者角色 → Portfolio聚合的所有者
- 系統管理員 → 整體系統的監控者
- 風控專員 → RiskManagement聚合的操作者

家庭財務系統：
- 家庭負責人 → Family聚合的管理者
- 家庭成員 → Expense聚合的創建者
- 預算管理員 → Budget聚合的維護者
```

**操作權限的聚合邊界**：

- 每個聚合定義了特定角色的操作邊界
- 跨聚合的操作需要通過領域服務協調
- 複雜的業務流程會產生多步驟的使用者故事

### 狀態場景的準備

每個聚合的狀態變化，都會產生對應的使用者操作場景：

**Portfolio 聚合的狀態轉換**：

```
空組合 → [初始化資金] → 活躍組合
活躍組合 → [執行交易] → 活躍組合 (狀態更新)
活躍組合 → [風險超限] → 受限組合
受限組合 → [風險調整] → 活躍組合
活躍組合 → [全部賣出] → 空組合
```

每個狀態轉換都對應一個使用者操作情境，明天我們將深入探討如何將這些轉換設計成直觀的 User Story。

## 設計決策的記錄

在進行聚合設計時，我們做出了許多重要決策。記錄這些決策對後續的 User Story 設計至關重要：

**決策 1：Portfolio 和 Order 分離**

- 理由：Portfolio 關注資產狀態，Order 關注交易流程
- 影響：使用者需要在不同界面查看資產狀態和交易狀態
- User Story 影響：需要設計跨聚合的操作流程

**決策 2：Family 聚合包含權限管理**

- 理由：權限設定與家庭結構緊密相關
- 影響：權限變更會觸發 Family 聚合的狀態變化
- User Story 影響：需要設計權限管理的使用者界面

**決策 3：健康數據採用讀寫分離**

- 理由：讀取頻率和寫入頻率差異巨大
- 影響：即時查詢和歷史分析使用不同的數據源
- User Story 影響：需要考慮數據同步的時間延遲

## 明天的橋接預告

通過今天的聚合設計，我們建立了業務邏輯的靜態結構。明天我們將深入探討：

1. **User Story 的編寫技巧**：如何將聚合的業務能力轉化為使用者故事
2. **角色權限設計**：如何基於聚合邊界設計角色和權限體系
3. **操作情境流程**：如何設計跨聚合的複雜業務流程
4. **狀態場景管理**：如何處理聚合狀態變化帶來的使用者體驗挑戰

## 今日的概念收穫

- **聚合是業務不變式的邊界**：技術實現必須服務於業務規則
- **領域服務協調跨聚合邏輯**：複雜業務流程的組織方式
- **不同領域有不同的聚合模式**：投資、家庭、健康各有特色
- **聚合設計直接影響使用者體驗**：好的領域模型是好 UX 的基礎

記住：我們今天建立的不是技術架構，而是業務邏輯的組織方式。這個組織方式將直接決定明天我們如何設計使用者的操作體驗。

---

> 「聚合不是代碼的分組，而是業務概念的生存單位。每個聚合都有其獨特的生命週期，而使用者的每個操作都是在參與這個生命週期的演進。」
