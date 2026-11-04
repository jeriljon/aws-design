# AWS Certified Solutions Architect - Professional (SAP-C03) Study Notes

Study notes for the SAP-C03 exam, organised by the official exam guide: every domain, task and skill, with key concepts, exam traps, hands-on exercises and 5 exam-style questions per task (105 in total) with explanations of every option.

> **Status:** draft, written 5 November 2026 against the SAP-C03 exam guide. SAP-C03 replaces SAP-C02 on 17 November 2026. AWS features change often, so check the linked documentation before relying on a detail, and log corrections in the [revision history](#revision-history).

## Contents

- [Exam at a glance](#exam-at-a-glance)
- [What changed from SAP-C02](#what-changed-from-sap-c02)
- [How to use these notes](#how-to-use-these-notes)
- [How to answer professional-level questions](#how-to-answer-professional-level-questions)
- [Domain 1: Architecture Design and Implementation Built for the Cloud (25%)](#domain-1-architecture-design-and-implementation-built-for-the-cloud-25)
- [Domain 2: Security, Compliance, and Governance (25%)](#domain-2-security-compliance-and-governance-25)
- [Domain 3: Cost-Optimized Architecture Design (15%)](#domain-3-cost-optimized-architecture-design-15)
- [Domain 4: Resilience, Migration, and Business Continuity (15%)](#domain-4-resilience-migration-and-business-continuity-15)
- [Domain 5: Operational Excellence and Automation (20%)](#domain-5-operational-excellence-and-automation-20)
- [In-scope AWS services and documentation](#in-scope-aws-services-and-documentation)
- [Out-of-scope services](#out-of-scope-services)
- [Revision history](#revision-history)

## Exam at a glance

| Item | Detail |
| --- | --- |
| Exam code | SAP-C03 (from 17 November 2026; SAP-C02 is used until 16 November 2026) |
| Scored questions | 65, plus 10 unscored questions that are not identified |
| Question types | Multiple choice (1 correct of 4) and multiple response (2 or more correct of 5 or more; all must be selected for credit) |
| Scoring | Scaled 100 to 1,000; pass at 720; compensatory, so you pass on the overall score, not per domain |
| Guessing | No penalty; unanswered questions count as wrong |
| Target candidate | 2+ years designing and implementing AWS solutions, including serverless and AI-integrated architectures in multi-account environments |
| Out of scope for the candidate | Deep OS knowledge; ML model training, fine-tuning and data science workflows; security forensics; application code development and debugging |

| Domain | Weight | Approx. scored questions |
| --- | --- | --- |
| 1. Architecture Design and Implementation Built for the Cloud | 25% | ~16 |
| 2. Security, Compliance, and Governance | 25% | ~16 |
| 3. Cost-Optimized Architecture Design | 15% | ~10 |
| 4. Resilience, Migration, and Business Continuity | 15% | ~10 |
| 5. Operational Excellence and Automation | 20% | ~13 |

The question counts are this guide's arithmetic from the weightings, not official figures.

## What changed from SAP-C02

| SAP-C02 domain | % | SAP-C03 domain | % |
| --- | --- | --- | --- |
| Design Solutions for Organizational Complexity | 26 | Architecture Design and Implementation Built for the Cloud | 25 |
| Design for New Solutions | 29 | Security, Compliance, and Governance | 25 |
| Continuous Improvement for Existing Solutions | 25 | Cost-Optimized Architecture Design | 15 |
| Accelerate Workload Migration and Modernization | 20 | Resilience, Migration, and Business Continuity | 15 |
| | | Operational Excellence and Automation | 20 |

The biggest shifts:

- **Generative AI and agents are now core.** Amazon Bedrock, Bedrock AgentCore, Knowledge Bases (RAG), Guardrails, Evaluations, SageMaker AI inference, AI cost optimization, AI observability and AI audit logging all appear in the task statements.
- **Five domains instead of four,** organised by Well-Architected pillar rather than by project stage. Cost and operations each have their own domain.
- **New content:** durable workflows, service mesh (VPC Lattice, ECS Service Connect), ECR lifecycle and scanning, multi-tenancy, WebSocket APIs, lakehouse and Iceberg, IPv6 and dual stack, hybrid compute (EKS Anywhere, ECS Anywhere, Outposts, Local Zones), EKS Pod Identity, IAM Roles Anywhere, post-quantum cryptography, data residency, threat modelling, chaos engineering, SLOs, synthetic and real user monitoring.
- **Services added to scope** include Amazon Bedrock, Data Lifecycle Manager, MemoryDB, Security Lake, Verified Permissions, VPC Lattice, Application Auto Scaling, AWS CDK, Clean Rooms, Cloud WAN, CodeCommit, Cost Categories, Data Transfer Terminal, AWS Distro for OpenTelemetry, Fault Injection Service, Local Zones, Resilience Hub, AWS Transform, Verified Access and Reserved Instances.
- **Removed from the in-scope list:** all IoT services, WorkSpaces and AppStream 2.0, Elastic Transcoder, Kinesis Video Streams, individual AI services (now covered under Bedrock and SageMaker AI), and services such as Pinpoint, SES, Amplify, App Runner, AWS Batch, CodeArtifact, Data Exchange, License Manager, Proton, Service Catalog and Wavelength.

## How to use these notes

1. **Read the skill notes for one task,** then do at least one hands-on exercise per skill. The exercises are designed for a personal account and say what they cost and what to delete.
2. **Answer the five questions without opening the answers.** Write down your choice and one line of reasoning for each option.
3. **Open the explanations** and compare your reasoning for every option, not just the correct one. The exam is passed by eliminating options that are feasible but fail one stated requirement.
4. **Track weak skills** in a list and revisit their notes and documentation (see [In-scope AWS services and documentation](#in-scope-aws-services-and-documentation)).
5. Budget: set an AWS Budgets alert on your practice account before starting. Some exercises use resources that bill by the hour (NAT gateways, Transit Gateway, EKS, MSK, OpenSearch, Redshift); each exercise says so.

A suggested 8-week plan, at roughly 8 to 10 hours a week:

| Week | Focus |
| --- | --- |
| 1 | Domain 2: Tasks 2.1 to 2.3 (identity, encryption, multi-account governance) |
| 2 | Domain 2: Tasks 2.4 to 2.5; Domain 1: Task 1.4 (networking) |
| 3 | Domain 1: Tasks 1.1 and 1.3 (cloud-native and hybrid) |
| 4 | Domain 1: Tasks 1.2 and 1.5 (AI/ML, data and databases) |
| 5 | Domain 5 (IaC, CI/CD, observability, automation) |
| 6 | Domain 4 (migration, DR, business continuity) |
| 7 | Domain 3 (cost); redo all questions you got wrong |
| 8 | Full timed practice exams; review weak skills; read the Well-Architected Framework pillars again |

## How to answer professional-level questions

At this level, most options would work. The question is which one meets **every** stated requirement best.

1. **Underline the qualifiers:** "least operational overhead", "most cost-effective", "minimal changes to the application", "within minutes", "without exposing data to the internet", "across all accounts". Each one eliminates options.
2. **Rank the qualifiers.** Hard constraints (compliance, RPO/RTO, data residency) beat soft ones (cost, effort).
3. **Prefer managed and native** when the question stresses operational overhead; prefer **preventive** controls (SCPs, RCPs, IAM conditions) over **detective** ones (Config, Security Hub) when it says "prevent" or "ensure".
4. **Watch for scope:** one account vs the whole organisation, one Region vs multi-Region, new resources vs existing ones.
5. **For "Select TWO/THREE",** each correct option must stand on its own as part of the solution. Do not pick two options that do the same job.
6. **Eliminate by mechanism:** name the specific limit, missing feature, pricing dimension or failure mode that rules each option out. The explanations in these notes model this.

---

# Domain 1: Architecture Design and Implementation Built for the Cloud (25%)

Tasks 1.1 to 1.5: cloud-native and event-driven design, AI/ML integration, hybrid integration, networking, and data, storage and databases.

## Task 1.1: Design architectures built for the cloud

This task tests whether you can pick the right managed building block for each part of a distributed system and justify it against a stated constraint. The decision the exam keeps asking for is the same each time: which service gives the required guarantee (ordering, durability, isolation, latency) with the least custom code and operational overhead.

### Skill 1.1.1: Event-driven messaging patterns (idempotency, ordering, delivery guarantees)

**Key concepts**

- Almost every AWS messaging service delivers at least once. Design every consumer to be idempotent, whatever service sits in front of it. A common way to do this is a conditional write (`attribute_not_exists`) to a DynamoDB table keyed on the event or business ID, with TTL to expire old keys. Powertools for AWS Lambda has an idempotency utility built on this pattern.
- **SQS standard:** close to unlimited throughput, at-least-once delivery, best-effort ordering. Use it when order does not matter and consumers are idempotent.
- **SQS FIFO:** ordering is strict only *within a message group ID*. Deduplication (content-based or by `MessageDeduplicationId`) works inside a 5-minute window. Throughput is 300 messages per second per API action without batching and 3,000 with batching. High-throughput mode raises this a lot, provided you spread load across many message groups. One hot group ID serialises processing, because only one in-flight batch per group can be consumed at a time.
- **SNS:** push-based fan-out. SNS FIFO topics keep order and deduplicate, and they fan out to SQS FIFO queues. Subscription filter policies let each subscriber receive only part of the stream, with no consumer code.
- **EventBridge:** content-based routing on the full event JSON. It offers schema discovery, archive and replay, scheduled rules, and API destinations for third-party HTTP targets. Delivery is at least once with **no ordering guarantee**. On failure it retries (by default for up to 24 hours) and then sends to a DLQ if you configure one per target. Choose EventBridge when routing logic and many targets matter more than order or raw throughput.
- **EventBridge Pipes:** point-to-point from a source (SQS, Kinesis, DynamoDB Streams, MSK, and others) to a target, with optional filtering, enrichment and transformation. It replaces "glue" Lambda functions.
- **Kinesis Data Streams:** order is kept per shard. The partition key is hashed to a shard, so all records for one key stay in order. Each shard takes 1 MB/s or 1,000 records/s on write. Readers share 2 MB/s per shard, or get 2 MB/s each with enhanced fan-out. Records can be replayed for the whole retention period, and many independent consumers can read the same stream. A low-cardinality partition key creates hot shards.
- Decision rule:
  - Strict per-entity order plus deduplication at moderate rate: SQS FIFO with entity ID as the group ID (SNS FIFO in front if you need fan-out).
  - Ordered high-volume stream with replay and several consumers: Kinesis with the entity ID as the partition key.
  - Routing by content to many targets, including other accounts: EventBridge.
- With Lambda event source mappings:
  - For SQS, set the queue's visibility timeout to at least six times the function timeout, and use partial batch responses (`ReportBatchItemFailures`) so one bad message does not force the whole batch to be retried.
  - For Kinesis, use bisect-on-error, maximum retry attempts, maximum record age and an on-failure destination, so a poison record does not block a shard until it expires.

**Exam traps**

- "Use SQS FIFO for exactly-once delivery end to end." FIFO deduplicates on the *producer* side inside 5 minutes. A consumer that crashes after its side effect but before it deletes the message will still see the message again. You still need idempotent consumers.
- Choosing EventBridge when the scenario says events must be processed in order. EventBridge has no ordering guarantee.
- Using a single constant `MessageGroupId` or partition key "to guarantee order". This works, but it caps throughput at one consumer. The correct answer uses a per-entity key (account ID, order ID).
- Increasing Kinesis shards does not fix a hot partition key. Only a better key distribution fixes it.

**Hands-on**

1. **FIFO ordering and dedupe.**
   - Create an SQS FIFO queue with content-based deduplication.
   - With the AWS CLI, send 10 messages across two message group IDs. Send one message body twice within a minute.
   - Receive with `max-number-of-messages 10` and see that order is kept within each group and the duplicate is dropped.
   - Then process the queue with a Lambda function that sleeps, and watch messages in one group wait behind each other.

   *Cost/cleanup:* SQS and Lambda free tier cover this. Delete the queue and the function.
2. **Idempotent consumer.**
   - Write a Lambda function, triggered by an SQS standard queue, that does a DynamoDB `PutItem` with `ConditionExpression attribute_not_exists(pk)` using the message's business ID.
   - Send the same business ID three times and confirm only one item is written. Check that the other attempts log a `ConditionalCheckFailedException` that you catch and treat as success.

   *Cost/cleanup:* on-demand DynamoDB and free tier cost close to nothing. Delete the table, queue and function.

### Skill 1.1.2: Stateless applications with externalised state

**Key concepts**

- Keep compute disposable. Store session, workflow and cache state outside the instance, task or function, so that scaling in, Spot interruption, or a deployment loses nothing.
- **DynamoDB:** the default store for session and entity state that must be durable. Expect single-digit-millisecond reads at any scale. TTL deletes expired sessions for free, and conditional writes and transactions give you optimistic locking. Items are limited to 400 KB, so large blobs go to S3 with a pointer in DynamoDB.
- **ElastiCache (Valkey, Redis OSS, Memcached):** sub-millisecond access for hot session data, rate-limit counters, leaderboards and pub/sub. ElastiCache Serverless removes node sizing. Use Valkey or Redis OSS with replication and Multi-AZ when losing the cache would hurt users. Use Memcached only for a simple, multi-threaded, non-persistent cache.
- **S3:** large objects, documents and checkpoints. It has strong read-after-write consistency for all operations. Conditional writes (`If-None-Match`) let you prevent overwriting an object that already exists, which is useful for idempotent writers.
- ALB sticky sessions are a smell in a "stateless" scenario. They pin users to a target, cause uneven load, and lose sessions when the target is replaced. They are the right answer only when you cannot change a legacy application.
- For Lambda, anything held in `/tmp` or global variables is only an optimisation inside one execution environment. Never rely on it for correctness.
- DynamoDB Accelerator (DAX) caches DynamoDB reads with an API-compatible client. It is eventually consistent for cached reads. Choose ElastiCache for general-purpose caching and DAX for read-heavy DynamoDB without changing your data access code much.

**Exam traps**

- "Store sessions on an EFS mount shared by all instances." This works, but it adds latency and gives no TTL or atomic counters. DynamoDB or ElastiCache is the expected answer.
- Choosing ElastiCache as the *only* store for state that must survive a failure, such as a shopping cart or workflow position. A cache is not a system of record unless the scenario accepts losing it.
- Enabling sticky sessions to fix lost sessions after Auto Scaling scale-in. The real fix is to externalise the session.

**Hands-on**

1. **Session store with TTL.**
   - Create a DynamoDB table `sessions` with TTL on attribute `expiresAt`.
   - Behind a Lambda function URL, write a function that issues a session ID cookie and stores the session with `expiresAt` set to 10 minutes ahead.
   - Call it repeatedly, then publish a new version, then invoke again. The session survives because state is not in the function.

   *Cost/cleanup:* free tier. Delete the table and function.
2. **ElastiCache Serverless counter.**
   - Create an ElastiCache Serverless Valkey cache in a VPC.
   - From a Lambda function in the same VPC, run `INCR` on a per-user key with `EXPIRE` to build a fixed-window rate limiter. Call it in a loop to see the limit apply.

   *Cost/cleanup:* ElastiCache Serverless bills per GB-hour stored and per ECPU, and each cache has a minimum billed storage amount. Even when idle, it keeps charging every hour until you delete it. Delete the cache the same day. A VPC Lambda function needs no NAT gateway for this, so do not create one.

### Skill 1.1.3: Serverless data processing and analysis pipelines

**Key concepts**

- **Standard pattern:** producers send to Kinesis Data Streams or straight to Amazon Data Firehose. Firehose then buffers, optionally transforms with Lambda, converts JSON to Parquet or ORC using a Glue Data Catalog schema, and writes partitioned objects to S3. The Glue Data Catalog holds the schema, and Athena queries it. There are no servers to manage at any stage.
- **Kinesis Data Streams vs Firehose:**
  - Kinesis Data Streams gives custom consumers, ordering, replay and sub-second latency.
  - Firehose is fully managed delivery to S3, Redshift, OpenSearch, Splunk, HTTP endpoints and others. It buffers by size or time (seconds to minutes) and has no replay.
  - If the scenario says "near real time into S3 with no code", choose Firehose.
- **AWS Glue:** serverless Spark or Python shell ETL jobs, crawlers to infer schemas, job bookmarks for incremental processing, and Glue Data Quality rules. Glue streaming jobs can read from Kinesis or Kafka.
- **Athena:** pay per data scanned. Cost and speed depend on columnar formats (Parquet or ORC), compression, partitioning and partition projection. Partition projection avoids crawling and managing very large numbers of partitions. Use workgroups for per-team query limits and result locations.
- **Lambda in pipelines:** good for light per-record transforms, enrichment and fan-out. It is limited to 15 minutes and a fixed memory ceiling. For joins, large aggregations or anything over large data volumes, use Glue or EMR Serverless.
- Step Functions Distributed Map processes millions of S3 objects in parallel by invoking Lambda per item or per batch. It replaces hand-built fan-out code.

**Exam traps**

- Choosing Lambda for a multi-hour batch transform of terabytes. This fails on the 15-minute limit. Glue or EMR Serverless is correct.
- Querying raw JSON in S3 with Athena when the scenario stresses cost. The answer converts to Parquet and partitions (Firehose format conversion or a Glue job).
- Choosing Kinesis Data Streams plus custom consumers when the requirement is just "deliver to S3 and Redshift with least operational overhead". Firehose does this with no consumer code.
- Expecting Firehose to deliver within milliseconds. It buffers.

**Hands-on**

1. **Firehose to Parquet to Athena.**
   - Create a Glue database and table describing a small JSON clickstream schema.
   - Create a Firehose stream with record format conversion to Parquet using that table, delivering to S3 with a prefix like `dt=!{timestamp:yyyy-MM-dd}/`.
   - Send 1,000 test records with a short script. After the buffer interval, query them in Athena and compare "data scanned" with a query over a raw JSON copy.

   *Cost/cleanup:* a few cents. Delete the Firehose stream, S3 objects and Glue table.
2. **Kinesis with Lambda and bisect-on-error.**
   - Create an on-demand Kinesis data stream and a Lambda consumer that throws an error on records containing `"bad": true`.
   - Configure bisect-on-error, maximum retry attempts of 2, and an SQS on-failure destination.
   - Put 50 records, one of them bad. Confirm the rest are processed and only the bad record's metadata lands in SQS.

   *Cost/cleanup:* on-demand Kinesis bills per stream-hour plus data, so delete the stream within the hour. Delete the function and queue.

### Skill 1.1.4: Multi-step workflows with durable execution (Step Functions, durable Lambda functions)

**Key concepts**

- **Step Functions Standard workflows:**
  - Up to one year per execution, with exactly-once execution of each workflow step.
  - Full execution history is visible in the console.
  - Priced per state transition.
  - Use them for long-running, auditable business processes and human approvals. The `.waitForTaskToken` callback pattern pauses until an external system returns the token.
- **Step Functions Express workflows:**
  - Up to 5 minutes, priced by request and duration.
  - Asynchronous Express runs are at least once. Synchronous Express runs are at most once.
  - Use them for high-volume, short, idempotent event processing.
- **Step Functions integrations:** more than 200 services through AWS SDK integrations, so many steps need no Lambda at all. `.sync` waits for jobs such as Glue, ECS or Batch tasks. Distributed Map gives massive parallelism. Retry and Catch are declared per state.
- **Lambda durable functions** (launched at re:Invent 2025):
  - You write the workflow as ordinary code (Python, Node.js or Java, and container images too) using the Durable Execution SDK.
  - `context.step()` checkpoints each step's result. `context.wait()` pauses without consuming compute. `context.waitForCallback()` pauses until an external system sends success or failure to the Lambda API, for example for human approval. `parallel()` and `map()` fan out work, and `invoke()` chains to another function in the same account.
  - On resume, Lambda re-runs the handler from the top and *replays* completed steps from the checkpoint log instead of re-executing them. Code outside steps must therefore be deterministic: no direct random values, clock reads or API calls.
- **Durable function limits that decide answers:**
  - Each invocation is still capped at 15 minutes. A whole durable execution can last up to one year when invoked asynchronously.
  - Synchronous invocations are capped at 15 minutes in total.
  - When an event source mapping (SQS, Kinesis, DynamoDB Streams, MSK and others) invokes a durable function, the call is synchronous. The *whole execution* must then finish within the function timeout limit (15 minutes on default capacity). For longer workflows, put a small standard Lambda function between the mapping and the durable function and have it invoke the durable function asynchronously.
  - Use the execution name parameter for idempotent starts.
  - You must invoke with a qualified ARN (version or alias). An execution is pinned to the version that started it, so new deployments do not break in-flight executions. Executions started on `$LATEST` can resume on changed code, which is unsafe in production.
  - Durable functions support DLQs but not Lambda destinations.
  - While an on-demand function is suspended you pay no duration charges. You do pay for replay invocations, durable operations and stored checkpoint data.
- **Choosing between them:**
  - Choose Step Functions when you want a visual, declarative workflow, direct service integrations without code, Distributed Map at very large scale, or orchestration across many services and teams.
  - Choose durable functions when the team wants the workflow as application code inside Lambda, tested with normal unit tests, with no separate state-machine language.

**Exam traps**

- Express workflows for anything that waits for a person or runs longer than 5 minutes.
- A plain Lambda function with `sleep` or polling to wait for hours. This fails on the 15-minute limit and you pay for idle time.
- An SQS event source mapping wired straight to a durable function whose workflow waits days for approval. The mapping invokes synchronously, so the execution fails at the timeout limit. Use the intermediary-function pattern.
- Non-deterministic code (timestamps, UUIDs, API calls) outside `step()` in a durable function. It breaks replay.

**Hands-on**

1. **Step Functions callback approval.**
   - Build a Standard workflow: a Lambda step, then an SQS `sendMessage.waitForTaskToken` step that puts the task token in a queue, then a Choice state on the returned approval.
   - Start an execution, read the token from the queue, and call `aws stepfunctions send-task-success`.
   - Watch the execution resume in the console's graph view.

   *Cost/cleanup:* within free tier (4,000 state transitions per month). Delete the state machine, queue and function.
2. **Durable function with a callback.**
   - Create a Node.js or Python Lambda function with durable execution enabled. Use one `step` to "reserve stock", then `waitForCallback` for approval, then a final `step`.
   - Publish version 1 and invoke `function:1` asynchronously with an execution name.
   - Send the callback success with the CLI. Inspect the durable execution state and note that the first step did not re-run on resume.
   - Invoke again with the same execution name and observe that no duplicate execution starts.

   *Cost/cleanup:* a few cents at most. Delete the function and its versions.

### Skill 1.1.5: Cross-account and cross-Region event routing with event buses

**Key concepts**

- **Bus-to-bus pattern:** a rule on the source bus targets the ARN of another event bus. The target bus can be in another account, another Region, or both. The same-account, different-Region case is supported.
  - The receiving bus needs a **resource-based policy** that allows the sender: specific account IDs, or the whole organization through an `aws:PrincipalOrgID` condition.
  - The sending rule needs an **IAM role** that allows `events:PutEvents` on the target bus. A role has been mandatory for new cross-account bus targets since March 2023.
- **Single-hop rule:** EventBridge does not forward an event a second time. If account B receives an event from account A and has a rule sending it on to account C, the event does not reach C. The same applies when chaining buses inside one account. Central aggregation designs must therefore send straight from each source to the final bus, not through regional "collector" buses.
- **Direct cross-account targets:** within the *same Region*, a rule can deliver straight to SQS, SNS, Lambda, Kinesis Data Streams or API Gateway in another account, with no bus in the target account.
  - This needs an execution role in the bus account plus a resource policy on the target.
  - Use it when the consumer team does not want to run a bus. Use bus-to-bus when the receiving team wants its own routing rules, or when the target is in another Region.
- **Billing:** the sending account pays for cross-account events as custom events. The receiving account is not charged for receiving them.
- In receiving rules, filter on `account` (and `region`) to tell sources apart. Never put `"*"` in the account field, because that matches nothing. Leave the field out to match all permitted senders.
- **Common designs:**
  - Hub-and-spoke: workload accounts send to a central bus in a security or operations account. The central rules fan out to SQS, Lambda, Step Functions or log archives.
  - The reverse: a central bus publishes to consumer accounts.
  - Multi-Region DR: replicate selected events to a bus in the standby Region. Archive and replay on the target bus lets you re-drive events after an incident.
- SCPs apply to the IAM roles that rules use, so organization guardrails can block unapproved cross-account event flows.

**Exam traps**

- A three-tier "workload bus to regional collector bus to central bus" chain. The third bus never receives the events because of the single-hop rule.
- Expecting a cross-account bus target to work with only the receiver's bus policy. New targets also need the sender-side IAM role.
- A direct cross-account SQS target in *another Region*. Direct service targets are same-Region only, so use a bus in that Region.
- Using `"account": ["*"]` in a pattern. It matches nothing.

**Hands-on**

1. **Cross-Region bus-to-bus in one account.**
   - Create a custom bus `central` in us-east-1 with a rule that sends everything to a CloudWatch Logs log group.
   - In eu-west-1, create a rule on the default bus that matches `source: ["demo.app"]` and targets the us-east-1 `central` bus ARN, letting the console create the IAM role.
   - Run `aws events put-events` in eu-west-1 and see the event appear in us-east-1 with `region: eu-west-1`.

   *Cost/cleanup:* fractions of a cent. Delete the rules, the bus, the log group and the role.
2. **Prove the single-hop rule.**
   - Add a third bus `sink` in us-east-1 and a rule on `central` that forwards to `sink`, with a log group target on `sink`.
   - Send the same eu-west-1 event. It reaches `central` and not `sink`.
   - With a second free AWS account, repeat with a cross-account target using an `aws:PrincipalOrgID` or account-ID bus policy. If you have only one account, read the "Sending and receiving events between AWS accounts" page instead.

   *Cost/cleanup:* fractions of a cent. Delete the buses, rules and roles.

### Skill 1.1.6: Container orchestration with managed services (EKS, ECS, Fargate)

**Key concepts**

- **Amazon ECS:** AWS-native and simpler. There is no control plane charge, and it integrates closely with IAM task roles, ALB, Service Connect and CloudWatch. Choose ECS when the team has no Kubernetes requirement and wants the least operational overhead.
- **Amazon EKS:** upstream-conformant Kubernetes. Choose it when the scenario requires Kubernetes APIs, Helm charts, operators or CRDs, portability or hybrid consistency (EKS Anywhere), or an existing Kubernetes skill base. The control plane is billed per cluster-hour.
- **EKS compute options:**
  - Managed node groups: AWS-managed EC2 Auto Scaling groups.
  - Karpenter: just-in-time node provisioning and consolidation.
  - EKS Auto Mode: AWS manages node provisioning, scaling and core add-ons for you.
  - Fargate profiles.
- **AWS Fargate:** serverless compute for ECS and EKS. There are no nodes to patch and each task or pod runs in its own isolated environment. Limits that decide answers:
  - No GPUs.
  - No privileged containers.
  - On EKS, no DaemonSets. Run agents as sidecars instead.
  - On EKS, pods must be exposed through IP-mode load balancer targets.
  - Choose EC2 capacity when you need GPUs, specific instance types, very high density, or the cheapest steady-state price with Savings Plans or Reserved Instances.
- **Capacity providers (ECS):** a capacity provider strategy can mix Fargate and Fargate Spot, or EC2 Auto Scaling groups with managed scaling. Use `base` and `weight` to keep a minimum on On-Demand and burst onto Spot. Fargate Spot tasks get a two-minute warning before interruption.
- **Identity:** on ECS, use task IAM roles per service, and keep the task execution role only for pulling images and fetching secrets. On EKS, use EKS Pod Identity or IRSA for per-pod IAM. Never use the node instance role for application permissions.
- **Secrets:** inject from Secrets Manager or SSM Parameter Store in the ECS task definition (`secrets`). On EKS, use the Secrets Store CSI Driver.

**Exam traps**

- EKS "because microservices". If nothing in the scenario requires Kubernetes and it says "least operational overhead", ECS on Fargate wins.
- Fargate for a GPU inference workload or a DaemonSet-based security agent. Both are unsupported.
- Giving application permissions to the EC2 node role or the ECS task *execution* role. The task role (ECS) or Pod Identity (EKS) is the least-privilege answer.
- All-Spot capacity for a service with a minimum availability requirement. Use a capacity provider strategy with an On-Demand base.

**Hands-on**

1. **ECS on Fargate with a Spot mix.**
   - Create an ECS cluster with the FARGATE and FARGATE_SPOT capacity providers.
   - Run a service of 4 tasks of a public nginx image with strategy FARGATE base 1 weight 1, FARGATE_SPOT weight 3. Use public subnets with public IPs so you need no NAT gateway.
   - In the console, check how the tasks are spread across the providers, and look at the task role vs the execution role.

   *Cost/cleanup:* small Fargate tasks cost cents per hour. Set the desired count to 0 and delete the service and cluster.
2. **EKS Pod Identity (read-only alternative).**
   - If you accept the EKS control plane charge (billed per hour, plus nodes), create a cluster with eksctl.
   - Install the Pod Identity Agent add-on and associate an IAM role with a service account.
   - From a pod, run `aws sts get-caller-identity` to see the assumed role.
   - If you do not want the charge, follow the EKS Workshop Pod Identity module in a Workshop Studio event instead.

   *Cost/cleanup:* the EKS control plane and nodes bill hourly, so delete the cluster (`eksctl delete cluster`) the same day.

### Skill 1.1.7: Service mesh and service-to-service networking (VPC Lattice, ECS Service Connect)

**Key concepts**

- **ECS Service Connect:** built into ECS. It uses an AWS Cloud Map namespace and injects a managed proxy into each task.
  - You get short service names for discovery, client-side load balancing, retries, timeouts and outlier detection.
  - Per-service traffic metrics appear in CloudWatch with no app changes.
  - It is ECS only. It works across accounts by sharing the Cloud Map namespace through AWS RAM (same Region only).
  - It does **not** provide network connectivity. Services in different VPCs still need VPC peering or other routing.
  - Choose it for ECS-only estates that want mesh-like features with almost no configuration.
- **Amazon VPC Lattice:** an application networking layer with no sidecars, which works across VPCs and accounts.
  - Building blocks: *services* (listeners, rules, target groups), a *service network* that you associate with consumer VPCs, and RAM sharing across accounts.
  - Consumers need no peering or Transit Gateway route to provider VPCs.
  - Targets can be EC2 instances, IP addresses, Lambda functions, ALBs, ECS services (native association since November 2024, no intermediate ALB needed) and EKS through the AWS Gateway API Controller.
  - It supports HTTP/HTTPS, gRPC and TCP.
  - **Auth policies** are IAM resource policies on the service network (coarse) and the service (fine). Callers sign requests with SigV4, which gives IAM-based zero-trust authorisation between services.
  - *Resource configurations* expose TCP resources such as an RDS database to other VPCs or accounts through the service network.
  - Pricing: per service-hour, per GB processed and per request (or per TCP connection for TLS listeners). VPC association and service network endpoints add no charge.
- **Choosing between them:**
  - Mixed compute (ECS + EKS + Lambda + EC2), many accounts, overlapping or unconnected VPCs, or IAM authorisation between services: VPC Lattice.
  - ECS only, one VPC or already-connected VPCs, and observability plus resilience with minimal setup: Service Connect.
  - AWS PrivateLink endpoint services publish one service to many consumers through an NLB. Lattice is better when there are many services in both directions.
- AWS App Mesh reached end of support on 30 September 2026. Treat it as a migration source, moving to Service Connect or Lattice, and never as the answer for a new design.

**Exam traps**

- Service Connect across accounts in unconnected VPCs. A shared namespace gives discovery, but without peering or other routing the traffic does not flow.
- Transit Gateway plus internal ALBs for services in VPCs with overlapping CIDRs. Routing cannot resolve the overlap, and Lattice avoids it.
- Self-managed Istio on EKS when the requirement is "IAM-based service authorisation with least operational overhead" across ECS and Lambda as well.
- Assuming Service Connect enforces authorisation between services. Its strength is discovery, resilience and metrics, not IAM auth policies.

**Hands-on**

1. **VPC Lattice with an auth policy.**
   - Create two VPCs with the *same* CIDR (for example 10.0.0.0/16).
   - In VPC A, create a Lattice service whose target group is a Lambda function. Create a service network associated with VPC B and the service.
   - Set the auth type to AWS_IAM with an auth policy that allows only one IAM role.
   - From a CloudShell VPC environment in VPC B (or a small EC2 instance), call the service's DNS name, first unsigned (you get 403), then signed with SigV4 using `awscurl` or a short SDK script.

   *Cost/cleanup:* Lattice bills per service-hour plus requests and GB, which is cents for an hour. Delete the service network, service, target group, function and VPCs. Do not leave a t-series instance running.
2. **ECS Service Connect metrics.**
   - In one VPC, create an ECS cluster with a Cloud Map namespace. Deploy two Fargate services, `front` and `back`, with Service Connect enabled, with `front` calling `http://back:8080`.
   - Generate traffic and look at the per-service request counts and response codes in the ECS console and in CloudWatch.

   *Cost/cleanup:* Fargate cents per hour. Delete the services, the cluster and the namespace.

### Skill 1.1.8: Container image security scanning and lifecycle policies (Amazon ECR)

**Key concepts**

- **Basic scanning:** AWS-native technology that looks for operating-system package CVEs. It runs on push (for repositories that match filters) or manually. Each image can be scanned once per 24 hours. ECR does not charge for it.
- **Enhanced scanning:** an integration with Amazon Inspector, configured at registry level with repository filters.
  - It scans both OS *and* programming-language packages.
  - It offers scan on push or **continuous** scanning. With continuous scanning, images are re-evaluated when new CVEs are published.
  - If filters overlap, the continuous scanning filter wins over the scan-on-push filter.
  - Repositories that match no filter are not scanned at all. Manual scans are not supported.
  - When you first turn it on, only images pushed in the last 14 days are eligible. Older images show `SCAN_ELIGIBILITY_EXPIRED` until you push them again.
  - In Inspector you set the push-date duration and the re-scan duration, which control how long images stay monitored.
  - Inspector charges per image scanned and re-scanned.
- Findings go to EventBridge (Inspector emits events when a scan completes and when findings change) and to Security Hub. A rule on critical findings can open a ticket, notify a team, or trigger a pipeline rebuild. Enhanced scanning also shows which images are in use on ECS and EKS clusters, which helps you prioritise fixes.
- **Lifecycle policies** expire images by tag status (tagged, untagged, any), tag prefix or pattern, image count (`imageCountMoreThan`) or age (`sinceImagePushed`). Rules are evaluated by priority.
- Since November 2025, ECR also has an **archive storage class**. Lifecycle rules can archive rarely used images, including by last pull time, instead of deleting them. Archived images are restored through the console, CLI or API before you can pull them (AWS quotes restore within about 20 minutes).
- **Supply-chain controls:**
  - Tag immutability stops `latest` or release tags from being overwritten.
  - Cross-Region and cross-account replication rules.
  - Pull-through cache rules to mirror public registries under your scanning and policy controls.
  - Repository policies that allow pulls only from your organization.
  - Image signing with AWS Signer (Notation) verified at deploy time.
- **In pipelines:** scan in CodeBuild, gate promotion on Inspector findings, and promote the *same* immutable digest across accounts rather than rebuilding.

**Exam traps**

- Basic scanning when the requirement includes vulnerabilities in application dependencies (npm, pip, Maven). Basic covers OS packages only, so you need enhanced scanning.
- Scan on push alone when the requirement is "detect newly disclosed CVEs in images already deployed". You need continuous scanning.
- Expecting enhanced scanning to cover old images right away. Images older than 14 days at enablement must be pushed again.
- A lifecycle rule `tagStatus: any, imageCountMoreThan: 10` on a repository where production pins old tags. It deletes images that are in use. Scope rules with tag prefixes, or archive instead of expiring.

**Hands-on**

1. **Basic vs enhanced findings.**
   - Push an old base image (for example an outdated `python` tag) to a private ECR repository with basic scan on push and record the findings.
   - Turn on enhanced scanning with a continuous filter for that repository, push again, and compare: enhanced shows language-package findings that basic did not.
   - Create an EventBridge rule on `aws.inspector2` findings with severity `CRITICAL` that sends to an SNS email subscription.

   *Cost/cleanup:* Inspector has a 15-day free trial for new accounts. After that, ECR scans are charged per image. Turn off enhanced scanning (or deactivate Inspector) and delete the repository.
2. **Lifecycle policy preview.**
   - Push 6 images tagged `dev-1` to `dev-6` and 3 images tagged `prod-1` to `prod-3`.
   - Write a lifecycle policy: rule 1 expires untagged images older than 1 day, rule 2 keeps only the last 3 images with prefix `dev-`.
   - Use **Lifecycle policy preview** to confirm the `prod-` images are untouched before you apply it.

   *Cost/cleanup:* ECR storage costs cents per GB-month. Delete the repository.

### Skill 1.1.9: Auto scaling for serverless and containerised workloads

**Key concepts**

- **Lambda concurrency:**
  - Concurrency is roughly requests per second times average duration. It is limited by the Regional account concurrency quota, which you can raise.
  - **Reserved concurrency** guarantees capacity to one function *and* caps it. Use it to protect a downstream database, or to throttle a function to zero as a kill switch.
  - **Provisioned concurrency** keeps initialised execution environments warm to remove cold starts. It is billed while configured, and you can scale it on a schedule or by target tracking through Application Auto Scaling.
  - SnapStart (Java, Python, .NET) cuts cold-start time without paying for always-on environments.
- **For SQS sources,** the event source mapping scales pollers automatically. Set *maximum concurrency* on the mapping to limit how many concurrent invocations one queue can drive without reserving concurrency.
- **AWS Application Auto Scaling** is the common engine behind scaling for ECS services, DynamoDB provisioned capacity, Lambda provisioned concurrency, Aurora replicas, SageMaker AI endpoints, Spot Fleet, ElastiCache, MSK storage and others. It supports target tracking, step scaling and scheduled scaling.
- **ECS service auto scaling:**
  - Target tracking on CPU, memory, or ALB `RequestCountPerTarget`, or on a custom metric. For queue workers, use *backlog per task*: queue depth divided by running tasks.
  - With EC2 capacity, the capacity provider's **managed scaling** adds or removes instances to fit the tasks.
- **EKS scaling has two layers:**
  - **Horizontal Pod Autoscaler (HPA)** scales pods on CPU, memory or custom metrics. KEDA adds event-driven scaling, for example on SQS queue depth, including scale to zero.
  - **Cluster Autoscaler** adjusts node group sizes when pods are pending. **Karpenter** provisions right-sized nodes directly, with no node groups, and consolidates underused nodes. Choose it for faster, cheaper scaling on varied instance types and Spot.
  - Vertical Pod Autoscaler right-sizes requests. Do not run it in automatic mode on the same metric HPA uses.
- **Choosing a policy type:** target tracking for steady proportional load, step scaling for large sudden spikes on alarms, and scheduled scaling for known business-hour or batch peaks. Combine scheduled and target tracking for predictable surges plus noise.

**Exam traps**

- Increasing reserved concurrency to fix cold starts. Reserved concurrency only guarantees and caps capacity; provisioned concurrency or SnapStart removes cold starts.
- Scaling an SQS worker service on CPU. Idle-waiting workers show low CPU while the backlog grows, so scale on backlog per task.
- HPA alone when pods are pending for lack of nodes. HPA creates pods, and you also need Cluster Autoscaler or Karpenter.
- Leaving a Lambda function that writes to a small RDS instance without a concurrency cap. A burst exhausts database connections. Use reserved or maximum concurrency, or RDS Proxy.

**Hands-on**

1. **Reserved vs provisioned concurrency.**
   - Create a Lambda function that sleeps 2 seconds and set reserved concurrency to 2.
   - Fire 20 parallel invocations with a short script and count the `TooManyRequestsException` throttles.
   - Then remove the reserved concurrency, publish a version, and configure provisioned concurrency 1 on an alias. Compare the `Init Duration` in the logs for the alias versus `$LATEST`.

   *Cost/cleanup:* provisioned concurrency is billed per hour while configured, which is small for one unit but not free. Remove it and delete the function.
2. **ECS scaling on queue backlog.**
   - Create an SQS queue and an ECS Fargate service of a worker container that polls slowly.
   - Publish a custom CloudWatch metric `BacklogPerTask` from a scheduled Lambda function (ApproximateNumberOfMessagesVisible divided by running tasks) and attach a target-tracking policy with target 10.
   - Load 500 messages and watch the task count rise and fall.

   *Cost/cleanup:* Fargate tasks and a scheduled function cost cents. Delete the scaling policy, service, queue and function.

### Skill 1.1.10: Ingress patterns (ALB, NLB, AWS Load Balancer Controller, API Gateway)

**Key concepts**

- **ALB:**
  - Layer 7 routing by host, path, header, query string and method, with weighted target groups for blue/green and canary.
  - Native authentication with Cognito or any OIDC provider.
  - WAF integration, gRPC, WebSockets, and Lambda targets.
  - Mutual TLS with trust stores, in verify or passthrough mode.
  - Choose it for HTTP microservices behind one entry point.
- **NLB:**
  - Layer 4 (TCP, UDP, TLS) with very high throughput, ultra-low latency and static IPs (or Elastic IPs) per AZ.
  - It preserves client IP and supports PrivateLink endpoint services.
  - Choose it for non-HTTP protocols, allow-listing by fixed IP, extreme connection volumes, or as the front of a PrivateLink service. NLBs now support security groups.
- **AWS Load Balancer Controller (EKS):** creates an ALB from a Kubernetes `Ingress` (or Gateway API) and an NLB from a `Service` of type LoadBalancer.
  - Use IP target mode to route straight to pod IPs; it is required for Fargate pods.
  - `IngressGroup` shares one ALB across many Ingress resources to save cost.
  - `TargetGroupBinding` attaches pods to an ALB managed outside Kubernetes.
- **API Gateway:**
  - **REST APIs** offer usage plans and API keys, request validation, WAF, caching, private APIs through interface endpoints, resource policies, canary deployments, and mapping templates.
  - **HTTP APIs** are cheaper and lower latency, with native JWT authorisers, Lambda and HTTP proxy integrations, and private integrations through VPC links to ALB, NLB or Cloud Map. They offer fewer features: no usage plans and no request validation.
  - Choose API Gateway for request-level throttling, per-client keys and quotas, authorisers, or fronting Lambda.
- **Common combinations:**
  - CloudFront → ALB, with WAF, for web apps.
  - API Gateway → VPC link → NLB or ALB for private microservices behind a managed API.
  - NLB → ALB, where the ALB is a target of the NLB, when you need static IPs plus layer-7 routing.
  - Global Accelerator → ALB for static anycast IPs and multi-Region failover.
- **Optimising for usage patterns:**
  - Spiky, low-volume APIs: HTTP API + Lambda, with no idle cost.
  - Steady high-RPS HTTP: ALB to containers, which is cheaper per request at high volume than API Gateway.
  - Long-lived TCP: NLB.
  - API Gateway caps the integration timeout (29 seconds by default for REST APIs; higher limits can be requested for some API types). Long-running requests belong on ALB or an async pattern.

**Exam traps**

- An ALB when partners must allow-list fixed IPs. ALB IPs change. Use an NLB (with an ALB as its target if you need layer 7) or Global Accelerator.
- An HTTP API when the scenario needs API keys with per-customer usage plans or request validation. Those are REST API features.
- Instance target mode for EKS on Fargate. Fargate pods need IP targets.
- A separate ALB per Kubernetes Ingress when cost matters. IngressGroup shares one ALB.

**Hands-on**

1. **ALB weighted canary with Lambda targets.**
   - Create two Lambda functions returning "v1" and "v2" and register each as a target group of an ALB (Lambda targets need no instances).
   - Create a listener rule forwarding 90/10 across the two groups. `curl` 100 times and count the responses.
   - Add an `X-Canary: true` header condition that routes 100% to v2.

   *Cost/cleanup:* the ALB bills hourly plus LCU, which is cents for an hour. Delete the ALB, target groups and functions.
2. **HTTP API vs REST API features.**
   - Put the same Lambda function behind an HTTP API with a JWT authoriser (Cognito user pool) and behind a REST API with a usage plan and API key.
   - Exceed the usage plan's throttle with a loop and observe 429 responses on the REST API.
   - Note in the console which features each type lacks.

   *Cost/cleanup:* free tier covers it. Delete both APIs, the user pool and the function.

### Skill 1.1.11: Edge architectures for latency (CloudFront Functions, Lambda@Edge)

**Key concepts**

- **CloudFront Functions:**
  - JavaScript only. Runs only on viewer request and viewer response.
  - Sub-millisecond execution, scales to millions of requests per second, very low price per invocation.
  - No network calls, no file system, no access to the request body.
  - Can read CloudFront KeyValueStore (JavaScript runtime 2.0) for edge lookups such as redirects, A/B buckets or feature flags.
  - Use it for header manipulation, URL rewrites and redirects, cache-key normalisation, simple token validation (for example verifying a signed cookie or JWT signature with a key from KeyValueStore), and adding security headers.
- **Lambda@Edge:**
  - Node.js or Python. Runs on viewer request and response *and* on origin request and response.
  - Has network access and can read the body. It allows far more memory and package size than CloudFront Functions, with the largest limits on origin triggers.
  - Higher latency and cost than CloudFront Functions.
  - You author it in us-east-1 and CloudFront replicates it to edge locations.
  - Use it for logic that calls other services, dynamic origin selection, body inspection or rewriting, or heavier authentication.
- **Rule of thumb:** if it is a lightweight viewer-side transform, choose CloudFront Functions. If it needs the origin events, the network or the body, choose Lambda@Edge.
- **Reduce latency without code first:**
  - Use cache policies to keep cache keys minimal; every extra header, cookie or query string lowers the hit ratio.
  - Use origin request policies to forward data to the origin without adding it to the cache key.
  - Origin Shield adds a central caching layer that collapses requests to the origin.
  - Origin groups give failover on 5xx responses or timeouts.
  - Origin access control (OAC) protects S3 origins.
- For dynamic APIs, CloudFront still helps: TLS terminates near the user, and the AWS backbone carries traffic to the origin. For non-HTTP traffic or static IPs, use Global Accelerator instead.

**Exam traps**

- Lambda@Edge for a simple header rewrite or redirect at very high request rates. CloudFront Functions is cheaper and faster.
- CloudFront Functions to call an authorisation service or DynamoDB. They have no network access. Use Lambda@Edge, or move the lookup data into KeyValueStore.
- Deploying Lambda@Edge from a Region other than us-east-1.
- Forwarding all headers to the origin "for flexibility". It destroys the cache hit ratio and raises latency and origin load.

**Hands-on**

1. **CloudFront Function with KeyValueStore redirects.**
   - Create a CloudFront distribution in front of an S3 bucket with OAC. Create a KeyValueStore with `/old` → `/new`.
   - Attach a viewer-request CloudFront Function (runtime 2.0) that looks up the URI and returns a 301 when it is found.
   - Test with `curl -I`, then update the key value and see the change without redeploying the function.

   *Cost/cleanup:* the CloudFront free tier covers this. Disable and then delete the distribution, function, KeyValueStore and bucket.
2. **Lambda@Edge origin selection.**
   - In us-east-1, create a Lambda@Edge origin-request function that routes requests with header `CloudFront-Viewer-Country: DE` to a second S3 origin in eu-central-1.
   - Add the header to the origin request policy (not the cache key, unless the response differs by country, in which case it must be in the cache key).
   - Use the CloudFront test feature or a VPN to compare responses.

   *Cost/cleanup:* pennies. Removing the association and deleting replicated functions can take a while after you detach them. Delete the distribution, function and buckets.

### Skill 1.1.12: Real-time processing with WebSocket and publish-subscribe

**Key concepts**

- **API Gateway WebSocket APIs:**
  - Routes are chosen by a route selection expression (for example `$request.body.action`) plus the `$connect`, `$disconnect` and `$default` routes.
  - Store connection IDs (with user and topic) in DynamoDB on `$connect`, and delete them on `$disconnect`.
  - The backend pushes to clients by calling the `@connections` management API (`PostToConnection`), signed with IAM.
  - To fan out to N subscribers, the backend loops over the stored connection IDs. API Gateway has no built-in broadcast or topics.
- **WebSocket quotas that change designs (the first three cannot be raised):**
  - Maximum connection duration is 2 hours, and the idle timeout is 10 minutes. Clients need reconnect logic and heartbeats.
  - Messages are up to 128 KB, sent in frames of up to 32 KB.
  - Integration timeout is 29 seconds.
  - New connections per second per account per Region is a raisable quota.
- **Authorisation:** apply it only on `$connect`, with a Lambda authoriser or IAM. Query string tokens are common because browsers cannot set custom headers on the WebSocket handshake.
- **Feeding updates in:** DynamoDB Streams, Kinesis or EventBridge trigger a Lambda function that looks up subscribed connections and pushes to them. For very large fan-out, shard the connection table and use SNS or Step Functions Distributed Map to parallelise the pushes.
- **AWS AppSync:** GraphQL subscriptions give managed pub/sub over WebSockets, with filtering and fan-out handled by the service. Choose it when the scenario wants subscriptions without managing connection IDs.
- **AWS IoT Core:** MQTT pub/sub with topic wildcards, built for very large numbers of device connections.
- **Event-driven pub/sub between services:** SNS (push fan-out) and EventBridge (routing). WebSockets are for clients such as browsers and mobile apps.

**Exam traps**

- Expecting a WebSocket connection to stay open all day. The 2-hour maximum and 10-minute idle limits require reconnect and keep-alive.
- Choosing WebSocket APIs and assuming API Gateway broadcasts to all clients. You must store connection IDs and call `PostToConnection` per client, or choose AppSync subscriptions.
- Polling a REST API every second for "real-time" updates when the requirement is low latency and low cost at scale.
- Storing connection state in Lambda memory. Use DynamoDB.

**Hands-on**

1. **Chat over a WebSocket API.**
   - Create a WebSocket API with `$connect` and `$disconnect` Lambda functions that write and delete connection IDs in DynamoDB, plus a `sendmessage` route whose function broadcasts with `PostToConnection`.
   - Connect two terminals using `wscat` and send messages.
   - Leave one idle for more than 10 minutes and observe the disconnect.

   *Cost/cleanup:* free tier covers it. Delete the API, the functions and the table.
2. **Stream-driven push.**
   - Enable DynamoDB Streams on an `orders` table and attach a Lambda function that pushes each `MODIFY` event to the connection IDs subscribed to that order.
   - Update an item with the CLI and see the update arrive in `wscat` within about a second.
   - If you prefer managed subscriptions, build the AWS AppSync quick-start GraphQL API with a subscription and compare how much code each needs.

   *Cost/cleanup:* free tier covers it. Delete the table, stream trigger, API and functions.

### Skill 1.1.13: Multi-tenant architectures and tenant isolation

**Key concepts**

- **Isolation models:**
  - **Silo:** resources per tenant, such as an account, VPC, database or stack. Strongest isolation, simplest noisy-neighbour story and per-tenant cost, but the highest cost and operational effort.
  - **Pool:** shared resources partitioned by tenant ID. Cheapest and easiest to operate, but isolation is enforced by policy and code.
  - **Bridge or tiered:** pool for basic tiers and silo for premium or regulated tenants. Often the expected SaaS answer.
  - The account-per-tenant silo is the strongest boundary and fits regulated FSI tenants. Automate it with Control Tower Account Factory and StackSets.
- **Tenant context with Amazon Cognito:**
  - **User pool per tenant:** maximum isolation, per-tenant MFA, password policy and federation. High operational effort, and users who belong to several tenants must sign up for each.
  - **App client per tenant:** per-tenant app settings and identity providers within one pool.
  - **Groups** or **custom scopes per tenant.**
  - **Custom attribute** (for example `custom:tenantId`) in one shared pool: the cheapest, but authorisation becomes your app's job. Make the attribute read-only to app clients so users cannot change their own tenant.
  - To put tenant ID and tier into the **access token** (not just the ID token), use the pre token generation Lambda trigger with event version 2, or version 3 to include machine-to-machine client credentials. Access token customisation requires the Essentials or Plus feature plan; the Lite plan can customise ID tokens only.
  - Downstream services read the tenant claim from the verified JWT and never from a request parameter.
- **Enforcing isolation in pooled data:**
  - **DynamoDB:** tenant ID as the partition key, plus IAM policies with `dynamodb:LeadingKeys` conditions. Each request assumes a role with a session tag `TenantId` (tag-based ABAC), so even buggy code cannot read another tenant's items.
  - **S3:** per-tenant prefixes with `${aws:PrincipalTag/TenantId}` in policies, or S3 Access Points per tenant.
  - **PostgreSQL (RDS or Aurora):** row-level security policies keyed on a session variable such as `app.tenant_id` that the app sets per connection. Shared tables, enforced in the database.
  - **Amazon Redshift:** row-level security policies too.
  - **Amazon Verified Permissions:** Cedar policies with a Cognito user pool as the identity source give externalised, tenant-aware authorisation.
- **Tenant-specific encryption:**
  - Silo uses a KMS key per tenant, which allows per-tenant revocation ("crypto-shredding") and per-tenant key policies and CloudTrail audit.
  - Pool can still use a per-tenant key for envelope encryption, or encryption context `tenantId` with a shared key. A condition on `kms:EncryptionContext:tenantId` stops one tenant's data key from decrypting another's.
  - Mind KMS request quotas and per-key monthly cost when you have thousands of tenants. Data key caching (AWS Encryption SDK) reduces calls.
- **Noisy neighbours:** API Gateway usage plans per tenant tier, SQS queues per tier, Lambda reserved concurrency per tier, and per-tenant metrics through tenant ID dimensions or embedded metric format logs for cost and usage attribution.

**Exam traps**

- Tenant ID passed as a header or query parameter and trusted by the backend. It must come from the verified token and be enforced in IAM or the database.
- A user pool per tenant for thousands of small tenants with the same configuration. The cost is high operational effort and service quotas. A shared pool with a tenant attribute or groups plus token claims is the expected answer.
- One shared KMS key when the requirement is "revoke one tenant's access to its data independently". You need a per-tenant key (or at least a key per silo tier).
- Application-only filtering (`WHERE tenant_id = ?`) when the requirement is defence in depth. The answer adds RLS or IAM `LeadingKeys`.

**Hands-on**

1. **ABAC with DynamoDB LeadingKeys.**
   - Create a table with partition key `tenantId` and sort key `orderId`, and add items for tenants `T1` and `T2`.
   - Create an IAM role whose policy allows `dynamodb:Query` and `dynamodb:GetItem` with the condition `ForAllValues:StringEquals` `dynamodb:LeadingKeys` = `${aws:PrincipalTag/TenantId}`. The trust policy must allow `sts:TagSession`.
   - Assume the role with session tag `TenantId=T1`, query T1's partition (allowed), then query T2's (AccessDenied).

   *Cost/cleanup:* free. Delete the table and role.
2. **Cognito tenant claim in the access token.**
   - Create a user pool on the Essentials plan with a custom attribute `tenantId` that is not writable by the app client.
   - Add a pre token generation trigger (event version 2) that copies `tenantId` into the access token.
   - Sign in a test user with the hosted login (or `admin-initiate-auth`), decode both tokens at the CLI, and confirm the claim is in the access token.
   - Then protect an HTTP API with a JWT authoriser and read the claim in the Lambda function.

   *Cost/cleanup:* Essentials includes a free tier of monthly active users. Delete the user pool, API and functions.

### Practice questions: Task 1.1

**Question 1.1.1** A retail bank processes card-account events (holds, captures, refunds) from a core banking adapter at up to 2,000 events per second during peaks. All events for one card account must be applied in the order they were produced. Events for different accounts can be processed in parallel. The adapter sometimes resends the same event after a network timeout, and a duplicate must not post twice. Processing must not let one problem account block others. The team wants a serverless design with the least operational overhead. Which solution meets these requirements?

- A. Publish events to an Amazon EventBridge custom bus. Create a rule that targets an AWS Lambda function with a dead-letter queue. In the function, sort events by producer timestamp before posting.
- B. Send events to an Amazon SQS FIFO queue with high-throughput mode, using the card account ID as the message group ID and the event ID as the deduplication ID. Process with Lambda and make the function's write conditional on the event ID.
- C. Send events to an Amazon SQS FIFO queue using one fixed message group ID and content-based deduplication. Process with a Lambda function and set reserved concurrency to protect the downstream database.
- D. Send events to an Amazon SQS standard queue. Process with Lambda using partial batch responses, and store the last processed sequence number per account in Amazon ElastiCache to discard late events.

<details>
<summary>Answer and explanation</summary>

**Correct answer: B**

**Why B is correct:**

- Ordering in SQS FIFO is scoped to the message group. Using the card account ID as the group keeps per-account order, and different accounts are processed in parallel, so one account's failures only hold up its own group.
- High-throughput mode supports the peak rate because load is spread over many groups.
- The deduplication ID drops adapter resends inside the 5-minute window.
- The conditional write on the event ID covers duplicates outside that window and consumer retries after partial failure, so the design stays idempotent.

**Why the other options are incorrect:**

- **A:** EventBridge gives no ordering guarantee. Sorting inside one invocation cannot order events that arrive in different invocations, and nothing deduplicates resends.
- **C:** A single message group ID serialises all processing to one in-flight batch. This fails the throughput requirement, and a problem event blocks every account.
- **D:** A standard queue can deliver out of order. Discarding "late" events drops valid transactions instead of applying them in order, and ElastiCache as the source of truth for sequence state risks loss on failover.

**Key takeaway:** Per-entity ordering at scale means SQS FIFO with the entity as the message group ID (or Kinesis with the entity as the partition key), plus an idempotent consumer.

</details>

**Question 1.1.2** A company uses AWS Organizations with 60 workload accounts in us-east-1, eu-west-1 and ap-southeast-1. The security team wants every Amazon GuardDuty finding and every AWS Config compliance change from all accounts and Regions delivered to an Amazon EventBridge custom bus named `sec-central` in a security tooling account in us-east-1. There, rules will invoke a Step Functions workflow. New accounts are added every week and must be covered automatically. The solution must use only native EventBridge routing. Which solution meets these requirements?

- A. In each workload account, create a rule in eu-west-1 and ap-southeast-1 that forwards matching events to the default bus in us-east-1 of the same account. In us-east-1, create a rule on the default bus that forwards both local and forwarded events to `sec-central`.
- B. In the security tooling account, create a rule on `sec-central` with `"account": ["*"]` to match every account's events. Attach a bus policy that allows `events:PutEvents` from the organization ID.
- C. In each workload account and Region, create a rule with an Amazon SQS queue in the security tooling account in us-east-1 as a direct cross-account target. Have a Lambda function in the tooling account read the queue and start the Step Functions workflow.
- D. Attach a resource policy to `sec-central` that allows `events:PutEvents` with an `aws:PrincipalOrgID` condition. Use CloudFormation StackSets with automatic deployment to create, in every account and Region, a rule that targets the `sec-central` bus ARN with an IAM role.

<details>
<summary>Answer and explanation</summary>

**Correct answer: D**

**Why D is correct:**

- A rule can target an event bus in another account and another Region directly.
- The `aws:PrincipalOrgID` condition on the bus policy admits every current and future member account without listing them.
- Since March 2023, a new cross-account bus target needs an IAM role on the sending rule, and the StackSet template creates that role in each account.
- StackSets with automatic deployment to the organization or target OUs creates the rule and role in each new account and Region as it joins, which meets the "covered automatically" requirement.
- Each event makes one hop from source to `sec-central`, so the single-hop rule is respected.

**Why the other options are incorrect:**

- **A:** This chains two hops: Regional default bus, then the us-east-1 default bus, then `sec-central`. EventBridge does not forward an event a second time, so events from eu-west-1 and ap-southeast-1 never reach `sec-central`.
- **B:** `"*"` in the account field matches nothing. A bus policy alone also moves no events, because each source account still needs a rule that sends to the bus.
- **C:** Direct cross-account service targets work only within the same Region, so eu-west-1 and ap-southeast-1 rules cannot target the us-east-1 queue. The design also adds a Lambda poller that native routing does not need.

**Key takeaway:** For central aggregation, send from every source straight to the final bus (one hop), allow the organization with `aws:PrincipalOrgID`, and roll out rules with StackSets automatic deployment.

</details>

**Question 1.1.3** An insurance company receives claim submissions on an Amazon SQS standard queue. Each claim needs fraud scoring through an external API, a document check, and then a manual adjuster approval that can take up to five days, before payment is issued. The development team writes Python, wants the whole workflow as application code with normal unit tests, and does not want to learn a separate workflow definition language. A claim must never start two parallel workflows if SQS delivers the message twice. Which solution meets these requirements?

- A. Configure the queue as an event source for a standard Lambda function that invokes a Lambda durable function asynchronously, passing a deterministic execution name derived from the claim ID. In the durable function, implement each stage as a step and use `waitForCallback` for the adjuster approval.
- B. Configure the queue as an event source mapping directly on a Lambda durable function version. Implement each stage as a step and use `waitForCallback` for the adjuster approval, with the durable execution timeout set to six days.
- C. Configure the queue as an event source for an AWS Step Functions Express workflow. Use a `.waitForTaskToken` task for the adjuster approval and Lambda tasks for the other stages.
- D. Configure the queue as an event source for a Lambda function that runs the scoring and document check, stores progress in Amazon DynamoDB, and polls DynamoDB every minute until the adjuster sets an approved flag.

<details>
<summary>Answer and explanation</summary>

**Correct answer: A**

**Why A is correct:**

- Durable functions let the team write the workflow as ordinary Python, using steps that checkpoint results and `waitForCallback` to suspend for days without paying for compute duration.
- An asynchronously invoked durable execution can run for up to a year, which covers the five-day approval.
- The small intermediary function is needed because event source mappings invoke durable functions synchronously.
- Using an execution name derived from the claim ID makes the start idempotent, so a duplicate SQS delivery does not start a second workflow.

**Why the other options are incorrect:**

- **B:** When an event source mapping invokes a durable function, the call is synchronous, and the entire execution must finish within the function timeout limit (15 minutes on default capacity). AWS refuses to create the mapping with a six-day execution timeout, so this design cannot be deployed.
- **C:** Express workflows run for at most 5 minutes and do not support the `.waitForTaskToken` pattern. It also uses the Amazon States Language, which the team wants to avoid. A Standard workflow would solve the duration problem but not the team-preference constraint.
- **D:** A Lambda invocation cannot exceed 15 minutes, so polling for five days needs repeated scheduled invocations and hand-built state handling. That is the custom orchestration durable functions remove, and it pays for idle polling.

**Key takeaway:** Lambda durable functions run for up to a year only when invoked asynchronously. Put an intermediary function with an idempotent execution name between an event source mapping and a long-running durable workflow.

</details>

**Question 1.1.4** A payments company runs microservices in four AWS accounts. Some services run on Amazon ECS with AWS Fargate, some on Amazon EKS, and some as AWS Lambda functions. Several VPCs use the same overlapping 10.0.0.0/16 CIDR range and cannot be re-addressed this year. The security team requires that every service-to-service call be authorised by the caller's IAM identity, with coarse rules defined centrally and fine-grained rules owned by each service team. The platform team wants no sidecar proxies and the least operational overhead. Which solution meets these requirements?

- A. Connect all VPCs to an AWS Transit Gateway shared through AWS RAM. Put an internal Application Load Balancer in front of each service with OIDC authentication, and register services in a shared AWS Cloud Map namespace for discovery.
- B. Enable Amazon ECS Service Connect with an AWS Cloud Map namespace shared through AWS RAM to all four accounts. Use the Service Connect proxy for retries and metrics, and configure VPC peering between the account VPCs.
- C. Create an Amazon VPC Lattice service network in a networking account and share it through AWS RAM. Register ECS services, EKS services through the AWS Gateway API Controller, and Lambda functions as Lattice services. Use IAM auth policies on the service network and each service.
- D. Deploy Istio on each EKS cluster and the AWS Distro for OpenTelemetry collector on ECS. Expose each service through an AWS PrivateLink endpoint service backed by a Network Load Balancer, and use mTLS certificates from AWS Private CA for service authorisation.

<details>
<summary>Answer and explanation</summary>

**Correct answer: C**

**Why C is correct:**

- VPC Lattice connects services across VPCs and accounts without routing between the VPCs, so overlapping CIDRs do not matter.
- It needs no sidecars, and it has native integrations for ECS services, EKS through the Gateway API Controller, and Lambda targets.
- Auth policies are IAM resource policies at two levels. The central team sets coarse rules on the service network and service teams set fine rules on their own services, with callers signing requests with SigV4.
- Sharing through AWS RAM gives multi-account access with a managed service.

**Why the other options are incorrect:**

- **A:** Transit Gateway cannot route between VPCs with overlapping CIDRs. OIDC authentication on an ALB authenticates users, not the caller's IAM identity.
- **B:** Service Connect works only for ECS, so it leaves the EKS and Lambda services out. It provides no IAM authorisation, and VPC peering is not possible between overlapping CIDRs.
- **D:** Istio runs sidecar proxies and takes heavy operational effort. mTLS certificates do not provide authorisation by IAM identity. One PrivateLink endpoint service per service across four accounts multiplies the NLBs and endpoints to manage.

**Key takeaway:** Mixed compute types, many accounts, overlapping CIDRs and IAM-based authorisation between services point to VPC Lattice. ECS Service Connect is the ECS-only, connectivity-not-included option.

</details>

**Question 1.1.5** A SaaS provider serves 3,000 small financial advisory firms from a pooled architecture. Requests pass through Amazon API Gateway, and a JWT from a shared Amazon Cognito user pool carries a `tenantId` claim. Tenant data is in a shared Amazon DynamoDB table with `tenantId` as the partition key. A regulator requires that a defect in application code must not let one tenant read another tenant's items. Twenty premium tenants have contracts that require the provider to make that tenant's data unreadable on request without affecting other tenants. Which combination of steps meets these requirements? (Select TWO.)

- A. Have each request assume an IAM role with session tag `TenantId` taken from the verified JWT claim. Grant DynamoDB access only through a policy with a `dynamodb:LeadingKeys` condition equal to `${aws:PrincipalTag/TenantId}`.
- B. Create a separate Cognito user pool for each of the 3,000 tenants, and route each sign-in to the tenant's pool so that tokens can be validated per tenant.
- C. Pass the tenant ID in an `X-Tenant-Id` request header that the frontend sets. Have every data-access function add a `tenantId` key condition to its DynamoDB queries.
- D. Encrypt the shared DynamoDB table with the AWS managed key for DynamoDB, and enable point-in-time recovery so that a premium tenant's data can be restored or removed on request.
- E. Move the premium tenants to a silo tier: a dedicated DynamoDB table per premium tenant, each encrypted with its own customer managed AWS KMS key. Disable or schedule deletion of the key to make that tenant's data unreadable.

<details>
<summary>Answer and explanation</summary>

**Correct answers: A and E**

**Why A and E are correct:**

- **A** enforces isolation in IAM rather than in application code.
  - The tenant ID comes from the verified token, not from client input, and becomes a session tag.
  - The `dynamodb:LeadingKeys` condition allows only the partition key that matches the tag, so a query bug that requests another tenant's key is denied by IAM.
  - This is the defence in depth the regulator requires.
- **E** applies a bridge model.
  - Premium tenants get their own table and their own customer managed key.
  - Disabling or deleting that key makes only that tenant's data unreadable (crypto-shredding), with no effect on other tenants.

**Why the other options are incorrect:**

- **B:** A user pool per tenant gives identity isolation but no data isolation, because the shared table still relies on application code. At 3,000 tenants it also brings heavy operational and automation effort for no gain over a shared pool with a tenant claim.
- **C:** A client-set header can be forged, and query conditions in application code are exactly the control a code defect would bypass.
- **D:** You cannot change the key policy of an AWS managed key, or disable or delete it, so per-tenant revocation is impossible. Point-in-time recovery restores a whole table to a point in time and does not remove one tenant's data.

**Key takeaway:** In pooled SaaS, derive tenant context from the verified token and enforce it in IAM (`LeadingKeys`, ABAC session tags) or in the database (row-level security). Use per-tenant customer managed KMS keys where independent revocation is a contractual requirement.

</details>

## Task 1.2: Integrate AI and ML services into architectures

This task tests whether you can place managed AI services inside an enterprise landing zone: who owns model access, how traffic and data stay private, where context comes from, and which inference option fits the traffic shape. The recurring decision is managed (Bedrock, Bedrock Knowledge Bases, AgentCore) versus self-hosted (SageMaker AI endpoints), and how to govern either across many accounts.

### Skill 1.2.1: Generative AI integrations and multi-account deployment patterns

**Key concepts**

- Amazon Bedrock is a Regional, API-driven service. Applications call `InvokeModel`/`Converse` (and streaming variants) on the `bedrock-runtime` endpoint. Use the Converse API when you want one request format across model providers and native tool use.
- Private access: create an interface VPC endpoint (AWS PrivateLink) for `bedrock-runtime` (and `bedrock-agent-runtime` for Knowledge Bases). Attach a VPC endpoint policy to restrict which principals and models can be called through it.
- Model access control is IAM-based. Scope `bedrock:InvokeModel` to specific foundation-model or inference-profile ARNs. Use SCPs to deny models or Regions organization-wide.
- **Inference profiles.** A system-defined cross-Region inference profile routes requests across a set of Regions for throughput. Geographic profiles (prefixes such as `us.`, `eu.`, `apac.`/`jp.`/`au.`) keep processing inside that geography. Global profiles (`global.` prefix) can route to any supported commercial Region. Inference does not store customer data in the destination Region; logs, knowledge bases and configuration stay in the source Region, and CloudTrail records the call in the source Region with the inference Region in the event data.
- **Application inference profiles** are profiles you create, wrapping a single-Region model or a system-defined cross-Region profile. Their main job is cost allocation: tag them per team or tenant and invoke through the profile ARN to get per-tag Bedrock spend.
- **IAM principal cost allocation** (since April 2026) attributes Bedrock usage to the calling IAM user or role automatically. Activate tags on those roles or users as cost allocation tags to group spend by caller in Cost Explorer and CUR 2.0. Use it when callers already map to distinct roles; use application inference profiles when one role serves several workloads.
- SCP gotcha for cross-Region inference: a Region-deny SCP must allow every destination Region in a geographic profile. Global profile requests show `aws:RequestedRegion` as `unspecified`, so a blanket Region-deny that does not exempt this blocks global inference. To forbid global routing for residency reasons, deny Bedrock calls that use `global.*` profile ARNs.
- **Multi-account patterns.**
  - *Centralised AI gateway account:* one platform account holds model access, Provisioned Throughput, Guardrails and invocation logging. Workload accounts call an internal API (API Gateway private API or ALB plus Lambda/containers) that enforces quotas, tenant tagging and guardrails, then calls Bedrock. Best for consistent governance, chargeback and rate limiting.
  - *Decentralised with guardrails:* each workload account calls Bedrock directly; the platform team enforces model and Region allow-lists through SCPs, and centralises invocation logs and CloudTrail. Best when teams need autonomy and lower latency, and quotas are per account anyway.
  - *Cross-account role assumption:* workload assumes a role in the AI account with a narrow `bedrock:InvokeModel` policy. Simple, but you lose the API layer for per-tenant throttling and prompt inspection.
- Bedrock service quotas (tokens per minute, requests per minute) are per account per Region. A central account concentrates all demand on one quota; spreading across accounts or using cross-Region inference raises effective throughput.
- **Model invocation logging** is off by default and configured per account per Region; it can send prompts and responses to CloudWatch Logs and/or S3. Use it for audit and for building evaluation datasets. CloudTrail records the API call, not the prompt content.
- **Throughput options:** on-demand (pay per token), batch inference (asynchronous jobs from S3, lower price than on-demand for supported models), and Provisioned Throughput (model units, required for some custom models). Choose batch for offline summarisation or classification of large corpora.
- Bedrock Guardrails can be applied to any model call through the `guardrailConfig` or the `ApplyGuardrail` API, which also works for models hosted outside Bedrock.

**Exam traps**

- "Use a global cross-Region inference profile to increase throughput" when the scenario has an EU data-residency rule. The geographic `eu.` profile meets residency; the global profile may route outside the EU.
- Choosing CloudTrail data events to capture prompts and completions. CloudTrail does not log prompt or response bodies; model invocation logging does.
- Putting all business units behind one account "for simplicity" when the scenario stresses per-team throughput. Quotas are per account per Region, so one account becomes the bottleneck unless you add cross-Region inference or quota increases.
- Expecting tags on the calling Lambda function or EC2 instance to split Bedrock spend. Use tagged application inference profiles (per workload) or IAM principal cost allocation (per caller role or user, since April 2026).
- Routing Bedrock traffic through a NAT gateway when the requirement says traffic must not traverse the internet. Use the PrivateLink interface endpoint.

**Hands-on**

1. **Private Bedrock call with logging.** Enable access to one low-cost text model in a Region, then turn on model invocation logging to a CloudWatch Logs group. Create an interface VPC endpoint for `bedrock-runtime` in a test VPC, attach an endpoint policy that allows only that model ARN, and call `Converse` from AWS CloudShell in VPC mode or a small EC2 instance in a private subnet. Try a second model and watch the endpoint policy deny it. Read the log group to see full prompt and response bodies, then compare with the CloudTrail event, which has no body. *Cost/cleanup:* a few cents of tokens; interface endpoints bill per hour per AZ, so delete the endpoint, the instance and the log group the same day.
2. **Chargeback with application inference profiles.** Create two application inference profiles for the same model with tags `team=alpha` and `team=beta`, activate the tag as a cost allocation tag in Billing, and send a few requests through each profile ARN. Next day, filter Cost Explorer by the tag. Then write an SCP (or IAM policy in a single account) that denies `bedrock:InvokeModel` unless the resource is an application inference profile, forcing all callers into tagged profiles. *Cost/cleanup:* cents; delete the profiles and policies. Tag data can take up to a day to appear.

### Skill 1.2.2: AI agent architectures that integrate with tools

**Key concepts**

- **Amazon Bedrock Agents** (now called Bedrock Agents Classic) gives an orchestration loop with action groups defined by OpenAPI schemas or function definitions, backed by Lambda or "return of control" to the caller. It closed to new customers on 30 July 2026, so treat it as existing customers only and use AgentCore for new designs.
- **Amazon Bedrock AgentCore** is a set of modular services for running agents built with any framework (Strands Agents, LangGraph, CrewAI, LlamaIndex, OpenAI Agents SDK and others) and any model. You can use the pieces independently:
  - *Runtime:* serverless hosting for agents and MCP servers, with each session isolated in its own microVM. Sessions can run for hours, far beyond Lambda's 15-minute limit. Supports MCP and the A2A protocol.
  - *Gateway:* turns Lambda functions, OpenAPI and Smithy API definitions, and existing MCP servers into MCP-compatible tools behind one endpoint, with semantic tool search so the agent does not need every tool in its prompt.
  - *Identity:* inbound authentication (IAM or JWT from an IdP such as Amazon Cognito, Okta or Microsoft Entra ID) and outbound credentials: OAuth 2.0 client credentials (2LO) and authorization code (3LO) flows, plus API keys, stored in a token vault.
  - *Memory:* short-term session events and long-term memory built by extraction strategies (semantic facts, summaries, user preferences, or custom).
  - *Code Interpreter* and *Browser:* managed sandboxes for running agent-written code and driving websites.
  - *Observability:* traces and metrics of agent steps in CloudWatch, using OpenTelemetry.
  - *Policy:* Cedar-based rules that intercept tool calls at the Gateway, default deny, evaluated against the caller's claims.
  - *Evaluations:* scoring of agent interactions against built-in or custom evaluators.
- VPC connectivity is available for Runtime, Gateway, Memory, Identity, Browser and Code Interpreter, so agents can reach private APIs and data.
- **Multi-account tool pattern:** each line of business hosts its tools (often as MCP servers on AgentCore Runtime) in its own account. A platform account runs one Gateway that registers each as a target. Users authenticate with the corporate IdP; Gateway validates the JWT, Policy checks claims per tool call, and Identity obtains an OAuth token to call the target. The target can restrict callers to the Gateway.
- Tool design rules: make tools idempotent, return structured errors, keep schemas small, and put a human approval step (Step Functions task token, or return of control) in front of irreversible actions.
- For new designs choose AgentCore: it covers simple agents and also your own framework, long sessions, MCP/A2A, enterprise OAuth delegation, and a shared tool catalogue across accounts. Bedrock Agents Classic appears only where an existing customer already runs it.

**Exam traps**

- Hosting a long-running research agent on Lambda. It fails once a session needs more than 15 minutes; AgentCore Runtime or a container service is the fit.
- Storing third-party OAuth refresh tokens in the agent's environment variables or DynamoDB. AgentCore Identity's token vault exists to keep agent credentials scoped and out of code.
- Giving the agent's execution role broad permissions to "every tool API" and relying on the prompt to restrict use. Enforcement belongs outside the model: Gateway Policy, IAM, and per-user OAuth scopes.
- Loading the full list of hundreds of tools into every prompt. Gateway semantic tool search reduces context cost and selection errors.

**Hands-on**

1. **Gateway over a Lambda tool.** Write a Lambda function that returns account balances from a hard-coded JSON file. In AgentCore, create a Gateway with a Cognito user pool as the inbound JWT authorizer and register the Lambda as a target with a small tool schema. Use an MCP client (for example the MCP Inspector or a short Strands Agents script) with a Cognito token to list and call the tool. Remove the token and confirm the call is rejected. *Cost/cleanup:* low; Gateway and Lambda are pay per use. Delete the Gateway, Lambda, and Cognito pool.
2. **Agent with memory on Runtime.** Deploy a sample Strands agent to AgentCore Runtime using the starter toolkit, attach an AgentCore Memory resource with a user-preference strategy, and have two conversations with the same user ID in different sessions. Confirm the second session recalls the preference, and view the trace in CloudWatch. *Cost/cleanup:* model tokens plus per-second Runtime compute, usually under a dollar for a short test. Delete the runtime, memory resource, ECR repository and CodeBuild project the toolkit created.

### Skill 1.2.3: ML inference architectures

**Key concepts**

- SageMaker AI has four inference options. Pick by payload, latency and traffic shape:

| Option | Fit | Notable limits and behaviour |
|---|---|---|
| Real-time endpoint | Steady traffic, low latency, GPUs | Persistent instances; 60-second request timeout for normal responses (streaming responses run longer); auto scaling; production variants; Model Monitor data capture |
| Serverless Inference | Spiky or intermittent traffic, tolerant of cold starts | Up to 4 MB payload and 60 seconds; no GPUs; scales to zero; pay per use |
| Asynchronous Inference | Large payloads or long processing, near-real-time | Up to 1 GB payload and up to 1 hour; requests queued internally, results to S3, SNS success and error notifications; can scale to zero |
| Batch transform | Offline scoring of a whole dataset | No persistent endpoint; reads from and writes to S3; jobs can run for days |

- **Multi-model endpoints (MME)** host many models behind one real-time endpoint, loading them from S3 on demand. Best for many small, similar models (per-tenant models) where some cold-load latency is acceptable. **Multi-container endpoints** host different containers, invoked directly or as a serial pipeline.
- **Inference components** let you place several models on one endpoint with per-model CPU/GPU/memory allocation and per-model scaling. With managed instance scaling, an endpoint built from inference components can scale down to zero instances. This is the modern answer for packing several large models (including LLMs) on GPU instances.
- Safe deployment: production variants with weights for A/B tests, shadow variants (shadow testing) to mirror live traffic to a new model without serving its answers, and deployment guardrails (blue/green with canary or linear traffic shifting plus CloudWatch alarm rollback).
- Network isolation: deploy endpoints in your VPC, call them through the SageMaker Runtime interface endpoint, and enable network isolation on the container if it must not make outbound calls.
- Bedrock or SageMaker AI? Bedrock for foundation models you consume as an API (including imported and fine-tuned custom models). SageMaker AI when you own the model artefact, need a specific container, framework, GPU type, or classic ML (XGBoost, fraud scoring, forecasting).
- Front-door patterns: API Gateway (REST, with a 29-second default integration timeout that can be raised for Regional and private REST APIs) or Lambda in front of real-time endpoints; S3 upload event plus async endpoint for long jobs; Step Functions to orchestrate batch transform.

**Exam traps**

- Serverless Inference for a model that needs a GPU, or for a 20-minute video analysis. It has no GPU support and a 60-second limit; asynchronous inference fits.
- Real-time endpoint kept running 24x7 for a nightly scoring job. Batch transform removes the idle cost.
- Multi-model endpoint for a few large models with strict latency. Dynamic loading adds cold-load delay; inference components or separate endpoints are better.
- Asynchronous inference for a synchronous user-facing call that needs a response in under a second. It is queue based and returns results to S3.

**Hands-on**

1. **Serverless versus asynchronous.** Train or download a small scikit-learn or XGBoost model (the SageMaker AI built-in XGBoost sample works). Deploy it once as a serverless endpoint (2048 MB memory) and once as an asynchronous endpoint on a small instance with an auto scaling policy that allows zero instances. Time the first (cold) and later calls on serverless; for async, upload an input file to S3, invoke, and read the output and SNS notification. *Cost/cleanup:* serverless is pay per request; the async instance bills while running, so delete both endpoints, endpoint configs and models right after.
2. **Batch transform and canary deployment.** Run a batch transform job against a CSV in S3 and confirm no endpoint remains afterwards. Then create a real-time endpoint with one variant, update it to a second model version using a canary traffic-shifting deployment guardrail with a CloudWatch alarm on `Invocation5XXErrors`, and watch the shift in the console. *Cost/cleanup:* smallest ml instance for under an hour costs little; delete the endpoint immediately because real-time endpoints bill per hour even when idle.

### Skill 1.2.4: RAG architectures, ingestion and retrieval strategies

**Key concepts**

- **Bedrock Knowledge Bases** manage the RAG pipeline: connect a data source, parse, chunk, embed, write to a vector store, then query with `Retrieve` (returns chunks you pass to your own prompt) or `RetrieveAndGenerate` (returns a grounded answer with citations).
- Unstructured data sources: Amazon S3, Confluence, Microsoft SharePoint, Google Drive, Microsoft OneDrive, a web crawler, and a custom source for direct ingestion through the API. Some sources support direct ingestion of individual documents without a full sync.
- **Structured data:** a knowledge base can connect to Amazon Redshift or the AWS Glue Data Catalog (with Lake Formation) and translate natural language into SQL through the Redshift query engine; `GenerateQuery` returns the SQL only. Use this instead of embedding tables.
- **Vector store options:**

| Store | Choose when |
|---|---|
| Amazon S3 Vectors | Large corpora, cost first, sub-second query latency acceptable; Knowledge Bases hybrid search not supported (semantic only); quick create supported |
| Amazon OpenSearch Serverless | Managed, hybrid (keyword plus vector) search, no cluster to run; has a baseline OCU charge even when idle |
| Amazon OpenSearch Service managed cluster | Existing OpenSearch estate or need for index tuning; Knowledge Bases requires the Faiss engine and does not use hybrid search on managed clusters |
| Amazon Aurora PostgreSQL (pgvector) | Team knows PostgreSQL, wants vectors next to relational data, moderate scale; supports Knowledge Bases hybrid search when the table has a filterable text field |
| Amazon Neptune Analytics | GraphRAG: relationships between entities matter |
| Pinecone, Redis Enterprise Cloud, MongoDB Atlas | Existing third-party vector platform |

- **Chunking** (fixed for the life of the data source; changing it means a new data source and re-ingestion): default, fixed-size with overlap, hierarchical (match small child chunks, return larger parents), semantic (splits on meaning, extra model cost at ingest), none (one chunk per file), or a custom Lambda transformation after chunking (to chunk yourself or add metadata).
- **Parsing:** the default parser reads text only. For tables, charts and scanned documents use a foundation-model parser or Bedrock Data Automation, both billed extra and applied to every document in the source.
- **Retrieval quality levers:** metadata filters (for tenant, department, document date), hybrid search where the store supports it, reranking models, query reformulation that breaks a complex question into sub-queries, and the number of results returned. Multimodal embedding models handle images.
- **Multi-tenant RAG:** a shared index with a mandatory tenant metadata filter applied by your backend (pooled), or one knowledge base or index per tenant (siloed) when contracts require hard isolation or tenant-specific KMS keys.
- **Freshness:** schedule syncs with EventBridge Scheduler or trigger `StartIngestionJob` from S3 events; syncs are incremental and process additions, changes and deletions.
- Security: S3 data source access through the knowledge base service role, customer managed KMS keys for the vector store and transient data, and document-level access control enforced through metadata filters set server side, never by the user's prompt.

**Exam traps**

- Choosing OpenSearch Serverless for a small, rarely queried proof of concept where cost is the main constraint. Its idle baseline cost makes S3 Vectors cheaper.
- Choosing S3 Vectors alone when the requirement is hybrid keyword plus semantic search on product codes. Knowledge Bases hybrid search works only with OpenSearch Serverless, Aurora PostgreSQL and MongoDB Atlas.
- Embedding a 50-million-row sales table into vectors to answer "total sales by region". Use a structured-data knowledge base over Redshift or the Glue Data Catalog, which generates SQL.
- Changing chunk size on an existing data source to fix bad answers. Chunking cannot be changed in place; create a new data source and re-ingest.
- Letting the client send the tenant ID as a free-text filter. A malicious client can query another tenant; the filter must be set server side from the authenticated identity.

**Hands-on**

1. **Knowledge base on S3 Vectors.** Upload 10 to 20 public PDFs (for example AWS whitepapers) to an S3 bucket, with a `.metadata.json` file next to each that sets a `department` attribute. Create a knowledge base using quick create with S3 Vectors and a Titan or Cohere embedding model, sync it, then call `Retrieve` with and without a metadata filter and compare the chunks returned. Call `RetrieveAndGenerate` and inspect the citations. *Cost/cleanup:* embedding a few hundred pages and storing vectors costs cents; delete the knowledge base, the vector bucket and index, and the source bucket.
2. **Chunking experiment.** Create a second data source on the same documents using hierarchical chunking and a third using semantic chunking (each requires a new data source). Ask the same five questions against each and note which returns complete answers for long procedural sections. Read the "how content chunking works" page in the Bedrock User Guide alongside. *Cost/cleanup:* semantic chunking adds model charges at ingest; keep the corpus small and delete the data sources and indexes afterwards.

### Practice questions: Task 1.2

**Question 1.2.1** A bank runs 40 workload accounts in AWS Organizations, all in eu-west-1 and eu-central-1. Several teams will call Amazon Bedrock models directly from their own accounts. Regulators require that all prompt processing stays within the EU. During peak hours teams hit throttling errors on on-demand inference. The platform team wants to raise throughput without moving teams into a shared account and wants a preventive control that stops anyone from accidentally sending prompts outside the EU. The design must do this in the most cost-effective way, without restricting teams to a single Region. Which solution meets these requirements?

- A. Have teams invoke the models through a global cross-Region inference profile, and attach an SCP that denies all actions where `aws:RequestedRegion` is not eu-west-1 or eu-central-1.
- B. Have teams invoke the models through the EU geographic cross-Region inference profile, update the Region-restriction SCP to allow every Region in that profile for Bedrock, and add an SCP that denies Bedrock invocation on `global.*` inference profile ARNs.
- C. Purchase Provisioned Throughput in eu-west-1 for each team, and attach an SCP that denies `bedrock:InvokeModel` in every Region other than eu-west-1.
- D. Create an application inference profile in each account that wraps the single-Region model in eu-west-1, and request a quota increase for each account.

<details>
<summary>Answer and explanation</summary>

**Correct answer: B**

**Why B is correct:** The EU geographic profile spreads requests across EU Regions, which raises usable throughput while keeping processing inside the EU. Geographic profiles need the SCP to allow every destination Region in the profile, otherwise calls fail. A separate deny on `global.*` profile ARNs is a preventive control that blocks the one routing option that could leave the EU. Teams keep calling from their own accounts.

**Why the other options are incorrect:**

- **A:** A global profile can route to any supported commercial Region, breaking the EU rule. The SCP as written would also block it, because global requests carry `unspecified` as the requested Region, so the design is both non-compliant and broken.
- **C:** Denying every Region except eu-west-1 breaks the teams that run in eu-central-1. Provisioned Throughput would add capacity, but it bills per model unit per hour whether used or not, often under a term commitment, so it is the most expensive way to fix peak throttling.
- **D:** An application inference profile wrapping a single-Region model only adds tagging for cost tracking. It does not spread load, and quota increases are not guaranteed, so it does not reliably address throttling.

**Key takeaway:** Geographic cross-Region inference raises throughput inside a residency boundary; global inference trades that boundary for more capacity, and SCPs must be written for each.

</details>

**Question 1.2.2** An insurance company is building an AI claims assistant. The agent must call 120 internal APIs owned by six business units, each in its own AWS account. The APIs are documented with OpenAPI specifications, and some are already exposed as MCP servers. Each tool call must be authorised against the end user's corporate identity in Okta, and business units must not hand long-lived credentials to the central team. Agent sessions sometimes run for over an hour while documents are analysed. Which solution meets these requirements with the least operational overhead?

- A. Host the agent on AWS Lambda with Step Functions to extend run time, store each business unit's API keys in AWS Secrets Manager in the central account, and load all 120 tool definitions into the system prompt.
- B. Build a Bedrock Agents agent with one action group per API, backed by Lambda functions in the central account that assume IAM roles in each business unit account.
- C. Run the agent on Amazon ECS on Fargate with a custom MCP proxy service that federates with Okta and stores OAuth tokens in DynamoDB encrypted with AWS KMS.
- D. Host the agent on Amazon Bedrock AgentCore Runtime, register the OpenAPI specifications and MCP servers as targets of one AgentCore Gateway with an Okta JWT inbound authorizer and AgentCore Policy rules, and use AgentCore Identity credential providers for outbound OAuth to each business unit.

<details>
<summary>Answer and explanation</summary>

**Correct answer: D**

**Why D is correct:** AgentCore Runtime hosts long sessions in isolated microVMs, so a one-hour analysis is not a problem. Gateway converts OpenAPI specifications and existing MCP servers into one MCP tool endpoint with semantic tool search, so the agent does not carry 120 tool schemas in every prompt. The Okta JWT authorizer plus Cedar policies enforce per-user authorisation on each tool call. AgentCore Identity obtains short-lived OAuth tokens from its vault, so business units issue client credentials scoped to the Gateway rather than long-lived keys.

**Why the other options are incorrect:**

- **A:** Splitting an agent loop across Lambda and Step Functions is complex, central API keys are the long-lived shared credentials the business units refuse to hand over, and 120 schemas in the prompt drive up cost and tool-selection errors.
- **B:** Bedrock Agents Classic closed to new customers on 30 July 2026, so a new build cannot start on it. Even for an existing customer, every API and existing MCP server would need its own Lambda-backed action group, and cross-account IAM role assumption authorises the Lambda function, not the end user's Okta identity.
- **C:** This can be built, but the team would own the MCP proxy, token storage and refresh, and per-user policy evaluation. That is the operational overhead the managed Gateway and Identity remove.

**Key takeaway:** For many tools across accounts with per-user authorisation, AgentCore Gateway plus Identity plus Policy is the managed pattern; enforcement sits outside the model.

</details>

**Question 1.2.3** A media company scores uploaded videos with a custom PyTorch model that needs a GPU. Each video file is 300 MB to 800 MB and inference takes 5 to 25 minutes. Uploads arrive in bursts during business hours and almost never at night or weekends. Users are notified by email when a result is ready. The company wants the most cost-effective design that does not require the team to manage queues. Which solution meets these requirements?

- A. Deploy the model to a SageMaker AI asynchronous inference endpoint on a GPU instance with an auto scaling policy that scales to zero, invoke it from an S3 event-driven Lambda function, and send the success notification from the endpoint's SNS topic to an email subscription.
- B. Deploy the model to a SageMaker AI serverless inference endpoint with maximum memory, and invoke it synchronously from a Lambda function triggered by S3 uploads.
- C. Deploy the model to a SageMaker AI real-time endpoint on GPU instances behind API Gateway, with target tracking scaling between one and four instances.
- D. Run a SageMaker AI batch transform job every hour from EventBridge Scheduler over all videos uploaded in the previous hour, and email users from a Lambda function when the job completes.

<details>
<summary>Answer and explanation</summary>

**Correct answer: A**

**Why A is correct:** Asynchronous inference accepts payloads up to 1 GB and processing up to an hour, which covers 800 MB files and 25-minute runs. It queues requests internally, so the team manages no queue, writes results to S3, and publishes success or error notifications to SNS for the email. It can scale to zero instances when idle, which removes night and weekend GPU cost.

**Why the other options are incorrect:**

- **B:** Serverless Inference has no GPU support, a 4 MB payload limit and a 60-second limit. It fails three stated requirements.
- **C:** Real-time endpoints cap normal requests at 60 seconds and keep at least one GPU instance running all the time, which wastes money nights and weekends.
- **D:** Batch transform works for bulk offline scoring but adds up to an hour of delay plus job start-up for every batch, and the team must write job tracking and per-user notification logic. It is not better on cost than async with scale to zero, and it is worse on latency.

**Key takeaway:** Large payload plus long processing plus bursty traffic points to SageMaker AI asynchronous inference with scale to zero.

</details>

**Question 1.2.4** A wealth management firm is building an internal research assistant over 30 million document chunks from analyst reports. Queries are infrequent, a few thousand per day, and sub-second response is acceptable. Advisers often search using exact ticker symbols and fund codes, and relevance tests show that pure semantic search misses these. The firm wants a managed Bedrock Knowledge Bases design, accepts reduced availability during an Availability Zone failure because the tool is internal, and wants the lowest-cost vector store that supports hybrid search in Bedrock Knowledge Bases. Which solution meets these requirements?

- A. Use Amazon S3 Vectors as the vector store and add a reranking model to the retrieval configuration so that ticker matches rise to the top.
- B. Use Amazon Aurora PostgreSQL Serverless v2 with pgvector as the vector store and set a metadata filter on ticker symbols for every query.
- C. Use an Amazon OpenSearch Service managed cluster with the S3 vector engine for low-cost storage, and enable hybrid search in the knowledge base retrieval configuration.
- D. Use Amazon OpenSearch Serverless as the vector store with standby replicas disabled for the development and test collection, and enable hybrid search.

<details>
<summary>Answer and explanation</summary>

**Correct answer: D**

**Why D is correct:** Exact codes need lexical matching, so the design needs Knowledge Bases hybrid search, which works with OpenSearch Serverless, Amazon RDS (Aurora PostgreSQL pgvector) and MongoDB vector stores. OpenSearch Serverless is fully managed and supports hybrid search directly in the retrieval configuration. Disabling standby replicas (the development and test setting) lowers the minimum OCU baseline, which the firm can accept because it tolerates reduced availability during an AZ failure. That makes it the lowest-cost managed store here that actually delivers hybrid search.

**Why the other options are incorrect:**

- **A:** Knowledge Bases cannot run hybrid search on S3 Vectors, so queries are semantic only. A reranker only reorders the candidates that semantic retrieval already found; if the ticker match was never retrieved, reranking cannot recover it.
- **B:** Aurora PostgreSQL does support Knowledge Bases hybrid search when the table has a filterable text field, but this option does not enable it. It relies on metadata filters instead, which work only if every chunk is tagged with every ticker it mentions and the query parser extracts tickers reliably.
- **C:** Knowledge Bases integration with an OpenSearch managed cluster requires the Faiss engine, not the S3 vector engine, and on managed clusters Knowledge Bases does not use hybrid search; queries fall back to semantic search, so tickers are still missed.

**Key takeaway:** Knowledge Bases hybrid search works only with OpenSearch Serverless, Amazon RDS/Aurora PostgreSQL and MongoDB Atlas; S3 Vectors and managed clusters fall back to semantic search.

</details>

**Question 1.2.5** A retail company has a single AWS account that hosts a customer support chatbot built on Amazon Bedrock. Security has three new requirements: no model traffic may leave the AWS network, the compliance team must be able to review the full text of prompts and model responses for 7 years, and finance wants Bedrock costs split between the web channel and the mobile channel. Which combination of actions meets these requirements? (Select TWO.)

- A. Create an interface VPC endpoint for `bedrock-runtime` with an endpoint policy that limits allowed models, and enable model invocation logging to an S3 bucket with a 7-year lifecycle and S3 Object Lock.
- B. Create a CloudTrail trail with data events for Amazon Bedrock and store the logs in S3 with a 7-year retention.
- C. Route all Bedrock calls through a NAT gateway with an AWS Network Firewall domain allow-list for the Bedrock endpoint.
- D. Tag each Bedrock-related Lambda function with `channel` and activate the tag for cost allocation.
- E. Create two application inference profiles tagged `channel=web` and `channel=mobile`, activate the tag for cost allocation, and have each backend invoke the model through its profile ARN.

<details>
<summary>Answer and explanation</summary>

**Correct answer: A, E**

**Why A and E are correct:** The interface endpoint keeps calls on PrivateLink inside the AWS network, and the endpoint policy narrows usage. Model invocation logging is the feature that captures full prompt and response bodies; sending it to S3 with lifecycle rules and Object Lock gives cheap, tamper-resistant 7-year retention. Tagged application inference profiles carry their tags onto Bedrock on-demand usage, so each channel's cost shows separately under the cost allocation tag.

**Why the other options are incorrect:**

- **B:** CloudTrail records who called which API and when, but not the prompt and completion text, so it does not meet the content review requirement.
- **C:** A NAT gateway sends traffic through the internet gateway to the public Bedrock endpoint, so it does not meet the private-path requirement the way PrivateLink does. It also adds per-GB NAT and firewall processing charges and does nothing for logging or cost split.
- **D:** Tags on the calling compute resources are not applied to Bedrock usage line items, so the Lambda tags never reach the Bedrock spend.

**Key takeaway:** Prompts in logs means model invocation logging; split Bedrock spend means tagged application inference profiles or IAM principal cost allocation, never tags on the calling compute.

</details>

## Task 1.3: Design hybrid cloud integration patterns

This task tests whether you can connect an on-premises estate to AWS so that it survives failures, keeps data in sync, resolves names both ways, and runs compute where latency or residency demands. The exam keeps asking you to choose between options with different failure domains, costs and lead times: Direct Connect versus VPN, DataSync versus Storage Gateway, Outposts versus Local Zones versus EKS/ECS on your own hardware.

### Skill 1.3.1: Hybrid connectivity with redundancy and failover

**Key concepts**

- **Direct Connect (DX) building blocks.** A dedicated connection is a physical port (1, 10 or 100 Gbps, with higher speeds at some locations) at a DX location. A hosted connection comes from a partner at fixed bandwidths. Virtual interfaces (VIFs): private VIF to a virtual private gateway (VGW) or DX gateway; transit VIF to a DX gateway associated with Transit Gateways; public VIF to reach AWS public endpoints (S3, DynamoDB, public service IPs) over DX.
- **DX gateway** is a global object: one private or transit VIF can reach VGWs or Transit Gateways in many Regions (not China). It does not route traffic between the VPCs or TGWs attached to it.
- **Resiliency models** (AWS Direct Connect Resiliency Toolkit): *maximum* resiliency is two connections at each of two DX locations (four in total); *high* resiliency is one connection at each of two locations; *development and test* is two connections at one location. The exam ties "critical production" to separate locations and separate devices.
- **DX is not encrypted by default.** Options: MACsec on dedicated 10/100 Gbps connections at supporting locations (layer 2, hop between your router and the AWS device), or IPsec Site-to-Site VPN over DX (a public VIF, or private IP VPN over a transit VIF to a TGW).
- **Site-to-Site VPN:** two IPsec tunnels per connection, terminated on a VGW or TGW (or, for private IP VPN, across DX). A standard tunnel supports up to about 1.25 Gbps; on a Transit Gateway you get more aggregate bandwidth with ECMP across several VPN connections using BGP. Accelerated VPN uses the Global Accelerator edge network to improve tunnel performance, available only on TGW attachments.
- **Failover design:** DX primary plus VPN backup is the classic low-cost pattern. Use BGP: AWS prefers DX over VPN for the same prefix when both are on the same VGW or TGW. Influence on-premises to AWS traffic with AS path prepending or more specific prefixes; influence AWS to on-premises with local preference BGP communities (7224:7100 low, 7224:7200 medium, 7224:7300 high) on private and transit VIFs. Enable BFD on DX for faster failure detection.
- **Transit Gateway** is the hub: VPC, VPN, DX gateway (via transit VIF), Connect (GRE plus BGP for SD-WAN appliances) and peering attachments. Route tables per attachment give segmentation (for example prod, non-prod, shared, inspection). TGW peering across Regions uses static routes, not BGP propagation.
- Lead time matters: a new DX connection takes weeks. If a scenario needs connectivity "next week", Site-to-Site VPN is the answer for now, possibly as permanent backup.
- **SiteLink** lets on-premises sites talk to each other over the AWS backbone through DX locations, without going into a VPC.
- **Cost model:** DX charges per port-hour plus data transfer out (lower per-GB than internet). TGW charges per attachment-hour and per GB processed. VPN charges per connection-hour. Remember TGW processing fees when an exam asks for the cheapest path for high-volume flows.

**Exam traps**

- Two DX connections at the same location called "highly available". They share the location as a failure domain; the guide calls this the development and test model.
- Expecting DX to encrypt traffic. It does not; add MACsec or IPsec VPN over DX when the requirement says encryption in transit.
- Using a DX gateway to route between two VPCs. It does not provide VPC-to-VPC transit; use TGW or peering.
- Single VPN tunnel throughput assumed to scale with a bigger VGW. VGW does not support ECMP across VPN connections; use TGW with ECMP.
- Ignoring BGP: setting static routes on both DX and VPN and expecting automatic failover.

**Hands-on**

1. **VPN with BGP failover to a simulated data centre.** Create two VPCs: "cloud" and "onprem". In onprem, launch a small Linux instance running strongSwan and FRR as a customer gateway with an Elastic IP. In cloud, create a Transit Gateway, a Site-to-Site VPN attachment with dynamic routing, and download the configuration. Bring up both tunnels with BGP, confirm routes propagate to the TGW route table, then stop one tunnel and watch traffic continue on the other. *Cost/cleanup:* VPN connection, TGW attachments and the instance each bill hourly (expect a few dollars per day); delete the VPN, TGW attachments, TGW and instances when done.
2. **Direct Connect design without buying a port.** Use the Direct Connect console's Resiliency Toolkit wizard to step through the maximum resiliency model and read the generated design, then read the "Direct Connect gateways" and "BGP communities" pages. Draw a diagram with a DX gateway, transit VIFs from two locations, and TGWs in two Regions, and mark where you would set local preference communities and AS path prepending. *Cost/cleanup:* free; do not order connections.

### Skill 1.3.2: Data synchronisation strategies for hybrid workloads

**Key concepts**

- **AWS DataSync** moves data between on-premises storage (NFS, SMB, HDFS, object storage), other clouds, and AWS storage (S3, EFS, all FSx file systems). An agent (VM) runs on-premises for on-premises sources; AWS-to-AWS transfers run agentless. It does scheduled, incremental transfers with integrity verification, bandwidth limits, and filters. Use it for migrations, one-way recurring sync, and data lake ingestion. It is not a live, low-latency file system.
- DataSync over DX: use a private (VPC) interface endpoint for the DataSync service so agent traffic stays on the private VIF.
- **AWS Storage Gateway** gives on-premises applications local protocol access backed by AWS storage, with a local cache:
  - *S3 File Gateway:* NFS/SMB to objects in S3, one object per file. Good for backups, archives and feeding a data lake while apps keep using file shares. Use `RefreshCache` or automatic cache refresh when objects change in S3 from elsewhere.
  - *Volume Gateway* (cached or stored iSCSI volumes, snapshots to EBS) and *Tape Gateway* (virtual tape library to S3 Glacier storage classes for existing backup software).
- **AWS Transfer Family** provides managed SFTP, FTPS, FTP and AS2 endpoints in front of S3 or EFS, with service-managed users, AWS Directory Service, or a custom identity provider (Lambda or API Gateway). Managed workflows can run post-upload processing. SFTP connectors let AWS push or pull files to partner SFTP servers. Use it to replace a partner-facing SFTP server without changing partners.
- **Decision rules:** one-time or scheduled bulk copy, use DataSync. On-premises apps need a share that lands in S3, use S3 File Gateway. Partner exchange over SFTP or AS2, use Transfer Family. Database changes, use AWS DMS (Task 4.1). Near-real-time object replication between buckets, use S3 Replication.
- Offline transfer: Snowball Edge is closed to new customers (7 Nov 2025), so do not pick it for new designs. For large one-off moves, consider DataSync over a temporary higher-bandwidth link or DX.

**Exam traps**

- Using DataSync as a shared file system for applications that read and write continuously. It is a transfer service, not a mount point.
- Using S3 File Gateway when other systems write directly to the bucket and the on-premises users must see the changes immediately. The gateway cache must be refreshed; without refresh, users see stale listings.
- Building an EC2-hosted SFTP server with a cron job to S3 when the scenario asks for least operational overhead. Transfer Family is managed and supports custom identity providers.
- Picking Snowball Edge for a new migration. It is closed to new customers.

**Hands-on**

1. **DataSync agentless copy and incremental behaviour.** Create an EFS file system and an S3 bucket in the same Region, put a few thousand small files on EFS from an EC2 instance, and create a DataSync task from EFS to S3 with verification enabled. Run it, change ten files, run it again, and compare the "files transferred" counts to see incremental transfer. *Cost/cleanup:* DataSync charges per GB copied, EFS and S3 storage are pennies at this size; delete the task, locations, EFS, instance and bucket.
2. **Transfer Family SFTP with workflow.** Create an SFTP server with service-managed users backed by an S3 bucket, add a user with an SSH key and a logical home directory, and attach a managed workflow that tags each uploaded file and copies it to an "inbound" prefix. Upload with `sftp` from your laptop or CloudShell and check the tags and copies. *Cost/cleanup:* Transfer Family bills per protocol-hour while the server exists (noticeable if left for days), so delete the server within the hour.

### Skill 1.3.3: Integration patterns to modernise legacy systems with data transformations

**Key concepts**

- **Amazon MQ** is managed Apache ActiveMQ or RabbitMQ. Choose it when existing applications speak JMS, AMQP 0-9-1 or 1.0, MQTT, STOMP or OpenWire and you want to move them without code changes. Use active/standby (ActiveMQ) or a cluster (RabbitMQ) across AZs for availability. For new cloud-native designs, SQS, SNS and EventBridge are cheaper and serverless.
- Lambda can consume from Amazon MQ (ActiveMQ and RabbitMQ) through an event source mapping, which lets you bolt serverless transformation onto a legacy broker.
- **API Gateway as a façade:** REST APIs support request and response mapping templates (VTL) to translate JSON to the XML or form formats a legacy backend expects, without Lambda. Use private integrations through a VPC link to reach on-premises or VPC backends (REST APIs classically through an NLB; HTTP APIs through an ALB, NLB or AWS Cloud Map). HTTP APIs are cheaper but have fewer transformation features (parameter mapping only).
- **Lambda for transformation** when logic is non-trivial (fixed-width mainframe records to JSON, enrichment from a lookup table). Keep functions idempotent.
- **EventBridge Pipes** connect a source (SQS, Kinesis, DynamoDB Streams, Amazon MQ, Amazon MSK, self-managed Kafka) to a target with optional filtering, enrichment (Lambda, Step Functions, API Gateway, API destinations) and input transformation. Fewer moving parts than a hand-written Lambda poller.
- **EventBridge API destinations** call external HTTP endpoints with managed authorisation and rate limiting, useful for pushing events into legacy SaaS or on-premises HTTP endpoints.
- **Amazon AppFlow** moves data between SaaS applications (Salesforce, SAP OData, ServiceNow and others) and S3, Redshift or Snowflake with field mapping and filtering.
- **Strangler fig with a façade:** put API Gateway or an ALB in front of the legacy system, route new paths to new services and leave others to the legacy backend, then move routes over time.
- Use Step Functions with direct service integrations for multi-step orchestration and compensation; use an SQS buffer to protect a fragile legacy system from bursts (with Lambda reserved concurrency or maximum concurrency on the event source to cap the rate).

**Exam traps**

- Rewriting producers to use SQS when the requirement is "no changes to the existing JMS applications". Amazon MQ preserves the protocol.
- Using HTTP APIs when the scenario needs a request body transformation from JSON to SOAP/XML without code. REST API mapping templates do this; HTTP APIs offer only parameter mapping.
- Calling a legacy on-premises system directly from a high-concurrency Lambda function. Lambda can open thousands of connections; put a queue in front and cap concurrency.
- Choosing Amazon MQ as the default for new event-driven designs. It runs brokers you pay for by the hour and is not serverless.

**Hands-on**

1. **SOAP façade with mapping templates.** Create a REST API with a POST method whose integration is an HTTP endpoint (use a public echo service or a Lambda function URL that returns its body). Add a request mapping template that turns `{"accountId":"123"}` into an XML envelope and a response mapping template that turns the XML back into JSON. Test in the console and inspect the transformed request in the execution logs. *Cost/cleanup:* within the API Gateway free tier for low volume; delete the API and log group.
2. **Amazon MQ to Lambda.** Create a single-instance RabbitMQ broker on the smallest instance type, publish messages to a queue from a Python script using the AMQP library, and configure a Lambda event source mapping from that queue (credentials in Secrets Manager). Have the function transform each message and write it to DynamoDB. *Cost/cleanup:* broker bills hourly (a few dollars per day on the smallest size, not free tier after the first month); delete the broker, function, secret and table the same day.

### Skill 1.3.4: DNS solutions for hybrid environments

**Key concepts**

- The Route 53 Resolver (at VPC CIDR base plus two, and `169.254.169.253`) answers VPC queries, including private hosted zones associated with the VPC.
- **Inbound endpoint:** ENIs in your VPC that accept DNS queries from on-premises resolvers. On-premises conditional forwarders send `aws.example.com` to these IPs. Use at least two AZs.
- **Outbound endpoint plus forwarding rules:** forward queries for `corp.example.com` from VPCs to on-premises DNS servers. Rules can be shared with other accounts through AWS RAM and associated with their VPCs. System rules override forwarding for subdomains you want resolved locally.
- **Centralised hybrid DNS:** put inbound and outbound endpoints in a shared-services VPC, share forwarding rules through RAM, and associate private hosted zones from many accounts with the shared VPC (cross-account association needs an authorisation from the zone owner, then the association from the VPC owner). **Route 53 Profiles** simplify this: a Profile bundles private hosted zone associations, Resolver rules and DNS Firewall rule groups, is shared through RAM, and is associated with VPCs (one Profile per VPC; Regional).
- **Split-horizon DNS:** a public hosted zone and a private hosted zone with the same name. VPCs associated with the private zone get private answers; everyone else gets public answers. Private zone resolution does not fall back to the public zone for records missing in the private zone.
- **Route 53 Resolver DNS Firewall** filters outbound DNS queries from VPCs by domain lists (allow, block, alert), including managed threat lists. It stops DNS-based exfiltration that security groups cannot see.
- **Resolver query logging** to CloudWatch Logs, S3 or Firehose for audit; can be shared across accounts with RAM.
- Interface VPC endpoints with private DNS only resolve privately inside VPCs. For on-premises clients, use the inbound endpoint so on-premises DNS can resolve the private DNS names, or use the endpoint-specific DNS names.

**Exam traps**

- Pointing on-premises DNS servers at the VPC resolver's base-plus-two IP. It is reachable only from inside the VPC; you need an inbound endpoint.
- Creating outbound endpoints in every VPC. Centralise endpoints and share rules or Profiles to cut cost (endpoint ENIs bill per hour).
- Expecting a private hosted zone to answer for records it lacks by falling back to the public zone of the same name. It returns NXDOMAIN.
- Forgetting that Resolver rules and Profiles are Regional; a second Region needs its own endpoints and rules.

**Hands-on**

1. **Two-way hybrid DNS between VPCs.** Reuse the "onprem" VPC from the VPN exercise (or peer two VPCs). In onprem, run a small instance with BIND or dnsmasq serving `corp.example.internal`. In cloud, create a private hosted zone `aws.example.internal`, an inbound endpoint and an outbound endpoint in two AZs, and a forwarding rule for `corp.example.internal` pointing to the BIND server. Configure BIND to forward `aws.example.internal` to the inbound endpoint IPs, then resolve both names from both sides with `dig`. *Cost/cleanup:* each endpoint ENI bills hourly (roughly a few dollars per day for two endpoints in two AZs) plus per-query charges; delete endpoints, rules and instances afterwards.
2. **Split-horizon and DNS Firewall.** Create a public hosted zone and a private hosted zone for a test domain (or a subdomain of one you own), give `app` different A records in each, and query from inside and outside the VPC. Then attach a DNS Firewall rule group that blocks a test domain and returns NXDOMAIN, and see the block in Resolver query logs. *Cost/cleanup:* hosted zones bill monthly (pro-rated small), queries are cheap; delete the zones, rule group and log configuration.

### Skill 1.3.5: Hybrid compute patterns

**Key concepts**

- **AWS Outposts:** AWS-owned racks (or servers) installed in your data centre, managed by AWS, running EC2, EBS, ECS, EKS, RDS and S3 on Outposts locally. Needs a reliable service link to the parent Region; the control plane is in the Region. Choose it for low-latency access to on-premises systems, local data processing, or data residency where data must stay on site, when you want AWS APIs and tooling.
- **Local Zones:** AWS infrastructure in metro areas, extending a parent Region with a subset of services (EC2, EBS, some ELB, ECS/EKS nodes, others vary by zone). You enable the zone group, create a subnet in it, and pay AWS rates. Choose for single-digit millisecond latency to end users in a city (media, gaming, real-time trading UI) without owning hardware. Dedicated Local Zones exist for customers needing exclusive infrastructure.
- **Wavelength Zones** place compute inside telecom 5G networks for mobile edge latency.
- **Amazon ECS Anywhere:** register your own servers or VMs as external instances (via SSM Agent and an activation) and run tasks with the `EXTERNAL` launch type. Control plane stays in AWS. Limits: no service load balancing, no service discovery, `bridge`/`host`/`none` network modes only (no `awsvpc`), no capacity providers, no EFS volumes. Fits batch and outbound-traffic workloads near on-premises data. Charged per instance-hour while registered. Windows support has been deprecated and several older Linux distributions were dropped in 2026, so check the supported OS list.
- **Amazon EKS Hybrid Nodes:** your on-premises machines join an EKS cluster whose control plane AWS manages in the Region. Needs reliable network connectivity (VPN or DX) to the cluster's VPC; not for disconnected sites. Charged per vCPU-hour of attached hybrid nodes. Supports EKS add-ons, Pod Identity and access entries; credentials via SSM hybrid activations or IAM Roles Anywhere.
- **Amazon EKS Anywhere:** you run the whole Kubernetes cluster, control plane included, on your own infrastructure (VMware vSphere, bare metal, Nutanix, Apache CloudStack). Can run fully disconnected or air-gapped. Software is free; paid Enterprise Subscription for AWS support.
- **Decision rules:** must run disconnected, use EKS Anywhere. Connected and want AWS to run the Kubernetes control plane, use EKS Hybrid Nodes. Want AWS-managed hardware and EC2/EBS/RDS on site, use Outposts. Need low latency for users in a city with no on-site hardware, use Local Zones. Simple container tasks on existing servers with outbound-only traffic, ECS Anywhere.
- **Systems Manager hybrid activations** manage on-premises servers (patching, Run Command, Session Manager) alongside EC2; IAM Roles Anywhere gives on-premises workloads temporary AWS credentials from X.509 certificates.

**Exam traps**

- Outposts for a site with frequent network outages that must keep provisioning new resources. Outposts depends on the service link to the Region for its control plane; EKS Anywhere is designed for disconnected operation.
- ECS Anywhere for an inbound web service that needs an ALB. Service load balancing is not supported on external instances.
- EKS Hybrid Nodes in an air-gapped plant. It needs a reliable connection to the Region-hosted control plane.
- Local Zones for data that must remain inside the customer's own building. A Local Zone is an AWS facility, not the customer's premises.
- Assuming every service exists in every Local Zone. Check per-zone service availability.

**Hands-on**

1. **ECS Anywhere on a VM.** On a spare Linux VM (a laptop VM, or an EC2 instance pretending to be on-premises, which still incurs the ECS Anywhere fee), create an ECS cluster, generate an SSM activation from the "Register external instance" wizard, and run the install script. Run a task definition with `EXTERNAL` launch type and `bridge` networking that writes logs to CloudWatch. Try to create a service with a load balancer and note that the option is not available for external instances. *Cost/cleanup:* ECS Anywhere bills per registered instance-hour; deregister the instance, delete the SSM activation and managed instance entry, and delete the cluster.
2. **Local Zone latency test.** Opt in to a Local Zone near you (if one exists), create a subnet in it, and launch the smallest supported instance there and another in the parent Region. Measure round-trip time from your laptop to both with `ping` or `mtr` and compare. If no Local Zone is near you, read the "AWS Local Zones features" page and list which services your nearest zone offers. *Cost/cleanup:* Local Zone instances cost more than Regional ones and are not free tier; terminate them within the hour and delete the subnet.

### Practice questions: Task 1.3

**Question 1.3.1** A payments processor connects its two data centres to AWS. It runs critical workloads in 30 VPCs across us-east-1 and us-west-2 attached to a Transit Gateway in each Region. The company requires that the loss of a single Direct Connect location, device or connection must not interrupt connectivity, traffic between data centres and AWS must be encrypted, and the network team wants to minimise the number of BGP sessions it manages. Which solution meets these requirements?

- A. Order two dedicated connections at one Direct Connect location with MACsec, create a private VIF on each to a virtual private gateway in every VPC, and use BGP to fail over between them.
- B. Order one dedicated connection at each of two Direct Connect locations, create a transit VIF on each to a single Direct Connect gateway associated with both Transit Gateways, and run Site-to-Site VPN on Transit Gateway over the internet for encryption.
- C. Order two dedicated connections at each of two Direct Connect locations with MACsec enabled, create a transit VIF on each connection to one Direct Connect gateway associated with both Transit Gateways.
- D. Order hosted connections at two Direct Connect locations from a partner, create a public VIF on each, and run Site-to-Site VPN tunnels over the public VIFs to each Transit Gateway.

<details>
<summary>Answer and explanation</summary>

**Correct answer: C**

**Why C is correct:** Two connections at each of two locations is the maximum resiliency model, so losing a location, device or connection leaves capacity. MACsec encrypts traffic on the dedicated connections at layer 2 without adding IPsec tunnels. Transit VIFs to one DX gateway associated with both Regional Transit Gateways reach all 30 VPCs in both Regions with one BGP session per connection.

**Why the other options are incorrect:**

- **A:** Both connections are in one location, which is a single failure domain. A private VIF per VPC also means dozens of BGP sessions, the opposite of what the team wants.
- **B:** The internet VPN encrypts only traffic that goes through its tunnels. Traffic on the transit VIFs stays unencrypted, so either the DX traffic breaks the encryption rule or all traffic must move to the VPN and DX is wasted. The extra tunnels also add BGP sessions.
- **D:** IPsec over public VIFs does give encryption over DX, but it needs VPN tunnels to each TGW with their own BGP sessions, and hosted connections cap bandwidth. It meets encryption but fails "minimise BGP sessions" and offers weaker resiliency.

**Key takeaway:** Critical workloads mean separate DX locations; encryption on DX means MACsec or IPsec on top; transit VIFs plus a DX gateway scale to many VPCs and Regions.

</details>

**Question 1.3.2** A hospital group has 40 TB of medical images on an on-premises NAS that clinicians access over SMB. The archive grows by 200 GB per week. The group wants new images to land in Amazon S3 so a research team can analyse them with Amazon Athena, while clinicians keep using SMB file shares with fast access to recent images. The NAS reaches end of support in four months and will not be renewed. The research team will not write back to the bucket. The group has a 1 Gbps Direct Connect link and wants the least operational overhead. Which solution meets these requirements?

- A. Deploy an S3 File Gateway on-premises, use a one-time AWS DataSync task over Direct Connect to copy the existing 40 TB into the bucket, and move clinicians to the gateway's SMB share for new and existing images.
- B. Run AWS DataSync on a nightly schedule from the NAS to S3, and keep clinicians on the existing NAS.
- C. Deploy a Volume Gateway in cached mode, present it as an iSCSI LUN to the file server, and export snapshots to S3 for the research team.
- D. Configure AWS Transfer Family with an SFTP endpoint backed by the S3 bucket, and have a script on the NAS upload new images every hour.

<details>
<summary>Answer and explanation</summary>

**Correct answer: A**

**Why A is correct:** S3 File Gateway presents an SMB share that writes each file as an S3 object, so new images land in S3 for Athena-based analysis while clinicians keep a share with a local cache for recent files. DataSync is the efficient way to seed the 40 TB existing archive over Direct Connect. Because researchers do not write to the bucket, the gateway cache does not need frequent refreshes.

**Why the other options are incorrect:**

- **B:** Scheduled DataSync gets data into S3, but it keeps clinicians on a NAS that is about to lose support, so the company would still have to replace it. It also adds up to a day of lag before research can see new images.
- **C:** Volume Gateway stores block volumes; snapshots become EBS snapshots, not individual S3 objects that Athena can query.
- **D:** Transfer Family serves external partners with SFTP. It needs custom scripts and does not give clinicians a cached SMB share.

**Key takeaway:** On-premises apps keep a file share and data must land as S3 objects: S3 File Gateway; bulk seeding: DataSync.

</details>

**Question 1.3.3** A logistics company runs 25 legacy Java applications that exchange messages through an on-premises Apache ActiveMQ broker that the applications reach through JMS over OpenWire. The company is moving the applications to Amazon EC2 in one Region and wants a managed broker with no application code changes. A new analytics service must receive a transformed JSON copy of every shipment message without the legacy applications being modified, and the solution must survive an Availability Zone failure. Which combination of steps meets these requirements? (Select TWO.)

- A. Migrate the broker to Amazon MQ for ActiveMQ in an active/standby deployment across two Availability Zones.
- B. Replace the broker with Amazon SQS FIFO queues and update the applications to use the Amazon SQS Java Messaging Library.
- C. Configure an Amazon EventBridge pipe with the Amazon MQ queue as its source, a Lambda enrichment step that converts messages to JSON, and the analytics service's Amazon SQS queue as the target.
- D. Migrate the broker to Amazon MQ for RabbitMQ in a single-instance deployment and use RabbitMQ shovels to copy messages to the analytics service.
- E. Configure Amazon Data Firehose to subscribe to the broker topics and deliver JSON records to Amazon S3 for the analytics service.

<details>
<summary>Answer and explanation</summary>

**Correct answer: A, C**

**Why A and C are correct:** Amazon MQ for ActiveMQ supports OpenWire and JMS, so applications move without code changes, and active/standby across two AZs survives an AZ failure. An EventBridge pipe can read from an Amazon MQ queue, enrich (transform) each message with Lambda, and deliver to an SQS target, adding the analytics feed without touching the legacy applications.

**Why the other options are incorrect:**

- **B:** SQS needs client changes, and its JMS library supports only part of the JMS API; the requirement forbids code changes.
- **D:** RabbitMQ does not speak OpenWire, so applications would need changes, and a single-instance broker does not survive an AZ failure.
- **E:** Firehose cannot subscribe to an ActiveMQ broker as a source, so this integration does not exist without custom code.

**Key takeaway:** Lift a JMS or OpenWire estate to Amazon MQ for ActiveMQ; add new consumers and transformations with EventBridge Pipes or Lambda event source mappings rather than changing producers.

</details>

**Question 1.3.4** A company has 120 AWS accounts in AWS Organizations, each with its own VPCs in eu-west-1, connected to on-premises through a Transit Gateway and Direct Connect. On-premises servers must resolve records in private hosted zones owned by individual application accounts, and every VPC must resolve the on-premises `corp.example.com` domain. The network team wants one place to manage DNS, consistent DNS Firewall rules in every VPC, and the lowest endpoint cost. Which solution meets these requirements?

- A. Create Route 53 Resolver inbound and outbound endpoints in every VPC, and create a forwarding rule for `corp.example.com` in each account.
- B. Create inbound and outbound endpoints in a shared-services VPC, and create a Route 53 Profile containing the `corp.example.com` forwarding rule, the DNS Firewall rule group and the application private hosted zones; share the Profile with the organization through AWS RAM and associate it with every VPC.
- C. Create inbound and outbound endpoints in a shared-services VPC, configure on-premises DNS to forward to the inbound endpoint, and set each VPC's DHCP option set to use the on-premises DNS servers.
- D. Peer every VPC to a shared-services VPC that runs BIND servers on EC2, and set all VPCs' DHCP option sets to the BIND servers, which forward to on-premises and to the VPC resolver.

<details>
<summary>Answer and explanation</summary>

**Correct answer: B**

**Why B is correct:** Centralised inbound and outbound endpoints minimise endpoint ENI charges. A Route 53 Profile bundles forwarding rules, DNS Firewall rule groups and private hosted zone associations, and sharing it through RAM applies the same DNS configuration to every associated VPC from one place. On-premises DNS forwards to the inbound endpoint in the shared VPC, which can answer for every private hosted zone included in the Profile associated with that VPC.

**Why the other options are incorrect:**

- **A:** Endpoints in every VPC multiply hourly ENI costs across 120 accounts and spread rule management, failing both the cost and central-management goals.
- **C:** Pointing VPCs at on-premises DNS sends every lookup, including AWS service names, across Direct Connect and makes AWS resolution depend on the data centre. DNS Firewall in VPCs would also be bypassed for those queries.
- **D:** Self-managed BIND servers work but add patching, scaling and availability work, and VPC peering does not scale well to 120 VPCs. It fails least operational overhead.

**Key takeaway:** Centralise Resolver endpoints and distribute DNS configuration with Route 53 Profiles or RAM-shared rules.

</details>

**Question 1.3.5** A manufacturer runs a quality-inspection application on Kubernetes in three factories. Two factories have stable fibre links to AWS. The third is in a remote area where the WAN link drops for several hours at a time, and production must continue during outages. The company wants AWS to operate the Kubernetes control plane wherever possible and wants to reuse its existing servers rather than buy new hardware. Which solution meets these requirements?

- A. Install AWS Outposts servers in all three factories and run Amazon EKS local clusters on them.
- B. Use Amazon EKS Hybrid Nodes in all three factories, connected to an EKS cluster in the nearest Region through Site-to-Site VPN.
- C. Deploy Amazon EKS Anywhere in all three factories with an Enterprise Subscription.
- D. Use Amazon EKS Hybrid Nodes for the two connected factories, and deploy Amazon EKS Anywhere on the existing servers in the remote factory.

<details>
<summary>Answer and explanation</summary>

**Correct answer: D**

**Why D is correct:** EKS Hybrid Nodes lets existing on-premises servers join an EKS cluster whose control plane AWS manages, which fits the two factories with reliable connectivity. EKS Hybrid Nodes is not designed for disconnected or intermittent sites, so the remote factory uses EKS Anywhere, which runs the full cluster locally and keeps operating while disconnected, on the hardware already there.

**Why the other options are incorrect:**

- **A:** Outposts is new AWS hardware, which conflicts with reusing existing servers, and it costs far more than the alternatives for this need.
- **B:** Hybrid nodes depend on a reliable link to the Region-hosted control plane, so the remote factory would lose scheduling and self-healing during outages.
- **C:** EKS Anywhere would work everywhere, but the customer operates every control plane, ignoring the stated preference for AWS to run it where connectivity allows.

**Key takeaway:** Connected and wants AWS-managed control plane: EKS Hybrid Nodes; disconnected or air-gapped: EKS Anywhere.

</details>

## Task 1.4: Design network architectures

This task tests whether you can design networks for large multi-account estates: how VPCs connect, how address space is planned and handed out, where traffic is inspected, how to adopt IPv6, and how users reach the nearest healthy Region. The recurring decision is which connectivity construct fits the scale, overlap and cost (peering, Transit Gateway, PrivateLink, VPC Lattice, Cloud WAN), and where to put inspection without creating asymmetric routing.

### Skill 1.4.1: Multi-VPC architectures with hybrid and multi-account connectivity

**Key concepts**

- **VPC peering:** one-to-one, non-transitive, no overlapping CIDRs, works across accounts and Regions. No hourly charge; you pay data transfer (same-AZ peering traffic is free, cross-AZ and cross-Region traffic is charged). Lowest latency and cost for a few high-volume pairs. No edge-to-edge routing: a peer cannot use your VPN, DX, internet gateway or NAT gateway.
- **Transit Gateway (TGW):** Regional hub with route tables, transitive routing, VPN/DX/peering/Connect attachments. Charged per attachment-hour and per GB processed. Share a TGW to other accounts with AWS RAM; accept attachments automatically within the organization. Use appliance mode on the inspection VPC attachment so stateful firewalls see both directions of a flow.
- **AWS Cloud WAN:** a global network defined by a JSON core network policy with segments, attachment policies (for example tag-based segment assignment) and Regions as core network edges. Use it when many Regions and sites need consistent segmentation without stitching TGW peering by hand.
- **AWS PrivateLink:** exposes one service (behind an NLB, or GWLB for appliances) to consumers through interface endpoints in their VPCs. Unidirectional (consumer initiates), works with overlapping CIDRs, scales to thousands of consumers, and the provider controls allowed principals and can require acceptance. Interface endpoints bill per hour per AZ plus per GB. Cross-Region PrivateLink to endpoint services is available in a set of Regions, so a consumer can reach a provider in another Region without its own peering.
- **Amazon VPC Lattice:** application-layer service network for HTTP, HTTPS, gRPC and TCP services across VPCs and accounts, with IAM auth policies, no IP overlap concerns and no route tables to manage. Shared through RAM. Use it for service-to-service connectivity across many teams; use PrivateLink for a single published service; use TGW for full network-layer reachability.
- **Shared VPCs (VPC sharing via RAM):** a central account owns the VPC; participant accounts launch resources into shared subnets. Fewer VPCs and TGW attachments, so lower cost and simpler IP planning, but participants cannot change routing or NACLs.
- **Centralised egress:** NAT gateways in an egress VPC behind TGW cut the number of NAT gateways, but TGW data processing is charged on every GB, so check cost when egress volume is large. Centralised interface endpoints in a shared-services VPC (with private hosted zones shared to spokes) cut endpoint-hour charges.
- **Overlapping CIDRs:** PrivateLink or VPC Lattice avoid the problem; otherwise use a private NAT gateway to translate to a routable range.

**Exam traps**

- Expecting transitive routing through a peered VPC to reach on-premises or a third VPC. Peering is non-transitive and does not allow edge-to-edge routing.
- Using TGW to connect two VPCs that exchange tens of TB per month when only those two need to talk. Peering avoids TGW per-GB processing.
- Picking TGW when the two VPCs have overlapping CIDRs. TGW cannot route between overlapping ranges; use PrivateLink, VPC Lattice or private NAT.
- Using PrivateLink when the requirement is bidirectional network access to many ports and hosts. PrivateLink is consumer-initiated to one service.
- Forgetting appliance mode on the inspection VPC attachment, which causes dropped return traffic when flows cross AZs.

**Hands-on**

1. **Peering versus PrivateLink with overlapping CIDRs.** Create two VPCs that both use `10.0.0.0/16`. Try to create a peering connection and note the error. Then put a small web service behind an internal NLB in one VPC, create an endpoint service, and create an interface endpoint in the other VPC; curl the endpoint DNS name from an instance there. *Cost/cleanup:* NLB and interface endpoint bill hourly (a few cents each); delete the endpoint, endpoint service, NLB and instances.
2. **Transit Gateway segmentation.** Create three small VPCs (prod, dev, shared) and attach them to a TGW with two route tables: prod and dev both propagate to and associate with routes that allow reaching shared, but not each other. Test with ping between instances and inspect the TGW route tables. Enable TGW Flow Logs to see the dropped flows. *Cost/cleanup:* each TGW attachment bills hourly (about a dollar or two for an afternoon with three attachments); delete attachments and the TGW the same day.

### Skill 1.4.2: VPC design with subnets, IPAM and segmentation

**Key concepts**

- **Amazon VPC IP Address Manager (IPAM):** organisation-wide planning, allocation and monitoring of IPv4 and IPv6 space. Create a hierarchy of pools (top-level, Regional, per-environment or per-business unit), share pools with accounts through RAM, and enforce allocation rules (allowed netmask lengths, required tags). VPCs created from an IPAM pool get non-overlapping CIDRs automatically. IPAM also discovers existing CIDRs, flags overlaps and non-compliance, and tracks public IP usage. It can manage BYOIP ranges and Amazon-provided contiguous IPv6 blocks.
- Integrate IPAM with Organizations by delegating an IPAM admin account; combine with SCPs that require `ec2:CreateVpc` to use an IPAM pool.
- **Subnet tiers:** public (route to internet gateway: ALBs, NAT gateways), private application (route to NAT or egress-only internet gateway), private data (no internet route), plus TGW attachment subnets (small, dedicated, so NACLs and route tables can differ). Spread every tier across at least two, ideally three, AZs.
- Plan for growth: a VPC can have additional secondary IPv4 CIDRs; EKS clusters burn IPs quickly (one per pod with the VPC CNI), so use prefix delegation or a secondary CIDR from the `100.64.0.0/10` range for pods.
- **Segmentation tools:** separate VPCs or accounts (strongest), TGW route tables, subnet route tables, NACLs (stateless, subnet level, ordered rules, allow and deny), security groups (stateful, ENI level, allow only, can reference other security groups), and prefix lists to manage CIDR sets in one place.
- **VPC Block Public Access** is an account-level, Regional control that blocks traffic through internet gateways and egress-only internet gateways, with exclusions for specific VPCs or subnets that must stay public. Use it as a guardrail against accidental public exposure.
- Reserved addresses: AWS reserves five IPs in each subnet (first four and last), which matters for tiny TGW or endpoint subnets.

**Exam traps**

- Using a spreadsheet and AWS Config rules to catch overlapping CIDRs after the fact. IPAM pools with allocation rules prevent the overlap at creation.
- Putting databases in a public subnet with a restrictive security group "because it is cheaper". Data tiers belong in subnets without an internet route.
- Using NACLs to block a single compromised IP inside a security group-referenced design and forgetting that NACLs are stateless; both inbound and ephemeral-port outbound rules are needed.
- Sizing EKS VPCs with a `/24` per AZ and then running out of pod IPs.

**Hands-on**

1. **IPAM pool hierarchy.** In a single account (free tier for IPAM covers basic features; advanced tier is charged per active IP), create an IPAM with a top-level pool `10.0.0.0/8`, a Regional pool `10.10.0.0/14`, and a dev pool `10.10.0.0/16` with an allocation rule of `/24` to `/20` and a required tag `env=dev`. Create a VPC by picking the dev pool and a `/22` netmask, and try a `/16` to see the rule reject it. View the resource discovery for existing VPCs. *Cost/cleanup:* use the free tier; delete the VPC, release the allocation, and delete the pools and IPAM.
2. **Three-tier subnets with NACL versus security group behaviour.** Build a VPC with public, app and data subnets in two AZs. Put an instance in app and one in data; allow MySQL from the app security group to the data security group. Then add a NACL on the data subnet that allows inbound 3306 from the app CIDR but no outbound ephemeral ports, and watch connections hang until you add the outbound rule. *Cost/cleanup:* use free-tier instances and no NAT gateway (use Session Manager through VPC endpoints, or EC2 Instance Connect Endpoint); terminate instances and delete the VPC.

### Skill 1.4.3: Network security: inspection, filtering and application layer protection

**Key concepts**

- **Layers:** security groups and NACLs for L3/L4 allow lists; AWS Network Firewall for stateful L3 to L7 inspection in VPC routing paths (Suricata-compatible rules, domain allow and deny lists for HTTP and TLS SNI, IPS signatures, TLS inspection); Gateway Load Balancer (GWLB) for third-party firewall appliances; AWS WAF for HTTP(S) at CloudFront, ALB, API Gateway REST APIs, AppSync, Cognito user pools and Verified Access; AWS Shield Standard (automatic) and Shield Advanced for DDoS.
- **Network Firewall deployment models:** distributed (firewall endpoints in each VPC), centralised inspection VPC behind TGW (east-west and egress), or combined. Firewall endpoints are per AZ; route each AZ's traffic to its local endpoint. Billed per endpoint-hour plus per GB processed, which adds up across many AZs.
- **GWLB:** distributes traffic to a fleet of appliances using GENEVE encapsulation, keeps flow symmetry, and is reached through GWLB endpoints (a PrivateLink type) in consumer VPCs. Choose it when the security team mandates a specific vendor firewall.
- **Ingress inspection patterns:** ALB in a public subnet with WAF for HTTP; for non-HTTP or mandated deep inspection, route internet gateway ingress (edge association with an IGW route table) through a firewall or GWLB endpoint before the load balancer.
- **AWS WAF:** managed rule groups (Core rule set, known bad inputs, SQL injection, IP reputation, Bot Control, Account Takeover Prevention, Account Creation Fraud Prevention), rate-based rules, geo match, custom responses, CAPTCHA and challenge actions, and labels. Centrally deploy WAF and Network Firewall policies across accounts with AWS Firewall Manager (requires Organizations and AWS Config).
- **VPC Flow Logs:** capture accepted and rejected IP traffic metadata at VPC, subnet or ENI level to CloudWatch Logs, S3 (Parquet option, partitioned) or Firehose. They do not capture payload, DNS queries to the Amazon resolver, or traffic to the instance metadata service. For payload, use Traffic Mirroring. For DNS, use Resolver query logs.
- **Egress filtering:** Network Firewall with domain allow lists, or Route 53 Resolver DNS Firewall for DNS-layer blocking. Security groups cannot filter by domain name.

**Exam traps**

- Using security groups to block a specific malicious IP range. Security groups have no deny rules; use NACLs, Network Firewall or WAF IP sets.
- Placing AWS WAF on a Network Load Balancer. WAF does not attach to NLBs; use Network Firewall, Shield Advanced or move HTTP to an ALB.
- Using VPC Flow Logs to detect SQL injection. Flow Logs have no payload; WAF logs or Traffic Mirroring do.
- Deploying Network Firewall in a centralised inspection VPC without TGW appliance mode, causing asymmetric routing drops.
- Managing WAF web ACLs separately in 50 accounts when Firewall Manager can apply and auto-remediate them.

**Hands-on**

1. **WAF on an ALB.** Deploy a small ALB with a Lambda target (free tier) or a tiny instance, attach a web ACL with the Core rule set, SQL injection rule group and a rate-based rule of a low threshold. Send a request with `?id=1' OR '1'='1` and a burst of requests from CloudShell, and inspect sampled requests and WAF logs to see which rule fired. *Cost/cleanup:* web ACL and rules bill monthly (pro-rated small) plus per million requests, the ALB bills hourly; delete the ALB, web ACL and log group the same day.
2. **Egress domain filtering with Network Firewall.** In one VPC, create a firewall subnet per AZ and a Network Firewall with a stateful domain list rule that allows only `.amazonaws.com` and `.github.com` over HTTPS (SNI). Route the private subnet through the firewall endpoint to a NAT gateway, then curl an allowed and a blocked domain. Read the alert logs in CloudWatch. *Cost/cleanup:* Network Firewall endpoints and NAT gateways both bill hourly and are not free tier (several dollars for a short test); delete the firewall, NAT gateway and Elastic IP within a couple of hours.

### Skill 1.4.4: IPv6 adoption and dual-stack migration

**Key concepts**

- **Why IPv6:** exhaustion of private IPv4 space in large estates (especially EKS), and public IPv4 addresses are now charged per hour. IPv6 addresses in AWS are globally unique; "public" or "private" behaviour comes from routing.
- **VPC options:** dual-stack VPCs and subnets (IPv4 plus an Amazon-provided `/56` or BYOIPv6 or IPAM-allocated block; subnets are `/64`), or IPv6-only subnets (no IPv4 CIDR). Instances in IPv6-only subnets must be Nitro-based.
- **Egress-only internet gateway:** stateful outbound-only for IPv6, the IPv6 equivalent of NAT for "private" subnets. Routing `::/0` to an internet gateway instead makes instances reachable from the internet (subject to security groups).
- **NAT64 and DNS64:** let IPv6-only workloads reach IPv4-only services. Enable DNS64 on the subnet so the Route 53 Resolver synthesises `64:ff9b::/96` addresses for names with only A records. Add a route `64:ff9b::/96` to a NAT gateway; NAT64 is automatically available on NAT gateways and translates to IPv4. The NAT gateway itself sits in a public subnet with an IPv4 route to the internet gateway (or forwards to TGW/VGW for on-premises IPv4). Security groups and NACLs must allow `64:ff9b::/96`.
- **Migration path:** add IPv6 to VPC and subnets, update route tables (`::/0` to egress-only IGW), update security groups and NACLs for IPv6 (they are separate rules), enable dual-stack on ALBs and NLBs, CloudFront and Global Accelerator, and on Route 53 add AAAA records. Move to IPv6-only subnets for new workloads (EKS IPv6 clusters, Lambda, ECS) once dependencies support it.
- **EKS IPv6 clusters:** pods get IPv6 addresses from the subnet's `/64`, removing pod IP exhaustion; the cluster IP family is set at creation and cannot be changed. Egress to IPv4 uses NAT64 or the node's IPv4 via the CNI's egress feature.
- **Service support** varies; check each service's IPv6 and dual-stack endpoint support before planning IPv6-only for a workload. Many AWS APIs offer dual-stack endpoints.
- ALB in dual-stack mode can front IPv4-only targets, so clients can move to IPv6 before backends.

**Exam traps**

- Routing IPv6 `::/0` from private subnets to a NAT gateway. NAT gateways do not do IPv6-to-IPv6 NAT; use an egress-only internet gateway for IPv6 egress, and NAT gateway only for NAT64 traffic to `64:ff9b::/96`.
- Enabling NAT64 but forgetting DNS64 on the subnet. Without synthesised AAAA answers, IPv6-only clients receive A records they cannot use.
- Assuming existing IPv4 security group rules cover IPv6 traffic. IPv6 rules are separate (`::/0` versus `0.0.0.0/0`).
- Trying to convert an existing IPv4 EKS cluster to IPv6 in place. The IP family is fixed at creation; build a new cluster and migrate.

**Hands-on**

1. **IPv6-only subnet reaching an IPv4-only site.** Create a dual-stack VPC with an Amazon-provided IPv6 block, a public dual-stack subnet with a NAT gateway, and an IPv6-only private subnet with DNS64 enabled. Add routes `::/0` to an egress-only internet gateway and `64:ff9b::/96` to the NAT gateway. Launch a Nitro instance in the IPv6-only subnet (connect through EC2 Instance Connect Endpoint or Session Manager), run `dig AAAA ipv4only.arpa` or another IPv4-only name and observe the synthesised `64:ff9b::` address, then curl it. *Cost/cleanup:* the NAT gateway bills hourly and per GB (not free tier); delete it and release the Elastic IP within an hour.
2. **Dual-stack ALB with IPv4 targets.** Create an ALB with IP address type dual-stack in dual-stack public subnets and an IPv4-only target. Add a Route 53 AAAA alias record and test with `curl -6` from an IPv6-capable network (or CloudShell if IPv6 is available to you) and `curl -4`. Check the ALB access logs to see client IPv6 addresses. *Cost/cleanup:* ALB hourly charge and a hosted zone; delete both and the target.

### Skill 1.4.5: Global traffic routing for multi-Region applications

**Key concepts**

- **Route 53 routing policies:** simple, weighted (canary or blue/green between Regions), latency-based (lowest latency Region for the resolver), failover (active-passive with health checks), geolocation (by country or continent, for compliance or localisation; needs a default record), geoproximity (by distance with bias, through traffic flow), IP-based (by client CIDR, for ISP-specific routing), and multivalue answer (up to eight healthy records, client-side load spreading).
- Health checks: endpoint (HTTP/HTTPS/TCP), calculated (combine child checks), and CloudWatch alarm-based (for private resources that health checkers cannot reach). Alias records to AWS resources can evaluate target health.
- DNS caveats: failover speed depends on TTL and client resolver caching; some clients ignore TTL. For exam purposes, DNS failover is minutes, not seconds.
- **AWS Global Accelerator:** two static anycast IPv4 addresses (four with dual-stack), traffic enters the AWS network at the nearest edge location and rides the backbone to the endpoint. Standard accelerators route to the nearest healthy endpoint group by location and health, with **traffic dials** per endpoint group (percentage of traffic to a Region) and **endpoint weights** within a group. Endpoints: ALB (internet-facing or internal), NLB, EC2 instances, Elastic IPs. Custom routing accelerators map ports deterministically to specific EC2 instances in subnets (for game sessions or VoIP); they do not support dual-stack.
- Global Accelerator fails over without waiting for DNS caches because client IPs stay the same; good for TCP/UDP, non-HTTP protocols, IP allow-listing by partners, and fast Regional failover. CloudFront is for HTTP caching and edge compute; Global Accelerator does not cache.
- **Combine with recovery controls:** Amazon Application Recovery Controller (ARC) routing controls give you manually flipped, highly available on/off switches backed by Route 53 health checks, for operator-driven Regional failover (covered in Task 4.2). Zonal shift and zonal autoshift move traffic away from an impaired AZ for supported load balancers.
- **Decision rules:** HTTP content with caching, use CloudFront (with origin groups for failover). Static IPs, non-HTTP, fastest failover, use Global Accelerator. Regulatory routing by country, use Route 53 geolocation. Lowest latency across Regions at lowest cost, use Route 53 latency routing.

**Exam traps**

- Using Route 53 latency routing when partners must allow-list fixed IP addresses. DNS answers change; Global Accelerator gives static anycast IPs.
- Expecting Route 53 failover to switch clients within seconds. Resolver caching and TTLs delay it; Global Accelerator health-based failover is faster.
- Using geolocation without a default record, so users from unmapped locations get no answer.
- Choosing Global Accelerator to reduce origin load for static content. It does not cache; CloudFront does.
- Using geoproximity or latency to keep EU users' data in the EU. Only geolocation guarantees country or continent mapping, and data residency also needs data-layer controls.

**Hands-on**

1. **Latency and failover routing.** Deploy a Lambda function URL or small API Gateway HTTP API in two Regions returning the Region name. In a hosted zone you own (or a cheap test domain), create latency-based records for both, each with a health check. Query from CloudShell in different Regions and from your laptop, then break one endpoint (return 500) and watch the health check fail and answers change. Note how long it takes with a 60-second TTL. *Cost/cleanup:* health checks bill monthly per check (pro-rated small), APIs within free tier; delete records, health checks and APIs.
2. **Global Accelerator traffic dial.** Create a standard accelerator with a TCP listener on port 80 and endpoint groups in two Regions, each pointing to a small ALB or EC2 instance. Hit the static IPs in a loop, set the traffic dial of your nearest Region to 0%, and watch responses come from the other Region without any DNS change. *Cost/cleanup:* Global Accelerator has a fixed hourly fee per accelerator plus premium data transfer, and ALBs bill hourly; delete the accelerator (it must be disabled first) and the load balancers within the hour.

### Practice questions: Task 1.4

**Question 1.4.1** A software company publishes a REST-based pricing service from a provider VPC to 400 customer VPCs owned by external customers. Many customer VPCs use `10.0.0.0/16`, which overlaps with the provider VPC. Customers must not gain network access to anything except the pricing service, and the provider must approve each new customer connection. Which solution meets these requirements with the least operational overhead?

- A. Create a Transit Gateway in the provider account, share it with each customer through AWS RAM, and use a separate Transit Gateway route table per customer.
- B. Create VPC peering connections from the provider VPC to each customer VPC and use security groups to allow only the pricing service port.
- C. Place the pricing service behind a Network Load Balancer, create a VPC endpoint service that requires acceptance, and add each customer's AWS account to the allowed principals.
- D. Deploy a private NAT gateway in the provider VPC to translate customer traffic into a non-overlapping range, and connect each customer through AWS Site-to-Site VPN.

<details>
<summary>Answer and explanation</summary>

**Correct answer: C**

**Why C is correct:** PrivateLink exposes one service through interface endpoints in each consumer VPC, so overlapping CIDRs do not matter and consumers cannot reach anything else. Acceptance-required and allowed principals let the provider approve each connection. It scales to hundreds of consumers without route tables or tunnels to manage.

**Why the other options are incorrect:**

- **A:** Transit Gateway cannot route between overlapping CIDRs, and network-layer attachments give broader reach than one service. Hundreds of route tables are also heavy to operate.
- **B:** VPC peering is not allowed between VPCs with overlapping CIDRs, and per-VPC peering for 400 customers runs into peering quotas and management effort.
- **D:** Private NAT plus VPN can solve the overlap, but running 400 VPN connections and NAT translation is far more work and cost than PrivateLink.

**Key takeaway:** One service, many consumers, overlapping CIDRs, consumer-initiated only: PrivateLink endpoint service.

</details>

**Question 1.4.2** A bank has 300 accounts and keeps finding overlapping VPC CIDRs that break Transit Gateway routing. The network team wants new VPCs in each business unit to get non-overlapping CIDRs automatically, with allowed sizes from `/24` to `/20`, and wants to stop teams from creating VPCs with hand-picked CIDRs. Existing overlaps must be reported. Which combination of actions meets these requirements? (Select TWO.)

- A. Delegate an Amazon VPC IPAM administrator account in AWS Organizations, create a pool hierarchy per Region and business unit with allocation rules for netmask length, share the pools with the matching OUs through AWS RAM, and attach an SCP that denies `ec2:CreateVpc` unless the `ec2:Ipv4IpamPoolId` condition key is present.
- B. Use IPAM resource discovery and the IPAM dashboard to report existing overlapping and non-compliant CIDRs.
- C. Create an AWS Config custom rule that marks VPCs with overlapping CIDRs as non-compliant and triggers an SSM Automation runbook to delete them.
- D. Publish a DynamoDB table of approved CIDR blocks and require teams to reserve a CIDR with a Lambda function before calling `CreateVpc`.
- E. Create a prefix list per business unit containing allowed CIDRs and share it through AWS RAM.

<details>
<summary>Answer and explanation</summary>

**Correct answer: A, B**

**Why A and B are correct:** IPAM pools allocate non-overlapping CIDRs from a hierarchy and enforce netmask length rules at creation, and sharing pools to OUs through RAM puts the allocation in each team's console and API calls. The SCP that denies `ec2:CreateVpc` without the `ec2:Ipv4IpamPoolId` condition key forces every new VPC to draw from a pool, which stops hand-picked CIDRs. IPAM resource discovery monitors existing VPCs across the organization and flags overlaps and non-compliance, meeting the reporting requirement.

**Why the other options are incorrect:**

- **C:** Config detects problems after creation, and deleting live VPCs automatically is destructive. It does not allocate CIDRs.
- **D:** A home-built registry works in principle but is custom code to maintain and does not stop someone from calling `CreateVpc` directly.
- **E:** Prefix lists group CIDRs for route tables and security groups; they do not allocate or enforce VPC CIDRs.

**Key takeaway:** IPAM pools with allocation rules, shared through RAM, prevent overlaps; IPAM discovery reports the old ones.

</details>

**Question 1.4.3** A company inspects all traffic between 60 spoke VPCs and to the internet with AWS Network Firewall in a central inspection VPC attached to a Transit Gateway in three Availability Zones. After go-live, some TCP sessions between spokes in different Availability Zones hang, while sessions within one Availability Zone work. Firewall logs show only one direction of the failing flows. Which change resolves the issue?

- A. Replace AWS Network Firewall with a Gateway Load Balancer and third-party appliances, which keep flow symmetry through GENEVE.
- B. Add a NACL rule in each spoke VPC that allows ephemeral return ports from all other spoke CIDRs.
- C. Deploy AWS Network Firewall endpoints in each spoke VPC instead of the inspection VPC.
- D. Enable appliance mode on the Transit Gateway attachment for the inspection VPC.

<details>
<summary>Answer and explanation</summary>

**Correct answer: D**

**Why D is correct:** Without appliance mode, the Transit Gateway keeps traffic in the source AZ, so request and response of a cross-AZ flow can land on firewall endpoints in different AZs. A stateful firewall that sees only one direction drops the session. Appliance mode makes the TGW pick one AZ per flow for the inspection attachment, so both directions go through the same endpoint.

**Why the other options are incorrect:**

- **A:** GWLB keeps symmetry within its own target group, but the TGW would still send the two directions to different AZ endpoints without appliance mode. It is a costly redesign that does not address the cause.
- **B:** The symptom matches asymmetric routing at the stateful firewall, not a NACL block; and the flows within one AZ already work with the same NACLs.
- **C:** Distributed firewalls would avoid the issue for some flows but multiply endpoint-hour charges by 60 VPCs and change the central inspection design the company chose.

**Key takeaway:** Centralised stateful inspection behind a TGW needs appliance mode on the inspection VPC attachment.

</details>

**Question 1.4.4** A fintech runs Amazon EKS on IPv4 and is running out of private IPv4 space for pods. It plans new IPv6 clusters in IPv6-only subnets. Pods must still call a payment partner that publishes only an IPv4 address and an A record, and pods must not be reachable from the internet. The team wants to avoid running any proxies. Which solution meets these requirements?

- A. Route `::/0` from the IPv6-only subnets to an internet gateway, and configure the partner's IPv4 address in a Kubernetes ExternalName service.
- B. Enable DNS64 on the IPv6-only subnets, route `64:ff9b::/96` to a NAT gateway in a public subnet, and route `::/0` to an egress-only internet gateway.
- C. Route `::/0` from the IPv6-only subnets to a NAT gateway, and enable DNS64 on the NAT gateway's subnet.
- D. Deploy an NLB with dual-stack listeners in a public subnet that forwards to the partner's IPv4 address, and point pods at the NLB's AAAA record.

<details>
<summary>Answer and explanation</summary>

**Correct answer: B**

**Why B is correct:** DNS64 on the pod subnets makes the Resolver return a synthesised `64:ff9b::/96` address for the partner's A-only name. The route for that prefix sends traffic to a NAT gateway, where NAT64 translates it to IPv4 automatically. The egress-only internet gateway lets pods reach IPv6 destinations without being reachable from the internet. No proxies are involved.

**Why the other options are incorrect:**

- **A:** An internet gateway route makes pods reachable over IPv6 from the internet (subject to security groups), and an IPv6-only pod still cannot reach an IPv4 address without translation.
- **C:** NAT gateways do not do IPv6-to-IPv6 NAT, so the `::/0` route is wrong. DNS64 must be enabled on the subnet where the clients live, not on the NAT gateway's subnet.
- **D:** An NLB cannot target an external internet IP address outside the VPC or connected networks, and this is a proxy in all but name.

**Key takeaway:** IPv6-only to IPv4-only: DNS64 on the client subnet, `64:ff9b::/96` to a NAT gateway, `::/0` to an egress-only internet gateway.

</details>

**Question 1.4.5** A trading platform serves institutional clients over a proprietary TCP protocol from us-east-1 and eu-west-1. Clients' firewalls allow outbound connections only to a fixed list of IP addresses that changes at most once a year. Each client must reach the closest healthy Region, and failover must not depend on client DNS caching. Operations wants to move 10% of traffic to a new Region build for testing. Which solution meets these requirements?

- A. Create Route 53 latency-based records with health checks for Network Load Balancers in both Regions, set a 10-second TTL, and give clients the NLBs' Elastic IP addresses.
- B. Put Amazon CloudFront in front of Network Load Balancers in both Regions with an origin group for failover, and give clients the CloudFront IP ranges.
- C. Create a Global Accelerator custom routing accelerator that maps client ports to EC2 instances in both Regions, and use endpoint weights for testing.
- D. Create a Global Accelerator standard accelerator with a TCP listener and an endpoint group per Region pointing to Network Load Balancers, and use the endpoint group traffic dial to shift a percentage of traffic.

<details>
<summary>Answer and explanation</summary>

**Correct answer: D**

**Why D is correct:** A standard accelerator gives two static anycast IPv4 addresses that clients can allow-list permanently. It routes each client to the nearest healthy endpoint group and fails over on health without DNS changes, so client DNS caching does not matter. Traffic dials on endpoint groups control the percentage of traffic sent to a Region, which supports the 10% test.

**Why the other options are incorrect:**

- **A:** Allow-listing per-Region NLB IPs means a client connected to one Region's IP does not move to the other on failure unless DNS is re-resolved, and short TTLs are often ignored by clients.
- **B:** CloudFront serves HTTP and HTTPS (and WebSocket), not a proprietary TCP protocol, and its IP ranges are large and change, so they are unsuitable for fixed allow-lists.
- **C:** Custom routing accelerators map each accelerator port to a specific EC2 instance and port, so your application must decide which instance each client uses. They do not pick the closest healthy Region for the client, and they target subnets of EC2 instances, not NLBs, so this would mean redesigning the service.

**Key takeaway:** Static IPs, non-HTTP, fast health-based Regional failover and percentage shifting: Global Accelerator standard accelerator with traffic dials.

</details>

## Task 1.5: Design data, storage, and database architectures

This task tests whether you can match data services to access patterns: lake versus warehouse, stream versus batch, relational versus key-value, which replication model gives the RPO you need, which storage fits the I/O profile, and how to share governed data across accounts and with outside parties. The exam rarely asks "what is X"; it asks which of four workable designs best meets a consistency, latency, cost or governance constraint.

### Skill 1.5.1: Data lake and lakehouse architectures

**Key concepts**

- **Lake layers:** raw (landing, immutable), curated (cleaned, columnar Parquet or Iceberg, partitioned), and consumption (aggregates, marts). Separate buckets or prefixes per layer and per sensitivity level, each with its own KMS key where needed.
- **Open table formats:** Apache Iceberg adds ACID transactions, schema and partition evolution, time travel, row-level updates and deletes (useful for GDPR erasure and CDC merges) on S3. Athena, EMR, Glue, Redshift and Amazon Data Firehose (delivery to Iceberg tables) can read or write Iceberg. Plain Parquet tables cannot do row-level deletes without rewriting partitions yourself.
- **Amazon S3 Tables:** a table bucket type that stores tables in Iceberg format with automatic maintenance (compaction, snapshot management, unreferenced file removal), higher transaction rates than self-managed Iceberg on general purpose buckets, and IAM permissions in the `s3tables` namespace. Integrate table buckets with the AWS Glue Data Catalog so Athena, Redshift, EMR and Spark can query them, with Lake Formation for fine-grained access. Choose S3 Tables when you want Iceberg without running compaction jobs; choose self-managed Iceberg in general purpose buckets when you need full control over layout or engines that integrate only with plain S3.
- **AWS Lake Formation:** central permissions on Glue Data Catalog databases, tables, columns, rows and cells (data filters), tag-based access control (LF-TBAC) for scale, cross-account sharing, and credential vending to integrated engines (Athena, Redshift Spectrum, EMR, Glue). Register S3 locations with Lake Formation so engines get temporary scoped credentials instead of direct bucket access. Hybrid access mode lets you move gradually from IAM/S3 policies to Lake Formation permissions.
- **Redshift Spectrum** queries data in S3 through external schemas mapped to the Glue Data Catalog, joining it with local Redshift tables; you pay per TB scanned. Use it to keep hot data in Redshift and long history in the lake. Redshift can also query Iceberg tables.
- **Ingestion:** Amazon Data Firehose for streaming to S3 (Parquet conversion, dynamic partitioning, Iceberg delivery), AWS Glue jobs and Glue zero-ETL integrations for SaaS and databases, AWS DMS for database CDC, DataSync for files.
- **Query engines:** Athena (serverless SQL, pay per TB scanned or provisioned capacity), Redshift (warehouse, concurrency scaling, Redshift Serverless), EMR (Spark, Hive, Presto/Trino on EC2, EKS or serverless), Glue (serverless Spark ETL).
- **Performance basics:** columnar formats, compression, partitioning on the filters you use, avoiding many small files (compaction), and partition projection in Athena for high-cardinality partitions.

**Exam traps**

- Granting analysts direct S3 bucket policies per table in a lake governed by Lake Formation. Fine-grained column and row control needs Lake Formation permissions and registered locations.
- Choosing plain Parquet in S3 when the requirement includes row-level updates, deletes for "right to be forgotten", or time travel. Iceberg (or S3 Tables) handles this.
- Running scheduled Glue compaction jobs for Iceberg tables when the requirement stresses least operational overhead. S3 Tables maintains tables automatically.
- Loading all historical data into Redshift when most of it is rarely queried. Redshift Spectrum or Athena on S3 keeps cold history cheap.

**Hands-on**

1. **S3 Tables with Athena.** Create a table bucket, enable integration with AWS analytics services, create a namespace and an Iceberg table from Athena, insert a few hundred rows, update and delete some rows, and query a previous snapshot with time travel. Look at the table's maintenance configuration in the S3 console. *Cost/cleanup:* S3 Tables charges for storage, requests and maintenance; at this size it is cents. Athena charges per TB scanned (minimum per query). Delete the table, namespace and table bucket.
2. **Lake Formation column and row security.** Upload a CSV of fake customers to S3, crawl it with a Glue crawler, register the location with Lake Formation, and create a data filter that hides the `email` column and only returns `country = 'PH'` rows. Grant `SELECT` with the filter to a test IAM role and query from Athena as that role. Compare with querying as the data lake admin. *Cost/cleanup:* crawler and Athena cost pennies; delete the crawler, database, filter, registered location and bucket.

### Skill 1.5.2: Real-time and batch analytics pipelines

**Key concepts**

- **Streaming ingestion choices:** Kinesis Data Streams (shards or on-demand, ordering per partition key, retention from 24 hours up to 365 days, replay, enhanced fan-out for many consumers); Amazon MSK (Apache Kafka, when you need Kafka APIs, ecosystem or existing producers; MSK Serverless removes broker sizing; MSK Connect runs Kafka Connect connectors); Amazon Data Firehose (fully managed delivery to S3, Redshift, OpenSearch, Splunk, Iceberg, HTTP, with buffering of seconds to minutes; not for custom consumers or replay).
- **Stream processing:** Amazon Managed Service for Apache Flink for stateful, windowed, exactly-once processing over Kinesis or MSK; Lambda for simple per-record logic; Spark Structured Streaming on EMR or Glue streaming.
- **Batch:** Amazon EMR for large Spark/Hive/Trino workloads where you need control over versions, instance types, Spot and custom libraries (EMR on EC2), Kubernetes consolidation (EMR on EKS), or no cluster management (EMR Serverless). AWS Glue for serverless Spark ETL with Data Catalog integration. Use Spot for task nodes and transient clusters to cut EMR cost.
- **Amazon Redshift:** RA3 node types separate compute and Redshift Managed Storage; Redshift Serverless for variable workloads; concurrency scaling for burst reads; data sharing across clusters and accounts without copying; streaming ingestion from Kinesis Data Streams and MSK into materialized views for near-real-time dashboards; zero-ETL integrations from Aurora, RDS and DynamoDB.
- **Amazon AppFlow:** managed, scheduled or event-triggered transfer from SaaS (Salesforce, SAP OData, ServiceNow, Zendesk, Google Analytics and others) to S3, Redshift and other targets, with mapping, filtering, validation and PrivateLink support for some sources. Choose it over custom Lambda pollers for SaaS ingestion.
- **Orchestration:** Step Functions or Amazon MWAA (managed Apache Airflow) for multi-stage pipelines; Glue workflows for Glue-only pipelines; EventBridge Scheduler for simple timed jobs.
- **Lambda architecture versus Kappa:** keep both a speed layer and batch layer only when needed; often a stream into Iceberg plus scheduled compaction covers both.

**Exam traps**

- Firehose when the scenario needs replay of the last 3 days or multiple independent consumers. Firehose has no replay; use Kinesis Data Streams or MSK.
- Self-managed Kafka on EC2 when the requirement is Kafka compatibility with least operational overhead. MSK (or MSK Serverless) is managed.
- Redshift provisioned cluster sized for monthly peak when usage is a few hours a day. Redshift Serverless or pause/resume fits better.
- Writing a custom Lambda to call the Salesforce API every hour. AppFlow does this as a managed flow.
- EMR on EC2 with On-Demand core nodes only for fault-tolerant batch. Spot task nodes and instance fleets reduce cost.

**Hands-on**

1. **Streaming into Redshift Serverless.** Create a Kinesis data stream (on-demand), a small Redshift Serverless workgroup with the lowest base capacity, and a streaming ingestion materialized view over the stream. Push JSON events from a Python script in CloudShell and refresh the view to see rows arrive within seconds. *Cost/cleanup:* Redshift Serverless bills RPU-hours while queries run and the stream bills per hour; delete the workgroup, namespace and stream right after. Use any free trial credit for Redshift Serverless if offered.
2. **EMR Serverless batch job.** Create an EMR Serverless Spark application, upload a sample PySpark script that reads a public dataset from S3 (for example NYC taxi trips from the Registry of Open Data), aggregates by day, and writes Parquet to your bucket. Run it, read the job metrics, then query the output from Athena. *Cost/cleanup:* pay per vCPU and GB-hour for the job (cents to a dollar); set the application to auto-stop, then delete it and the output.

### Skill 1.5.3: Selecting database services by workload

**Key concepts**

| Need | Service | Why |
|---|---|---|
| Relational, standard engines, moderate scale | Amazon RDS (PostgreSQL, MySQL, MariaDB, Oracle, SQL Server, Db2) | Managed engines, Multi-AZ, read replicas |
| Relational, high throughput, fast failover, storage auto-grows | Amazon Aurora (MySQL or PostgreSQL compatible) | Six-way replicated storage across three AZs, up to 15 low-lag replicas, Global Database, Serverless v2 |
| Distributed SQL, active-active multi-Region, serverless | Amazon Aurora DSQL | PostgreSQL-compatible, strong consistency across Regions, no instances to manage; check feature compatibility (not every PostgreSQL feature is supported) |
| Key-value or document, single-digit ms at any scale | Amazon DynamoDB | Partition key design, on-demand or provisioned, global tables, streams |
| MongoDB API | Amazon DocumentDB | MongoDB-compatible document store |
| Graph traversals | Amazon Neptune | Gremlin, openCypher, SPARQL |
| Time series | Amazon Timestream | Time-series storage and queries |
| Wide column, Cassandra CQL | Amazon Keyspaces | Serverless Cassandra-compatible |
| Analytics over large data | Amazon Redshift | Columnar MPP, not for OLTP |
| In-memory primary database with durability | Amazon MemoryDB | Redis/Valkey-compatible with multi-AZ transaction log |
| Full-text and log analytics | Amazon OpenSearch Service | Search, log analytics, vector search |

- Decision questions: Do you know your access patterns (DynamoDB rewards known patterns; relational handles ad-hoc queries)? Need joins and multi-row transactions across entities (relational)? Need strong consistency across Regions with zero RPO (DynamoDB global tables in multi-Region strong consistency mode, or Aurora DSQL)? Need to keep a commercial engine for licensing or compatibility (RDS for Oracle or SQL Server)?
- DynamoDB essentials: item size up to 400 KB; strongly consistent reads within a Region; transactions across up to 100 items; GSIs are eventually consistent; design partition keys for even distribution.
- Aurora versus RDS: Aurora usually fails over faster (well under a minute is typical) and replicas share storage; RDS Multi-AZ instance deployment has a standby that does not serve reads; RDS Multi-AZ DB cluster deployment (MySQL and PostgreSQL) has two readable standbys.
- Commercial engines with features RDS does not support may need EC2 self-managed; RDS Custom for Oracle is out of scope for C03.

**Exam traps**

- DynamoDB for a reporting workload with unpredictable ad-hoc joins. Relational or Redshift fits; DynamoDB scans are slow and costly.
- Redshift as the backend for an OLTP order system. It is built for analytics, not high-rate single-row writes.
- Aurora Global Database when the requirement is active-active writes in two Regions with strong consistency. Global Database has one writer Region (write forwarding exists but is not multi-writer); Aurora DSQL or DynamoDB MRSC fit.
- ElastiCache as a system of record. Use MemoryDB when an in-memory store must be durable.

**Hands-on**

1. **DynamoDB access-pattern design.** Model an orders application with the access patterns "get order by ID", "list a customer's orders by date" and "list open orders by warehouse". Create one table with a composite key (`PK=CUSTOMER#id`, `SK=ORDER#date`) and a GSI for warehouse status, load 1,000 items with a script, and run each query. Try a `Scan` with a filter for the third pattern first and compare consumed capacity with the GSI query. *Cost/cleanup:* on-demand mode at this size is free tier or cents; delete the table.
2. **Aurora DSQL or Aurora Serverless v2 tryout.** Create an Aurora DSQL single-Region cluster (or an Aurora PostgreSQL Serverless v2 cluster with the minimum capacity), connect with `psql` using an IAM auth token, create a table and run a few transactions. If using DSQL, read the "unsupported PostgreSQL features" page and note what your applications would need to change. *Cost/cleanup:* DSQL bills per usage and storage; Serverless v2 bills ACU-hours even when idle unless it scales to zero, so delete the cluster within the hour.

### Skill 1.5.4: Database scaling and high availability

**Key concepts**

- **RDS Multi-AZ (instance):** synchronous standby in another AZ, automatic failover in one to two minutes, standby not readable. **RDS Multi-AZ DB cluster:** one writer and two readable standbys in three AZs, faster failover, MySQL and PostgreSQL only.
- **Read replicas:** asynchronous, in-Region or cross-Region, for read scaling and DR. A cross-Region replica can be promoted manually; RPO equals replica lag.
- **Aurora:** storage replicated six ways across three AZs, up to 15 Aurora Replicas sharing storage, automatic failover to a replica (set promotion tiers), reader endpoint for load-balanced reads, custom endpoints for groups of replicas. Aurora Auto Scaling adds replicas based on CPU or connections. Aurora Serverless v2 scales capacity in fine increments for spiky load.
- **Aurora Global Database:** storage-level replication to up to several secondary Regions with typically sub-second lag. Switchover (planned, no data loss) and failover (unplanned, RPO equals lag at the time) promote a secondary in about a minute. Write forwarding lets readers in secondary Regions send writes to the primary. Use it for DR with low RPO and RTO and for low-latency global reads.
- **RDS Proxy:** pools connections, reduces failover time for applications by holding connections, and works with IAM auth and Secrets Manager. Essential for Lambda with relational databases.
- **DynamoDB global tables:**
  - *Multi-Region eventual consistency (MREC):* default; asynchronous replication, usually within a second; last writer wins; any Regions; RPO of seconds.
  - *Multi-Region strong consistency (MRSC):* three Regions (three replicas, or two replicas plus a witness); writes replicate synchronously before acknowledging; strongly consistent reads return the latest version in any Region; RPO zero. Limitations: no transactions, no TTL, no LSIs, conversion only from an empty table, and the mode cannot be changed after creation. Higher write latency.
- **DynamoDB scaling:** on-demand mode for unknown traffic; provisioned with auto scaling for predictable traffic; adaptive capacity handles moderate hot keys but cannot fix a bad key design. Use write sharding for very hot keys.
- **ElastiCache and MemoryDB** in front of or instead of databases for read scale (see 1.5.6).

**Exam traps**

- Using an RDS Multi-AZ standby to offload reads. In the instance deployment the standby is not readable; use read replicas or the Multi-AZ DB cluster deployment.
- Choosing cross-Region read replicas for an RPO of about one second and RTO under a few minutes on Aurora. Aurora Global Database is built for that; Aurora MySQL cross-Region read replicas use binlog replication, with higher lag and a slower, manual promotion.
- Choosing DynamoDB global tables MREC when the requirement is "zero data loss if a Region fails" or "reads must always see the latest write in any Region". That needs MRSC.
- Choosing MRSC for a workload that relies on `TransactWriteItems` or TTL. MRSC does not support them.
- Scaling Lambda against Aurora without RDS Proxy and hitting connection limits.

**Hands-on**

1. **Aurora failover timing.** Create the smallest Aurora PostgreSQL provisioned cluster with one writer and one reader (or Serverless v2 with minimum capacity). Run a loop from CloudShell or an instance that writes a timestamp every second through the cluster endpoint, then trigger a failover from the console and measure how long writes fail. Repeat through RDS Proxy and compare. *Cost/cleanup:* Aurora instances and Proxy bill hourly (a few dollars for an hour); delete the proxy, cluster and snapshots right after.
2. **DynamoDB global tables both modes.** Create one global table in MREC mode across two Regions and write to the same item in both Regions within a second; observe last-writer-wins. Then create an empty table and convert it to MRSC in three supported Regions (or two replicas plus witness), write to an item from two Regions concurrently in a script, and observe `ReplicatedWriteConflictException` on one of them. Compare write latency from CloudWatch. *Cost/cleanup:* on-demand writes are cheap but replicated writes bill per Region; delete all replicas and tables the same day.

### Skill 1.5.5: Storage architectures by access pattern and performance

**Key concepts**

- **Amazon S3:** object storage, effectively unlimited, strong read-after-write consistency. Storage classes by access: Standard, Intelligent-Tiering (unknown patterns), Standard-IA and One Zone-IA (infrequent), Glacier Instant Retrieval (milliseconds, rare access), Glacier Flexible Retrieval and Deep Archive (hours). **S3 Express One Zone** gives single-digit millisecond latency in one AZ through directory buckets for hot ML training or analytics scratch data. Scale request rates by spreading across prefixes. Use multipart upload and byte-range fetches for large objects; Transfer Acceleration for long-distance uploads.
- **Amazon EBS:** block storage for a single EC2 instance in one AZ (io1/io2 support Multi-Attach for clustered apps in the same AZ). gp3 (baseline 3,000 IOPS and 125 MiB/s, set IOPS and throughput independently of size), io2 Block Express (highest IOPS, sub-millisecond latency, high durability) for critical databases, st1 (throughput HDD for big sequential), sc1 (cold HDD). Instance store is fastest but ephemeral.
- **Amazon EFS:** NFS for Linux, multi-AZ (or One Zone), elastic capacity, thousands of concurrent clients across AZs; throughput modes Elastic (default for spiky), Provisioned, Bursting; lifecycle to IA and Archive classes. Use for shared content, home directories, container persistent volumes.
- **Amazon FSx:**
  - *FSx for Windows File Server:* SMB, Active Directory integration, DFS namespaces, Multi-AZ.
  - *FSx for Lustre:* high-performance parallel file system for HPC and ML, links to S3 (lazy load and export), scratch or persistent deployment.
  - *FSx for NetApp ONTAP:* multi-protocol (NFS, SMB, iSCSI), snapshots, SnapMirror, FlexClone, data tiering; good lift-and-shift for NetApp estates and Windows plus Linux sharing.
  - *FSx for OpenZFS:* NFS with ZFS snapshots and clones, low latency.
- **Decision rules:** shared Linux files across many instances, EFS. Windows SMB with AD, FSx for Windows. HPC scratch over S3 data, FSx for Lustre. Existing NetApp features or multi-protocol, FSx for ONTAP. Single-instance database volume, EBS io2 or gp3. Object data lake or static content, S3. Mount S3 buckets as a file system for read-heavy apps, Mountpoint for Amazon S3 (it does not support all POSIX operations, such as in-place edits).

**Exam traps**

- EFS for a Windows application that needs SMB and NTFS ACLs. EFS is NFS only; use FSx for Windows File Server.
- EBS volume shared across AZs. EBS is AZ-scoped; Multi-Attach is same-AZ only and needs a cluster-aware file system.
- S3 Glacier Flexible Retrieval for data that must be readable in milliseconds a few times a year. Glacier Instant Retrieval meets that.
- gp2 for a small volume that needs high IOPS. gp3 lets you buy IOPS independent of size at lower cost.
- FSx for Lustre persistent storage for data that must survive AZ loss. Lustre file systems are single-AZ; keep the durable copy in S3.

**Hands-on**

1. **EBS gp3 tuning versus gp2.** Launch a small instance, attach a 100 GiB gp2 volume and a 100 GiB gp3 volume set to 6,000 IOPS, and run `fio` random read tests on both. Then use Elastic Volumes to change the gp2 volume to gp3 without detaching. Compare results with the CloudWatch `VolumeReadOps` metric. *Cost/cleanup:* EBS bills per GB-month and provisioned IOPS above baseline, pro-rated by the hour; delete volumes and terminate the instance within the hour.
2. **FSx for Lustre linked to S3.** Create a scratch FSx for Lustre file system of the smallest size with a data repository association to an S3 bucket containing sample files. Mount it on an instance, list the files (lazy-loaded), read them, write a result file and export it to S3. *Cost/cleanup:* Lustre bills per GB-month of provisioned capacity (the smallest is over 1 TiB), so this is not cheap; delete it within an hour. If cost is a concern, read the FSx for Lustre "data repository" documentation instead and do the EFS lifecycle exercise in the console.

### Skill 1.5.6: Data caching strategies

**Key concepts**

- **Amazon ElastiCache** (Valkey, Redis OSS, Memcached): sub-millisecond in-memory cache. Valkey and Redis OSS support replication, Multi-AZ failover, cluster mode sharding, persistence options, pub/sub, sorted sets and Global Datastore for cross-Region replicas. Memcached is multi-threaded, simple, no replication. ElastiCache Serverless removes node sizing.
- **DynamoDB Accelerator (DAX):** write-through, in-memory cache for DynamoDB with API compatibility (swap the client), microsecond reads for eventually consistent reads. Strongly consistent reads pass through to DynamoDB. Item cache plus query cache. Runs in your VPC. Choose it for read-heavy DynamoDB with hot items and minimal code change.
- **Patterns:** cache-aside (lazy loading: read cache, on miss read DB and populate; stale data possible, set TTL), write-through (write cache and DB together; fresher but writes cost more and cache fills with unread data), write-behind (buffer writes; risk of loss). Use TTLs plus explicit invalidation on update. Protect against thundering herd with jitter on TTLs and request coalescing.
- **Other caches:** CloudFront for HTTP content at the edge; API Gateway REST API caching per stage; RDS read replicas are not caches but offload reads. Session state in ElastiCache or DynamoDB for stateless apps.
- **Bedrock prompt caching** reduces cost and latency for repeated prompt prefixes on supported models (Domain 3 covers cost detail).
- Sizing: monitor `CacheHitRate`, evictions and memory; a low hit rate means wrong keys or TTLs, not too little memory.

**Exam traps**

- DAX for an application that needs strongly consistent reads to be faster. DAX does not cache strongly consistent reads.
- DAX in front of Aurora. DAX only works with DynamoDB; use ElastiCache for relational databases.
- Memcached when the requirement includes replication, failover or persistence. Use Valkey or Redis OSS.
- Caching with no TTL or invalidation on a pricing table that changes hourly, then blaming stale data on the database.

**Hands-on**

1. **Cache-aside with ElastiCache Serverless.** Create an ElastiCache Serverless Valkey cache and a small Aurora Serverless v2 or RDS free-tier PostgreSQL database in the same VPC. Write a Lambda function that reads a product by ID using cache-aside with a 60-second TTL and logs hits and misses. Invoke it repeatedly, update the product row, and observe stale reads until the TTL expires; then add delete-on-update and repeat. *Cost/cleanup:* ElastiCache Serverless has a minimum data storage charge billed hourly; delete the cache, database and function the same day.
2. **DAX versus direct DynamoDB.** Create a DAX cluster with the smallest node type and one node in a test VPC, and a DynamoDB table with 1,000 items. From an EC2 instance, run 10,000 `GetItem` calls with the DynamoDB client and then with the DAX client, comparing latency; then repeat with `ConsistentRead=true` through DAX and note there is no improvement. *Cost/cleanup:* DAX nodes bill hourly and are not free tier; delete the cluster and instance within an hour.

### Skill 1.5.7: Data cataloguing and cross-account data sharing

**Key concepts**

- **AWS Glue Data Catalog:** Hive-compatible metastore per account per Region for databases, tables, partitions and schemas, used by Athena, Redshift Spectrum, EMR, Glue and Lake Formation. Populate with crawlers, Glue jobs, Athena DDL or the API. Resource policies on the catalog allow cross-account access, but Lake Formation sharing is the governed path.
- **Lake Formation cross-account sharing:** grant table or database permissions (named resource or LF-Tags) to another account, OU or organization. Lake Formation uses AWS RAM behind the scenes. In the consumer account, create a resource link to the shared table, then grant local principals access to the resource link and the target. Data stays in the producer's S3 bucket; consumers query with their own Athena or Redshift.
- **Data mesh:** producers own data products in their accounts, a central governance account holds the catalog and LF-Tags, and consumers subscribe. Amazon DataZone and SageMaker Unified Studio add a business catalog and subscription workflows on top of Lake Formation.
- **Redshift data sharing:** live, read-only (and write for some configurations) access to Redshift data across clusters, workgroups, accounts and Regions without copying; consumers pay their own compute.
- **AWS Clean Rooms:** lets multiple parties analyse their combined data without sharing raw rows. Each member keeps data in their own S3 (catalogued in Glue) or supported sources; they create configured tables with **analysis rules**:
  - *Aggregation:* only aggregate functions (COUNT, SUM, AVG and so on) on allowed columns, with minimum aggregation thresholds.
  - *List:* returns row-level lists of the overlap between tables on allowed columns, with no aggregation.
  - *Custom:* only pre-approved SQL analysis templates (or approved query authors), with optional **differential privacy** to add calibrated noise and a privacy budget.
  - The most restrictive rule across referenced tables applies. A table without an analysis rule cannot be queried.
- Clean Rooms roles: members who can run queries, members who receive results, and a configurable payer for query compute (billed in Clean Rooms Processing Units). Clean Rooms ML offers lookalike modelling and custom model training on collaboration data; AWS Entity Resolution integrates for identity matching.
- **Choosing a sharing mechanism:** same organisation, governed table access, Lake Formation. Warehouse-to-warehouse, Redshift data sharing. External partners who must not see each other's raw data, Clean Rooms. Raw file exchange with partners, S3 bucket policies or Access Points (with care), or Transfer Family.

**Exam traps**

- Copying data to the partner's bucket to compute overlap when the requirement says neither party may see the other's raw records. Clean Rooms analysis rules prevent row exposure.
- Sharing a Glue Data Catalog through a catalog resource policy plus S3 bucket policy when column-level restrictions are needed. Lake Formation permissions give column and row filters.
- Forgetting the resource link in the consumer account after a Lake Formation cross-account share; the shared table is not visible to local engines until a resource link is created and permissions are granted.
- Choosing the list analysis rule when the partners require that only aggregated results with a minimum group size leave the collaboration. Use the aggregation rule with thresholds (or custom with differential privacy).

**Hands-on**

1. **Lake Formation cross-account share.** Using two accounts (a second free account works), in the producer account create a Glue table over a small S3 dataset, register the location with Lake Formation, and grant `SELECT` on the table to the consumer account. In the consumer account, accept the RAM invitation if prompted, create a resource link, grant a local role access to the link and the target table, and query from Athena. *Cost/cleanup:* Athena queries and S3 storage cost pennies; revoke grants, delete the resource link, the table and the bucket.
2. **Clean Rooms aggregation rule.** Using your main account and a second free account as the partner, create two Glue tables of fake customers with overlapping hashed emails, one in each account. Configure each with an aggregation analysis rule that allows `COUNT DISTINCT` on the email column, `region` as a dimension, and a minimum threshold of 3. Create a collaboration, invite the second account, associate both tables, and run an overlap count by region; then try a `SELECT *` and watch it get rejected. *Cost/cleanup:* SQL queries bill in Clean Rooms Processing Unit hours with a minimum per query, so run only a few; delete the collaboration, configured tables and Glue tables.

### Practice questions: Task 1.5

**Question 1.5.1** A retail bank streams card transactions into its data lake and must honour customer erasure requests within 30 days by deleting individual rows. Analysts query the curated layer with Amazon Athena and Amazon Redshift, and compliance requires column-level masking of card numbers for most analysts. The data platform team is small and does not want to operate table maintenance jobs such as compaction and snapshot expiry. Which solution meets these requirements?

- A. Store curated data as partitioned Parquet in an S3 general purpose bucket, run a weekly AWS Glue job that rewrites affected partitions to remove erased customers, and control column access with S3 bucket policies.
- B. Store curated data in Amazon S3 Tables, integrate the table bucket with the AWS Glue Data Catalog and AWS Lake Formation, delete rows with Athena `DELETE` statements, and grant column-level permissions through Lake Formation.
- C. Store curated data in Apache Iceberg tables in an S3 general purpose bucket, schedule AWS Glue jobs for compaction and snapshot expiry, and grant column-level permissions through Lake Formation.
- D. Load all curated data into Amazon Redshift managed storage, delete rows with SQL, and use Redshift dynamic data masking for card numbers, with Athena querying through a federated connector.

<details>
<summary>Answer and explanation</summary>

**Correct answer: B**

**Why B is correct:** S3 Tables stores tables in Iceberg format, so row-level `DELETE` works for erasure requests. It runs compaction, snapshot management and unreferenced file removal automatically, meeting the small team's no-maintenance requirement. Integrating with the Glue Data Catalog and Lake Formation makes the tables queryable from Athena and Redshift with column-level permissions.

**Why the other options are incorrect:**

- **A:** Plain Parquet needs partition rewrites for every deletion, which is the custom maintenance the team wants to avoid, and S3 bucket policies cannot restrict individual columns.
- **C:** Self-managed Iceberg works for deletes and Lake Formation masking, but the scheduled compaction and snapshot jobs are exactly the operational work the team does not want.
- **D:** Redshift could handle deletes and masking, but moving the whole lake into the warehouse costs more and forces Athena through a federated connector, adding a component to maintain.

**Key takeaway:** Row-level changes plus no maintenance jobs on a lake points to Iceberg in S3 Tables, governed by Lake Formation.

</details>

**Question 1.5.2** A logistics company collects GPS events from 50,000 trucks. Three teams consume the stream: a fraud team that needs per-truck ordering and stateful 5-minute windows, a data lake team that needs Parquet files in S3, and a dispatch team that occasionally replays the last 48 hours after bugs. Producers use the AWS SDK, not Kafka. The company wants managed services with the least operational overhead. Which solution meets these requirements?

- A. Send events to Amazon Data Firehose with dynamic partitioning to S3, and have the fraud and dispatch teams query the S3 files with Amazon Athena.
- B. Send events to a self-managed Apache Kafka cluster on Amazon EC2 with 7-day retention, and run Kafka Streams applications for each team.
- C. Send events to an Amazon MSK provisioned cluster, use MSK Connect with an S3 sink connector for the lake, and run Apache Flink on Amazon EMR for fraud detection.
- D. Send events to Amazon Kinesis Data Streams in on-demand mode with truck ID as the partition key and 7-day retention, process fraud windows with Amazon Managed Service for Apache Flink, and attach Amazon Data Firehose to the stream to write Parquet to S3.

<details>
<summary>Answer and explanation</summary>

**Correct answer: D**

**Why D is correct:** Kinesis Data Streams preserves order per partition key (truck ID), supports multiple consumers, and extended retention allows the dispatch team to replay 48 hours. Managed Service for Apache Flink handles stateful windowed processing without clusters. Firehose reading from the stream converts to Parquet and delivers to S3 with no code. On-demand mode avoids shard planning.

**Why the other options are incorrect:**

- **A:** Firehose alone has no replay and adds buffering delay; querying S3 files does not give the fraud team ordered, stateful stream processing.
- **B:** Self-managed Kafka on EC2 means running brokers, patching and scaling, which fails least operational overhead.
- **C:** MSK is a good fit when producers need Kafka, but these producers use the AWS SDK, and Flink on EMR plus a provisioned MSK cluster is more to operate than the Kinesis option.

**Key takeaway:** Multiple consumers, ordering per key and replay point to Kinesis Data Streams; Firehose is a delivery sink, not a replayable stream.

</details>

**Question 1.5.3** A global payments company is building a wallet balance service in three Regions: us-east-1, us-east-2 and us-west-2. Customers can top up or spend from any Region. Regulators require that no acknowledged transaction is lost if a Region fails, and a balance read in any Region must reflect the latest write. Each write updates a single item. The team prefers serverless services. Which solution meets these requirements?

- A. Use an Aurora PostgreSQL Global Database with write forwarding enabled in the secondary Regions.
- B. Use DynamoDB global tables in multi-Region eventual consistency mode with strongly consistent reads in each Region.
- C. Use DynamoDB global tables in multi-Region strong consistency mode across the three Regions, with strongly consistent reads.
- D. Use Amazon RDS for PostgreSQL with cross-Region read replicas and RDS Proxy in each Region.

<details>
<summary>Answer and explanation</summary>

**Correct answer: C**

**Why C is correct:** MRSC replicates each write synchronously to another Region before acknowledging, giving an RPO of zero, and strongly consistent reads return the latest version in any replica Region. Writes are allowed in every Region. The single-item writes avoid the MRSC limitation on transactions, and DynamoDB is serverless.

**Why the other options are incorrect:**

- **A:** Aurora Global Database replicates asynchronously, so a Regional failure can lose writes within the lag window, and readers in secondary Regions may see stale data. It is also instance based, not serverless.
- **B:** MREC replicates asynchronously with last writer wins; a strongly consistent read is only strong for writes made in the same Region, and a Regional failure can lose recent writes.
- **D:** Cross-Region read replicas are asynchronous and read-only, so writes in other Regions are not possible and RPO is not zero.

**Key takeaway:** Zero RPO with latest-write reads in every Region and single-item writes: DynamoDB global tables MRSC.

</details>

**Question 1.5.4** A media company runs a rendering pipeline on 400 Linux EC2 instances in one Availability Zone. Input assets (200 TB) live in Amazon S3. Each render job needs hundreds of GB/s of aggregate throughput and sub-millisecond latency to a shared POSIX file system for its duration. Results must be written back to S3. Separately, the company's Windows-based asset management tool needs an SMB share joined to its AWS Managed Microsoft AD that survives an AZ failure. Which combination of storage services meets these requirements most cost-effectively? (Select TWO.)

- A. Amazon FSx for Lustre scratch file systems linked to the S3 bucket through a data repository association, created per render batch.
- B. Amazon EFS with Max I/O performance mode and Provisioned Throughput, mounted by all render instances.
- C. Amazon FSx for Windows File Server Multi-AZ deployment joined to AWS Managed Microsoft AD.
- D. Amazon EBS io2 Block Express volumes with Multi-Attach shared across the render instances.
- E. Amazon FSx for NetApp ONTAP Single-AZ deployment with SMB shares for the asset tool.

<details>
<summary>Answer and explanation</summary>

**Correct answer: A, C**

**Why A and C are correct:** FSx for Lustre delivers parallel file system throughput and low latency for HPC-style rendering, lazy-loads input from S3 and exports results back, and scratch file systems created per batch avoid paying for persistent storage when idle. FSx for Windows File Server provides SMB with Active Directory integration, and the Multi-AZ deployment survives an AZ failure.

**Why the other options are incorrect:**

- **B:** EFS is a good shared POSIX file system but does not reach Lustre-class aggregate throughput and latency for hundreds of nodes at this scale, and it has no native S3 linkage for lazy loading.
- **D:** EBS Multi-Attach supports a limited number of instances in one AZ and requires a cluster-aware file system; it cannot serve 400 instances.
- **E:** FSx for ONTAP supports SMB, but a Single-AZ deployment does not survive an AZ failure.

**Key takeaway:** HPC over S3 data: FSx for Lustre; Windows SMB with AD and AZ resilience: FSx for Windows File Server Multi-AZ.

</details>

**Question 1.5.5** A credit card issuer and an airline want to measure how many of the airline's frequent flyers hold the issuer's card, broken down by tier and region, to plan a co-branded product. Neither company may see the other's customer-level records, the results must only include groups of at least 50 customers, and the airline agrees to pay for query compute. Both companies store their data in Amazon S3 in their own AWS accounts. Which solution meets these requirements?

- A. Have the airline share its table with the issuer through AWS Lake Formation cross-account grants with a data filter that hides direct identifiers, and run the overlap query in the issuer's Athena.
- B. Create an AWS Clean Rooms collaboration, configure each table with a list analysis rule on the hashed email column, and set the airline as the result receiver and payer.
- C. Have both companies upload hashed customer files to a shared S3 bucket in a third account, and run Athena queries from a role that both companies audit through CloudTrail.
- D. Create an AWS Clean Rooms collaboration, configure each table with an aggregation analysis rule that allows `COUNT DISTINCT` on the hashed email join key with dimensions tier and region and a minimum aggregation threshold of 50, and configure the airline as the payer for query compute.

<details>
<summary>Answer and explanation</summary>

**Correct answer: D**

**Why D is correct:** Clean Rooms lets each party keep data in its own account while queries run on the combined data. The aggregation analysis rule restricts output to aggregate functions on allowed dimensions and enforces a minimum aggregation threshold of 50, so no customer-level rows or small groups leave the collaboration. Clean Rooms lets members configure which member pays for query compute, so the airline can be the payer.

**Why the other options are incorrect:**

- **A:** Lake Formation sharing gives the issuer direct row-level access to the airline's data (even with some columns hidden), so the issuer sees the airline's customer records.
- **B:** The list analysis rule returns row-level overlap lists, which exposes individual matched customers and does not enforce the 50-customer threshold.
- **C:** A shared bucket hands each party access to the other's records, and hashing emails does not prevent re-identification by the party that holds the original emails.

**Key takeaway:** Joint analysis without exposing rows, with minimum group sizes: AWS Clean Rooms with an aggregation analysis rule.

</details>

---

# Domain 2: Security, Compliance, and Governance (25%)

Tasks 2.1 to 2.5: identity, encryption and key management, multi-account governance, compliance, and responsible AI controls.

## Task 2.1: Design identity security architectures

This task tests whether you can pick the right identity mechanism for each kind of principal: workforce humans, customer users, AWS workloads, workloads outside AWS, and cross-account callers. Then it tests whether you can bound what each one can do with layered policies. The recurring decision: which mechanism issues short-lived credentials here, and which policy layer stops the blast radius when something goes wrong.

### Skill 2.1.1: Identity federation with standard protocols

**Key concepts**

- Workforce access to many accounts: use AWS IAM Identity Center with an organization instance, connected to the corporate IdP (Entra ID, Okta, Ping) over SAML 2.0 for sign-in and SCIM for automatic user and group provisioning. SAML carries the authentication; SCIM keeps users and groups in sync. They solve different problems and you usually need both.
- Identity Center does not create users at first SAML sign-in. Without SCIM, someone must create users and groups in Identity Center before sign-in works. A scenario where "new hires cannot sign in until an admin creates them" is fixed by enabling SCIM, not by changing SAML settings.
- Permission sets are templates. Identity Center provisions them into each assigned account as IAM roles named `AWSReservedSSO_<set>_<suffix>`, under the `aws-reserved/sso.amazonaws.com/` path. Do not edit those roles directly; change the permission set and re-provision. In resource policies, match them with a wildcard on the role ARN (`aws:PrincipalArn` with `ArnLike`) rather than hard-coding the generated suffix.
- Identity Center attributes for access control pass IdP attributes (department, cost center) as session tags, which enables ABAC across accounts with one permission set instead of one per team.
- Delegate Identity Center administration to a member account (delegated administrator) so day-to-day work does not happen in the management account. Permission sets that target the management account can only be managed from the management account.
- Direct IAM SAML or OIDC federation (an IAM identity provider in each account plus `AssumeRoleWithSAML` or `AssumeRoleWithWebIdentity`) is still valid for a single account, for machine flows such as GitHub Actions OIDC to AWS, or where an existing IdP integration must be kept per account. For many accounts, it is more operational overhead than Identity Center.
- OIDC for CI/CD: an IAM OIDC identity provider for `token.actions.githubusercontent.com`, with a role trust policy that pins `aud` and the `sub` claim (repository and branch or environment). An unpinned `sub` lets any repository on that platform assume the role.
- Identity Center trusted identity propagation passes the end user identity to services such as Amazon Redshift, Lake Formation and S3 Access Grants, so data access decisions and audit logs see the human, not a shared role.

**Exam traps**

- "Create IAM users in each account and sync passwords from Active Directory." It works but fails any "least operational overhead" or "short-lived credentials" requirement.
- Choosing SAML alone when the requirement is "users and group membership changes in the IdP must appear in AWS automatically." That is SCIM.
- Picking Amazon Cognito for workforce access to the AWS console. Cognito is for application users, not for AWS account access.
- Editing the `AWSReservedSSO_*` role in a member account to fix permissions. The next re-provisioning overwrites it.

**Hands-on**

1. **Identity Center with an external IdP.** Enable an organization instance of IAM Identity Center in a test organization (or an account instance if you only have one account). Create a free developer tenant at an IdP that supports SAML and SCIM (Okta and Microsoft Entra ID both offer free developer or trial tenants). Configure SAML with the Identity Center metadata, enable automatic provisioning, and paste the SCIM endpoint and token into the IdP. Add a user to a group in the IdP and watch it appear in Identity Center within minutes, then assign a permission set and sign in through the access portal. *Cost/cleanup:* Identity Center has no charge. Delete the permission set assignments, revoke the SCIM token, and delete the IdP test app.
2. **GitHub Actions OIDC role.** Create an IAM OIDC provider for GitHub, then a role whose trust policy requires `token.actions.githubusercontent.com:aud` equal to `sts.amazonaws.com` and `sub` matching `repo:<you>/<repo>:ref:refs/heads/main`. Run a workflow that calls `aws sts get-caller-identity`. Then push from another branch and confirm the assume fails. This proves the `sub` condition is the real security boundary. *Cost/cleanup:* free. Delete the role and the OIDC provider.

### Skill 2.1.2: Application authentication and authorization

**Key concepts**

- Amazon Cognito user pools authenticate application users and issue OIDC tokens (ID, access, refresh). Identity pools exchange a token (from a user pool, a social IdP, SAML, or an OIDC IdP) for temporary AWS credentials through an IAM role. Rule: need JWTs for your API, use a user pool. Need the browser or mobile app to call AWS services directly (S3 upload, DynamoDB), add an identity pool.
- Identity pools can choose the IAM role from token claims (role mapping rules), or use principal tags from claims for ABAC, for example to restrict S3 access to a prefix equal to the user's `sub` using `${cognito-identity.amazonaws.com:sub}`.
- OAuth 2.0 flows: authorization code with PKCE for browser and mobile apps; client credentials for machine-to-machine. Cognito supports client credentials with resource servers and custom scopes. The implicit grant is legacy and should not be the answer for a new app.
- Cognito user pools have feature plans (Lite, Essentials, Plus). Threat protection features such as adaptive authentication and compromised-credential detection sit in the Plus plan. If a scenario demands risk-based MFA for customer sign-ins with minimum custom code, that is the Cognito feature, not a Lambda trigger you write.
- API authorization choices: API Gateway Cognito authorizer (validates user pool tokens, scope checks), JWT authorizer on HTTP APIs (any OIDC issuer), Lambda authorizer (custom logic, cache the result), IAM authorization (SigV4, for AWS callers). For fine-grained, policy-based authorization inside the application, Amazon Verified Permissions evaluates Cedar policies against principals and resources.
- ALB can authenticate users itself (OIDC or Cognito) before forwarding, which removes auth code from legacy backends that cannot be changed.
- Amazon EKS Pod Identity gives pods AWS credentials. The role trusts the single service principal `pods.eks.amazonaws.com`, so the same role works across clusters without per-cluster OIDC providers. An association in the EKS API maps a namespace and service account to a role, and the EKS Pod Identity Agent (a DaemonSet add-on) serves credentials on each node. Credentials carry session tags such as cluster and namespace, which you can use for ABAC.
- IRSA (IAM roles for service accounts) uses a per-cluster IAM OIDC provider and a trust policy that names that provider and the service account. Pod Identity works only on Linux EC2 worker nodes. It is not supported on Fargate, Windows nodes, Outposts, EKS Anywhere or self-managed Kubernetes on EC2. For those, IRSA (or another mechanism) is still the answer. Both can run in the same cluster during migration.

**Exam traps**

- Using node instance profile permissions for all pods. Every pod on the node inherits them; it fails least privilege. Pod Identity or IRSA, plus blocking pod access to IMDS, is the fix.
- Choosing Pod Identity for pods on Fargate. It is not supported there; use IRSA.
- Using an identity pool when the backend only needs to validate a JWT. The identity pool is for AWS credentials, not API authentication.
- Embedding long-term access keys in a mobile app to upload to S3. Use an identity pool, or presigned URLs generated by a backend.
- Choosing a Lambda authorizer when the token is a standard Cognito or OIDC JWT and only scope checks are needed. The managed Cognito or JWT authorizer has less code and no extra Lambda cost.

**Hands-on**

1. **User pool plus identity pool for direct S3 upload.** Create a Cognito user pool with a hosted or managed login app client using the authorization code grant with PKCE. Create an identity pool that trusts the user pool and an authenticated role whose policy allows `s3:PutObject` only on `arn:aws:s3:::bucket/private/${cognito-identity.amazonaws.com:sub}/*`. Sign in with two test users from a small web page or the AWS CLI and confirm each can only write under its own prefix. *Cost/cleanup:* within free usage for a handful of users. Delete the identity pool, user pool and bucket.
2. **EKS Pod Identity versus IRSA (cost warning).** Create a small EKS cluster with one Linux managed node group and install the EKS Pod Identity Agent add-on. Create a role trusting `pods.eks.amazonaws.com` with read access to one S3 bucket, associate it with a service account, and run a pod that lists the bucket. Then run a pod with a different service account and confirm it is denied. If cost is a concern, do the same lab in AWS Workshop Studio (EKS workshops) instead. *Cost/cleanup:* the EKS control plane bills per hour and the nodes bill as EC2; finish in one sitting and delete the cluster, node group and any load balancers.

### Skill 2.1.3: Least-privilege access controls

**Key concepts**

- Policy evaluation order for a request: explicit deny anywhere wins. Then the request must be allowed by every applicable guardrail (SCPs on the principal's account, RCPs on the resource's account, permissions boundary, session policy) and granted by an identity-based or resource-based policy. Guardrails never grant.
- Cross-account access: either a role in the target account that the caller assumes (role trust policy plus caller permission for `sts:AssumeRole`), or a resource-based policy that names the external principal. With a resource-based policy, the caller keeps its own identity and permissions; with a role, it gives them up for the role's. This matters for S3 object ownership and for services that need the caller's own permissions at the same time.
- External ID: a condition `sts:ExternalId` in the role trust policy. Its purpose is the confused deputy problem when a third party (a SaaS vendor) assumes roles in many customer accounts. It is not a secret password and not needed for your own accounts. For AWS service principals, the equivalent protection is `aws:SourceArn` and `aws:SourceAccount` conditions.
- Permissions boundaries: a managed policy that caps an IAM user or role. Classic use is delegated administration: developers may create roles only if they attach a specific boundary (`iam:PermissionsBoundary` condition), so they cannot escalate beyond it.
- Session policies: passed on `AssumeRole` or `GetFederationToken`; they further restrict that one session. Use them to vend scoped-down credentials (for example per-tenant) from a broader role without creating a role per tenant.
- ABAC: compare `aws:PrincipalTag/x` with `aws:ResourceTag/x` (or `aws:RequestTag` on create). It scales with fewer policies, but you must protect the tags: deny `iam:TagRole`, resource tag changes, and require tags on create, or users can tag themselves into access.
- Role chaining (assuming a role from a role session) caps the session at one hour, regardless of the role's maximum session duration. A scenario that needs a 4-hour chained session is a trap.
- `aws:PrincipalOrgID` and `aws:PrincipalOrgPaths` in resource policies are the clean way to allow "any principal in our organization or OU" without listing account IDs.

**Exam traps**

- Using a permissions boundary or SCP to grant access. Neither grants; you still need an allow in an identity or resource policy.
- Using an external ID between your own accounts and calling it the security control. Scenarios with internal accounts want `aws:PrincipalOrgID` or specific role ARNs.
- ABAC without tag governance. The plausible option tags resources but lets the same users change tags.
- Long-term access keys for a cross-account batch job. Use role assumption.

**Hands-on**

1. **Delegated role creation with a boundary.** In a test account, create a boundary policy that allows only S3 and CloudWatch Logs actions. Give a "developer" role `iam:CreateRole` and `iam:PutRolePolicy` with a condition that `iam:PermissionsBoundary` must equal that boundary ARN, and deny changing or deleting the boundary. As the developer, create a role with `AdministratorAccess` attached and the boundary set, then use the IAM policy simulator to show it can still only do S3 and Logs. *Cost/cleanup:* free. Delete the roles and policies.
2. **ABAC across two projects.** Create a role with tag `project=blue` and a policy allowing `ec2:StartInstances` and `ec2:StopInstances` where `aws:ResourceTag/project` equals `${aws:PrincipalTag/project}`. Launch two `t3.micro` instances tagged `blue` and `green`, then try to stop each. Add a deny on `ec2:CreateTags` and `ec2:DeleteTags` for the `project` key and show the role cannot retag the green instance. *Cost/cleanup:* a few cents of EC2; terminate both instances.

### Skill 2.1.4: Data perimeter controls and external access validation

**Key concepts**

- A data perimeter has three conditions: only trusted identities, accessing trusted resources, from expected networks. Implement with SCPs (identity side: which resources and networks your principals may use), RCPs (resource side: who may access your resources), and VPC endpoint policies (network side: what can pass through the endpoint).
- Key condition keys: `aws:PrincipalOrgID`, `aws:ResourceOrgID`, `aws:SourceVpce`, `aws:SourceVpc`, `aws:SourceIp`, `aws:ViaAWSService`, `aws:PrincipalIsAWSService`. Perimeter deny statements need exceptions for AWS services acting on your behalf (`aws:PrincipalIsAWSService` or `aws:ViaAWSService`), or you break CloudTrail delivery, replication and similar flows.
- Resource control policies are an AWS Organizations policy type applied to resources in member accounts. They cap the maximum permissions on those resources for any caller, including principals outside the organization. They do not grant permissions, do not apply to resources in the management account, do not affect service-linked roles, and do not apply to AWS managed KMS keys. Supported services include Amazon S3, AWS STS, AWS KMS, Amazon SQS, AWS Secrets Manager, Amazon DynamoDB, Amazon ECR, Amazon CloudWatch Logs, Amazon Cognito, Amazon OpenSearch Serverless and others. EC2, RDS and Lambda are not on the list, so a design that relies on an RCP to protect an RDS snapshot or Lambda function does not work.
- VPC endpoint policies: an S3 gateway endpoint policy that allows only buckets in your organization (`aws:ResourceOrgID`) stops data exfiltration to an attacker's bucket from inside the VPC. Pair it with a bucket policy that denies access not from your endpoint (`aws:SourceVpce`).
- IAM Access Analyzer has three analyzer types. External access analyzers (free) find resource policies that grant access outside your zone of trust (account or organization) for resources such as S3 buckets, IAM role trust policies, KMS keys, Lambda functions, SQS queues, Secrets Manager secrets, SNS topics, EBS and RDS snapshots, ECR repositories, EFS file systems and DynamoDB tables and streams. Internal access analyzers (paid per resource monitored) find which principals inside the organization or account can reach selected critical resources; they cover a smaller set (S3 buckets, S3 directory buckets, RDS DB and cluster snapshots, DynamoDB tables and streams). Unused access analyzers (paid per role and user analyzed) find unused roles, unused access keys and passwords, and unused service-level and action-level permissions.
- Access Analyzer also validates policies: policy validation (grammar and best-practice warnings), and custom policy checks (`CheckNoNewAccess`, `CheckAccessNotGranted`, `CheckNoPublicAccess`) that you can run in a CI/CD pipeline to block changes before deployment. It can also generate a least-privilege policy from CloudTrail activity.
- AWS Verified Access provides VPN-less, zero trust access to internal applications. Each request is evaluated against Cedar policies using user identity from a trust provider (IAM Identity Center or any OIDC IdP) and device posture from a device trust provider (Jamf, CrowdStrike, JumpCloud). It supports HTTP(S) applications and, since early 2025, TCP-based access such as SSH, RDP and databases. Endpoint types include load balancer, network interface, network CIDR and RDS. Non-HTTP access needs the Verified Access connectivity client on the user's device.

**Exam traps**

- Using an SCP to stop an external account from reading your bucket. SCPs only restrict principals in your organization. Restricting external callers on your resources is an RCP (or the resource policy itself).
- Using an RCP to protect an EC2 or RDS resource type that RCPs do not support.
- Expecting the management account to be protected by SCPs or RCPs. It is not; keep workloads out of it.
- Choosing Client VPN plus security groups when the requirement is per-request authorization based on identity and device posture without network-level access. That is Verified Access.
- Running an unused access analyzer to find public buckets. Public or cross-account exposure is the external access analyzer; unused access is about over-permissioned identities.

**Hands-on**

1. **Access Analyzer findings and policy checks.** Create an account-level external access analyzer (free). Create an S3 bucket with a bucket policy granting `s3:GetObject` to a random other account ID and watch the finding appear; archive it with a rule, then remove the policy and see it resolve. Then run `aws accessanalyzer check-no-public-access` against a bucket policy JSON that allows `"Principal": "*"` and observe the FAIL result you would wire into a pipeline. *Cost/cleanup:* external analyzers are free; custom checks have a small per-call charge. Delete the bucket and analyzer.
2. **Endpoint policy exfiltration block.** In a VPC with a private EC2 instance (use Session Manager over interface endpoints, or a temporary public IP), add an S3 gateway endpoint whose policy allows only buckets where `aws:ResourceOrgID` equals your organization ID (or, with one account, `aws:ResourceAccount` equals your account). From the instance, copy a file to your own bucket (works) and to a public bucket owned by someone else (denied). *Cost/cleanup:* gateway endpoints are free; interface endpoints for Session Manager bill per hour, so delete them and terminate the instance afterwards.

### Skill 2.1.5: Workload identities for hybrid and multi-cloud

**Key concepts**

- IAM Roles Anywhere lets workloads outside AWS (on-premises servers, other clouds, edge devices) get temporary AWS credentials using X.509 certificates instead of long-term access keys. Building blocks: a trust anchor (an AWS Private CA, or your own CA certificate bundle), a profile (which roles may be assumed, optional session policy and managed policies, session duration from 15 minutes to 12 hours), and the role's trust policy, which trusts `rolesanywhere.amazonaws.com` and can condition on certificate attributes (subject, issuer, SAN).
- The workload uses the credential helper (`aws_signing_helper`) configured as a `credential_process` in the AWS CLI or SDK config. Private keys can stay in a TPM or PKCS#11 store.
- Revocation: import a CRL into Roles Anywhere; it does not query OCSP. If the scenario requires revoking a compromised server quickly, the design must include a process to update the imported CRL, or disable the profile or trust anchor.
- Decision rule for non-AWS workloads: workload already has an OIDC token from its platform (GitHub, GitLab, another cloud's workload identity, Kubernetes) use `AssumeRoleWithWebIdentity` with an IAM OIDC provider. Workload has, or can get, an X.509 certificate from an enterprise PKI, use Roles Anywhere. Workload is a managed server you want to patch and access, use Systems Manager hybrid activations (which also give the instance credentials for SSM).
- IAM Roles Anywhere supports ML-DSA PKIs as trust anchors, which matters for post-quantum planning (see 2.2.4).
- Roles Anywhere is in the list of services that RCPs support, so you can cap which trust anchors and profiles can be used across the organization.

**Exam traps**

- Creating an IAM user with access keys for an on-premises application and rotating keys with a script. It works, but fails "no long-term credentials."
- Picking AWS Private CA as mandatory. You can use an existing enterprise CA as the trust anchor; Private CA is optional and has a monthly charge.
- Using IAM Roles Anywhere for human users. It is for workloads; humans use Identity Center.
- Assuming certificate revocation works through OCSP automatically.

**Hands-on**

1. **Roles Anywhere with a self-made CA.** Use OpenSSL on your laptop to create a root CA and a client certificate with the correct key usage. Create a Roles Anywhere trust anchor from the CA certificate, a role trusting `rolesanywhere.amazonaws.com` with a condition on the certificate subject CN, and a profile that references the role with a one-hour session. Configure `credential_process` with `aws_signing_helper` and run `aws s3 ls`. *Cost/cleanup:* Roles Anywhere has no charge when you bring your own CA. Delete the profile, trust anchor and role.
2. **Revocation test.** Issue a second client certificate, generate a CRL that revokes it, and import the CRL with `aws rolesanywhere import-crl`. Confirm the revoked certificate fails `CreateSession` while the first still works. This shows the CRL is a static import you must refresh. *Cost/cleanup:* free. Delete the CRL and certificates.

### Practice questions: Task 2.1

**Question 2.1.1** A bank has 140 AWS accounts in AWS Organizations and uses Microsoft Entra ID as its workforce IdP. Today each account has an IAM SAML identity provider and hand-built roles, and joiners often wait two days for access because an administrator must create matching groups. Leavers sometimes keep access until the next manual review. The security team wants one place to define access per job function, automatic onboarding and offboarding based on Entra ID group membership, and the least operational overhead. Which solution meets these requirements?

- A. Keep the per-account IAM SAML providers, deploy the roles with CloudFormation StackSets, and map Entra ID groups to roles through SAML attribute assertions.
- B. Enable IAM Identity Center with an organization instance, connect Entra ID over SAML 2.0, enable SCIM provisioning, and assign permission sets to synced groups.
- C. Enable IAM Identity Center with Entra ID over SAML 2.0, create users and groups in the Identity Center directory, and run a nightly Lambda function that compares them with Entra ID.
- D. Create an Amazon Cognito user pool federated with Entra ID over OIDC, and use an identity pool with role mapping rules to issue credentials for each account.

<details>
<summary>Answer and explanation</summary>

**Correct answer: B**

**Why B is correct:** IAM Identity Center gives one place to define permission sets and assign them to groups across all accounts in the organization. SAML 2.0 handles sign-in with Entra ID, and SCIM pushes user and group membership changes automatically. Removing someone from the Entra ID group removes the assignment without any manual work. This is the managed path with the least ongoing effort.

**Why the other options are incorrect:**

- **A:** StackSets reduce role drift, but every account still needs its own provider and roles. Joiners and leavers still depend on group-to-role mapping in each account, and there is no central assignment view. It is more overhead than Identity Center.
- **C:** SAML works, but the custom Lambda sync is code you must build and maintain, and nightly runs leave a gap for leavers. SCIM is the managed protocol that removes this.
- **D:** Cognito is for application end users. It is not designed for workforce access to AWS accounts and the console across an organization, and identity pool roles per account would be heavy to manage.

**Key takeaway:** SAML signs people in; SCIM keeps users and groups current. Workforce access to many accounts means Identity Center with both.

</details>

**Question 2.1.2** A payments company runs microservices on Amazon EKS. One cluster has Linux managed node groups; a second cluster runs a batch namespace on AWS Fargate. Each service must have its own AWS permissions, and the platform team wants to stop maintaining a separate IAM OIDC provider and trust policy edit for every new cluster. Pods must not be able to use the node's instance profile. Which solution meets these requirements?

- A. Use EKS Pod Identity for every workload in both clusters and block pod access to IMDS on all nodes.
- B. Use IRSA for every workload in both clusters and create one IAM role per service per cluster with the cluster's OIDC provider in the trust policy.
- C. Attach a node IAM role per node group with only the permissions needed by the pods scheduled there, and use taints to place each service on its own node group.
- D. Use EKS Pod Identity for workloads on the Linux managed node groups, use IRSA for the Fargate workloads, and block pod access to IMDS on the EC2 nodes.

<details>
<summary>Answer and explanation</summary>

**Correct answer: D**

**Why D is correct:** EKS Pod Identity uses a single service principal, so one role can be reused across clusters without adding OIDC providers or editing trust policies per cluster. It runs through the Pod Identity Agent on Linux EC2 nodes. It is not supported on Fargate, so the Fargate namespace needs IRSA. Blocking IMDS on EC2 nodes stops pods from falling back to the node role.

**Why the other options are incorrect:**

- **A:** Pod Identity is not supported on Fargate, so the batch namespace would have no credentials.
- **B:** IRSA works everywhere but keeps the per-cluster OIDC provider and trust policy maintenance the team wants to remove.
- **C:** Node roles give every pod on the node the same permissions. Taints help placement but are not an identity boundary, and this multiplies node groups.

**Key takeaway:** Pod Identity for Linux EC2 nodes; IRSA is still required on Fargate, Windows, Outposts and EKS Anywhere.

</details>

**Question 2.1.3** A fintech SaaS provider needs read access to an S3 bucket and a CloudWatch Logs group in each of 300 customer AWS accounts. Customers create a role from a template the provider publishes. The provider's platform assumes those roles from a single role in its own account. The provider's security review highlights that a malicious customer could enter another customer's role ARN in the onboarding portal. The provider also wants the role sessions to be limited to read-only actions even if a customer over-grants the role. Which actions should the provider take? (Select TWO.)

- A. Generate a unique external ID per customer, store it with the customer record, require it in the template's trust policy, and always pass it on `sts:AssumeRole`.
- B. Add `aws:PrincipalOrgID` to the customer role trust policies so only principals in the provider's organization can assume them.
- C. Pass a session policy that allows only the required read actions when the platform calls `sts:AssumeRole`.
- D. Ask each customer to attach a permissions boundary to the provider's role in the provider's account.
- E. Require customers to create an IAM user for the provider with access keys stored in AWS Secrets Manager in the provider's account.

<details>
<summary>Answer and explanation</summary>

**Correct answer: A, C**

**Why A and C are correct:** The confused deputy risk is that the provider's own role can be tricked into assuming a different customer's role. A per-customer external ID that the provider generates and always sends means a customer cannot make the provider assume someone else's role, because that role expects a different ID. A session policy on `AssumeRole` intersects with the role's permissions, so even an over-granted customer role yields only the read actions the provider passes.

**Why the other options are incorrect:**

- **B:** The provider's role is legitimately in the provider's organization for every customer, so this condition does not distinguish between customers and does not stop the confused deputy.
- **D:** A permissions boundary attaches to a principal in its own account. Customers cannot attach boundaries to the provider's role, and a boundary would not limit what the assumed customer role can do.
- **E:** Long-term keys across 300 accounts add rotation burden and leak risk, and do nothing about scoping beyond what each user policy allows.

**Key takeaway:** External ID solves the third-party confused deputy; session policies scope down a session without new roles.

</details>

**Question 2.1.4** A lender's organization has a workload OU containing accounts that store customer documents in S3 buckets, secrets in AWS Secrets Manager, and keys in AWS KMS. Auditors require a preventive control so that no principal outside the organization can access these resources, even if a developer writes an overly permissive bucket or key policy. The control must cover future accounts in the OU automatically and must not break AWS services that write logs into those buckets. Which solution meets these requirements?

- A. Attach an SCP to the workload OU that denies all S3, Secrets Manager and KMS actions when `aws:PrincipalOrgID` does not equal the organization ID.
- B. Enable an organization-wide IAM Access Analyzer external access analyzer and use an EventBridge rule to invoke a Lambda function that removes any statement granting external access.
- C. Attach an RCP to the workload OU that denies S3, Secrets Manager and KMS actions when `aws:PrincipalOrgID` does not equal the organization ID and `aws:PrincipalIsAWSService` is false.
- D. Deploy an AWS Config conformance pack to the OU with rules that detect public S3 buckets and KMS keys, with automatic remediation through Systems Manager Automation.

<details>
<summary>Answer and explanation</summary>

**Correct answer: C**

**Why C is correct:** RCPs set the maximum permissions on resources in member accounts for every caller, including principals outside the organization. S3, Secrets Manager and KMS all support RCPs. Attaching to the OU covers new accounts through inheritance. The `aws:PrincipalIsAWSService` exception keeps AWS service principals, such as log delivery, working.

**Why the other options are incorrect:**

- **A:** SCPs restrict only principals inside the organization. An external account's principal is not subject to your SCPs, so this does nothing to stop outsiders.
- **B:** This is detective with automated cleanup. There is a window of exposure between the policy change and the Lambda fix, so it is not preventive.
- **D:** Config rules are detective and the remediation runs after the fact. They also do not cover all external-access cases such as cross-account grants to a specific foreign account.

**Key takeaway:** SCPs bound your principals; RCPs bound your resources against everyone. Add the AWS service exception.

</details>

**Question 2.1.5** An insurer runs a claims-scoring application on servers in its own data center and on virtual machines in another cloud provider. The application reads from an Amazon S3 bucket and writes to an Amazon SQS queue. The company already runs an enterprise PKI that issues X.509 certificates to every server, with private keys held in TPMs. Security policy bans long-term AWS access keys. Operations wants to be able to cut off a single compromised server. Which solution meets these requirements?

- A. Configure IAM Roles Anywhere with the enterprise CA as a trust anchor, a profile that maps to a scoped IAM role, the credential helper on each server, and a process that imports updated CRLs.
- B. Register each server with AWS Systems Manager using hybrid activations and grant the activation's IAM service role permission to read S3 and write to SQS for the application.
- C. Create an IAM OIDC identity provider for the enterprise PKI and have each server call `AssumeRoleWithWebIdentity` with its certificate.
- D. Issue short-lived certificates from AWS Private CA and use them for mutual TLS to an Amazon API Gateway endpoint that proxies S3 and SQS calls.

<details>
<summary>Answer and explanation</summary>

**Correct answer: A**

**Why A is correct:** Roles Anywhere exchanges the existing X.509 certificate for temporary credentials, so there are no long-term keys. The enterprise CA can be the trust anchor without buying Private CA, and the credential helper works with keys held in TPM or PKCS#11 stores. Importing an updated CRL revokes a single server's certificate.

**Why the other options are incorrect:**

- **B:** Hybrid activations give credentials for the SSM Agent to manage the server. Using that role for the application mixes management and application identity and is not the intended pattern; it also does not reuse the PKI.
- **C:** `AssumeRoleWithWebIdentity` takes a signed OIDC token from an OIDC issuer, not an X.509 certificate. A PKI is not an OIDC provider.
- **D:** This builds a custom proxy and replaces the existing PKI. It adds latency and operational work and still needs the proxy to hold AWS permissions.

**Key takeaway:** Certificates outside AWS map to IAM Roles Anywhere; OIDC tokens map to web identity federation.

</details>

## Task 2.2: Design solutions to implement encryption and key management strategies

This task tests whether you can match a key custody requirement to the right AWS option (AWS owned key, AWS managed key, customer managed key, imported material, CloudHSM-backed custom key store, external key store) and handle certificates for TLS without manual renewals. The recurring decision: who must control the key material, and what is the least operational way to prove it.

### Skill 2.2.1: Encryption at rest with customer managed keys and automatic rotation

**Key concepts**

- Key types by control: AWS owned keys (no visibility, no key policy), AWS managed keys (`aws/s3`, `aws/ebs`; visible in your account, AWS rotates them every year, you cannot change their policy or share them cross-account), customer managed keys (you own the key policy, grants, rotation, deletion and cross-account use). Any requirement for cross-account access, a custom key policy, auditing who used a specific key, or a custom rotation period means a customer managed key.
- Automatic rotation for customer managed keys: only for symmetric encryption keys whose material AWS KMS generated. The rotation period is configurable from 90 to 2,560 days (default 365). On-demand rotation (`RotateKeyOnDemand`) is also available, up to 25 times per key, and does not change the automatic schedule. Rotation keeps the same key ID and ARN and keeps old material to decrypt old ciphertext, so no re-encryption is needed.
- Not supported for automatic rotation: asymmetric keys, HMAC keys, keys with imported material, and keys in custom key stores. For these, rotate manually by creating a new key and moving an alias. For imported material, on-demand rotation works after you import new material in the pending-rotation state.
- Key policy is the root of trust. If the key policy does not allow the account (the default `arn:aws:iam::<acct>:root` statement), IAM policies alone cannot grant access. Cross-account use needs both the key policy in the owning account and an IAM policy in the caller's account.
- `kms:ViaService` limits a key to use through specific services (for example only S3 in one Region). `kms:EncryptionContext` conditions bind usage to context values, which helps tenant isolation.
- Grants give temporary, programmatic permissions, used by services such as EBS and RDS on your behalf. They are the answer when a service needs to use a key for a resource's lifetime without a policy edit.
- Multi-Region keys share key material and key ID across Regions, so you can decrypt in another Region without a call back to the source Region. Use them for DynamoDB global tables client-side encryption, cross-Region backups and DR. They are not required for S3 replication of SSE-KMS objects (S3 re-encrypts with a key in the destination Region).
- S3 Bucket Keys reduce KMS request volume and cost for SSE-KMS by caching a bucket-level data key. A scenario that complains about KMS throttling or KMS request costs on a busy bucket points here.
- Key deletion has a waiting period of 7 to 30 days. Deleting a key makes data unrecoverable; disabling is reversible. A preventive control is an SCP that denies `kms:ScheduleKeyDeletion` except for a break-glass role.

**Exam traps**

- "Enable automatic rotation on the imported-material key." Not supported; use manual or on-demand rotation after importing new material.
- Choosing an AWS managed key when another account must decrypt. AWS managed key policies cannot be edited, so cross-account sharing (for example of an encrypted RDS snapshot or AMI) fails.
- Re-encrypting all data after a rotation. Not needed; KMS keeps prior versions.
- Assuming a rotation period shorter than 90 days is possible with automatic rotation.

**Hands-on**

1. **Custom rotation period.** Create a symmetric customer managed key, then run `aws kms enable-key-rotation --rotation-period-in-days 180` and `aws kms get-key-rotation-status` to see the next rotation date. Run `aws kms rotate-key-on-demand` and then `aws kms list-key-rotations` to see the new material entry. Encrypt a string before and after rotation and decrypt both with the same key ID. *Cost/cleanup:* a customer managed key has a monthly charge, and rotated material versions can add to it; schedule deletion with the 7-day minimum.
2. **Cross-account snapshot sharing.** Create an encrypted EBS snapshot with an AWS managed key and try to share it with a second account; observe it cannot be used there. Copy the snapshot re-encrypting with a customer managed key, add the second account to the key policy, share again, and copy it in the second account. *Cost/cleanup:* a 1 GiB volume and snapshot cost cents; delete snapshots, volumes and schedule key deletion.

### Skill 2.2.2: Encryption in transit with certificate management

**Key concepts**

- Standard (non-exportable) ACM public certificates are free when used with integrated services such as ELB, CloudFront and API Gateway, and ACM renews them automatically when DNS validation stays in place. DNS validation is the answer for "no manual renewal"; email validation needs a person to click.
- Classic ACM public certificates cannot be exported. Since mid-2025, ACM can issue exportable public certificates if you choose the export option at request time. You get the private key to install on EC2, containers or on-premises. These carry a per-certificate charge and certificates issued before that launch cannot be exported. If a scenario needs a public certificate on an EC2 web server or an on-premises appliance with AWS managing issuance, exportable ACM certificates are now a valid answer.
- CloudFront requires its ACM certificate in us-east-1. Regional services (ALB, API Gateway Regional endpoints) need the certificate in their own Region.
- TLS 1.3: choose an ELB security policy that includes TLS 1.3, and on CloudFront pick a security policy that sets the minimum protocol. For end-to-end encryption, ALB re-encrypts to targets over HTTPS; targets can use self-signed or Private CA certificates because ALB does not validate target certificates. NLB can pass TLS through (TCP listener) when the target must terminate TLS, for example for mutual TLS or a fixed certificate.
- Mutual TLS: ALB supports mTLS in verify mode with a trust store (CA bundle and optional CRLs) or passthrough mode. API Gateway custom domains support mTLS with a trust store in S3.
- AWS Private CA issues private certificates for internal services, mTLS clients, IoT devices and service meshes. It can be shared across accounts with AWS RAM. Short-lived certificate mode is cheaper per CA and suits certificates valid for a few days, where revocation is less needed. Private CA certificates requested through ACM renew automatically, but only from a general-purpose CA; short-lived mode caps validity at 7 days and is issued through the Private CA API, not ACM. Private CA has a monthly per-CA charge, so a hierarchy with a root plus subordinates is a real cost line.
- Private CA supports ML-DSA certificate authorities for post-quantum certificates.
- ELB TLS policies with hybrid post-quantum key agreement (ML-KEM) became available for ALB and NLB in late 2025.

**Exam traps**

- Requesting the CloudFront certificate in the application Region.
- Assuming email validation renews automatically.
- Choosing ALB to terminate TLS when the requirement says the backend must see the original client certificate or end-to-end TLS with no decryption in between. Use NLB TLS passthrough, or ALB mTLS passthrough which forwards the certificate in headers.
- Running a self-managed OpenSSL CA on EC2 for internal mTLS when the requirement says "managed" and "automatic renewal."

**Hands-on**

1. **ALB with TLS 1.3 and mTLS.** Request a public ACM certificate with DNS validation for a domain you own in Route 53. Create an ALB with an HTTPS listener using a TLS 1.3 security policy, and a Lambda target. Create a trust store from a CA you generate with OpenSSL, enable mTLS verify mode, and test with `curl --cert` with and without a client certificate. *Cost/cleanup:* the ALB bills hourly plus LCUs; delete the ALB, trust store and certificate the same day.
2. **Private CA short-lived mode (cost warning).** Create a Private CA in short-lived certificate mode, issue a certificate with `aws acm-pca issue-certificate` (ACM cannot issue from a short-lived mode CA), and inspect the validity period. Share the CA with a second account through AWS RAM and issue a certificate there. *Cost/cleanup:* Private CA charges per CA per month, prorated; delete the CA as soon as you finish (it enters a restorable deleted state, set the shortest period).

### Skill 2.2.3: Dedicated hardware security modules

**Key concepts**

- AWS CloudHSM gives single-tenant HSMs in your VPC that you control. AWS manages the hardware but cannot access your keys. Choose it when a regulation or contract requires a dedicated HSM, exclusive control of key material, or specific crypto APIs (PKCS#11, JCE, Microsoft CNG/KSP, OpenSSL Dynamic Engine), or for Oracle or SQL Server TDE and SSL/TLS offload on your own servers.
- The current hsm2m.medium type is validated for FIPS 140-3 Level 3 and lets you choose FIPS or non-FIPS mode per cluster. KMS itself uses FIPS-validated HSMs too, so "FIPS validated" alone does not force CloudHSM; "single-tenant" or "customer exclusive control of the HSM" does.
- High availability: a cluster with HSMs in at least two Availability Zones. A single HSM is a single point of failure. Backups are automatic and can be copied to another Region for DR.
- KMS custom key store backed by CloudHSM: KMS keys whose material lives in your cluster. You keep KMS integration with S3, EBS, RDS and others, with key material in your HSMs. It needs at least two active HSMs in different Availability Zones, and you pay for the HSMs and run the cluster. Automatic rotation is not available for these keys.
- External key store (XKS): KMS keys backed by an HSM you run outside AWS, through an XKS proxy. Use it when keys must never be in AWS at all. Every crypto operation calls your external system, so availability and latency depend on you.
- Cost: CloudHSM bills per HSM per hour; an HA pair runs well over a thousand dollars per month. It is not a lab you leave running.

**Exam traps**

- Choosing CloudHSM directly when the application only needs S3 and EBS encryption with a requirement for "keys in a dedicated HSM." The better answer is a KMS custom key store backed by CloudHSM, which keeps native integrations.
- Deploying one HSM to save cost in a production design that needs availability.
- Choosing XKS when the requirement is only "dedicated HSM." XKS is for keys held outside AWS and adds an availability dependency.
- Assuming AWS can recover a CloudHSM key if you lose crypto user credentials. It cannot.

**Hands-on**

1. **Short CloudHSM cluster lab (cost warning).** If budget allows one or two hours, create a CloudHSM cluster in two private subnets, add one hsm2m.medium HSM, initialize it, activate it with the CloudHSM CLI from an EC2 client, and create a crypto user and a key. Observe the security group the cluster creates and the backup that appears. Delete the HSM, then the cluster. *Cost/cleanup:* billed per HSM-hour from creation; delete the HSM and cluster and the EC2 client immediately. If you cannot spend this, use the CloudHSM getting-started guide as a console walkthrough.
2. **Design exercise: custom key store versus XKS.** Without creating resources, write a one-page decision record for a payments platform: list which services need encryption, availability targets, who holds key material, and failure modes. Compare a CloudHSM-backed custom key store, XKS and standard KMS keys against each requirement, then check each assumption against the KMS Developer Guide pages for custom key stores and external key stores. *Cost/cleanup:* free.

### Skill 2.2.4: Post-quantum cryptography strategies

**Key concepts**

- Two threats, two answers. "Harvest now, decrypt later" targets confidentiality of data in transit today; the fix is post-quantum key exchange (ML-KEM) in TLS. Forged signatures in the future target long-lived trust (code signing, firmware, root CAs); the fix is post-quantum signatures (ML-DSA). Data at rest under AES-256 symmetric keys is already considered quantum resistant, so you do not re-encrypt S3 because of quantum risk.
- Hybrid PQ TLS: AWS uses hybrid key agreement (for example X25519 combined with ML-KEM-768) so the connection is at least as strong as classical. AWS KMS, ACM and AWS Secrets Manager endpoints support it on non-FIPS endpoints. The client must offer it: for example AWS SDK for Java v2 with the AWS CRT HTTP client and the post-quantum TLS option enabled. Clients still offering the older pre-standard Kyber fall back to classical key agreement.
- Customer-facing endpoints: ALB and NLB have TLS security policies with hybrid PQ key agreement (late 2025). AWS Transfer Family and API Gateway also offer PQ-capable policies. You must choose those policies on new and existing resources; it is not automatic.
- KMS ML-DSA keys (mid-2025): key specs `ML_DSA_44`, `ML_DSA_65`, `ML_DSA_87`, used with the normal `Sign` and `Verify` APIs. KMS supports raw message signing and a pre-hashed "external mu" mode for large messages.
- KMS ML-KEM keys (October 2026): key specs `ML_KEM_512`, `ML_KEM_768`, `ML_KEM_1024` with key usage `ENCAPSULATE_DECAPSULATE`. The public key can encapsulate outside KMS; decapsulation happens inside KMS. Treat this as a new building block for custom key establishment; most designs still get ML-KEM through TLS.
- AWS Private CA can run ML-DSA CAs and issue ML-DSA certificates; IAM Roles Anywhere accepts ML-DSA PKIs as trust anchors.
- A strategy answer usually reads: inventory where asymmetric crypto is used, prioritize long-lived confidential data in transit, turn on hybrid PQ TLS on clients and load balancers, plan PQ signatures for long-lived roots and code signing, keep symmetric encryption at rest as is.

**Exam traps**

- "Re-encrypt all S3 data with a post-quantum key." Symmetric AES-256 is not the weak point; there is no such S3 option.
- Using ML-DSA for confidentiality. ML-DSA signs; ML-KEM establishes keys.
- Assuming server-side PQ support protects traffic from an old client. The client must offer the hybrid group.
- Assuming FIPS endpoints of KMS, ACM and Secrets Manager already negotiate hybrid PQ TLS; the published support is for non-FIPS endpoints.

**Hands-on**

1. **ML-DSA signing in KMS.** Create an asymmetric KMS key with key spec `ML_DSA_65` and key usage `SIGN_VERIFY`. Sign a small file with `aws kms sign --signing-algorithm ML_DSA_SHAKE_256 --message-type RAW`, then verify with `aws kms verify`. Change one byte and confirm verification fails. Download the public key and note its size compared with an ECC key. *Cost/cleanup:* monthly key charge plus small per-request charges; schedule deletion.
2. **PQ TLS on a load balancer.** Create an ALB (or NLB with a TLS listener) and select a TLS security policy whose name indicates post-quantum support, from the ELB security policy documentation. Use a recent OpenSSL build or a browser that supports X25519MLKEM768 and inspect the negotiated key exchange group. Switch back to a classical policy and compare. *Cost/cleanup:* load balancer hourly charge; delete it the same day.

### Practice questions: Task 2.2

**Question 2.2.1** A securities firm stores trade confirmations in Amazon S3 and Amazon DynamoDB in two Regions. A regulator requires encryption keys to be rotated at least every 180 days and every key use to be auditable per key. Another account in the organization runs a reporting job that must decrypt the S3 objects. The firm wants no custom code for rotation and no re-encryption of existing data. Which solution meets these requirements?

- A. Use AWS managed keys for S3 and DynamoDB, which AWS rotates automatically, and grant the reporting account access through an S3 bucket policy.
- B. Use customer managed keys with imported key material, schedule a Lambda function to import new material every 180 days, and add the reporting account to the key policies.
- C. Use AWS CloudHSM to generate keys and a scheduled script to create new KMS keys every 180 days, then move the aliases to the new keys.
- D. Use symmetric customer managed keys generated by AWS KMS with automatic rotation set to 180 days, and allow the reporting account's role in the key policy.

<details>
<summary>Answer and explanation</summary>

**Correct answer: D**

**Why D is correct:** Customer managed keys support a custom automatic rotation period between 90 and 2,560 days, so 180 days is configurable with no code. Rotation keeps the same key ID and older material, so existing data does not need re-encryption. Each use is logged in CloudTrail against that key. A key policy statement lets the reporting account decrypt, combined with an IAM policy there.

**Why the other options are incorrect:**

- **A:** AWS managed keys rotate yearly, which does not meet the 180-day rule, and their key policies cannot be edited to allow another account.
- **B:** Imported material does not support automatic rotation, so this needs the custom Lambda function the firm wants to avoid.
- **C:** New keys with alias moves mean older data stays under old keys and the firm must manage key lifecycle with scripts. It adds an HSM cluster without a requirement for one.

**Key takeaway:** Custom rotation periods (90 to 2,560 days) exist only for KMS-generated symmetric customer managed keys.

</details>

**Question 2.2.2** A health insurer runs a public web portal on Amazon EC2 instances behind a Network Load Balancer with TCP listeners, because the application must terminate TLS itself to inspect client certificates. The team currently buys certificates from a commercial CA and renews them manually, causing two outages last year. The team wants AWS to issue and track the public certificate, a short renewal process, and no change to where TLS terminates. Which solution meets these requirements?

- A. Request a standard ACM public certificate with DNS validation and attach it to a new TLS listener on the NLB.
- B. Request an exportable ACM public certificate with DNS validation, export it with its private key to the EC2 instances, and automate re-export and install on renewal.
- C. Create an AWS Private CA, issue a certificate for the portal domain, and install it on the EC2 instances.
- D. Move the portal behind an Application Load Balancer with an ACM certificate and enable mTLS passthrough so the instances receive the client certificate in headers.

<details>
<summary>Answer and explanation</summary>

**Correct answer: B**

**Why B is correct:** Exportable ACM public certificates provide the private key so TLS can still terminate on EC2 while AWS issues and tracks the certificate. DNS validation lets ACM renew it; the team then automates exporting the renewed certificate to the instances. This keeps the termination point and removes the commercial CA process.

**Why the other options are incorrect:**

- **A:** A standard ACM certificate on an NLB TLS listener moves TLS termination to the load balancer, which breaks the requirement that the application terminates TLS.
- **C:** Private CA certificates are not trusted by public browsers, so a public portal would show errors.
- **D:** This changes where TLS terminates. It can work for some designs, but it violates the stated constraint and needs application changes to read headers.

**Key takeaway:** Need an AWS-issued public certificate on your own servers? Use an exportable ACM certificate, chosen at request time.

</details>

**Question 2.2.3** A central bank's settlement platform must encrypt data in Amazon S3, Amazon EBS and Amazon RDS. The regulator requires keys to be generated and stored in single-tenant HSMs under the bank's exclusive control, validated to FIPS 140-3 Level 3. The platform team wants to keep using native service encryption without changing application code, and the design must tolerate the loss of one Availability Zone. Which solutions meet these requirements? (Select TWO.)

- A. Create an AWS CloudHSM cluster with HSMs in at least two Availability Zones.
- B. Use standard KMS customer managed keys, because KMS HSMs are FIPS validated.
- C. Configure an external key store with an XKS proxy pointing to HSMs in the bank's data center.
- D. Use client-side encryption in the applications with keys fetched from CloudHSM through PKCS#11.
- E. Create a KMS custom key store connected to the CloudHSM cluster and create the service keys in it.

<details>
<summary>Answer and explanation</summary>

**Correct answer: A, E**

**Why A and E are correct:** CloudHSM provides single-tenant HSMs under customer control, and the current HSM type is validated to FIPS 140-3 Level 3. A cluster with HSMs in two or more Availability Zones survives an AZ loss, and the custom key store requires at least two HSMs in different AZs anyway. The KMS custom key store keeps native KMS integration for S3, EBS and RDS, so applications do not change.

**Why the other options are incorrect:**

- **B:** KMS uses validated HSMs, but they are multi-tenant and managed by AWS, which fails the single-tenant, exclusive-control requirement.
- **C:** XKS keeps keys outside AWS. That is not required here, and it makes every encryption operation depend on the bank's data center and network.
- **D:** Client-side encryption requires application code changes, and the services would store ciphertext they cannot manage natively.

**Key takeaway:** Dedicated HSM plus native service integration means a CloudHSM-backed KMS custom key store across two or more AZs.

</details>

**Question 2.2.4** A wealth management firm has customer portfolio data that must stay confidential for at least 25 years. Its threat model now includes adversaries that record encrypted traffic today to decrypt later with a quantum computer. Services call AWS KMS and AWS Secrets Manager from Java applications, and customers reach the firm through an Application Load Balancer. The CISO asks for the change that addresses this specific threat first, with the least disruption. Which solution meets these requirements?

- A. Re-encrypt all data at rest in Amazon S3 and Amazon RDS with new KMS keys that use the ML-DSA key spec.
- B. Replace all ACM certificates with certificates from an ML-DSA AWS Private CA and keep the current TLS policies.
- C. Enable hybrid post-quantum TLS in the Java SDK clients and select an ALB TLS security policy that supports hybrid ML-KEM key agreement.
- D. Move all traffic to AWS Site-to-Site VPN tunnels and rotate the IKE pre-shared keys monthly.

<details>
<summary>Answer and explanation</summary>

**Correct answer: C**

**Why C is correct:** Harvest-now-decrypt-later attacks the key exchange of recorded TLS sessions, so the fix is post-quantum key agreement. KMS and Secrets Manager endpoints support hybrid ML-KEM TLS, and the Java SDK client must opt in. ALB offers TLS policies with hybrid PQ key agreement for customer traffic. This changes configuration only, not data or applications.

**Why the other options are incorrect:**

- **A:** ML-DSA is a signature algorithm, not encryption, and symmetric AES-256 at rest is not the weak point for this threat.
- **B:** PQ certificates protect authentication against future forgery. They do not change the key exchange, so recorded sessions remain exposed.
- **D:** Classical IKE key exchange has the same weakness, and moving customer traffic to VPN is not feasible for a public portal.

**Key takeaway:** For "record now, decrypt later," fix key exchange first: hybrid ML-KEM in TLS on both client and server.

</details>

**Question 2.2.5** A lending platform uses mutual TLS between 60 internal microservices across 12 AWS accounts. Certificates are issued from a self-managed OpenSSL CA on an EC2 instance, and expired certificates have caused incidents. The new design must issue certificates from a managed CA shared across all accounts, issue fresh certificates automatically as services redeploy, so certificates never expire in service, and keep CA costs low. Certificates only need to be valid for a few days because services redeploy frequently. Which solution meets these requirements?

- A. Create a general-purpose AWS Private CA in each of the 12 accounts and issue one-year certificates through ACM with automatic renewal.
- B. Create one AWS Private CA in short-lived certificate mode in a security account, share it with AWS RAM, and have services request certificates with a validity of a few days.
- C. Request ACM public certificates for each service's internal DNS name and use them for mTLS between services.
- D. Store the OpenSSL CA key in AWS Secrets Manager with rotation enabled, and run a Lambda function that issues and distributes certificates daily.

<details>
<summary>Answer and explanation</summary>

**Correct answer: B**

**Why B is correct:** A single Private CA shared through AWS RAM serves all accounts from one managed CA. Short-lived certificate mode is priced lower than general-purpose mode and fits certificates that last a few days, which also reduces dependence on revocation. Services request new certificates as they redeploy, so expiry incidents disappear.

**Why the other options are incorrect:**

- **A:** ACM managed renewal would prevent expiry, but twelve general-purpose CAs multiply the monthly CA charge, which fails the low-cost requirement, and one-year certificates do not match the short-lived design.
- **C:** Public certificates need publicly resolvable domains and validation, and public CAs are not meant for internal service identities.
- **D:** This keeps a self-managed CA with custom issuance code, which is the operational risk the firm wants to remove.

**Key takeaway:** Internal mTLS at scale means AWS Private CA shared with RAM; use short-lived mode when certificates live only days.

</details>

## Task 2.3: Design multi-account governance and compliance solutions

This task tests how you set up an AWS Organizations estate so that guardrails, logging, sharing and compliance checks apply automatically to every account, including ones that do not exist yet. The recurring decision: which organization-level feature does this centrally with the least per-account work, and is it preventive or detective.

### Skill 2.3.1: Organizational structures, landing zones, account provisioning

**Key concepts**

- OU design follows policy, not the org chart. A common baseline: Security (log archive, audit/security tooling), Infrastructure (networking, shared services), Workloads (prod and non-prod as separate OUs), Sandbox, Policy Staging, Suspended. Separate prod from non-prod so SCPs differ.
- Keep the management account empty of workloads; SCPs and RCPs do not apply to it. Delegate administration of services (GuardDuty, Security Hub, Config, Identity Center, Firewall Manager, Backup, IPAM) to member accounts.
- AWS Control Tower sets up a landing zone: management, log archive and audit accounts, an organization CloudTrail trail, AWS Config, IAM Identity Center, and controls. Controls come in three behaviours: preventive (SCPs and RCPs), detective (AWS Config rules), and proactive (CloudFormation Hooks that block non-compliant resources before creation).
- Control Tower Region deny control blocks API use outside governed Regions, with built-in exceptions for global services.
- Account Factory creates accounts with a standard network and baseline through the console or Service Catalog. Account Factory for Terraform (AFT) runs a GitOps pipeline for account requests and per-account customizations. Account Factory Customization (blueprints) applies a CloudFormation or Terraform template during provisioning. Customizations for Control Tower (CfCT) deploys StackSets and SCPs in response to lifecycle events.
- Existing organizations can enroll accounts and OUs into Control Tower; drift appears when someone changes Control Tower-managed resources outside it.

**Exam traps**

- Building a landing zone with custom scripts when the requirement says "AWS managed" and "least operational overhead."
- Putting workloads in the management account to save an account.
- Using detective Config rules when the requirement says "prevent creation." Use a proactive control (Hooks) or an SCP.
- Choosing AFT when the team does not use Terraform; Account Factory with blueprints or CfCT fits CloudFormation teams.

**Hands-on**

1. **Control Tower landing zone in a fresh organization.** Use a new management account (Control Tower needs one without conflicting setup). Launch the landing zone with two governed Regions, create a Sandbox OU, and provision an account with Account Factory. Enable the Region deny control and try to create an S3 bucket in an ungoverned Region from the new account. *Cost/cleanup:* Control Tower itself has no charge, but Config, CloudTrail and logging storage accrue small charges per account; decommission the landing zone and close test accounts when done.
2. **Proactive control test.** Enable a proactive control such as one requiring S3 bucket versioning or encryption settings on an OU. Deploy a CloudFormation template that violates it and observe the Hook failure during stack creation, then fix the template. *Cost/cleanup:* Hooks invocations from managed controls are low cost; delete stacks and disable the control.

### Skill 2.3.2: Centralized logging and configuration management, including AI invocations

**Key concepts**

- CloudTrail organization trail: created from the management account or a delegated administrator, applies to all accounts, and member accounts cannot modify or delete it. Deliver to an S3 bucket in the log archive account, encrypt with a customer managed key, enable log file integrity validation, and protect the bucket with Object Lock and restrictive bucket policy. Data events (S3 object-level, Lambda invoke, many Bedrock resource calls) cost extra; choose them with advanced event selectors. Query the trail bucket with Amazon Athena (CloudTrail Lake closed to new customers on 31 May 2026; existing customers only).
- AWS Config: enable the recorder in every account and Region (Control Tower does this). An aggregator in a delegated administrator account gives an organization-wide, read-only view of configuration and compliance. Aggregators do not deploy rules or remediate; use organization Config rules or conformance packs for that.
- Amazon GuardDuty: delegated administrator, auto-enable for all existing and new members, protection plans (S3, EKS, runtime monitoring, malware protection, RDS login activity, Lambda network activity). Findings are per Region; aggregate through Security Hub or EventBridge. Extended Threat Detection correlates signals into multi-stage attack sequences.
- AWS Security Hub naming: AWS Security Hub (the unified service, generally available December 2025) correlates findings from GuardDuty, Amazon Inspector, Amazon Macie and Security Hub CSPM, computes exposures (for example a reachable instance with a critical vulnerability), uses the OCSF format, and integrates with ticketing and EventBridge. AWS Security Hub CSPM is the posture management part: security standards and controls (such as AWS Foundational Security Best Practices and CIS) and the earlier ASFF findings model. Exam wording will name the one it means.
- Amazon Security Lake: builds a security data lake in your account in S3, normalizes AWS sources to OCSF in Apache Parquet, and supports rollup Regions to consolidate data. Native sources include CloudTrail management and data events, VPC Flow Logs, Route 53 Resolver query logs, Security Hub CSPM findings, EKS audit logs and AWS WAF logs, plus custom and third-party sources. Subscribers either get notifications of new objects or query through Lake Formation tables (Athena, Redshift, partner SIEMs). Choose Security Lake when the requirement is "normalized, long-term, queryable security data owned by us, shared with a SIEM."
- AI invocation logging: Amazon Bedrock model invocation logging is off by default and is configured per account per Region. It records request and response bodies for `InvokeModel`, `InvokeModelWithResponseStream`, `Converse` and `ConverseStream` to S3, CloudWatch Logs or both. Destinations must be in the same account and Region. Bodies larger than 100 KB and binary outputs go to S3 only. To centralize, enable logging in each account (StackSets or Control Tower customizations) and forward from there: S3 replication or CloudWatch Logs subscription filters into the log archive account.
- CloudTrail for Bedrock: `InvokeModel` and `Converse` calls are management events without prompt content. Knowledge base `Retrieve` and `RetrieveAndGenerate`, `InvokeAgent`, `InvokeFlow` and `ApplyGuardrail` are data events that need advanced event selectors. So CloudTrail answers "who called which model when"; invocation logging answers "what was asked and answered."
- CloudWatch Logs: set retention per log group, use subscription filters for streaming to Firehose or a cross-account destination, and data protection policies to mask sensitive data such as PII in logs.

**Exam traps**

- Expecting CloudTrail to contain prompts and responses. It does not; enable Bedrock model invocation logging.
- Pointing Bedrock invocation logging directly at a bucket in another account. Not supported; log locally, then replicate or forward.
- Using a Config aggregator to remediate. It only aggregates.
- Assuming GuardDuty findings are global. They are Regional.
- Using Security Lake as a real-time detection engine. It is a data lake; detection happens in GuardDuty, Security Hub or the SIEM that subscribes.

**Hands-on**

1. **Bedrock invocation logging.** In one Region, create an S3 bucket and a CloudWatch log group with the documented bucket policy for `bedrock.amazonaws.com`. Enable model invocation logging for text to both destinations. Call a low-cost model with `aws bedrock-runtime converse`, then compare the CloudTrail event (model ID, no prompt) with the CloudWatch Logs record (full prompt and response). *Cost/cleanup:* a few cents of tokens and logs; disable logging, delete the log group and bucket.
2. **Organization trail and aggregator.** With two accounts in an organization, create an organization trail to a bucket in the second account with log file integrity validation, and confirm the member account cannot stop it. Register the second account as Config delegated administrator and create an organization aggregator; view compliance across accounts. *Cost/cleanup:* the first management-event trail copy is free; Config charges per configuration item and rule evaluation. Delete the trail, aggregator and recorder.

### Skill 2.3.3: Cross-account resource sharing

**Key concepts**

- AWS RAM shares resources without copying them: Transit Gateways, VPC subnets (VPC sharing), Route 53 Resolver rules, prefix lists, IPAM pools, AWS Private CA, License Manager configurations, Aurora DB clusters (for cloning), Glue Data Catalog resources, Network Firewall policies and others. Enable sharing with AWS Organizations so shares within the organization need no invitation and can target OUs, which covers new accounts automatically.
- VPC sharing: the owner account (usually networking) owns the VPC, subnets, route tables, NAT and endpoints; participant accounts launch resources into shared subnets. Participants cannot change the subnets or route tables, and each account pays for its own resources. It reduces the number of VPCs, Transit Gateway attachments and duplicated endpoints, and keeps the network team in control.
- Transit Gateway sharing: share the TGW from the network account; workload accounts create attachments. Enable auto-accept of shared attachments if you want no manual approval, and control routing with TGW route tables and associations (segmentation stays with the owner).
- Resolver rules sharing: create forwarding rules once in the network account and share them, so every VPC resolves on-premises names through central outbound endpoints.
- Use resource policies instead of RAM where a service supports them for cross-account access (S3, KMS, SQS, Lambda, ECR, EventBridge buses).

**Exam traps**

- Creating a TGW per account and peering them, instead of sharing one TGW.
- Expecting participants in a shared VPC to edit security groups owned by others or change route tables.
- Forgetting to enable RAM sharing with Organizations, which forces per-account invitations.
- Using VPC peering for 50 VPCs when the requirement is transitive routing and central inspection.

**Hands-on**

1. **VPC sharing between two accounts.** In account A, create a VPC with two private subnets and share one subnet with account B through AWS RAM. In account B, launch a `t3.micro` into the shared subnet and observe that B sees the subnet but cannot edit its route table. Check which account the instance bills to. *Cost/cleanup:* small EC2 charge; terminate the instance, delete the share and VPC.
2. **Transit Gateway share (cost warning).** Create a TGW in account A with auto-accept enabled and share it with account B. In B, attach a VPC and add a route; ping between instances in both VPCs. *Cost/cleanup:* each TGW attachment bills hourly plus data processing; delete attachments and the TGW within the hour.

### Skill 2.3.4: Policy enforcement with SCPs and RCPs

**Key concepts**

- SCPs define the maximum permissions for IAM principals in member accounts. They do not grant, do not affect the management account, and do not affect service-linked roles. Each OU or account can have up to five SCPs attached, and each policy has a size limit of 5,120 characters. Deny-list style (keep `FullAWSAccess`, add deny statements) is easier to maintain than allow-list style.
- RCPs define the maximum permissions on resources in member accounts, evaluated for every caller including external ones. Use RCPs to enforce organization-wide resource rules such as "only our organization can access our S3 buckets, KMS keys and secrets" or "require TLS for S3 access," without editing thousands of resource policies. RCPs are not supported by every service (see 2.1.4), do not apply to the management account, AWS managed KMS keys or service-linked roles.
- Combined effect: a request from principal P (account X) to resource R (account Y) must pass SCPs on X, RCPs on Y, plus the normal identity and resource policies.
- Common SCP patterns: deny leaving the organization, deny disabling CloudTrail, GuardDuty, Config and Security Hub, deny actions outside approved Regions (`aws:RequestedRegion` with exceptions for global services), deny root user actions, require IMDSv2 on `ec2:RunInstances`, protect security roles with `aws:PrincipalArn` exceptions.
- Declarative policies (Organizations) enforce service configuration baselines in the service control plane rather than at the API permission layer. For EC2 they cover settings such as allowed AMIs, block public access for AMIs, EBS snapshots and VPCs, serial console access and instance metadata defaults. They keep working as the service adds APIs, apply to service-linked roles, support custom error messages, and offer an account status report. Use them for "this setting must stay on everywhere" rather than writing many deny statements.
- Other organization policy types matter in scenarios: tag policies (standardize tag keys and values), backup policies, AI services opt-out policies, and Amazon Bedrock policies that enforce a guardrail on model inference across accounts (see 2.5.1).
- Test SCPs and RCPs in a Policy Staging OU first; an SCP error can lock out whole OUs.

**Exam traps**

- Using an SCP to restrict what an external account can do to your resources. That is an RCP.
- Applying an SCP and expecting the management account to be restricted.
- A Region deny SCP without exceptions for global services such as IAM, Organizations, CloudFront and Route 53, which breaks them.
- Writing an SCP condition with `aws:PrincipalOrgID` to restrict your own principals; your own principals already satisfy it. Use `aws:ResourceOrgID` for "our principals may only access our resources."

**Hands-on**

1. **Region deny SCP with exceptions.** In a test OU, attach an SCP denying all actions when `aws:RequestedRegion` is not in your two Regions, with `NotAction` for global services, and an exception for an admin role using `aws:PrincipalArn`. From a member account, try to launch an instance in a third Region and call IAM; observe which is denied. *Cost/cleanup:* free; detach and delete the SCP.
2. **RCP for S3 TLS and org-only access.** Enable the RCP policy type, then attach an RCP to a test OU that denies `s3:*` when `aws:SecureTransport` is false, and denies access when `aws:PrincipalOrgID` is not your organization unless `aws:PrincipalIsAWSService` is true. Test from an account outside the organization with a bucket policy that allows it, and observe the deny. *Cost/cleanup:* free; detach the RCP.

### Skill 2.3.5: Automated compliance monitoring

**Key concepts**

- AWS Config rules: managed rules for common checks; custom rules using Lambda or CloudFormation Guard. Rules can run on configuration change or periodically. Proactive evaluation mode checks resource definitions before deployment for supported rules.
- Conformance packs bundle rules and remediation actions and deploy organization-wide from the delegated administrator. Remediation uses Systems Manager Automation documents, either manual or automatic, with retry limits.
- Security Hub CSPM runs security standards (AWS Foundational Security Best Practices, CIS AWS Foundations, PCI DSS, NIST SP 800-53 and others) as controls. Most controls use AWS Config rules under the hood, so Config recording must be on. Use central configuration from the delegated administrator to enable standards and controls across accounts and Regions with policies, and cross-Region aggregation to a home Region. Automation rules update or suppress findings automatically.
- The new Security Hub consumes Security Hub CSPM findings together with GuardDuty, Inspector and Macie to prioritize exposures.
- Decision rule: need a specific custom check with remediation, use a Config rule or conformance pack. Need a framework score and standard controls across the organization, use Security Hub CSPM. Need prioritization of combined threats, vulnerabilities and posture, use Security Hub.

**Exam traps**

- Enabling Security Hub CSPM standards without Config recording in each account and Region; controls show no data.
- Enabling standards account by account when central configuration does it for the whole organization.
- Using Trusted Advisor as the compliance framework engine. It gives best-practice checks, not framework controls with organization policies.
- Expecting Config rules to block creation. In detective mode they evaluate after the change.

**Hands-on**

1. **Conformance pack with auto-remediation.** Deploy a conformance pack containing `s3-bucket-public-read-prohibited` and `s3-bucket-server-side-encryption-enabled` with an SSM Automation remediation. Create a non-compliant bucket and watch Config flag it and remediation fix it. *Cost/cleanup:* Config charges per item and evaluation, small for a lab; delete the pack and bucket.
2. **Security Hub CSPM central configuration.** In a two-account organization, set a delegated administrator for Security Hub, create a configuration policy enabling FSBP in two Regions, and apply it to the root. Review failed controls in the member account from the administrator account. *Cost/cleanup:* Security Hub CSPM has a free trial period, then charges per check; disable it and delete the policy afterwards.

### Skill 2.3.6: Automated vulnerability scanning and patch management

**Key concepts**

- Amazon Inspector scans EC2 instances (through the SSM Agent, or agentless using EBS snapshots for instances without the agent), ECR container images, and Lambda functions (package vulnerabilities and, optionally, code scanning). Enable it organization-wide from a delegated administrator with auto-enable for new accounts. Findings include a risk-adjusted Inspector score that considers network reachability.
- ECR scanning: basic scanning on push, or enhanced scanning (Inspector) with continuous re-scanning when new CVEs appear. Continuous scanning answers "detect new vulnerabilities in images already pushed."
- Inspector findings flow to Security Hub and EventBridge, so you can open tickets or block deployments. Inspector can export an SBOM.
- AWS Systems Manager Patch Manager: patch baselines (approval rules, auto-approval delay, approved and rejected patches), scan or install operations, maintenance windows. Patch policies through Quick Setup apply patching configuration across accounts and Regions in an organization. Results appear in Systems Manager compliance and can feed Config and Security Hub.
- Managed nodes need the SSM Agent, an instance profile (or Default Host Management Configuration), and network reach to Systems Manager endpoints (interface endpoints for private subnets).
- Immutable infrastructure alternative: rebuild AMIs with EC2 Image Builder and replace instances through Auto Scaling instance refresh, instead of patching in place.

**Exam traps**

- Choosing ECR basic scanning when the requirement says "re-evaluate when new CVEs are published."
- Patching instances in private subnets without interface endpoints or NAT for Systems Manager.
- Running Inspector per account without a delegated administrator, leaving new accounts unscanned.
- Patching containers in place. Rebuild the image and redeploy.

**Hands-on**

1. **Inspector on EC2, ECR and Lambda.** Enable Amazon Inspector in one account. Push an old public image (for example an outdated base image) to ECR with enhanced scanning, deploy a Lambda function with an old library version, and launch a `t3.micro` with the SSM Agent. Review findings and the Inspector score, and trace one to Security Hub. *Cost/cleanup:* Inspector has a free trial, then per-instance, per-image and per-function charges; disable Inspector, delete the repository, function and instance.
2. **Patch policy with Quick Setup.** Create a patch policy in Quick Setup that scans daily and installs weekly for instances tagged `Patch=yes`. Launch an older Amazon Linux instance, wait for the scan, and view compliance; then run the install from a maintenance window. *Cost/cleanup:* Patch Manager has no extra charge for EC2; terminate the instance and delete the policy.

### Practice questions: Task 2.3

**Question 2.3.1** A bank is building a new AWS estate for 80 product teams. The cloud platform team uses CloudFormation, not Terraform. Every new account must get a standard VPC, an organization trail, AWS Config, approved Regions only, and a set of baseline IAM roles, and the bank wants to block creation of unencrypted EBS volumes before resources are deployed. The platform team has three engineers and wants the least operational overhead. Which solution meets these requirements?

- A. Use AWS Organizations with custom scripts that create accounts through `CreateAccount`, then use CloudFormation StackSets to deploy the baseline and an SCP that denies unencrypted volumes.
- B. Deploy AWS Control Tower, use Account Factory with a blueprint for the VPC and roles, enable the Region deny control, and enable a proactive control for EBS encryption.
- C. Deploy AWS Control Tower with Account Factory for Terraform, store account requests in a Git repository, and use detective controls for EBS encryption.
- D. Use AWS Service Catalog with a custom account vending product built on Step Functions, and AWS Config rules with automatic remediation for EBS encryption.

<details>
<summary>Answer and explanation</summary>

**Correct answer: B**

**Why B is correct:** Control Tower provides the organization trail, Config and governed Regions out of the box. Account Factory with a CloudFormation blueprint adds the VPC and baseline roles during provisioning. A proactive control uses CloudFormation Hooks to block non-compliant resources before creation. This is managed and fits a small CloudFormation team.

**Why the other options are incorrect:**

- **A:** It works, but the team owns account vending scripts, Region controls and logging setup that Control Tower provides. That is more overhead.
- **C:** AFT requires running and maintaining Terraform pipelines, which the team does not use, and detective controls do not block creation.
- **D:** Custom account vending is code to maintain, and Config remediation acts after the volume exists.

**Key takeaway:** Control Tower gives the landing zone; proactive controls block before creation, detective controls report after.

</details>

**Question 2.3.2** A healthcare SaaS company runs a generative AI assistant on Amazon Bedrock in 25 workload accounts across two Regions. Compliance requires a tamper-resistant central record of who invoked which model and the full prompts and responses, retained for seven years, with security analysts able to query it. Some responses include generated images. Which combination of actions meets these requirements? (Select THREE.)

- A. Enable Amazon Bedrock model invocation logging in every workload account and Region with an S3 destination in the same account and Region, deployed through CloudFormation StackSets.
- B. Configure Bedrock model invocation logging in each workload account to write directly to an S3 bucket in the log archive account.
- C. Replicate the local invocation log buckets to a log archive bucket protected with S3 Object Lock in compliance mode and a seven-year retention.
- D. Create a CloudTrail organization trail that delivers to the log archive account, and query it with Amazon Athena.
- E. Rely on the CloudTrail organization trail alone, because `InvokeModel` and `Converse` events include the request and response.
- F. Send invocation logs only to CloudWatch Logs with a seven-year retention setting.

<details>
<summary>Answer and explanation</summary>

**Correct answer: A, C, D**

**Why A, C and D are correct:** Invocation logging is off by default and configured per account and Region, so StackSets can enable it everywhere. Destinations must be in the same account and Region, and S3 is required for large bodies and image outputs. Replicating to a log archive bucket with Object Lock in compliance mode makes the records tamper resistant for seven years. The organization trail records who called which model and when, and analysts can query it with Amazon Athena.

**Why the other options are incorrect:**

- **B:** Invocation logging does not support destinations in another account.
- **E:** CloudTrail records the model ID and caller, not prompt and response content.
- **F:** CloudWatch Logs cannot hold large bodies and binary image outputs, which go to S3 only, and it does not give Object Lock-style immutability.

**Key takeaway:** CloudTrail answers who and when; Bedrock invocation logging answers what. Log locally to S3, then replicate and lock.

</details>

**Question 2.3.3** A trading company has a networking account that owns a Transit Gateway and egress VPC, and 40 workload accounts that need connectivity to on-premises through it. The network team must keep sole control of routing and segmentation. New workload accounts are created weekly and must be able to attach without a ticket. Workload teams want to avoid running their own NAT gateways. Which solution meets these requirements?

- A. Create a Transit Gateway in each workload account and peer each with the networking account's Transit Gateway.
- B. Use VPC peering between each workload VPC and the egress VPC, with route tables managed by the network team through StackSets.
- C. Share the Transit Gateway with the workload OU through AWS RAM with sharing enabled for AWS Organizations, enable auto-accept for shared attachments, and route egress through the central VPC.
- D. Share the Transit Gateway with each workload account individually through AWS RAM invitations, and have workload teams create TGW route tables for their attachments.

<details>
<summary>Answer and explanation</summary>

**Correct answer: C**

**Why C is correct:** Sharing the TGW with an OU through RAM, with Organizations sharing enabled, covers new accounts without invitations. Auto-accept lets workload accounts attach without a ticket. Route tables and associations stay with the TGW owner, so the network team keeps routing control. Central egress through the egress VPC removes per-account NAT gateways.

**Why the other options are incorrect:**

- **A:** Forty TGWs with peering is costly and complex, and each workload account would control its own TGW routing.
- **B:** VPC peering is not transitive, so traffic cannot flow through the egress VPC to on-premises or the internet through its NAT.
- **D:** Individual invitations add manual steps for each new account, and TGW route tables are created by the owner, not by attachment accounts.

**Key takeaway:** Share one TGW with an OU through RAM; the owner keeps routing, auto-accept removes tickets.

</details>

**Question 2.3.4** A payments processor's security team must guarantee three rules across all member accounts: (1) no one in the organization may disable GuardDuty or stop CloudTrail, (2) no principal from outside the organization may read objects in any of the company's S3 buckets even if a bucket policy allows it, and (3) EBS snapshots and AMIs must never be made public, including through APIs AWS adds later. Which combination of controls meets these requirements with the least ongoing maintenance? (Select THREE.)

- A. An SCP that denies S3 access when `aws:PrincipalOrgID` is not the organization ID.
- B. An RCP that denies S3 access when `aws:PrincipalOrgID` is not the organization ID, with an exception for AWS service principals.
- C. An AWS Config rule that detects public snapshots, with automatic remediation that removes the public permission.
- D. A declarative policy for EC2 that enables block public access for EBS snapshots and AMIs.
- E. A permissions boundary on all IAM roles that denies `ec2:ModifySnapshotAttribute`.
- F. An SCP that denies GuardDuty disable and delete actions and CloudTrail stop and delete actions, with an exception for a break-glass role.

<details>
<summary>Answer and explanation</summary>

**Correct answer: B, D, F**

**Why B, D and F are correct:** SCPs cap what principals in member accounts can do, so they prevent disabling GuardDuty and CloudTrail. RCPs cap access to resources for any caller, so they stop outside principals even when a bucket policy is too permissive. Declarative policies enforce the block public access settings in the EC2 control plane, so they keep working as new APIs appear, without new deny statements.

**Why the other options are incorrect:**

- **A:** SCPs do not apply to principals outside the organization, so external readers are not affected.
- **C:** This is detective and leaves an exposure window before remediation.
- **E:** Boundaries must be attached to every role and maintained, they do not cover new APIs, and they do not apply to service-linked roles.

**Key takeaway:** SCP for your principals, RCP for your resources, declarative policy for service settings that must stay on.

</details>

**Question 2.3.5** A credit card issuer must show PCI DSS control status for 60 accounts in three Regions on one dashboard, re-evaluate container images in Amazon ECR when new CVEs are published, and patch EC2 instances in private subnets with no internet access every Sunday. New accounts must be covered automatically. Which solution meets these requirements?

- A. Enable Security Hub CSPM in each account by script, use ECR basic scanning on push, and run a Lambda function that calls `yum update` through EC2 Instance Connect.
- B. Use AWS Config aggregators for the PCI view, Amazon Inspector only for EC2, and AWS Systems Manager Run Command with a cron schedule over NAT gateways.
- C. Use Trusted Advisor organizational view for PCI checks, enhanced ECR scanning in each account set manually, and EC2 Image Builder with weekly AMI rebuilds.
- D. Use Security Hub CSPM central configuration with the PCI DSS standard and cross-Region aggregation, Amazon Inspector with a delegated administrator and enhanced ECR scanning, and a Quick Setup patch policy with Systems Manager interface endpoints.

<details>
<summary>Answer and explanation</summary>

**Correct answer: D**

**Why D is correct:** Security Hub CSPM central configuration enables the PCI DSS standard across accounts and Regions, and cross-Region aggregation provides one dashboard. Inspector with a delegated administrator auto-enables for new accounts and re-scans ECR images continuously when new CVEs appear. A Quick Setup patch policy schedules patching across the organization, and interface endpoints let private instances reach Systems Manager without internet access.

**Why the other options are incorrect:**

- **A:** Scripted enablement misses new accounts unless maintained, basic scanning only scans on push, and SSH-style patching does not scale or report compliance.
- **B:** A Config aggregator shows rule compliance but not a PCI standard view, Inspector only for EC2 misses ECR, and NAT contradicts the no-internet requirement.
- **C:** Trusted Advisor does not provide PCI DSS controls, and manual per-account ECR setup misses new accounts.

**Key takeaway:** Organization-wide compliance means delegated administrator plus auto-enable: Security Hub CSPM central configuration, Inspector, and Quick Setup patch policies.

</details>

## Task 2.4: Design architectures to meet compliance requirements

This task tests whether you can turn a regulation into concrete AWS controls: where data may live, who can read it, how you prove it, and how you keep evidence. The recurring decision: which control makes the rule preventive and provable, rather than a promise in a policy document.

### Skill 2.4.1: Data protection requirements

**Key concepts**

- Layer the controls: classify (Amazon Macie for S3 sensitive data discovery), encrypt (KMS customer managed keys where you need key policy control and per-key audit), restrict (identity policies, resource policies, RCPs, VPC endpoint policies, S3 Block Public Access at account level), log (CloudTrail data events for sensitive buckets, S3 server access logs where needed), and protect from deletion (versioning, Object Lock, AWS Backup Vault Lock).
- Use the key policy as a second authorization layer. Data encrypted with a customer managed key needs both S3 permission and `kms:Decrypt`, so you can separate "storage admins" from "data readers."
- Field-level protection: CloudFront field-level encryption for specific form fields, client-side encryption with the AWS Encryption SDK, or DynamoDB client-side encryption (AWS Database Encryption SDK) when even administrators of the data store must not see plaintext.
- Tokenization and masking: CloudWatch Logs data protection policies mask PII in logs; Glue DataBrew or Glue sensitive data detection for pipelines; Bedrock Guardrails sensitive information filters for AI prompts and responses.
- S3 Object Ownership set to bucket owner enforced disables ACLs, which removes a common source of accidental exposure.
- Lake Formation fine-grained permissions (column, row and cell level, tag-based access control) for analytics data instead of bucket-level IAM.

**Exam traps**

- Using SSE-S3 when the requirement is "separate key permissions from storage permissions" or "audit every use of the key."
- Relying on Macie to prevent access. Macie discovers and reports; it does not block.
- Encrypting with KMS and assuming that alone stops a privileged S3 admin. If that admin also has `kms:Decrypt` through the key policy's account root statement and broad IAM policies, it does not.

**Hands-on**

1. **Macie discovery job.** Upload a few files with fake test data (sample credit card numbers from test ranges, fake names and addresses) to an S3 bucket. Run a one-time Macie sensitive data discovery job and review the findings, then send them to EventBridge and an SNS email. *Cost/cleanup:* Macie has a trial and charges per bucket evaluated and per GB scanned; delete the job, disable Macie and delete the bucket.
2. **Separation of duties with a key policy.** Create a customer managed key whose key policy grants `kms:Decrypt` only to a `DataReader` role, and an S3 bucket using SSE-KMS with that key. Give a `StorageAdmin` role full S3 permissions but no KMS permission. Show that `StorageAdmin` can list and delete objects but cannot read them. *Cost/cleanup:* key monthly charge; schedule deletion and delete the bucket.

### Skill 2.4.2: Regulatory requirements such as data sovereignty

**Key concepts**

- Sovereignty has three parts: location (data stays in approved Regions), access (who, including the provider, can reach it), and control (who holds the keys and can turn things off).
- Location controls: Region deny SCP or the Control Tower Region deny control, plus per-service settings that could move data (S3 replication rules, backup copy actions, DynamoDB global table replicas, Aurora Global Database secondary Regions, cross-Region snapshot copy). Deny the APIs that configure replication to unapproved Regions.
- Key control: customer managed keys in-Region; CloudHSM-backed custom key stores for single-tenant control; external key store when the regulation requires keys to remain outside AWS and the ability to cut off access by withdrawing the key.
- AWS Nitro System design prevents AWS operators from accessing customer memory on EC2; this is part of the AWS answer to "provider access" in sovereignty discussions.
- Local processing requirements: AWS Outposts or Dedicated Local Zones when data must stay in a specific facility or country without an AWS Region.
- Generative AI: Bedrock cross-Region inference profiles can route requests to other Regions. Geographic profiles route within a geography (for example US or EU); global profiles can route to any supported commercial Region. A residency requirement rules out the global profile, and Region deny SCPs must allow the destination Regions a geographic profile uses, or requests fail.

**Exam traps**

- Treating "data stays in the EU" as satisfied by deploying the application in an EU Region while backups copy to us-east-1 by default in a backup plan.
- Choosing XKS only because the requirement says "customer managed key." A standard customer managed key in-Region usually meets that.
- Using a global Bedrock inference profile for workloads with strict residency.

**Hands-on**

1. **Block replication outside approved Regions.** Write an SCP that denies `s3:PutReplicationConfiguration` unless a tag or condition approves it, and denies `backup:StartCopyJob` and copy actions to non-approved Regions where condition keys allow. Test by creating a replication rule to a bucket in an unapproved Region. Read the AWS Backup and S3 condition key docs to confirm which keys exist before relying on them. *Cost/cleanup:* free; detach the SCP and delete buckets.
2. **Inspect inference profiles.** Run `aws bedrock list-inference-profiles` in an EU Region and inspect which destination Regions each system-defined profile lists. Note which are geographic and which are global, and write down which ones your residency policy allows. *Cost/cleanup:* free.

### Skill 2.4.3: Audit trail management for regulated workloads, including AI services

**Key concepts**

- CloudTrail organization trail to a log archive account, SSE-KMS, log file integrity validation (digest files prove that delivered log files were not changed or deleted), S3 Object Lock (compliance mode for regulators who require that no one, including root, can delete within retention), and lifecycle to S3 Glacier storage classes for cost.
- CloudTrail Lake (SQL over event data stores) closed to new customers on 31 May 2026 and is for existing customers only. For new designs, query the trail bucket with Amazon Athena (partition projection keeps scans cheap) or send events to CloudWatch Logs.
- Data events matter for regulated data: S3 object reads, DynamoDB item activity, Lambda invokes, and Bedrock knowledge base, agent, flow and guardrail calls. Use advanced event selectors to limit them to sensitive resources and control cost.
- AI audit trail: combine CloudTrail (caller, model ID, time) with Bedrock model invocation logging (prompt and response bodies) and, for agents, the agent's trace and AgentCore observability. Store invocation logs with Object Lock if they are records. Remember invocation logs can themselves contain PII, so encrypt with a customer managed key and restrict access.
- AWS Config history and Config advanced queries show what a resource looked like at a past time, which auditors often ask for alongside "who changed it."

**Exam traps**

- Using log file integrity validation alone as immutability. It detects tampering; Object Lock prevents deletion.
- Object Lock governance mode where the requirement says "no user, including administrators, can delete." That is compliance mode.
- Logging all S3 data events organization-wide when only a few buckets hold regulated data. Cost explodes; scope with selectors.

**Hands-on**

1. **Tamper-evident trail.** Create a trail with log file integrity validation to a bucket with Object Lock enabled (default retention in governance mode for the lab). Generate some events, then run `aws cloudtrail validate-logs`. Try to delete a log object and observe Object Lock blocking it. *Cost/cleanup:* a few cents; governance mode lets you remove objects with the bypass permission after the test, so avoid compliance mode in a lab.
2. **Athena query over the trail bucket.** From the CloudTrail console's event history, use "Create Athena table" to generate a table over the trail bucket from exercise 1 (or write one with partition projection on Region and date). In Athena, query for `ConsoleLogin` and `CreateAccessKey` events in the last day, filtering on the date partition. Note the bytes scanned per query and how the partition filter reduces it. *Cost/cleanup:* Athena charges per TB scanned, so a lab query costs fractions of a cent; drop the table and delete the query results bucket.

### Skill 2.4.4: Data residency controls

**Key concepts**

- Region selection is the primary control. Deploy everything for the regulated workload in approved Regions, and block others with SCPs.
- Global services need attention: IAM, Organizations, Route 53 and CloudFront are global control planes. CloudFront caches content at edge locations worldwide; restrict with geo restriction or do not put regulated content through it. Route 53 records are global metadata, not workload data.
- S3: avoid cross-Region replication to unapproved Regions; S3 Multi-Region Access Points route to buckets you choose, so include only approved Regions.
- Databases: DynamoDB global tables and Aurora Global Database replicate to every configured Region; RDS cross-Region read replicas and automated backup replication do as well.
- Logging can break residency: centralizing logs from an EU Region into a US log archive bucket moves data. Use a log archive bucket per jurisdiction, or Security Lake rollup Regions within the jurisdiction.
- Bedrock: choose in-Region model invocation or geographic inference profiles inside the permitted geography; invocation logs stay in the account and Region where you configure them.

**Exam traps**

- Centralizing all logs, including CloudWatch Logs with customer data, into one Region outside the jurisdiction.
- Using CloudFront in front of a residency-restricted API without considering edge caching.
- A Region deny SCP that blocks the Regions an approved Bedrock geographic profile routes to.

**Hands-on**

1. **Per-jurisdiction log archive.** Create two log archive buckets in different Regions. Configure CloudWatch Logs subscription filters in an EU Region workload to send to a Firehose stream that writes only to the EU bucket, and confirm no cross-Region delivery is configured. *Cost/cleanup:* Firehose charges per GB ingested, tiny for a lab; delete the streams, subscriptions and buckets.
2. **Residency audit with Config advanced query.** Use an AWS Config advanced query across your aggregator to list S3 buckets and DynamoDB tables by Region, and list any S3 bucket with a replication configuration. Compare against your approved Region list. *Cost/cleanup:* advanced queries have no extra charge beyond Config recording.

### Skill 2.4.5: Compliance evidence collection and reporting

**Key concepts**

- AWS Artifact provides AWS's own compliance reports (SOC, ISO, PCI attestation and others) and agreements (for example the Business Associate Addendum for HIPAA). The management account can accept an agreement for the whole organization, which covers existing and new member accounts; this needs all features enabled. You cannot accept an organization agreement for only some accounts.
- Artifact notifications alert you when new reports or agreements appear, and Artifact also exposes third-party ISV reports to customers granted access through AWS Marketplace Vendor Insights.
- Artifact is evidence about AWS (the "of the cloud" side). Evidence about your own configuration (the "in the cloud" side) comes from AWS Config conformance packs and Security Hub CSPM standards, which map controls to frameworks and report compliance, backed by CloudTrail records. AWS Audit Manager closed to new customers on 30 April 2026, so do not choose it for new designs.
- Shared responsibility shows up in questions: an auditor asking for proof that AWS data centers meet ISO 27001 wants an Artifact report, not a Config rule.

**Exam traps**

- Using AWS Artifact to show that your own S3 buckets are encrypted.
- Accepting the BAA in each account separately when the organization agreement covers all accounts automatically.
- Expecting Security Hub CSPM controls to produce AWS's SOC 2 report.

**Hands-on**

1. **Download and track an Artifact report.** In the Artifact console, accept the Artifact NDA, download the current SOC 2 report, and set up a notification configuration for new versions of SOC and PCI reports. Note what the report covers and what it explicitly leaves to customers (complementary user entity controls). *Cost/cleanup:* free.
2. **Map your own evidence.** Pick five PCI DSS requirements. For each, list the AWS evidence source that proves your side: Security Hub CSPM control IDs, a Config rule, a CloudTrail query, or an Artifact report for AWS's side. Check each control name against the Security Hub CSPM controls reference. *Cost/cleanup:* free desk exercise.

### Skill 2.4.6: Risk assessment, threat modeling and mitigation

**Key concepts**

- Threat modeling method: describe the system and data flows, identify threats per component (STRIDE is the common checklist), rank by likelihood and impact, choose mitigations, and track them. AWS publishes Threat Composer, an open-source tool for writing threat statements and mitigations in a structured form.
- AWS Well-Architected Tool runs reviews against the Security pillar and lenses (for example the Generative AI and Financial Services lenses) and records risks and improvement plans per workload.
- Continuous risk signals: Amazon GuardDuty (threats), Amazon Inspector (vulnerabilities and network reachability), IAM Access Analyzer (external and unused access), Security Hub CSPM (misconfigurations), and Security Hub (correlated exposures with prioritization).
- Mitigation patterns to name in answers: reduce attack surface (private subnets, VPC endpoints, no public IPs), limit blast radius (account separation, least privilege, RCPs), detect and respond (GuardDuty with EventBridge and automation), recover (backups with Vault Lock, cross-Region DR).
- For generative AI, add threats such as prompt injection, sensitive data disclosure, excessive agent permissions and ungrounded output; mitigations include Bedrock Guardrails, scoped agent identities, human approval steps and contextual grounding checks.

**Exam traps**

- Treating a penetration test or a one-time Well-Architected review as continuous risk management.
- Choosing more logging when the scenario asks for prioritization of the highest-risk issues; that is Security Hub exposure findings or Inspector scores.

**Hands-on**

1. **Threat model with Threat Composer.** Use the hosted Threat Composer tool (or run it locally from its GitHub repository) to model a small serverless API with API Gateway, Lambda, DynamoDB and Bedrock. Write at least eight threat statements and link each to a mitigation using a named AWS feature. *Cost/cleanup:* free.
2. **Well-Architected review.** In the Well-Architected Tool, define a workload, answer the Security pillar questions honestly for one of your own projects, and generate the improvement plan report. Add the Generative AI lens if your workload uses Bedrock. *Cost/cleanup:* the tool is free.

### Practice questions: Task 2.4

**Question 2.4.1** A European bank runs a customer analytics platform in eu-central-1 and eu-west-1. A national regulator requires that customer data, including backups and logs containing customer identifiers, never leaves the EU. The cloud team already has an organization-wide AWS Backup plan that copies backups to us-east-1 for DR, and a central log archive in us-east-1. The bank is also adding an Amazon Bedrock assistant that must stay within EU Regions. Which solution meets these requirements?

- A. Create an EU-only OU with an SCP that denies actions outside approved EU Regions, change the backup copy destination to an EU Region, use an EU log archive bucket for that OU, and use a Bedrock geographic inference profile for the EU.
- B. Keep the existing backup and logging design but encrypt all backups and logs with KMS multi-Region keys whose primary key is in eu-central-1.
- C. Create an EU-only OU with an SCP that denies actions outside EU Regions, keep the us-east-1 log archive, and use a Bedrock global inference profile for better throughput.
- D. Move the workload to AWS Outposts racks in the bank's EU data centers and replicate backups to us-east-1 through AWS Backup.

<details>
<summary>Answer and explanation</summary>

**Correct answer: A**

**Why A is correct:** The SCP blocks resource creation outside the approved Regions. Changing the backup copy destination and the log archive keeps copies inside the EU, because those central designs were the ones moving data out. A geographic EU inference profile routes Bedrock requests only within EU Regions, and the SCP must allow those destination Regions.

**Why the other options are incorrect:**

- **B:** Encryption does not change where data is stored. The regulator cares about location, so copies in us-east-1 still breach the rule.
- **C:** The us-east-1 log archive moves logs with customer identifiers out of the EU, and a global inference profile can route requests to non-EU Regions.
- **D:** Outposts keeps primary data local, but the backup copy to us-east-1 still breaks residency, and Outposts adds cost without need when EU Regions are allowed.

**Key takeaway:** Residency fails through side channels: backups, logs and AI routing. Fix each, then enforce with an SCP.

</details>

**Question 2.4.2** A health insurer stores claim documents in Amazon S3 and runs a Bedrock-based summarization service. Auditors require that (1) every read of claim documents is attributable to a principal, (2) records of AI prompts and responses cannot be deleted by anyone for six years, and (3) any tampering with audit logs is detectable. The company wants to control logging costs. Which combination of actions meets these requirements? (Select TWO.)

- A. Enable S3 server access logging on all buckets in all accounts and send the logs to CloudWatch Logs with indefinite retention.
- B. Enable CloudTrail S3 data events with advanced event selectors scoped to the claim document buckets, delivered with log file integrity validation.
- C. Store Bedrock invocation logs in an S3 bucket with Object Lock in governance mode and a six-year retention.
- D. Enable CloudTrail data events for all S3 buckets in the organization to keep the configuration simple.
- E. Store Bedrock model invocation logs in an S3 bucket with Object Lock in compliance mode and a six-year retention, encrypted with a customer managed key.

<details>
<summary>Answer and explanation</summary>

**Correct answer: B, E**

**Why B and E are correct:** CloudTrail S3 data events record each object read with the calling principal, and advanced event selectors limit them to the regulated buckets, which controls cost. Integrity validation makes tampering with delivered trail files detectable. Compliance mode Object Lock prevents anyone, including the root user, from deleting invocation log records during the six years, and a customer managed key restricts who can read the PII inside them.

**Why the other options are incorrect:**

- **A:** Server access logs are best effort and less structured than CloudTrail. Enabling them everywhere and keeping them forever in CloudWatch Logs is expensive and does not address the AI records.
- **C:** Governance mode allows users with the bypass permission to delete objects, so it fails "cannot be deleted by anyone."
- **D:** It works for attribution, but logging data events for every bucket in the organization adds large cost with no requirement behind it.

**Key takeaway:** Scope data events with selectors; compliance mode for "nobody can delete," governance mode only when admins may bypass.

</details>

**Question 2.4.3** A US hospital network wants to start storing protected health information in AWS across 30 accounts in its organization, and expects to add accounts every quarter. Its compliance officer needs the HIPAA agreement in place for all current and future accounts with the least effort, and must give the board AWS's latest third-party audit reports on data center controls. Which solution meets these requirements?

- A. Accept the Business Associate Addendum in AWS Artifact in each account individually, and request SOC reports from AWS Support.
- B. Enable Security Hub CSPM with the HIPAA-related controls in all accounts, and export the compliance summary as the BAA evidence.
- C. Accept the organization BAA in AWS Artifact from the management account with all features enabled in AWS Organizations, and download SOC and ISO reports from AWS Artifact.
- D. Use AWS Audit Manager with a HIPAA framework to generate the agreement for each account, and share assessment reports with the board.

<details>
<summary>Answer and explanation</summary>

**Correct answer: C**

**Why C is correct:** An organization agreement accepted in AWS Artifact from the management account covers all existing and future member accounts, which removes per-account effort as accounts are added. It requires all features enabled in Organizations. Artifact is also where AWS's SOC and ISO reports are downloaded, which is evidence about AWS's controls.

**Why the other options are incorrect:**

- **A:** Per-account acceptance works but must be repeated for every new account, and Artifact already provides the reports.
- **B:** Security Hub CSPM reports on the customer's configuration. It does not create a legal agreement with AWS or prove AWS's data center controls.
- **D:** Audit Manager closed to new customers on 30 April 2026, and even for existing customers it collects evidence about your own environment. It does not create the BAA with AWS.

**Key takeaway:** AWS Artifact covers AWS's side: reports and agreements, including organization-wide BAA from the management account.

</details>

**Question 2.4.4** A brokerage is launching a new trading API built on Amazon API Gateway, AWS Lambda, Amazon DynamoDB and an AI agent on Amazon Bedrock AgentCore that can place orders. The CISO wants a structured view of threats before launch, ranked mitigations that reference specific AWS controls, and continuing visibility after launch into which issues are most likely to be exploited. Which approach meets these requirements?

- A. Run a threat modeling workshop using STRIDE and record threats and mitigations in Threat Composer before launch, then use AWS Security Hub with GuardDuty, Inspector and Security Hub CSPM to prioritize exposures after launch.
- B. Run a one-time penetration test against the API before launch and repeat it each year, storing the reports in AWS Artifact.
- C. Enable AWS Config with all managed rules before launch, and treat the compliance percentage as the risk score after launch.
- D. Complete a Well-Architected review of the Reliability pillar before launch, then use AWS Trusted Advisor checks for ongoing risk ranking.

<details>
<summary>Answer and explanation</summary>

**Correct answer: A**

**Why A is correct:** Threat modeling with STRIDE gives a structured list of threats per component, including agent-specific risks such as excessive permissions to place orders. Threat Composer records threat statements and links them to mitigations. After launch, Security Hub correlates GuardDuty, Inspector and posture findings into prioritized exposures, which answers "most likely to be exploited."

**Why the other options are incorrect:**

- **B:** A penetration test is a point-in-time check, not a threat model, and Artifact holds AWS's reports, not customer test reports.
- **C:** Config compliance shows configuration drift. It does not model threats or rank exploitability.
- **D:** The Reliability pillar is not the security review, and Trusted Advisor checks are best-practice guidance, not exposure prioritization.

**Key takeaway:** Threat model before launch; correlated, prioritized findings after launch.

</details>

**Question 2.4.5** A retail bank must prove to its regulator that only a small group of analysts can read unmasked customer columns in its S3 data lake, while other analysts see masked or filtered data. Data is queried with Amazon Athena and Amazon Redshift Spectrum. The bank also wants an automated way to discover new sensitive data that lands in the lake. Which solution meets these requirements?

- A. Create separate S3 buckets for masked and unmasked copies, run a nightly Glue job to produce the masked copy, and grant bucket-level IAM access by group.
- B. Use SSE-KMS with one key per column group and grant `kms:Decrypt` only to the privileged analysts.
- C. Use S3 Access Points per analyst group with access point policies that allow only certain prefixes.
- D. Register the lake with AWS Lake Formation, use column-level and row-level permissions with tag-based access control, and run Amazon Macie discovery jobs on the lake buckets.

<details>
<summary>Answer and explanation</summary>

**Correct answer: D**

**Why D is correct:** Lake Formation enforces column, row and cell-level permissions for Athena and Redshift Spectrum queries, and tag-based access control scales as tables are added. Its permissions and audit trail show the regulator exactly who can read the unmasked columns. Macie discovers sensitive data in the S3 buckets as new data lands.

**Why the other options are incorrect:**

- **A:** Duplicate copies double storage, add a nightly gap, and bucket-level access cannot express row-level rules.
- **B:** SSE-KMS encrypts whole objects, not columns, so it cannot give some users only some columns of the same table.
- **C:** Prefix-based access points work at object level. They cannot mask columns or filter rows within a file.

**Key takeaway:** Column and row-level analytics access means Lake Formation; discovery of sensitive data in S3 means Macie.

</details>

## Task 2.5: Design security and responsible AI controls

This task tests whether you can put safety and access controls around generative AI and agents: filter what goes in and out, give agents scoped identities instead of shared keys, and insert humans where an AI action carries risk. The recurring decision: enforce the control centrally in a managed service, not in each application's code.

### Skill 2.5.1: Content filtering and compliance controls with Amazon Bedrock Guardrails

**Key concepts**

- A guardrail combines policies: content filters (hate, insults, sexual, violence, misconduct, and prompt attack, with a strength per category, for text and images), denied topics (described in natural language), word filters (custom words and a managed profanity list), sensitive information filters (PII types to block or mask, plus custom regex), contextual grounding checks (flag responses not grounded in the source or not relevant to the query, for RAG), and Automated Reasoning checks (validate responses against formal rules derived from your policy documents).
- Apply a guardrail in three ways: pass the guardrail ID and version on `InvokeModel` or `Converse`; attach it to Bedrock Knowledge Bases or AgentCore agents; or call the `ApplyGuardrail` API on any text, which works for models outside Bedrock (SageMaker AI endpoints, self-hosted, third-party APIs).
- Input tagging lets you evaluate only part of a prompt (for example the user input, not your system prompt or retrieved documents). Reasoning content blocks are not evaluated.
- Use a numbered guardrail version in production, not the working draft, so changes are reviewed and reversible.
- Central enforcement: an AWS Organizations Amazon Bedrock policy references a guardrail ARN and version in the management account and enforces it on model inference in member accounts. The guardrail needs a resource-based policy allowing `bedrock:ApplyGuardrail` from those accounts. Account-level enforcement can also be set per account and Region. Enforced guardrails and request-level guardrails are combined, with the most restrictive result winning. Guardrails are Regional, so create one per Region. Automated Reasoning policies are not supported in enforced guardrails.
- `ApplyGuardrail` calls, including evaluations made during model invocation, can be logged as CloudTrail data events for the guardrail resource.

**Exam traps**

- Writing a Lambda function with regex to redact PII from prompts when a sensitive information filter does it with masking.
- Assuming a guardrail attached in one Region protects calls in another.
- Using denied topics to stop prompt injection. Prompt attack is a content filter category.
- Choosing contextual grounding checks to enforce business rules like eligibility logic; that is what Automated Reasoning checks are for, and they cannot be used in organization or account-level enforced guardrails.
- Relying on each team to add the guardrail parameter when the requirement says "all accounts must be protected." Use the Organizations Bedrock policy.

**Hands-on**

1. **Guardrail with PII masking and grounding.** Create a guardrail with a denied topic ("investment advice"), prompt attack filter at high strength, PII masking for email and phone, and contextual grounding. Publish version 1 and call `Converse` with it on a low-cost model, then send a prompt containing a fake email and a request for stock tips. Inspect the trace to see which policy intervened. *Cost/cleanup:* guardrails charge per text unit evaluated plus model tokens; a lab costs cents. Delete the guardrail.
2. **ApplyGuardrail on non-Bedrock text.** Call `aws bedrock-runtime apply-guardrail` with `source` set to `OUTPUT` and a paragraph you wrote that includes a fake phone number. Observe the masked output and the action `GUARDRAIL_INTERVENED`. This shows the same policy can protect an external model's responses. *Cost/cleanup:* cents; nothing to delete beyond the guardrail.

### Skill 2.5.2: Access controls for generative and agentic AI with AgentCore Identity

**Key concepts**

- Model access control: IAM policies on `bedrock:InvokeModel` and `bedrock:Converse` scoped to specific model or inference profile ARNs; SCPs to block unapproved models organization-wide; VPC interface endpoints for Bedrock with endpoint policies so traffic stays private.
- Amazon Bedrock AgentCore Identity handles two directions. Inbound auth decides who can invoke an agent runtime: IAM SigV4 (default) or JWT bearer tokens validated against an OIDC discovery URL with allowed audiences, clients and scopes (for example from Cognito, Okta or Entra ID). A runtime uses one inbound mechanism.
- Outbound auth lets the agent call external tools and APIs. Credential providers store OAuth client configuration or API keys. The agent gets a workload identity automatically and exchanges the caller's JWT for a workload access token. For user-delegated access (three-legged OAuth, `USER_FEDERATION`), the user gives consent once and AgentCore Identity stores the external token in its token vault, keyed by workload identity and user, so the agent acts with that user's permissions only. Machine-to-machine (two-legged) flows use client credentials.
- Design rules: agents should act on behalf of the signed-in user with that user's scopes, not with a shared service account that can read everyone's data. Keep tool credentials out of prompts, code and environment variables; use the token vault or Secrets Manager. Give each agent's execution role only the AWS actions its tools need.
- The user ID header option for invoking on behalf of a user is not verified against an authenticated identity, so production designs should use JWT inbound auth.
- Bedrock Knowledge Bases do not do per-user document authorization by themselves; use metadata filtering keyed to the user's attributes, or separate knowledge bases per tenant, to prevent the agent from retrieving documents the user may not see.

**Exam traps**

- Storing a third-party API key in the agent's prompt or Lambda environment variables when a credential provider or Secrets Manager is available.
- Giving the agent a single OAuth service account with access to all users' calendars or files. Use user federation so each call uses that user's token.
- Assuming the agent runtime accepts both SigV4 and JWT on the same runtime.
- Expecting knowledge base retrieval to respect source system ACLs automatically.

**Hands-on**

1. **JWT inbound auth with Cognito.** Create a Cognito user pool and app client. Deploy a sample agent to AgentCore Runtime with a custom JWT authorizer pointing at the user pool's OIDC discovery URL and allowing your client ID. Invoke the agent with and without a valid access token and observe the 401 response with the `WWW-Authenticate` header. *Cost/cleanup:* AgentCore Runtime charges for active compute and memory, small for a short lab; delete the runtime, user pool and ECR image.
2. **Outbound OAuth through the token vault.** Register an OAuth app with a provider you use (for example a Google or GitHub developer app), create an AgentCore Identity credential provider, and use the SDK's `requires_access_token` decorator with `USER_FEDERATION` in a tool function. Complete the consent flow once and call the tool twice to see the cached token reused. *Cost/cleanup:* small runtime charges; delete the credential provider and revoke the OAuth app.

### Skill 2.5.3: Human oversight workflows with approval mechanisms

**Key concepts**

- AWS Step Functions callback pattern: a task with `.waitForTaskToken` sends a task token (for example through SNS, SQS or a Lambda function that emails a link or posts to a ticket system) and pauses. A person approves or rejects through an API that calls `SendTaskSuccess` or `SendTaskFailure` with the token. Set a timeout (and optionally a heartbeat) so pending approvals do not hang forever, and route timeouts to a safe default such as reject.
- Standard workflows support the callback pattern and run up to one year, with full execution history for audit. Express workflows do not support `.waitForTaskToken`, so they are wrong for human approval.
- Pattern for AI operations: the model or agent proposes an action (refund, account change, infrastructure change), a Choice state checks risk (amount, confidence, guardrail result), low-risk actions run automatically, high-risk actions wait for approval, and every decision is logged with the approver identity. The approval API should authenticate the approver (for example Cognito or IAM Identity Center behind API Gateway), so the record shows a real person.
- For agents on AgentCore Runtime, use a user-confirmation or return-of-control pattern: the tool for an irreversible action pauses and hands the proposed call back to the application, which asks the user and resumes only on approval. For approvals by someone other than the user, have the tool start a Step Functions execution that waits on a task token. (Bedrock Agents Classic offered built-in user confirmation but closed to new customers on 30 July 2026.)
- For human review of low-confidence ML predictions, use a Step Functions callback with a custom review UI. (SageMaker Augmented AI closed to new customers on 30 July 2026.)
- Systems Manager Automation also has an approval step (`aws:approve`) for runbooks, useful when the AI output triggers an operational change.

**Exam traps**

- Using an Express workflow, or a Lambda function that sleeps, to wait for approval.
- Polling a DynamoDB table in a loop to detect approvals instead of the callback pattern.
- Letting the agent call the approval API itself with its own role. Separate the approver identity from the agent identity.
- No timeout on the approval state.

**Hands-on**

1. **Approval with task token.** Build a Standard state machine: a Lambda task that returns a proposed refund amount, a Choice state that auto-approves under a threshold, and a `.waitForTaskToken` task that publishes the token to SNS email for larger amounts. Create a small API Gateway plus Lambda endpoint that calls `SendTaskSuccess` or `SendTaskFailure`. Run both paths and review the execution history. *Cost/cleanup:* Standard workflows charge per state transition, with a free tier; delete the state machine, API, functions and topic.
2. **Approval gate before an AI action.** Build a second Standard state machine whose first state calls Amazon Bedrock `Converse` through the direct service integration to propose an action ("cancel order 123") as JSON. Add a `.waitForTaskToken` task that sends the proposal and token to an SQS queue, with `TimeoutSeconds` of 300, and a Catch on `States.Timeout` that routes to a "Rejected" state. Approve one execution with `aws stepfunctions send-task-success`, let another time out, and confirm that only the approved one reaches the Lambda function that would perform the action. *Cost/cleanup:* a few model tokens plus state transitions within the free tier; delete the state machine, queue and function.

### Practice questions: Task 2.5

**Question 2.5.1** A bank has 45 accounts where teams build generative AI applications on Amazon Bedrock in eu-west-1 and eu-central-1. The responsible AI committee requires that every model inference call in every account blocks hate and violence content, blocks prompt attacks, and masks customer account numbers, even if a team forgets to configure a guardrail. Teams may add stricter guardrails of their own. Which solution meets these requirements with the least operational overhead?

- A. Create a guardrail and version in the management account in each of the two Regions, add a resource-based policy allowing `bedrock:ApplyGuardrail` from the organization, and attach an Amazon Bedrock policy in AWS Organizations that enforces it on member accounts.
- B. Create one guardrail in eu-west-1 in the management account and attach an Amazon Bedrock policy in AWS Organizations that enforces it in both Regions.
- C. Publish a shared SDK wrapper that adds the guardrail ID to every `Converse` call, and use AWS Config to detect applications that do not use it.
- D. Deploy an SCP that denies `bedrock:InvokeModel` and `bedrock:Converse` unless the request includes a guardrail identifier condition.

<details>
<summary>Answer and explanation</summary>

**Correct answer: A**

**Why A is correct:** Organizations Bedrock policies enforce a referenced guardrail version on model inference in member accounts, whether or not teams configure one. Guardrails are Regional, so one is needed in each Region. The resource-based policy lets member accounts apply the management account's guardrail. Enforced and request-level guardrails combine, with the most restrictive result winning, so teams can add stricter controls.

**Why the other options are incorrect:**

- **B:** A guardrail in one Region does not protect inference in another; the policy references Region-specific guardrail ARNs.
- **C:** It depends on every team using the wrapper, and Config cannot see request parameters, so a forgotten call goes unfiltered.
- **D:** IAM and SCP conditions on the guardrail identifier can force callers to name a guardrail, but every application must still be changed to pass it, and forgotten calls fail instead of being protected. Pinning one guardrail ARN in the condition also stops teams from using their own stricter guardrail on the request.

**Key takeaway:** Organization-wide AI safety means an Organizations Bedrock policy pointing at a versioned guardrail in each Region.

</details>

**Question 2.5.2** An insurer is building an AI agent on Amazon Bedrock AgentCore Runtime that employees reach from an internal web app signed in through Okta. The agent reads each employee's own documents from a third-party document management SaaS over OAuth 2.0 and creates tickets in an internal system. Security requires that the agent never sees documents the signed-in employee cannot see, that no long-lived SaaS credentials are stored in code, and that only authenticated employees can invoke the agent. Which actions meet these requirements? (Select TWO.)

- A. Configure the runtime for IAM SigV4 inbound auth and give the web app's server role permission to invoke the agent for all users.
- B. Configure a custom JWT authorizer on the runtime with Okta's OIDC discovery URL and the web app's client ID.
- C. Create one OAuth service account in the SaaS with read access to all documents and store its client secret in AWS Secrets Manager for the agent.
- D. Pass the employee's user ID in the `X-Amzn-Bedrock-AgentCore-Runtime-User-Id` header so the agent can look up that user's documents.
- E. Create an AgentCore Identity OAuth credential provider for the SaaS and use three-legged user federation so tokens are stored per user in the token vault.

<details>
<summary>Answer and explanation</summary>

**Correct answer: B, E**

**Why B and E are correct:** A custom JWT authorizer validates the Okta token's issuer, client and audience, so only authenticated employees invoke the agent, and the runtime knows which user is calling. Three-legged user federation obtains a token with each employee's consent and scopes, so the SaaS returns only that user's documents. The token vault stores those tokens keyed by workload identity and user, so no long-lived secrets sit in code.

**Why the other options are incorrect:**

- **A:** SigV4 from a shared server role authenticates the app, not the employee, so the agent cannot act on behalf of a specific user.
- **C:** A service account with access to all documents lets the agent return documents the employee should not see, which breaks the first requirement.
- **D:** The user ID header is not verified against an authenticated identity, so any caller could claim to be another user; it is meant for development or trusted IDs.

**Key takeaway:** Authenticate the user inbound with JWT, then act as that user outbound with per-user OAuth tokens from the token vault.

</details>

**Question 2.5.3** A lending company uses a Bedrock model to recommend credit limit increases. Regulations require a human credit officer to approve any increase above $5,000, with a record of who approved, while smaller increases should apply automatically. Approvals can take up to three business days. The team wants a managed workflow with built-in audit history and minimum custom code. Which solution meets these requirements?

- A. An AWS Step Functions Express workflow with a Wait state that polls a DynamoDB approvals table every hour.
- B. A Lambda function that writes pending requests to SQS and a second Lambda function that sleeps until an approval arrives.
- C. An Amazon EventBridge Scheduler schedule that re-runs the model daily until a credit officer marks the request approved in a spreadsheet.
- D. An AWS Step Functions Standard workflow with a Choice state on the amount and a `.waitForTaskToken` task that notifies credit officers, who approve through an authenticated API that calls `SendTaskSuccess` or `SendTaskFailure`.

<details>
<summary>Answer and explanation</summary>

**Correct answer: D**

**Why D is correct:** Standard workflows support the callback pattern and run long enough for multi-day approvals. The Choice state auto-applies small increases. The task token pauses the execution until an authenticated officer approves or rejects, and the execution history plus the API's identity record who decided. A timeout on the task handles approvals that never arrive.

**Why the other options are incorrect:**

- **A:** Express workflows have a five-minute maximum duration, so they cannot wait three days, and polling adds code.
- **B:** Lambda functions time out after 15 minutes, so a sleeping function cannot wait for days, and there is no built-in audit history.
- **C:** Re-running the model creates new recommendations each day and the spreadsheet is not an auditable approval record.

**Key takeaway:** Human approval in AWS workflows means Step Functions Standard with `.waitForTaskToken` and a timeout.

</details>

**Question 2.5.4** A wealth manager runs a RAG assistant on Amazon Bedrock Knowledge Bases for financial advisers. Compliance found that the assistant sometimes states product fees that are not in the source documents and occasionally reveals client phone numbers from retrieved notes. The firm wants managed controls that reduce ungrounded answers and mask phone numbers in responses, without retraining a model. Which solution meets these requirements?

- A. Fine-tune the foundation model on the fee schedules and use a system prompt that instructs it never to output phone numbers.
- B. Add a Lambda function after each response that compares fees with a DynamoDB table and removes digits that look like phone numbers.
- C. Attach a guardrail with contextual grounding checks and a sensitive information filter that masks phone numbers to the knowledge base retrieve-and-generate requests.
- D. Increase the number of retrieved chunks and switch to a larger model to improve answer accuracy.

<details>
<summary>Answer and explanation</summary>

**Correct answer: C**

**Why C is correct:** Contextual grounding checks score each response against the retrieved source and the query and block or flag responses below the threshold, which targets fees not in the documents. The sensitive information filter masks phone numbers in output. Both are managed guardrail policies that need no model retraining.

**Why the other options are incorrect:**

- **A:** Fine-tuning is retraining, and prompt instructions are not a reliable control against disclosure.
- **B:** It is custom code to maintain, only covers fees in one table, and regex digit stripping is brittle.
- **D:** More context and a bigger model may help accuracy but do not enforce grounding or mask PII, and increase cost.

**Key takeaway:** Hallucination in RAG is a contextual grounding check; PII in output is a sensitive information filter.

</details>

**Question 2.5.5** A payments company runs a fraud model on an Amazon SageMaker AI endpoint and a customer-facing chatbot on a third-party model API outside AWS. The responsible AI team wants the same content policy (blocked topics, prompt attack detection and PII masking) applied to the chatbot's inputs and outputs that is already applied to its Bedrock applications, and wants each evaluation recorded centrally for audit. Which solution meets these requirements?

- A. Call the Amazon Bedrock `ApplyGuardrail` API with the existing guardrail version on chatbot inputs and outputs, and log `ApplyGuardrail` data events with a CloudTrail trail using advanced event selectors.
- B. Move the chatbot to Amazon Bedrock and pass the guardrail on each request, because guardrails only work with Bedrock-hosted models.
- C. Put AWS WAF in front of the chatbot API with managed rule groups for SQL injection and a custom rule for PII patterns.
- D. Use Amazon Comprehend to detect PII in chatbot traffic and write a denied topics list in the application code.

<details>
<summary>Answer and explanation</summary>

**Correct answer: A**

**Why A is correct:** `ApplyGuardrail` evaluates any text against a guardrail without invoking a Bedrock model, so the same versioned policy can protect a third-party model's prompts and responses. `ApplyGuardrail` calls are CloudTrail data events for the guardrail resource, which a trail with advanced event selectors records centrally.

**Why the other options are incorrect:**

- **B:** Guardrails are not limited to Bedrock-hosted models; migrating the chatbot is unnecessary work.
- **C:** AWS WAF inspects HTTP requests for web attacks. It does not understand topics or prompt attacks and does not inspect model responses.
- **D:** This rebuilds part of the guardrail in custom code, splits the policy into two places, and does not detect prompt attacks.

**Key takeaway:** `ApplyGuardrail` brings the same Bedrock guardrail to any model, inside or outside AWS.

</details>

---

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

# Domain 4: Resilience, Migration, and Business Continuity (15%)

Tasks 4.1 to 4.3: migration strategies, disaster recovery, and business continuity and resilience.

## Task 4.1: Design migration strategies by using AWS frameworks

This task tests whether you can pick the right migration path (the 7 Rs) for each workload and the right AWS tool for each phase: discover, assess, plan waves, move servers, move data, then modernize. The decision the exam keeps asking for is the lowest-effort path that still meets the stated deadline, downtime window and target-state goal. Watch for answers that recommend tools now closed to new customers.

### Skill 4.1.1: Discovery, portfolio assessment, dependency mapping and wave planning

**Key concepts**

- AWS Migration Hub and AWS Application Discovery Service (ADS) closed to new customers on 7 November 2025. Existing customers can finish their projects, but neither gets new features. For a new design, the answer is **AWS Transform**, which AWS names as the replacement for both.
- AWS Transform for migrations (originally VMware-focused, now covering VMware, Hyper-V, other virtual platforms and bare metal) runs as an agentic job in a workspace. It covers discovery, 7R strategy recommendations per server and application (with a confidence score and rationale), dependency-based wave planning, landing zone generation, network conversion to VPCs and transit gateways, and rehost through AWS Transform MGN or replatform to Amazon ECS or Amazon EKS.
- The **AWS Transform discovery tool** replaces the ADS Agentless Collector. It is an OVA you deploy into vCenter. It reads vCenter inventory and performance data with a read-only account, and it logs into guests over SSH (Linux) or WinRM (Windows) to collect network connections and SQL Server details. It needs no outbound internet. Data stays on the appliance until you export it (in Migration Portfolio Assessment format) and upload it.
- Why the network data matters: dependency mapping comes from observed connections (source IP, destination IP, port, protocol). That is what lets you put chatty servers in the same wave. vCenter inventory alone tells you what exists, not who talks to whom.
- **AWS Transform assessments** build the business case: right-sized EC2 recommendations from utilization data, pricing comparisons across On-Demand, Reserved Instances and Savings Plans, storage mapping (S3, EBS, FSx for NetApp ONTAP), what-if scenarios across Regions and instance families, and PPTX or PDF output. Inputs can be the discovery tool export, RVTools, CMDB exports, Migration Evaluator or MPA files.
- Fastest business case with least effort: an existing RVTools export or CMDB extract uploaded to an AWS Transform assessment. Most accurate right-sizing: performance data collected over time (discovery tool or Migration Evaluator), because allocated vCPU and memory usually overstate real use.
- Wave planning rules: group by dependency and shared data first, then by business owner and change window. Put low-risk, low-dependency applications in the first wave to prove the factory. Keep tightly coupled databases and their app servers in one wave, or plan for hybrid latency.
- The 7 Rs: retire, retain, rehost, relocate (for example VMware Cloud on AWS), repurchase (SaaS), replatform, refactor. The exam often hides the right R inside a constraint: "no code changes" means rehost or replatform; "license end in 3 months" means rehost first, optimize later; "vendor offers SaaS" means repurchase.
- The AWS Migration Acceleration Program (MAP) frames migrations in three phases: assess, mobilize, migrate and modernize. Mobilize is where the landing zone, wave plan and pilot migrations happen.

**Exam traps**

- Recommending Application Discovery Service agents or the Migration Hub console for a new customer. Both are closed to new customers; AWS Transform is the stated replacement.
- Using only the vCenter inventory to plan waves. It has no connection data, so it cannot find dependencies between VMs.
- Using AWS Config or Systems Manager Inventory for on-premises dependency mapping. Systems Manager Inventory can collect software inventory from hybrid nodes, but it does not build network dependency graphs or wave plans.
- Choosing a full refactor in the assessment phase for every application. The exam expects a portfolio approach: most servers rehost or replatform; only a few high-value applications justify refactoring.

**Hands-on**

1. **Build an assessment from an RVTools-style file.** Create an AWS Transform workspace in a supported Region and start a migration assessment job. Upload a small inventory file (you can build an RVTools-format sample with 10 to 20 fictional VMs and utilization columns). Review the right-sizing output, compare the On-Demand and Savings Plans scenarios, and export the report. This proves that a business case does not need agents or a live collector. *Cost/cleanup:* AWS Transform documentation states there is no additional charge for its migration capabilities, but confirm on the pricing page before you start; delete the workspace and job when done.
2. **Simulate dependency capture.** Launch two t3.micro Linux instances in one VPC (one "app", one "db" running a small PostgreSQL in a container). Generate traffic between them, then run `ss -tnap` on each to see the connection tuples the discovery tool collects. Turn on VPC Flow Logs to CloudWatch Logs and query them with Logs Insights to see the same dependency from the network side. This shows why connection data, not inventory, drives wave grouping. *Cost/cleanup:* Free tier eligible instances; Flow Logs ingestion costs a few cents. Terminate instances, delete the flow log and log group.

### Skill 4.1.2: Rehosting with AWS Transform MGN and AWS Transform

**Key concepts**

- AWS Application Migration Service was renamed **AWS Transform MGN** in June 2026. It is the same replication engine (still "MGN" in the API, CLI and docs paths). It keeps its compliance certifications and is available in all commercial and both GovCloud (US) Regions.
- Two ways to rehost now: the **AWS Transform MGN console**, which gives you direct control of replication, testing and cutover, or an **AWS Transform agentic migration job**, which drives discovery, wave planning, landing zone, network creation and then calls MGN for rehosting. Choose the console when the team wants hands-on control of a small number of servers. Choose AWS Transform when the scope is a large VMware estate and the ask is end-to-end automation with least effort.
- How MGN works: install the replication agent on each source server (or use agentless replication for vCenter), and it replicates block-level changes continuously to lightweight replication servers in a staging-area subnet in your AWS account, with data on low-cost EBS volumes. At test or cutover, MGN converts the replicated disks and launches the target EC2 instance from a launch template. Source servers keep running during replication, so downtime is limited to the cutover window.
- Lifecycle: install agent, initial sync, continuous data replication, launch test instance, mark ready for cutover, launch cutover instance, finalize cutover (stops replication and removes staging resources for that server). Always launch a test instance before cutover; the exam treats skipping tests as wrong.
- Post-launch actions run Systems Manager documents on the launched instance, for example to install the CloudWatch agent, swap licensing, or run custom scripts. This is how you automate "after migration, install agent X" with no manual steps.
- Network requirements: source servers need outbound TCP 443 to the MGN endpoint and TCP 1500 to the replication servers in the staging subnet. Over Direct Connect or VPN you can use private connectivity so replication never crosses the internet.
- MGN also migrates EC2 instances between Regions or accounts, and supports physical, VMware, Hyper-V and other-cloud sources because it replicates at the OS block level.
- Pricing shape: MGN has a free period per source server; you always pay for the staging-area EC2 and EBS resources and for the launched instances.
- AWS Transform can also containerize applications from source code and deploy them to Amazon ECS or Amazon EKS, which is a replatform path rather than a rehost.

**Exam traps**

- Choosing VM Import/Export or AMI export for dozens of live servers with a short cutover window. VM Import is a one-time image copy with no continuous replication, so the downtime equals the full transfer time.
- Choosing AWS Elastic Disaster Recovery for a one-way migration. DRS is for ongoing DR (continuous replication with failover and failback); MGN is the migration service.
- Recommending CloudEndure Migration or Server Migration Service. Both are retired.
- Forgetting the staging subnet and TCP 1500 in network designs. If a firewall blocks it, replication never starts.

**Hands-on**

1. **Rehost an EC2 instance as if it were on premises.** In Region A, launch a small Amazon Linux instance with a simple web page. In Region B, initialize AWS Transform MGN, create the replication settings template (small replication server type, default staging subnet), and install the agent on the Region A instance using credentials from a dedicated IAM user or role. Watch initial sync, launch a test instance, check the web page, then do a cutover and finalize it. This proves the full lifecycle and the TCP 443/1500 requirements. *Cost/cleanup:* You pay for the staging replication server, EBS and the launched instances; finalize cutover and terminate all instances, then delete the source server record and leftover EBS snapshots.
2. **Add a post-launch action.** In the MGN launch settings for the source server, add a post-launch action that runs the `AWS-ConfigureAWSPackage` Systems Manager document to install the CloudWatch agent. Launch a new test instance and confirm in Systems Manager Run Command history that the document ran. This proves migration-time automation without touching the source. *Cost/cleanup:* Small EC2 cost for the test instance; terminate it and remove the action.

### Skill 4.1.3: Database and data migration strategies (DMS, DataSync)

**Key concepts**

- **AWS DMS** moves data with full load, full load plus change data capture (CDC), or CDC only. Full load plus CDC keeps the target in sync while the source stays live, so cutover downtime is only the time to stop writes, let CDC drain and repoint the application.
- DMS deployment options: provisioned replication instances (you size them), or **DMS Serverless**, which scales capacity automatically. Choose Serverless when the load is unpredictable and the team wants least operational overhead.
- **Homogeneous data migrations** in DMS use native database tools (for example, for MySQL or PostgreSQL to the same engine on RDS or Aurora). For like-for-like moves, native tooling or Aurora/RDS-specific methods (snapshot restore, read replica promotion, Percona XtraBackup to Aurora MySQL) are often simpler than DMS.
- **Schema conversion** is needed only for heterogeneous migrations (for example, Oracle or SQL Server to PostgreSQL). Two tools:
  - **DMS Schema Conversion**: managed and browser-based inside DMS. You create an instance profile (network and encryption), data providers (source and target connection details) and a migration project (with credentials in Secrets Manager). It produces an assessment report of what converts automatically and what needs manual work, lets you edit converted code, and applies it to the target or exports SQL to S3. Its rules engine builds on the SCT engine, and generative AI can convert more of the objects the rules cannot handle on selected paths (for example Oracle, SQL Server, Db2 and SAP ASE to PostgreSQL) in selected Regions.
  - **AWS SCT**: a downloadable desktop application with the same conversion heritage. Use it when you need source/target pairs that DMS Schema Conversion does not support, such as many data warehouse sources to Amazon Redshift, or when you need its data extraction agents.
  - Rule of thumb: managed, no local install, least overhead and a supported path means DMS Schema Conversion. Unsupported pair means SCT.
- Neither tool converts application code automatically. Stored procedures and embedded SQL still need review. For SQL Server applications, AWS Transform for .NET can also move SQL Server schemas to Aurora PostgreSQL as part of a full-stack modernization.
- **AWS DataSync** moves files and objects online: NFS, SMB, HDFS, self-managed object storage, and other clouds (Azure Blob, Google Cloud Storage and others) to S3, EFS or FSx, and between AWS storage services. It handles encryption, integrity checks, scheduling, bandwidth limits and incremental transfers. On-premises sources need a DataSync agent (VM or EC2). **Enhanced mode** removes the object-count limits of the older mode for S3-to-S3, and supports agentless transfers from Google Cloud Storage, Azure Blob Storage and Oracle Cloud Object Storage to S3.
- DataSync over Direct Connect or VPN can use a VPC interface endpoint so traffic stays private.
- Offline transfer: Snowball Edge is closed to new customers. For new large offline moves, **AWS Data Transfer Terminal** lets you reserve time at a secure AWS facility and upload from your own storage devices over a high-speed connection. It is currently available to Enterprise Support customers (others ask AWS Support). Otherwise, size the network: a sustained DataSync transfer over a dedicated link is usually the answer.
- Transfer Family is for ongoing partner file exchange over SFTP, FTPS, FTP or AS2, not bulk migration. Storage Gateway is for hybrid access to cloud storage during or after migration, not for one-time bulk copy.

**Exam traps**

- Using DMS full load only when the requirement says "minimal downtime". Without CDC, every write after the load starts is lost or needs a second outage.
- Saying DMS converts schemas or stored procedures. DMS moves data; schema conversion is DMS Schema Conversion or SCT.
- Using S3 sync or robocopy scripts for tens of millions of files with "verify integrity" and "least operational overhead". DataSync does verification, retries and scheduling for you.
- Recommending Snowball Edge for a new customer. It is closed to new customers.
- Choosing a heterogeneous engine change for a "migrate in two months, no application changes" requirement. Move to the same engine on RDS or Aurora first.

**Hands-on**

1. **Full load plus CDC with DMS Serverless.** Create a small RDS for MySQL source (enable binary logging with a custom parameter group, `binlog_format=ROW`) and an Aurora MySQL Serverless v2 or RDS for PostgreSQL target. Create a DMS Serverless replication with full load and CDC, insert rows into the source while it runs, and watch them appear on the target. Stop writes, confirm zero CDC latency in the CloudWatch metric, and treat that as the cutover moment. *Cost/cleanup:* RDS and Aurora bill per hour, and DMS Serverless bills by capacity used; run this for an hour or two, then delete the replication, both databases and their snapshots.
2. **DMS Schema Conversion assessment.** Create an instance profile, two data providers (for example SQL Server Express on RDS as source and Aurora PostgreSQL as target) and a migration project. Load a sample schema with a few stored procedures and run the assessment report. Read which objects need manual action, then export the converted SQL to S3. *Cost/cleanup:* Check DMS Schema Conversion pricing first; the RDS and Aurora databases are the main cost. Delete the project, data providers, databases and the S3 export.

### Skill 4.1.4: Replatforming to managed services

**Key concepts**

- Replatform means "lift, tinker and shift": change the platform under the application without changing its architecture. Typical moves: self-managed database on EC2 to RDS or Aurora, app server to ECS on Fargate, cron box to EventBridge Scheduler plus Lambda, self-managed queue to Amazon MQ (same protocols) or SQS (code change).
- Pick by what the team must stop doing. RDS and Aurora remove patching, backups, Multi-AZ failover and minor upgrades. Fargate removes host management. Lambda removes servers entirely but adds the 15-minute limit and event-driven design.
- RDS vs Aurora: RDS for engine and version flexibility (including Oracle and SQL Server, with license-included or BYOL for Oracle). Aurora for MySQL or PostgreSQL workloads that need fast failover, up to 15 low-latency read replicas, storage that grows on its own, Global Database or Serverless v2.
- Amazon MQ (ActiveMQ or RabbitMQ) is the replatform answer when the application uses JMS, AMQP, MQTT or STOMP and must not change code. SQS and SNS need code changes.
- Windows workloads: AWS Transform for .NET ports .NET Framework applications to cross-platform .NET (8 or later) so they can run on Linux containers, which removes Windows licensing cost. That is a replatform with code transformation.
- Containerize without code access: AWS Transform can containerize from source code; App2Container-style tooling needs access to the running app. If neither is possible, rehost first.
- Elastic Beanstalk still exists for simple replatforms of web apps with little platform skill, but ECS on Fargate is the more common exam answer for containerized targets.

**Exam traps**

- Choosing Lambda for a long-running batch job that runs for hours. The 15-minute timeout breaks it; use ECS on Fargate, AWS Batch or Step Functions.
- Moving to Aurora when the application depends on Oracle-specific features and the deadline does not allow conversion. RDS for Oracle is the replatform.
- Choosing EC2 with self-managed MySQL for "reduce operational overhead". That is a rehost and keeps the patching and backup work.
- Choosing SQS to replace ActiveMQ when the requirement says "no code changes".

**Hands-on**

1. **Database replatform with a snapshot path.** Run MySQL in a container on a small EC2 instance, load sample data, take a `mysqldump`, and import it into an RDS for MySQL instance with Multi-AZ turned off for cost. Then trigger a reboot with failover on a Multi-AZ test copy if budget allows, and observe the DNS endpoint staying the same. This proves what the managed service takes off your hands. *Cost/cleanup:* RDS Single-AZ db.t4g.micro may be free tier eligible; Multi-AZ doubles cost. Delete the instance without a final snapshot and terminate EC2.
2. **App server to ECS on Fargate.** Containerize a simple web app, push it to Amazon ECR, and create an ECS service on Fargate behind an ALB with two tasks across two AZs. Stop one task and watch ECS replace it. *Cost/cleanup:* ALB and Fargate bill hourly; avoid NAT gateways by using public subnets with public IPs for the lab. Delete the service, cluster, ALB, target group and ECR repository.

### Skill 4.1.5: Refactoring: microservices, serverless and incremental modernization (strangler fig)

**Key concepts**

- **Strangler fig**: put a routing layer in front of the monolith, move one capability at a time to a new service, and route that path to the new service. The monolith shrinks until it can be retired. You reduce risk because each step is small and reversible.
- Routing layer options: Amazon API Gateway (path-based routes to Lambda, HTTP backends or VPC links to private ALBs or NLBs), an ALB with path or host rules, or CloudFront with multiple origins and cache behaviors. Use API Gateway when you also need throttling, API keys, authorizers or request transformation. Use ALB rules when everything is inside the VPC and HTTP routing is all you need.
- Do not build new designs on AWS Migration Hub Refactor Spaces, which sits under Migration Hub (closed to new customers). Build the strangler routing yourself with API Gateway, ALB and IaC.
- Data is the hard part. Options: the new service owns a new database and syncs with the monolith through events (change data capture with DMS into Kinesis or MSK, or the outbox pattern); or the new service calls the monolith's data through an anti-corruption layer API. Avoid two services writing the same tables.
- Decomposition: split by business capability or bounded context, not by technical layer. Each service owns its data. Use asynchronous messaging (EventBridge, SNS plus SQS) between services to reduce temporal coupling; use Step Functions for multi-step business processes with compensation (saga).
- Serverless adoption: Lambda plus API Gateway plus DynamoDB for spiky, event-driven parts; keep steady, long-running parts on containers.
- Branch by abstraction and feature flags (AWS AppConfig) let you switch traffic between old and new code paths safely.
- AWS Transform custom can automate repetitive code modernization across many repositories (language version upgrades, AWS SDK v1 to v2, x86 to Graviton, framework upgrades), which supports incremental refactoring at scale.

**Exam traps**

- A "big bang" rewrite when the requirement says "reduce risk" or "deliver value incrementally". The strangler fig is the expected answer.
- Splitting the monolith into services that still share one database schema. That is a distributed monolith; coupling stays, and deployments still block each other.
- Routing with Route 53 weighted records to do the strangling. DNS cannot route by URL path, so it cannot send `/orders` to a new service and `/billing` to the monolith.
- Choosing synchronous chains of Lambda calls for a long business process. Use Step Functions for orchestration and error handling.

**Hands-on**

1. **Strangler fig with API Gateway.** Deploy a "monolith" (a small container on ECS or an EC2 instance behind an ALB). Create an HTTP API in API Gateway that proxies all paths to the ALB. Then write a Lambda function for `/orders`, add a more specific route for it, and confirm that `/orders` now hits Lambda while all other paths still reach the monolith. Remove the route to roll back. *Cost/cleanup:* HTTP API and Lambda are free tier friendly; the ALB bills hourly. Delete the API, function, ALB and compute.
2. **Event-driven data sync between old and new.** Have the monolith publish an event to an EventBridge custom bus whenever it writes an order (simulate with a script). Add a rule that targets a Lambda function writing to a DynamoDB table owned by the new service. Replay a few events and check the table. This shows how a new service builds its own data store without sharing the monolith's schema. *Cost/cleanup:* Within free tier for small volumes; delete the bus, rule, function and table.

### Skill 4.1.6: Modernizing legacy workloads: mainframes, applications and databases

**Key concepts**

- **AWS Transform for mainframe** handles IBM z/OS (and Fujitsu GS21) modernization: codebase analysis of COBOL, PL/I, JCL and related artifacts, dependency mapping and missing-artifact detection, business rule extraction and documentation, automated refactoring to Java (or C#), data migration of VSAM, IMS DB and Db2 data to cloud databases, and automated equivalence testing to prove functional parity. A "reimagine" workflow extracts business rules into requirements, then generates cloud-native code with traceability back to the source.
- Mainframe patterns and when to use them:
  - **Refactor** (automated conversion to Java): when the goal is to leave the mainframe skill set and license, and keep business logic as is.
  - **Replatform** (run COBOL on an emulation or rehosting runtime on AWS): when the code must stay COBOL for now and speed matters more than modernizing.
  - **Reimagine**: when the business wants new capabilities and the old code is only a source of rules.
  - **Augment**: keep the mainframe and replicate data to AWS for analytics or new digital channels (for example, CDC from Db2 into AWS). Choose this when the mainframe cannot move yet but new apps need the data.
- **.NET and Windows**: AWS Transform for .NET upgrades .NET Framework applications to cross-platform .NET 8 or later, can upgrade UI frameworks (for example Web Forms to Blazor), moves SQL Server databases to Aurora PostgreSQL or other engines, and converts IIS deployments to containers on ECS or EKS. The main business driver is removing Windows and SQL Server licensing.
- **VMware estates**: AWS Transform migrations converts VMware networks to VPCs and transit gateways, generates landing zone configuration (Landing Zone Accelerator, CloudFormation, CDK or Terraform), and rehosts through AWS Transform MGN or replatforms to containers. It can also move block storage to Amazon EBS or FSx for NetApp ONTAP.
- **Database modernization**: commercial to open source (Oracle or SQL Server to Aurora PostgreSQL) uses DMS Schema Conversion or SCT plus DMS. Analytics workloads move to Redshift. Key-value hot paths can move to DynamoDB, which is a refactor.
- **Code modernization at scale**: AWS Transform custom runs built-in transformations (Java, Python and Node.js upgrades, AWS SDK v1 to v2, x86 to Graviton, Log4J to SLF4J, framework upgrades) and natural-language custom transformations across many repositories from the CLI or pipelines.
- Every AWS Transform job is human-in-the-loop: the agent raises collaborator requests for approvals and decisions. Design for that review step in the plan; it is not "fire and forget".

**Exam traps**

- A manual COBOL rewrite by a large team when the requirement is "preserve business logic, shortest timeline, least specialized skill". Automated refactoring with equivalence testing fits better.
- Moving the mainframe when the requirement only asks for "make mainframe data available to a new mobile app quickly". Augment with data replication; do not migrate the whole system.
- Lifting .NET Framework applications into Linux containers without porting. .NET Framework needs Windows; you must port to modern .NET first, or use Windows containers.
- Treating generated code as production-ready without testing. Equivalence testing is part of the design.

**Hands-on**

1. **Run an AWS Transform .NET or custom transformation on a sample repo.** Clone a small public .NET Framework sample (or a Java 8 sample for AWS Transform custom). Start a transformation job from the AWS Transform web console or CLI, review the plan, answer the agent's collaborator requests, and inspect the diff it produces. Build the result locally and run its tests. *Cost/cleanup:* Check current AWS Transform pricing for the transformation type before starting; delete the workspace and any connectors afterwards.
2. **Mainframe modernization without a mainframe.** Download the AWS CardDemo open-source mainframe sample application (COBOL, JCL, VSAM definitions). Create an AWS Transform for mainframe job, upload the source to S3, and run the analysis step only. Review the dependency graph, missing artifacts and generated business rule documentation. *Cost/cleanup:* Small S3 storage cost; delete the bucket and job. Stop before the full refactor step if you only want to see the analysis output.

### Practice questions: Task 4.1

**Question 4.1.1** A regional bank runs 1,400 VMs on VMware vSphere across two data centers. The bank signed with AWS in early 2026 and has never used AWS migration tooling. The CIO wants a business case with right-sized EC2 recommendations and a dependency-based wave plan within eight weeks. The security team will not allow any software that sends data to the internet without review, and installing agents on every guest is not approved. The migration team is small and wants the least operational overhead. Which solution meets these requirements?

- A. Deploy the AWS Application Discovery Service Agentless Collector in vCenter, view the data in the AWS Migration Hub console, and use Migration Hub Strategy Recommendations to group applications into waves.
- B. Export an RVTools inventory from vCenter, load it into an AWS Transform assessment to build the business case, and group servers into waves by business owner and VLAN.
- C. Deploy the AWS Transform discovery tool OVA in vCenter with a read-only account and WinRM or SSH access to guests, review the export locally, upload it to AWS Transform, and use AWS Transform for the assessment and wave plan.
- D. Install the AWS Systems Manager Agent on all VMs as hybrid managed nodes, collect Systems Manager Inventory, and build waves in Amazon QuickSight from the inventory data.

<details>
<summary>Answer and explanation</summary>

**Correct answer: C**

**Why C is correct:** The AWS Transform discovery tool is the replacement for the ADS Agentless Collector. It runs as an appliance in vCenter, so no agents go on guests, and it collects utilization data for right-sizing plus network connections for dependency mapping. It needs no outbound internet connection, and the data stays on the appliance until the team exports and uploads it, which satisfies the security review. AWS Transform then builds the assessment and the dependency-based wave plan as a managed workflow, which keeps overhead low for a small team.

**Why the other options are incorrect:**

- **A:** Application Discovery Service and Migration Hub closed to new customers on 7 November 2025. A customer starting in 2026 cannot sign up, so this design cannot be built.
- **B:** RVTools gives inventory and allocation data, which is enough for a quick business case, but it contains no network connection data. Waves grouped by owner and VLAN miss real application dependencies, so the plan does not meet the "dependency-based" requirement.
- **D:** SSM Agent is an agent on every guest, which is not approved. Systems Manager Inventory also collects software and configuration data, not network dependency graphs, and building waves in QuickSight is custom work.

**Key takeaway:** For new migrations, discovery and wave planning go through AWS Transform and its agentless discovery tool, not Migration Hub or Application Discovery Service.

</details>

**Question 4.1.2** A logistics company must exit a colocation facility in 10 weeks when its contract ends. It has 220 Linux and Windows servers on a mix of VMware and physical hardware. The applications are commercial off-the-shelf products, and the vendors do not support code or platform changes. The maximum allowed downtime per application is 2 hours during a weekend window. A 10 Gbps AWS Direct Connect connection is already in place, and security requires that migration traffic does not cross the internet. Which solution meets these requirements?

- A. Use AWS Transform MGN with the replication agent on each server, replicate over Direct Connect to a staging subnet using private connectivity, launch test instances for each wave, and cut over during the weekend windows.
- B. Use VM Import/Export to convert VMware images to AMIs, copy the images over Direct Connect to Amazon S3, and launch EC2 instances from the imported AMIs during each weekend window.
- C. Use AWS Elastic Disaster Recovery to replicate all servers over Direct Connect, then run a recovery job for each wave and keep the source servers registered for failback.
- D. Use AWS Transform to containerize each application from its source code and deploy the containers to Amazon ECS on AWS Fargate, with an ALB in front of each service.

<details>
<summary>Answer and explanation</summary>

**Correct answer: A**

**Why A is correct:** AWS Transform MGN replicates block-level changes continuously while the source servers run, so the cutover window only needs the time to stop the app, let the final changes sync, and launch the target. That fits a 2-hour window. It works for both VMware and physical servers because the agent works at the OS level. Replication can use private connectivity over Direct Connect, so traffic stays off the internet. Test launches per wave reduce the risk of a failed cutover.

**Why the other options are incorrect:**

- **B:** VM Import/Export is a one-time image copy with no continuous replication. Large disks take longer than 2 hours to export, transfer and import, and it does not handle the physical servers.
- **C:** Elastic Disaster Recovery uses similar replication technology, but it is built for ongoing DR with failback, not one-way migration. The exam expects MGN for migration, and keeping DR replication running after the move adds cost and operational work.
- **D:** The vendors do not support platform changes, and these are COTS products without source code available to the company. Containerization from source code is not possible and would break vendor support.

**Key takeaway:** Short cutover windows plus "no changes" plus mixed VMware and physical sources means rehost with AWS Transform MGN and continuous replication.

</details>

**Question 4.1.3** An insurance company is moving a 6 TB Oracle 19c OLTP database to Amazon Aurora PostgreSQL to remove Oracle licensing. The database has about 900 PL/SQL packages. The DBA team has no Windows workstations and must not install desktop tools, by policy. Business requires the source to stay online during migration and allows a 30-minute cutover. The team wants an early estimate of how much code needs manual conversion before committing to the project. Which combination of steps meets these requirements?

- A. Use AWS SCT on an EC2 Windows instance to generate the assessment report and convert the schema, then use AWS DMS full load to copy the data during a scheduled 30-minute window.
- B. Use AWS DMS homogeneous data migration to copy the Oracle schema and data to Aurora PostgreSQL with native tools, and use ongoing replication to keep the target in sync until cutover.
- C. Use AWS DataSync to copy Oracle Data Pump export files to Amazon S3, convert the dump with DMS Schema Conversion, and import the result into Aurora PostgreSQL.
- D. Use DMS Schema Conversion to produce the assessment report and convert the schema, use its generative AI conversion for objects the rules cannot convert, and use AWS DMS full load plus CDC to keep Aurora PostgreSQL in sync until cutover.

<details>
<summary>Answer and explanation</summary>

**Correct answer: D**

**Why D is correct:** DMS Schema Conversion is managed and runs in the browser, so no desktop tool is needed. Its assessment report shows what converts automatically and what needs manual work, which gives the early effort estimate. For Oracle to PostgreSQL, generative AI conversion can handle more of the objects the rules engine cannot, which helps with 900 PL/SQL packages (in supported Regions, with human review). DMS full load plus CDC keeps the target current while the source stays online, so cutover is just stopping writes and letting CDC drain.

**Why the other options are incorrect:**

- **A:** A 6 TB full load does not fit in 30 minutes, and full load alone loses changes made during the copy. Running SCT on an EC2 Windows instance also works around the "no desktop tools" policy rather than meeting it.
- **B:** Homogeneous data migrations are for the same engine on both sides (for example PostgreSQL to PostgreSQL). Oracle to PostgreSQL is heterogeneous and needs schema conversion first.
- **C:** DMS Schema Conversion works against a live source database through a data provider, not against Data Pump files. This path also has no change capture, so it cannot keep downtime to 30 minutes.

**Key takeaway:** Heterogeneous database moves need schema conversion (DMS Schema Conversion when managed and supported, SCT otherwise) plus DMS full load and CDC for low downtime.

</details>

**Question 4.1.4** A retailer runs a 15-year-old order management monolith on Java and a shared Oracle database. Leadership wants to modernize toward microservices on AWS but will not accept a big-bang rewrite. Each new capability must go live on its own and be easy to roll back. The first capability to move is order tracking, which is read-heavy and spiky. The platform team wants managed services and least operational overhead. Which TWO actions should the solutions architect take? (Select TWO.)

- A. Put Amazon API Gateway in front of the monolith as the single entry point, then add a specific route that sends order tracking requests to the new service while all other paths still reach the monolith.
- B. Create Amazon Route 53 weighted records that send 10 percent of all traffic to the new order tracking service and 90 percent to the monolith, then increase the weight over time.
- C. Have the new order tracking service read and write the existing Oracle tables directly, so both systems always see the same data during the transition.
- D. Use AWS Migration Hub Refactor Spaces to create the routing environment and manage the proxy between the monolith and the new service.
- E. Build order tracking with AWS Lambda and Amazon DynamoDB, and feed DynamoDB from order change events captured from the monolith's database with AWS DMS CDC into Amazon Kinesis Data Streams.

<details>
<summary>Answer and explanation</summary>

**Correct answer: A, E**

**Why A and E are correct:** A implements the strangler fig: one routing layer in front of the monolith, with path-based routes that move one capability at a time. Rollback is removing the route. E gives the new service its own data store, which suits a read-heavy, spiky pattern with Lambda and DynamoDB scaling on demand. DMS CDC into Kinesis streams the monolith's changes to the new store without the new service touching the shared schema, so the two systems stay decoupled.

**Why the other options are incorrect:**

- **B:** Route 53 weighted routing splits traffic by DNS name, not by URL path. It cannot send only order tracking calls to the new service; it would send a share of every request type, including ones the new service cannot handle.
- **C:** Sharing the same tables couples the two systems at the data layer. Schema changes in one break the other, and you end up with a distributed monolith that cannot be released on its own.
- **D:** Refactor Spaces is part of AWS Migration Hub, which closed to new customers on 7 November 2025. A new design should build the routing with API Gateway or an ALB instead.

**Key takeaway:** Strangler fig equals path-based routing at a facade plus a new service that owns its data and syncs through events.

</details>

**Question 4.1.5** A financial services firm runs core batch and online processing on an IBM z/OS mainframe in COBOL, JCL and VSAM. The mainframe contract expires in 30 months. Most of the original developers have retired, and documentation is incomplete. The firm wants to move off COBOL to a language its current Java developers can maintain, keep business logic the same, and prove to auditors that the new system produces the same results as the old one. Which approach meets these requirements with the least risk?

- A. Rehost the COBOL code on a mainframe emulation runtime on Amazon EC2, keep the VSAM files on Amazon EFS, and train the Java team in COBOL over the contract period.
- B. Use AWS Transform for mainframe to analyze the code base and extract business rules, refactor the COBOL to Java, migrate VSAM data to a managed database, and run automated equivalence testing against the mainframe outputs.
- C. Replicate VSAM and Db2 changes to Amazon S3 with CDC, build new Java microservices from the replicated data over time, and keep the mainframe running for core processing.
- D. Have the Java team rewrite each COBOL program by hand from interviews with business users, and use AWS Step Functions to orchestrate the new batch jobs that replace the JCL.

<details>
<summary>Answer and explanation</summary>

**Correct answer: B**

**Why B is correct:** AWS Transform for mainframe analyzes COBOL, JCL and related artifacts, finds dependencies and missing pieces, and documents business rules, which fills the documentation gap. It refactors the code to Java so the current team can maintain it, keeping the business logic as is. It also migrates VSAM data to cloud databases. Automated equivalence testing compares outputs between the old and new systems, which gives auditors evidence of functional parity.

**Why the other options are incorrect:**

- **A:** Replatforming COBOL to an emulator gets off the hardware but keeps COBOL, which fails the "language the Java team can maintain" goal. Training Java developers in COBOL does not fix the skills risk.
- **C:** This is the augment pattern. It is valid when the mainframe must stay, but the firm needs to exit the mainframe in 30 months, and this keeps core processing on it.
- **D:** A manual rewrite from interviews is the highest-risk path. With incomplete documentation, business rules will be missed, and there is no systematic way to prove parity.

**Key takeaway:** To leave COBOL, keep business logic and prove parity, use automated analysis, refactoring and equivalence testing (AWS Transform for mainframe), not a manual rewrite or emulation.

</details>

## Task 4.2: Design disaster recovery architectures

This task tests whether you can turn an RTO, RPO and budget into one of the four DR strategies, and then pick the replication and failover mechanisms that actually deliver those numbers. The exam keeps asking for the cheapest strategy that still meets the stated RTO and RPO, and for a failover path that does not depend on the failed Region.

### Skill 4.2.1: DR solutions that meet RTO, RPO and cost

**Key concepts**

- RPO is how much data you can lose (set by replication or backup frequency). RTO is how long you can be down (set by how much has to be built, started, scaled and repointed after the event). Treat them separately: a design can have near-zero RPO and a long RTO.
- The four strategies, from cheapest to most expensive:
  - **Backup and restore**: copy backups to the DR Region (AWS Backup cross-Region copy, S3 replication). RPO equals backup frequency (hours, or minutes with continuous backups). RTO is hours, because you rebuild infrastructure from IaC and restore data.
  - **Pilot light**: data is replicated live (Aurora Global Database, DynamoDB global tables, S3 CRR, DRS), core infrastructure is defined but compute is off or not deployed. RPO seconds to minutes; RTO tens of minutes to hours, mostly the time to deploy and scale compute.
  - **Warm standby**: a smaller but fully working copy runs in the DR Region and takes test traffic. RPO seconds; RTO minutes, because you only scale up and shift traffic.
  - **Multi-site active/active**: full capacity serves traffic in two or more Regions. RPO near zero (depends on the data store's replication); RTO near zero. Most expensive and hardest, because the data layer must handle writes in several Regions.
- Decision rule: start from RTO. Hours means backup and restore. Under an hour with a small budget means pilot light. Minutes means warm standby. Near zero means active/active. Then check that the data replication meets RPO.
- Within one Region, Multi-AZ designs protect against AZ failure. Cross-Region DR protects against Region-wide events and is often required by regulators. Do not pay for multi-Region if the requirement is only "survive an AZ failure".
- IaC is part of DR. CloudFormation templates or CDK apps for the DR Region, plus StackSets for many accounts, are what make backup and restore or pilot light RTOs repeatable.
- Data store replication speeds to know: Aurora Global Database replicates storage with typical lag under a second, and failover to a secondary Region is typically under a minute. DynamoDB global tables replicate asynchronously, usually within a second or so, with last writer wins on conflicts. S3 Replication Time Control gives an SLA-backed replication time for most objects within 15 minutes.
- Service quotas in the DR Region are part of the plan. A pilot light that needs 200 instances fails if the DR account's EC2 quota is lower. Raise quotas ahead of time.

**Exam traps**

- Choosing active/active when the stated RTO is several hours and cost is the priority. Backup and restore meets it at a fraction of the cost.
- Choosing backup and restore for an RPO of seconds. Daily or hourly backups cannot meet it; you need continuous replication.
- Confusing RDS cross-Region read replicas with Aurora Global Database. Both work, but promoting an RDS read replica is a manual or scripted operation and breaks replication; Aurora Global Database has managed switchover and failover with lower lag.
- Assuming Auto Scaling will deliver capacity in the DR Region during a Region-wide event. Capacity in the target Region is not guaranteed; warm standby or capacity reservations reduce the risk.

**Hands-on**

1. **Measure backup-and-restore RTO.** Write a CloudFormation template for a small stack (VPC, one EC2 instance, an RDS for MySQL db.t4g.micro). Take an AWS Backup on-demand backup of the instance and database and copy it to a second Region. In the second Region, deploy the template and restore the database from the copied recovery point. Time each step; that number is your real RTO for this strategy. *Cost/cleanup:* Small RDS and EC2 costs plus cross-Region data transfer for the copy; delete stacks, recovery points in both vaults and restored resources.
2. **Pilot light with DynamoDB global tables.** Create a DynamoDB table with on-demand capacity and add a replica in a second Region. Deploy a Lambda function and API Gateway API in both Regions from the same template, but leave the DR API unused. Write items in Region A, read them in Region B, and note the replication delay. *Cost/cleanup:* On-demand DynamoDB, Lambda and API Gateway are cheap at lab volume; replicated writes are billed in each Region. Delete the replica, then the table, the APIs and functions.

### Skill 4.2.2: Continuous replication with AWS Elastic Disaster Recovery

**Key concepts**

- **AWS Elastic Disaster Recovery (AWS DRS)** replicates servers continuously at the block level from physical servers, VMware vSphere, Microsoft Hyper-V, other clouds, or EC2 in another Region or AZ into a low-cost staging area in your AWS account. It is the recommended DR service for server-based workloads, including databases such as Oracle, SQL Server and MySQL running on those servers, and SAP.
- What you pay for while idle: lightweight replication servers and low-cost EBS storage in the staging subnet, plus the per-server DRS fee. Full-size recovery instances are launched only during drills or real recovery. That is why DRS is the cost-effective answer for "RPO of seconds, RTO of minutes" for many servers without running a warm copy.
- **Point-in-time recovery**: DRS keeps a series of recovery snapshots, so you can recover to a state before corruption or ransomware, not only to "now". This matters when the latest replicated state is already damaged.
- **Drills** launch recovery instances from current or past points in time without stopping replication or touching the source. Run them regularly; they are how you prove the RTO.
- **Failback**: after the event, DRS replicates changes from the recovery instances back to the original source environment (on premises with the failback client, or to the original Region for cross-Region DR), then you fail back. Cross-Region and cross-AZ failback are supported.
- Launch settings: launch templates control instance type, subnet, security groups and IP. Post-launch actions can run Systems Manager documents after recovery (for example, update DNS or rejoin a domain).
- DRS vs MGN: same replication approach, different job. MGN is a one-way migration with cutover and finalize. DRS is ongoing protection with drills, point-in-time recovery and failback.
- DRS vs AWS Backup: Backup gives scheduled, policy-driven recovery points (RPO in hours, or minutes with continuous backup for supported services). DRS gives seconds of RPO for whole servers.

**Exam traps**

- Using AWS Backup for an RPO of seconds on on-premises servers. Backup is scheduled; it cannot meet that RPO.
- Running a full-size warm standby fleet of EC2 instances replicated with custom scripts when DRS can keep compute off until recovery. That fails "most cost-effective".
- Recovering to the latest point after ransomware. If encryption has already replicated, you must pick an earlier point-in-time snapshot.
- Forgetting failback. The question often says "return to on premises after the event"; DRS has built-in failback, VM Import does not.

**Hands-on**

1. **Cross-Region DR for an EC2 instance.** In Region B, initialize AWS DRS and accept the default replication settings template. Install the DRS replication agent on a small instance in Region A that runs a simple web page. After initial sync, run a drill: launch a recovery instance from the latest point, check the page, and confirm replication stayed healthy during the drill. *Cost/cleanup:* DRS charges per source server per hour, plus staging EC2 and EBS and the drill instance; terminate the drill instance, disconnect and delete the source server in DRS, then remove the agent.
2. **Point-in-time recovery after "corruption".** On the protected source instance, write a file with known content, wait for a recovery snapshot, then overwrite the file. Launch a drill from the earlier point in time and confirm the original content is there. This proves why point-in-time recovery matters for ransomware. *Cost/cleanup:* Same as above; terminate drill instances right away.

### Skill 4.2.3: DR assessment and recovery procedures (AWS Resilience Hub, Systems Manager runbooks)

**Key concepts**

- **AWS Resilience Hub** assesses an application's resilience against targets you set. In the original model, you define an application from sources such as CloudFormation stacks, Terraform state, AWS Resource Groups, myApplications or EKS clusters, attach a **resiliency policy** with RTO and RPO targets for each disruption type (application, infrastructure, AZ, Region), and run an assessment. It estimates the workload's RTO and RPO, reports whether the policy is met, and recommends changes. Operational recommendations include CloudWatch alarms, standard operating procedures (SOPs) and FIS experiments. Scheduled assessments and drift detection tell you when a change breaks compliance with the policy.
- The **next generation of Resilience Hub** (generally available May 2026, with the earlier experience still available to existing customers) changes the model: "applications" become **services**, grouped into **systems**, with **user journeys** across them. It discovers dependencies automatically (AWS services, internal and third-party endpoints), uses generative AI for failure mode assessments against Well-Architected and the AWS Resilience Analysis Framework, supports modular policies that combine DR, availability SLO and data recovery conditions, and integrates with AWS Organizations through a delegated administrator. Later 2026 additions include policy sharing across the organization and dependency insights that flag cross-Region and new dependencies.
- **Systems Manager Automation runbooks** are the executable form of recovery procedures: promote a read replica, update a Route 53 record, scale an Auto Scaling group, restore from a recovery point. Runbooks give you approvals (the `aws:approve` step), rate control and cross-account and cross-Region execution. They remove manual steps, which shortens RTO and reduces error under stress.
- Resilience Hub SOP recommendations map to Systems Manager documents you can deploy and run. Resilience Hub can also publish findings to EventBridge so you can automate a response when an assessment fails a policy.
- Systems Manager Incident Manager is closed to new customers, so do not choose it for new incident response designs. Use EventBridge, CloudWatch alarms, Systems Manager Automation and your own paging tool instead.

**Exam traps**

- Using AWS Trusted Advisor or Well-Architected Tool reviews as the way to verify that an application meets its RTO and RPO. They give general checks; Resilience Hub assesses against the stated targets.
- Writing recovery procedures as wiki pages. The exam prefers automation runbooks that can be tested and run with one action.
- Using Incident Manager response plans for a new design. It is closed to new customers.
- Assuming a passing Resilience Hub assessment means DR works. Assessment is analysis; you still need tests (Skill 4.2.5).

**Hands-on**

1. **Assess a CloudFormation stack with Resilience Hub.** Deploy a small stack with an ALB, an Auto Scaling group in one AZ and a Single-AZ RDS instance. In Resilience Hub, add the stack as an application (or as a service in the next generation experience), attach a policy that requires AZ disruption RTO of 1 hour, and run an assessment. Read why it fails, apply the recommendations (second AZ, Multi-AZ RDS), redeploy and assess again. *Cost/cleanup:* Check Resilience Hub pricing for the experience you use; RDS Multi-AZ and the ALB bill hourly. Delete the application in Resilience Hub and the stack.
2. **Write a recovery runbook.** Create a Systems Manager Automation document with three steps: an `aws:approve` step that waits for an approver, an `aws:executeAwsApi` step that calls `PromoteReadReplica` on an RDS read replica, and an `aws:executeAwsApi` step that updates a Route 53 record to the new endpoint. Run it against a lab replica. *Cost/cleanup:* Automation steps are low cost; the RDS instances bill hourly. Delete the replica, the primary and the hosted zone records.

### Skill 4.2.4: Multi-Region DR with Route 53 health checks, Global Accelerator and Amazon Application Recovery Controller

**Key concepts**

- **Route 53 failover routing** with health checks: primary and secondary records; when the primary's health check fails, Route 53 answers with the secondary. Health checkers run from locations on the internet, so they cannot reach private endpoints directly. For private resources, create a health check based on a CloudWatch alarm. **Calculated health checks** combine child checks (for example, "healthy if at least 2 of 3 are healthy").
- Route 53 health checks and DNS answers are data plane operations and keep working during events. Changing records or health check configuration uses the control plane. Design failover so it does not depend on editing records during the event.
- DNS failover is slowed by TTLs and client caching. Use low TTLs on failover records; alias records to AWS resources use the target's TTL behavior.
- **AWS Global Accelerator**: two static anycast IPv4 addresses (dual-stack is also available) that route over the AWS network to endpoint groups in one or more Regions. Health checks and **traffic dials** per endpoint group, plus endpoint weights, decide where traffic goes. Failover does not depend on DNS caching, so it is faster and more predictable for clients that cache DNS. It also supports TCP and UDP, which CloudFront does not cover for non-HTTP traffic. Choose it when clients need fixed IPs (firewall allow lists) or fast, DNS-independent failover.
- **Amazon Application Recovery Controller (ARC)** has several features:
  - **Routing controls**: on/off switches wired to Route 53 health checks. You flip them through a highly available cluster data plane with endpoints in several Regions, so failover does not need the Route 53 control plane. **Safety rules** (assertion rules and gating rules) stop you from turning off all cells at once.
  - **Zonal shift**: move traffic away from one AZ for a supported resource (ALB, NLB, EC2 Auto Scaling groups, Amazon EKS). A manual shift lasts up to three days initially and can be extended or cancelled. It works for ALB and NLB with cross-zone load balancing on or off, but if several load balancers share targets, a shift on a cross-zone enabled load balancer reduces target capacity for all of them. A load balancer in fail-open state ignores the shift.
  - **Zonal autoshift**: you authorize AWS to shift traffic away from an AZ automatically when AWS telemetry detects an impairment. It requires a **practice run configuration**: AWS shifts the resource away from one AZ for about 30 minutes each week, judged by a required CloudWatch **outcome alarm** (ALARM means FAILED); an optional **blocking alarm** prevents or interrupts runs, and you can set allowed or blocked windows. Pre-scale all AZs, because autoshift does not wait for scaling.
  - **Region switch**: a managed multi-Region recovery orchestrator. A plan (active/passive or active/active) has workflows made of execution blocks, such as EC2 Auto Scaling, EKS and ECS scaling, Aurora Global Database failover or switchover, DocumentDB global cluster failover, ARC routing controls, Route 53 health checks, manual approval, custom Lambda actions and child plans. Plans run from the Region being activated, so the impaired Region is not needed. Region switch evaluates plans every 30 minutes for permission and resource problems, writes plan execution reports to S3 (with recovery timeline and RTO calculation), supports cross-account resources through roles, and can be triggered by CloudWatch alarms or manually. It is billed per plan per month.
- Choosing: Route 53 failover alone for simple, cost-sensitive DR. Global Accelerator for static IPs and fast, DNS-independent failover. ARC routing controls when humans must make the Region failover decision with a highly available switch. Region switch when the failover involves ordered steps across databases, compute scaling and traffic, and you want one tested, audited plan.
- Data plane over control plane: AWS guidance is to fail over with data plane actions (health checks, routing controls, Global Accelerator dials) because control planes can be impaired during large events.

**Exam traps**

- Updating Route 53 records through the API as the failover step. That uses the Route 53 control plane, which can be the thing that is impaired.
- Using a standard Route 53 health check against a private IP or an internal ALB. Health checkers cannot reach it; use a CloudWatch alarm-based health check.
- Expecting DNS failover to be instant for clients that cache DNS for a long time. Global Accelerator avoids that.
- Using zonal shift for a Region-level event. Zonal shift moves traffic between AZs in one Region; Region-level failover needs Route 53, Global Accelerator, routing controls or Region switch.
- Enabling zonal autoshift without pre-scaled capacity. It does not wait for Auto Scaling, so the remaining AZs can be overloaded.

**Hands-on**

1. **Route 53 failover between two Regions.** Deploy a static website on an S3 bucket or a small Lambda function URL in two Regions. In a hosted zone you own, create failover records with a health check on the primary endpoint. Break the primary (change the function to return 500 or block access) and watch the health check go unhealthy and `dig` return the secondary. *Cost/cleanup:* A hosted zone costs a small monthly fee, and health checks bill per month; delete records, health checks and the zone if it was made for the lab.
2. **Zonal shift on an ALB.** Deploy an ALB with targets in two AZs. Start a zonal shift away from one AZ with a short expiry from the ARC console or `aws arc-zonal-shift start-zonal-shift`. Check the ALB's per-AZ request metrics in CloudWatch to see traffic drain. Then configure a practice run with an outcome alarm on target 5xx errors, but do not enable autoshift unless you want weekly runs. *Cost/cleanup:* Zonal shift has no extra charge as far as the docs show; the ALB and instances bill hourly. Cancel the shift, delete the practice run configuration, ALB and instances. Region switch plans bill per plan per month, so read the Region switch docs instead of creating one unless you need to.

### Skill 4.2.5: DR testing and validation (AWS Resilience Hub)

**Key concepts**

- A DR plan is unproven until it is tested. Test types: tabletop walkthrough, component tests (restore a backup, fail over a database), and full failover (switch production traffic to the DR Region). The exam prefers automated, repeatable tests on a schedule.
- **Resilience Hub** links assessment to testing. It recommends FIS experiments tailored to the application; running them, or your own tests, can raise the resiliency score. The next-generation experience adds pre-configured resilience tests built on AWS FIS, with execution roles for single-account and multi-account tests.
- **AWS FIS** runs the faults (Task 4.3 covers it in depth). **ARC zonal autoshift practice runs** test AZ evacuation every week. **ARC Region switch** gives plan evaluation every 30 minutes plus execution reports with a measured recovery timeline, which is evidence for auditors.
- **AWS DRS drills** launch recovery instances without stopping replication. **AWS Backup restore testing** restores recovery points on a schedule and can validate them, proving that backups are usable.
- **Aurora Global Database switchover** (planned, no data loss) lets you test Region role change without losing data; use it for scheduled DR tests, and keep managed failover (which can lose recent data) for real events.
- Measure, do not estimate: record time to detect, time to decide, time to fail over and time to recover service, and compare to RTO. Record data lost and compare to RPO.

**Exam traps**

- Testing DR only by reviewing architecture diagrams or running Resilience Hub assessments. Assessments estimate; only tests measure.
- Running DR tests by failing over production databases with unplanned failover. Use planned switchover for tests to avoid data loss.
- Testing only once a year by hand. The exam expects scheduled automated tests.

**Hands-on**

1. **Restore testing with AWS Backup.** Create a backup plan for a small EBS volume or DynamoDB table. Create a restore testing plan that restores the latest recovery point weekly, and add a Lambda function triggered by the restore job event that checks the restored resource and reports a validation result. Check the restore job status and duration. *Cost/cleanup:* You pay for backup storage and for restored resources while they exist; restore testing deletes them after the retention you set. Delete the plans and recovery points.
2. **Run a Resilience Hub recommended experiment.** For the application from Skill 4.2.3, open the testing recommendations, create the suggested FIS experiment template (for example, terminate instances in one AZ of the Auto Scaling group), add a stop condition on an ALB 5xx alarm, and run it. Re-run the assessment and see the score change. *Cost/cleanup:* FIS bills per action-minute; the stack resources bill hourly. Delete the experiment template and the stack.

### Skill 4.2.6: Multi-cluster container strategies for HA and DR (Amazon EKS, Amazon ECS)

**Key concepts**

- An EKS or ECS cluster is a Regional resource. For Region-level DR you run separate clusters in each Region and route traffic to them. Inside one Region, spread nodes or tasks across at least three AZs for AZ resilience.
- **Stateless first**: keep containers stateless and put state in replicated data stores (Aurora Global Database, DynamoDB global tables, S3 CRR, ElastiCache Global Datastore). Then a DR cluster only needs the same images and configuration.
- **Images**: enable Amazon ECR cross-Region replication (and cross-account if needed) so images exist in the DR Region even if the primary Region's ECR is unavailable. Pull through cache rules do not replace replication for DR.
- **Configuration**: deploy every cluster from the same source with GitOps (for example Argo CD or Flux on EKS) or the same CloudFormation, CDK or Terraform code for ECS. A cluster you cannot rebuild from code is a single point of failure.
- **Traffic**: Route 53 failover or latency records, Global Accelerator endpoint groups, or ARC routing controls in front of the per-Region load balancers (ALB or NLB created by the AWS Load Balancer Controller on EKS, or by ECS services).
- **Patterns**: active/passive (DR cluster scaled down; Region switch has ECS and EKS scaling execution blocks to scale it up during recovery), active/active (both clusters serve traffic; needs multi-Region data). Blue/green cluster upgrades on EKS also use two clusters with traffic shifting.
- **Within a Region**: ARC zonal shift works for EKS, and topology spread constraints or pod anti-affinity keep replicas across AZs. For ECS, the service scheduler spreads tasks across AZs by default.
- Stateful workloads on Kubernetes (persistent volumes) need their own backup and replication, such as EBS snapshots copied cross-Region or EFS replication.

**Exam traps**

- Stretching one EKS cluster across Regions. EKS control planes are Regional; that design is not possible.
- Relying on the primary Region's ECR during a Region outage. Without replication, the DR cluster cannot pull images.
- Keeping session state in container memory. Failover to another cluster loses every session.
- Building the DR cluster by hand once and never updating it. Configuration drift breaks the failover; use GitOps or IaC.

**Hands-on**

1. **Two ECS clusters behind Route 53 failover.** Push an image to ECR in Region A and set up a replication rule to Region B. Deploy the same ECS on Fargate service with an ALB in both Regions from one CloudFormation template, with the Region B service at one task. Point Route 53 failover records at both ALBs with health checks, then set the Region A service to zero tasks and watch DNS fail over. *Cost/cleanup:* Two ALBs and Fargate tasks bill hourly; use public subnets to avoid NAT gateway charges. Delete both stacks, the replication rule, the images and the Route 53 records.
2. **EKS AZ evacuation.** If you accept the EKS control plane cost (it bills per hour per cluster), create a small cluster with managed nodes across three AZs, deploy an app with topology spread constraints, enable the cluster for ARC zonal shift, and shift away from one AZ. Watch the load balancer stop sending traffic to pods in that AZ. A cheaper substitute is the EKS workshop's zonal shift module in AWS Workshop Studio. *Cost/cleanup:* Delete the cluster, node groups and load balancers right after; leftover load balancers created by the controller keep billing.

### Practice questions: Task 4.2

**Question 4.2.1** A payments company runs a customer portal on Amazon EC2 behind an ALB in us-east-1, with an Aurora PostgreSQL cluster. A new regulation requires recovery in a second Region with an RPO of 1 minute and an RTO of 30 minutes. The CFO wants the lowest steady-state cost that still meets these numbers. The application is deployed with CloudFormation, and AMIs are built by a pipeline. Which DR strategy meets these requirements?

- A. Back up the Aurora cluster with AWS Backup every hour and copy the recovery points to us-west-2. During a disaster, deploy the CloudFormation stack in us-west-2 and restore the database from the latest copy.
- B. Add an Aurora Global Database secondary cluster in us-west-2 and copy the AMIs there. Keep the CloudFormation stack deployed with the Auto Scaling group at zero instances, and during a disaster fail over the database and scale the group to full capacity.
- C. Run the full application in both Regions behind AWS Global Accelerator, add an Aurora Global Database secondary with write forwarding, and serve users from both Regions at all times.
- D. Add an Aurora Global Database secondary cluster in us-west-2 and run a full-size copy of the production fleet there behind its own ALB at all times, with Route 53 failover records between the two ALBs.

<details>
<summary>Answer and explanation</summary>

**Correct answer: B**

**Why B is correct:** This is pilot light. Aurora Global Database keeps storage-level replication to the secondary Region with lag usually under a second, which meets a 1-minute RPO. The infrastructure is already deployed with zero instances, so recovery is a managed database failover plus scaling the Auto Scaling group from pre-copied AMIs, which fits inside 30 minutes if quotas in us-west-2 are raised ahead of time. Steady-state cost is the secondary cluster and idle infrastructure, far less than a full second fleet.

**Why the other options are incorrect:**

- **A:** Hourly backups give an RPO of up to an hour, which fails the 1-minute RPO. Restoring a large Aurora cluster and building the stack can also exceed 30 minutes.
- **C:** Active/active meets the numbers but runs full capacity in two Regions, which is the most expensive option and fails the lowest-cost requirement.
- **D:** The data layer is right, but a full-size fleet running all the time in the DR Region costs far more than needed for a 30-minute RTO. Scaling up from zero meets the RTO at lower cost.

**Key takeaway:** Minutes of RTO plus seconds of RPO at low cost points to pilot light with live data replication and compute that scales up at recovery.

</details>

**Question 4.2.2** A hospital group runs 60 on-premises Windows and Linux servers on VMware, including a SQL Server cluster and a file-based imaging system. The board wants DR to AWS with an RPO of seconds and an RTO under 1 hour. The group must be able to recover to a point before a ransomware infection and, after the event, return workloads to the data center. The IT team is small and has no deep application-specific DR skills. Which solution meets these requirements most cost-effectively?

- A. Use AWS Backup with the Backup gateway for VMware to take hourly VM backups, copy them to an AWS Region, and restore VMs as EC2 instances during a disaster.
- B. Use AWS Transform MGN to replicate all servers continuously and launch cutover instances during a disaster, then reinstall the agents on premises to move back.
- C. Use AWS Elastic Disaster Recovery to replicate all servers to a staging area, recover from a chosen point-in-time snapshot during a disaster, and use DRS failback to replicate back to VMware.
- D. Run a warm standby copy of every server on Amazon EC2 and replicate data with SQL Server Always On availability groups and Windows DFS Replication.

<details>
<summary>Answer and explanation</summary>

**Correct answer: C**

**Why C is correct:** AWS DRS replicates block-level changes continuously, giving an RPO of seconds, and launches recovery instances in minutes, so the 1-hour RTO is reachable. It keeps point-in-time recovery snapshots, so the team can recover to a state before the ransomware hit. Built-in failback replicates changes back to the source environment. Only lightweight replication servers and low-cost storage run all the time, and one process covers all applications, which suits a small team.

**Why the other options are incorrect:**

- **A:** Hourly backups give an RPO of up to an hour, not seconds. AWS Backup can restore VMware backups to on-premises VMware, but each restore is a whole-VM, point-in-time restore, not continuous replication with a managed failback workflow.
- **B:** MGN is built for one-way migration. After cutover and finalize, replication stops, and it has no failback workflow or drill model for ongoing DR.
- **D:** A full warm standby of 60 servers costs much more, and building and running Always On and DFS Replication needs application-specific skills the team does not have. It also does not give point-in-time recovery from ransomware for the imaging system.

**Key takeaway:** Server-based DR with seconds of RPO, point-in-time recovery and failback is AWS Elastic Disaster Recovery.

</details>

**Question 4.2.3** A trading platform runs active/passive across eu-west-1 and eu-central-1. Recovery needs several ordered steps: fail over an Aurora Global Database, scale ECS services and EC2 Auto Scaling groups in the standby Region, get a sign-off from the duty manager, and then move traffic. Auditors want evidence of each recovery with a timeline and measured RTO. The team wants to avoid custom orchestration code and must not depend on the impaired Region during recovery. Which solution meets these requirements with the least operational overhead?

- A. Write an AWS Step Functions state machine in eu-west-1 that calls the Aurora, ECS and Auto Scaling APIs, waits for a task token from the duty manager, and updates Route 53 records at the end.
- B. Use Amazon Application Recovery Controller Region switch with a plan that includes Aurora Global Database, ECS and EC2 Auto Scaling execution blocks, a manual approval block, and a routing control or Route 53 health check block, with plan execution reports sent to Amazon S3.
- C. Create a Systems Manager Automation runbook in each Region that runs the same steps with an approval step, and record the start and end time of each run in an Amazon DynamoDB table for the auditors.
- D. Use AWS Resilience Hub to assess the application against a resiliency policy with a 15-minute Region RTO and run its recommended SOPs during an event.

<details>
<summary>Answer and explanation</summary>

**Correct answer: B**

**Why B is correct:** Region switch is a managed multi-Region recovery orchestrator. It has execution blocks for Aurora Global Database failover, ECS and EC2 Auto Scaling scaling, manual approval, and traffic control through ARC routing controls or Route 53 health checks, so no custom code is needed. Plans run from the Region being activated, so the impaired Region is not on the recovery path. Plan execution reports in S3 include the recovery timeline and RTO calculation that the auditors want, and plan evaluation flags problems before an event.

**Why the other options are incorrect:**

- **A:** The state machine lives in eu-west-1, which may be the impaired Region. It is also custom orchestration code, and updating Route 53 records uses the Route 53 control plane during the event.
- **C:** This works but is custom work to build, keep in sync across Regions and turn into audit evidence. The manual DynamoDB timing records are weaker evidence than managed execution reports.
- **D:** Resilience Hub assesses and recommends; it does not orchestrate a multi-step Region failover. Running SOPs one by one during an event does not give an ordered, audited plan.

**Key takeaway:** Ordered multi-Region recovery with approvals and audit-ready reports, without custom code, is ARC Region switch.

</details>

**Question 4.2.4** A gaming company serves a UDP-based real-time API from Network Load Balancers in us-east-1 and ap-northeast-1. Many players connect from corporate and school networks that only allow outbound traffic to approved IP addresses. When a Region fails, players must be moved to the other Region within seconds, and some client libraries cache DNS answers for hours. The company wants a managed solution. Which solution meets these requirements?

- A. Create Route 53 latency-based records for both NLBs with health checks and a 30-second TTL, and publish the NLBs' IP addresses to customers for their allow lists.
- B. Put Amazon CloudFront in front of both NLBs with an origin group for failover, and publish the CloudFront IP ranges to customers for their allow lists.
- C. Create Route 53 failover records with ARC routing controls for the two NLBs, and assign Elastic IP addresses to each NLB for the allow lists.
- D. Create an AWS Global Accelerator standard accelerator with an endpoint group for each Region pointing to the NLBs, use health checks for failover, and publish the two static anycast IP addresses for the allow lists.

<details>
<summary>Answer and explanation</summary>

**Correct answer: D**

**Why D is correct:** Global Accelerator gives two static anycast IP addresses that stay the same during failover, which fits the allow lists. It supports UDP. Health checks on endpoint groups move traffic to the healthy Region within seconds, and failover does not depend on DNS, so clients that cache DNS for hours are not affected.

**Why the other options are incorrect:**

- **A:** Clients that cache DNS for hours ignore the 30-second TTL, so they keep connecting to the failed Region. Publishing two Regional NLB IP sets also means clients must try both, which does not give seconds of failover.
- **B:** CloudFront does not proxy arbitrary UDP traffic, and its IP ranges are large and change, which makes allow lists hard to manage.
- **C:** Routing controls give a reliable switch, but the failover is still DNS-based, so the cached DNS problem remains. Clients would also need both Regions' Elastic IPs in their allow lists and would only move when DNS changes.

**Key takeaway:** Static IPs, UDP, and fast failover that does not depend on DNS caching point to Global Accelerator.

</details>

**Question 4.2.5** A media company runs Amazon EKS clusters in us-east-1 (active) and us-west-2 (standby) with stateless microservices. Data is in DynamoDB global tables. During a recent DR test, the standby cluster could not start several services: images were missing, and some Kubernetes manifests in us-west-2 were outdated. The company wants the standby Region to always be ready with the least manual work. Which TWO actions should the solutions architect take? (Select TWO.)

- A. Configure Amazon ECR replication rules to replicate repositories from us-east-1 to us-west-2 so images exist in the standby Region.
- B. Configure an ECR pull through cache rule in us-west-2 that pulls images from the us-east-1 registry when pods start.
- C. Deploy both clusters from the same Git repository with a GitOps controller such as Argo CD or Flux, so every change reaches both clusters.
- D. Take a weekly snapshot of the us-east-1 cluster's etcd data with a scheduled job and restore it into the us-west-2 cluster.
- E. Extend the us-east-1 EKS cluster with a managed node group in us-west-2 so one control plane manages both Regions.

<details>
<summary>Answer and explanation</summary>

**Correct answer: A, C**

**Why A and C are correct:** ECR replication copies images to us-west-2 as they are pushed, so the standby cluster can pull them even if us-east-1 is impaired. GitOps keeps the desired state of both clusters in one repository, and the controller in each cluster applies every change, which removes manifest drift without manual steps.

**Why the other options are incorrect:**

- **B:** A pull through cache fetches from the upstream registry on first pull. During a us-east-1 outage, the upstream is unavailable, so images that were never cached cannot be pulled.
- **D:** EKS manages the control plane, and customers do not have direct access to etcd for snapshot and restore. A weekly copy would also leave up to a week of drift.
- **E:** An EKS cluster and its control plane are Regional; you cannot place a managed node group in another Region. A single control plane would also be a single point of failure.

**Key takeaway:** Multi-Region container DR needs replicated images (ECR replication) and identical configuration from code (GitOps or IaC) in every cluster.

</details>

## Task 4.3: Design solutions to meet business continuity and resiliency requirements

This task tests whether you can make backups governed, immutable and recoverable across accounts, and whether you can prove resilience with controlled failure rather than assume it. The decision the exam keeps asking for is which control actually stops a privileged or compromised identity from destroying data, and which test actually measures recovery.

### Skill 4.3.1: Centralized backup governance and cross-account backup policies (AWS Backup)

**Key concepts**

- **AWS Backup** centralizes backup plans (schedule, lifecycle to cold storage, retention, copy actions to other Regions or accounts) and resource assignments (by tag or resource type) across services such as EC2, EBS, RDS, Aurora, DynamoDB, EFS, FSx, S3, Storage Gateway volumes and others.
- **Organizations integration**: only the **management account** enables backup policies, cross-account monitoring and cross-account backup (copy) in AWS Backup settings. It can register up to five **delegated administrator** member accounts that can manage backup policies and monitor jobs across the organization. Delegated administrators cannot register other delegated administrators or turn on cross-account management.
- **Backup policies** are an AWS Organizations policy type. You attach them to the root, OUs or accounts. They create backup plans in member accounts that local administrators cannot change. Policies merge through inheritance, so a root policy can set 7-day retention and a Finance OU policy can set 30 days for the same plan name. Resource selection in backup policies is by resource type or tag, not by individual ARN.
- Each member account needs a backup vault (or a logically air-gapped vault) and an IAM role that the policy references before the policy can work.
- **Cross-account copy**: accounts must be in the same organization. The destination vault's access policy must allow the copy. For resources that AWS Backup does not fully manage (for example EBS snapshots and RDS), the source must be encrypted with a customer managed KMS key that is shared with the destination account; AWS managed keys cannot be shared across accounts.
- **Central backup account pattern**: each workload account backs up locally (fast restore), and a copy action sends recovery points to a vault in a separate backup account in a separate OU with tight SCPs. A compromise of the workload account then cannot reach the copies.
- **Cross-account monitoring** shows backup, copy and restore jobs across the organization from the management or delegated administrator account. **AWS Backup Audit Manager** (a feature of AWS Backup, not the separate AWS Audit Manager service, which closed to new customers on 30 April 2026) evaluates controls (for example "resources are protected by a plan", "recovery points are encrypted", "minimum retention") and produces reports to S3 for auditors.
- **Restore testing** restores recovery points on a schedule and can run your validation, which proves the backups can be used.
- **Data Lifecycle Manager** automates EBS snapshot and EBS-backed AMI lifecycles inside one account. Choose AWS Backup when you need cross-service, cross-account and organization-wide policy; choose DLM for simple EBS snapshot schedules.

**Exam traps**

- Using CloudFormation StackSets to push backup plans into every account. It works, but local admins can edit or delete the plans; backup policies give central control with less overhead.
- Cross-account copy of RDS or EBS backups encrypted with the AWS managed key. The copy fails because AWS managed keys cannot be shared.
- Relying on backups that stay in the same account as production. A compromised admin in that account can delete them unless the vault is locked.
- Targeting resources by ARN in a backup policy. Policies select by tag or resource type.

**Hands-on**

1. **Backup policy in a small organization.** If you have two free accounts, create an organization, enable backup policies and cross-account monitoring in AWS Backup settings, and attach a backup policy to an OU that backs up resources tagged `backup=daily`. Tag a DynamoDB table in the member account and check that the plan appears there and cannot be edited locally. With one account only, read the backup policy syntax page and create the equivalent local backup plan. *Cost/cleanup:* Backup storage for a small table is cheap; detach the policy, delete recovery points and vaults, and close or remove the lab account if you created one.
2. **Cross-account copy with a customer managed key.** In account A, create a KMS customer managed key with a key policy that allows account B. Encrypt an EBS volume with it, back it up, and add a copy action to a vault in account B whose access policy allows account A. Confirm the copy job succeeds, then repeat with a volume encrypted by the AWS managed key and observe the failure. *Cost/cleanup:* KMS keys cost a small monthly fee; schedule key deletion, delete recovery points in both vaults and the volumes.

### Skill 4.3.2: Failover tests and measuring recovery time (FIS, Route 53 health checks)

**Key concepts**

- A failover test needs three things: a fault that causes the failover, a signal that shows when it is detected, and a measurement of when service is back. FIS injects the fault; Route 53 health checks and CloudWatch alarms give the signals; synthetic canaries or real traffic measure user impact.
- **AWS FIS actions** for failover tests: stop or terminate EC2 instances, reboot or fail over RDS and Aurora clusters, interrupt Spot instances, disrupt network connectivity for subnets, stop ECS tasks, terminate EKS nodes, inject API errors, and more. The **scenario library** includes AWS-built scenarios such as **AZ: Power Interruption** (symptoms of a full power loss in one AZ across several resource types) and **Cross-Region: Connectivity** (blocks traffic between Regions).
- **Route 53 health checks** have their own timing: request interval (standard or fast), failure threshold, and the health checkers' consensus. Detection time is roughly interval times threshold, then DNS TTL and client caching add to the time before users move. Measure all of it.
- Measure recovery time with a **CloudWatch Synthetics** canary hitting the public endpoint every minute: the gap between the first failed and the first successful run, plus the error count, is the user-visible RTO.
- **Aurora failover tests**: use FIS or the `FailoverDBCluster` action within a Region; use Global Database switchover for planned cross-Region tests. Watch how long the application's connection pool takes to reconnect; driver DNS caching often makes this longer than the database failover.
- **ARC zonal autoshift practice runs** are weekly built-in failover tests for AZ evacuation, judged by your outcome alarm.
- Region switch execution reports record the timeline of a Region failover and compute the recovery time, which is test evidence for regulated firms.

**Exam traps**

- Measuring RTO only from the database failover time. The user-visible RTO includes detection, DNS propagation, client reconnect and cache warm-up.
- Testing failover by deleting resources by hand in production. FIS gives scoped targets, stop conditions and a CloudTrail record of every action.
- Using a Route 53 health check with a high failure threshold and slow interval, then expecting failover in seconds.

**Hands-on**

1. **Measure Aurora failover as the application sees it.** Create an Aurora MySQL cluster with one writer and one reader (smallest instance class). Run a script that writes a row every second and logs errors with timestamps. Create an FIS experiment template with the `aws:rds:failover-db-cluster` action and run it. Compare the database event time with the gap in your script's writes. *Cost/cleanup:* Aurora instances bill hourly, FIS bills per action-minute; delete the cluster and the template.
2. **Time a Route 53 failover with a canary.** Using the failover records from Skill 4.2.4, add a CloudWatch Synthetics canary that calls the endpoint every minute. Break the primary, and record the time from failure to the first successful canary run against the secondary. Change the health check interval and failure threshold and repeat. *Cost/cleanup:* Canary runs bill per run and store artifacts in S3; delete the canary, its S3 bucket and the health checks.

### Skill 4.3.3: Technically resilient architectures: redundancy, fault tolerance, self-healing

**Key concepts**

- **Redundancy**: run across at least two, preferably three AZs; size so that losing one AZ still leaves enough capacity (static stability). For example, with three AZs and a peak need of 6 instances, run 9 so that 6 remain after losing an AZ, instead of waiting for Auto Scaling during the event.
- **Static stability**: the system keeps working during a failure without needing control plane changes (launching instances, changing DNS, creating resources). Pre-provision capacity, pre-create DR infrastructure, and fail over with data plane actions.
- **Self-healing**: EC2 Auto Scaling replaces instances that fail EC2 or ELB health checks; ECS replaces stopped tasks; Kubernetes restarts failed pods and reschedules them; EC2 auto recovery moves an instance to new hardware on system status check failure. Use ELB health checks in the Auto Scaling group, not only EC2 status checks, so an instance with a broken application is replaced.
- **Fault isolation**: cells (independent copies of the stack serving a subset of customers), shuffle sharding, and per-tenant limits contain the blast radius. Bulkheads separate resource pools so one dependency cannot exhaust all threads or connections.
- **Loose coupling**: SQS between tiers absorbs spikes and lets consumers fail and retry without losing work. Dead-letter queues keep failed messages for analysis. Idempotent consumers make retries safe.
- **Graceful degradation**: timeouts, retries with exponential backoff and jitter, circuit breakers, and fallbacks (serve cached or static content from CloudFront when the origin fails, using origin failover groups).
- **Managed services with built-in resilience**: S3, DynamoDB, SQS, Lambda and Aurora storage replicate across AZs in a Region by default. RDS Multi-AZ adds a synchronous standby; Multi-AZ DB clusters add two readable standbys and faster failover.
- **Health check design**: shallow checks (process is up) for load balancer routing; deep checks (dependencies work) used carefully, because a shared dependency failure can make every instance fail its health check at once. ALB and NLB fail open when all targets are unhealthy.

**Exam traps**

- Relying on Auto Scaling to replace an AZ's capacity during an AZ event. Launches may be slow or capacity may be short; static stability means the capacity is already there.
- Deep health checks that test the database on every instance. When the database blips, every instance is marked unhealthy and replaced, which turns a small event into an outage.
- Retries without backoff and jitter. They amplify load on a struggling dependency (retry storms).
- Single zonal NAT gateway for a multi-AZ private fleet. If its AZ fails, private subnets in every AZ lose outbound access. Use one zonal NAT gateway per AZ, or a regional NAT gateway, which spans AZs automatically.

**Hands-on**

1. **Self-healing with ELB health checks.** Create an Auto Scaling group of two t3.micro instances across two AZs behind an ALB, with health check type set to ELB. Stop the web server process on one instance (do not stop the instance). Watch the target go unhealthy and Auto Scaling replace the instance. Repeat with health check type EC2 and see that nothing happens. *Cost/cleanup:* Free tier instances may apply; the ALB bills hourly. Delete the group, launch template, ALB and target group.
2. **Queue-based decoupling with a DLQ.** Create an SQS queue with a dead-letter queue (max receives 3) and a Lambda consumer that fails for messages containing "bad". Send good and bad messages, confirm good ones are processed and bad ones land in the DLQ, then use DLQ redrive to send them back after fixing the function. *Cost/cleanup:* Within free tier for small volumes; delete queues and the function.

### Skill 4.3.4: Protect data against ransomware and accidental deletion (S3 Object Lock, EBS snapshot lock, AWS Backup Vault Lock)

**Key concepts**

- **S3 Object Lock** (WORM) works on object versions and needs versioning. It can be enabled on new buckets and on existing buckets. Modes:
  - **Governance**: users with `s3:BypassGovernanceRetention` can delete or shorten retention by sending the bypass header. Use it to test policies or to allow a small break-glass group.
  - **Compliance**: no one, including the root user, can delete the version or shorten retention until it expires. Retention can only be extended.
  - **Legal hold**: no expiry date; stays until removed by someone with `s3:PutObjectLegalHold`. It is independent of retention mode.
  - A bucket default retention applies to new versions. Deleting an object without a version ID only adds a delete marker; the locked version is still there.
- **EBS snapshot lock**: locks individual snapshots for a duration (1 to 36,500 days) or until a date.
  - **Governance mode**: users with the right IAM permissions can unlock or change the lock at any time.
  - **Compliance mode**: an optional cooling-off period of 1 to 72 hours lets authorized users unlock or change it; after that, no one can unlock it, and the duration can only be increased. Without a cooling-off period it takes effect immediately.
  - Use it for EBS snapshots managed outside AWS Backup (for example by Data Lifecycle Manager or scripts).
- **AWS Backup Vault Lock**: applies to a whole vault and enforces minimum and maximum retention on new backup and copy jobs.
  - **Governance mode**: users with sufficient IAM permissions can remove or change the lock.
  - **Compliance mode**: requires a grace time of at least 3 days (72 hours). After grace time, the lock is immutable; no user, the root user or AWS can change or delete it, and recovery points cannot be deleted or have their lifecycle shortened before retention ends. A compliance-locked vault can be deleted only when empty.
  - Vault Lock covers actions through AWS Backup. Direct EC2 snapshot actions are not covered, so pair it with SCPs or EBS snapshot lock where needed. Recovery points that existed before the lock keep their old lifecycle. Check retention before grace time ends: a recovery point set to "Always" retention is kept, and billed, forever.
- **Logically air-gapped vault**: an AWS Backup vault type that is always locked in compliance mode, stores backups in an AWS Backup service-owned account, and is encrypted with an AWS owned key by default (or a customer managed key chosen at creation). You share it through AWS RAM with specific account IDs (in your organization or another one), and those accounts can restore directly without copying first, which shortens recovery. **Multi-party approval** lets a separate recovery organization get access to restore even if the owning account is compromised or unreachable, after a team of approvers agrees. Minimum retention is 7 days.
- Layered ransomware design: immutable copies (Vault Lock compliance or logically air-gapped vault) in a separate account and OU, SCPs that deny deleting vaults, recovery points, KMS keys and changes to lock settings, MFA and least privilege for backup admins, GuardDuty Malware Protection and S3 protection for detection, and tested restores.
- KMS is part of the blast radius. If an attacker can schedule deletion of the CMK that encrypts backups, the backups become unreadable. Protect key policies, use SCPs to deny `kms:ScheduleKeyDeletion` and `kms:DisableKey` for backup keys, or use the AWS owned key option for logically air-gapped vaults.

**Exam traps**

- S3 Versioning plus MFA Delete as ransomware protection. It helps against accidental deletion, but only the root user can configure MFA Delete, it is hard to operate, and it does not stop a root compromise. Object Lock compliance mode is the stronger control.
- Governance mode when the requirement says "no one, including administrators". Governance can be bypassed; compliance cannot.
- Vault Lock compliance mode with the minimum grace time set and no review. After 3 days, mistakes in retention are permanent.
- Assuming Vault Lock protects snapshots from direct EC2 API deletion. It covers AWS Backup actions only.
- Keeping immutable backups in the same account but encrypting them with a key the compromised admin can delete.

**Hands-on**

1. **S3 Object Lock modes.** Create a bucket with Object Lock enabled. Upload an object with governance retention of one day and try to delete the version (fails), then delete it with the bypass header as a user who has `s3:BypassGovernanceRetention` (works). Upload another object with a legal hold and try to delete it, then remove the hold. Only use compliance mode with a retention of one day, because you cannot delete that version or the bucket until it expires. *Cost/cleanup:* Pennies of storage; remove holds, wait for retention to expire, then delete versions and the bucket.
2. **Backup Vault Lock and EBS snapshot lock in governance mode.** Create a backup vault and put a governance-mode Vault Lock with minimum retention 1 day (omit `ChangeableForDays`). Back up a small EBS volume into it and try to delete the recovery point (fails until retention allows). Separately, lock a manual EBS snapshot with `aws ec2 lock-snapshot --lock-mode governance --lock-duration 1`, try to delete it, then unlock it. Do not use compliance mode in a lab unless you accept paying for storage until it expires. *Cost/cleanup:* Small snapshot storage; remove the vault lock, unlock the snapshot, delete recovery points, snapshots, the volume and the vault.

### Skill 4.3.5: Chaos engineering with controlled failure (FIS experiment templates, steady-state hypotheses)

**Key concepts**

- Chaos engineering loop: define **steady state** with metrics that reflect user experience (for example p99 latency under 300 ms and error rate under 0.1 percent), form a **hypothesis** ("if one AZ loses power, steady state holds"), inject a real-world fault, observe, then fix and repeat. Start small in a test environment, then move to production with a limited blast radius.
- **FIS experiment template** parts:
  - **Actions**: the faults (for example `aws:ec2:terminate-instances`, `aws:ec2:stop-instances`, `aws:ecs:stop-task`, `aws:eks:terminate-nodegroup-instances`, `aws:rds:failover-db-cluster`, `aws:network:disrupt-connectivity`, `aws:fis:inject-api-throttle-error` and SSM-based CPU, memory, disk and network latency stress through `aws:ssm:send-command`). Actions have durations and can run in sequence or in parallel.
  - **Targets**: resources chosen by ARN, tags, filters or resource type, with selection modes such as ALL, COUNT or PERCENT to limit blast radius.
  - **Stop conditions**: CloudWatch alarms. If an alarm enters ALARM, FIS stops the experiment. This is the main safety mechanism. You can also stop an experiment manually.
  - **IAM role**: FIS assumes a role with only the permissions the actions need.
  - **Logging**: experiment logs to CloudWatch Logs or S3; every action is in CloudTrail.
- **Scenario library**: AWS-built scenarios (for example AZ: Power Interruption, Cross-Region: Connectivity) are the fastest way to test realistic, multi-resource failures. You copy the scenario into a template and set targets by tag.
- **Multi-account experiments** run from an orchestrator account against target accounts through target account configurations and roles, which suits workloads spread across accounts.
- Instance-level faults need the SSM Agent on the instance, because FIS runs them through Systems Manager documents.
- Pricing is per action-minute, so long-running experiments across many targets cost more.
- Guardrails for production chaos: restrict who can start experiments with IAM (for example, tag conditions on templates), require stop conditions, run in business hours with people watching, and start with a small percentage of targets.
- Use Resilience Hub recommended experiments and ARC practice runs as part of a continuous resilience program, and run FIS experiments in CI/CD pipelines against staging environments.

**Exam traps**

- An experiment with no stop condition in production. The exam expects CloudWatch alarm stop conditions tied to the steady-state metrics.
- Defining steady state as CPU utilization. Steady state should reflect what users see (errors, latency, throughput), not internal resource metrics.
- Writing custom Lambda functions that terminate random instances. FIS gives scoped targets, stop conditions, IAM control and audit logs with less effort.
- Expecting instance CPU or memory stress to work on instances without SSM Agent.
- Starting with a full AZ power interruption in production before smaller experiments pass.

**Hands-on**

1. **Steady-state hypothesis with a stop condition.** Using the ALB and Auto Scaling group from Skill 4.3.3, create a CloudWatch alarm on ALB `HTTPCode_Target_5XX_Count` and on p99 `TargetResponseTime`. Write the hypothesis: "terminating 50 percent of instances keeps 5xx at zero". Create an FIS template with `aws:ec2:terminate-instances`, target the group's instances by tag with PERCENT 50, and the alarms as stop conditions. Run it and check whether the hypothesis held; then lower the alarm threshold and watch FIS stop the experiment. *Cost/cleanup:* FIS bills per action-minute; delete the template, alarms and lab stack.
2. **AZ power interruption scenario in a test VPC.** From the FIS scenario library, open AZ: Power Interruption, review the actions it bundles, and create a template that targets only resources tagged `chaos=allowed` in one AZ of a test environment (EC2 instances in an Auto Scaling group and, optionally, an Aurora cluster). Run it with a stop condition and observe what recovered on its own and what did not. *Cost/cleanup:* Scenario actions run for the duration you set, so shorten them; the resources bill hourly. Delete the template and resources.

### Practice questions: Task 4.3

**Question 4.3.1** A bank has 140 AWS accounts in AWS Organizations. Auditors found that several application teams deleted or shortened backup plans in their own accounts. The bank now requires that every Amazon RDS database and Amazon EBS volume tagged `tier=critical` is backed up daily, kept for 35 days, and copied to a central backup account, and that account administrators cannot change these settings. The backup team is small and wants one place to see job status for all accounts. Which solution meets these requirements with the least operational overhead?

- A. Use CloudFormation StackSets from the management account to deploy an AWS Backup plan with a cross-account copy action into every account, and use AWS Config rules to detect when a plan is changed.
- B. In the management account, enable AWS Backup backup policies, cross-account backup and cross-account monitoring; attach a backup policy to the root that selects resources by the `tier=critical` tag and copies to the central vault; register the backup account as a delegated administrator.
- C. Use Amazon Data Lifecycle Manager cross-account copy policies in each account for EBS snapshots, and schedule an AWS Lambda function in each account to create and copy RDS snapshots to the central account.
- D. Create a backup plan in the central backup account that selects resources by tag across all member accounts, and grant the plan's IAM role access to the member accounts through an IAM role in each account.

<details>
<summary>Answer and explanation</summary>

**Correct answer: B**

**Why B is correct:** Backup policies are an AWS Organizations policy type that creates backup plans in member accounts that local administrators cannot change, which closes the audit gap. Selection by tag picks the `tier=critical` resources, and the policy can include the schedule, 35-day retention and a copy action to the central account once cross-account backup is enabled. Cross-account monitoring gives one view of jobs, and the delegated administrator lets the backup team work from the backup account rather than the management account.

**Why the other options are incorrect:**

- **A:** StackSets deploys the plans, but local administrators can still edit or delete them; Config only detects the change after it happens. It also adds two systems to maintain.
- **C:** Data Lifecycle Manager covers only EBS snapshots and AMIs, and custom Lambda code for RDS adds a lot of operational work. Neither is protected from local administrators.
- **D:** An AWS Backup plan selects resources only in its own account. A central plan cannot back up resources in other accounts by tag.

**Key takeaway:** Organization-wide, tamper-proof backup rules are AWS Backup backup policies managed from the management account or a delegated administrator.

</details>

**Question 4.3.2** An asset manager must keep trade confirmation backups for 7 years under a WORM rule. The rule says no user, including the root user of any account, can delete or shorten retention. The security team also worries that the production account could be fully compromised, and it wants to restore backups into a clean account without first copying data, to shorten recovery. Backups are taken by AWS Backup for Amazon DynamoDB and Amazon EFS. Which TWO actions meet these requirements? (Select TWO.)

- A. Copy recovery points to an AWS Backup logically air-gapped vault, which is locked in compliance mode, and share it through AWS RAM with a dedicated recovery account.
- B. Store the backups in a standard backup vault with an AWS Backup Vault Lock in governance mode and a minimum retention of 7 years.
- C. Set up multi-party approval for the logically air-gapped vault so a recovery organization can gain restore access after a team of approvers agrees, even if the owning account is unavailable.
- D. Export the backups to an S3 bucket with S3 Versioning and MFA Delete enabled, and replicate the bucket to the recovery account with S3 Cross-Region Replication.
- E. Lock every EBS snapshot with EBS snapshot lock in compliance mode with a 72-hour cooling-off period.

<details>
<summary>Answer and explanation</summary>

**Correct answer: A, C**

**Why A and C are correct:** A logically air-gapped vault is always locked in compliance mode, so no one can delete recovery points or shorten retention before it expires, which meets the WORM rule. It supports the AWS Backup-managed resources here once they are copied in. Sharing it through AWS RAM lets a separate recovery account restore directly from the shared vault without making a copy first. Multi-party approval adds a path for a separate recovery organization to get restore access when the owning account is compromised or unreachable, after approvers agree.

**Why the other options are incorrect:**

- **B:** Governance mode can be changed or removed by users with enough IAM permissions, so it does not meet "no user, including root".
- **D:** Versioning and MFA Delete do not stop the root user, who controls MFA Delete. Exporting AWS Backup recovery points to S3 is also not how these backups are stored, and it adds custom work.
- **E:** The workloads are DynamoDB and EFS, not EBS. EBS snapshot lock protects individual EBS snapshots only.

**Key takeaway:** Immutable plus recoverable from a clean account points to logically air-gapped vaults shared through RAM, with multi-party approval as the break-glass path.

</details>

**Question 4.3.3** An e-commerce company runs its checkout service on Amazon ECS with AWS Fargate across three Availability Zones behind an Application Load Balancer, with Aurora MySQL. Leadership wants proof that checkout survives the loss of an AZ, with tests that run in production every month. The tests must stop at once if customer impact passes an agreed level, every action must be auditable, and the SRE team wants a realistic, multi-resource AZ failure without writing custom scripts. Which solution meets these requirements?

- A. Write an AWS Lambda function that stops a random 33 percent of ECS tasks and fails over the Aurora writer, schedule it monthly with Amazon EventBridge Scheduler, and log each action to CloudWatch Logs.
- B. Enable ARC zonal autoshift on the ALB with weekly practice runs and an outcome alarm on checkout error rate, and treat the practice run results as the monthly AZ failure test.
- C. Use AWS Resilience Hub to run monthly assessments of the checkout service against a resiliency policy with an AZ RTO of 5 minutes, and report the estimated RTO to leadership.
- D. Create an AWS FIS experiment template from the AZ: Power Interruption scenario, target resources by tag in one AZ, add CloudWatch alarms on checkout error rate and p99 latency as stop conditions, and schedule it monthly.

<details>
<summary>Answer and explanation</summary>

**Correct answer: D**

**Why D is correct:** The AZ: Power Interruption scenario is an AWS-built, multi-resource simulation of losing an AZ, so no custom scripts are needed. Targeting by tag keeps the blast radius to the checkout resources in one AZ. CloudWatch alarms on customer-facing metrics as stop conditions halt the experiment once impact passes the agreed level. FIS records every action in CloudTrail and can log the experiment, which gives the audit trail.

**Why the other options are incorrect:**

- **A:** It is custom code with no built-in stop condition, so a bad run keeps going while customers are affected. Stopping random tasks across all AZs also does not simulate an AZ failure.
- **B:** Practice runs shift load balancer traffic away from an AZ; they do not simulate the failure of compute and database resources in that AZ. They test traffic evacuation, not survival of an AZ power loss.
- **C:** Resilience Hub assessments estimate RTO from configuration. They do not inject faults or prove behavior in production.

**Key takeaway:** Realistic AZ failure tests with safety and audit come from FIS scenarios with CloudWatch alarm stop conditions.

</details>

**Question 4.3.4** A healthcare SaaS company runs an API on Amazon EC2 in an Auto Scaling group across two AZs behind an Application Load Balancer. During a recent incident, an application thread pool deadlocked on several instances. The instances passed EC2 status checks and stayed in service for an hour, returning errors. A week later, a short Amazon RDS outage caused every instance to fail its health check, and Auto Scaling replaced the whole fleet, which made the outage longer. Which change best addresses both problems?

- A. Change the Auto Scaling group health check type to ELB, and point the ALB target group health check at a lightweight endpoint that checks the application process and thread pool but not the database.
- B. Keep the EC2 health check type, and add a CloudWatch alarm on ALB 5xx errors that runs a Systems Manager Automation runbook to reboot all instances in the group.
- C. Change the Auto Scaling group health check type to ELB, and point the ALB target group health check at a deep endpoint that runs a test query against the RDS database.
- D. Enable EC2 auto recovery on every instance, and set the Auto Scaling group to use a warm pool so that replacement instances start faster.

<details>
<summary>Answer and explanation</summary>

**Correct answer: A**

**Why A is correct:** With ELB health checks, Auto Scaling replaces instances that fail the load balancer's check, not only instances with failed hardware or OS status. A lightweight check that tests the local application and thread pool catches the deadlock, so those instances are removed and replaced. Leaving the database out of the check stops a shared dependency outage from marking every instance unhealthy at once.

**Why the other options are incorrect:**

- **B:** EC2 status checks do not see application deadlocks, so bad instances stay in service. Rebooting all instances on any 5xx spike also takes down healthy instances and repeats the second incident.
- **C:** ELB health checks fix the first problem, but a deep check that queries RDS makes every instance fail during a database blip, which is exactly the second incident.
- **D:** Auto recovery acts on system status check failures caused by hardware, not on application deadlocks. A warm pool speeds replacements but does not stop the wrong replacements.

**Key takeaway:** Use ELB health checks for self-healing, and keep health checks shallow enough that a shared dependency failure does not fail the whole fleet.

</details>

**Question 4.3.5** A media archive keeps Amazon EBS snapshots of editing workstations, created by Amazon Data Lifecycle Manager, and stores final assets in Amazon S3. After a ransomware attempt that used stolen administrator credentials, the CISO wants the snapshots and final assets protected from deletion for 1 year. Legal also needs to keep certain assets indefinitely while litigation is active, and the security team wants a short window after setting each snapshot lock in which mistakes can be corrected. Which solution meets these requirements?

- A. Enable S3 Versioning with MFA Delete on the bucket, and move the snapshots into an AWS Backup vault with Vault Lock in compliance mode and a 3-day grace time.
- B. Enable S3 Object Lock in governance mode with a 1-year default retention, and lock each EBS snapshot with EBS snapshot lock in governance mode for 365 days.
- C. Enable S3 Object Lock in compliance mode with a 1-year default retention and place legal holds on litigation assets, and lock each EBS snapshot in compliance mode for 365 days with a cooling-off period.
- D. Replicate the bucket to a second account with S3 Replication and delete marker replication turned off, and copy the EBS snapshots to the second account with Data Lifecycle Manager cross-account copy.

<details>
<summary>Answer and explanation</summary>

**Correct answer: C**

**Why C is correct:** S3 Object Lock in compliance mode stops anyone, including administrators with stolen credentials, from deleting locked versions or shortening retention for a year. Legal holds keep litigation assets with no expiry until removed, independent of the retention period. EBS snapshot lock in compliance mode protects the DLM-created snapshots for 365 days, and the cooling-off period (1 to 72 hours) gives the requested window to fix mistakes before the lock becomes permanent.

**Why the other options are incorrect:**

- **A:** MFA Delete does not stop a root user and is not a retention control. The final assets in S3 therefore stay deletable by a compromised root user, and no indefinite hold is placed on them. Moving the DLM snapshots into AWS Backup also changes the backup design when EBS snapshot lock can protect them where they are.
- **B:** Governance mode can be bypassed or removed by users with the right IAM permissions, which is exactly what stolen administrator credentials provide. It also has no legal hold for indefinite retention.
- **D:** Replication to a second account helps, but without locks an attacker with access to the second account can still delete data, and there is no legal hold or fixed retention.

**Key takeaway:** For deletion protection against privileged credentials, use compliance modes (S3 Object Lock, EBS snapshot lock with a cooling-off period, Vault Lock), and legal holds for indefinite retention.

</details>

---

# Domain 5: Operational Excellence and Automation (20%)

Tasks 5.1 to 5.4: infrastructure as code, CI/CD and DevSecOps, observability, and operational automation.

## Task 5.1: Design infrastructure as code

This task tests whether you can pick the right IaC layer for each control: authoring (CloudFormation or CDK), proactive validation (Hooks, CDK aspects), detective and corrective controls (AWS Config, drift detection), preventive guardrails (SCPs), and fleet distribution (StackSets). Most questions reduce to the same choice: stop a bad change before it is provisioned, or find and fix it afterwards, and which mechanism does each with the least operational overhead.

### Skill 5.1.1: Declarative and imperative IaC (CloudFormation, CDK)

**Key concepts**

- CloudFormation is declarative. You describe the end state in a template and the service works out create, update and delete order from references and `DependsOn`. The stack is the unit of lifecycle, rollback and drift detection.
- AWS CDK is an imperative authoring layer on top of the same engine. You write TypeScript, Python, Java, C# or Go. `cdk synth` produces CloudFormation templates and assets (the cloud assembly), and `cdk deploy` deploys those templates as stacks. Deployment behavior, rollback and drift are therefore CloudFormation behavior. CDK does not bypass the engine.
- Use CDK when you need loops, conditionals, shared libraries of L2 and L3 constructs, unit tests (the `assertions` module) and type checking. Use raw CloudFormation when templates must be readable by auditors, consumed by Service Catalog or StackSets with no build step, or owned by teams without a software toolchain.
- CDK bootstrapping creates the `CDKToolkit` stack in each target account and Region: an S3 bucket and ECR repository for assets plus deploy, file-publishing, image-publishing, lookup and CloudFormation execution roles. For cross-account pipelines, bootstrap each target with `--trust <pipeline account>` and set `--cloudformation-execution-policies` to a least-privilege policy instead of the default AdministratorAccess.
- Change sets preview what an update will do (add, modify, replace). Replacement of a stateful resource is the classic risk. Protect with `DeletionPolicy: Retain` or `Snapshot`, `UpdateReplacePolicy`, and stack policies that deny `Update:Replace` on databases.
- Rollback triggers let CloudFormation watch CloudWatch alarms during and for a monitoring period after a stack operation, and roll back if an alarm fires. This is the IaC equivalent of alarm-based deployment rollback.
- Stack refactoring (CLI first, console added in 2025) moves resources between stacks and renames logical IDs without deleting and recreating them. Use it to split a monolithic stack. Before this existed, the only route was `DeletionPolicy: Retain` plus resource import.
- Resource import brings existing resources under a stack. The IaC generator scans an account and produces a template for resources created by hand. Both are the answer for "bring click-ops resources under IaC without recreating them".
- Nested stacks reuse components inside one deployment and share one rollback. Cross-stack references (`Export` and `Fn::ImportValue`) link independent stacks but lock the exporting value: you cannot change or delete an export while another stack imports it.

**Exam traps**

- "Use CDK so the deployment can bypass CloudFormation limits." CDK still deploys CloudFormation stacks, so per-stack resource limits, rollback behavior and Hooks all still apply.
- "Rename the logical ID in the template to move a resource." A changed logical ID makes CloudFormation create a new resource and delete the old one. Use stack refactoring instead.
- Choosing a CloudFormation macro or custom resource for logic that CDK or a module can express. Macros need a Lambda function you own, and StackSets with service-managed permissions do not support macros or transforms.
- Treating `cdk diff` as a safety control. It is a preview for humans. It enforces nothing.

**Hands-on**

1. **Watch CDK become CloudFormation.** Create a CDK app in TypeScript or Python with one S3 bucket (with versioning) and one SQS queue. Run `cdk synth` and read the generated template under `cdk.out`, including the asset metadata and the bootstrap version check. Deploy it, change the bucket's logical ID in code, and run `cdk diff` to see that CDK proposes a replacement. This shows why logical IDs matter and that CDK is only an authoring layer. *Cost/cleanup:* Free tier level. Run `cdk destroy`, empty and delete the bucket if it was retained, and delete the `CDKToolkit` stack if you no longer need it.
2. **Protect a stateful resource.** Write a raw CloudFormation template with a DynamoDB table that has `DeletionPolicy: Retain` and `UpdateReplacePolicy: Retain`, plus a stack policy that denies `Update:Replace` on that table. Create a change set that changes the table's key schema and confirm the change set shows `Replacement: True`, then try to execute it and watch the stack policy block it. *Cost/cleanup:* On-demand table with no data costs nothing. Delete the stack, then delete the retained table by hand.

### Skill 5.1.2: Policy as code for proactive validation, enforcement and drift remediation

**Key concepts**

- Think of four layers, each with a different timing:
  - **Synthesis time (CDK aspects, cdk-nag).** An aspect is a visitor that CDK applies to every construct in a scope during the prepare phase of synthesis. It can mutate constructs (this is how `Tags.of()` works) or add an error annotation that makes `cdk synth` fail. cdk-nag is an open-source rule pack library built on aspects. Aspects do not cross `Stage` boundaries, so apply them to the stage. They only protect teams that use your CDK code, so they are a developer feedback loop, not an organization control.
  - **Provisioning time (CloudFormation Hooks).** Hooks inspect resources, whole stacks or change sets before CloudFormation provisions them, and also Cloud Control API operations. Implementation choices: Guard rules stored in S3 (`AWS::CloudFormation::GuardHook`), a Lambda function you own (`AWS::CloudFormation::LambdaHook`), custom Hooks built with the CloudFormation CLI and published to the registry, or AWS Control Tower proactive controls from the Control Catalog that are implemented as managed Hooks. `FailureMode` is `FAIL` (block) or `WARN` (log and continue). Filters narrow which stacks, resource types and actions (create, update, delete) are evaluated. Hooks are per account and Region, so distribute them with StackSets or Control Tower.
  - **Preventive at the API (SCPs, RCPs).** SCPs deny API actions and condition keys regardless of tool. They cannot inspect the full configuration of a resource, only what the request and its condition keys carry. Use them for Region restrictions, protecting security tooling, denying `iam:CreateUser`, requiring tags at create time where the service supports `aws:RequestTag`.
  - **After the fact (AWS Config rules, drift detection).** Config evaluates actual resource state no matter how it was created, including console and Terraform changes. Proactive evaluation mode exists for some managed rules, but Config is mainly detective. Remediation runs an SSM Automation runbook, manually or automatically, with retry settings.
- CloudFormation drift detection compares a stack's resources with their template properties. Drift-aware change sets (November 2025) do a three-way comparison of the new template, the last deployed template and the live state. Executing one with deployment mode `REVERT_DRIFT` puts drifted properties back to template values and recreates resources deleted outside CloudFormation. A normal change set ignores drift, so redeploying the same template does not fix it.
- Guard (`cfn-guard`) rules can be run in three places: locally or in CodeBuild against templates before deploy (shift left), inside a Guard Hook at provisioning, and as a custom Config rule using Guard against recorded configuration items. Same language, three timings.
- Decision rule: block non-compliant CloudFormation deployments in every account → Hook (or Control Tower proactive control) in FAIL mode. Catch anything regardless of tool → Config rule plus remediation. Make a class of action impossible → SCP. Give developers fast feedback → aspects or cdk-nag plus Guard in CI.

**Exam traps**

- Picking an AWS Config rule when the requirement says "prevent" or "before the resource is created". Config sees the resource after it exists. Even with automatic remediation, the non-compliant resource existed for a period.
- Picking CDK aspects as the organization-wide enforcement. Any team using plain CloudFormation, the console or another tool bypasses them.
- Picking an SCP to enforce a property that is not a condition key on the create call. Check whether the service publishes a condition key for the property; when it does not, a Hook or Guard rule is the way to validate it.
- Assuming Hooks cover resources created through the console or direct service APIs. They cover CloudFormation operations and Cloud Control API operations only.
- Starting in FAIL mode across an organization. The safer rollout is WARN first, review the Hooks invocation summary, then switch to FAIL.

**Hands-on**

1. **Guard Hook in WARN then FAIL.** Write a Guard rule file that requires `AWS::S3::Bucket` to have `VersioningConfiguration.Status == "Enabled"`, upload it to an S3 bucket, and deploy an `AWS::CloudFormation::GuardHook` with `TargetOperations: [RESOURCE]`, `FailureMode: WARN` and a `LogBucket`. Deploy a stack with an unversioned bucket, see the warning in the stack events and the Hooks invocation summary, then switch `FailureMode` to `FAIL` and watch the same deployment fail before the bucket is created. *Cost/cleanup:* Hooks invocations for a few test stacks are negligible; check the CloudFormation pricing page for third-party and Hook handler operations. Delete the test stacks, the Hook stack and the rules and log buckets.
2. **Aspect and drift-aware change set.** In a CDK app, add an aspect that calls `Annotations.of(node).addError()` for any `CfnBucket` without encryption settings, and confirm `cdk synth` fails. Fix it and deploy. Then in the console change the bucket's tags or a security group rule that the stack owns, run drift detection, and create a change set with the drift-aware type (`--deployment-mode REVERT_DRIFT` in the CLI) to see the out-of-band change listed and reverted. *Cost/cleanup:* Free. `cdk destroy` and delete any retained bucket.

### Skill 5.1.3: Manage secrets in infrastructure code

**Key concepts**

- Never place a secret value in a template, a parameter default, CDK source or a pipeline variable. Templates are stored, shown in the console and copied to artifact buckets.
- Dynamic references resolve values at deploy time without storing them in the template:
  - `{{resolve:ssm:name:version}}` for plaintext String parameters.
  - `{{resolve:ssm-secure:name:version}}` for SecureString parameters. This works only for a defined list of resource properties, and CloudFormation cannot create SecureString parameters itself.
  - `{{resolve:secretsmanager:secret-id:SecretString:json-key:version-stage:version-id}}` for Secrets Manager. You can reference a specific JSON key and the `AWSCURRENT` or `AWSPREVIOUS` stage.
- Secrets Manager vs Parameter Store:
  - Secrets Manager: built-in rotation (managed rotation for supported services, or a rotation Lambda function), cross-Region replica secrets, resource policies for cross-account use, `GenerateSecretString` in `AWS::SecretsManager::Secret`, `AWS::SecretsManager::RotationSchedule`, and `SecretTargetAttachment` to wire a secret to a database. Charged per secret per month plus API calls.
  - Parameter Store: hierarchy, String, StringList and SecureString (KMS), standard tier (free, up to 4 KB values) and advanced tier (charged, up to 8 KB, parameter policies such as expiration notifications). No native rotation.
  - Rule: database credentials or anything that must rotate → Secrets Manager. Configuration values, feature settings, AMI IDs → Parameter Store.
- Amazon RDS and Aurora can manage the master password in Secrets Manager for you (`ManageMasterUserPassword`). RDS creates the secret and rotates it. This removes both the password parameter and the rotation function from your template.
- CDK: `Secret.fromSecretNameV2()` and `SecretValue.secretsManager()` render as dynamic references, so the value never appears in `cdk.out`. `SecretValue.unsafePlainText()` is named that way for a reason.
- `NoEcho: true` masks a parameter in console and API output, but the value still has to come from somewhere (often a pipeline variable) and is not rotated. It is a masking feature, not secret management.
- Consumers should read secrets at runtime (Secrets Manager caching libraries, the Lambda extension for Parameters and Secrets, ECS task definition `secrets` that inject from Secrets Manager or Parameter Store, the Secrets Store CSI driver on EKS). Injecting into an environment variable at deploy time means a rotation is not picked up until redeploy.
- Cross-account: a secret shared with another account must use a customer managed KMS key whose key policy allows that account, plus a resource policy on the secret. The default `aws/secretsmanager` key cannot be used cross-account.

**Exam traps**

- "Store the password in a SecureString parameter created by the CloudFormation stack." CloudFormation cannot create SecureString parameters, and Parameter Store has no rotation.
- "Use `NoEcho` parameters to meet a rotation requirement." Nothing rotates.
- Cross-account secret access with the AWS managed key. It fails on KMS, not on the secret policy.
- Injecting a rotating secret into a Lambda environment variable via the template. The function keeps the old value until the next deployment.

**Hands-on**

1. **Generated secret with rotation.** Deploy a template with `AWS::SecretsManager::Secret` using `GenerateSecretString` (exclude characters your database rejects) and a `ReplicaRegions` entry for a second Region. Reference the secret from an SSM String parameter's description or a Lambda environment value through `{{resolve:secretsmanager:...}}` and confirm in the processed template view that only the reference appears. Then call `RotateSecret` with a simple rotation Lambda function (use the Secrets Manager rotation template for a generic secret) and see versions move between `AWSPENDING`, `AWSCURRENT` and `AWSPREVIOUS`. *Cost/cleanup:* Secrets are billed per secret per month, prorated, plus API calls; a replica counts as a secret. Delete with `--force-delete-without-recovery` to avoid the recovery window.
2. **ECS secret injection.** Create a SecureString parameter in the console and a Fargate task definition whose container `secrets` block maps it to an environment variable. Run one task that prints only the variable's length to logs, change the parameter value, run a new task and observe the new value is picked up only at task start. *Cost/cleanup:* A single small Fargate task for a few minutes costs cents; standard parameters are free. Delete the task definition revisions, cluster and parameter.

### Skill 5.1.4: Multi-account and multi-Region deployments with StackSets

**Key concepts**

- A StackSet is one template plus parameters deployed as stack instances to account and Region pairs, managed from an administrator account.
- **Self-managed permissions:** you create `AWSCloudFormationStackSetAdministrationRole` in the administrator account and `AWSCloudFormationStackSetExecutionRole` in each target account. Targets are account IDs, can be any account (including outside the organization and the management account), and new accounts are not added automatically. Use for accounts outside AWS Organizations or when you need custom execution roles per target.
- **Service-managed permissions:** requires AWS Organizations with trusted access for StackSets enabled (only a management account administrator can enable it). CloudFormation creates the roles. You target the whole organization or OUs (parent OU includes children), optionally filtered by account lists (intersection, difference, union). **Automatic deployment** deploys to accounts that join a target OU later, and on leave either deletes or retains the stack (retained stacks keep resources but leave the StackSet). Automatic deployment is set for the whole StackSet, not per OU.
- Service-managed StackSets never deploy to the management account, even if you target the root. Delegated administrators (register a member account, then call with `--call-as DELEGATED_ADMIN`) can deploy to any account in the organization; the management account cannot narrow that scope. Service-managed StackSets do not support nested stacks, macros or transforms.
- Operation preferences: maximum concurrent accounts, failure tolerance (count or percentage), Region order and Region concurrency (sequential or parallel), and concurrency mode (strict reduces concurrency as failures occur; soft keeps it). For a safe rollout: sequential Regions, low concurrency, failure tolerance of 0 for the first wave.
- StackSet dependencies (2025) order automatic deployments between StackSets, up to 10 dependencies, with cycle detection. Use when, for example, a networking baseline must exist before an application baseline in a newly vended account.
- Drift detection works at StackSet level across instances. Stack instance parameter overrides let one template take account-specific values.
- Alternatives and when they win:
  - AWS Control Tower with Account Factory customization or Customizations for Control Tower: landing zone baselines tied to account vending.
  - CDK Pipelines or CodePipeline with cross-account CloudFormation actions: application stacks with build, test and approval stages per environment.
  - Service Catalog (with StackSet constraints): let account owners self-serve approved products.

**Exam traps**

- Using service-managed permissions to deploy a baseline to the management account. It is skipped silently. Use a separate self-managed StackSet or a plain stack.
- Using service-managed permissions for accounts outside the organization. Not supported.
- Self-managed StackSets when the requirement says "new accounts must receive the baseline automatically with no manual steps". Only service-managed automatic deployment does that.
- Expecting a delegated administrator's scope to be limited to certain OUs. It cannot be limited.
- Templates with `Transform: AWS::Serverless-2016-10-31` or custom macros in a service-managed StackSet. Not supported; package to plain CloudFormation first.

**Hands-on**

1. **Self-managed StackSet across two Regions.** In one account, create the administration and execution roles from the AWS sample templates (the execution role can be in the same account). Create a self-managed StackSet that deploys an SNS topic and a CloudWatch alarm on `EstimatedCharges` (billing alarm only in us-east-1, so use a simple SNS topic elsewhere) to two Regions, with sequential Region order and failure tolerance 0. Update the template and watch instances update one Region at a time. *Cost/cleanup:* Free. Delete stack instances first, then the StackSet, then the roles.
2. **Service-managed with automatic deployment (needs two accounts).** If you can create a small organization with a free second account, enable trusted access, create an OU, and create a service-managed StackSet targeting it with automatic deployment and retain-on-removal enabled. Move the member account into the OU and watch the instance appear, then move it out and confirm the stack is retained but no longer listed. If you only have one account, read the "Create CloudFormation StackSets with service-managed permissions" page and the Control Tower Account Factory customization docs instead. *Cost/cleanup:* Free for StackSets; closing a test account takes a waiting period, so plan before creating it.

### Practice questions: Task 5.1

**Question 5.1.1** A bank runs 40 workload accounts in AWS Organizations across two Regions. Application teams deploy with raw CloudFormation templates and with AWS CDK, and the platform team cannot review every template. A new policy requires that no Amazon RDS DB instance can be provisioned through infrastructure code unless storage encryption is enabled and backup retention is at least 7 days. Non-compliant deployments must be stopped before any resource is created, and the platform team wants to roll the control out without blocking teams on day one. Which solution meets these requirements with the least operational overhead?

- A. Deploy the AWS Config managed rules for RDS storage encryption and backup retention through an organization conformance pack, with automatic remediation that runs an SSM Automation runbook to modify non-compliant instances.
- B. Publish an internal CDK construct library with an aspect that fails synthesis for non-compliant DB instances, and require all teams to add the aspect to their CDK apps through a pull request template.
- C. Write CloudFormation Guard rules for `AWS::RDS::DBInstance`, store them in S3, and deploy a Guard Hook to every account and Region with a service-managed StackSet, starting in WARN mode and switching to FAIL after review.
- D. Attach an SCP to the workload OUs that denies `rds:CreateDBInstance` unless the request includes the `rds:StorageEncrypted` condition key set to true and a backup retention value of at least 7.

<details>
<summary>Answer and explanation</summary>

**Correct answer: C**

**Why C is correct:** A Guard Hook evaluates each `AWS::RDS::DBInstance` in a CloudFormation operation before it is provisioned, which covers both raw templates and CDK apps because CDK deploys through CloudFormation. Guard rules can check any template property, including `StorageEncrypted` and `BackupRetentionPeriod`. Hooks are account and Region scoped, so a service-managed StackSet distributes the Hook to all 40 accounts and both Regions, including new accounts with automatic deployment. `FailureMode: WARN` lets the platform team watch the invocation results first, then switch to `FAIL`.

**Why the other options are incorrect:**

- **A:** AWS Config evaluates resources after they exist, so the requirement to stop the deployment before creation fails. Remediating backup retention after creation is possible, but you cannot turn on storage encryption for an existing unencrypted instance in place.
- **B:** Aspects run only in CDK apps that include them. Teams using raw CloudFormation, or any team that omits the aspect, bypass the control.
- **D:** SCPs can only test the condition keys that RDS exposes on the API request, so they cannot validate arbitrary resource properties the way a template rule can, and an SCP has no warn-only mode for a gradual rollout.

**Key takeaway:** "Before provisioning, for everything deployed through CloudFormation, across accounts" points to CloudFormation Hooks distributed by StackSets or Control Tower proactive controls.

</details>

**Question 5.1.2** An operations team manages a fleet of networking stacks with CloudFormation. During incidents, engineers sometimes edit security group rules and route table entries in the console, and once an engineer deleted a VPC endpoint by mistake. The team wants a repeatable procedure that shows exactly which live properties differ from the templates, lets a reviewer approve the changes, and then restores template values and recreates deleted resources without rebuilding the stacks. Which approach meets these requirements?

- A. Run drift detection on each stack, then create a drift-aware change set with the deployment mode set to revert drift, review the listed drifted properties and recreations, and execute the change set after approval.
- B. Deploy the AWS Config managed rule that checks CloudFormation stack drift, and attach an automatic remediation runbook that calls `UpdateStack` with the currently deployed template for each drifted stack.
- C. Use CloudFormation stack refactoring to move the drifted resources into a new stack, then import the deleted VPC endpoint with a resource import operation and delete the original stack.
- D. Enable termination protection and a stack policy that denies all updates on the networking stacks, and run a nightly CodeBuild job that deletes and recreates any stack that reports drift.

<details>
<summary>Answer and explanation</summary>

**Correct answer: A**

**Why A is correct:** Drift-aware change sets compare the new template, the last deployed template and the actual resource state, so the change set preview lists each out-of-band change, which gives the reviewer something to approve. Executing it in revert drift mode sets drifted properties back to template values and recreates resources that were deleted outside CloudFormation, which covers the deleted VPC endpoint. The stacks are updated in place, not rebuilt.

**Why the other options are incorrect:**

- **B:** The Config rule detects drift, but a normal update with the same template compares only against the last deployed template. CloudFormation reports no changes and does not revert the live properties. There is also no human review step before the automatic remediation.
- **C:** Stack refactoring moves resources between stacks and renames logical IDs. It does not revert property drift. Resource import needs an existing resource, and the endpoint was deleted.
- **D:** Stack policies control updates through CloudFormation; they do not stop console changes. Deleting and recreating networking stacks causes outages and breaks the "without rebuilding" requirement.

**Key takeaway:** Plain redeploys ignore drift; drift-aware change sets with revert drift mode are the native way to preview and restore template state.

</details>

**Question 5.1.3** A company deploys an Amazon RDS for PostgreSQL instance and an AWS Lambda function with CloudFormation from a CodePipeline pipeline. Security requires that the database password never appears in templates, pipeline variables or artifacts, is rotated every 30 days without a redeploy of the stack, and is available in a second Region for a warm standby. The Lambda function must always use the current password. Which solution meets these requirements?

- A. Create a SecureString parameter in Parameter Store from the template with a generated value, reference it in the DB instance with an `ssm-secure` dynamic reference, and schedule an EventBridge rule that calls a Lambda function every 30 days to update the parameter and the database.
- B. Pass the password into the stack as a `NoEcho` parameter from a CodePipeline variable, store it in the Lambda function environment variables, and rotate it by updating the pipeline variable and redeploying every 30 days.
- C. Store the password in an advanced tier SecureString parameter created by the security team, reference it with an `ssm-secure` dynamic reference, and use a parameter policy with an expiration notification to remind the team to rotate it.
- D. Create an `AWS::SecretsManager::Secret` with `GenerateSecretString` and a replica Region, attach it to the DB instance with a secret target attachment, add a 30-day rotation schedule, reference it with a `secretsmanager` dynamic reference, and have the function read the secret at runtime through a caching client.

<details>
<summary>Answer and explanation</summary>

**Correct answer: D**

**Why D is correct:** `GenerateSecretString` creates the password inside Secrets Manager, so it never exists in the template or the pipeline. The dynamic reference resolves the value at deploy time without storing it. The rotation schedule rotates the secret every 30 days through a rotation function with no stack update. Replica Regions keep a copy in the standby Region. Reading the secret at runtime with a caching client means the function picks up the rotated value.

**Why the other options are incorrect:**

- **A:** CloudFormation cannot create SecureString parameters, so the first step is not possible. A custom rotation Lambda function also reimplements what Secrets Manager provides.
- **B:** The password lives in a pipeline variable and in the function's environment, which violates the "never in pipeline variables" rule, and rotation requires a redeploy.
- **C:** The parameter policy only sends a notification. Nothing rotates the password, and Parameter Store has no native cross-Region replication.

**Key takeaway:** Rotation plus cross-Region availability plus no plaintext in IaC means Secrets Manager with generated secrets, rotation schedules, replicas and dynamic references.

</details>

**Question 5.1.4** A company uses AWS Organizations with a delegated administrator account for CloudFormation StackSets. A security baseline template (IAM roles for auditors and an Amazon EventBridge rule that forwards findings to a central bus) must be in every member account in both used Regions, including accounts created in the future, with no manual steps. The same baseline is also required in the management account and in an acquired company's account that will stay outside the organization for a year. Which combination of steps meets these requirements? (Select TWO.)

- A. Create one service-managed StackSet from the delegated administrator account that targets the organization root and adds the acquired company's account ID as an additional target with a union account filter.
- B. Create a service-managed StackSet from the delegated administrator account that targets the organization root in both Regions, with automatic deployment enabled.
- C. Create a self-managed StackSet for all member accounts, and add an EventBridge rule on the `CreateAccountResult` event that calls a Lambda function to add each new account to the StackSet.
- D. Rely on the service-managed StackSet that targets the organization root to also cover the management account, because the management account is a member of the root.
- E. Create a self-managed StackSet with the administration role in the delegated administrator account and the execution role in the management account and the acquired account, targeting those two accounts in both Regions.

<details>
<summary>Answer and explanation</summary>

**Correct answers: B and E**

**Why B and E are correct:** A service-managed StackSet targeting the root with automatic deployment covers every current member account and deploys to new accounts as they join, with no manual steps (B). Service-managed StackSets never deploy to the management account and cannot target accounts outside the organization, so those two accounts need a self-managed StackSet, where you create the execution role in each target and can target any account ID (E).

**Why the other options are incorrect:**

- **A:** Account filters in service-managed StackSets only select accounts within the targeted OUs of the organization. An account outside the organization cannot be a target.
- **C:** This works technically but adds a custom Lambda function and event plumbing to replicate what automatic deployment already does, and a self-managed StackSet needs the execution role to exist in each new account before deployment, which is another manual or custom step.
- **D:** CloudFormation does not deploy service-managed stack instances to the management account, even when you target the root.

**Key takeaway:** Service-managed StackSets for organization members and future accounts; self-managed for the management account and anything outside the organization.

</details>

**Question 5.1.5** A platform team uses AWS CDK and a CodePipeline pipeline in a dedicated tools account to deploy application stacks into development, staging and production accounts. The security team has rejected the current setup, where each target account was bootstrapped with default settings and the pipeline uses long-term access keys of an IAM user in each account. They require that the pipeline uses only temporary credentials, that only the tools account can deploy, and that CloudFormation in production runs with a scoped permissions policy rather than AdministratorAccess. Which solution meets these requirements with the least operational overhead?

- A. Keep the IAM users but store their access keys in Secrets Manager with 30-day rotation, and add a permissions boundary to each user that matches the scoped policy.
- B. Re-bootstrap each target account with `cdk bootstrap --trust <tools account ID>` and `--cloudformation-execution-policies` set to a customer managed policy, and let the pipeline assume the bootstrap deploy roles.
- C. Convert the CDK apps to plain templates with `cdk synth` and deploy them from the tools account as a self-managed StackSet with a custom execution role in each target account.
- D. Create a cross-account IAM role in each target account with AdministratorAccess that trusts the tools account, and add an SCP on the production OU that denies IAM actions to that role.

<details>
<summary>Answer and explanation</summary>

**Correct answer: B**

**Why B is correct:** Bootstrapping with `--trust` creates deploy, publishing and lookup roles in each target account that the tools account can assume with temporary credentials, and no other account is trusted. `--cloudformation-execution-policies` replaces the default AdministratorAccess on the CloudFormation execution role with the scoped policy. CDK Pipelines and `cdk deploy` already use these roles, so there is nothing custom to maintain.

**Why the other options are incorrect:**

- **A:** Rotated access keys are still long-term credentials, so the temporary credentials requirement fails.
- **C:** StackSets can deploy synthesized templates, but CDK assets (Lambda code, container images) then need a separate publishing process to each account, and you lose the CDK pipeline's per-environment stages and approvals. This adds overhead.
- **D:** The role still has AdministratorAccess. An SCP that denies IAM actions breaks legitimate stacks that create IAM roles and does not give CloudFormation the scoped policy required.

**Key takeaway:** For cross-account CDK, bootstrap with `--trust` and a least-privilege `--cloudformation-execution-policies`; never use IAM user keys in pipelines.

</details>

## Task 5.2: Design CI/CD and DevSecOps pipelines

This task tests whether you can assemble a pipeline that is triggered precisely, scans and signs what it builds, promotes the same artifact across accounts and Regions safely, evaluates AI outputs before release, and rolls back on its own. The recurring decision is which native feature (CodePipeline V2 triggers and conditions, InspectorScan, ECR managed signing, CodeDeploy or ECS native deployment strategies) removes custom glue code.

### Skill 5.2.1: Orchestrate build and deployment pipelines (CodePipeline, CodeBuild, CodeDeploy, CodeCommit)

**Key concepts**

- **AWS CodeCommit status.** In July 2024 AWS stopped onboarding new customers to CodeCommit. On 24 November 2025 AWS reversed that and returned CodeCommit to full general availability, with sign-ups reopened. It is again a valid answer for a private Git repository inside AWS with IAM authentication and no third-party dependency. GitHub, GitLab and Bitbucket connect through AWS CodeConnections (formerly CodeStar Connections).
- **CodePipeline V2 vs V1.** V2 is the type to choose for new pipelines. V2 adds:
  - Triggers with Git filters for connection-based sources: push filters on branches, file paths and tags, and pull request filters on events (opened, updated, closed), branches and file paths. Filter types inside one entry are ANDed; separate entries are ORed. This is the native answer for monorepos.
  - Pipeline-level variables with defaults that can be overridden when a run starts.
  - Execution modes: `QUEUED` (runs in order, no run is skipped), `SUPERSEDED` (a newer run can overtake an older one waiting between stages, the V1 behavior) and `PARALLEL` (independent runs, useful for feature branches). In `PARALLEL` mode, stage rollback and rollback-type conditions are not available.
  - Stage rollback to the last successful pipeline execution, manual or automatic on failure.
  - Stage conditions: entry conditions (result Fail or Skip), on-failure conditions (result Rollback), on-success conditions (Rollback or Fail). Rule types: `CloudWatchAlarm`, `DeploymentWindow` (cron-defined change windows), `LambdaInvoke`, `VariableCheck` and `Commands`.
  - Pricing: V1 is charged per active pipeline per month; V2 is charged per action execution minute. Low-frequency or many-pipeline estates often get cheaper on V2; check the pricing page for your volume.
- **Commands action** runs shell commands on managed compute inside the pipeline without defining a separate CodeBuild project. Use CodeBuild projects when you need custom images, VPC access, caching, batch builds or reports.
- **CodeBuild** runs in a managed container, optionally inside your VPC (needs NAT or VPC endpoints for outbound access). Reserved capacity fleets give warm, non-ephemeral capacity for faster starts. CodeBuild can also act as a self-hosted runner for GitHub Actions and GitLab.
- **CodeDeploy** compute platforms:
  - EC2 and on-premises: in-place or blue/green (blue/green requires Auto Scaling groups or tagged EC2 instances, not on-premises). Deployment configurations are `AllAtOnce`, `HalfAtATime` and `OneAtATime` (default), or custom minimum healthy hosts. On-premises servers need the CodeDeploy agent and registration with IAM credentials.
  - Lambda: traffic shifting between alias versions with canary, linear or all-at-once configurations, plus `BeforeAllowTraffic` and `AfterAllowTraffic` hooks.
  - Amazon ECS: blue/green task sets with canary, linear or all-at-once traffic shifting.
- Use AWS CodeArtifact for package dependencies (npm, PyPI, Maven) with upstream repositories and a domain shared across accounts.

**Exam traps**

- Saying CodeCommit is unavailable to new customers. That was true from July 2024 to November 2025 but is no longer true.
- Building an EventBridge rule plus Lambda function to filter monorepo changes or tags when a V2 trigger filter does it natively.
- Choosing `SUPERSEDED` when every commit must deploy in order (for example, database migrations). Use `QUEUED`.
- Expecting stage rollback in a `PARALLEL` execution mode pipeline.
- Using CodeDeploy blue/green for on-premises servers. On-premises supports in-place only.

**Hands-on**

1. **Monorepo trigger and conditions.** Connect a GitHub repository through CodeConnections and create a V2 pipeline with a push trigger that includes branch `main` and file path `services/api/**`, plus a second trigger for tags `v*`. Add a deploy stage with an entry condition using a `DeploymentWindow` rule and an on-failure condition with result Rollback. Push changes inside and outside the path and tag a commit to see which runs start, then break the deploy to watch the rollback. *Cost/cleanup:* V2 is billed per action execution minute and CodeBuild per build minute; a few runs cost cents. Delete the pipeline, artifact bucket, CodeBuild project and connection.
2. **CodeDeploy canary for Lambda.** Deploy a SAM application with `AutoPublishAlias: live` and `DeploymentPreference: Type: Canary10Percent5Minutes`, plus a CloudWatch alarm on the function's `Errors` metric listed under `Alarms`. Deploy a version that throws on 20 percent of calls, invoke it in a loop, and watch CodeDeploy shift 10 percent, the alarm fire and traffic return to the old version. *Cost/cleanup:* Within free tier. `sam delete` removes the stack, CodeDeploy application and alarm.

### Skill 5.2.2: Security scanning and artifact signing in pipelines

**Key concepts**

- **Amazon ECR scanning.** Basic scanning scans on push or manually for OS package CVEs. Enhanced scanning uses Amazon Inspector, scans OS and programming language packages, and rescans continuously as new CVEs are published (continuous or on-push duration settings). Enhanced findings go to Inspector and Security Hub; EventBridge events let you react.
- **Amazon Inspector in CI/CD (shift left).** The Inspector SBOM Generator builds a CycloneDX SBOM from a container image, archive, directory or compiled Go and Rust binaries, and the Inspector Scan API (`inspector-scan:ScanSbom`) returns vulnerabilities. Plugins exist for Jenkins, TeamCity and GitHub Actions, with GitLab and CodeCatalyst integrations. You do not have to push to ECR first.
- **CodePipeline `InspectorScan` action.** Run mode `SourceCodeScan` scans a source artifact from a third-party repository; `ECRImageScan` scans a named ECR repository and tag. You set thresholds per severity (critical, high, medium, low); the action fails when the count exceeds the threshold. It outputs an SBOM artifact and a `HighestScannedSeverity` variable that later actions or a `VariableCheck` rule can use. It runs on CodeBuild compute billed separately.
- **Image signing.** AWS Signer manages signing keys and certificates for container images using the Notation (Notary Project) format.
  - ECR managed signing (November 2025): you create a signing rule that names an AWS Signer signing profile and the repositories to cover. ECR signs images automatically on push. Signing calls are logged in CloudTrail. This is the least-overhead option.
  - Manual signing: Notation CLI with the AWS Signer plugin signs before or after push. Use it when signing must happen outside the push path or you need fine control.
  - Signatures are stored as OCI reference artifacts in the same repository and count against the images-per-repository quota. Lifecycle policies clean up signatures of deleted images.
  - Verification at deploy time on Amazon EKS uses an admission controller that understands Notation signatures with the AWS Signer plugin, such as Kyverno or Ratify with Gatekeeper. Signer profiles can be revoked, which invalidates their signatures during verification.
- CodeGuru Reviewer is closed to new customers, so for code scanning prefer the Inspector `SourceCodeScan` mode or third-party SAST tools in CodeBuild. Secrets in repositories: scan in CodeBuild with an open-source tool and block merges.
- Lambda code signing: AWS Signer profiles plus a code signing configuration on the function reject unsigned or tampered zip packages (enforce or warn).

**Exam traps**

- Relying on ECR basic scanning to find vulnerable application libraries (for example a Java JAR). Basic scanning covers OS packages; use enhanced scanning or Inspector CI/CD scanning.
- Treating an on-push scan as continuous protection. A clean image today can have critical CVEs tomorrow; enhanced continuous scanning covers that.
- Saying ECR image tag immutability proves provenance. It prevents tag overwrite; it does not prove who built the image. Signatures do.
- Picking Amazon GuardDuty to block unsigned images. GuardDuty detects runtime threats; admission control blocks deployment.

**Hands-on**

1. **Fail a pipeline on critical CVEs.** Build an image from an old base (for example an outdated Python image) in CodeBuild and push it to ECR. Add an `InspectorScan` action with `ECRImageScan` mode and a critical threshold of 0, run the pipeline and see it fail with the SBOM artifact and `HighestScannedSeverity` set. Change the base to a current slim image and rerun. *Cost/cleanup:* Inspector scan charges per image scanned plus CodeBuild minutes; a few scans are low cost. Delete the ECR images and pipeline; disable Inspector if you enabled it only for this.
2. **Managed signing and verification.** Create an AWS Signer signing profile for the Notation container platform, then an ECR signing rule that applies it to one repository. Push an image and list its referrers to see the signature artifact, then install Notation with the AWS Signer plugin locally, create a trust policy for the profile and run `notation verify`. Revoke the profile and verify again to see the failure. *Cost/cleanup:* Check AWS Signer pricing for container signatures; a handful of images is low cost. Delete images, the signing rule and the profile.

### Skill 5.2.3: Multi-account and multi-Region pipelines with role delegation and artifact promotion

**Key concepts**

- Standard pattern: pipeline in a tools (deployment) account, deploy actions in development, staging and production accounts. Build once, promote the same artifact; never rebuild per environment.
- Requirements for cross-account actions:
  - The artifact bucket must be encrypted with a customer managed KMS key whose key policy allows the target accounts. The AWS managed `aws/s3` key cannot be shared across accounts. Reference the key by ARN, not alias.
  - The artifact bucket policy grants the target accounts read (and write for output artifacts).
  - A role in each target account trusts the pipeline's service role; the action's `roleArn` points to it. For CloudFormation actions there are two roles: the action role that CodePipeline assumes and the CloudFormation service role (`RoleArn` in configuration) that CloudFormation uses to create resources.
  - The pipeline service role needs `sts:AssumeRole` on those roles.
  - Artifacts can only move between the pipeline account and another account, not between two non-pipeline accounts.
- Cross-Region actions need an artifact store (bucket and key) in each Region used; use `artifactStores` instead of `artifactStore`. CDK Pipelines creates these support stacks for you.
- Container promotion: replicate ECR images with ECR replication rules (cross-Region and cross-account, with repository filters) or grant cross-account pull with repository policies. Promote by digest, not tag, so production runs exactly what was tested.
- Approvals: manual approval actions with SNS notifications, or automated gates with stage conditions (`CloudWatchAlarm`, `LambdaInvoke`, `DeploymentWindow`).
- CDK Pipelines (`pipelines.CodePipeline`) models waves and stages across accounts and Regions, self-mutates when the pipeline definition changes, and uses bootstrap roles with `--trust`.
- StackSets as a deploy action deploy one template to many accounts in one stage; use it when the same baseline goes to dozens of accounts rather than a handful of named environments.

**Exam traps**

- Leaving the artifact bucket on the default AWS managed key and adding only an IAM role in the target account. The target role gets access denied from KMS.
- Rebuilding the image or package in each account. It breaks the "same artifact" requirement and can pull different dependency versions.
- Using one IAM user's keys per environment for deployments. Use role assumption.
- Forgetting the regional artifact store for cross-Region actions.

**Hands-on**

1. **Cross-account CloudFormation deploy (two accounts).** In a tools account, create a customer managed KMS key and a pipeline whose artifact store uses it. In a second free account, create an action role trusted by the pipeline role and a CloudFormation service role. Update the pipeline JSON with `roleArn` on the deploy action and the key ARN, deploy a small stack (an SNS topic), then switch to the AWS managed key and observe the KMS access denied failure. With one account only, follow the "Create a pipeline that uses resources from another AWS account" doc and inspect the policies instead. *Cost/cleanup:* KMS key is about a dollar per month prorated; schedule its deletion. Delete stacks, pipeline and roles.
2. **CDK Pipelines with a cross-Region wave.** Bootstrap two Regions with `--trust` for your own account, create a CDK Pipelines app with a `Wave` that deploys the same stage to both Regions, and push a change. Inspect the support stacks CDK creates for the regional artifact buckets and keys. *Cost/cleanup:* Pipeline minutes and two KMS keys; destroy the app stacks, the pipeline stack and the support stacks.

### Skill 5.2.4: Evaluate AI/ML applications and validate outputs (Amazon Bedrock Evaluations)

**Key concepts**

- Amazon Bedrock Evaluations job types:
  - Automatic (programmatic) model evaluation with built-in or custom prompt datasets and metrics such as accuracy, robustness and toxicity.
  - LLM-as-a-judge model evaluation: an evaluator model scores responses on metrics such as correctness, completeness, faithfulness, harmfulness and answer refusal. Custom metrics with your own judge prompts are supported.
  - Human evaluation with your own work team or an AWS managed team, for subjective criteria such as style, tone or brand voice.
  - RAG evaluation for Bedrock Knowledge Bases or your own RAG system: retrieve-only (context relevance, context coverage) or retrieve-and-generate (faithfulness, correctness, completeness and others).
  - Bring your own inference responses: evaluate outputs produced by a model or RAG system hosted anywhere, including SageMaker AI or another provider, by supplying the responses in the dataset.
- Compare jobs side by side to see the effect of a prompt, model or chunking change.
- Amazon Bedrock AgentCore Evaluations (generally available March 2026) evaluates agents: on-demand evaluation for regression tests in CI/CD and online evaluation that samples live production traces. It has built-in evaluators (for example correctness, faithfulness, helpfulness, harmfulness, goal success rate, tool selection accuracy and tool parameter accuracy), ground truth support (reference answers, expected tool sequences), and custom evaluators that are LLM-based or code-based in Lambda.
- Pipeline pattern: a CodeBuild or Lambda step starts an evaluation job on a curated golden dataset in S3, waits for completion, parses the scores, and publishes them as output variables; a `VariableCheck` or `LambdaInvoke` condition blocks promotion when scores fall below thresholds.
- Evaluation is a pre-release (and sampled production) quality gate. Bedrock Guardrails is a runtime filter on each request. They complement each other; one does not replace the other.
- SageMaker AI equivalents for custom models: SageMaker Clarify for bias and explainability, Model Monitor for drift after deployment, shadow tests to compare a new variant on live traffic.

**Exam traps**

- Proposing human evaluation for every release when the requirement is automated and fast. Use LLM-as-a-judge, with periodic human review.
- Using Guardrails contextual grounding checks as the "evaluation before release". They block ungrounded responses at runtime; they do not produce comparative scores across candidate configurations.
- Using SageMaker Model Monitor to compare two RAG chunking strategies. Model Monitor watches deployed endpoint data and model quality over time.
- Assuming Bedrock Evaluations works only for Bedrock-hosted models. Bring-your-own-response jobs cover external models.

**Hands-on**

1. **LLM-as-a-judge comparison.** Create a JSONL dataset of 20 prompts with reference answers from a domain you know (for example KYC policy questions). Run two LLM-as-a-judge evaluation jobs with different generator models or prompts, select correctness, completeness and faithfulness, and compare the jobs in the console. *Cost/cleanup:* You pay for generator and evaluator model tokens; 20 prompts is cents to low dollars. Delete the S3 output.
2. **RAG evaluation of a knowledge base.** Build a small Bedrock knowledge base over a few PDFs using an inexpensive vector store option, then run a retrieve-and-generate RAG evaluation job with a prompt dataset. Change the chunking strategy, re-sync and rerun, and compare context relevance and faithfulness. *Cost/cleanup:* Vector stores such as OpenSearch Serverless have a minimum hourly charge that adds up quickly; choose a lower-cost supported store if available to you and delete the knowledge base, vector store and S3 data the same day.

### Skill 5.2.5: Deployment strategies with rollback (blue/green, canary, rolling)

**Key concepts**

- **Amazon ECS deployment options.**
  - Rolling update (ECS controller) with `minimumHealthyPercent` and `maximumPercent`, plus the deployment circuit breaker that rolls back to the last completed deployment when tasks fail to reach steady state, and optional CloudWatch alarm-based rollback.
  - ECS native blue/green (2025) and built-in linear and canary (October 2025): ECS shifts traffic between revisions itself, with step bake time, canary bake time and a deployment bake time that keeps the old revision for fast rollback. Rollback on CloudWatch alarms, lifecycle hooks backed by Lambda for validation. Works with Application Load Balancer or ECS Service Connect. AWS recommends this as the default for new services.
  - CodeDeploy blue/green for ECS: canary, linear or all-at-once with `CodeDeployDefault.ECSCanary10Percent5Minutes`, `ECSCanary10Percent15Minutes`, `ECSLinear10PercentEvery1Minutes`, `ECSLinear10PercentEvery3Minutes` and `ECSAllAtOnce`. With a Network Load Balancer only `ECSAllAtOnce` is supported. Lifecycle hook time is capped for the whole deployment. Keep it where pipelines and governance already depend on CodeDeploy.
- **Lambda.** CodeDeploy with alias traffic shifting: canary (10 percent for 5, 10, 15 or 30 minutes), linear (10 percent every 1, 2, 3 or 10 minutes) or all at once, rollback on alarms or hook failure.
- **EC2.** CodeDeploy in-place or blue/green with automatic rollback on deployment failure or alarms. For Auto Scaling groups, instance refresh replaces instances with a new launch template, honors minimum healthy percentage, supports checkpoints, skip matching and automatic rollback on failure or alarms.
- **API Gateway** canary release settings on a stage split traffic between the deployed and canary deployment. **CloudFront** continuous deployment uses a staging distribution with weight or header-based routing for edge configuration changes.
- **Infrastructure:** CloudFormation rollback triggers watch alarms during a stack update. CodePipeline V2 stage rollback and on-success conditions with `CloudWatchAlarm` rules roll the whole stage back after deployment.
- **Database changes** do not roll back with traffic. Use backward-compatible expand-and-contract migrations so the old version can run against the new schema.
- Choosing: blue/green when you need instant cutback and can afford double capacity; canary when you want to limit blast radius with real traffic; linear for gradual confidence; rolling when capacity cost matters and brief mixed versions are acceptable.

**Exam traps**

- Using Route 53 weighted records for canary on internal service traffic. DNS caching makes shifts and rollbacks slow and uneven; use load balancer or Service Connect level shifting.
- Choosing CodeDeploy canary for an ECS service behind a Network Load Balancer. Only all-at-once is supported there.
- Assuming the ECS deployment circuit breaker gives percentage-based traffic shifting. It only detects failed rollouts and rolls back.
- Ignoring schema compatibility. Rolling back the application after a destructive migration breaks the old version.

**Hands-on**

1. **ECS native canary with alarm rollback.** Create a Fargate service behind an ALB with the deployment strategy set to canary (for example 10 percent), a canary bake time of a few minutes and a deployment alarm on the target group's 5XX count. Deploy a revision that returns errors, generate traffic with a simple loop, and watch ECS roll back without operator action. *Cost/cleanup:* ALB hourly charge plus two small tasks for an hour is under a few dollars. Delete the service, ALB, target groups and cluster.
2. **Instance refresh with automatic rollback.** Create an Auto Scaling group of two `t3.micro` instances behind an ALB, then start an instance refresh to a launch template version whose user data breaks the web server, with automatic rollback enabled and a checkpoint at 50 percent. Watch health checks fail and the group roll back to the previous launch template. *Cost/cleanup:* Free tier eligible instances plus the ALB hourly charge. Delete the group, launch template and ALB.

### Practice questions: Task 5.2

**Question 5.2.1** A payments company keeps 30 microservices in one GitHub monorepo. Each service has its own CodePipeline pipeline that currently starts on every push to `main`, so a single change triggers 30 runs and wastes build minutes. The company wants each pipeline to start only when files under that service's folder change on `main`, and release pipelines to start only when a tag matching `release-*` is pushed. Every qualifying commit must deploy in order because some releases include database migrations. Which solution meets these requirements with the least operational overhead?

- A. Keep V1 pipelines, disable change detection, and create an EventBridge rule for each service that invokes a Lambda function to inspect the commit's changed files through the GitHub API and start the matching pipeline.
- B. Use CodeBuild webhook filter groups on the repository with file path and tag patterns, and have each CodeBuild project call `StartPipelineExecution` for its pipeline at the end of the build.
- C. Split the monorepo into one repository per service in AWS CodeCommit, with one V1 pipeline per repository and change detection on `main`, and create tag-based release branches for release pipelines.
- D. Convert the pipelines to V2, use a CodeConnections source with push triggers filtered by branch `main` and the service's file path, add tag filters for `release-*` on release pipelines, and set the execution mode to `QUEUED`.

<details>
<summary>Answer and explanation</summary>

**Correct answer: D**

**Why D is correct:** V2 pipeline triggers support push filters on branches, file paths and tags for connection-based Git sources, so each pipeline starts only for its folder, and release pipelines start only on matching tags. No custom code is needed. `QUEUED` execution mode processes every run in order instead of letting a newer run supersede an older one, which protects the ordered database migrations.

**Why the other options are incorrect:**

- **A:** It works but adds 30 rules and custom Lambda code that calls the GitHub API, which is the operational overhead the question tells you to avoid. V1 also lacks Git trigger filters.
- **B:** It moves filtering into CodeBuild and still runs extra builds whose only job is to start pipelines. It also splits pipeline start logic across two services, which is harder to operate.
- **C:** Splitting the monorepo is a large change to developer workflow and does not provide tag triggers. It also runs V1 pipelines in superseded mode, so ordered execution is not guaranteed.

**Key takeaway:** For monorepos and tag-driven releases, use CodePipeline V2 trigger filters, and pick the execution mode that matches ordering needs.

</details>

**Question 5.2.2** A financial services company builds container images in CodePipeline and deploys them to Amazon EKS clusters in three production accounts. Auditors require that (1) the pipeline stops before deployment if an image has any critical vulnerability in OS or application packages, (2) every image is cryptographically signed by an AWS-managed signing identity with no signing keys handled by the build team, and (3) the clusters refuse to run images without a valid signature from that identity. Which combination of steps meets these requirements with the least operational overhead? (Select TWO.)

- A. Add a CodePipeline `InspectorScan` action in `ECRImageScan` mode with a critical threshold of 0 before the deploy stage.
- B. Turn on ECR basic scanning on push for the repositories and add a lifecycle policy that expires images with critical findings.
- C. Create an ECR managed signing rule that uses an AWS Signer signing profile for the repositories, and run an admission controller such as Kyverno or Ratify with the AWS Signer plugin on each cluster to verify signatures.
- D. Enable Amazon GuardDuty EKS Runtime Monitoring in the production accounts and configure it to terminate pods whose images lack a signature.
- E. Generate an asymmetric AWS KMS key, have a CodeBuild step sign the image digest with the `Sign` API, and store the signature in an image tag that a deployment script checks before running `kubectl apply`.

<details>
<summary>Answer and explanation</summary>

**Correct answers: A and C**

**Why A and C are correct:** The `InspectorScan` action scans the ECR image with Amazon Inspector, which covers OS and programming language packages, and fails the action when the critical count exceeds the threshold, so the deploy stage never runs (A). ECR managed signing signs each image on push with an AWS Signer profile, and Signer manages key material and certificates, so the build team handles no keys. An admission controller that verifies Notation signatures with the AWS Signer plugin rejects unsigned or wrongly signed images at the cluster (C).

**Why the other options are incorrect:**

- **B:** Basic scanning focuses on OS packages and would miss application library CVEs. A lifecycle policy cannot filter on scan findings, so it cannot expire vulnerable images.
- **D:** GuardDuty Runtime Monitoring detects threats in running workloads. It does not verify image signatures or block admission.
- **E:** It is a custom signing scheme. The build team must manage key usage and a bespoke tag format, and a deployment script check can be bypassed by anyone deploying outside the script, so the cluster itself does not refuse unsigned images.

**Key takeaway:** Scan in the pipeline with Inspector, sign with ECR managed signing and AWS Signer, and verify at admission on EKS.

</details>

**Question 5.2.3** A company has a CodePipeline pipeline in a tools account in us-east-1. It builds a CloudFormation template once and must deploy it to a staging account and then to a production account, both in us-east-1 and eu-west-1. The first cross-account deploy action fails with an access denied error when the target account reads the input artifact. The artifact bucket uses default encryption with the AWS managed key. The security team forbids IAM users and requires that production resources are created with a role that only has the permissions the template needs. Which solution meets these requirements?

- A. Grant the staging and production accounts access to the artifact bucket through a bucket policy, keep the AWS managed key, and give the deploy action an IAM user's access keys stored in Secrets Manager.
- B. Encrypt the artifact stores with customer managed KMS keys that allow the target accounts, add bucket policies, add an artifact store in eu-west-1, and set each deploy action's `roleArn` to a role in the target account that passes a scoped CloudFormation service role.
- C. Replace the deploy actions with a CodeBuild action in each target account that clones the source repository and rebuilds the template, then deploys it with the CodeBuild service role in that account.
- D. Enable S3 Cross-Region Replication from the artifact bucket to buckets in the target accounts, and point CloudFormation actions at the replicated objects using the pipeline's own service role.

<details>
<summary>Answer and explanation</summary>

**Correct answer: B**

**Why B is correct:** Cross-account actions need a customer managed KMS key whose key policy lets the target accounts decrypt, because the AWS managed key cannot be shared across accounts, which is the cause of the access denied error. Bucket policies grant read access to the artifacts. Cross-Region actions need an artifact store in each Region, so eu-west-1 needs its own bucket and key. The action `roleArn` uses temporary credentials in the target account, and the CloudFormation service role limits what the stack can create.

**Why the other options are incorrect:**

- **A:** The AWS managed key still blocks decryption, and IAM user keys break the security rule.
- **C:** Rebuilding in each account breaks build-once promotion; the template deployed to production is not the one tested in staging, and dependencies can drift.
- **D:** Replication does not solve KMS access to objects encrypted with the AWS managed key, and the pipeline's own service role cannot create resources in another account without assuming a role there.

**Key takeaway:** Cross-account CodePipeline needs a customer managed KMS key, bucket policy, a target account role on the action, and an artifact store per Region.

</details>

**Question 5.2.4** A bank runs a retrieval-augmented chatbot that answers staff questions from policy documents in an Amazon Bedrock knowledge base. The team wants every release to be scored automatically before promotion, comparing a new chunking strategy and a new prompt against the current configuration for faithfulness to retrieved content, correctness and completeness. One candidate generator model is hosted on Amazon SageMaker AI, and its responses must be scored in the same way. Releases must be blocked when faithfulness drops below a threshold. Which solution meets these requirements with the least operational overhead?

- A. Create Amazon Bedrock Guardrails with contextual grounding checks at a high threshold, deploy the new configuration to production, and roll back if the number of blocked responses rises in CloudWatch.
- B. Create a human evaluation job in Amazon Bedrock with the bank's compliance team as the work team for every release, and have the release manager approve the pipeline stage after reviewing the scores.
- C. Deploy both configurations to SageMaker AI endpoints, enable SageMaker Model Monitor model quality monitoring with ground truth labels, and promote the configuration with the lower drift.
- D. Add a pipeline step that runs Bedrock RAG evaluation jobs (retrieve and generate) with an LLM judge on a golden dataset, using bring-your-own inference responses for the SageMaker model, and block promotion with a `VariableCheck` condition on the faithfulness score.

<details>
<summary>Answer and explanation</summary>

**Correct answer: D**

**Why D is correct:** RAG retrieve-and-generate evaluation scores end-to-end answers on faithfulness, correctness and completeness with an LLM judge, which is automated and repeatable for each release. Bring-your-own inference responses let the job score answers produced by the SageMaker-hosted model with the same metrics. Publishing the score as a variable and checking it with a `VariableCheck` rule gates promotion without manual work.

**Why the other options are incorrect:**

- **A:** Guardrails filters responses at runtime in production. It does not compare candidate configurations before release, and testing in production breaks the "before promotion" requirement.
- **B:** Human evaluation works for subjective criteria but adds reviewer time to every release, which is not automatic and not least overhead.
- **C:** Model Monitor watches deployed endpoint data and quality over time. It does not score RAG faithfulness against retrieved context, and it requires deploying both candidates first.

**Key takeaway:** Use Bedrock Evaluations (RAG and LLM-as-a-judge, with bring-your-own responses) as an automated pre-release gate; Guardrails is runtime protection.

</details>

**Question 5.2.5** A company is launching a new Amazon ECS service on AWS Fargate. External traffic arrives through an Application Load Balancer, and other internal services call it through ECS Service Connect. Releases must send 10 percent of traffic to the new revision for 15 minutes, roll back automatically if a CloudWatch alarm on error rate fires, run a Lambda validation step before production traffic moves, and keep the old revision running for 30 minutes after full cutover for instant rollback. The team wants as few services to manage as possible. Which solution meets these requirements?

- A. Use the ECS rolling update deployment controller with the deployment circuit breaker and alarm-based rollback enabled, and set `minimumHealthyPercent` to 100 and `maximumPercent` to 110.
- B. Use the CodeDeploy deployment controller with `CodeDeployDefault.ECSCanary10Percent15Minutes`, a `BeforeAllowTraffic` hook, and a 30-minute wait before terminating the original task set.
- C. Use ECS native canary deployment with a 10 percent canary and 15-minute canary bake time, a deployment alarm with rollback, a Lambda lifecycle hook for validation, and a 30-minute deployment bake time.
- D. Run two ECS services, each with its own target group, and use Route 53 weighted records at 90 and 10 to shift traffic, with a Route 53 health check on the new service that removes it on failure.

<details>
<summary>Answer and explanation</summary>

**Correct answer: C**

**Why C is correct:** ECS built-in canary deployments shift a set percentage of traffic, wait for the canary bake time, roll back on CloudWatch alarms, run Lambda lifecycle hooks for validation, and keep the old revision during the deployment bake time for fast rollback. They work with both an ALB and Service Connect, and need no additional service, which meets the "fewest services" requirement for a new service.

**Why the other options are incorrect:**

- **A:** A rolling update replaces tasks in batches; it does not hold 10 percent of traffic for 15 minutes or keep the old revision after cutover. The circuit breaker only detects failed rollouts.
- **B:** CodeDeploy adds another service to manage, and ECS Service Connect is not supported with the CodeDeploy (blue/green) deployment controller, so internal callers would not get the shifted traffic.
- **D:** DNS weights do not apply to Service Connect callers, client DNS caching makes the split and rollback imprecise, and running two services with health check logic is more to manage.

**Key takeaway:** For new ECS services, native ECS blue/green, canary and linear deployments give traffic shifting, bake time, alarm rollback and hooks without CodeDeploy.

</details>

## Task 5.3: Design observability and monitoring solutions

This task tests whether you can instrument workloads once with open standards, collect telemetry across many accounts without building plumbing, alarm on what users feel rather than on every metric, and express reliability as SLOs with burn rate alerts. The recurring decision is between sharing telemetry in place (cross-account observability), copying it (log centralization or subscriptions), or archiving it cheaply (S3 storage classes).

### Skill 5.3.1: Metrics, logs and distributed tracing with standardized instrumentation

**Key concepts**

- Three signals, three default AWS paths:
  - Metrics: CloudWatch metrics, custom metrics through `PutMetricData`, or Embedded Metric Format (EMF) log lines that CloudWatch turns into metrics without API calls (best for Lambda and high-cardinality dimensions you also want to query in logs).
  - Logs: CloudWatch Logs with structured JSON so Logs Insights can query fields. Lambda supports JSON log format and log level filtering natively.
  - Traces: AWS X-Ray. Instrument with OpenTelemetry through AWS Distro for OpenTelemetry (ADOT) or the CloudWatch agent. AWS steers new work toward OpenTelemetry-based instrumentation rather than the older X-Ray SDKs and daemon.
- ADOT gives one collector and one SDK set for metrics and traces, sending to CloudWatch, X-Ray, Amazon Managed Service for Prometheus or third-party backends. This is the answer for "standardized, vendor-neutral instrumentation" or "may change backend later".
- CloudWatch Application Signals auto-instruments supported runtimes (for example Java, Python, .NET and Node.js) on Amazon EKS, Amazon ECS, EC2 and Lambda through the CloudWatch agent and ADOT, and builds standard request, latency, fault and error metrics per service and operation, a service map, and links to traces and logs. It is the fastest path from zero to service-level health dashboards and SLOs.
- Amazon Managed Service for Prometheus and Amazon Managed Grafana: choose them when teams already use PromQL and Grafana, especially on EKS. CloudWatch is the default when you want one AWS-native store with alarms and no extra service.
- Metric resolution: standard (1 minute) or high resolution (down to 1 second) for custom metrics; high resolution costs more and alarms on 10-second or 30-second periods cost more than standard alarms.
- Cardinality drives cost. Every unique combination of dimensions is a separate custom metric. Put high-cardinality identifiers (user ID, request ID) in logs, not metric dimensions.
- Correlation: propagate a trace ID in logs (Lambda and ADOT do this when configured) so you can jump from a metric spike to traces to log lines.

**Exam traps**

- Installing a different vendor agent per signal when the requirement is "single standardized instrumentation". ADOT or the CloudWatch agent with OpenTelemetry covers metrics and traces together.
- Using user ID as a custom metric dimension. Cost grows with every user.
- Choosing CloudTrail for application request tracing. CloudTrail records AWS API calls, not request paths through your code.
- Turning on 100 percent trace sampling for a high-volume API. X-Ray charges per trace recorded and scanned; use sampling rules.

**Hands-on**

1. **One app, three signals.** Deploy a Python Lambda function behind an API Gateway HTTP API with Lambda Powertools (or plain code) emitting JSON logs, an EMF metric for orders processed with a `tenant_tier` dimension, and X-Ray active tracing. Send traffic, then find a slow request in the X-Ray trace map, jump to its log lines with the trace ID in Logs Insights, and graph the EMF metric. *Cost/cleanup:* Free tier covers this volume. Delete the stack and its log groups.
2. **ADOT on ECS.** Run a small Fargate task with the ADOT collector as a sidecar and a sample app instrumented with the OpenTelemetry SDK, exporting traces to X-Ray and metrics to CloudWatch. Change the collector config to add a second exporter (for example a debug exporter) without touching the app code, to see why the collector decouples instrumentation from backend. *Cost/cleanup:* A single small task for an hour costs cents. Stop the service and delete the cluster and log groups.

### Skill 5.3.2: Monitoring for containerized applications (ECS, EKS, Container Insights)

**Key concepts**

- Container Insights standard mode gives cluster, service and task-level aggregated metrics and performance log events.
- Container Insights with enhanced observability:
  - Amazon EKS (November 2023): installed with the Amazon CloudWatch Observability EKS add-on, which deploys the CloudWatch agent and Fluent Bit. It adds detailed pod, container, node and control plane (API server, etcd) health metrics, with drill-down from cluster to container, and ships container logs. It uses observation-based pricing rather than per-metric pricing. The same add-on can turn on Application Signals for workloads on the cluster.
  - Amazon ECS (December 2024): task and container level metrics with drill-down and curated dashboards. Turn it on per cluster or as an ECS account default, so new clusters get it automatically.
  - Accelerated compute observability on EKS covers GPU and AWS Trainium and Inferentia metrics for ML workloads.
- Logs from containers: ECS uses the `awslogs` driver or FireLens (Fluent Bit sidecar) when you need routing, filtering or multiple destinations. EKS uses Fluent Bit as a DaemonSet (installed by the add-on). Sending debug-level logs to an S3 bucket through Firehose and only errors to CloudWatch Logs is a common FireLens cost pattern.
- Prometheus metrics from EKS: Amazon Managed Service for Prometheus with managed collectors (agentless scrapers) or ADOT, visualized in Amazon Managed Grafana.
- Lifecycle events: ECS task state changes and service deployment events go to EventBridge, useful for alerting on stopped tasks with their stop reason.

**Exam traps**

- Enabling enhanced observability by running a custom metrics exporter in every task. For ECS it is a cluster or account setting; for EKS it is the add-on.
- Assuming standard Container Insights shows per-container CPU for each task. Container-level drill-down is the enhanced mode.
- Using CloudTrail to find why ECS tasks stopped. Use the task stopped reason in the ECS console or EventBridge task state change events.
- Forgetting node IAM permissions (or EKS Pod Identity for the agent) when installing the add-on; metrics silently do not arrive.

**Hands-on**

1. **ECS enhanced observability.** Set the ECS account default for Container Insights to enhanced, create a Fargate cluster, and run a service with two containers in each task where one leaks memory slowly. In the Container Insights console drill from cluster to service to task to container and find the leaking container, then alarm on its memory utilization. *Cost/cleanup:* Enhanced observability bills per observation and Fargate per vCPU and GB hour; an hour of testing is low cost. Delete the service, cluster and alarm and set the account default back.
2. **EKS add-on (only if you accept the cost).** Create an EKS cluster with one small managed node, install the `amazon-cloudwatch-observability` add-on, deploy a sample app and view pod and control plane metrics. If you do not want to pay, read the "Container Insights with enhanced observability for Amazon EKS" docs and use the One Observability Workshop in AWS Workshop Studio. *Cost/cleanup:* The EKS control plane is billed per hour (roughly a few dollars per day) plus nodes. Delete the add-on, node group and cluster the same day.

### Skill 5.3.3: Multi-account log aggregation, cross-account subscriptions and centralized archives

**Key concepts**

- Three different mechanisms, and the exam expects you to pick the right one:
  - **CloudWatch cross-account observability (OAM).** A monitoring account creates a sink, source accounts create links. The monitoring account queries and alarms on metrics, logs and traces that stay in the source accounts. No data is copied (except traces), no extra charge for sharing logs and metrics. Same Region only. Up to 100,000 source accounts per monitoring account; a source account can link to up to five monitoring accounts. Best for central operations visibility.
  - **CloudWatch Logs centralization (September 2025).** Organization-level rules created from the management account or a delegated administrator copy new log events from selected accounts, OUs and Regions into one destination account and Region, with an optional backup Region. Requires AWS Organizations trusted access. Only logs that arrive after the rule is created are copied. The first copy is free; the backup Region copy is charged. Customer managed KMS keys at the destination need specific handling and a key tag. Filters by log group name and data source. Best for a single queryable security or audit log store across Regions with no plumbing.
  - **Subscription filters to a cross-account destination.** A CloudWatch Logs destination in the central account fronts a Kinesis Data Streams stream or Amazon Data Firehose stream; its destination policy allows source accounts (use `aws:PrincipalOrgID` for the whole organization). Source accounts add subscription filters (a log group allows up to two) or one account-level subscription filter policy that covers all log groups. Best when logs must flow in near real time to another system: a SIEM, OpenSearch Service, or S3 through Firehose.
- Centralized archive: Firehose to S3 (buffering, compression, dynamic partitioning by account or source), then S3 Lifecycle to S3 Glacier Flexible Retrieval or S3 Glacier Deep Archive for multi-year retention; add S3 Object Lock in compliance mode for immutability. Query archives with Athena.
- `CreateExportTask` is a batch export of a log group to S3; only one export task per account can run at a time, so it is not a streaming or organization-scale solution.
- CloudTrail organization trails, VPC Flow Logs and many AWS service logs can be sent straight to an S3 bucket in a log archive account; that is cheaper than routing them through CloudWatch Logs when you only need retention and Athena queries.
- Log classes: Standard and Infrequent Access. Infrequent Access has a lower ingestion price but supports fewer features (for example no metric filters or subscription filters), so use it for logs you keep for occasional investigation only.
- Retention: set retention on every log group (the default is never expire). Use an AWS Config rule or account-level policy to catch log groups without retention.

**Exam traps**

- Using OAM to satisfy "copy all logs to a central account in another Region for retention". OAM does not copy logs and does not cross Regions.
- Using subscription filters plus Firehose for every account and Region when the requirement is simply "centralize all CloudWatch logs in one account with least overhead". Centralization rules do it natively.
- Expecting centralization rules to backfill history. They copy new events only; use export tasks for history.
- Using the CloudWatch cross-account cross-Region console (`CrossAccountSharingRole`) for logs. It covers metrics, alarms and traces, not logs.
- Archiving logs to S3 Glacier Deep Archive when investigators need minutes-level access. Retrieval takes hours; use S3 Glacier Instant Retrieval or Standard-IA.

**Hands-on**

1. **OAM in two accounts (or console walkthrough).** In a monitoring account, create a sink and allow a second free account to link. In the source account, create a link sharing metrics, logs and traces. From the monitoring account, run a Logs Insights query across both accounts and create an alarm on a source account metric. With one account, open CloudWatch Settings to see the monitoring account wizard and read the "Monitor across accounts and Regions" page. *Cost/cleanup:* No extra charge for sharing logs and metrics. Delete the link, then the sink.
2. **Subscription to S3 archive.** In one account create a Firehose stream to S3 with GZIP compression, a CloudWatch Logs destination that writes to it, and a subscription filter on a Lambda function's log group that matches `ERROR`. Add an S3 Lifecycle rule to move objects to Glacier Flexible Retrieval after 30 days, then query the delivered files with Athena. *Cost/cleanup:* Firehose and S3 costs are cents at this volume. Delete the subscription filter, destination, Firehose stream, bucket and Athena table.

### Skill 5.3.4: Alerting (CloudWatch alarms, Amazon SNS, Systems Manager)

**Key concepts**

- Alarm types: static threshold, anomaly detection band, metric math expression, Metrics Insights query alarm (one alarm over many resources, for example all instances matching a query), and composite alarms that combine others with AND, OR and NOT.
- Tune noise with `M out of N` datapoints, the missing data treatment (`notBreaching`, `breaching`, `ignore`, `missing`) and composite alarm action suppression (for example, suppress child alarms during a known dependency outage).
- Alarm actions: SNS topics (fan out to email, SMS, HTTPS, Lambda, Amazon Q Developer in chat applications for Slack or Microsoft Teams), EC2 actions (stop, terminate, reboot, recover), Auto Scaling policies, direct Lambda invocation, and Systems Manager OpsCenter OpsItems. Alarm state changes are also EventBridge events, which is how you route to Step Functions or SSM Automation.
- Systems Manager Incident Manager closed to new customers on 7 November 2025, so do not design new incident response around it. Use alarms and EventBridge to OpsCenter, SSM Automation runbooks, chat notifications and a third-party paging tool if required.
- Multi-account alerting: alarms in the OAM monitoring account can watch metrics from source accounts. Otherwise, forward alarm state change events through EventBridge cross-account buses to a central account.
- Log-based alerting: metric filters turn log patterns into metrics, then alarm on them. For security events, EventBridge rules on CloudTrail management events (for example a root login or `StopLogging`) are faster and simpler.
- Alarm on symptoms (error rate, latency percentiles, SLO burn rate) for paging, and on causes (CPU, queue depth) for tickets or automation.

**Exam traps**

- Creating one alarm per instance in a fleet of thousands. A Metrics Insights query alarm or an Auto Scaling group aggregate metric is less to manage.
- Paging on CPU utilization. It is a cause signal and leads to alert fatigue.
- Treating missing data as breaching for metrics that are only published on errors. The alarm sits in ALARM forever.
- Recommending Incident Manager response plans for a new design.

**Hands-on**

1. **Composite alarm with suppression.** Create two alarms on an API: 5XX rate above 1 percent and p99 latency above a threshold, plus a third alarm on a dependency health metric. Create a composite alarm `ALARM(errors) AND ALARM(latency)` that notifies an SNS topic, with action suppression while the dependency alarm is in ALARM. Trigger each condition with a test function and see which notifications are sent. *Cost/cleanup:* Standard alarms cost cents per month each; delete them and the topic.
2. **Alarm to Automation through EventBridge.** Create an alarm on a test EC2 instance's `StatusCheckFailed_System` (use `set-alarm-state` to simulate), and an EventBridge rule on the alarm state change that starts the `AWS-RestartEC2Instance` Automation runbook and creates an OpsItem. Confirm the runbook ran in the Automation console. *Cost/cleanup:* Free tier instance; terminate it and delete the rule and alarm.

### Skill 5.3.5: Metrics for AI/ML workloads (model performance, agents, latency, drift)

**Key concepts**

- Amazon Bedrock publishes CloudWatch runtime metrics per model, including invocation count, invocation latency, client and server errors, throttles, and input and output token counts. Use token counts for cost and quota tracking, throttles to size provisioned throughput or use cross-Region inference, and latency percentiles for user experience.
- Bedrock model invocation logging sends full request and response data (and metadata) to CloudWatch Logs, S3 or both. It is off by default. Turn it on for audit and for quality analysis; protect the destination because it contains prompts and outputs.
- Bedrock Guardrails publishes metrics on guardrail interventions; a rising intervention rate is a signal of misuse or a prompt regression.
- Agents: Amazon Bedrock AgentCore Observability provides traces, sessions and step-level metrics (latency, tool calls, errors, token usage) using OpenTelemetry, viewable in CloudWatch. AgentCore Evaluations online mode samples live traces and scores quality (for example goal success rate, tool selection accuracy), which turns "agent quality" into a metric you can alarm on.
- SageMaker AI endpoints: `Invocations`, `ModelLatency` (time in the model container), `OverheadLatency` (SageMaker overhead), 4XX and 5XX errors, and instance CPU, GPU and memory. Split latency into model vs overhead before choosing a fix.
- Drift: SageMaker Model Monitor schedules data quality, model quality (needs ground truth labels), bias drift and feature attribution drift jobs against a baseline built from training data, using data capture on the endpoint. Violations emit CloudWatch metrics you can alarm on and use to trigger a retraining pipeline in SageMaker Pipelines.
- For generative AI, "drift" usually shows up as falling evaluation scores on a fixed golden dataset, rising guardrail interventions or rising user thumbs-down rates, so schedule periodic Bedrock evaluation jobs and publish scores as custom metrics.

**Exam traps**

- Using SageMaker Model Monitor model quality without a ground truth labeling process. It needs labels to compare predictions against.
- Assuming Bedrock logs prompts by default. Invocation logging must be enabled.
- Scaling a SageMaker endpoint when `ModelLatency` is low and `OverheadLatency` is high, or the reverse without checking which part is slow.
- Treating CloudTrail as the record of prompts and responses. CloudTrail records the API call, not the content.

**Hands-on**

1. **Bedrock telemetry dashboard.** Enable Bedrock model invocation logging to a CloudWatch log group, call a low-cost model 50 times from a script with varying prompt lengths, and build a dashboard with invocation latency, token counts and throttles. Use Logs Insights to find the longest prompts from the invocation logs. *Cost/cleanup:* Model tokens for 50 short calls cost cents. Disable invocation logging and delete the log group.
2. **Model Monitor data quality.** Train or use a built-in SageMaker sample model, deploy a serverless or small real-time endpoint with data capture, create a baseline from the training CSV and schedule a data quality monitor. Send skewed data and inspect the violations report and CloudWatch metrics. *Cost/cleanup:* Real-time endpoints bill per hour even when idle and monitoring jobs bill per instance minute; delete the endpoint, monitoring schedule and S3 data the same day.

### Skill 5.3.6: User experience monitoring (CloudWatch Synthetics, CloudWatch RUM)

**Key concepts**

- **CloudWatch Synthetics** runs canaries (scripts on Lambda) on a schedule against endpoints and user journeys, even when there is no real traffic. Runtimes include Node.js with Puppeteer or Playwright and Python with Selenium. Blueprints cover heartbeat checks, API checks, broken link checking, visual monitoring (screenshot comparison against a baseline) and multi-step GUI workflows. Canaries can run inside a VPC to test private endpoints. They publish `SuccessPercent` and `Duration` metrics and store screenshots, HAR files and logs in S3. Enable X-Ray tracing to follow a canary request through the backend.
- **CloudWatch RUM** collects data from real user sessions: page load performance, JavaScript and HTTP errors, and session behavior for web apps, plus screen loads, crashes and app hangs for Android and iOS apps (through the ADOT mobile SDKs). You set a session sampling percentage to control cost. Data can be broken down by browser, device and location. RUM data is kept for 30 days, or sent to CloudWatch Logs for longer retention. The web client snippet typically gets credentials from an Amazon Cognito identity pool created with the app monitor. With X-Ray active tracing turned on, RUM client requests appear in Application Signals connected to backend services.
- Decision rule: Synthetics proves availability and catches regressions before users see them (including low-traffic hours and other Regions). RUM shows what real users actually experience across devices and networks. Mature designs use both.
- Synthetics canaries and RUM app monitors can back SLOs (Synthetics as period-based on success or duration, RUM as request-based).
- Route 53 health checks verify endpoint reachability from many locations and drive DNS failover; they are not a user journey test.

**Exam traps**

- Using RUM to detect an outage at 3 AM when there are almost no users. Without traffic there is no data; use canaries.
- Using Synthetics to measure performance for real users in a specific country on mobile networks. Canaries run from AWS Regions; RUM measures real users.
- Setting RUM sampling to 100 percent on a high-traffic site without considering per-event cost.
- Using a Route 53 health check to validate a multi-step checkout. It checks one endpoint and an optional string match only.

**Hands-on**

1. **Canary with visual monitoring.** Host a static site on S3 behind CloudFront. Create a Synthetics canary from the visual monitoring blueprint that runs every 5 minutes and compares screenshots with a baseline, plus an alarm on `SuccessPercent`. Change the page layout and watch the canary fail on the visual difference. *Cost/cleanup:* Canaries are billed per run plus Lambda and S3; a day at 5-minute intervals is low cost. Stop and delete the canary, its S3 artifacts and Lambda function.
2. **RUM on the same site.** Create a RUM app monitor for the CloudFront domain, add the generated snippet to the page, set sampling to 100 percent for the test, and load the page from different browsers and a phone. Throw a deliberate JavaScript error and find it, with its session, in the RUM console. *Cost/cleanup:* RUM is billed per event; a few hundred events cost cents. Delete the app monitor, the Cognito identity pool it created, and the site.

### Skill 5.3.7: SLOs with measurable indicators and alerts (CloudWatch, Application Signals)

**Key concepts**

- Application Signals SLOs can be built on Application Signals service or operation metrics (latency, availability), on any CloudWatch metric or metric math expression that returns one time series, on Synthetics canaries and on RUM app monitors.
- Availability from Application Signals is based on faults (5XX); 4XX responses count as successful, which matters when you define the SLI.
- Two evaluation types:
  - Period-based: each period (for example 1 minute) is good or bad against the SLI threshold; attainment is good periods over total periods. A bad minute costs a full minute of budget.
  - Request-based: attainment is good requests over total requests. Better for spiky traffic, because a bad minute with 5 requests counts less than one with 50,000.
- Intervals: rolling (for example the last 30 days, best for operations) or calendar aligned (day, week, month, best for business reporting and contracts).
- Error budget is the unhealthy time or bad requests you can tolerate while still meeting the goal. Burn rate is the error rate in a look-back window divided by (100 percent minus the goal). A burn rate of 1 uses the budget exactly over the interval; higher means trouble.
- Alarms: SLI health alarms, SLO warning alarms (remaining budget below a warning threshold), and burn rate alarms with look-back windows. Use multi-window alerting with a composite alarm (for example a long window AND a short window both burning fast) so you page on sustained burn and stop paging quickly after recovery.
- Composite SLOs aggregate availability across several operations of one service.
- Exclusion windows (for planned maintenance) can be defined on SLOs.
- Operational details: editing an SLO's core settings resets attainment history; alarms are not updated or deleted with the SLO, so manage alarms as code alongside it.

**Exam traps**

- Using a static CPU alarm as the "SLO alert". An SLO must be based on a user-facing indicator.
- Using period-based SLOs for very low or spiky traffic APIs where one failed request in a quiet minute burns a whole minute. Request-based fits better.
- Paging on a single short-window burn rate alarm. It is noisy; use multi-window, multi-burn-rate composite alarms.
- Assuming deleting an SLO cleans up its alarms.

**Hands-on**

1. **Request-based SLO with burn rate alarms.** Turn on Application Signals for a Lambda function behind API Gateway, or define an SLI on the API Gateway 5XX and Count metrics with metric math. Create a request-based availability SLO of 99.5 percent over a rolling 7 days, add burn rate alarms on 5-minute and 1-hour windows and a composite alarm that needs both. Inject a 10 percent error rate for 15 minutes and watch the budget and burn rate. *Cost/cleanup:* Application Signals and alarms bill by usage; a test day is low cost. Delete the SLO, then its alarms separately, and disable Application Signals.
2. **Synthetics-backed period-based SLO.** Reuse the canary from skill 5.3.6 and create a period-based SLO on its `SuccessPercent` with 5-minute periods and a calendar-month interval. Add a maintenance exclusion window and run a deliberate failure inside and outside it to see the difference. *Cost/cleanup:* Same as the canary; delete the SLO and alarms with it.

### Practice questions: Task 5.3

**Question 5.3.1** A company has 300 AWS accounts in AWS Organizations, with workloads in us-east-1 and eu-west-1. The security team needs a copy of all CloudWatch Logs log groups from every account and both Regions in a single security account in us-east-1, with a second copy in us-west-2 for disaster recovery. New accounts must be included automatically. The team wants no Lambda functions, Kinesis streams or per-account configuration to maintain, and the solution should minimize data transfer and processing costs. Which solution meets these requirements?

- A. Configure CloudWatch cross-account observability with the security account as the monitoring account in each Region, and link all accounts with an organization-wide StackSet.
- B. Create a CloudWatch Logs destination backed by Amazon Data Firehose in the security account, and deploy an account-level subscription filter policy to every account and Region with a service-managed StackSet.
- C. Create a CloudWatch Logs centralization rule from a delegated administrator account that targets the organization and both source Regions, with the security account in us-east-1 as the destination and us-west-2 as the backup Region.
- D. Turn on a scheduled export of every log group to an S3 bucket in the security account with `CreateExportTask` from an EventBridge Scheduler job, and replicate the bucket to us-west-2 with S3 Cross-Region Replication.

<details>
<summary>Answer and explanation</summary>

**Correct answer: C**

**Why C is correct:** Centralization rules are configured once at organization level and copy new log events from selected accounts, OUs and Regions into one destination account and Region, with an optional backup Region for a second copy. New accounts in the targeted scope are included without extra setup. There are no streams or functions to run, and the first centralized copy is not charged, which keeps cost down.

**Why the other options are incorrect:**

- **A:** Cross-account observability shares logs in place. It does not copy them into the security account, and it works within one Region, so it cannot create a us-west-2 copy.
- **B:** This works but runs Firehose streams with processing charges, and the StackSet and destination policy add per-account and per-Region plumbing the team wanted to avoid.
- **D:** Only one export task per account runs at a time, exports are batch, and the scheduler would need cross-account access to every account. It is slow and needs custom orchestration.

**Key takeaway:** Copying logs organization-wide across Regions into one account with a backup Region is what CloudWatch Logs centralization rules were built for.

</details>

**Question 5.3.2** A trading platform runs on Amazon EKS and Amazon ECS on Fargate in one account. The SRE team wants per-service latency and fault rates, a service map, and traces linked to logs, without asking developers to change application code for each service. They also need availability SLOs where 4XX client errors do not count against the SLO, and alerts that page only when the error budget is being consumed fast over both a short and a long window. Which solution meets these requirements with the least operational overhead?

- A. Enable CloudWatch Application Signals with auto-instrumentation through the CloudWatch agent on EKS and ECS, create availability SLOs on the discovered services, and add burn rate alarms on two look-back windows combined in a composite alarm.
- B. Add the X-Ray SDK to each service, publish custom latency and error metrics with `PutMetricData`, and create static threshold alarms on p99 latency and 5XX counts for each service.
- C. Deploy Amazon Managed Service for Prometheus with managed collectors and Amazon Managed Grafana, have each team expose RED metrics from their code, and define SLO alert rules in Grafana.
- D. Enable Container Insights with enhanced observability for both platforms, and create anomaly detection alarms on CPU and memory utilization per service as the SLO indicators.

<details>
<summary>Answer and explanation</summary>

**Correct answer: A**

**Why A is correct:** Application Signals auto-instruments supported runtimes on EKS and ECS through the CloudWatch agent, so services produce standard latency, fault and error metrics, a service map and trace-to-log links without per-service code changes. Its availability metric counts faults (5XX) and treats 4XX as success. SLOs on the discovered services support burn rate alarms, and a composite alarm over two look-back windows gives multi-window paging.

**Why the other options are incorrect:**

- **B:** Developers must add the SDK and custom metrics in every service, and static threshold alarms are not error budget or burn rate alerts.
- **C:** Teams must change code to expose metrics, and running Prometheus, collectors and Grafana with hand-written SLO rules is more to operate.
- **D:** Container Insights gives infrastructure metrics. CPU and memory are cause signals, not user-facing SLIs, and anomaly alarms do not track an error budget.

**Key takeaway:** For no-code service-level metrics and SLOs with burn rate alerts on AWS compute, use Application Signals.

</details>

**Question 5.3.3** An insurance company has a customer portal used mostly between 8 AM and 8 PM. Overnight, a broken deployment went unnoticed until customers complained the next morning. The company also wants to know how page load times and JavaScript errors differ between mobile browsers in three countries. The team wants to detect broken login and quote flows within 10 minutes at any hour, and measure real user performance by device and location. Which combination of steps meets these requirements? (Select TWO.)

- A. Create a CloudWatch Synthetics canary from the GUI workflow blueprint that runs the login and quote flow every 5 minutes, with an alarm on `SuccessPercent` that notifies the on-call SNS topic.
- B. Create Route 53 health checks against the portal's login URL with a string match, and alarm on the health check status metric.
- C. Create a CloudWatch RUM app monitor for the portal, add the web client snippet with a sampling rate that fits the budget, and analyze performance and errors by browser, device and country.
- D. Turn on CloudFront standard logs and analyze them with Athena to compute page load times by country and device.
- E. Run the canary scripts from Lambda functions in three Regions near the target countries to measure mobile user performance.

<details>
<summary>Answer and explanation</summary>

**Correct answers: A and C**

**Why A and C are correct:** A Synthetics canary runs the multi-step login and quote journey on a schedule, so it detects a broken flow overnight when there is no real traffic, and the `SuccessPercent` alarm notifies within the 10-minute target (A). RUM collects page load performance and JavaScript errors from real sessions and breaks them down by browser, device and location, which answers the mobile and country question; sampling controls cost (C).

**Why the other options are incorrect:**

- **B:** A health check confirms one URL responds with a string. It cannot log in and request a quote, so a broken flow behind a working login page passes.
- **D:** CloudFront logs show request timing at the edge, not browser page load time or JavaScript errors.
- **E:** Canaries from AWS Regions measure synthetic performance from data centers, not real mobile devices and networks in those countries.

**Key takeaway:** Synthetics finds breakage when no users are present; RUM measures what real users experience.

</details>

**Question 5.3.4** A retail bank serves a credit-decision model on an Amazon SageMaker AI real-time endpoint. Over three months, approval rates have shifted and the risk team suspects the incoming applicant data no longer looks like the training data. Ground truth (whether a loan defaulted) arrives only after 90 days. The team wants an automated, recurring check that compares live input features to the training baseline, raises a CloudWatch alarm on violations, and starts a retraining pipeline. Which solution meets these requirements?

- A. Turn on Amazon Bedrock model invocation logging for the endpoint and use CloudWatch Logs Insights queries on a schedule to compare feature distributions.
- B. Create a SageMaker Model Monitor model quality schedule that compares predictions with ground truth labels every hour, and alarm on the accuracy metric.
- C. Alarm on the endpoint's `ModelLatency` and `Invocation5XXErrors` metrics with anomaly detection, and trigger retraining when either alarm fires.
- D. Enable data capture on the endpoint, create a SageMaker Model Monitor data quality baseline from the training dataset and a monitoring schedule, alarm on its violation metrics, and start a SageMaker Pipelines retraining run from an EventBridge rule.

<details>
<summary>Answer and explanation</summary>

**Correct answer: D**

**Why D is correct:** Data quality monitoring compares captured live inputs against statistics and constraints baselined from the training data, which detects input drift without waiting for labels. Model Monitor emits CloudWatch metrics for violations, so an alarm and an EventBridge rule on the alarm state can start a SageMaker Pipelines retraining run.

**Why the other options are incorrect:**

- **A:** Bedrock invocation logging applies to Bedrock model calls, not to SageMaker AI endpoints, and hand-written Logs Insights statistics are not a drift baseline.
- **B:** Model quality monitoring needs ground truth, which arrives after 90 days, so hourly checks would have nothing to compare and drift would be found far too late.
- **C:** Latency and error metrics show operational health, not changes in the data distribution. A drifting model can be fast and error-free.

**Key takeaway:** When labels are delayed, detect drift with Model Monitor data quality (or feature attribution) monitoring on captured inputs.

</details>

**Question 5.3.5** A company has 120 accounts in AWS Organizations in a single Region. Its central operations team wants to build dashboards and CloudWatch alarms in one monitoring account that use metrics from all accounts, run Logs Insights queries across accounts during incidents, and view X-Ray traces end to end. Application teams must keep owning their logs in their own accounts for chargeback, and the company does not want to pay to store a second copy of the logs. Which solution meets these requirements?

- A. Create a CloudWatch Logs centralization rule that copies all log groups to the monitoring account, and enable the cross-account cross-Region console with `CrossAccountSharingRole` for metrics.
- B. Set up CloudWatch cross-account observability with a sink in the monitoring account and links from all source accounts for metrics, logs and traces, deployed through AWS Organizations.
- C. Create subscription filters in every account that stream logs to a Kinesis Data Streams stream in the monitoring account, and use metric streams to send metrics to the same account.
- D. Use the cross-account cross-Region CloudWatch console with `CrossAccountSharingRole` in each source account, and switch between accounts during incidents.

<details>
<summary>Answer and explanation</summary>

**Correct answer: B**

**Why B is correct:** Cross-account observability links source accounts to a monitoring account so it can build dashboards and alarms on their metrics, query their logs with Logs Insights and view traces, while the data stays in the source accounts. Logs and metrics are shared at no extra charge, and everything is in one Region, which matches this estate.

**Why the other options are incorrect:**

- **A:** Centralization copies logs into the monitoring account, so the company stores a second copy and the source teams no longer have the only copy for chargeback.
- **C:** Streams and metric streams duplicate data and add Kinesis and processing costs plus per-account plumbing.
- **D:** The cross-account console switches to one account at a time and does not cover logs, so cross-account Logs Insights queries are not possible.

**Key takeaway:** Share in place with OAM when the goal is central visibility within a Region; copy with centralization or subscriptions only when you need a separate stored copy.

</details>

## Task 5.4: Design operational automation and resilient systems

This task tests whether you can turn signals into safe, automatic action: detect health and capacity problems early, scale ahead of demand, replace failed components without humans, and run remediation that cannot make an incident worse. The recurring decision is which native automation (Auto Scaling, ECS, Lambda retry settings, EventBridge, Systems Manager Automation with rate controls) does the job, and which safety limit stops it from running away.

### Skill 5.4.1: Health monitoring, capacity planning and operational responses

**Key concepts**

- AWS Health: account-specific events (scheduled EC2 retirement, RDS maintenance, service issues affecting your resources) arrive as EventBridge events from `aws.health`. With an organization view enabled in the management account or a delegated administrator, you see events across all accounts. Route them with EventBridge to SNS, chat, or SSM Automation (for example, stop and start an instance before its scheduled retirement).
- Service Quotas:
  - Many services publish usage metrics in CloudWatch that you can graph against the quota with the `SERVICE_QUOTA()` metric math function, for example `m1/SERVICE_QUOTA(m1)*100`, and alarm at 80 percent.
  - Automatic quota management (late 2025) sends utilization notifications (email, SMS, Slack) as usage approaches limits and can adjust quotas automatically. It is free.
  - Quota request templates apply increase requests to new accounts in an organization when they are created.
  - Trusted Advisor service limit checks are an older path; prefer Service Quotas for new designs.
- Capacity planning:
  - Compute Optimizer recommendations for EC2, Auto Scaling groups, EBS, Lambda, ECS on Fargate and RDS.
  - On-Demand Capacity Reservations (and future-dated reservations) guarantee EC2 capacity in an Availability Zone for events or DR; they do not give a discount by themselves. Pair them with Savings Plans for price.
  - Predictive scaling forecasts (see 5.4.3) double as a capacity planning view.
- Event-driven responses: EventBridge rules on alarm state changes, AWS Health events, Config compliance changes, GuardDuty findings or ECS task state changes, targeting Step Functions, Lambda or SSM Automation.
- Systems Manager pieces that matter here: Automation (runbooks), Run Command (ad hoc commands at scale with rate controls), State Manager (keep configuration in a desired state on a schedule), Patch Manager (patch policies), OpsCenter (OpsItems for operational issues), Fleet Manager and Session Manager (access without SSH keys or bastions), and Explorer for an operations summary.
- Quick Setup deploys recommended Systems Manager configurations to one account or across an organization: host management (agent updates, inventory, instance profile), Default Host Management Configuration, patch policies, Config recorder and conformance packs, DevOps Guru, Distributor packages, Resource Scheduler (stop and start on a schedule), OpsCenter setup and Resource Explorer.
- Default Host Management Configuration lets Systems Manager manage EC2 instances without attaching an instance profile to each one.

**Exam traps**

- Requesting quota increases reactively after a failure. Alarm on usage against `SERVICE_QUOTA()`, use automatic quota management, and pre-request with quota request templates.
- Buying Reserved Instances to "guarantee capacity" in an AZ for a launch event. Zonal RIs reserve capacity, but the flexible answer for a dated event is an On-Demand Capacity Reservation or a future-dated reservation; Savings Plans do not reserve capacity.
- Polling the AWS Health API from a cron Lambda function. Use EventBridge Health events and the organization view.
- Choosing Incident Manager or Change Manager for a new design. Both closed to new customers on 7 November 2025.

**Hands-on**

1. **Quota alarm.** In the CloudWatch console browse Usage metrics, pick a quota-backed metric such as running On-Demand standard instance vCPUs, add `m1/SERVICE_QUOTA(m1)*100` and create an alarm at 80 percent that notifies an SNS topic. Then open Service Quotas and turn on automatic quota management notifications for your account to compare the two approaches. *Cost/cleanup:* One alarm costs cents per month. Delete the alarm and topic.
2. **Health event automation.** Create an EventBridge rule matching `aws.health` events with an SNS target, then use a sample event in the EventBridge sandbox to test the pattern. Add a second target that starts an SSM Automation runbook (for example `AWS-StopEC2Instance` on a test instance) and test it with the sample event mapped through an input transformer. *Cost/cleanup:* Free; terminate the test instance and delete the rule.

### Skill 5.4.2: Controlled remediation with safety constraints

**Key concepts**

- Automation can make an incident worse: a remediation that restarts every instance at once is an outage. Build limits in.
- **Rate limits and concurrency.** SSM Automation, Run Command and State Manager take `MaxConcurrency` (count or percentage) and `MaxErrors` (stop sending to more targets after this many failures). An Automation run can also watch a CloudWatch alarm and cancel itself when the alarm goes into ALARM, running `onCancel` steps. Config auto-remediation has maximum automatic attempts, a retry interval, and execution controls (`SsmControls`: `ConcurrentExecutionRatePercentage` and `ErrorPercentage`) that limit how many remediations run at once. StackSets have concurrency and failure tolerance. Patch Manager maintenance windows and patch policies use the same concurrency and error thresholds.
- **Circuit breakers.**
  - ECS deployment circuit breaker rolls back a failing deployment.
  - Application-level circuit breaker: after N failures to a dependency, stop calling it for a cool-down period and fail fast or serve a fallback. On AWS, keep the breaker state in DynamoDB or ElastiCache shared by all instances, or use Step Functions with a state check before calling the dependency.
  - Lambda event source mappings for SQS can have their maximum concurrency capped so a slow downstream is not flooded.
- **Retries with exponential backoff and jitter.** AWS SDKs retry throttling and transient errors with backoff and jitter by default (standard and adaptive retry modes). Step Functions `Retry` blocks set `IntervalSeconds`, `BackoffRate`, `MaxAttempts`, `MaxDelaySeconds` and `JitterStrategy`, and `Catch` blocks route to fallbacks. Only retry idempotent operations, or make them idempotent with idempotency keys.
- **Bulkheads.** Isolate resource pools so one tenant or feature cannot consume everything: separate queues or Lambda functions with reserved concurrency per workload, separate ECS services or target groups, separate connection pools, per-tenant API Gateway usage plans, and cell-based architecture at the largest scale.
- **Load shedding and throttling.** API Gateway stage and method throttling and usage plans, WAF rate-based rules, SQS as a buffer in front of a constrained database, RDS Proxy to pool database connections.
- **Human checkpoints.** SSM Automation `aws:approve` steps and Step Functions task tokens for approval before destructive actions; AWS FIS stop conditions on CloudWatch alarms for experiments.

**Exam traps**

- Running a remediation runbook against all targets at 100 percent concurrency with no error threshold.
- Retrying non-idempotent payment operations without idempotency keys, causing duplicates.
- Retrying immediately in a tight loop. Without backoff and jitter, retries synchronize and amplify the outage (retry storm).
- Assuming Lambda reserved concurrency is only a cost control. It is also a bulkhead that protects the function and caps its pressure on downstream systems.
- Using an SQS dead-letter queue as the circuit breaker. A DLQ isolates poison messages; it does not stop calls to a failing dependency.

**Hands-on**

1. **Rate-controlled Automation with alarm stop.** Tag four `t3.micro` instances `Patch=Test`. Run `AWS-RestartEC2Instance` with targets by tag, `MaxConcurrency` of 1 and `MaxErrors` of 1, and attach a CloudWatch alarm. Watch instances restart one at a time, then set the alarm state to ALARM mid-run with `set-alarm-state` and see the execution cancel. *Cost/cleanup:* Free tier instances for an hour; terminate them and delete the alarm.
2. **Step Functions retry, backoff and circuit breaker.** Build a state machine that calls a Lambda function which fails 70 percent of the time, with `Retry` (`BackoffRate` 2, `MaxAttempts` 3, `JitterStrategy` FULL) and a `Catch` to a fallback state. Add a DynamoDB `GetItem` state before the call that checks a circuit state item and skips the call when it is open, and a step that opens the circuit after the catch. Run it several times and read the execution history to see the delays and short-circuited runs. *Cost/cleanup:* Standard workflow state transitions and Lambda calls are within free tier at this volume; delete the state machine, function and table.

### Skill 5.4.3: Metric-based and scheduled auto scaling policies

**Key concepts**

- Policy types for EC2 Auto Scaling (Application Auto Scaling has the same ideas for ECS, DynamoDB, Aurora replicas, Lambda provisioned concurrency, SageMaker AI endpoints and others):
  - **Target tracking:** keep a metric at a value (CPU 50 percent, ALB requests per target, SQS backlog per instance through metric math). AWS creates and manages the alarms. Default choice.
  - **Step scaling:** different adjustments for different breach sizes. Use when you need explicit control, or to scale on a metric target tracking cannot express well.
  - **Simple scaling:** legacy, waits for cooldown; avoid.
  - **Scheduled scaling:** set min, max or desired at times (cron with time zone). Use for known events (market open, payroll run, batch window).
  - **Predictive scaling:** learns daily and weekly patterns from history and scales out ahead of forecast load.
- Predictive scaling details (EC2 Auto Scaling):
  - Needs at least 24 hours of metric history to start; it uses up to 14 days. Forecasts are hourly for the next 48 hours and refreshed every 6 hours.
  - Start in forecast-only mode, compare forecasts with actual load, then switch to forecast and scale.
  - It only scales out from forecasts; it never scales in because the forecast drops. Pair it with target tracking, which handles scale-in and unexpected spikes. When several policies apply, the group uses the highest desired capacity.
  - `SchedulingBufferTime` launches instances before each hour so they are warm when load arrives.
  - `MaxCapacityBreachBehavior` can let the forecast raise the group's maximum by a buffer; the raised maximum stays until you change it.
  - Works best for homogeneous groups with recurring patterns; mixed instance types with different sizes can skew forecasts.
  - Predictive scaling is also available for Amazon ECS services (November 2024), with forecast-only mode.
- Warm-up and responsiveness: set default instance warmup so new instances do not distort metrics, use warm pools for applications with long boot times, and use lifecycle hooks for setup before `InService`.
- Scaling on queues: scale on backlog per instance (queue depth divided by running capacity, via metric math) rather than raw `ApproximateNumberOfMessagesVisible`.
- Lambda: concurrency scales automatically; reserved concurrency caps it, provisioned concurrency pre-warms it, and provisioned concurrency can itself be scheduled or target-tracked with Application Auto Scaling.
- Kubernetes on EKS: Horizontal Pod Autoscaler for pods, and Karpenter or Cluster Autoscaler (or EKS Auto Mode) for nodes. KEDA for event-driven pod scaling.

**Exam traps**

- Using only scheduled scaling for a pattern that shifts week to week. Predictive scaling adapts; schedules need manual edits.
- Using predictive scaling alone and wondering why capacity never drops. It does not scale in.
- Using predictive scaling for a new application with no history or for unpredictable spikes. Use target tracking (and warm pools if boot is slow).
- Scaling a queue worker group on CPU. Use backlog per instance.
- Setting `MaxCapacityBreachBehavior` to increase max capacity and forgetting the raised maximum persists.

**Hands-on**

1. **Predictive scaling in forecast-only mode.** Create an Auto Scaling group with one or two `t3.micro` instances and a target tracking CPU policy, then add a predictive scaling policy in forecast-only mode. Generate a repeating load pattern with a scheduled Lambda function or a small load script for a day or two, then view the forecast graph and the `GetPredictiveScalingForecast` output against actual load. *Cost/cleanup:* Free tier instances if you stay within limits; delete the group, launch template and load generator afterwards.
2. **Queue backlog target tracking.** Create an SQS queue and an Auto Scaling group (or ECS service) of workers. Define a target tracking policy with a metric math expression of `ApproximateNumberOfMessagesVisible` divided by the group's `GroupInServiceInstances`, target 10 messages per instance. Push 500 messages and watch the group scale out, then drain and scale in. *Cost/cleanup:* Low cost; delete the group, queue and policy.

### Skill 5.4.4: Automated failure recovery with health checks and replacement

**Key concepts**

- **EC2 Auto Scaling health checks.** EC2 status checks are always used. Add ELB health checks so an instance that is running but failing the application check is replaced. Set a health check grace period that covers boot time. Custom health status through `SetInstanceHealth` for application-specific signals. Auto Scaling also rebalances across Availability Zones.
- **EC2 auto recovery** (simplified automatic recovery on supported instance types, or a CloudWatch alarm action on `StatusCheckFailed_System`) moves the instance to healthy hardware with the same instance ID, private IP and EBS volumes. Use it for single-instance workloads that cannot be in an Auto Scaling group. It does not fix an application or OS failure.
- **Amazon ECS.** The service scheduler replaces tasks that stop or fail ELB health checks or container health checks (`healthCheck` in the task definition). The deployment circuit breaker stops failing deployments. Availability Zone rebalancing for ECS services redistributes tasks after an AZ recovers.
- **Amazon EKS.** Liveness and readiness probes restart or remove pods; node auto repair in managed node groups or EKS Auto Mode replaces unhealthy nodes.
- **AWS Lambda.**
  - Synchronous invocations: the caller retries.
  - Asynchronous invocations: Lambda retries up to two times by default; you can set retries (0 to 2) and maximum event age, and send failures to an on-failure destination (SQS, SNS, Lambda, EventBridge, S3) or a DLQ. Destinations carry more context than DLQs.
  - Stream event source mappings (Kinesis, DynamoDB Streams): `BisectBatchOnFunctionError`, `MaximumRetryAttempts`, `MaximumRecordAgeInSeconds`, on-failure destination, and `ReportBatchItemFailures` so one bad record does not block the shard.
  - SQS event source mappings: failed messages become visible again after the visibility timeout; configure a redrive policy with `maxReceiveCount` to a DLQ, and use partial batch responses.
- **Databases.** RDS Multi-AZ fails over automatically; Aurora promotes a replica (set failover priority tiers); DynamoDB is Multi-AZ by design. Use RDS Proxy to reduce failover time seen by applications.
- **Load balancers and DNS.** ALB and NLB stop routing to unhealthy targets; Route 53 health checks remove unhealthy endpoints from DNS answers; Amazon Application Recovery Controller provides routing controls and readiness checks for cross-Region recovery.
- Static stability: provision enough capacity in each AZ so the loss of one AZ does not need scaling actions to survive.

**Exam traps**

- Relying on EC2 status checks alone when the application hangs. The instance stays "healthy" without ELB health checks.
- A grace period shorter than boot time, which causes replacement loops.
- Using EC2 auto recovery for instance store data. Instance store data is lost.
- Configuring a DLQ on the Lambda function for an SQS-triggered function. For SQS sources, the DLQ belongs on the source queue.
- Leaving default asynchronous retries on a non-idempotent Lambda function.

**Hands-on**

1. **ELB health check replacement.** Create an Auto Scaling group of two instances behind an ALB with only EC2 health checks. SSH or use Session Manager to stop the web server on one instance and see the ALB mark it unhealthy while Auto Scaling keeps it. Turn on ELB health checks for the group and watch Auto Scaling replace it. *Cost/cleanup:* ALB hourly charge plus free tier instances; delete the group, ALB and launch template.
2. **Lambda stream failure handling.** Create a Kinesis data stream with one shard (on-demand or provisioned) and a Lambda consumer that fails on records containing `"bad"`. First run with defaults and watch the shard iterator age grow. Then set `BisectBatchOnFunctionError`, `MaximumRetryAttempts` of 2, an SQS on-failure destination and `ReportBatchItemFailures`, and see good records flow while the bad record lands in SQS. *Cost/cleanup:* A provisioned shard bills per shard hour; delete the stream, mapping, function and queue within the hour.

### Skill 5.4.5: Automated runbooks and incident response (patching, remediation, scaling)

**Key concepts**

- **Systems Manager Automation runbooks** are YAML or JSON documents of steps (`aws:executeAwsApi`, `aws:executeScript` with Python or PowerShell, `aws:runCommand`, `aws:branch`, `aws:approve`, `aws:waitForAwsResourceProperty`, `aws:changeInstanceState` and others). Use AWS-owned runbooks (for example `AWS-RestartEC2Instance`, `AWS-CreateSnapshot`, `AWSSupport-*` troubleshooting runbooks) before writing your own. Runbooks can be shared with other accounts.
- **Multi-account and multi-Region Automation.** Run from a central account or delegated administrator, targeting accounts or OUs and Regions, using an administration role in the central account and an execution role in each target account, with concurrency and error thresholds at account and Region level.
- **Triggers:** EventBridge rules (alarm state changes, Config compliance changes, GuardDuty or Security Hub findings), AWS Config remediation actions, State Manager associations on a schedule, maintenance windows, or manual start from OpsCenter OpsItems.
- **Patching.** Patch Manager patch policies created through Quick Setup patch an organization's managed nodes across accounts and Regions with patch baselines, scan and install schedules, and rate controls. Compliance reports go to S3 and Security Hub can consume patch compliance. Amazon Inspector finds the vulnerabilities; Patch Manager remediates OS patches.
- **Security remediation.** GuardDuty or Security Hub finding → EventBridge → Step Functions or Automation: isolate the instance (swap to a quarantine security group), snapshot EBS for forensics, revoke IAM role sessions, notify. Security Hub custom actions let an analyst trigger the same automation manually.
- **Scaling responses.** Runbooks can adjust capacity (update an Auto Scaling group, change a DynamoDB capacity mode) when an alarm fires, but prefer native scaling policies first; use runbooks for actions scaling policies cannot do.
- **Step Functions vs Automation.** Automation fits resource operations on AWS and managed nodes, with built-in rate controls and targets. Step Functions fits multi-service workflows, human approvals with task tokens, long waits and complex branching. Step Functions can start Automation executions and the reverse.
- **Incident response record.** With Incident Manager closed to new customers, use OpsCenter OpsItems for tracking, alarms and EventBridge for detection, chat notifications through Amazon Q Developer in chat applications, and runbooks for response. AWS Security Incident Response exists for security incidents.
- Test runbooks in non-production with the same rate controls, and keep them in version control and deploy them as `AWS::SSM::Document` resources.

**Exam traps**

- Writing a Lambda function to restart instances or create snapshots when an AWS-owned Automation runbook already does it.
- Using Run Command for a multi-step remediation with approval and branching. Use Automation.
- Logging in to each account to run a runbook. Use multi-account and multi-Region Automation or Quick Setup.
- Treating Amazon Inspector as the patching tool. It finds vulnerabilities; Patch Manager installs patches.
- Designing new incident response around Incident Manager response plans.

**Hands-on**

1. **Config non-compliance to Automation.** Enable the AWS Config managed rule `s3-bucket-public-read-prohibited` or `restricted-ssh`, and attach automatic remediation with the matching AWS-owned runbook (for example `AWS-DisablePublicAccessForSecurityGroup`) with retry attempts of 2. Open SSH to 0.0.0.0/0 on a test security group and watch Config flag it and the runbook close it. *Cost/cleanup:* Config charges per configuration item recorded and rule evaluation; limit recording to the resource types you test. Delete the rule, remediation configuration and test group, and stop the recorder.
2. **Patch policy with Quick Setup.** Launch two `t3.micro` Amazon Linux instances. In Systems Manager Quick Setup, create a patch policy for the current account and Region that scans daily and installs weekly, with rate control. Trigger an on-demand scan, review patch compliance in Patch Manager, and see how the policy created State Manager associations. *Cost/cleanup:* Free tier instances; Systems Manager patching for EC2 has no extra charge. Delete the Quick Setup configuration and terminate the instances.

### Practice questions: Task 5.4

**Question 5.4.1** An online brokerage runs its order API on an EC2 Auto Scaling group behind an Application Load Balancer. Traffic rises sharply every weekday at market open and the exact pattern shifts by a few minutes from week to week. Instances take 6 minutes to boot and warm caches, so target tracking alone causes errors for the first minutes of every open. The company wants capacity to be ready before market open without maintaining schedules, to scale in after the morning peak, and to handle unexpected news-driven spikes. Which solution meets these requirements?

- A. Add a predictive scaling policy in forecast and scale mode with a scheduling buffer time that covers boot time, and keep a target tracking policy on ALB requests per target.
- B. Create scheduled actions that raise minimum capacity 15 minutes before market open each weekday and lower it at noon, and keep a step scaling policy for spikes.
- C. Replace target tracking with a predictive scaling policy only, and set the maximum capacity breach behavior to increase maximum capacity when the forecast exceeds it.
- D. Configure a warm pool of stopped instances and a step scaling policy on CPU utilization with a short cooldown, and remove target tracking.

<details>
<summary>Answer and explanation</summary>

**Correct answer: A**

**Why A is correct:** Predictive scaling learns the daily and weekly pattern from history and adapts as it shifts, so no schedules need maintaining. The scheduling buffer time launches instances ahead of the forecast hour so they are warm at market open. Predictive scaling does not scale in, so the target tracking policy handles scale-in after the peak and reacts to unexpected spikes; the group uses the higher of the two desired capacities.

**Why the other options are incorrect:**

- **B:** Scheduled actions work but must be maintained as the open pattern shifts, which the company wants to avoid, and step scaling is more tuning effort than target tracking.
- **C:** With predictive scaling alone the group never scales in from forecasts, and there is no dynamic policy to react to unexpected spikes.
- **D:** A warm pool shortens start time but scaling is still reactive to CPU after load arrives. Step scaling on CPU is a weaker signal than requests per target.

**Key takeaway:** Predictive scaling for recurring patterns plus target tracking for scale-in and surprises; use scheduling buffer time for slow-booting instances.

</details>

**Question 5.4.2** A company uses AWS Config across 50 accounts to detect EC2 instances that are missing the required CloudWatch agent configuration. A remediation runbook restarts the agent with new configuration, but last month an automatic remediation ran against 400 production instances at once and a bug in the configuration took a payment service offline. The operations team wants remediation to continue automatically but with limits that stop a bad change from spreading, and to halt immediately if the payment service's error rate rises. Which combination of steps meets these requirements? (Select TWO.)

- A. Replace Config automatic remediation with a State Manager association or scheduled multi-account Automation execution that runs the runbook against noncompliant instances with `MaxConcurrency` of 5 percent and `MaxErrors` of 2 per account and Region.
- B. Attach the payment service's error rate CloudWatch alarm to that execution so it is canceled when the alarm enters ALARM.
- C. Set the Config remediation's maximum automatic attempts to 1 and its retry interval to 60 seconds.
- D. Replace automatic remediation with a weekly manual remediation by an engineer who reviews the noncompliant list in each account.
- E. Add an SCP that denies `ssm:SendCommand` in production accounts during business hours.

<details>
<summary>Answer and explanation</summary>

**Correct answers: A and B**

**Why A and B are correct:** Rate controls make Automation work through targets in small batches, and the error threshold stops sending the runbook to more instances after a few failures, which contains a bad change (A). Automation can watch a CloudWatch alarm and cancel the execution when it goes into ALARM, so a rising payment error rate halts remediation immediately (B). Together they keep automation while adding a circuit breaker.

**Why the other options are incorrect:**

- **C:** Retry settings control how many times Config retries remediation for one resource. They do not limit how many resources are remediated at the same time. (Config's separate `SsmControls` execution controls can rate-limit remediations, but they cannot cancel a run when a business alarm fires, which is why the remediation moves to a rate-controlled, alarm-aware Automation execution.)
- **D:** It removes automation, which the team wants to keep, and adds manual effort across 50 accounts.
- **E:** It blocks remediation by time window but does nothing to limit the blast radius outside those hours, and it can block legitimate operations.

**Key takeaway:** Safe remediation uses concurrency limits, error thresholds and an alarm-based stop condition.

</details>

**Question 5.4.3** A payments processor consumes card authorization events from a Kinesis data stream with an AWS Lambda function. Occasionally one malformed record causes the function to throw, and the whole shard stops progressing for hours while Lambda retries the same batch, delaying thousands of valid events. The team wants valid records to keep flowing, malformed records to be kept for investigation with their context, and no change to the producer. Which solution meets these requirements?

- A. Add an SQS dead-letter queue to the Lambda function's asynchronous invocation configuration, and set the maximum retry attempts for asynchronous invocation to 0.
- B. Increase the batch size and parallelization factor on the event source mapping so the bad record is processed together with more good records.
- C. Configure the event source mapping with `BisectBatchOnFunctionError`, a low `MaximumRetryAttempts`, partial batch responses through `ReportBatchItemFailures`, and an SQS on-failure destination.
- D. Wrap the function's handler in a try and catch that logs and discards any record that fails parsing, and return success for every batch.

<details>
<summary>Answer and explanation</summary>

**Correct answer: C**

**Why C is correct:** For stream sources, retry behavior is configured on the event source mapping. Bisecting a failing batch isolates the bad record, partial batch responses let the function report only the failed sequence number so the good records are checkpointed, and a low retry limit stops the shard from being blocked. The on-failure destination receives details of the failed records for investigation.

**Why the other options are incorrect:**

- **A:** Stream event source mappings invoke the function synchronously, so asynchronous invocation settings and function DLQs do not apply.
- **B:** A larger batch still fails as a whole, and parallelization keeps per-key ordering, so the bad record still blocks its partition.
- **D:** Records continue to flow, but malformed records are discarded, which fails the requirement to keep them with context.

**Key takeaway:** For Kinesis and DynamoDB Streams, use bisect, retry limits, partial batch responses and on-failure destinations on the event source mapping.

</details>

**Question 5.4.4** A company runs 2,000 EC2 instances across 30 accounts and three Regions in AWS Organizations. Amazon Inspector reports critical OS vulnerabilities, and auditors want a consistent patching process with weekly installation during maintenance windows, daily compliance scans, central compliance reporting, and no more than 10 percent of instances in an application patched at the same time. Several accounts have instances without an instance profile. Which solution meets these requirements with the least operational overhead?

- A. Create a Lambda function in each account that reads Inspector findings and runs `yum update` through Run Command on affected instances, scheduled weekly by EventBridge Scheduler.
- B. Create maintenance windows and patch baselines in each account and Region with a CloudFormation StackSet, and attach the AmazonSSMManagedInstanceCore policy to each instance role by hand.
- C. Bake patched AMIs monthly with EC2 Image Builder, and replace instances by updating launch templates in every Auto Scaling group across the organization.
- D. Use Systems Manager Quick Setup from the delegated administrator to enable Default Host Management Configuration and create an organization patch policy with scan and install schedules, rate control, and reports to S3.

<details>
<summary>Answer and explanation</summary>

**Correct answer: D**

**Why D is correct:** Quick Setup patch policies deploy patch baselines, scan and install schedules and rate controls across accounts and Regions in an organization from one place, with compliance data collected centrally. Default Host Management Configuration lets Systems Manager manage instances that have no instance profile, so the unmanaged instances are covered without editing each one. Rate control holds patching to 10 percent at a time.

**Why the other options are incorrect:**

- **A:** It is custom code in every account, patches only instances with current findings rather than following a baseline, and has no central compliance reporting.
- **B:** StackSets can distribute maintenance windows, but manual instance role changes across 30 accounts and three Regions is high effort, and compliance reporting still needs to be built.
- **C:** Image Builder pipelines are a good immutable pattern for Auto Scaling groups, but many of the 2,000 instances may not be in groups, the monthly cadence misses the weekly requirement, and there is no daily compliance scan.

**Key takeaway:** For organization-wide patching, use Quick Setup patch policies with Default Host Management Configuration; Inspector finds, Patch Manager fixes.

</details>

**Question 5.4.5** A SaaS company runs a multi-tenant document processing service. Each request calls a third-party OCR API that sometimes slows down for minutes. When that happens, Lambda functions wait on the API, concurrency rises to the account limit, and unrelated functions in the same account are throttled. Large tenants also send bursts that delay small tenants. The company wants to protect other workloads, stop hammering the OCR API while it is unhealthy, and keep small tenants responsive. Which combination of steps meets these requirements? (Select THREE.)

- A. Set reserved concurrency on the OCR processing function to cap how much of the account concurrency it can use.
- B. Put requests on separate SQS queues for large and small tenants, each with its own consumer function and maximum concurrency on its event source mapping.
- C. Add a circuit breaker that records OCR failures in a DynamoDB table and skips the OCR call for a cool-down period after a failure threshold, returning a retry-later status.
- D. Increase the Lambda function timeout to 15 minutes so slow OCR calls complete instead of failing.
- E. Configure the AWS SDK and HTTP client to retry OCR calls immediately up to 10 times without delay.
- F. Raise the account's Lambda concurrency quota through Service Quotas so the OCR function no longer reaches the limit.

<details>
<summary>Answer and explanation</summary>

**Correct answers: A, B and C**

**Why A, B and C are correct:** Reserved concurrency is a bulkhead that caps the OCR function so the rest of the account keeps capacity (A). Separate queues with capped consumers isolate large tenants from small tenants, so a burst fills only its own queue (B). A shared circuit breaker stops calls to the OCR API while it is unhealthy and lets the system fail fast instead of tying up concurrency (C).

**Why the other options are incorrect:**

- **D:** Longer timeouts keep more invocations waiting on the slow API, which uses more concurrency and makes the throttling worse.
- **E:** Immediate retries without backoff increase load on an already unhealthy API and create a retry storm.
- **F:** A higher quota delays the problem but does not isolate other workloads; a slow dependency will eventually consume the larger limit too, and it does nothing for tenant fairness.

**Key takeaway:** Combine bulkheads (reserved concurrency, per-tenant queues) with circuit breakers; never respond to a slow dependency with longer timeouts or immediate retries.

</details>

---

## In-scope AWS services and documentation

All links were checked on 5 November 2026, either by fetching the page or by matching the exact URL in AWS documentation search results.

### Analytics

| Service | Documentation | Read first |
|---|---|---|
| Amazon AppFlow | https://docs.aws.amazon.com/appflow/latest/userguide/what-is-appflow.html | Flows from SaaS apps to S3 and Redshift; private flows over PrivateLink |
| Amazon Athena | https://docs.aws.amazon.com/athena/latest/ug/what-is.html | Querying S3 data lakes with the Glue Data Catalog; federated queries |
| Amazon EMR | https://docs.aws.amazon.com/emr/latest/ManagementGuide/emr-what-is-emr.html | Cluster node types and Spot usage; EMR Serverless versus EMR on EC2 |
| Amazon Kinesis Data Streams | https://docs.aws.amazon.com/streams/latest/dev/introduction.html | Shards and on-demand capacity mode; enhanced fan-out consumers |
| Amazon MSK | https://docs.aws.amazon.com/msk/latest/developerguide/what-is-msk.html | MSK Serverless versus provisioned; MSK Connect |
| Amazon OpenSearch Service | https://docs.aws.amazon.com/opensearch-service/latest/developerguide/what-is.html | Domain sizing and Multi-AZ with standby; OpenSearch Serverless collections |
| Amazon QuickSight | https://docs.aws.amazon.com/quicksight/latest/user/welcome.html | SPICE versus direct query; row-level security (page now titled "What is Amazon Quick?") |
| Amazon Redshift | https://docs.aws.amazon.com/redshift/latest/mgmt/welcome.html | Redshift Spectrum and data sharing; RA3 nodes and Redshift Serverless |
| AWS Clean Rooms | https://docs.aws.amazon.com/clean-rooms/latest/userguide/what-is.html | Collaborations and analysis rules; sharing data without copying it |
| AWS Glue | https://docs.aws.amazon.com/glue/latest/dg/what-is-glue.html | Data Catalog and crawlers; ETL jobs and job bookmarks |
| AWS Lake Formation | https://docs.aws.amazon.com/lake-formation/latest/dg/what-is-lake-formation.html | Fine-grained and tag-based access control; cross-account data sharing |

### Application Integration

| Service | Documentation | Read first |
|---|---|---|
| Amazon EventBridge | https://docs.aws.amazon.com/eventbridge/latest/userguide/eb-what-is.html | Event buses and cross-account or cross-Region routing; EventBridge Pipes and Scheduler |
| Amazon MQ | https://docs.aws.amazon.com/amazon-mq/latest/developer-guide/welcome.html | When to pick MQ over SQS/SNS (lift-and-shift of JMS, AMQP, MQTT apps); active/standby brokers |
| Amazon SNS | https://docs.aws.amazon.com/sns/latest/dg/welcome.html | Fan-out to SQS; message filtering and FIFO topics |
| Amazon SQS | https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/welcome.html | Standard versus FIFO queues; visibility timeout and dead-letter queues |
| AWS Step Functions | https://docs.aws.amazon.com/step-functions/latest/dg/welcome.html | Standard versus Express workflows; error handling with Retry and Catch |

### Cloud Financial Management

| Service | Documentation | Read first |
|---|---|---|
| AWS Budgets | https://docs.aws.amazon.com/cost-management/latest/userguide/budgets-managing-costs.html | Budget types and alerts; budget actions that apply IAM policies or SCPs |
| AWS Cost and Usage Report | https://docs.aws.amazon.com/cur/latest/userguide/what-is-cur.html | Delivering reports to S3 and querying with Athena; CUR 2.0 through Data Exports |
| AWS Cost Categories | https://docs.aws.amazon.com/awsaccountbilling/latest/aboutv2/manage-cost-categories.html | Category rules by account, tag and service; split charges for shared costs |
| AWS Cost Explorer | https://docs.aws.amazon.com/cost-management/latest/userguide/ce-what-is.html | Filtering and grouping by tag and account; RI and Savings Plans recommendations |
| Reserved Instances | https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-reserved-instances.html | Standard versus Convertible RIs; how RI discounts apply and are shared across an organization |
| Savings Plans | https://docs.aws.amazon.com/savingsplans/latest/userguide/what-is-savings-plans.html | Compute versus EC2 Instance Savings Plans; sharing across consolidated billing |

### Compute

| Service | Documentation | Read first |
|---|---|---|
| Amazon EC2 | https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/concepts.html | Purchasing options (On-Demand, Spot, Capacity Reservations); placement groups |
| AWS Application Auto Scaling | https://docs.aws.amazon.com/autoscaling/application/userguide/what-is-application-auto-scaling.html | Target tracking and step scaling; which services it supports (ECS, DynamoDB, Aurora, and more) |
| AWS Elastic Beanstalk | https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/Welcome.html | Deployment policies (rolling, immutable, blue/green with URL swap) |
| AWS Fargate | https://docs.aws.amazon.com/AmazonECS/latest/developerguide/AWS_Fargate.html | Task sizing and networking (awsvpc mode); Fargate Spot |
| AWS Lambda | https://docs.aws.amazon.com/lambda/latest/dg/welcome.html | Concurrency (reserved and provisioned); Lambda in a VPC |
| AWS Local Zones | https://docs.aws.amazon.com/local-zones/latest/ug/what-is-aws-local-zones.html | Local Zones for single-digit millisecond latency; which services run in them |
| AWS Outposts | https://docs.aws.amazon.com/outposts/latest/userguide/what-is-outposts.html | Outposts racks versus servers; service link and local gateway connectivity |

### Containers

| Service | Documentation | Read first |
|---|---|---|
| Amazon ECR | https://docs.aws.amazon.com/AmazonECR/latest/userguide/what-is-ecr.html | Cross-Region and cross-account replication; image scanning |
| Amazon ECS | https://docs.aws.amazon.com/AmazonECS/latest/developerguide/Welcome.html | Capacity providers and launch types; blue/green deployments with CodeDeploy |
| Amazon EKS | https://docs.aws.amazon.com/eks/latest/userguide/what-is-eks.html | Managed node groups, Fargate profiles and Auto Mode; IAM roles for service accounts and Pod Identity |

### Database

| Service | Documentation | Read first |
|---|---|---|
| Amazon Aurora | https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/CHAP_AuroraOverview.html | Aurora Global Database and failover; Aurora Serverless v2 |
| Amazon DocumentDB | https://docs.aws.amazon.com/documentdb/latest/devguide/what-is.html | Cluster architecture and replicas; global clusters |
| Amazon DynamoDB | https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/Introduction.html | Global tables; capacity modes and DAX |
| Amazon ElastiCache | https://docs.aws.amazon.com/AmazonElastiCache/latest/dg/WhatIs.html | Caching strategies (lazy loading, write-through); Valkey/Redis OSS replication and global datastore |
| Amazon Keyspaces | https://docs.aws.amazon.com/keyspaces/latest/devguide/what-is-keyspaces.html | Migrating Cassandra workloads; multi-Region replication |
| Amazon MemoryDB | https://docs.aws.amazon.com/memorydb/latest/devguide/what-is-memorydb.html | Durable in-memory primary database versus ElastiCache cache |
| Amazon Neptune | https://docs.aws.amazon.com/neptune/latest/userguide/intro.html | Graph use cases (fraud, social, knowledge graphs); Neptune global database |
| Amazon RDS | https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Welcome.html | Multi-AZ instance versus Multi-AZ cluster; cross-Region read replicas and RDS Proxy |
| Amazon Timestream | https://docs.aws.amazon.com/timestream/latest/developerguide/what-is-timestream.html | Timestream for LiveAnalytics versus Timestream for InfluxDB; memory and magnetic storage tiers |

### Developer Tools

| Service | Documentation | Read first |
|---|---|---|
| AWS CDK | https://docs.aws.amazon.com/cdk/v2/guide/home.html | Constructs, stacks and apps; CDK Pipelines |
| AWS CloudFormation | https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/Welcome.html | StackSets with service-managed permissions across an organization; change sets, drift detection and DeletionPolicy |
| AWS CodeBuild | https://docs.aws.amazon.com/codebuild/latest/userguide/welcome.html | buildspec file; running builds inside a VPC |
| AWS CodeCommit | https://docs.aws.amazon.com/codecommit/latest/userguide/welcome.html | Repository access with IAM; check current availability for new customers |
| AWS CodeDeploy | https://docs.aws.amazon.com/codedeploy/latest/userguide/welcome.html | In-place versus blue/green deployments; deployment configurations for EC2, Lambda and ECS |
| AWS CodePipeline | https://docs.aws.amazon.com/codepipeline/latest/userguide/welcome.html | Cross-account and cross-Region pipelines; manual approval actions |
| AWS Distro for OpenTelemetry | https://aws-otel.github.io/docs/introduction | Collector deployment patterns; sending traces to X-Ray and metrics to CloudWatch (this is the AWS-run project site, not docs.aws.amazon.com) |
| AWS X-Ray | https://docs.aws.amazon.com/xray/latest/devguide/aws-xray.html | Service map and traces; sampling rules |

### Machine Learning and AI

| Service | Documentation | Read first |
|---|---|---|
| Amazon Bedrock | https://docs.aws.amazon.com/bedrock/latest/userguide/what-is-bedrock.html | Model access and inference options (on-demand, provisioned throughput, cross-Region inference); private access over PrivateLink |
| Amazon Bedrock AgentCore | https://docs.aws.amazon.com/bedrock-agentcore/latest/devguide/what-is-bedrock-agentcore.html | Runtime, Gateway, Memory and Identity components; securing agent tool access |
| Amazon Bedrock Knowledge Bases | https://docs.aws.amazon.com/bedrock/latest/userguide/knowledge-base.html | How retrieval-augmented generation works; choosing a vector store |
| Amazon Bedrock Guardrails | https://docs.aws.amazon.com/bedrock/latest/userguide/guardrails.html | Content filters, denied topics and sensitive information filters; applying guardrails to agents and knowledge bases |
| Amazon Bedrock Evaluations | https://docs.aws.amazon.com/bedrock/latest/userguide/evaluation.html | Model evaluation jobs; RAG evaluation and LLM-as-a-judge |
| Amazon SageMaker AI | https://docs.aws.amazon.com/sagemaker/latest/dg/whatis.html | Inference endpoint options (real-time, serverless, asynchronous, batch); running in a VPC without internet access |

### Management and Governance

| Service | Documentation | Read first |
|---|---|---|
| Amazon CloudWatch | https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/WhatIsCloudWatch.html | Cross-account observability; composite alarms and metric math |
| Amazon CloudWatch Logs | https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/WhatIsCloudWatchLogs.html | Subscription filters to central accounts; Logs Insights queries |
| CloudWatch Application Signals | https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/CloudWatch-Application-Monitoring-Sections.html | Service level objectives (SLOs); automatic instrumentation with OpenTelemetry |
| CloudWatch Synthetics | https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/CloudWatch_Synthetics_Canaries.html | Canary blueprints; canaries that run inside a VPC |
| CloudWatch RUM | https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/CloudWatch-RUM.html | App monitor setup; page load and error metrics from real users |
| CloudWatch Container Insights | https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/ContainerInsights.html | Setting it up on ECS and EKS; enhanced observability metrics |
| AWS CloudTrail | https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-user-guide.html | Organization trails; data events; CloudTrail Insights |
| AWS Compute Optimizer | https://docs.aws.amazon.com/compute-optimizer/latest/ug/what-is-compute-optimizer.html | Rightsizing recommendations; enabling it across an organization |
| AWS Config | https://docs.aws.amazon.com/config/latest/developerguide/WhatIsConfig.html | Conformance packs and aggregators; automatic remediation with Systems Manager documents |
| AWS Control Tower | https://docs.aws.amazon.com/controltower/latest/userguide/what-is-control-tower.html | Landing zone and Account Factory (including AFT); preventive, detective and proactive controls |
| AWS Organizations | https://docs.aws.amazon.com/organizations/latest/userguide/orgs_introduction.html | OU design; delegated administrator for other services |
| Organizations: service control policies (SCPs) | https://docs.aws.amazon.com/organizations/latest/userguide/orgs_manage_policies_scps.html | How SCP inheritance and evaluation work; deny-list versus allow-list strategy |
| Organizations: resource control policies (RCPs) | https://docs.aws.amazon.com/organizations/latest/userguide/orgs_manage_policies_rcps.html | Which services support RCPs; using RCPs for a data perimeter |
| Organizations: tag policies | https://docs.aws.amazon.com/organizations/latest/userguide/orgs_manage_policies_tag-policies.html | Enforcing tag keys and values; reporting on non-compliant resources |
| AWS Resilience Hub | https://docs.aws.amazon.com/resilience-hub/latest/userguide/what-is.html | Resiliency policies with RTO and RPO targets; assessments and recommendations |
| AWS Resource Access Manager (RAM) | https://docs.aws.amazon.com/ram/latest/userguide/what-is.html | Sharing subnets, Transit Gateways and Resolver rules across accounts |
| Service Quotas | https://docs.aws.amazon.com/servicequotas/latest/userguide/intro.html | Quota request templates for new accounts; CloudWatch alarms on quota usage |
| AWS Systems Manager | https://docs.aws.amazon.com/systems-manager/latest/userguide/what-is-systems-manager.html | Session Manager, Patch Manager and Automation; Parameter Store |

### Migration and Transfer

| Service | Documentation | Read first |
|---|---|---|
| AWS Transform MGN (previously AWS Application Migration Service) | https://docs.aws.amazon.com/mgn/latest/ug/what-is-mgn.html | Replication agent and staging area; test and cutover launches |
| AWS Data Transfer Terminal | https://docs.aws.amazon.com/datatransferterminal/latest/userguide/what-is-dtt.html | When to bring your own storage devices to a terminal for fast physical upload |
| AWS Database Migration Service (DMS) | https://docs.aws.amazon.com/dms/latest/userguide/Welcome.html | Full load plus change data capture (CDC); DMS Serverless and homogeneous migrations |
| AWS DataSync | https://docs.aws.amazon.com/datasync/latest/userguide/what-is-datasync.html | Agents and supported locations; DataSync versus Storage Gateway versus Transfer Family |
| AWS Schema Conversion Tool (SCT) | https://docs.aws.amazon.com/SchemaConversionTool/latest/userguide/CHAP_Welcome.html | Assessment reports for heterogeneous migrations; DMS Schema Conversion as the managed alternative |
| AWS Storage Gateway | https://docs.aws.amazon.com/storagegateway/ | S3 File Gateway, Volume Gateway and Tape Gateway use cases (each gateway type has its own user guide, linked from this page) |
| AWS Transfer Family | https://docs.aws.amazon.com/transfer/latest/userguide/what-is-aws-transfer-family.html | SFTP, FTPS and AS2 endpoints; custom identity providers |
| AWS Transform | https://docs.aws.amazon.com/transform/latest/userguide/what-is-service.html | Agent-driven migration and modernization (VMware, mainframe, .NET); how it relates to MGN |

### Networking and Content Delivery

| Service | Documentation | Read first |
|---|---|---|
| Amazon API Gateway | https://docs.aws.amazon.com/apigateway/latest/developerguide/welcome.html | REST versus HTTP APIs and endpoint types (edge, Regional, private); throttling and usage plans |
| Amazon CloudFront | https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/Introduction.html | Origin access control for S3; origin failover and caching policies |
| Amazon Route 53 | https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/Welcome.html | Routing policies (failover, latency, weighted, geolocation); health checks |
| Route 53 Resolver | https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/resolver.html | Inbound and outbound endpoints for hybrid DNS; forwarding rules shared with RAM |
| Amazon VPC | https://docs.aws.amazon.com/vpc/latest/userguide/what-is-amazon-vpc.html | VPC peering limits (no transitive routing); gateway and interface endpoints |
| VPC IP Address Manager (IPAM) | https://docs.aws.amazon.com/vpc/latest/ipam/what-it-is-ipam.html | IPAM pools across an organization; avoiding overlapping CIDRs |
| Amazon VPC Lattice | https://docs.aws.amazon.com/vpc-lattice/latest/ug/what-is-vpc-lattice.html | Service networks and auth policies; Lattice versus Transit Gateway versus PrivateLink |
| AWS Cloud WAN | https://docs.aws.amazon.com/network-manager/latest/cloudwan/what-is-cloudwan.html | Core network policy and segments; Cloud WAN versus Transit Gateway |
| AWS Direct Connect | https://docs.aws.amazon.com/directconnect/latest/UserGuide/Welcome.html | Direct Connect gateway and virtual interface types (private, public, transit); resiliency models and MACsec |
| AWS Global Accelerator | https://docs.aws.amazon.com/global-accelerator/latest/dg/what-is-global-accelerator.html | Static anycast IPs and endpoint groups; Global Accelerator versus CloudFront |
| AWS Network Firewall | https://docs.aws.amazon.com/network-firewall/latest/developerguide/what-is-aws-network-firewall.html | Centralized inspection VPC deployment models; stateful rule groups |
| AWS PrivateLink | https://docs.aws.amazon.com/vpc/latest/privatelink/what-is-privatelink.html | Endpoint services behind a Network Load Balancer; sharing services across accounts and overlapping CIDRs |
| AWS Transit Gateway | https://docs.aws.amazon.com/vpc/latest/tgw/what-is-transit-gateway.html | Transit Gateway route tables; inter-Region peering |
| AWS VPN | https://docs.aws.amazon.com/vpn/latest/s2svpn/VPC_VPN.html | Site-to-Site VPN on a Transit Gateway with ECMP; Client VPN |
| Elastic Load Balancing | https://docs.aws.amazon.com/elasticloadbalancing/latest/userguide/what-is-load-balancing.html | ALB versus NLB; cross-zone load balancing and target types |
| Gateway Load Balancer | https://docs.aws.amazon.com/elasticloadbalancing/latest/gateway/introduction.html | Inserting third-party appliances with GWLB endpoints; GENEVE encapsulation |
| Amazon Application Recovery Controller (ARC) | https://docs.aws.amazon.com/r53recovery/latest/dg/what-is-route53-recovery.html | Routing control and readiness checks; zonal shift and zonal autoshift |

### Security, Identity, and Compliance

| Service | Documentation | Read first |
|---|---|---|
| Amazon Cognito | https://docs.aws.amazon.com/cognito/latest/developerguide/what-is-amazon-cognito.html | User pools versus identity pools; federation with SAML and OIDC |
| Amazon GuardDuty | https://docs.aws.amazon.com/guardduty/latest/ug/what-is-guardduty.html | Delegated administrator across an organization; protection plans (S3, EKS, Malware, RDS) |
| Amazon Inspector | https://docs.aws.amazon.com/inspector/latest/user/what-is-inspector.html | Scanning EC2, ECR images and Lambda; organization-wide activation |
| Amazon Security Lake | https://docs.aws.amazon.com/security-lake/latest/userguide/what-is-security-lake.html | OCSF format and log sources; subscribers and rollup Regions |
| Amazon Verified Permissions | https://docs.aws.amazon.com/verifiedpermissions/latest/userguide/what-is-avp.html | Cedar policies for application authorization; integration with Cognito |
| AWS Certificate Manager (ACM) | https://docs.aws.amazon.com/acm/latest/userguide/acm-overview.html | Which services can use ACM certificates; certificates for CloudFront must be in us-east-1 |
| AWS Private CA | https://docs.aws.amazon.com/privateca/latest/userguide/PcaWelcome.html | CA hierarchy design; sharing a private CA across accounts with RAM |
| AWS CloudHSM | https://docs.aws.amazon.com/cloudhsm/latest/userguide/introduction.html | When CloudHSM is required over KMS (single-tenant, FIPS 140-3 level 3 control); KMS custom key stores |
| AWS Directory Service | https://docs.aws.amazon.com/directoryservice/latest/admin-guide/what_is.html | AWS Managed Microsoft AD versus AD Connector; trusts with on-premises AD |
| AWS Identity and Access Management (IAM) | https://docs.aws.amazon.com/IAM/latest/UserGuide/introduction.html | Policy evaluation logic across accounts; permissions boundaries |
| IAM Identity Center | https://docs.aws.amazon.com/singlesignon/latest/userguide/what-is.html | Permission sets and account assignments; connecting an external identity provider |
| IAM Access Analyzer | https://docs.aws.amazon.com/IAM/latest/UserGuide/what-is-access-analyzer.html | External and unused access findings; policy validation and generation |
| IAM Roles Anywhere | https://docs.aws.amazon.com/rolesanywhere/latest/userguide/introduction.html | Trust anchors and profiles; giving on-premises workloads temporary credentials with X.509 certificates |
| AWS Key Management Service (KMS) | https://docs.aws.amazon.com/kms/latest/developerguide/overview.html | Key policies and cross-account key use; multi-Region keys |
| AWS Secrets Manager | https://docs.aws.amazon.com/secretsmanager/latest/userguide/intro.html | Automatic rotation; replicating secrets to other Regions |
| AWS Security Hub | https://docs.aws.amazon.com/securityhub/latest/userguide/what-is-securityhub.html | Cross-Region aggregation and central configuration; security standards (page now titled "Security Hub CSPM") |
| AWS Shield | https://docs.aws.amazon.com/waf/latest/developerguide/ddos-overview.html | Shield Standard versus Shield Advanced; DDoS-resilient architecture with CloudFront and Route 53 |
| AWS Verified Access | https://docs.aws.amazon.com/verified-access/latest/ug/what-is-verified-access.html | VPN-less access with trust providers; access policies |
| AWS WAF | https://docs.aws.amazon.com/waf/latest/developerguide/what-is-aws-waf.html | Managed rule groups and rate-based rules; central management with Firewall Manager |
| AWS Artifact | https://docs.aws.amazon.com/artifact/latest/ug/what-is-aws-artifact.html | Downloading compliance reports; accepting agreements such as the BAA for an organization |

### Storage

| Service | Documentation | Read first |
|---|---|---|
| Amazon Data Lifecycle Manager | https://docs.aws.amazon.com/ebs/latest/userguide/snapshot-lifecycle.html | Snapshot and AMI policies; cross-Region and cross-account copy |
| Amazon EBS | https://docs.aws.amazon.com/ebs/latest/userguide/what-is-ebs.html | Volume types (gp3, io2 Block Express); snapshots and fast snapshot restore |
| Amazon EFS | https://docs.aws.amazon.com/efs/latest/ug/whatisefs.html | Storage classes and lifecycle management; EFS replication |
| Amazon FSx | https://docs.aws.amazon.com/fsx/ | Choosing among FSx for Windows File Server, Lustre, NetApp ONTAP and OpenZFS; Multi-AZ deployments |
| Amazon S3 | https://docs.aws.amazon.com/AmazonS3/latest/userguide/Welcome.html | Storage classes and lifecycle rules; Cross-Region Replication and Object Lock |
| AWS Backup | https://docs.aws.amazon.com/aws-backup/latest/devguide/whatisbackup.html | Organization backup policies and cross-account, cross-Region copies; Backup Vault Lock |
| AWS Elastic Disaster Recovery | https://docs.aws.amazon.com/drs/latest/userguide/what-is-drs.html | Continuous block-level replication with low RPO; drills, recovery and failback |

### Testing and Resilience

| Service | Documentation | Read first |
|---|---|---|
| AWS Fault Injection Service | https://docs.aws.amazon.com/fis/latest/userguide/what-is.html | Experiment templates and stop conditions; AZ power interruption scenarios |

### Also read

| Resource | Documentation | Read first |
|---|---|---|
| AWS Well-Architected Framework (six pillars: Operational Excellence, Security, Reliability, Performance Efficiency, Cost Optimization, Sustainability) | https://docs.aws.amazon.com/wellarchitected/latest/framework/welcome.html | Design principles for each pillar; the Reliability and Cost Optimization pillars |
| Disaster Recovery of Workloads on AWS (Reliability pillar whitepaper) | https://docs.aws.amazon.com/whitepapers/latest/disaster-recovery-workloads-on-aws/disaster-recovery-workloads-on-aws.html | The four strategies (backup and restore, pilot light, warm standby, multi-site active/active); RTO and RPO tradeoffs |
| Organizing Your AWS Environment Using Multiple Accounts | https://docs.aws.amazon.com/whitepapers/latest/organizing-your-aws-environment/organizing-your-aws-environment.html | Recommended OU structure; Security and Infrastructure OUs |
| AWS Security Reference Architecture (AWS SRA) | https://docs.aws.amazon.com/prescriptive-guidance/latest/security-reference-architecture/welcome.html | Security Tooling and Log Archive accounts; where each security service is deployed |
| AWS Prescriptive Guidance: migration strategies (7 Rs) | https://docs.aws.amazon.com/prescriptive-guidance/latest/large-migration-guide/migration-strategies.html | Retire, retain, rehost, relocate, repurchase, replatform, refactor; how to pick one per application |
| SaaS Tenant Isolation Strategies | https://docs.aws.amazon.com/whitepapers/latest/saas-tenant-isolation-strategies/saas-tenant-isolation-strategies.html | Silo, pool and bridge models; isolation with IAM policies |
| Building a Data Perimeter on AWS | https://docs.aws.amazon.com/whitepapers/latest/building-a-data-perimeter-on-aws/building-a-data-perimeter-on-aws.html | Identity, resource and network perimeters; using SCPs, RCPs and VPC endpoint policies together |
| AWS Certified Solutions Architect - Professional exam page | https://aws.amazon.com/certification/certified-solutions-architect-professional/ | Registration and exam details; the page shows SAP-C03 |
| SAP-C03 exam guide | https://docs.aws.amazon.com/aws-certification/latest/solutions-architect-professional-03/solutions-architect-professional-03.html | Content domains and weightings; the comparison of SAP-C02 and SAP-C03 |

## Out-of-scope services

From the exam guide. The list is not exhaustive, and services unrelated to the role are not listed at all.

| Category | Services |
| --- | --- |
| Blockchain | Amazon Managed Blockchain |
| Compute | Amazon Lightsail |
| Databases | Amazon RDS Custom for Oracle |
| Frontend web and mobile | AWS Device Farm |
| Game tech | Amazon GameLift |
| Machine learning | Amazon Comprehend Medical, Amazon Transcribe Medical, AWS DeepComposer, AWS DeepRacer |
| Media | Amazon IVS, AWS Elemental MediaConvert, AWS Elemental MediaLive |
| Quantum | Amazon Braket |
| Satellite | AWS Ground Station |

### Services closed to new customers or retired

These may still appear in older study material. Do not choose them as the answer for a new design.

| Service | Status |
| --- | --- |
| Amazon Fraud Detector, AWS Migration Hub, AWS Application Discovery Service, AWS Snowball Edge, Amazon S3 Object Lambda, Amazon Glacier (vault service; S3 Glacier storage classes are unaffected), Systems Manager Change Manager and Incident Manager, Amazon CodeGuru Reviewer | No longer open to new customers from 7 November 2025 ([AWS announcement](https://aws.amazon.com/about-aws/whats-new/2025/10/aws-service-availability)) |
| AWS Audit Manager | No longer open to new customers from 30 April 2026 |
| AWS CloudTrail Lake | No longer open to new customers from 31 May 2026; AWS points to Amazon CloudWatch ([availability change](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-lake-service-availability-change.html)) |
| Amazon Bedrock Agents (now "Agents Classic"), Amazon SageMaker Augmented AI (A2I), Amazon Q Business | No longer open to new customers from 30 July 2026; for new agent designs use Amazon Bedrock AgentCore ([availability updates, June 2026](https://aws.amazon.com/about-aws/whats-new/2026/06/aws-service-availability/)) |
| Amazon QLDB | End of support 31 July 2025 |
| AWS App Mesh | End of support 30 September 2026; use Amazon VPC Lattice or ECS Service Connect |
| AWS CodeCommit | Was closed to new customers in July 2024, then returned to full availability in November 2025, and is in scope for SAP-C03 |

## Revision history

| Date | Change |
| --- | --- |
| 2026-11-05 | First draft against the SAP-C03 exam guide: all 5 domains, 21 tasks and 115 skills, with 105 practice questions (5 per task), 230 hands-on exercises and verified service documentation links. Every question was answered independently by a reviewer before checking the key; two answer keys (1.2.4, 1.2.5) and about 20 factual details were corrected as a result, including services AWS closed to new customers in 2026. A few details on 2025–2026 features rest on prior knowledge rather than a fresh documentation check; verify before relying on them. |
