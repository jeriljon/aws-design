# Domain 3: Cost-Optimized Architecture Design (15%)

Tasks 3.1 to 3.4: pricing models, data transfer and network costs, cost allocation and visibility, and storage and database costs.

## Task 3.1: Design architectures based on AWS pricing models

This task tests whether you can match each part of a workload to the cheapest purchase option and compute model that still meets its availability, latency and interruption constraints. The recurring decision is "which slice of this usage is steady, which is flexible, which is interruptible, and which needs guaranteed capacity", because each slice gets a different pricing instrument.

### Skill 3.1.1: Select purchase options based on usage patterns

**Key concepts**

- Layer the purchase options. A typical answer is: commitments (Savings Plans or Reserved Instances) for the steady 24x7 baseline, On-Demand for the unpredictable middle, Spot for interruptible and fault-tolerant work, and capacity reservations only where you must guarantee capacity in a specific Availability Zone.
- Four Savings Plans types exist as of late 2026. Know what each covers:
  - **Compute Savings Plans**: EC2 in any family, size, Region, OS or tenancy, plus Fargate and Lambda. Up to 66% off On-Demand. 1 or 3 years.
  - **EC2 Instance Savings Plans**: one instance family in one Region (for example m7g in eu-west-1), any size, OS or tenancy within it. Up to 72% off. 1 or 3 years.
  - **SageMaker AI Savings Plans**: SageMaker AI instance usage across families, sizes, Regions and components (notebooks, training, inference). Up to 64% off. 1 or 3 years.
  - **Database Savings Plans** (launched December 2025): an hourly dollar commitment for 1 year that covers Aurora, RDS, DynamoDB, ElastiCache, DocumentDB, Neptune, Keyspaces, Timestream and AWS DMS, across engines, instance families, sizes, deployment options and Regions. Discount depth varies by usage type: highest for serverless (up to 35%), lower for provisioned instances, and lower again for DynamoDB and Keyspaces throughput. No-upfront payment only.
- Compute, EC2 Instance and SageMaker plans support No Upfront, Partial Upfront and All Upfront. More upfront and a longer term give a deeper discount.
- Savings Plans do not apply to Spot. They do not cover RDS (that is Database Savings Plans or RDS Reserved Instances). Usage above the hourly commitment is billed On-Demand.
- By default a Savings Plan benefit floats across all accounts in the Organization (consolidated billing). You can restrict it to the purchasing account, or group accounts with cost categories for group-level purchasing.
- Savings Plans with an hourly commitment of $100 or less can be returned within 7 days if bought in the same calendar month. Otherwise the commitment is fixed. Savings Plans cannot be resold. Standard RIs can be sold on the RI Marketplace; Convertible RIs can be exchanged but not sold.
- **Reserved Instances** still matter for services without Savings Plans coverage of the right shape, and for **zonal RIs**, which are the only RI type that also reserves capacity. Regional RIs give a billing discount and size flexibility within a family (for Linux shared tenancy) but no capacity reservation.
- **On-Demand Capacity Reservations (ODCRs)** guarantee capacity in one AZ. You pay the On-Demand rate whether or not instances run in them. Regional RIs and Savings Plans apply to matching reservations automatically; zonal RI discounts do not. Use ODCRs for DR capacity, quarter-end events, or instance types that are scarce. Future-dated ODCRs let you book capacity for a planned event: request them 5 to 120 days ahead, for at least 32 vCPUs, with a minimum 14-day commitment for most instance types.
- **Capacity Blocks for ML** reserve GPU or Trainium capacity for a fixed future window, paid upfront. Use them for planned training runs, not for steady inference.
- **Spot**: deep discount, two-minute interruption notice, plus an earlier rebalance recommendation signal. Use the price-capacity-optimized allocation strategy and diversify across many instance types and AZs (ASG mixed instances policy, EC2 Fleet). Good for batch, CI runners, stateless web tiers behind a load balancer, big data on EMR task nodes, and checkpointed ML training.
- "Provisioned capacity" in the guide also means service-level provisioned modes: DynamoDB provisioned capacity with reserved capacity, Kinesis Data Streams provisioned shards, Bedrock Provisioned Throughput, and Lambda provisioned concurrency. These trade a fixed hourly cost for guaranteed throughput or warm capacity.
- Buy commitments after rightsizing, not before. Cost Explorer gives Savings Plans and RI purchase recommendations based on your past usage, plus coverage and utilization reports.

**Exam traps**

- EC2 Instance Savings Plans look better because the maximum discount is higher, but they lock family and Region. If the scenario says the team plans to move to Graviton, change Regions, or shift to Fargate or Lambda, Compute Savings Plans is right.
- "Reserved Instances guarantee capacity" is true only for zonal RIs. A Savings Plan never reserves capacity. If the requirement is "capacity must be available in the DR AZ", the answer needs an ODCR or zonal RI.
- Spot for a stateful single-instance database, or for any job that cannot checkpoint and has a hard deadline, fails the reliability requirement even though it is cheapest.
- Database Savings Plans are 1-year only. A question that asks for a 3-year database commitment points to RDS Reserved Instances or DynamoDB reserved capacity.
- Buying a 3-year All Upfront plan for a workload being retired in 12 months wastes money; the scenario's timeline decides the term.

**Hands-on**

1. **Read your own Savings Plans recommendations.** In Billing and Cost Management, open Savings Plans, then Recommendations, and compare Compute versus EC2 Instance plans for 1 and 3 years with each payment option. If your account has little usage, run a small t4g instance for a few days first. Open the coverage and utilization reports and note how they are calculated hourly. This proves how the hourly commitment model works and why coverage below 100% is normal. *Cost/cleanup:* Viewing recommendations is free. Do not purchase anything. Stop any instance you launched.
2. **Spot with a mixed instances Auto Scaling group.** Create a launch template for a small Amazon Linux arm64 instance, then an Auto Scaling group with a mixed instances policy: On-Demand base capacity 1, 0% On-Demand above base, price-capacity-optimized Spot allocation, and four or five instance types. Set desired capacity to 3 and watch which types and AZs it picks. Add an EventBridge rule for "EC2 Spot Instance Interruption Warning" that targets an SNS email so you see what the notice looks like if one arrives. *Cost/cleanup:* A few cents per hour for small instances. Delete the Auto Scaling group, launch template, EventBridge rule and SNS topic.

### Skill 3.1.2: Design cost-optimized architectures to meet workload requirements

**Key concepts**

- **Graviton**: lower hourly price than comparable x86 instances and generally better price performance. Available for EC2, Lambda (arm64 has a lower per-GB-second price), Fargate (ARM64), RDS, Aurora, ElastiCache, OpenSearch and EMR. Managed services often need only an instance class change. Self-managed code needs arm64 builds (multi-arch container images in ECR).
- **Lambda pricing**: requests plus GB-seconds at 1 ms granularity. Memory also sets CPU, so more memory can make a function faster and cheaper. Provisioned concurrency adds a charge for keeping environments warm. Compute Savings Plans cover Lambda duration and provisioned concurrency.
- When Lambda stops being cheap: sustained high-throughput, long-running or steady 24x7 work is usually cheaper on Fargate or EC2 with commitments. Lambda Managed Instances (launched late 2025) run Lambda functions on EC2 capacity in your account, billed as EC2 instance cost plus a management fee and a per-request charge, with no per-duration charge, and are eligible for Savings Plans. It fits high-volume steady workloads that want the Lambda programming model.
- **Step Functions Standard**: billed per state transition, including retries. Executions can run up to one year, are exactly-once, and keep full execution history. Use it for long-running, auditable workflows, human approvals (callback task tokens), and anything non-idempotent.
- **Step Functions Express**: billed by number of requests, duration (rounded up to 100 ms) and memory used. Maximum duration 5 minutes. Asynchronous Express is at-least-once, synchronous Express is at-most-once. Execution history goes to CloudWatch Logs only if you enable logging. Use it for high-volume, short, idempotent event processing (IoT ingestion, stream transforms, API orchestration).
- Cost crossover: a short workflow with many states and millions of executions per day costs far less as Express. A workflow that waits (for example for a callback or a `Wait` state of hours) is cheap on Standard because waiting does not create transitions, but impossible on Express because of the 5-minute limit.
- Nested pattern: a Standard parent workflow for the durable business process, with Express child workflows for high-volume inner loops. Distributed Map can run child executions as Express.
- Reduce transitions in Standard: use direct service integrations instead of Lambda glue functions, batch items, and avoid needless `Pass` states.
- Other architectural cost levers: S3 plus CloudFront instead of web servers for static content, SQS buffering to smooth peaks so you provision for average not peak, Aurora Serverless v2 or DynamoDB on-demand for spiky databases, and event-driven designs that scale to zero.

**Exam traps**

- Choosing Express Workflows for a process that includes a human approval or runs longer than 5 minutes fails on duration even if it is cheaper.
- Choosing Standard for tens of millions of short executions per day "for reliability" ignores the per-transition cost; the exam expects Express with idempotent handlers.
- Moving everything to Lambda for cost. A steady, CPU-heavy service at constant high utilisation is usually cheaper on Graviton instances or Fargate with a Compute Savings Plan.
- Assuming Graviton is a drop-in change for software with x86-only binaries or licensed agents. The scenario may say a vendor library only ships for x86.

**Hands-on**

1. **Standard versus Express cost comparison.** Build the same three-state workflow (Lambda invoke, Choice, DynamoDB PutItem via direct integration) twice: once as Standard and once as Express with CloudWatch logging set to ERROR. Start 1,000 executions of each with a small script. Compare the Standard execution history in the console with the Express logs, then use the AWS Pricing Calculator to price 10 million executions per month for each type. *Cost/cleanup:* Standard stays within the 4,000 free transitions per month for small tests; Express costs fractions of a cent. Delete both state machines, the Lambda function, the DynamoDB table and the log group.
2. **Graviton Lambda comparison.** Deploy the same Python function twice, once x86_64 and once arm64, both at 1,024 MB. Invoke each 500 times with a CPU-bound task (for example hashing) and compare average duration in CloudWatch metrics. Multiply duration by each architecture's GB-second price from the Lambda pricing page. This shows that arm64 wins on both price and often duration, but check your own code. *Cost/cleanup:* Within the Lambda free tier. Delete both functions and their log groups.

### Skill 3.1.3: Implement rightsizing and scaling strategies based on performance metrics

**Key concepts**

- **AWS Compute Optimizer** gives rightsizing recommendations for EC2 instances, Auto Scaling groups, EBS volumes, Lambda memory, ECS services on Fargate, RDS and Aurora DB instances and storage, and SQL Server licensing on EC2. It also flags idle resources such as unused NAT gateways, idle DynamoDB provisioned tables, ElastiCache and SageMaker endpoints. Cost Optimization Hub collects these with Savings Plans and other recommendations in one place across the Organization.
- Compute Optimizer uses CloudWatch metrics over the last 14 days by default. Enhanced infrastructure metrics (a paid option) extend the lookback to about three months, which matters for monthly cycles. EC2 memory-based recommendations need the CloudWatch agent publishing memory metrics.
- Rightsize first, then scale, then commit. Commitments bought on an oversized fleet lock in waste.
- **Scaling policy choice**:
  - **Target tracking**: keep a metric near a target (for example 50% CPU, or ALB requests per target). The default answer for most workloads.
  - **Step scaling**: different actions at different alarm breach sizes; use when you need custom step sizes.
  - **Scheduled scaling**: known patterns such as business hours or a nightly batch. Pair with target tracking for the unknown part.
  - **Predictive scaling** (EC2 Auto Scaling): forecasts from history and scales ahead of cyclical load; useful when instances take a long time to boot.
- Application Auto Scaling applies the same policy types to ECS services, DynamoDB provisioned capacity, Aurora replicas, Lambda provisioned concurrency, SageMaker endpoint variants and others.
- **Scale in non-production to zero**: scheduled actions on ASGs and ECS services, Instance Scheduler on AWS, or stopping RDS instances (RDS restarts stopped instances automatically after seven days, so a stop-only scheme needs a scheduler).
- **Fargate Spot** (Amazon ECS only): interruption-tolerant tasks at a discount to Fargate. Interruptions come with a two-minute warning delivered as SIGTERM and as an ECS task state change event in EventBridge. Set `stopTimeout` up to 120 seconds and handle SIGTERM. If Spot capacity is unavailable, ECS keeps retrying; it does not fall back to regular Fargate. So use a capacity provider strategy with a `base` on FARGATE for the minimum tasks you always need and a `weight` split between FARGATE and FARGATE_SPOT above that.
- Fargate rightsizing: tasks are billed on requested vCPU and memory, not on what the container uses. Over-requested task sizes are pure waste. Compute Optimizer and Container Insights show actual usage.
- Karpenter on EKS consolidates pods onto fewer, cheaper nodes and can mix Spot and On-Demand; Cluster Autoscaler scales node groups but does less bin packing.

**Exam traps**

- Putting an entire ECS service on Fargate Spot with no FARGATE base. If Spot capacity is reclaimed in that Region, the service can drop to zero tasks because there is no automatic fallback.
- Fargate Spot for EKS. It is an ECS capacity provider; EKS on Fargate does not offer Spot. For EKS cost savings use Spot managed node groups or Karpenter with Spot.
- Using Compute Optimizer default lookback for a workload that peaks at month-end. Without enhanced infrastructure metrics it may recommend a size that fails the peak.
- Scheduled scaling alone for a workload that has both a daily pattern and unpredictable spikes. Combine scheduled actions with target tracking.
- Scaling on CPU for a queue worker. Scale on queue depth per task (backlog per instance) instead; CPU can stay low while the backlog grows.

**Hands-on**

1. **Fargate Spot capacity provider strategy.** Create an ECS cluster with FARGATE and FARGATE_SPOT capacity providers. Deploy a service of a small nginx task with strategy FARGATE base 1 weight 1, FARGATE_SPOT weight 3, desired count 4. Check in the console which tasks landed on which provider. Create an EventBridge rule for ECS Task State Change with `stopCode` SpotInterruption that sends to SNS, so you would see an interruption. *Cost/cleanup:* A few cents per hour at 0.25 vCPU and 0.5 GB. Set desired count to 0, delete the service, cluster, rule and topic. Use public subnets with public IPs instead of a NAT gateway to avoid NAT charges.
2. **Compute Optimizer opt-in and review.** Opt in to Compute Optimizer (free for the default 14-day lookback). Run a t3.medium with a light workload for at least 30 hours so it has enough data, then read the recommendation, the projected utilisation graph, and the "performance risk" field. Read the EC2 Auto Scaling predictive scaling documentation page and note when predictive scaling is preferred. *Cost/cleanup:* Instance cost only; Compute Optimizer default mode is free. Terminate the instance. Do not enable enhanced infrastructure metrics unless you accept the charge.

### Skill 3.1.4: Optimize costs for AI/ML workloads with service tiers, batching and caching

**Key concepts**

- **Amazon Bedrock pricing dimensions**: on-demand inference is priced per input and output token (per image or per second for some media models). Output tokens usually cost more than input tokens, so limiting output length matters.
- **Bedrock service tiers** (selected per request through a service tier parameter; on-demand quota is shared across Priority, Standard and Flex):
  - **Standard**: default on-demand pricing.
  - **Priority**: a premium over Standard for faster, preferred processing of latency-sensitive customer-facing calls without a reservation.
  - **Flex**: a discount to Standard for work that can tolerate longer processing time, such as evaluations, summarisation and agent background steps.
  - **Reserved**: reserved tokens-per-minute capacity for 1 or 3 months at a fixed monthly price, arranged through the AWS account team, with overflow to Standard.
  - CloudWatch reports the requested and the actually resolved tier, so you can verify where traffic went.
- **Batch inference**: submit a JSONL file in S3 as an asynchronous job; results land in S3. Supported models are priced at 50% of on-demand. Jobs have a minimum record count per model and no latency guarantee. Use for nightly or bulk document processing, classification, embeddings backfill and evaluation sets.
- **Provisioned Throughput**: model units billed hourly whether used or not, with no-commitment, 1-month or 6-month terms (longer is cheaper). Historically required to serve customised models; Bedrock now offers on-demand deployment for supported customised models (for example Amazon Nova and Meta Llama), so Provisioned Throughput is needed only for customised models without that option. Use it for steady, high, predictable throughput, not for spiky or low traffic.
- **Prompt caching**: mark a cache checkpoint after a long static prefix (system prompt, tool definitions, a long document). Cache writes cost more than normal input tokens and cache reads cost far less (about 90% less on supported models). The default TTL is 5 minutes; some models support 1 hour at a higher write price. Each checkpoint has a model-specific minimum token count. Caches are scoped to account and Region, so cross-Region inference routing can cause extra writes. It pays off when the same prefix is reused several times within the TTL.
- **Model choice**: the biggest lever is usually a smaller model. Use a small model for classification and routing and a large one only where needed. Bedrock Intelligent Prompt Routing can route between models in a family for a per-request fee. Model distillation trains a smaller model from a larger one's outputs.
- **Application-level caching**: cache full responses for repeated identical prompts (ElastiCache) and use a semantic cache in a vector store (for example Amazon MemoryDB vector search) for near-duplicate questions. Cache retrieval results in RAG. Trim retrieved chunks and conversation history to cut input tokens.
- **SageMaker AI inference options**:
  - **Real-time endpoints**: always-on instances; cheapest per request at steady high load; use auto scaling and SageMaker AI Savings Plans.
  - **Serverless Inference**: pay per request duration, no idle cost, tolerates cold starts; spiky low-volume traffic.
  - **Asynchronous Inference**: queued requests with payloads up to 1 GB, results to S3, can scale to zero instances when idle.
  - **Batch Transform**: offline inference over a dataset with no persistent endpoint.
  - Consolidate with multi-model endpoints (many models share instances) or multi-container endpoints. Use Inference Recommender to pick instance types, AWS Inferentia instances for price performance, and SageMaker Neo to compile models for smaller hardware.
- **SageMaker training**: Managed Spot Training with checkpoints to S3 cuts training cost for interruptible jobs. Capacity Blocks or training plans reserve accelerators for planned runs.

**Exam traps**

- Provisioned Throughput for a workload that runs a few hours a day. You pay every hour; on-demand, Flex or batch is cheaper. The exception is a customised model that has no on-demand deployment option, which still needs Provisioned Throughput to be served.
- Batch inference for an interactive chatbot. Batch has no latency guarantee; the right levers there are prompt caching, a smaller model, or response caching.
- Priority tier as a cost optimisation. It costs more than Standard; it buys latency.
- Prompt caching when every request has a different prefix, or when calls are spaced further apart than the TTL. You pay the write premium and never get reads.
- SageMaker real-time endpoint for an idle-most-of-the-day model with large payloads and no latency need. Asynchronous Inference with scale to zero fits; Serverless Inference does not handle very large payloads or long processing well.

**Hands-on**

1. **Measure prompt caching.** In a Region where a Claude or Nova model supports caching, call the Converse API with a system prompt of a few thousand tokens followed by a `cachePoint`, then send five different short questions within five minutes. Read `usage` in each response for cache write and cache read token counts and compare with a run without the checkpoint. Check the Bedrock CloudWatch metrics for input tokens and cache tokens. *Cost/cleanup:* Usually well under one dollar for a small model; check the model's pricing first. Nothing to delete except optional log groups.
2. **Run a small batch inference job.** Create a JSONL file that meets the minimum record count for a small model, upload it to S3, create an IAM service role, and submit a batch inference job with `CreateModelInvocationJob`. Watch the job status and inspect the output files. Compare its cost with the same records sent on-demand using the pricing page. *Cost/cleanup:* Tokens at half the on-demand rate for supported models; keep records short. Delete the S3 objects, buckets and role.

### Practice questions: Task 3.1

**Question 3.1.1** A financial services company runs a trading analytics platform in two AWS Regions. The steady baseline is about 400 vCPUs of EC2 across several instance families, plus Lambda functions and ECS tasks on Fargate that run around the clock. Over the next 12 months the platform team plans to migrate most EC2 instances to Graviton and move some services from EC2 to Fargate. A nightly risk batch runs on EC2, checkpoints its progress to Amazon S3, and can be restarted. For quarter-end, the DR Region must have a specific GPU instance type available in one Availability Zone for 2 weeks. Which solution is the most cost-effective while meeting these requirements?

- A. Buy EC2 Instance Savings Plans for each current instance family in both Regions, run the nightly batch on On-Demand Instances, and buy zonal Reserved Instances for the GPU instance type in the DR Region.
- B. Buy a 3-year Compute Savings Plan sized to peak usage, run the nightly batch on the same Savings Plan coverage, and rely on the Savings Plan for quarter-end GPU capacity in the DR Region.
- C. Buy a Compute Savings Plan sized to the steady baseline, run the nightly batch on Spot Instances with a diversified instance list, and create a future-dated On-Demand Capacity Reservation for the GPU instance type in the DR Availability Zone.
- D. Buy Standard Reserved Instances for the current instance families, run the nightly batch on Spot Instances, and request a Service Quotas increase for the GPU instance type in the DR Region before quarter-end.

<details>
<summary>Answer and explanation</summary>

**Correct answer: C**

**Why C is correct:** A Compute Savings Plan covers EC2 in any family and Region, plus Fargate and Lambda, so the discount follows the workload through the Graviton and Fargate migrations. Sizing it to the steady baseline avoids paying for unused commitment. The batch checkpoints and can restart, which makes it a good fit for Spot with diversification. A future-dated On-Demand Capacity Reservation guarantees the GPU instance type in the specific AZ. It must be requested 5 to 120 days in advance and carries a minimum 14-day commitment, which is still far cheaper than a 1- or 3-year zonal RI for a quarterly event.

**Why the other options are incorrect:**

- **A:** EC2 Instance Savings Plans lock each plan to one instance family in one Region, so the Graviton migration would leave the plans unused. Zonal RIs for 2 weeks of need each quarter are a 1- or 3-year commitment, which is expensive for a quarterly event. Running a restartable batch On-Demand also gives up Spot savings.
- **B:** Sizing a commitment to peak guarantees unused commitment for most hours. A Savings Plan is a billing discount and never reserves capacity, so it does not guarantee GPU availability in the DR AZ.
- **D:** Standard RIs are tied to the current families and cannot follow the move to Graviton or Fargate. A Service Quotas increase raises the allowed limit but does not reserve physical capacity in an AZ.

**Key takeaway:** Commit to the baseline with the most flexible plan that still gives a good discount, use Spot for restartable work, and use capacity reservations only when the requirement is guaranteed capacity.

</details>

**Question 3.1.2** A payments company processes card authorisation events from Amazon EventBridge. Each event triggers a workflow of six steps that validates the payload, calls two internal APIs, writes to Amazon DynamoDB and publishes to Amazon SNS. Each workflow finishes in under 2 seconds. Volume is about 40 million events per day. All steps are idempotent and keyed on the authorisation ID. The current AWS Step Functions Standard Workflows bill is the largest line item on the account, and the team wants the lowest cost without rewriting the business logic. Which solution meets these requirements?

- A. Convert the state machine to an Express Workflow started asynchronously from EventBridge, keep the idempotent handlers, and enable CloudWatch Logs at the ERROR level for troubleshooting.
- B. Keep the Standard Workflow and replace the Lambda glue steps with direct service integrations to reduce state transitions, then buy a Compute Savings Plan to cover the Step Functions charges.
- C. Replace the workflow with a single Lambda function that performs all six steps, with provisioned concurrency sized to the peak event rate to avoid cold starts.
- D. Convert the state machine to a synchronous Express Workflow called through Amazon API Gateway so each event is processed at most once.

<details>
<summary>Answer and explanation</summary>

**Correct answer: A**

**Why A is correct:** Express Workflows are billed on requests, duration and memory rather than per state transition, which makes them far cheaper for high-volume, short executions. The workflow completes well within the 5-minute Express limit. Asynchronous Express runs are at-least-once, which is safe here because every step is idempotent on the authorisation ID. The state machine definition stays the same apart from the type, so the business logic is not rewritten. Logging at ERROR keeps CloudWatch Logs ingestion cost low while preserving troubleshooting.

**Why the other options are incorrect:**

- **B:** Removing Lambda glue steps does reduce transitions, but the bill is still per transition at 40 million executions a day. Compute Savings Plans cover EC2, Fargate and Lambda, not Step Functions, so that part of the option gives no discount.
- **C:** Collapsing everything into one function rewrites the business logic and loses the workflow's visibility and error handling. Provisioned concurrency sized to peak adds a constant charge for warm capacity that sits idle off-peak.
- **D:** Synchronous Express is at-most-once and is meant for request/response callers. Putting API Gateway in front of an EventBridge-driven flow adds a per-request charge and an extra hop with no benefit, and at-most-once semantics can drop events.

**Key takeaway:** For millions of short, idempotent executions per day, Express Workflows (asynchronous) are the cost answer; Standard is for long, exactly-once or human-in-the-loop workflows.

</details>

**Question 3.1.3** A media company runs an image-resizing worker service on Amazon ECS with AWS Fargate. Workers pull jobs from an Amazon SQS queue, and each job can be retried safely. Traffic is predictable: high from 07:00 to 22:00 local time and close to zero overnight. The service must always keep at least 2 tasks running to meet an internal processing SLA, even during capacity shortages. Container Insights shows that tasks use about 30% of their requested memory. Which combination of actions will reduce cost the most while meeting these requirements? (Select TWO.)

- A. Move the service to Amazon EKS with Fargate profiles and enable Fargate Spot for the worker pods, keeping 2 pods on regular Fargate.
- B. Use a capacity provider strategy with FARGATE base 2 and weights that place additional tasks mostly on FARGATE_SPOT, and handle SIGTERM in the worker to return in-flight jobs to the queue.
- C. Buy an EC2 Instance Savings Plan for the instance family that Fargate uses in the Region, and keep all tasks on regular Fargate.
- D. Configure target tracking on average CPU utilisation of 70% as the only scaling policy, so that the service scales in overnight automatically.
- E. Reduce the task memory size based on the Compute Optimizer and Container Insights data, and add Application Auto Scaling scheduled actions for the daytime and overnight ranges combined with target tracking on SQS backlog per task.

<details>
<summary>Answer and explanation</summary>

**Correct answer: B, E**

**Why B and E are correct:** B puts the 2 always-needed tasks on regular Fargate through the `base`, so the SLA holds even if Spot capacity is unavailable, because Fargate does not fall back from Spot to On-Demand on its own. Extra tasks run on Fargate Spot at a discount, and handling the two-minute SIGTERM warning makes interruptions harmless for retryable SQS jobs. E removes waste: Fargate bills requested memory, not used memory, so shrinking an over-sized task cuts cost on every task-hour. Scheduled actions match the known daily pattern, and target tracking on backlog per task handles variation inside each window.

**Why the other options are incorrect:**

- **A:** Fargate Spot is an ECS capacity provider. EKS on Fargate does not offer Spot, so this design cannot be built, and the migration itself adds operational work.
- **C:** EC2 Instance Savings Plans do not apply to Fargate. Only Compute Savings Plans cover Fargate. Keeping all tasks on regular Fargate also forgoes Spot savings.
- **D:** CPU is a poor scaling signal for a queue worker because backlog can grow while CPU stays moderate. Using CPU target tracking as the only policy also ignores the known schedule, so scale-out lags the 07:00 ramp.

**Key takeaway:** For Fargate cost, rightsize the task definition first, then use a FARGATE base for guaranteed tasks and FARGATE_SPOT weight for the rest.

</details>

**Question 3.1.4** An insurance company must summarise about 2 million claim documents each night with a foundation model on Amazon Bedrock. Documents arrive in Amazon S3 during the day. Summaries must be available by 07:00 the next morning, and nobody reads them before then. The company also runs a customer-facing chatbot on the same model with on-demand Standard tier requests. The finance team wants to reduce the nightly job's inference cost as much as possible without affecting the chatbot. Which solution meets these requirements?

- A. Purchase Bedrock Provisioned Throughput with a 6-month commitment sized for the nightly peak, and send the nightly job and the chatbot through the provisioned model.
- B. Send the nightly job through on-demand requests using the Priority service tier so it finishes faster and frees quota for the chatbot before morning.
- C. Deploy the open-weight version of the model on an Amazon SageMaker AI real-time endpoint with GPU instances and run the nightly job against that endpoint.
- D. Write the documents to JSONL files in Amazon S3 each evening and submit a Bedrock batch inference job, with results written to S3 for downstream processing.

<details>
<summary>Answer and explanation</summary>

**Correct answer: D**

**Why D is correct:** Bedrock batch inference processes JSONL input from S3 asynchronously and writes results back to S3. For supported models it is priced at 50% of on-demand token pricing. The workload has an overnight window and no reader until morning, so the lack of a latency guarantee does not matter. Batch jobs also do not consume the on-demand request path the chatbot uses, so the chatbot is unaffected.

**Why the other options are incorrect:**

- **A:** Provisioned Throughput is billed every hour for the whole term, whether or not it is used. A job that runs a few hours a night would leave most of the paid capacity idle, which costs more than batch pricing.
- **B:** Priority tier costs more than Standard. It buys latency, which this job does not need. It also shares on-demand quota with the chatbot, which can increase contention.
- **C:** A GPU real-time endpoint is billed per instance-hour while running and adds model hosting operations. It is not guaranteed to be cheaper than half-price batch tokens, and the option does not mention scaling the endpoint to zero.

**Key takeaway:** For large, latency-tolerant Bedrock workloads with a deadline in hours, use batch inference; use Provisioned Throughput only for steady high throughput or customised models that lack on-demand deployment.

</details>

**Question 3.1.5** A healthcare analytics company hosts a computer vision model that analyses medical images of 200 MB to 800 MB each. Requests arrive in bursts a few times a day from a partner upload process, and processing each image takes 3 to 6 minutes. Results are written back for later review, and the partner accepts results within 1 hour. The model is currently on an Amazon SageMaker AI real-time endpoint with two GPU instances that sit idle about 80% of the time. The company wants to minimise cost with the least operational overhead. Which solution meets these requirements?

- A. Move the model to SageMaker Serverless Inference with the maximum memory setting so the company pays only while requests are processed.
- B. Move the model to SageMaker Asynchronous Inference with input and output in Amazon S3, and configure auto scaling with a minimum of zero instances based on the queue backlog.
- C. Keep the real-time endpoint, buy a SageMaker AI Savings Plan for the two GPU instances, and enable target tracking on invocations per instance.
- D. Run a SageMaker Batch Transform job every 5 minutes from an Amazon EventBridge schedule to process any new images in the S3 upload prefix.

<details>
<summary>Answer and explanation</summary>

**Correct answer: B**

**Why B is correct:** Asynchronous Inference queues requests, reads large payloads (up to 1 GB) from S3, writes results to S3, and suits long processing times. It can scale down to zero instances when the queue is empty and scale out on backlog, so the idle 80% stops costing money. The 1-hour result window easily absorbs scale-from-zero time. It is a managed endpoint type, so the model container is reused with little change.

**Why the other options are incorrect:**

- **A:** Serverless Inference is for synchronous, short requests and has limits on payload size and processing time that payloads of hundreds of megabytes and multi-minute runs exceed. It also does not support GPUs.
- **C:** A Savings Plan lowers the hourly rate but still pays for two GPU instances that are idle most of the day. A real-time endpoint cannot drop to zero instances with plain target tracking, so the waste remains.
- **D:** Batch Transform suits large scheduled datasets. Starting a job every 5 minutes adds start-up overhead each time, many runs would find nothing to process, and building the scheduling and tracking adds operational work compared with a managed queue.

**Key takeaway:** Large payloads, long processing, bursty traffic and a relaxed latency need point to SageMaker Asynchronous Inference with scale to zero.

</details>


## Task 3.2: Optimize data transfer and network costs

This task tests whether you know where AWS charges for bytes (internet egress, inter-Region, inter-AZ, NAT gateway processing, Transit Gateway processing, PrivateLink processing) and can redesign traffic paths to avoid them. The recurring decision is which path a given flow should take so that it is private and compliant without paying a processing charge on every gigabyte.

### Skill 3.2.1: Design content delivery architectures, origin selection, caching and failover

**Key concepts**

- Data transfer from AWS origins (S3, EC2, ALB) to CloudFront is not charged. You pay CloudFront's own egress and request rates instead, which are generally lower than direct internet egress from a Region at scale. CloudFront also offers a flat-rate pricing plan option for some customers; check the current pricing page before relying on it.
- Cache hit ratio is the main lever. Raise it by:
  - Using cache policies that include only the headers, cookies and query strings the origin really needs in the cache key. Forward extra values to the origin with an origin request policy without putting them in the key.
  - Setting long TTLs on versioned static assets (file names with a content hash) and short TTLs on HTML.
  - Normalising inputs with CloudFront Functions (for example lower-casing or sorting query strings, or reducing `Accept-Language` to a few values).
  - Enabling compression (Gzip and Brotli) to cut bytes served.
- **Origin Shield** is an extra caching layer in one chosen Region between the regional edge caches and the origin. All cache layers fetch through it, so concurrent misses collapse into roughly one origin request. Use it when viewers are spread across many geographies, the origin is expensive per request (just-in-time video packaging, image transformation, on-premises origin with limited bandwidth), or multiple CDNs front the same origin. Pick the Origin Shield Region closest to the origin (the same Region when the origin is in a supported Region).
- Origin Shield pricing: a per-request charge (priced per 10,000 requests) for requests that pass through it as an incremental layer. Dynamic requests (PUT, POST, PATCH, DELETE, and GET or HEAD with caching disabled or a short TTL) are always charged. Cacheable requests are charged only when they come from a regional edge cache in a different Region from the Origin Shield. Origin Shield is a poor fit for mostly dynamic or low-cacheability traffic: you pay the extra layer without a hit-ratio gain.
- **Origin failover** with an origin group: CloudFront sends to the primary origin and retries the secondary on configured HTTP status codes or connection failures. It works for GET, HEAD and OPTIONS only, not writes. Combine with S3 Cross-Region Replication for a static site that survives a Regional S3 problem.
- For S3 origins use Origin Access Control so the bucket stays private and all reads go through CloudFront (security plus no direct S3 egress bypass).
- Price classes restrict which edge locations serve traffic. A cheaper price class lowers cost but raises latency for viewers in excluded regions.
- CloudFront also accelerates dynamic content (persistent connections to the origin, TLS termination at the edge) even with caching disabled, and supports POST and PUT through to the origin.

**Exam traps**

- Turning on Origin Shield for a mostly dynamic API to "save cost". Dynamic requests are always charged as an incremental layer and gain nothing from caching.
- Including the `Authorization` header or all cookies in the cache key for public assets. Every user gets a separate cache entry and the hit ratio collapses.
- Origin groups for failover of POST requests. Origin failover applies only to GET, HEAD and OPTIONS; write failover needs an application or Route 53 design.
- S3 Transfer Acceleration to cut cost for a global download audience. Transfer Acceleration speeds uploads and downloads to a bucket and adds a per-GB charge; CloudFront caching is the cost answer for repeated downloads.

**Hands-on**

1. **Measure cache hit ratio and cache keys.** Create a private S3 bucket with a few images, then a CloudFront distribution using Origin Access Control and the managed CachingOptimized policy. Request the same object with different query strings using curl and read the `X-Cache` header. Then create a custom cache policy that ignores query strings and repeat. Enable standard logging or check the CloudFront cache statistics report to see the hit ratio change. *Cost/cleanup:* Within the CloudFront and S3 free tier for a few requests. Disable and delete the distribution, then delete the bucket.
2. **Origin Shield and origin failover.** On the same distribution, enable Origin Shield in the bucket's Region and request objects from two distant locations (for example a CloudShell session in another Region). Look for `OriginShieldHit` in the logs. Then add a second bucket in another Region with S3 replication, create an origin group with failover on 403, 404, 500, 502, 503 and 504, and temporarily block the primary bucket policy to watch failover. *Cost/cleanup:* Origin Shield requests cost a small amount per 10,000; keep tests short. Remove Origin Shield, delete the distribution, replication rule and both buckets.

### Skill 3.2.2: Optimize data transfer with private connectivity, Regional traffic patterns and AZ awareness

**Key concepts**

- **Know the meters**:
  - Data into AWS from the internet: free.
  - Data out to the internet: charged per GB (tiered).
  - Between AZs in a Region: charged per GB in each direction for EC2-to-EC2 style traffic, including over VPC peering.
  - Within one AZ using private IPs: free.
  - Between Regions: charged per GB at the sending side.
  - NAT gateway: an hourly charge per gateway plus a per-GB data processing charge on every byte, on top of any data transfer.
- **Gateway VPC endpoints** for S3 and DynamoDB have no hourly or data processing charge. They are route-table targets that work only from inside the VPC in the same Region. Use them first; they remove NAT processing for S3 and DynamoDB traffic.
- **Interface VPC endpoints** (PrivateLink) have an hourly charge per endpoint per AZ plus a per-GB processing charge (tiered, lower than NAT processing). They are worth it when the traffic would otherwise cross a NAT gateway in volume, or when you need private access from on premises or another VPC. For many low-traffic services, the per-AZ hourly cost can exceed what you save.
- S3 also supports interface endpoints. Use those when you need access from on premises over Direct Connect or VPN, or from peered VPCs, because gateway endpoints are not reachable from outside the VPC.
- Centralised interface endpoints in a shared services VPC reduce the number of endpoints, but traffic from spokes then pays Transit Gateway processing plus endpoint processing. Do the arithmetic: centralise for many accounts with light traffic; keep local endpoints for heavy-traffic VPCs.
- **AZ-aware design**:
  - Keep chatty tiers in the same AZ where availability allows. With an NLB, cross-zone load balancing is off by default; with an ALB it is on, but ALB does not charge for inter-AZ traffic between the load balancer and targets.
  - Use a NAT gateway per AZ with per-AZ route tables. One shared NAT gateway saves the hourly charge but adds inter-AZ charges and a single-AZ failure point.
  - Use topology-aware routing in EKS and read from same-AZ replicas where the client library supports it (for example ElastiCache reader endpoints and Kafka rack awareness on Amazon MSK).
  - Some managed services do not charge for their own internal cross-AZ replication (for example Aurora storage replication and RDS Multi-AZ replication are included in the service price). Application traffic to those services across AZs is still charged.
- Regional traffic patterns: keep compute next to data. Moving a 50 TB daily analytics job to the data's Region is cheaper than pulling the data across Regions. Use S3 Replication only for data that must exist in two Regions.
- Direct Connect data transfer out is charged at a lower per-GB rate than internet egress, plus port hours. For large steady egress to on premises, Direct Connect is cheaper than VPN over the internet.

**Exam traps**

- Using an interface endpoint for S3 from EC2 in the same VPC to remove NAT charges. A gateway endpoint does that for free; interface endpoints add hourly and per-GB charges.
- Sharing one NAT gateway across three AZs to save money. The exam often counts the inter-AZ data and the resilience loss against it.
- Assuming VPC peering traffic is free. It is free only within the same AZ; cross-AZ and cross-Region peering traffic is charged at normal data transfer rates (but there is no peering processing charge).
- Assuming the security benefit of PrivateLink also saves money. It saves only when it replaces something more expensive, such as NAT processing at volume.

**Hands-on**

1. **Prove gateway endpoints remove NAT processing.** In a test VPC with a private subnet, launch a small instance and give it a route to a NAT gateway. Download a 1 GB object from S3 in the same Region, then add an S3 gateway endpoint to the private route table and download again. Check VPC Flow Logs or NAT gateway CloudWatch metrics (`BytesOutToDestination`) to confirm the second download skipped the NAT gateway. *Cost/cleanup:* A NAT gateway costs money every hour plus per GB; do this in under an hour. Delete the NAT gateway and release its Elastic IP immediately, then delete the endpoint, instance and VPC.
2. **Find your cross-AZ and NAT spend.** In Cost Explorer, group by Usage Type and filter for usage types containing `DataTransfer-Regional-Bytes` (inter-AZ) and `NatGateway-Bytes`. If your own account has little traffic, read the AWS "Overview of data transfer costs for common architectures" blog post and draw the charge points for a three-tier web app on a diagram. *Cost/cleanup:* Cost Explorer UI is free. Nothing to delete.

### Skill 3.2.3: Configure cost-aware multi-Region data replication

**Key concepts**

- **S3 Cross-Region Replication (CRR)** charges: inter-Region data transfer per GB, replication PUT requests in the destination, destination storage, and optionally S3 Replication Time Control (RTC, a per-GB fee that adds a 15-minute replication SLA and metrics). Use RTC only when an RPO or compliance requirement demands the SLA.
- Reduce CRR cost:
  - Filter replication rules by prefix or tag so only data that must exist in two Regions is copied.
  - Write replicas to a cheaper storage class (for example S3 Standard-IA or S3 Glacier Flexible Retrieval for a backup copy), set in the replication rule.
  - Do not replicate delete markers or old versions unless needed, and apply lifecycle rules in the destination.
  - Use S3 Batch Replication only for one-off backfills of existing objects.
- Same-Region Replication (SRR) has no inter-Region transfer charge; use it for account separation (for example a log archive account) in the same Region.
- **DynamoDB global tables**: you pay for replicated writes in each replica Region, plus inter-Region data transfer, plus storage in each Region. Since November 2024, replicated write pricing equals single-Region write pricing, which made global tables much cheaper. Each extra replica multiplies write cost; add replicas only where users or DR need them.
- DynamoDB global tables now offer multi-Region strong consistency as an option in addition to the default asynchronous, eventually consistent replication. Pick by requirement: default mode for most active-active apps; strong consistency mode only when zero-RPO reads across Regions are required (it has Region set constraints and higher write latency).
- **Aurora Global Database**: storage-level replication to up to several secondary Regions, typically under a second of lag. You pay for replicated write I/O and inter-Region transfer, plus instances in the secondary Regions. A headless secondary cluster (no instances, only storage) reduces DR cost until failover, at the cost of a longer RTO while you add instances.
- **RDS cross-Region read replicas** and **automated backup replication** are cheaper DR options when a longer RPO and RTO are acceptable. AWS Backup cross-Region copy of snapshots is cheaper still for backup-and-restore DR.
- Match replication to RPO: backup copy (hours) is cheapest, pilot light with replicas (minutes) is in the middle, active-active global tables or Aurora Global Database (seconds) costs the most.

**Exam traps**

- Enabling S3 RTC for all buckets "to be safe". It adds a per-GB charge and is only needed when a 15-minute replication SLA is a stated requirement.
- Replicating the whole bucket including logs and temp data when only one prefix is regulated. Filter the rule.
- Adding DynamoDB global table replicas in every Region for read latency when only one Region has users. Each replica bills every write.
- Using Aurora Global Database for a workload with a 24-hour RPO. Snapshot copies with AWS Backup meet that at a fraction of the cost.

**Hands-on**

1. **Cost-aware CRR rule.** Create two versioned buckets in different Regions. Add a replication rule that replicates only the `regulated/` prefix and sets the destination storage class to Standard-IA. Upload objects to `regulated/` and `scratch/` and confirm only the first are replicated, with the expected storage class shown in the destination. Read the replication status header on the source objects. *Cost/cleanup:* A few cents for small objects; Standard-IA has a 30-day minimum storage charge per object. Delete all object versions and both buckets, and remove the IAM replication role.
2. **DynamoDB global table write cost.** Create an on-demand DynamoDB table in one Region, add a replica in a second Region, and write 1,000 items with a script. In CloudWatch compare `ConsumedWriteCapacityUnits` in the source and `ReplicationLatency` metrics, and in the billing console see replicated write request usage in the replica Region. *Cost/cleanup:* Within cents. Delete the replica first, then the table.

### Skill 3.2.4: Design cost-effective network architectures (Transit Gateway and PrivateLink pricing)

**Key concepts**

- **Transit Gateway**: billed per attachment-hour (VPC, VPN, Direct Connect gateway, peering, Connect) plus a per-GB data processing charge on traffic sent from an attachment into the transit gateway. The sending account pays. Traffic arriving from a peering attachment is not charged processing on the receiving transit gateway, but the sending side pays processing and inter-Region transfer. Standard data transfer charges apply on top.
- **VPC peering**: no hourly charge and no data processing charge. Same-AZ peering traffic is free; cross-AZ and inter-Region traffic is charged at normal data transfer rates. Peering is non-transitive and becomes hard to manage beyond tens of VPCs.
- **PrivateLink (interface endpoints or endpoint services)**: the consumer pays per endpoint per AZ per hour plus tiered per-GB processing. For cross-Region PrivateLink, the consumer also pays inter-Region data transfer and the provider pays an hourly charge per remote Region. PrivateLink exposes one service, not a network; it suits SaaS or shared services consumed by many VPCs with overlapping CIDRs.
- **VPC Lattice**: billed per service per hour, per GB processed, and per request. It fits service-to-service traffic across many accounts with auth policies, replacing some Transit Gateway plus load balancer designs.
- Decision rules:
  - A few VPCs with very high traffic between two of them: peering for that pair (no processing charge), even if Transit Gateway connects the rest.
  - Many VPCs needing full or hub routing, centralised inspection or hybrid access: Transit Gateway, and accept processing charges.
  - One service consumed by many VPCs or accounts, overlapping CIDRs, or a provider/consumer model: PrivateLink.
  - Centralised egress through a shared NAT gateway VPC: saves NAT hourly charges across many VPCs, but adds Transit Gateway processing per GB. It wins when VPCs are numerous and egress volume is moderate.
- AWS Network Firewall endpoints have an hourly charge per endpoint plus per-GB processing; centralised inspection behind Transit Gateway pays both TGW and firewall processing, so inspect only traffic that requires it.
- Share one Transit Gateway across accounts with AWS RAM rather than building one per account.

**Exam traps**

- Routing two high-volume VPCs (for example an analytics VPC pulling hundreds of TB a month from a data VPC) through Transit Gateway because "it is the standard hub". A peering connection between them removes the per-GB processing.
- Believing Transit Gateway charges for data on both attachments. Processing is charged once, on the sending attachment.
- Using PrivateLink to connect whole networks for many protocols and services. It is per service; Transit Gateway or peering connects networks.
- Centralising egress through one NAT VPC for two VPCs with heavy traffic. The added TGW processing can exceed the saved NAT hours.

**Hands-on**

1. **Price three topologies.** In the AWS Pricing Calculator, model 10 VPCs in one Region where two VPCs exchange 200 TB per month and the rest exchange 5 TB per month in total. Price (a) all through Transit Gateway, (b) Transit Gateway plus a peering connection for the heavy pair, and (c) full-mesh peering. Note where attachment hours, processing and inter-AZ charges appear. *Cost/cleanup:* Free; nothing to delete. Building a real Transit Gateway costs per attachment-hour, so avoid it unless you need the console experience, and delete attachments within an hour.
2. **PrivateLink endpoint service between two VPCs.** In one VPC, put a tiny nginx instance behind an NLB and create an endpoint service. In a second VPC (with an overlapping CIDR to prove the point), create an interface endpoint in one AZ and curl the service through its DNS name. Check the endpoint's per-AZ billing line the next day. *Cost/cleanup:* NLB hourly, endpoint hourly per AZ and instance hours; keep under two hours. Delete the endpoint, endpoint service, NLB, instance and VPCs.

### Skill 3.2.5: Optimize data transfer costs for AI/ML workloads (cross-Region inference routing)

**Key concepts**

- **Bedrock cross-Region inference** uses inference profiles to route requests from a source Region to other Regions with capacity. Two types:
  - **Geographic** profiles (for example `us.`, `eu.`, `apac.` prefixes) keep processing within one geography. Pricing is based on the source Region.
  - **Global** profiles (`global.` prefix) can route to any supported commercial Region. Global cross-Region inference is priced about 10% lower on input and output tokens than geographic cross-Region inference. It suits workloads without data residency limits.
- With cross-Region inference, CloudWatch and CloudTrail records stay in the source Region. Data is not stored in the destination Region beyond processing. Check whether the compliance framework allows processing in other Regions before choosing global profiles.
- SCPs that deny unused Regions can block cross-Region inference. The documented approaches are to allow the destination Regions for Bedrock in the geography, or for global profiles to allow the `unspecified` requested Region for the inference profile (an SCP can also explicitly deny `global.` profiles to enforce residency).
- **Application inference profiles** wrap a model or a system inference profile and can carry tags, which enables cost allocation by team or application (covered in Task 3.3).
- Data transfer for AI/ML is mostly about where the data and the callers live:
  - Keep RAG data (S3, vector store) and the Bedrock calls in the same Region as the application.
  - Call Bedrock from private subnets through an interface VPC endpoint (bedrock-runtime) to avoid NAT processing for high-volume token traffic.
  - Keep SageMaker AI training data, checkpoints and the training job in the same Region; use S3 gateway endpoints for training data access from VPC-attached jobs.
  - For distributed training, keep instances in one AZ (cluster placement group or the AZ of the Capacity Block) to avoid inter-AZ charges on gradient traffic.
- Prompt caching is scoped to Region, so cross-Region routing can lower the cache hit rate. Weigh the capacity benefit against extra cache writes for cache-heavy workloads.
- Bedrock batch inference reads and writes S3 in the same Region; place input buckets accordingly.

**Exam traps**

- Choosing a global inference profile for an EU regulated workload with an EU-only processing rule. Use an EU geographic profile, even though global is cheaper.
- Building a custom multi-Region router (Lambda plus clients in several Regions) to get throughput. Cross-Region inference profiles do this with no added routing cost and pricing based on the source Region.
- Assuming a Region-deny SCP will allow cross-Region inference automatically. It can block the routed calls; the SCP must account for the destination Regions or the global profile.
- Sending Bedrock traffic from private subnets through a NAT gateway at high volume when an interface endpoint costs less per GB and keeps traffic private.

**Hands-on**

1. **Compare inference profiles.** In the Bedrock console, list system-defined inference profiles in your Region and note which destination Regions each geographic and global profile can route to. Call the Converse API once with a geographic profile ID and once with a global profile ID for the same model. Check CloudTrail in your source Region for both calls and read the fields that show the inference profile. *Cost/cleanup:* A few input and output tokens, a fraction of a cent. Nothing to delete.
2. **Bedrock through a VPC endpoint.** Create an interface endpoint for `bedrock-runtime` in a private subnet with no NAT gateway, enable private DNS, and call the model from a small instance using the AWS CLI. Add an endpoint policy that allows only one model ARN and confirm a call to another model is denied. *Cost/cleanup:* Endpoint hourly per AZ plus instance; finish within an hour. Delete the endpoint and instance.

### Practice questions: Task 3.2

**Question 3.2.1** A retail company serves product images and video segments to customers in North America, Europe and Asia through Amazon CloudFront. Video segments are packaged just in time by a fleet of Amazon EC2 instances in us-east-1, and the packaging fleet is overloaded during flash sales because each regional edge cache requests the same segment from the origin. Product images come from an Amazon S3 bucket in us-east-1. The cache hit ratio for images is low because the distribution forwards all query strings and cookies to the origin. The company wants to reduce origin load and cost with minimal changes to the application. Which combination of actions meets these requirements? (Select TWO.)

- A. Enable CloudFront Origin Shield in us-east-1 for the video packaging origin.
- B. Enable S3 Transfer Acceleration on the image bucket and point CloudFront at the accelerated endpoint.
- C. Create a cache policy for the image behaviour that excludes cookies and includes only the query strings used for image sizing in the cache key.
- D. Deploy additional packaging fleets in eu-west-1 and ap-southeast-1 and use Route 53 latency-based routing as the CloudFront origin.
- E. Enable Origin Shield in ap-southeast-1 for the S3 origin to protect it from Asian traffic.

<details>
<summary>Answer and explanation</summary>

**Correct answer: A, C**

**Why A and C are correct:** Origin Shield adds a single caching layer in front of the packaging origin, so concurrent misses from every regional edge cache collapse into about one origin request. Just-in-time packaging is one of the documented best fits. Choosing us-east-1, the origin's Region, is the recommended placement. For images, a cache policy that drops cookies and unused query strings from the cache key stops each user creating a separate cache entry, which raises the hit ratio without changing the application.

**Why the other options are incorrect:**

- **B:** Transfer Acceleration speeds transfers to a bucket over the edge network and adds a per-GB charge. CloudFront to S3 transfer is already free, and acceleration does nothing for the cache key problem.
- **D:** More packaging fleets in other Regions triple the compute cost and duplicate work. It treats the symptom (origin load) more expensively than collapsing requests with Origin Shield.
- **E:** Origin Shield should sit in the Region with the lowest latency to the origin, here us-east-1. A shield in Asia for a US origin adds latency and charges for cacheable requests from other Regions, and the image problem is the cache key, not origin protection.

**Key takeaway:** Use Origin Shield in the origin's Region for expensive or capacity-limited origins with a global audience, and fix hit ratio through cache key design.

</details>

**Question 3.2.2** A bank runs a data processing platform in private subnets across three Availability Zones in eu-west-1. EC2 instances read about 80 TB per month from Amazon S3 and write about 20 TB per month to Amazon DynamoDB in the same Region. They also call AWS Secrets Manager and Amazon CloudWatch a few thousand times per hour. All traffic currently leaves through a single NAT gateway in one AZ. The security team requires that S3 and DynamoDB traffic stay on the AWS network, and the operations team wants to remove the single-AZ dependency. Which solution is the most cost-effective while meeting these requirements?

- A. Create interface VPC endpoints for S3, DynamoDB, Secrets Manager and CloudWatch in all three AZs, and remove the NAT gateway.
- B. Create gateway VPC endpoints for S3 and DynamoDB on all private route tables, add interface endpoints for Secrets Manager and CloudWatch Logs and monitoring, and deploy one NAT gateway per AZ for any remaining internet egress.
- C. Keep the single NAT gateway, add a second NAT gateway in another AZ for failover, and use S3 bucket policies with the `aws:SourceVpc` condition to restrict S3 access.
- D. Move S3 and DynamoDB traffic through a Transit Gateway to a shared services VPC that hosts centralised interface endpoints for all services used by the platform.

<details>
<summary>Answer and explanation</summary>

**Correct answer: B**

**Why B is correct:** Gateway endpoints for S3 and DynamoDB have no hourly or per-GB charge, and they move 100 TB a month off NAT gateway processing. The traffic stays on the AWS network as required. The low-volume control-plane calls to Secrets Manager and CloudWatch can use interface endpoints so they also stay private. One NAT gateway per AZ with per-AZ route tables removes the single-AZ dependency and avoids inter-AZ charges for remaining egress.

**Why the other options are incorrect:**

- **A:** Interface endpoints for S3 and DynamoDB work but add per-AZ hourly charges and per-GB processing on 100 TB a month, where gateway endpoints are free. The cost is far higher for the same privacy result.
- **C:** The NAT gateways still process every S3 and DynamoDB byte, which is the largest cost. Also, `aws:SourceVpc` only works when traffic arrives through a VPC endpoint, so the policy would deny the NAT-routed traffic rather than secure it.
- **D:** Transit Gateway adds per-GB processing on 100 TB, and interface endpoints add more processing. Gateway endpoints also cannot be used from another VPC through Transit Gateway, so centralising S3 and DynamoDB means paying for interface endpoints.

**Key takeaway:** Gateway endpoints for S3 and DynamoDB are free and should be the first move to cut NAT processing charges.

</details>

**Question 3.2.3** A company operates 40 VPCs across 12 accounts in one Region, connected through an AWS Transit Gateway shared with AWS RAM. A data science VPC pulls about 300 TB per month from a data lake VPC that hosts a self-managed query engine on Amazon EC2. All other inter-VPC traffic totals about 8 TB per month. A new finance review shows Transit Gateway data processing as the top networking charge. The network team must keep central routing and inspection for all other VPCs and wants the smallest design change. Which solution meets these requirements most cost-effectively?

- A. Replace the Transit Gateway with a full mesh of VPC peering connections between all 40 VPCs.
- B. Expose the query engine through an AWS PrivateLink endpoint service and create an interface endpoint in the data science VPC.
- C. Move the data science workload into a separate Region and connect it through Transit Gateway inter-Region peering.
- D. Create a VPC peering connection between the data science VPC and the data lake VPC, add more specific routes for each other's CIDRs, and keep all other VPCs on the Transit Gateway.

<details>
<summary>Answer and explanation</summary>

**Correct answer: D**

**Why D is correct:** VPC peering has no hourly charge and no data processing charge, so the 300 TB flow stops paying Transit Gateway processing. Longest-prefix routes for the peer CIDRs send only this pair's traffic over peering, and everything else stays on Transit Gateway with central routing and inspection. It is a small change to two VPCs. Keeping the two VPCs' heavy traffic within the same AZs where possible also removes inter-AZ charges, since same-AZ peering traffic is free.

**Why the other options are incorrect:**

- **A:** A full mesh of 40 VPCs needs hundreds of peering connections and removes central routing and inspection, which the team must keep.
- **B:** PrivateLink charges per-GB processing on the interface endpoint, plus NLB charges in the provider VPC. At 300 TB per month that processing is still large, so it is more expensive than peering and adds components.
- **C:** Moving to another Region adds inter-Region data transfer on all 300 TB plus Transit Gateway processing on the sending side. It increases cost and latency.

**Key takeaway:** Keep Transit Gateway as the hub, but peer the one or two VPC pairs with very high traffic to avoid per-GB processing.

</details>

**Question 3.2.4** A media company stores 2 PB of content in an Amazon S3 bucket in us-west-2. A regulator requires that the `contracts/` prefix (about 5 TB) have a copy in a second Region with a documented replication SLA of 15 minutes. The rest of the bucket, mostly re-creatable transcodes, needs no second copy. The company also has a DynamoDB table for user sessions that is read and written only in us-west-2, and a DR plan with a 24-hour RPO for that table. The company wants to meet the requirements at the lowest cost. Which combination of actions meets these requirements? (Select THREE.)

- A. Create an S3 replication rule filtered to the `contracts/` prefix with a destination bucket in another Region.
- B. Enable S3 Replication Time Control on the `contracts/` replication rule.
- C. Create a replication rule for the whole bucket and set the destination storage class to S3 Glacier Deep Archive.
- D. Convert the DynamoDB table to a global table with a replica in the DR Region.
- E. Use AWS Backup to take daily backups of the DynamoDB table and copy them to a backup vault in the DR Region.
- F. Enable S3 Transfer Acceleration on the destination bucket to meet the 15-minute SLA.

<details>
<summary>Answer and explanation</summary>

**Correct answer: A, B, E**

**Why A, B and E are correct:** A prefix-filtered rule copies only the 5 TB that must exist in two Regions, so inter-Region transfer and destination storage are paid only for regulated data. S3 Replication Time Control adds the 15-minute replication SLA with metrics and notifications, which is the only way to document that SLA; it is paid per GB, so applying it to only 5 TB keeps the charge small. A 24-hour RPO for the sessions table is met by daily AWS Backup backups with cross-Region copy, which costs far less than continuous replication.

**Why the other options are incorrect:**

- **C:** Replicating 2 PB pays inter-Region transfer on data that needs no second copy. Deep Archive also has a 180-day minimum storage charge, and the option does not provide the 15-minute SLA for contracts.
- **D:** A global table replica bills every write again in the DR Region plus storage and transfer, to give a seconds-level RPO the business does not require.
- **F:** Transfer Acceleration speeds client uploads and downloads over the edge network. It does not affect replication between buckets and provides no SLA.

**Key takeaway:** Replicate only what must be replicated, buy RTC only where an SLA is required, and match database DR to the stated RPO.

</details>

**Question 3.2.5** A European insurer runs a claims assistant on Amazon Bedrock in eu-central-1. During peak hours the application receives throttling errors from on-demand inference. Compliance rules allow processing in any AWS Region within the EU but not outside it. The organisation has an SCP that denies all actions outside eu-central-1 and eu-west-1. The application runs in private subnets and currently reaches Bedrock through a NAT gateway, which shows high data processing charges. The team wants to increase throughput and reduce network cost with the least operational overhead. Which solution meets these requirements?

- A. Switch the application to a global cross-Region inference profile for the model, which costs less per token, and add the `unspecified` Region to the SCP allow list.
- B. Deploy the application in a second EU Region and use Route 53 weighted routing to spread requests, keeping the NAT gateways in both Regions.
- C. Switch the application to the EU geographic cross-Region inference profile, update the SCP so that Bedrock actions are allowed in the EU destination Regions of that profile, and create an interface VPC endpoint for bedrock-runtime.
- D. Purchase Provisioned Throughput with no commitment in eu-central-1 for peak hours and create an S3 gateway endpoint for the application subnets.

<details>
<summary>Answer and explanation</summary>

**Correct answer: C**

**Why C is correct:** An EU geographic inference profile routes requests across EU Regions only, which raises available throughput while meeting the EU processing rule. Pricing is based on the source Region, and no custom routing is needed. Because the current SCP denies Regions the profile may route to, it must allow Bedrock in those EU destination Regions. An interface endpoint for bedrock-runtime removes NAT gateway processing for the token traffic and keeps calls private.

**Why the other options are incorrect:**

- **A:** A global profile is about 10% cheaper per token but can route to Regions outside the EU, which breaks the compliance rule. Allowing `unspecified` in the SCP would open that path.
- **B:** A second application deployment doubles infrastructure and operations, and NAT gateway processing remains in both Regions. It does not reuse Bedrock's built-in routing.
- **D:** Provisioned Throughput bills every hour it exists and needs to be created and deleted around peaks, which adds operational work. An S3 gateway endpoint does nothing for Bedrock traffic, so NAT charges stay.

**Key takeaway:** Use geographic cross-Region inference when residency rules apply, global inference for the lowest token price when they do not, and adjust Region-deny SCPs to match.

</details>


## Task 3.3: Design cost allocation and visibility solutions

This task tests whether you can make every dollar attributable to an owner across a multi-account organisation, enforce the metadata that makes that possible, and alert before overspend becomes a surprise. The recurring decision is which mechanism (account structure, tags, cost categories, split cost allocation, CUR 2.0, Budgets, Cost Anomaly Detection) answers the specific allocation or alerting need with the least custom work.

### Skill 3.3.1: Design tagging strategies for cost allocation

**Key concepts**

- **Account structure first, tags second.** Separate accounts per business unit, product or environment give clean allocation with no tagging gaps, because the linked account is always on every line item. Use tags for allocation inside an account (shared platform accounts, data lakes, EKS clusters).
- Define a small mandatory tag set with fixed keys and allowed values, for example `CostCenter`, `Project` or `Application`, `Environment` (`prod`, `nonprod`, `sandbox`), `Owner`. Publish the dictionary and enforce it (Skill 3.3.3).
- Tag keys and values are case-sensitive. `CostCenter` and `costcenter` are different keys and show as separate groups in Cost Explorer.
- **Cost allocation tags** must be activated in the management (payer) account in Billing and Cost Management before they appear in Cost Explorer, Budgets and CUR as allocation dimensions. Activated tags apply from activation onward; AWS offers a backfill request for earlier months, but plan activation early. New tags can take up to a day to appear for activation.
- **AWS-generated tags** such as `aws:createdBy` can be activated to show who created a resource. Service-generated tags like `aws:eks:cluster-name` arrive with split cost allocation data.
- Some resources and charges cannot carry tags (for example some data transfer and support charges). Plan to handle them with cost categories and split charge rules, not by expecting 100% tag coverage.
- Propagate tags automatically: CloudFormation stack-level tags propagate to supported resources, ECS supports `propagateTags` from the service or task definition and ECS managed tags, Auto Scaling groups can propagate tags to instances, and AWS Backup can copy tags to recovery points.
- Tag Editor and the Resource Groups Tagging API find untagged resources across Regions. AWS Config `required-tags` shows non-compliant resources.

**Exam traps**

- Expecting a tag created in a member account to show in Cost Explorer automatically. Only the management account can activate it as a cost allocation tag.
- Tagging alone to separate production and development costs in one shared account, when the scenario allows a new account structure. Separate accounts give guaranteed allocation and also give blast-radius separation.
- Using tag values with free text (for example project names typed by hand). Inconsistent values defeat grouping; allowed values must be enforced.
- Assuming every charge can be tagged. Shared charges like Support, some data transfer and untaggable resources need cost category split rules.

**Hands-on**

1. **Activate and use a cost allocation tag.** Tag a few resources (an S3 bucket, a Lambda function, an EC2 instance) with `CostCenter` values `A` and `B`. In the management account (or your single account) open Cost allocation tags, activate `CostCenter` and the AWS-generated `aws:createdBy` tag. After a day, group Cost Explorer by the tag and look at the "No tag key" group. *Cost/cleanup:* Activation is free; resources cost cents. Delete the test resources.
2. **Find untagged resources.** Use Tag Editor to search all Regions for resources missing `CostCenter`, then deploy the AWS Config managed rule `required-tags` for `CostCenter` and check its compliance view. *Cost/cleanup:* AWS Config charges per configuration item recorded and per rule evaluation; limit recording to a few resource types and delete the rule and recorder afterwards.

### Skill 3.3.2: Configure cost reporting and analytics with detailed billing data

**Key concepts**

- **AWS Cost Explorer**: interactive analysis with filters and group-by on service, account, Region, usage type, tag and cost category. It shows about 13 months of history by default, forecasts, RI and Savings Plans recommendations, and coverage and utilisation. Hourly granularity and resource-level data are opt-in features with a charge for hourly data. The Cost Explorer API has a per-request charge.
- **AWS Data Exports** delivers billing data to Amazon S3 on a recurring schedule. Export tables include:
  - **CUR 2.0** (the recommended detailed cost and usage data).
  - **FOCUS** with AWS columns (the FinOps Open Cost and Usage Specification format, useful for multi-cloud tools).
  - **Cost optimization recommendations** from Cost Optimization Hub.
  - **Carbon emissions**.
  - A cost and usage dashboard option that deploys a prebuilt Amazon QuickSight dashboard.
  - Legacy CUR is still available but CUR 2.0 is preferred for new builds.
- **CUR 2.0 versus legacy CUR**:
  - CUR 2.0 has a fixed schema. Legacy CUR adds columns as tags and cost categories appear, which breaks downstream schemas.
  - CUR 2.0 stores tags, cost categories, product and discount attributes in nested key-value columns.
  - CUR 2.0 adds columns such as account names, `line_item_iam_principal` and user identifier fields.
  - CUR 2.0 lets you choose columns, filter rows and rename columns with basic SQL in the export definition (for example to drop a sensitive account or a large unused column set).
  - Formats are Parquet or compressed CSV. You query with Amazon Athena, load into Amazon Redshift, or visualise in QuickSight.
- **Cloud Intelligence Dashboards** (CUDOS and others) are AWS-provided QuickSight templates on top of CUR or Data Exports for organisation-wide FinOps views.
- Pick CUR or Data Exports when you need line-item, resource-level, amortised or custom analysis; Cost Explorer for interactive questions; Budgets for thresholds.
- Amortised versus unblended cost: amortised spreads upfront RI and Savings Plans fees across the term and is the right basis for showback. Net amortised also reflects discounts.
- Billing data access: by default only the management account sees organisation-wide data. Use IAM policies for Billing and Cost Management actions, and give member-account owners only the data they need, for example through Data Exports with SQL row filters delivered to their own bucket.

**Exam traps**

- Building a Lambda that calls the Cost Explorer API every hour to populate a data warehouse. Data Exports delivers the full detail to S3 without API charges or custom code.
- Choosing legacy CUR for a pipeline that keeps breaking when new tags appear. CUR 2.0's fixed schema with nested tag columns solves that.
- Using unblended cost for team showback when the organisation bought large All Upfront Savings Plans. Teams see near-zero cost in some months; use amortised cost.
- Expecting Cost Explorer to give per-pod or per-task cost for shared clusters. That needs split cost allocation data in CUR (Skill 3.3.4).

**Hands-on**

1. **Create a CUR 2.0 export and query it.** In Data Exports, create a standard export with the CUR 2.0 table, Parquet format, daily granularity, include resource IDs, delivered to a new S3 bucket. Use the provided CloudFormation template or AWS Glue crawler to make it queryable in Athena, then run a query that sums `line_item_unblended_cost` grouped by `line_item_product_code` and by a tag key from the `resource_tags` map. *Cost/cleanup:* Data Exports itself has no charge; you pay for S3 storage and Athena data scanned (pennies). Delete the export, crawler, database and bucket when done.
2. **Use SQL in the export definition.** Create a second export using the CLI with a SQL statement that selects only a few columns and filters out one usage account ID or one service. Compare file sizes in S3 with the full export. Read the CUR 2.0 table dictionary page for column names before you write the query. *Cost/cleanup:* Same as above; delete the export and bucket.

### Skill 3.3.3: Implement policy-based tagging enforcement

**Key concepts**

- **Tag policies** (AWS Organizations) standardise tag keys (including capitalisation) and allowed values. When you enable enforcement for specific resource types (`enforced_for`), a create or tag operation with a non-compliant value for that key is blocked. Tag policies do **not** require a tag to be present: a resource created without the tag is not blocked by basic tag policy rules. Tag policies also produce organisation-wide compliance reports.
- Tag policies also support a **required tag keys** reporting mode aimed at infrastructure-as-code tools (CloudFormation, Terraform, Pulumi), which reports resources deployed without required keys.
- **SCPs** enforce presence. Deny `ec2:RunInstances` (and other create actions) when `aws:RequestTag/CostCenter` is null (`"Null": {"aws:RequestTag/CostCenter": "true"}`), and deny `ec2:DeleteTags` or `ec2:CreateTags` changes to protected keys except by an admin role. The condition keys only work for services and actions that support tag-on-create.
- Combine them: SCP for "must be present", tag policy for "must be one of these values with this exact case", Config for "find and report the ones that slipped through or predate the policy".
- **AWS Config rules**: `required-tags` (managed rule) detects missing tags on supported resource types after creation. Pair with automatic remediation through Systems Manager Automation to add a default tag or notify the owner. Use a conformance pack or an organisation Config rule to deploy across accounts.
- **Proactive controls**: AWS Control Tower proactive controls and CloudFormation Hooks can check templates for tags before deployment. AWS CDK `Tags.of(scope).add()` and aspects add tags consistently at synthesis time.
- Test enforcement in a test OU first. Strict enforcement can break services that create resources on your behalf (for example Auto Scaling, EMR, or service-linked roles) if they do not pass the tags.

**Exam traps**

- Using tag policies alone to "prevent creation of untagged resources". They validate supplied tags; they do not require them. An SCP with `aws:RequestTag` and a `Null` condition is required.
- Using an SCP with `aws:RequestTag` for a resource type that does not support tag-on-create. The condition never matches or blocks all creates. Check the service authorisation reference.
- Using AWS Config to block creation. Config is detective (plus remediation after the fact); it does not stop the API call. Proactive blocking needs SCPs, tag policies, Hooks or proactive controls.
- Expecting a deny SCP attached at the root to enforce tagging in the management account. SCPs do not apply to the management account, so enforcement there needs IAM policies (and the account should hold no workloads).

**Hands-on**

1. **SCP plus tag policy in a test organisation.** If you can create a second free account, create an organisation with a test OU. Attach a tag policy that defines `Environment` with values `prod` and `nonprod`, enforced for `ec2:instance`, and an SCP that denies `ec2:RunInstances` on instances and volumes when `aws:RequestTag/Environment` is null. From the member account, try launching an instance with no tag, with `Environment=test`, and with `Environment=prod`. *Cost/cleanup:* Organizations, SCPs and tag policies are free; a t4g.nano for a minute costs almost nothing. Detach the policies and terminate the instance. If you only have one account, read the AWS Organizations "Understanding enforcement" page and the SCP tag examples instead.
2. **Detect and remediate missing tags.** Deploy the `required-tags` Config rule for `Environment` and attach automatic remediation with a Systems Manager Automation runbook (an AWS-provided tagging runbook if one fits, or a small custom runbook that calls the tagging API) that adds `Environment=unknown`. Create an untagged S3 bucket and watch it become non-compliant and then remediated. *Cost/cleanup:* Config items and evaluations cost a little; delete the rule, remediation configuration, bucket and (if you created it) the recorder.

### Skill 3.3.4: Implement chargeback and showback (cost categories, split cost allocation, Bedrock token usage)

**Key concepts**

- **Showback** reports costs to teams without moving money; **chargeback** bills them internally. Both need allocation rules everybody accepts, usually amortised cost.
- **AWS Cost Categories** map costs to business groupings with rules on linked account, service, charge type, tag, and other cost categories. A cost category becomes a dimension in Cost Explorer, Budgets, Cost Anomaly Detection and CUR. Use them to model your org chart (for example `BusinessUnit` built from accounts plus a tag in shared accounts).
- **Split charge rules** within a cost category spread a shared cost (central networking, Support, security tooling, untagged shared services) across target values proportionally, evenly or by fixed percentages. This is how you allocate the parts tags cannot reach.
- **Split cost allocation data** (opt-in in Billing and Cost Management preferences) adds container-level cost rows to CUR for Amazon ECS tasks (EC2 and Fargate), AWS Batch, and Amazon EKS pods. Cost is based on the instance's amortised cost and each task's or pod's share of CPU and memory (and for EKS on GPU, Trainium and Inferentia instances, accelerator use). Unused instance capacity is reported and allocated proportionally. EKS rows carry tags such as `aws:eks:cluster-name`, `aws:eks:namespace` and `aws:eks:workload-name`, which you can use for namespace-level showback. ECS rows use ECS managed tags or your own tags.
- Without split cost allocation, a shared EKS or ECS-on-EC2 cluster shows only EC2 instance cost and needs a third-party tool or Kubecost-style estimates to allocate by team.
- **Bedrock cost allocation options**:
  - **Separate accounts** per team or application: simplest, works for everything.
  - **Application inference profiles**: create a profile per team or application around a model, tag it, and pass its ARN as the `modelId`. Activated tags then show Bedrock cost by team in Cost Explorer and CUR, at the same per-token rate.
  - **IAM principal cost allocation** (2026): tag IAM users and roles with attributes such as team or cost center, activate those tags, and enable the "include caller identity (IAM principal) allocation data" option in a CUR 2.0 export. Costs are then attributed by caller in CUR 2.0 and Cost Explorer. It cannot separate users who share one application role.
  - **Request-level metadata**: tags attached to individual inference calls appear in model invocation logs, not in Cost Explorer or CUR. Use them for fine-grained usage analysis (for example per end customer in a SaaS app) by processing the logs.
  - **CloudWatch metrics** such as `InputTokenCount`, `OutputTokenCount` and `Invocations` by model give near-real-time token monitoring and alarms, and model invocation logging to S3 or CloudWatch Logs gives per-request detail.
- For a multi-tenant SaaS on Bedrock that bills customers per token, compute tenant usage from invocation logs or the response `usage` fields recorded by the application, because billing data does not know tenant IDs.

**Exam traps**

- Tagging the EC2 nodes of a shared EKS cluster by team to allocate cost. Nodes are shared by many teams' pods; split cost allocation data is the AWS-native answer.
- Expecting Bedrock costs to split by team using tags on the calling Lambda function. The Bedrock charge is not a Lambda charge; use application inference profiles, IAM principal allocation or separate accounts.
- Using request-level metadata tags for chargeback in Cost Explorer. They appear only in invocation logs.
- Creating dozens of custom Lambda jobs to divide shared networking cost. Cost category split charge rules do it natively.

**Hands-on**

1. **Cost category with a split charge.** Create a cost category `Team` with rules mapping your accounts or a `CostCenter` tag to `TeamA` and `TeamB`, plus a value `Shared` for a networking service or untagged costs. Add a split charge rule that splits `Shared` proportionally across `TeamA` and `TeamB`. View it in Cost Explorer grouped by the cost category after it processes. *Cost/cleanup:* Cost categories are free. Delete it when done.
2. **Bedrock application inference profiles for showback.** Create two application inference profiles for the same small model with `CreateInferenceProfile`, tagged `Team=Alpha` and `Team=Beta`. Call Converse a few times with each profile ARN as `modelId`, activate the `Team` cost allocation tag, and after a day or two group Bedrock cost by `Team` in Cost Explorer. Also view `InputTokenCount` in CloudWatch for each profile. *Cost/cleanup:* A few cents of tokens. Delete the profiles.

### Skill 3.3.5: Configure cost monitoring and alerting with budget thresholds and anomaly detection

**Key concepts**

- **AWS Budgets** types: cost, usage, Savings Plans and RI utilisation, and Savings Plans and RI coverage budgets. Budgets can be fixed, planned per period, or auto-adjusting from history. Thresholds can be on actual or forecasted spend. Alerts go to email, SNS topics, or chat through Amazon Q Developer in chat applications.
- **Budget actions** run automatically or after approval when a threshold is crossed: apply an IAM policy, apply an SCP to an account or OU, or stop specific EC2 or RDS instances. Use them for sandbox accounts and hard spending caps. Budgets do not stop billing on their own; actions are what enforce.
- Budgets can be filtered by account, service, tag, cost category and more. Create per-team budgets on cost categories for showback alerting.
- Budget data refreshes several times a day, not in real time; it is a guardrail, not a meter.
- **AWS Cost Anomaly Detection** uses machine learning on your spend history to find unusual spend and root causes. It is free to use.
  - **AWS managed monitors** for services, linked accounts, cost allocation tags or cost categories track every value of that dimension independently and pick up new values automatically.
  - **Customer managed monitors** watch a selected set (for example up to 10 accounts or tag values) aggregated.
  - **Alert subscriptions**: individual alerts (immediate, through SNS), or daily or weekly email summaries. Thresholds can be an absolute dollar impact, a percentage impact, or both combined with AND or OR.
  - Monitors need some history and can take up to a day to start alerting.
- Use Budgets for known limits ("this account must not exceed X per month"), Cost Anomaly Detection for unknown problems ("a new service suddenly costs 5 times normal"). Most good designs use both.
- Route alerts into operations tooling: SNS to a Lambda that opens a ticket, or EventBridge integration with AWS User Notifications.
- Near-real-time guards for specific services: CloudWatch billing alarms (estimated charges metric in us-east-1), service-level CloudWatch alarms on usage (for example Bedrock token counts or Lambda invocations), and service quotas as a hard cap.

**Exam traps**

- Using a fixed monthly budget to catch a sudden spike in a new service. The budget alerts only when the total crosses the threshold, perhaps weeks later; anomaly detection catches the pattern change.
- Expecting AWS Budgets to stop resources without configuring a budget action. A budget alone only notifies.
- Budget actions that stop instances in production. Use them in sandbox or development accounts; for production, alert and route to people.
- Building a custom ML model on CUR data to detect cost spikes. Cost Anomaly Detection does this for free with no code.

**Hands-on**

1. **Budget with an action.** Create a monthly cost budget of a small amount for your account, with an alert at 80% forecasted to email and an action at 100% actual that applies an IAM policy denying `ec2:RunInstances` to a test role, requiring approval. Trigger it by setting a tiny budget amount and check the pending action. *Cost/cleanup:* The first budgets are free and budget actions have a small charge per budget per day after the free allowance; check the Budgets pricing page. Delete the budget and the test role.
2. **Cost Anomaly Detection monitors.** Create an AWS managed monitor for AWS services and another for a cost allocation tag. Add an individual-alert subscription to an SNS topic with an absolute threshold of a few dollars OR a 40% threshold, and a daily summary to your email. Wait for at least a day and review the Detected anomalies tab and the root cause fields if any appear. *Cost/cleanup:* Free. Delete the monitors, subscription and SNS topic.

### Practice questions: Task 3.3

**Question 3.3.1** A bank has 120 AWS accounts in AWS Organizations. Most product teams have their own accounts, but a shared data platform account runs Amazon EMR, Amazon S3 and AWS Glue for eight business units. The finance team needs monthly showback by business unit that covers all accounts and the shared account. It must also allocate the central networking account's Transit Gateway and NAT gateway charges to business units in proportion to their other spend. Teams already tag resources in the shared account with a `BusinessUnit` tag. Which solution meets these requirements with the least operational effort?

- A. Activate the `BusinessUnit` cost allocation tag in the management account, create a cost category that maps product accounts and the tag values in the shared account to business units, and add a proportional split charge rule for the networking account's costs.
- B. Move each business unit's EMR and Glue jobs into separate accounts so that allocation is by account only, and charge the networking account's costs to the central IT cost centre.
- C. Create a CUR 2.0 export and an Amazon Athena view that joins account IDs to a business unit lookup table, and write a scheduled Lambda function that redistributes networking costs by ratio.
- D. Create one AWS Budget per business unit filtered by the `BusinessUnit` tag and send the monthly budget reports to the finance team as the showback.

<details>
<summary>Answer and explanation</summary>

**Correct answer: A**

**Why A is correct:** Activating the tag in the management account makes `BusinessUnit` usable in billing tools. A cost category combines rules on linked accounts (for product accounts) and on the tag (for the shared account) into one `BusinessUnit` dimension. A split charge rule spreads the networking account's cost across business units in proportion to their other spend, with no custom code. The result appears in Cost Explorer and CUR, so finance can report monthly.

**Why the other options are incorrect:**

- **B:** Splitting the shared platform into eight accounts is a large migration and more operational effort, and charging networking to central IT fails the proportional allocation requirement.
- **C:** This works but needs custom SQL, a lookup table to maintain, and a scheduled function, which is more operational effort than native cost categories and split charges.
- **D:** Budgets alert on thresholds; they are not an allocation model. They cannot allocate the networking account's cost by ratio, and tag-filtered budgets miss product accounts whose resources are not tagged.

**Key takeaway:** Cost categories combine account and tag rules into business dimensions, and split charge rules allocate shared costs natively.

</details>

**Question 3.3.2** A company's FinOps team loads the legacy AWS Cost and Usage Report into Amazon Redshift each day. The load job breaks several times a month because new tag keys and cost categories add columns to the report. The team also needs to remove one subsidiary's accounts from the data before sharing it with a vendor, and wants a standard format that its multi-cloud FinOps tool can ingest. Which solution meets these requirements with the least ongoing maintenance?

- A. Keep the legacy CUR, add an AWS Glue crawler before each load to detect schema changes, and use a Redshift view that excludes the subsidiary's account IDs.
- B. Use the Cost Explorer API to extract daily cost by account, service and tag into Amazon S3, and transform it with an AWS Glue job into the vendor's format.
- C. Create a separate legacy CUR in each subsidiary's management account and merge the files in Amazon S3 with an AWS Lambda function before loading Redshift.
- D. Create AWS Data Exports for CUR 2.0 with a SQL statement that filters out the subsidiary's account IDs, and create a separate FOCUS export for the multi-cloud tool.

<details>
<summary>Answer and explanation</summary>

**Correct answer: D**

**Why D is correct:** CUR 2.0 has a fixed schema, with tags and cost categories held in nested key-value columns, so new tag keys no longer add columns and break the load. Data Exports lets you define the export with SQL that filters rows, so the subsidiary's accounts are removed before the data lands in S3. The FOCUS export table provides the open FinOps format that multi-cloud tools ingest, with no custom transformation.

**Why the other options are incorrect:**

- **A:** A crawler detects changes but the target Redshift tables still need altering when columns appear, so the pipeline still needs attention. The legacy CUR keeps its dynamic schema.
- **B:** The Cost Explorer API charges per request, returns aggregated data rather than line items, and needs a custom transformation into the vendor's format.
- **C:** Splitting reports by management account only works if the subsidiary is a separate organisation, and the merge function adds custom code while keeping the dynamic schema problem.

**Key takeaway:** For durable billing pipelines, use Data Exports: CUR 2.0 for a fixed schema with SQL filtering, and FOCUS for cross-cloud tools.

</details>

**Question 3.3.3** A company wants every new Amazon EC2 instance and Amazon EBS volume in its workload accounts to carry a `CostCenter` tag whose value is one of the approved codes, with exact capitalisation of the key. Developers deploy with AWS CloudFormation and the AWS CLI. Many existing resources were created before the rule and must be reported for cleanup but not deleted. Which combination of actions meets these requirements? (Select TWO.)

- A. Attach a tag policy that defines `CostCenter` with the approved values and enforces it for EC2 instances and volumes, and rely on it to block resources created without the tag.
- B. Deploy an AWS Config rule that triggers an AWS Lambda function to terminate any EC2 instance without a `CostCenter` tag.
- C. Attach an SCP to the workload OUs that denies `ec2:RunInstances` and `ec2:CreateVolume` when `aws:RequestTag/CostCenter` is null, together with a tag policy that enforces the key case and the approved values for EC2 instances and volumes.
- D. Activate `CostCenter` as a cost allocation tag in each member account so that untagged resources are visible in each account's Cost Explorer.
- E. Deploy the AWS Config managed rule `required-tags` for `CostCenter` across the organisation with a conformance pack, and use its compliance results to report existing non-compliant resources.

<details>
<summary>Answer and explanation</summary>

**Correct answer: C, E**

**Why C and E are correct:** The SCP blocks create calls that do not include the tag, which enforces presence for both CloudFormation and CLI because both call the same APIs. The tag policy blocks values outside the approved list and enforces the key's capitalisation for the enforced resource types. Together they cover "must exist" and "must be valid". The Config `required-tags` rule is detective: it reports existing resources missing the tag without deleting anything, which meets the cleanup reporting requirement.

**Why the other options are incorrect:**

- **A:** Basic tag policy enforcement checks values and case of tags that are supplied. It does not block creating a resource without the tag, so presence is not enforced.
- **B:** Terminating instances violates the requirement that existing resources be reported but not deleted, and it is detective after the fact rather than preventive.
- **D:** Cost allocation tags are activated in the management account only, and activation does not enforce or validate anything.

**Key takeaway:** SCPs enforce tag presence, tag policies enforce values and case, and AWS Config reports what already exists.

</details>

**Question 3.3.4** A software company runs a shared Amazon EKS cluster on Amazon EC2 nodes for 15 product teams, each in its own Kubernetes namespace. The company also runs a generative AI feature in which four internal applications call Amazon Bedrock models from the same AWS account, each application with its own IAM role. Finance wants monthly showback of the EKS cluster cost by namespace and of Bedrock cost by application, using AWS native billing data and no third-party tools. Which solution meets these requirements?

- A. Tag each EKS node group with the name of the team that uses it most, activate the tag, and allocate Bedrock costs using CloudWatch `InputTokenCount` metrics multiplied by the published price in a spreadsheet.
- B. Enable split cost allocation data for Amazon EKS and include it in a CUR 2.0 export, then group pod costs by the `aws:eks:namespace` tag. Tag each application's IAM role with an `Application` tag, activate it, and enable IAM principal allocation data in the CUR 2.0 export.
- C. Install a cost monitoring agent in the cluster that writes per-namespace usage to Amazon S3, and add request-level metadata tags to every Bedrock InvokeModel call so that costs show in Cost Explorer by application.
- D. Move each product team to its own EKS cluster in a separate account, and move each Bedrock application to its own account so that costs separate by linked account.

<details>
<summary>Answer and explanation</summary>

**Correct answer: B**

**Why B is correct:** Split cost allocation data adds pod-level rows to CUR based on each pod's share of node CPU and memory, including unused capacity, and tags them with EKS attributes such as `aws:eks:namespace`, which gives namespace showback from native billing data. For Bedrock, IAM principal cost allocation attributes inference cost to the calling role by its tags in CUR 2.0 and Cost Explorer. Because each application has its own role, an `Application` tag on the role separates costs by application.

**Why the other options are incorrect:**

- **A:** Node groups are shared by many namespaces, so tagging them by the heaviest user misallocates cost. A spreadsheet of token counts times list price is manual and ignores discounts and tier pricing.
- **C:** An in-cluster agent is a third-party or custom tool, which the requirement excludes. Request-level metadata tags appear in model invocation logs, not in Cost Explorer or CUR.
- **D:** This would work for allocation, but it is a large re-architecture into 19 or more accounts and clusters, which is far beyond what showback needs.

**Key takeaway:** Use split cost allocation data for shared container clusters, and application inference profiles or IAM principal allocation for Bedrock costs within one account.

</details>

**Question 3.3.5** A company gives each of its 300 developers a sandbox account through AWS Control Tower Account Factory. Last quarter, two sandboxes ran large GPU instances for weeks before anyone noticed. Leadership wants each sandbox capped at a fixed monthly amount with automatic enforcement when the cap is reached. The central cloud team also wants to be notified quickly of unusual spend in any account in the organisation, including in production accounts and new services, without writing custom code. Which solution meets these requirements?

- A. Create an AWS Budget for each sandbox account with a budget action that applies a restrictive SCP when actual spend reaches the cap, and create an AWS managed Cost Anomaly Detection monitor for linked accounts with individual alerts sent to an Amazon SNS topic.
- B. Create a CloudWatch billing alarm on the `EstimatedCharges` metric for each sandbox account that invokes an AWS Lambda function to terminate all instances, and enable AWS Trusted Advisor cost checks for the organisation.
- C. Use an SCP that denies all GPU instance types in sandbox OUs, and create a single organisation-wide AWS Budget with an alert at 80% of forecasted spend.
- D. Enable AWS Compute Optimizer for all accounts and send its idle resource findings to the cloud team weekly, and set Service Quotas for vCPU limits in each sandbox to a low value.

<details>
<summary>Answer and explanation</summary>

**Correct answer: A**

**Why A is correct:** A per-account budget gives each sandbox its own cap, and a budget action can apply an SCP to that account automatically when actual spend crosses the threshold, which stops further resource creation. Budgets and actions can be created as part of account vending. An AWS managed Cost Anomaly Detection monitor on linked accounts evaluates every account independently, picks up new accounts automatically, finds unusual patterns including new services, and sends individual alerts through SNS soon after detection. No custom code is needed.

**Why the other options are incorrect:**

- **B:** The terminate-everything Lambda is custom code and destroys data without approval. Billing alarms per account need the metric enabled and managed in each account, and Trusted Advisor cost checks are not anomaly detection.
- **C:** Denying GPU types prevents one cause but does not cap spend on anything else. A single organisation budget cannot enforce a per-sandbox cap or detect unusual spend in a specific account.
- **D:** Compute Optimizer findings are weekly rightsizing advice, not spend enforcement. Low vCPU quotas limit instance count but not cost from other services, and quota changes are not tied to a monthly amount.

**Key takeaway:** Use AWS Budgets with budget actions for known caps and enforcement, and Cost Anomaly Detection managed monitors for unknown spikes across the organisation.

</details>


## Task 3.4: Optimize storage and database costs

This task tests whether you can choose storage classes, volume types, database capacity modes and retention rules that match real access patterns, without breaking performance or recovery objectives. The recurring decision is "is the access pattern known or unknown, steady or spiky", because known patterns favour explicit lifecycle rules and commitments, and unknown patterns favour automatic tiering and on-demand modes.

### Skill 3.4.1: Implement intelligent storage tiering and lifecycle policies

**Key concepts**

- **S3 storage classes by access pattern**:
  - **S3 Standard**: frequent access.
  - **S3 Standard-IA** and **S3 One Zone-IA**: infrequent access with millisecond retrieval, per-GB retrieval fee, 30-day minimum storage duration and a 128 KB minimum billable object size. One Zone-IA stores data in one AZ only; use it for re-creatable data.
  - **S3 Glacier Instant Retrieval**: data accessed about once a quarter, millisecond retrieval, 90-day minimum.
  - **S3 Glacier Flexible Retrieval**: minutes to hours retrieval (expedited, standard, bulk), 90-day minimum.
  - **S3 Glacier Deep Archive**: lowest cost, retrieval within hours (standard within 12 hours, bulk within 48 hours), 180-day minimum.
  - **S3 Express One Zone**: single-AZ, high-performance directory buckets for latency-sensitive hot data; not a cost tier for cold data.
- **S3 Intelligent-Tiering** moves objects between Frequent, Infrequent (after 30 days without access) and Archive Instant Access (after 90 days) automatically, with no retrieval fees. Optional asynchronous Archive Access and Deep Archive Access tiers can be enabled for objects not read for longer periods (configurable), and objects there must be restored before reading. It charges a small per-object monitoring and automation fee. Objects smaller than 128 KB are not monitored or tiered and are always billed at the Frequent tier rate.
- Use Intelligent-Tiering when access patterns are unknown or change, for data lakes and user-generated content. Use explicit lifecycle rules when the pattern is known (logs read for 30 days then kept for 7 years for compliance).
- **Lifecycle rules**: transition actions (by age, or noncurrent version age) and expiration actions. Filter by prefix, tag or object size. Also:
  - Expire noncurrent versions in versioned buckets, often the hidden cost.
  - Clean up incomplete multipart uploads (`AbortIncompleteMultipartUpload`).
  - Delete expired delete markers.
  - Lifecycle transitions charge a per-request fee per object, so transitioning millions of tiny objects can cost more than it saves. Filter by minimum object size or aggregate small files first.
  - S3 applies a default minimum object size of 128 KB for transitions unless you override it.
- **S3 Storage Lens** gives organisation-wide visibility into storage by bucket, prefix and class, incomplete uploads and noncurrent versions; advanced metrics are paid. **S3 Storage Class Analysis** observes a bucket or prefix and recommends when to transition to Standard-IA.
- **Amazon EFS storage classes**: Standard, Infrequent Access (IA), and Archive. Lifecycle management has three policies on the whole file system:
  - Transition into IA after N days without access (default 30 days).
  - Transition into Archive after N days without access (default 90 days).
  - Transition back into Standard on first access, or not (default is to stay in IA or Archive).
  - IA and Archive charge per-GB for reads (and data written into those classes is first written to Standard). Archive is for data read a few times a year or less. Metadata always stays in Standard.
  - Choose "transition into Standard on first access" for workloads with many small files that become hot again; leave it off for genuinely cold data so you do not pay Standard storage after a one-off read.
- EFS throughput modes affect cost too: Elastic throughput (pay per GB transferred) suits spiky workloads; Provisioned throughput suits steady high throughput; Bursting ties throughput to storage size.
- **FSx**: FSx for Windows File Server and FSx for NetApp ONTAP support data deduplication and compression, and ONTAP has a capacity pool tier for cold data. FSx for Lustre can link to S3 so only hot data sits on the file system.

**Exam traps**

- Lifecycle rule to Glacier Deep Archive for objects that are read every few weeks. Retrieval fees and hours of delay break the workload, and the 180-day minimum adds charges if objects are deleted early.
- Intelligent-Tiering for millions of objects smaller than 128 KB. They never tier, and the cost case collapses; aggregate them first.
- Transitioning to Standard-IA after 7 days. Lifecycle does not allow Standard-IA or One Zone-IA transitions before 30 days of object age.
- One Zone-IA for the only copy of irreplaceable data. It stores data in a single AZ.
- Forgetting noncurrent versions. A versioned bucket with frequent overwrites can store many times its visible size.

**Hands-on**

1. **Lifecycle rule that targets the real waste.** Create a versioned bucket. Upload and overwrite the same objects several times, start a multipart upload and abandon it. Add one lifecycle rule that transitions current objects larger than 128 KB to Standard-IA at 30 days and Glacier Flexible Retrieval at 90 days, expires noncurrent versions after 30 days keeping the 3 newest, aborts incomplete multipart uploads after 7 days and removes expired delete markers. Enable S3 Storage Lens free metrics and note the noncurrent version bytes. *Cost/cleanup:* Cents. Delete all versions, then the bucket. Transitions only happen after the age you set, so you will see the rule, not the transition, in a short test.
2. **EFS lifecycle with Archive.** Create an EFS file system with Elastic throughput, set transition into IA at 1 day, transition into Archive at 90 days, and transition into Standard on first access. Mount it from a small instance, write a few hundred MB, and watch the `StorageBytes` metric by storage class over the following days. Read a file after it moves to IA and observe it return to Standard. *Cost/cleanup:* Pennies for small data; instance hours apply. Unmount, delete the file system and mount targets, and terminate the instance.

### Skill 3.4.2: Optimize block storage costs with volume types and IOPS provisioning

**Key concepts**

- **gp3** is the default general-purpose SSD. You pay for storage, plus any IOPS and throughput provisioned above the included baseline (3,000 IOPS and 125 MiB/s at any size). Performance is set independently of size. Since September 2025, gp3 volumes scale up to 64 TiB, 80,000 IOPS and 2,000 MiB/s.
- **gp2** ties IOPS to size (3 IOPS per GiB with burst credits for small volumes). To get more IOPS you must buy more storage. gp3's per-GB price is lower than gp2's, so migrating gp2 to gp3 typically saves money at the same or better performance. Migrate in place with Elastic Volumes (no detach, no downtime); a volume can be modified again only after a waiting period after each modification.
- **io2 Block Express**: the choice for very high IOPS, sub-millisecond latency, higher durability, and features such as Multi-Attach. You pay for storage and every provisioned IOPS, so do not use it where gp3 meets the requirement. io1 is the older generation; io2 offers better durability at similar pricing.
- **st1** (throughput-optimised HDD) for large sequential reads such as big data and log processing; **sc1** (cold HDD) for infrequently accessed sequential data. Neither can be a boot volume.
- **Instance store** for temporary high-IOPS scratch data at no extra charge (included in the instance price), but the data is lost on stop or termination.
- Rightsizing: Compute Optimizer recommends EBS volume types and sizes based on `VolumeReadOps`, `VolumeWriteOps` and throughput metrics. Look for over-provisioned io1/io2 IOPS, gp2 volumes, and unattached volumes.
- Remove waste:
  - Unattached volumes keep billing after an instance is terminated unless `DeleteOnTermination` was set.
  - Old snapshots: snapshots are incremental, but each one still holds changed blocks.
  - Snapshots kept only for compliance can move to EBS Snapshots Archive (lower storage price, 90-day minimum, restore takes hours). Recycle Bin protects against accidental deletion but retained snapshots still cost.
- Also check that instance EBS bandwidth limits match the volume; provisioning more volume throughput than the instance can drive is waste.

**Exam traps**

- Choosing io2 for a database that needs 12,000 IOPS on 500 GiB. gp3 delivers that by provisioning IOPS above baseline at a fraction of the cost.
- Growing a gp2 volume to 4 TiB only to get IOPS. gp3 decouples IOPS from size.
- Detaching and copying volumes to change type. Elastic Volumes changes type, size and performance online.
- Using st1 for a random-I/O database because "it is cheaper". HDD volumes perform poorly for small random I/O.
- Archiving snapshots that are needed for fast restore. Archived snapshots take hours to restore and have a minimum retention charge.

**Hands-on**

1. **gp2 to gp3 migration in place.** Launch a small instance with a 100 GiB gp2 data volume. Run `fio` for a random read test and record IOPS. Modify the volume to gp3 at 3,000 IOPS and 125 MiB/s with Elastic Volumes while the test runs, then repeat the test. Compare the per-month price of both configurations in the Pricing Calculator. *Cost/cleanup:* A few cents for an hour. Terminate the instance and delete the volume (check it is not left unattached).
2. **Find EBS waste.** Use the EC2 console filter "Volume state: available" to find unattached volumes and the Compute Optimizer EBS view for over-provisioned volumes. Create a snapshot, archive it to EBS Snapshots Archive, and read the restore time and minimum period on the snapshot page. *Cost/cleanup:* Archived snapshots are billed for at least 90 days, so archive only a tiny test snapshot or just read the documentation. Delete the volumes and snapshots you created.

### Skill 3.4.3: Optimize database costs with capacity modes and reserved capacity

**Key concepts**

- **DynamoDB capacity modes**:
  - **On-demand**: pay per read and write request unit. No capacity planning and no charge for idle throughput. On-demand is the default and recommended mode for new and variable workloads. Since November 2024 on-demand throughput costs half what it did, which narrowed the gap with provisioned. You can set a maximum throughput per table or index to cap cost and runaway traffic.
  - **Provisioned**: pay per hour for read and write capacity units whether used or not. Use with auto scaling for steady, predictable traffic with high utilisation. DynamoDB reserved capacity (1 or 3 years) discounts provisioned capacity further for stable baselines.
  - Switching: you can move a table from provisioned to on-demand up to four times in a rolling 24-hour period, and from on-demand to provisioned at any time.
  - Table classes: **DynamoDB Standard-IA** table class lowers storage price at higher request prices; use it for tables where storage dominates cost (for example old order history).
  - Other DynamoDB levers: TTL to delete expired items for free, project only needed attributes into GSIs, compress large attributes, export to S3 for analytics instead of scanning.
- **RDS and Aurora**:
  - **Reserved DB instances**: 1 or 3 years, for a specific engine family and Region. Size flexibility applies within an instance family for most engines (not for license-included engines such as some SQL Server and Oracle editions).
  - **Database Savings Plans** (December 2025): a 1-year hourly commitment covering Aurora, RDS, DynamoDB, ElastiCache, DocumentDB, Neptune, Keyspaces, Timestream and DMS across engines, families, sizes, deployment options (including serverless) and Regions. Discounts are larger for serverless than for provisioned instances. Choose it over RIs when you expect engine or Region changes, or move between provisioned and serverless. Choose RDS RIs when you want a 3-year term for a stable fleet and the deepest instance discount.
  - **Aurora Serverless v2**: capacity in ACUs that scales in fine increments; it can scale down to 0 ACUs (auto-pause) for idle development and test clusters on supported versions. Best for spiky or intermittent load. For steady high load, provisioned instances with RIs or Savings Plans are usually cheaper.
  - **Aurora I/O-Optimized**: no charge for read and write I/O; higher instance and storage prices. AWS guidance: consider it when I/O is more than about 25% of the Aurora bill. You can switch a cluster to I/O-Optimized once every 30 days and back to Standard at any time. It works with existing Aurora RIs. Cross-Region replication I/O for Global Database is still charged.
  - **Aurora Standard**: pay per I/O; cheaper for low or moderate I/O.
  - RDS storage: gp3 for most; io2 only for high IOPS needs. Stop idle non-production RDS instances (they restart after seven days) or schedule them.
  - Graviton DB instance classes give better price performance for supported engines.
  - Licensing: bring-your-own-license versus license-included; moving from commercial engines to Aurora PostgreSQL or MySQL removes license cost (a modernisation answer).
- **ElastiCache**: Serverless (pay per data stored and ElastiCache processing units) for variable load; node-based with reserved nodes or Database Savings Plans for steady load. Valkey engine pricing is lower than Redis OSS on ElastiCache.
- **Redshift**: Reserved nodes for provisioned clusters; Redshift Serverless (RPUs per second, with a base capacity) for intermittent analytics; pause and resume for provisioned clusters.

**Exam traps**

- Choosing DynamoDB provisioned mode for a new application with unknown traffic "because it is cheaper". On-demand avoids throttling and paying for idle capacity, and is the AWS default.
- Choosing on-demand for a table with flat, high, predictable throughput all day. Provisioned with auto scaling and reserved capacity is cheaper.
- Assuming Database Savings Plans support 3-year terms. They are 1-year only.
- Moving to Aurora I/O-Optimized for a low-I/O database. You pay higher instance and storage rates without enough I/O savings to offset them.
- Using Aurora Serverless v2 for a database that runs at high constant load. You pay more per unit of capacity than for a committed provisioned instance.

**Hands-on**

1. **Compare DynamoDB modes on your own traffic.** Create a table in provisioned mode with auto scaling (min 1, max 50, target 70%). Run a script that writes in bursts (quiet, then a spike) and watch consumed versus provisioned capacity and throttle events in CloudWatch. Switch the table to on-demand, set a maximum write throughput, and repeat the burst. Use the Pricing Calculator to compare monthly cost for your observed pattern. *Cost/cleanup:* Cents. Delete the table.
2. **Aurora Standard versus I/O-Optimized decision.** Create a small Aurora PostgreSQL Serverless v2 cluster (minimum ACU set low, or 0 with auto-pause where supported), run `pgbench` for 20 minutes, and read `VolumeReadIOPs` and `VolumeWriteIOPs` in CloudWatch. Estimate I/O cost as a share of the total with the Aurora pricing page and decide whether I/O-Optimized would pay off. Read the Aurora billing FAQ for the switching rules. *Cost/cleanup:* ACU-hours and I/O for under an hour cost a small amount. Delete the cluster without a final snapshot, and delete any manual snapshots.

### Skill 3.4.4: Implement backup and DR with cost-optimized retention policies

**Key concepts**

- **AWS Backup** centralises backup plans across services (EBS, EC2, RDS, Aurora, DynamoDB, EFS, FSx, S3, and others) with schedules, retention, lifecycle and cross-Region and cross-account copies. Use backup policies in AWS Organizations to apply plans across accounts by tag.
- **Retention design**:
  - Keep frequent short-retention backups (daily for 35 days) and fewer long-retention backups (monthly for 1 year, yearly for 7 years), instead of keeping every daily backup for years.
  - Use backup plan lifecycle to move recovery points to cold storage for supported resource types (for example EBS, EFS and DynamoDB with advanced features enabled). Cold storage has a 90-day minimum, and the plan must keep the backup at least 90 days after transition.
  - Copy to another Region only the backups needed for the DR objective (often the latest few dailies), with shorter retention in the copy.
- **Amazon Data Lifecycle Manager (DLM)**: policy-based EBS snapshot and EBS-backed AMI creation, retention, cross-Region copy and archive to EBS Snapshots Archive, by tag. No charge for DLM itself. Use when you only need EBS snapshots and AMIs; use AWS Backup for multi-service, organisation-wide governance and Vault Lock.
- **Snapshots are incremental**. Deleting an old snapshot only frees blocks not referenced by later snapshots. The cost is driven by change rate, not by volume size.
- Database native options: RDS automated backups (up to 35 days, free up to the size of the database storage per Region), manual snapshots for longer retention, Aurora backtrack for fast rewind (Aurora MySQL), DynamoDB PITR (35-day window by default, configurable shorter) and on-demand backups.
- **DR cost ladder** (cheapest to most expensive): backup and restore, pilot light, warm standby, active-active. Match to RTO and RPO. Backup and restore with cross-Region copies is the cheapest DR when hours of RTO are acceptable.
- Protect backups without paying twice: AWS Backup Vault Lock (compliance mode) and logically air-gapped vaults protect against deletion; S3 Object Lock for S3-based archives.

**Exam traps**

- Keeping daily backups for 7 years to satisfy a "7-year retention" rule that applies to month-end records. A monthly rule with 7-year retention meets it at a fraction of the storage.
- Transitioning backups to cold storage after 30 days and deleting them at 60 days. Cold storage needs at least 90 days; AWS Backup enforces the minimum.
- Using AWS Backup cold storage for a resource type that does not support it. Check support (for example RDS snapshots do not tier to cold storage in AWS Backup).
- Building warm standby in a second Region for a workload with a 24-hour RTO. Backup and restore is enough.
- Writing custom Lambda snapshot scripts for EBS when DLM or AWS Backup does it with policies.

**Hands-on**

1. **Tiered AWS Backup plan.** Create a backup plan with two rules: daily backups retained 35 days, and monthly backups moved to cold storage after 90 days and retained 1 year, with a cross-Region copy of the monthly rule retained 1 year. Assign resources by tag `backup=standard` and tag a small EBS volume and a DynamoDB table. Run an on-demand backup and inspect the recovery points and their lifecycle fields. *Cost/cleanup:* Backup storage for tiny resources costs cents. Delete recovery points (both Regions), the plan, the vault if empty, and the test resources.
2. **DLM for EBS snapshots and archive.** Create a DLM policy targeting volumes with tag `dlm=yes`: snapshots every 12 hours retained for 7, plus a weekly schedule with archive to EBS Snapshots Archive. Tag a small volume and watch snapshots appear. *Cost/cleanup:* Snapshot storage is cents; archived snapshots carry a 90-day minimum charge, so either skip the archive step or accept the small charge. Delete the policy, snapshots and volume.

### Skill 3.4.5: Design cost-effective observability (log filtering, metric aggregation, retention)

**Key concepts**

- **CloudWatch Logs cost drivers**: ingestion per GB (usually the biggest), storage per GB-month, Logs Insights queries per GB scanned, and vended log delivery for some services. Reduce ingestion first:
  - Lower log levels in production.
  - Drop health-check and debug lines at source (application logging config, Fluent Bit filters on containers, Lambda advanced logging controls with log level and JSON format).
  - Send high-volume, rarely read logs straight to S3 where the service supports it.
- **Log classes**: Standard and **Infrequent Access** (being renamed Essentials). Infrequent Access has a lower ingestion price; storage and Logs Insights query prices are the same. It supports Logs Insights queries, KMS encryption, export to S3, cross-account features and data protection masking. It does not support metric filters, subscription filters, Live Tail, anomaly detection, embedded metric format extraction, Container Insights or Lambda Insights ingestion, or the `GetLogEvents` and `FilterLogEvents` APIs. The class is set when the log group is created and cannot be changed afterwards.
- Use Infrequent Access for logs kept for audit or occasional forensic queries (for example VPC Flow Logs copies, detailed application debug logs). Keep Standard for logs that drive alarms through metric filters or stream to other systems through subscription filters.
- **Retention**: log groups default to "never expire". Set retention on every log group (for example 30 days in CloudWatch), and archive to S3 with lifecycle rules into S3 Glacier storage classes for long-term compliance. Use an AWS Config rule or Organizations-wide automation to enforce retention.
- Centralised logging: CloudWatch Logs subscription filters to Amazon Data Firehose to S3 (Parquet, compressed) is cheaper to store and query with Athena for large volumes than keeping everything in CloudWatch.
- **Metric costs**: custom metrics are priced per metric per month, and each unique dimension combination is a separate metric. High-cardinality dimensions (user ID, request ID) explode cost. Aggregate before publishing, use embedded metric format with few dimensions, publish at standard resolution unless high resolution is needed, and use CloudWatch metric streams only when an external tool needs the data.
- Metric filters on logs create custom metrics; keep them targeted.
- CloudWatch Contributor Insights and Logs Insights are good for ad hoc high-cardinality analysis without creating permanent metrics.
- **Tracing**: X-Ray and OpenTelemetry sampling rules control trace volume; sample a small percentage of normal traffic and all errors.
- **Alarms**: each alarm has a monthly charge; composite alarms reduce noise. Anomaly detection alarms cost more than static alarms.
- VPC Flow Logs: publish directly to S3 in Parquet with hourly partitions for cheap storage and fast Athena queries, and select only the fields you need.

**Exam traps**

- Moving a log group to Infrequent Access when its metric filters drive production alarms. Metric filters are not supported, and the class cannot be changed in place; you must create a new log group.
- Leaving log retention at "never expire" and archiving to S3 as well. You pay twice.
- Adding a `customerId` dimension to a custom metric for a SaaS with 50,000 customers. That creates 50,000 metrics per metric name.
- Exporting CloudWatch Logs to S3 with `CreateExportTask` for continuous archiving. Export tasks are batch and limited in concurrency; use a subscription filter to Firehose for continuous delivery.

**Hands-on**

1. **Infrequent Access versus Standard log group.** Create two log groups, one Standard and one Infrequent Access, both with 7-day retention. Send the same JSON log lines to both with the CLI. Run the same Logs Insights query on each, then try to create a metric filter and a subscription filter on the Infrequent Access group and note the errors. *Cost/cleanup:* Fractions of a cent for small data. Delete both log groups.
2. **Find and fix observability waste.** In Cost Explorer, filter on CloudWatch and group by usage type to see ingestion (`DataProcessing-Bytes`), storage and metrics charges. Write a short script or use the console to list log groups with no retention set, and set retention to 30 days on test groups. In a test Lambda function, enable advanced logging controls with JSON format and application log level WARN, invoke it, and confirm DEBUG and INFO lines are dropped. *Cost/cleanup:* Free to inspect; Lambda free tier. Delete the function and its log group.

### Practice questions: Task 3.4

**Question 3.4.1** A research organisation stores 3 PB of genomic data in Amazon S3 Standard. Analysis teams read recent datasets heavily for a few weeks, but older datasets are re-read unpredictably when new studies start, sometimes after a year. Teams need millisecond access when they do re-read older data. Most objects are larger than 100 MB, but one prefix holds 400 million small metadata files of about 10 KB each that are read constantly. The organisation wants to reduce storage cost with the least operational overhead. Which solution meets these requirements?

- A. Create a lifecycle rule that transitions all objects to S3 Glacier Flexible Retrieval after 30 days and restores them on demand when a study starts.
- B. Create a lifecycle rule that transitions all objects, including the metadata prefix, to S3 Intelligent-Tiering after 0 days and enable the Deep Archive Access tier for all objects.
- C. Create a lifecycle rule filtered to objects larger than 128 KB that transitions them to S3 Intelligent-Tiering, without enabling the optional asynchronous archive tiers, and leave the metadata prefix in S3 Standard.
- D. Create a lifecycle rule that transitions all objects to S3 One Zone-IA after 30 days and uses S3 Storage Class Analysis to move frequently re-read objects back to S3 Standard.

<details>
<summary>Answer and explanation</summary>

**Correct answer: C**

**Why C is correct:** Access to older datasets is unpredictable, which is the case Intelligent-Tiering is built for: it moves objects to cheaper tiers after 30 and 90 days without access and back when they are read, with no retrieval fees. Leaving out the optional asynchronous archive tiers keeps every object available in milliseconds. Filtering by size avoids the per-object monitoring fee and transition requests on tiny objects, which Intelligent-Tiering would not tier anyway because objects under 128 KB stay at the Frequent rate. The hot metadata prefix stays in S3 Standard.

**Why the other options are incorrect:**

- **A:** Glacier Flexible Retrieval requires a restore that takes minutes to hours, which fails the millisecond access requirement, and retrieval fees apply to every re-read.
- **B:** Deep Archive Access is an asynchronous tier that needs a restore taking hours, which fails millisecond access. Transitioning 400 million small files adds transition request charges with no tiering benefit.
- **D:** One Zone-IA keeps data in one AZ, which is risky for the primary copy of research data, and charges retrieval fees. Storage Class Analysis only recommends transitions; it does not move objects back.

**Key takeaway:** Use Intelligent-Tiering for unpredictable access, keep its archive tiers off when millisecond access is required, and exclude objects smaller than 128 KB.

</details>

**Question 3.4.2** A company runs 600 Amazon EC2 instances, each with a 1 TiB gp2 data volume used by a self-managed database. Monitoring shows each volume needs at most 6,000 IOPS and 250 MiB/s. Twenty larger instances run a trading database on io1 volumes provisioned with 20,000 IOPS each, but CloudWatch shows peak usage of 9,000 IOPS with no latency issues. The company wants to reduce EBS cost without downtime or any loss of performance. Which combination of actions meets these requirements? (Select TWO.)

- A. Use Elastic Volumes to change the 600 gp2 volumes to gp3 with 6,000 IOPS and 250 MiB/s provisioned.
- B. Change the 600 gp2 volumes to st1 throughput-optimised HDD volumes to reduce the per-GB price.
- C. Take snapshots of the io1 volumes, create new io2 Block Express volumes from them with 20,000 IOPS, and swap them in during a maintenance window.
- D. Shrink each gp2 volume to 500 GiB so that its baseline IOPS fall to the measured need.
- E. Use Elastic Volumes to change the trading database io1 volumes to gp3 with about 12,000 IOPS provisioned, after confirming that the latency profile meets the database's needs.

<details>
<summary>Answer and explanation</summary>

**Correct answer: A, E**

**Why A and E are correct:** gp3 has a lower per-GB price than gp2 and lets you provision 6,000 IOPS and 250 MiB/s independently of size, so the 600 volumes keep their performance at a lower cost. Elastic Volumes makes the change online. For the trading volumes, peak use is 9,000 IOPS, which gp3 can serve with headroom (gp3 now supports up to 80,000 IOPS per volume), at far lower cost than paying for 20,000 provisioned io1 IOPS. Elastic Volumes again avoids downtime, and validating latency first addresses the performance requirement.

**Why the other options are incorrect:**

- **B:** st1 is designed for large sequential throughput and performs poorly on the small random I/O of a database, so performance would drop.
- **C:** io2 Block Express keeps paying for 20,000 provisioned IOPS that are not used, and swapping volumes from snapshots requires downtime, which the requirement forbids.
- **D:** EBS volumes cannot be shrunk in place, and gp2 at 500 GiB gives a baseline of 1,500 IOPS, far below the 6,000 IOPS needed.

**Key takeaway:** gp3 decouples IOPS and throughput from size; migrate gp2 and over-provisioned io1 to gp3 with Elastic Volumes when its limits meet the measured need.

</details>

**Question 3.4.3** A fintech company runs Aurora PostgreSQL provisioned clusters in two Regions, Amazon RDS for MySQL in one Region, Amazon ElastiCache, and Amazon DynamoDB on-demand tables. Over the next year it plans to move the RDS for MySQL workload to Aurora, try Aurora Serverless v2 for some services, and possibly move one workload to a third Region. Its database spend is steady and predictable at the account level. The main Aurora cluster spends about 45% of its bill on I/O charges. The CFO wants a commitment-based discount that keeps working through these changes, plus any configuration change that lowers the Aurora bill. Which solution meets these requirements?

- A. Buy 3-year Aurora Reserved Instances for the current instance classes in each Region, convert the RDS for MySQL instances to reserved instances, and keep the clusters on Aurora Standard.
- B. Buy a Database Savings Plan sized to the steady database spend, and switch the main Aurora cluster to Aurora I/O-Optimized.
- C. Buy a 1-year Compute Savings Plan sized to the database spend, and enable Aurora auto scaling for read replicas.
- D. Switch all DynamoDB tables to provisioned capacity with reserved capacity, and move the main Aurora cluster to Aurora Serverless v2 to remove I/O charges.

<details>
<summary>Answer and explanation</summary>

**Correct answer: B**

**Why B is correct:** A Database Savings Plan is a 1-year hourly commitment that applies across Aurora, RDS, ElastiCache and DynamoDB, across engines, instance families, deployment options including serverless, and Regions. That flexibility matches the planned move from RDS for MySQL to Aurora, the use of Serverless v2 and a possible new Region. The main cluster spends about 45% of its bill on I/O, well above the roughly 25% level at which AWS suggests I/O-Optimized saves money, because I/O-Optimized has no read or write I/O charge.

**Why the other options are incorrect:**

- **A:** Reserved Instances are tied to engine, instance family and Region, so the migration to Aurora, the use of serverless and the new Region would leave RIs unused. Staying on Aurora Standard keeps the large I/O charge.
- **C:** Compute Savings Plans cover EC2, Fargate and Lambda, not database services. Read replica auto scaling adds capacity and does not reduce I/O cost.
- **D:** The DynamoDB tables are on-demand by choice, and nothing says their traffic is steady per table. Aurora Serverless v2 still charges for I/O unless the cluster uses I/O-Optimized, so it does not remove I/O charges.

**Key takeaway:** Database Savings Plans give database-wide flexibility on a 1-year term, and Aurora I/O-Optimized pays off when I/O is a large share of the Aurora bill.

</details>

**Question 3.4.4** A regulated lender must keep backups of its Amazon EBS volumes, Amazon RDS databases and Amazon DynamoDB tables for 7 years. Auditors require month-end recovery points for the full 7 years and daily recovery points for 35 days. The DR plan has an RPO of 24 hours and an RTO of 12 hours and needs a copy in a second Region. Backups must be protected against deletion by administrators. The company uses 40 accounts in AWS Organizations and wants the lowest storage cost with centralised management. Which solution meets these requirements?

- A. Create Amazon Data Lifecycle Manager policies in each account for daily EBS snapshots retained 7 years, use RDS automated backups for 7 years, and enable DynamoDB point-in-time recovery for 7 years.
- B. Create an AWS Backup plan with daily backups retained 7 years and copied to the second Region, and apply AWS Backup Vault Lock in governance mode on the vaults.
- C. Write an AWS Lambda function in each account that takes monthly snapshots of all resources, copies them to the second Region, and stores the snapshot IDs in a DynamoDB table for audit.
- D. Use an AWS Organizations backup policy that creates a daily rule retained 35 days with a cross-Region copy retained 7 days, and a monthly rule retained 7 years with lifecycle to cold storage where supported, into vaults protected with AWS Backup Vault Lock in compliance mode.

<details>
<summary>Answer and explanation</summary>

**Correct answer: D**

**Why D is correct:** An Organizations backup policy applies the same AWS Backup plan to all 40 accounts centrally. Keeping daily backups for 35 days and only month-end backups for 7 years stores far less data than keeping every daily backup for 7 years. Copying the daily backups to the second Region with short retention meets the 24-hour RPO for DR at low cost. Cold storage lifecycle lowers the price of long-retention recovery points for resource types that support it. Vault Lock in compliance mode prevents anyone, including administrators and the root user, from deleting recovery points before retention ends.

**Why the other options are incorrect:**

- **A:** RDS automated backups retain at most 35 days and DynamoDB PITR covers at most 35 days, so neither meets 7 years. Per-account DLM policies are not centrally managed and keep every daily snapshot for 7 years.
- **B:** Retaining every daily backup for 7 years and copying all of them to a second Region multiplies storage cost. Governance mode can be overridden by principals with the right permissions, so it does not protect against administrator deletion.
- **C:** Custom functions in 40 accounts are high operational overhead, provide no daily recovery points, and offer no immutable protection against deletion.

**Key takeaway:** Tier retention (short daily, long monthly), copy only what DR needs, and use Backup Vault Lock compliance mode with Organizations backup policies for regulated backups.

</details>

**Question 3.4.5** A SaaS company runs 300 microservices on Amazon ECS. CloudWatch is now its third-largest cost. Analysis shows that most cost is log ingestion from DEBUG-level application logs that are kept for 12 months but queried only during rare incident investigations. A smaller set of log groups have metric filters that drive production alarms. The company also publishes a custom latency metric with a `customerId` dimension for 40,000 customers, and all log groups have retention set to never expire. Which combination of actions will reduce CloudWatch cost while keeping alarms and incident investigation working? (Select THREE.)

- A. Create new log groups in the Infrequent Access log class for the DEBUG application logs, keep the alarm-driving log groups in the Standard class, and point each service at the right group.
- B. Change the existing alarm-driving log groups to the Infrequent Access log class so their ingestion is cheaper.
- C. Set retention on the CloudWatch log groups to match how long logs are queried there, and archive older logs to Amazon S3 with lifecycle rules to S3 Glacier storage classes for the rest of the 12 months.
- D. Publish the latency metric with only service and Region dimensions, and use CloudWatch Logs Insights or Contributor Insights on structured logs for per-customer analysis.
- E. Enable CloudWatch metric streams to Amazon Data Firehose for all metrics so that they can be stored in Amazon S3 instead of CloudWatch.
- F. Increase the logging level of all services to TRACE during business hours and DEBUG at night to balance cost and detail.

<details>
<summary>Answer and explanation</summary>

**Correct answer: A, C, D**

**Why A, C and D are correct:** Infrequent Access has a lower ingestion price and still supports Logs Insights queries, so it fits rarely queried DEBUG logs. The class is set at log group creation, so new groups are needed, and the alarm-driving groups stay Standard because Infrequent Access does not support metric filters. Setting retention stops paying for never-expiring storage, and S3 with lifecycle to Glacier storage classes is a cheaper long-term archive. Removing the `customerId` dimension cuts tens of thousands of custom metrics down to a handful, while Logs Insights or Contributor Insights still answer per-customer questions on demand.

**Why the other options are incorrect:**

- **B:** The log class of an existing log group cannot be changed, and Infrequent Access does not support metric filters, so the production alarms would stop working.
- **E:** Metric streams add a per-update charge and Firehose costs. They do not reduce the number of custom metrics stored in CloudWatch, so the high-cardinality cost remains.
- **F:** TRACE logging during business hours increases ingestion, which is already the largest cost driver.

**Key takeaway:** Cut CloudWatch cost by lowering ingestion (log level, Infrequent Access class where features allow), setting retention, archiving to S3, and keeping metric dimensions low-cardinality.

</details>

---

[Back to contents](README.md) · [Next: Domain 4](domain-4-resilience-migration-continuity.md)
