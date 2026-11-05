# Domain 1: Architecture Design and Implementation Built for the Cloud (25%)

Tasks 1.1 to 1.5: cloud-native and event-driven design, AI/ML integration, hybrid integration, networking, and data, storage and databases.

## Task 1.1: Design architectures built for the cloud

This task tests whether you can pick the right managed building block for each part of a distributed system and justify it against a stated constraint. The decision the exam keeps asking for is the same each time: which service gives the required guarantee (ordering, durability, isolation, latency) with the least custom code and operational overhead.

### Skill 1.1.1: Event-driven messaging patterns (idempotency, ordering, delivery guarantees)

**Key concepts**

- Almost every AWS messaging service delivers at least once. Design every consumer to be idempotent, whatever service sits in front of it. 
A common way to do this is a conditional write (`attribute_not_exists`) to a DynamoDB table keyed on the event or business ID, with TTL to expire old keys. 
Powertools for AWS Lambda has an idempotency utility built on this pattern.

- **SQS standard:** close to unlimited throughput, at-least-once delivery, best-effort ordering. 
Use it when order does not matter and consumers are idempotent.

- **SQS FIFO:** 
ordering is strict only *within a message group ID*. 
Deduplication (content-based or by `MessageDeduplicationId`) works inside a 5-minute window. 
Throughput is 300 messages per second per API action without batching and 3,000 with batching. 
High-throughput mode raises this a lot, provided you spread load across many message groups. 
One hot group ID serialises processing, because only one in-flight batch per group can be consumed at a time.

- **SNS:** 
Push-based fan-out. 
SNS FIFO topics keep order and deduplicate, and they fan out to SQS FIFO queues. 
Subscription filter policies let each subscriber receive only part of the stream, with no consumer code.

- **EventBridge:** 
Content-based routing on the full event JSON. It offers schema discovery, archive and replay, scheduled rules, and API destinations for third-party HTTP targets. 
Delivery is at least once with **no ordering guarantee**. 
On failure it retries (by default for up to 24 hours) and then sends to a DLQ if you configure one per target. Choose EventBridge when routing logic and many targets matter more than order or raw throughput.

- **EventBridge Pipes:** 
point-to-point from a source (SQS, Kinesis, DynamoDB Streams, MSK, and others) to a target, with optional filtering, enrichment and transformation. It replaces "glue" Lambda functions.

- **Kinesis Data Streams:** 
Order is kept per shard. 
The partition key is hashed to a shard, so all records for one key stay in order. 
Each shard takes 1 MB/s or 1,000 records/s on write. 
Readers share 2 MB/s per shard, or get 2 MB/s each with enhanced fan-out. 
Records can be replayed for the whole retention period, and many independent consumers can read the same stream. 
A low-cardinality partition key creates hot shards.

- Decision rule:
  - Strict per-entity order plus deduplication at moderate rate: 
  SQS FIFO with entity ID as the group ID (SNS FIFO in front if you need fan-out).
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

- **DynamoDB:** 
the default store for session and entity state that must be durable. 
Expect single-digit-millisecond reads at any scale. 
TTL deletes expired sessions for free, and conditional writes and transactions give you optimistic locking. 
Items are limited to 400 KB, so large blobs go to S3 with a pointer in DynamoDB.

- **ElastiCache (Valkey, Redis OSS, Memcached):** 
sub-millisecond access for hot session data, rate-limit counters, leaderboards and pub/sub. 
ElastiCache Serverless removes node sizing. Use Valkey or Redis OSS with replication and Multi-AZ when losing the cache would hurt users. Use Memcached only for a simple, multi-threaded, non-persistent cache.

- **S3:** 
large objects, documents and checkpoints. It has strong read-after-write consistency for all operations. Conditional writes (`If-None-Match`) let you prevent overwriting an object that already exists, which is useful for idempotent writers.
- ALB sticky sessions are a smell in a "stateless" scenario. They pin users to a target, cause uneven load, and lose sessions when the target is replaced. They are the right answer only when you cannot change a legacy application.
- For Lambda, anything held in `/tmp` or global variables is only an optimisation inside one execution environment. Never rely on it for correctness.
- DynamoDB Accelerator (DAX) caches DynamoDB reads with an API-compatible client. It is eventually consistent for cached reads. Choose ElastiCache for general-purpose caching and DAX for read-heavy DynamoDB without changing your data access code much.

**Exam traps**

- "Store sessions on an EFS mount shared by all instances." 
This works, but it adds latency and gives no TTL or atomic counters. DynamoDB or ElastiCache is the expected answer.

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

   *Cost/cleanup:* 
   ElastiCache Serverless bills per GB-hour stored and per ECPU, and each cache has a minimum billed storage amount. Even when idle, it keeps charging every hour until you delete it. 
   Delete the cache the same day. 
   A VPC Lambda function needs no NAT gateway for this, so do not create one.

### Skill 1.1.3: Serverless data processing and analysis pipelines

**Key concepts**

- **Standard pattern:** 
Producers send to Kinesis Data Streams or straight to Amazon Data Firehose. 
Firehose then buffers, optionally transforms with Lambda, converts JSON to Parquet or ORC using a Glue Data Catalog schema, and writes partitioned objects to S3. 
The Glue Data Catalog holds the schema, and Athena queries it. 
There are no servers to manage at any stage.

- **Kinesis Data Streams vs Firehose:**
  - Kinesis Data Streams gives custom consumers, ordering, replay and sub-second latency.
  - Firehose is fully managed delivery to S3, Redshift, OpenSearch, Splunk, HTTP endpoints and others. It buffers by size or time (seconds to minutes) and has no replay.
  - If the scenario says "near real time into S3 with no code", choose Firehose.

- **AWS Glue:** 
serverless Spark or Python shell ETL jobs, crawlers to infer schemas, job bookmarks for incremental processing, and Glue Data Quality rules. Glue streaming jobs can read from Kinesis or Kafka.

- **Athena:** 
pay per data scanned. Cost and speed depend on columnar formats (Parquet or ORC), compression, partitioning and partition projection. Partition projection avoids crawling and managing very large numbers of partitions. Use workgroups for per-team query limits and result locations.

- **Lambda in pipelines:** 
good for light per-record transforms, enrichment and fan-out. It is limited to 15 minutes and a fixed memory ceiling. For joins, large aggregations or anything over large data volumes, use Glue or EMR Serverless.

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

   *Cost/cleanup:* 
   On-demand Kinesis bills per stream-hour plus data, so delete the stream within the hour. 
   Delete the function and queue.

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

- **Step Functions integrations:** 
more than 200 services through AWS SDK integrations, so many steps need no Lambda at all. 
`.sync` waits for jobs such as Glue, ECS or Batch tasks. 
Distributed Map gives massive parallelism. Retry and Catch are declared per state.

- **Lambda durable functions** (launched at re:Invent 2025):

  - You write the workflow as ordinary code (Python, Node.js or Java, and container images too) using the Durable Execution SDK.
  - `context.step()` checkpoints each step's result. `context.wait()` pauses without consuming compute. `context.waitForCallback()` pauses until an external system sends success or failure to the Lambda API, for example for human approval. `parallel()` and `map()` fan out work, and `invoke()` chains to another function in the same account.
  - On resume, Lambda re-runs the handler from the top and *replays* completed steps from the checkpoint log instead of re-executing them. Code outside steps must therefore be deterministic: no direct random values, clock reads or API calls.

- **Durable function limits that decide answers:**
  - Each invocation is still capped at 15 minutes. 
  A whole durable execution can last up to one year when invoked asynchronously.
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
   - Create a Node.js or Python Lambda function with durable execution enabled. 
   Use one `step` to "reserve stock", then `waitForCallback` for approval, then a final `step`.
   - Publish version 1 and invoke `function:1` asynchronously with an execution name.
   - Send the callback success with the CLI. Inspect the durable execution state and note that the first step did not re-run on resume.
   - Invoke again with the same execution name and observe that no duplicate execution starts.

   *Cost/cleanup:* a few cents at most. Delete the function and its versions.

### Skill 1.1.5: Cross-account and cross-Region event routing with event buses

**Key concepts**

- **Bus-to-bus pattern:** a rule on the source bus targets the ARN of another event bus. The target bus can be in another account, another Region, or both. The same-account, different-Region case is supported.

  - The receiving bus needs a **resource-based policy** that allows the sender: specific account IDs, or the whole organization through an `aws:PrincipalOrgID` condition.
  - The sending rule needs an **IAM role** that allows `events:PutEvents` on the target bus. A role has been mandatory for new cross-account bus targets since March 2023.

- **Single-hop rule:** 
EventBridge does not forward an event a second time. If account B receives an event from account A and has a rule sending it on to account C, the event does not reach C. The same applies when chaining buses inside one account. Central aggregation designs must therefore send straight from each source to the final bus, not through regional "collector" buses.

- **Direct cross-account targets:** 

Within the *same Region*, a rule can deliver straight to SQS, SNS, Lambda, Kinesis Data Streams or API Gateway in another account, with no bus in the target account.
  - This needs an execution role in the bus account plus a resource policy on the target.
  - Use it when the consumer team does not want to run a bus. Use bus-to-bus when the receiving team wants its own routing rules, or when the target is in another Region.

- **Billing:** 
the sending account pays for cross-account events as custom events. The receiving account is not charged for receiving them.
- In receiving rules, filter on `account` (and `region`) to tell sources apart. Never put `"*"` in the account field, because that matches nothing. Leave the field out to match all permitted senders.

- **Common designs:**
  - Hub-and-spoke: workload accounts send to a central bus in a security or operations account. The central rules fan out to SQS, Lambda, Step Functions or log archives.
  - The reverse: a central bus publishes to consumer accounts.
  - Multi-Region DR: replicate selected events to a bus in the standby Region. Archive and replay on the target bus lets you re-drive events after an incident.
- SCPs apply to the IAM roles that rules use, so organization guardrails can block unapproved cross-account event flows.

**Exam traps**

- A three-tier "workload bus to regional collector bus to central bus" chain. The third bus never receives the events because of the single-hop rule.
- Expecting a cross-account bus target to work with only the receiver's bus policy. New targets also need the sender-side IAM role.
- A direct cross-account SQS target in *another Region*. 
Direct service targets are same-Region only, so use a bus in that Region.
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

[Back to contents](README.md) · [Next: Domain 2](domain-2-security-compliance-governance.md)
