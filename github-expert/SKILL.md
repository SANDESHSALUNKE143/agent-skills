---
name: github-expert
description: >-
  Creates GitHub issues, projects, pull requests, and reviews, and enforces
  feature-to-dev-to-uat-to-main with required checks. Use when working with
  issues, projects, commits, pull requests, branch protection, or GitHub Actions.
---

# GitHub expert

Use `gh` and git. Creating a commit, pushing, or opening a pull request still waits for an explicit ask. When a commit is made, its message includes the issue id.

Promotion is the same for the API, the UI, and each project's infra repository. A feature branch opens a pull request into `dev`. `dev` opens a pull request into `uat`. `uat` opens a pull request into `main`. The three environments stay dev, UAT, and prod. Merge into `dev` deploys dev. Merge into `uat` deploys UAT. Merge into `main` deploys prod. A pull request only runs tests. Deploy runs after the merge, and only when the required checks and the one company reviewer have passed.

## Issues and projects

Onboarding a repository creates its GitHub Project before feature work starts. The project has the states Todo, In Progress, In Review, In Dev, In UAT, and Done.

A new issue is created with `gh issue create`. It states the problem, who is affected, and the acceptance criteria, then is added to that project in Todo.

Every commit message contains the issue id, written as `#123` in the subject. A commit without an issue id is not made. Moving the work moves the issue:

- In Progress when the feature branch opens
- In Review when the pull request opens
- In Dev when it merges to `dev`
- In UAT when it merges to `uat`
- Done when it merges to `main`

Commit analysis reads `git log` for the range named, groups the history by issue id, and reports any commit that has no issue id. It includes authors and dates and summarizes why the history looks like that. It does not rewrite history.

## Pull requests

Pull requests follow one path. A feature branch targets `dev`. `dev` targets `uat`. `uat` targets `main`. A pull request does not skip a branch. The production pull request is the UAT-to-`main` promotion.

The description includes the issue id and these sections:

- Problem and resulting behavior
- Validation actually run
- Compatibility, security, cost, and rollback

A one-line pull request is sent back. Push and open the pull request only when asked.

Company policy requires one approving reviewer. Branch protection on `dev`, `uat`, and `main` sets that required review. Direct pushes to those branches are off.

## Required checks

All of these must pass before the pull request is mergeable:

- Python unit tests, Python integration tests, and `ruff check`.
- UI unit tests, UI integration tests, and the UI lint check.
- Coverage greater than 90 percent for the Python tree and greater than 90 percent for the UI unit and integration runs. A miss fails the check.

The pull request workflow runs those checks and, for infra, a Terraform plan. It does not deploy. After the checks and the reviewer pass, the merge is what deploys. Push to `dev` deploys dev, push to `uat` deploys UAT, and push to `main` deploys prod. The deploy job uses GitHub OIDC to Azure (`id-token: write`, no cloud client secret) and reads values from that GitHub environment's secrets. A workflow file does not hardcode a token, a password, or a connection string.

## Secrets

Secrets produced by Azure resources stay the GitHub environment secrets Terraform writes. Do not set those with `gh secret set` or the GitHub UI. A missing secret is a gap in the infra repo, not a manual step.

## Review

A review reads the full diff and comments on correctness, missing tests, the issue id, security, and cost or contract impact. It notes what was not run. It does not approve a pull request whose checks are failing or that has no tests for the behavior it changes. Review a production pull request as a promotion of what UAT already ran.
