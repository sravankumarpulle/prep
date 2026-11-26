# Terraform Infrastructure CI/CD

## Overview

Infrastructure is provisioned with reusable Terraform modules and GitHub Actions. Pull requests run validation, security scanning, and `terraform plan`; approved merges run `terraform apply` using Azure OIDC authentication.

## Pipeline Flow

```text
Terraform Change -> Pull Request -> terraform fmt/validate
                 -> Gitleaks -> Checkov -> Terraform Plan
                 -> Review/Approval -> Merge -> Environment Approval
                 -> OIDC Login -> Terraform Apply -> Validation
```

- **Gitleaks:** detects credentials accidentally committed to Terraform, tfvars, scripts, or workflows.
- **Checkov:** detects insecure IaC, such as public storage, permissive network rules, missing encryption, or missing diagnostics.
- **Plan review:** checks unexpected create, update, replace, and destroy operations.
- **Apply:** runs only from the reviewed commit and approved environment.

## Repository Structure

```text
terraform-infrastructure/
├── modules/
│   ├── aks/
│   ├── key-vault/
│   ├── storage-account/
│   ├── postgresql/
│   └── private-endpoint/
├── environments/
│   ├── dev/
│   │   ├── main.tf
│   │   ├── providers.tf
│   │   ├── backend.hcl
│   │   └── dev.tfvars
│   ├── qa/
│   ├── stage/
│   └── prod/
└── .github/workflows/
    ├── terraform-plan.yml
    └── terraform-apply.yml
```

Modules contain reusable resource logic. Environment folders contain module calls, backend configuration, and non-sensitive values such as names, regions, SKUs, CIDRs, and tags.

Never commit passwords, client secrets, private keys, storage keys, or database credentials to `.tfvars` files.

## Backend and State Files

Terraform state is stored remotely in a private Azure Storage Account:

```text
tfstate container
├── dev/network.tfstate
├── dev/platform.tfstate
├── stage/network.tfstate
├── prod/network.tfstate
└── prod/docgen.tfstate
```

Separate state by environment and component/lifecycle to reduce blast radius. Do not create one file per resource unless there is a clear ownership or lifecycle requirement.

Backend controls:

- Public access disabled
- Azure RBAC and least privilege
- Blob lease-based state locking
- Encryption at rest
- Blob versioning and soft delete
- Storage firewall/private endpoint where required
- Diagnostic logging
- No state file committed to GitHub

State may contain sensitive attributes, even when Terraform output is marked `sensitive`, so it must be protected as confidential data.

## OIDC and Azure Authentication

GitHub Actions uses an Entra ID federated identity instead of a stored client secret:

```text
GitHub Workflow -> Short-lived OIDC Token -> Microsoft Entra ID
                -> Short-lived Azure Token -> Terraform/Azure
```

Use a separate identity and GitHub Environment for each environment:

```text
DEV identity   -> DEV subscription/resource group
STAGE identity -> STAGE subscription/resource group
PROD identity  -> PROD subscription/resource group + approval
```

Required workflow permission:

```yaml
permissions:
  contents: read
  id-token: write
```

The repository, branch, tag, or GitHub Environment must match the federated credential subject. Store subscription ID, tenant ID, and client ID as environment variables or GitHub configuration variables. Retrieve sensitive runtime values from Azure Key Vault when required.

## Reusable Terraform Workflow

```text
central-workflows/.github/workflows/
├── terraform-plan.yml
├── terraform-apply.yml
└── terraform-security.yml
```

The calling repository supplies:

- Environment name
- Terraform version
- Working directory
- Backend configuration path
- tfvars file
- Azure client, tenant, and subscription IDs

Example calling job:

```yaml
jobs:
  plan:
    uses: organization/central-workflows/.github/workflows/terraform-plan.yml@v2
    with:
      environment: prod
      working_directory: environments/prod
      tfvars_file: prod.tfvars
    permissions:
      contents: read
      id-token: write
```

## Validation and Rollback

After apply, validate resource status, private DNS, network access, RBAC, diagnostic settings, database/storage connectivity, alerts, and application dependencies.

Terraform rollback is not the same as application rollback. Reverting code can replace or destroy resources. Always review the new plan, protect stateful resources with appropriate lifecycle controls such as `prevent_destroy`, and use backup/restore or blue-green patterns where required.

## Interview Summary

> We store reusable Terraform modules in GitHub and maintain separate non-sensitive tfvars and state files for Dev, QA, Stage, and Production. A pull request runs formatting, validation, Gitleaks, Checkov, and Terraform plan. After code review and environment approval, GitHub Actions authenticates to Azure through OIDC and applies the reviewed change. State is stored securely in Azure Blob Storage with RBAC, locking, versioning, soft delete, and restricted network access. After apply, we validate networking, security, monitoring, and application dependencies before closing the change.
