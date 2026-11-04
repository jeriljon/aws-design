# Guide

## Content Domain 1: Architecture Design and Implementation Built for the Cloud

### Tasks
• Task 1.1: Design architectures built for the cloud.
• Task 1.2: Integrate AI and ML services into architectures.
• Task 1.3: Design hybrid cloud integration patterns.
• Task 1.4: Design network architectures.
• Task 1.5: Design data, storage, and database architectures

#### Task 1.1: Design architectures built for the cloud.

##### Skill 1.1.1: 
Design event-driven architectures that use appropriate messaging patterns such as
idempotency, ordering, and delivery guarantees (for example, by using Amazon EventBridge,
Amazon SQS FIFO queues and standard queues, Amazon SNS, Amazon Kinesis with partition
keys).

##### Skill 1.1.2: 
Design stateless applications that use externalized state management (for example,
by using Amazon DynamoDB, Amazon S3, Amazon ElastiCache).

##### Skill 1.1.3: 
Design serverless data processing and analysis pipelines (for example, by using
Amazon Kinesis Data Streams, AWS Glue, Amazon Athena, AWS Lambda).

##### Skill 1.1.4: 
Design multi-step workflows that have durable execution patterns (for example, by
using AWS Step Functions, durable Lambda functions).
##### Skill 1.1.5: 
Implement cross-account and cross-Region event routing patterns that use event
buses and event forwarding.

##### Skill 1.1.1: 
Design event-driven architectures that use appropriate messaging patterns such as
idempotency, ordering, and delivery guarantees (for example, by using Amazon EventBridge,
Amazon SQS FIFO queues and standard queues, Amazon SNS, Amazon Kinesis with partition
keys).

###### Comprehensive Architectural Breakdown: Event-Driven Messaging Patterns

In the AWS Certified Solutions Architect – Professional (SAP-C03) exam, Skill 1.1.1 tests your ability to evaluate trade-offs among AWS messaging services. Questions in this domain rarely test basic service capabilities; instead, they present complex constraints regarding throughput, strict ordering, deduplication, fan-out capabilities, operational overhead, and retry behavior.

Key Architectural Concepts & Service Trade-Off Matrix

┌────────────────────────────────────────────────────────────────────────────────────────┐
│                              EVENT-DRIVEN ROUTING FLOW                                 │
└────────────────────────────────────────────────────────────────────────────────────────┘
  Producers                  Routers / Buffers                     Consumers
┌───────────┐  Pub/Sub     ┌───────────────────┐    Polling     ┌────────────────────┐
│ Applications│───────────>│ Amazon EventBridge │───────────────>│ AWS Lambda         │
│ S3 / Dynamo │ (Filtering)│   / Amazon SNS    │                │ (State Machine)    │
└───────────┘              └───────────────────┘                └────────────────────┘
                                    │
                                    │ Fan-out
                                    ▼
                           ┌───────────────────┐    Polling     ┌────────────────────┐
                           │ Amazon SQS FIFO / │───────────────>│ Amazon EC2 Worker  │
                           │ Standard Queue    │                │ Auto Scaling Fleet │
                           └───────────────────┘                └────────────────────┘
                                    │
                                 Streaming
                                    ▼
                           ┌───────────────────┐    Shard Poll  ┌────────────────────┐
                           │ Amazon Kinesis    │───────────────>│ Kinesis Client App │
                           │ Data Streams      │                │ (KCL) / Fargate    │
                           └───────────────────┘                └────────────────────┘


1. Ordering Mechanisms & Partitioning

Amazon SQS FIFO: 
Guarantees strict First-In-First-Out ordering using a MessageGroupId. 
Messages with the same MessageGroupId are delivered in exact order to a single consumer at a time. 
Throughput is limited to 300 messages/sec (or up to 3,000 messages/sec with batching) unless high-throughput mode for FIFO is enabled.

Amazon Kinesis Data Streams: Guarantees strict ordering per partition key within a specific shard. 
Multiple partition keys can map to the same shard, but records with the exact same partition key go to the same shard and are read in exact sequence. 
Kinesis supports high-throughput streaming (1 MB/s or 1,000 records/s ingress per shard) and replayability up to 365 days.

Amazon EventBridge & Standard SQS: Provide best-effort ordering. 
Do not use them if out-of-order processing compromises business logic (e.g., financial ledger entries or inventory updates).

2. Delivery Guarantees & Deduplication

At-Least-Once Delivery (Standard SQS, SNS Standard, Kinesis, EventBridge): 
Messages are delivered at least once, but duplicate messages can occur due to network retries or visibility timeout expirations. Consumers must be designed to be idempotent.

Exactly-Once Delivery / Processing:

SQS FIFO & SNS FIFO: Provide exactly-once delivery at the messaging tier using a MessageDeduplicationId 
(or content-based deduplication within a 5-minute deduplication window).

Database-Level Idempotency: 
True exactly-once processing in distributed systems is achieved by making the target consumer idempotent (e.g., using DynamoDB conditional writes attribute_not_exists(transaction_id) or maintaining a processed-message state table).

3. Fan-Out & Content-Based Filtering

SNS to SQS Fan-Out: SNS publishes a single event to multiple SQS queues in parallel. 
Adding SNS FIFO in front of SQS FIFO allows ordered fan-out.

Amazon EventBridge: Preferred over SNS when complex content-based filtering (pattern matching on JSON bodies), 
schema registry integration, native cross-account routing, or third-party SaaS event ingestion is required. 
EventBridge routes to up to 5 targets per rule.

Architectural Feature Comparison Matrix

Feature / Requirement

Amazon EventBridge

Amazon SNS (Standard / FIFO)

Amazon SQS (Standard / FIFO)

Amazon Kinesis Data Streams

Primary Pattern

Event Bus (Pub/Sub & Schema)

Pub/Sub Topic

Message Queue (Point-to-Point)

Real-time Data Streaming

Delivery Guarantee

At-least-once

At-least-once (Standard) / Exactly-once (FIFO)

At-least-once (Standard) / Exactly-once (FIFO)

At-least-once

Ordering Scope

Best-effort

None (Standard) / Strict within MessageGroupId (FIFO)

Best-effort (Standard) / Strict within MessageGroupId (FIFO)

Strict per Partition Key within a Shard

Max Retain/Buffer

Up to 24 hours (EventBridge Archive)

Immediate delivery (No queuing buffer)

14 days

24 hours (default) up to 365 days

Filtering Capabilities

Advanced JSON body pattern matching

Message attribute / Payload matching

None (Consumers poll filtered queues)

None (Consumers process shard data)

Replayability

Yes (EventReplay feature)

No

No

Yes (Re-read data stream from sequence number)

2. Decision Framework for SAP-C03 Questions

When evaluating scenario options on the exam, apply this step-by-step logic filter:

                                    ┌────────────────────────┐
                                    │   Identify System      │
                                    │     Requirement        │
                                    └───────────┬────────────┘
                                                │
                 ┌──────────────────────────────┴──────────────────────────────┐
                 ▼                                                             ▼
       Ordering Required?                                           Real-Time Replay Needed?
   ┌─────────────┴─────────────┐                                 ┌─────────────┴─────────────┐
   │                           │                                 │                           │
  YES                          NO                               YES                          NO
   │                           │                                 │                           │
   ▼                           ▼                                 ▼                           ▼
Does it require >3,000      Needs Fan-out?                  Amazon Kinesis            Check Buffering
msg/sec per group?           ┌─┴─┐                          Data Streams               Needs:
 ┌─┴─┐                      YES  NO                           (Shard key)             ┌────┴────┐
YES  NO                      │    │                                                  SQS     EventBridge
 │    │                      ▼    ▼                                              Standard    Custom Bus
 ▼    ▼                     SNS  SQS
Kinesis SQS FIFO           Topic Standard
Streams (Partition Key)


Check for Order & Throughput:

If strict ordering is required and throughput exceeds SQS FIFO limits (3,000 msg/s batched), choose Amazon Kinesis Data Streams partitioned by entity ID.

If strict ordering is required for specific entity IDs at moderate throughput, choose SQS FIFO / SNS FIFO with MessageGroupId.

Check for Fan-Out & Filtering:

If multiple downstream systems need the exact same message with schema matching or advanced JSON filtering, choose EventBridge.

If simple fan-out to standard queues is needed, choose SNS to SQS.

Check for Idempotency Requirements:

If duplicate events can arrive (e.g., from Standard SQS or API Gateway retries), the solution must implement conditional writes or a deduplication state table (such as DynamoDB) on the consumer side. Placing SQS FIFO in front of a non-idempotent consumer does not prevent duplicate executions if Lambda times out after database mutation but before queue deletion.

3. Practice Questions

Question 1

A financial services firm processes credit card transactions. The system must process transactions for each credit card number in the exact order they occur. The global peak throughput across all cards is expected to reach 25,000 transactions per second. The application must support message replay for up to 7 days in the event of downstream audit failure. Which solution meets these requirements with the LEAST operational overhead?

A. Send transactions to an Amazon SNS topic. Subscribe an Amazon SQS FIFO queue to the topic, setting the credit card number as the MessageGroupId. Use an EC2 Auto Scaling group to process messages from the queue.

B. Write transactions to an Amazon Kinesis Data Stream using the credit card number as the partition key. Configure a 7-day stream retention period. Use AWS Lambda with an Event Source Mapping to process records from the stream.

C. Send transactions to an Amazon EventBridge custom event bus. Configure an EventBridge Archive with a 7-day retention period. Set up an EventBridge Rule targeting an Amazon SQS Standard queue polled by AWS Lambda.

D. Send transactions to an Amazon SQS Standard queue. Configure AWS Lambda triggers. Store transaction IDs in an Amazon DynamoDB table with strong consistency, and use custom application logic to re-order messages before writing to the primary database.

Question 2

An e-commerce platform allows customers to place orders. When an order is placed, three downstream microservices—Inventory, Billing, and Shipping—must process the event independently. The Inventory service requires raw JSON payload filtering to receive only OrderCreated events where item availability is marked as "Local". The platform needs to support cross-account event publishing with minimal operational management. Which solution fulfills these requirements?

A. Publish order events to an Amazon SNS topic. Configure SNS subscription filter policies for each microservice queue, and map the filtered messages to three separate SQS queues owned by the microservice accounts.

B. Publish order events to an Amazon EventBridge custom event bus in the primary account. Create EventBridge rules with pattern matching on the event pattern to route filtered events to target SQS queues in respective microservice accounts.

C. Publish order events to an Amazon Kinesis Data Stream. Configure AWS Lambda functions in each microservice account to read from the stream and execute JSON filtering logic in application code.

D. Publish order events to an Amazon SQS Standard queue in the primary account. Configure SQS redrive policies to fan-out filtered messages to cross-account Amazon Simple DB instances.

Question 3

A healthcare provider processes patient vital metrics collected from IoT sensors. Due to network instability, IoT gateways frequently re-send metric payloads. Downstream analytics algorithms update patient health records in an Amazon DynamoDB table, but processing duplicate metric events results in incorrect aggregated clinical averages. The team needs to ensure processing is idempotent without sacrificing throughput. Which implementation achieves this goal?

A. Ingest metric events into an Amazon SQS FIFO queue with Content-Based Deduplication enabled. Have an AWS Lambda function poll the queue and update DynamoDB.

B. Ingest metric events into an Amazon SQS Standard queue. Have AWS Lambda poll the queue and perform an UpdateItem operation in DynamoDB using a conditional expression that checks if the metric payload's unique EventID already exists in a tracking attribute.

C. Ingest metric events into an Amazon Kinesis Data Stream. Configure AWS Lambda with an event source mapping set to ReportBatchItemFailures. Rely on Kinesis's built-in deduplication filter to drop duplicate partition keys before Lambda invocation.

D. Ingest metric events into an Amazon EventBridge bus. Create a rule with an API Destination target pointing to DynamoDB directly, setting the DeduplicationHeader parameter in the HTTP configuration.

Question 4

A retail company processes flash-sale orders. Orders are placed into an Amazon SQS Standard queue. A fleet of worker instances running on Amazon EC2 in an Auto Scaling group processes the messages and writes finalized orders to an Amazon Aurora PostgreSQL database. During heavy traffic spikes, database connections spike, causing some workers to crash mid-execution before deleting the message from the queue. This causes duplicate order writes to Aurora. What combination of actions resolves this issue while maintaining high throughput? (Select TWO.)

A. Convert the SQS Standard queue to an SQS FIFO queue to prevent duplicate processing.

B. Implement a unique order_id database constraint in PostgreSQL, and update the worker logic to perform an ON CONFLICT DO NOTHING or UPSERT operation on database insertion.

C. Increase the SQS Queue Visibility Timeout to exceed the maximum possible worker processing time including database retry logic.

D. Decrease the Visibility Timeout on the SQS queue to zero so that other worker instances can immediately take over failed database connections.

E. Replace the SQS Queue with an Amazon SNS Topic configured with raw message delivery enabled.

Question 5

A supply chain management application requires that order state updates (Created, Shipped, Delivered) for any given PackageID are processed strictly sequentially. However, order state updates across different PackageIDs can be processed concurrently. Total daily throughput across all packages is 500 messages per second. The architecture must minimize cost and operational complexity. Which design pattern should be selected?

A. Publish order state updates to an Amazon Kinesis Data Stream with PackageID as the partition key. Process the stream using AWS Lambda.

B. Publish order state updates to an Amazon SQS FIFO queue using PackageID as the MessageGroupId. Process the queue using an AWS Lambda function with concurrency scaling enabled.

C. Publish order state updates to an Amazon SQS Standard queue. Configure AWS Lambda to process messages, and use an ElastiCache for Redis cluster to maintain a lock on PackageID during processing.

D. Publish order state updates to an Amazon EventBridge Bus. Use an EventBridge Rule to route messages directly to an AWS Step Functions Standard Workflow mapped to each PackageID.

Question 6

An online ticketing platform uses an API Gateway integrated with an Amazon SQS FIFO queue to process high-volume ticket purchases during mega-events. Customers report that during peak surges, all ticket purchases stall for several minutes, even though total system capacity is well within limits. Investigation reveals that a tiny percentage of purchases (e.g., fraudulent requests or corrupted records) fail processing repeatedly and cause a line-of-blocking issue for valid orders behind them in the same queue. How should the solutions architect resolve this issue?

A. Switch the queue type from SQS FIFO to SQS Standard to eliminate line-of-blocking behavior for corrupted messages.

B. Configure a Dead-Letter Queue (DLQ) on the SQS FIFO queue, and set maxReceiveCount on the Redrive Policy to a low number (e.g., 3). Ensure the DLQ is also an SQS FIFO queue.

C. Increase the MessageRetentionPeriod of the SQS FIFO queue to 14 days so corrupted messages have more time to process without failing.

D. Decrease the VisibilityTimeout of the SQS FIFO queue to 0 seconds so failed messages bypass processing and move directly to the back of the queue.

Question 7

A logistics company uses an event-driven architecture to track vehicle telemetry. Thousands of trucks emit GPS signals every 5 seconds. The central monitoring system processes these signals using an Amazon Kinesis Data Stream with 10 shards. Due to sudden fleet expansion, telemetry volume doubles, leading to ProvisionedThroughputExceededException errors during peak hours. The solution must scale automatically to handle load spikes without message loss or manual shard management. Which solution provides the LEAST operational overhead?

A. Write a Python script using the AWS SDK (UpdateShardCount) and run it as an AWS Lambda function scheduled via EventBridge every 5 minutes to scale shards based on CloudWatch metrics.

B. Switch the Amazon Kinesis Data Stream capacity mode from Provisioned to On-Demand mode.

C. Migrate the ingestion layer from Kinesis Data Streams to an Amazon SQS Standard queue with an EC2 Auto Scaling group scaling on the ApproximateNumberOfMessagesVisible metric.

D. Increase the retention period of the Kinesis Data Stream from 24 hours to 7 days to buffer incoming data during throughput spikes.

Question 8

A media streaming enterprise processes user upload events. Upload events are published to an Amazon EventBridge custom event bus. Two separate applications consume these events: a Transcoding engine (which must process events as soon as possible) and an Archival service (which stores events in Amazon S3 for historical analytics). During a maintenance window, the Archival service was offline for 12 hours, causing all upload events destined for Archival during that period to be missed. The architect must ensure future component outages do not result in dropped events, and missed events can be replayed. Which design meets this requirement with minimal changes?

A. Replace EventBridge with an Amazon SQS Standard queue. Have both Transcoding and Archival services pull from the same queue.

B. Configure EventBridge Event Archiving on the custom event bus with a retention period of at least 14 days, and attach an SQS Dead-Letter Queue (DLQ) to the Archival rule target.

C. Configure an SNS Topic as the target for the EventBridge Rule. Subscribe an SQS queue to the SNS topic for each application, relying on SQS's default 4-day retention buffer.

D. Change the EventBridge event bus target to write directly to an Amazon Kinesis Data Firehose stream delivering to Amazon S3, bypassing application processing.

Question 9

A payment gateway processes cross-border transfers. To comply with auditing regulations, transaction requests must be recorded in an Amazon DynamoDB table and broadcasted to three external banking partner webhooks via HTTPS POST requests. If an external webhook call fails or times out, the system must retry the specific failed destination endpoint independently without re-processing successful webhook calls or failing the DynamoDB transaction write. Which architecture satisfies these constraints?

A. Write transaction data to DynamoDB. Enable DynamoDB Streams, and trigger a single AWS Lambda function that synchronously sends HTTP POST requests to all three external webhooks in a sequential loop.

B. Write transaction data to DynamoDB. Enable DynamoDB Streams, and configure an AWS Lambda function to publish the stream event to an Amazon EventBridge custom event bus. Set up three separate EventBridge Rules targeting distinct API Destinations for each banking partner webhook, with configured Dead-Letter Queues (DLQs) and retry policies for each rule target.

C. Send transaction requests directly to an Amazon SNS FIFO topic. Subscribe three SQS FIFO queues (one for each partner). Have three separate Lambda functions read from the queues, make the webhook calls, and write the record to DynamoDB upon success.

D. Send transaction requests to an Amazon SQS Standard queue. Use AWS Lambda to process the queue, write to DynamoDB, and use Step Functions inline parallel states to make the HTTP requests.

Question 10

An enterprise application generates auditing logs that are published to an Amazon SNS topic. The security team mandates that log records must be archived in an Amazon S3 bucket encrypted with AWS Key Management Service (AWS KMS) customer managed keys (CMK). However, messages published to the SNS topic fail to reach the Amazon S3 delivery pathway via Amazon Kinesis Data Firehose. Investigation shows that KMS permissions are preventing message flow. Which set of IAM and Key Policy configurations is required to fix the issue securely?

A. Configure SNS Server-Side Encryption using the AWS managed key aws/sns. Update the Kinesis Data Firehose IAM execution role to include kms:Decrypt permissions for aws/sns.

B. Configure SNS Server-Side Encryption using a Customer Managed Key (CMK). Update the KMS key policy to grant the SNS service principal (sns.amazonaws.com) permission to execute kms:GenerateDataKey* and kms:Decrypt operations.

C. Disable Server-Side Encryption on the SNS topic. Configure client-side encryption in the application before publishing events to SNS.

D. Attach an IAM Policy to the SNS topic allowing * principal access for all kms:* actions across the account.

4. Comprehensive Answer Key & Deep Distractor Breakdown

Question 1

Correct Option: B

Why B is correct:

Requirement Analysis: Requires ordering per credit card number, massive scale (25,000 transactions/sec), 7-day replay capability, and LEAST operational overhead.

Kinesis Data Streams easily handles 25,000 TPS by scaling out shards. Using the credit card number as the partition key guarantees strict ordering for transactions belonging to the exact same card. Kinesis natively supports up to 365 days of data retention for message replay, satisfying all constraints with minimal overhead.

Why incorrect options are wrong:

Option A is incorrect: SQS FIFO max throughput is 300 msg/sec (or 3,000 msg/sec with batching). It cannot support 25,000 TPS. Furthermore, SQS does not natively support message replay for historical auditing.

Option C is incorrect: While EventBridge Archive supports replay, routing to an SQS Standard queue destroys strict message ordering per credit card number.

Option D is incorrect: Building custom re-ordering logic in Lambda on top of SQS Standard introduces massive operational overhead, high failure probability under race conditions, and violates the "LEAST operational overhead" constraint.

Question 2

Correct Option: B

Why B is correct:

Requirement Analysis: Needs multi-consumer fan-out, cross-account capabilities, advanced JSON payload pattern filtering, and low operational overhead.

Amazon EventBridge is designed for custom event routing. It supports direct cross-account event targets, natively parses JSON payload contents (filtering on attributes inside the event body, such as availability == Local), and eliminates the need for consumers to execute filtering code.

Why incorrect options are wrong:

Option A is incorrect: SNS filter policies filter based on message attributes (or payload if explicitly configured), but cross-account SNS-to-SQS setup across multiple target accounts requires manual resource policy maintenance on every target SQS queue and SNS topic, creating higher operational friction than native EventBridge event bus rules.

Option C is incorrect: Requiring individual Lambda functions to read the entire stream and apply JSON filtering in custom application code increases cost and compute overhead compared to serverless declarative filtering in EventBridge.

Option D is incorrect: SimpleDB is an obsolete service. SQS redrive policies are designed for Dead-Letter Queues (DLQs), not payload-based fan-out.

Question 3

Correct Option: B

Why B is correct:

Requirement Analysis: Handing duplicate events caused by network retries (At-Least-Once delivery) while maintaining high throughput without data corruption.

DynamoDB Conditional Writes: Making the consumer idempotent is the industry standard for distributed systems. By writing metrics with an attribute_not_exists(EventID) or using an UpdateItem with conditional expressions, duplicate deliveries of the same EventID are safely rejected or overwritten without corrupting aggregated patient clinical averages.

Why incorrect options are wrong:

Option A is incorrect: SQS FIFO deduplication only works within a 5-minute sliding window (MessageDeduplicationId). If the IoT gateway retries an event after 6 minutes, SQS FIFO will treat it as a new message and process it again. Furthermore, SQS FIFO imposes throughput limits compared to Standard queues.

Option C is incorrect: Kinesis does not have a "built-in deduplication filter" that drops duplicate partition keys. Partition keys determine shard placement, not record uniqueness.

Option D is incorrect: EventBridge API Destinations cannot automatically parse arbitrary payloads and construct conditional updates directly against DynamoDB tables without an intermediary compute step (e.g., Lambda).

Question 4

Correct Options: B and C

Why B & C are correct:

Requirement Analysis: Address database connection crashes causing workers to fail mid-execution, resulting in messages returning to SQS Standard and generating duplicate records.

B (Idempotent Database Writer): Implementing an ON CONFLICT DO NOTHING or UPSERT on the order_id primary key makes the database write operation idempotent. Even if a worker crashes and SQS redelivers the message to a second worker, the insertion will not duplicate the order.

C (Adjusting Visibility Timeout): Increasing the Visibility Timeout ensures that while a worker is actively retrying database operations, SQS will keep the message invisible to other workers, preventing concurrent execution of the exact same message.

Why incorrect options are wrong:

Option A is incorrect: SQS FIFO limits throughput (3,000 TPS max) and does not fix duplicate writes caused by worker crashes after a database write has occurred but before deleting the message from the queue.

Option D is incorrect: Setting the visibility timeout to 0 seconds makes messages immediately visible to all other workers instantly, guaranteeing widespread concurrent processing collisions and DB connection pool exhaustion.

Option E is incorrect: SNS Topics deliver messages directly to subscribers and do not buffer messages for polling workers, removing the decoupling layer entirely.

Question 5

Correct Option: B

Why B is correct:

Requirement Analysis: Strict sequential ordering per PackageID, parallel execution across different PackageIDs, 500 msg/sec throughput, minimal cost.

SQS FIFO with MessageGroupId: Setting MessageGroupId = PackageID ensures messages with the same PackageID are delivered sequentially to consumers one at a time. Messages with different MessageGroupIds are processed in parallel by AWS Lambda concurrent execution threads. SQS FIFO comfortably handles 500 TPS (limit is 3,000 TPS batched) at a significantly lower cost than Kinesis streams.

Why incorrect options are wrong:

Option A is incorrect: While Kinesis Data Streams can achieve this, managing Kinesis shard provisioning is more operationally complex and cost-prohibitive for a modest throughput of 500 msg/sec compared to a serverless SQS FIFO queue.

Option C is incorrect: Using an external ElastiCache Redis distributed locking mechanism adds complex state management, latency, potential lock-deadlocks, and significant operational overhead when native SQS FIFO grouping exists out of the box.

Option D is incorrect: Step Functions Standard Workflows invoked per message at 500 msg/sec would incur extreme state-transition costs compared to queue-based polling.

Question 6

Correct Option: B

Why B is correct:

Requirement Analysis: SQS FIFO queue experiences "Line-of-Blocking" (head-of-line blocking), where a poisoned message repeatedly fails, causing all subsequent messages in the same MessageGroupId to stall.

Dead-Letter Queue (DLQ) with Redrive Policy: Configuring a DLQ on an SQS FIFO queue with a low maxReceiveCount (e.g., 3) moves corrupted or unprocessable messages out of the main queue into the DLQ after 3 failed processing attempts. This unblocks the MessageGroupId sequence and allows valid purchases behind it to resume processing immediately.

Why incorrect options are wrong:

Option A is incorrect: Converting to SQS Standard destroys strict transaction ordering, violating the business logic required for ticket sales.

Option C is incorrect: Increasing retention time keeps the corrupted message in the queue longer, aggravating the line-of-blocking problem.

Option D is incorrect: Setting VisibilityTimeout to 0 seconds causes Lambda/EC2 to endlessly pull the poisoned message in an infinite CPU loop without ever moving it out of the path of valid messages.

Question 7

Correct Option: B

Why B is correct:

Requirement Analysis: Telemetry doubling, causing ProvisionedThroughputExceededException on Kinesis Data Streams. Requires automatic scaling, zero message loss, and LEAST operational overhead.

Kinesis On-Demand Mode: Automatically scales capacity up and down in response to incoming traffic surges without manual shard management or custom scripting. It manages read/write throughput dynamically, eliminating ProvisionedThroughputExceededException with zero operational management.

Why incorrect options are wrong:

Option A is incorrect: Custom auto-scaling scripts using UpdateShardCount and Lambda introduce operational maintenance overhead, lag behind real-time traffic spikes, and can trigger Kinesis API rate limits.

Option C is incorrect: SQS Standard does not maintain strict per-truck order sequence for telemetry analytics, and replacing the stream infrastructure requires major application architecture changes.

Option D is incorrect: Retention period controls how long data stays in the stream after ingestion; it does nothing to fix ingress write throttling (ProvisionedThroughputExceededException).

Question 8

Correct Option: B

Why B is correct:

Requirement Analysis: EventBridge events missed during downstream component maintenance outage; need ability to replay past events and prevent future event loss with minimal architectural changes.

EventBridge Archiving & DLQ: EventBridge allows enabling Event Archiving directly on the event bus, which records all events and allows administrators to re-process/replay events for a specific timeframe back to targets after maintenance. Adding an SQS Dead-Letter Queue (DLQ) to the Archival rule target ensures failed deliveries during downtime are captured.

Why incorrect options are wrong:

Option A is incorrect: SQS Standard does not support fan-out to multiple independent applications natively; once one consumer pulls and deletes a message, it is gone for the other consumer.

Option C is incorrect: Re-architecting the event routing path to SNS-to-SQS requires changing event producers and rule structures, violating the "minimal changes" constraint.

Option D is incorrect: Writing directly to Firehose disables the real-time Transcoding engine consumer altogether.

Question 9

Correct Option: B

Why B is correct:

Requirement Analysis: DynamoDB transaction must succeed independently. Three external HTTP webhooks must be called, and individual webhook retries/failures must NOT affect the other webhooks or roll back the database transaction.

DynamoDB Streams + EventBridge API Destinations: DynamoDB Streams captures the write asynchronously without delaying or blocking the DB commit. Lambda publishes this change to EventBridge. EventBridge rules route the event to three distinct API Destinations (HTTPS endpoints). Each target has its own independent retry policy and Dead-Letter Queue (DLQ). If Webhook A fails, EventBridge retries Webhook A independently without re-invoking Webhooks B or C.

Why incorrect options are wrong:

Option A is incorrect: Executing webhooks sequentially inside a single Lambda function means if Webhook 3 fails, the Lambda retries all webhooks or fails the batch, causing duplicate calls to Webhooks 1 and 2.

Option C is incorrect: Placing SNS FIFO in front forces tight coupling prior to database insertion, risk of transaction rollback on webhook failures, and unnecessary cost/complexity.

Option D is incorrect: Polling SQS Standard and using inline Step Functions for synchronous webhooks introduces state machine execution costs and complex manual error handling compared to native EventBridge API Destinations.

Question 10

Correct Option: B

Why B is correct:

Requirement Analysis: Fix KMS permissions preventing SNS topics encrypted with KMS Customer Managed Keys (CMK) from delivering messages to downstream services (Kinesis Firehose / SQS).

KMS Key Policy for SNS: When SNS is encrypted using a Customer Managed Key (CMK), AWS services emitting or reading messages via SNS act as service principals. The KMS Key Policy must explicitly grant the SNS service principal (sns.amazonaws.com) permission to use the key (kms:GenerateDataKey* and kms:Decrypt). Without this explicit Key Policy grant, SNS cannot generate data keys to encrypt/decrypt payloads for downstream targets.

Why incorrect options are wrong:

Option A is incorrect: AWS managed keys (aws/sns) cannot have their key policies modified, and cross-service access (like Firehose accessing encrypted SNS topics) often fails when using AWS managed keys because key policies on default keys cannot be customized to grant cross-account or cross-service permissions.

Option C is incorrect: Disabling encryption violates the security mandate requiring KMS encryption at rest.

Option D is incorrect: Granting kms:* to * principals creates an unsecure, overly permissive policy that violates AWS security best practices and the Principle of Least Privilege.

