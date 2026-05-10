# CI/CD Pipeline Governance Guide

## Leadership Context

Pipeline governance is a delivery confidence problem, not a DevOps tooling problem. Without it, the org cannot answer basic questions that engineering leadership needs answered: what changed, who approved it, and could it have caused this incident? DORA metrics — the primary lens for executive-level delivery reporting — are only credible when the pipeline enforces the gates that make them meaningful.

## Purpose

This guide covers the decisions and controls that determine whether a CI/CD pipeline is trustworthy at scale. It is written for engineering orgs that have pipelines running but have not formalized the rules around who can change them, what gates are required, and how to audit what ran.

A pipeline with no governance is a deployment mechanism. A pipeline with governance is part of the software supply chain, with accountability to match.

> **Demonstration sandbox:** [lifting-logbook](https://github.com/brownm09/lifting-logbook)
> is a personal-project monorepo, not a production system at scale. The artifacts linked
> in the Further-reading section illustrate the techniques described here; production-scale
> application of the same techniques is documented in [ORIGINS.md](../ORIGINS.md) where
> applicable.

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

The artifacts below illustrate the techniques described in this guide against the demonstration sandbox introduced after the Purpose section. See [LINKING.md](../LINKING.md) for the full convention. Citation links pin to commit [`413f8a6`](https://github.com/brownm09/lifting-logbook/tree/413f8a62f43f12fa200be3e3307da7ef72c7b446) per the LINKING.md SHA-pinning rule. Where an artifact is intended to evolve as the pipeline does, a `main` link is provided alongside.

### On required gates and gate ordering

- **CI workflow** — citation: [`.github/workflows/ci.yml` at 413f8a6](https://github.com/brownm09/lifting-logbook/blob/413f8a62f43f12fa200be3e3307da7ef72c7b446/.github/workflows/ci.yml); live state: [same path on `main`](https://github.com/brownm09/lifting-logbook/blob/main/.github/workflows/ci.yml). Defines the build-stage gates: lint and test under Turborepo, an analytics-taxonomy validator that runs as a separate enforced step, and a parallel `db-integration` job that spins up Postgres as a service container so DB-touching tests run against a real engine rather than a mock. Demonstrates the guide's claim that gates are only meaningful if they actually block — every step here exits non-zero on failure and there is no `|| true` suppression.
- **Deploy workflow** — citation: [`.github/workflows/deploy.yml` at 413f8a6](https://github.com/brownm09/lifting-logbook/blob/413f8a62f43f12fa200be3e3307da7ef72c7b446/.github/workflows/deploy.yml); live state: [same path on `main`](https://github.com/brownm09/lifting-logbook/blob/main/.github/workflows/deploy.yml). Worked example of the sequential `terraform staging → build images → deploy staging → smoke test → manual approval → terraform production → deploy production` pipeline this guide describes. Notable: the `terraform plan` output is reviewed before apply; production is gated behind an explicit GitHub Environment approval; smoke tests run between staging deploy and the production gate.

### On Turborepo task dependencies

- **`turbo.json`** — citation: [`turbo.json` at 413f8a6](https://github.com/brownm09/lifting-logbook/blob/413f8a62f43f12fa200be3e3307da7ef72c7b446/turbo.json); live state: [same path on `main`](https://github.com/brownm09/lifting-logbook/blob/main/turbo.json). Encodes the build/test/lint dependency graph and input scoping that lets the CI run only the affected packages. The explicit `inputs` arrays for each task are the part most worth reading: they define what changes invalidate the cache, which is where most monorepo CI bugs originate.

### On secret management and OIDC

- **OIDC federation in deploy.yml** — citation: [`google-github-actions/auth@v2` step at 413f8a6](https://github.com/brownm09/lifting-logbook/blob/413f8a62f43f12fa200be3e3307da7ef72c7b446/.github/workflows/deploy.yml). Demonstrates the guide's recommendation to prefer short-lived OIDC-issued credentials over long-lived service account keys: the workflow exchanges GitHub's OIDC token for GCP credentials per run via `workload_identity_provider` and `service_account` secrets. There are no long-lived GCP keys in repo or in GitHub secrets — only the WIF binding identifiers.

### What is missing (honestly)

- **Branch protection rules** are not visible from the repo. They are configured via GitHub's API/UI, not in committed YAML. The guide's "branch protection enforced" gate is a real requirement; it just cannot be inspected by reading the repo. To verify on a project of your own, query `gh api repos/{owner}/{repo}/branches/{branch}/protection`.
- **No CI-specific ADR exists yet.** The pipeline's design decisions (Turborepo over alternatives, the `terraform plan → manual approval` sequencing, OIDC over keys) are not yet recorded as ADRs in `docs/adr/`. This is a gap relative to the standard the rest of the stack holds itself to.
