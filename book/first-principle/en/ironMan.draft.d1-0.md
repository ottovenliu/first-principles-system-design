# Day 1 | Series Introduction and Guide: Building a Deliverable System Design from Scratch - Using AWS as an Example

## Why Does an Architect Need Philosophical Thinking?

ChatGPT can help you write code, but it can't tell you "what is the purpose of this system's existence." This is precisely the core value of philosophical thinking for an architect.

Before I was an engineer, I had been a lifeguard (a part-time job in high school), a swimming coach (yes, after a short time as a lifeguard, I was given a new task), a data analyst, a mountain guide, a private chef, a digital marketing planner, and even a tarot reader.

Among the many different job contents, I have summarized a context and found that every job and everything you want to do can be carried out according to this context, **purpose-oriented**. Later in the world of software engineering, I learned that this is called "**Domain-Driven Design**".

## The Organic Nature and Purpose of a System

**A system, representing the complete realization of a function, has its philosophical meaning and organic nature.**

Just as the purpose of a living being's life is to perpetuate itself, the birth of every system is to achieve a specific purpose:

- A retrieval system is to find the information you want in massive data.
- AWS Lambda is to let you run code without managing servers.
- API Gateway is to manage all API entry points in a unified way.
- S3 is not just a "cloud hard drive," but to meet the business purposes of "arbitrary expansion, high durability, and low-cost storage."

System design is actually a conceptual biomimicry—we design its ingestion (input), digestion (processing), transformation (calculation), and purpose achievement (output). Just like a human body:

- The digestive system corresponds to the parsing and transformation of input data.
- The nervous system corresponds to Event-Driven Architecture.
- The immune system corresponds to security protection, disaster recovery, and monitoring and alarming.
- The circulatory system corresponds to data flow, message bus, and API calls.

As software engineers, we must design systems strictly according to their "purpose," otherwise disasters will occur.
Imagine a situation that might occur (and in my daily experience, has definitely occurred):

- You have built a perfect API, but it cannot connect with the actual business.
- You spent time optimizing the UI, but the underlying data flow does not meet the user's core needs at all.

This is like an exquisite meal without a main course, or a mountain guide without a correct map. It's not the ingredients that are wrong, but that the designer forgot the "purpose."
As software engineers, we must design systems strictly according to their purpose, otherwise disasters will occur: a gear that does not meet the factory standards, a meal that cannot satisfy the gourmets. It's not the materials that are wrong, but the designer.

## Domain-Driven vs Behavior-Driven: The Architect's Perspective Difference

Many people are used to thinking about systems with BDD (Behavior-Driven Development), designing functions along the user's operating trajectory.

This is not wrong, especially for product validation and rapid iteration at the beginning.

But just as a dense forest grows gradually from a grassland, when the complexity of the system increases, different operating scenarios will become vines that cover the sunlight, the maintenance cost of business logic will increase entropically, and eventually reach the critical point of being abandoned or refactored.

This is why architects need to master the high-level perspective of DDD.

**The value of DDD is that it allows us to overlook the entire system architecture from a higher perspective.**

The value of DDD is not only in "code layering," but also in that it allows us to return to the "purpose":
The domain model is an abstraction of the laws of the real world.
Bounded Context helps us to define "what belongs to this subsystem and what does not."
Aggregate is our complete definition of "things," not just a data table.

When we follow each "domain" to think about the context, design operating scenarios, and converge state changes, we can let developers who are new to the system quickly grasp the rules and not fall into the quagmire constructed by behavioral special cases.

The value of DDD is not only in "code layering," but also in that it allows us to return to the "purpose":

- The domain model is an abstraction of the laws of the real world.
- Bounded Context helps us to define "what belongs to this subsystem and what does not."
- Aggregate is our complete definition of "things," not just a data table.

This is particularly important for AWS architects because there are too many cloud service options. Without the guidance of domain logic, you can easily get lost in the service list.
Here is an example of an AWS architecture:

---

If you only use BDD, you might directly put an EC2 + FTP because the user needs to "upload files."

But if you use DDD, you will ask: "In this domain, what is the purpose and value of the file?"

- Is it just for storage? → S3.

- Does it need version control? → S3 + DynamoDB metadata.

- Does it need to trigger events? → S3 + EventBridge + Lambda.

- Does it need cross-region distribution? → S3 + CloudFront.

---

DDD helps architects avoid seeing only the "behavior" and ignoring the "purpose."

## The Value of an Architect in the AI Era

Especially now, when the explosion of computing power allows AI to generate code at a speed far exceeding manual coding, **the depth of domain knowledge and the grasp of business logic have become the real competitive barriers.** The ability to understand the business domain and purpose, and translate them into architecture and design has become the most important and most needed ability.

This is the core I want to share with you in 30 days: not just how to use AWS tools, but how I use domain-driven thinking to design cloud architecture.

AI can help you complete the syntax details of the code, but it cannot answer for you:

"Should this service be placed in a monolithic architecture or a microservice?"
"Should the database be partitioned? Why is it necessary to partition it from a business perspective?"
"Is cross-border deployment a simple technical option or a requirement of business strategy?"

Behind these questions is philosophical thinking: asking "why," not just "how."

## The Daily Dilemma and Mindset Shift of an Architect

Many people think that the job of an architect is to "draw architecture diagrams" or "select AWS services." In fact, this is just the tip of the iceberg.

In the real world, architects face the following dilemmas:

1. Requirements are always vague

- The business department often says: "I need something like Uber."
- What does this sentence mean? Hailing a car? Payment? Real-time location? Or driver rating?
- The architect must be able to break down the vague vision into clear system boundaries.

2. Technology is always changing

- Today AWS releases a new service, and tomorrow Google Cloud or Azure may provide a cheaper alternative.
- Architects must avoid the trap of "chasing trends" and return to the purpose: Does this service really help us achieve our business?

3. Differences in team cognition

- Front-end engineers want to iterate quickly.
- Back-end engineers want data consistency.
- Operations engineers want high availability.

The architect must be able to find a "purpose-oriented balance point" among these pulls.

## The Intersection of Philosophy and Engineering: From Aristotle to Cloud Native

The ancient Greek philosopher Aristotle proposed the concept of "Final Cause": the reason for a thing's existence is not what it is made of, but what purpose it is to achieve.

The purpose of a knife is to cut.
The purpose of a ship is to sail.
The purpose of a system is to solve a specific business problem.

This is highly consistent with the questions we ask in software architecture.

When you are designing a microservice architecture, you can ask yourself:

**What is the final cause of this service?**

If it's just because **"everyone is using microservices,"** then that's the wrong reason.

If its purpose is **"to support simultaneous use by users across countries and to be able to be deployed independently and quickly,"** then it has a reason to exist.

Philosophy helps us return from "tool worship" to "purpose-oriented."

## Human Factors: Team, Communication, and Decision-Making Philosophy

Architecture is not just technology, but also the common language of the team.

- Communication Philosophy:

- The architect must translate abstract technical concepts into a language that the business can understand.
- At the same time, the business requirements must be translated into a design that engineers can implement.

- Decision-Making Philosophy:

- Many times there is no "best solution," only the "most suitable solution at the moment."
- For example: RDS vs DynamoDB. Choosing RDS may encounter expansion bottlenecks in the future, but choosing DynamoDB requires more consideration for consistency.

The architect must be able to accept "bounded rationality" and continue to iterate.

## 30-Day Complete Learning Path: An Overview of 8 Major Stages

This series will walk through the entire life cycle of a system from scratch. Each stage will delve into the specific practices of AWS, while emphasizing the importance of domain thinking:

### 🎯 Stage 1: Product Ideation and Opportunity Exploration (Day 1-4)

**Core question: What is the purpose of the system?**

- The thinking transformation from business domain to AWS service selection
- Domain modeling with DDD thinking
- How to rethink system boundaries in the cloud era
- Philosophical thinking of the AWS Well-Architected Framework

### 📋 Stage 2: Requirements Definition and Prioritization (Day 5-8)

**Core question: How to divide the domain boundaries?**

- Functional requirements vs. non-functional requirements in AWS
- Domain mapping from user stories to AWS services
- The art of balancing cost-benefit analysis and technical decisions
- The practice of Bounded Context in cloud architecture

### 🎨 Stage 3: Product Design and User Experience (Day 9-12)

**Core question: How do users interact with our domain?**

- API design: from domain language to RESTful implementation
- Domain service design with AWS API Gateway + Lambda
- Decoupling strategy for front-end architecture and back-end domain
- Content distribution architecture thinking with CloudFront + S3

### 🏗️ Stage 4: Technical Planning and System Design (Day 13-18)

**Core question: How to map the domain to the technical architecture?**

- Microservice architecture on AWS and its correspondence with domain boundaries
- Database selection: adaptation of domain model and RDS/DynamoDB
- Event-driven architecture: domain event design with AWS EventBridge
- System resilience thinking for high availability and disaster recovery

### 💻 Stage 5: Software Development and Continuous Integration (Day 19-24)

**Core question: How to make the system grow automatically?**

- Infrastructure as Code: abstraction of domain infrastructure
- CI/CD Pipeline: the philosophy of automation from code to deployment
- Enterprise-level practice of AWS CodePipeline
- Multi-environment governance and domain consistency assurance

### ✅ Stage 6: Validation and Quality Assurance (Day 25-27)

**Core question: How to verify that the system has achieved its intended purpose?**

- Testing strategy: verification from domain logic to system behavior
- Building an automated testing environment on AWS
- System thinking for performance testing and stress testing
- The communication bridge between domain experts and technical implementation

### 🚀 Stage 7: Release and Operational Monitoring (Day 28-30)

**Core question: Is the system healthy in the production environment?**

- Observability: system insight with CloudWatch + X-Ray
- Systematized process for alarm design and incident response
- Risk control for blue-green deployment and canary release
- AWS security best practices and compliance thinking

### 📈 Stage 8: Data Analysis and Continuous Improvement (Day 31-35)

**Core question: How to make the system evolve continuously?**

- Data analysis architecture and domain insight on AWS
- The correspondence between system metrics and business value
- Implementation of A/B testing in cloud architecture
- Philosophical thinking on system evolution strategy and architectural debt

## Future Outlook: The Role of an Architect in the AI and Post-Human Era

When AI can automatically generate systems, will architects be replaced?

The answer is no.

AI can automatically generate code, but it needs "problem definition."

The value of an architect lies in "defining the problem" and "defining the purpose."

In the post-human era, architects are more like "digital philosophers," responsible for answering:

Should this system exist? (Who am I?)
What are its boundaries and responsibilities? (Where am I from?)
What impact will it have on human society? (Where am I going?)

## The Unique Value of This Series

**Domain-driven AWS architecture thinking**: Not only sharing tool usage experience, but also sharing cloud design thinking from a domain perspective.

**Both philosophical depth and technical practice are emphasized**: The "why" behind every technical decision will be explored.

**Unique perspectives from a diverse background**: Combining the risk assessment awareness of a lifeguard, the quality pursuit of a food lover, and the path and schedule planning thinking of a mountain guide.

**Practical experience sharing**: Real project experience with business and context removed, including lessons from pitfalls and successful practices.

## What Will We Gain After 30 Days?

A complete framework for mastering AWS with domain thinking, the core competitiveness that will not be eliminated in the AI era, and the philosophical depth of architectural design starting from the purpose of the system.

More importantly, we will start to overlook the entire system from the high-level perspective of an architect, instead of just focusing on local technical implementation.

Are you ready to start this 30-day journey? Next, we will start with "requirements analysis and business logic transformation" and talk about how AWS architects understand the true purpose of a system's existence.

---

> "The design of a system is conceptual biomimicry. We are not just writing code, but creating digital creatures with purpose and vitality."
