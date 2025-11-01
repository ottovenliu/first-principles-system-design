# Day 3 | 需求萃取方法論-抽象建模(Abstract Modeling)

前三天我們完成了一個重要的認知轉換：從用戶的模糊表達中萃取出功能需求的本質，再從功能需求推導出具體的技術邊界。我們知道了系統要做什麼，也知道了要做到什麼程度。

但現在面臨一個更深層的挑戰：**如何將這些理解轉化為可以實現的系統設計？**

當我們知道投資交易系統需要支撐 2000 RPS、延遲低於 100ms、RPO 為零時，具體應該設計什麼樣的概念模型？什麼樣的行為流程？什麼樣的數據結構？什麼樣的架構模式？

這就是**抽象建模**要解決的核心問題：**從需求理解到系統設計的系統化轉換方法。**

## 抽象建模的四重境界

抽象建模不是單一的技術方法，而是一個**從四個不同維度理解同一個系統**的哲學框架。就像我們無法用一個視角完全理解複雜的現實，我們也無法用單一模型完全描述一個系統的本質。

```
                抽象建模
                   |
┌────────┬─────────────┬────────────┬────────────┐
| 概念建模 | 行為建模      | 資料建模     | 架構建模     |
|---------|--------------|-------------|-------------|
| DDD     | Use Case     | ER Model    | C4 Model    |
| Ontology| EventStorming| NoSQL Schema| 4+1 View    |
| ORM     | State Machine| Normalization| Clean Arch |
```

### 四個維度的本體論意義

每個維度回答了系統設計中的一個根本問題：

- **概念建模**：這個系統的本質是什麼？（What IS it?）
- **行為建模**：這個系統會做什麼？（What DOES it do?）
- **資料建模**：這個系統記住什麼？（What does it REMEMBER?）
- **架構建模**：這個系統如何組織？（How is it STRUCTURED?）

從前面的分析中，我們已經獲得了四個維度的設計約束：

**需求分析 → 建模約束的對應關係**：

- 用戶認知模式 → 概念建模的抽象層次
- 業務事件流程 → 行為建模的時序邏輯
- RPS 和容量需求 → 資料建模的存儲策略
- 可用性和恢復需求 → 架構建模的韌性設計

## 第一重境界：概念建模 - 系統的存在論

### DDD：從抽象概念到靜態結構

概念建模的核心問題是：**在去除所有技術細節後，這個系統代表什麼現實？**

但更關鍵的是：**如何將這些抽象概念轉化為可以用程式語言表達的靜態結構？**

回到投資交易案例，從 Day 2 的 INTJ 用戶分析中，我們理解了核心需求是「對時間的極致控制」。這個抽象理解需要轉化為具體的領域概念：

**從抽象概念到領域實體的轉換**：

```
抽象概念層：
"對時間的極致控制" → 具體表現為什麼？

業務概念層：
- 決策主體：誰在控制時間？ → Trader（交易者）
- 控制對象：控制什麼？ → Portfolio（投資組合）
- 控制工具：如何控制？ → TradeOrder（交易指令）
- 外部依賴：時間壓力來源？ → MarketData（市場數據）

領域實體層：
每個業務概念對應的程式概念：
- Trader → Entity（有身份的主體）
- Portfolio → Aggregate Root（聚合根）
- Holding → Entity（組合內實體）
- TradeOrder → Value Object（值對象）
- Price → Value Object（不可變值）
```

**領域實體的靜態結構設計**：

```
交易者（Trader）- Entity
屬性：
- traderId: String（唯一標識）
- riskProfile: RiskProfile（風險偏好）
- tradingPreferences: TradingPreferences（交易偏好）

職責：
- 創建投資組合
- 設定風險限制
- 下達交易指令

不變式：
- 交易者ID一旦創建不可變更
- 風險偏好必須在系統允許範圍內

投資組合（Portfolio）- Aggregate Root
屬性：
- portfolioId: String（唯一標識）
- traderId: String（所有者）
- holdings: List<Holding>（持倉列表）
- totalValue: Money（總價值）
- lastUpdated: DateTime（最後更新時間）

職責：
- 管理所有持倉
- 計算總體風險
- 執行交易操作
- 維護數據一致性

聚合邊界：
- Portfolio是聚合根
- Holding只能通過Portfolio訪問
- 同一Portfolio內的操作必須保持一致性

不變式：
- 總價值 = Σ(持倉數量 × 當前價格)
- 現金持倉不能為負數
- 風險指標不能超過設定限制
```

**概念間的關係設計**：

```
關係類型分析：
1. 所有權關係（Ownership）
   Trader --owns--> Portfolio
   - 一對多關係
   - 強所有權，Portfolio不能獨立存在
   - 級聯刪除：Trader刪除時Portfolio也刪除

2. 組合關係（Composition）
   Portfolio --contains--> Holding
   - 一對多關係
   - 強組合，Holding是Portfolio的組成部分
   - 生命週期綁定：Portfolio刪除時Holding也刪除

3. 引用關係（Reference）
   Holding --references--> Asset
   - 多對一關係
   - 弱引用，Asset獨立存在
   - Asset是系統外部概念，由市場定義

4. 歷史關係（History）
   Portfolio --records--> TradeTransaction
   - 一對多關係
   - 時間序列關係，只增不減
   - 用於審計和分析
```

這些關係設計直接為明天的類圖設計奠定基礎。每種關係類型都有其特定的實現方式和約束條件。

### Ontology：概念間的存在依賴

更深層的概念建模需要理解：**這些概念之間的存在依賴關係是什麼？**

**存在依賴的層次結構**：

```
獨立存在層（Independent Existence）：
- Trader：業務主體，可以獨立存在
- Asset：市場概念，外部定義，系統外獨立存在
- MarketData：外部事件流，獨立於我們的系統

依存存在層（Dependent Existence）：
- Portfolio：依賴Trader而存在，但有自己的生命週期
- Holding：依賴Portfolio而存在，沒有獨立意義
- TradeOrder：依賴Portfolio而存在，表達交易意圖

關係存在層（Relational Existence）：
- Price：Asset與時間的關係
- Risk：Portfolio與Market的關係
- Profit/Loss：時間序列上的價值變化關係
```

**存在依賴的實現策略**：

```python
# 獨立存在：可以單獨創建和管理
class Trader:
    def __init__(self, trader_id: str):
        self.trader_id = trader_id
        self.portfolios = {}  # 管理但不擁有生命週期

    def create_portfolio(self) -> Portfolio:
        portfolio = Portfolio(self.trader_id, generate_id())
        self.portfolios[portfolio.id] = portfolio
        return portfolio

# 依存存在：必須通過聚合根創建
class Portfolio:
    def __init__(self, trader_id: str, portfolio_id: str):
        self.trader_id = trader_id  # 存在依賴
        self.portfolio_id = portfolio_id
        self.holdings = {}

    def add_holding(self, symbol: str, quantity: int) -> None:
        # Holding只能通過Portfolio創建
        holding = Holding(self.portfolio_id, symbol, quantity)
        self.holdings[symbol] = holding

    def remove_holding(self, symbol: str) -> None:
        # 控制Holding的生命週期
        if symbol in self.holdings:
            del self.holdings[symbol]

# 關係存在：計算得出，不獨立存儲
class PortfolioAnalyzer:
    def calculate_risk(self, portfolio: Portfolio, market_data: MarketData) -> RiskMetrics:
        # Risk是Portfolio和Market的關係，通過計算得出
        return RiskMetrics(
            volatility=self._calculate_volatility(portfolio, market_data),
            var=self._calculate_var(portfolio, market_data)
        )
```

這種存在依賴分析為明天的聚合設計和類間交互設計提供了基礎。

## 第二重境界：行為建模 - 概念間的交互

### 行為作為概念間的協作

行為建模的核心洞察是：**系統的行為不是單個對象的行為，而是多個概念之間的協作交互。**

這個認知為明天的序列圖設計奠定了重要基礎：每個用例都是一系列概念之間的消息傳遞和狀態變更。

**投資交易用例的概念交互分析**：

```
Use Case: 執行快速交易
參與概念: Trader, Portfolio, Holding, MarketData, TradeOrder

交互序列：
1. Trader.createTradeIntent(symbol, quantity)
   - Trader內部驗證交易意圖
   - 創建TradeIntent值對象

2. Portfolio.validateTrade(tradeIntent)
   - Portfolio檢查資金充足性
   - Portfolio檢查風險限制
   - 調用RiskCalculator.assess(portfolio, tradeIntent)

3. MarketData.getCurrentPrice(symbol)
   - 獲取實時市場價格
   - 返回Price值對象

4. Portfolio.executeTradeOrder(tradeOrder)
   - 創建TradeOrder
   - 更新相應的Holding
   - 記錄TradeTransaction
   - 觸發PortfolioUpdated事件

5. NotificationService.notify(trader, tradeResult)
   - 外部服務處理通知
```

**概念交互的設計原則**：

```
1. 依賴方向控制：
   - 高層概念可以依賴低層概念
   - 聚合根可以依賴聚合內實體
   - 領域服務可以協調多個聚合

2. 消息傳遞模式：
   - 同步調用：同一聚合內的操作
   - 事件發布：跨聚合的通信
   - 命令模式：複雜業務流程

3. 狀態變更控制：
   - 只有聚合根可以修改聚合內狀態
   - 跨聚合狀態變更通過事件實現
   - 所有狀態變更必須維護不變式
```

### EventStorming：概念交互的時序建模

EventStorming 不僅是需求分析工具，更是概念交互設計的建模方法：

**領域事件驅動的概念交互**：

```
事件流中的概念角色：
TradeIntentCreated
- 觸發者：Trader（創建交易意圖）
- 參與者：Portfolio（接收並驗證）
- 結果：FundsValidationRequested

FundsValidated
- 觸發者：Portfolio（完成資金檢查）
- 參與者：RiskCalculator（風險評估服務）
- 結果：RiskAssessmentRequested

RiskAssessed
- 觸發者：RiskCalculator（完成風險評估）
- 參與者：Portfolio（決策執行）
- 結果：TradeOrderCreated 或 TradeRejected

TradeOrderSubmitted
- 觸發者：Portfolio（提交訂單）
- 參與者：MarketGateway（外部市場接口）
- 結果：MarketExecutionRequested

MarketExecutionConfirmed
- 觸發者：MarketGateway（市場確認）
- 參與者：Portfolio（更新狀態）
- 結果：HoldingUpdated, TransactionRecorded

PortfolioUpdated
- 觸發者：Portfolio（狀態已更新）
- 參與者：NotificationService（通知服務）
- 結果：TraderNotified
```

每個事件都代表概念之間的一次重要交互，這為明天的序列圖提供了詳細的交互腳本。

### State Machine：概念狀態的協調

狀態機不只是單個對象的狀態變化，更重要的是多個概念之間的狀態協調：

**TradeOrder 的狀態變化與概念協調**：

```
狀態轉換中的概念協作：

Created → Validating:
- Portfolio啟動驗證流程
- RiskCalculator參與風險計算
- MarketData提供價格信息

Validating → Approved:
- Portfolio確認所有驗證通過
- TradeOrder狀態更新為Approved
- Portfolio準備提交到市場

Approved → Submitted:
- Portfolio調用MarketGateway
- TradeOrder狀態更新為Submitted
- 建立與外部系統的關聯

Submitted → Executed:
- MarketGateway接收市場確認
- Portfolio更新Holding狀態
- TradeOrder標記為Executed
- 觸發後續通知流程
```

這種多概念協調的狀態機設計，為明天的系統設計圖奠定了基礎。

## 第三重境界：資料建模 - 概念的持久化

### 從領域概念到數據結構

資料建模的關鍵是：**如何將概念模型中的實體關係忠實地映射到數據存儲結構？**

**DynamoDB 的聚合存儲設計**：

```
聚合根導向的分區策略：
PK (Partition Key) | SK (Sort Key)        | 實體類型
TRADER#123        | PROFILE              | Trader
TRADER#123        | PORTFOLIO#001        | Portfolio
TRADER#123        | HOLDING#001#AAPL     | Holding
TRADER#123        | TRADE#001#20240101   | TradeTransaction

查詢模式與概念模型對應：
1. 獲取Trader的所有Portfolio:
   PK = TRADER#123, begins_with(SK, "PORTFOLIO")

2. 獲取Portfolio的所有Holding:
   PK = TRADER#123, begins_with(SK, "HOLDING#001")

3. Portfolio聚合的完整重構:
   PK = TRADER#123, SK between "PORTFOLIO#001" and "PORTFOLIO#001~"

一致性保證與聚合邊界對應：
- 同一Partition內操作 = 同一聚合內操作
- DynamoDB Transaction = 跨聚合操作
- Event發布 = 聚合間通信
```

### 概念關係的數據實現

不同的概念關係需要不同的數據實現策略：

```python
# 所有權關係：嵌入式設計
{
    "PK": "TRADER#123",
    "SK": "PORTFOLIO#001",
    "ownerId": "TRADER#123",  # 所有權標識
    "portfolioData": {
        "totalValue": 100000,
        "createdAt": "2024-01-01",
        "riskProfile": "MODERATE"
    }
}

# 組合關係：同partition內的多記錄
{
    "PK": "TRADER#123",
    "SK": "HOLDING#001#AAPL",
    "portfolioId": "PORTFOLIO#001",  # 組合關係標識
    "symbol": "AAPL",
    "quantity": 100,
    "avgPrice": 150.50
}

# 引用關係：外鍵 + 緩存
{
    "PK": "TRADER#123",
    "SK": "HOLDING#001#AAPL",
    "assetSymbol": "AAPL",  # 引用外部Asset
    # Asset詳細信息通過ElastiCache緩存
}

# 歷史關係：時間序列存儲
{
    "PK": "TRADER#123",
    "SK": "TRADE#001#2024-01-01T10:30:00",
    "portfolioId": "PORTFOLIO#001",
    "tradeType": "BUY",
    "symbol": "AAPL",
    "quantity": 100,
    "price": 150.50,
    "timestamp": "2024-01-01T10:30:00Z"
}
```

## 第四重境界：架構建模 - 概念的技術映射

### AWS 服務與領域概念的映射

每個 AWS 服務都應該對應到特定的領域概念或職責：

**服務映射策略**：

```
領域概念 → AWS服務 → 架構職責

Trader身份管理 → Cognito User Pool → 認證授權
Portfolio聚合 → DynamoDB + Lambda → 狀態管理
MarketData流 → Kinesis Data Streams → 外部事件接入
TradeOrder處理 → Step Functions → 複雜業務流程
領域事件發布 → EventBridge → 概念間通信
價格計算緩存 → ElastiCache → 性能優化
交易通知 → SNS + SES → 外部集成
```

### Clean Architecture 中的概念分層

```python
# 領域層：純粹的概念模型
class Portfolio:  # 領域概念
    def execute_trade(self, trade_order: TradeOrder) -> TradeResult:
        # 純粹的業務邏輯，不依賴任何技術實現
        pass

# 應用層：概念協調
class ExecuteTradeUseCase:  # 概念間的協作
    def __init__(self, portfolio_repo: PortfolioRepository,
                 market_gateway: MarketGateway):
        self.portfolio_repo = portfolio_repo
        self.market_gateway = market_gateway

    def execute(self, trade_intent: TradeIntent) -> TradeResult:
        # 協調Portfolio, MarketGateway等概念的交互
        portfolio = self.portfolio_repo.find_by_id(trade_intent.portfolio_id)
        return portfolio.execute_trade(trade_intent.to_trade_order())

# 基礎設施層：AWS技術實現
class DynamoDBPortfolioRepository(PortfolioRepository):  # 概念持久化
    def save(self, portfolio: Portfolio) -> None:
        # 將領域概念映射到DynamoDB
        pass

class KinesisMarketGateway(MarketGateway):  # 外部概念接入
    def submit_order(self, order: TradeOrder) -> OrderStatus:
        # 通過Kinesis與外部市場交互
        pass
```

## 為明天的 DDD 深化做準備

通過今天的抽象建模，我們建立了系統設計的四重基礎：

**概念建模**建立了領域實體和它們的關係
**行為建模**定義了概念間的交互模式
**資料建模**設計了概念的持久化策略  
**架構建模**規劃了概念的技術實現

但這些還只是抽象的框架。明天我們將深入 DDD 的具體實踐，學習：

- **如何將今天的抽象概念轉化為具體的類圖設計？**
- **概念間的交互如何用序列圖精確描述？**
- **聚合設計的邊界如何確定和實現？**
- **從用例到聚合的完整設計流程是什麼？**

特別是，今天我們理解了「行為是不同概念間的交互」這個核心洞察。明天我們將具體學習如何設計這些交互，如何確保概念間的協作既滿足業務需求，又符合技術約束。

## 今日的建模哲學總結

- **抽象建模是理解系統本質的四重透鏡**
- **概念模型必須為具體實現提供清晰指導**
- **行為本質上是概念間的協作交互**
- **每個抽象層次都為下一層實現奠定基礎**

記住：我們不是在建模技術，而是在建模現實。每個概念都有其存在的理由，每個交互都有其業務的意義。明天我們將看到這些抽象概念如何在 DDD 的指導下變為可執行的系統。

---

> 「概念的交互構成了系統的行為。我們不是在設計對象，而是在設計對象之間的關係。」
