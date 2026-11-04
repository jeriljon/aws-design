# AWS Certified Solutions Architect - Professional (SAP-C03) Study Notes

Study notes for the SAP-C03 exam, one file per domain (this file is the index), organised by the official exam guide: every domain, task and skill, with key concepts, exam traps, hands-on exercises and 5 exam-style questions per task (105 in total) with explanations of every option.

> **Status:** draft, written 5 November 2026 against the SAP-C03 exam guide. SAP-C03 replaces SAP-C02 on 17 November 2026. AWS features change often, so check the linked documentation before relying on a detail, and log corrections in the [revision history](#revision-history).

## Contents

- [Exam at a glance](#exam-at-a-glance)
- [What changed from SAP-C02](#what-changed-from-sap-c02)
- [How to use these notes](#how-to-use-these-notes)
- [How to answer professional-level questions](#how-to-answer-professional-level-questions)
- [Domain 1: Architecture Design and Implementation Built for the Cloud (25%)](domain-1-architecture-design.md)
- [Domain 2: Security, Compliance, and Governance (25%)](domain-2-security-compliance-governance.md)
- [Domain 3: Cost-Optimized Architecture Design (15%)](domain-3-cost-optimization.md)
- [Domain 4: Resilience, Migration, and Business Continuity (15%)](domain-4-resilience-migration-continuity.md)
- [Domain 5: Operational Excellence and Automation (20%)](domain-5-operational-excellence.md)
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
