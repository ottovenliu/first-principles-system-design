````markdown
# Day 29-1 | A Concerto of Architectural Evolution: The Graceful Pivot of a Monolithic System with the Strangler Fig Pattern and BFF

In the industry, we are rarely handed a blank slate. Most of the time, we face a "Monolithic System" that has been running for years, carrying core business logic, with its code tangled and complex. It's like an old but still functioning city; we can't just raze it to the ground and build a new one, because the citizens (our users and business) are still living inside.

A junior engineer sees the chaos and their first reaction is: "Oh my god, let's start refactoring! (with joy)"

But for experienced personnel, they've seen enough. "Please, take it away."

Such "Big Bang Rewrite" projects often end up as out-of-control fireworks—dazzling at the start, but ending in a mess, consuming millions of dollars and years of the team's effort, only to deliver a system that is even less stable than the old one.

What we are about to discuss is a more mature, more elegant art, like a conductor leading a complex symphony—System Evolution. The core of this evolution is our theme for today: the concerto of the Strangler Fig Pattern and Backend for Frontend (BFF).

We will explore a powerful and low-risk architectural migration pattern—the Strangler Fig Pattern. Proposed by Martin Fowler, this pattern aims to achieve a smooth transition from a monolith to microservices by gradually and incrementally replacing specific functionalities of the old monolithic application with new microservices.

The core of the Strangler Fig Pattern is to avoid the immense risks and business disruptions caused by a "big bang" refactor. By building a new "facade" (usually an API Gateway) around the monolithic application, we can intercept requests and route those for specific functions to newly developed microservices, while other requests continue to be handled by the old monolith. Over time, the new services will gradually "wrap" and eventually "strangle" the old system. We will learn how to use AWS API Gateway as a proxy to implement this gradual migration.

Backend for Frontends (BFF) is an extremely common optimization pattern in microservice architecture. The BFF pattern advocates for creating a dedicated, lightweight backend service for each different type of frontend client (e.g., web, iOS App, Android App). The sole responsibility of this BFF is to aggregate, tailor, and format data from multiple downstream general-purpose microservices, **crafting a backend API tailored for the frontend experience**.

The BFF pattern solves the "one-size-fits-all, fits-nowhere" dilemma of generic APIs. Mobile clients need streamlined data to save bandwidth, while web clients may need rich data to render complex UIs. By providing each frontend with a dedicated "translator" and "data aggregator," it greatly simplifies frontend development and optimizes the end-user experience.

With the prelude over, let's begin the first movement. (Thanks to the inspiration from the National Theater and Concert Hall's system and the Taipei Symphony Orchestra, whose performance fees are so low it's a steal, all to promote symphonic music).

## First Movement: The Philosophy - Why Choose Evolution Over Reconstruction?

Before we discuss any specific AWS services or design patterns, we must first reach a consensus in our thinking: **Our primary duty is not to build shiny technical temples, but to `ensure the vessel carrying our business can navigate safely and continuously through the ever-changing, turbulent waves`.**

A "Big Bang Rewrite" seems to promise a brand-new, perfect warship, but the truth is, we are asking all passengers (the business) and crew (the team) to jump from the old ship directly onto a new, untried vessel that is still under construction, in the middle of the ocean. The slightest misstep could mean the ship sinks, taking everyone with it.

Therefore, the core skill of a modern cloud architect is Guiding, not Revolution. We are conductors, coordinating new and old instruments to ensure a smooth musical transition; we are gardeners, carefully nurturing the garden to ensure it thrives.

### First Theme: "Carmen" - The Fatal Temptation of a Rewrite

First, we must honestly face and deconstruct why the "rewrite" option is so tempting, especially for engineers.

Bizet's "Carmen" portrays a fatal attraction, passion, and seduction with its fiery, unrestrained melodies, yet the story ends in tragedy. In a way, it's an early tale of a good person's life being ruined by a femme fatale/homme fatal.

This aptly serves as a metaphor for the irresistible temptation of a "rewrite from scratch" for engineers. Before dismissing this option, we must honestly deconstruct the source of its appeal.

1. The Craving for Technical Purity

**The idealistic impulse to build a "perfect" architecture.**

Deep within us often lives an idealist who pursues order and perfection. Faced with an old system full of compromises and historical baggage, an impulse to "format" everything arises. This is an instinctive pursuit of "order" and "perfection," deeply embedded in the DNA of excellent engineers.

Faced with an old system full of technical debt, compromises, and "dirty code," a strong "formatting" impulse arises within us. We yearn to build a perfect technical temple on a pristine "Greenfield," using the most cutting-edge tech stack and the most elegant design patterns, free from any historical constraints.

But do we truly understand all the "hidden rules" within that decade-old monolithic system? The business logic that isn't written in any document, existing only in the mind of a long-gone engineer, could have disastrous consequences if missed. (A classic case: "//Damn, how did this even run?")

**Remember the stringent requirements of the investment trading system we analyzed in Day 2-2?** The system required an API response time of < 100ms, support for 2000+ concurrent queries, and strong data consistency guarantees <Day 2-2 | Requirement Confirmation × System Design Starting Point (II): Domain Boundaries and Basic Requirement Confirmation>. Faced with such demands, technical purism would tempt us to rewrite with the latest tech stack—but this is the most dangerous temptation, as we can easily overlook the "hidden rules" buried deep within the old system.

We must remember:

**`The nature of a business system is "evolution," not "creation."`**

It is **`organic`**, chaotic, and full of compromises made to survive in the real world. The pursuit of "absolute purity" is, in itself, a form of dogmatism detached from reality.

2. The Illusion of "Rewriting is Easier Than Understanding"

**This is a common cognitive shortcut.**

This is a cognitive bias. Understanding a large, complex old system requires the patience of an archaeologist and a detective to excavate, sort, and interpret long-forgotten business logic. This process of unraveling is tangled, frustrating, and slow. In contrast, rewriting a new system feels like the god-like thrill of creation.

The former is a passive, frustrating process of excavation; the latter is an active, fulfilling process of construction. Human nature naturally gravitates towards the latter. However, the cost of this illusion is that we abandon the priceless treasures unearthed during the excavation—the business logic behind the old code, earned through blood, sweat, and tears.

**Day 3 and Day 4 showed us the true difficulty of understanding a system.** In abstract modeling, we learned that a system needs to be understood from four dimensions: conceptual modeling answers what the system is, behavioral modeling answers what the system does, data modeling answers what the system remembers, and architectural modeling answers how the system is organized <Day 3 | Requirement Extraction Methodology - Abstract Modeling>. **The Portfolio aggregation case in Day 4** further revealed the complexity of business invariants: total asset value must equal the sum of all position values plus cash balance, no transaction can result in a negative balance, and risk exposure cannot exceed a preset limit <Day 4 | Building Business Logic with DDD: From Use Cases to Aggregate Design>. These business rules hidden in the code are easily missed during a rewrite, leading to catastrophic consequences.

**And the caching strategies in Day 10 revealed the temporal dimension of system knowledge.** Different types of data have different access patterns and lifecycle characteristics: hot data requires extremely high access frequency, warm data requires moderate access frequency, and cold data can tolerate lower access efficiency <Day 10 | The Philosophy of Caching Strategies>. This deep understanding of the data lifecycle cannot be obtained through a simple "rewrite"; it requires long-term observation and practical accumulation of business knowledge.

There's an old system joke that goes like this:

> An ancient system is like a ruin.
>
> The moment we, like Lara Croft or Indiana Jones, take even a single "artifact" from the ruin,
>
> we summon its guardians—the senior engineers, the business owners, or even the boss—who will hunt us down until the artifact is returned to its original place, or the ruin completely collapses.

3. The Heroism Script

**Our industry culture celebrates innovation and disruption.**

Overthrowing an old system and establishing a new order has a heroic narrative. Leading a major rewrite project is often seen as a golden opportunity to showcase technical vision and leadership, adding a significant achievement to one's resume.

But a long rewrite project is a graveyard for team morale. When the end is not in sight, when pressure mounts, the person who advocated for the rewrite might burn out and leave. Then, it costs even more to find someone to clean up the mess, successfully creating three job opportunities from one position, boosting both GDP and employment rates.

**In Day 12's version control strategies**, we discussed the conflict between agile thinking and heroism: excellent engineers often pursue order and perfection, wanting to be the star of the show, not just a supporting player who quietly passes the ball <Day 12 | Version Control Strategy × Git Flow × Lint>. This mentality is the source of the "heroism script"—we all want to be the hero who "saves the system."

**But the harsh reality of cost control in Day 6 tells a different story.** System development and maintenance require continuous human and financial investment, including engineer salaries, infrastructure costs, and long-term operational expenses <Day 6 | The Art of Trade-offs and Cost Control>. The true cost of a rewrite project is not just the technical investment, but also the long-term drain on human resources and opportunity cost. The advocate who "ran away" leaves behind a tired team and unfulfilled promises.

True superiority is not demonstrated by tearing everything down with vast resources and risk, but by completing a smooth, elegant system transformation with surgical precision and quiet wisdom, under extremely complex and constrained conditions.

### The "1812 Overture" of a Big Bang Rewrite

Choosing a "Big Bang Rewrite" is like pushing our project onto a battlefield ablaze with cannon fire. We must face the following four deadly cannonballs head-on:

**Cannonball I: The Value Vacuum**

**`Time is the absolute, most expensive, and non-renewable resource in modern business warfare.`**

Imagine reporting to the CEO: "For the next two years, my team will not contribute any new features or business growth to the company. Please continue to invest millions of dollars in resources." How absurd does that sound? I must pay my highest respects to anyone who can say this; they have achieved the feat of dancing gracefully in a minefield—trying to snatch a single gold coin from a dragon.

In a rapidly changing market, a two-year "value vacuum" is enough for competitors to leave us far behind. A rewrite project lasting one to two years means that for this long period, the team cannot deliver any new features or value to the business. It becomes a black hole that continuously swallows company resources (time, money, top talent) with no output. In the fast-paced market competition, this "value vacuum" is absolutely fatal.

**The cost analysis in Day 6 shows us the real business impact.** The investment trading system has extremely high availability requirements, as a trading interruption can lead to huge user losses, damage to brand reputation, and regulatory compliance risks <Day 6 | The Art of Trade-offs and Cost Control>. If we spend two years rewriting the system, we can't perform any business optimization during this period. Every unimplemented feature request could mean tens of thousands of dollars in opportunity cost.

**The user scenarios for the family finance system in Day 5** further remind us of the urgency of value delivery: users need budget monitoring, automatic alerts, and real-time financial status updates <Day 5 | User's System Operation Scenarios - User Story and Scenario Flow>. Users will not wait two years for these critical features; they will switch to a competitor.

**Cannonball II: The Unbounded Risk**

**`This is an all-or-nothing gamble.`**

This is not just a technical risk, but a business risk. The old system is like a battle-hardened veteran, covered in scars but still fighting; the new system is like a rookie who has only trained in a simulator and never seen real combat. Replacing the veteran with the rookie in the most critical battle, all at once, is not courage; it's gambling (classic examples: Waterloo and Ma Su). Any tiny, unforeseen scenario (like a specific client's peculiar data format) could cause the entire new system to collapse.

All resources and expectations are bet on a single final delivery date. There are no intermediate validation points, no gradual risk release, and no chance to go home. Any butterfly flapping its wings could cause the entire project to be delayed or even fail, and the cost of failure is catastrophic, with no turning back.

**Remember the disaster recovery requirements for different systems in Day 2-2?** The investment trading system required an extremely short Recovery Time Objective (RTO < 30 seconds) and zero data loss (RPO = 0); the health monitoring system also had strict RTO and RPO requirements <Day 2-2 | Requirement Confirmation × System Design Starting Point (II): Domain Boundaries and Basic Requirement Confirmation>. A big bang rewrite means that at the moment of switchover, we must ensure the new system can perfectly meet these extremely stringent requirements. If it fails, a 30-second RTO means we have no buffer time, and the losses will be immediate and massive.

**The risk of a cache stampede discussed in Day 10** is infinitely amplified in a big bang rewrite: when a large number of caches expire simultaneously, all requests will hit the database directly, causing the system to crash; the cache for hot data expiring will also cause a large number of concurrent requests to focus on the same data point <Day 10 | The Philosophy of Caching Strategies>. In a gradual migration, these risks can be exposed and fixed step by step; but in a big bang rewrite, all risks will erupt at the same time, forming a catastrophic chain reaction.

**Cannonball III: The Knowledge Drain**

**`The old system contains a large amount of undocumented, implicit business rules and boundary conditions.`**

The rewrite process will almost inevitably lead to the permanent loss of this valuable knowledge, and the problems that were already solved will reappear in the new system.

A five-year-old monolithic system is a living fossil of the company's five years of business logic. Those strange if-else statements might correspond to a special requirement from a big client signed by a long-gone sales champion; that seemingly redundant data table might be the key to the finance department's survival during quarterly settlements. A rewrite is an organizational memory wipe. We think we're getting rid of technical debt, but in reality, we're throwing away valuable business assets along with it.

The old system contains a large amount of undocumented, implicit business rules and boundary conditions. The rewrite process will almost inevitably lead to the permanent loss of this valuable knowledge, and the problems that were already solved will reappear in the new system.

**The state transition logic of the Portfolio aggregate in Day 4** is a typical example of this implicit knowledge: from the initial state to active trading, risk management, and final liquidation, each state transition has complex business rules and boundary conditions behind it <Day 4 | Building Business Logic with DDD: From Use Cases to Aggregate Design>. The design of these state machines is often stabilized after countless online issue fixes. When rewriting, we can easily miss some rare but critical state transition paths.

**The multi-role permission design in Day 5** further reveals the complexity of knowledge: different roles (trader, risk manager, compliance officer) have different levels of system access and operational scope <Day 5 | User's System Operation Scenarios - User Story and Scenario Flow>. Behind these permission matrices often lie internal company politics, compliance requirements, and even painful lessons from past incidents. During a rewrite, all this context disappears, and we have to step on all the same landmines again from scratch.

**Cannonball IV: The Organizational Exhaustion**

**`In the long run, regardless of the project's final success or failure, the organization's vitality and trust are severely damaged.`**

A rewrite creates an "A/B team" dilemma. The best engineers are assigned to the "delicious and nutritious" new project (Team A), while others are left to do the thankless maintenance work on the old system (Team B). Team B will feel abandoned and demoralized; Team A will carry the weight of unknown future expectations and immense pressure. In the end, the project is delayed, internal conflicts are constant, and regardless of success or failure, the organization is already severely weakened.

Excellent engineers are basically romantic idealists who pursue order and perfection. None of them want to be constantly passing the ball without ever aspiring to be the champion who scores the winning goal in the spotlight.

A long-cycle rewrite project splits the team (maintaining the old system vs. developing the new one) and puts all participants under immense psychological pressure, eventually leading to talent loss and the collapse of team morale.

**The cross-team collaboration design in Day 13** reveals the exponential growth of communication costs in large projects: as the team size expands, we need clear API contracts, shared architectural documents, and detailed decision records to ensure smooth collaboration <Day 13 | Cross-Team Collaboration Design>. In a big bang rewrite, the coordination cost between Team A and Team B becomes extremely high, because they are not only working on different codebases but also in different psychological states.

**The multi-role collaboration scenario in the family finance system in Day 5** also hints at the risk of organizational division: the system has different roles such as decision-makers, executors, and coordinators, each with its own responsibilities and permission boundaries <Day 5 | User's System Operation Scenarios - User Story and Scenario Flow>. When Team A becomes the "decision-maker" and Team B becomes the "executor," this unequal power structure will quickly erode team cohesion, eventually making the ghost story of the "runaway" advocate a reality.

Tchaikovsky used real cannons in the finale of his "1812 Overture." The sound is magnificent and shocking, but also filled with destructive power. The four cannonballs mentioned above are the fire we will inevitably face when we choose a "Big Bang Rewrite."

### The Gardener's Ethos: Core Principles of Evolutionary Architecture - "From the New World"

After revealing the terror of "revolution," we formally propose the philosophy of "evolution."

Dvořák's "New World Symphony" was composed during his stay in America, blending his observations of the new continent with his longing for his homeland. It is not a complete break from the past, but a new life for the old soul in new soil. This is the philosophy of our evolutionary architecture.

An excellent architect is more like a gardener. A gardener doesn't level the entire garden but prunes, grafts, and fertilizes on the existing foundation, guiding the garden to grow in a healthier, more beautiful direction.

**Principle One: Deliver Value Continuously**

This is the "heartbeat" of evolutionary architecture, the blood that sustains the project's life.

Every step of evolution, no matter how small, must bring perceptible value to the user or the business. We no longer pursue a perfect, distant utopia, but focus on the next most valuable, smallest, deliverable improvement. Today, we optimize an API to make the app 100 milliseconds faster; next week, we split off a reporting service to stop the backend from lagging. Every small success injects vitality into the entire system and wins trust and resources for the team, creating a positive **"flywheel effect."**

**The User Story design in Day 5** emphasizes the concreteness of value delivery. The story for the investment trading system requires professional traders to get a complete position risk analysis within thirty seconds to make quick decisions during market volatility <Day 5 | User's System Operation Scenarios - User Story and Scenario Flow>. This is not an empty technical metric, but real business value—every evolution should shorten this "30 seconds," make decisions more precise, and make risks more controllable. **The domain-driven thinking in Day 1** further reminds us: technology choices must serve business purposes. Adopting a microservice architecture simply because it's a trend is an irresponsible decision <Day 1 | Series Kick-off and Introduction: Building a Deliverable System Design from Scratch - with AWS as an Example>.

**Principle Two: Safety First**

This is a professional engineer's "Hippocratic Oath" - _"Primum non nocere" (First, do no harm)._

All operations must be performed with a safety net. No operation can harm the stability of the existing business. The Strangler Fig Pattern, Blue-Green Deployment, Canary Releases... behind these technical patterns lies this sacred philosophical principle. We must have the utmost respect for the production environment, like a bomb disposal expert.

**The CI/CD full automation implementation in Day 15** is a concrete practice of this principle. By establishing a multi-stage deployment process, automated health check mechanisms, and rapid rollback capabilities, we build a complete safety net for system evolution <Day 15 | CI/CD Full Automation Implementation>. **The Infrastructure as Code in Day 14** further ensures that all infrastructure changes have complete version traceability and rollback mechanisms, allowing the team to clearly grasp the content and impact of every change <Day 14 | Infrastructure as Code (Terraform)>. These practices allow us to evolve boldly because we know that even if we fail, we can recover in minutes.

**Principle Three: Build Feedback Loops**

Evolution is not blind. We must let the system "speak for itself."

We use data (like latency, error rates) to verify whether our changes have truly produced positive benefits. Comprehensive monitoring (Metrics), logging (Logging), and tracing (Tracing) equip our system with eyes and ears. Data is our only source of truth. Every architectural decision we propose is a "scientific hypothesis" (e.g., "Splitting out the user service will reduce the latency of order creation"), and the online monitoring data is the only standard for verifying this hypothesis—**driving every decision with reason and data**.

**The high concurrency and rate limiting design in Day 9** demonstrates the importance of feedback loops: the system needs to continuously monitor key metrics such as request volume, response time, and error rate, and dynamically adjust rate limiting strategies based on this data to maintain service stability <Day 9 | High Concurrency and Rate Limiting Design>. During evolution, these metrics will tell us whether the new service is truly better than the old system. **The caching strategy analysis in Day 10** further emphasizes the importance of a monitoring system: cache hit rate, potential stampede risks, and data consistency latency are all key data points that guide our continuous optimization of caching strategies <Day 10 | The Philosophy of Caching Strategies>. Without these feedback mechanisms, our evolution is just blind gambling.

**Principle Four: Embrace the Imperfect Transition**

During the process of evolution, the system will inevitably be "ugly." New and old code coexist, data needs to be synchronized in both directions, and the routing rules of the API Gateway may be complex. But this "chaotic" transitional state is a sign of healthy evolution, not a symbol of failure. We must accept and learn to manage the "chaos" of this intermediate state. What we pursue is not static perfection, but dynamic beauty full of vitality—this city is always under good construction, and that is the proof of its vitality.

**The version control strategy in Day 12** taught us how to maintain order in chaos: using the Feature Branch strategy allows new and old code to be developed in parallel, while a rigorous Pull Request review process ensures that code quality does not decline during the transition period <Day 12 | Version Control Strategy × Git Flow × Lint>. During the transition period of system evolution, we need to carefully handle data synchronization and cache consistency issues when new and old systems coexist, accepting the additional complexity and cost that will inevitably arise during this time.

**The philosophy of system organicity in Day 1** provides the philosophical basis for this principle: the nature of a business system is continuous evolution, not a one-time creation. It is organic, chaotic, and full of pragmatic compromises made to survive in the real world <Day 1 | Series Kick-off and Introduction: Building a Deliverable System Design from Scratch - with AWS as an Example>. If we can accept that the system itself is a product of organic evolution, we can more calmly face the "imperfection" of the evolution process—this is not failure, but proof of vitality.

## Second Movement: The Macro-Strategy - The Strangler Fig Pattern

Imagine an old tree in the rainforest (our monolithic system). The seed of a strangler fig (our new service) lands on its branches, sending roots down and sprouting upwards.

At first, it is small, clinging to the old tree. But over time, its vines grow more numerous and thicker, eventually forming a net that completely envelops the old tree, absorbing all the sunlight and nutrients. Finally, when the old tree withers and rots away, the strangler fig has taken its place, becoming an independent, thriving new tree.

This is our macro core strategy with 3 key points:

1.  **Identify**: In the old city of the monolith, find the first district to renovate. The boundaries of this district must be clear, such as "user profile management," "product catalog query," or "mobile API service."
2.  **Intercept & Replace**: Build a new microservice (the first vine) to handle the business of this district. Then, set up a "traffic police" at the city entrance.
3.  **Strangle**: As more and more districts are replaced by new services, the vines weave into a dense, impenetrable net. The core functions of the old city are almost all bypassed. At this point, it can be safely "turned off," completing the migration.

The subtlety of this process lies in its gradualness and quiet security. Throughout the process, the system continues to provide services to the outside world, and users may not even perceive that such a drastic structural change is happening internally.

### Step One: Identify the Seam

How to find the place to make the "first cut" on the monolithic system.

The selection criteria are: clear business boundaries (e.g., user, product, order), high value of change, and relatively low coupling with other parts of the system. You can call this finding the first "beachhead."

The seed will first grow a vine, rooting downwards. This is our first step: in the vast monolithic system, find a clear, valuable business boundary. This boundary might be a functional module (e.g., user authentication) or a specific user scenario (e.g., mobile device API).

**This identification process is the practical application of "Service Boundary Identification" that we emphasized in architecture selection and design.** `Through DDD's Bounded Context analysis, we can identify which business functions have clear domain boundaries, independent business logic, and minimal external dependencies` <Day 7 | Architecture Selection and Design>. At the same time, **the OpenAPI specification and contract-first design discussed in Day 13** provide us with concrete tools to define these boundaries: `before starting to split, we need to use OpenAPI to clearly define the API contract between the new and old systems, ensuring the clarity and testability of the boundary` <Day 13 | Cross-Team Collaboration Design>.

Furthermore, **database-level boundary identification is equally crucial**. As discussed in Day 11, `the data layer is often the most difficult part of a monolithic system to split. We need to analyze the data dependency relationships of the Aggregate Root to identify business entities that can independently manage their data lifecycle` <Day 11 | Database Design Philosophy>.

### Step Two: Gradually Replace

Build a new microservice to implement the identified functionality.

The vines will grow more numerous (we develop more new services), and they will entwine around the trunk of the old tree, absorbing sunlight and nutrients. In our world, this means we will place a traffic control center, setting up a proxy or gateway to "intercept" specific requests pointing to that function and "reroute" them. It acts like a smart router, directing requests for new functions (e.g., /api/v2/...) to the nascent service, while the rest of the requests continue to be handled by the old tree (the monolithic system).

At this point, the new and old systems are in a state of "coexistence," quietly providing services to the outside world together.

**This gradual replacement process requires us to integrate multiple previously discussed technical capabilities.** First, **the art of cost control in Day 6 provides us with a decision-making framework for service selection**. In the case scenarios at that time, we faced the dilemma that the investment trading system required a response time of less than 100 milliseconds, but Lambda cold starts could exceed 100 milliseconds; the family finance system was extremely cost-sensitive, but the minimum ECS configuration of $58 per month was still over budget; and the health monitoring system's IoT data stream needed to run for a long time, but Lambda had a 15-minute execution limit <Day 6 | The Art of Trade-offs and Cost Control>.

These seemingly unsolvable contradictions actually reveal the essence of service selection: different systems have vastly different cost models and optimization goals. The investment trading system adopts a performance-first model, with low cost sensitivity, where minimizing latency takes precedence over maximizing availability, which in turn takes precedence over cost control; it's better to over-provision than to have insufficient performance. The family finance system uses a cost-sensitive model, with high cost sensitivity, where cost control takes precedence over meeting basic functions, which takes precedence over performance improvement. The health monitoring system uses a balanced optimization model, with medium cost sensitivity, where stability takes precedence over cost control, which takes precedence over advanced features <Day 6 | The Art of Trade-offs and Cost Control>.

For a lightweight aggregation layer like a BFF, Lambda is usually the ideal choice; for services that need to run for a long time or have complex state management, ECS may be more suitable. Furthermore, when we consider multi-region deployment, the decision complexity increases exponentially: the investment trading system adopts a multi-region deployment in the US East, Western Europe, and Northeast Asia. Although the infrastructure cost increases by 200% and cross-region data transfer costs $0.02 per GB, the latency drops from 150ms to 20ms between the US and Europe, and from 200ms to 30ms between the US and Asia, achieving an annual ROI of 15:1. The family finance system chooses a single-region plus CDN solution, with a regional cost of $100 per month compared to $400 per month for multi-region, plus $20 per month for CDN, saving 70% in total costs. The health monitoring system adopts a tiered strategy, with a core single region costing $500 per month plus $200 per month for processing in each region, totaling $1100, which is more cost-effective than a fully multi-region setup at $1500 per month <Day 6 | The Art of Trade-offs and Cost Control>.

**The Infrastructure as Code practice in Day 14 is crucial**. In the era before infrastructure as code, we faced three fatal dilemmas: configuration drift leading to untraceable manual adjustments and inconsistent behavior between testing and production environments; uncertainty in disaster recovery, where a crash at 3 a.m. would require six hours to rebuild, relying entirely on personal memory to recall all manual configuration steps; and scaling bottlenecks, where requesting resources took three days and configuring the environment took five, making it impossible to respond quickly to market demands <Day 14 | Infrastructure as Code (Terraform)>.

Even more terrifying is the risk of knowledge concentration: a newcomer asks how to deploy the code, and a senior engineer has to recall SSHing into the server, then git pulling, then recompiling, remembering to restart nginx, and clearing the cache, oh, and don't forget to back up the database... When the key person leaves, no one knows the complete deployment process, and the production environment configuration exists only in someone's head <Day 14 | Infrastructure as Code (Terraform)>.

Therefore, we need to use Terraform to manage the infrastructure of the new services, including the routing rules of the API Gateway, the configuration of Lambda functions, the definition of ECS services, etc., to ensure the versioning and rollback capability of the infrastructure. Specifically, the production environment needs to be configured with `instance_type` as `t3.medium`, a minimum size of 2, a maximum size of 10, and a desired capacity of 3, with a database instance class of `db.t3.medium` and 100GB of allocated storage. The development environment, on the other hand, is configured with `instance_type` as `t3.micro`, a minimum size of 1, a maximum size of 3, and a desired capacity of 1, with a database instance class of `db.t3.micro` and 20GB of allocated storage <Day 14 | Infrastructure as Code (Terraform)>.

More critically, **the CI/CD full automation implementation in Day 15 provides us with a secure deployment mechanism**. Remember those horror stories of manual deployment? Deploying a new feature at 5 p.m. on a Friday, with colleagues collectively trying to stop you, saying "don't deploy on Friday, if something goes wrong, we'll have to work overtime through the weekend"; the disaster of inconsistent environments, where "it works on my machine" but there are bugs in the test environment, and the production environment is different from the test environment; the black box of the deployment process, where a newcomer asks how to deploy, and a senior engineer has to recall N steps like SSH, git pull, recompile, restart nginx, clear cache, back up the database, etc. <Day 15 | CI/CD Full Automation Implementation>. To avoid these ghost stories from repeating, we need to establish a complete GitHub Actions fragmented management process: the first layer of basic checks runs `code-quality` with a 10-minute timeout for fast failure; the second layer of testing runs `unit-tests`, `integration-tests`, and `e2e-tests` in parallel with 15 to 30-minute timeouts and uses a matrix strategy for multi-version testing; the third layer of security and quality checks runs `security-scan` with a 15-minute timeout, including SAST and dependency scanning; the fourth layer of build and deployment preparation runs `build` with a 10-minute timeout to generate artifacts; the fifth layer of environment deployment runs `deploy-staging` and `deploy-production` with a 10-minute timeout, triggered based on environment conditions <Day 15 | CI/CD Full Automation Implementation>. Through a multi-stage approval process and automated traffic switching, we can achieve a gradual traffic migration from 10% → 50% → 100%, and quickly roll back if problems are found at any stage.

During this transition period, **the caching strategy in Day 10 becomes particularly important**. Since the new and old systems may need to temporarily maintain dual writes, we must face the core dilemma of cache consistency: the investment trading system requires strong consistency but will sacrifice 50% of performance; the family finance system accepts eventual consistency but collaboration conflicts are difficult to resolve; the health monitoring system requires causal consistency but the implementation is extremely complex and requires techniques like vector clocks <Day 10 | The Philosophy of Caching Strategies>.

But we must be wary of cache stampede and penetration disasters: a large number of caches expire at the same time, for example, due to a service restart or the same TTL setting, and a flood of requests instantly overwhelms the database; the cache for hot data happens to expire, and countless requests bypass the cache and hit the same data point in the database, like using a magnifying glass to focus sunlight and easily burn a hole <Day 10 | The Philosophy of Caching Strategies>. We must handle cache invalidation strategies with care to ensure eventual data consistency between the two systems.

Finally, **the version control strategy in Day 12 helps us manage this complex parallel development**. Feature Branch parallel development can lead to merge hell: multiple feature branches are developed simultaneously, and when merging to `develop`, a large number of conflicts occur; subfeatures must first be merged into the feature, the feature must first be merged into `develop`, `develop` must be tested before a release branch can be created, the release branch must be stable before it can be merged into `main`, and any failure in the process will block the entire pipeline <Day 12 | Version Control Strategy × Git Flow × Lint>.

To ensure code quality, we need to establish a three-party review threshold: a peer review threshold to confirm the architecture and style and avoid confusion between `service` and `repository`; a business review threshold to confirm the most perfect business execution rate in a single environment; and a quality review threshold to verify the business execution success rate with maximum coverage in multiple environments <Day 12 | Version Control Strategy × Git Flow × Lint>. We need to maintain fixes for the monolithic system and the development of new microservices in the same codebase, making the Feature Branch strategy and a strict PR review process crucial.

### Step Three: Continue the Strangulation

Repeat the first two steps, gradually migrating more functionality to new microservices, letting the new "vines" grow more and more numerous.

Years later, the fig vines have become so thick that they form a new, independent trunk structure, while the old tree inside, no longer receiving sunlight, eventually withers, rots, and disappears. In our system, when all traffic has been directed to the new services, the core responsibilities of the old monolithic system are gradually hollowed out, and the old system can be safely decommissioned, completing its historical mission (the old tree withers naturally).

**This continuous evolution process requires us to have comprehensive system thinking and a long-term technical vision. The high concurrency and rate limiting design discussed in Day 9 plays a key role here.** If we do not properly handle concurrency bottlenecks, we will face the disaster of failing to identify application layer bottlenecks: connection pool resource exhaustion leading to all new requests being blocked; task queue backlogs increasing from 1 minute to 10 minutes; memory GC pauses causing P99 latency to deteriorate from 50 milliseconds to 5000 milliseconds <Day 9 | High Concurrency and Rate Limiting Design>.

Even more terrifying is the database layer bottleneck disaster: the slow query ratio suddenly soars, with a single SQL query time deteriorating from 10 milliseconds to 10000 milliseconds; the index hit rate drops, causing full table scans and QPS to fall from 5000 to 50; lock waits and deadlocks cause a chain reaction, blocking all transactions and completely paralyzing the system; the connection pool usage reaches 100%, and all new connections time out <Day 9 | High Concurrency and Rate Limiting Design>.

Therefore, as more and more traffic is directed to new services, we need to design appropriate rate limiting strategies for each new service, using Token Bucket or Sliding Window algorithms to protect the services from sudden traffic spikes, especially at the critical moment of traffic switching. In AWS architecture choices, the compute layer can use ECS Fargate for serverless container auto-scaling cost-effectiveness, or Lambda for event-driven extreme elasticity with pay-per-use, or ECS on EC2 for full control and cost optimization for stable loads. The storage layer can use RDS or Aurora for relational ACID complex queries, or DynamoDB for NoSQL infinite scaling and microsecond latency, or S3 with Athena for data lake analytics and optimal query costs. The caching layer can use ElastiCache for distributed Redis or Memcached high availability, or DAX for DynamoDB acceleration with transparent caching and microsecond latency, or CloudFront for global CDN edge caching of static content <Day 9 | High Concurrency and Rate Limiting Design>.

**The atomic architecture thinking in Day 8 helps us understand the coordinated evolution of the frontend and backend.** But in practice, we will encounter the dilemma of blurred boundaries in frontend-backend collaboration: the business logic application section of the `service` and the external system interface section of the `repository` are confused and misused. Although the functionality is normal, as the code grows, the logic becomes difficult to replace cleanly, turning into a "shit mountain" <Day 8 | Designing Systems and Atomic Architecture>.

To solve these dilemmas, we need to adopt an Atomic Design hierarchical implementation: the Atoms layer corresponds to UI expressions of domain value objects like `StatusBadge`, `QRCode`, `PriceDisplay`; the Molecules layer corresponds to `TicketInfo`, combining related value objects into meaningful UI fragments; the Organisms layer corresponds to `TicketCard`, implementing complete business functions corresponding to the core capabilities of an aggregate; the Templates layer corresponds to `TicketRedemptionWorkflow`, for cross-aggregate business processes; the Pages layer corresponds to `CoffeeRedemptionPage`, for a complete operation in a specific role's context <Day 8 | Designing Systems and Atomic Architecture>. When we split the backend services into microservices, the frontend component architecture also needs to be adjusted accordingly. The BFF pattern is the product of this redefinition of frontend-backend boundaries. It maps the backend's Aggregate Roots to frontend-friendly APIs, achieving an elegant bridge from the domain model to UI components.

**The abstract modeling and DDD aggregate design discussed in-depth in Day 3 and Day 4** provide us with the theoretical basis for continuous evolution. But there is a core dilemma in identifying domain boundaries: excessive subdivision leads to aggregates that are too small, creating a distributed transaction hell; excessive merging leads to aggregates that are too large, violating the single responsibility principle; chaotic context mapping leads to teams having different understandings of the same concept <Day 3 | Requirement Extraction Methodology - Abstract Modeling>. Every new service split requires a re-examination of the domain boundaries to ensure that what we are extracting is not just a technical module, but a business-meaningful aggregate root. From conceptual modeling to behavioral modeling, from data modeling to architectural modeling, the four-fold modeling framework ensures that every evolution is the result of careful consideration <Day 3 | Requirement Extraction Methodology - Abstract Modeling>, <Day 4 | Building Business Logic with DDD: From Use Cases to Aggregate Design>.

**The user operation scenario analysis in Day 5 reminds us** that the ultimate purpose of system evolution is to create value for the user. The User Story must include clear technical constraint requirements: the investment trading system needs a response time of less than 100 milliseconds, 2000+ TPS concurrency, strong consistency, and complex transaction states; the family finance system needs to be cost-sensitive, handle collaboration conflicts, have eventual consistency, and intermittent use; the health monitoring system needs to handle IoT data streams, growing device numbers, temporal consistency, and background analysis tasks <Day 5 | User's System Operation Scenarios - User Story and Scenario Flow>. Every microservice that is split out should correspond to a clear User Story and operation scenario, ensuring that changes in the technical architecture can be translated into improvements in user experience or enhancements in business capabilities.

Ultimately, **this three-step evolution framework is the culmination of all the content we have learned from Day 1 to Day 15.** It proves that the elegant evolution of a system architecture does not rely on a single technical solution, but requires the integration of knowledge from multiple dimensions, including domain modeling, user experience design, cost trade-offs, architectural patterns, performance optimization, data management, engineering practices, and automated infrastructure, to achieve a truly sustainable system transformation <Day 1 | Series Kick-off and Introduction: Building a Deliverable System Design from Scratch - with AWS as an Example>.

**And the business logic transformation methodology discussed in Day 2** further reminds us that behind every architectural evolution, there should be clear business value support. We are not doing microservices for the sake of microservices, but to better serve business needs, reduce the cost of change, and improve team efficiency. The three-layer analysis of requirements from the phenomenon layer, essence layer, to existence layer helps us understand the true motivation behind every evolution decision <Day 2-1 | Requirement Confirmation × System Design Starting Point (I): The Transformation of Business Logic>.

## Third Movement: The Precise First Strike - BFF as the Beachhead for the Strangler Strategy

### The Core Philosophy of Backend for Frontend (BFF)

We must first return to the origin and thoroughly understand the "original intention" of BFF.

Forget that it's a backend technology. Philosophically speaking, **BFF is essentially a frontend concept**. Its birth was to solve an increasingly sharp contradiction: `the contradiction between a One-Size-Fits-All API and the increasingly diverse frontend experiences`.

Imagine our monolithic system, which provides a set of generic RESTful APIs. This set of APIs is like a one-size-fits-all T-shirt, trying to make everyone wear it.

-   The **Mobile Client** comes along and complains: "This T-shirt is too big! I only need the `username` and `avatar` fields, but you gave me a full User Profile with 50 fields, wasting my bandwidth and battery. Also, to display the home page, I have to call `/user`, `/notifications`, and `/timeline` three separate APIs, which is a disaster on a 3G network."
-   The **Web Client** then says: "For me, this T-shirt is a bit small. I need a richer, more complex data structure to render my dashboard. This API is too simplistic, and I have to combine and process data on the frontend myself, which makes the frontend logic bloated."
-   The **3rd-party Dev** finally adds: "This API changes every day. Can you give me a stable, versioned interface? I just want to get a product list."

See? The end result of a "one-size-fits-all" backend is **"fits-no-one."** Every frontend is compromising, doing work it shouldn't be doing.

> **`The core philosophy of BFF is "Inversion of Responsibility."`**

It declares: **"Instead of letting the backend define a set of standards it deems universal, let the 'representative' of each frontend experience tailor-make a dedicated backend for itself."**

This "representative" is the BFF. So, we will see:

-   `iOS-BFF`
-   `Android-BFF`
-   `WebApp-BFF`

The sole mission of each BFF is to serve its corresponding frontend with the highest efficiency, the fewest requests, and the most ideal data format. It is a translator, an aggregator, a tailor. This conceptual shift is fundamental to understanding the subsequent strategy.

### How to Apply BFF - The Beachhead Strategy

Now, let's apply this concept to the grand campaign of "strangling the monolith." Why is BFF the perfect "beachhead"?

In military terms, selecting a beachhead has several key principles: easy to capture, easy to defend, and has strategic value, creating conditions for the subsequent landing of the main forces. BFF perfectly meets all these conditions.

-   **Easy to Capture (Minimal Political Resistance and Technical Complexity):**
    -   Politically: When we promote this project within the company, we can call it the "Mobile User Experience Optimization Project" instead of the "Monolith System Refactoring Plan." The former is more likely to gain support from product managers and business stakeholders. We are not challenging the "old guard" who maintain the monolith; we are empowering the frontend team. The resistance is minimal.
    -   Technically: Its boundary is extremely clear—it's that app. We don't need to perform surgery deep in the most complex business core of the monolith. We just need to figure out: "To render the app's home page, which APIs of the old system do I need to call?" This is a relatively simple external observation, not an internal modification.

-   **Easy to Defend (Clear Combat Boundary):**
    -   The BFF's responsibility is fixed, and its service object is unique. This means its requirements come from a single source, and there won't be requirement changes from all directions. This makes the newly established service very stable, easy to maintain and test, ensuring the stability of our first position.

-   **Huge Strategic Value (Immediate Tactical Victory):**
    -   Visible Value: As mentioned earlier, a BFF can immediately bring measurable performance improvements. The app loads faster and is more power-efficient. This "battle report" will be very impressive, quickly building the team's and leadership's confidence in the new architecture.
    -   Establishing a Bridgehead: The success of the BFF is not just its own success. It establishes a critical infrastructure and knowledge base for our subsequent strangling operations. Through the development of the BFF, the team learns how to deploy Lambda, how to configure API Gateway, and how to set up monitoring. We have quietly landed our future "main forces" on the shore.

Choosing BFF is choosing the hardest bone to chew that is also the easiest to conquer and yields the most significant credit in the entire campaign.

### Wrapping the First Vine: How BFF Initiates the Strangling Process

Alright, the beachhead has been chosen. How do we get the first vine to start wrapping?

**Phase One: Passive Proxy & Aggregation**

At the very beginning, the BFF doesn't even need to write any business logic itself. Its working mode is very "passive."

1.  **Traffic Interception:** Through API Gateway, we change the app's first API request, for example, `GET /api/mobile/homepage`, from pointing to the monolith to pointing to our Mobile-BFF. Note, only this one API. All other API requests from the app still pass through the API Gateway and continue to hit the old monolith. The risk is controlled to a minimum.
2.  **API Aggregation:** Now, the Mobile-BFF receives this request. What it does internally might be to concurrently call three APIs of the old monolith: `GET /api/v1/user/123`, `GET /api/v1/timeline/123`, `GET /api/v1/ads/for-user/123`.
3.  **Data Tailoring:** After getting the bloated JSON data returned by these three APIs, the BFF selects the fields that the app truly needs, combines them into a clean, lightweight new JSON structure, and then returns it to the app.

At this stage, the strangulation has quietly begun. Although the business logic is still in the monolith, the monolith has lost the right to define the "mobile API contract." The final form of the API is now decided by the BFF. A small piece of the old tree's bark has been covered by the vine.

**Phase Two: Active Logic Migration**

Once the beachhead is stable, the vine starts to become more aggressive. We are no longer satisfied with just doing data aggregation.

1.  **New Feature Development:** Suppose the product needs to add a "like" feature to the app. We implement the business logic for this `POST /api/mobile/posts/{id}/like` request directly in the BFF. When the BFF receives the request, it might directly operate on the database (under strict permission control) or call a newly created, very small "like microservice." The code for this new feature has never existed in the monolith.
2.  **Old Feature Migration:** We find that the "update user profile" feature in the monolith is very slow and buggy. Now, we can "copy" or "rewrite" the logic of this feature completely from the monolith into the BFF (or a new User microservice). Then, through API Gateway, we switch the traffic for `PUT /api/mobile/profile` from pointing to the monolith to pointing to the BFF.

**Phase Three: The Final Strangulation**

Repeat the process of Phase Two. With each completed feature migration, the vine wraps tighter. The code in the monolith that once served the mobile client slowly turns into "Zombie Code"—it still exists, but no traffic will ever reach it again.

When all traffic from the Mobile App no longer points to the monolith but is handled entirely by the Mobile-BFF, the strangulation for the "mobile" business dimension is complete. On the old tree, all the branches facing south have withered because the sunlight (traffic) is completely blocked by the new vines.

At this point, we can safely go into the monolith's codebase and completely delete that zombie code. Every deletion is a declaration of victory.

To summarize, the process of BFF-initiated strangulation is:

1.  **Seize Control:** Start by proxying traffic to seize control of the API definition.
2.  **Establish a New Order:** Implement new features and establish new, more elegant logic on your own turf (the BFF).
3.  **Cannibalize the Old World:** Gradually migrate old logic, causing the corresponding parts in the monolith to become obsolete.
4.  **Declare Victory:** Once a complete business boundary (e.g., "mobile experience") is fully covered, clear out the obsolete code in the monolith.

## Fourth Movement: The AWS Blueprint for Implementation - Conducting a Smooth Traffic Migration

The fourth movement is the cadenza of the entire symphony, where theory and reality converge. A perfect strategy, without excellent tools and a clear construction blueprint, is just talk. We must not only draw the diagram but also understand the responsibilities, boundaries, and interactions of every component on it.

### Revisiting the Blueprint: From "Chaos" to "Order"

First, let's clearly etch the comparison of the two states in our minds.

**Before Evolution (The Monolithic State):**

This is a state with a single entry point and ambiguous responsibilities. All requests, regardless of their origin or purpose, flow to the same destination.

```
Client (Mobile/Web) → ALB/ELB → Monolith (EC2/ECS Cluster)
```

**During Evolution (The Evolutionary State):**

This is a state with clear responsibilities and precisely directed traffic. Our conductor—API Gateway—takes center stage.

```
Mobile Client  ┐
Web Client     ├→ API Gateway → AWS Lambda (BFF) → Monolith's Internal API/DB
               └→ API Gateway → ALB/ELB → Monolith
```

Now, let's deconstruct the core components of this new state one by one, understanding their respective roles and how they work together.

### A. The Command Center: Amazon API Gateway

If the entire migration is a precision military operation, API Gateway is our combat command center. It's not a simple request forwarder, but an intelligent gate that integrates routing, security, monitoring, and governance.

**Core Responsibility: Smooth Traffic Orchestration**

This is its most core and magical function—Path-based Routing. In API Gateway, we will define a set of rules like this:

| Priority | HTTP Method | Resource Path             | Integration Target                 | Description                                                                                             |
| -------- | ----------- | ------------------------- | ---------------------------------- | ------------------------------------------------------------------------------------------------------- |
| 1        | ANY         | `/api/v2/mobile/{proxy+}` | Lambda Function: Mobile-BFF-Lambda | [Strangler Rule] All V2 mobile requests are handled by the new BFF. `{proxy+}` is a greedy variable that matches all sub-paths. |
| 2        | ANY         | `/{proxy+}`               | HTTP Proxy: ALB of the Monolith    | [Default Rule] All other requests that do not match the above rule continue to be sent to the old monolith. |

The power of this rule set lies in:

**Seamless Migration:** For the client (Mobile App), the domain it accesses never changes. It doesn't know that its requests are being quietly sent to different destinations in the cloud.

**Low-Risk Changes:** When we need to migrate a new API (`/api/v2/mobile/profile`) from the monolith to the BFF, we don't need to redeploy a single line of code. We just need to add or modify a routing rule in the API Gateway console. This is an operation that can be completed in minutes and can be quickly reverted.

**Canary Deployments:** API Gateway also supports more advanced traffic allocation. We can set it to send 10% of the traffic for `/api/v2/mobile/search` to the new BFF and 90% to the old monolith. This allows us to validate the stability of the new service in a real production environment with a small portion of traffic. This is the ultimate weapon for achieving a "smooth" migration.

**Additional Responsibilities: Offloading Common Capabilities**

Besides routing, API Gateway acts like a capable adjutant, handling a large number of common tasks for us:

-   **Authentication/Authorization:** It can directly validate incoming JWTs (JSON Web Tokens) or API Keys. Only legitimate requests can enter the backend, greatly simplifying the security logic of the BFF and the monolith.
-   **Request Validation/Transformation:** Ensures the correctness of the request format.
-   **Rate Limiting/Throttling:** Protects backend services from being overwhelmed by malicious traffic.

### B. The Elite Unit: AWS Lambda (or Fargate)

Lambda is our light cavalry or special forces for executing the "beachhead" tactic. It is the ideal platform for running the BFF service.

**Why Choose Lambda?**

**Focus on Business Logic:** We only need to upload our BFF code (e.g., a Node.js/Python/Go application). We don't need to worry about the underlying server's operating system, patches, or scaling. AWS handles everything.

**Event-driven & Scalable:** Lambda automatically scales based on the number of requests. When there are no requests, it doesn't run, and the cost is zero. When a traffic spike arrives (e.g., from an app push notification campaign), it can instantly scale up to hundreds or thousands of concurrent execution environments to handle it. This elasticity is hard to match with traditional EC2.

**Low Operational Overhead:** Especially in the early stages of migration, the functionality and traffic of the BFF are limited. Maintaining a 7x24 running EC2 instance for it is a waste. Lambda's pay-per-use model perfectly fits this scenario.

**How the BFF Lambda Works Internally:**

After receiving a request from API Gateway, it executes the core logic we learned in the third movement:

1.  Parse the request to get user identity and other information.
2.  Concurrently make 1 to N requests to the monolith's internal APIs (or directly query its database).
3.  Wait for all requests to return, then aggregate, tailor, and transform the data into the format most needed by the mobile client.
4.  Return a single, clean response to API Gateway, which then sends it back to the phone.

### C. The Secure Messenger: IAM Roles

IAM (Identity and Access Management) is the security cornerstone of the entire AWS universe. Here, it plays the role of precise authorization, ensuring our BFF Lambda does not "overstep its authority."

This is a classic application of the **Principle of Least Privilege**. We will create a dedicated Execution Role for the Mobile-BFF-Lambda. This role will have several Policy statements attached to it:

**Basic Execution Permissions:** Allows the Lambda to write logs to Amazon CloudWatch Logs.

**Network Access Permissions:** If the monolith is deployed within a VPC, this authorizes the Lambda function to attach to the specified subnets and security groups of that VPC, allowing it to "see" the monolith's servers on the network.

**Resource Access Permissions (A Trade-off in the Transition Phase):**

-   **Best Practice (API Call):** If the monolith has internal APIs, no special permissions are needed here, as long as it's network-reachable.
-   **Good Practice (DB Access via Secrets Manager):** Authorize the Lambda to read the read-only database credentials from AWS Secrets Manager, and then perform queries. This is the most common transitional solution.
-   **Acceptable Practice (Direct DB Credential):** Writing the database password in an environment variable, which is riskier.

IAM ensures that even if the BFF's code has a vulnerability, the damage it can cause is limited to a very small scope. It's like giving the BFF a pass to a destination, but the pass precisely states which rooms it can access and which devices it can operate.

### D. The Situation Dashboard: Amazon CloudWatch & AWS X-Ray

If API Gateway is the command center, then CloudWatch and X-Ray are our intelligence and reconnaissance systems. Evolution without data is blind gambling.

**How to Use Them to Conduct the Migration?**

**Create a Comparison Dashboard:** In the CloudWatch Dashboard, we create two side-by-side views:

**Left Side: New Path Performance**

-   Request count, P90/P99 latency, 4xx/5xx error rate for the API Gateway `/v2/mobile` path.
-   Invocation count, execution duration, error rate, and throttle count for the Mobile-BFF-Lambda.

**Right Side: Old Path Performance**

-   Request count and target response time for the ALB.
-   CPU/Memory utilization of the Monolith EC2/ECS cluster.

This dashboard allows us to objectively answer with data: "Did our migration bring positive benefits?"

**Set Critical Alarms:**

-   If the error rate of the Mobile-BFF-Lambda exceeds 1% within 5 minutes, immediately send an alert to the team's Slack/Email via SNS.
-   If the P99 latency of the API Gateway exceeds 500ms, immediately trigger an alarm.

These alarms are our sentinels, ensuring we can intervene before problems escalate.

**Deep Diagnostics with X-Ray:**

When the dashboard shows that the new path has slowed down, how do we locate the problem? Is it API Gateway's fault? Is the Lambda slow? Or did the Lambda's call to the monolith slow down?

By enabling X-Ray, it will draw a complete **Service Map** and **Trace Details** for us. We will clearly see the complete lifecycle of a request:

```
Client -> API Gateway (5ms) -> Lambda (150ms) -> [Call Monolith /getUser (120ms)] -> Lambda -> Client
```

With this diagram, we can immediately identify that the performance bottleneck is in the call to the monolith's `getUser` service. X-Ray is the critical bridge from "discovering a problem" to "locating the problem."

## Finale: The Flywheel of Continuous Evolution

A heavy flywheel requires immense effort to push for the first time. This is the process of building our first BFF and configuring the first routing rule—full of learning, trial and error, and political communication. But once it starts turning, inertia makes every subsequent push easier.

Every successful migration injects new energy into this flywheel. Eventually, it will start turning on its own, driving the entire organization's technical culture forward. This energy is composed of three mutually reinforcing parts:

### The Technical Flywheel: Capability Accumulation and Reuse

The first success leaves us with a valuable technical asset.

**Reusable Infrastructure as Code (IaC):** The Terraform modules we wrote for Lambda and API Gateway (Day 14) can be directly reused when migrating the Web-BFF next time, reducing deployment time from days to hours.

**Mature CI/CD Pipeline:** The automated testing, approval, and canary release process we established (Day 15) has become the team's Standard Operating Procedure (SOP). The delivery of new services is no longer an uncertain adventure but a predictable, safe, standard process.

**Standardized Observability:** The CloudWatch dashboards and X-Ray tracing practices we established have become the "factory standard" for all new services. The team has a unified language to discuss performance and problems.

This means that the marginal cost of each evolution is significantly decreasing. We no longer need to start from scratch but are constantly performing a "copy-modify-deploy" cycle on a solid platform.

### The Organizational Flywheel: The Compounding of Trust and the Diffusion of Knowledge

Technical success will eventually transform into organizational change.

**The Compounding of Trust:** The success of the first BFF proved the value of evolutionary architecture with data ("Mobile P99 latency reduced by 80%"). This battle report wins you the trust and political capital of the management. When you propose the next migration plan, the resistance will be much smaller, and the resources you get will be more abundant.

**The Diffusion of Knowledge:** The first successful team will become the "Whampoa Military Academy" within the organization. Their experiences, lessons, and even failed attempts will become valuable learning materials for other teams. The best practices they establish will gradually spread and become the common wealth of the entire technical department.

**Agile Empowerment:** As more and more functions are split out, the autonomy of the teams also increases. The frontend team can iterate quickly with their BFF team without having to wait for the monolith's long release schedule. This breaks the organizational bottleneck and truly unleashes the team's agility and creativity.

The turning of this flywheel is quietly changing the organization's DNA, evolving from a centralized, slow-to-react structure to a distributed, fast-responding network.

### The Value Flywheel: From Technical Metrics to Business Impact

Ultimately, all efforts must return to business value.

**Faster Time to Market:** The organization's agility directly translates into new features being delivered to users faster, giving us an edge in market competition.

**Better User Experience:** The improvement in system performance and the reduction in error rates directly increase user satisfaction and retention.

**Stronger Talent Attraction:** No excellent engineer wants to waste their career maintaining an old, incomprehensible monolithic system. An environment that continuously evolves and embraces modern technology is key to attracting and retaining top talent.

This flywheel ensures that technical investment can be continuously transformed into visible business returns, thus forming a virtuous cycle of **"Investment → Output → Reinvestment."**

### Summary

We can condense all of today's discussion on the "Concerto of Architectural Evolution," from the underlying philosophy to the AWS implementation blueprint, into the following three core levels of mindset shift:

1.  **Philosophical Level**: From "Heroic Revolution" to "Gardener-like Evolution": Our discussion began with a fundamental mindset shift: rejecting the "Big Bang Rewrite," a revolutionary path full of heroic temptation but fraught with immense risk. We choose to be pragmatic gardeners or conductors, acknowledging the organic nature and historical value of the existing system. Through continuous pruning, grafting, and guidance, we ensure the business vessel never stops sailing during its evolution.

2.  **Strategic Level**: From "All-out War" to "Beachhead Landing": We break down the grand migration goal into a precise, executable military strategy. We adopt the "Strangler Fig Pattern" as the overall guiding principle and choose "BFF (Backend for Frontend)" as the "beachhead" with the highest value, clearest boundary, and lowest risk. We launch the first battle that we are sure to win, thereby building technical confidence and organizational trust.

3.  **Tactical Level**: From "Manual Construction" to "Cloud-based Command and Control": We no longer rely on manual operations but utilize AWS as a modern combat command system. We use API Gateway for precise traffic orchestration, Lambda as the light cavalry for executing assault missions, IAM to ensure secure authorization of actions, and CloudWatch/X-Ray to provide comprehensive battlefield intelligence. This transforms a chaotic migration into an observable, controllable, and rollback-able elegant exercise.

> **Key Takeaways**:
>
> -   **Evolve, Don't Rewrite**: This is our supreme guiding principle. A big bang rewrite is a tempting trap that brings a value vacuum, unbounded risk, knowledge drain, and organizational exhaustion. True superiority is achieving an elegant transformation within constraints.
> -   **BFF as the Perfect First Strike**: Choosing BFF as the starting point for the strangler strategy is because it can exchange the minimal technical and political resistance for the fastest and most significant improvement in user experience. It is the best leverage point to start the entire evolution flywheel.
> -   **The Value-Driven Flywheel**: Evolution is not a project, but a continuous process. Every successful, small value delivery (e.g., optimizing an API) injects energy into the technical, organizational, and business value flywheels, eventually forming a self-driving, continuously improving virtuous cycle.
> -   **The Entry Point is the Control Point**: The essence of the Strangler Fig Pattern is to peacefully seize control of traffic through a proxy layer like API Gateway, without modifying the old system. Whoever controls the entry point controls the rhythm and direction of the evolution.
>
> ### **The ultimate achievement is not delivering a static "perfect system," but successfully nurturing an organization and culture with the "capability for continuous evolution." We are not just building a new city, but the life force of this city to renew itself and thrive endlessly.**
````