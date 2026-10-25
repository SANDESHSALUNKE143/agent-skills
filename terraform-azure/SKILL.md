---
name: terraform-azure
description: >-
  Authors Azure Terraform from the shared module library in each project's
  infra repository. Use when changing Terraform, Azure resources, dev, UAT,
  prod, OIDC, SKUs, night shutdown, or secrets Terraform writes to GitHub.
---

# Terraform for Azure

Each project has its own infra repository. That repository calls a shared Terraform module library so every project creates the same kinds of resources. Environment differences are variables, not a copy of the modules. The API and UI repositories do not grow their own resource definitions.

This skill encodes the senior Azure engineer's decision. It does not choose SKUs, public exposure, budgets, or recovery objectives on its own.

## What Terraform is given, and what it creates

- The only input that already exists is an empty resource group per environment. Its name is a variable. Terraform does not create that group, does not delete it, and does not ask anyone to create anything else in the portal.
- Every other resource is Terraform: identity, network, registry, compute, data, schedules, role assignments, the GitHub OIDC trust, Log Analytics and metric alerts, a monthly budget alert per environment, and the production recovery resources. Run `terraform fmt` and `terraform validate` before a plan. Remote state has locking. Tags include the project and the environment. A stack with no diagnostic settings, no price alert, or no production recovery path is incomplete.
- A secret the resources generate (registry password, connection string, key) is written by Terraform into that environment's GitHub Actions secrets, and into Key Vault for the running app. It is not committed, not printed in the plan, and not pasted by hand. The GitHub provider is the only writer of those secrets.

## GitHub and Azure OIDC

GitHub and Azure trust each other with OIDC, in the pipeline.

- The Azure app has a federated credential whose issuer is `https://token.actions.githubusercontent.com` and whose subject is this infra repository and one branch. `dev` deploys the dev environment, `uat` deploys UAT, and `main` deploys prod.
- The workflow for that branch requests `id-token: write` and logs in with `azure/login` using the client id, tenant id, and subscription id. There is no `AZURE_CLIENT_SECRET` and no long-lived cloud password in GitHub.
- The same workflow identity is what Terraform uses to update GitHub environment secrets. A stored GitHub personal access token is not the path.

## Apply follows the branch

- A pull request plans only. After the checks and the required reviewer pass, merge into `dev` applies dev, merge into `uat` applies UAT, and merge into `main` applies prod. Do not run `terraform apply` locally.
- Dev and UAT use smaller SKUs than production. Production is the final size. The same module is called three times. `dev` is the smallest, `uat` is between dev and prod, and `prod` is the full SKU. Those sizes are tfvars.
- Dev and UAT have a Terraform-managed stop schedule for the night, default 20:00 to 08:00 in the deployment timezone. The window is a variable. Production has no stop schedule. The schedule is a resource, not a reminder for someone to shut the environment down.

## Required with every stack

- Diagnostic settings into Log Analytics, metric alerts for availability and error rate, and an action group.
- A monthly budget alert per environment, warning before the budget is spent and again when it is exceeded. Dev and UAT budgets are lower than prod.
- Production recovery resources for the volume that must survive a restart, Key Vault, and the registry. Dev and UAT keep backups of stateful data. They do not get a second region unless the Azure decision says so.
