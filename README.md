For this role, I would avoid too many “What is EKS?” or “What is Terraform?” questions. For a **Sr Cloud Engineer / Architect with 10+ years expected experience**, scenario questions should force the candidate to explain **design choices, troubleshooting sequence, tradeoffs, security, cost, and operational impact**.

Here is a strong panel-ready set you can use.

### 1. Production AWS outage — architecture + troubleshooting

**Scenario:**
A customer-facing publishing application suddenly becomes very slow. It runs behind an ALB, with services in EKS and Aurora as the database. CPU on the application pods looks normal, but users are reporting timeouts.

**Question:**
“Walk us through how you would troubleshoot this incident from the AWS infrastructure side. What would you check first, and how would you narrow down whether the issue is networking, Kubernetes, load balancer, database, or application related?”

**Good candidate should mention:** CloudWatch/Container Insights, ALB target health/latency/5xx metrics, EKS pod/service health, ingress, node resource pressure, security groups/NACLs, DNS, Aurora connections/CPU/locks/latency, dependency failures, logs/traces, recent deployments, and rollback/mitigation before deep root-cause analysis.

**Follow-up:**
“The incident started immediately after an infrastructure deployment. How would that change your investigation?”

---

### 2. EKS pods cannot access the internet

**Scenario:**
“An EKS application running in private subnets can communicate internally, but pods suddenly cannot reach an external SaaS API.”

**Question:**
“What would you investigate?”

Listen for understanding of:

* Private/public subnet architecture
* Route tables
* NAT Gateway
* Internet Gateway
* Security groups/NACLs
* Kubernetes network policies
* CoreDNS/DNS resolution
* VPC CNI
* Proxy/firewall restrictions
* NAT port exhaustion or gateway failure

**Strong follow-up:**
“How would you design this differently if we wanted to minimize NAT Gateway cost?”

---

### 3. EKS cluster design from scratch

**Scenario:**
“We're launching a new publication platform. Dev, test, and production will run containerized workloads in AWS.”

**Question:**
“Design the EKS architecture. Explain VPC layout, public/private subnets, node groups, ingress, IAM, secrets, logging, autoscaling, deployment strategy, and how you would separate environments.”

A senior candidate should naturally discuss things such as multi-AZ architecture, managed node groups/Karpenter, IRSA or EKS Pod Identity, ALB controller, GitOps/ArgoCD, autoscaling, secrets management, network policies, observability and environment/account isolation.

---

### 4. EKS upgrade

**Scenario:**
“Your production EKS cluster is several Kubernetes versions behind and AWS support for the current version is approaching its end.”

**Question:**
“How would you upgrade it with minimal impact to production?”

Look for a sequence such as:

control plane compatibility → add-on compatibility → API deprecations → testing in lower environments → node group upgrade/replacement → PodDisruptionBudgets → rolling workload migration → validation → rollback planning.

**Red flag:**
“I'd just upgrade the cluster through the AWS console.”

---

### 5. Terraform production change

**Scenario:**
“A developer modifies Terraform to change an existing production security group. The Terraform plan unexpectedly shows that several resources will be destroyed and recreated.”

**Question:**
“What do you do?”

Look for:

* Stop the deployment
* Understand dependency graph
* Examine Terraform plan
* Check provider/resource changes
* Terraform state
* `terraform state` / imports where appropriate
* lifecycle rules
* immutable resource behavior
* blast radius
* peer review
* backup/recovery
* applying targeted changes only with caution

A strong candidate should say that **Terraform plan output must be treated as a deployment artifact requiring review**, particularly for production.

---

### 6. Terraform state corruption / drift

**Scenario:**
“Someone manually changed several AWS resources through the console. Terraform no longer matches reality.”

**Question:**
“How would you identify and correct the drift without accidentally destroying production resources?”

Follow-up:

“How would you prevent engineers from continuously making manual changes outside Terraform?”

This tests IaC governance rather than just Terraform syntax.

---

### 7. Infrastructure CI/CD

**Scenario:**
“We want Terraform changes stored in GitHub and deployed automatically through CI/CD.”

**Question:**
“Design the pipeline from developer commit through production deployment.”

Look for:

PR → lint/format → validation → security scanning → Terraform plan → review/approval → protected branches/environments → temporary AWS credentials/OIDC → apply → logging/audit → rollback strategy.

Because the JD specifically mentions GitHub and IaC pipelines, ask:

**“Would you store AWS access keys in GitHub Secrets?”**

Strong answer: ideally **no long-lived AWS credentials**; use GitHub OIDC federation with short-lived role credentials.

---

### 8. AWS IAM architecture

**Scenario:**
“We have developers, CloudOps engineers, application teams and contractors. Everyone currently has broad IAM permissions.”

**Question:**
“How would you redesign the access model?”

Expect discussion around:

IAM Identity Center/SSO, groups/permission sets, least privilege, temporary credentials, role assumption, account separation, permission boundaries, SCPs, CloudTrail auditing, break-glass access and periodic access reviews.

---

### 9. SCP versus IAM policy

Give them a real situation:

> “Management says developers must never be able to disable CloudTrail or GuardDuty, even if an administrator accidentally grants them broad permissions.”

Ask:

**“Would you solve this with IAM policies, permission boundaries or an SCP? Explain why.”**

A senior architect should understand that SCPs provide **organization/account-level maximum permission guardrails**, while IAM policies actually grant permissions.

---

### 10. Cross-account access

**Scenario:**
“An application running in AWS Account A needs to read files from an S3 bucket in Account B. We don't want to create IAM users or store access keys.”

**Question:**
“How would you implement this?”

Strong candidate should describe IAM roles, trust policy, resource policy where appropriate, `sts:AssumeRole`, least privilege and possibly external IDs depending on the trust relationship.

Follow-up:

“How would this change if the application was running inside EKS?”

You are looking for IRSA/EKS Pod Identity → IAM role → cross-account role.

---

### 11. Secrets exposed in GitHub

**Scenario:**
“A developer accidentally commits an AWS access key and a database password to a GitHub repository.”

**Question:**
“You're the senior cloud engineer responding to this. What do you do?”

Good sequence:

Immediately revoke/rotate → determine exposure → review CloudTrail/access logs → rotate dependent credentials → Secrets Manager/SSM migration → Git history remediation where appropriate → scanning → preventative controls.

Important senior-level signal: **removing the secret from Git is not enough.**

---

### 12. AWS bill suddenly increases

This should definitely be part of the interview because cost optimization is unusually prominent in this JD.

**Scenario:**
“Our AWS bill increased by approximately 30% compared with the previous month. Nobody knows why.”

**Question:**
“How would you investigate?”

Look for:

Cost Explorer → cost by account/service/region/tag → CUR → usage type → recent infrastructure changes → EC2/RDS/EKS/data transfer/NAT/S3 → anomalies → budgets → tagging → CloudTrail/configuration changes.

Then ask:

**“Suppose you determine that the biggest contributors are EKS, RDS and NAT Gateway. What optimization opportunities would you evaluate?”**

---

### 13. RDS cost optimization

**Scenario:**
“We have multiple non-production RDS databases running 24x7, although teams generally use them only during US business hours.”

**Question:**
“What would you recommend?”

A strong candidate may discuss scheduling, right-sizing, serverless options depending on workload, storage optimization, reserved commitments for predictable production usage, CloudWatch utilization analysis and business requirements before shutting things down.

Follow-up:

“How would you automate stopping and starting those databases?”

EventBridge Scheduler + Lambda is one reasonable implementation.

---

### 14. EC2 right-sizing

**Scenario:**
“A team says they need `m6i.4xlarge` instances because that's what they've always used. CloudWatch shows average CPU around 8%.”

**Question:**
“Would you resize them immediately?”

Good senior answer should be **not based on CPU alone**. Examine memory, network, disk IOPS, peaks, application characteristics, availability requirements and historical utilization; potentially use Compute Optimizer.

This separates experienced engineers from people who memorize cost recommendations.

---

### 15. S3 publishing content architecture

**Scenario:**
“We store millions of publication-related objects in S3. Older content is accessed infrequently but must remain available, and losing the data would be unacceptable.”

**Question:**
“How would you design lifecycle, protection and recovery?”

Listen for:

Versioning, lifecycle policies, Standard-IA/Intelligent-Tiering/Glacier where appropriate, replication, Object Lock depending on requirements, encryption, bucket policies, backup considerations and restore testing.

---

### 16. S3 cross-region DR

**Scenario:**
“A business unit requires critical S3 data to survive a regional outage.”

Ask:

**“How would you implement it, and what are the limitations of S3 replication that the business should understand?”**

Good candidates should understand replication configuration, versioning, IAM, existing-object replication considerations, replication lag and that replication is not automatically the same thing as backup.

---

### 17. RDS/Aurora disaster recovery

**Scenario:**
“The primary production Aurora database becomes unavailable during a regional AWS outage.”

**Question:**
“What DR architecture would you recommend, and how would you determine RTO and RPO?”

Excellent candidates first ask:

**“What are the business RTO and RPO?”**

Then discuss Multi-AZ versus regional DR, Aurora Global Database/cross-region replicas, DNS/application failover, backups and actual restore/failover testing.

---

### 18. Backup exists—but can you restore it?

Ask this exactly:

> “A team tells you that they're compliant because automated RDS snapshots are enabled. Are you satisfied?”

Follow with:

**“How would you prove that the backup strategy actually works?”**

Look for scheduled restore testing, validation, integrity checks, RTO measurement, documented DR runbooks, ownership and evidence.

That is a very good architect-level question.

---

### 19. Linux troubleshooting

**Scenario:**
“An EC2 Linux server suddenly becomes extremely slow. CPU appears normal.”

Ask:

**“Walk me through the Linux commands and investigation you would perform.”**

Experienced candidates may mention:

`top`, `htop`, `free`, `vmstat`, `iostat`, `df`, `du`, `ps`, `ss`, `netstat`, `dmesg`, `journalctl`, `systemctl`, `/var/log`, inode usage, memory swapping, disk latency and network connections.

Don't focus on whether they remember every exact command; focus on their troubleshooting methodology.

---

### 20. Serverless architecture

**Scenario:**
“Every night we receive files in S3. Each file must be validated, metadata extracted and results stored in a database.”

Ask:

**“Would you use EC2, EKS or serverless? Design the solution.”**

A good solution might involve:

S3 event → EventBridge/SQS → Lambda → DynamoDB/RDS → DLQ → CloudWatch.

Then challenge them:

**“What happens when 50,000 files arrive simultaneously?”**

You want discussion of concurrency, throttling, queue buffering, retries, idempotency and back-pressure.

---

### 21. Application team versus infrastructure team

This tests the soft skills from the JD.

**Scenario:**
“An application team says, ‘CloudOps is blocking us. Just give us AdministratorAccess so we can finish our project.’ Their release is tomorrow.”

Ask:

**“How would you handle this situation?”**

A senior candidate should balance delivery with security rather than simply saying **no**.

Look for understanding requirements, temporary/scoped access, identifying exact APIs required, least privilege, collaboration and avoiding production shortcuts.

---

### 22. Architecture disagreement

**Scenario:**
“Two senior engineers disagree strongly. One wants EKS; another wants Lambda/ECS. You're responsible for the final architecture recommendation.”

Ask:

**“How would you make the decision?”**

Strong answers should move away from personal preference and discuss:

workload characteristics, operational overhead, scalability, portability, latency, skills, cost, security, maintainability and measurable decision criteria.

---

### 23. Handling a bad production deployment

**Scenario:**
“GitHub Actions and ArgoCD deploy a new application release to EKS. Within five minutes, error rates increase sharply.”

Ask:

**“What should happen operationally?”**

Look for:

Observability → health checks → rollback → ArgoCD rollback/Git revert → blue/green or canary strategies → automated thresholds → incident communications → root-cause review.

---

### 24. Multi-account AWS architecture

This is a good **whiteboard question**.

> “You are joining an organization that currently runs development, test, production, security and shared services in one AWS account. Design a better AWS Organizations structure.”

Ask them to draw:

**Organizations → OUs → accounts → SCPs → centralized logging/security → networking → shared services → application accounts.**

Then ask:

**“Where would you put CloudTrail logs so application administrators cannot delete them?”**

---

### 25. Final senior-level scenario

I'd make this the final question:

> “You join ACS and inherit approximately 100 AWS workloads spread across several accounts. Infrastructure was created over many years. Some resources use Terraform, some CloudFormation and some were manually created. Costs are increasing, IAM permissions are inconsistent, several EKS clusters need upgrades and nobody is fully confident in the DR documentation. You cannot fix everything immediately.”

Ask:

**“What would your first 30, 60 and 90 days look like?”**

This one question exposes a tremendous amount.

A genuine senior architect will usually prioritize something like **discovery and inventory → risk/security → production stability → cost visibility → IaC/governance → modernization**, rather than immediately proposing migrations.

---

For your panel, I would especially use **questions 1, 3, 5, 8, 12, 17, 21 and 25**. Those eight alone can tell you whether someone is actually operating at **Senior Cloud Engineer / Architect level** rather than someone who mainly knows AWS certification concepts.

A useful interviewing technique is to keep asking **“Why?”**, **“What could go wrong?”**, and **“How would you prove it works?”** after their initial answer. Senior candidates usually become stronger as the scenario gets deeper; candidates relying mainly on memorized AWS terminology tend to struggle once you introduce failure conditions and tradeoffs.
