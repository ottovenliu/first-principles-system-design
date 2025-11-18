# Day 2-2 | Requirements Confirmation × System Design Starting Point (2): Domain Boundaries and Basic Requirements Confirmation

Yesterday, we extracted the essence of functional requirements from user's words and understood the purpose of the system's existence under different cognitive models. But when we are ready to transform these abstract understandings into a concrete AWS architecture, we will find a key problem:

**Functional requirements answer "what to do", but not "to what extent".**

When investment trading users say "I'm afraid that slow speed will cause me to lose money", what they really mean is how many milliseconds the delay should be controlled within? When family finance users say "organize expenses at any time", how many concurrent users should the system support? When health monitoring users say "it can't affect judgment", how long is the acceptable downtime?

These are the areas of **non-functional requirements**, which determine the technical boundaries of the system and the specific configuration of AWS services.

## The Ontology of Non-Functional Requirements: From Qualitative to Quantitative Transformation

### Three Levels of Technical Boundaries

**Performance Boundaries**

- The upper and lower limits of API requests per second (RPS)
- The expected value and tolerance range of response time
- The peak and average values of data processing throughput

**Capacity Boundaries**

- The expected curve and peak planning of user growth
- The growth model and historical retention of data storage
- The elastic expansion and cost control of computing resources

**Resilience Boundaries**

- Acceptable Recovery Time Objective (RTO)
- Tolerable Recovery Point Objective (RPO)
- The graceful range of system degradation and core function assurance

### Deriving Technical Specifications from Domain Events

Each domain has its unique **technical rhythm**, which determines the performance characteristics of the system:

**Financial Trading Domain**: Microsecond-level competitive rhythm

- Market data updates: thousands of times per second
- Trading decision delay: milliseconds are unacceptable
- Position calculation accuracy: 8 decimal places

**Family Management Domain**: Collaborative rhythm of daily life

- Expense recording frequency: several to dozens of times a day
- Synchronization delay tolerance: seconds are acceptable
- Data consistency requirements: eventual consistency is sufficient

**Health Monitoring Domain**: Monitoring rhythm of physiological cycles

- Device data synchronization: minutes to hours
- Anomaly alarm delay: key indicators need to be real-time
- Historical data preservation: long-term storage at the year level

## API Request Per Second: From User Behavior to Technical Indicators

### The Philosophical Foundation of RPS

RPS is not a number estimated out of thin air, but an inevitable result derived from user behavior patterns. Behind every API call is a user's intention, and the frequency distribution of user intentions determines the load characteristics of the system.

#### RPS Derivation for Investment Trading System

**User Behavior Pattern Analysis**:

**Behavioral Stratification**:

- Ordinary investors: check positions 2-5 times a day, trade 0-2 times
- Active traders: check positions 20-50 times a day, trade 5-15 times
- High-frequency traders: check positions every second, may trade every minute

**Time Distribution**:

- 30 minutes before the market opens: traffic peaks, 10 times the usual
- Trading session: continuous high traffic, 5 times the usual
- After the market closes: gradually falls back to the baseline

**Calculation Logic**:

- Ordinary users: 5 API calls/day ÷ 8 hours = 0.17 RPS/user
- Active users: 50 API calls/day ÷ 8 hours = 1.74 RPS/user
- High-frequency users: 3600 API calls/hour ÷ 3600 seconds = 60 RPS/user
- Assuming user distribution: 80% ordinary, 15% active, 5% high-frequency
- Total RPS = (0.8 × 0.17 + 0.15 × 1.74 + 0.05 × 60) × total_users
- = (0.136 + 0.261 + 3) × total_users
- = 3.397 × total_users

**RPS Thresholds for AWS Service Selection**:

- < 100 RPS: API Gateway + Lambda (Serverless first)
- 100-1000 RPS: ALB + ECS/EC2 (Container or VM)
- 1000-10000 RPS: ALB + Auto Scaling Groups
- > 10000 RPS: Custom Load Balancer + Multi-AZ deployment

#### RPS Derivation for Family Financial System

**Time Aggregation of Collaborative Behavior**:

**Family Collaboration Model**:

- Average of 3.5 people per family, with 2 frequent users
- Concentrated usage periods: after dinner (19:00-21:00), weekend shopping periods
- Collaboration conflicts: probability of multiple people recording the same expense at the same time

**Calculation Logic**:

- Weekday evenings: 2 users/family × 5 records/2hours = 2.5 RPS/family/1000families
- Weekend peak: 2.5 × 3 = 7.5 RPS/1000families
- Annual tax season: 7.5 × 2 = 15 RPS/1000families

#### RPS Derivation for Health Monitoring System

**Data Push Model of IoT Devices**:

**Device Type and Frequency**:

- Smart bracelet: heart rate every 30 seconds, steps every 10 minutes
- Blood pressure monitor: manual measurement, 1-3 times a day
- Blood glucose meter: manual measurement, 2-6 times a day
- Smart scale: manual measurement, 2-3 times a week

**Calculation Logic**:

- Bracelet: (120 + 6) API calls/day = 126/day = 0.0015 RPS/user
- Blood pressure monitor: 3 API calls/day = 0.000035 RPS/user
- Blood glucose meter: 6 API calls/day = 0.00007 RPS/user
- Scale: 0.4 API calls/day = 0.000005 RPS/user
- Total: ≈ 0.002 RPS/user
- But considering batch processing of data synchronization and retries for exceptions:
- Actual RPS = 0.002 × retry_factor × batch_factor ≈ 0.01 RPS/user

### The Philosophy of API Design: From Interface to Contract

An API is not just a technical interface, but a **programmatic expression of domain concepts**. Each API endpoint should correspond to a meaningful operation in the domain.

**RESTful Domain Semantic Mapping**:

**Investment Trading Domain**:

- POST /portfolios/{id}/orders - Submit a trade order
- GET /portfolios/{id}/positions - Query position status
- PATCH /accounts/{id}/risk-preferences - Update risk preference
- GET /market/quotes/{symbol} - Get market quote

**Family Finance Domain**:

- POST /families/{id}/expenses - Record an expense
- GET /families/{id}/budget/status - Query budget status
- PUT /families/{id}/members/{member_id}/permissions - Set permissions
- GET /families/{id}/reports/{period} - Generate a financial report

**AWS API Gateway Throttling Strategy**:

```yaml
ThrottlingSettings:
  BurstLimit: 5000 # Burst traffic limit
  RateLimit: 2000 # Steady-state traffic limit

UsagePlan:
  BasicTier:
    Throttle: { BurstLimit: 100, RateLimit: 50 }
    Quota: { Limit: 1000, Period: DAY }
  PremiumTier:
    Throttle: { BurstLimit: 1000, RateLimit: 500 }
    Quota: { Limit: 10000, Period: DAY }
```

## Capacity Planning: System Thinking for Growth Patterns

### Pattern Recognition of Business Growth

Different businesses have different growth curves, which directly affects capacity planning strategies:

**Linear Growth Model (Family Finance Management)**

- Characteristics: Stable user growth, predictable usage patterns
- Growth function: capacity(t) = baseline + growth_rate × t
- AWS strategy: Reserved Instances + Predictable Scaling
- Cost optimization: Long-term commitment in exchange for cost savings

**Exponential Growth Model (Social Sharing Platform)**

- Characteristics: Viral spread, extremely volatile traffic
- Growth function: capacity(t) = baseline × (1 + growth_rate)^t
- AWS strategy: Serverless + Auto Scaling + Spot Instances
- Cost optimization: Pay-as-you-go, rapid elastic scaling

**Cyclical Growth Model (Investment Trading System)**

- Characteristics: Highly correlated with external cycles (market opening hours)
- Growth function: capacity(t) = baseline + seasonal_factor × sin(2π × t/period)
- AWS strategy: Scheduled Scaling + Predictive Scaling
- Cost optimization: Predictive scaling to avoid sudden traffic spikes

### Lifecycle Management of Storage Capacity

**Time Value Decay of Data**:

Each type of data has its own temporal significance and access pattern:

**Lifecycle of Investment Trading Data**:

- Real-time data (within 1 hour): Extremely high access frequency → S3 Standard
- Same-day data (within 24 hours): High access frequency → S3 Standard
- Historical data (within 30 days): Medium access frequency → S3 IA
- Quarterly reports (within 1 year): Low access frequency → S3 Glacier
- Regulatory archives (within 7 years): Extremely low access frequency → S3 Deep Archive

**Lifecycle of Health Monitoring Data**:

- Current metrics (within 1 day): User checks daily → S3 Standard
- Periodic trends (within 30 days): Periodic analysis → S3 IA
- Annual records (within 1 year): Comparison for annual check-ups → S3 Glacier
- Long-term history (lifelong): Medical research value → S3 Deep Archive

**Cost-Benefit Analysis of S3 Storage Classes**:

**Storage Cost (per GB/month)**:

- Standard: $0.023
- IA: $0.0125 (45% cheaper storage, but higher retrieval cost)
- Glacier: $0.004 (83% cheaper storage, but retrieval takes hours)
- Deep Archive: $0.00099 (96% cheaper storage, but retrieval takes 12 hours)

**Cost Thresholds for Access Patterns**:

- >1 access/month → Standard
- <1 but >0.1 access/month → IA
- <0.1 but >0.01 access/month → Glacier
- <0.01 access/month → Deep Archive

## Recovery Strategy: The Philosophy of Resilience

### Ontological Thinking about Failures

In a distributed system, failure is not an exception, but the norm. The philosophical attitude of different domains towards failure determines the design of disaster recovery strategies.

#### Financial Trading: A Zero-Tolerance Philosophy of Failure

**Philosophical basis**: Time is money, and any delay is a loss

- RTO requirement: < 30 seconds (users cannot tolerate longer waits)
- RPO requirement: = 0 (any loss of transaction data is unacceptable)

**AWS implementation strategy**:

- Multi-AZ RDS: Synchronous replication, automatic failover
- DynamoDB Global Tables: Multi-region strong consistency
- Lambda@Edge: Edge computing to reduce latency
- Route 53: DNS failover, switch within 30 seconds

**Cost impact**: 200-300% increase in infrastructure costs

#### Family Finance: A Pragmatic Philosophy of Failure

**Philosophical basis**: Life must go on, perfection is not necessary

- RTO requirement: < 4 hours (generally acceptable to wait)
- RPO requirement: < 1 hour (lost records can be re-entered)

**AWS implementation strategy**:

- RDS Automated Backup: Daily backups, point-in-time recovery
- S3 Cross-Region Replication: Cross-region replication
- CloudFormation: Infrastructure as Code for rapid reconstruction
- SNS: Failure notification, setting user expectations

**Cost impact**: 30-50% increase in infrastructure costs

#### Health Monitoring: A Philosophy of Prevention over Cure

**Philosophical basis**: Health data is a matter of life, but continuity is more important than completeness

- RTO requirement: < 2 hours (affects daily monitoring but not fatal)
- RPO requirement: < 30 minutes (short-term data loss is acceptable)

**AWS implementation strategy**:

- IoT Device Shadow: Local caching of device state
- DynamoDB Point-in-time Recovery: Automatic backups
- Timestream: Automatic backup and compression of time-series data
- CloudWatch: Monitoring and alarming, proactive problem detection

**Cost impact**: 50-80% increase in infrastructure costs

### Disaster Recovery Architecture Patterns

#### Pilot Light Pattern: Minimum Viable Backup

**Design philosophy**: A backup light for core functions
**Applicable scenarios**: Medium RTO requirements, cost-sensitive

**Implementation**:

- Key data is continuously replicated to a backup region
- Compute resources are kept at a minimum configuration (or completely turned off)
- In case of failure, quickly start and scale up

**AWS implementation**:

- RDS Cross-Region Read Replica
- S3 Cross-Region Replication
- AMI + Launch Template pre-configuration
- Route 53 health checks trigger failover

**Cost characteristics**: Low cost during normal operation, medium failover time

#### Warm Standby Pattern: The Art of Balancing a Warm Backup

**Design philosophy**: Always ready but not wasteful
**Applicable scenarios**: Higher RTO requirements, controllable costs

**Implementation**:

- The backup environment is kept in a minimum running state
- Key components are started but with a lower load
- In case of failure, quickly scale up to full capacity

**AWS implementation**:

- EC2 instances are kept running but with smaller instance types
- Auto Scaling Group min=1, max=production_size
- ELB health checks for automatic traffic switching
- Lambda pre-warmed containers to avoid cold starts

**Cost characteristics**: Continuous running costs, but fast failover

#### Multi-Site Active-Active Pattern: Seamless Full Redundancy

**Design philosophy**: The sense of security of never being down
**Applicable scenarios**: Zero downtime requirements, not cost-sensitive

**Implementation**:

- Multiple regions provide full service simultaneously
- Load is balanced across all sites
- If any site fails, other sites take over transparently

**AWS implementation**:

- Route 53 Latency-based Routing
- CloudFront Global Distribution
- DynamoDB Global Tables
- Multi-Region VPC Peering

**Cost characteristics**: Highest cost, but best experience

## Concretizing Non-Functional Requirements from the Six Cases

### Case 1: Complete Technical Specifications for the Investment Trading System

**Performance Requirements**:

**API Response Time**:

- Market data query: < 50ms (P99)
- Trade submission: < 100ms (P99)
- Position query: < 200ms (P99)

**Concurrency Requirements**:

- Normal period: 500 concurrent users
- Opening peak: 2000 concurrent users
- System limit: 5000 concurrent users

**Throughput**:

- Normal: 1000 RPS
- Peak: 5000 RPS
- Emergency: 10000 RPS (within 5 minutes)

**Capacity Planning**:

**User Growth Forecast**:

- Current: 10,000 active users
- 6 months: 25,000 active users
- 12 months: 50,000 active users

**Data Growth**:

- Per user per day: 100KB of transaction data
- Annual growth rate: 500GB/year
- 5-year total capacity: 2.5TB + 50% growth buffer = 3.75TB

**Compute Resources**:

- CPU intensive: Risk calculation, technical analysis
- Memory intensive: Real-time data caching
- Network intensive: Market data synchronization

**Disaster Recovery**:

**Key Metrics**:

- RTO: 30 seconds
- RPO: 0 seconds
- Availability: 99.99% (8.76 hours/year downtime)

**Implementation Strategy**:

- Primary: us-east-1 (Multi-AZ)
- Secondary: us-west-2 (Hot Standby)
- Data: DynamoDB Global Tables + RDS Read Replica
- DNS: Route 53 Health Check + automatic failover

### Case 6: Comparison of Technical Specifications for the Travel Sharing Platform

**Performance Requirements**:

**API Response Time (Comparison with Trading System)**:

- Content upload: < 3 seconds (vs 100ms for trading, 300 times more lenient)
- Social interaction: < 1 second (vs 50ms for trading, 20 times more lenient)
- Content browsing: < 500ms (vs 200ms for trading, similar but higher tolerance)

**Concurrency Patterns**:

- Creation peak: Weekend travel periods
- Browsing peak: Weekday evening sharing periods
- Seasonality: 5-10 times difference between high and low travel seasons

**Capacity Planning Philosophy Differences**:

**Growth Model**:

- Trading system: Stable growth, predictable capacity needs
- Travel system: Explosive growth, potential for viral spread

**Data Characteristics**:

- Trading system: Structured data, precise calculations
- Travel system: Multimedia data, storage intensive

**Cost Sensitivity**:

- Trading system: Performance first, cost second
- Travel system: Cost-sensitive, moderate performance

**Disaster Recovery Philosophy**:

**Data Value**:

- Trading system: Every transaction is money, loss is unacceptable
- Travel system: Creative content can be recreated, loss has limited impact

**Recovery Strategy**:

- Trading system: Multi-AZ + real-time backup + zero RPO
- Travel system: Cross-region backup + periodic snapshots + hourly RPO

**User Expectations**:

- Trading system: Professional tool, perfect operation is a basic requirement
- Travel system: Lifestyle tool, occasional failures are understandable

## Tomorrow's Abstract Modeling Preview

Through today's technical boundary confirmation, we have derived specific technical specifications from vague business requirements. We now know:

- How many RPS: A quantitative indicator derived from user behavior
- How much capacity: Storage requirements predicted from growth patterns
- How high availability: Fault tolerance determined by business value

But these technical specifications are still isolated indicators. Tomorrow we will enter the "abstract modeling" stage and learn how to transform these technical constraints into the system's conceptual model, behavioral model, data model, and architectural model.

**Key questions include**:

- How to map DDD's conceptual modeling to AWS services?
- How to design API flows with EventStorming's behavioral modeling?
- How to use data modeling to decide between SQL vs NoSQL?
- How to use architectural modeling to weigh the trade-offs between Serverless vs Container?

## Today's Technical Philosophy Summary

- Non-functional requirements are the quantitative boundaries of functional requirements
- RPS is not a guess, but an inevitable result derived from user behavior
- Capacity planning is a technical mapping of business growth patterns
- Disaster recovery strategy reflects the domain's philosophical attitude towards risk

Remember: Every technical indicator has its business meaning. We are not optimizing technical indicators, but optimizing the efficiency and reliability of the transformation of the user's state of existence.
