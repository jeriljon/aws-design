# AWS Enterprise Architecture Notes

Working notes for designing enterprise-grade systems on AWS: multi-account, secure, resilient, scalable and cost-optimized. Each section pairs core concepts with design scenarios that show how to choose between options and why.

> **Living document.** 
Service names, features and availability change often. 
Check the linked AWS documentation before relying on a detail, and record changes in the [revision history](#revision-history).

## Contents

- [How to use these notes](#how-to-use-these-notes)
- [Knowledge map](#knowledge-map)
- [1. Designing for organizational complexity](#1-designing-for-organizational-complexity)
- [2. Designing new solutions](#2-designing-new-solutions)
- [3. Continuously improving existing solutions](#3-continuously-improving-existing-solutions)
- [4. Migration and modernization](#4-migration-and-modernization)
- [Service availability notes](#service-availability-notes)
- [Revision history](#revision-history)

## How to use these notes

A useful mental model for enterprise architecture on AWS is to group services into operational pillars:

- Governance
- Networking
- Compute, containers and serverless
- Storage and data
- Resilience and migration
- Security, operations and cost

Real architectures span all of them at once: a multi-account, secure, resilient, cost-optimized system touches every pillar. 

The [AWS Well-Architected Framework](https://docs.aws.amazon.com/wellarchitected/latest/framework/welcome.html) is the reference for trade-offs across them.

Each design scenario follows the same shape: the situation, the approaches considered, the recommended approach, and why the others fall short. Use them to practise reasoning about trade-offs, not to memorise answers.

## Knowledge map

| Pillar | Key services and concepts | What an architect focuses on |
| --- | --- | --- |

| 1. Organizational governance and multi-account strategy | AWS Organizations · AWS Control Tower · Service Control Policies (SCPs) · AWS IAM Identity Center · AWS Config and CloudTrail | Moving away from siloed single accounts. Multi-account designs for billing isolation, security boundaries and autonomous business units, provisioned with automated guardrails. |

| 2. Advanced networking and hybrid connectivity | Amazon VPC · Transit Gateway · Direct Connect and VPN · Route 53 (routing policies, Resolver) · VPC sharing, endpoints and PrivateLink | Enterprise network topologies across regions and hybrid data centres; centralized inspection VPCs; egress control; IP exhaustion and overlapping CIDRs. |

| 3. Compute, containers and serverless | Amazon EC2 and Auto Scaling · ECS and EKS (Fargate and EC2) · AWS Lambda and Step Functions · Elastic Load Balancing (ALB/NLB) | Choosing the execution model by scalability, state management, operational complexity, cold-start profile and service decoupling. |

| 4. Storage, databases and data management | Amazon S3 (lifecycle, storage classes) · EBS, EFS, FSx · Amazon Aurora (Global Database, Serverless) · DynamoDB (Global Tables) · Redshift | Durability, throughput and latency; relational vs NoSQL; multi-region replication strategies. |

| 5. Resilience, disaster recovery and migration | AWS Backup · AWS Elastic Disaster Recovery · Route 53 Application Recovery Controller · AWS Database Migration Service · the 7 Rs of migration | Setting RTO/RPO and choosing a DR pattern (backup and restore, pilot light, warm standby, multi-region active-active); leading large data migrations. |

| 6. Security, operations and cost optimization (FinOps) | KMS, ACM, Secrets Manager · Trusted Advisor and Cost Explorer · Systems Manager (Patch Manager, Automation) · GuardDuty, Security Hub, Inspector | Least privilege across accounts, automated patching, centralized log analytics, and pricing models (Savings Plans, Spot, lifecycle policies). |

---

## 1. Designing for organizational complexity

Multi-account strategy, centralized governance, cross-account access, and hybrid networking at scale.

### Key services and documentation

| Service or concept | Documentation |

| --- | --- |
| AWS Organizations | [User guide](https://docs.aws.amazon.com/organizations/latest/userguide/orgs_introduction.html) |

| Organizational units (OUs) | [Managing OUs](https://docs.aws.amazon.com/organizations/latest/userguide/orgs_manage_ous.html) |

| Management account | [Security reference architecture](https://docs.aws.amazon.com/prescriptive-guidance/latest/security-reference-architecture/management-account.html) |

| Service control policies (SCPs) | [SCPs](https://docs.aws.amazon.com/organizations/latest/userguide/orgs_manage_policies_scps.html) |

| AWS Control Tower | [User guide](https://docs.aws.amazon.com/controltower/latest/userguide/what-is-control-tower.html) |

| Account Factory | [Account Factory](https://docs.aws.amazon.com/controltower/latest/userguide/account-factory.html) |

| Control Tower controls (guardrails) | [Controls reference](https://docs.aws.amazon.com/controltower/latest/controlreference/introduction.html) |

| Landing zone | [Prescriptive guidance](https://docs.aws.amazon.com/prescriptive-guidance/latest/strategy-migration/aws-landing-zone.html) |

| AWS Resource Access Manager (RAM) | [User guide](https://docs.aws.amazon.com/ram/latest/userguide/what-is.html) |

| AWS CloudTrail | [User guide](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-user-guide.html) |

| AWS Config | [Developer guide](https://docs.aws.amazon.com/config/latest/developerguide/WhatIsConfig.html) · [with Control Tower](https://docs.aws.amazon.com/controltower/latest/userguide/config.html) |

| Amazon VPC | [User guide](https://docs.aws.amazon.com/vpc/latest/userguide/what-is-amazon-vpc.html) |

| AWS Transit Gateway | [User guide](https://docs.aws.amazon.com/vpc/latest/tgw/what-is-transit-gateway.html) |

| AWS License Manager | [User guide](https://docs.aws.amazon.com/license-manager/latest/userguide/license-manager.html) |

| Amazon EC2 | [Concepts](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/concepts.html) |

| EC2 Image Builder | [User guide](https://docs.aws.amazon.com/imagebuilder/latest/userguide/what-is-image-builder.html) |

| AWS Systems Manager | [User guide](https://docs.aws.amazon.com/systems-manager/latest/userguide/what-is-systems-manager.html) |

| Amazon EBS | [User guide](https://docs.aws.amazon.com/ebs/latest/userguide/what-is-ebs.html) |

| Amazon EKS | [Deployment options](https://docs.aws.amazon.com/whitepapers/latest/overview-deployment-options/amazon-elastic-kubernetes-service.html) |

| Amazon ECS | [Developer guide](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/Welcome.html) |

| Amazon S3 | [User guide](https://docs.aws.amazon.com/AmazonS3/latest/userguide/Welcome.html) |

| Billing and Cost Management | [User guide](https://docs.aws.amazon.com/cost-management/latest/userguide/what-is-costmanagement.html) |

### 1.1 Multi-account strategy and governance

**Scope:** multi-account environments with **AWS Organizations** and **AWS Control Tower**; guardrails with **SCPs**; resource sharing with **AWS RAM**; centralized logging with **CloudTrail** and **Config**.

Enterprise architectures rarely use a single AWS account. 
They rely on multi-account structures governed by automated guardrails, cross-account resource sharing, and centralized auditing.

#### Core concepts

**AWS Organizations and AWS Control Tower.** 
A multi-account architecture separates workloads by security, billing and operational boundaries.

- **AWS Organizations** provides central governance through a hierarchy of **organizational units (OUs)** under a single **management account** (formerly the master account).

- **AWS Control Tower** builds on Organizations to automate a secure, compliant multi-account environment, known as a **landing zone**.

- **Account Factory** is the automated provisioning workflow that gives every new account standard baselines: networking, CloudTrail, IAM Identity Center integration.

- **Controls (guardrails)** come in two kinds: 
*preventive* (implemented with SCPs) and 
*detective* (implemented with AWS Config rules). 
Control Tower also offers *proactive* controls that check resources before deployment.

**Service control policies (SCPs).** 
JSON policies that set the maximum permissions for accounts in an OU or the whole organization.

- **SCPs never grant permissions.** 
They act as a ceiling on what IAM policies in the account can grant.

- **Default state:** 
the root has the `FullAWSAccess` SCP attached, which allows everything. 
A restrictive SCP cascades to all child OUs and accounts.

- **Explicit deny wins:** 
a `Deny` anywhere in the hierarchy overrides any `Allow`.

- **Management account exception:** 
SCPs never apply to the management account, which keeps full administrative power.

- **Related policy types:** 
resource control policies (RCPs) set a ceiling on permissions granted by resource-based policies, complementing SCPs.

**AWS Resource Access Manager (RAM).** 
Governed sharing of specific resources across accounts in an organization or OU. 

Commonly shared resources:
- VPC subnets
- Transit gateways
- Route 53 Resolver rules
- License Manager configurations
- EC2 Image Builder components

**Security boundaries in a shared VPC:** the VPC owner keeps control of subnets, route tables and network ACLs. Participant accounts launch their own instances into shared subnets, using their own security groups.

#### Enterprise use cases

- **Regulatory boundary in financial services:** 
isolate the PCI-DSS cardholder data environment in dedicated OUs. 
Use SCPs to block internet gateways, public S3 buckets and unapproved regions. 
Aggregate all security logs in an isolated, tamper-resistant audit account.

- **Shared network infrastructure:** 
centralize hybrid connectivity on Transit Gateway and share subnets with application teams through RAM. 
Teams cannot misconfigure enterprise routing, and internal traffic stays fast.

#### Best practices

1. **Never run workloads in the management account.** 
Restrict it to billing, Organizations management and cross-account role assumption.

2. **Delegate administration.** 
Use delegated administrator accounts for GuardDuty, Config, Security Hub and Macie, so security teams work from member accounts, not the management account.

3. **Enforce an organization trail.** Configure an organization-wide CloudTrail trail with global service events, and protect the destination bucket with strict bucket policies and S3 Object Lock.

#### Scenario 1: Central guardrails that no account can bypass

**Situation:** 
A multinational enterprise needs a multi-account architecture. 
Compliance requires that no account can disable CloudTrail and that all S3 buckets are blocked from public access. How should these guardrails be enforced centrally ?

| Approach | Assessment |
| --- | --- |

| IAM policies on the administrator role in each account, denying CloudTrail deletion and S3 public access | Weak. Local administrators with enough privilege can change or detach IAM policies, so this is not a hard organizational guardrail. |

| Lambda script that scans every account and deletes non-compliant buckets | Weak. Reactive, prone to race conditions, adds operational overhead, and does not stop the action happening. |

| **SCPs at the organization root or OU with explicit `Deny` for CloudTrail deletion and S3 public-access changes** | **Recommended.** 
SCPs set the maximum permissions for every account in scope. An explicit deny cannot be overridden even by the member account's root user. |

| Trusted Advisor with email alerts in each account | Weak. Advisory checks and notifications, not preventive controls. |

#### Scenario 2: Why an SCP seems not to work

**Situation:** 
An organization has 150 member accounts. Security attaches a custom SCP to the Production OU to stop accounts disabling GuardDuty or changing critical logging. 
An administrator in a Production account can still delete a custom IAM policy, and reports the SCP is "failing". An auditor also finds the management account running resources in an unapproved region. 
What explains both behaviours?

**Correct explanations:**

- **SCPs do not apply to the management account.** 
Guardrails for it must come from IAM policies and permissions boundaries inside it, and 
from keeping workloads out of it.

- **SCPs set a ceiling, not a grant.** 
The SCP did not deny IAM policy deletion, so whether the administrator could delete the policy depended on IAM permissions in the account. 
That is expected behaviour, not a failure.

**Common misconceptions:**

| Misconception | Reality |
| --- | --- |

| An explicit deny in an SCP only works when paired with an explicit allow for the same resource ARN | An explicit deny blocks the action on its own. |

| `FullAWSAccess` attached alongside a restrictive SCP cancels the restriction | Policies at each level combine as an intersection; `FullAWSAccess` does not neutralize a deny. |

| SCPs cannot filter resource-level permissions unless attached at the root | SCPs can use resources and conditions at any level. Attachment level changes inheritance scope only. |

#### Scenario 3: Centralized CloudTrail logs with a cross-account KMS key

**Situation:** CloudTrail logs from 45 accounts must land in one S3 bucket in a Log Archive account, encrypted with a customer managed KMS key owned by a separate Security account. In testing, delivery fails with a KMS access error. What fixes it?

| Approach | Assessment |
| --- | --- |
| Make the bucket publicly readable so CloudTrail can validate ownership | Wrong. Breaks basic security and does not address KMS permissions. |
| **Update the KMS key policy to let the CloudTrail service principal for the member accounts' trails use the key (`kms:GenerateDataKey*`), and update the bucket policy to allow CloudTrail `s3:PutObject` from all authorized accounts** | **Recommended.** Cross-account log encryption needs both the key policy and the bucket policy to be right. Readers of the logs also need `kms:Decrypt`. |
| Attach an IAM user policy to a CloudTrail role in each account with admin access to the bucket | Wrong. CloudTrail writes as a service principal, not with IAM user credentials. |
| Share the KMS key through RAM and add a lifecycle rule to remove unencrypted logs | Wrong. KMS keys are not shared through RAM; cross-account access is granted in the key policy. |

#### Scenario 4: Instances in a shared subnet cannot talk to each other

**Situation:** A Network account shares private subnets with 10 application accounts through RAM. Instances launched by Account A and Account B sit in the same shared subnet. Each team says its security group allows traffic from the other's security group, but the instances cannot communicate. What is the cause?

| Approach | Assessment |
| --- | --- |
| Instances in shared subnets cannot communicate across accounts; add VPC peering | Wrong. Instances in the same subnet communicate directly; peering is irrelevant inside one VPC. |
| **Security groups belong to their own account. A rule that references another participant's security group must use the form `account-number/security-group-id`, or the rule must use CIDR ranges instead** | **Recommended.** This is the documented way for participants to reference security groups owned by other participants or the VPC owner. |
| RAM subnet sharing is read-only; participants need a Transit Gateway to run workloads | Wrong. Participants create and manage their own ENIs and instances in shared subnets. |
| Network ACLs block cross-account traffic inside a subnet | Wrong. NACLs apply at the subnet boundary to all traffic, regardless of which account owns the instance. |

See [VPC sharing responsibilities and permissions](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-share-limitations.html).

### 1.2 Hybrid connectivity and advanced networking

**Scope:** large-scale hybrid topologies with AWS Transit Gateway, AWS Direct Connect (private and transit virtual interfaces), Site-to-Site VPN with ECMP, and cross-region peering.

#### Scenario 5: Connecting 50 VPCs across three regions to a data centre

**Situation:** A financial institution has 50 VPCs in three regions and needs secure, low-latency connectivity between all VPCs and its data centre, which connects to AWS over a dedicated 10 Gbps Direct Connect link. What gives the lowest operational overhead?

| Approach | Assessment |
| --- | --- |
| Full mesh of VPC peering plus a VPN from on-premises to each VPC | Does not scale: N(N−1)/2 peering connections, and peering is not transitive. |
| **A Transit Gateway per region, peered across regions, with all VPCs and the Direct Connect gateway attached to the local Transit Gateway** | **Recommended.** Transit Gateway acts as a cloud router with transitive routing between VPCs and on-premises across regions. |
| A cluster of EC2 routing appliances in one shared VPC | Single points of failure, throughput limits, heavy maintenance. |
| Client VPN endpoints with NAT instances in every VPC | Client VPN is for individual users, not network interconnects. |

### 1.3 Hybrid DNS and cross-account identity federation

**Scope:** Route 53 Resolver inbound and outbound endpoints for hybrid DNS; enterprise identity providers integrated with AWS IAM Identity Center (successor to AWS SSO) over SAML 2.0.

#### Scenario 6: DNS resolution in both directions

**Situation:** Applications move to AWS while core databases stay on-premises. Resources in VPCs must resolve names held by corporate DNS servers, and on-premises systems must resolve names in AWS. What is the most scalable approach?

| Approach | Assessment |
| --- | --- |
| BIND servers on EC2 in every VPC with manual zone transfers | High operational overhead and availability risk. |
| **Route 53 Resolver: inbound endpoints for queries from on-premises; outbound endpoints with conditional forwarding rules for queries from AWS** | **Recommended.** Managed, low-latency hybrid resolution in both directions. Share forwarding rules across accounts with RAM. |
| Edit `/etc/hosts` on every instance through Systems Manager | Does not scale with Auto Scaling or containers. |
| Public hosted zones for internal names | Exposes internal namespaces; a security risk. |

### 1.4 Centralized security auditing and compliance

**Scope:** cross-account audit with AWS Security Hub, Amazon GuardDuty and Amazon Inspector; CloudTrail logs aggregated in a central S3 bucket with log file integrity validation.

#### Scenario 7: Aggregated findings and automatic remediation

**Situation:** Security findings from 30 accounts must be aggregated in one security account in near real time, and any IAM policy change must trigger automated remediation. Which native combination fits?

| Approach | Assessment |
| --- | --- |
| AWS Config in each account plus daily Athena queries over logs | Batch analytics, not real-time aggregation. |
| **Security account as delegated administrator for GuardDuty and Security Hub; EventBridge rules invoking Lambda for remediation** | **Recommended.** Native multi-account aggregation, and EventBridge reacts to findings and IAM API calls immediately. |
| SNS topics in each account emailing the security team | Does not scale and cannot remediate automatically. |
| Open-source SIEM agents on every instance | Misses control-plane activity such as IAM changes; adds overhead. |

### 1.5 Cost allocation and enterprise chargeback

**Scope:** tagging strategy, AWS Cost Explorer, AWS Budgets, Cost and Usage Reports, and Trusted Advisor to allocate and govern spend across business units.

#### Scenario 8: Charging back a shared EKS cluster

**Situation:** A media company must attribute costs to Marketing, Engineering and Finance, some of which share multi-tenant EKS clusters. What gives the most granular allocation?

| Approach | Assessment |
| --- | --- |
| Separate by account ID only | Works only when workloads are isolated per account; fails for shared clusters. |
| **Cost allocation tags combined with Kubernetes namespace labels and container-level cost allocation** | **Recommended.** Account-level separation cannot see inside a shared cluster; namespace-level data can. |
| Split the bill evenly | Removes accountability and transparency. |
| Buy all capacity in one Reserved Instance pool | Changes discounts, not attribution. |

---

## 2. Designing new solutions

Resilient, high-performing, secure and cost-optimized applications designed from scratch.

### 2.1 Multi-region resilience (RTO and RPO)

**Scope:** disaster recovery patterns (backup and restore, pilot light, warm standby, active-active) with defined recovery time objectives (RTO) and recovery point objectives (RPO), using Aurora Global Database and DynamoDB Global Tables.

| Pattern | Typical RPO | Typical RTO | Relative cost |
| --- | --- | --- | --- |
| Backup and restore | Hours | Hours | Lowest |
| Pilot light | Minutes | Tens of minutes to hours | Low |
| Warm standby | Seconds to minutes | Minutes | Medium |
| Multi-region active-active | Near zero | Near zero | Highest |

#### Scenario 9: RTO under 1 minute, RPO under 5 seconds on Aurora

**Situation:** A mission-critical application on Aurora needs multi-region DR with RTO under 1 minute and RPO under 5 seconds. What meets it?

| Approach | Assessment |
| --- | --- |
| Daily snapshots copied to a second region | RPO up to 24 hours. |
| **Aurora Global Database: primary cluster in region A, secondary cluster in region B (read-only, optionally with write forwarding) that can be promoted quickly** | **Recommended.** Cross-region replication lag is typically under a second, and managed failover promotes the secondary in about a minute. |
| DMS replicating hourly to MySQL on EC2 in another region | One-hour RPO gap. |
| Manual SQL dumps copied with S3 cross-region replication | High RTO and RPO. |

### 2.2 Containers and orchestration

**Scope:** choosing between Amazon ECS (EC2 or Fargate) and Amazon EKS; microservice patterns; service discovery; ingress; container security hardening.

#### Scenario 10: 20 microservices with minimal operations

**Situation:** A startup runs 20 microservices. It wants no VM patching, no cluster control plane to manage, and native IAM permissions per task. What should the architect recommend?

| Approach | Assessment |
| --- | --- |
| Self-managed Kubernetes on EC2 | Heaviest operational load: control plane and node patching. |
| ECS on the EC2 launch type with Auto Scaling groups | Still requires managing and patching container hosts. |
| **ECS on AWS Fargate** | **Recommended.** Serverless containers: no servers to provision or patch, with IAM roles per task. |
| Elastic Beanstalk single-container environments | Poor fit for many cooperating microservices. |

### 2.3 Serverless and event-driven patterns

**Scope:** asynchronous decoupling with Amazon SQS, Amazon SNS, AWS Step Functions, Amazon EventBridge and AWS Lambda.

#### Scenario 11: Parallel order processing with retries and compensation

**Situation:** On checkout, inventory, payment and email must run in parallel. Failures need retries with exponential backoff and compensation. Which pattern fits?

| Approach | Assessment |
| --- | --- |
| One synchronous Lambda calling all three services over HTTP | No state tracking or built-in retry; timeouts cause partial failure. |
| SQS standard queue with EC2 workers | No native coordination of parallel branches. |
| **Step Functions Standard Workflow with a Parallel state, Retry and Catch handlers** | **Recommended.** Built for coordinating distributed steps, with state tracking, parallel branches, retries and compensation paths. |
| SNS fan-out to three HTTP endpoints | Fire-and-forget; no workflow state or compensation. |

### 2.4 Caching and edge delivery

**Scope:** Amazon CloudFront, AWS Global Accelerator, Amazon ElastiCache for database offloading, and Route 53 routing policies.

#### Scenario 12: Global players, one region

**Situation:** A game backend runs in `us-east-1`. Players in Asia and Europe see high API latency. The application needs static asset delivery and low-latency TCP/UDP for telemetry. What combination fits?

| Approach | Assessment |
| --- | --- |
| **CloudFront for static assets; Global Accelerator for TCP/UDP traffic** | **Recommended.** CloudFront caches at edge locations; Global Accelerator brings traffic onto the AWS backbone at the nearest edge using anycast IPs. |
| Route 53 geolocation routing to instances in `us-east-1` | Changes DNS answers only; packets still cross the public internet. |
| Direct Connect locations in every country | Direct Connect is for enterprise private circuits, not consumer traffic. |
| ElastiCache on client devices | Not possible. |

### 2.5 Purpose-built database selection

**Scope:** selecting Amazon Aurora, DynamoDB, Amazon DocumentDB, Amazon Neptune or Amazon Redshift by access pattern. (Amazon QLDB reached end of support on 31 July 2025; for ledger-style needs, use an append-only design on a supported database.)

#### Scenario 13: Friend-of-a-friend queries

**Situation:** A social platform needs millisecond reads over highly connected profile relationships with frequent graph traversals. Which database?

| Approach | Assessment |
| --- | --- |
| Aurora PostgreSQL with relational tables | Deep traversals need costly recursive joins. |
| DynamoDB with single-table design | Key-value access; not built for multi-hop traversal. |
| **Amazon Neptune** | **Recommended.** Managed graph database built for traversals (property graph and RDF). |
| S3 with Athena over CSV | Analytical, not an operational graph store. |

---

## 3. Continuously improving existing solutions

Performance, operational excellence, reliability and security improvements to systems already running.

### 3.1 Performance and compute tuning

**Scope:** rightsizing EC2 with AWS Compute Optimizer; Auto Scaling policies (target tracking, step, predictive); EC2 placement groups (cluster, spread, partition).

#### Scenario 14: Low-latency HPC cluster

**Situation:** A financial HPC cluster on EC2 needs very low latency and high throughput between instances running parallel models. Which feature?

| Approach | Assessment |
| --- | --- |
| Auto Scaling with simple scaling across AZs | Does not affect inter-instance latency. |
| **Cluster placement group in one Availability Zone** | **Recommended.** Packs instances close together for low latency and high throughput. |
| Spread placement group | Reduces correlated hardware failure; does not improve latency. |
| Spot Instances with hibernation | Pricing choice; unrelated to network performance. |

### 3.2 Security hardening and automated remediation

**Scope:** AWS Config rules for compliance, Systems Manager Automation for remediation, secrets in AWS Secrets Manager and Parameter Store.

#### Scenario 15: Auto-remediate public S3 buckets

**Situation:** Audit found public-read S3 buckets in several accounts. Future public buckets must be detected and fixed within minutes without manual work. How?

| Approach | Assessment |
| --- | --- |
| Weekly Trusted Advisor emails and manual review | Not automated or timely. |
| **AWS Config managed rule `s3-bucket-public-read-prohibited` with automatic remediation through a Systems Manager Automation runbook or Lambda** | **Recommended.** Continuous evaluation plus automated correction. Pair with account-level S3 Block Public Access and an SCP to prevent it in the first place. |
| IAM policies preventing bucket creation | Blocks legitimate work. |
| Monthly CloudTrail analysis in QuickSight | Reactive and slow. |

### 3.3 Operational excellence and monitoring

**Scope:** centralized observability with Amazon CloudWatch (metrics, logs, alarms, Container Insights), AWS X-Ray for distributed tracing, and Systems Manager runbooks.

#### Scenario 16: Intermittent HTTP 500s across many services

**Situation:** Requests pass through 10 Lambda functions and ECS services. Engineers cannot find where intermittent HTTP 500 errors originate. What helps most?

| Approach | Assessment |
| --- | --- |
| Text searches in CloudWatch Logs | Slow and fragmented across services. |
| **AWS X-Ray (or OpenTelemetry-based tracing) with service maps** | **Recommended.** Traces each request end to end and shows where latency and errors arise. |
| CloudTrail | Records control-plane API calls, not application request flows. |
| GuardDuty | Threat detection, not performance tracing. |

### 3.4 Reliability and fault tolerance

**Scope:** removing single points of failure, self-healing infrastructure, managing service quotas proactively, load shedding and circuit breakers.

#### Scenario 17: Traffic spikes overwhelm the database

**Situation:** A payment service's traffic spikes exhaust database connections and cause cascading failures. Which pattern protects the database?

| Approach | Assessment |
| --- | --- |
| Keep increasing the database instance size | Expensive, with hard limits; does not fix the burst pattern. |
| **SQS queue in front of the processor, consumed at a rate the database can sustain (queue-based load leveling)** | **Recommended.** Absorbs spikes, smooths throughput and prevents cascading failure. Add idempotent consumers and a dead-letter queue. |
| Synchronous blocking calls across all upstream tiers | Makes cascades worse. |
| Disable connection pooling | Increases connection pressure and failures. |

### 3.5 Cost optimization of existing workloads

**Scope:** Reserved Instances, Savings Plans, Spot Instances, S3 lifecycle policies and Intelligent-Tiering, and Cost and Usage Reports.

#### Scenario 18: Steady database fleet plus nightly batch

**Situation:** 100 EC2 instances run a steady production database workload around the clock, plus batch jobs that run 4 hours a night and tolerate interruption. What is most cost-effective?

| Approach | Assessment |
| --- | --- |
| On-Demand for everything | Most expensive for predictable workloads. |
| **Savings Plans or Reserved Instances for the steady workload; Spot Instances for the interruptible batch** | **Recommended.** Commitment discounts (AWS cites up to 72%) for steady use; Spot for interruptible jobs. |
| Dedicated Hosts for everything | Costly unless licensing or isolation requires it. |
| Spot for everything | Interruptions would take down production. |

---

## 4. Migration and modernization

Migration strategies, assessment, data transfer and application modernization.

### 4.1 Migration strategies (the 7 Rs) and assessment

**Scope:** the 7 Rs: rehost, relocate, replatform, refactor (re-architect), repurchase, retire and retain. Portfolio assessment tools include AWS Migration Evaluator; AWS Application Discovery Service and AWS Migration Hub stopped accepting new customers on 7 November 2025 (see [Service availability notes](#service-availability-notes)).

#### Scenario 19: Legacy ASP.NET app with minimal code change

**Situation:** A monolithic ASP.NET application on IIS must move from an on-premises Windows server to AWS with minimal code changes, and without the burden of OS patching. Which strategy?

| Approach | Assessment |
| --- | --- |
| Refactor into microservices on containers | Too much code change for this phase. |
| Rehost on an EC2 Windows instance | Minimal change, but OS patching stays with the team. |
| **Replatform onto Elastic Beanstalk (Windows Server with IIS) or a managed container runtime** | **Recommended.** A few cloud optimizations without changing the core architecture; AWS manages more of the platform. |
| Retire and rebuild as a new app | Contradicts the goal of migrating. |

### 4.2 Large-scale data transfer

**Scope:** choosing transfer mechanisms by data size and available bandwidth: AWS DataSync, AWS Direct Connect, and offline transfer devices.

A quick bandwidth check: 800 TB takes about 740 days at 100 Mbps, about 74 days at 1 Gbps, and about 7.5 days at 10 Gbps (ignoring overhead).

#### Scenario 20: 800 TB over a 100 Mbps link

**Situation:** A research organization must move 800 TB of genomic data to S3. Its internet link is 100 Mbps, so an online upload would take years. What should the architect recommend?

| Approach | Assessment |
| --- | --- |
| DataSync over the existing 100 Mbps link | Correct tool, wrong pipe: about two years of transfer. |
| **Offline transfer devices where available to the account (AWS Snowball Edge Storage Optimized), or temporary high-bandwidth connectivity (Direct Connect at 10 Gbps) with DataSync** | **Recommended.** Bandwidth is the constraint, so either move the data physically or buy bandwidth. Note that Snowball Edge stopped accepting new customers on 7 November 2025, so check availability for your account first. |
| Storage Gateway file gateway | Designed for hybrid file access and backup, not bulk initial migration over a slow link. |
| Upload zip files through the console | Unreliable at this scale. |

### 4.3 Database migration

**Scope:** homogeneous and heterogeneous database migration with minimal downtime using AWS Database Migration Service (DMS) and the AWS Schema Conversion Tool (SCT), or DMS Schema Conversion.

#### Scenario 21: Oracle to Aurora PostgreSQL with near-zero downtime

**Situation:** A 15 TB production Oracle database moves to Aurora PostgreSQL. Oracle must stay the primary write database until a short final cutover. Which toolset?

| Approach | Assessment |
| --- | --- |
| **Schema conversion (SCT or DMS Schema Conversion), then DMS full load plus ongoing change data capture (CDC)** | **Recommended.** Converts the schema, loads existing data, and keeps replicating changes until cutover. |
| Export to CSV with SQL\*Loader and import by script | Long downtime; no change capture. |
| Restore an RMAN backup onto PostgreSQL | Incompatible engines. |
| Triggers and linked servers over VPN | Error-prone and insecure across engines. |

### 4.4 Server migration

**Scope:** AWS Application Migration Service (MGN) for block-level replication and lift-and-shift. (AWS Migration Hub stopped accepting new customers on 7 November 2025; track portfolios in your own tooling or a supported alternative.)

#### Scenario 22: 200 VMware servers without downtime

**Situation:** 200 physical and virtual servers on VMware vSphere must move to EC2 with continuous block-level replication, no disruption to production, and automated launch testing. Which service?

| Approach | Assessment |
| --- | --- |
| **AWS Application Migration Service (MGN)** | **Recommended.** Continuous block-level replication into a staging area, non-disruptive test launches, and controlled cutover. |
| Systems Manager Patch Manager | Patching, not migration. |
| Amazon WorkSpaces | Virtual desktops, not server migration. |
| AWS Backup | Protects resources already in AWS. |

### 4.5 Modernizing legacy applications

**Scope:** breaking monoliths into decoupled, containerized or serverless services using EventBridge, SQS, Lambda and Fargate.

#### Scenario 23: A failure in one module crashes another

**Situation:** A monolithic claims system on one oversized VM is tightly coupled: a failure in document scanning crashes payment approval. How should it be redesigned for resilience and independent scaling?

| Approach | Assessment |
| --- | --- |
| Move to a larger instance | Does not fix coupling or single points of failure. |
| **Split into decoupled services communicating asynchronously through SQS or EventBridge, running on ECS with Fargate** | **Recommended.** Isolates failure domains and scales each service independently. Use the strangler fig pattern to migrate module by module. |
| Put the whole monolith in one container on one instance | Same coupling, new packaging. |
| Move the code into a static S3 bucket | S3 hosts static content, not backend logic. |

---

## Service availability notes

Checked November 2026. Re-check before relying on any of these.

| Service | Status | Affects |
| --- | --- | --- |
| Amazon QLDB | End of support 31 July 2025 | 2.5 |
| AWS Snowball Edge (Storage and Compute Optimized) | No longer open to new customers from 7 November 2025 | 4.2 |
| AWS Migration Hub | No longer open to new customers from 7 November 2025 | 4.1, 4.4 |
| AWS Application Discovery Service | No longer open to new customers from 7 November 2025 | 4.1 |
| Amazon S3 Object Lambda | No longer open to new customers from 7 November 2025 | Knowledge map |
| Amazon Glacier (the original vault service; S3 Glacier storage classes are unaffected) | No longer open to new customers from 7 November 2025 | Knowledge map |
| Amazon Fraud Detector | No longer open to new customers from 7 November 2025 | General |

Sources: [AWS service availability updates (October 2025)](https://aws.amazon.com/about-aws/whats-new/2025/10/aws-service-availability), [Migration Hub availability change](https://docs.aws.amazon.com/migrationhub/latest/ug/migrationhub-availability-change.html), [QLDB end-of-support notice (Classmethod)](https://dev.classmethod.jp/articles/amazon-qldb-eos-announcement/).

## Revision history

| Date | Change |
| --- | --- |
| 2026-11-05 | Converted from Word to Markdown. Restructured as enterprise architecture notes, with each topic's examples written as design scenarios. Corrected the shared-VPC security group reference format, the Aurora Global Database secondary description, and mislabelled option explanations. Updated 6 Rs to 7 Rs. Added service availability notes. |
