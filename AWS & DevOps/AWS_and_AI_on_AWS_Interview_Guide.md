# AWS & AI on AWS — Senior Software Engineer Interview Guide

Each section explains **what the service is, why it exists, key concepts, real-world uses**, followed by **5+ frequently asked interview questions with senior-level answers** (trade-offs, internals, failure modes — not just definitions).

> Service limits and features evolve fast. Numbers here reflect AWS as of 2025–2026; always double-check the latest docs before quoting exact limits in an interview.

---

## Table of Contents

**Part 1 — Core AWS**
1. Compute: [EC2](#1-ec2-elastic-compute-cloud) · [Auto Scaling](#2-auto-scaling-groups-asg) · [Lambda](#3-aws-lambda) · [ECS/Fargate/ECR](#4-ecs-fargate--ecr) · [EKS](#5-eks-elastic-kubernetes-service) · [App Runner & Beanstalk](#6-app-runner--elastic-beanstalk)
2. Networking: [VPC](#7-vpc-virtual-private-cloud) · [VPC Connectivity](#8-vpc-connectivity-peering-transit-gateway-privatelink-endpoints) · [Load Balancers](#9-elastic-load-balancing-alb-nlb-gwlb) · [Route 53](#10-route-53) · [CloudFront](#11-cloudfront) · [Hybrid & Global](#12-direct-connect-site-to-site-vpn-global-accelerator)
3. Storage & DB: [S3](#13-s3-simple-storage-service) · [EBS/EFS/FSx](#14-ebs-efs-fsx) · [RDS & Aurora](#15-rds--aurora-incl-aurora-dsql-rds-proxy) · [DynamoDB](#16-dynamodb) · [ElastiCache/MemoryDB](#17-elasticache--memorydb-redis--valkey) · [Analytics & Purpose-built DBs](#18-purpose-built-data-services-redshift-athena-opensearch-neptune-documentdb)
4. Messaging & Integration: [SQS](#19-sqs-simple-queue-service) · [SNS](#20-sns-simple-notification-service) · [EventBridge](#21-eventbridge) · [Kinesis & MSK](#22-kinesis--msk) · [Step Functions](#23-step-functions) · [API Gateway](#24-api-gateway) · [AppSync](#25-appsync)
5. Security & Identity: [IAM](#26-iam-identity-and-access-management) · [Organizations/SCP/Control Tower](#27-aws-organizations-scps-rcps-control-tower) · [Identity Center & Cognito](#28-iam-identity-center--cognito) · [KMS/Secrets/SSM](#29-kms-secrets-manager-ssm-parameter-store) · [Security Services](#30-security-services-waf-shield-guardduty-inspector-security-hub-macie-cloudtrail-config) · [Verified Permissions/Access](#31-verified-permissions--verified-access)
6. Observability: [CloudWatch](#32-cloudwatch) · [X-Ray, ADOT, AMP, AMG](#33-x-ray-adot-managed-prometheus--grafana)
7. Architecture: [Well-Architected](#34-well-architected-framework) · [HA & DR](#35-high-availability--disaster-recovery) · [Multi-Region & Cells](#36-multi-region--cell-based-architecture) · [Resilience Patterns](#37-distributed-system-patterns-on-aws) · [Cost/FinOps](#38-cost-optimization--finops) · [Migration 7 Rs](#39-migration-strategies-the-7-rs) · [Shared Responsibility](#40-shared-responsibility-model)
8. IaC & CI/CD: [CloudFormation](#41-cloudformation) · [CDK](#42-aws-cdk) · [SAM](#43-aws-sam) · [CodePipeline/Build/Deploy](#44-codepipeline-codebuild-codedeploy)

**Part 2 — AI / GenAI on AWS**
[Bedrock](#45-amazon-bedrock) · [Knowledge Bases](#46-bedrock-knowledge-bases-managed-rag) · [Bedrock Agents](#47-bedrock-agents) · [AgentCore](#48-bedrock-agentcore) · [Guardrails](#49-bedrock-guardrails) · [Evals/Distillation/Prompt Mgmt/Flows](#50-bedrock-evaluations-model-distillation-prompt-management-flows) · [Nova](#51-amazon-nova-models) · [Strands Agents](#52-strands-agents-sdk) · [Q Developer](#53-amazon-q-developer) · [Kiro](#54-kiro) · [Frontier Agents](#55-aws-frontier-agents-devops-agent-security-agent-kiro-autonomous-agent) · [AWS MCP Servers](#56-aws-mcp-servers) · [Q Business](#57-amazon-q-business) · [SageMaker AI](#58-sagemaker-ai) · [DevOps Guru & CodeGuru](#59-devops-guru--codeguru) · [Trainium & Inferentia](#60-trainium--inferentia) · [GenAI System Design](#61-genai-system-design-on-aws--scenario-questions)

---

# PART 1 — CORE AWS

---

## 1. EC2 (Elastic Compute Cloud)

**What it is:** Resizable virtual machines (instances) in AWS data centers. The foundational IaaS building block — almost every other compute service (ECS on EC2, EKS nodes, EMR, SageMaker) runs on EC2 underneath.

**Purpose:** Full control over the OS, runtime, networking and storage when managed/serverless options don't fit (legacy apps, licensed software, GPU workloads, custom kernels, long-running stateful processes).

**Key concepts**
- **Instance families:** `t` (burstable, CPU credits), `m` (general purpose), `c` (compute), `r`/`x` (memory), `i`/`d` (storage/NVMe), `p`/`g` (GPU), `trn`/`inf` (AWS ML chips). Suffix `g` = Graviton (ARM), e.g. `m7g`.
- **Nitro System:** AWS hypervisor offloading networking/storage/security to dedicated hardware → near-bare-metal performance, enables ENA, EBS-optimized by default.
- **AMI:** Machine image (OS + software); region-scoped, can be copied/shared.
- **User data:** Bootstrap script run on first boot (cloud-init).
- **Instance metadata (IMDS):** `169.254.169.254` — IMDSv2 (session tokens) is required for security (mitigates SSRF credential theft).
- **Placement groups:** *Cluster* (low latency, same rack — HPC), *Spread* (max 7 per AZ, distinct hardware — critical instances), *Partition* (Kafka/HDFS/Cassandra — rack-aware).
- **Storage:** EBS (network block, persistent) vs Instance Store (ephemeral NVMe, lost on stop).

**Pricing models**
| Model | Discount | Use for |
|---|---|---|
| On-Demand | 0% | Spiky, unknown workloads |
| Savings Plans (Compute / EC2 Instance) | up to ~66–72% | Steady baseline, flexible across family/region (Compute SP also covers Fargate & Lambda) |
| Reserved Instances (Standard/Convertible) | up to ~72% | Steady, specific instance type; can sell Standard RIs |
| Spot | up to ~90% | Fault-tolerant, stateless, batch, CI, Karpenter nodes — 2-min interruption notice |
| Dedicated Hosts | — | BYOL licensing, compliance |

**Uses:** Web/app servers, batch processing, GPU training/inference, self-managed databases, CI runners, Windows workloads.

### Interview Questions

**Q1. How do you choose between On-Demand, Savings Plans, Reserved Instances and Spot?**
Layer them. Cover the steady 24×7 baseline (e.g. the minimum ASG capacity) with Compute Savings Plans — they're more flexible than RIs (any family, size, region, OS, plus Fargate/Lambda). Use Spot for fault-tolerant, stateless capacity above baseline (batch, CI, stateless web tiers behind ALB, Karpenter-managed K8s nodes) with diversification across many instance types and AZs and `capacity-optimized`/`price-capacity-optimized` allocation. Leave On-Demand for the unpredictable remainder. Revisit coverage/utilization monthly with Cost Explorer recommendations.

**Q2. How do you handle Spot interruptions gracefully?**
Spot gives a 2-minute warning via instance metadata and an EventBridge event (plus an earlier *rebalance recommendation*). Strategies: (1) stateless design — work must be checkpointed or idempotent; (2) diversify across 10+ instance types and all AZs; (3) use ASG *Capacity Rebalancing* to launch a replacement proactively; (4) on EKS use Karpenter or the Node Termination Handler to cordon/drain nodes; (5) for batch, checkpoint progress to S3/DynamoDB; (6) use mixed instances policy with an On-Demand base.

**Q3. What is IMDSv2 and why does it matter?**
The instance metadata service exposes the instance role's temporary credentials. IMDSv1 is a simple GET, so an SSRF vulnerability in an app could leak credentials (the Capital One 2019 breach). IMDSv2 requires a PUT to obtain a session token (with a TTL) and sends it as a header; PUT with a custom header isn't typical in SSRF and the token response has hop-limit = 1 by default so containers/proxies can't easily forward it. Enforce `HttpTokens=required` via launch templates, account-level defaults, or SCP.

**Q4. Why would you move to Graviton, and what are the risks?**
Graviton (ARM) instances typically offer ~20–40% better price-performance and lower energy use. Risks: native dependencies compiled only for x86, older container images without `arm64` variants, some commercial agents/licensed software. Approach: build multi-arch images (`docker buildx --platform linux/amd64,linux/arm64`), canary a slice of traffic onto Graviton, compare p99 latency and cost, then roll out. Interpreted/JIT runtimes (Java, Node, Python, Go) usually migrate with zero code change.

**Q5. An EC2 instance is unreachable via SSH. How do you troubleshoot?**
Work layer by layer: (1) instance status checks — system check failure → stop/start to migrate host; instance check failure → OS problem (check *Get system log*/serial console). (2) Security group allows port 22 from your IP? (3) NACL allows inbound 22 **and** outbound ephemeral ports (1024–65535)? (4) Route table has a route to IGW (public subnet) and the instance has a public IP/EIP? (5) Key pair/username correct? (6) Disk full or sshd crashed — use EC2 Serial Console or SSM. Better long-term answer: don't use SSH at all — use **SSM Session Manager** (no open ports, IAM-controlled, audited in CloudTrail).

**Q6. Instance store vs EBS — when would you use instance store?**
Instance store is physically attached NVMe — very high IOPS and low latency, but data is lost on stop/terminate/host failure. Use it for caches, scratch space, buffers, or replicated data stores where replication provides durability (Cassandra, Kafka with replication factor 3, Elasticsearch). Use EBS for anything that must survive instance lifecycle.

---

## 2. Auto Scaling Groups (ASG)

**What it is:** A managed group of EC2 instances that maintains desired capacity, replaces unhealthy instances, and scales out/in based on policies.

**Purpose:** Elasticity (pay for what you need), self-healing, and AZ-balanced high availability.

**Key concepts**
- **Launch Template:** Versioned instance config (AMI, type, SG, user data, IAM profile). Launch Configurations are legacy.
- **Min / Desired / Max** capacity.
- **Scaling policies:**
  - *Target tracking* — "keep average CPU at 50%" or ALB `RequestCountPerTarget` (most common, recommended).
  - *Step scaling* — add N instances based on alarm breach size.
  - *Scheduled* — known patterns (business hours).
  - *Predictive* — ML forecasts daily/weekly cycles and pre-scales.
- **Health checks:** EC2 (status checks) or ELB (application health) — use ELB for web tiers.
- **Lifecycle hooks:** Pause on launch/terminate (e.g. warm caches, drain connections, ship logs).
- **Warm pools:** Pre-initialized stopped instances to cut scale-out time for slow-booting apps.
- **Instance refresh:** Rolling replacement to roll out a new AMI/launch template, with checkpoints and auto rollback.
- **Mixed instances policy:** Combine On-Demand base + Spot across multiple types.
- **Cooldown / instance warmup:** Prevents thrashing.

**Uses:** Stateless web/API fleets, worker fleets scaling on SQS backlog, EKS/ECS capacity providers.

### Interview Questions

**Q1. Your app takes 8 minutes to boot. How do you make scaling responsive?**
(1) Bake an AMI (EC2 Image Builder/Packer) instead of installing at boot; (2) use **warm pools** of stopped/hibernated pre-initialized instances; (3) **predictive scaling** for regular traffic patterns; (4) scale on a leading indicator (request count, queue depth) rather than CPU; (5) lower target utilization to keep headroom; (6) scheduled scaling ahead of known events; (7) longer term, containerize to reduce startup time.

**Q2. How do you scale a worker fleet that consumes from SQS?**
Don't scale on `ApproximateNumberOfMessagesVisible` alone — it doesn't account for fleet size. Use target tracking on a custom metric **backlog per instance** = visible messages ÷ running instances, with target = acceptable latency ÷ average processing time per message (e.g. 10 min tolerable latency ÷ 0.1 s per msg = 6000 msgs/instance). Publish the metric via a Lambda/CloudWatch metric math. Use scale-in protection while instances process messages.

**Q3. Why use ELB health checks instead of EC2 health checks?**
EC2 checks only detect hypervisor/OS failures. If the app process crashes or deadlocks, EC2 status is still "ok" and the broken instance stays in service. With ELB health checks, the ASG replaces instances that fail the application-level health endpoint. Make the health endpoint meaningful but shallow — don't fail on a downstream dependency outage or you'll cause a cascading mass replacement.

**Q4. How do you deploy a new AMI with zero downtime using ASG?**
Options: (1) **Instance Refresh** with `MinHealthyPercentage` (e.g. 90%), checkpoints, skip matching, and auto-rollback on CloudWatch alarms; (2) **blue/green** — new ASG behind the same ALB target group or a new target group with weighted listener rules, shift traffic, then delete old; (3) CodeDeploy blue/green for EC2. Blue/green gives instant rollback; instance refresh is cheaper/simpler.

**Q5. What's the difference between target tracking and step scaling, and which do you prefer?**
Target tracking behaves like a thermostat — you set the target and AWS creates/manages alarms and computes adjustments proportionally; it scales out fast and in conservatively. Step scaling requires you to define alarm thresholds and step sizes manually — useful for very specific, non-linear responses. Prefer target tracking (possibly multiple policies — ASG scales out if *any* says out and scales in only if *all* agree), add step scaling only for edge cases.

**Q6. How does ASG ensure AZ balance and what happens during an AZ outage?**
ASG distributes instances evenly across configured AZs. If an AZ fails, ELB health checks fail, ASG launches replacements in the healthy AZs. When the AZ recovers, ASG *rebalances* (launch new before terminating old). To survive an AZ loss without degradation, provision static stability: run enough capacity so that N-1 AZs can carry full load (e.g. 3 AZs at ≤66% utilization) instead of depending on scaling during the outage.

---

## 3. AWS Lambda

**What it is:** Serverless, event-driven compute. You upload a function; AWS runs it on demand in isolated Firecracker micro-VMs, scales automatically, and bills per request + GB-second (1 ms granularity).

**Purpose:** Run code without managing servers, scale to zero, pay only for execution — ideal for event-driven glue, APIs with variable traffic, and async processing.

**Key concepts & limits**
- Memory 128 MB–10 GB (CPU scales proportionally; ~1,769 MB = 1 vCPU), timeout **15 min max**, `/tmp` up to 10 GB.
- Payload: 6 MB synchronous; asynchronous invocation payload was raised to 1 MB in 2025.
- **Invocation models:**
  - *Synchronous* (API Gateway, ALB, SDK `RequestResponse`) — caller handles errors.
  - *Asynchronous* (S3, SNS, EventBridge) — Lambda queues internally, retries 2×, then **on-failure destination**/DLQ.
  - *Event source mapping / poll-based* (SQS, Kinesis, DynamoDB Streams, MSK, Kafka) — Lambda polls and invokes with batches.
- **Cold start:** New execution environment (download code, start runtime, run init code). Mitigations: Provisioned Concurrency, **SnapStart** (Java, Python, .NET — snapshot of initialized environment), smaller packages, init outside handler, avoid VPC-heavy init (VPC cold start penalty was largely fixed in 2019 with Hyperplane ENIs).
- **Concurrency:** Account default 1,000 per region (soft). *Reserved concurrency* = guaranteed and capped; *Provisioned concurrency* = pre-warmed environments.
- **Layers:** Share libraries/runtimes across functions. **Container images** up to 10 GB.
- **Lambda@Edge / CloudFront Functions:** Code at the edge.
- **Function URLs:** Built-in HTTPS endpoint without API Gateway.
- **Newer (2025):** *Lambda Managed Instances* (run Lambda on EC2 capacity you choose, for steady high-volume workloads with EC2 pricing) and *durable functions* (checkpointed, long-running multi-step workflows in code).

**Uses:** REST/GraphQL backends, S3 file processing, stream processing, cron jobs (EventBridge Scheduler), webhooks, glue between services, chatbot/tool backends for AI agents.

### Interview Questions

**Q1. What causes cold starts and how do you reduce them?**
A cold start happens when there's no warm execution environment: Lambda must provision a micro-VM, download code, start the runtime, and run your init code. It's worst for large packages, JVM/.NET runtimes, and heavy init (SDK clients, DI frameworks). Fixes: (1) keep packages small, tree-shake, lazy-load; (2) initialize clients outside the handler so they're reused; (3) **SnapStart** for Java/Python/.NET; (4) **Provisioned Concurrency** (with Application Auto Scaling schedules) for latency-critical paths; (5) choose faster runtimes (Node, Python, Go, Rust) or Graviton; (6) more memory = more CPU = faster init. Measure using `Init Duration` in REPORT logs.

**Q2. Reserved vs Provisioned concurrency?**
*Reserved* concurrency carves out part of the account pool for a function — it both **guarantees** capacity and **caps** the function (useful to protect a downstream DB from being overwhelmed; setting it to 0 throttles the function entirely). It costs nothing. *Provisioned* concurrency keeps N environments initialized and warm to eliminate cold starts; it costs money even when idle. They're often combined.

**Q3. How do you handle failures and retries for Lambda consuming from SQS?**
Lambda polls SQS and invokes with batches. If the function throws, the whole batch becomes visible again after the visibility timeout → duplicates. Best practices: (1) set visibility timeout ≥ 6× function timeout; (2) enable **`ReportBatchItemFailures`** and return only failed message IDs (partial batch response); (3) configure a **DLQ on the SQS queue** with `maxReceiveCount` (redrive policy); (4) make processing **idempotent** (e.g. Powertools idempotency utility with DynamoDB); (5) use `MaximumConcurrency` on the event source mapping to protect downstreams.

**Q4. Lambda vs containers (ECS/Fargate) — how do you decide?**
Choose Lambda for event-driven, spiky or low-volume workloads, short tasks (<15 min), and when you want zero ops and scale-to-zero. Choose containers for long-running processes, steady high throughput (Lambda becomes more expensive at sustained high utilization), WebSockets/long connections, large memory/GPU needs, heavy frameworks with long init, or when you need portability. Many systems mix both: containers for the core API, Lambda for async glue.

**Q5. How do you connect Lambda to RDS without exhausting connections?**
Each concurrent Lambda environment opens its own connection, so a spike of 1,000 concurrent invocations can exhaust `max_connections`. Use **RDS Proxy** (pools and multiplexes connections, handles failover faster, supports IAM auth/Secrets Manager), initialize the connection outside the handler for reuse, cap function concurrency with reserved concurrency, or use the **Aurora Data API** (HTTP-based, connectionless) or DynamoDB.

**Q6. What are Lambda's main limits that influence architecture?**
15-minute timeout (split long jobs with Step Functions or use Fargate/durable functions), 6 MB sync payload (use S3 presigned URLs for large files), 10 GB memory, 250 MB unzipped deployment (10 GB images), account concurrency (request increase; burst scaling of 1,000 environments every 10 seconds per function), ephemeral environment (no local state guarantees). Mentioning these and the workarounds shows senior awareness.

**Q7. How do you observe and debug Lambda in production?**
Structured JSON logs to CloudWatch Logs (Powertools Logger with correlation IDs), metrics (`Errors`, `Throttles`, `Duration`, `ConcurrentExecutions`, `IteratorAge` for streams), embedded metric format for custom metrics, **X-Ray / OpenTelemetry** tracing across services, CloudWatch Lambda Insights for memory/CPU, and Application Signals for SLOs. Alarm on throttles and iterator age — they're early signals of backpressure.

---

## 4. ECS, Fargate & ECR

**What they are:**
- **ECS (Elastic Container Service):** AWS-native container orchestrator — simpler than Kubernetes, deeply integrated with IAM, ALB, CloudWatch.
- **Fargate:** Serverless compute engine for containers (ECS and EKS) — no EC2 nodes to manage; you pay per vCPU/GB-second per task.
- **ECR (Elastic Container Registry):** Private (and public) OCI image registry with image scanning, lifecycle policies, replication and pull-through cache.

**Key concepts**
- **Task definition:** Blueprint (images, CPU/memory, ports, env vars, secrets, IAM roles, log config).
- **Task:** Running instance of a task definition. **Service:** Keeps N tasks running, integrates with ALB/NLB, handles rolling or blue/green deploys.
- **Launch types / capacity providers:** EC2 (you manage instances, cheaper at scale, GPUs, daemon tasks) vs Fargate / Fargate Spot. **ECS Managed Instances** (2025) — AWS manages the EC2 fleet for you.
- **IAM:** *Task execution role* (ECS agent pulls image, writes logs, fetches secrets) vs *Task role* (app's own AWS permissions).
- **Networking:** `awsvpc` mode gives each task its own ENI & security group.
- **Service Connect / Cloud Map:** Service discovery and service-to-service mesh features.
- **Deployments:** Rolling (min healthy %/max %), circuit breaker with auto rollback, **native blue/green** (added 2025) or CodeDeploy blue/green.

**Uses:** Microservices, APIs, background workers, batch jobs (ECS scheduled tasks), migrating Docker Compose apps.

### Interview Questions

**Q1. ECS vs EKS — when would you choose each?**
ECS: simpler, no control-plane fee, tight AWS integration, smaller learning curve — great for teams that want containers without Kubernetes complexity. EKS: Kubernetes standard API, portability (multi-cloud/on-prem), huge ecosystem (Helm, Argo CD, operators, service meshes, KEDA, Karpenter), needed when the org already has K8s expertise or needs CRDs/operators. Cost: EKS charges per cluster hour plus more ops overhead. Senior answer: choose based on team skills and ecosystem needs, not hype.

**Q2. Task execution role vs task role?**
The **execution role** is used by the ECS agent/Fargate platform: pull from ECR, push logs to CloudWatch, read secrets from Secrets Manager/SSM to inject into env vars. The **task role** is assumed by your application code to call AWS APIs (S3, DynamoDB). Keep them separate for least privilege — the app should never be able to read every secret the platform can.

**Q3. Fargate vs EC2 launch type?**
Fargate: no node management/patching, per-task isolation, right-sized billing, faster to adopt; but higher per-unit cost, no GPU (on ECS), no privileged containers/daemons, limited host-level tuning. EC2: cheaper at high steady utilization (with Savings Plans/Spot), GPUs, custom AMIs, daemon tasks, bin-packing. Common pattern: Fargate for most services, EC2 capacity providers for GPU or very large steady workloads; Fargate Spot for interruption-tolerant tasks.

**Q4. How do you do zero-downtime deployments on ECS?**
Rolling update with `minimumHealthyPercent=100`, `maximumPercent=200` so new tasks start before old ones stop; ALB health checks + `healthCheckGracePeriodSeconds`; **deployment circuit breaker** with rollback; connection draining via target group deregistration delay; graceful SIGTERM handling in the app (`stopTimeout`). For safer releases use blue/green with test listener and traffic shifting (canary/linear) plus CloudWatch alarm-based rollback.

**Q5. How do you secure and optimize container images in ECR?**
Enable **enhanced scanning** (Amazon Inspector) for OS & language package CVEs, use **immutable tags** to prevent overwrites, lifecycle policies to expire old images, cross-region replication for DR and latency, **pull-through cache** for Docker Hub/public images (avoids rate limits), image signing (AWS Signer/Notation or cosign) with admission verification, minimal/distroless base images, multi-stage builds, and KMS encryption.

**Q6. How do ECS tasks get secrets securely?**
Reference Secrets Manager or SSM Parameter Store ARNs in the task definition `secrets` field — the execution role fetches them at task start and injects them as env vars (never put secrets in plain `environment` or images). For rotation without restarts, have the app read from Secrets Manager at runtime with caching (the secrets caching library/Lambda extension).

---

## 5. EKS (Elastic Kubernetes Service)

**What it is:** Managed Kubernetes — AWS runs and scales the control plane (API server, etcd) across 3 AZs; you run workloads on managed node groups, self-managed nodes, Fargate, or **EKS Auto Mode**.

**Purpose:** Run standard Kubernetes on AWS with AWS integrations (IAM, VPC networking, ELB, EBS/EFS).

**Key concepts**
- **Data plane options:**
  - *Managed node groups* — AWS manages ASG lifecycle/updates of EC2 nodes.
  - *Karpenter* — open-source, fast, just-in-time node provisioner; picks optimal instance types per pending pod requirements, consolidates underutilized nodes, handles Spot.
  - *Fargate* — pod-per-microVM, no nodes (no DaemonSets, no GPUs).
  - *EKS Auto Mode* (Dec 2024) — AWS manages nodes, Karpenter-based autoscaling, networking, storage, load balancing add-ons, and OS patching.
- **Networking:** *Amazon VPC CNI* gives pods real VPC IPs (watch IP exhaustion → prefix delegation, secondary CIDRs, IPv6).
- **IAM for pods:** **IRSA** (IAM Roles for Service Accounts via OIDC) and the newer, simpler **EKS Pod Identity**.
- **Access:** EKS **access entries** (API-based) replace the `aws-auth` ConfigMap.
- **Load balancing:** AWS Load Balancer Controller (Ingress → ALB, Service → NLB), Gateway API support.
- **Add-ons:** CoreDNS, kube-proxy, VPC CNI, EBS CSI, EFS CSI, ADOT, GuardDuty agent.
- **Upgrades:** One minor version at a time; standard support ~14 months, then paid extended support.

**Uses:** Large microservice platforms, internal developer platforms, ML/LLM inference on GPUs (vLLM, KServe), batch with Kueue/Volcano, hybrid via EKS Anywhere/Hybrid Nodes.

### Interview Questions

**Q1. IRSA vs EKS Pod Identity?**
Both let pods assume IAM roles without node-level credentials. **IRSA:** the cluster's OIDC provider is registered in IAM; a service account is annotated with a role ARN; the SDK exchanges the projected JWT via `AssumeRoleWithWebIdentity`. Trust policy is per cluster OIDC issuer — painful at scale across many clusters. **Pod Identity:** install the Pod Identity Agent add-on, create an association (namespace + service account → role) via the EKS API; the role trusts the `pods.eks.amazonaws.com` service principal, so the same role can be reused across clusters without editing trust policies, and it supports session tags for ABAC. Prefer Pod Identity for new setups.

**Q2. Why Karpenter over Cluster Autoscaler?**
Cluster Autoscaler scales pre-defined node groups (ASGs) — you must predefine instance types, it's slower, and bin-packing is limited. **Karpenter** watches unschedulable pods and directly launches the best-fit instances (from a flexible set of types, sizes, archs, Spot/On-Demand) in seconds, and actively **consolidates** — replacing or removing underutilized nodes to cut cost. It also handles Spot interruptions and node expiry/drift for patching. Trade-off: must configure disruption budgets carefully so consolidation doesn't churn critical pods (use PDBs, `do-not-disrupt` annotations).

**Q3. Pods are stuck in `Pending` with "insufficient IPs" — what's happening?**
The VPC CNI assigns each pod a VPC IP, and each instance type has an ENI/IP limit; subnets may also be exhausted. Fixes: enable **prefix delegation** (assigns /28 prefixes per ENI slot — many more pods per node), add secondary CIDR (100.64.0.0/10 range) with **custom networking** for pods, use larger subnets, or move to **IPv6** clusters. Also check `max-pods` settings match.

**Q4. How do you perform an EKS version upgrade safely?**
(1) Read deprecation notes; scan manifests for removed APIs (`kubent`, `pluto`, EKS upgrade insights). (2) Upgrade in non-prod first. (3) Upgrade control plane (one minor version at a time). (4) Upgrade add-ons (VPC CNI, CoreDNS, kube-proxy) to compatible versions. (5) Roll nodes — managed node groups update, or Karpenter drift replaces nodes automatically; PDBs protect availability. (6) Alternatively blue/green **clusters** (new cluster, shift traffic via DNS/ALB) for zero-risk major changes. Upgrade regularly to avoid extended support fees.

**Q5. How do you expose services on EKS?**
Install the **AWS Load Balancer Controller**: `Service type: LoadBalancer` → NLB (L4), `Ingress` → ALB (L7 path/host routing, WAF, Cognito/OIDC auth), Gateway API supported too. Use **IP target mode** so traffic goes directly to pod IPs (skips kube-proxy hop). Group multiple Ingresses onto one ALB with `group.name` to save cost. Use ExternalDNS for Route 53 records and cert-manager/ACM for TLS.

**Q6. How do you secure an EKS cluster?**
Private API endpoint (or restricted CIDRs), access entries + least-privilege RBAC, Pod Identity per workload, Pod Security Standards (restricted) via admission (or Kyverno/OPA Gatekeeper), NetworkPolicies (VPC CNI network policy or Cilium), encrypt Secrets with KMS envelope encryption, use External Secrets Operator/Secrets Store CSI with Secrets Manager, ECR image scanning and signature verification, GuardDuty EKS Runtime Monitoring, control-plane audit logs to CloudWatch, Bottlerocket nodes, and IMDSv2 hop limit 1 to block pods from stealing node credentials.

**Q7. What is EKS Auto Mode and when would you use it?**
Auto Mode offloads node management to AWS: it provisions EC2 nodes (Bottlerocket-based) using built-in Karpenter, manages core add-ons (VPC CNI, CoreDNS, kube-proxy, EBS CSI, load balancer controller), patches and rotates nodes (max ~21-day node lifetime), and scales to fit pods. Use it when you want Kubernetes APIs without operating the data plane. Trade-offs: an extra management fee per instance, less low-level customization (no SSH, limited AMI customization).

---

## 6. App Runner & Elastic Beanstalk

**What they are:**
- **App Runner:** Fully managed service that builds and runs containerized web apps/APIs from source or an image — handles TLS, load balancing, autoscaling (incl. based on concurrent requests), and deployments. Minimal config.
- **Elastic Beanstalk:** Older PaaS that provisions EC2, ASG, ELB, and monitoring from your code (Java, .NET, Node, Python, PHP, Ruby, Go, Docker). You keep access to the underlying resources.

**Purpose:** Fastest path from code to running web app without designing infrastructure.

**Uses:** Internal tools, prototypes, small/medium web APIs, lift-and-shift of traditional web apps (Beanstalk).

### Interview Questions

**Q1. When would you pick App Runner over ECS Fargate?**
When the team wants a Heroku-like experience: single HTTP service, auto TLS/domain, request-based autoscaling, no need to configure VPCs, ALBs or task definitions. Choose ECS Fargate when you need fine control: sidecars, custom networking, non-HTTP workloads, background workers, service mesh, or complex deployments. (Note: AWS has signaled reduced investment in App Runner for new customers — check current status; ECS Express-style options are the suggested direction.)

**Q2. What deployment policies does Elastic Beanstalk support?**
*All at once* (fast, downtime), *Rolling* (batches, reduced capacity), *Rolling with additional batch* (maintains full capacity), *Immutable* (new ASG with new instances, swap when healthy — safest in-place), *Traffic splitting* (canary), and *Blue/Green* via environment **CNAME swap**.

**Q3. What are the limitations of Beanstalk for a senior-level production platform?**
Limited IaC fit (configuration via `.ebextensions` is clunky compared to CDK/Terraform), slower deployments, platform version lock-in, harder to implement microservices/mesh, and less community momentum. It's fine for monoliths; for new platforms prefer ECS/EKS with IaC.

**Q4. How does App Runner connect to private resources like RDS?**
Via a **VPC connector** — outbound traffic from the service is routed into specified VPC subnets/security groups so it can reach private RDS/ElastiCache. Inbound can be made private using a VPC interface endpoint.

**Q5. How does Elastic Beanstalk blue/green deployment work and what's the catch?**
You clone the environment (green), deploy the new version, test, then swap CNAMEs so DNS points to green. Catch: DNS TTL/caching means some clients hit the old env for a while; and if the RDS instance is *inside* the Beanstalk environment, it gets tied to the environment lifecycle — always create RDS separately and pass connection info in.

---

## 7. VPC (Virtual Private Cloud)

**What it is:** A logically isolated virtual network in an AWS region where you define IP ranges (CIDR), subnets, routing, gateways and firewalls.

**Purpose:** Network isolation and control over how resources communicate with each other, the internet, and on-prem.

**Key concepts**
- **CIDR:** e.g. `10.0.0.0/16` (65,536 IPs). AWS reserves 5 IPs per subnet (first 4 + last). Plan non-overlapping CIDRs across VPCs/on-prem (essential for peering/TGW later).
- **Subnets:** AZ-scoped. *Public* = route `0.0.0.0/0 → Internet Gateway`. *Private* = route to NAT Gateway for outbound only. *Isolated* = no internet route (DBs).
- **Internet Gateway (IGW):** Horizontally scaled, HA gateway for public internet; performs 1:1 NAT for public IPs.
- **NAT Gateway:** Managed outbound-only internet for private subnets; AZ-scoped (deploy one per AZ for HA); charged per hour + per GB (a frequent cost surprise). *Regional NAT Gateway* mode (2025) can span AZs automatically.
- **Egress-only IGW:** Outbound-only for IPv6.
- **Route tables:** Longest prefix match wins.
- **Security Groups vs NACLs:**

| | Security Group | NACL |
|---|---|---|
| Level | ENI/instance | Subnet |
| State | **Stateful** (return traffic auto-allowed) | **Stateless** (must allow return/ephemeral ports) |
| Rules | Allow only | Allow and Deny |
| Evaluation | All rules evaluated | Numbered order, first match wins |
| Reference | Can reference other SGs | CIDRs only |

- **VPC Flow Logs:** Capture IP traffic metadata (to CloudWatch/S3) for troubleshooting and security.
- **Elastic IP:** Static public IPv4 (all public IPv4 addresses are now billed hourly since 2024).

**Uses:** Every production workload — 3-tier architecture (public ALB, private app, isolated DB subnets across 3 AZs).

### Interview Questions

**Q1. Design a VPC for a highly available 3-tier web app.**
`/16` VPC across 3 AZs. Per AZ: public subnet (ALB, NAT Gateway), private app subnet (ECS/EC2/EKS), isolated data subnet (RDS/ElastiCache, no internet route). One NAT Gateway per AZ with per-AZ route tables (avoid cross-AZ dependency). Security groups chained: ALB-SG allows 443 from internet; App-SG allows app port only from ALB-SG; DB-SG allows 5432 only from App-SG. Gateway endpoints for S3/DynamoDB, interface endpoints for ECR/Secrets Manager/STS/CloudWatch to keep traffic private and cut NAT cost. Flow logs enabled. Leave CIDR room for growth/EKS pods.

**Q2. Security Group vs NACL — which do you use when?**
SGs are the primary control — stateful, instance-level, allow-only, and can reference other SGs (identity-like rules that survive IP changes). NACLs are a stateless subnet-level guardrail — useful for explicit **deny** (block a malicious CIDR) or coarse subnet isolation. Since NACLs are stateless, forgetting ephemeral return ports (1024–65535) is a classic outage cause.

**Q3. Private instances need to download packages but must not be reachable from the internet. How?**
Route `0.0.0.0/0` from the private subnet to a NAT Gateway in a public subnet. NAT allows outbound-initiated connections and their responses only. For AWS services, prefer VPC endpoints (no NAT cost). For tighter control, route egress through AWS Network Firewall or a proxy with domain allow-lists.

**Q4. Our NAT Gateway bill is huge. How do you reduce it?**
Identify top talkers with VPC Flow Logs / Cost Explorer (`NatGateway-Bytes`). Common causes: pulling images from ECR, S3 traffic, CloudWatch logs going through NAT. Fixes: **S3/DynamoDB gateway endpoints** (free), interface endpoints for ECR, logs, STS, etc. (cheaper per GB than NAT for high volume), ECR pull-through cache, keep traffic in-AZ (cross-AZ to NAT adds data transfer charges), and avoid NAT for IPv6 (egress-only IGW).

**Q5. Two VPCs can't communicate after peering. What do you check?**
(1) Peering connection accepted; (2) **route tables on both sides** have routes to the peer CIDR via the `pcx-`; (3) SGs/NACLs allow the traffic (SG can reference peer SG in same region); (4) no **overlapping CIDRs**; (5) peering isn't transitive — traffic can't go A→B→C or use B's IGW/NAT/VPN; (6) DNS resolution option enabled if using private hostnames.

**Q6. How do you troubleshoot connectivity in a VPC?**
Use **VPC Reachability Analyzer** (static path analysis between source/destination, tells you which component blocks), **VPC Flow Logs** (ACCEPT/REJECT records — REJECT on return traffic often indicates NACL), **Network Access Analyzer** for unintended access paths, plus traditional tools (curl, telnet, dig) via SSM Session Manager.

---

## 8. VPC Connectivity: Peering, Transit Gateway, PrivateLink, Endpoints

**What they are:**
- **VPC Peering:** 1:1 private connection between two VPCs (same/different accounts/regions). Non-transitive, no overlapping CIDRs, no bandwidth bottleneck, no hourly charge.
- **Transit Gateway (TGW):** Regional hub router connecting thousands of VPCs, VPNs, and Direct Connect. Supports route tables for segmentation, inter-region peering, multicast. Charged per attachment hour + per GB.
- **AWS PrivateLink:** Expose a service (behind an NLB) to other VPCs/accounts via **interface endpoints** — consumers see a private ENI in their own subnet. One-directional, works with overlapping CIDRs, no routing between networks.
- **VPC Endpoints:**
  - *Gateway endpoints* — S3 and DynamoDB only; route table entry; free.
  - *Interface endpoints* — PrivateLink ENIs for most AWS services (and SaaS); per-hour + per-GB charge; supports endpoint policies & private DNS.
- **VPC Lattice:** Application-layer service networking across VPCs/accounts (service discovery, auth policies, routing) without managing network connectivity.
- **Cloud WAN:** Global managed WAN using policy-defined segments across regions.

### Interview Questions

**Q1. You have 50 VPCs that need to communicate. Peering or Transit Gateway?**
Full-mesh peering needs n(n-1)/2 = 1,225 connections and route entries — unmanageable. Use **Transit Gateway** as a hub: each VPC attaches once; segment with TGW route tables (e.g. prod can't reach dev, all can reach shared services). Trade-offs: TGW adds per-GB processing cost and a small latency hop; for a couple of very high-bandwidth VPC pairs, peering can be added alongside TGW for cost.

**Q2. When would you use PrivateLink instead of peering/TGW?**
When you want to expose **one service**, not a whole network: SaaS providers serving customers, central platform teams exposing an internal API to many accounts, or when **CIDRs overlap**. Consumers only get access to that endpoint (least privilege), and the provider never sees the consumer's network. Requires NLB (or ALB-backed NLB / Lattice) on the provider side.

**Q3. Gateway endpoint vs interface endpoint for S3?**
Gateway endpoint: free, route-table based, works only from inside the VPC (not from on-prem or peered VPCs). Interface endpoint for S3: costs per hour/GB, but gets private IPs reachable from on-prem via DX/VPN and peered VPCs. Use gateway endpoints for in-VPC workloads; interface endpoints when on-prem/hybrid access needs a private path. Apply **endpoint policies** + bucket policies with `aws:sourceVpce` to restrict data exfiltration.

**Q4. How do you design centralized egress and inspection for a multi-account org?**
Hub-and-spoke with Transit Gateway: spoke VPCs have no IGW/NAT; default route goes to TGW → **egress/inspection VPC** containing AWS Network Firewall (or GWLB + third-party appliances) and NAT Gateways. Use TGW appliance mode to keep flows symmetric. Centralized interface endpoints in a shared services VPC with Route 53 Resolver/private hosted zones shared via RAM. Trade-off: central egress reduces NAT/endpoint duplication but adds TGW data processing charges.

**Q5. What is VPC Lattice and when is it a better choice?**
Lattice is a managed application-networking layer: you define services and a service network; any associated VPC/account can call services by DNS name with **IAM-based auth policies**, weighted routing, and observability — without peering/TGW or overlapping-CIDR concerns. It fits service-to-service communication across many accounts (microservices on EC2, ECS, EKS, Lambda) and is a simpler alternative to running a service mesh for cross-VPC traffic.

---

## 9. Elastic Load Balancing (ALB, NLB, GWLB)

**What it is:** Managed load balancers that distribute traffic across targets (EC2, IPs, containers, Lambda) in multiple AZs with health checks.

| | ALB | NLB | GWLB |
|---|---|---|---|
| Layer | 7 (HTTP/HTTPS, gRPC, WebSocket) | 4 (TCP/UDP/TLS) | 3 (IP packets, GENEVE) |
| Routing | Host/path/header/query/method rules, weighted target groups | Port-based | To virtual appliances |
| Static IP | No (use Global Accelerator) | Yes, one EIP per AZ | — |
| Performance | Good, scales gradually | Millions of RPS, ultra-low latency | — |
| Features | WAF, OIDC/Cognito auth, Lambda targets, sticky sessions, mTLS | Preserves client IP, TLS passthrough/termination, PrivateLink | Firewalls, IDS/IPS inline |

**Key concepts:** Listeners, rules, target groups, health checks, cross-zone load balancing (ALB on by default; NLB off by default), deregistration delay (connection draining), sticky sessions (duration-based or app cookie), slow start, ALB **mTLS** (2023), **TLS via ACM** certificates, SNI for multiple certs.

### Interview Questions

**Q1. ALB vs NLB — how do you choose?**
ALB for HTTP microservices: path/host routing, WAF, authentication offload, gRPC, header-based canaries, Lambda targets. NLB for non-HTTP protocols (TCP/UDP — gaming, IoT, MQTT, databases), extreme throughput/low latency, **static IPs** for client allow-listing, preserving source IP, or as the front of a **PrivateLink** service. Sometimes both: NLB (static IP) → ALB target.

**Q2. How would you implement canary releases with an ALB?**
Use **weighted target groups** on a listener rule: e.g. 95% to `v1-tg`, 5% to `v2-tg`; increase gradually while watching CloudWatch metrics (5xx, latency) and roll back by setting weight to 0. Enable target group stickiness if users must stay on one version. CodeDeploy (ECS blue/green) or Argo Rollouts (EKS) can automate weight shifting with alarm-based rollback.

**Q3. Users get logged out randomly behind an ALB. Why?**
Sessions are stored in the instance memory and requests are spread across instances. Short-term fix: sticky sessions. Proper fix: make the app stateless — store sessions in ElastiCache/DynamoDB or use stateless JWTs — so scale-in, deployments and failures don't lose sessions.

**Q4. What is cross-zone load balancing and why does it matter?**
With cross-zone enabled, each LB node distributes traffic evenly across all targets in all AZs. Without it, each AZ's node only sends to its own AZ's targets — uneven if AZs have different target counts. ALB has it always on (no charge); NLB/GWLB default off and incur inter-AZ data charges when enabled. Disabling cross-zone is sometimes used intentionally for AZ isolation/static stability.

**Q5. How does connection draining work, and why do clients see 502s during deploys?**
On deregistration, the target enters `draining` for the deregistration delay (default 300 s) — no new requests, in-flight ones complete. 502s during deploys usually mean: the app was killed before draining completed (container stop timeout shorter than delay), keep-alive timeout of the app lower than ALB idle timeout (60 s) causing the app to close connections ALB still reuses — set app keep-alive **greater** than ALB idle timeout — or health check grace period too short.

**Q6. Why can't you give an ALB a static IP, and how do you work around it?**
ALB nodes scale and change IPs, so you must use its DNS name. Workarounds: put **Global Accelerator** in front (2 static anycast IPs) or an NLB with EIPs forwarding to ALB-type target group.

---

## 10. Route 53

**What it is:** Highly available (100% SLA) managed DNS service + domain registration + health checking.

**Key concepts**
- **Hosted zones:** Public (internet) and private (associated with VPCs).
- **Records:** A, AAAA, CNAME, MX, TXT, NS, SOA, CAA, and **Alias** (AWS extension — points to AWS resources like ALB, CloudFront, S3 website; works at zone apex; free queries).
- **Routing policies:** Simple, **Weighted**, **Latency-based**, **Failover** (active-passive), **Geolocation**, **Geoproximity** (with bias, via Traffic Flow), **Multivalue answer**, **IP-based** (CIDR).
- **Health checks:** Endpoint checks, calculated (combine children), CloudWatch alarm-based.
- **Route 53 Resolver:** Inbound/outbound endpoints for hybrid DNS (on-prem ↔ VPC), Resolver rules, DNS Firewall.
- **Application Recovery Controller (ARC):** Routing controls and readiness checks for reliable regional failover; **zonal shift** to move traffic away from an impaired AZ.
- **TTL** controls caching and failover speed.

### Interview Questions

**Q1. CNAME vs Alias record?**
CNAME maps a name to another name; not allowed at the zone apex (`example.com`) and queries are charged. **Alias** is Route 53-specific: maps to AWS resource DNS names (ALB, CloudFront, API Gateway, S3 website, another record), works at the apex, free for AWS targets, and automatically tracks the target's IP changes.

**Q2. How would you implement active-passive multi-region failover?**
Failover routing: primary record → us-east-1 ALB with a health check; secondary → us-west-2 ALB. Low TTL (e.g. 60 s). Health checks should test a deep endpoint representing real health. Data layer must already be replicated (Aurora Global Database, DynamoDB Global Tables). For more control and to avoid relying on control-plane operations during an outage, use **Route 53 ARC routing controls** (data-plane operations, highly available) to flip traffic deliberately.

**Q3. Latency-based vs geolocation routing?**
Latency-based routes the user to the region with the lowest measured network latency — best for performance. Geolocation routes based on the user's country/continent — used for compliance, data residency, content localization or licensing. Geolocation needs a default record for unmatched locations.

**Q4. Why might failover not happen quickly even though the health check failed?**
Client/resolver caching beyond TTL (some resolvers ignore TTL), long TTLs configured, health check thresholds/intervals (default 30 s × 3 failures), JVM DNS caching (`networkaddress.cache.ttl`), or connection pooling keeping old connections. Mitigate with short TTLs, fast health checks (10 s), clients that re-resolve, and Global Accelerator (anycast IP failover not dependent on DNS caching).

**Q5. How do you resolve on-prem hostnames from a VPC and vice versa?**
**Route 53 Resolver endpoints:** *Outbound endpoint* + forwarding rules (e.g. `corp.example.com` → on-prem DNS IPs) for VPC → on-prem; *Inbound endpoint* gives IPs on-prem DNS servers can forward to, for on-prem → private hosted zones. Share resolver rules across accounts with RAM.

---

## 11. CloudFront

**What it is:** AWS's global CDN with 600+ points of presence — caches content at edge locations close to users, and accelerates dynamic content over the AWS backbone.

**Key concepts**
- **Origins:** S3, ALB, EC2, API Gateway, any HTTP server, MediaPackage; origin groups for failover.
- **Cache behaviors:** Path patterns with separate cache/origin request policies.
- **Cache key & TTLs:** Cache policies define which headers/cookies/query strings are part of the cache key — fewer = higher hit ratio.
- **Origin Access Control (OAC):** Lets CloudFront access a private S3 bucket (replaces legacy OAI) — bucket not public.
- **Signed URLs / Signed Cookies:** Restrict access to paid/private content.
- **Edge compute:** *CloudFront Functions* (JS, sub-ms, viewer request/response — headers, redirects, URL rewrites) vs *Lambda@Edge* (Node/Python, more power, network access, origin events).
- **Origin Shield:** Additional regional caching layer to reduce origin load.
- **Security:** AWS WAF, Shield Standard included, field-level encryption, geo-restriction, TLS via ACM (certificate must be in us-east-1).
- **Invalidations:** Purge cached objects (prefer versioned file names).
- **VPC origins:** Serve from private ALB/EC2 without exposing them publicly.

### Interview Questions

**Q1. How do you serve a static SPA securely from S3 with CloudFront?**
Private S3 bucket (Block Public Access on), CloudFront distribution with **OAC** and a bucket policy allowing only that distribution (`AWS:SourceArn`), ACM certificate in us-east-1, HTTPS redirect, default root object `index.html`, custom error response mapping 403/404 → `/index.html` (200) for client-side routing, long TTLs for hashed assets and short/no-cache for `index.html`, security headers via response headers policy, WAF attached.

**Q2. Cache hit ratio is low. How do you improve it?**
Reduce the cache key — only forward headers/cookies/query strings the origin actually varies on (use origin request policy for things the origin needs but shouldn't vary the cache). Normalize query strings/headers with CloudFront Functions, enable compression, use longer TTLs + versioned filenames, enable **Origin Shield**, and avoid `Cache-Control: no-cache` from origin unless needed.

**Q3. CloudFront Functions vs Lambda@Edge?**
CloudFront Functions: JavaScript, run at all edge locations, sub-millisecond, very cheap, viewer events only, no network/body access — use for header manipulation, redirects, URL normalization, simple auth token validation (with KeyValueStore). Lambda@Edge: runs at regional edge caches, supports origin request/response events, network calls, longer execution, larger packages — use for dynamic origin selection, image transformation, complex auth.

**Q4. How do you prevent users from bypassing CloudFront and hitting the origin directly?**
S3: OAC + bucket policy. ALB: use **VPC origins** (ALB stays private), or restrict ALB SG to the CloudFront **managed prefix list** + require a secret custom header injected by CloudFront and validated by an ALB rule/WAF.

**Q5. How do you deploy a new frontend version without serving stale files?**
Use content-hashed filenames for JS/CSS (immutable, long TTL), and short TTL or `no-cache` for `index.html`. Then only `index.html` needs to be refreshed — invalidate `/index.html` (invalidations are eventually consistent across edges and have a cost beyond the free tier), rather than `/*` on every deploy.

---

## 12. Direct Connect, Site-to-Site VPN, Global Accelerator

**What they are:**
- **Site-to-Site VPN:** IPsec tunnels over the internet between on-prem (customer gateway) and AWS (virtual private gateway or TGW). Two tunnels for redundancy; ~1.25 Gbps per tunnel; quick to set up.
- **Direct Connect (DX):** Dedicated private fiber connection from on-prem/colo to AWS (1/10/100/400 Gbps dedicated or hosted). Consistent latency, lower data-transfer-out cost, not encrypted by default (use MACsec or VPN over DX). Takes weeks to provision. **DX Gateway** connects to multiple regions/VPCs.
- **Global Accelerator:** Two static anycast IPs that route user traffic onto the AWS global backbone at the nearest edge, then to regional endpoints (ALB, NLB, EC2, EIP). Fast failover (seconds), not DNS-dependent; works for TCP/UDP. No caching (unlike CloudFront).

### Interview Questions

**Q1. Direct Connect vs VPN — when to use which?**
VPN: fast to set up, cheap, encrypted, but variable internet latency and limited throughput — good for small offices, PoCs, and backup. DX: consistent latency, high bandwidth, cheaper egress for large data volumes — good for hybrid production, data migration, latency-sensitive apps. Common pattern: DX primary + VPN backup.

**Q2. How do you make Direct Connect highly available?**
AWS's "maximum resiliency" model: two DX locations, two connections per location (4 connections, separate devices). At minimum, two connections at different DX locations, or one DX + Site-to-Site VPN as failover with BGP preferring DX. Use BFD for fast failure detection.

**Q3. Is Direct Connect encrypted? How do you encrypt it?**
No, DX is private but not encrypted. Options: **MACsec** (Layer 2 encryption on supported 10/100/400G dedicated connections) or run an IPsec VPN over the DX public/transit VIF. Plus application-level TLS.

**Q4. Global Accelerator vs CloudFront?**
CloudFront caches HTTP(S) content at the edge — best for static and cacheable dynamic web content. Global Accelerator doesn't cache; it accelerates any TCP/UDP traffic over the AWS backbone and provides static anycast IPs with sub-minute regional failover — best for gaming, VoIP, IoT, non-HTTP APIs, or when clients need fixed IPs for allow-listing.

**Q5. What are DX virtual interfaces (VIFs)?**
*Private VIF*: access a VPC via VGW or a DX Gateway. *Transit VIF*: connect to Transit Gateway via DX Gateway (multi-VPC/multi-region). *Public VIF*: access AWS public endpoints (S3, DynamoDB public IPs) over DX.

---

## 13. S3 (Simple Storage Service)

**What it is:** Object storage with 11 nines (99.999999999%) durability, virtually unlimited capacity, accessed via HTTP API. Objects (up to 5 TB historically; raised to 50 TB in late 2025) live in buckets with keys.

**Purpose:** Durable, cheap storage for files, backups, data lakes, static websites, logs, ML datasets, and increasingly as the foundation for analytics (Iceberg tables) and AI (vector storage).

**Key concepts**
- **Consistency:** Strong read-after-write consistency for all operations (since Dec 2020).
- **Storage classes:** Standard → Intelligent-Tiering (auto-moves between tiers, no retrieval fees) → Standard-IA / One Zone-IA (infrequent, retrieval fee, 30-day min) → Glacier Instant Retrieval (ms access) → Glacier Flexible Retrieval (minutes–hours) → Glacier Deep Archive (12–48 h, cheapest). **S3 Express One Zone** — single-AZ, single-digit ms latency directory buckets for high-performance workloads.
- **Lifecycle policies:** Transition/expire objects and old versions, abort incomplete multipart uploads.
- **Versioning**, **MFA Delete**, **Object Lock** (WORM — Governance/Compliance mode, legal hold).
- **Replication:** CRR (cross-region) / SRR (same-region); requires versioning; Replication Time Control (15-min SLA); batch replication for existing objects.
- **Encryption:** SSE-S3 (default), SSE-KMS (audit + key control; use S3 Bucket Keys to cut KMS costs), DSSE-KMS, SSE-C (customer-provided key), client-side.
- **Access control:** IAM policies, bucket policies, **Block Public Access**, Access Points, ACLs (disabled by default — "bucket owner enforced").
- **Performance:** ≥3,500 PUT/COPY/POST/DELETE and 5,500 GET/HEAD per second **per prefix**; multipart upload (recommended >100 MB, required >5 GB), byte-range fetches, Transfer Acceleration.
- **Presigned URLs:** Time-limited access to upload/download without AWS credentials.
- **Event notifications:** To SNS, SQS, Lambda, or EventBridge.
- **Newer:** **S3 Tables** (managed Apache Iceberg tables with automatic compaction), **S3 Metadata** (queryable object metadata), **S3 Vectors** (native vector storage and similarity search at low cost — usable as a Bedrock Knowledge Base vector store).

**Uses:** Static website hosting, data lakes, backups/DR, media storage, log archives, ML training data, RAG document stores, vector indexes.

### Interview Questions

**Q1. How would you let users upload large files directly to S3 from a browser securely?**
Backend authenticates the user and generates a **presigned PUT URL** (or presigned POST with conditions like content-type and max size), short expiry (minutes), scoped key prefix (`uploads/{userId}/...`). For large files use **multipart upload** with presigned URLs per part and parallel uploads. Client uploads directly to S3 (no app server bandwidth). S3 event → Lambda/EventBridge for validation, virus scanning, thumbnailing. Configure CORS on the bucket. Never give the client IAM credentials.

**Q2. How do you choose a storage class and reduce S3 costs?**
Analyze with **S3 Storage Lens** and Storage Class Analysis. Use **Intelligent-Tiering** when access patterns are unknown (watch the per-object monitoring fee for many tiny objects — objects <128 KB aren't tiered). Lifecycle rules: e.g. logs → Standard-IA at 30 days → Glacier at 90 → expire at 365. Expire noncurrent versions, abort incomplete multipart uploads, compress/aggregate small objects, and use gateway endpoints to avoid NAT charges.

**Q3. How do you protect S3 data against accidental deletion or ransomware?**
Versioning + MFA Delete, **Object Lock** in Compliance mode for immutable backups, replication to a separate account (with different credentials) and region, AWS Backup for S3 with vault lock, least-privilege policies denying `s3:DeleteObject*`, SCPs preventing disabling of versioning/Block Public Access, and CloudTrail data events + GuardDuty S3 protection to detect unusual access.

**Q4. A bucket is getting 503 Slow Down errors. Why and how do you fix it?**
Request rate exceeds per-prefix limits (3,500 writes / 5,500 reads per second per prefix), or S3 is still scaling partitions for a sudden spike. Fix: spread keys across multiple prefixes (e.g. hash-based prefixes, date + shard), implement exponential backoff with jitter (SDKs do this), ramp traffic gradually, cache hot objects with CloudFront, or use S3 Express One Zone for very high request rates.

**Q5. SSE-S3 vs SSE-KMS vs SSE-C?**
SSE-S3: AWS-managed keys, zero effort, default. SSE-KMS: keys in KMS — gives key policy control, separation of duties (a reader needs both S3 and `kms:Decrypt` permission), CloudTrail audit of key use, and key rotation; adds KMS request costs/throttling (mitigate with **S3 Bucket Keys**). SSE-C: you supply the key on each request; AWS doesn't store it — you manage key storage. Use SSE-KMS with customer-managed keys for regulated data.

**Q6. How do you make a bucket accessible only from a specific VPC?**
Bucket policy with `Deny` unless `aws:SourceVpce` equals your VPC endpoint ID (or `aws:SourceVpc`), combined with a VPC gateway endpoint and an endpoint policy restricting which buckets can be reached (prevents data exfiltration to attacker-owned buckets). Be careful: a blanket deny can lock out admins/console — add exceptions for break-glass roles.

**Q7. What is S3 Vectors and why does it matter for GenAI?**
S3 Vectors adds **vector buckets and vector indexes** natively in S3 — you store embeddings with metadata and run similarity queries via API, with sub-second query latency at much lower cost than keeping all vectors in a memory-based vector DB. It integrates as a vector store for Bedrock Knowledge Bases and can be tiered with OpenSearch for hot, low-latency queries. Good fit for large, infrequently queried RAG corpora where cost matters more than millisecond latency.

---

## 14. EBS, EFS, FSx

**What they are:**
- **EBS (Elastic Block Store):** Network-attached block volumes for a single EC2 instance (Multi-Attach for io1/io2 in same AZ). AZ-scoped. Types: **gp3** (general SSD — baseline 3,000 IOPS/125 MB/s, independently provisionable IOPS & throughput, ~20% cheaper than gp2), **io2 Block Express** (up to 256k IOPS, 99.999% durability, databases), **st1** (throughput HDD — big data/logs), **sc1** (cold HDD). Snapshots are incremental and stored in S3; can be copied cross-region; Fast Snapshot Restore avoids lazy-loading latency.
- **EFS (Elastic File System):** Managed NFS file system, multi-AZ, shared across thousands of instances/containers/Lambda, grows automatically. Performance modes and throughput modes (Elastic throughput recommended). Storage classes Standard/IA/Archive with lifecycle.
- **FSx:** Managed third-party file systems: **FSx for Windows File Server** (SMB, AD), **FSx for Lustre** (HPC/ML, S3-linked, hundreds of GB/s), **FSx for NetApp ONTAP** (multi-protocol, snapshots, dedupe), **FSx for OpenZFS**.

### Interview Questions

**Q1. EBS vs EFS vs S3 — how do you choose?**
EBS: block storage for one instance — OS disks, databases needing low-latency random I/O. EFS: shared POSIX file system across many instances/AZs — CMS uploads, shared configs, home directories, ML notebooks, Lambda needing a file system. S3: object storage via API — unlimited, cheapest, highly durable, not a mountable file system (Mountpoint for S3 is read-heavy/sequential). Pick by access pattern (block vs file vs object) and sharing needs.

**Q2. Why migrate gp2 to gp3?**
gp2 IOPS scale with size (3 IOPS/GB) so people over-provision capacity to get IOPS. gp3 decouples IOPS and throughput from size, gives a 3,000 IOPS baseline at any size, and is ~20% cheaper per GB. Migration is online via Elastic Volumes (no downtime).

**Q3. How do you back up EBS volumes consistently and do cross-region DR?**
Use **AWS Backup** or Data Lifecycle Manager for scheduled, incremental snapshots. For application consistency: freeze the file system or use multi-volume crash-consistent snapshots, or VSS on Windows. Copy snapshots cross-region/cross-account (Backup copy rules), encrypt with KMS (re-encrypt with destination key). Restored volumes lazy-load blocks from S3 — use **Fast Snapshot Restore** or pre-warm for performance-sensitive restores.

**Q4. When would you choose FSx for Lustre?**
For HPC and ML training needing very high throughput/low latency shared storage — e.g. hundreds of GPU nodes reading a training dataset. It links to an S3 bucket (lazy loads objects, writes back), so S3 remains the durable data lake while Lustre provides the fast scratch layer.

**Q5. EFS performance is slow for many small files. What can you do?**
Use **Elastic throughput** mode, parallelize I/O across threads/clients, increase I/O size, cache locally where possible, avoid metadata-heavy operations (e.g. `ls` on huge directories, npm installs on EFS), consider FSx for NetApp ONTAP/OpenZFS for low-latency small-file workloads, or bundle small files into archives in S3.

---

## 15. RDS & Aurora (incl. Aurora DSQL, RDS Proxy)

**What they are:**
- **RDS:** Managed relational databases — PostgreSQL, MySQL, MariaDB, Oracle, SQL Server, Db2. AWS handles provisioning, patching, backups, failover.
- **Aurora:** AWS-built MySQL/PostgreSQL-compatible engine with a distributed storage layer: data replicated **6 ways across 3 AZs**, auto-grows to 128 TB (256 TB on newer versions), up to 15 low-lag read replicas sharing storage, fast failover (~30 s or less).
- **Aurora Serverless v2:** Scales capacity in fine-grained ACUs in seconds; can scale to zero (auto-pause, 2024).
- **Aurora Global Database:** Storage-level cross-region replication (<1 s typical lag), up to 5+ secondary regions, managed switchover/failover (RPO ~1 s, RTO ~1 min).
- **Aurora DSQL (GA 2025):** Serverless, distributed SQL (PostgreSQL-compatible) with **active-active multi-region** strong consistency, optimistic concurrency control, virtually unlimited scale, no instances to manage.
- **RDS Proxy:** Managed connection pooler — reduces connection storms (Lambda), speeds failover by up to 66%, enforces IAM auth.
- **Aurora Limitless Database:** Horizontal write scaling via sharding (PostgreSQL).

**Key concepts:** Multi-AZ (synchronous standby, HA) vs Read Replicas (asynchronous, read scaling), automated backups + PITR (up to 35 days), manual snapshots, Blue/Green Deployments for upgrades, Performance Insights / Database Insights, parameter groups, IAM DB authentication, encryption at rest (must be enabled at creation).

### Interview Questions

**Q1. Multi-AZ vs Read Replicas?**
**Multi-AZ** is for **availability**: synchronous replication to a standby in another AZ; automatic failover (DNS endpoint flips) on failure, patching or AZ outage; the classic standby isn't readable (Multi-AZ *DB cluster* deployments have 2 readable standbys). **Read replicas** are for **read scalability**: asynchronous replication (replica lag), readable, can be cross-region, can be promoted manually (DR). In Aurora, replicas serve both roles — any replica can be promoted on failover.

**Q2. Why is Aurora faster/more resilient than standard RDS?**
Aurora separates compute from a distributed, log-structured storage layer. The writer sends only redo log records to storage (not full pages), using a 4-of-6 write quorum and 3-of-6 read quorum across 3 AZs — tolerates loss of an entire AZ plus one more node for reads. Replicas share the same storage volume, so replica lag is typically <100 ms and failover doesn't need data copy. Crash recovery is near-instant because redo is applied continuously by storage.

**Q3. How do you scale a relational database on AWS?**
Vertical: larger instance (quick, limited). Read scaling: read replicas + reader endpoint, cache (ElastiCache) for hot reads, CQRS read models. Connection scaling: RDS Proxy/PgBouncer. Write scaling: optimize queries/indexes, partition tables, batch writes, move hot workloads to DynamoDB, functional partitioning (split by service), sharding (application-level or Aurora Limitless), or Aurora DSQL for distributed writes. Use Performance Insights to find top waits before throwing hardware at it.

**Q4. How do you upgrade a major PostgreSQL version with minimal downtime?**
Use **RDS Blue/Green Deployments**: creates a green copy synced via logical replication, you upgrade/test green, then switchover (typically under a minute), with guardrails that block switchover if replication is unhealthy. Test extensions/queries in staging first and take a snapshot. Alternative: AWS DMS or native logical replication to a new cluster and cut over at the app layer.

**Q5. What is RDS Proxy and when is it needed?**
A fully managed connection pool in front of RDS/Aurora. Needed when many short-lived clients (Lambda, autoscaling containers) open connections, causing `too many connections` and memory pressure. It multiplexes client connections over fewer DB connections, holds connections during failover (reducing failover impact), and supports IAM auth with Secrets Manager. Caveat: "pinning" — session-level state (temp tables, SET statements, prepared statements in some cases) pins a client to a connection, reducing multiplexing.

**Q6. How do you design for multi-region with relational data?**
Options: (1) **Aurora Global Database** — single writer region, sub-second replication, read locally in secondary regions; write forwarding for occasional writes from secondaries; managed failover for DR (RPO ~1 s). (2) **Aurora DSQL** — active-active writes in multiple regions with strong consistency, at the cost of PostgreSQL feature subset and OCC (transactions may need retries on conflict). (3) Application-level partitioning by region (users homed to a region). Choose based on RPO/RTO, write-locality needs and consistency requirements.

**Q7. RDS storage is filling up — what do you do?**
Enable **storage autoscaling** (with a max), investigate growth (bloat from dead tuples → tune autovacuum; large logs/temp files; unbounded audit tables), archive cold data to S3 (export, partition drop), and alarm on `FreeStorageSpace`. Note that RDS storage can't be shrunk in place — you'd need a migration.

---

## 16. DynamoDB

**What it is:** Fully managed, serverless key-value and document NoSQL database delivering single-digit millisecond latency at any scale. Multi-AZ by default.

**Purpose:** High-scale, predictable-latency workloads with known access patterns: user profiles, sessions, carts, gaming state, IoT, metadata, idempotency keys, leaderboards.

**Key concepts**
- **Primary key:** *Partition key* (hash) or *Partition key + Sort key* (composite). Items up to **400 KB**.
- **Partitions:** Data distributed by hash of partition key; each partition supports ~**3,000 RCU / 1,000 WCU** and ~10 GB. **Adaptive capacity** shifts throughput to hot partitions, but a single hot key is still limited.
- **Capacity modes:** *On-demand* (pay per request, instant scaling — default recommendation now, prices cut ~50% in late 2024) vs *Provisioned* (with auto scaling, cheaper for steady traffic, reserved capacity).
- **Read consistency:** Eventually consistent (default, half cost) vs strongly consistent (not on GSIs).
- **Indexes:** **GSI** — different partition/sort key, own capacity, eventually consistent, can add anytime (limit 20). **LSI** — same partition key, different sort key, strongly consistent reads possible, must be created at table creation, imposes 10 GB per partition-key-value limit (limit 5).
- **Query vs Scan:** Query uses keys (efficient); Scan reads the whole table (expensive).
- **DynamoDB Streams:** Ordered change log (24 h) → Lambda/Kinesis for event-driven patterns. Kinesis Data Streams integration as an alternative.
- **TTL:** Automatic deletion of expired items (free, within ~48 h).
- **Transactions:** `TransactWriteItems`/`TransactGetItems` — ACID across up to 100 items, 2× cost.
- **Conditional writes & optimistic locking** (version attribute).
- **Global Tables:** Multi-region, multi-active replication (last-writer-wins); **multi-region strong consistency** option (2025).
- **DAX:** In-memory write-through cache, microsecond reads.
- **Backups:** PITR (35 days), on-demand backups, export to S3, zero-ETL to OpenSearch/Redshift.

### Interview Questions

**Q1. How do you design a DynamoDB data model?**
Start from **access patterns**, not entities. List every query the app needs (with frequency/latency), then design primary keys and GSIs to satisfy them with `Query`/`GetItem` only. Often use **single-table design**: generic keys like `PK = USER#123`, `SK = ORDER#2025-01-01#456` so related items share a partition and can be fetched in one query (item collections). Use GSIs for alternate access patterns (`GSI1PK = STATUS#PENDING`), sparse indexes for filtering, composite sort keys for hierarchical queries (`begins_with`). Trade-off: single-table is efficient but harder to evolve and less readable; multi-table is fine when patterns are simple or analytics-heavy.

**Q2. What is a hot partition and how do you avoid it?**
When traffic concentrates on one partition key (e.g. `status=active`, today's date, a celebrity user), it exceeds per-partition throughput and gets throttled even though table capacity is available. Avoid with high-cardinality partition keys, **write sharding** (append a random/calculated suffix `DATE#2025-01-01#7` and scatter-gather on read), caching hot reads with DAX/ElastiCache, and spreading time-series writes. Use CloudWatch Contributor Insights to find hot keys.

**Q3. GSI vs LSI?**
GSI: any attributes as keys, created anytime, separate throughput, eventually consistent only, no size constraint per partition key. LSI: same partition key with a different sort key, must be defined at creation, supports strongly consistent reads, shares table throughput, and limits item collections to 10 GB. In practice, GSIs are used ~95% of the time; LSIs only when strong consistency on an alternate sort order is required. Note GSI write throttling can back-pressure the base table.

**Q4. On-demand vs provisioned capacity?**
On-demand: no capacity planning, scales instantly (up to 2× previous peak instantly, more over time), pay per request — best for new, spiky or unpredictable workloads, and dev/test. Provisioned + auto scaling: cheaper for steady, predictable traffic (especially with reserved capacity) but auto scaling reacts in minutes, so sudden spikes throttle. You can switch modes (limited number of times per day).

**Q5. How do you implement idempotency or prevent double-processing using DynamoDB?**
Use a **conditional write**: `PutItem` with `ConditionExpression: attribute_not_exists(PK)` on an idempotency key (request ID / message ID). If the condition fails, the request was already processed — return the stored result. Add TTL to clean up keys. For updates, use **optimistic locking** with a `version` attribute (`ConditionExpression: version = :expected`). Powertools for Lambda implements this pattern.

**Q6. How do you handle analytics or ad-hoc queries on DynamoDB data?**
DynamoDB isn't built for ad-hoc queries or aggregations. Options: **export to S3** (PITR-based, no RCU consumption) and query with Athena; **zero-ETL integration** to Redshift or OpenSearch; DynamoDB Streams → Lambda/Firehose → data lake; maintain pre-aggregated counters with atomic `UpdateItem ADD`.

**Q7. How does DynamoDB Global Tables handle conflicts?**
Default (multi-region eventual consistency): all replicas accept writes, replicate asynchronously (typically <1 s), and conflicts are resolved with **last-writer-wins** based on timestamps — so concurrent updates to the same item in two regions can lose one write. Mitigate by routing writes for a given entity to a home region, or use **multi-region strong consistency (MRSC)** mode which provides RPO=0 with strongly consistent reads at higher write latency (requires 3 regions/witness).

---

## 17. ElastiCache & MemoryDB (Redis / Valkey)

**What they are:**
- **ElastiCache:** Managed in-memory cache — **Valkey** (open-source Redis fork, AWS-recommended, cheaper), **Redis OSS**, **Memcached**. Provisioned clusters or **ElastiCache Serverless**.
- **MemoryDB:** Redis/Valkey-compatible **durable** in-memory **primary database** using a multi-AZ transactional log — microsecond reads, single-digit ms writes. Also supports **vector search**.

**Key concepts:** Cluster mode (sharding across up to 500 shards), replicas & Multi-AZ auto-failover, eviction policies (`allkeys-lru`, `volatile-ttl`...), data structures (strings, hashes, sorted sets, streams, HyperLogLog, geo), Pub/Sub, persistence (RDB/AOF — not durable in ElastiCache), Global Datastore (cross-region replication).

**Caching strategies:**
- **Cache-aside (lazy loading):** App reads cache → on miss reads DB → populates cache. Simple; first request slow; possible stale data.
- **Write-through:** Write to cache and DB together. Fresh cache; writes slower; caches unread data.
- **Write-behind:** Write to cache, asynchronously flush to DB. Fast; risk of loss.
- **TTL + jitter** for all strategies.

### Interview Questions

**Q1. Explain cache-aside vs write-through. Which do you use?**
Cache-aside only caches what's read, resilient to cache failure (falls back to DB), but has miss latency and staleness until TTL/invalidation. Write-through keeps cache fresh but writes every item (even never-read) and adds write latency. Typical production answer: cache-aside with TTL, plus **invalidate (delete) the key on writes** — not update, to avoid race conditions where an older value overwrites newer data.

**Q2. What is a cache stampede (thundering herd) and how do you prevent it?**
When a hot key expires, many concurrent requests miss simultaneously and hammer the DB. Prevent with: **request coalescing / locking** (only one request rebuilds — e.g. `SET key:lock NX PX`), **TTL jitter** (randomize expiry so keys don't expire together), **probabilistic early refresh** (refresh before expiry), serve-stale-while-revalidate, and pre-warming hot keys.

**Q3. ElastiCache vs MemoryDB vs DAX?**
ElastiCache: cache in front of any data store — data loss tolerable. MemoryDB: when Redis/Valkey **is the primary database** and you need durability (Multi-AZ transaction log) — e.g. leaderboards, session stores, real-time state you can't lose. DAX: transparent, API-compatible cache specifically for DynamoDB (write-through for item cache), microsecond reads with minimal code change.

**Q4. Why has AWS been promoting Valkey?**
After Redis changed its license (2024), the Linux Foundation forked it as Valkey (BSD). ElastiCache/MemoryDB for Valkey is priced lower (~20–33% cheaper) with compatible APIs, and offers performance improvements (multi-threaded I/O). Migration from Redis OSS is typically an in-place engine upgrade with no downtime.

**Q5. How do you implement a distributed rate limiter with Redis/Valkey?**
Sliding-window or token bucket using atomic operations: e.g. a sorted set per user with timestamps (`ZADD`, `ZREMRANGEBYSCORE`, `ZCARD`) inside a Lua script / `MULTI` for atomicity, or fixed-window counters with `INCR` + `EXPIRE`. Use cluster hash tags to keep a user's keys on one shard. Decide fail-open vs fail-closed if the cache is unavailable. (API Gateway usage plans or WAF rate-based rules are alternatives at the edge.)

---

## 18. Purpose-built Data Services: Redshift, Athena, OpenSearch, Neptune, DocumentDB

**What they are:**
- **Redshift:** Columnar, MPP **data warehouse** for analytics (OLAP). Provisioned RA3 (compute/storage separation) or **Redshift Serverless**. Spectrum queries S3; zero-ETL integrations from Aurora/RDS/DynamoDB; data sharing across clusters.
- **Athena:** Serverless **SQL on S3** (Trino/Presto engine), pay per TB scanned. Works with Glue Data Catalog, Iceberg tables. Optimize with Parquet/ORC, partitioning, compression.
- **OpenSearch Service:** Managed OpenSearch (Elasticsearch fork) — full-text search, log analytics, observability, and **vector search** (k-NN for RAG). Provisioned or Serverless collections.
- **Neptune:** Managed **graph database** (property graph with Gremlin/openCypher, RDF/SPARQL). Neptune Analytics for graph algorithms and GraphRAG.
- **DocumentDB:** MongoDB-compatible document database (Aurora-like storage).
- Others worth knowing: **Keyspaces** (Cassandra), **Timestream** (time series, incl. InfluxDB), **QLDB** (deprecated), **Glue** (ETL, catalog), **EMR** (Spark/Hadoop), **Lake Formation** (data lake permissions), **SageMaker Lakehouse/Unified Studio**.

### Interview Questions

**Q1. Athena vs Redshift — when to use which?**
Athena: ad-hoc, intermittent queries on S3 data lake, no infrastructure, pay per scan — great for log analysis, exploration. Redshift: frequent, complex, concurrent BI queries with predictable performance, joins across large fact tables, materialized views, workload management. Many orgs use both: S3 lake (Iceberg) + Athena for exploration, Redshift for curated warehouse & dashboards, Redshift Spectrum to bridge.

**Q2. How do you reduce Athena cost and improve performance?**
Store data in **columnar formats (Parquet/ORC)** with compression — scans only needed columns; **partition** by commonly filtered columns (date, region) and use partition projection; avoid many tiny files (compact to 128 MB–1 GB); use Iceberg tables for compaction and time travel; `SELECT` only needed columns; use workgroups with per-query data scan limits.

**Q3. When is OpenSearch the right choice for vector search vs alternatives?**
OpenSearch: when you need **hybrid search** (BM25 keyword + vector), filtering, aggregations, and low-latency queries at scale — common for production RAG and semantic search. Alternatives: Aurora/RDS PostgreSQL with **pgvector** (if data already relational, moderate scale, transactional consistency), **S3 Vectors** (cheapest, large corpora, higher latency acceptable), MemoryDB (ultra-low latency), Neptune Analytics for GraphRAG.

**Q4. When would you use a graph database like Neptune?**
When relationships are first-class and queries traverse many hops: social networks, fraud rings detection, recommendation engines, knowledge graphs, identity/access graphs, network topology. Doing multi-hop traversals in SQL needs recursive joins that degrade quickly; graph DBs traverse via index-free adjacency.

**Q5. What is zero-ETL and why does it matter?**
Managed, near-real-time replication from operational stores (Aurora MySQL/PostgreSQL, RDS, DynamoDB) into Redshift/OpenSearch/SageMaker Lakehouse without building pipelines (no Glue/DMS jobs). Reduces pipeline maintenance and latency of analytics — good answer for "how do you do analytics on transactional data without hurting the OLTP database".

---

## 19. SQS (Simple Queue Service)

**What it is:** Fully managed message queue for decoupling producers and consumers. Pull-based, virtually unlimited throughput (Standard).

**Key concepts**
- **Standard queue:** Nearly unlimited TPS, **at-least-once** delivery, best-effort ordering → consumers must be idempotent.
- **FIFO queue:** Exactly-once processing (5-minute deduplication window via dedup ID or content-based), strict ordering per **message group ID**; 300 TPS per API (3,000 with batching), high-throughput mode up to tens of thousands TPS.
- **Visibility timeout:** After a consumer receives a message, it's hidden (default 30 s, max 12 h). If not deleted in time → visible again (reprocessed). Extend with `ChangeMessageVisibility` for long tasks.
- **Dead-letter queue (DLQ):** After `maxReceiveCount` failures, message moves to DLQ; **DLQ redrive** sends back to source.
- **Long polling:** `WaitTimeSeconds` up to 20 s — fewer empty responses, lower cost.
- **Delay queues / message timers:** Up to 15 min.
- **Retention:** 1 min–14 days (default 4 days). **Message size:** up to 1 MiB (raised from 256 KB in 2025); larger payloads → S3 with the Extended Client Library (claim-check pattern).
- **Fair queues** (2025): mitigates noisy-neighbor tenants in multi-tenant Standard queues.

**Uses:** Decoupling microservices, buffering spikes (load leveling), background jobs, fan-out (with SNS), order processing (FIFO).

### Interview Questions

**Q1. Standard vs FIFO — how do you choose?**
Standard for maximum throughput when order doesn't matter and consumers are idempotent (most cases). FIFO when order matters per entity (e.g. events for one account must apply in order) — use message group ID = entity ID so different entities still process in parallel. FIFO has throughput limits and a head-of-line blocking risk: a failing message blocks its group until it succeeds or goes to DLQ.

**Q2. How do you guarantee a message isn't processed twice?**
With Standard you can't — design **idempotent consumers**: dedupe using a processed-message table (DynamoDB conditional write on message ID / business key), make operations naturally idempotent (upserts, `SET` vs `INCREMENT`), or use FIFO deduplication (only within 5 minutes, and only for producer-side duplicates — consumer-side retries after visibility timeout can still cause reprocessing of side effects).

**Q3. What happens if processing takes longer than the visibility timeout?**
The message becomes visible and another consumer picks it up → duplicate processing and possibly a loop. Fix: set visibility timeout > max processing time (for Lambda, ≥6× function timeout), extend visibility via heartbeat (`ChangeMessageVisibility`) for long tasks, or break the work into smaller steps.

**Q4. How do you handle poison messages?**
Configure a **redrive policy** with a DLQ and `maxReceiveCount` (e.g. 3–5). Alarm on DLQ depth (`ApproximateNumberOfMessagesVisible > 0`). Investigate, fix the bug, then use **DLQ redrive** to move messages back. For FIFO, the DLQ must be FIFO too. Use partial batch responses with Lambda so one bad message doesn't fail the whole batch.

**Q5. SQS vs SNS vs EventBridge vs Kinesis?**
SQS: queue — one consumer group pulls, buffering, retries, decoupling. SNS: pub/sub push fan-out to many subscribers (SQS, Lambda, HTTP, email/SMS), very high throughput, minimal filtering. EventBridge: event bus with rich content-based routing rules, schema registry, SaaS integrations, archive/replay, cross-account — best for event-driven architectures between domains. Kinesis: ordered, replayable **stream** with multiple consumers reading the same data independently at their own position (analytics, real-time processing). Common combo: SNS/EventBridge → multiple SQS queues (fan-out with buffering).

**Q6. How do you send messages larger than the limit?**
**Claim-check pattern**: store the payload in S3 and send a pointer (bucket/key) in the message — the SQS Extended Client Library automates this. Or compress the payload. Make sure consumers have S3 permissions and lifecycle rules clean up payloads.

---

## 20. SNS (Simple Notification Service)

**What it is:** Fully managed pub/sub messaging. Publishers send to a **topic**; SNS pushes to all subscribers.

**Key concepts**
- **Subscribers:** SQS, Lambda, HTTP/S endpoints, Firehose, email, SMS, mobile push.
- **Standard topics:** At-least-once, best-effort ordering, very high throughput. **FIFO topics:** ordering + dedupe, only SQS FIFO subscribers.
- **Message filtering:** Subscription filter policies on message attributes or body — subscribers receive only relevant messages.
- **Delivery retries & DLQ** per subscription; raw message delivery.
- **Message archiving & replay** for FIFO topics.

**Uses:** Fan-out pattern (one event → many services), alerts/notifications (CloudWatch alarms → SNS → PagerDuty/Slack), mobile push, SMS OTP.

### Interview Questions

**Q1. Explain the SNS + SQS fan-out pattern and why it's used.**
Producer publishes once to an SNS topic; multiple SQS queues subscribe (e.g. billing, shipping, analytics). Each service consumes at its own pace, gets durable buffering, retries and DLQs independently, and a slow/down consumer doesn't affect others. Adding a new consumer requires no producer change. Use filter policies so each queue receives only relevant events.

**Q2. How do you ensure SNS can deliver to an encrypted SQS queue?**
If the queue uses SSE-KMS with a customer-managed key, the KMS key policy must allow the `sns.amazonaws.com` principal `kms:GenerateDataKey` and `kms:Decrypt`; and the SQS queue policy must allow `sqs:SendMessage` from the topic ARN (`aws:SourceArn` condition). The AWS-managed `aws/sqs` key won't work for SNS because you can't edit its policy.

**Q3. What happens when an HTTP subscriber is down?**
SNS retries with a delivery policy (immediate retries, backoff phase, up to ~23 days for HTTP depending on policy). After retries are exhausted, the message is dropped unless a **subscription DLQ** (SQS) is configured. For critical delivery prefer SQS subscribers (durable) and have the consumer call the HTTP service.

**Q4. SNS vs EventBridge for pub/sub?**
SNS: higher throughput, lower latency, simpler, mobile/SMS/email delivery, cheaper at scale. EventBridge: richer filtering on any JSON field (content-based, prefix/numeric/anything-but), 20+ AWS service and SaaS sources, schema registry, archive & replay, cross-account/cross-region buses, API destinations, input transformation. Use SNS for high-volume simple fan-out, EventBridge for domain events and integrations.

**Q5. How do you guarantee ordering with SNS?**
Use an SNS **FIFO topic** with message group IDs and SQS FIFO queue subscribers. Ordering is per message group; throughput is limited compared to standard topics.

---

## 21. EventBridge

**What it is:** Serverless event bus for building event-driven applications — routes events from AWS services, your apps, and SaaS partners to targets based on rules.

**Key concepts**
- **Event buses:** Default (AWS service events), custom, partner (SaaS like Datadog, Zendesk, Stripe via partner sources).
- **Rules:** Event pattern matching (content-based filtering) → up to 5 targets per rule (Lambda, SQS, SNS, Step Functions, Kinesis, API destinations, another bus...).
- **Input transformation:** Reshape event before delivering.
- **Archive & replay:** Store events and replay for recovery/testing.
- **Schema registry & discovery:** Auto-generate code bindings.
- **EventBridge Pipes:** Point-to-point integration: source (SQS, Kinesis, DynamoDB Streams, MSK) → filter → enrich (Lambda/API) → target. Reduces glue code.
- **EventBridge Scheduler:** Serverless cron/one-time schedules at massive scale (millions of schedules), time zones, retries, flexible windows. Replaces CloudWatch Events scheduled rules.
- **Cross-account/cross-region** event routing; DLQs and retry policies per target.

**Uses:** Event-driven microservices, reacting to AWS changes (EC2 state change, GuardDuty findings → remediation), SaaS integrations, scheduled jobs, decoupling domains.

### Interview Questions

**Q1. Design an event-driven order system using EventBridge.**
Order service publishes `OrderPlaced` to a custom bus with a well-defined schema (`source: com.shop.orders`, `detail-type: OrderPlaced`). Rules route to: Payment (via SQS for buffering), Inventory (Step Functions workflow), Notifications (Lambda), Analytics (Firehose → S3). Each target has its own DLQ and retry policy. Archive enabled for replay. Consumers are idempotent (event `id` dedupe). Versioned schemas in the registry; additive changes only.

**Q2. EventBridge Pipes vs a Lambda that reads from SQS and calls a service?**
Pipes removes the boilerplate glue Lambda: managed polling, batching, filtering (pay only for filtered events), optional enrichment step, and delivery to the target. Less code to maintain, fewer failure modes. Use a Lambda when you need complex business logic.

**Q3. How do you guarantee events aren't lost with EventBridge?**
EventBridge itself is durable once `PutEvents` succeeds — but check the `FailedEntryCount` in the response and retry failed entries. For targets: configure retry policy (up to 24 h / 185 attempts) and a **DLQ** per target. Use archive for replay. For atomic "DB write + publish", use the **transactional outbox pattern** (write event to an outbox table in the same transaction, then publish via DynamoDB Streams/CDC + Pipes).

**Q4. EventBridge Scheduler vs cron on EC2 or CloudWatch Events?**
Scheduler is serverless, supports millions of one-time or recurring schedules, time zones and DST, flexible time windows, retries, DLQ, and direct invocation of 270+ AWS services. Ideal for per-user reminders, delayed tasks, scheduled jobs. A cron on EC2 is a single point of failure and requires management.

**Q5. What's the latency and throughput profile, and when wouldn't you use EventBridge?**
Typical latency is low hundreds of ms; throughput is limited by `PutEvents` quotas (regional soft limits, tens of thousands TPS in large regions). Not ideal for ultra-low-latency, very high-volume streaming (use Kinesis/MSK) or ordered processing (no ordering guarantee — use SQS FIFO/Kinesis).

---

## 22. Kinesis & MSK

**What they are:**
- **Kinesis Data Streams (KDS):** Real-time data streaming with **shards** (provisioned) or on-demand mode. Each shard: 1 MB/s or 1,000 records/s in, 2 MB/s out (shared) or 2 MB/s per consumer with **Enhanced Fan-Out**. Ordering per partition key within a shard. Retention 24 h default up to 365 days — **replayable**.
- **Amazon Data Firehose:** Fully managed delivery to S3, Redshift, OpenSearch, Splunk, HTTP, Iceberg tables — with buffering, batching, compression, format conversion (Parquet), Lambda transforms. Near-real-time (buffer intervals).
- **Managed Service for Apache Flink:** Stateful stream processing (windows, joins, aggregations).
- **MSK (Managed Streaming for Apache Kafka):** Managed Kafka clusters (provisioned, **Express brokers**, or **MSK Serverless**), MSK Connect for Kafka Connect.

### Interview Questions

**Q1. Kinesis Data Streams vs SQS?**
Kinesis: ordered per shard, multiple independent consumers reading the same records, replay within retention, real-time analytics/aggregations. SQS: each message processed by one consumer then deleted, no replay, per-message visibility/retries, easy scaling of workers. Use Kinesis for event streams/telemetry/clickstreams; SQS for task queues.

**Q2. Kinesis vs MSK?**
Kinesis: fully serverless-ish, AWS-native, simpler, IAM integrated, pay per shard/throughput. MSK: Kafka API compatibility (existing Kafka apps, ecosystem like Kafka Connect, Kafka Streams, Schema Registry), finer control, higher throughput per $ at scale, longer/unlimited retention with tiered storage; more operational knobs (partitions, broker sizing). Choose MSK when you need Kafka compatibility or portability; Kinesis for simplicity.

**Q3. You see `ProvisionedThroughputExceededException` on Kinesis. What do you do?**
Hot shard due to a low-cardinality partition key, or total throughput exceeds shard capacity. Fixes: use a higher-cardinality/random partition key (if ordering per key isn't needed), reshard (split hot shards), switch to **on-demand** mode, use KPL aggregation/batching, add retries with backoff. On the read side, use Enhanced Fan-Out for multiple consumers.

**Q4. How do you handle failures in Lambda processing a Kinesis stream?**
A failed batch blocks the shard (Lambda retries until success or record expiry → `IteratorAge` grows). Configure: `BisectBatchOnFunctionError` (split batch to isolate bad record), `MaximumRetryAttempts`, `MaximumRecordAgeInSeconds`, on-failure destination (SQS/SNS) for bad records, `ReportBatchItemFailures`, and **parallelization factor** (up to 10 concurrent batches per shard) to increase throughput while preserving per-key ordering. Alarm on `IteratorAge`.

**Q5. How would you build a real-time clickstream analytics pipeline?**
Clients → API Gateway/ALB → Kinesis Data Streams (partition key = sessionId) → (a) Managed Flink for real-time aggregations (sessions/minute, anomalies) → DynamoDB/OpenSearch dashboards; (b) Firehose → S3 in Parquet partitioned by date (Iceberg) → Athena/Redshift for historical analytics. Kinesis retention allows replays after bugs.

---

## 23. Step Functions

**What it is:** Serverless workflow orchestration using state machines defined in **Amazon States Language (ASL)** (JSON), with a visual designer (Workflow Studio).

**Key concepts**
- **Standard workflows:** Up to 1 year, **exactly-once** execution, full execution history, priced per state transition. For long-running, auditable business processes.
- **Express workflows:** Up to 5 minutes, high volume (100k+/s), **at-least-once** (async) / at-most-once (sync), priced per execution & duration. For high-throughput event processing.
- **States:** Task, Choice, Parallel, **Map** (incl. **Distributed Map** — process millions of S3 objects with up to 10,000 parallel child executions), Wait, Pass, Succeed, Fail.
- **Integrations:** 200+ AWS services via **SDK integrations** (call any AWS API without Lambda). Patterns: Request-Response, **Run a Job (`.sync`)**, **Wait for callback (`.waitForTaskToken`)** — human approvals or external systems.
- **Error handling:** `Retry` (with backoff, jitter) and `Catch` per state.
- **JSONata & variables** (2024) simplify data transformation.
- Bedrock integration — invoke models directly as workflow steps.

**Uses:** Order processing, ETL orchestration, ML pipelines, human approval flows, **Saga pattern** for distributed transactions, batch processing, GenAI prompt chaining.

### Interview Questions

**Q1. Standard vs Express workflows?**
Standard: long-running (up to a year), exactly-once, visual history for 90 days, expensive per transition at high volume — use for business processes, approvals, ETL. Express: short (≤5 min), very high throughput, cheaper for high volume, at-least-once semantics (make steps idempotent), logs to CloudWatch — use for IoT ingestion, streaming transformations, sync microservice orchestration. Can nest Express inside Standard.

**Q2. How do you implement the Saga pattern with Step Functions?**
Model each local transaction as a Task (ReserveInventory → ChargePayment → CreateShipment). Add `Catch` blocks: if ChargePayment fails, go to compensating tasks in reverse (ReleaseInventory), then Fail state. Each step and compensation must be idempotent. Step Functions gives visibility into which step failed and retries with backoff, replacing fragile choreography for complex flows.

**Q3. How do you implement a human approval step?**
Use the `.waitForTaskToken` integration: a Task sends the task token (e.g. via SNS/SES email or a Slack message with approve/reject links → API Gateway → Lambda), and the workflow pauses (up to a year for Standard). The callback calls `SendTaskSuccess`/`SendTaskFailure` with the token. Set `HeartbeatSeconds`/`TimeoutSeconds` to avoid waiting forever.

**Q4. Orchestration (Step Functions) vs choreography (EventBridge)?**
Orchestration: central coordinator, explicit flow, easy to see state/debug, good for complex business transactions with compensation. Choreography: services react to events independently — loose coupling, easy to add consumers, but the end-to-end flow is implicit and harder to trace. Common hybrid: choreography between bounded contexts (domain events), orchestration within a bounded context.

**Q5. How would you process 10 million files in S3 using Step Functions?**
**Distributed Map** state reading an S3 inventory/prefix or CSV manifest, batching items (`ItemBatcher`), running up to 10,000 parallel child workflows (Express) invoking Lambda/ECS, with `ToleratedFailurePercentage` and results written to S3 (`ResultWriter`). Control concurrency to protect downstream services.

---

## 24. API Gateway

**What it is:** Fully managed service to create, publish, secure, and monitor APIs at any scale.

**API types**
| | REST API | HTTP API | WebSocket API |
|---|---|---|---|
| Features | Full: usage plans/API keys, request validation, transformations (VTL), caching, WAF, private APIs, edge-optimized | Lean: JWT/OIDC authorizers, Lambda & HTTP proxies, CORS, private integrations via VPC Link | Persistent bidirectional connections, routes by message, `@connections` callback API |
| Cost/latency | Higher | ~70% cheaper, lower latency | Per message + connection minutes |

**Key concepts**
- **Endpoint types:** Edge-optimized, Regional, Private (via VPC interface endpoint).
- **Integrations:** Lambda proxy, HTTP, AWS service (direct to SQS/DynamoDB/Step Functions without Lambda), mock, **VPC Link** to NLB/ALB/Cloud Map.
- **Auth:** IAM (SigV4), Cognito user pools, Lambda authorizers (token/request, with caching), JWT authorizers (HTTP API), mTLS, resource policies.
- **Throttling:** Account-level (10,000 RPS steady, 5,000 burst, soft), stage/method throttling, **usage plans** with API keys for per-client quotas.
- **Limits:** 29 s default integration timeout (can be increased for Regional/Private REST APIs), 10 MB payload.
- **Stages & stage variables**, canary releases, custom domains, caching (REST), CloudWatch & X-Ray, access logging.

### Interview Questions

**Q1. REST API vs HTTP API — which would you choose?**
Default to **HTTP API** for simple Lambda/HTTP proxies with JWT auth — cheaper and faster. Choose **REST API** when you need usage plans/API keys (monetized/partner APIs), request validation and transformation, response caching, WAF integration, private APIs, edge-optimized endpoints, or direct AWS service integrations with mapping templates.

**Q2. How do you protect a backend from being overwhelmed through API Gateway?**
Layered throttling: account and stage/method-level rate & burst limits, **usage plans** per API key/client, WAF rate-based rules (per IP), Lambda reserved concurrency, request validation to reject bad requests early, caching (REST), and asynchronous patterns (API Gateway → SQS directly, return 202) for spiky writes. Return 429 with Retry-After and have clients back off.

**Q3. How would you handle requests that take longer than 29 seconds?**
Make it **asynchronous**: accept the request, enqueue (SQS/Step Functions), return `202 Accepted` with a job ID; the client polls a status endpoint or receives a callback via WebSocket/webhook/AppSync subscription. Alternatively request a timeout increase (REST regional/private) — but long-held connections are fragile. For LLM responses, use **response streaming** (Lambda Function URLs / API Gateway REST response streaming support) or WebSockets.

**Q4. Lambda authorizer vs Cognito authorizer vs JWT authorizer?**
Cognito authorizer (REST): validates Cognito user pool tokens — zero code. JWT authorizer (HTTP API): validates any OIDC JWT (Cognito, Auth0, Okta) issuer/audience/scopes — zero code. Lambda authorizer: custom logic (opaque tokens, API keys from DB, fine-grained policies, multi-tenant checks) — returns an IAM policy, cache the result by token for TTL to reduce latency/cost. Use Verified Permissions for fine-grained authorization decisions.

**Q5. How do you expose a private service in a VPC through API Gateway?**
HTTP API: VPC Link (to ALB, NLB, or Cloud Map) — VPC Link v2. REST API: VPC Link to an NLB (and now ALB supported directly). For APIs that should only be reachable inside a VPC, use a **Private REST API** with an `execute-api` interface endpoint and a resource policy restricting `aws:SourceVpce`.

**Q6. How do you version APIs and do canary deployments?**
Versioning: path-based (`/v1`, `/v2`) or separate stages/custom domain base path mappings; avoid breaking changes in place. Canary: REST API stage **canary settings** route a percentage of traffic to a new deployment; monitor metrics; promote or roll back. Lambda aliases with weighted routing provide another canary layer.

---

## 25. AppSync

**What it is:** Managed **GraphQL** (and Pub/Sub **Events** API) service with real-time subscriptions over WebSockets, offline sync, and resolvers connecting to DynamoDB, Lambda, RDS (Data API), OpenSearch, HTTP, EventBridge, and Bedrock.

**Key concepts:** Schema, resolvers (JavaScript `APPSYNC_JS` or VTL), pipeline resolvers, auth modes (API key, Cognito, IAM, OIDC, Lambda — multiple per API), caching, subscriptions, **merged APIs** (federate multiple teams' schemas), **AppSync Events** (serverless WebSocket pub/sub channels).

**Uses:** Mobile/web apps needing flexible queries and real-time updates (chat, live dashboards), BFF (backend-for-frontend) aggregating microservices, streaming AI responses to clients.

### Interview Questions

**Q1. When would you use AppSync instead of API Gateway REST?**
When clients need flexible data shapes (avoid over/under-fetching), aggregation of multiple backends in a single request, **real-time subscriptions** without managing WebSocket connections, offline-first mobile sync (Amplify DataStore). REST via API Gateway is simpler for resource-oriented, cache-friendly public APIs.

**Q2. How do real-time subscriptions work in AppSync?**
Clients subscribe over a managed WebSocket to a subscription field tied to a mutation (`@aws_subscribe(mutations: ["createMessage"])`). When the mutation runs, AppSync pushes the result to subscribed clients, with server-side filtering (enhanced filters). AppSync handles connection scaling, auth and fan-out. For publishing from backends without mutations, use AppSync Events or a NONE-datasource mutation.

**Q3. How do you prevent the N+1 problem in GraphQL resolvers on AppSync?**
Use **batch resolvers** (Lambda batch invoke with `BatchInvoke` operation, or DynamoDB `BatchGetItem`), design pipeline resolvers to fetch related data in bulk, use caching, and design schema/data model so common nested queries hit a single item collection in DynamoDB.

**Q4. How do you secure an AppSync API with multiple client types?**
Configure multiple auth modes: Cognito for end users, IAM for backend services, API key for public read-only (with expiry), Lambda auth for custom tokens. Use field-level directives (`@aws_cognito_user_pools(cognito_groups: ["admin"])`, `@aws_iam`) and resolver-level checks of `ctx.identity` for ownership. Attach WAF for rate limiting.

**Q5. What are AppSync Merged APIs?**
A way to combine multiple source GraphQL APIs owned by different teams into one merged endpoint for clients — similar to schema federation — so teams deploy independently while clients see a unified graph.

---

## 26. IAM (Identity and Access Management)

**What it is:** Global service controlling **who** (authentication) can do **what** on **which resources** under **which conditions** (authorization) in AWS.

**Key concepts**
- **Principals:** Root user (lock away, MFA, no access keys), IAM users (avoid for humans — use Identity Center), **roles** (temporary credentials via STS — preferred for everything), federated identities, AWS services.
- **Policies (JSON):** `Effect`, `Action`, `Resource`, `Condition`, `Principal` (resource-based only).
  - *Identity-based* (attached to user/group/role) — managed (AWS/customer) or inline.
  - *Resource-based* (S3 bucket policy, KMS key policy, SQS/SNS policy, Lambda resource policy, role **trust policy**).
  - *Permission boundaries* — max permissions an identity can have (delegated admin).
  - *SCPs / RCPs* (Organizations) — account-level guardrails.
  - *Session policies* — further restrict an assumed-role session.
- **Policy evaluation logic:** Default implicit deny → **explicit Deny anywhere wins** → must be allowed by SCP **and** RCP **and** permission boundary **and** session policy **and** (identity policy **or** resource policy, for same-account).
  - Cross-account: need allow on **both** sides (identity policy in caller account AND resource policy/trust in target account).
- **STS:** `AssumeRole`, `AssumeRoleWithWebIdentity` (OIDC — EKS IRSA, GitHub Actions), `AssumeRoleWithSAML`, `GetSessionToken`. `ExternalId` prevents confused-deputy in third-party access.
- **Conditions:** `aws:PrincipalOrgID`, `aws:SourceIp`, `aws:SourceVpce`, `aws:RequestedRegion`, `aws:MultiFactorAuthPresent`, `aws:PrincipalTag`/`aws:ResourceTag` (**ABAC**), `aws:SourceArn`/`aws:SourceAccount` (confused deputy protection for services).
- **Tools:** IAM Access Analyzer (external access findings, unused access, policy validation, policy generation from CloudTrail), credential reports, last-accessed data, IAM Roles Anywhere (for on-prem workloads with X.509 certs).

### Interview Questions

**Q1. Walk through IAM policy evaluation logic.**
1. Start with implicit deny. 2. Evaluate all applicable policies; if any **explicit Deny** matches → deny. 3. Organizations **SCPs** (and RCPs for resources) must allow. 4. Resource-based policies — within the same account, an allow in a resource policy can grant access by itself. 5. Identity-based policy must allow. 6. **Permission boundary** must allow (if set). 7. **Session policy** must allow (if used). Effective permission = intersection of guardrails, union of identity + resource grants (same account). Cross-account requires allows on both sides.

**Q2. How does cross-account access work?**
Preferred: **role assumption**. Account B creates role `ReadOnlyFromA` with a trust policy allowing principal `arn:aws:iam::A:role/App` (optionally with `ExternalId`, `aws:PrincipalOrgID`, MFA conditions). Account A's role has an identity policy allowing `sts:AssumeRole` on that role ARN. The app calls STS and uses temporary credentials. Alternative: resource-based policy on the resource (S3 bucket policy granting account A) — no role switching, caller keeps its identity — but both sides still need allow.

**Q3. What is a permission boundary and when do you use it?**
A managed policy that sets the **maximum** permissions an IAM user/role can get, regardless of attached policies. Used to safely delegate role creation: e.g. developers can create roles for their Lambdas but only if they attach the `DevBoundary` boundary (enforced via condition `iam:PermissionsBoundary`), preventing privilege escalation to admin.

**Q4. What is ABAC and why use it over RBAC?**
Attribute-Based Access Control uses tags: e.g. allow `ec2:StopInstances` when `aws:ResourceTag/team == ${aws:PrincipalTag/team}`. Scales better than RBAC — new projects/resources don't require new policies, just correct tags. Requires governance of tags (tag policies, SCPs preventing tag changes) since tags become security controls.

**Q5. What is the confused deputy problem?**
A third party (or AWS service) with permission to act on behalf of many customers is tricked into using its privileges on your resources for someone else. Mitigations: **`sts:ExternalId`** in trust policies for third-party SaaS roles (unique per customer), and `aws:SourceArn` / `aws:SourceAccount` conditions in resource policies for service principals (e.g. allow `sns.amazonaws.com` only from your topic ARN).

**Q6. How do you implement least privilege in practice?**
Start with roles per workload (never shared), use AWS managed policies only to bootstrap, then **generate policies from CloudTrail activity** with Access Analyzer, review **last-accessed** info to remove unused permissions, scope `Resource` ARNs and add conditions, use permission boundaries and SCP guardrails, Access Analyzer custom policy checks in CI (fail PRs that add `*:*` or new public access), and continuous unused-access findings.

**Q7. How should CI/CD (e.g. GitHub Actions) authenticate to AWS?**
**OIDC federation** — no long-lived access keys. Create an IAM OIDC provider for `token.actions.githubusercontent.com` and a role whose trust policy restricts `sub` to a specific repo/branch/environment (`repo:org/app:ref:refs/heads/main`) and `aud` to `sts.amazonaws.com`. The workflow uses `aws-actions/configure-aws-credentials` to get short-lived STS credentials. Separate roles per environment with least privilege.

---

## 27. AWS Organizations, SCPs, RCPs, Control Tower

**What they are:**
- **AWS Organizations:** Centrally manage multiple accounts — **Organizational Units (OUs)**, consolidated billing (volume discounts, shared RIs/SPs), policies.
- **SCPs (Service Control Policies):** Guardrails limiting the **maximum** permissions for principals in member accounts (do not grant; don't affect the management account; affect root user of member accounts).
- **RCPs (Resource Control Policies, Nov 2024):** Guardrails on **resources** (S3, STS, KMS, SQS, Secrets Manager...) — e.g. ensure no resource in the org can be accessed by identities outside the org (data perimeter).
- **Other policy types:** Tag policies, backup policies, AI services opt-out policies, declarative policies (enforce EC2 settings like IMDSv2, block public AMIs).
- **Control Tower:** Automates a well-architected **landing zone**: creates Log Archive and Audit accounts, centralized CloudTrail/Config, IAM Identity Center, preventive (SCP/RCP), detective (Config) and proactive (CloudFormation hooks) **controls**, and **Account Factory** (incl. AFT — Account Factory for Terraform).
- **Delegated administrator:** Run security services (GuardDuty, Security Hub) from a security account, not management account.

### Interview Questions

**Q1. Why use multiple AWS accounts?**
Accounts are the strongest isolation boundary: **blast radius** containment (security incidents, misconfiguration), separate **service quotas**, clear **cost allocation**, environment separation (prod vs dev), independent team autonomy, and compliance scoping (PCI workloads in dedicated accounts). Organizations + Control Tower make many accounts manageable.

**Q2. Describe a typical multi-account OU structure.**
Root → **Security OU** (Log Archive, Audit/Security Tooling), **Infrastructure OU** (Networking/Transit, Shared Services — CI/CD, DNS, artifact registries), **Workloads OU** with sub-OUs Prod/Non-Prod (accounts per team/app/env), **Sandbox OU** (experimentation with budgets and strict SCPs), **Suspended OU** (deny-all for closing accounts), **Policy Staging OU** for testing SCPs. Management account used only for billing/org admin.

**Q3. Give examples of useful SCPs.**
Deny leaving the organization; deny disabling CloudTrail/GuardDuty/Config/Security Hub; **region restriction** (deny all except approved regions, with exceptions for global services); deny creating IAM users/access keys; require IMDSv2; deny root user actions; deny modification of baseline security roles; deny unencrypted S3 uploads/EBS volumes. Always use `Deny` with exception conditions for break-glass roles, and test in a staging OU first.

**Q4. SCP vs RCP vs IAM policy?**
IAM policies **grant** permissions to identities. SCPs **limit** what identities in an account can do (principal-side guardrail). RCPs **limit** what can be done **to resources** in your accounts, regardless of who calls (resource-side guardrail) — e.g. "S3 buckets in our org can only be accessed by principals in our org or AWS services on our behalf", which SCPs can't enforce against external principals. Together they form a **data perimeter** (identity, resource, network perimeters).

**Q5. What does Control Tower give you beyond Organizations?**
Opinionated, automated landing zone setup (log archive & audit accounts, org-wide CloudTrail, Config aggregation), a library of managed controls with compliance dashboards, Account Factory for consistent account vending with baseline networking/guardrails, drift detection of the landing zone, and Identity Center integration. You can still customize via Customizations for Control Tower (CfCT) or AFT.

---

## 28. IAM Identity Center & Cognito

**What they are:**
- **IAM Identity Center (successor to AWS SSO):** Workforce identity — single sign-on for **employees** to multiple AWS accounts and business apps. Connects to an external IdP (Okta, Entra ID, Google) via SAML + SCIM provisioning or uses its own directory. **Permission sets** define roles provisioned into accounts. Provides short-lived CLI credentials (`aws sso login`). Trusted identity propagation for analytics/AI services (e.g. Q Business).
- **Amazon Cognito:** Customer identity (CIAM) for **your app's end users**.
  - *User Pools:* user directory + authentication (sign-up/sign-in, MFA, social/SAML/OIDC federation, hosted UI/managed login, passwordless/passkeys), issues **JWTs** (ID, access, refresh tokens). Lambda triggers for customization.
  - *Identity Pools (Federated Identities):* exchange tokens (from User Pools, social, SAML) for **temporary AWS credentials** to call AWS services directly (e.g. upload to S3 from mobile app), including guest access.

### Interview Questions

**Q1. Identity Center vs Cognito?**
Identity Center: workforce (employees/contractors) accessing AWS accounts, console, CLI and internal apps — integrated with corporate IdP. Cognito: customers/end users of your application — sign-up, social login, app tokens, scaling to millions of users. Don't use Cognito for engineers' AWS access or Identity Center for app customers.

**Q2. Cognito User Pools vs Identity Pools?**
User Pools **authenticate** users and give JWTs your backend/API Gateway validates. Identity Pools **authorize access to AWS resources** by exchanging an identity token for temporary IAM credentials mapped to roles (authenticated/unauthenticated, with role mapping by claims). Often used together: sign in via User Pool → exchange ID token at Identity Pool → get scoped AWS creds (e.g. S3 prefix `${cognito-identity.amazonaws.com:sub}/*`).

**Q3. How do you implement multi-tenant authorization with Cognito?**
Options: tenant ID as a custom attribute or group added to tokens (pre-token-generation Lambda trigger to inject claims/scopes), one user pool with tenant claims (simplest, scalable), or user pool per tenant (stronger isolation, custom settings, more ops). Enforce tenant isolation at the API (authorizer validates `tenant_id`) and data layer (DynamoDB partition key prefix + IAM `dynamodb:LeadingKeys` conditions, or Postgres RLS). Use Verified Permissions for fine-grained policies.

**Q4. How do you securely give engineers access to production accounts?**
Identity Center with corporate IdP + MFA, SCIM-synced groups mapped to permission sets (ReadOnly, Developer, Admin) per account. Prod write access via just-in-time elevated access (temporary group membership with approval — e.g. TEAM solution or third-party), session duration limits, CloudTrail logging, break-glass accounts with alerts. No IAM users or long-lived keys.

**Q5. How do you validate a Cognito JWT in a backend service?**
Fetch the JWKS from `https://cognito-idp.{region}.amazonaws.com/{userPoolId}/.well-known/jwks.json` (cache it), verify signature (RS256) using the `kid`, then validate claims: `iss` (user pool), `exp`, `token_use` (`access` vs `id`), `client_id`/`aud`, and required scopes/groups. Use libraries like `aws-jwt-verify`. Prefer access tokens for API authorization.

---

## 29. KMS, Secrets Manager, SSM Parameter Store

**What they are:**
- **KMS (Key Management Service):** Create and control cryptographic keys (FIPS 140-3 validated HSMs). Integrated with almost every AWS service for encryption at rest.
  - Key types: **AWS owned**, **AWS managed** (`aws/s3`, can't edit policy), **customer managed (CMK)** — full control of key policy, rotation, grants. Symmetric (default) and asymmetric/HMAC keys. Multi-Region keys. External key store (XKS) / CloudHSM custom key store.
  - **Envelope encryption:** KMS generates a **data key** (`GenerateDataKey`) → plaintext data key encrypts data locally → encrypted data key stored alongside data → to decrypt, KMS decrypts the data key. Keeps large data out of KMS (4 KB limit on direct `Encrypt`), reduces latency/cost.
  - Key policies (resource policy — required), grants (temporary delegations), automatic rotation (yearly or custom 90 days–7 years; old material retained).
- **Secrets Manager:** Store, retrieve, and **automatically rotate** secrets (DB credentials, API keys) with Lambda rotation functions (built-in for RDS/Aurora/Redshift/DocumentDB); cross-region replication; resource policies; ~$0.40/secret/month.
- **SSM Parameter Store:** Hierarchical config and secrets storage (`/app/prod/db/url`). Standard (free, 4 KB) and Advanced tiers (8 KB, policies like expiry). `SecureString` encrypted with KMS. No native rotation.

### Interview Questions

**Q1. Explain envelope encryption and why it's used.**
Encrypting large data directly with a KMS key isn't possible (4 KB limit) and would require sending data over the network. Instead, call `GenerateDataKey` → receive a plaintext data key and the same key encrypted under the KMS key. Encrypt data locally with the plaintext key (AES-256-GCM), discard it from memory, and store the encrypted data key with the ciphertext. To decrypt, call `Decrypt` on the encrypted data key, then decrypt data locally. Benefits: performance, unlimited data size, KMS key never leaves HSMs, and rotating/revoking the KMS key controls access to all data keys.

**Q2. Secrets Manager vs Parameter Store?**
Secrets Manager: built-in rotation, cross-region replication, cross-account access via resource policy, larger secrets (64 KB), purpose-built for credentials — costs per secret + API calls. Parameter Store: free standard tier, great for configuration and non-rotating secrets, hierarchical paths, integration everywhere — no rotation, lower throughput by default. Common setup: Parameter Store for config, Secrets Manager for DB creds/API keys needing rotation. (Parameter Store can also reference Secrets Manager secrets.)

**Q3. How does Secrets Manager rotate an RDS password without downtime?**
Rotation Lambda follows 4 steps: `createSecret` (generate new password stored as `AWSPENDING`), `setSecret` (update the DB), `testSecret` (log in with pending), `finishSecret` (move `AWSCURRENT` label to the new version). For zero downtime use the **alternating users** strategy: two DB users, rotation updates the inactive one, so apps holding the old credentials still work until they refresh. Apps should fetch secrets via cached client and retry on auth failure by re-fetching.

**Q4. A Lambda gets AccessDenied on KMS Decrypt although its IAM role allows `kms:Decrypt`. Why?**
The **key policy** is authoritative for KMS — IAM policies only work if the key policy delegates to the account (`"Principal": {"AWS": "arn:aws:iam::ACCOUNT:root"}`, `kms:*`). Other causes: cross-account key where the key policy doesn't allow the external role, encryption context mismatch, wrong region (keys are regional unless multi-region), SCP/VPC endpoint policy denying, or `kms:ViaService` condition restricting usage to a specific service.

**Q5. How do you handle KMS throttling at high volume?**
KMS has per-region request quotas (shared across symmetric operations, thousands to tens of thousands RPS depending on region). Reduce calls with **data key caching** (AWS Encryption SDK), **S3 Bucket Keys** (reduces S3→KMS calls by up to 99%), DynamoDB uses table-level keys, batch operations; request quota increases; implement retries with backoff.

**Q6. What happens when you delete a KMS key?**
Deletion has a mandatory waiting period (7–30 days) during which the key is disabled — you can cancel. After deletion, all data encrypted under it is **unrecoverable**. Best practice: disable first and watch CloudTrail/CloudWatch for usage attempts; restrict `kms:ScheduleKeyDeletion` via SCP.

---

## 30. Security Services: WAF, Shield, GuardDuty, Inspector, Security Hub, Macie, CloudTrail, Config

**What they are:**
| Service | Purpose |
|---|---|
| **AWS WAF** | Layer 7 web application firewall for CloudFront, ALB, API Gateway, AppSync, Cognito — managed rule groups (OWASP core, known bad inputs, SQLi, IP reputation, **Bot Control**, **Account Takeover/Fraud Prevention**), rate-based rules, geo-match, custom rules, CAPTCHA/challenge |
| **Shield** | DDoS protection. *Standard* (free, L3/L4, automatic). *Advanced* (paid: L7 detection with WAF, 24/7 Shield Response Team, cost protection for scaling during attacks, health-based detection) |
| **GuardDuty** | Managed **threat detection** using ML/threat intel on CloudTrail, VPC Flow Logs, DNS logs, plus protection plans for S3, EKS audit logs, **runtime monitoring** (EKS/ECS/EC2), Lambda, RDS login activity, malware scanning (EBS, S3); **Extended Threat Detection** correlates multi-stage attacks |
| **Inspector** | Automated **vulnerability scanning** of EC2, ECR images, Lambda functions (CVEs, network reachability, code vulnerabilities), SBOM export, CIS benchmarks |
| **Security Hub** | Central **posture management** — aggregates findings (GuardDuty, Inspector, Macie, Config, third party) in ASFF/OCSF format, runs security standards (AWS Foundational Best Practices, CIS, PCI DSS, NIST), cross-account/region aggregation, automation rules. Newer version (2025) adds risk correlation/exposure analysis |
| **Macie** | Discover and classify **sensitive data (PII)** in S3 using ML and pattern matching |
| **CloudTrail** | **Audit log of API calls** (who did what, when, from where). Management events (default 90-day event history), data events (S3 object/Lambda invoke — extra cost), Insights (unusual activity), org trails, CloudTrail Lake (SQL queries) |
| **AWS Config** | **Resource configuration history** and compliance evaluation with Config rules (managed/custom), conformance packs, auto-remediation via SSM Automation, aggregators |
| **Detective** | Investigate root cause of findings using graph analysis |
| **Network Firewall** | Managed stateful network firewall/IPS for VPC traffic (Suricata rules, domain filtering) |
| **Firewall Manager** | Centrally manage WAF/Shield/SG/Network Firewall policies across accounts |

### Interview Questions

**Q1. CloudTrail vs Config vs CloudWatch?**
CloudTrail answers **"who did what?"** — API activity logs (user, time, source IP, request parameters). Config answers **"what did the resource look like and is it compliant?"** — configuration snapshots/timeline and rules (e.g. "is this S3 bucket public?"). CloudWatch answers **"how is it performing?"** — metrics, logs, alarms. During an incident: Config shows SG changed at 10:02, CloudTrail shows which role changed it, CloudWatch shows the error spike after.

**Q2. How do you protect a public web app from DDoS and common attacks?**
Edge-first architecture: CloudFront (absorbs L3/L4 with Shield Standard, caches), **AWS WAF** with managed rules (core rule set, SQLi, known bad inputs, IP reputation), **rate-based rules** per IP/header, Bot Control, geo-blocking if relevant; Route 53 (resilient DNS); origin locked to CloudFront only; autoscaling behind ALB; Shield Advanced for critical apps (L7 auto-mitigation, SRT, cost protection). Reduce attack surface — no public EC2s.

**Q3. GuardDuty vs Inspector vs Macie vs Security Hub?**
GuardDuty: **threats/active malicious activity** (compromised credentials, crypto mining, port scanning, C2 communication). Inspector: **vulnerabilities** (CVEs in packages/images, network exposure). Macie: **sensitive data** in S3. Security Hub: **aggregator & posture** — single pane of glass, compliance standards, prioritization, automation. Typical: enable all org-wide with a delegated admin security account; route high-severity findings via EventBridge to Slack/PagerDuty/automated remediation.

**Q4. GuardDuty reports an EC2 instance communicating with a known crypto-mining domain. What's your response?**
Contain: isolate instance by swapping its SG to a quarantine SG (no inbound/outbound), detach/revoke the instance role sessions (deny policy with `aws:TokenIssueTime` condition), remove from ASG/target group. Preserve evidence: EBS snapshot, memory capture if possible, tag the instance. Investigate: CloudTrail for actions by the role, Detective, VPC Flow Logs, how it was compromised (vulnerability via Inspector, leaked key). Eradicate & recover: rebuild from a known-good AMI, patch, rotate credentials. Post-incident: automate this runbook with EventBridge → Step Functions/SSM Automation.

**Q5. How do you make CloudTrail logs tamper-proof?**
Organization trail for all accounts and regions, delivered to an S3 bucket in a dedicated **Log Archive account**, with **log file validation** (SHA-256 digest files), SSE-KMS, S3 Object Lock (compliance mode), versioning, bucket policy denying deletes, SCP denying `cloudtrail:StopLogging/DeleteTrail`, and replication/CloudTrail Lake for analysis.

**Q6. How do you automatically remediate non-compliant resources?**
AWS Config rule (e.g. `s3-bucket-public-read-prohibited`) → automatic remediation with an SSM Automation document; or Security Hub findings → EventBridge rule → Lambda/Step Functions. Prefer **preventive** controls first (SCPs, RCPs, declarative policies, Block Public Access at account level, CloudFormation Guard/hooks), detective + auto-remediation second.

---

## 31. Verified Permissions & Verified Access

**What they are:**
- **Amazon Verified Permissions (AVP):** Managed fine-grained **authorization** service for *your applications*, using the open-source **Cedar** policy language (RBAC + ABAC + ReBAC). Your app calls `IsAuthorized(principal, action, resource, context)`; policies are managed centrally, versioned and analyzable.
- **AWS Verified Access:** **Zero-trust** access to corporate applications (HTTP and, now, non-HTTP like SSH/RDP/databases) **without a VPN** — every request is evaluated against identity (IdP) and device posture (CrowdStrike, Jamf, etc.) policies written in Cedar.

### Interview Questions

**Q1. Why externalize authorization to Verified Permissions instead of coding it?**
Authorization logic scattered in code (`if user.role == 'admin' || user.id == doc.ownerId ...`) is hard to audit, change, and keep consistent across services. AVP centralizes policies (Cedar), decouples them from deployments, supports automated reasoning (prove a policy change doesn't grant unintended access), provides audit logs, and handles multi-tenant policy stores. Trade-off: a network call per decision (mitigate with batch authorization and caching).

**Q2. What does a Cedar policy look like?**
```
permit (
  principal in Group::"editors",
  action in [Action::"view", Action::"edit"],
  resource in Folder::"project-x"
) when { resource.classification != "secret" };
```
`forbid` policies override `permit` — default deny, explicit forbid wins (like IAM). Entities and their hierarchy (user in group, doc in folder) are passed with the request or derived from tokens.

**Q3. How does Verified Access differ from a VPN?**
A VPN grants network-level access once connected — lateral movement risk. Verified Access evaluates **every request** using identity + device posture + context, grants access only to specific applications, logs every access decision, and needs no client VPN software (for HTTP apps). It implements zero-trust principles: never trust the network, always verify.

**Q4. How would you implement authorization for a multi-tenant SaaS on AWS?**
Authenticate via Cognito/IdP with tenant claims; API Gateway Lambda authorizer or middleware calls AVP with principal (user, tenant, roles), action, and resource (with tenant attribute); Cedar policy enforces `resource.tenantId == principal.tenantId` plus role permissions. Data-layer isolation as defense in depth (DynamoDB leading keys conditions via tenant-scoped IAM session, RLS in Postgres).

**Q5. How do you test and safely change authorization policies?**
Store Cedar policies in Git, run unit tests with the Cedar CLI/SDK against fixture requests, use policy validation against a schema, and analyze changes (Cedar's analysis tools can verify "no new permits for X"). Deploy via IaC to a staging policy store first.

---

## 32. CloudWatch

**What it is:** AWS's native monitoring and observability service: metrics, logs, alarms, dashboards, events, application monitoring.

**Key concepts**
- **Metrics:** Namespaces/dimensions; standard (1-min, many services 5-min basic for EC2) vs detailed; **custom metrics** (PutMetricData or **Embedded Metric Format** from logs); high resolution (1 s). Metric math, anomaly detection, Metrics Insights (SQL).
- **Alarms:** Threshold or anomaly-based; states OK/ALARM/INSUFFICIENT_DATA; `M out of N` datapoints; **composite alarms** to reduce noise; actions to SNS, ASG, EC2, SSM, Lambda.
- **Logs:** Log groups/streams, retention policies (default *never expire* — set it!), **Logs Insights** query language, metric filters, subscription filters (to Lambda/Kinesis/Firehose/OpenSearch), Live Tail, data protection (PII masking), Infrequent Access log class, centralized cross-account logging.
- **Application observability:** **Application Signals** (auto-instrumented SLOs, service maps via OpenTelemetry), **Synthetics** canaries, **RUM** (real user monitoring), **Container Insights**, **Lambda Insights**, **Database Insights**, Internet Monitor, Network Monitor.
- **Cross-account observability** (monitoring account linking source accounts).
- **CloudWatch Investigations** (AI-assisted, formerly Q operational investigations) — generates hypotheses for root cause using telemetry.

### Interview Questions

**Q1. How do you design alerting that isn't noisy?**
Alert on **symptoms users feel** (error rate, latency SLO burn rate, availability), not every cause (CPU 80%). Use SLO-based multi-window burn-rate alarms, `M of N` datapoints to avoid flapping, **composite alarms** (e.g. page only if error rate high AND traffic not near zero), anomaly detection for seasonal metrics, route severity appropriately (page vs ticket), and ensure every page has a runbook. Regularly prune alarms nobody acts on.

**Q2. How do you publish custom application metrics efficiently?**
Use **Embedded Metric Format (EMF)** — write structured JSON logs that CloudWatch automatically extracts as metrics (no synchronous PutMetricData calls, no latency impact, high-cardinality details kept in logs). Powertools Metrics does this. Be careful with dimension cardinality (each unique combination is a separate billed metric). For Prometheus-native workloads use Amazon Managed Prometheus.

**Q3. How do you reduce CloudWatch Logs costs?**
Set retention on every log group, reduce verbosity (log level by env, sampling of debug logs), use the **Infrequent Access** log class for logs rarely queried, export/subscribe old logs to S3 (cheaper storage + Athena), avoid logging large payloads, and use metric filters/EMF instead of querying logs for metrics.

**Q4. Write a Logs Insights query to find the top 10 slowest API endpoints.**
```
fields @timestamp, path, duration
| filter ispresent(duration)
| stats avg(duration) as avgMs, pct(duration, 99) as p99, count(*) as calls by path
| sort p99 desc
| limit 10
```
Structured JSON logs make fields auto-discovered.

**Q5. How would you monitor a microservices system end-to-end?**
The three pillars correlated by trace ID: metrics (RED: rate, errors, duration per service; USE for resources), structured logs with trace/correlation IDs, distributed tracing via **OpenTelemetry (ADOT) → X-Ray/Application Signals**. Define SLOs per user journey in Application Signals, Synthetics canaries for critical flows, RUM for frontend, cross-account observability into a central monitoring account, dashboards per service and per business KPI.

---

## 33. X-Ray, ADOT, Managed Prometheus & Grafana

**What they are:**
- **AWS X-Ray:** Distributed tracing — traces, segments/subsegments, service map, sampling rules, annotations (indexed) vs metadata. Now accepts **OpenTelemetry** natively (OTLP endpoint) and is integrated into CloudWatch (Transaction Search, Application Signals). The X-Ray SDKs/daemon are moving to maintenance in favor of OpenTelemetry.
- **ADOT (AWS Distro for OpenTelemetry):** AWS-supported OTel distribution (collector + SDKs/auto-instrumentation) to send traces/metrics/logs to X-Ray, CloudWatch, Managed Prometheus, or third parties.
- **Amazon Managed Service for Prometheus (AMP):** Serverless, Prometheus-compatible metrics storage & PromQL queries; scrape with ADOT/agentless collectors on EKS.
- **Amazon Managed Grafana (AMG):** Managed Grafana workspaces with IAM Identity Center auth, data sources for CloudWatch, AMP, X-Ray, OpenSearch, Athena, etc.

### Interview Questions

**Q1. What is distributed tracing and how does context propagate?**
A trace represents one request's path across services; each unit of work is a span with timing and attributes. Context (trace ID, parent span ID, sampling decision) propagates via headers — W3C `traceparent` (OTel standard) or `X-Amzn-Trace-Id` — and through messaging attributes (SQS message attributes, EventBridge). This lets you find which service in a chain caused latency or errors.

**Q2. Why standardize on OpenTelemetry instead of vendor SDKs?**
Vendor-neutral instrumentation: instrument once, send to any backend (X-Ray, Datadog, Honeycomb, Grafana Tempo) by changing collector config. Broad auto-instrumentation, one standard for traces/metrics/logs, community momentum, and AWS itself is converging on OTel (X-Ray SDK in maintenance). The Collector also allows processing — batching, tail sampling, PII scrubbing.

**Q3. How do you choose sampling rates?**
Head sampling (decide at start) is cheap but may miss rare errors; e.g. 1 request/sec reservoir + 5% for high-volume endpoints, 100% for low-volume critical ones. **Tail-based sampling** in the OTel Collector keeps all traces with errors or high latency and samples the rest — better signal, more infrastructure. Always keep metrics unsampled so SLOs remain accurate.

**Q4. When would you use AMP + AMG instead of CloudWatch?**
When running Kubernetes with the Prometheus ecosystem (kube-state-metrics, exporters, existing PromQL dashboards and alert rules), needing high-cardinality metrics cost-effectively, or wanting portability. CloudWatch is better for native AWS service metrics and simplicity. AMG can visualize both in one place.

**Q5. How do you correlate logs, metrics and traces during an incident?**
Inject trace ID and span ID into every structured log line (OTel log correlation), use exemplars linking metrics to traces, consistent service/resource attributes (`service.name`, `deployment.environment`). In CloudWatch, Application Signals/Transaction Search links traces ↔ logs; in Grafana, configure data links between AMP, X-Ray/Tempo, and logs.

---

## 34. Well-Architected Framework

**What it is:** AWS's set of best practices and design principles for evaluating and improving cloud architectures, organized into **six pillars**, with a free Well-Architected Tool and specialized **Lenses** (Serverless, SaaS, Generative AI, Machine Learning, Container Build, Financial Services...).

**The 6 pillars**
1. **Operational Excellence** — IaC, small reversible changes, observability, runbooks, learn from failures.
2. **Security** — strong identity foundation, traceability, defense in depth, encrypt everywhere, automate security, prepare for incidents.
3. **Reliability** — automatic recovery, test recovery procedures, scale horizontally, stop guessing capacity, manage change via automation.
4. **Performance Efficiency** — use managed/serverless, go global in minutes, experiment, mechanical sympathy (right tool for the job).
5. **Cost Optimization** — consumption model, measure efficiency, stop spending on undifferentiated heavy lifting, attribute expenditure.
6. **Sustainability** — maximize utilization, use managed services, efficient hardware (Graviton), reduce data movement.

### Interview Questions

**Q1. Name the 6 pillars and give one concrete practice for each.**
Operational Excellence — all infrastructure in CDK/Terraform with CI/CD. Security — OIDC federation and least-privilege roles, no long-lived keys. Reliability — multi-AZ with health-checked auto recovery and tested backups. Performance — choose DynamoDB for key-value access at scale, CloudFront caching. Cost — Savings Plans for baseline, Spot for batch, tagging for allocation. Sustainability — Graviton and right-sizing to raise utilization.

**Q2. How do pillars conflict, and how do you make trade-offs?**
Reliability vs Cost: multi-region active-active doubles infrastructure — justify via business RTO/RPO and cost of downtime. Security vs Performance: encryption/inspection adds latency. Performance vs Cost: provisioned concurrency removes cold starts but costs idle. A senior engineer makes the trade-off explicit, ties it to business requirements (SLOs, compliance, budget), and documents it in an ADR.

**Q3. How would you run a Well-Architected Review for an existing system?**
Gather stakeholders (dev, ops, security, product), answer pillar questions in the WA Tool (and relevant lens), identify **high-risk issues (HRIs)**, prioritize by business impact and effort, create an improvement plan with owners and milestones, track in backlog, and re-review periodically (e.g. every 6–12 months or after major changes).

**Q4. What's in the Generative AI Lens?**
Best practices for GenAI workloads across the lifecycle: model selection, prompt engineering and management, RAG data pipelines, evaluation (accuracy, hallucination), responsible AI (guardrails, bias, transparency), security (prompt injection, data leakage, least-privilege agent tools), cost (model choice, caching, batch), and operations (observability of prompts/tokens, model versioning).

**Q5. Explain "static stability".**
A system is statically stable if it continues to operate correctly during a dependency failure **without needing to make changes** (no control-plane calls, no scaling). E.g. pre-provision capacity in each AZ to handle the loss of one AZ instead of relying on autoscaling during the event; data-plane-only failover (Route 53 health checks/ARC routing controls rather than editing records). It's a core AWS reliability principle.

---

## 35. High Availability & Disaster Recovery

**Key definitions**
- **High Availability (HA):** Keep the system running through component/AZ failures (typically multi-AZ, automatic).
- **Disaster Recovery (DR):** Recover from large-scale events (region outage, data corruption, ransomware, human error).
- **RTO (Recovery Time Objective):** Max acceptable downtime.
- **RPO (Recovery Point Objective):** Max acceptable data loss (time).
- Availability math: 99.9% = ~8.76 h/year downtime; 99.99% = ~52.6 min/year; 99.999% = ~5.26 min/year. Serial dependencies multiply (0.999 × 0.999 = 0.998).

**The 4 DR strategies (cost ↑, RTO/RPO ↓)**
| Strategy | Description | RPO / RTO |
|---|---|---|
| **Backup & Restore** | Backups (AWS Backup, snapshots) copied to DR region; infrastructure rebuilt via IaC on disaster | Hours / hours–day |
| **Pilot Light** | Core data continuously replicated (DB replicas), minimal infra always on; compute provisioned at failover | Minutes / tens of minutes |
| **Warm Standby** | Scaled-down but fully functional copy running in DR region; scale up on failover | Seconds–minutes / minutes |
| **Multi-site Active/Active** | Full production in multiple regions serving traffic simultaneously | Near zero / near zero |

**Services:** AWS Backup (cross-region/account copies, vault lock), Elastic Disaster Recovery (DRS — block-level replication of servers), Aurora Global Database, DynamoDB Global Tables, S3 CRR, Route 53/ARC, Global Accelerator, AWS Resilience Hub (assess RTO/RPO), Fault Injection Service (FIS).

### Interview Questions

**Q1. Explain the four DR strategies and how you'd choose.**
(See table.) Choose by business requirements: classify each workload by criticality and cost of downtime/data loss. Internal reporting tool → backup & restore. Core e-commerce checkout with RTO 15 min/RPO 1 min → warm standby with Aurora Global DB. Global payments with near-zero tolerance → active/active. The cost difference is large, so not every system needs the same tier.

**Q2. HA vs DR — what's the difference?**
HA handles expected, localized failures (instance, AZ) automatically with redundancy within a region — usually invisible to users. DR handles rare, large-scale events (region outage, corruption, security incident) with a planned recovery process, often involving a different region/account. Important nuance: replication protects against infrastructure failure but **not** against corruption or deletion — those replicate too. You need point-in-time backups (PITR, immutable backups) as well.

**Q3. How do you test DR?**
Regular game days and DR drills: actually fail over to the DR region (not just tabletop), measure achieved RTO/RPO, automate runbooks (failover as code), use **AWS FIS** to inject failures (AZ power interruption scenarios, API throttling, instance termination), validate backups by restoring them periodically (automated restore testing in AWS Backup), and track findings. Untested DR plans usually fail.

**Q4. Design a warm standby DR for a web app (primary us-east-1, DR us-west-2).**
Infrastructure defined in IaC deployed to both regions. DR region runs a minimal ASG/ECS service (e.g. 2 tasks), ALB, Aurora Global Database secondary cluster, DynamoDB Global Tables, S3 CRR, secrets replicated, ECR replication, AMIs copied. Route 53 failover/ARC routing control. On disaster: promote Aurora secondary (managed failover), scale up compute, flip traffic. Monitor replication lag (RPO) continuously and rehearse quarterly.

**Q5. How do you protect backups from ransomware or a compromised admin?**
Copy backups to a separate **backup account** (different credentials, minimal access), use **AWS Backup Vault Lock** in compliance mode (immutable for retention period, even root can't delete), Object Lock for S3, SCPs denying backup deletion, KMS keys in the backup account, logically air-gapped vaults, and MFA for sensitive operations.

**Q6. What's the availability of a system with an ALB → app (99.95%) → DB (99.95%) chain, and how do you increase it?**
Serial: roughly 0.9995 × 0.9995 ≈ 99.90% (plus the LB). Increase with redundancy in parallel (multi-AZ: availability of parallel components = 1 − (1−a)^n), removing hard dependencies (graceful degradation — serve cached data if DB down), async processing, and reducing the number of serial critical dependencies.

---

## 36. Multi-Region & Cell-Based Architecture

**What it is:**
- **Multi-region:** Deploying workloads in two or more AWS regions for DR, lower latency to global users, or data residency.
- **Cell-based architecture:** Partition the workload into multiple independent, identical copies (**cells**), each serving a subset of customers, with a thin **routing layer** mapping customers → cell. Failures (bad deployment, poison request, noisy neighbor) are contained to one cell. AWS itself uses this widely.
- **Shuffle sharding:** Assign each customer to a random combination of resources so two customers rarely share all the same resources — limits blast radius from a single bad customer.

**Key considerations:** Data replication & consistency, write conflicts, routing (Route 53 latency/geo, Global Accelerator), deployment orchestration across regions (waves), regional service availability, cost (data transfer, duplicated infra), and compliance.

### Interview Questions

**Q1. When should you go multi-region and when is it overkill?**
Justified when: business requires surviving a full region outage (RTO/RPO stricter than single-region can provide), global users need low latency, or data residency laws require in-region processing. Overkill for most workloads — multi-AZ already provides very high availability, and multi-region adds significant complexity (data consistency, testing, cost, operational burden). Make it a business decision backed by numbers.

**Q2. Active/active multi-region: how do you handle writes?**
Options: (1) **Write locality / home region** — each user/entity is assigned a home region for writes, reads anywhere (avoids conflicts); (2) **conflict-tolerant data** — DynamoDB Global Tables with last-writer-wins, CRDT-style designs, idempotent commutative operations; (3) **strongly consistent distributed DB** — Aurora DSQL or DynamoDB MRSC at higher write latency; (4) single global writer (Aurora Global DB with write forwarding) — simpler but not truly active/active for writes.

**Q3. What is a cell-based architecture and why use it?**
Instead of one big shared deployment, you run N identical cells, each with its own full stack and data partition. A cell router maps a tenant to a cell. Benefits: blast radius limited to 1/N customers, predictable scaling (add cells instead of growing one system past tested limits), safer deployments (deploy cell by cell), noisy-neighbor isolation. Costs: routing layer complexity, cross-cell operations (migrating tenants, global queries), more infrastructure to manage.

**Q4. How do you deploy safely across many regions/cells?**
Progressive **wave-based** deployments: one-box/canary in a single cell → one AZ/cell → one region → remaining regions in increasing waves, with bake times and automated rollback on alarms between waves; never deploy to all regions at once. Keep changes backward compatible, use feature flags for risky changes, and separate config deployments with the same discipline (AppConfig with validators and gradual rollout).

**Q5. Explain shuffle sharding.**
With 8 workers and simple sharding into 4 shards of 2 workers, a poison customer takes down 25% of customers. With shuffle sharding, each customer gets a random pair out of 8 workers (28 combinations), so a bad customer that kills its two workers affects only customers whose pair overlaps fully — a tiny fraction. Clients retrying across their shard members are mostly unaffected. Route 53 and many AWS services use this.

---

## 37. Distributed System Patterns on AWS

**Patterns frequently asked**
- **Idempotency:** Same request applied multiple times has the same effect. Idempotency keys stored in DynamoDB with conditional writes; Powertools idempotency utility.
- **Retries with exponential backoff + jitter:** Avoid retry storms; AWS SDKs have adaptive/standard retry modes; set **timeouts** everywhere; use **retry budgets/token buckets**.
- **Circuit breaker:** Stop calling a failing dependency, fail fast, probe for recovery (half-open).
- **Bulkhead:** Isolate resources (separate pools, queues, concurrency limits per dependency/tenant).
- **Load shedding & backpressure:** Reject excess work early (429) to protect goodput.
- **Transactional outbox:** Atomically write state + event, then publish (DynamoDB Streams/CDC → EventBridge).
- **Saga:** Distributed transactions via local transactions + compensations (Step Functions orchestration or choreography).
- **CQRS:** Separate write model (normalized/DynamoDB) and read models (OpenSearch, Redis, materialized views) updated via events.
- **Event sourcing:** Store state as an immutable sequence of events (Kinesis/DynamoDB/EventStore), rebuild state by replay.
- **Strangler Fig:** Incrementally replace a monolith by routing specific paths to new services (API Gateway/ALB path routing, Migration Hub Refactor Spaces).
- **Claim check, competing consumers, fan-out, priority queues, cache-aside, sidecar, ambassador, anti-corruption layer.**

### Interview Questions

**Q1. Why add jitter to exponential backoff?**
Without jitter, many clients that failed at the same time retry at the same intervals (1 s, 2 s, 4 s...), creating synchronized waves that re-overload the recovering service. Jitter (e.g. "full jitter": `sleep = random(0, min(cap, base × 2^attempt))`) spreads retries over time, dramatically reducing contention. Also cap retries, use retry budgets, and only retry idempotent/safe operations or with idempotency keys.

**Q2. How do you design an idempotent payment API?**
Client sends an `Idempotency-Key` header (UUID per logical operation). Server does a conditional put into an idempotency table (`attribute_not_exists`) with status `IN_PROGRESS`; on success stores the response with status `COMPLETED` and TTL (e.g. 24 h). Duplicate requests with the same key return the stored response; concurrent duplicates get 409/retry-later while in progress. Also validate the payload hash matches the original request to detect key misuse. Downstream calls (payment provider) also use idempotency keys.

**Q3. Explain the transactional outbox pattern and its AWS implementation.**
Problem: writing to the DB and publishing an event are two systems — one can fail ("dual write"). Solution: in the same DB transaction, write the business change and an event row to an outbox table. A separate relay publishes outbox rows to the broker and marks them sent (at-least-once → consumers must be idempotent). AWS: DynamoDB `TransactWriteItems` (state + outbox item) → DynamoDB Streams → EventBridge Pipes → EventBridge bus; for Aurora/RDS use CDC (DMS or Debezium on MSK Connect) from the outbox table.

**Q4. How does the Strangler Fig pattern work in a monolith-to-microservices migration?**
Put a routing façade (API Gateway, ALB rules, or CloudFront) in front of the monolith. Extract one bounded context at a time into a new service, route its paths to the new service, keep the rest to the monolith. Sync data via events/CDC during transition, use an anti-corruption layer to translate models. Gradually the monolith shrinks until it can be retired. Low risk, incremental, reversible.

**Q5. What is CQRS and when is it worth the complexity?**
Command Query Responsibility Segregation separates writes (commands, validated domain model) from reads (queries served by denormalized, purpose-built read models). Worth it when read and write workloads differ greatly in shape/scale (e.g. complex search over orders stored in DynamoDB → stream to OpenSearch), or with event sourcing. Cost: eventual consistency between write and read models, more infrastructure, sync pipelines. Overkill for simple CRUD.

**Q6. How do you prevent cascading failures in microservices?**
Timeouts on every call (shorter than callers' timeouts), retries with backoff/jitter and budgets, circuit breakers, bulkheads (separate connection pools/concurrency limits per dependency), load shedding, async queues to absorb spikes, graceful degradation (fallbacks/cached responses), avoid deep synchronous call chains, and health checks that don't fail on dependency issues.

---

## 38. Cost Optimization & FinOps

**What it is:** Practices and tools to understand, allocate, and optimize cloud spend continuously — engineering owns cost as a non-functional requirement.

**Tools:** Cost Explorer, **AWS Budgets** (alerts/actions), **Cost and Usage Report (CUR 2.0) / Data Exports** + Athena/QuickSight, Cost Anomaly Detection, **Compute Optimizer** (rightsizing EC2/EBS/Lambda/ECS/RDS), Trusted Advisor, **Cost Optimization Hub**, Savings Plans/RI recommendations, cost allocation **tags**, Cost Categories, Billing Conductor, Pricing Calculator.

**Main levers**
1. **Right-size** (Compute Optimizer), turn off non-prod off-hours (Instance Scheduler).
2. **Pricing models:** Savings Plans/RIs for baseline, Spot for flexible.
3. **Modernize:** Graviton, serverless, managed services, gp3, Aurora I/O-Optimized when I/O-heavy.
4. **Storage:** S3 lifecycle/Intelligent-Tiering, delete unattached EBS volumes & old snapshots.
5. **Data transfer:** VPC endpoints vs NAT, keep traffic in-AZ, CloudFront for egress (cheaper than direct), compression.
6. **Architecture:** caching, async/batch, autoscaling, scale to zero.
7. **Governance:** tagging policy, budgets per team, showback/chargeback, unit economics (cost per order/user).

### Interview Questions

**Q1. Your AWS bill grew 40% last month. How do you investigate?**
Cost Explorer grouped by service → linked account → usage type → tags to pinpoint the delta; Cost Anomaly Detection alerts; CUR in Athena for resource-level detail. Common culprits: NAT Gateway data processing, cross-AZ/region data transfer, CloudWatch Logs ingestion, forgotten resources (dev environments, GPU instances, unattached EBS, idle load balancers), on-demand capacity growth without SP coverage, a misconfigured autoscaler or recursive Lambda loop. Then fix, and add guardrails (budgets, alarms, tagging enforcement).

**Q2. How do you reduce costs by 30% without hurting reliability?**
Quick wins: delete waste (idle/unattached), rightsize via Compute Optimizer, gp2→gp3, log retention, S3 lifecycle. Commitments: Compute Savings Plans for steady baseline (often 20–40% alone). Architecture: Graviton migration, Spot for stateless/batch/CI, VPC endpoints to cut NAT, caching to reduce DB size, scheduling non-prod off nights/weekends (~65% saving on those). Track **unit cost** and validate SLOs unchanged after each change.

**Q3. How do you implement cost accountability across many teams?**
Mandatory tags (`team`, `service`, `env`, `cost-center`) enforced with tag policies/SCPs and IaC checks; account-per-team-per-env structure (cleanest allocation); Cost Categories; per-team budgets with alerts; monthly showback dashboards (CUR + QuickSight); include cost in architecture reviews and in CI (Infracost for Terraform PRs); share commitment savings fairly.

**Q4. What hidden costs do engineers often miss?**
Data transfer (inter-AZ $0.01/GB each way, inter-region, internet egress), NAT Gateway processing, public IPv4 addresses, CloudWatch Logs ingestion and custom metrics cardinality, KMS requests, S3 request costs for many small objects, idle provisioned resources (provisioned concurrency, RDS instances), cross-region replication, and GenAI token usage (long prompts, no caching).

**Q5. How do you control GenAI (Bedrock) costs?**
Choose the smallest model meeting quality (model routing/cascades), **prompt caching** for repeated context, trim prompts and RAG context (better retrieval, reranking), cap `max_tokens`, **batch inference** (~50% cheaper) for offline jobs, provisioned throughput only for steady high volume, response caching for repeated queries (semantic cache), distillation to smaller models, application inference profiles with tags for per-team cost allocation, and budgets/quotas per tenant.

---

## 39. Migration Strategies (The 7 Rs)

**The 7 Rs**
1. **Retire** — decommission what's no longer needed.
2. **Retain** — keep on-prem for now (compliance, recent investment, not ready).
3. **Rehost** ("lift and shift") — move VMs as-is (AWS Application Migration Service — MGN).
4. **Relocate** — move at hypervisor level (VMware Cloud on AWS / Elastic VMware Service) without changes.
5. **Repurchase** ("drop and shop") — switch to SaaS (e.g. CRM → Salesforce).
6. **Replatform** ("lift, tinker and shift") — minor optimizations (self-managed DB → RDS, app → Beanstalk/containers).
7. **Refactor / Re-architect** — redesign as cloud-native (microservices, serverless) for agility/scale.

**Migration services:** Migration Hub, Application Discovery Service, **Application Migration Service (MGN)**, **Database Migration Service (DMS)** + Schema Conversion (SCT / DMS Schema Conversion with GenAI), DataSync, Snow Family (legacy, offline transfer), Transfer Family, **AWS Transform** (agentic AI service for mainframe, VMware, .NET modernization — 2025).

**Phases:** Assess (business case, discovery) → Mobilize (landing zone, skills, pilot) → Migrate & Modernize (waves).

### Interview Questions

**Q1. Explain the 7 Rs and when you'd use each.**
(See list.) Typically a portfolio mix: retire 10–20% of apps found unused, rehost most to exit data centers quickly (deadline-driven), replatform databases to managed services for quick ops wins, refactor only the strategic, high-change apps where cloud-native benefits justify effort, repurchase commodity apps, retain those with blockers.

**Q2. Why rehost first and modernize later?**
Speed to exit data center contracts, lower risk, quick cost/ops benefits, and teams build cloud skills; once in AWS, modernization is easier (data close by, managed services available). Risk: "lift and shift and forget" — carry inefficiencies into cloud and higher costs if not optimized afterward. Plan modernization waves explicitly.

**Q3. How do you migrate a large production database with minimal downtime?**
Full load + **CDC** using DMS (or native replication: MySQL binlog replication, PostgreSQL logical replication) into RDS/Aurora; for heterogeneous migrations (Oracle → PostgreSQL), convert schema with DMS Schema Conversion first, then remediate code. Validate data (DMS validation), run app tests against target, plan a short cutover window: stop writes, let CDC catch up, switch connection strings (DNS/secrets), keep reverse replication ready for rollback.

**Q4. How do you migrate 500 TB of data to S3?**
Calculate network time: 500 TB over 10 Gbps ≈ ~5 days at full utilization (practically 1–2 weeks). Options: **DataSync** over Direct Connect (parallel, verifies integrity, incremental), S3 Transfer Acceleration for distributed sources, or physical transfer devices for constrained bandwidth (Snowball availability has been reduced — check current offering). Then incremental syncs before cutover.

**Q5. What is AWS Transform?**
An agentic AI service (2025) that automates large-scale modernization: mainframe COBOL analysis/decomposition and refactoring to Java, VMware workload migration planning (network conversion, wave planning), .NET Framework → cross-platform .NET porting, and custom code transformations — reducing months of manual effort; humans review and approve agent plans.

---

## 40. Shared Responsibility Model

**What it is:** Security and compliance split: **AWS is responsible for security "OF" the cloud** (physical data centers, hardware, network, hypervisor, managed service infrastructure); **the customer is responsible for security "IN" the cloud** (data, IAM, OS patching on EC2, application security, network configuration, encryption choices).

The line shifts by service type:
- **IaaS (EC2):** Customer patches guest OS, configures firewall/SGs, installs agents, manages data.
- **Containers (ECS on EC2 vs Fargate):** Fargate shifts host OS patching to AWS.
- **Managed/abstracted (S3, DynamoDB, Lambda):** AWS manages OS/platform; customer manages data, IAM, encryption settings, code.
- **GenAI (Bedrock):** AWS secures model hosting and infrastructure, doesn't use your data to train models; customer secures prompts, data access, guardrail configuration, agent tool permissions, and output handling.

### Interview Questions

**Q1. Explain the shared responsibility model with examples.**
AWS secures facilities, hardware, network, virtualization and managed service infrastructure. You secure what you put in and configure: IAM, data classification/encryption, security groups, OS patching on EC2, application code, logging. Example: S3 durability is AWS's job; a publicly exposed bucket is yours.

**Q2. Who patches the OS for RDS? For EC2? For Lambda?**
RDS: AWS patches OS and DB engine (you choose maintenance windows and control major version upgrades). EC2: you (SSM Patch Manager helps). Lambda: AWS patches runtime and OS (managed runtimes); you update dependencies and migrate off deprecated runtimes.

**Q3. Is AWS responsible if data is leaked from a misconfigured S3 bucket?**
No — configuration of access controls is the customer's responsibility. AWS provides guardrails (Block Public Access default on new buckets, Access Analyzer, Macie, Config rules), but using them is your responsibility.

**Q4. How does the model apply to compliance (e.g. PCI DSS)?**
AWS provides compliance of the infrastructure (attestations available in **AWS Artifact**), and many services are in scope. You must still configure your workload to meet requirements (encryption, logging, access control, segmentation) and get your own assessment. Inherited controls reduce your scope but don't eliminate it.

**Q5. What changes with serverless and AI services?**
More responsibility shifts to AWS (no OS/runtime management), but new customer responsibilities appear: function permissions (over-privileged Lambda roles), event-data injection, dependency vulnerabilities, and for GenAI — prompt injection defenses, guardrails, data access controls for RAG (users should only retrieve documents they're entitled to), and human oversight of agent actions.

---

## 41. CloudFormation

**What it is:** AWS-native **Infrastructure as Code** — declarative JSON/YAML templates defining resources; CloudFormation provisions them as **stacks**, managing dependencies, ordering, updates and rollbacks.

**Key concepts**
- **Template sections:** Parameters, Mappings, Conditions, Resources (only required section), Outputs, Transform (SAM/macros).
- **Intrinsic functions:** `!Ref`, `!GetAtt`, `!Sub`, `!Join`, `!If`, `!ImportValue`, `!FindInMap`.
- **Change sets:** Preview changes before applying (watch for **Replacement: True**).
- **Rollback:** Automatic on failure; rollback triggers (CloudWatch alarms); `--disable-rollback` for debugging.
- **Drift detection:** Detect manual changes.
- **Stack policies, DeletionPolicy (`Retain`, `Snapshot`), UpdateReplacePolicy**, termination protection.
- **Nested stacks** (reuse), **cross-stack references** (exports/imports), **StackSets** (deploy across accounts/regions — e.g. org baselines).
- **Custom resources** (Lambda-backed) and the **CloudFormation Registry** (resource types, modules, **hooks** for proactive compliance checks), Guard (policy-as-code).
- **IaC generator:** generate templates from existing resources; resource import.

### Interview Questions

**Q1. What is a change set and why use it in production?**
A change set shows what CloudFormation will do (add/modify/remove) and whether modifications require **replacement** (e.g. changing an RDS identifier or DynamoDB key schema replaces the resource → data loss). Reviewing change sets in the pipeline (manual approval for prod or automated checks for replacements of stateful resources) prevents destructive surprises.

**Q2. How do you protect stateful resources like databases in CloudFormation?**
`DeletionPolicy: Retain` or `Snapshot` and `UpdateReplacePolicy: Retain/Snapshot`, stack termination protection, **stack policies** to deny updates/replacement of specific logical IDs, keep stateful resources in separate stacks from frequently changing stateless ones, and review change sets.

**Q3. A stack is stuck in `UPDATE_ROLLBACK_FAILED`. What do you do?**
Find the root cause in stack events (often a resource modified/deleted outside CloudFormation, or a dependency like a missing IAM permission). Fix the underlying issue manually, then **ContinueUpdateRollback**, optionally skipping resources that can't be rolled back (`--resources-to-skip`) — then reconcile them later. Prevent with drift detection and no manual changes (SCPs/permissions).

**Q4. Cross-stack references vs nested stacks vs SSM parameters?**
Exports/imports: share outputs across independent stacks, but create hard coupling — you can't change/delete an exported value while it's imported. Nested stacks: compose reusable components deployed as one unit/lifecycle. SSM parameters: loosest coupling — producer writes values, consumers read at deploy time — no lock-in, but no dependency tracking. Prefer SSM/loose coupling for values shared across teams.

**Q5. How do you deploy the same baseline to 200 accounts?**
**StackSets** with service-managed permissions integrated with Organizations: target OUs, automatic deployment to new accounts, region concurrency and failure tolerance settings. (Or Control Tower customizations / AFT with Terraform.)

**Q6. CloudFormation vs Terraform?**
CloudFormation: AWS-native, managed state (no backend to manage), rollback on failure, day-0 support for many new features via the registry, StackSets, free. Terraform: multi-cloud and SaaS providers (Datadog, GitHub, Cloudflare), HCL is concise, rich module ecosystem, `plan` output, mature workflows (Terraform Cloud/Atlantis), but you manage state and locking, and no automatic rollback. Choose based on org standards and multi-provider needs.

---

## 42. AWS CDK

**What it is:** **Cloud Development Kit** — define infrastructure in general-purpose languages (TypeScript, Python, Java, C#, Go). CDK **synthesizes** CloudFormation templates and deploys them.

**Key concepts**
- **Constructs:** Building blocks. **L1** (`Cfn*` — 1:1 CloudFormation resources), **L2** (curated, sensible defaults, helper methods like `bucket.grantRead(fn)`), **L3 / patterns** (multi-resource patterns, e.g. `ApplicationLoadBalancedFargateService`).
- **App → Stacks → Constructs** tree; **Stages** for environments.
- **Bootstrapping:** `cdk bootstrap` creates S3/ECR assets and deployment roles per account/region.
- **Commands:** `cdk synth`, `cdk diff`, `cdk deploy`, `cdk watch`/hotswap (dev only).
- **CDK Pipelines:** Self-mutating CI/CD pipeline construct.
- **Aspects:** Visit all constructs to apply/validate rules (e.g. enforce encryption, tags), **cdk-nag** for compliance checks.
- **Testing:** Fine-grained assertions (`Template.fromStack().hasResourceProperties`) and snapshot tests.
- **CDK for Terraform (CDKTF)** — deprecated in 2025; Construct Hub for sharing libraries. Refactor support for logical ID changes (2025).

### Interview Questions

**Q1. What are L1, L2, L3 constructs?**
L1 (`CfnBucket`) maps exactly to CloudFormation — full control, no defaults. L2 (`s3.Bucket`) adds intent-based APIs, secure defaults and glue (`grantRead` generates least-privilege IAM policies). L3 patterns compose multiple resources into an architecture (e.g. ALB + Fargate service + logging). You can drop to L1 via escape hatches (`node.defaultChild`) to set properties not exposed by L2.

**Q2. What are the benefits and risks of CDK over raw CloudFormation/Terraform?**
Benefits: real programming language (loops, types, IDE autocomplete, refactoring), reusable abstractions shared as libraries, less boilerplate, generated least-privilege IAM via grants, unit testing. Risks: abstractions hide what's created (always review `cdk diff`), changing construct IDs/paths changes logical IDs → resource **replacement**, non-deterministic code (calling APIs at synth time) breaks reproducibility, and teams can over-engineer abstractions.

**Q3. How do you prevent accidental resource replacement when refactoring CDK code?**
Logical IDs derive from the construct path; moving/renaming constructs changes them → CloudFormation deletes and recreates. Mitigate: run `cdk diff` in CI and fail on replacement of stateful resources, use `overrideLogicalId` to preserve IDs, `RemovalPolicy.RETAIN` on stateful resources, and newer `cdk refactor` support for mapping old→new IDs.

**Q4. How do you enforce security standards across all CDK apps?**
Publish an internal construct library with secure defaults (encrypted buckets, private by default, logging) — e.g. `MyCompanyBucket`; apply **Aspects** to validate/enforce (fail synth if an S3 bucket isn't encrypted, auto-add tags/permission boundaries); run **cdk-nag** rule packs (AWS Solutions, HIPAA, NIST) in CI; CloudFormation Guard/hooks as server-side enforcement.

**Q5. How do you structure CDK for multiple environments and accounts?**
One app with **Stages** (Dev, Staging, Prod) each instantiating the same stacks with environment-specific props (account/region, sizes), deployed by **CDK Pipelines** or GitHub Actions with OIDC into separate accounts (cross-account deploy roles trusted via bootstrap `--trust`). Keep config in typed objects, avoid context lookups at deploy time for reproducibility (commit `cdk.context.json`).

---

## 43. AWS SAM

**What it is:** **Serverless Application Model** — an extension of CloudFormation (`Transform: AWS::Serverless-2016-10-31`) with shorthand resource types for serverless apps, plus the **SAM CLI** for local development, testing and deployment.

**Key concepts:** `AWS::Serverless::Function`, `::Api`, `::HttpApi`, `::SimpleTable`, `::StateMachine`, `::LayerVersion`, `::Connector` (generates IAM permissions between resources); policy templates (`DynamoDBCrudPolicy`); `sam build`, `sam local invoke`/`start-api` (Docker-based emulation), `sam deploy --guided`, `sam sync --watch` (fast cloud dev), `sam logs`, `sam pipeline init`; built-in **gradual deployments** (`DeploymentPreference: Canary10Percent5Minutes`) via CodeDeploy with alarms and pre/post traffic hooks.

### Interview Questions

**Q1. SAM vs CDK vs Serverless Framework?**
SAM: CloudFormation-native YAML with serverless shortcuts, great local testing and simple learning curve — best for Lambda-centric apps. CDK: general-purpose programming language, better for complex infrastructure and reusable abstractions (can also be used with SAM CLI for local testing). Serverless Framework: multi-cloud, plugin ecosystem, but v4 requires licensing for larger orgs. Choose based on team language preference and infrastructure complexity.

**Q2. How do you do canary deployments of Lambda with SAM?**
Set `AutoPublishAlias: live` and `DeploymentPreference` (e.g. `Canary10Percent5Minutes` or `Linear10PercentEvery1Minute`) with CloudWatch `Alarms` and optional `PreTraffic`/`PostTraffic` hook functions. CodeDeploy shifts alias weights between versions and rolls back automatically if alarms fire.

**Q3. How do you test Lambda functions locally with SAM, and what are the limitations?**
`sam local invoke` with event payloads (`sam local generate-event`), `sam local start-api` for API Gateway emulation, `sam local start-lambda` for SDK-based tests, debugging via IDE attach. Limitations: emulation isn't exact (IAM, VPC, event source behavior, service integrations aren't emulated) — so prefer unit tests with mocks + integration tests against real AWS in an ephemeral stack (`sam sync` to a dev stack).

**Q4. What are SAM Connectors?**
`AWS::Serverless::Connector` declares relationships ("this function reads/writes this table") and SAM generates least-privilege IAM policies (and resource policies where needed) automatically — less error-prone than hand-written policies.

**Q5. How does `sam sync` speed up development?**
It bypasses full CloudFormation deployments for code-only changes, updating Lambda code/Step Functions definitions/API definitions directly via service APIs, and with `--watch` syncs on file save. Only use in dev stacks — production should always go through CloudFormation deployments.

---

## 44. CodePipeline, CodeBuild, CodeDeploy

**What they are:**
- **CodePipeline:** Managed CI/CD orchestration — stages (Source → Build → Test → Deploy) with actions; V2 pipelines support triggers on git tags/branches/file paths, variables, stage-level conditions and automatic rollback.
- **CodeBuild:** Managed build service running `buildspec.yml` in containers (or Lambda compute, reserved fleets, GPU); pay per build minute; can act as **self-hosted GitHub Actions runners**.
- **CodeDeploy:** Automated deployments to EC2/on-prem, Lambda, and ECS with in-place or **blue/green**, traffic shifting (canary/linear/all-at-once), lifecycle hooks (`appspec.yml`), and alarm-based automatic rollback.
- (CodeCommit was closed to new customers in 2024 — use GitHub/GitLab/Bitbucket via CodeConnections. CodeArtifact provides package repositories — npm, PyPI, Maven.)

### Interview Questions

**Q1. Design a CI/CD pipeline for a containerized microservice on ECS.**
Source (GitHub via CodeConnections) → CodeBuild: lint, unit tests, SAST, build image, scan (Inspector/Trivy), push to ECR with commit SHA tag, generate SBOM → deploy to Dev (CloudFormation/CDK) → integration tests → Staging → automated/manual approval → Prod via **CodeDeploy ECS blue/green** with canary traffic shifting and CloudWatch alarm rollback. Cross-account deploy roles per environment, artifacts encrypted with KMS. (Equivalent in GitHub Actions + OIDC is common in practice.)

**Q2. How does CodeDeploy blue/green for ECS work?**
CodeDeploy creates a replacement (green) task set with the new task definition, registers it to a second target group, optionally routes a **test listener** to it for validation hooks (`BeforeAllowTraffic`), then shifts production listener traffic (canary e.g. 10% for 5 min, linear, or all-at-once). It monitors CloudWatch alarms and rolls back by rerouting traffic to blue if alarms trigger, then terminates the old task set after a wait time.

**Q3. CodeDeploy lifecycle hooks for EC2?**
`ApplicationStop → DownloadBundle → BeforeInstall → Install → AfterInstall → ApplicationStart → ValidateService` (plus `BeforeBlockTraffic/BlockTraffic/AfterBlockTraffic` and `BeforeAllowTraffic/AllowTraffic/AfterAllowTraffic` with load balancers). Scripts in `appspec.yml` handle stopping services, migrations, health validation.

**Q4. How do you deploy across multiple AWS accounts securely from a central pipeline?**
Pipeline runs in a tooling/CI account. Each target account has a **deploy role** trusting the tooling account's pipeline role (with conditions). Artifact bucket and KMS key policies allow target-account roles to read artifacts. The CloudFormation action assumes the target role with a CloudFormation execution role scoped to what that stack needs. No long-lived credentials.

**Q5. How do you speed up slow CodeBuild builds?**
Enable caching (local Docker layer cache, S3 cache for dependencies), use larger compute or ARM/Graviton builds, run test suites in parallel (batch builds / build matrix), use custom pre-baked build images (with dependencies) in ECR, reserved capacity fleets to avoid provisioning time, and only build what changed in monorepos.

---

# PART 2 — AI / GENERATIVE AI ON AWS

---

## 45. Amazon Bedrock

**What it is:** Fully managed, serverless service to build GenAI applications using **foundation models (FMs)** from many providers — Anthropic (Claude), Amazon (Nova, Titan), Meta (Llama), Mistral, Cohere, AI21, DeepSeek, Qwen, OpenAI open-weight models, Stability AI, and more — through a single API, without managing infrastructure.

**Purpose:** Access state-of-the-art models securely inside your AWS account boundary (data isn't used to train models, stays in region unless you opt into cross-region inference, private connectivity via PrivateLink), with built-in tools for RAG, agents, guardrails, evaluation, and customization.

**Key concepts**
- **APIs:** `InvokeModel` (model-specific payloads), **`Converse` / `ConverseStream`** (unified, model-agnostic chat API with tool use, system prompts, multimodal input — preferred), streaming responses, OpenAI-compatible endpoints for some models, Bedrock API keys for quick starts (IAM SigV4 preferred).
- **Inference options:**
  - *On-demand* — pay per input/output token.
  - *Cross-region inference profiles* (geo, e.g. `us.`/`eu.` prefixes, and *global*) — route requests across regions for higher throughput/resilience.
  - *Provisioned Throughput* — reserved model units for consistent high volume (required for some custom models).
  - *Batch inference* — async jobs from S3, ~50% cheaper.
  - *Service tiers* (Priority / Standard / Flex) — trade latency priority vs price.
- **Prompt caching:** Cache long, repeated prompt prefixes (system prompts, documents, tool definitions) → lower latency and cost on cache hits.
- **Intelligent Prompt Routing:** Routes requests between models in a family (e.g. smaller vs larger) based on predicted quality/cost.
- **Model customization:** Fine-tuning (labeled data), continued pre-training (unlabeled domain data), **reinforcement fine-tuning**, **distillation**, **Custom Model Import** (bring your own open-weight model weights), Bedrock Marketplace (100+ specialized models on managed endpoints).
- **Tool use (function calling):** Model returns structured tool calls, your code executes and returns results.
- **Security:** IAM (model-level permissions, `bedrock:InvokeModel` on specific model ARNs), VPC endpoints, KMS encryption of customizations/logs, CloudTrail, **model invocation logging** to S3/CloudWatch, no data used for training.
- **Ecosystem:** Knowledge Bases, Agents, AgentCore, Guardrails, Evaluations, Prompt Management, Flows, Data Automation (multimodal document/video/audio extraction — IDP).

**Uses:** Chatbots & assistants, RAG over enterprise documents, summarization, content generation, code generation, document processing (IDP), classification/extraction, agentic workflows, multimodal (image/video understanding & generation).

### Interview Questions

**Q1. Why use Bedrock instead of calling a model provider's API directly or self-hosting?**
Bedrock gives: data stays within AWS with enterprise controls (IAM, VPC PrivateLink, KMS, CloudTrail, no training on your data), one API across many models (easy switching/A-B testing), consolidated billing and existing AWS commitments, integrated RAG/agents/guardrails/evals, and no GPU infrastructure. Self-hosting (SageMaker/EKS + vLLM) makes sense for open-weight models at very high steady volume, special latency/customization needs, or full control; direct provider APIs may get newest features first. Trade-off discussion is what interviewers want.

**Q2. `InvokeModel` vs `Converse` API?**
`InvokeModel` takes a model-specific request body — switching models means rewriting payloads. `Converse` provides a **uniform message format** (roles, content blocks for text/image/document, system prompts, `toolConfig` for tool use, inference config) across supported models, so swapping models is a model ID change. Use `ConverseStream` for token streaming to improve perceived latency. Use `InvokeModel` only for model-specific features not yet in Converse.

**Q3. You're getting `ThrottlingException` from Bedrock in production. How do you fix it?**
Check service quotas (tokens per minute / requests per minute per model per region) and request increases; use **cross-region inference profiles** (geo or global) to spread load across regions; implement retries with exponential backoff and jitter; smooth traffic with a queue (SQS) for non-interactive workloads; use **batch inference** for offline jobs; reduce token usage (prompt caching, shorter context, lower `max_tokens`); consider Provisioned Throughput or a priority service tier for guaranteed capacity; fall back to an alternative model if the primary is throttled.

**Q4. How do you choose a model for a use case?**
Define requirements: quality on your task, latency (time-to-first-token, tokens/sec), cost per request, context window, modality (vision, documents), tool-use reliability, language support, data residency. Build an **evaluation dataset** from real examples, run candidates via Bedrock Evaluations (automatic metrics + LLM-as-judge + human review), compare quality vs cost/latency, and pick the smallest model meeting the quality bar. Often use **routing**: small, fast model for simple requests, frontier model for complex reasoning.

**Q5. How do you secure a Bedrock-based application?**
Least-privilege IAM restricting allowed model IDs/inference profiles (and SCPs denying unapproved models), PrivateLink endpoints, invocation logging (with care for PII) and CloudTrail, **Guardrails** for input/output filtering and PII redaction, defense against **prompt injection** (separate system instructions from untrusted content, don't let model output directly drive privileged actions, validate tool arguments, human approval for destructive actions), per-user authorization in RAG retrieval (metadata filters), KMS encryption, rate limits per tenant, and output encoding before rendering (XSS).

**Q6. What is prompt caching and when does it help?**
Marks a prefix of the prompt (system prompt, long documents, few-shot examples, tool definitions) as cacheable; subsequent requests with the identical prefix within the cache TTL (~5 minutes, refreshed on hit) reuse the processed state — reducing input token cost substantially (cache reads are heavily discounted) and latency (faster time-to-first-token). Helps multi-turn chat, agents with large tool lists, Q&A over the same document. Put stable content first and variable content (user question) last.

**Q7. Fine-tuning vs RAG vs prompt engineering — when do you use each?**
Prompt engineering first: cheapest, fastest iteration (instructions, few-shot, structured output). **RAG** when the model needs **knowledge** that's private, large, or changes frequently — grounds answers with citations, data updated without retraining, access control at retrieval. **Fine-tuning/distillation** when you need to change **behavior/style/format** consistently, teach domain-specific tasks, or make a smaller model match a bigger one for cost/latency. Fine-tuning is poor for injecting frequently changing facts. Often combined: RAG + a fine-tuned smaller model.

---

## 46. Bedrock Knowledge Bases (Managed RAG)

**What it is:** Fully managed **Retrieval-Augmented Generation** — connects FMs to your data sources, handling ingestion, parsing, chunking, embedding, vector storage, retrieval, and prompt augmentation with citations.

**Key concepts**
- **Data sources:** S3, web crawler, Confluence, SharePoint, Salesforce, custom connectors (direct ingestion API); sync jobs for incremental updates.
- **Parsing:** Default text parsing, **foundation-model parsing** or **Bedrock Data Automation** for complex PDFs (tables, figures, images).
- **Chunking strategies:** Fixed-size (with overlap), **hierarchical** (parent/child — retrieve small chunks, return larger parent context), **semantic** (split by meaning), none (one doc per chunk), or **custom Lambda** chunking (e.g. LangChain/LlamaIndex splitters).
- **Embedding models:** Titan Text Embeddings V2, Cohere Embed, Nova Multimodal Embeddings; dimensions & binary embeddings to save cost.
- **Vector stores:** OpenSearch Serverless (default quick create), OpenSearch managed clusters, Aurora PostgreSQL (pgvector), **S3 Vectors**, Neptune Analytics (**GraphRAG**), Pinecone, Redis Enterprise, MongoDB Atlas.
- **APIs:** `Retrieve` (get chunks — you build the prompt) vs `RetrieveAndGenerate` (managed end-to-end with citations and session context).
- **Retrieval features:** Hybrid search (semantic + keyword), **metadata filtering** (e.g. by department, date, tenant), **reranking** models, query decomposition/reformulation, number of results.
- **Structured data retrieval:** Natural language → SQL against Redshift / Glue catalog.
- **Multimodal retrieval:** images/audio/video content.

**Uses:** Internal knowledge assistants (HR, IT, policies), customer support bots, document Q&A with citations, agent knowledge.

### Interview Questions

**Q1. Explain the RAG pipeline end to end.**
**Ingestion (offline):** load documents → parse (text, tables, OCR) → chunk → embed each chunk with an embedding model → store vectors + text + metadata in a vector index. **Query (online):** user question → (optional rewrite/decompose) → embed query → similarity (and/or keyword) search with metadata filters → rerank top-k → build prompt with instructions + retrieved chunks → LLM generates answer with citations → (guardrails, grounding check) → response. Knowledge Bases manages both halves.

**Q2. How do you choose a chunking strategy?**
Depends on document structure and questions. Fixed-size with overlap (~300–500 tokens, 10–20% overlap) is a solid baseline. **Hierarchical** when answers need broader context than small precise matches (retrieve child, send parent). **Semantic** for long unstructured prose where topics shift. Custom chunking for structured docs (split by headings/sections, keep tables intact, prepend section titles/document metadata to chunks). Evaluate with a retrieval test set (recall@k, MRR) rather than guessing.

**Q3. RAG answers are poor or hallucinated. How do you debug and improve?**
Separate retrieval from generation: check whether the right chunks are retrieved (log retrieved chunks; measure context recall/precision). Retrieval fixes: better parsing (tables/PDFs), chunking, hybrid search, metadata filters, query rewriting, more top-k + **reranker**, better embedding model, enrich chunks with titles. Generation fixes: stronger instructions ("answer only from context, say 'I don't know' otherwise"), citations required, **contextual grounding check** in Guardrails, better model. Use Bedrock **RAG evaluation** (faithfulness, correctness, context relevance) on a golden dataset and iterate.

**Q4. How do you enforce document-level permissions in RAG?**
Store ACL attributes as metadata (department, tenant ID, allowed groups) during ingestion; at query time, derive the user's entitlements from their authenticated identity (server-side, never from the prompt) and apply **metadata filters** in the `Retrieve` call so unauthorized chunks are never retrieved. For multi-tenant systems consider separate indexes/KBs per tenant for stronger isolation. Never rely on the LLM to "not reveal" restricted content.

**Q5. Which vector store would you choose and why?**
OpenSearch Serverless/managed: low-latency hybrid search, filtering, high QPS — production default. Aurora pgvector: data already in Postgres, need joins/transactions, moderate scale. **S3 Vectors**: very large, cost-sensitive corpora with relaxed latency, or tiering cold vectors. Neptune Analytics: GraphRAG when relationships between entities matter (multi-hop questions). Pinecone/MongoDB/Redis: existing org standards. Consider cost at scale, latency SLO, filtering needs, operational overhead.

**Q6. What is GraphRAG and when does it help?**
GraphRAG extracts entities and relationships from documents into a knowledge graph and combines graph traversal with vector search. It helps for multi-hop or relationship questions ("Which suppliers of products affected by recall X also supply our EU plants?") and global summarization across documents, where plain chunk similarity misses connections. Bedrock Knowledge Bases supports it with Neptune Analytics.

**Q7. How do you keep the knowledge base fresh?**
Schedule or event-trigger **sync jobs** (S3 event → EventBridge → Lambda `StartIngestionJob`) — ingestion is incremental (only changed/deleted docs re-embedded); use direct ingestion APIs for real-time updates; include `last_updated` metadata to prefer recent docs; remove deleted docs promptly (important for compliance/right-to-be-forgotten).

---

## 47. Bedrock Agents

**What it is:** Managed capability to build **agents** that use an FM to reason, plan multi-step tasks, call APIs (**action groups**), query Knowledge Bases, and maintain conversation context — configured rather than coded.

**Key concepts**
- **Instructions:** Natural-language role and behavior.
- **Action groups:** Tools defined via OpenAPI schema or function details, executed by a Lambda function (or **return of control** to your application).
- **Knowledge base association** for RAG.
- **Orchestration:** ReAct-style reasoning loop (or custom orchestration via Lambda); trace for each step (rationale, tool calls, observations).
- **Memory:** Session state and long-term summarized memory across sessions.
- **Code interpreter:** Run generated code in a sandbox.
- **User confirmation** before executing actions.
- **Multi-agent collaboration:** Supervisor agent routes/coordinates specialist sub-agents.
- **Aliases & versions** for deployment; guardrails attachable.

**Note:** For new, production-grade agent platforms AWS increasingly steers toward **AgentCore** (framework-agnostic runtime) — Bedrock Agents remains the low-code, fully managed option.

### Interview Questions

**Q1. How does a Bedrock agent process a request?**
Pre-processing (optional input validation/classification) → **orchestration loop**: the model reads instructions, conversation, available actions and KBs, decides the next step (call an action, query a KB, ask the user, or answer); the agent executes the tool via Lambda/return-of-control, feeds the observation back, and repeats until it can produce a final response → optional post-processing. Traces show each reasoning step for debugging.

**Q2. What is "return of control" and why use it?**
Instead of the agent invoking a Lambda, it returns the chosen action and parameters to your application, which executes it (e.g. in your existing service with user context, or after showing the user a confirmation UI) and sends the result back. Useful when actions need user-session credentials, run outside AWS, require long-running processing, or human approval.

**Q3. How do you prevent an agent from taking harmful or unintended actions?**
Least-privilege Lambda roles per action group, validate all tool parameters server-side (the model's output is untrusted input), enable **user confirmation** for state-changing actions, idempotent and reversible operations, limit available actions to what's needed, Guardrails for prompt attacks, authorization checks using the end-user's identity (not the agent's broad role), rate limits, and full tracing/audit.

**Q4. When would you use multi-agent collaboration?**
When a domain is too broad for one agent's instructions/tool list (tool selection accuracy drops with many tools): a supervisor routes to specialists (billing agent, technical support agent, order agent), each with focused instructions, tools, and KBs. Costs: more latency and tokens, harder debugging. Start with one agent and split when accuracy suffers.

**Q5. Bedrock Agents vs building your own agent with Strands/LangGraph on AgentCore?**
Bedrock Agents: fastest to build, low-code, fully managed, good for standard tool-calling assistants. Custom frameworks + AgentCore: full control over orchestration logic, custom memory, any model (even non-Bedrock), complex multi-agent graphs, framework portability, richer testing — at the cost of more code. Choose based on complexity and control needs.

---

## 48. Bedrock AgentCore

**What it is:** A set of modular, enterprise-grade services (GA October 2025) to **deploy and operate AI agents securely at scale**, working with **any framework** (Strands, LangGraph, CrewAI, LlamaIndex, OpenAI Agents SDK...) and **any model** (Bedrock or not).

**Components**
| Service | Purpose |
|---|---|
| **Runtime** | Serverless hosting for agents and MCP servers with **per-session microVM isolation**, long-running sessions (up to 8 hours), streaming, multimodal payloads, consumption-based pricing |
| **Memory** | Short-term (session events) and long-term memory (extracted facts, user preferences, summaries) with configurable strategies |
| **Gateway** | Converts APIs, Lambda functions, OpenAPI/Smithy specs and existing MCP servers into **MCP-compatible tools**; semantic tool search; handles auth to targets |
| **Identity** | Agent identity & credential management — inbound auth (who can invoke the agent; IdP integration) and outbound auth (OAuth tokens/API keys to call third-party tools on behalf of users) with a token vault |
| **Code Interpreter** | Sandboxed code execution for agents |
| **Browser** | Managed, isolated cloud browser for web-navigating agents |
| **Observability** | OpenTelemetry-based traces, metrics, dashboards in CloudWatch for agent steps, tool calls, token usage |
| **Policy** (preview) | Deterministic Cedar-based controls over which tools/actions agents may perform, evaluated in the Gateway outside the model |
| **Evaluations** (preview) | Continuous quality evaluation of agent behavior (correctness, helpfulness, tool selection) |

### Interview Questions

**Q1. What problem does AgentCore solve?**
Prototype agents are easy; production agents need secure isolation between user sessions, long-running execution, memory, identity and delegated auth to tools, tool discovery, observability, governance and scaling. Teams were building this undifferentiated infrastructure themselves. AgentCore provides these as managed, composable services while letting teams keep their framework and model choice.

**Q2. Why is session isolation important for agents, and how does AgentCore Runtime provide it?**
Agents execute code, browse, hold user data and credentials in memory — sharing a process between users risks data leakage across sessions and cross-tenant attacks via prompt injection. Runtime runs each session in its own **dedicated microVM** with isolated CPU, memory and file system, destroyed/sanitized at session end — similar to Lambda's Firecracker model but for long-lived sessions.

**Q3. What is AgentCore Gateway and how does it relate to MCP?**
Gateway is a managed tool layer: register targets (Lambda, REST APIs via OpenAPI, Smithy models, SaaS, existing MCP servers) and it exposes them through a single **MCP endpoint** to agents. It handles protocol translation, authentication to the target (via AgentCore Identity), and semantic search to pick relevant tools from hundreds — reducing prompt size and tool-selection errors. It lets you "MCP-enable" existing enterprise APIs without rewriting them.

**Q4. How do agents call third-party APIs on behalf of a user securely?**
Via **AgentCore Identity**: the user authenticates to the app (inbound auth via your IdP); when the agent needs, e.g., the user's Google Calendar or Salesforce, Identity runs the OAuth flow (user consent) and stores tokens in a secure token vault scoped to agent + user; the agent retrieves short-lived tokens at call time. The agent never holds long-lived secrets and acts with the user's delegated permissions, not a super-user service account.

**Q5. How would you deploy a LangGraph or Strands agent to production on AWS?**
Containerize (or use the starter toolkit) with an entrypoint handler; deploy to **AgentCore Runtime** (with IAM execution role, inbound auth via Cognito/OIDC); use **Memory** for conversation history/long-term preferences; expose enterprise tools through **Gateway**; outbound credentials via **Identity**; enable **Observability** (OTel traces into CloudWatch); attach **Guardrails** to model calls and Policy to tool access; CI/CD with versioned endpoints, evaluation suites as release gates, and canary traffic between agent versions.

**Q6. How do you observe and debug agents in production?**
Trace every step (model calls with token counts/latency, tool calls with inputs/outputs, retries, errors) using OpenTelemetry GenAI conventions → AgentCore Observability/CloudWatch dashboards; log reasoning traces (with PII redaction); track metrics like task success rate, tool error rate, steps per task, cost per session; capture user feedback; run offline evaluations on recorded sessions; alert on anomalies (loops, runaway token usage).

---

## 49. Bedrock Guardrails

**What it is:** Configurable safeguards applied to model **inputs and outputs** to implement responsible AI policies — usable with any Bedrock model, Agents, Knowledge Bases, and via the standalone **`ApplyGuardrail` API** for any model (including self-hosted or third-party).

**Policies**
- **Content filters:** Hate, insults, sexual, violence, misconduct, and **prompt attacks** (jailbreaks, prompt injection) with configurable strengths; text and image.
- **Denied topics:** Natural-language topic definitions to block (e.g. "investment advice").
- **Word filters:** Profanity and custom word lists.
- **Sensitive information filters:** Detect PII types (email, SSN, credit card...) and custom regex → **block or mask/anonymize**.
- **Contextual grounding checks:** Score response **grounding** (supported by the source) and **relevance** (answers the query) — reduces hallucinations in RAG.
- **Automated Reasoning checks:** Validate responses against formal logic rules derived from your policy documents (mathematically verifiable — e.g. HR leave policy rules).
- Tiers (Classic/Standard) with multilingual support; guardrail versions; cross-region guardrail profiles.

### Interview Questions

**Q1. How do Guardrails work in the request flow?**
Input assessment runs on the user prompt before the model call — if blocked, a configured blocked message is returned and the model isn't invoked (no inference cost). If allowed, the model generates; then output assessment runs on the response (content filters, PII masking, grounding check) — blocked or masked as configured. Traces show which policy triggered. With streaming, output can be evaluated in chunks (sync or async mode trade-offs).

**Q2. How do you defend against prompt injection?**
Defense in depth: Guardrails **prompt attack filter** on user input (and tag untrusted content — e.g. retrieved documents, tool outputs — so filters evaluate it as user content), clear separation of system instructions from user/untrusted data, least-privilege tools with server-side validation and authorization, human confirmation for sensitive actions, never put secrets in prompts, output validation (schemas), monitoring for anomalous tool use, and red-team testing. No single filter is 100% effective — limit the blast radius.

**Q3. How do you handle PII in a GenAI application?**
Sensitive information filters to **mask** PII in inputs before they reach the model or logs and in outputs before reaching users (or block entirely for certain types); redact PII before indexing documents for RAG if not needed; configure invocation logging carefully (disable or protect logs with KMS + restricted access); data classification with Macie; regional data residency via geo inference profiles; retention policies.

**Q4. What is the contextual grounding check?**
For RAG/summarization, the guardrail receives the source context, query, and model response; it scores **grounding** (is each claim supported by the source?) and **relevance** (does it answer the question?). Responses below configured thresholds are blocked or flagged — catching hallucinations that aren't in the retrieved documents. Tune thresholds on an evaluation set to balance false positives.

**Q5. Why use `ApplyGuardrail` independently of model invocation?**
To apply the same centrally governed safety policy to any model (self-hosted on SageMaker/EKS, other providers), to check content at other points (tool outputs, retrieved documents, user uploads), or to validate content in non-LLM flows (moderating user-generated comments). It decouples safety policy from the model.

---

## 50. Bedrock Evaluations, Model Distillation, Prompt Management, Flows

**What they are:**
- **Bedrock Evaluations:** Evaluate models and RAG systems — *automatic* (accuracy, robustness, toxicity with built-in/custom datasets), **LLM-as-a-judge** (correctness, completeness, faithfulness, helpfulness, harmfulness, custom metrics), *human evaluation* (your team or AWS-managed workforce), **RAG evaluation** for Knowledge Bases or any RAG source (retrieval: context relevance/coverage; generation: correctness, faithfulness, citation precision).
- **Model Distillation:** Uses a larger "teacher" model to generate responses for your prompts, then fine-tunes a smaller "student" model — achieving close-to-teacher accuracy for your use case at much lower cost/latency.
- **Prompt Management:** Create, version, and share prompts with variables, model/inference config, and compare variants; reference prompt versions by ARN from apps/flows.
- **Bedrock Flows:** Visual workflow builder linking prompts, models, KBs, agents, Lambda, conditions, iterators — deploy as versioned, aliased APIs.
- **Bedrock Data Automation (BDA):** Extract structured insights from documents, images, audio, video (IDP) with blueprints.

### Interview Questions

**Q1. How do you evaluate an LLM application before production?**
Build a **golden dataset** of representative inputs with expected outputs/reference answers (including edge cases and adversarial prompts). Define metrics per use case (correctness, faithfulness, format compliance, safety, latency, cost). Run automatic metrics + **LLM-as-judge** (calibrated against human labels on a subset) + targeted human review. Make evaluation a **CI gate** — every prompt/model/retrieval change runs the suite, with regression thresholds. After launch, monitor with online signals (user feedback, sampling for judge scoring).

**Q2. What are the risks of LLM-as-a-judge and how do you mitigate them?**
Biases (position bias, verbosity preference, self-preference for same model family), inconsistency, and blind spots on domain correctness. Mitigate: clear rubrics with scoring criteria, reference answers, use a different/stronger judge model, randomize ordering in pairwise comparisons, calibrate against human ratings periodically, and multiple judgments/aggregation.

**Q3. When would you use model distillation?**
When a large model meets quality but is too slow or expensive at your volume, and your task is relatively narrow (classification, extraction, domain Q&A, routing). Distill into a smaller model of the same family, then evaluate against the teacher on held-out data. Not ideal when the task is broad/open-ended or changes frequently.

**Q4. Why manage prompts outside of code?**
Prompts are effectively code that changes behavior — versioning, review, A/B comparison and rollback are needed; non-engineers (domain experts) can iterate; the same prompt can be reused across services; deployments of prompt versions can be decoupled from app releases. Still keep them in source control/IaC for auditability and gate changes with evals.

**Q5. Bedrock Flows vs Step Functions vs agent frameworks?**
Flows: low-code, GenAI-centric pipelines (prompt chaining, conditions, KB lookups) — fast to build. Step Functions: general-purpose, durable orchestration with retries, long waits, human approvals, 200+ service integrations, plus direct Bedrock calls — better for business-critical workflows. Agent frameworks: when the path isn't predetermined and the model must decide steps dynamically. Prefer deterministic workflows when the steps are known — they're cheaper, faster, more reliable than autonomous agents.

---

## 51. Amazon Nova Models

**What it is:** Amazon's own family of foundation models available on Bedrock, focused on strong price-performance.

**Family (as of late 2025)**
- **Understanding models:** Nova **Micro** (text-only, lowest latency/cost), **Lite** (multimodal: text, image, video input; low cost), **Pro** (balanced accuracy/speed/cost), **Premier** (most capable, also a teacher for distillation). **Nova 2** generation (re:Invent 2025): Nova 2 Lite (reasoning with adjustable thinking), Nova 2 Pro, Nova 2 Omni (multimodal input & output), Nova 2 Sonic.
- **Creative models:** Nova **Canvas** (image generation/editing), Nova **Reel** (video generation).
- **Speech:** Nova **Sonic** — real-time speech-to-speech conversational model (voice agents) with tool use.
- **Embeddings:** Nova Multimodal Embeddings (text, image, video, audio into one space).
- **Nova Act:** Model + SDK/service for building reliable **browser-automation agents** (UI workflows).
- **Nova Forge:** Build custom models by blending your data into Nova training checkpoints ("open training").
- Customizable via fine-tuning/distillation on Bedrock and SageMaker.

### Interview Questions

**Q1. When would you choose a Nova model over other models in Bedrock?**
High-volume, cost-sensitive tasks where Nova Micro/Lite provides sufficient quality (classification, extraction, summarization, routing, simple RAG), low-latency needs, multimodal video understanding at low cost, AWS-native customization/distillation, or voice agents with Sonic. For the most complex reasoning/coding tasks, frontier models (e.g. Claude) may be preferred. The answer should be eval-driven: compare on your task.

**Q2. How would you build a voice customer-service agent on AWS?**
Real-time bidirectional audio stream (WebRTC/WebSocket, or Amazon Connect for telephony) → **Nova Sonic** speech-to-speech model handling turn-taking and interruptions, with tool use calling backend APIs (order lookup), RAG via Knowledge Bases for policies, guardrails, session memory, and escalation to human agents. Alternative pipeline: Transcribe (ASR) → LLM → Polly (TTS) — more components and latency but more control.

**Q3. What is Nova Act used for?**
Automating tasks in web browsers where APIs don't exist — filling forms, navigating portals, extracting data, QA testing. It breaks workflows into reliable atomic commands, supports Python for deterministic parts, and can run headless at scale. Production concerns: robustness to UI changes, credentials handling, human approval for sensitive steps, and site terms of service.

**Q4. How do you reduce cost in a GenAI pipeline using a model family like Nova?**
Cascade/routing: Micro or Lite for classification/simple requests, Pro/Premier only for complex ones; distill Premier → Lite for your specific task; batch inference for offline jobs; prompt caching; trim context. Evaluate quality at each tier.

**Q5. What's the difference between understanding, generation and embedding models?**
Understanding (LLM/multimodal) models take text/images/video and produce text (answers, reasoning, tool calls). Creative generation models produce images/video (Canvas, Reel). Embedding models map content to vectors for semantic search/clustering/RAG — not used for generating answers. A RAG system uses an embedding model for retrieval and an understanding model for generation.

---

## 52. Strands Agents SDK

**What it is:** AWS's **open-source** SDK (Python, and TypeScript) for building AI agents with a **model-driven** approach: you give the model a prompt and tools, and the model's reasoning drives the loop (plan → call tools → reflect) instead of hand-coded workflows. Used internally at AWS (e.g. Q Developer, Kiro, AWS Glue features).

**Key concepts**
- **Agent = model + tools + prompt.** Tools defined with a `@tool` decorator on Python functions (docstring → description), pre-built tools, or any **MCP server**.
- **Model providers:** Bedrock (default), Anthropic, OpenAI, LiteLLM, Ollama, Llama API, etc.
- **Multi-agent patterns:** Agents-as-tools, **Swarm** (self-organizing team), **Graph** (deterministic DAG), **Workflow**; **A2A** (Agent2Agent protocol) support.
- **Conversation management**, session persistence, structured output (Pydantic), hooks, streaming.
- **Observability** via OpenTelemetry; deploy to Lambda, Fargate, EKS, EC2, or **AgentCore Runtime**.

```python
from strands import Agent, tool

@tool
def get_order_status(order_id: str) -> str:
    """Return the shipping status for an order."""
    return orders_api.status(order_id)

agent = Agent(
    model="us.anthropic.claude-sonnet-4-5-20250929-v1:0",
    system_prompt="You are a helpful support agent.",
    tools=[get_order_status],
)
agent("Where is my order 12345?")
```

### Interview Questions

**Q1. What does "model-driven" agent development mean, versus workflow-driven?**
Workflow-driven frameworks hard-code the steps/graph and call the LLM at specific nodes. Model-driven relies on modern models' reasoning and tool-use ability: provide goals, instructions and tools; the model decides which tools to call and when, in a loop. Less code and more adaptable to varied requests, but less predictable — so you add guardrails, evaluation, and use deterministic Graph/Workflow patterns where steps are fixed.

**Q2. How does the agent loop work?**
Send system prompt + conversation + tool specs to the model → model returns either a final answer or tool-use requests → SDK executes tools (possibly in parallel) → append results as tool-result messages → call the model again → repeat until the model ends its turn (or max iterations/limits reached). Strands handles message formatting, tool execution, retries, streaming and metrics.

**Q3. How do you give a Strands agent access to existing enterprise tools?**
Wrap internal APIs as `@tool` functions, or connect to **MCP servers** (e.g. AWS MCP servers, company MCP servers, or tools exposed via AgentCore Gateway) using the MCP client — tools are discovered automatically. Keep tool descriptions precise, inputs strongly typed, and outputs concise to save tokens.

**Q4. When would you use multi-agent patterns (Swarm/Graph) vs a single agent?**
Single agent with focused tools first. Use **agents-as-tools** when sub-domains need specialized prompts/tools; **Graph/Workflow** when the process has known stages (research → draft → review) and needs determinism; **Swarm** for open-ended collaborative problem solving. Multi-agent adds latency, cost and debugging complexity — justify it with evaluation results.

**Q5. How do you take a Strands agent to production?**
Deploy to AgentCore Runtime (session isolation, scaling) or Lambda/Fargate; persist sessions (S3/AgentCore Memory); OTel traces to CloudWatch; Guardrails on model calls; least-privilege IAM for tools; limits on iterations/tokens/timeouts; evaluation suite in CI; versioned prompts; canary rollouts and cost dashboards.

---

## 53. Amazon Q Developer

**What it is:** AWS's generative AI assistant for software development and AWS operations — in IDEs (VS Code, JetBrains, Visual Studio, Eclipse), the command line, the AWS Console, chat apps (Slack/Teams), and GitHub/GitLab integrations.

**Capabilities**
- Inline code suggestions and chat in IDE; **agentic coding** (multi-file feature implementation, test generation, documentation, code review) with MCP support.
- **CLI agent** (Q Developer CLI — evolved into the **Kiro CLI** in late 2025): agentic coding and AWS operations in the terminal.
- **Security scanning** and auto-fix suggestions.
- **Code transformation:** Java version upgrades (8/11 → 17/21), .NET porting, mainframe — now largely under **AWS Transform**.
- **AWS expertise in the console:** Answer AWS questions, generate CLI commands, explain errors, cost analysis (Cost Explorer queries), network troubleshooting (Reachability Analyzer), and **operational investigations** of CloudWatch alarms (root-cause hypotheses).
- Tiers: Free and Pro (per user; IP indemnity, admin controls, higher limits, customization on private codebases).

### Interview Questions

**Q1. How would you roll out an AI coding assistant like Q Developer to an engineering org?**
Pilot with a few teams, define success metrics (PR cycle time, acceptance rates, developer satisfaction — not lines of code), configure org-level controls via Identity Center (Pro tier: IP indemnity, no training on your content, reference tracking for open-source code), set policies on what data/repos can be used for customization, keep code review and CI gates unchanged (AI code must pass the same tests/scans), provide training on effective prompting, then expand based on measured results.

**Q2. What are the risks of AI-generated code and how do you mitigate them?**
Subtle bugs, insecure patterns, outdated APIs, license contamination, overconfidence in reviews, and skill atrophy. Mitigations: mandatory human review and ownership, automated tests and coverage gates, SAST/SCA/secret scanning in CI, reference tracking for licensed code, small PRs, threat modeling for security-sensitive code, and treating agent changes like any contributor's.

**Q3. How can Q Developer help in operations/incident response?**
In the console it can investigate a CloudWatch alarm — correlating metrics, logs, traces, CloudTrail changes and deployments to suggest root-cause hypotheses and next steps; explain errors (e.g. AccessDenied causes), troubleshoot network paths, and answer cost questions in natural language. Engineers still verify and execute fixes.

**Q4. What is code transformation and how does it work for Java upgrades?**
The agent analyzes a project (Maven/Gradle), upgrades the language version and dependencies/frameworks (e.g. Spring Boot), rewrites deprecated APIs, builds and runs tests iteratively to fix compilation errors, then presents a diff for review. It saves large amounts of manual effort for portfolio-wide upgrades; the team still reviews, runs full test suites and performance checks.

**Q5. How do you ensure an AI assistant doesn't leak proprietary code?**
Use enterprise tiers with contractual guarantees (content not used for service improvement/training), admin-controlled opt-outs, Identity Center authentication, customization stored within your account controls, CloudTrail/usage logging, policies against pasting secrets, secret scanning, and network controls for which AI tools are allowed.

---

## 54. Kiro

**What it is:** AWS's **agentic IDE** (preview July 2025, GA November 2025; built on Code OSS/VS Code base) plus **Kiro CLI**, centered on **spec-driven development** — turning prompts into structured requirements, design, and tasks before code is written — aiming to move from "vibe coding" to production-quality software.

**Key concepts**
- **Specs:** For a feature, Kiro generates `requirements.md` (user stories with acceptance criteria in **EARS** notation — "WHEN … THE SYSTEM SHALL …"), `design.md` (architecture, data models, interfaces, sequence diagrams), and `tasks.md` (sequenced implementation tasks linked to requirements). Specs live in the repo and stay in sync with code.
- **Steering files:** `.kiro/steering/*.md` — persistent project context (tech stack, conventions, product goals) included in agent interactions.
- **Agent hooks:** Event-driven automations triggered by IDE events (file save/create/delete, manual) — e.g. update tests when a component changes, update docs, run security checks.
- **Vibe mode vs spec mode;** autopilot vs supervised execution.
- **MCP integration** for external tools/docs; **property-based testing** to verify code against spec requirements; checkpoints for rollback.
- **Kiro autonomous agent** (frontier agent — see next section).

### Interview Questions

**Q1. What is spec-driven development and why does it matter for AI coding?**
Instead of prompting an agent directly to write code ("vibe coding"), you first agree on **what** to build (requirements with acceptance criteria) and **how** (design), then break it into tasks the agent executes. Benefits: fewer misunderstandings, reviewable intent before code, traceability from requirement → task → code → test, better results on complex multi-file features, and documentation that persists for future changes and teammates. Cost: more upfront time for small changes — use vibe mode for quick fixes.

**Q2. What are steering files and agent hooks?**
Steering files give the agent persistent knowledge about your project (e.g. "use TypeScript strict, Jest for tests, DynamoDB single-table design, follow our API error format") so you don't repeat context. Hooks are automated agent actions on events — e.g. on saving a React component, update its test file; on adding an API route, update OpenAPI docs; on commit, scan for secrets — like CI checks running inside the IDE with an agent.

**Q3. Why is EARS notation used for requirements?**
Easy Approach to Requirements Syntax structures requirements into unambiguous, testable statements ("WHEN a user submits an invalid email THE SYSTEM SHALL display an error"). Testable statements make it easier to generate tests (including property-based tests) and to verify the implementation meets requirements.

**Q4. How do you keep quality high when an agent implements tasks autonomously?**
Supervised mode for critical code, small tasks with clear acceptance criteria, tests generated from requirements and run after each task, checkpoints to revert, hooks for linting/security, code review on the resulting PRs, and steering files encoding standards. Treat agent output like a junior engineer's PR.

**Q5. Kiro vs Q Developer vs other AI coding tools?**
Q Developer: assistant across IDE, console and AWS operations, strong AWS knowledge and transformation features. Kiro: a full IDE built around spec-driven agentic workflows, steering and hooks — focus on structured feature development. Other tools (Claude Code, GitHub Copilot, Cursor) offer similar agentic capabilities with different UX. In an interview, emphasize evaluation by team needs, governance, security posture and measured productivity rather than tool loyalty.

---

## 55. AWS Frontier Agents (DevOps Agent, Security Agent, Kiro Autonomous Agent)

**What they are:** A class of autonomous, long-running AI agents announced at re:Invent 2025 (preview) that work for hours/days on behalf of teams, with scale and persistent context:
- **AWS DevOps Agent:** "Always-on" on-call engineer — when an alarm/incident fires, it investigates autonomously across telemetry (CloudWatch, Datadog, Dynatrace, New Relic, Splunk), code repos and CI/CD (GitHub, GitLab), and ticketing (ServiceNow, PagerDuty), builds a topology of your application, identifies probable root cause, coordinates in Slack, and recommends mitigations plus longer-term prevention improvements.
- **AWS Security Agent:** Reviews design documents and code/PRs against your organization's security requirements, and performs on-demand context-aware **penetration testing** of applications.
- **Kiro autonomous agent:** Works independently on development tasks across multiple repositories (bug triage, features, upgrades), maintains context across sessions, learns from team feedback, and opens PRs for review.

### Interview Questions

**Q1. How would an AI DevOps agent change incident response, and what are the risks?**
It can reduce MTTD/MTTR by instantly correlating alarms, recent deployments, config changes, logs and traces — work that usually takes humans the first 30–60 minutes — and present ranked hypotheses with evidence. Risks: wrong conclusions presented confidently, over-reliance, access to sensitive data, and automated actions causing more damage. Mitigate: read-only investigation by default, humans approve mitigations, evidence-linked findings, audit trails, and measuring its accuracy over time.

**Q2. What permissions should an autonomous ops agent have?**
Start with **read-only** access to telemetry, code and deployment history using a dedicated role (least privilege, scoped to relevant accounts). Any write actions (rollback, scale, restart) go through existing guarded mechanisms (runbooks/SSM Automation with approvals, deployment pipeline rollbacks) rather than broad direct API access. Log every agent action; use SCPs/permission boundaries as hard limits.

**Q3. How do you evaluate whether an AI security agent's findings are trustworthy?**
Compare against known baselines (seeded vulnerabilities, past pentest reports), measure precision/recall and false positive rates, require reproducible evidence (request/response proofs for pentest findings), triage by humans before tickets are created, and run pentests only against authorized, non-production or explicitly approved targets.

**Q4. How do "frontier agents" differ from coding assistants?**
Assistants respond within a session and require continuous human prompting. Frontier agents are autonomous: given a goal, they work asynchronously for long periods, scale out across many tasks in parallel, keep persistent memory/context about your systems and team preferences, and integrate with team workflows (tickets, Slack, PRs). This demands stronger governance — scoped permissions, review gates, and observability of agent work.

**Q5. How would you integrate an autonomous coding agent into the SDLC safely?**
Assign well-scoped tickets with clear acceptance criteria; agent works on branches and opens PRs only (no direct merge to main); branch protection, required reviews, CI tests, security scans; restricted credentials (no prod access); track metrics (PR acceptance rate, defects introduced, cycle time); start with low-risk work (dependency upgrades, test coverage, small bugs) before expanding scope.

---

## 56. AWS MCP Servers

**What it is:** **Model Context Protocol (MCP)** is an open protocol (introduced by Anthropic) standardizing how AI applications (clients — IDEs, agents, chat apps) connect to external tools and data (**servers**) — "USB-C for AI tools". Servers expose **tools** (actions), **resources** (data), and **prompts**. Transports: stdio (local) and Streamable HTTP (remote), with OAuth-based auth for remote servers.

**AWS MCP offerings**
- **Open-source AWS MCP servers** (`awslabs/mcp` on GitHub): AWS Documentation, AWS API (call AWS CLI-style operations), CDK, Terraform, CloudFormation, Cost Analysis/Billing, CloudWatch, EKS, ECS, Lambda tool, DynamoDB, Aurora/RDS, Bedrock Knowledge Bases retrieval, Serverless/SAM, IAM, diagram generation, and more.
- **AWS Knowledge MCP Server:** Remote, AWS-hosted server providing up-to-date docs, API references, blogs and best practices.
- **AgentCore Gateway:** Turn your own APIs/Lambda into MCP tools; **AgentCore Runtime** can host MCP servers.
- MCP supported in Q Developer, Kiro, Strands, Bedrock agents and third-party clients (Claude Code, Cursor, etc.).

### Interview Questions

**Q1. What is MCP and why has it become important?**
Before MCP, each AI app needed custom integrations for each tool (N×M problem). MCP defines a standard client-server protocol, so any MCP-compatible client can use any MCP server — tool providers build once, all agents benefit. It enables agents to access live data (docs, cloud resources, databases, tickets) and take actions with standardized discovery, schemas, and auth.

**Q2. What are MCP's main primitives?**
**Tools:** functions the model can call (with JSON schema inputs), e.g. `describe_log_groups`. **Resources:** read-only data the client can fetch/attach (files, records). **Prompts:** reusable prompt templates. Plus client features like sampling and elicitation. Communication via JSON-RPC over stdio or Streamable HTTP.

**Q3. What are the security risks of MCP servers and how do you mitigate them?**
Risks: over-privileged credentials (an AWS API MCP server with admin rights lets a prompt-injected agent delete resources), **tool poisoning** (malicious tool descriptions instructing the model), untrusted third-party servers (supply-chain risk), data exfiltration via tools, confused-deputy issues with shared tokens. Mitigations: run servers with least-privilege, read-only IAM roles by default; require human approval for mutating tools; vet/pin server versions from trusted sources; OAuth with per-user scoped tokens for remote servers; allow-list servers organization-wide; log all tool calls; network isolation.

**Q4. How would you expose an internal REST API to AI agents company-wide?**
Option A: build an MCP server (SDKs in Python/TypeScript) wrapping the API with well-described, task-oriented tools. Option B: register the API's OpenAPI spec as a target in **AgentCore Gateway**, which exposes it as MCP tools with inbound auth (company IdP) and outbound auth to the API, plus semantic tool search. Either way: design tools around user tasks (not raw endpoints), concise outputs, pagination, and enforce authorization in the API itself.

**Q5. How do AWS MCP servers help developers day to day?**
In an IDE agent, the Documentation/Knowledge server provides current AWS docs (reducing hallucinated APIs); CDK/Terraform servers provide best-practice guidance and security checks (e.g. cdk-nag/Checkov) for generated IaC; cost servers answer "what will this architecture cost"; CloudWatch/EKS servers let the agent query logs and cluster state while debugging — all from natural language within the coding workflow.

---

## 57. Amazon Q Business

**What it is:** Fully managed GenAI assistant for **enterprise employees** — answers questions, summarizes, generates content, and takes actions using company data from 40+ connectors (SharePoint, Confluence, Jira, Salesforce, ServiceNow, Slack, Google Drive, S3, databases...), respecting existing **document-level permissions (ACLs)**. Includes **Q Apps** (lightweight GenAI apps built from prompts), plugins for actions in third-party apps, admin guardrails, and integrates with IAM Identity Center. In late 2025 AWS introduced **Amazon Quick Suite**, which brings Q Business capabilities together with QuickSight BI, research and automation agents into one workspace.

### Interview Questions

**Q1. Q Business vs building your own RAG with Bedrock Knowledge Bases?**
Q Business: turnkey employee assistant with a web UI, many prebuilt connectors, **ACL-aware retrieval** out of the box, admin controls, per-user pricing — fastest for internal knowledge search. Bedrock KB: building blocks for **custom** applications (customer-facing, embedded in products, custom UX, specific models, custom orchestration), pay per usage, more engineering effort. Choose by audience (employees vs customers) and customization needs.

**Q2. How does Q Business enforce permissions?**
Connectors crawl both content and ACLs (users/groups) from source systems; users authenticate via Identity Center (federated with corporate IdP); at query time only documents the user is entitled to are retrieved and used for answers. Group membership mapping must be correct — misconfigured identity mapping is the main risk.

**Q3. What admin controls exist to keep responses appropriate?**
Blocked words/topics, topic-specific rules, ability to restrict answers to enterprise data only (vs general model knowledge), document attribute boosting, response settings, plugin/action approvals, and audit logs.

**Q4. What are typical use cases?**
Policy/HR/IT helpdesk Q&A, onboarding, searching across Confluence/Jira/SharePoint, summarizing tickets and documents, drafting emails/reports, creating Jira tickets via plugins, and team-built Q Apps (e.g. "generate a release note from these PR descriptions").

**Q5. How would you measure success of an enterprise assistant rollout?**
Adoption (weekly active users), answer helpfulness feedback (thumbs up/down), deflected support tickets, time saved surveys, coverage gaps (unanswered queries → missing content), and incident tracking (wrong or unauthorized answers). Iterate on data sources and content quality — retrieval quality depends on document quality.

---

## 58. SageMaker AI

**What it is:** AWS's fully managed platform to **build, train, tune, deploy and monitor** machine learning models (renamed **SageMaker AI** in Dec 2024). The broader "next generation" **Amazon SageMaker** is the unified data + analytics + AI platform with **SageMaker Unified Studio** (single environment for EMR, Glue, Athena, Redshift, Bedrock, and SageMaker AI) and **SageMaker Lakehouse** (Iceberg-compatible unified access to S3 and Redshift data).

**Key capabilities**
- **Develop:** Studio (JupyterLab, Code Editor), notebooks, Data Wrangler, Feature Store, Ground Truth (labeling), **managed MLflow** (experiment tracking).
- **Train:** Managed training jobs (any framework, built-in algorithms), **managed Spot training** with checkpoints (up to ~90% savings), distributed training libraries, automatic model tuning (hyperparameter optimization), **HyperPod** (resilient, self-healing clusters for large-scale FM training/fine-tuning with Slurm or EKS orchestration, task governance).
- **JumpStart:** Hub of pre-trained/open-weight models (Llama, Mistral, DeepSeek, etc.) deployable/fine-tunable in a few clicks.
- **Deploy (inference):** Real-time endpoints (with auto scaling, **inference components** to pack multiple models per endpoint, scale-to-zero), **serverless inference**, **asynchronous inference** (large payloads, long processing, queue-based, scale to zero), **batch transform**, multi-model endpoints, large model inference containers (vLLM/TensorRT-LLM), shadow testing, deployment guardrails (blue/green, canary).
- **MLOps:** SageMaker Pipelines, Model Registry (versioning, approval), Projects (templates), Model Monitor (data/model drift), Clarify (bias & explainability), Model Cards, lineage.

### Interview Questions

**Q1. Bedrock vs SageMaker AI — when do you use which?**
Bedrock: consume foundation models via API, serverless, no infrastructure, managed RAG/agents/guardrails — best for building GenAI applications quickly. SageMaker AI: full control over training and hosting — custom ML models (fraud detection, recommendations, forecasting), fine-tuning/pre-training open-weight models with your own code, deploying models not on Bedrock, specific GPU/latency tuning. Many orgs use both (and Bedrock Custom Model Import bridges them).

**Q2. Explain SageMaker inference options and when to use each.**
**Real-time:** persistent endpoints, low latency, steady traffic (auto scaling, inference components for multi-model GPU sharing). **Serverless:** intermittent/spiky traffic, tolerates cold starts, CPU only, scale to zero. **Asynchronous:** large payloads (up to 1 GB) or long processing (up to 1 hour), queued requests, scale to zero — e.g. document/video processing. **Batch transform:** offline scoring of large datasets, no persistent endpoint.

**Q3. How do you implement MLOps on SageMaker?**
Source control for code and pipeline definitions → **SageMaker Pipelines** (data processing → training → evaluation → conditional registration) → **Model Registry** with approval status → CI/CD (CodePipeline/GitHub Actions) deploys approved models to staging then prod endpoints with canary/blue-green guardrails and auto rollback → **Model Monitor** for data quality and drift, Clarify for bias, alarms trigger retraining pipelines. Experiments tracked in MLflow; Feature Store for consistent online/offline features; lineage for audit.

**Q4. How do you cut model training costs?**
Managed Spot training with checkpointing, right-size instances (profile GPU utilization with Debugger/Profiler), use Trainium (Trn1/Trn2) instances, distributed training efficiency (data/model parallel libraries), mixed precision, early stopping in hyperparameter tuning, warm pools to avoid repeated provisioning, smaller/parameter-efficient fine-tuning (LoRA/QLoRA) instead of full fine-tuning, and SageMaker HyperPod flexible training plans for reserved GPU capacity.

**Q5. What is model drift and how do you detect it?**
**Data drift:** input feature distribution in production diverges from training data (e.g. new customer segments). **Concept drift:** relationship between inputs and target changes (fraud patterns evolve). Detect with Model Monitor baselines (statistics/constraints) and scheduled monitoring jobs comparing captured endpoint data, plus model quality monitoring when ground truth labels arrive. Respond by investigating and retraining on fresh data via pipelines.

**Q6. What is SageMaker HyperPod and why is it used for foundation models?**
Persistent, managed clusters for distributed training of large models over weeks: automatic health checks and **node replacement with automatic resume from checkpoints** (hardware failures are frequent at scale), Slurm or EKS orchestration, task governance to share GPU capacity across teams with priorities, and observability. It reduces wasted GPU time from failures significantly versus self-managed clusters.

---

## 59. DevOps Guru & CodeGuru

**What they are:**
- **Amazon DevOps Guru:** ML-powered **AIOps** service that analyzes CloudWatch metrics, logs, events, CloudTrail and Config to detect **operational anomalies** (e.g. Lambda latency spike, DynamoDB throttling, RDS performance issues with DevOps Guru for RDS), groups related anomalies into **insights**, and recommends remediation. Coverage defined by CloudFormation stacks/tags.
- **Amazon CodeGuru:** *Reviewer* (ML-based code review for Java/Python — security and quality issues) and *Profiler* (runtime performance and cost profiling of applications, identifies expensive lines of code). Note: AWS is consolidating these capabilities — code security scanning lives in **Amazon Inspector code security** and **Q Developer/Kiro**; CodeGuru Reviewer stopped accepting new repository associations, so check status before recommending it.

### Interview Questions

**Q1. What problem does DevOps Guru solve compared with static CloudWatch alarms?**
Static thresholds miss problems that don't cross fixed values and generate noise for seasonal patterns. DevOps Guru learns normal behavior per resource, flags anomalies, **correlates** related anomalies across services (e.g. a deployment → increased Lambda errors → API 5xx) into one insight with probable cause and recommendations — reducing alert fatigue and investigation time. Limitations: needs history to learn baselines, costs per resource analyzed, may miss app-specific business signals.

**Q2. What is AIOps and how would you apply it on AWS?**
Applying ML/AI to operations data for anomaly detection, event correlation/noise reduction, root-cause analysis and automated remediation. On AWS: CloudWatch anomaly detection alarms, DevOps Guru insights, CloudWatch investigations (AI-generated root-cause hypotheses), AWS DevOps Agent, Security Hub/GuardDuty correlated findings, EventBridge → Step Functions/SSM Automation for auto-remediation runbooks with human approval gates.

**Q3. How do you find performance bottlenecks in production code?**
Continuous profiling (CodeGuru Profiler or open-source alternatives such as Pyroscope/Grafana, Datadog Profiler) to view CPU/allocation flame graphs from real traffic; correlate with tracing (X-Ray/OTel spans) to find slow dependencies; Database Insights/Performance Insights for slow queries. Optimize the hottest paths first and measure impact on cost and p99 latency.

**Q4. How would you use AI for automated code review safely?**
Run AI review (Q Developer, Kiro hooks, Inspector code security, third-party tools) as an additional CI check commenting on PRs — never as the sole approver. Tune rules to reduce false positives, track which suggestions are accepted, combine with deterministic SAST/SCA/secret scanning, and keep human reviewers accountable for design and business logic.

**Q5. How do you auto-remediate common operational issues safely?**
Detect (CloudWatch alarm / DevOps Guru insight / Config rule) → EventBridge → Step Functions or **SSM Automation runbook** with pre-checks, rate limiting (don't restart all instances), approval steps for risky actions, verification after the action, and rollback; log every action and notify the on-call. Start with low-risk remediations (disk cleanup, restarting a stuck worker, scaling up) before higher-risk ones.

---

## 60. Trainium & Inferentia

**What they are:** AWS-designed ML accelerator chips offering better price-performance than comparable GPUs for many workloads.
- **AWS Trainium:** Training (and increasingly inference). **Trn1** (Trainium1), **Trn2** instances and **Trn2 UltraServers** (64 chips interconnected via NeuronLink), **Trainium3** (3 nm, re:Invent 2025, Trn3 UltraServers). Used for large-scale training (e.g. Anthropic's Project Rainier cluster).
- **AWS Inferentia:** Inference-optimized. **Inf2** (Inferentia2) supports large models with high throughput and low cost per inference.
- **AWS Neuron SDK:** Compiler, runtime, and libraries integrating with PyTorch (via PyTorch/XLA and native support) and JAX, Hugging Face Optimum Neuron, vLLM Neuron support, NxD (distributed training/inference libraries). Available on EC2, EKS, ECS, SageMaker AI and HyperPod.

### Interview Questions

**Q1. Why would you consider Trainium/Inferentia instead of NVIDIA GPUs?**
Lower cost per training run/per inference (AWS cites up to ~30–50% better price-performance for supported models), better availability than scarce high-end GPUs, energy efficiency, and tight AWS integration. Trade-offs: requires Neuron SDK compilation; not every model architecture/custom kernel/library is supported or optimized; ecosystem and community tooling is smaller than CUDA; migration and testing effort.

**Q2. How would you evaluate migrating an inference workload to Inferentia2?**
Check model architecture support in Neuron (popular LLMs and transformers are supported), compile the model with Neuron, benchmark latency (p50/p99, time-to-first-token), throughput and accuracy parity vs the GPU baseline at realistic batch sizes, calculate cost per 1M tokens/inferences, test autoscaling and cold-start (compile caching), then canary traffic in production.

**Q3. How do you serve an open-weight LLM cost-effectively on AWS?**
Options by effort: Bedrock (if available, or Custom Model Import — serverless pay per use), SageMaker AI endpoints with LMI containers (vLLM) and inference components, or EKS with vLLM/KServe on GPU or Inferentia instances with Karpenter. Cost levers: quantization (INT8/FP8/AWQ), continuous batching, KV cache optimizations, right GPU/accelerator size, autoscaling on queue depth/concurrency, Spot for non-critical batch, scale-to-zero for dev.

**Q4. What is an UltraServer and why does interconnect matter?**
An UltraServer links multiple instances (e.g. 64 Trainium2 chips across 4 Trn2 instances) with high-bandwidth, low-latency NeuronLink so they act like one large accelerator. Large models need tensor/pipeline parallelism across many chips; the communication between chips becomes the bottleneck — faster interconnect enables bigger models and faster training/inference.

**Q5. How do you control GPU/accelerator capacity costs for training?**
Use **capacity blocks** or SageMaker **flexible training plans** to reserve accelerators for specific windows (instead of paying for idle reservations), Spot for fault-tolerant jobs with checkpointing, HyperPod task governance to share clusters across teams, monitor utilization (Neuron monitor/DCGM), and choose parameter-efficient fine-tuning instead of full training where possible.

---

## 61. GenAI System Design on AWS — Scenario Questions

These combine many services and are increasingly common in senior interviews.

**Q1. Design an enterprise RAG chatbot over internal documents.**
- **Ingestion:** Documents in S3/SharePoint/Confluence → Bedrock Knowledge Bases (FM/Data Automation parsing for PDFs with tables, hierarchical chunking, Titan/Cohere embeddings) → OpenSearch Serverless (hybrid search) or S3 Vectors for cost; metadata (department, ACL groups, updated date). Sync via EventBridge-triggered ingestion jobs.
- **Query path:** Client → CloudFront + WAF → API Gateway (Cognito/IdP JWT) → Lambda/ECS orchestrator → Guardrails input check → query rewrite → KB `Retrieve` with **user-ACL metadata filters** → rerank → prompt with instructions + context (prompt caching for system prompt) → Claude/Nova via `ConverseStream` → Guardrails output (PII masking, contextual grounding) → stream answer with citations.
- **State:** Conversation history in DynamoDB (TTL).
- **Ops:** Invocation logging, OTel traces, token/cost metrics per team, feedback capture, offline evaluation suite (RAG evaluation) in CI, quotas and cross-region inference for throughput.

**Q2. How do you reduce latency in an LLM application?**
Stream tokens (`ConverseStream`) for perceived latency; prompt caching; smaller/faster models or routing; reduce input tokens (concise prompts, fewer/better chunks via reranking); parallelize retrieval and independent tool calls; cache full responses (semantic cache) for frequent queries; colocate services in the same region; use provisioned/priority tiers for consistent latency; precompute (summaries, embeddings) offline.

**Q3. Design a multi-tenant SaaS GenAI platform on Bedrock.**
Tenant identity in JWT; per-tenant **application inference profiles** (cost tracking via tags) and quotas/rate limits (API Gateway usage plans or token-bucket in DynamoDB/Valkey); tenant data isolation in RAG (separate KBs/indexes for premium tenants, metadata filters for pooled); tenant-specific guardrails and prompts (Prompt Management versions); per-tenant cost metering from invocation logs/CloudWatch token metrics; noisy-neighbor protection (SQS fair queues, per-tenant concurrency); evaluation per tenant segment.

**Q4. How do you handle hallucinations in production?**
Ground answers with RAG and require citations; instructions to abstain when context is insufficient; **contextual grounding checks**/Automated Reasoning checks in Guardrails; structured outputs with schema validation; lower temperature for factual tasks; verification step (second model or rules) for high-stakes outputs; human-in-the-loop for critical decisions; monitor via sampled LLM-as-judge scoring and user feedback; continuously improve retrieval.

**Q5. Design an AI agent that can perform actions in your AWS environment (e.g. ops assistant).**
Agent (Strands/LangGraph) on **AgentCore Runtime**; tools via **AgentCore Gateway** wrapping read-only diagnostics (CloudWatch queries, describe calls via MCP servers) and a small set of approved remediation runbooks (SSM Automation); per-user identity via AgentCore Identity — agent acts with the requesting engineer's permissions, never an admin role; **Policy** (Cedar) + IAM boundaries restrict tools; mutating actions require explicit human approval (Slack confirmation); full audit (CloudTrail + agent traces); Guardrails for prompt injection (logs can contain attacker-controlled text!); evaluation on historical incidents before rollout.

**Q6. How do you process millions of documents with an LLM cost-effectively (e.g. classification/extraction)?**
Batch, not real-time: documents in S3 → Step Functions **Distributed Map** or Bedrock **batch inference** jobs (~50% cheaper) using JSONL input; small model (Nova Micro/Lite or a distilled model) with a tight prompt and structured JSON output; validate outputs with schemas, route low-confidence items to a larger model or human review (A2I); idempotent writes of results to DynamoDB/S3; track cost per document; use Bedrock Data Automation for complex documents.

**Q7. How do you build an LLMOps pipeline (CI/CD for prompts, models, RAG)?**
Version everything — prompts, model IDs, retrieval config, guardrail versions — in Git/IaC; on PR run the evaluation suite (golden dataset: correctness, faithfulness, safety, latency, cost) and block regressions; deploy to staging, run red-team/adversarial tests; production rollout via canary/A-B (e.g. 5% traffic to new prompt/model through feature flags/AppConfig), monitor online metrics (feedback, judge scores, error rates, cost) and auto-rollback; continuously add failing production examples to the eval set.

**Q8. What are the top security risks for LLM applications (OWASP Top 10 for LLMs) and AWS mitigations?**
Prompt injection (Guardrails prompt-attack filters, input tagging, least-privilege tools), sensitive information disclosure (PII filters, ACL-filtered retrieval, no secrets in prompts), supply chain (vetted models/MCP servers, Bedrock-hosted models), data/model poisoning (curated KB sources, ingestion validation), improper output handling (escape/validate model output before rendering or executing — XSS, SQL injection), excessive agency (limited tools, human approval, AgentCore Policy), system prompt leakage (no secrets in system prompts), vector/embedding weaknesses (tenant isolation in vector stores), misinformation (grounding, citations), unbounded consumption (rate limits, `max_tokens`, budgets, per-tenant quotas).

---

## Quick Revision Checklist

- [ ] Can explain IAM policy evaluation and cross-account access from memory
- [ ] Can design a 3-AZ VPC and explain every route table entry
- [ ] Know SQS vs SNS vs EventBridge vs Kinesis trade-offs
- [ ] Can model a DynamoDB table from access patterns (single-table design)
- [ ] Know the 4 DR strategies with RTO/RPO and AWS services for each
- [ ] Can explain Lambda cold starts, concurrency types, and SQS integration failure handling
- [ ] Can design CI/CD with OIDC, blue/green/canary, and automatic rollback
- [ ] Can walk through a RAG architecture on Bedrock with security (ACL filtering, Guardrails)
- [ ] Know when to choose Bedrock vs SageMaker vs self-hosting
- [ ] Can discuss AgentCore components, MCP, and governance of AI agents
- [ ] Can discuss cost optimization including GenAI token costs
