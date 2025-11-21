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

- [繁體中文 (Traditional Chinese)](../zh-TW/index.md) - Completed ✓
- [English (英文版)](../en-US/index.md) - Completed ✓
- [日本語 (日文版)](../ja/index.md) - Completed ✓
- [Español (Spanish)](../es/index.md) - Completed ✓

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