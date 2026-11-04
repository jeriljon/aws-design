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

[Back to contents](README.md) · [Next: Domain 5](domain-5-operational-excellence.md)
