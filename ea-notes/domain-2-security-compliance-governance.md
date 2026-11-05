# Domain 2: Security, Compliance, and Governance (25%)

Tasks 2.1 to 2.5: 
identity, encryption and key management, 
multi-account governance, compliance, and 
responsible AI controls.

## Task 2.1: Design identity security architectures

This task tests whether you can pick the right identity mechanism for each kind of principal: 
workforce humans, customer users, AWS workloads, workloads outside AWS, and cross-account callers. 
Then it tests whether you can bound what each one can do with layered policies. 
The recurring decision: which mechanism issues short-lived credentials here, and 
which policy layer stops the blast radius when something goes wrong.

### Skill 2.1.1: Identity federation with standard protocols

**Key concepts**

- Workforce access to many accounts: 
Use AWS IAM Identity Center with an organization instance, connected to the corporate IdP (Entra ID, Okta, Ping) over SAML 2.0 for sign-in and SCIM for automatic user and group provisioning. 
SAML carries the authentication; 
SCIM keeps users and groups in sync. 
They solve different problems and you usually need both.

- Identity Center does not create users at first SAML sign-in. 
Without SCIM, someone must create users and groups in Identity Center before sign-in works. 
A scenario where "new hires cannot sign in until an admin creates them" is fixed by enabling SCIM, not by changing SAML settings.

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

- Amazon Cognito user pools authenticate application users and issue OIDC tokens (ID, access, refresh). Identity pools exchange a token (from a user pool, a social IdP, SAML, or an OIDC IdP) for temporary AWS credentials through an IAM role. 
Rule: need JWTs for your API, use a user pool. 
Need the browser or mobile app to call AWS services directly (S3 upload, DynamoDB), add an identity pool.

- Identity pools can choose the IAM role from token claims (role mapping rules), or use principal tags from claims for ABAC, for example to restrict S3 access to a prefix equal to the user's `sub` using `${cognito-identity.amazonaws.com:sub}`.

- OAuth 2.0 flows: authorization code with PKCE for browser and mobile apps; client credentials for machine-to-machine. 
Cognito supports client credentials with resource servers and custom scopes. 
The implicit grant is legacy and should not be the answer for a new app.

- Cognito user pools have feature plans (Lite, Essentials, Plus). 
Threat protection features such as adaptive authentication and compromised-credential detection sit in the Plus plan. 
If a scenario demands risk-based MFA for customer sign-ins with minimum custom code, that is the Cognito feature, not a Lambda trigger you write.

- API authorization choices: 
API Gateway Cognito authorizer (validates user pool tokens, scope checks), JWT authorizer on HTTP APIs (any OIDC issuer), Lambda authorizer (custom logic, cache the result), IAM authorization (SigV4, for AWS callers). 
For fine-grained, policy-based authorization inside the application, Amazon Verified Permissions evaluates Cedar policies against principals and resources.

- ALB can authenticate users itself (OIDC or Cognito) before forwarding, which removes auth code from legacy backends that cannot be changed.

- Amazon EKS Pod Identity gives pods AWS credentials. The role trusts the single service principal `pods.eks.amazonaws.com`, so the same role works across clusters without per-cluster OIDC providers. An association in the EKS API maps a namespace and service account to a role, and the EKS Pod Identity Agent (a DaemonSet add-on) serves credentials on each node. Credentials carry session tags such as cluster and namespace, which you can use for ABAC.

- IRSA (IAM roles for service accounts) uses a per-cluster IAM OIDC provider and a trust policy that names that provider and the service account. 
Pod Identity works only on Linux EC2 worker nodes. 
It is not supported on Fargate, Windows nodes, Outposts, EKS Anywhere or self-managed Kubernetes on EC2. For those, IRSA (or another mechanism) is still the answer. Both can run in the same cluster during migration.

##### Note: 
IAM Roles for Service Accounts (IRSA) is an Amazon EKS feature that securely maps Kubernetes service accounts to AWS IAM roles, allowing Pod-level workloads to access AWS resources without static credentials.

Why IRSA is Crucial

Historically, pods requiring AWS access had to use long-lived access keys (stored in Kubernetes secrets) or leverage the overarching IAM role of the EC2 worker node.
• The Risk: Sharing an IAM role across a whole EC2 node grants every pod on that node the exact same broad permissions, violating the principle of least privilege.
• The IRSA Solution: It creates a dedicated security boundary per workload. Workload A can be limited to reading a specific S3 bucket, while Workload B on the same node can only access a DynamoDB table.


How IRSA Works Under the Hood

IRSA relies on workload identity federation through OpenID Connect (OIDC) and the AWS Security Token Service (STS):
1. OIDC Establishment: You associate an IAM OIDC Identity Provider with your EKS cluster.
2. The Link: A Kubernetes Service Account is configured with a specific annotation linking it to an IAM Role (eks.amazonaws.com/role-arn). The IAM Role's trust policy explicitly trusts the EKS cluster's OIDC issuer URL for that specific namespace and service account name.
3. Token Exchange: When a Pod launches, EKS automatically injects a projected OIDC token and two environment variables (AWS_ROLE_ARN and AWS_WEB_IDENTITY_TOKEN_FILE) into the container.
4. AWS API Call: The application's AWS SDK reads these variables and calls sts:AssumeRoleWithWebIdentity. AWS STS verifies the token against the cluster's OIDC endpoint and returns short-lived, auto-rotating temporary credentials.

**Exam traps**

- Using node instance profile permissions for all pods. Every pod on the node inherits them; it fails least privilege. Pod Identity or IRSA, plus blocking pod access to IMDS, is the fix.
- Choosing Pod Identity for pods on Fargate. It is not supported there; use IRSA.
- Using an identity pool when the backend only needs to validate a JWT. The identity pool is for AWS credentials, not API authentication.
- Embedding long-term access keys in a mobile app to upload to S3. Use an identity pool, or presigned URLs generated by a backend.
- Choosing a Lambda authorizer when the token is a standard Cognito or OIDC JWT and only scope checks are needed. The managed Cognito or JWT authorizer has less code and no extra Lambda cost.

**Hands-on**

1. **User pool plus identity pool for direct S3 upload.** 
Create a Cognito user pool with a hosted or managed login app client using the authorization code grant with PKCE. Create an identity pool that trusts the user pool and an authenticated role whose policy allows `s3:PutObject` only on `arn:aws:s3:::bucket/private/${cognito-identity.amazonaws.com:sub}/*`. Sign in with two test users from a small web page or the AWS CLI and confirm each can only write under its own prefix. *Cost/cleanup:* within free usage for a handful of users. Delete the identity pool, user pool and bucket.
2. **EKS Pod Identity versus IRSA (cost warning).** Create a small EKS cluster with one Linux managed node group and install the EKS Pod Identity Agent add-on. Create a role trusting `pods.eks.amazonaws.com` with read access to one S3 bucket, associate it with a service account, and run a pod that lists the bucket. Then run a pod with a different service account and confirm it is denied. If cost is a concern, do the same lab in AWS Workshop Studio (EKS workshops) instead. *Cost/cleanup:* the EKS control plane bills per hour and the nodes bill as EC2; finish in one sitting and delete the cluster, node group and any load balancers.

### Skill 2.1.3: Least-privilege access controls

**Key concepts**

- Policy evaluation order for a request: 
Explicit deny anywhere wins. 
Then the request must be allowed by every applicable guardrail (SCPs on the principal's account, RCPs on the resource's account, permissions boundary, session policy) and granted by an identity-based or resource-based policy. 
Guardrails never grant.

- Cross-account access: 
Either a role in the target account that the caller assumes (role trust policy plus caller permission for `sts:AssumeRole`), or a resource-based policy that names the external principal. 
With a resource-based policy, the caller keeps its own identity and permissions; 
with a role, it gives them up for the role's. 
This matters for S3 object ownership and for services that need the caller's own permissions at the same time.

- External ID: 
A condition `sts:ExternalId` in the role trust policy. 
Its purpose is the confused deputy problem when a third party (a SaaS vendor) assumes roles in many customer accounts. 
It is not a secret password and not needed for your own accounts. 
For AWS service principals, the equivalent protection is `aws:SourceArn` and `aws:SourceAccount` conditions.

- Permissions boundaries: 
A managed policy that caps an IAM user or role. 
Classic use is delegated administration: 
Developers may create roles only if they attach a specific boundary (`iam:PermissionsBoundary` condition), so they cannot escalate beyond it.

- Session policies: 
Passed on `AssumeRole` or `GetFederationToken`; they further restrict that one session. 
Use them to vend scoped-down credentials (for example per-tenant) from a broader role without creating a role per tenant.

- ABAC: 
compare `aws:PrincipalTag/x` with `aws:ResourceTag/x` (or `aws:RequestTag` on create). 
It scales with fewer policies, but you must protect the tags: deny `iam:TagRole`, resource tag changes, and require tags on create, or users can tag themselves into access.
- Role chaining (assuming a role from a role session) caps the session at one hour, regardless of the role's maximum session duration. A scenario that needs a 4-hour chained session is a trap.
- `aws:PrincipalOrgID` and `aws:PrincipalOrgPaths` in resource policies are the clean way to allow "any principal in our organization or OU" without listing account IDs.

##### Note: 
Attribute-Based Access Control (ABAC) in AWS is an authorization strategy that defines permissions based on attributes—which AWS implements as tags or session attributes.
Instead of hardcoding individual user or resource ARNs into IAM policies, ABAC grants access dynamically when a principal's tag matches the target resource's tag.

**Exam traps**

- Using a permissions boundary or SCP to grant access. 
Neither grants; you still need an allow in an identity or resource policy.

- Using an external ID between your own accounts and calling it the security control. 
Scenarios with internal accounts want `aws:PrincipalOrgID` or specific role ARNs.

- ABAC without tag governance. 
The plausible option tags resources but lets the same users change tags.

- Long-term access keys for a cross-account batch job. 
Use role assumption.

**Hands-on**

1. **Delegated role creation with a boundary.** 
In a test account, create a boundary policy that allows only S3 and CloudWatch Logs actions. 
Give a "developer" role `iam:CreateRole` and `iam:PutRolePolicy` with a condition that `iam:PermissionsBoundary` must equal that boundary ARN, and deny changing or deleting the boundary. 
As the developer, create a role with `AdministratorAccess` attached and the boundary set, then use the IAM policy simulator to show it can still only do S3 and Logs. *Cost/cleanup:* free. Delete the roles and policies.

2. **ABAC across two projects.** 
Create a role with tag `project=blue` and a policy allowing `ec2:StartInstances` and `ec2:StopInstances` where `aws:ResourceTag/project` equals `${aws:PrincipalTag/project}`. 
Launch two `t3.micro` instances tagged `blue` and `green`, then try to stop each. 
Add a deny on `ec2:CreateTags` and `ec2:DeleteTags` for the `project` key and show the role cannot retag the green instance. 

*Cost/cleanup:* a few cents of EC2; terminate both instances.

### Skill 2.1.4: Data perimeter controls and external access validation

**Key concepts**

- A data perimeter has three conditions: 
only trusted identities, 
accessing trusted resources, from expected networks. 
Implement with SCPs (identity side: which resources and networks your principals may use), 
RCPs (resource side: who may access your resources), and 
VPC endpoint policies (network side: what can pass through the endpoint).

- Key condition keys: 
`aws:PrincipalOrgID`, 
`aws:ResourceOrgID`, 
`aws:SourceVpce`, 
`aws:SourceVpc`, 
`aws:SourceIp`, 
`aws:ViaAWSService`, 
`aws:PrincipalIsAWSService`. 
Perimeter deny statements need exceptions for AWS services acting on your behalf 
(`aws:PrincipalIsAWSService` or `aws:ViaAWSService`), or 
you break CloudTrail delivery, replication and similar flows.

- Resource control policies are an AWS Organizations policy type applied to resources in member accounts. 
They cap the maximum permissions on those resources for any caller, including principals outside the organization. 
They do not grant permissions, do not apply to resources in the management account, do not affect service-linked roles, and do not apply to AWS managed KMS keys. 
Supported services include Amazon S3, AWS STS, AWS KMS, Amazon SQS, AWS Secrets Manager, Amazon DynamoDB, Amazon ECR, Amazon CloudWatch Logs, Amazon Cognito, Amazon OpenSearch Serverless and others. 
EC2, RDS and Lambda are not on the list, so a design that relies on an RCP to protect an RDS snapshot or Lambda function does not work.

- VPC endpoint policies: an S3 gateway endpoint policy that allows only buckets in your organization (`aws:ResourceOrgID`) stops data exfiltration to an attacker's bucket from inside the VPC. 
Pair it with a bucket policy that denies access not from your endpoint (`aws:SourceVpce`).

- IAM Access Analyzer has three analyzer types. 

External access analyzers (free) find resource policies that grant access outside your zone of trust (account or organization) for resources such as S3 buckets, IAM role trust policies, KMS keys, Lambda functions, SQS queues, Secrets Manager secrets, SNS topics, EBS and RDS snapshots, ECR repositories, EFS file systems and DynamoDB tables and streams. 

Internal access analyzers (paid per resource monitored) find which principals inside the organization or account can reach selected critical resources; they cover a smaller set (S3 buckets, S3 directory buckets, RDS DB and cluster snapshots, DynamoDB tables and streams). 

Unused access analyzers (paid per role and user analyzed) find unused roles, unused access keys and passwords, and unused service-level and action-level permissions.

- Access Analyzer also validates policies: 
policy validation (grammar and best-practice warnings), and 
custom policy checks (`CheckNoNewAccess`, `CheckAccessNotGranted`, `CheckNoPublicAccess`) that you can run in a CI/CD pipeline to block changes before deployment. 
It can also generate a least-privilege policy from CloudTrail activity.

- AWS Verified Access provides VPN-less, zero trust access to internal applications. 
Each request is evaluated against Cedar policies using user identity from a trust provider (IAM Identity Center or any OIDC IdP) and device posture from a device trust provider (Jamf, CrowdStrike, JumpCloud). 
It supports HTTP(S) applications and, since early 2025, TCP-based access such as SSH, RDP and databases. 
Endpoint types include load balancer, network interface, network CIDR and RDS. 
Non-HTTP access needs the Verified Access connectivity client on the user's device.

**Exam traps**

- Using an SCP to stop an external account from reading your bucket. 
SCPs only restrict principals in your organization. 
Restricting external callers on your resources is an RCP (or the resource policy itself).

- Using an RCP to protect an EC2 or RDS resource type that RCPs do not support.

- Expecting the management account to be protected by SCPs or RCPs. It is not; keep workloads out of it.

- Choosing Client VPN plus security groups when the requirement is per-request authorization based on identity and device posture without network-level access. That is Verified Access.

- Running an unused access analyzer to find public buckets. Public or cross-account exposure is the external access analyzer; unused access is about over-permissioned identities.

##### Note: 

In AWS Organizations, Service Control Policies (SCPs) and Resource Control Policies (RCPs) are both organization-level guardrails that do not grant permissions, but instead set maximum permission ceilings. 

The core difference lies in who or what they target.'

Service Control Policy (SCP) is Principal Centric while RCP is Resource-centric. 
SCP restricts IAM users, roles, and accounts inside your organization while RCP restricts access to resources inside your organization.

SCP does not block external principals if a resource policy allows them, but RCP Can block external principals (e.g., stopping anyone outside your Org ID from accessing an S3 bucket).

SCP Generally does not apply to AWS service principals, but RCP Applies to service principals in certain cases. 

SCP Service Support broadly applies across most AWS services, but RCP Service Support is limited to specific services supporting resource-based policies (44+ services).

SCP was introduced in 2017 and RCP was introduced recently in 2024. 


**Hands-on**

1. **Access Analyzer findings and policy checks.** 
Create an account-level external access analyzer (free). 
Create an S3 bucket with a bucket policy granting `s3:GetObject` to a random other account ID and watch the finding appear; archive it with a rule, then remove the policy and see it resolve. 
Then run `aws accessanalyzer check-no-public-access` against a bucket policy JSON that allows `"Principal": "*"` and observe the FAIL result you would wire into a pipeline. 
*Cost/cleanup:* external analyzers are free; custom checks have a small per-call charge. 
Delete the bucket and analyzer.

2. **Endpoint policy exfiltration block.** 
In a VPC with a private EC2 instance (use Session Manager over interface endpoints, or a temporary public IP), 
add an S3 gateway endpoint whose policy allows only buckets 
where `aws:ResourceOrgID` equals your organization ID (or, with one account, `aws:ResourceAccount` equals your account). From the instance, copy a file to your own bucket (works) and to a public bucket owned by someone else (denied). 
*Cost/cleanup:* gateway endpoints are free; interface endpoints for Session Manager bill per hour, 
so delete them and terminate the instance afterwards.

### Skill 2.1.5: Workload identities for hybrid and multi-cloud

**Key concepts**

- IAM Roles Anywhere lets workloads outside AWS (on-premises servers, other clouds, edge devices) get temporary AWS credentials using X.509 certificates instead of long-term access keys. 
Building blocks: 
a trust anchor (an AWS Private CA, or your own CA certificate bundle), 
a profile (which roles may be assumed, optional session policy and managed policies, session duration from 15 minutes to 12 hours), and 
the role's trust policy, which trusts `rolesanywhere.amazonaws.com` and can condition on certificate attributes (subject, issuer, SAN).

- The workload uses the credential helper (`aws_signing_helper`) configured as a `credential_process` in the AWS CLI or SDK config. 
Private keys can stay in a TPM or PKCS#11 store.

- Revocation: 
Import a CRL into Roles Anywhere; it does not query OCSP. 
If the scenario requires revoking a compromised server quickly, the design must include a process to update the imported CRL, or disable the profile or trust anchor.

- Decision rule for non-AWS workloads: workload already has an OIDC token from its platform (GitHub, GitLab, another cloud's workload identity, Kubernetes) use `AssumeRoleWithWebIdentity` with an IAM OIDC provider. 
Workload has, or can get, an X.509 certificate from an enterprise PKI, use Roles Anywhere. Workload is a managed server you want to patch and access, use Systems Manager hybrid activations (which also give the instance credentials for SSM).

- IAM Roles Anywhere supports ML-DSA PKIs as trust anchors, which matters for post-quantum planning (see 2.2.4).

- Roles Anywhere is in the list of services that RCPs support, so you can cap which trust anchors and profiles can be used across the organization.

**Exam traps**

- Creating an IAM user with access keys for an on-premises application and rotating keys with a script. 
It works, but fails "no long-term credentials."

- Picking AWS Private CA as mandatory. You can use an existing enterprise CA as the trust anchor; Private CA is optional and has a monthly charge.

- Using IAM Roles Anywhere for human users. It is for workloads; humans use Identity Center.

- Assuming certificate revocation works through OCSP automatically.

**Hands-on**

1. **Roles Anywhere with a self-made CA.** 

Use OpenSSL on your laptop to create a root CA and a client certificate with the correct key usage. 

Create a Roles Anywhere trust anchor from the CA certificate, a role trusting `rolesanywhere.amazonaws.com` with a condition on the certificate subject CN, and a profile that references the role with a one-hour session. 

Configure `credential_process` with `aws_signing_helper` and run `aws s3 ls`. 

*Cost/cleanup:* Roles Anywhere has no charge when you bring your own CA. 
Delete the profile, trust anchor and role.

2. **Revocation test.** 
Issue a second client certificate, generate a CRL that revokes it, and 
import the CRL with `aws rolesanywhere import-crl`. 
Confirm the revoked certificate fails `CreateSession` while the first still works. 
This shows the CRL is a static import you must refresh. 
*Cost/cleanup:* free. Delete the CRL and certificates.

### Practice questions: Task 2.1

**Question 2.1.1** 
A bank has 140 AWS accounts in AWS Organizations and uses Microsoft Entra ID as its workforce IdP. 
Today each account has an IAM SAML identity provider and hand-built roles, and joiners often wait two days for access because an administrator must create matching groups. 
Leavers sometimes keep access until the next manual review. 
The security team wants one place to define access per job function, automatic onboarding and 
offboarding based on Entra ID group membership, and the least operational overhead. 
Which solution meets these requirements?

- A. Keep the per-account IAM SAML providers, deploy the roles with CloudFormation StackSets, and 
map Entra ID groups to roles through SAML attribute assertions.

- B. Enable IAM Identity Center with an organization instance, connect Entra ID over SAML 2.0, enable SCIM provisioning, and assign permission sets to synced groups.

- C. Enable IAM Identity Center with Entra ID over SAML 2.0, create users and groups in the Identity Center directory, and run a nightly Lambda function that compares them with Entra ID.

- D. Create an Amazon Cognito user pool federated with Entra ID over OIDC, and use an identity pool with role mapping rules to issue credentials for each account.

<details>
<summary>Answer and explanation</summary>

**Correct answer: B**

**Why B is correct:** 
IAM Identity Center gives one place to define permission sets and assign them to groups across all accounts in the organization. 
SAML 2.0 handles sign-in with Entra ID, and SCIM pushes user and group membership changes automatically. 
Removing someone from the Entra ID group removes the assignment without any manual work. 
This is the managed path with the least ongoing effort.

**Why the other options are incorrect:**

- **A:** 
StackSets reduce role drift, but every account still needs its own provider and roles. Joiners and leavers still depend on group-to-role mapping in each account, and there is no central assignment view. 
It is more overhead than Identity Center.

- **C:** 
SAML works, but the custom Lambda sync is code you must build and maintain, and 
nightly runs leave a gap for leavers. SCIM is the managed protocol that removes this.

- **D:** 
Cognito is for application end users. 
It is not designed for workforce access to AWS accounts and the console across an organization, and 
Identity pool roles per account would be heavy to manage.

**Key takeaway:** 
SAML signs people in; SCIM keeps users and groups current. 
Workforce access to many accounts means Identity Center with both.

</details>

**Question 2.1.2** 
A payments company runs microservices on Amazon EKS. 
One cluster has Linux managed node groups; 
a second cluster runs a batch namespace on AWS Fargate. 
Each service must have its own AWS permissions, and 
the platform team wants to stop maintaining a separate IAM OIDC provider and trust policy edit for every new cluster. Pods must not be able to use the node's instance profile. 
Which solution meets these requirements?

- A. Use EKS Pod Identity for every workload in both clusters and block pod access to IMDS on all nodes.

- B. Use IRSA for every workload in both clusters and create one IAM role per service per cluster with the cluster's OIDC provider in the trust policy.

- C. Attach a node IAM role per node group with only the permissions needed by the pods scheduled there, and use taints to place each service on its own node group.

- D. Use EKS Pod Identity for workloads on the Linux managed node groups, use IRSA for the Fargate workloads, and block pod access to IMDS on the EC2 nodes.

<details>
<summary>Answer and explanation</summary>

**Correct answer: D**

**Why D is correct:** 
EKS Pod Identity uses a single service principal, so one role can be reused across clusters without adding OIDC providers or editing trust policies per cluster. 
It runs through the Pod Identity Agent on Linux EC2 nodes. 
It is not supported on Fargate, so the Fargate namespace needs IRSA. 
Blocking IMDS on EC2 nodes stops pods from falling back to the node role.

**Why the other options are incorrect:**

- **A:** 
Pod Identity is not supported on Fargate, so the batch namespace would have no credentials.

- **B:** 
IRSA works everywhere but keeps the per-cluster OIDC provider and trust policy maintenance the team wants to remove.

- **C:** Node roles give every pod on the node the same permissions. 
Taints help placement but are not an identity boundary, and this multiplies node groups.

**Key takeaway:** 
Pod Identity for Linux EC2 nodes; IRSA is still required on Fargate, Windows, Outposts and EKS Anywhere.

**Note:**
IRSA stands for IAM Roles for Service Accounts, an Amazon EKS feature that lets individual Kubernetes Pods assume specific AWS IAM roles.

</details>

**Question 2.1.3** 
A fintech SaaS provider needs read access to an S3 bucket and a CloudWatch Logs group in each of 300 customer AWS accounts. 
Customers create a role from a template the provider publishes. 
The provider's platform assumes those roles from a single role in its own account. 
The provider's security review highlights that a malicious customer could enter another customer's role ARN in the onboarding portal. 
The provider also wants the role sessions to be limited to read-only actions even if a customer over-grants the role. Which actions should the provider take? (Select TWO.)

- A. Generate a unique external ID per customer, store it with the customer record, require it in the template's trust policy, and always pass it on `sts:AssumeRole`.
- B. Add `aws:PrincipalOrgID` to the customer role trust policies so only principals in the provider's organization can assume them.
- C. Pass a session policy that allows only the required read actions when the platform calls `sts:AssumeRole`.
- D. Ask each customer to attach a permissions boundary to the provider's role in the provider's account.
- E. Require customers to create an IAM user for the provider with access keys stored in AWS Secrets Manager in the provider's account.

<details>
<summary>Answer and explanation</summary>

**Correct answer: A, C**

**Why A and C are correct:** 
The confused deputy risk is that the provider's own role can be tricked into assuming a different customer's role. 
A per-customer external ID that the provider generates and always sends means a customer cannot make the provider assume someone else's role, because that role expects a different ID. A session policy on `AssumeRole` intersects with the role's permissions, so even an over-granted customer role yields only the read actions the provider passes.

**Why the other options are incorrect:**

- **B:** The provider's role is legitimately in the provider's organization for every customer, so this condition does not distinguish between customers and does not stop the confused deputy.
- **D:** A permissions boundary attaches to a principal in its own account. Customers cannot attach boundaries to the provider's role, and a boundary would not limit what the assumed customer role can do.
- **E:** Long-term keys across 300 accounts add rotation burden and leak risk, and do nothing about scoping beyond what each user policy allows.

**Key takeaway:** External ID solves the third-party confused deputy; session policies scope down a session without new roles.

</details>

**Question 2.1.4** 
A lender's organization has a workload OU containing accounts that store customer documents in S3 buckets, secrets in AWS Secrets Manager, and keys in AWS KMS. 
Auditors require a preventive control so that no principal outside the organization can access these resources, even if a developer writes an overly permissive bucket or key policy. 
The control must cover future accounts in the OU automatically and must not break AWS services that write logs into those buckets. 
Which solution meets these requirements ?

- A. Attach an SCP to the workload OU that denies all S3, Secrets Manager and KMS actions when `aws:PrincipalOrgID` does not equal the organization ID.
- B. Enable an organization-wide IAM Access Analyzer external access analyzer and use an EventBridge rule to invoke a Lambda function that removes any statement granting external access.
- C. Attach an RCP to the workload OU that denies S3, Secrets Manager and KMS actions when `aws:PrincipalOrgID` does not equal the organization ID and `aws:PrincipalIsAWSService` is false.
- D. Deploy an AWS Config conformance pack to the OU with rules that detect public S3 buckets and KMS keys, with automatic remediation through Systems Manager Automation.

<details>
<summary>Answer and explanation</summary>

**Correct answer: C**

**Why C is correct:** 
RCPs set the maximum permissions on resources in member accounts for every caller, including principals outside the organization. S3, Secrets Manager and KMS all support RCPs. 
Attaching to the OU covers new accounts through inheritance. 
The `aws:PrincipalIsAWSService` exception keeps AWS service principals, such as log delivery, working.

**Why the other options are incorrect:**

- **A:** SCPs restrict only principals inside the organization. An external account's principal is not subject to your SCPs, so this does nothing to stop outsiders.
- **B:** This is detective with automated cleanup. There is a window of exposure between the policy change and the Lambda fix, so it is not preventive.
- **D:** Config rules are detective and the remediation runs after the fact. They also do not cover all external-access cases such as cross-account grants to a specific foreign account.

**Key takeaway:** 
SCPs bound your principals; 
RCPs bound your resources against everyone. 
Add the AWS service exception.

</details>

**Question 2.1.5** 
An insurer runs a claims-scoring application on servers in its own data center and on virtual machines in another cloud provider. The application reads from an Amazon S3 bucket and writes to an Amazon SQS queue. 
The company already runs an enterprise PKI that issues X.509 certificates to every server, with private keys held in TPMs. 
Security policy bans long-term AWS access keys. 
Operations wants to be able to cut off a single compromised server. Which solution meets these requirements?

- A. Configure IAM Roles Anywhere with the enterprise CA as a trust anchor, a profile that maps to a scoped IAM role, the credential helper on each server, and a process that imports updated CRLs.
- B. Register each server with AWS Systems Manager using hybrid activations and grant the activation's IAM service role permission to read S3 and write to SQS for the application.
- C. Create an IAM OIDC identity provider for the enterprise PKI and have each server call `AssumeRoleWithWebIdentity` with its certificate.
- D. Issue short-lived certificates from AWS Private CA and use them for mutual TLS to an Amazon API Gateway endpoint that proxies S3 and SQS calls.

<details>
<summary>Answer and explanation</summary>

**Correct answer: A**

**Why A is correct:** 
Roles Anywhere exchanges the existing X.509 certificate for temporary credentials, so there are no long-term keys. 
The enterprise CA can be the trust anchor without buying Private CA, and the credential helper works with keys held in TPM or PKCS#11 stores. 
Importing an updated CRL revokes a single server's certificate.

**Why the other options are incorrect:**

- **B:** Hybrid activations give credentials for the SSM Agent to manage the server. Using that role for the application mixes management and application identity and is not the intended pattern; it also does not reuse the PKI.
- **C:** `AssumeRoleWithWebIdentity` takes a signed OIDC token from an OIDC issuer, not an X.509 certificate. A PKI is not an OIDC provider.
- **D:** This builds a custom proxy and replaces the existing PKI. It adds latency and operational work and still needs the proxy to hold AWS permissions.

**Key takeaway:** 
Certificates outside AWS map to IAM Roles Anywhere; 
OIDC tokens map to web identity federation.

</details>

## Task 2.2: Design solutions to implement encryption and key management strategies

This task tests whether you can match a key custody requirement to the right AWS option (AWS owned key, AWS managed key, customer managed key, imported material, CloudHSM-backed custom key store, external key store) and handle certificates for TLS without manual renewals. 
The recurring decision: who must control the key material, and what is the least operational way to prove it.

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

1. **Custom rotation period.** 
Create a symmetric customer managed key, then run `aws kms enable-key-rotation --rotation-period-in-days 180` and `aws kms get-key-rotation-status` to see the next rotation date. 
Run `aws kms rotate-key-on-demand` and then `aws kms list-key-rotations` to see the new material entry. 
Encrypt a string before and after rotation and decrypt both with the same key ID. 
*Cost/cleanup:* a customer managed key has a monthly charge, and rotated material versions can add to it; schedule deletion with the 7-day minimum.

2. **Cross-account snapshot sharing.** 
Create an encrypted EBS snapshot with an AWS managed key and try to share it with a second account; observe it cannot be used there. 
Copy the snapshot re-encrypting with a customer managed key, add the second account to the key policy, share again, and copy it in the second account. 
*Cost/cleanup:* a 1 GiB volume and snapshot cost cents; delete snapshots, volumes and schedule key deletion.

### Skill 2.2.2: Encryption in transit with certificate management

**Key concepts**

- Standard (non-exportable) ACM public certificates are free when used with integrated services such as ELB, CloudFront and API Gateway, and ACM renews them automatically when DNS validation stays in place. 
DNS validation is the answer for "no manual renewal"; email validation needs a person to click.

- Classic ACM public certificates cannot be exported. Since mid-2025, ACM can issue exportable public certificates if you choose the export option at request time. You get the private key to install on EC2, containers or on-premises. 
These carry a per-certificate charge and certificates issued before that launch cannot be exported. 
If a scenario needs a public certificate on an EC2 web server or an on-premises appliance with AWS managing issuance, exportable ACM certificates are now a valid answer.

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

[Back to contents](README.md) · [Next: Domain 3](domain-3-cost-optimization.md)
