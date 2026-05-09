# On-Call Restructuring Framework

## Leadership Context

On-call structure is a talent and delivery risk, not just an ops concern. Unsustainable rotation load drives attrition among your most experienced engineers, increases incident duration, and degrades the reliability signal that leadership depends on for investment decisions. A well-designed on-call program is an argument for trust — it demonstrates that the org can absorb failure without heroics.

## Background and Motivation

This framework was developed from the on-call restructuring at ActBlue Technical Services (2022–2024). I redesigned the cross-team incident handover process and redefined responder roles across a 5-person SRE team that was absorbing every incident in the organization. The structural changes — separating coordination from resolution, introducing per-team rotations, and centralizing alert instrumentation — reduced friction and improved stakeholder resolution clarity.

## Problem Statement

A common failure mode in on-call programs: a small team gets paged for everything, regardless of who actually owns the problem or can fix it. Alerts are not standardized, so some issues surface only when an engineer notices something is wrong, not when a monitor fires. There is no mechanism for recording incident start and end times, so the org has no data on frequency, duration, or trend.

This is the state ActBlue's on-call program was in before restructuring. The 5-person SRE team, plus a handful of staff-level volunteers, was absorbing every incident across the organization. Responders were expected to both coordinate the incident and resolve it, which degrades response quality under pressure. Alerts existed across systems but were not centralized or standardized, which meant detection lag varied by incident type and by who happened to be paying attention.

## Structural Changes

### Separate Coordination from Resolution

The core change is splitting the incident responder role into two distinct functions:

**Incident Commander (IC):** Runs the incident. Owns communication, coordinates between responders, makes scope and escalation decisions, writes the timeline. Does not resolve the technical problem directly. Filled by staff engineers and engineering managers with cross-team context.

**Responder:** Resolves the technical problem. Has subject matter expertise for the systems in scope. Focused on diagnosis and fix, not on managing the room.

This separation reduces cognitive load on responders and improves incident coherence. An engineer who is simultaneously troubleshooting a database failure and answering questions from leadership is doing both jobs worse than they would do either alone.

### Rotation Structure

**Before:**
- 5-person SRE team plus ad hoc staff+ volunteers
- Called for all incidents regardless of team ownership
- No formal tiering

**After:**
- IC rotation: ~12 staff engineers and engineering managers on a shared rotation
- Responder rotations: per-team rotations of 4-5 engineers each, scoped to systems that team owns
- Teams handle their own incidents; IC rotation provides coordination support and handles incidents that cross team boundaries

This distributes on-call load across the org while keeping subject matter expertise in the responder role where it belongs.

### Rotation Cadence

Weekly rotation, shifted from Monday-to-Monday to Wednesday-to-Wednesday. The Mon-Mon cadence concentrated handoffs around weekends and holidays, which created coverage gaps. Wed-Wed puts the handoff midweek when staffing is most predictable.

## Metrics and Tooling

On-call health is only improvable if it is measurable. Two metrics were tracked:

- **Incident frequency:** Number of incidents per week/month, by team and by severity tier
- **Time-to-resolve (TTR):** Time from incident open to resolution, tracked per incident

These were captured via Jeli for incident retrospectives and a dedicated Jira incident project for tracking. The combination gives a running record of incidents (Jira) and structured postmortems with timeline reconstruction (Jeli).

Before this tooling was in place, the org had no reliable way to answer: how many incidents did we have last quarter? Which team had the highest TTR? Is frequency trending up or down? After, those answers were available with a filter query.

## Rollout Notes

No significant pushback from engineers. The restructuring reduced on-call burden for the teams that had been absorbing everything, and scoped responsibility more clearly for everyone else. Engineers who previously felt over-exposed got relief; engineers who had been invisible to the rotation got a defined, bounded responsibility.

The IC rotation is the role most likely to generate questions. Staff engineers and managers are not always accustomed to running an incident without also being a technical contributor. The framing that worked: the IC's job is to make the responders faster, not to be the smartest person in the room about the underlying system.

## Implementation Checklist

- [ ] Define severity tiers (P0-P3 or equivalent) with clear criteria for each
- [ ] Map system ownership: which team owns which services and is responsible for the responder rotation
- [ ] Identify IC rotation candidates: staff engineers and managers with cross-team context
- [ ] Establish alert standards: all alerts go to a centralized channel with consistent severity labeling
- [ ] Configure Jeli (or equivalent) for postmortems; create Jira project for incident tracking
- [ ] Set rotation cadence and handoff protocol: what information transfers at handoff, in what format
- [ ] Shift rotation window to midweek (Wed-Wed) to avoid weekend/holiday handoff problems
- [ ] Run one full incident cycle before declaring the restructuring complete; adjust based on what breaks

## Common Failure Modes

**Paging the same team for everything:** It looks like a coverage guarantee but is actually a single point of failure. When the SRE team is unavailable or overwhelmed, incidents stall.

**Alerts without owners:** An alert that fires to a shared channel with no assigned team produces slow response by diffusion of responsibility. Every alert should have a team and a responder rotation behind it.

**No postmortem process:** TTR and frequency data without retrospectives identifies that problems exist but not why. Jeli-style postmortems with timeline reconstruction surface the systemic causes.

**Mon-Mon rotations:** Weekend and holiday handoffs create gaps that are predictable and preventable. Move the boundary to Wednesday.

---

## Further reading: demonstration artifacts

> **Demonstration sandbox:** [lifting-logbook](https://github.com/brownm09/lifting-logbook) is a personal-project monorepo, not a production system at scale. The framework above is grounded in the ActBlue restructuring documented in [ORIGINS.md](../ORIGINS.md); the artifacts below are complementary — they show the diagnostic infrastructure that makes any rotation tractable, including a rotation of one. See [LINKING.md](../LINKING.md) for the full convention.

A single-operator on-call posture is the worst case for the framework's *separate coordination from resolution* split — there is no second person to take either role. The compensating mechanism is making the system legible enough that the same operator can fall back to the runbook rather than reasoning from first principles under pressure. Citation links pin to commit [`413f8a6`](https://github.com/brownm09/lifting-logbook/tree/413f8a62f43f12fa200be3e3307da7ef72c7b446).

### On the operational contract

- **On-call ops doc** — citation: [`docs/operations/on-call.md` at `413f8a6`](https://github.com/brownm09/lifting-logbook/blob/413f8a62f43f12fa200be3e3307da7ef72c7b446/docs/operations/on-call.md); live state: [same path on `main`](https://github.com/brownm09/lifting-logbook/blob/main/docs/operations/on-call.md). Defines severity tiers (SEV1–3), alert-to-severity mapping, escalation paths (15-min page for SEV1, DM for SEV2), and the postmortem template. Demonstrates the framework's Implementation Checklist as an artifact: every item the checklist names has a corresponding line in this doc.
- **On-call readiness proposal** — [`docs/proposals/2026-05-08-on-call-readiness.md`](https://github.com/brownm09/lifting-logbook/blob/413f8a62f43f12fa200be3e3307da7ef72c7b446/docs/proposals/2026-05-08-on-call-readiness.md) (citation, SHA-pinned). Captures the problem framing and sequencing — runbooks, SLOs, and incident response are deliberately staged after the observability stack lands, not before. Demonstrates the framework's claim that on-call structure depends on detection infrastructure being in place first.

### On reliability targets and burn-rate alerting

- **SLO definitions** — citation: [`docs/operations/slo.md` at `413f8a6`](https://github.com/brownm09/lifting-logbook/blob/413f8a62f43f12fa200be3e3307da7ef72c7b446/docs/operations/slo.md); live state: [same path on `main`](https://github.com/brownm09/lifting-logbook/blob/main/docs/operations/slo.md). 99.5% availability, p95 latency < 1s, 28-day rolling window, error budget = 3.36h, burn-rate thresholds (14×, 6×, 3×). Defines the contract that determines whether a page is warranted. Demonstrates the framework's Metrics and Tooling section: incident frequency and TTR are only meaningful against an explicit reliability target.
- **SLO methodology rationale** — [ADR-019: SLO Methodology at `413f8a6`](https://github.com/brownm09/lifting-logbook/blob/413f8a62f43f12fa200be3e3307da7ef72c7b446/docs/adr/ADR-019-slo-methodology.md) (citation, SHA-pinned). Records the choice to adopt burn-rate alerting over simple threshold alerts, with a deferral until production traffic exists for calibration.

### On alert ownership and routing

- **Prometheus alert rules** — [`infra/observability/alerts/api.yaml`](https://github.com/brownm09/lifting-logbook/blob/main/infra/observability/alerts/api.yaml) (live state). Three rules (`APIHighErrorRate` > 1% for 5m, `APIHighP95Latency` > 1s for 5m, `APINoRequests`); each carries a `runbook_url` annotation linking to a specific runbook. Demonstrates the *Alerts without owners* failure mode at the artifact level: every alert in this file has an explicit destination *and* a documented response path, which is the minimum bar before any rotation is sustainable.

### What is missing — and what that demonstrates

The single-operator rotation has no IC/responder split, no per-team rotation, no Wed-Wed handoff cadence. Those mechanisms only have meaning at organizational scale. The lifting-logbook artifacts show the diagnostic substrate the framework's structural changes depend on — they are not a substitute for the organizational restructuring documented in ORIGINS.md.
