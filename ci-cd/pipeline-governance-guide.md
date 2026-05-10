# CI/CD Pipeline Governance Guide

## Leadership Context

Pipeline governance is a delivery confidence problem, not a DevOps tooling problem. Without it, the org cannot answer basic questions that engineering leadership needs answered: what changed, who approved it, and could it have caused this incident? DORA metrics — the primary lens for executive-level delivery reporting — are only credible when the pipeline enforces the gates that make them meaningful.

## Purpose

This guide covers the decisions and controls that determine whether a CI/CD pipeline is trustworthy at scale. It is written for engineering orgs that have pipelines running but have not formalized the rules around who can change them, what gates are required, and how to audit what ran.

A pipeline with no governance is a deployment mechanism. A pipeline with governance is part of the software supply chain, with accountability to match.

## Background and Motivation

This framework draws from two deployments:

1. **Capital One (2019–2022):** I led the standardization of CI/CD pipelines across multiple engineering departments, completing the migration two months ahead of schedule. Outcome: 300 engineering hours saved per team.

2. **ActBlue Technical Services (2022–2024):** I chartered the DevEx team, which owned application-level delivery tooling — CI/CD pipelines, the internal developer platform, and deployment scaffolding. The cloud infrastructure layer was owned by a separate team outside my direct management.

## Pipeline Ownership Model

Every pipeline should have a clearly assigned owner. Pipelines without owners accumulate configuration drift and are the last to get updated when security requirements change.

**Ownership responsibilities:**
- Keep pipeline configuration current with platform standards
- Review and approve changes to pipeline steps with elevated permissions (deploy steps, secret access)
- Respond to pipeline failures that are not caught by on-call rotation

**Ownership model options:**

| Model | When it works | Watch out for |
|-------|--------------|---------------|
| Application team owns their pipeline | Teams move fast, pipelines match team needs | Inconsistency across teams; security controls vary |
| Platform team owns shared pipeline templates; app teams extend | Consistency with flexibility | Template updates can break team pipelines if not versioned |
| Platform team owns all pipelines | Maximum consistency and control | Bottleneck; teams cannot iterate quickly |

For most orgs with 3+ application teams, the shared template model (option 2) provides the right balance. Platform team defines the required gates; teams configure application-specific steps within those boundaries.

## Required Gates

These gates should be non-negotiable for any pipeline deploying to production:

**Build stage:**
- [ ] Dependency vulnerability scan (SCA) — blocks on critical/high CVEs with a defined exception process
- [ ] Static analysis (SAST) — results visible to developers; severity thresholds enforced
- [ ] Unit and integration tests — failure blocks promotion; flaky tests are tracked and fixed, not suppressed
- [ ] Container image scan — if building Docker images, scan before push

**Pre-deploy stage:**
- [ ] Branch protection rules enforced — direct commits to `main`/`production` are blocked; changes require PR + review
- [ ] Secrets scanning — no credentials, API keys, or tokens in committed code or pipeline configuration
- [ ] Infrastructure-as-code linting and validation (Terraform `validate`, `plan` output reviewed for destructive changes)

**Deploy stage:**
- [ ] Environment promotion is sequential: dev → staging → production; no skipping
- [ ] Production deploys require explicit approval (manual gate or automated approval from a defined approver group)
- [ ] Rollback procedure is defined and tested before the pipeline is considered production-ready

**Post-deploy:**
- [ ] Smoke tests or synthetic monitoring validates the deployment succeeded
- [ ] Deployment is logged with: who triggered it, what artifact was deployed, what SHA, at what time
- [ ] Alerting is in place to detect regressions within a defined window after deploy

## Secret Management

Secrets in pipeline configuration are one of the most common sources of credential exposure. The rules:

- Secrets are never stored in repository code, pipeline YAML, or environment variable configuration in plaintext
- Secrets are injected at runtime from a secrets manager (AWS Secrets Manager, HashiCorp Vault, GitHub Actions secrets with OIDC)
- Service accounts used by pipelines have the minimum permissions required for the steps they run
- Pipeline credentials are rotated on a defined schedule and immediately after team membership changes
- Access to production secrets is logged and reviewable

**OIDC over long-lived credentials:** Where the CI platform and cloud provider support it (GitHub Actions + AWS, for example), use OIDC federation to issue short-lived credentials per pipeline run rather than storing long-lived access keys. This eliminates the rotation problem entirely.

## Branch and Environment Strategy

The pipeline governance model should match the branching strategy. Common patterns:

**Trunk-based development:**
- All work merges to `main` via short-lived feature branches
- `main` is always in a deployable state
- Feature flags control what is live, not branches
- Pipeline: every merge to `main` triggers a deploy to staging; production deploy is a promotion with approval

**GitFlow (or modified GitFlow):**
- `develop` branch for integration, `release` branches for production candidates
- More pipeline complexity; more opportunities for gates to be bypassed if not enforced at each branch
- Appropriate for teams with long release cycles or strict change management requirements

For most modern web platforms, trunk-based is the right default. GitFlow adds overhead that is only justified when release cadence is constrained externally (regulatory approval, coordinated launches).

## Audit and Observability

A pipeline that cannot be audited is a liability in an incident or a compliance review.

**Minimum audit requirements:**
- Every pipeline run is logged with: triggering user or event, branch/SHA, environment, start/end time, pass/fail per stage
- Logs are retained for at least 90 days (12 months if in scope for PCI-DSS or similar)
- Who approved a production deploy is recorded and queryable
- Pipeline configuration changes are version-controlled and reviewed like application code

**Metrics worth tracking:**
- Deployment frequency (per team, per service)
- Change failure rate (what percentage of deploys require a rollback or hotfix)
- Mean time to restore (MTTR) after a failed deploy
- Pipeline duration trends (increasing duration is a signal that something needs attention)

These four metrics map directly to DORA metrics and give a factual answer to "how is our delivery health?"

## Change Control for Pipeline Configuration

Pipeline configuration is infrastructure. Treat it accordingly.

- Pipeline config lives in version control alongside application code, or in a dedicated platform configuration repo
- Changes to pipeline config require code review, the same as application changes
- Changes that add or modify steps with elevated permissions (secret access, production deploy) require review from the platform team or a designated approver
- Hotfix procedures for pipeline config are defined in advance (not improvised during an incident)

## Common Governance Failures

**Tests that always pass because failures are suppressed:** `|| true` at the end of a test command, or test results uploaded but not enforced. The gate exists in the config but does not block anything.

**Manual steps that are undocumented:** Someone SSH'd into a server once to fix a deploy, and now that step is assumed as part of the process but is not in the pipeline. The next person to run the deploy discovers it at 11pm.

**Production credentials in staging pipelines:** Staging pipelines use production credentials "temporarily" and the temporary state becomes permanent. Isolate credentials per environment.

**No rollback test:** Rollback procedures are defined but have not been executed since the pipeline was built. Test rollback before you need it.

**Drift between environments:** Staging and production pipelines diverge over time because fixes go to production first and are not backported. The staging pipeline stops being representative. Run the same pipeline config across all environments; parameterize what needs to differ.

---

## Further reading: demonstration artifacts

> **Demonstration sandbox:** [lifting-logbook](https://github.com/brownm09/lifting-logbook)
> is a personal-project monorepo, not a production system at scale. The artifact linked
> here illustrates the technique; production-scale application of the same technique is
> documented in [ORIGINS.md](../ORIGINS.md) where applicable.

The artifacts below illustrate the techniques described in this guide against the demonstration sandbox. See [LINKING.md](../LINKING.md) for the full convention. Citation links pin to commit [`413f8a6`](https://github.com/brownm09/lifting-logbook/tree/413f8a62f43f12fa200be3e3307da7ef72c7b446) per the LINKING.md SHA-pinning rule. Where an artifact is intended to evolve as the stack does, a `main` link is provided alongside.

### On pipeline ownership and shared template configuration

- **Monorepo and pipeline structure decision** — citation: [ADR-001: Monorepo Structure](https://github.com/brownm09/lifting-logbook/blob/413f8a62f43f12fa200be3e3307da7ef72c7b446/docs/adr/ADR-001-monorepo-structure.md); live state: [same path on `main`](https://github.com/brownm09/lifting-logbook/blob/main/docs/adr/ADR-001-monorepo-structure.md). Records the Turborepo + npm workspaces decision. The `turbo.json` task graph is the mechanism by which a platform team defines the required pipeline steps — build dependencies, caching contracts, test ordering — that all application packages inherit. Demonstrates the shared-template ownership model: the platform team sets the structure; teams configure application-specific inputs and outputs within it.

### On required gates and sequential environment promotion

- **CI pipeline** — citation: [`.github/workflows/ci.yml` at `413f8a6`](https://github.com/brownm09/lifting-logbook/blob/413f8a62f43f12fa200be3e3307da7ef72c7b446/.github/workflows/ci.yml); live state: [same path on `main`](https://github.com/brownm09/lifting-logbook/blob/main/.github/workflows/ci.yml). The Lint & Test job runs `npx turbo run lint test` with Turborepo caching, enforces Prisma client generation, and validates the analytics taxonomy schema. All gates block the PR — they are not advisory. Demonstrates that the required-gate checklist from this guide translates to enforced pipeline steps, not just documentation.
- **Deploy pipeline** — citation: [`.github/workflows/deploy.yml` at `413f8a6`](https://github.com/brownm09/lifting-logbook/blob/413f8a62f43f12fa200be3e3307da7ef72c7b446/.github/workflows/deploy.yml); live state: [same path on `main`](https://github.com/brownm09/lifting-logbook/blob/main/.github/workflows/deploy.yml). Multi-stage promotion: CI must pass before staging deploy runs; production deploy requires a manual approval gate. The deploy pipeline is the place where "environment promotion is sequential" becomes enforceable rather than advisory.

### On secret management (OIDC over long-lived credentials)

- **Workload Identity Federation** — citation: [`.github/workflows/deploy.yml` at `413f8a6`](https://github.com/brownm09/lifting-logbook/blob/413f8a62f43f12fa200be3e3307da7ef72c7b446/.github/workflows/deploy.yml) (same artifact as above). The deploy pipeline uses GCP Workload Identity Federation (`GCP_STAGING_WORKLOAD_IDENTITY_PROVIDER`, `GCP_PROD_WORKLOAD_IDENTITY_PROVIDER`) rather than stored long-lived service account keys. Each pipeline run receives a short-lived credential federated from GitHub's OIDC token — illustrating the "OIDC over long-lived credentials" rule from this guide's Secret Management section in practice.

---

These artifacts are not exhaustive. Per [LINKING.md](../LINKING.md), additional cross-references are added only where they add evaluative power — not as breadth for its own sake.
