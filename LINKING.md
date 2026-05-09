# Linking Conventions

This repo (`tech-leadership-reference`) is a reference library for engineering leadership
frameworks. Some frameworks describe practices the author has shipped at production scale —
those are documented in [ORIGINS.md](ORIGINS.md) with the engagement, program, and outcome.

Other frameworks describe practices that a reader can only evaluate by inspecting a working
system. For those, this repo links out to [`lifting-logbook`](https://github.com/brownm09/lifting-logbook) —
a personal-project monorepo (NestJS + GCP Cloud Run + Kubernetes + Terraform) that serves as
a demonstration substrate.

This file defines when to link out, how to label the link, and how to keep the links durable.

---

## When to link out to `lifting-logbook`

Link out only when **both** are true:

1. The framework makes a claim about practice (telemetry posture, runbook structure,
   IaC governance, on-call diagnostic flow, AI-assisted development workflow, etc.)
   that benefits from an inspectable artifact.
2. `ORIGINS.md` and `career-playbook` do **not** already attest to the practice from
   production experience.

When `ORIGINS.md` already grounds the framework in production (e.g., the on-call
restructuring case study), `lifting-logbook` plays a complementary role at most — the
production attribution remains primary, and a sandbox link is a "for the curious" footnote.

Do not link out when the framework's claims cannot be credibly demonstrated by a
personal-scale stack — e.g., PCI-DSS controls, M&A diligence, multi-team org design.
A forced link weakens the framing.

---

## Labeling convention

Every cross-link from this repo into `lifting-logbook` must carry a visible callout
distinguishing it from production attribution:

> **Demonstration sandbox:** [lifting-logbook](https://github.com/brownm09/lifting-logbook)
> is a personal-project monorepo, not a production system at scale. The artifact linked
> here illustrates the technique; production-scale application of the same technique is
> documented in [ORIGINS.md](ORIGINS.md) where applicable.

Place the callout once per page, near the first cross-link. Subsequent links on the
same page do not need to repeat it.

---

## Link stability

Two patterns, one rule each:

- **Citation links** ("this is the choice I made and the reasoning") — pin to a commit
  SHA or tagged release. These should not change underneath the reference doc.
  Format: `https://github.com/brownm09/lifting-logbook/blob/<SHA>/path/to/file#Lstart-Lend`

- **Live-state links** ("this is the current configuration") — link to `main`. These are
  expected to drift; the framework doc should say so.
  Format: `https://github.com/brownm09/lifting-logbook/blob/main/path/to/file`

If a link rots (file moved, deleted, refactored), prefer **removing the link** over
chasing the new path. The framework's argument should never depend on a working link.

---

## Footnote, not body prose

Cross-links go in footnotes or end-of-section "Further reading" blocks, not inline in
load-bearing paragraphs. The framework's argument must remain coherent if every link is
removed. Treat sandbox links as evidence a curious reader can verify, not as the
substance of the claim.

---

## Initial backfill targets (in priority order)

1. [`incident-management/system-legibility-and-diagnostic-playbook.md`](incident-management/system-legibility-and-diagnostic-playbook.md) —
   highest ROI; pair the high-cardinality telemetry argument with the actual telemetry
   choices made in the lifting-logbook stack.
2. [`ai-adoption/ai-adoption-readiness-framework.md`](ai-adoption/ai-adoption-readiness-framework.md) —
   the `.claude/` scaffolding (hooks, skills, settings) is itself the demonstration artifact.
3. [`incident-management/on-call-restructuring-framework.md`](incident-management/on-call-restructuring-framework.md) —
   complementary to the production case study; show the IaC + alerting config that
   makes a single-operator rotation tractable.

Other frameworks are linked opportunistically as content evolves. Do not backfill
exhaustively; link only where the link adds evaluative power.
