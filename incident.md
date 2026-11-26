# SRE / InfraOps Interview Preparation

> Interview-ready speaking guide covering production incidents, day-to-day responsibilities, and Korea platform onboarding.

Use this as a speaking guide rather than memorizing every sentence.

Below is a polished, interview-ready explanation based on the incidents, responsibilities, and Korea onboarding work you previously shared. Use it as a speaking guide rather than memorizing every sentence.

## 1. Three major production incidents I handled

### How to introduce this section

“In my current role, I work as an L3 SRE/InfraOps engineer supporting the ECON and DocGen platforms across APAC markets. I handle incidents involving AKS, application services, storage, networking, certificates, secrets, databases, and deployment components. I would like to explain three significant production incidents that demonstrate my troubleshooting and RCA experience.”

### Incident 1: Storage account connectivity caused application pod restarts

### Situation

“One of the major incidents I handled was in the West Europe production environment. Several application pods started restarting because they were unable to connect to an Azure Storage Account used by the application through persistent storage.”

The incident occurred around 6 PM CET. The L2 operations team detected the service degradation through monitoring, created a production incident, and escalated it to the L3 InfraOps team.

### Task

“I took ownership of the infrastructure investigation. My priority was to determine whether the failure was caused by Kubernetes, storage configuration, DNS, firewall rules, private networking, or an Azure platform issue.”

### Actions

“I first identified the affected environment, namespace, deployments, and restarting pods. I reviewed Datadog metrics, Kubernetes events, current and previous container logs, pod restart reasons, and PVC/PV status.”

Then I checked:

- Storage Account and file-share availability

- PVC and PV mount events

- DNS resolution using nslookup and dig

- HTTPS and storage endpoint connectivity using curl

- Port 443 connectivity

- VNet and subnet configuration

- Firewall and allowlist rules

Private endpoint and private DNS resolution

Network and firewall logs

Recent application or infrastructure changes

“The application configuration, Kubernetes resources, firewall rules, and network allowlists were correct, but connectivity to the storage endpoint was failing. After collecting all the timestamps, resource details, logs, and test results, I raised a P1 support case with Microsoft.”

### Resolution

“Microsoft confirmed that the problem was related to an Azure backend connectivity issue. Once the service was restored from their side, the storage connection recovered and the Kubernetes pods became stable.”

### Validation

I validated:

- Storage endpoint resolution and connectivity

- PVC mounts

- Pod restart count

- Application logs

- Readiness and liveness status

- Application health endpoints

- Functional checks with the application team

### RCA and preventive actions

“For the RCA, I documented the incident timeline, business impact, investigation steps, Microsoft’s confirmation, recovery time, and validation evidence. As preventive actions, we strengthened storage endpoint monitoring, PVC mount failure alerts, pod-restart alerts, synthetic availability checks, and the troubleshooting runbook.”

### Strong closing line

“The main learning was to collect enough technical evidence before escalating to the cloud provider. That enabled Microsoft to investigate quickly and avoided unnecessary application or firewall changes.”

### Incident 2: Expired certificate affected login/application access

### Situation

“The second incident involved a certificate expiry that affected application access for multiple markets. The issue was initially reported by the Australia market after users encountered login or application-access failures.”

The certificate had not been included correctly in the existing monitoring coverage, so the first indication came from business users rather than a proactive expiry alert.

### Task

“As the L3 engineer responsible for the application infrastructure, I acknowledged the incident, assessed the affected markets and users, and started checking the AKS ingress, application certificates, and TLS configuration.”

### Actions

I checked:

Datadog availability and error-rate metrics

Ingress controller and application gateway logs

Kubernetes TLS secrets

Certificate validity and expiration date

Certificate chain

Domain and DNS mapping

Application pod status and logs

Key Vault certificate version

Recent certificate synchronization events

Example commands:

```bash
kubectl get certificate -A
kubectl get secret <tls-secret> -n <namespace>
kubectl describe certificate <certificate> -n <namespace>
openssl s_client -connect <hostname>:443 -servername <hostname>
```

“The investigation confirmed that the TLS certificate used by the application had expired. I renewed the certificate, updated the Kubernetes or Key Vault integration as required, and validated that the ingress was serving the new certificate.”

### Resolution

“The application was restored within approximately 30 minutes. After recovery, I worked with L2 and the governance team to communicate the service restoration to the affected markets.”

### RCA and preventive actions

“The root cause was not only the certificate expiry. The process gap was that the certificate had not been included in proactive expiry monitoring.”

Preventive actions included:

- Certificate inventory across all environments

- Alerts at 30, 15, and 7 days before expiry

- Automated certificate synchronization through Key Vault and cert-manager where supported

- Runbook for certificate renewal and rollback

- Ownership and escalation mapping

- Post-renewal synthetic endpoint validation

### Strong closing line

“The immediate fix was certificate renewal, but the permanent improvement was proactive certificate lifecycle monitoring so that future renewals happen before business impact.”

### Incident 3: Application pods failed because Key Vault secrets were missing

### Situation

“The third incident involved eKYC application pods continuously restarting in the integration environment. The JP market reported the issue, and several pods entered CrashLoopBackOff after a deployment or restart.”

The application depended on akv2k8s AzureKeyVaultSecret resources to retrieve sensitive values from Azure Key Vault. Only a small subset of the expected resources existed in the namespace, and the generated Kubernetes secret helm-secrets was missing.

### Task

“My responsibility was to determine whether this was an application defect, a container issue, a Key Vault access problem, or a FluxCD and Helm deployment dependency problem.”

### Actions

I first reviewed:

```bash
kubectl get pods -n ekyc
kubectl describe pod <pod> -n ekyc
kubectl logs <pod> -n ekyc --previous
kubectl get azurekeyvaultsecret -n ekyc
kubectl get secret helm-secrets -n ekyc
flux get hr -n ekyc
```

The evidence showed:

- Secret injection failed during container startup.

- Several AzureKeyVaultSecret custom resources were missing.

- helm-secrets had not been generated.

- Application HelmReleases could not resolve values such as kongIntrospectionKey.

- Pods restarted because the required environment variables were unavailable.

“I compared the running and failing workloads, reviewed the Helm values, checked the ekyc-secrets chart, and verified the FluxCD reconciliation status. That confirmed the dependency chain had broken.”

The expected flow was:

Azure Key Vault

↓

AzureKeyVaultSecret CR

↓

akv2k8s synchronization

↓

Kubernetes helm-secrets

↓

Application HelmRelease

↓

Application pods

### Resolution

“We restored the missing AzureKeyVaultSecret resources and ensured that helm-secrets was generated correctly. After confirming that the required keys existed, we reconciled the secrets and application HelmReleases and restarted the affected workloads in a controlled manner.”

### Validation

I validated:

- All required AzureKeyVaultSecret resources were synchronized

- helm-secrets existed with the expected keys

- FluxCD HelmReleases showed Ready=True

- Pods reached Running and Ready state

- Kubernetes events were clean

- Application endpoints and dependent services were accessible

- Business functionality was confirmed with the application team

### RCA and preventive actions

“The root cause was a deployment dependency failure involving missing Key Vault secret resources. The application chart depended on a Kubernetes secret that had not been created, which prevented the Helm release and application startup.”

Preventive actions included:

- Explicit HelmRelease dependsOn configuration

- CI validation of required secret resources

- FluxCD alerts for failed HelmReleases

- Predeployment checks for required Kubernetes secrets

- Environment comparison between working and failing namespaces

- Secret synchronization runbook

- Controlled rollback when a secrets-chart version introduces a regression

### Strong closing line

“This incident demonstrated the importance of treating secret synchronization as a deployment dependency rather than only as a runtime configuration.”

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
