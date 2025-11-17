# Day 4 | Building Business Logic with DDD: From Use Cases to Aggregate Design

Yesterday, we completed the four realms of abstract modeling, understanding the essence of the system from the four dimensions of concept, behavior, data, and architecture. But when we want to transform these abstract understandings into executable business logic, we face a key challenge:

**How to organize domain concepts into cohesive and meaningful business units?**

This is the core problem of DDD practice: **the systematic transformation from use case analysis to aggregate design**. Today, we will establish the static structure of the domain model to lay the foundation for an in-depth discussion of user operation scenarios tomorrow.

## Use Case Analysis: From Intent to Boundary Identification

### Re-examining the Six Cases from Day 2

In Day 2, we analyzed six typical user cases, each with its **core use case pattern**. Let's deconstruct them again from a DDD perspective:

**Investment Trading System [I-N-T-J] - Control-oriented Use Case Pattern**

```
Core Use Case: Execute real-time trading decisions
Primary Actor: Professional Trader
Goal in Context: Complete high-quality trades in the shortest possible time
Scope: Personal investment portfolio management system
Level: User goal level

Main Success Scenario:
1. The trader monitors market data
2. Identifies trading opportunities
3. Calculates risk impact
4. Submits a trade order
5. Confirms trade execution
6. Updates the portfolio status

Extensions:
2a. Market data is delayed → The system provides a delay warning
3a. Risk exceeds the limit → The system requires explicit confirmation
4a. Insufficient funds → The system suggests adjustments or financing
5a. Trade fails → The system enters a retry or manual processing state
```

From this use case analysis, we can identify several **core domain concepts**:

- **Trader** - A subject with decision-making ability
- **Portfolio** - A state container for asset allocation
- **TradeOrder** - A structured expression of trading intent
- **MarketData** - A source of external price information
- **RiskAssessment** - Calculation logic for decision support

**Family Financial System [E-N-T-J] - Coordination-oriented Use Case Pattern**

```
Core Use Case: Manage collaborative family expenses
Primary Actor: Head of family finance
Goal in Context: Coordinate the spending behavior of family members to maintain budget balance
Scope: Family financial management system
Level: Business summary level

Main Success Scenario:
1. Set family budget and permissions
2. Family members record expenses
3. The system checks the budget status
4. Triggers budget warnings or adjustments
5. Generates a family financial report

Role Interaction Complexity:
- Main decision-maker: Sets rules, monitors the overall situation
- General members: Make expenses, follow the rules
- System coordination: Ensures rule execution, provides transparency
```

Domain Concept Identification:

- **Family** - The boundary of collaboration and the rule-maker
- **FamilyMember** - A participant with specific permissions
- **Budget** - A constraint rule for resource allocation
- **Expense** - A factual record of resource consumption
- **PermissionManagement** - A control mechanism for collaborative behavior

### Domain Characteristics of Use Case Patterns

Through the comparative analysis of the six cases, we found that different use case patterns correspond to different **domain complexity characteristics**:

| Use Case Pattern | Main Complexity | Core Challenge | Design Focus |
| --- | --- | --- | --- |
| Control-oriented (Investment Trading) | Time sensitivity | Millisecond-level decisions | State consistency |
| Coordination-oriented (Family Finance) | Multi-role collaboration | Permissions and transparency | Role boundaries |
| Certainty-oriented (Health Monitoring) | Data reliability | Continuous monitoring | Data integrity |
| Adaptive (Smart Home) | Flexible response | Progressive control | Graceful degradation |
| Transparent (Parcel Tracking) | Information synchronization | Multi-source data integration | State synchronization |
| Creative (Travel Sharing) | Expressive flexibility | Creative support tools | Content fluidity |

Each pattern will affect our aggregate design strategy.

## Aggregate Identification: The Art of Dividing Business Boundaries

### Three Principles of Aggregate Identification

**Principle 1: Business Invariant Boundary**
The boundary of an aggregate should correspond to a set of concepts that must remain consistent in the business.

**Invariant Analysis of the Investment Trading System**:

```
Invariants of the Portfolio Aggregate:
- Total asset value = Σ(holding value) + cash balance
- No trade can result in a negative cash balance
- Risk exposure cannot exceed the preset limit

Invariants of the Order Aggregate:
- Core parameters of an order cannot be modified once submitted
- Execution records must correspond to the original instruction
- State transitions must comply with the business process
```

**Principle 2: Transaction Boundary Correspondence**
Modifications within an aggregate should be able to be completed in a single transaction.

**Transaction Analysis of the Family Financial System**:

```
Family Aggregate:
✓ Can be completed in a single transaction:
  - Set budget amount
  - Adjust member permissions
  - Update basic family information

✗ Cannot be completed in a single transaction:
  - Count the monthly expenses of all members (across multiple aggregates)
  - Generate an annual report (data volume is too large)
```

**Principle 3: Business Language Correspondence**
The naming and structure of an aggregate should directly correspond to the language of business experts.

### Comparison of Aggregate Designs for the Six Cases

**Case 1: Investment Trading System**

```
Aggregate Design:
┌─────────────────┐    ┌─────────────────┐
│   Portfolio     │    │      Order      │
│  (Investment Portfolio)     │    │    (Trade Order)   │
├─────────────────┤    ├─────────────────┤
│ - portfolioId   │    │ - orderId       │
│ - traderId      │    │ - portfolioId   │
│ - holdings[]    │    │ - tradeOrder    │
│ - cashBalance   │    │ - status        │
│ - riskLimit     │    │ - execution[]   │
└─────────────────┘    └─────────────────┘
        │                       │
        └───────────────────────┘
             Domain Event Communication

Design Logic:
- Portfolio manages asset status and risk control
- Order manages the trading process and execution tracking
- The two maintain eventual consistency through events
```

**Case 2: Family Financial System**

```
Aggregate Design:
┌─────────────────┐    ┌─────────────────┐
│     Family      │    │     Budget      │
│    (Family)       │    │    (Budget)       │
├─────────────────┤    ├─────────────────┤
│ - familyId      │    │ - budgetId      │
│ - members[]     │    │ - familyId      │
│ - permissions   │    │ - categories[]  │
│ - settings      │    │ - limits        │
└─────────────────┘    │ - period        │
                       └─────────────────┘
              │                 │
              └─────────────────┘
                Event-driven Coordination

┌─────────────────┐
│    Expense      │
│   (Expense Record)    │
├─────────────────┤
│ - expenseId     │
│ - familyId      │
│ - memberId      │
│ - amount        │
│ - category      │
│ - timestamp     │
└─────────────────┘

Design Logic:
- Family manages members and permission structure
- Budget manages budget rules and limits
- Expense records actual spending facts
- The three collaborate to achieve budget control
```

**Case 3: Health Monitoring System**

```
Aggregate Design:
┌─────────────────┐    ┌─────────────────┐
│   HealthProfile │    │  DeviceReading  │
│   (Health Profile)    │    │   (Device Data)    │
├─────────────────┤    ├─────────────────┤
│ - userId        │    │ - readingId     │
│ - metrics[]     │    │ - userId        │
│ - thresholds    │    │ - deviceType    │
│ - history       │    │ - value         │
└─────────────────┘    │ - timestamp     │
                       │ - status        │
                       └─────────────────┘

Design Logic:
- HealthProfile manages long-term health status
- DeviceReading manages real-time data collection
- Separate design supports different data lifecycles
```

### Interaction Patterns between Aggregates

Different domains will produce different aggregate interaction patterns:

**Pattern 1: Event-driven Coordination (Investment Trading)**

```
Portfolio Aggregate → TradeValidatedEvent → Order Aggregate
Order Aggregate → OrderExecutedEvent → Portfolio Aggregate
Portfolio Aggregate → PortfolioUpdatedEvent → RiskManagement Aggregate
```

**Pattern 2: Permission-driven Collaboration (Family Finance)**

```
Family Aggregate → Set permission rules → Expense Aggregate checks permissions
Budget Aggregate → Set limit rules → Expense Aggregate checks limits
Expense Aggregate → Record expense → Budget Aggregate updates status
```

**Pattern 3: Data-driven Aggregation (Health Monitoring)**

```
Device → Push data → DeviceReading Aggregate
DeviceReading Aggregate → Data validation complete → HealthProfile Aggregate
HealthProfile Aggregate → Threshold check → Alert Aggregate
```

## Domain Services: Business Logic Across Aggregates

### When to Identify Domain Services

We need domain services when business logic **does not naturally belong to any single aggregate**.

**Domain Service Case for the Investment Trading System**:

```typescript
// Risk calculation service: requires collaboration between Portfolio and MarketData
class RiskCalculationService {
  calculate(portfolio: Portfolio, marketData: MarketData): RiskMetric {
    // This logic does not belong to Portfolio (because it needs external market data)
    // Nor does it belong to MarketData (because it needs portfolio-specific information)

    const portfolioValue = portfolio.getTotalValue(marketData);
    const volatility = marketData.calculateVolatility(portfolio.getSymbols());
    const concentration = portfolio.calculateConcentration();

    return new RiskMetric(portfolioValue, volatility, concentration);
  }
}

// Trade execution coordination service: coordinates Portfolio and Order aggregates
class TradeExecutionService {
  async executeTrade(
    portfolio: Portfolio,
    tradeOrder: TradeOrder
  ): Promise<ExecutionResult> {
    // 1. Validate the trading ability of the Portfolio
    const validation = portfolio.validateTrade(tradeOrder);
    if (!validation.isValid) {
      return ExecutionResult.rejected(validation.reason);
    }

    // 2. Create an Order aggregate to manage the execution process
    const order = Order.create(portfolio.getId(), tradeOrder);

    // 3. Submit to the market (external service)
    const marketResult = await this.marketService.submit(order);

    // 4. Update the status of related aggregates
    if (marketResult.isSuccessful) {
      portfolio.applyExecution(marketResult);
      order.markAsExecuted(marketResult);
    }

    return ExecutionResult.fromMarket(marketResult);
  }
}
```

### Design Principles for Domain Services

**Statelessness**: Domain services themselves do not maintain state, they only coordinate interactions between aggregates

**Business-oriented**: The naming and interface of a service should reflect business concepts, not technical implementations

**Minimal Responsibility**: Each service only handles one clear business scenario

## Preparing for User Operation Scenarios

### From Domain Model to User Roles

The aggregate design established today will directly affect how we design user operation scenarios tomorrow:

**Correspondence between Roles and Aggregates**:

```
Investment Trading System:
- Trader role → Owner of the Portfolio aggregate
- System administrator → Monitor of the entire system
- Risk control specialist → Operator of the RiskManagement aggregate

Family Financial System:
- Head of family → Manager of the Family aggregate
- Family member → Creator of the Expense aggregate
- Budget manager → Maintainer of the Budget aggregate
```

**Aggregate Boundaries of Operation Permissions**:

- Each aggregate defines the operation boundaries for specific roles
- Operations across aggregates need to be coordinated through domain services
- Complex business processes will produce multi-step user stories

### Preparation for State Scenarios

The state change of each aggregate will produce a corresponding user operation scenario:

**State Transitions of the Portfolio Aggregate**:

```
Empty Portfolio → [Initialize funds] → Active Portfolio
Active Portfolio → [Execute trade] → Active Portfolio (status update)
Active Portfolio → [Risk exceeded] → Restricted Portfolio
Restricted Portfolio → [Risk adjustment] → Active Portfolio
Active Portfolio → [Sell all] → Empty Portfolio
```

Each state transition corresponds to a user operation scenario. Tomorrow, we will delve into how to design these transitions into intuitive User Stories.

## Recording Design Decisions

When designing aggregates, we made many important decisions. Recording these decisions is crucial for the subsequent User Story design:

**Decision 1: Separate Portfolio and Order**

- Reason: Portfolio focuses on asset status, Order focuses on the trading process
- Impact: Users need to view asset status and trade status on different interfaces
- User Story Impact: Need to design an operation flow across aggregates

**Decision 2: Family Aggregate Includes Permission Management**

- Reason: Permission settings are closely related to the family structure
- Impact: Permission changes will trigger state changes in the Family aggregate
- User Story Impact: Need to design a user interface for permission management

**Decision 3: Adopt Read-Write Separation for Health Data**

- Reason: The frequency of reads and writes is vastly different
- Impact: Real-time queries and historical analysis use different data sources
- User Story Impact: Need to consider the time delay of data synchronization

## Bridge to Tomorrow

Through today's aggregate design, we have established the static structure of business logic. Tomorrow, we will delve into:

1. **User Story Writing Techniques**: How to transform the business capabilities of aggregates into user stories
2. **Role and Permission Design**: How to design a role and permission system based on aggregate boundaries
3. **Operation Scenario Flows**: How to design complex business flows across aggregates
4. **State Scenario Management**: How to handle the user experience challenges brought by aggregate state changes

## Today's Conceptual Takeaways

- **Aggregates are the boundaries of business invariants**: Technical implementation must serve business rules
- **Domain services coordinate cross-aggregate logic**: The organization of complex business processes
- **Different domains have different aggregate patterns**: Investment, family, and health each have their own characteristics
- **Aggregate design directly affects user experience**: A good domain model is the foundation of a good UX

Remember: What we established today is not a technical architecture, but the organization of business logic. This organization will directly determine how we design the user's operation experience tomorrow.

---

> "An aggregate is not a grouping of code, but a living unit of a business concept. Each aggregate has its own unique lifecycle, and every user operation is participating in the evolution of this lifecycle."
