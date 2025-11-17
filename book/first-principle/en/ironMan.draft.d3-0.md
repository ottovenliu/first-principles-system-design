# Day 3 | Requirements Extraction Methodology - Abstract Modeling

In the first three days, we completed an important cognitive transformation: extracting the essence of functional requirements from users' vague expressions, and then deriving specific technical boundaries from functional requirements. We know what the system should do and to what extent.

But now we face a deeper challenge: **How to transform these understandings into an achievable system design?**

When we know that the investment trading system needs to support 2000 RPS, with a latency of less than 100ms and an RPO of zero, what kind of conceptual model, behavioral flow, data structure, and architectural pattern should we design?

This is the core problem that **abstract modeling** aims to solve: **a systematic transformation method from requirements understanding to system design.**

## The Four Realms of Abstract Modeling

Abstract modeling is not a single technical method, but a philosophical framework for **understanding the same system from four different dimensions**. Just as we cannot fully understand complex reality from a single perspective, we cannot fully describe the essence of a system with a single model.

```
                Abstract Modeling
                   |
┌────────┬─────────────┬────────────┬────────────┐
| Conceptual Modeling | Behavioral Modeling | Data Modeling | Architectural Modeling |
|---------|--------------|-------------|-------------|
| DDD     | Use Case     | ER Model    | C4 Model    |
| Ontology| EventStorming| NoSQL Schema| 4+1 View    |
| ORM     | State Machine| Normalization| Clean Arch |
```

### The Ontological Significance of the Four Dimensions

Each dimension answers a fundamental question in system design:

- **Conceptual Modeling**: What IS it? (What is the essence of this system?)
- **Behavioral Modeling**: What DOES it do? (What will this system do?)
- **Data Modeling**: What does it REMEMBER? (What does this system remember?)
- **Architectural Modeling**: How is it STRUCTURED? (How is this system organized?)

From the previous analysis, we have obtained design constraints for the four dimensions:

**Correspondence between Requirements Analysis and Modeling Constraints**:

- User cognitive model → Abstraction level of conceptual modeling
- Business event flow → Temporal logic of behavioral modeling
- RPS and capacity requirements → Storage strategy of data modeling
- Availability and recovery requirements → Resilience design of architectural modeling

## The First Realm: Conceptual Modeling - The Ontology of the System

### DDD: From Abstract Concepts to Static Structures

The core question of conceptual modeling is: **After removing all technical details, what reality does this system represent?**

But more importantly: **How to transform these abstract concepts into static structures that can be expressed in a programming language?**

Returning to the investment trading case, from the INTJ user analysis in Day 2, we understood that the core requirement is "the ultimate control of time". This abstract understanding needs to be transformed into specific domain concepts:

**Transformation from Abstract Concepts to Domain Entities**:

```
Abstract Concept Layer:
"The ultimate control of time" → What is its concrete manifestation?

Business Concept Layer:
- Decision-making subject: Who is controlling time? → Trader
- Control object: What is being controlled? → Portfolio
- Control tool: How to control? → TradeOrder
- External dependency: Where does the time pressure come from? → MarketData

Domain Entity Layer:
The programming concept corresponding to each business concept:
- Trader → Entity (a subject with identity)
- Portfolio → Aggregate Root
- Holding → Entity (an entity within the aggregate)
- TradeOrder → Value Object
- Price → Value Object (immutable value)
```

**Static Structure Design of Domain Entities**:

```
Trader - Entity
Attributes:
- traderId: String (unique identifier)
- riskProfile: RiskProfile
- tradingPreferences: TradingPreferences

Responsibilities:
- Create portfolios
- Set risk limits
- Place trade orders

Invariants:
- The trader ID cannot be changed once created
- The risk profile must be within the system's allowed range

Portfolio - Aggregate Root
Attributes:
- portfolioId: String (unique identifier)
- traderId: String (owner)
- holdings: List<Holding>
- totalValue: Money
- lastUpdated: DateTime

Responsibilities:
- Manage all holdings
- Calculate overall risk
- Execute trade operations
- Maintain data consistency

Aggregate Boundary:
- Portfolio is the aggregate root
- Holding can only be accessed through Portfolio
- Operations within the same Portfolio must remain consistent

Invariants:
- Total value = Σ(holding quantity × current price)
- Cash holding cannot be negative
- Risk indicators cannot exceed the set limits
```

**Design of Relationships between Concepts**:

```
Analysis of Relationship Types:
1. Ownership
   Trader --owns--> Portfolio
   - One-to-many relationship
   - Strong ownership, Portfolio cannot exist independently
   - Cascade delete: Portfolio is deleted when Trader is deleted

2. Composition
   Portfolio --contains--> Holding
   - One-to-many relationship
   - Strong composition, Holding is a component of Portfolio
   - Lifecycle binding: Holding is deleted when Portfolio is deleted

3. Reference
   Holding --references--> Asset
   - Many-to-one relationship
   - Weak reference, Asset exists independently
   - Asset is an external concept defined by the market

4. History
   Portfolio --records--> TradeTransaction
   - One-to-many relationship
   - Time-series relationship, only increases
   - Used for auditing and analysis
```

These relationship designs directly lay the foundation for tomorrow's class diagram design. Each relationship type has its specific implementation and constraints.

### Ontology: Existential Dependence between Concepts

Deeper conceptual modeling requires understanding: **What is the existential dependence relationship between these concepts?**

**Hierarchy of Existential Dependence**:

```
Independent Existence Layer:
- Trader: Business subject, can exist independently
- Asset: Market concept, externally defined, exists independently of our system
- MarketData: External event stream, independent of our system

Dependent Existence Layer:
- Portfolio: Depends on Trader for existence, but has its own lifecycle
- Holding: Depends on Portfolio for existence, has no independent meaning
- TradeOrder: Depends on Portfolio for existence, expresses trading intent

Relational Existence Layer:
- Price: The relationship between Asset and time
- Risk: The relationship between Portfolio and Market
- Profit/Loss: The relationship of value change over time
```

**Implementation Strategy for Existential Dependence**:

```python
# Independent existence: can be created and managed separately
class Trader:
    def __init__(self, trader_id: str):
        self.trader_id = trader_id
        self.portfolios = {}  # Manages but does not own the lifecycle

    def create_portfolio(self) -> Portfolio:
        portfolio = Portfolio(self.trader_id, generate_id())
        self.portfolios[portfolio.id] = portfolio
        return portfolio

# Dependent existence: must be created through the aggregate root
class Portfolio:
    def __init__(self, trader_id: str, portfolio_id: str):
        self.trader_id = trader_id  # Existential dependence
        self.portfolio_id = portfolio_id
        self.holdings = {}

    def add_holding(self, symbol: str, quantity: int) -> None:
        # Holding can only be created through Portfolio
        holding = Holding(self.portfolio_id, symbol, quantity)
        self.holdings[symbol] = holding

    def remove_holding(self, symbol: str) -> None:
        # Controls the lifecycle of Holding
        if symbol in self.holdings:
            del self.holdings[symbol]

# Relational existence: calculated, not stored independently
class PortfolioAnalyzer:
    def calculate_risk(self, portfolio: Portfolio, market_data: MarketData) -> RiskMetrics:
        # Risk is the relationship between Portfolio and Market, calculated
        return RiskMetrics(
            volatility=self._calculate_volatility(portfolio, market_data),
            var=self._calculate_var(portfolio, market_data)
        )
```

This existential dependence analysis provides the basis for tomorrow's aggregate design and inter-class interaction design.

## The Second Realm: Behavioral Modeling - Interaction between Concepts

### Behavior as Collaboration between Concepts

The core insight of behavioral modeling is: **The behavior of a system is not the behavior of a single object, but the collaborative interaction between multiple concepts.**

This cognition lays an important foundation for tomorrow's sequence diagram design: each use case is a series of message passing and state changes between concepts.

**Conceptual Interaction Analysis of the Investment Trading Use Case**:

```
Use Case: Execute a fast trade
Participating Concepts: Trader, Portfolio, Holding, MarketData, TradeOrder

Interaction Sequence:
1. Trader.createTradeIntent(symbol, quantity)
   - Trader internally validates the trade intent
   - Creates a TradeIntent value object

2. Portfolio.validateTrade(tradeIntent)
   - Portfolio checks for sufficient funds
   - Portfolio checks risk limits
   - Calls RiskCalculator.assess(portfolio, tradeIntent)

3. MarketData.getCurrentPrice(symbol)
   - Gets the real-time market price
   - Returns a Price value object

4. Portfolio.executeTradeOrder(tradeOrder)
   - Creates a TradeOrder
   - Updates the corresponding Holding
   - Records a TradeTransaction
   - Triggers a PortfolioUpdated event

5. NotificationService.notify(trader, tradeResult)
   - External service handles notifications
```

**Design Principles for Conceptual Interaction**:

```
1. Control of Dependency Direction:
   - High-level concepts can depend on low-level concepts
   - Aggregate roots can depend on entities within the aggregate
   - Domain services can coordinate multiple aggregates

2. Message Passing Patterns:
   - Synchronous calls: Operations within the same aggregate
   - Event publishing: Communication across aggregates
   - Command pattern: Complex business processes

3. State Change Control:
   - Only the aggregate root can modify the state within the aggregate
   - State changes across aggregates are implemented through events
   - All state changes must maintain invariants
```

### EventStorming: Temporal Modeling of Conceptual Interaction

EventStorming is not only a requirements analysis tool, but also a modeling method for conceptual interaction design:

**Conceptual Interaction Driven by Domain Events**:

```
Conceptual Roles in the Event Stream:
TradeIntentCreated
- Trigger: Trader (creates a trade intent)
- Participant: Portfolio (receives and validates)
- Result: FundsValidationRequested

FundsValidated
- Trigger: Portfolio (completes fund check)
- Participant: RiskCalculator (risk assessment service)
- Result: RiskAssessmentRequested

RiskAssessed
- Trigger: RiskCalculator (completes risk assessment)
- Participant: Portfolio (decides to execute)
- Result: TradeOrderCreated or TradeRejected

TradeOrderSubmitted
- Trigger: Portfolio (submits the order)
- Participant: MarketGateway (external market interface)
- Result: MarketExecutionRequested

MarketExecutionConfirmed
- Trigger: MarketGateway (market confirmation)
- Participant: Portfolio (updates state)
- Result: HoldingUpdated, TransactionRecorded

PortfolioUpdated
- Trigger: Portfolio (state has been updated)
- Participant: NotificationService (notification service)
- Result: TraderNotified
```

Each event represents an important interaction between concepts, which provides a detailed interaction script for tomorrow's sequence diagram.

### State Machine: Coordination of Conceptual States

A state machine is not just the state change of a single object, but more importantly, the state coordination between multiple concepts:

**State Change and Conceptual Coordination of TradeOrder**:

```
Conceptual Collaboration in State Transitions:

Created → Validating:
- Portfolio starts the validation process
- RiskCalculator participates in risk calculation
- MarketData provides price information

Validating → Approved:
- Portfolio confirms all validations have passed
- TradeOrder state is updated to Approved
- Portfolio prepares to submit to the market

Approved → Submitted:
- Portfolio calls MarketGateway
- TradeOrder state is updated to Submitted
- Establishes a relationship with the external system

Submitted → Executed:
- MarketGateway receives market confirmation
- Portfolio updates Holding state
- TradeOrder is marked as Executed
- Triggers subsequent notification processes
```

This state machine design with multi-concept coordination lays the foundation for tomorrow's system design diagram.

## The Third Realm: Data Modeling - Persistence of Concepts

### From Domain Concepts to Data Structures

The key to data modeling is: **How to faithfully map the entity relationships in the conceptual model to the data storage structure?**

**Aggregate Storage Design in DynamoDB**:

```
Partitioning Strategy Oriented by Aggregate Roots:
PK (Partition Key) | SK (Sort Key)        | Entity Type
TRADER#123        | PROFILE              | Trader
TRADER#123        | PORTFOLIO#001        | Portfolio
TRADER#123        | HOLDING#001#AAPL     | Holding
TRADER#123        | TRADE#001#20240101   | TradeTransaction

Correspondence between Query Patterns and Conceptual Model:
1. Get all Portfolios of a Trader:
   PK = TRADER#123, begins_with(SK, "PORTFOLIO")

2. Get all Holdings of a Portfolio:
   PK = TRADER#123, begins_with(SK, "HOLDING#001")

3. Complete reconstruction of the Portfolio aggregate:
   PK = TRADER#123, SK between "PORTFOLIO#001" and "PORTFOLIO#001~"

Correspondence between Consistency Guarantee and Aggregate Boundary:
- Operations within the same Partition = Operations within the same aggregate
- DynamoDB Transaction = Operations across aggregates
- Event publishing = Communication between aggregates
```

### Data Implementation of Conceptual Relationships

Different conceptual relationships require different data implementation strategies:

```python
# Ownership relationship: Embedded design
{
    "PK": "TRADER#123",
    "SK": "PORTFOLIO#001",
    "ownerId": "TRADER#123",  # Ownership identifier
    "portfolioData": {
        "totalValue": 100000,
        "createdAt": "2024-01-01",
        "riskProfile": "MODERATE"
    }
}

# Composition relationship: Multiple records within the same partition
{
    "PK": "TRADER#123",
    "SK": "HOLDING#001#AAPL",
    "portfolioId": "PORTFOLIO#001",  # Composition relationship identifier
    "symbol": "AAPL",
    "quantity": 100,
    "avgPrice": 150.50
}

# Reference relationship: Foreign key + cache
{
    "PK": "TRADER#123",
    "SK": "HOLDING#001#AAPL",
    "assetSymbol": "AAPL",  # References an external Asset
    # Asset details are cached via ElastiCache
}

# History relationship: Time-series storage
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

## The Fourth Realm: Architectural Modeling - Technical Mapping of Concepts

### Mapping between AWS Services and Domain Concepts

Each AWS service should correspond to a specific domain concept or responsibility:

**Service Mapping Strategy**:

```
Domain Concept → AWS Service → Architectural Responsibility

Trader identity management → Cognito User Pool → Authentication and authorization
Portfolio aggregate → DynamoDB + Lambda → State management
MarketData stream → Kinesis Data Streams → External event ingestion
TradeOrder processing → Step Functions → Complex business processes
Domain event publishing → EventBridge → Communication between concepts
Price calculation cache → ElastiCache → Performance optimization
Trade notification → SNS + SES → External integration
```

### Conceptual Layering in Clean Architecture

```python
# Domain Layer: Pure conceptual model
class Portfolio:  # Domain concept
    def execute_trade(self, trade_order: TradeOrder) -> TradeResult:
        # Pure business logic, does not depend on any technical implementation
        pass

# Application Layer: Conceptual coordination
class ExecuteTradeUseCase:  # Collaboration between concepts
    def __init__(self, portfolio_repo: PortfolioRepository,
                 market_gateway: MarketGateway):
        self.portfolio_repo = portfolio_repo
        self.market_gateway = market_gateway

    def execute(self, trade_intent: TradeIntent) -> TradeResult:
        # Coordinates the interaction of concepts like Portfolio, MarketGateway, etc.
        portfolio = self.portfolio_repo.find_by_id(trade_intent.portfolio_id)
        return portfolio.execute_trade(trade_intent.to_trade_order())

# Infrastructure Layer: AWS technical implementation
class DynamoDBPortfolioRepository(PortfolioRepository):  # Persistence of concepts
    def save(self, portfolio: Portfolio) -> None:
        # Maps domain concepts to DynamoDB
        pass

class KinesisMarketGateway(MarketGateway):  # Ingestion of external concepts
    def submit_order(self, order: TradeOrder) -> OrderStatus:
        # Interacts with the external market via Kinesis
        pass
```

## Preparing for Tomorrow's Deep Dive into DDD

Through today's abstract modeling, we have established the four-fold foundation of system design:

**Conceptual modeling** established domain entities and their relationships
**Behavioral modeling** defined the interaction patterns between concepts
**Data modeling** designed the persistence strategy for concepts
**Architectural modeling** planned the technical implementation of concepts

But these are still abstract frameworks. Tomorrow we will delve into the specific practices of DDD and learn:

- **How to transform today's abstract concepts into a concrete class diagram design?**
- **How to accurately describe the interaction between concepts with a sequence diagram?**
- **How to determine and implement the boundaries of aggregate design?**
- **What is the complete design process from use case to aggregate?**

In particular, today we understood the core insight that "behavior is the interaction between different concepts". Tomorrow we will specifically learn how to design these interactions and how to ensure that the collaboration between concepts not only meets business requirements but also complies with technical constraints.

## Today's Modeling Philosophy Summary

- **Abstract modeling is a four-fold lens for understanding the essence of a system**
- **The conceptual model must provide clear guidance for concrete implementation**
- **Behavior is essentially collaborative interaction between concepts**
- **Each level of abstraction lays the foundation for the implementation of the next level**

Remember: We are not modeling technology, but modeling reality. Every concept has its reason for existence, and every interaction has its business meaning. Tomorrow we will see how these abstract concepts become an executable system under the guidance of DDD.

---

> "The interaction of concepts constitutes the behavior of the system. We are not designing objects, but the relationships between objects."
