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

[Back to contents](README.md)
