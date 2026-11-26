## 2. My day-to-day activities for ECON and DocGen

### Spoken interview answer

“I work from the APAC time zone and support the ECON and DocGen platforms across markets such as Malaysia, Singapore, Korea, Thailand, India, Australia, and other APAC regions. I work as an L3 SRE/InfraOps engineer, so my day combines operational monitoring, incident response, platform engineering, deployment support, automation, and coordination with global teams.”

### How my day starts

“I start by reviewing the operational handover from the previous shift and checking whether there are any unresolved incidents, failed deployments, major alerts, or planned production activities.”

My initial checks generally include:

- ServiceNow incidents and escalations

- Datadog monitors and dashboards

- Application availability and synthetic tests

- AKS node and pod health

- CrashLoopBackOff, Pending, OOMKilled, and failed-probe events

- CPU, memory, node, and PVC utilization

- FluxCD reconciliation and HelmRelease status

Azure service-health issues

Certificate and secret-expiry alerts

PostgreSQL, storage, Key Vault, and networking health

Pending Jira and change-management activities

### During the day

“After the operational health check, I join the daily stand-up and coordinate with L2 operations, developers, QA, architects, release management, business teams, and global colleagues.”

My regular responsibilities include:

### Production and platform support

- Handle L3 incidents and service requests

- Troubleshoot AKS, application, storage, database, DNS, certificate, and networking issues

- Perform RCA and create corrective and preventive actions

- Coordinate Microsoft or vendor escalations

- Support APAC market issues based on priority and impact

### ECON and DocGen application operations

Monitor DocGen, Exstream, OCR, archive, Camunda, RabbitMQ, and related services

Support application onboarding and Helm-chart changes

Troubleshoot RabbitMQ queues, consumers, vhosts, clustering, and PVC issues

Support hotfixes, version upgrades, and application image deployments

Review application logs, pod events, Datadog metrics, and dependency health

Coordinate production validation after deployments

Recent work has included Korean DocGen Stage/Production upgrades, security-vulnerability mitigation, dedicated AKS node-pool enablement, DocGen pod scaling in production, and post-deployment application validation.

### Infrastructure and automation

Write and update Terraform modules

Maintain GitHub Actions infrastructure pipelines

Manage Helm charts and FluxCD deployments

Provision Key Vault, PostgreSQL, Storage Accounts, and tenant resources

Automate operational tasks using PowerShell, Bash, Python, and Azure Automation

Improve alerts, dashboards, runbooks, and operational documentation

Optimize AKS resources, node pools, HPA settings, and cloud costs

### Change and release management

Prepare implementation, validation, rollback, and communication plans

Obtain business and release-management approvals

Execute Stage first and then Production

Monitor during the agreed change window

Coordinate business sanity tests

Communicate start, progress, completion, and issues

Close the change only after validation

### How my day ends

“Before ending my APAC shift, I verify that there are no unresolved P1 or P2 incidents, unstable deployments, or outstanding production risks. I update Jira, ServiceNow, and Confluence, document technical findings, and hand over any open issues to the next regional shift.”

### The handover contains:

- Incident number and severity

- Business impact

- Affected environment and services

- Current technical status

- Troubleshooting completed

- Actions already taken

- Pending next steps

Vendor case number, if applicable

Owner and escalation contacts

Expected next update

### Strong closing line

“My goal at the end of every shift is to leave the platform stable and ensure the next team has a clear, actionable handover with no information gaps.”

## 3. Major initiative: Korea infrastructure and application onboarding

### Interview question

“Tell me about a major project you owned from design through go-live.”

### Complete spoken answer

“One of my major initiatives was onboarding the Korea market onto the ECON and DocGen cloud platform. I supported this initiative from the initial requirements and cloud-assessment stage through infrastructure provisioning, application onboarding, go-live, and hypercare.”

### Phase 1: Requirement gathering and cloud decision

“Initially, the Korea local IT team proposed using AWS and requested a sandbox environment to validate the application and connectivity requirements. We conducted multiple technical discussions covering data residency, Korean regional requirements, connectivity, security, access, governance, application dependencies, supportability, and operational ownership.”

“After comparing AWS and Azure, we selected Azure because the existing ECON and DocGen platforms, operational tooling, application deployment patterns, and enterprise support model were already more aligned with Azure. This reduced operational complexity and allowed us to reuse existing Terraform, GitOps, security, and monitoring standards.”

Be careful with the wording here. Do not say you fully provisioned a production AWS platform if only a sandbox or assessment was performed.

### Phase 2: Architecture and design

“After selecting Azure, I worked with the Korea IT, business, product, security, networking, and platform teams to prepare the high-level and low-level designs.”

The high-level design covered:

- Azure subscription and governance model

- Korea region selection and data residency

- Network topology and enterprise connectivity

- Security controls and access model

- High availability and disaster-recovery considerations

- Platform ownership and support model

- Monitoring and operational readiness

The low-level design covered:

- Stage and Production subscriptions

- Resource groups

- AKS clusters

- PostgreSQL Flexible Servers

- Key Vaults

- Storage Accounts for NFS and SMB requirements

- Azure Container Registries

Application Gateway and ingress

Private endpoints and DNS

Managed identities and RBAC

Application namespaces

Helm and FluxCD structure

The implemented Korea setup included separate Stage and Production Azure environments, AKS clusters, PostgreSQL Flexible Servers, Key Vaults, Storage Accounts, ACRs, and Application Gateways.

### Phase 3: Infrastructure as Code

“I developed and maintained Terraform code for the product-level Azure resources. The infrastructure was stored in Git repositories and deployed through controlled CI/CD pipelines.”

The flow was:

```text
Requirement
↓
Terraform module development
↓
Pull request and code review
↓
Terraform validate and plan
↓
Approval
↓
Terraform apply
↓
Infrastructure validation
```

I ensured:

- Separate environment configuration

- Remote Terraform state

- State locking

- Reusable modules

- Environment-specific service connections

- Approval before apply

- Naming and tagging standards

Secure Key Vault integration

No secrets stored directly in Git

### Phase 4: Non-production implementation

“We first provisioned and validated the non-production environment. This allowed us to test networking, storage mounting, database connectivity, secrets, ingress, DNS, certificates, container images, and application functionality before touching production.”

The non-production validation included:

- AKS connectivity and namespace setup

- PostgreSQL connectivity

- Key Vault secret synchronization

- Storage mounting

- ACR image pulls

- Helm rendering

- FluxCD reconciliation

Application health checks

Ingress and certificate validation

Monitoring and log collection

Business and QA testing

### Phase 5: Production provisioning and onboarding

“Once non-production testing was completed and stakeholders approved the solution, we provisioned the Production environment using the same Terraform and GitOps patterns.”

Applications were deployed through:

```text
Application source
↓
CI pipeline
↓
Container image
↓
Azure Container Registry
↓
Helm chart
↓
FluxCD
↓
AKS
```

“For production, we followed change management, performed deployment during the approved window, monitored the application, and worked with QA and business teams to complete sanity checks.”

The Korean production application installation and go-live support were tracked as specific DocGen and InfraOps activities, followed by vulnerability mitigation and Stage/Production upgrades.

### Phase 6: Go-live and hypercare

“After applications went live, I supported the hypercare period. During hypercare, I closely monitored application availability, AKS workloads, databases, storage, ingress, certificates, logs, and business transactions.”

Hypercare included:

- Daily platform-health checks

- Datadog monitoring

- Incident response

- ServiceNow support

- Application and infrastructure troubleshooting

- Business validation

- Defect and issue tracking

Performance and capacity observation

Runbook and documentation updates

Knowledge transfer to operations

The Korea environment was supported through onboarding, go-live, and hypercare as part of the APAC operating model.

### Project outcome

“The outcome was that the Korea Stage and Production environments were provisioned using repeatable Infrastructure as Code, the DocGen applications were onboarded to AKS through Helm and FluxCD, production readiness was validated, and the platform successfully transitioned into live operations with hypercare support.”

### Strong closing line

“This initiative is important to me because I was involved across the entire lifecycle: cloud assessment, architecture, infrastructure provisioning, application onboarding, production release, hypercare, and operational handover. It demonstrates that I can take ownership beyond individual technical tasks and deliver a complete platform outcome.”

## Short versions for quick interview answers

### Three production incidents

“Three major incidents I handled were an Azure Storage Account connectivity issue that caused pod restarts, a production certificate expiry that affected application access, and an eKYC deployment failure caused by missing Key Vault secret resources. In each case, I assessed the business impact, reviewed Datadog and Kubernetes evidence, isolated the affected dependency, restored the service, validated application functionality, documented the RCA, and created preventive monitoring or deployment controls.”

### Day-to-day responsibilities

“I work in the APAC time zone as an L3 SRE/InfraOps engineer supporting ECON and DocGen. I start with shift handover, ServiceNow incidents, Datadog alerts, AKS health, FluxCD status, and planned changes. During the day, I handle incidents, deployments, Terraform and GitOps changes, monitoring, automation, security remediation, and stakeholder coordination. I end by validating platform stability, updating Jira and Confluence, and handing over any open issues to the next shift.”

### Korea initiative

“I supported the Korea onboarding from cloud assessment through go-live. AWS was initially proposed by the local IT team, but after technical and operational discussions, Azure was selected. I contributed to HLD and LLD, provisioned Stage and Production product infrastructure using Terraform, onboarded DocGen applications to AKS through Helm and FluxCD, supported production deployment and validation, and provided hypercare before operational handover.”

## Main writing improvements

I structured your experience using Situation, Task, Action, Resolution, RCA, and Prevention, separated immediate recovery from permanent corrective actions, and clarified your ownership without overstating unsupported details. The answers now show both technical depth and end-to-end operational responsibility.
