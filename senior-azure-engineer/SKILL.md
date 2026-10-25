---
name: senior-azure-engineer
description: >-
  Decides Azure resource shape, SKUs, security, disaster recovery, monitoring,
  and price alerts before Terraform is written. Use when planning Azure
  architecture, cost, identity, networking, recovery, or budgets.
---

# Senior Azure engineer

Decide before any Terraform is written. The decision is the same resource set for dev, UAT, and prod. Only size, exposure, and the night window change. Hand the decision to the Terraform skill. Do not leave a portal step for anything inside the resource group.

## Workload shape

For this platform, follow `infrastructure/README.md`: Azure Container Registry, Container Apps or App Service, an internal API, an external console, a volume only when data must survive restart, and secrets in Key Vault references.

- One empty resource group per environment is the only thing that exists before Terraform. Record its name. Do not design a portal step for anything inside it.
- SKUs: dev smaller than UAT, UAT smaller than prod, prod the final size. Name the three SKUs. Dev and UAT stop at night (default 20:00–08:00 in the deployment timezone). Production stays on.
- Identity is GitHub OIDC into Azure for `dev`, `uat`, and `main`. `main` is the prod environment. Terraform writes generated secrets back to that GitHub environment. No long-lived cloud password.

## Security

Security is part of the design, not a follow-up. Managed identity, least-privilege RBAC, public data access off, private access for data stores, Key Vault for secrets, and no keys in client apps. A design that leaves a store public or a secret in the app is not ready for Terraform.

## Monitoring

Monitoring is Azure Monitor on every environment. A Log Analytics workspace receives diagnostic settings from the compute, registry, Key Vault, and data resources. Alerts cover availability and error rate, and they notify an action group. Dev and UAT use the same signals with a smaller retention than prod.

## Disaster recovery

Disaster recovery is required for production. Name the recovery region or the backup store, the recovery point objective, and the recovery time objective. Stateful data (the volume that must survive a restart, Key Vault, and the registry) has a backup or a paired copy. Dev and UAT keep backups of stateful data. They do not get a second region unless the workload requires it. The recovery path is a Terraform resource, not a document only.

## Price alerts

A price alert is required on each environment. Terraform creates a monthly budget at the resource group for the SKU of that environment, with an alert before the budget is spent and an alert when it is exceeded. The action group receives it. Dev and UAT budgets are lower than prod because their SKUs are lower. The night stop remains the other cost control for dev and UAT. Name the meters that dominate: always-on compute, registry, egress, and log ingestion.

## Decision to hand off

Output a short decision the Terraform skill can implement:

- Resource group names
- The three SKUs
- The night window
- Network exposure
- The OIDC subject for `dev`, `uat`, and `main`
- The monitor and alert set
- The budget amounts
- The production recovery point and time objectives
- Cost drivers
- What must not be created
