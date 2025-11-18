# Day 3 | 要件抽出方法論 - 抽象モデリング

最初の3日間で、私たちは重要な認識の転換を完了しました。ユーザーの曖昧な表現から機能要件の本質を抽出し、そして機能要件から具体的な技術的境界を導き出しました。システムが何をすべきか、そしてどの程度までやるべきかが分かりました。

しかし今、私たちはより深い課題に直面しています：**これらの理解を達成可能なシステム設計に変換するにはどうすればよいか？**

投資取引システムが2000 RPS、100ms未満のレイテンシ、RPOゼロをサポートする必要があることが分かった時、どのような概念モデル、動作フロー、データ構造、アーキテクチャパターンを設計すべきでしょうか？

これが**抽象モデリング**が解決しようとする中核的な問題です：**要件理解からシステム設計への体系的な変換方法。**

## 抽象モデリングの4つの領域

抽象モデリングは単一の技術的手法ではなく、**4つの異なる次元から同じシステムを理解する**ための哲学的フレームワークです。単一の視点から複雑な現実を完全に理解できないのと同じように、単一のモデルでシステムの本質を完全に記述することはできません。

```
                抽象モデリング
                   |
┌────────┬─────────────┬────────────┬────────────┐
| 概念モデリング | 振る舞いモデリング | データモデリング | アーキテクチャモデリング |
|---------|--------------|-------------|-------------|
| DDD     | ユースケース     | ERモデル    | C4モデル    |
| オントロジー| EventStorming| NoSQLスキーマ| 4+1ビュー    |
| ORM     | ステートマシン| 正規化| クリーンアーキテクチャ |
```

### 4つの次元の存在論的意義

各次元はシステム設計における基本的な問いに答えます：

- **概念モデリング**：それは何であるか？（このシステムの本質は何か？）
- **振る舞いモデリング**：それは何をするか？（このシステムは何をするか？）
- **データモデリング**：それは何を記憶するか？（このシステムは何を記憶するか？）
- **アーキテクチャモデリング**：それはどう構造化されているか？（このシステムはどのように組織化されているか？）

前の分析から、4つの次元の設計制約を得ました：

**要件分析とモデリング制約の対応関係**：

- ユーザー認知モデル → 概念モデリングの抽象レベル
- ビジネスイベントフロー → 振る舞いモデリングの時間的ロジック
- RPSと容量要件 → データモデリングのストレージ戦略
- 可用性と復旧要件 → アーキテクチャモデリングのレジリエンス設計

## 第一の領域：概念モデリング - システムの存在論

### DDD：抽象概念から静的構造へ

概念モデリングの中核的な問いは：**すべての技術的詳細を取り除いた後、このシステムはどのような現実を表しているか？**

しかしより重要なのは：**これらの抽象概念をプログラミング言語で表現できる静的構造に変換するにはどうすればよいか？**

投資取引のケースに戻ると、Day 2のINTJユーザー分析から、中核的な要件は「時間の究極的なコントロール」であることを理解しました。この抽象的な理解を具体的なドメイン概念に変換する必要があります：

**抽象概念からドメインエンティティへの変換**：

```
抽象概念レイヤー：
「時間の究極的なコントロール」 → その具体的な現れは何か？

ビジネス概念レイヤー：
- 意思決定主体：誰が時間をコントロールしているか？ → トレーダー
- コントロール対象：何がコントロールされているか？ → ポートフォリオ
- コントロールツール：どのようにコントロールするか？ → TradeOrder
- 外部依存：時間的プレッシャーはどこから来るか？ → MarketData

ドメインエンティティレイヤー：
各ビジネス概念に対応するプログラミング概念：
- Trader → エンティティ（アイデンティティを持つ主体）
- Portfolio → 集約ルート
- Holding → エンティティ（集約内のエンティティ）
- TradeOrder → 値オブジェクト
- Price → 値オブジェクト（不変の値）
```

**ドメインエンティティの静的構造設計**：

```
Trader - エンティティ
属性：
- traderId: String（一意識別子）
- riskProfile: RiskProfile
- tradingPreferences: TradingPreferences

責任：
- ポートフォリオの作成
- リスク制限の設定
- 取引注文の発注

不変条件：
- トレーダーIDは一度作成されたら変更できない
- リスクプロファイルはシステムの許容範囲内でなければならない

Portfolio - 集約ルート
属性：
- portfolioId: String（一意識別子）
- traderId: String（所有者）
- holdings: List<Holding>
- totalValue: Money
- lastUpdated: DateTime

責任：
- すべての保有株式を管理
- 全体的なリスクを計算
- 取引操作を実行
- データの整合性を維持

集約境界：
- Portfolioは集約ルート
- Holdingはポートフォリオを通じてのみアクセス可能
- 同じPortfolio内の操作は一貫性を保たなければならない

不変条件：
- 総価値 = Σ（保有数量 × 現在価格）
- 現金保有はマイナスにならない
- リスク指標は設定された制限を超えてはならない
```

**概念間の関係の設計**：

```
関係タイプの分析：
1. 所有関係
   Trader --owns--> Portfolio
   - 1対多の関係
   - 強い所有権、Portfolioは独立して存在できない
   - カスケード削除：Traderが削除されるとPortfolioも削除される

2. 構成関係
   Portfolio --contains--> Holding
   - 1対多の関係
   - 強い構成、HoldingはPortfolioの構成要素
   - ライフサイクルバインディング：Portfolioが削除されるとHoldingも削除される

3. 参照関係
   Holding --references--> Asset
   - 多対1の関係
   - 弱い参照、Assetは独立して存在
   - Assetは市場によって定義された外部概念

4. 履歴関係
   Portfolio --records--> TradeTransaction
   - 1対多の関係
   - 時系列関係、増加のみ
   - 監査と分析に使用
```

これらの関係設計は、明日のクラス図設計の基盤を直接築きます。各関係タイプには特定の実装と制約があります。

### オントロジー：概念間の存在依存

より深い概念モデリングには理解が必要です：**これらの概念間の存在依存関係は何か？**

**存在依存の階層**：

```
独立存在レイヤー：
- Trader：ビジネス主体、独立して存在可能
- Asset：市場概念、外部定義、システムから独立して存在
- MarketData：外部イベントストリーム、システムから独立

依存存在レイヤー：
- Portfolio：Traderに依存して存在、しかし独自のライフサイクルを持つ
- Holding：Portfolioに依存して存在、独立した意味を持たない
- TradeOrder：Portfolioに依存して存在、取引意図を表現

関係存在レイヤー：
- Price：Assetと時間の関係
- Risk：PortfolioとMarketの関係
- Profit/Loss：時間経過による価値変化の関係
```

**存在依存の実装戦略**：

```python
# 独立存在：個別に作成・管理可能
class Trader:
    def __init__(self, trader_id: str):
        self.trader_id = trader_id
        self.portfolios = {}  # 管理するがライフサイクルは所有しない

    def create_portfolio(self) -> Portfolio:
        portfolio = Portfolio(self.trader_id, generate_id())
        self.portfolios[portfolio.id] = portfolio
        return portfolio

# 依存存在：集約ルートを通じて作成される必要がある
class Portfolio:
    def __init__(self, trader_id: str, portfolio_id: str):
        self.trader_id = trader_id  # 存在依存
        self.portfolio_id = portfolio_id
        self.holdings = {}

    def add_holding(self, symbol: str, quantity: int) -> None:
        # HoldingはPortfolioを通じてのみ作成可能
        holding = Holding(self.portfolio_id, symbol, quantity)
        self.holdings[symbol] = holding

    def remove_holding(self, symbol: str) -> None:
        # Holdingのライフサイクルを制御
        if symbol in self.holdings:
            del self.holdings[symbol]

# 関係存在：計算される、独立して保存されない
class PortfolioAnalyzer:
    def calculate_risk(self, portfolio: Portfolio, market_data: MarketData) -> RiskMetrics:
        # RiskはPortfolioとMarketの関係、計算される
        return RiskMetrics(
            volatility=self._calculate_volatility(portfolio, market_data),
            var=self._calculate_var(portfolio, market_data)
        )
```

この存在依存分析は、明日の集約設計とクラス間相互作用設計の基礎を提供します。

## 第二の領域：振る舞いモデリング - 概念間の相互作用

### 概念間の協調としての振る舞い

振る舞いモデリングの中核的な洞察は：**システムの振る舞いは単一のオブジェクトの振る舞いではなく、複数の概念間の協調的相互作用である。**

この認識は明日のシーケンス図設計のための重要な基盤を築きます：各ユースケースは概念間の一連のメッセージパッシングと状態変化です。

**投資取引ユースケースの概念的相互作用分析**：

```
ユースケース：迅速な取引を実行
参加概念：Trader、Portfolio、Holding、MarketData、TradeOrder

相互作用シーケンス：
1. Trader.createTradeIntent(symbol, quantity)
   - Trader内部で取引意図を検証
   - TradeIntent値オブジェクトを作成

2. Portfolio.validateTrade(tradeIntent)
   - Portfolioが十分な資金をチェック
   - Portfolioがリスク制限をチェック
   - RiskCalculator.assess(portfolio, tradeIntent)を呼び出す

3. MarketData.getCurrentPrice(symbol)
   - リアルタイム市場価格を取得
   - Price値オブジェクトを返す

4. Portfolio.executeTradeOrder(tradeOrder)
   - TradeOrderを作成
   - 対応するHoldingを更新
   - TradeTransactionを記録
   - PortfolioUpdatedイベントをトリガー

5. NotificationService.notify(trader, tradeResult)
   - 外部サービスが通知を処理
```

**概念的相互作用の設計原則**：

```
1. 依存方向の制御：
   - 高レベル概念は低レベル概念に依存可能
   - 集約ルートは集約内のエンティティに依存可能
   - ドメインサービスは複数の集約を調整可能

2. メッセージパッシングパターン：
   - 同期呼び出し：同じ集約内の操作
   - イベント公開：集約間の通信
   - コマンドパターン：複雑なビジネスプロセス

3. 状態変更制御：
   - 集約ルートのみが集約内の状態を変更可能
   - 集約間の状態変更はイベントを通じて実装
   - すべての状態変更は不変条件を維持しなければならない
```

### EventStorming：概念的相互作用の時間的モデリング

EventStormingは要件分析ツールであるだけでなく、概念的相互作用設計のモデリング手法でもあります：

**ドメインイベント駆動の概念的相互作用**：

```
イベントストリーム内の概念的役割：
TradeIntentCreated
- トリガー：Trader（取引意図を作成）
- 参加者：Portfolio（受信して検証）
- 結果：FundsValidationRequested

FundsValidated
- トリガー：Portfolio（資金チェック完了）
- 参加者：RiskCalculator（リスク評価サービス）
- 結果：RiskAssessmentRequested

RiskAssessed
- トリガー：RiskCalculator（リスク評価完了）
- 参加者：Portfolio（実行を決定）
- 結果：TradeOrderCreated または TradeRejected

TradeOrderSubmitted
- トリガー：Portfolio（注文を送信）
- 参加者：MarketGateway（外部市場インターフェース）
- 結果：MarketExecutionRequested

MarketExecutionConfirmed
- トリガー：MarketGateway（市場確認）
- 参加者：Portfolio（状態を更新）
- 結果：HoldingUpdated、TransactionRecorded

PortfolioUpdated
- トリガー：Portfolio（状態が更新された）
- 参加者：NotificationService（通知サービス）
- 結果：TraderNotified
```

各イベントは概念間の重要な相互作用を表し、明日のシーケンス図のための詳細な相互作用スクリプトを提供します。

### ステートマシン：概念的状態の調整

ステートマシンは単一のオブジェクトの状態変化だけでなく、より重要なのは複数の概念間の状態調整です：

**TradeOrderの状態変化と概念的調整**：

```
状態遷移における概念的協調：

Created → Validating：
- Portfolioが検証プロセスを開始
- RiskCalculatorがリスク計算に参加
- MarketDataが価格情報を提供

Validating → Approved：
- Portfolioがすべての検証が通過したことを確認
- TradeOrderの状態がApprovedに更新
- Portfolioが市場への送信を準備

Approved → Submitted：
- PortfolioがMarketGatewayを呼び出す
- TradeOrderの状態がSubmittedに更新
- 外部システムとの関係を確立

Submitted → Executed：
- MarketGatewayが市場確認を受信
- PortfolioがHoldingの状態を更新
- TradeOrderがExecutedとマーク
- 後続の通知プロセスをトリガー
```

この多概念調整を伴うステートマシン設計は、明日のシステム設計図の基盤を築きます。

## 第三の領域：データモデリング - 概念の永続化

### ドメイン概念からデータ構造へ

データモデリングの鍵は：**概念モデル内のエンティティ関係をデータストレージ構造に忠実にマッピングするにはどうすればよいか？**

**DynamoDBにおける集約ストレージ設計**：

```
集約ルート指向のパーティショニング戦略：
PK (Partition Key) | SK (Sort Key)        | エンティティタイプ
TRADER#123        | PROFILE              | Trader
TRADER#123        | PORTFOLIO#001        | Portfolio
TRADER#123        | HOLDING#001#AAPL     | Holding
TRADER#123        | TRADE#001#20240101   | TradeTransaction

クエリパターンと概念モデルの対応：
1. TraderのすべてのPortfolioを取得：
   PK = TRADER#123, begins_with(SK, "PORTFOLIO")

2. PortfolioのすべてのHoldingを取得：
   PK = TRADER#123, begins_with(SK, "HOLDING#001")

3. Portfolio集約の完全な再構成：
   PK = TRADER#123, SK between "PORTFOLIO#001" and "PORTFOLIO#001~"

整合性保証と集約境界の対応：
- 同じPartition内の操作 = 同じ集約内の操作
- DynamoDBトランザクション = 集約間の操作
- イベント公開 = 集約間の通信
```

### 概念的関係のデータ実装

異なる概念的関係には異なるデータ実装戦略が必要です：

```python
# 所有関係：埋め込み設計
{
    "PK": "TRADER#123",
    "SK": "PORTFOLIO#001",
    "ownerId": "TRADER#123",  # 所有権識別子
    "portfolioData": {
        "totalValue": 100000,
        "createdAt": "2024-01-01",
        "riskProfile": "MODERATE"
    }
}

# 構成関係：同じパーティション内の複数レコード
{
    "PK": "TRADER#123",
    "SK": "HOLDING#001#AAPL",
    "portfolioId": "PORTFOLIO#001",  # 構成関係識別子
    "symbol": "AAPL",
    "quantity": 100,
    "avgPrice": 150.50
}

# 参照関係：外部キー + キャッシュ
{
    "PK": "TRADER#123",
    "SK": "HOLDING#001#AAPL",
    "assetSymbol": "AAPL",  # 外部Assetを参照
    # Asset詳細はElastiCache経由でキャッシュ
}

# 履歴関係：時系列ストレージ
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

## 第四の領域：アーキテクチャモデリング - 概念の技術的マッピング

### AWSサービスとドメイン概念のマッピング

各AWSサービスは特定のドメイン概念または責任に対応すべきです：

**サービスマッピング戦略**：

```
ドメイン概念 → AWSサービス → アーキテクチャ上の責任

Traderアイデンティティ管理 → Cognito User Pool → 認証と認可
Portfolio集約 → DynamoDB + Lambda → 状態管理
MarketDataストリーム → Kinesis Data Streams → 外部イベント取り込み
TradeOrder処理 → Step Functions → 複雑なビジネスプロセス
ドメインイベント公開 → EventBridge → 概念間の通信
価格計算キャッシュ → ElastiCache → パフォーマンス最適化
取引通知 → SNS + SES → 外部統合
```

### クリーンアーキテクチャにおける概念的レイヤリング

```python
# ドメインレイヤー：純粋な概念モデル
class Portfolio:  # ドメイン概念
    def execute_trade(self, trade_order: TradeOrder) -> TradeResult:
        # 純粋なビジネスロジック、技術的実装に依存しない
        pass

# アプリケーションレイヤー：概念的調整
class ExecuteTradeUseCase:  # 概念間の協調
    def __init__(self, portfolio_repo: PortfolioRepository,
                 market_gateway: MarketGateway):
        self.portfolio_repo = portfolio_repo
        self.market_gateway = market_gateway

    def execute(self, trade_intent: TradeIntent) -> TradeResult:
        # Portfolio、MarketGatewayなどの概念の相互作用を調整
        portfolio = self.portfolio_repo.find_by_id(trade_intent.portfolio_id)
        return portfolio.execute_trade(trade_intent.to_trade_order())

# インフラストラクチャレイヤー：AWS技術実装
class DynamoDBPortfolioRepository(PortfolioRepository):  # 概念の永続化
    def save(self, portfolio: Portfolio) -> None:
        # ドメイン概念をDynamoDBにマッピング
        pass

class KinesisMarketGateway(MarketGateway):  # 外部概念の取り込み
    def submit_order(self, order: TradeOrder) -> OrderStatus:
        # Kinesisを介して外部市場と相互作用
        pass
```

## 明日のDDD深掘りの準備

今日の抽象モデリングを通じて、システム設計の4つの基盤を確立しました：

**概念モデリング**はドメインエンティティとその関係を確立
**振る舞いモデリング**は概念間の相互作用パターンを定義
**データモデリング**は概念の永続化戦略を設計
**アーキテクチャモデリング**は概念の技術的実装を計画

しかし、これらはまだ抽象的なフレームワークです。明日はDDDの具体的な実践を深掘りし、次のことを学びます：

- **今日の抽象概念を具体的なクラス図設計に変換するには？**
- **シーケンス図で概念間の相互作用を正確に記述するには？**
- **集約設計の境界をどのように決定し実装するか？**
- **ユースケースから集約への完全な設計プロセスとは？**

特に、今日「振る舞いは異なる概念間の相互作用である」という中核的洞察を理解しました。明日はこれらの相互作用をどのように設計し、概念間の協調がビジネス要件を満たすだけでなく技術的制約にも準拠することを確保する方法を具体的に学びます。

## 今日のモデリング哲学のまとめ

- **抽象モデリングはシステムの本質を理解するための4つのレンズである**
- **概念モデルは具体的な実装のための明確なガイダンスを提供しなければならない**
- **振る舞いは本質的に概念間の協調的相互作用である**
- **各抽象レベルは次のレベルの実装のための基盤を築く**

覚えておいてください：私たちは技術をモデリングしているのではなく、現実をモデリングしています。すべての概念には存在理由があり、すべての相互作用にはビジネス上の意味があります。明日は、これらの抽象概念がDDDのガイダンスの下で実行可能なシステムになる様子を見ていきます。

---

> 「概念の相互作用がシステムの振る舞いを構成する。私たちはオブジェクトを設計しているのではなく、オブジェクト間の関係を設計している。」
