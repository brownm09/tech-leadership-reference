# Tech Leadership Reference

Operational and leadership frameworks from production engineering work across payments, platform, and compliance contexts. Each document reflects decisions made in real systems, not hypothetical best practices. Organized by the leadership domain each framework addresses.

**[ORIGINS.md](ORIGINS.md)** traces each framework back to the specific engagement, program, and outcome that produced it, with first-person framing for each.

**[LINKING.md](LINKING.md)** documents the convention for cross-references to [lifting-logbook](https://github.com/brownm09/lifting-logbook) as a demonstration sandbox for frameworks where production experience is not the basis of the claim.

## Org Design & Strategy

Frameworks for how a Director or Head of Engineering structures the organization, sets technical direction, and makes investment decisions.

**[org-design/team-topology-framework.md](org-design/team-topology-framework.md)**
How to design and evolve engineering org structure using the four team types (stream-aligned, platform, enabling, complicated-subsystem). Covers cognitive load thresholds, the Inverse Conway Maneuver, reorg execution, and post-reorg metrics. Grounded in real reorg patterns from high-growth platforms.

**[metrics/engineering-health-scorecard.md](metrics/engineering-health-scorecard.md)**
How to build a leadership-facing engineering health dashboard across four domains: delivery (DORA), quality (SLO attainment, defect escape rate), reliability (MTTR, incident frequency), and team health (on-call load, lead time). Covers instrumentation, target-setting, QBR narrative, and what not to put in front of executives.

**[strategy/technical-strategy-template.md](strategy/technical-strategy-template.md)**
Full skeleton and guidance for a 1–3 year technical strategy document: current state assessment, target state vision, gap analysis, investment roadmap (3-horizon), and success metrics. Includes communication format guidance for engineers, executives, and board audiences.

**[strategy/build-vs-buy-framework.md](strategy/build-vs-buy-framework.md)**
Three-way decision framework (build / buy / partner) across five scored dimensions: strategic differentiation, 5-year TCO, integration complexity, velocity impact, and vendor risk. Includes weighted scoring matrices by org type, a bias inventory, and three worked examples (feature flags, data pipeline, auth provider).

**[strategy/ma-technical-due-diligence.md](strategy/ma-technical-due-diligence.md)**
Engineering assessment framework for acquisitions. Covers 8 domains (architecture, technical debt, security, team, operational maturity, data practices, IP/licensing, integration complexity) with artifacts to request, interview questions, and red/yellow flags per domain. Includes a 2-week diligence sprint structure and executive report format.

---

## People Systems

Frameworks for managing engineering talent: defining expectations clearly, running promotions fairly, and planning the headcount required to deliver on commitments.

**[people/career-ladder-calibration.md](people/career-ladder-calibration.md)**
How to build or inherit a career ladder and run calibration. Covers IC vs. manager track design, a four-axis level definition format (impact scope, decision autonomy, execution, collaboration), full L3–Staff worked definitions, calibration session structure, the three failure modes (grade inflation, recency bias, proximity bias), and a documented appeals process.

**[people/headcount-capacity-planning.md](people/headcount-capacity-planning.md)**
How to model HC needs, build the finance ask, and sequence hiring against roadmap risk. Covers the four capacity inputs, initiative-level modeling with worked examples, the one-page HC request format, backfill vs. new headcount criteria, common planning errors, and a 12-month rolling model template.

---

## Operating Cadence

Frameworks for the Chief of Staff, Engineering function: how information flows, how decisions are made, how leadership time is spent, and how engineering communicates upward.

**[operating-cadence/engineering-rhythm-of-business.md](operating-cadence/engineering-rhythm-of-business.md)**
The weekly/monthly/quarterly/annual meeting cadence for a 50-engineer org. Covers agenda templates, decision authority (DACI), meeting health audit methodology, CoS responsibilities, and a sample weekly and monthly calendar.

**[operating-cadence/engineering-okr-framework.md](operating-cadence/engineering-okr-framework.md)**
How to set, cascade, and report engineering OKRs without the common failure modes (activity OKRs, unmeasurable aspirations). Covers the org → team cascade, grading, quarterly review format, and anti-patterns.

**[operating-cadence/executive-communication-templates.md](operating-cadence/executive-communication-templates.md)**
Five ready-to-use templates for written artifacts engineering leaders produce for executive and board audiences: engineering QBR, board engineering update, incident executive summary, major initiative status memo, and headcount request memo.

**[operating-cadence/cross-functional-alignment-playbook.md](operating-cadence/cross-functional-alignment-playbook.md)**
How engineering works with product, design, legal, finance, and sales. Covers decision authority models, SLAs, intake processes, and a RACI template for cross-functional program decisions. Opens with the three partnership failure modes.

**[operating-cadence/escalation-framework.md](operating-cadence/escalation-framework.md)**
When and how to escalate issues at the leadership layer (distinct from on-call escalation). Covers the three trigger categories (delivery risk, people risk, reputational/external risk), four escalation levels with response SLAs, the 5-sentence escalation memo format, and anti-patterns.

---

## Program Management

Frameworks for TPMs and engineering leads running programs that span multiple teams or exceed a single team's ownership.

**[program-management/program-planning-playbook.md](program-management/program-planning-playbook.md)**
How to scope, launch, and run a multi-team technical program. Covers the program brief format, kickoff structure, milestone tracking (leading vs. lagging indicators), the weekly sync format, closure criteria, and common failure modes including the coordinator tax.

**[program-management/raid-dependency-tracking.md](program-management/raid-dependency-tracking.md)**
How to run a RAID log and dependency matrix. Covers risk register, assumption log, issue tracker, and dependency matrix formats with worked team-by-team examples. Includes the weekly RAID review ritual and health signals distinguishing live maintenance from theater.

**[program-management/launch-readiness-checklist.md](program-management/launch-readiness-checklist.md)**
Go/no-go gate framework for major releases. Eight readiness domains with owner-per-domain accountability, blocker vs. waiver decision process, launch day runbook (T-24h through T+24h), and post-launch review cadence.

**[program-management/stakeholder-communication-templates.md](program-management/stakeholder-communication-templates.md)**
Templates for weekly status reports, escalation memos, kickoff emails, milestone announcements, and stakeholder tier maps. Includes frequency calibration guidance and the three writing principles for upward communication.

**[program-management/change-management-framework.md](program-management/change-management-framework.md)**
How to manage technical, process, and organizational transitions. Covers a 5-phase change model, a resistance root cause framework, communication plan template, rollback criteria, and success metrics including a one-question pulse survey format.

---

## Operations

Frameworks for running reliable engineering operations: on-call structure, incident readiness, CI/CD governance, and release management.

**[incident-management/on-call-restructuring-framework.md](incident-management/on-call-restructuring-framework.md)**
Framework for restructuring an on-call program that has outgrown its original design. Covers the IC/responder split, rotation structure, Wed-Wed cadence, metrics instrumentation via Jeli and Jira, and the rollout approach. Based on a restructuring that expanded the on-call program from a 5-person SRE team absorbing everything to per-team responder rotations backed by a 12-person incident commander rotation.

**[disaster-recovery/fire-drill-template.md](disaster-recovery/fire-drill-template.md)**
Tabletop exercise format for stress-testing DR posture and incident management protocols. Covers scenario sequencing (catastrophic failure first), participant structure, pre/post checklists, and what to watch for. Built from exercises run at ActBlue to validate payment continuity under failure scenarios.

**[ci-cd/pipeline-governance-guide.md](ci-cd/pipeline-governance-guide.md)**
Governance model for CI/CD pipelines at scale: ownership, required gates, secret management, audit requirements, and change control. Covers the shared template model, DORA metric instrumentation, and the most common governance failures.

**[experimentation/launchdarkly-rollout-governance.md](experimentation/launchdarkly-rollout-governance.md)**
Flag lifecycle management for LaunchDarkly at scale. Covers flag taxonomy, naming conventions, rollout sequencing, targeting rules, experiment guardrails, and the cleanup policy that prevents flag debt from accumulating.

---

## Compliance & Security

**[compliance/pci-dss-gap-analysis-checklist.md](compliance/pci-dss-gap-analysis-checklist.md)**
Engineering-focused checklist for a PCI-DSS v4.0 gap analysis. Organized by all 12 requirement domains, with emphasis on the controls engineering teams are most likely to own or partially own. Includes a section on common gaps found in first assessments.

---

## Technology Adoption

**[ai-adoption/ai-adoption-readiness-framework.md](ai-adoption/ai-adoption-readiness-framework.md)**
Framework for rolling out AI-assisted development tools (GitHub Copilot, Claude, Claude Code, Cursor) across engineering organizations. Covers a four-stage IC fluency rubric, role-differentiated use cases (IC vs. manager), task classification by risk level, a three-tier observability model, and a phased gate model that sequences access to higher-risk workflows on demonstrated readiness. Developed across two production rollouts (ActBlue 2024–2025, CTA 2025–2026). Includes a worked example and rollback triggers.

---

## Product & Documentation

Frameworks for product definition and AI-assisted documentation standards.

**[leadership/ai-documentation-standards.md](leadership/ai-documentation-standards.md)**
Auditability and verifiability standards for AI-generated architectural documentation, ADRs, design docs, and playbooks. Covers attribution requirements, verification obligations, confidence labeling, and the review process for AI-assisted content. Applicable to any project using Claude Code or another AI assistant as a contributor.

**[leadership/prd-lifecycle.md](leadership/prd-lifecycle.md)**
One living PRD per product. Covers PRD structure (job-outcome table, personas, hypothesis and bets, append-only changelog), two-tier update process (minor vs. major changes), lifecycle stages with gates (Discovery → Alignment → Delivery → Shipped), ownership model, and common failure modes including the frozen PRD, PRD-as-contract, and solution PRD anti-patterns.

---

## Context

These frameworks were developed and applied across high-volume, regulated platforms including a payments processor handling 1M+ transactions/day under PCI-DSS. They span the full scope of engineering leadership: from org design and technical strategy at the director level, to operating cadence and executive communication at the chief-of-staff level, to cross-team program delivery at the TPM level. They are written to be adapted, not followed verbatim. The right answer for your system depends on your scale, team structure, and constraints.
