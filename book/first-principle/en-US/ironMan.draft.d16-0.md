# Day 16 | Dev / Staging / Prod Multi-Environment Governance and Architecture Strategy: AWS Multi-Environment Configuration Management and Deployment Strategy

In the previous two topics, <Infrastructure as Code: Codifying and Version Controlling Infrastructure with Terraform> and <Full CI/CD Automation Implementation - GitHub Actions × CodePipeline × CodeBuild>, we discussed the importance of **"making infrastructure predictable, repeatable, and versionable"** and the proactiveness of **"protecting existing business logic from being corrupted by code changes,"** respectively. Both are aimed at solving the issue of `not having a stable environment and a fixed business logic validation process for testing and deployment`, ensuring that our **`business logic validation`** can grow strong like a towering tree in stable soil (I'm not talking about a certain Taiwanese bank). After briefly talking about the importance of environments and the solutions to corresponding pain points, we will now discuss - **`What environments do I need?`**

Recently, I had the pleasure of visiting a restaurant with a friend, Impromptu by Paul Lee at the Regent Taipei, to taste the 2025 summer flavors. It was a unique design that combined an open kitchen with the seating area, allowing diners to see simply and transparently how the chef and his team present a beautiful and magnificent tapestry of the 2025 summer using ingredients, spices, and selected wines. I personally loved this menu design; you could feel a clear, crisp, and straightforward elegance.

But speaking of which, how is a Michelin-starred dish designed? A new concept is exciting news for all foodies, but what if it's made and served directly to the customer at the counter, and it fails? Too much salt, wrong cooking time—at best, you lose your reputation; at worst, the customer gets a stomachache, and the next day, your restaurant's name is ruined in the headlines.

So let's take a step back. How about a trial run in a corner of the restaurant's "main kitchen" backstage before an official launch?

This is a good method. We avoid customers tasting experimental dishes immediately. However, although customers can't see it, we might disrupt the existing normal operation of the kitchen. We might occupy a cutting board being used for prep, find that a sauce is used up by another chef when we need it, or even the smoke from our experiment might affect the carefully cooked dishes next to us.

That being said, if funds and space allow, trying to realize our new concept in a completely independent "Test Kitchen" seems to be the optimal solution. Here, we have our own full set of pots, pans, ingredients, and ovens, allowing us to experiment, fail, and retry freely without worrying about affecting the restaurant's operations outside.

This is the **Development Environment (Dev)**. It is a completely isolated sandbox that allows every developer to freely write, modify, and test their own code without worrying about "breaking" anything or affecting anyone. It prevents us from affecting colleagues who are testing other features in the **"main kitchen" backstage (shared environment)**, and vice versa. Most importantly, it avoids writing code directly in the production environment (Production / Prod) to the greatest extent possible. Any small mistake, such as an incorrect database command, could lead to service interruption for all users and data loss. **`This is an absolutely, absolutely forbidden behavior.`**

Before it's officially served, we've created the new dish in our own **R&D kitchen (development environment)**, and it tastes heavenly. What's next? Put it directly on the menu and sell it to the most important food critics?

If we don't have a master chef hiding in a chef's hat above us, and what we're serving is ratatouille (Ratatouille is a great movie, watch it if you have time), this is a very risky move.

Before the official launch, we need a place for a **"final rehearsal."** We call this place the "Staging Kitchen." Its configuration, stoves, workflow, and even the brand of plates must be identical to the official "main kitchen." Here, we need to simulate the real serving process: the waiter has to run through the entire ordering and serving process, the chef team has to simulate whether they can stably produce the dish according to the intended process under the pressure of peak hours, and finally, we need to test whether this new dish will conflict with the process when served with other classic dishes.

This is the importance of the **Staging / Pre-production Environment**. We must have an environment for comprehensive testing in various scenarios to ensure that our business logic meets the requirements and quality standards.

Now, we can abstract the software development lifecycle into a clear "Progression":

```python
Dev => Staging => Prod
```

- **Development Environment (Dev): The Test Kitchen**
  - **Purpose**: Feature development, unit testing, rapid iteration.
  - **Characteristics**: Chaos is the norm. Frequent modifications, deployments, and even complete tear-downs are allowed. Developers have higher privileges. Data is usually fake or anonymized.
  - **Core Focus**: Development efficiency.

- **Staging Environment: The Staging Kitchen**
  - **Purpose**: Integration testing, User Acceptance Testing (UAT), performance stress testing. Simulates all real-world scenarios.
  - **Characteristics**: The environment must be extremely close to the production environment. Configuration, operating system, network rules, database version, etc., should be consistent. The deployment process should also be identical to the official launch process. Data can be a replica of the production environment (after desensitization).
  - **Core Focus**: Stability and Consistency. This is the last line of defense for quality before going live.

- **Production Environment (Prod): The Michelin-Starred Restaurant**
  - **Purpose**: Serve real users and create value.
  - **Characteristics**: Extremely stable, secure, and high-performance. Any changes must go through strict approval and standardized processes. Access control is the strictest, and usually, only automated tools or a very small number of authorized personnel can perform deployments.
  - **Core Focus**: Reliability and Security.

This one-way flow of **`Dev => Staging => Prod`** is the `core workflow` of software quality assurance. Code, like water, can only flow from a lower environment to a higher one, never in reverse. Each flow is a check at a **`"Quality Gate,"`** whether it's the **`automated checks`** in the **CI (Continuous Integration)** process to **`protect existing business logic from being corrupted by code changes`**, or the three-way verification of **Peer Review Gate**, **Business Review Gate**, and **Quality Review Gate** mentioned in <Version Control Strategy(PR Review strategy)>. Once **`any condition for the implementation of business logic is not met`**, it must be immediately returned for emergency repair. We must have a deep and clear understanding - **`a system is the implementation of abstract business logic`**.

In modern software development, multi-environment management is key to ensuring application stability and reliability. Next, we will delve into how to build and manage Dev/Staging/Prod multi-environment architectures on AWS, focusing on core technologies such as Docker containerization, ECS cluster management, and EKS/K8s orchestration.

> 1. Multi-Environment Architecture Design Principles - Environment Isolation Strategy and IaC Practice
> 2. Docker Containerization Strategy - Multi-stage Builds and Environment-specific Configurations
> 3. Amazon ECS Cluster Management - Service Definition and Auto-Scaling Configuration
> 4. Amazon EKS and Kubernetes Management - Cluster Configuration, Application Deployment, and Ingress Setup
> 5. CI/CD Pipeline Integration - GitHub Actions Workflows and Blue/Green Deployment
> 6. Configuration Management and Secrets Management - AWS Secrets Manager and K8s ConfigMap/Secret
> 7. Monitoring and Logging Management - CloudWatch and Prometheus/Grafana Integration
> 8. Security Best Practices - Network Security and RBAC Configuration
> 9. Cost Optimization Strategy - Automated Resource Management and Spot Instance Integration
> 10. Disaster Recovery and Backup - Cross-region Backup and Database Failover

## 1. Multi-Environment Architecture Design Principles

### 1.1 Environment Isolation Strategy

We discussed above that we need to divide our restaurant (our system) into a "R&D Kitchen (Dev)," a "Staging Kitchen," and a "Flagship Store (Prod)." Let's talk about how to actually "demarcate the boundaries" and ensure that the "building codes" for these three plots of land are consistent.

```python
Production Environment
├── High availability configuration
├── Auto-scaling
├── Complete monitoring and alerting
└── Strict deployment process

Staging Environment
├── Production environment mirror
├── Full functional testing
├── Performance testing environment
└── Integration test validation

Development Environment
├── Rapid deployment
├── Developer-friendly
├── Resource cost optimization
└── Flexible configuration adjustments
```

The simplest way is, of course, to lock each kitchen and give exclusive keys to people with corresponding functional needs. Lock the doors first to prevent people from wandering around. It's bad enough if our own people accidentally adjust parameters or make configuration errors that cause service paralysis. At the very least, we must ensure that thieves and robbers without keys can't even get in. So our first core principle is simple: **`Minimize Blast Radius`**. This means that when a disaster occurs in one environment (e.g., being hacked, or a configuration error causing service paralysis), it must absolutely not affect other environments.

On AWS, the gold standard for achieving this goal is called the **`Multi-Account Strategy`**.

**1. AWS Organizations: The Cloud Corporate Headquarters**

**`AWS Organizations`** is our "corporate headquarters," through which we can manage different AWS accounts and formulate basic policies. Just as a group has different business units for finance, sales, legal, HR, development, and maintenance, we can create a `DevOU` and a `ProdOU` as concepts for business units. `Account-Dev` (department) is placed in `DevOU` (business unit); `Account-Staging` and `Account-Prod` are placed in `ProdOU`. Then we can set **`Service Control Policies (SCPs)`** to restrict what the accounts under them **can** or **cannot** do. This is like a "supreme executive order" issued by the group. When there is a conflict between business unit policies and the group headquarters, the `SCPs` regulations will be followed preferentially and compulsorily. And `AWS Organizations` has another benefit: all bills and costs of the business units are aggregated at the headquarters, which is convenient for monitoring and management.

`SCPs` prevent human error or malicious behavior from the root, which is much more effective than remedying it afterward. It defines the "behavioral constitution" and "physical laws" of each environment. For example:

- For all accounts under DevOU:
  - Prohibit creating databases connected to the external network (RDS public access = false).
  - Prohibit disabling CloudTrail (audit logs) to ensure all operations are recorded.
  - Restrict resource creation to the Asia-Pacific region (e.g., Tokyo, Singapore) to prevent someone from accidentally creating resources in expensive European or American regions.
- For all accounts under ProdOU:
  - Except for specific administrator roles, prohibit anyone from deleting databases or S3 buckets.

**2. IAM Permissions: "Passes" for Different Identities**

After account isolation, how do people get in to work? The answer is IAM Roles, not creating a set of usernames and passwords (IAM User) for everyone in each account.

- **Concept**: Create IAM Users in our Management Account, and then define different IAM Roles (e.g., DeveloperRole, QARole, OperatorRole). Then let these Users "Assume Role" to perform tasks in other accounts.
- **Analogy**: We are employees of the company (IAM User), and the company gives us different work badges (IAM Role).
  - With a "Developer Badge," we can swipe into all rooms of Account-Dev.
  - With a "QA Badge," we can enter Account-Staging, but only observe and record, not modify equipment.
  - Only with a "Flagship Store Manager Badge" can one enter Account-Prod for operations after approval.

This approach centralizes permission management, making it secure and clear.

### 1.2 Infrastructure as Code (IaC)

We've ensured there are strong walls between environments, but how do we guarantee that the interior decoration, pipelines, and stoves of the "Staging Kitchen" and the "Flagship Store" are identical? This is where the power of `Infrastructure as Code (IaC)` comes in. According to the `blueprint (IaC)`, we can quickly build an identical architecture and environment. We mentioned the importance of blueprints in <Infrastructure as Code: Codifying and Version Controlling Infrastructure with Terraform>. It makes the infrastructure `predictable`, `repeatable`, and `evolvable`. Let's continue with Terraform as an example.

```
terraform/
├── modules/
│   ├── vpc/
│   │   ├── main.tf
│   │   └── variables.tf
│   └── ec2_instance/
│       ├── main.tf
│       └── variables.tf
│
└── environments/
    ├── dev/
    │   ├── main.tf
    │   └── terraform.tfvars
    ├── staging/
    │   ├── main.tf
    │   └── terraform.tfvars
    └── prod/
        ├── main.tf
        └── terraform.tfvars
```

- `modules/` stores our standardized building modules.
- `environments/` stores the "construction instructions" for each environment.

In `environments/dev/main.tf`, we present it with fake code:

```yaml
# Reference the VPC module
module "my_vpc" {
  source = "../../modules/vpc"
  cidr_block = var.vpc_cidr
}

# Reference the EC2 module
module "web_server" {
  source      = "../../modules/ec2_instance"
  instance_type = var.instance_type
  vpc_id      = module.my_vpc.id
}
```

Notice? This `main.tf` file will look almost the same in dev, staging, and prod. The real difference lies in the `terraform.tfvars` parameter file next to it:

- `environments/dev/terraform.tfvars`
```
vpc_cidr      = "10.10.0.0/16"
instance_type = "t3.micro"  // Use small machines in the dev environment to save money
```

- `environments/prod/terraform.tfvars`
```
vpc_cidr      = "10.20.0.0/16"
instance_type = "m5.large"  // Use large machines in the production environment to handle traffic
```

When we want to deploy the Dev environment, we just go into the `environments/dev` folder and run `terraform apply`. Terraform will automatically read the `dev` parameters, apply them to the shared modules, and build an environment that meets the development specifications. The same applies to deploying `Prod`.

This perfectly achieves our IaC goals:
1. **Consistency**: The "architecture" of all environments comes from the same blueprint (modules).
2. **Traceability**: All infrastructure changes must be made by modifying the code. Every change is recorded in the version control system (like Git), which can be reviewed and traced.
3. **Reproducibility**: Even if the entire environment is destroyed, we can quickly rebuild an identical one with the same code.

## 2. Docker Containerization Strategy

We have already drawn the architectural blueprints for the three kitchens using IaC. Now, we need to standardize our "cooking tools." This is the problem that the Docker containerization strategy aims to solve.

We have all heard, and perhaps even experienced, this classic scenario:
> Developer A: "This feature works fine on my machine! Why did it break after deployment?"

This "It works on my machine" problem is the chronic disease that Docker aims to cure. The root of the problem is that there are subtle but fatal differences in the "environments" of the developer's computer, the test server, and the production server:
- Different operating system versions (Ubuntu 20.04 vs 22.04)
- Different versions of dependent libraries (Python 3.8 vs 3.9, OpenSSL v1 vs v3)
- Different environment variable settings
- Different hardware parameters
- Different system paths or file permissions

Even if everything is the same, one might pass the test while the other fails. With the same OS like Windows (no bias, but this is where `underlying differences` are most likely to occur), we can only put out offerings, holy water, or praise the Omnissiah to beg the machine spirit for a smooth execution.

The core idea of **`Docker`** is to package and take away the application along with its "entire execution environment."

Instead of giving a chef a recipe for "Beef Bourguignon" and letting him use the pots, stoves, and seasonings in his own kitchen (which would likely result in a different taste due to the differences in each kitchen), we might as well give him the semi-finished product, a special sauce, and a "standardized heating box" with precise temperature control. All he has to do is press a button.

This "standardized heating box" is the **Docker Container**. This box contains:
1. Our application code (The dish itself)
2. The runtime, e.g., Node.js v18 or Java 17 (The specific cooking sauce)
3. All system-level libraries and tools (The special salt and pepper)

No matter if this "box" is put in the microwave of the `Dev` environment or the top-tier oven of the `Prod` environment (as long as they are both qualified Docker Hosts), the dish that comes out after heating is guaranteed to taste exactly the same. This is the `Portability` and `Consistency` that containerization brings.

### 2.1 Multi-stage Build Optimization

We divide the Dockerfile build process into two stages, like an assembly line:
- **Stage 1 (Builder Stage):** A spacious, well-equipped "processing plant."
- **Stage 2 (Final Stage):** A clean, lightweight "delivery box."

```dockerfile
# Dockerfile.multi-stage
# Build stage
FROM node:18-alpine AS builder
WORKDIR /app
# Copy only the package.json needed for the build
COPY package*.json ./
# Install only the necessary dependencies for production
RUN npm ci --only=production

# Development stage
FROM builder AS development
RUN npm ci
COPY . .
EXPOSE 3000
CMD ["npm", "run", "dev"]

# Production stage
FROM node:18-alpine AS production
WORKDIR /app
COPY --from=builder /app/node_modules ./node_modules
COPY . .
EXPOSE 3000
USER node
CMD ["npm", "start"]
```

The benefits of doing this are obvious:
- **Extremely small final image:** It only contains `node_modules` (production), `dist`, and `package.json`. No source code, no development tools.
- **Higher security:** The attack surface is minimized. News of packages being attacked or embedded with Trojans is not common, but it has been heard of. Compilation tools (like gcc, npm) and development libraries can become vulnerabilities for hackers. So the production environment should only contain the `"minimum necessary"` execution components.
- **Clear build process:** It separates the concerns of "how to build" and "how to run."

### 2.2 Environment-specific Configuration Management

How does one box adapt to different kitchens?

We already have a standardized "heating box," but now there's a new problem:
> In the Dev environment, this dish needs to connect to `dev-database`; in the Prod environment, it needs to connect to `prod-database`.
> 
> But we only have one box (Docker Image), so what should we do?

```yaml
# docker-compose.dev.yml
version: '3.8'
services:
  app:
    build:
      context: .
      target: development
    environment:
      - NODE_ENV=development
      - LOG_LEVEL=debug
    volumes:
      - .:/app
      - /app/node_modules
    ports:
      - "3000:3000"

# docker-compose.prod.yml
version: '3.8'
services:
  app:
    build:
      context: .
      target: production
    environment:
      - NODE_ENV=production
      - LOG_LEVEL=error
    restart: unless-stopped
    ports:
      - "80:3000"
```

First, we must remember to abide by an iron law, an absolute rule:
> #### **Build once, run anywhere**

We cannot just open a sealed lunch box because of the needs of different environments. This would completely violate **`Image Immutability`**. We must absolutely not build different `Images` for different environments (e.g., `my-app:dev`, `my-app:prod`). This would destroy all the **`consistency guarantees`** we have established.

```
The Image that passes testing in Dev must be the exact same, unmodified Image that is deployed to Staging and Prod.
```

But what if we really have a need for `external injection`? What should we do?

So, our "standard heating box" has a few slots reserved, like a "database connection slot" and an "API key slot." The box itself doesn't carry these things. Instead, when it's placed in a designated spot in a specific kitchen, the kitchen's "automated arm" (container orchestration system) will plug in the corresponding plugs.

Or, like some Japanese instant noodles, the hot water inlet and the water outlet are different holes, each for its own purpose. We can pour in and drain hot water, hot tea (which is also good, I've tried), or even hot milk tea (???, seen it but respect it), but the main body (the noodles and seasoning) inside remains unchanged.

So looking back at the YAML example, we find some clues:

```yaml
# docker-compose.dev.yml
    environment:
      - NODE_ENV=development
      - LOG_LEVEL=debug

# docker-compose.prod.yml
    environment:
      - NODE_ENV=production
      - LOG_LEVEL=error
```

We can set different interfaces by declaring **`environment variables`**. This is the most standard method and the one that best fits the philosophy of the 12-Factor App - **application code should not hardcode any configuration, but read it from environment variables**.

When the container starts (managed by ECS/Kubernetes), we would execute it as follows:
- In the Dev environment: `docker run -e DATABASE_HOST=dev.db.internal ... my-app`
- In the Prod environment: `docker run -e DATABASE_HOST=prod.db.internal ... my-app`

For more complex configurations (like an entire `settings.json`), the container orchestration system can "mount" the configuration file for the corresponding environment to a specific path inside the container when it starts (e.g., `/etc/config/settings.json`). Our application just needs to read the file from this fixed path.

Finally, there is a more advanced and safer way: **`Fetching from Config Service on startup`**. When the container starts, the application uses its assigned identity (e.g., an AWS ECS Task Role) to call an external service (like AWS Secrets Manager or Parameter Store) to get the database password, API keys, etc., that it needs. This way, even environment variables do not appear in plain text, which is the highest level of security.

## 3. Amazon ECS Cluster Management

We have already built standardized kitchens (IaC) and designed standardized "semi-finished meal kits" (Docker Images). Now, we need an intelligent and efficient "head chef system" to manage the entire backend operation. This system needs to know:
- What is the standard cooking procedure for a dish (our application)?
- How many stoves should be running simultaneously during peak hours (high traffic) to make this dish?
- If a stove breaks down (server failure), how to automatically replace it with a new one to continue cooking?

This "head chef system" is **`Amazon ECS (Elastic Container Service)`**.

### 3.1 ECS Service Definition

To understand ECS, we first need to get to know its 4 core components: `ECS Cluster - "the kitchen itself"`, `Task Definition - "the Standard Operating Procedure (SOP) card"`, `Task - "the wok on the high-fire stove"`, and `Service - "the tireless head chef"`.

1. **ECS Cluster - "the kitchen itself"**
- **Core Concept:** A logical grouping used to contain all our containerized applications. We can think of it as `Dev Kitchen Worktop`, `Staging Kitchen Worktop`, `Prod Kitchen Worktop`. It's like telling people how many square feet your kitchen work area is, without getting into the actual layout inside. It doesn't cost anything itself, it's just a management boundary.
- **Key Point:** A cluster needs "compute power" to operate, just like a kitchen needs stoves. There are two modes for the source of compute power:
  - **EC2 Mode:** We buy a batch of "stoves" (EC2 cloud servers) ourselves and register them in our kitchen. We have full control over the brand and model of the stoves and can adjust them freely, but we are also responsible for their maintenance and management.
  - **Fargate Mode (Serverless - the mainstream trend in the industry):** We don't buy stoves. When we need to cook, we just tell AWS: "I need a stove with 500W of power and an area of 1 square meter." AWS will automatically conjure up a stove that meets our requirements from their vast resource pool, and it's gone after we're done.
  - For the vast majority of new applications, Fargate is the preferred choice. It frees us from tedious server management and allows us to focus more on our application itself.

2. **Task Definition - "the Standard Operating Procedure (SOP) card"**
- **Concept:** This is the core blueprint of ECS. It is a JSON-formatted instruction manual that defines in detail "how to cook a dish."
- **Key Point:** It's like a foolproof SOP card for the chef, which precisely states:
  - `image:` Which version of the "semi-finished meal kit" to use (Docker Image URL).
  - `cpu & memory:` How much "firepower" and "space" to allocate to this task.
  - `environment:` What "seasonings" to inject (environment variables, e.g., DATABASE_URL).
  - `secrets:` Which "secret recipes" to retrieve from the "safe" (AWS Secrets Manager) (API Key, database password).
  - `logConfiguration:` Where to send the cooking process records (logs) (e.g., to CloudWatch Logs).
  - `taskRoleArn:` What kind of "permission card" (IAM Role) to give this chef, allowing it to interact with other AWS services (e.g., read files from S3).

3. **Task - "the wok on the high-fire stove"**
- **Concept:** A "Running Instance" of a task definition. When ECS actually starts a container based on our SOP card, that running container is a Task. We can decide how many woks are stir-frying at the same time by setting the number of Tasks.

4. **Service - "the tireless head chef"**
- **Concept:** The Service is the brain and butler of ECS. Its duty is to maintain the state we desire.
- **Key Point:** Ensure that everything in the kitchen runs according to the execution order we want.
  - `desiredCount: { N }`: "I want there to always be `{ N }` portions of 'Beef Bourguignon' cooking on the fire, no more, no less."
  - Through `Health Check`, the head chef constantly patrols all the stoves to see if they are normal. If he finds that one of them has failed due to a faulty stove (`HC fail`), he will immediately discard it and start a new stove, ensuring that the number of active stoves is always 3.
  - **Load Balancing:** When a stove is lit (a new Task starts), the head chef will automatically notify the waiters at the front (Application Load Balancer), telling them that there is now an additional serving opening, and they can send customer orders (traffic) over.

**Implementation Example**

```json
{
  "family": "app-service",
  "networkMode": "awsvpc",
  "requiresCompatibilities": ["FARGATE"],
  "cpu": "256",
  "memory": "512",
  "executionRoleArn": "arn:aws:iam::account:role/ecsTaskExecutionRole",
  "taskRoleArn": "arn:aws:iam::account:role/ecsTaskRole",
  "containerDefinitions": [
    {
      "name": "app",
      "image": "your-registry/app:${ENVIRONMENT}",
      "portMappings": [
        {
          "containerPort": 3000,
          "protocol": "tcp"
        }
      ],
      "environment": [
        {
          "name": "NODE_ENV",
          "value": "${ENVIRONMENT}"
        }
      ],
      "secrets": [
        {
          "name": "DATABASE_URL",
          "valueFrom": "arn:aws:secretsmanager:region:account:secret:${ENVIRONMENT}/database-url"
        }
      ],
      "logConfiguration": {
        "logDriver": "awslogs",
        "options": {
          "awslogs-group": "/ecs/${ENVIRONMENT}/app",
          "awslogs-region": "us-west-2",
          "awslogs-stream-prefix": "ecs"
        }
      }
    }
  ]
}
```

#### ECS Practice in Multi-Environment

After briefly explaining the meaning of ECS, how does our `Dev Kitchen Worktop`, `Staging Kitchen Worktop`, `Prod Kitchen Worktop` strategy manifest at the ECS level?

- **Cluster Isolation:** Following the multi-account strategy, we will create `dev-cluster` in `Account-Dev` and `prod-cluster` in `Account-Prod`. This is the most thorough isolation.
- **SOP Card (Task Definition) Management:** We still use IaC (Terraform) to manage task definitions. We will have a shared `task-definition.tf` template and pass different parameters for different environments.
  - `dev.tfvars`: `cpu = 256 (1/4 vCPU)`, `memory = 512 (MB)`, `API_KEY_SECRET_ARN = "arn:aws:secretsmanager:ap-northeast-1:DEV_ACCOUNT_ID..."
  - `prod.tfvars`: `cpu = 1024 (1 vCPU)`, `memory = 2048 (MB)`, `API_KEY_SECRET_ARN = "arn:aws:secretsmanager:ap-northeast-1:PROD_ACCOUNT_ID..."
  - Note that our `image` parameter will point to the **same Docker Image**, but the injected `CPU`, `memory`, and `Secrets` still come from different environment configurations, maintaining our **`Consistency`** and **`Immutability`**.
- **Head Chef Command (Service) Management:** Also defined using IaC.
  - The `dev` environment's Service might have a `desiredCount` of just `1`, and auto-scaling might not be configured to save costs.
  - The `prod` environment's Service might start with a `desiredCount` of `3` and will be configured with the auto-scaling strategy we'll discuss next.

### 3.2 ECS Auto Scaling Configuration

Our restaurant can't just rely on a fixed number of 3 chefs. Sometimes it's predictable, like on a Tuesday afternoon one might be enough, but on a Friday night, we might need 10 to handle the endless stream of orders. Sometimes, the crowd might suddenly surge, requiring 20 chefs to handle.

Manually adding or removing chefs at this point is too slow and impractical. So we need to set up a **`"Customer Flow - Stove Monitoring Linkage System" (ECS Service Auto Scaling)`** that allows us to automatically increase or decrease staff based on "customer flow" to optimize our costs.

- **Core Idea:** Monitor a metric and automatically adjust the Service's `desiredCount` based on the changes in this metric.
- **Most Common Metrics:**
  - **CPU Utilization:** "If the chefs' average business level exceeds 70%, add more staff." This is the most common metric.
  - **Memory Utilization:** "If the average space occupancy in the kitchen exceeds 75%, expand the kitchen (add more Tasks)."
  - **ALB Request Count Per Target:** This is the metric that best reflects real user pressure. "If the average number of orders each serving opening has to handle per minute exceeds 500, immediately add more serving openings."
- **Set Threshold:** I want this `ALBRequestCountPerTarget` metric to stay around 500.
- **Set Boundaries:**
  - `min_capacity = 2`: No matter what, there must be at least 2 chefs on standby in the kitchen.
  - `max_capacity = 20`: The maximum is 20. Any more, and my costs might explode, or the backend database won't be able to handle it.

The operational flow is roughly as follows:
- **Scale Out:** When traffic surges at 7 PM on a Friday night, the average request count soars to 800. Auto Scaling detects this and immediately increases the Service's `desiredCount` (e.g., from 2->3, 3->5...). ECS then starts new Tasks. The new Tasks are automatically registered with the ALB, sharing the traffic until the average request count drops back to around 500.
- **Scale In:** At 10 PM, as customers gradually leave, the average request count drops to 100. Auto Scaling detects this and gradually reduces the `desiredCount`. ECS will then gracefully terminate the excess Tasks, releasing the resources and saving us costs.
- **Schedule (Period):** If we already know that there is a **fixed traffic surge** every Friday at 7 PM and the customers are mostly gone by **11 PM**, we can set a fixed number of Tasks `{ N }` for the `19:00-23:00` period to handle the traffic. This can further reduce costs - monitoring also costs money.

```yaml
# ecs-auto-scaling.yml
Resources:
  ServiceScalingTarget:
    Type: AWS::ApplicationAutoScaling::ScalableTarget
    Properties:
      MaxCapacity: !Ref MaxCapacity
      MinCapacity: !Ref MinCapacity
      ResourceId: !Sub "service/${ClusterName}/${ServiceName}"
      RoleARN: !Sub "arn:aws:iam::${AWS::AccountId}:role/aws-service-role/ecs.application-autoscaling.amazonaws.com/AWSServiceRoleForApplicationAutoScaling_ECSService"
      ScalableDimension: ecs:service:DesiredCount
      ServiceNamespace: ecs

  ServiceScalingPolicy:
    Type: AWS::ApplicationAutoScaling::ScalingPolicy
    Properties:
      PolicyName: !Sub "${ServiceName}-scaling-policy"
      PolicyType: TargetTrackingScaling
      ScalingTargetId: !Ref ServiceScalingTarget
      TargetTrackingScalingPolicyConfiguration:
        PredefinedMetricSpecification:
          PredefinedMetricType: ECSServiceAverageCPUUtilization
        TargetValue: 70.0
```

## 4. Amazon EKS and Kubernetes Management

Human resources are sometimes limited, and taste buds can get tired. Sometimes, opening more **Michelin restaurants** to attract more people is not necessarily a better choice. Perhaps it's a matter of taste, or perhaps it's a matter of budget. When our service expands its business logic, continuously adding new dishes to the same restaurant will gradually increase management costs and complexity. At the same time, it's also possible that after opening a new store, everyone points to a specific dish (like a Michelin chef's cheeseburger and a restaurant that automatically explodes after eating), and other services are not used at all - this is not the fault of the vendor, it is undoubtedly a waste of cost.

If our next goal is to open a restaurant group with dozens of different brands (microservices) for different customer groups, and possibly even open branches in Google or Microsoft malls (multi-cloud), then we need an industry-standard, standardized "group operating system." This system is Kubernetes (often abbreviated as K8s).

Amazon EKS (Elastic Kubernetes Service) is the service provided by AWS that helps us manage the most core and brain-burning "group headquarters management office" (control plane) of this complex operating system.

### 4.1 EKS Cluster Configuration — Building Our Food Court

If an ECS Cluster is the back kitchen of an independent restaurant, then an EKS Cluster is the venue and basic infrastructure of the entire food court. The most important parts are the **"Food Court Management Office" (Control Plane)** and the **"Food Stall Spaces" (Data Plane / Worker Nodes)**.

- **Control Plane - "Food Court Management Office"**
  - **Concept:** This is the brain of Kubernetes. It is responsible for receiving instructions, scheduling the opening locations of all stalls (applications), monitoring the operating status of all stalls, storing the layout of the entire square, and so on. This part is extremely complex and crucial, and must be highly available 24/7.
  - **Value of EKS:** We don't have to build this management office ourselves, nor do we have to hire security guards or IT staff to maintain it. AWS EKS provides us with a highly available, secure, and automatically upgraded management office. This is the main reason we pay for EKS. We just need to focus on running our food stalls.

- **Data Plane / Worker Nodes - "Food Stall Spaces"**
  - **Concept:** This is where the actual "cooking" happens, i.e., the servers where our application containers actually run.
  - In EKS, there are mainly two forms:
    - **EC2 Managed Node Groups:** Similar to the EC2 mode of ECS. We rent a row of "standard stalls" (EC2 instances) from the square management. We can choose the specifications of the stalls (e.g., `m5.large`), but the power, network, fire protection, etc., of the stalls are managed by AWS, such as automated node upgrades and replacements. This is the most common mode.
    - **Fargate Profiles:** Same as the ECS Fargate mode. We don't even have to rent a stall. We just tell the management office: "I want to open a temporary takoyaki stall and need a space with 1vCPU and 2GB of memory." EKS will automatically create a space that meets our requirements in a corner of the square and take it away when we're done. It's very suitable for stateless applications or scenarios that need to cope with sudden traffic surges.

**Multi-environment Strategy:** The principle remains the same. We will create a `dev-eks-cluster` in `Account-Dev` and a `prod-eks-cluster` in `Account-Prod`. IaC (Terraform) is still our tool for standardizing the definition of these cluster configurations.

```yaml
# eks-cluster.yaml
apiVersion: eksctl.io/v1alpha5
kind: ClusterConfig

metadata:
  name: app-cluster-${ENVIRONMENT}
  region: us-west-2
  version: "1.28"

nodeGroups:
  - name: worker-nodes
    instanceType: ${INSTANCE_TYPE}
    minSize: ${MIN_SIZE}
    maxSize: ${MAX_SIZE}
    desiredCapacity: ${DESIRED_SIZE}
    privateNetworking: true
    labels:
      environment: ${ENVIRONMENT}
    tags:
      Environment: ${ENVIRONMENT}
      Project: "multi-env-demo"

addons:
  - name: vpc-cni
  - name: coredns
  - name: kube-proxy
  - name: aws-load-balancer-controller
```

### 4.2 Kubernetes Application Deployment

In the world of Kubernetes, there is a standard vocabulary and set of documents (YAML files) for describing "how to run a stall." This is more detailed and powerful than ECS's Task Definition.

1. **Pod - "An Independent Cooking Workstation"**
   - **Concept:** The smallest deployable unit in Kubernetes. It can contain one or more tightly coupled containers.
   - **Analogy:** A Pod is not just the pot (Container); it also includes a small workbench next to the pot, an independent sink and power outlet (independent network IP address), and a dedicated small refrigerator (storage Volume). The containers within a Pod can share this small environment.
   - **Key Feature:** Pods are ephemeral; they can be destroyed and recreated at any time. We must never directly depend on the existence of a specific Pod.

2. **Deployment - "Franchise Handbook"**
   - **Concept:** A Deployment is a higher-level controller whose core responsibility is to ensure that a specified number of identical Pod replicas are running.
   - **Analogy:** We are the brand owner of "A+ Fried Chicken." Our Deployment YAML is our franchise handbook, which states:
     - `replicas: 3`: I require that there must always be 3 "A+ Fried Chicken" cooking workstations (Pods) in operation in the food court.
     - `template`: The handbook details the specifications of a standard workstation (which Docker Image to use, how much CPU/Memory is needed, etc.).
     - **Self-healing:** If the Deployment finds that only 2 workstations are running, it will immediately create a third one.
     - **Rolling Update:** When we want to launch a new flavor (update the Docker Image), the Deployment will be very smart: it will first open a new workstation -> wait for the new station to be ready -> then close an old one. This gradual replacement ensures that our stall service is not interrupted.

3. **Service - "Stall's Calling Number and Internal Line"**
   - **Concept:** Pods are constantly being created and destroyed, and their IP addresses are always changing. A Service provides a stable, unchanging internal network endpoint to represent a group of functionally identical Pods.
   - **Analogy:** Our 3 fried chicken workstations (Pods) may be in different locations in the square and may change positions at any time. Customers can't possibly chase after them. So, we register a fixed stall number "B-52" (Service) at the food court's service desk.
   - **How it works:** When other stalls in the food court (e.g., the "Good Drinks" Pod) want to order fried chicken, they don't need to know the actual location of any fried chicken Pod. They just need to call the internal line "B-52," and the Kubernetes network system will automatically forward the request to a currently healthy fried chicken Pod. This ensures the stability of communication between services and is the cornerstone of a microservices architecture.

```yaml
# k8s/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app-deployment
  namespace: ${ENVIRONMENT}
  labels:
    app: demo-app
    environment: ${ENVIRONMENT}
spec:
  replicas: ${REPLICA_COUNT}
  selector:
    matchLabels:
      app: demo-app
  template:
    metadata:
      labels:
        app: demo-app
        environment: ${ENVIRONMENT}
    spec:
      containers:
        - name: app
          image: your-registry/app:${IMAGE_TAG}
          ports:
            - containerPort: 3000
          env:
            - name: NODE_ENV
              value: ${ENVIRONMENT}
            - name: DATABASE_URL
              valueFrom:
                secretKeyRef:
                  name: database-secret
                  key: url
          resources:
            requests:
              memory: "128Mi"
              cpu: "100m"
            limits:
              memory: "256Mi"
              cpu: "200m"
          livenessProbe:
            httpGet:
              path: /health
              port: 3000
            initialDelaySeconds: 30
            periodSeconds: 10
          readinessProbe:
            httpGet:
              path: /ready
              port: 3000
            initialDelaySeconds: 5
            periodSeconds: 5
---
apiVersion: v1
kind: Service
metadata:
  name: app-service
  namespace: ${ENVIRONMENT}
spec:
  selector:
    app: demo-app
  ports:
    - port: 80
      targetPort: 3000
  type: ClusterIP
```

### 4.3 Ingress and Load Balancer Configuration

Now that the stalls are open and internal staff can order from each other, the most important question is: how do outside customers get in and find our stalls?

A simple and crude method is to give each stall a direct entrance to the outside (Service type LoadBalancer). This would create an independent AWS Load Balancer for each service, which would be incredibly expensive with dozens of services.

A more elegant and economical approach is to use **`Ingress`**. `Ingress` is an API object that manages external traffic entering the cluster. It provides HTTP/HTTPS-based routing rules. It's like the entire food court having only one main entrance, and at the entrance, there's a huge electronic directory board that says:
- To go to "A+ Fried Chicken" (`host: food.com, path: /chicken` ) -> Please go to Area B, No. 52 (directs to `chicken-service`)
- To go to "C- Pizza House" (`host: food.com, path: /pizza` ) -> Please go to Area C, No. 03 (directs to `pizza-service`)

The two key components that make up our entrance (Ingress) system are:
1. **Ingress Controller - "The Guard and Guide at the Entrance"**: It is a program that actually runs in the cluster (itself also a Pod). It is responsible for listening to the Ingress rules we define and actually configuring the AWS Application Load Balancer (ALB) at the main entrance. In an EKS environment, the most commonly used is the AWS Load Balancer Controller.
2. **Ingress Resource - "The Directory Map Rules We Write"**: This is a YAML file. We submit this file to Kubernetes to tell the Ingress Controller the routing rules we want.

The huge advantage of Ingress is that we can use one ALB as the traffic entry point for dozens or even hundreds of backend services, and accurately route traffic to the corresponding Service through different domain names or URL paths. This greatly saves costs and simplifies the management of external traffic.

```yaml
# k8s/ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: app-ingress
  namespace: ${ENVIRONMENT}
  annotations:
    kubernetes.io/ingress.class: "alb"
    alb.ingress.kubernetes.io/scheme: internet-facing
    alb.ingress.kubernetes.io/target-type: ip
    alb.ingress.kubernetes.io/certificate-arn: ${CERTIFICATE_ARN}
    alb.ingress.kubernetes.io/ssl-redirect: "443"
spec:
  rules:
    - host: ${ENVIRONMENT}.example.com
      http:
        paths:
          - path: / 
            pathType: Prefix
            backend:
              service:
                name: app-service
                port:
                  number: 80
```

### ECS vs. EKS

Today (?), from a macro perspective of cluster building to the micro perspective of application deployment and traffic ingress management, we have walked through the core path of Kubernetes completely. Although this system is complex, it is the skeleton that supports today's large-scale microservices applications.

Finally, let's briefly compare the differences between the two:
> ### ECS is a well-organized fine dining restaurant:
> AWS has defined many rules for us. We just need to focus on the dishes (our application). It has high integration and a gentle learning curve.

> ### EKS/Kubernetes is a vibrant international food court:
> It provides a set of extremely powerful, flexible, and vendor-neutral "operating standards." We need to learn its entire set of terms and concepts (Pod, Deployment, Service, Ingress...), but the return is unparalleled flexibility, scalability, and a vibrant ecosystem.

## 5. CI/CD Pipeline Integration

Given that we have already talked about the integration and process concepts of CI/CD in <Full CI/CD Automation Implementation - GitHub Actions × CodePipeline × CodeBuild>, we will now briefly describe the process and organize fragmented jobs, and then move on to the **`Blue/Green Deployment`** section, which combines the multi-environment applications of <Infrastructure as Code: Terraform-based Infrastructure Codification and Version Control> and <Full CI/CD Automation Implementation - GitHub Actions × CodePipeline × CodeBuild>.

### 5.1 GitHub Actions Workflow

To review, the most important concept of the CI/CD process is to `establish a stable and fixed business logic verification and delivery process,` **`actively protecting existing business logic from being corrupted by code changes.`**

Whenever code is pushed to the code repository (e.g., GitHub), the pipeline starts automatically. It fetches the latest code and executes a series of "quality checks" such as compilation, unit tests, and code quality scans. This is like the "quality control" part of a food factory. Whenever new raw materials arrive, the quality inspector immediately takes samples to ensure that this batch of materials is okay and will not contaminate the entire production line - the goal of CI is to **discover problems as early as possible**. When all CI quality control steps are passed, the pipeline proceeds to the next step: packaging (creating a Docker Image), pushing to the repository (ECR), and then automatically triggering deployment to the Dev -> Staging -> Prod environments. The goal of CD is to make releases a **routine**, **stable**, and **trackable** event, just like quality-approved raw materials being automatically sent to the production line, processed, cooked, vacuum-packed, and finally delivered directly to trucks by conveyor belt for distribution to major stores.

```yaml
# .github/workflows/deploy.yml
name: Multi-Environment Deployment

on:
  push:
    branches: [release, develop]
  pull_request:
    branches: [release]

env:
  AWS_REGION: us-west-2
  ECR_REPOSITORY: demo-app

jobs:
  build:
    runs-on: ubuntu-latest
    outputs:
      image-tag: ${{ steps.build.outputs.image-tag }}
    steps:
      - uses: actions/checkout@v3

      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v2
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: ${{ env.AWS_REGION }}

      - name: Login to Amazon ECR
        id: login-ecr
        uses: aws-actions/amazon-ecr-login@v1

      - name: Build and push Docker image
        id: build
        run: |
          IMAGE_TAG=${GITHUB_SHA:0:8}
          docker build -t $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG .
          docker push $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG
          echo "image-tag=$IMAGE_TAG" >> $GITHUB_OUTPUT

  deploy-dev:
    if: github.ref == 'refs/heads/develop'
    needs: build
    runs-on: ubuntu-latest
    environment: development
    steps:
      - name: Deploy to Development
        run: |
          # ECS deployment
          aws ecs update-service \
            --cluster dev-cluster \
            --service app-service \
            --task-definition app-service:LATEST

  deploy-staging:
    if: github.ref == 'refs/heads/main'
    needs: build
    runs-on: ubuntu-latest
    environment: staging
    steps:
      - name: Deploy to Staging
        run: |
          # Kubernetes deployment
          kubectl set image deployment/app-deployment \
            app=$ECR_REGISTRY/$ECR_REPOSITORY:${{ needs.build.outputs.image-tag }} \
            -n staging

  deploy-prod:
    if: github.ref == 'refs/heads/main'
    needs: [build, deploy-staging]
    runs-on: ubuntu-latest
    environment: production
    steps:
      - name: Deploy to Production
        run: |
          # Blue-Green deployment
          ./scripts/blue-green-deploy.sh ${{ needs.build.outputs.image-tag }}
```

And according to what we said in <Full CI/CD Automation Implementation - GitHub Actions × CodePipeline × CodeBuild>, <Enterprise-level Job Modularization and Cross-Domain Referencing>, when we need to deploy, we can extract common processes (Jobs), pipeline them for management, and perform automated deployment based on the behavior of the current branch.

```yaml
# build
parameters:
  - name: ngCliVersion
    type: string
    default: latest
    values:
      - latest
      - 16
      - 12
      - 11
jobs:
  build:
    runs-on: ubuntu-latest
    outputs:
      image-tag: ${{ steps.build.outputs.image-tag }}
    steps:
      - uses: actions/checkout@v3

      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v2
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: ${{ env.AWS_REGION }}

      - name: Login to Amazon ECR
        id: login-ecr
        uses: aws-actions/amazon-ecr-login@v1

      - name: Build and push Docker image
        id: build
        run: |
          IMAGE_TAG=${GITHUB_SHA:0:8}
          docker build -t $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG .
          docker push $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG
          echo "image-tag=$IMAGE_TAG" >> $GITHUB_OUTPUT
```

```yaml
#  deploy-dev
parameters:
  - name: ngCliVersion
    type: string
    default: latest
    values:
      - latest
      - 16
      - 12
      - 11
jobs:
  deploy-dev:
    if: github.ref == 'refs/heads/develop'
    needs: build
    runs-on: ubuntu-latest
    environment: development
    steps:
      - name: Deploy to Development
        run: |
          # ECS deployment
          aws ecs update-service \
            --cluster dev-cluster \
            --service app-service \
            --task-definition app-service:LATEST
```

```yaml
#  deploy-staging
parameters:
  - name: ngCliVersion
    type: string
    default: latest
    values:
      - latest
      - 16
      - 12
      - 11

jobs:
  deploy-staging:
    if: github.ref == 'refs/heads/main'
    needs: build
    runs-on: ubuntu-latest
    environment: staging
    steps:
      - name: Deploy to Staging
        run: |
          # Kubernetes deployment
          kubectl set image deployment/app-deployment \
            app=$ECR_REGISTRY/$ECR_REPOSITORY:${{ needs.build.outputs.image-tag }} \
            -n staging
```

```yaml
#  deploy-prod
parameters:
  - name: ngCliVersion
    type: string
    default: latest
    values:
      - latest
      - 16
      - 12
      - 11

jobs:
  deploy-prod:
    if: github.ref == 'refs/heads/main'
    needs: [build, deploy-staging]
    runs-on: ubuntu-latest
    environment: production
    steps:
      - name: Deploy to Production
        run: |
          # Blue-Green deployment
          ./scripts/blue-green-deploy.sh ${{ needs.build.outputs.image-tag }}
```

The extracted process would look like this:
```yaml
# .github/workflows/deploy.yml
name: Multi-Environment Deployment

on:
  push:
    branches: [release, develop]
  pull_request:
    branches: [release]

env:
  AWS_REGION: us-west-2
  ECR_REPOSITORY: demo-app

jobs:
  - template: jobs/build.yml@templates
  - template: jobs/deploy-dev.yml@templates
  - template: jobs/deploy-staging.yml@templates
    parameters:
      Version: ${{ variables.Version }}
      env: $(env)
      hasPreview: true
```

### 5.2 Blue/Green Deployment: Monolithic Instances (ECS/EC2) and Clusters (EKS)

#### Blue/Green Deployment for Monolithic Instances (ECS/EC2)
After briefly reviewing the CI/CD process and the independent splitting of `Jobs`, let's understand the practical application of multi-environment deployment - **`Blue/Green Deployment`**.

There are two main strategies in current system development. One is **Rolling Update**, which deploys after `traffic is reduced until the service is completely cool`, and the other is a `zero-downtime` release strategy - **Blue/Green Deployment**. A rolling update is like in an F1 race, where `quick repairs and updates` are only allowed when the car enters the pit stop and is completely stationary (`traffic reduced until the service is completely cool`). In the pit stop, you have to race against the countdown. Any mistake will delay the time to exit the pit stop. Even worse, if you get back on the track and find a problem with the new tire, the car is already in an unstable state, and you have to re-enter the pit stop to make adjustments - an even worse situation is having to **`perform emergency repairs on the track at high speed`**.

**`Blue/Green Deployment`**, on the other hand, offers a safer and more elegant philosophy. We don't work on the moving race car (`Blue environment`). On the side track, we prepare an identical new car with new tires installed (`Green environment`). We let this new car start, warm up, and check all the dashboards. When we are 100% sure that the new car is flawless (of course, this is the ideal situation, haha), we gradually switch the pressure to the new car. The old car is gradually decommissioned on the side. If there's a problem with the new car, we can **`instantly switch back`**.

- **Execution Steps:**
  1. **Initial State:** Assume our current production environment is V1, which we call the blue environment. All user traffic is directed to the blue environment's Target Group through the Application Load Balancer (ALB).
  2. **Deploy Green Environment:**
     - The CI/CD pipeline starts.
     - It will create a brand new, independent set of infrastructure (e.g., a new ECS Service or K8s Deployment) and deploy the V2 version of the software on it. This is the green environment. At this point, the green environment is running but has no real user traffic.
  3. **Verify Green Environment:**
     - This is the most critical advantage of blue-green deployment. Before switching traffic, we can perform complete, isolated automated testing on the green environment. For example, sending simulated requests from within the pipeline to the internal endpoint of the green environment to verify that its core functions are normal and the API returns correctly.
  4. **Traffic Switch (The Switch):**
     - When all tests have passed (usually with a "manual approval" step), the CI/CD pipeline will execute a single, atomic operation: modify the ALB Listener's rules to redirect a **custom percentage** (depending on business needs, but 20% is a common ratio) of traffic from the blue Target Group to the green Target Group. This switch is instantaneous to the user; they won't feel any service interruption.
  5. **Standby and Decommission:**
     - After the switch, the green environment gradually takes over and officially becomes the new production environment.
     - The old blue environment is not destroyed immediately. It remains on standby as a "hot backup." If the monitoring system detects a serious problem with the V2 version a few minutes or hours after going live, we can perform a "one-click rollback" - that is, perform another traffic switch to point the traffic back to the blue environment, achieving second-level disaster recovery.
     - After confirming that the new version has been running stably for a period of time, the CI/CD pipeline will perform the final cleanup work, destroying the blue environment to save costs.

AWS CodeDeploy is deeply integrated with ECS and EC2 and can automate the complex blue-green deployment process described above. We only need to do some simple configuration.

The division of labor and collaboration model of **`"managing environment state with IaC, managing the deployment lifecycle with CI/CD, and gradually verifying environment differences"`** that we just discussed is currently the most talked-about, recognized **Optimal Implementation** and **Gold Standard** in the Cloud-Native field.

"Optimal" does not mean it is the "simplest" or "fastest to build" solution, but that it achieves the most elegant balance between several key, often conflicting, engineering goals. First, it balances **"Stability"** and **"Agility."** In the past, to make a system stable meant reducing changes, with deployment cycles of possibly several months. Wanting to iterate quickly (agile) often sacrificed stability, leading to frequent online errors. The modern cloud-native approach uses IaC to guarantee the cornerstone of "stability." Our infrastructure is no longer a black box. Every change has a record, is reviewed (Code Review), and is traceable. This eliminates most stability problems caused by "manual errors" or "inconsistent environments" - the infrastructure becomes as solid as a rock. CI/CD provides the engine for "agility." On this solid foundation, the automated pipeline makes application deployment extremely fast and low-risk. Developers can deploy dozens of times a day without worrying about crashing the underlying infrastructure. Strategies like blue-green deployment provide a top-level safety net for this high-speed iteration.

Second, there is often a chasm between the requirements team (Require) and the operations team (Maintain). Requirers want to quickly deliver new features, while maintainers want to stabilize existing functions. Mutual blame and inefficiency are the norms. But `"managing environment state with IaC, managing the deployment lifecycle with CI/CD, and gradually verifying environment differences"` strikes a balance between **"Clear Responsibilities"** and **"Team Collaboration."** Through a Clear Boundary, our division of labor model provides an extremely clear "contract":
- **Platform/Ops Team:** Their core output is a stable, reliable, and standardized infrastructure platform (defined by Terraform). They are like city planners, responsible for building water, electricity, gas pipelines, and standardized building foundations.
- **Application/Dev Team:** Their core output is high-quality applications (encapsulated in Docker) and a safe and reliable deployment process (defined by GitHub Actions Workflow). They are like construction teams building houses on standard foundations, only needing to focus on the house itself without worrying about where the water and electricity come from.

Once the platform team provides a stable platform, the application team can perform "self-service deployments" through the CI/CD pipeline without having to bother the platform team every time, greatly improving development autonomy and efficiency.

Finally, a balance is struck between **"Controllability"** and **"Complexity."** Modern cloud systems are indeed very complex, often involving hundreds of resources and settings. Although there is a learning curve, IaC is the only effective way to manage this complexity. It transforms vast cloud resources into human-readable, machine-executable structured text. We can review, test, and modularize our infrastructure just like any software. At the same time, CI/CD hides the complexity. For most developers, they don't need to understand the complex details of ALB rule switching behind blue-green deployment. They only need to know that "when I merge the code into the main branch and click the 'approve' button in the Pipeline, my new feature will be safely deployed online" - the CI/CD pipeline encapsulates the complex deployment process into a simple, reliable automated process.

#### Blue/Green Deployment for Clusters (EKS)
Now that we know the "philosophy" of blue/green deployment and its application at the ECS and EC2 levels, let's see how Kubernetes, the "food court manager," uses its tools to perform this precise traffic switching operation.

Unlike ECS, Kubernetes does not have a ready-made resource called `blue-green-deployment`. However, it provides a series of more flexible and powerful basic components (which we call primitives) that allow us to build more flexible blue-green deployment processes than AWS CodeDeploy, like assembling Legos. We will focus on the two most mainstream and Kubernetes-design-philosophy-embodying implementation methods in the industry: **`Service layer switching`** and **`Ingress layer switching`**.

To review, a key idea in Kubernetes is: **`Deployments/Pods (application instances)`** are the "stalls" that are actually running; they are **changeable** and **ephemeral**. And **`Services/Ingresses (service entrances)`** are the "fixed stall numbers" or "food court entrances" provided to internal and external customers; they should be **stable and unchanging**. The essence of K8s blue-green deployment is to keep the "service entrance" unchanged and only change the "application instance" it points to.

##### Service Selector Switching (The most classic K8s approach)
This is the method that best embodies the core concept of Kubernetes, by modifying the Service's selector to instantly switch traffic. According to the food court case we used when discussing EKS, the food court's "B-52 calling device" (Service) itself does not move. We just, in its backend system, instantly switch the service staff from the "blue uniform team" to the "green uniform team."

Illustrative Steps:
1. **Initial State (Blue environment running):**
   - We have a `Deployment-Blue` that manages `v1` version Pods. These Pods are all labeled `version: blue`.
   ```yaml
   # deployment-blue.yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: myapp-blue
   spec:
     replicas: 3
     template:
       metadata:
         labels:
           app: myapp
           version: blue # Key label
       spec:
         containers:
         - name: myapp
           image: my-app:v1
   ```
   - We have a stable `Service`. Its `selector` will only find `Pods` with the label `version: blue` and forward traffic to them.
   ```yaml
   # service.yaml
   apiVersion: v1
   kind: Service
   metadata:
     name: myapp-service # This is the stable, unchanging entry point
   spec:
     selector:
       app: myapp
       version: blue # Currently points to blue
     ports:
     - protocol: TCP
       port: 80
       targetPort: 8080
   ```
   At this point, all traffic flows to the `v1` Pods.

2. **Deploy Green Environment:**
   - The CI/CD pipeline will create a brand new `Deployment-Green` that manages `v2` version `Pods`, and these `Pods` have the label `version: green`.
   ```yaml
   # deployment-green.yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: myapp-green
   spec:
     replicas: 3
     template:
       metadata:
         labels:
           app: myapp
           version: green # Key label
       spec:
         containers:
         - name: myapp
           image: my-app:v2
   ```
   - After deployment is complete, the v2 Pods are running healthily in the cluster, but because the `myapp-service` selector is still `version: blue`, the green environment receives no official traffic at all.

3. **Verify Green Environment:**
   - At this time, we can create an internal-only test Service, such as `myapp-green-test-service`, whose selector points to `version: green`. The CI/CD pipeline can use the address of this internal Service to perform complete automated testing on the green environment.

4. **Execute Traffic Switch (The Switch):**
   - When everything is ready, the CI/CD pipeline will execute a single, atomic command:
   ```bash
   kubectl patch service myapp-service -p '{"spec":{"selector":{"version":"green"}}}'
   ```
   - Kubernetes' core networking will instantly update the routing rules. All subsequent traffic to the stable `myapp-service` entry point will be immediately directed to the `Pods` with `version: green`. Switch complete!

5. **Rollback and Cleanup:**
   - If a problem occurs with the `v2` version, the rollback operation is just as simple and fast:
   ```bash
   kubectl patch service myapp-service -p '{"spec":{"selector":{"version":"blue"}}}'
   ```
   - After confirming that `v2` is running stably, the CI/CD pipeline can safely delete the old blue environment:
   ```bash
   kubectl delete deployment myapp-blue
   ```

##### Ingress Layer Switching (More suitable for microservices and Canary releases)
This method goes one step further. It does not touch the Service, but modifies the Ingress routing rules. This time, the "blue team" and the "green team" have their own independent internal calling devices (`service-blue` and `service-green`). We modify the directory board (Ingress) at the main entrance of the food court, changing the direction for "A+ Fried Chicken" from "blue counter" to "green counter."

Illustrative Steps:
1. **Initial State:**
   - We have `deployment-blue` and `deployment-green`.
   - There are two `Services`: `service-blue` points to `Pods` with `version: blue`, and `service-green` points to `Pods` with `version: green`.
   - There is an `Ingress` resource whose rules point to `service-blue`.
   ```yaml
   # ingress.yaml
   apiVersion: networking.k8s.io/v1
   kind: Ingress
   metadata:
     name: myapp-ingress
   spec:
     rules:
     - host: myapp.example.com
       http:
         paths:
         - path: / 
           pathType: Prefix
           backend:
             service:
               name: service-blue # Points to the blue service
               port:
                 number: 80
   ```

2. **Traffic Switching:**
   - The CI/CD pipeline executes a command to modify the `Ingress` resource, changing the `backend.service.name` from `service-blue` to `service-green`.
   ```bash
   # Simple example, in practice you would use kubectl patch or apply -f
   # Modify the service name in ingress.yaml and then execute apply
   kubectl apply -f ingress.yaml
   ```
   - The Ingress Controller (e.g., AWS Load Balancer Controller) detects the change and updates the ALB's forwarding rules to route traffic to `service-green`.

This model is the basis for implementing Canary Releases. We can configure the Ingress to send 95% of the traffic to `service-blue` and 5% to `service-green` to achieve small-scale grayscale verification. Also, because there are fewer changes at the Service layer, it better fits its positioning as a "stable entry point," and responsibilities are clearer.

The mental model we have now mastered, which combines `IaC, containerization, CI/CD, and advanced deployment strategies`, is not just a set of technical operating instructions; it is a **philosophy of modern software engineering**. It is the core secret that allows top tech companies like Google, Netflix, and Amazon to perform thousands of deployments per week while still maintaining highly stable systems. Because it does not simply solve a technical problem, but a systemic engineering problem. It has a deep understanding that software delivery is not just about writing code, but also includes testing, deployment, operations, team collaboration, risk control, and a series of other links.

In your future career, regardless of how the tools evolve (from Terraform to OpenTofu, from GitHub Actions to GitLab CI), the core ideas behind `separating state and process`, and `declarative and imperative collaboration` will remain applicable for a long time and will become a solid foundation for building high-quality software systems. Tools will evolve with the times, but the design philosophy is a process of logical deduction and dialectics, just like:
> Xunzi, "The Effects of the Ru": "A thousand moves and ten thousand changes, the Way is one."

Or as I prefer, from Zhuangzi:
> Zhuangzi, "Tianxia": "Not departing from the ancestor, one is called a man of Heaven."

## 6. Configuration Management and Secrets Management
Finally, for today(?), let's talk about how to manage `environment parameters`, `authorization public keys`, and `connection ports`.

So far, our automated food court is very advanced: we have standardized stall blueprints (IaC), standardized meal kits (Docker), an efficient manager (ECS/EKS), and a fully automated delivery pipeline (CI/CD).

But now, we have to deal with an extremely sensitive and important issue: **Where do we put the "exclusive secret recipe" (Secrets) for each stall?**

We can't just stick a piece of paper with the secret marinade recipe for "A+ Fried Chicken" on the wall of the `Dockerfile` or the `Git` repository, right? That would be a security disaster. So next, we need to learn the spirit of Mr. Krabs and hold on tight to our confidential data, separating "Configuration" and "Secrets" from our application code, and "injecting" them only when needed to the right application.

**K8s Native Solutions — ConfigMap & Secret**
Kubernetes provides two native resources to handle this problem.

1. **ConfigMap - "Public Menus and Announcements"**
   - **Purpose:** Used to store non-sensitive configuration data.
   - **Analogy:** Imagine the announcement board at the food court. It has all kinds of public information posted on it:
     - `API_URL: https://api.internal.mycompany.com` (the address of the backend API)
     - `FEATURE_FLAG_NEW_UI: "true"` (a feature flag to enable the new UI)
     - `LOG_LEVEL: "info"` (the detail level for logging)
   - **Characteristics:**
     - Stores plain text data in key-value pairs.
     - Very flexible, can be used by Pods in two main ways:
       1. **Injected as Environment Variables:** This is the most common way.
       2. **Mounted as a file:** When the configuration content is large or has a complex format (e.g., an `nginx.conf` file), the entire ConfigMap can be mounted into the Pod's filesystem.
   - **Key Point:** ConfigMap is plain text and must never be used to store any sensitive information.

2. **Secret - "Secret Recipe Locked in an Ordinary Drawer"**
   - **Purpose:** Used to store sensitive data.
   - **Analogy:** This is like the "secret recipe notebook" that the stall manager keeps in an ordinary drawer in his office. It might say:
     - `DATABASE_PASSWORD: "mypassword123"`
     - `STRIPE_API_KEY: "sk_live_..."`
   - **Characteristics:**
     - Also stores data in key-value pairs.
     - The biggest difference from ConfigMap is that when Kubernetes stores the value of a Secret, it Base64 encodes it.
       - **Base64 is not encryption!** It's just an encoding method. Anyone who gets the encoded string can easily decode it back to the original text.
       1. **Injected as Environment Variables:** This is the most common way.
       2. **Mounted as a file:** When the configuration content is large or has a complex format (e.g., an `nginx.conf` file), the entire Secret can be mounted into the Pod's filesystem.
   - **Key Point:** The security of a Kubernetes Secret mainly relies on RBAC (Role-Based Access Control). We can set strict permissions to specify that only certain Pods or administrators have the right to "read" this Secret object. It protects against gentlemen, not villains, and mainly prevents accidental information leakage.

```yaml
# k8s/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
  namespace: ${ENVIRONMENT}
data:
  NODE_ENV: ${ENVIRONMENT}
  LOG_LEVEL: ${LOG_LEVEL}
  API_BASE_URL: ${API_BASE_URL}
---
apiVersion: v1
kind: Secret
metadata:
  name: database-secret
  namespace: ${ENVIRONMENT}
type: Opaque
data:
  url: ${DATABASE_URL_BASE64}
```

But for a serious production environment, simply using K8s Secret is not enough. As we just said, it `protects against gentlemen, not villains, and mainly prevents accidental information leakage.` It's like locking the secret recipe in a drawer that anyone can easily pry open. We are still missing:
- **Encryption at Rest:** The secret in the database (etcd) is only encoded, not encrypted.
- **Audit Trail:** Who peeked at the secret recipe and when? (Did someone peek?) We have no way of knowing.
- **Automatic Rotation:** For security, passwords should be changed regularly. Manual rotation is both tedious and error-prone.

**AWS's Top-Tier Safe — Secrets Manager**
To solve the above problems, cloud service providers offer dedicated "secrets management services." On AWS, this service is AWS Secrets Manager.

- **Purpose:** Centrally store, manage, audit, and rotate all your sensitive secrets.
- **Analogy:** This is no longer an ordinary drawer, but a bank-grade, top-tier safe.
- **Core Advantages:**
  1. **Strong Encryption:** All secrets are strongly encrypted using AWS KMS (Key Management Service) when stored.
  2. **Fine-grained IAM Permission Control:** You can use IAM Policies to specify precisely "which application (which IAM Role) can only read which version of which secret."
  3. **Complete Audit Trail:** Deeply integrated with CloudTrail. Every read, modification, and deletion of a secret is recorded in detail for security team audits.
  4. **Automatic Rotation:** This is the trump card of Secrets Manager. It can be integrated with services like RDS to achieve fully automatic regular rotation of database passwords, and your application does not need to modify the code or restart.

```yaml
# terraform/secrets.tf
resource "aws_secretsmanager_secret" "database_url" {
  name = "${var.environment}/database-url"
  description = "Database connection string for ${var.environment}"

  tags = {
    Environment = var.environment
    Project     = "multi-env-demo"
  }
}

resource "aws_secretsmanager_secret_version" "database_url" {
  secret_id     = aws_secretsmanager_secret.database_url.id
  secret_string = jsonencode({
    url = var.database_url
  })
}
```

**Best Practice: Let K8s Pods Fetch Directly from the AWS Safe**
Now, we have K8s applications (Pods) and an AWS safe (Secrets Manager). The question is, how can a Pod securely get the secret recipe from the safe, instead of through unreliable manual transmission?

This requires a key "bridge" technology: IAM Roles for Service Accounts (IRSA), and an important tool: AWS Secrets and Configuration Provider (ASCP).

Just as we wouldn't tell the stall chef (Pod) the password to the safe. Instead, we issue this chef a special **"Authorized Employee Card" (Service Account with an IAM Role)**. When the chef needs the secret recipe, he uses this card to swipe a specific gate at the bank (ASCP). After the banking system verifies the card's permissions, it will automatically and temporarily put the page of the secret recipe needed for today into a dedicated, locked box in front of him (mounted as a file in the Pod). The box is destroyed after the chef is done.

**Operational Flow:**
1. **Set up IRSA:**
   - In AWS IAM, create a Role and authorize it to read the `my-app/db-password` Secret.
   - In EKS, create a Kubernetes ServiceAccount.
   - "Bind" these two, telling EKS: "Any Pod that uses this ServiceAccount can assume the identity of that IAM Role."
2. **Specify ServiceAccount in Deployment:**
   - In your `deployment.yaml`, explicitly specify `spec.template.spec.serviceAccountName` as the ServiceAccount you just created.
3. **Mount Secrets using ASCP:**
   1. You first need to install the AWS Secrets and Configuration Provider add-on in your EKS cluster.
   2. Then, in your `deployment.yaml`, define a `Volume` and tell ASCP:
      1. I want to mount a secret.
      2. The name of the secret is `my-app/db-password` (the name in Secrets Manager).
      3. Please mount it to the path `/etc/secrets/db-password` in the Pod.
4. **Application Reads the Secret:**
   - After your application starts, it no longer reads the password from an environment variable, but directly from the local file `/etc/secrets/db-password`.

The huge advantages of this solution:
- **Highest Security:** The secret never appears in plain text in any Kubernetes resource (YAML, environment variables). It is transmitted encrypted from AWS Secrets Manager and mounted directly into the Pod's memory filesystem.
- **Zero Code Coupling:** Your application doesn't know or care whether the secret comes from AWS or elsewhere; it just needs to read from a fixed file path.
- **Dynamic Updates:** When you update the secret's value in Secrets Manager, ASCP can update the mounted file content in the Pod almost in real-time, and the application can dynamically read the latest value.

Finally, we have created a layered configuration and secrets management strategy:
> **ConfigMap:** Used to store public application configurations.
> **K8s Secret:** As a basic secrets management tool, mainly protected by RBAC, suitable for some less sensitive scenarios.
**AWS Secrets Manager + IRSA + ASCP** is the gold standard and best practice for managing highly sensitive secrets on EKS. It provides end-to-end encryption, auditing, and automatic rotation capabilities, and is a necessary part of building a secure and reliable system.

Let's leave it on a cliffhanger for now. We will discuss the three topics of **Monitoring and Logging Management**, **Security Best Practices**, and **Disaster Recovery and Backup** in detail in the future **Release and Operational Monitoring (Security and Compliance)** stage. What we mainly discussed today (?) is **Software Development and Continuous Integration - Dev / Staging / Prod Multi-Environment Governance and Architecture Strategy**. And tomorrow, we will discuss the last content of **Software Development and Continuous Integration** - Developer Experience (DX) Optimization: Internal Tools and Troubleshooting Design.
