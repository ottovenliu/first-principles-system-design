# System Design from First Principles - Complete 30-Day Learning Guide

> **From Philosophical Thinking to Production Practice, Building Systems Worth Running**

Welcome to this in-depth journey of system design! This is not just a technical guide, but also a philosophical exploration of "why".

---

## Learning Path Overview

This 30-day learning journey is divided into **6 major phases**, each of which will lead you to understand system design from a different dimension:

| Phase | Theme | Days | Core Question |
|---|---|---|---|
| **Phase 1** | Philosophical Foundation | Day 1-7 | Why design it this way? |
| **Phase 2** | Technical Architecture | Day 8-16 | How to achieve system goals? |
| **Phase 3** | Quality and Testing | Day 17-20 | How to ensure quality? |
| **Phase 4** | Production Readiness | Day 21-25 | How to run it stably? |
| **Phase 5** | Data and Optimization | Day 26-28 | How to continuously improve? |
| **Phase 6** | Future and Summary | Day 29-30 | How to continuously evolve? |

---

## Complete Article List

### Phase 1: Philosophical Foundation and Domain Modeling (Day 1-7)

**Core Question: What is the purpose of the system? How to divide the domain boundaries?**

<details open>
<summary><strong>Expand to see all articles ▼</strong></summary>

#### [Day 1 | Series Introduction and Guide: Building a Deliverable System Design from Scratch - Using AWS as an Example](./ironMan.draft.d1-0.md)

**Key Concepts**: First-principles thinking, DDD vs BDD, the organicity and purpose of systems, the philosophical value of architects

**You will learn**:

- Why do architects need philosophical thinking?
- The essence of system design: purpose-oriented and domain-driven
- How DDD helps architects avoid getting lost in technology
- The core competitiveness of architects in the AI era

---

#### [Day 2-1 | Requirement Confirmation × System Design Starting Point (1): Transformation of Business Logic](./ironMan.draft.d2-1.md)

**Key Concepts**: Requirement extraction, user cognitive models, the nature of functional requirements

**You will learn**:

- How to extract real requirements from user discourse
- Analysis of system purpose under different cognitive models
- Methods for transforming business logic into technical architecture

---

#### [Day 2-2 | Requirement Confirmation × System Design Starting Point (2): Domain Boundary and Basic Requirement Confirmation](./ironMan.draft.d2-2.md)

**Key Concepts**: Non-functional requirements, performance boundaries, capacity planning, disaster recovery

**You will learn**:

- How to derive API RPS from user behavior
- Capacity planning models for different businesses
- Philosophical thinking on disaster recovery strategies
- Critical point analysis for AWS service selection

---

#### [Day 3 | Requirement Extraction Methodology - Abstract Modeling](./ironMan.draft.d3-0.md)

**Key Concepts**: Abstract modeling, conceptual model, behavioral model, data model

**You will learn**:

- The four levels of abstract modeling
- How to transform domain concepts into system design
- Model validation and iteration methods

---

#### [Day 4 | Building Business Logic with DDD: From Use Cases to Aggregate Design](./ironMan.draft.d4-0.md)

**Key Concepts**: Aggregate, Entity, Value Object, Domain Event

**You will learn**:

- Practical application of DDD tactical patterns
- How to identify aggregate roots and design aggregate boundaries
- Design and timing of using domain events

---

#### [Day 5 | User's System Operation Context - User Story and Scenario Flow](./ironMan.draft.d5-0.md)

**Key Concepts**: User Story, user journey, scenario flow design

**You will learn**:

- How to write effective User Stories
- Methods for drawing user journey maps
- Mapping from scenario flow to API design

---

#### [Day 6 | The Art of Cost Control in Trade-offs: Instance Selection in Cloud Architecture](./ironMan.draft.d6-0.md)

**Key Concepts**: Architectural trade-offs, cost optimization, AWS Instance selection

**You will learn**:

- Trade-off thinking in architectural decisions
- Applicable scenarios for different AWS Instance types
- The art of balancing cost and performance

---

#### [Day 7 | Drawing Your First System Blueprint: Architecture Selection and Design](./ironMan.draft.d7-0.md)

**Key Concepts**: System architecture diagram, C4 Model, architecture documentation

**You will learn**:

- How to draw a clear system architecture diagram
- Hierarchical architecture expression with the C4 Model
- Best practices for writing architecture documentation

</details>

---

### Phase 2: Technical Architecture Design (Day 8-16)

**Core Question: How to map the domain model to technical implementation?**

<details open>
<summary><strong>Expand to see all articles ▼</strong></summary>

#### [Day 8 | Systematizing UI Component Module Design: Introducing Design Systems and Atomic Architecture](./ironMan.draft.d8-0.md)

**Key Concepts**: Atomic Design, design system, front-end architecture

**You will learn**:

- The five levels of Atomic Design
- How to build a maintainable design system
- Combining front-end architecture with DDD

---

#### [Day 9 | High Concurrency and Rate Limiting Design: How to Avoid Resource Bottlenecks](./ironMan.draft.d9-0.md)

**Key Concepts**: Rate limiting strategies, token bucket algorithm, leaky bucket algorithm, concurrency control

**You will learn**:

- Common rate limiting algorithms and their application scenarios
- Rate limiting configuration in AWS API Gateway
- How to design a high-concurrency system

---

#### [Day 10 | The Philosophy of Caching Strategies: The Art of Balancing Time, Space, and Consistency](./ironMan.draft.d10-0.md)

**Key Concepts**: Caching strategies, Cache-Aside, Write-Through, cache invalidation

**You will learn**:

- Applicable scenarios for different caching patterns
- Trade-off thinking on cache consistency
- Strategies for using Redis and ElastiCache

---

#### [Day 11 | Database Design Philosophy: Requirement Analysis, Technology Selection, and Schema Design Strategies](./ironMan.draft.d11-0.md)

**Key Concepts**: Relational vs NoSQL, normalization, Schema design

**You will learn**:

- How to choose the right type of database
- Normalization design for relational databases
- Selection decisions for DynamoDB and RDS

---

#### [Day 12 | Version Control Strategy × Git Flow × Lint Introduction](./ironMan.draft.d12-0.md)

**Key Concepts**: Git Flow, Code Review, Linting

**You will learn**:

- Branch management strategy of Git Flow
- Best practices for PR Review
- Automated code quality checking

---

#### [Day 13 | Cross-Team Collaboration Design: Technical Documents, OpenAPI, Shared Contracts](./ironMan.draft.d13-0.md)

**Key Concepts**: API documentation, OpenAPI Specification, contract testing

**You will learn**:

- How to write OpenAPI specifications
- How to establish API contracts between teams
- Implementation strategies for contract testing

---

#### [Day 14 | Infrastructure as Code: Terraform Infrastructure Codification and Version Control](./ironMan.draft.d14-0.md)

**Key Concepts**: IaC, Terraform, infrastructure version control

**You will learn**:

- The core value of Infrastructure as Code
- Basic syntax and module design of Terraform
- Version management and collaboration for infrastructure

---

#### [Day 15 | Fully Automated CI/CD Implementation - GitHub Actions × CodePipeline](./ironMan.draft.d15-0.md)

**Key Concepts**: CI/CD, automated deployment, Pipeline design

**You will learn**:

- Complete CI/CD Pipeline design
- Integration of GitHub Actions and AWS CodePipeline
- Automated testing and deployment strategies

---

#### [Day 16 | Dev / Staging / Prod Multi-Environment Governance and Architecture Strategy](./ironMan.draft.d16-0.md)

**Key Concepts**: Multi-environment management, configuration management, environment isolation

**You will learn**:

- Governance strategies for multiple environments
- AWS multi-account architecture design
- Configuration management methods between environments

</details>

---

### Phase 3: Quality Assurance and Testing (Day 17-20)

**Core Question: How to verify that the system has achieved its intended purpose?**

<details open>
<summary><strong>Expand to see all articles ▼</strong></summary>

#### [Day 17 | Developer Experience (DX) Optimization: Internal Tools and Troubleshooting Design](./ironMan.draft.d17-0.md)

**Key Concepts**: Developer Experience, internal tools, debugging design

**You will learn**:

- How to improve developer experience
- Design principles for internal tools
- Effective debugging and logging strategies

---

#### [Day 18 | System Acceptance Criteria Formulation: From Verification Logic to Functional Acceptance Manual](./ironMan.draft.d18-0.md)

**Key Concepts**: UAT, acceptance criteria, functional acceptance

**You will learn**:

- How to formulate clear acceptance criteria
- Design and execution of the UAT process
- Methods for writing acceptance documentation

---

#### [Day 19 | UX Testing and Usability Validation: From Observing User Behavior to Correcting Design](./ironMan.draft.d19-0.md)

**Key Concepts**: Usability testing, user research, UX optimization

**You will learn**:

- Methods for conducting usability testing
- How to observe and analyze user behavior
- From test results to design improvements

---

#### [Day 20 | Design Thinking for Testable Systems: A Complete Guide from Components to API Testing](./ironMan.draft.d20-0.md)

**Key Concepts**: Test pyramid, unit testing, integration testing, E2E testing

**You will learn**:

- Design principles of the test pyramid
- Strategies for different levels of testing
- How to design a testable system architecture

</details>

---

### Phase 4: Production Readiness and Reliability (Day 21-25)

**Core Question: Is the system healthy and secure in the production environment?**

<details open>
<summary><strong>Expand to see all articles ▼</strong></summary>

#### [Day 21 | Performance Testing and Load Stress Testing](./ironMan.draft.d21-0.md)

**Key Concepts**: Performance testing, stress testing, bottleneck analysis

**You will learn**:

- Types and methods of performance testing
- How to conduct stress testing
- Identification and optimization of performance bottlenecks

---

#### [Day 22 | The Cornerstone of Modern Security "Zero Trust Architecture"](./ironMan.draft.d22-0.md)

**Key Concepts**: Zero Trust, IAM, VPC, principle of least privilege

**You will learn**:

- Core principles of Zero Trust architecture
- Least privilege design with AWS IAM
- VPC network isolation and security group configuration

---

#### [Day 23 | The Three Pillars of Observability: From Monitoring to Answering Unknown Questions](./ironMan.draft.d23-0.md)

**Key Concepts**: Logs, Metrics, Traces, observability

**You will learn**:

- The difference between observability and monitoring
- Integrated practice of logs, metrics, and traces
- Strategies for using CloudWatch and X-Ray

---

#### [Day 24 | Defining and Measuring Reliability: SRE Method and Error Budget Practice](./ironMan.draft.d24-0.md)

**Key Concepts**: SLI, SLO, SLA, error budget, SRE

**You will learn**:

- Core concepts and practices of SRE
- How to define SLI, SLO, SLA
- Use and management of error budgets

---

#### [Day 25 | Proactive Resilience Verification: Chaos Engineering](./ironMan.draft.d25-0.md)

**Key Concepts**: Chaos Engineering, fault injection, resilience testing

**You will learn**:

- The principles and value of Chaos Engineering
- AWS Fault Injection Simulator in action
- How to design resilience testing experiments

</details>

---

### Phase 5: Data-Driven and Continuous Optimization (Day 26-28)

**Core Question: How to make the system continuously evolve?**

<details open>
<summary><strong>Expand to see all articles ▼</strong></summary>

#### [Day 26 | Data-Driven Product Decisions: From A/B Testing to North Star Metrics](./ironMan.draft.d26-0.md)

**Key Concepts**: A/B Testing, data analysis, North Star metric

**You will learn**:

- How to design effective A/B tests
- A framework for data-driven decision making
- Selection and tracking of North Star metrics

---

#### [Day 27 | Managing Intangible Costs: Technical Debt Identification and Repayment Strategies](./ironMan.draft.d27-0.md)

**Key Concepts**: Technical debt, technical debt quadrant, Boy Scout rule

**You will learn**:

- Types and identification methods of technical debt
- Application of the technical debt quadrant
- Repayment strategies for technical debt

---

#### [Day 28 | Data Governance and Privacy Protection: GDPR Compliance Design](./ironMan.draft.d28-0.md)

**Key Concepts**: Data governance, GDPR, privacy protection, data lifecycle

**You will learn**:

- Core requirements for GDPR compliance
- Data lifecycle management
- Design patterns for privacy protection

</details>

---

### Phase 6: Architectural Evolution and Future Outlook (Day 29-30)

**Core Question: How does the system evolve and grow gracefully?**

<details open>
<summary><strong>Expand to see all articles ▼</strong></summary>

#### [Day 29-1 | The Concerto of Architectural Evolution: Combining the Strangler Fig Pattern and BFF to Achieve an Elegant Transformation of a Monolithic System](./ironMan.draft.d29-1.md)

**Key Concepts**: Strangler Fig Pattern, BFF, microservices migration

**You will learn**:

- Application scenarios of the Strangler Fig Pattern
- Backend for Frontend (BFF) design
- Gradual migration from monolith to microservices

---

#### [Day 29-2 | Socrates Encounters System Architecture: An AI Augmentation Technique for System Designers](./ironMan.draft.d29-2.md)

**Key Concepts**: AI-assisted design, philosophical thinking, system design framework

**You will learn**:

- How to enhance system design capabilities with AI
- Application of the Socratic method
- The mindset of an architect in the AI era

---

#### [Day 30 | Series Finale: A System Design Learning Path and Reflections for Future Engineers](./ironMan.draft.d30-0.md)

**Key Concepts**: Learning path, continuous growth, engineer's philosophy

**You will learn**:

- Recommended learning path for system design
- How to continuously improve architectural skills
- Philosophical reflections for engineers

</details>

---

## How to Use This Guide

### Complete Learning Path (Recommended for Beginners)

1. Start from Day 1 and read in order to Day 30
2. Think about how to apply it to actual projects every day
3. Summarize and reflect after completing each phase

### Thematic In-depth Learning (Suitable for Experienced People)

**Architecture Design Theme**:

- Day 1 (Philosophical Foundation)
- Day 6 (The Art of Trade-offs)
- Day 7 (Architecture Blueprint)
- Day 14 (IaC)
- Day 16 (Multi-Environment Governance)
- Day 29 (Architectural Evolution)

**DevOps & CI/CD Theme**:

- Day 12 (Version Control)
- Day 14 (Infrastructure as Code)
- Day 15 (CI/CD)
- Day 16 (Multi-Environment Management)

**Security & Reliability Theme**:

- Day 22 (Zero Trust Architecture)
- Day 23 (Observability)
- Day 24 (SRE)
- Day 25 (Chaos Engineering)

**Testing & Quality Theme**:

- Day 17 (Developer Experience)
- Day 18 (Acceptance Criteria)
- Day 19 (UX Testing)
- Day 20 (Test Strategy)
- Day 21 (Performance Testing)

**Data & Optimization Theme**:

- Day 26 (Data-Driven Decisions)
- Day 27 (Technical Debt Management)
- Day 28 (Data Governance)

---

## Learning Suggestions

### Recommended Practices

- Spend 30-60 minutes on in-depth reading every day
- Combine with actual projects to think about application scenarios
- Take notes and summarize your own understanding
- Discuss and share learning experiences with your team

### Practices to Avoid

- Skimming without in-depth thinking
- Only looking at conclusions without understanding the principles
- Not practicing but only looking at theory
- Learning in isolation without discussing with others

---

## Other Language Versions

- [English (英文版)](../en-US/index.md) - In preparation
- [日本語 (日文版)](../ja-JP/index.md) - In preparation

---

## Related Resources

- [Domain Index](../index.md) - Back to domain homepage
- [All Book Resources](../../index.md) - View all domain themes
- [Project README](../../../README.md) - Project overview

---

## 💬 Feedback and Contribution

Found an error or have suggestions for improvement? Welcome to:

- 📝 Submit an Issue
- 🔄 Send a Pull Request
- 💭 Share your learning experience

---

**Start your system design journey! 🚀**

> **"The unexamined system is not worth running."**

**© 2025 First Principles System Design**

---
# 從第一性原理出發的系統設計 - 完整 30 天學習指南

> **從哲學思維到生產實踐，建構值得運行的系統**

歡迎來到這趟深度的系統設計之旅！這不僅是一份技術指南，更是一場對「為什麼」的哲學探索。

---

## 學習路徑總覽

這 30 天的學習旅程分為 **6 大階段**，每個階段都將帶你從不同維度理解系統設計：

| 階段 | 主題 | 天數 | 核心問題 |
|---|---|---|---|
| **Phase 1** | 哲學基礎 | Day 1-7 | 為什麼要這樣設計？ |
| **Phase 2** | 技術架構 | Day 8-16 | 如何實現系統目標？ |
| **Phase 3** | 品質測試 | Day 17-20 | 如何確保品質？ |
| **Phase 4** | 生產就緒 | Day 21-25 | 如何穩定運行？ |
| **Phase 5** | 數據優化 | Day 26-28 | 如何持續改進？ |
| **Phase 6** | 未來總結 | Day 29-30 | 如何持續演進？ |

---

## 完整文章列表

### Phase 1: 哲學基礎與領域建模 (Day 1-7)

**核心問題：系統的目的是什麼？領域邊界如何劃分？**

<details open>
<summary><strong>展開查看所有文章 ▼</strong></summary>

#### [Day 1 | 系列開場與導讀：從 0 開始打造可交付的系統設計-以 AWS 為例](./ironMan.draft.d1-0.md)

**關鍵概念**：第一性原理思維、DDD vs BDD、系統的有機性與目的、架構師的哲學價值

**你將學到**：

- 為什麼架構師需要哲學思維？
- 系統設計的本質：目的取向與領域驅動
- DDD 如何幫助架構師避免技術迷失
- AI 時代下的架構師核心競爭力

---

#### [Day 2-1 | 需求確認 × 系統設計起點(一)：商業邏輯的轉化法](./ironMan.draft.d2-1.md)

**關鍵概念**：需求萃取、用戶認知模式、功能需求的本質

**你將學到**：

- 如何從用戶話語中萃取真實需求
- 不同認知模式下的系統目的分析
- 從業務邏輯到技術架構的轉換方法

---

#### [Day 2-2 | 需求確認 × 系統設計起點(二)：領域邊界與基礎需求確認](./ironMan.draft.d2-2.md)

**關鍵概念**：非功能需求、性能邊界、容量規劃、災難恢復

**你將學到**：

- 如何從用戶行為推導 API RPS
- 不同業務的容量規劃模式
- 災難恢復策略的哲學思考
- AWS 服務選型的臨界點分析

---

#### [Day 3 | 需求萃取方法論-抽象建模(Abstract Modeling)](./ironMan.draft.d3-0.md)

**關鍵概念**：抽象建模、概念模型、行為模型、資料模型

**你將學到**：

- 抽象建模的四個層次
- 如何將領域概念轉化為系統設計
- 模型驗證與迭代方法

---

#### [Day 4 | 用 DDD 建構業務邏輯：從用例到聚合設計](./ironMan.draft.d4-0.md)

**關鍵概念**：聚合（Aggregate）、實體（Entity）、值對象（Value Object）、領域事件

**你將學到**：

- DDD 戰術模式的實際應用
- 如何識別聚合根與設計聚合邊界
- 領域事件的設計與使用時機

---

#### [Day 5 | 使用者的系統操作情境 - User Story 與 Scenario Flow](./ironMan.draft.d5-0.md)

**關鍵概念**：User Story、使用者旅程、情境流程設計

**你將學到**：

- 如何撰寫有效的 User Story
- 使用者旅程地圖的繪製方法
- 從情境流程到 API 設計的映射

---

#### [Day 6 | Trade-off 的成本管控藝術：雲端架構的 Instance 選用](./ironMan.draft.d6-0.md)

**關鍵概念**：架構權衡、成本優化、AWS Instance 選型

**你將學到**：

- 架構決策中的權衡思維
- AWS 不同 Instance 類型的適用場景
- 成本與性能的平衡藝術

---

#### [Day 7 | 畫出你的第一份系統藍圖：架構選型與設計](./ironMan.draft.d7-0.md)

**關鍵概念**：系統架構圖、C4 Model、架構文檔化

**你將學到**：

- 如何繪製清晰的系統架構圖
- C4 Model 的層次化架構表達
- 架構文檔的撰寫最佳實踐

</details>

---

### Phase 2: 技術架構設計 (Day 8-16)

**核心問題：如何將領域模型映射到技術實現？**

<details open>
<summary><strong>展開查看所有文章 ▼</strong></summary>

#### [Day 8 | 畫面元件模組設計系統化：設計系統與原子化架構導入](./ironMan.draft.d8-0.md)

**關鍵概念**：Atomic Design、設計系統、前端架構

**你將學到**：

- Atomic Design 的五個層次
- 如何建立可維護的設計系統
- 前端架構與 DDD 的結合

---

#### [Day 9 | 高併發與限流設計：如何避免資源瓶頸](./ironMan.draft.d9-0.md)

**關鍵概念**：限流策略、令牌桶算法、漏桶算法、併發控制

**你將學到**：

- 常見的限流算法與應用場景
- AWS API Gateway 的限流配置
- 如何設計高併發系統

---

#### [Day 10 | 快取策略的哲學：時間、空間與一致性的權衡藝術](./ironMan.draft.d10-0.md)

**關鍵概念**：快取策略、Cache-Aside、Write-Through、快取失效

**你將學到**：

- 不同快取模式的適用場景
- 快取一致性的權衡思維
- Redis、ElastiCache 的使用策略

---

#### [Day 11 | 資料庫設計哲學：需求解析、技術選型與 Schema 設計策略](./ironMan.draft.d11-0.md)

**關鍵概念**：關聯式 vs NoSQL、正規化、Schema 設計

**你將學到**：

- 如何選擇合適的資料庫類型
- 關聯式資料庫的正規化設計
- DynamoDB、RDS 的選型決策

---

#### [Day 12 | 版本控制策略 × Git Flow × Lint 導入思維](./ironMan.draft.d12-0.md)

**關鍵概念**：Git Flow、Code Review、Linting

**你將學到**：

- Git Flow 的分支管理策略
- PR Review 的最佳實踐
- 自動化程式碼品質檢查

---

#### [Day 13 | 跨團隊協作設計：技術文件、OpenAPI、共用契約](./ironMan.draft.d13-0.md)

**關鍵概念**：API 文檔、OpenAPI Specification、契約測試

**你將學到**：

- OpenAPI 規範的撰寫方法
- 如何建立團隊間的 API 契約
- 契約測試的實作策略

---

#### [Day 14 | Infrastructure as Code：Terraform 基礎設施代碼化與版本管控](./ironMan.draft.d14-0.md)

**關鍵概念**：IaC、Terraform、基礎設施版本控制

**你將學到**：

- Infrastructure as Code 的核心價值
- Terraform 的基礎語法與模組設計
- 基礎設施的版本管理與協作

---

#### [Day 15 | CI/CD 全自動化實作 - GitHub Actions × CodePipeline](./ironMan.draft.d15-0.md)

**關鍵概念**：CI/CD、自動化部署、Pipeline 設計

**你將學到**：

- 完整的 CI/CD Pipeline 設計
- GitHub Actions 與 AWS CodePipeline 整合
- 自動化測試與部署策略

---

#### [Day 16 | Dev / Staging / Prod 多環境治理與架構策略](./ironMan.draft.d16-0.md)

**關鍵概念**：多環境管理、配置管理、環境隔離

**你將學到**：

- 多環境的治理策略
- AWS 多帳號架構設計
- 環境間的配置管理方法

</details>

---

### Phase 3: 品質保證與測試 (Day 17-20)

**核心問題：如何驗證系統實現了預期目的？**

<details open>
<summary><strong>展開查看所有文章 ▼</strong></summary>

#### [Day 17 | 開發者體驗（DX）優化：內部工具與排錯設計](./ironMan.draft.d17-0.md)

**關鍵概念**：Developer Experience、內部工具、除錯設計

**你將學到**：

- 如何提升開發者體驗
- 內部工具的設計原則
- 有效的除錯與日誌策略

---

#### [Day 18 | 系統驗收準則制定：從驗證邏輯到功能驗收手冊](./ironMan.draft.d18-0.md)

**關鍵概念**：UAT、驗收標準、功能驗收

**你將學到**：

- 如何制定清晰的驗收準則
- UAT 流程的設計與執行
- 驗收文檔的撰寫方法

---

#### [Day 19 | UX 測試與可用性驗證：從觀察使用者行為到修正設計](./ironMan.draft.d19-0.md)

**關鍵概念**：易用性測試、使用者研究、UX 優化

**你將學到**：

- 易用性測試的執行方法
- 如何觀察與分析使用者行為
- 從測試結果到設計改進

---

#### [Day 20 | 可測試系統的設計思維：從元件到 API 測試全攻略](./ironMan.draft.d20-0.md)

**關鍵概念**：測試金字塔、單元測試、集成測試、E2E 測試

**你將學到**：

- 測試金字塔的設計原則
- 不同層級測試的策略
- 如何設計可測試的系統架構

</details>

---

### Phase 4: 生產就緒與可靠性 (Day 21-25)

**核心問題：系統在生產環境中是否健康且安全？**

<details open>
<summary><strong>展開查看所有文章 ▼</strong></summary>

#### [Day 21 | 性能測試與負載壓力測試](./ironMan.draft.d21-0.md)

**關鍵概念**：性能測試、壓力測試、瓶頸分析

**你將學到**：

- 性能測試的類型與方法
- 如何進行壓力測試
- 性能瓶頸的識別與優化

---

#### [Day 22 | 現代安全基石「零信任架構」](./ironMan.draft.d22-0.md)

**關鍵概念**：Zero Trust、IAM、VPC、最小權限原則

**你將學到**：

- 零信任架構的核心原則
- AWS IAM 的最小權限設計
- VPC 網路隔離與安全組配置

---

#### [Day 23 | 可觀測性三大支柱：從監控到回答未知問題](./ironMan.draft.d23-0.md)

**關鍵概念**：Logs、Metrics、Traces、可觀測性

**你將學到**：

- 可觀測性 vs 監控的差異
- 日誌、指標、追蹤的整合實踐
- CloudWatch、X-Ray 的使用策略

---

#### [Day 24 | 定義與衡量可靠性：SRE 方法與錯誤預算的實踐](./ironMan.draft.d24-0.md)

**關鍵概念**：SLI、SLO、SLA、錯誤預算、SRE

**你將學到**：

- SRE 的核心概念與實踐
- 如何定義 SLI、SLO、SLA
- 錯誤預算的使用與管理

---

#### [Day 25 | 主動式韌性驗證：混沌工程](./ironMan.draft.d25-0.md)

**關鍵概念**：Chaos Engineering、故障注入、韌性測試

**你將學到**：

- 混沌工程的原理與價值
- AWS Fault Injection Simulator 實戰
- 如何設計韌性測試實驗

</details>

---

### Phase 5: 數據驅動與持續優化 (Day 26-28)

**核心問題：如何讓系統持續進化？**

<details open>
<summary><strong>展開查看所有文章 ▼</strong></summary>

#### [Day 26 | 數據驅動的產品決策：從 A/B 測試到北極星指標](./ironMan.draft.d26-0.md)

**關鍵概念**：A/B Testing、數據分析、北極星指標

**你將學到**：

- 如何設計有效的 A/B 測試
- 數據驅動決策的框架
- 北極星指標的選擇與追蹤

---

#### [Day 27 | 管理無形成本：技術債的識別與償還策略](./ironMan.draft.d27-0.md)

**關鍵概念**：技術債、技術債象限、童子軍規則

**你將學到**：

- 技術債的類型與識別方法
- 技術債象限的應用
- 技術債的償還策略

---

#### [Day 28 | 數據治理與隱私保護：GDPR 合規性設計](./ironMan.draft.d28-0.md)

**關鍵概念**：數據治理、GDPR、隱私保護、數據生命週期

**你將學到**：

- GDPR 合規性的核心要求
- 數據生命週期管理
- 隱私保護的設計模式

</details>

---

### Phase 6: 架構演進與未來展望 (Day 29-30)

**核心問題：系統如何優雅地演進與成長？**

<details open>
<summary><strong>展開查看所有文章 ▼</strong></summary>

#### [Day 29-1 | 架構演進的協奏曲：結合絞殺者模式與 BFF 實現單體系統的優雅轉身](./ironMan.draft.d29-1.md)

**關鍵概念**：Strangler Fig Pattern、BFF、微服務遷移

**你將學到**：

- 絞殺者模式的應用場景
- Backend for Frontend (BFF) 設計
- 單體到微服務的漸進式遷移

---

#### [Day 29-2 | 系統架構間遇見蘇格拉底：系統設計者的 AI 增幅術](./ironMan.draft.d29-2.md)

**關鍵概念**：AI 輔助設計、哲學思辨、系統設計框架

**你將學到**：

- 如何用 AI 增強系統設計能力
- 蘇格拉底式詰問法的應用
- AI 時代的架構師思維

---

#### [Day 30 | 系列完結篇：給未來工程師的系統設計學習路線與反思](./ironMan.draft.d30-0.md)

**關鍵概念**：學習路線、持續成長、工程師哲學

**你將學到**：

- 系統設計的學習路徑建議
- 如何持續精進架構能力
- 工程師的哲學反思

</details>

---

## 如何使用本指南

### 完整學習路徑（推薦新手）

1. 從 Day 1 開始，按順序閱讀至 Day 30
2. 每天搭配實際專案思考如何應用
3. 完成每個階段後進行總結與反思

### 主題式深入學習（適合有經驗者）

**架構設計主題**：

- Day 1（哲學基礎）
- Day 6（權衡藝術）
- Day 7（架構藍圖）
- Day 14（IaC）
- Day 16（多環境治理）
- Day 29（架構演進）

**DevOps & CI/CD 主題**：

- Day 12（版本控制）
- Day 14（Infrastructure as Code）
- Day 15（CI/CD）
- Day 16（多環境管理）

**安全與可靠性主題**：

- Day 22（零信任架構）
- Day 23（可觀測性）
- Day 24（SRE）
- Day 25（混沌工程）

**測試與品質主題**：

- Day 17（開發者體驗）
- Day 18（驗收準則）
- Day 19（UX 測試）
- Day 20（測試策略）
- Day 21（性能測試）

**數據與優化主題**：

- Day 26（數據驅動決策）
- Day 27（技術債管理）
- Day 28（數據治理）

---

## 學習建議

### 建議做法

- 每天花 30-60 分鐘深度閱讀
- 結合實際專案思考應用場景
- 做筆記並總結自己的理解
- 與團隊討論分享學習心得

### 避免的做法

- 快速瀏覽不深入思考
- 只看結論不理解原理
- 不動手實踐只看理論
- 孤立學習不與人討論

---

## 其他語言版本

- [English (英文版)](../en-US/index.md) - 籌備中
- [日本語 (日文版)](../ja-JP/index.md) - 籌備中

---

## 相關資源

- [領域總索引](../index.md) - 返回領域首頁
- [所有書籍資源](../../index.md) - 查看所有領域主題
- [專案 README](../../../README.md) - 專案總覽

---

## 💬 回饋與貢獻

發現錯誤或有改進建議？歡迎：

- 📝 提交 Issue
- 🔄 發送 Pull Request
- 💭 分享您的學習心得

---

**開始你的系統設計之旅吧！🚀**

> **"The unexamined system is not worth running."** > **「未經審視的系統不值得運行。」**

**© 2025 First Principles System Design | 從第一性原理出發的系統設計**