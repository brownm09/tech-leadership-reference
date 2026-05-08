# Head of Engineering Operational Dashboard

## Leadership Context

The [Engineering Health Scorecard](engineering-health-scorecard.md) is a communication artifact — it answers the questions a non-technical executive will ask about engineering health. The operational dashboard described here serves a different function: it is the weekly instrument the Head of Engineering uses to know whether anything is silently on fire before it becomes someone else's problem.

The distinction matters because the two artifacts optimize for different things. A scorecard is designed to be credible to an audience that does not have context; it must be curated, narrativized, and defensible. A dashboard is designed to be fast for the one person who has full context; it must surface anomalies immediately, require no manual curation, and answer the Monday morning question: *is anything wrong that I do not already know about?*

If everything is green, the dashboard should take 90 seconds to review. It earns its keep when something is amber or red and you already know exactly where to look.

## Background and Motivation

This framework was developed from direct experience running engineering at ActBlue Technical Services (2024–2025), where I led a six-team platform directorate and owned the metrics infrastructure for engineering health reporting. The distinction between "what I show the CTO" and "what I actually watch every week" became clear after the first QBR cycle: the scorecard was useful for quarterly storytelling but too lagging and too curated to catch in-week drift. The operational dashboard fills that gap.

---

## When to Use This

| Trigger | What the dashboard provides |
|---------|-----------------------------|
| Weekly leadership ritual | Answers "is anything off?" before the weekly team sync |
| First 90 days in a new HoE role | Fastest way to build a baseline and spot which signals are already in the red |
| After a major incident or miss | Ongoing monitoring to confirm recovery is real, not just declared |
| Scaling the org | People and capacity signals degrade before delivery signals do — catch it early |
| Pre-planning / pre-QBR | Raw material for identifying the two or three things worth discussing with leadership |

The dashboard is not a management tool for engineers. Do not share it directly with ICs or use it in performance conversations — it lacks the context to be fair at that resolution. It is a tool for the person who holds the org's risk.

---

## The Five Domains

### Domain 1: Delivery Pulse

**Question it answers:** Are we shipping, and is the pipeline healthy?

Delivery is the most instrumented domain and usually the easiest to get right. The risk is that aggregate numbers mask team-level outliers.

**Deployment frequency (per team, trailing 7 days)**

The org-level aggregate hides the team that has not shipped in three weeks. Track per-team, not per-org. One team at zero deployments for two consecutive weeks is a signal worth a conversation; it rarely shows up in org-level DORA reporting.

- Watch for: a team that was shipping twice a week suddenly shipping zero
- Common cause: unresolved merge conflicts, a release freeze nobody communicated, a team absorbed into an incident

**Lead time for change (trailing 14 days)**

Track the P90, not just the median. A P90 that is two or three times the median means a class of work is getting stuck — usually large PRs, dependency-blocked tickets, or tickets waiting on a specific reviewer.

- Watch for: P90 climbing while median stays flat
- Common cause: a small number of long-running branches, a bottleneck reviewer, or tickets classified as "in progress" that have been idle for a week

**PR cycle time (open to merge)**

Distinct from lead time. This is purely the code review and merge window. A PR that sits for three days before getting a review is a team health signal as much as a delivery signal — it often means the team is heads-down on incidents, on-call, or a high-priority fire, and code review has been deprioritized.

- Target: <24 hours for P90 on standard PRs (not draft/WIP)
- Watch for: specific engineers whose PRs consistently sit longer — may indicate a review bottleneck or an interpersonal dynamic

**Change failure rate (trailing 30 days, per team)**

Same as the scorecard metric but tracked at team resolution. An org-level CFR of 6% can hide one team at 25% and four teams at 2%. The team at 25% has a testing or deployment process problem that will not fix itself.

---

### Domain 2: Quality and Reliability

**Question it answers:** Is what we shipped holding up?

**SLO burn rate by service (rolling 30 days)**

More useful than point-in-time SLO attainment because burn rate tells you whether you are on a trajectory to breach, not just whether you have already breached. A service that is burning 3× its error budget in week 1 of a 30-day window will breach even if its current attainment looks fine.

- Instrument: Datadog SLO burn rate alerts, or equivalent in New Relic / Honeycomb
- Watch for: any service burning >2× budget; more than one service in degraded state simultaneously

**P0/P1 incident count and MTTR (trailing 30 days, week-over-week trend)**

Not just the count — the trend. Two P1s per month is fine if it was four last month and two the month before. Two P1s per month is alarming if it was zero for the prior two quarters.

- Watch for: MTTR trending up over multiple weeks (the team is getting slower at recovery, not faster); incident count that stops declining after a remediation effort (the fix was cosmetic, not structural)

**Escaped defect rate (quarterly)**

The dashboard refreshes this monthly, not weekly, because it requires Jira classification discipline that does not happen in real time. Flag it at the dashboard level when the monthly update shows it crossing 30% — that is the threshold at which testing and staging fidelity have materially broken down.

**Open critical vulnerabilities (count, aging)**

CVEs at critical/high severity with no assigned owner, or with an owner but no action in 14+ days. This one tends to be invisible until it is an incident. One line on the dashboard — count and average age. If the count is growing or the average age is above 30 days, it warrants direct attention.

- Instrument: Snyk, Dependabot, or your security scanning tool of choice — the dashboard pulls the count and age, not the full report

---

### Domain 3: People and Org Health

**Question it answers:** Is the team sustainable?

This is the domain most HoE dashboards omit and the one that causes the most expensive surprises. People signals lead delivery signals by 60–90 days. By the time delivery metrics degrade, the attrition or overload that caused it has usually been visible in the people data for months.

**Capacity vs. plan (headcount and open requisitions)**

Actual engineers on staff versus planned headcount, broken down by team. Include contractor ratio if contractors carry production load. Also: open requisitions and their age. A req that has been open for 90 days without a hire is a real capacity gap, not a future one.

- Watch for: teams operating at >20% below planned headcount for more than 60 days — delivery risk accumulates silently before it shows up in DORA metrics

**On-call incident load per engineer (P2+, per week, trailing 4 weeks)**

Already in the scorecard but worth tracking at weekly cadence on the operational dashboard. When this crosses 6 pages/engineer/week sustained for more than 3 weeks, attrition risk becomes measurable. The distribution matters as much as the average — one engineer absorbing 80% of pages while others see none is a structural problem the average hides.

**Unplanned leave (trailing 30 days)**

Not sick days specifically — abrupt leave patterns that correlate with burnout or disengagement. Most HRIS systems can surface this. A team where 3 of 6 engineers took unplanned leave in the same two-week window is a signal worth a conversation with the manager, not an accusation but a check-in.

**1:1 completion rate by manager (rolling 4 weeks)**

Managers who stop having regular 1:1s are usually underwater — on-call, context-switching, or managing up too much. A manager's 1:1 cadence is one of the cheapest leading indicators of team cohesion available. Most people-management tools (Lattice, Culture Amp, even calendar analytics) can surface this.

- Target: >90% of scheduled 1:1s held over a rolling 4-week window
- Watch for: a manager whose 1:1 rate drops below 70% for two consecutive weeks

**Engineers with no PTO logged in 90+ days**

Systematically invisible until someone burns out or resigns. Run a query against HRIS at the start of each quarter. This is not a performance signal — it is a health signal. Flag individuals to their manager for a check-in conversation.

---

### Domain 4: Strategy Execution

**Question it answers:** Are we building the right things and making progress?

**OKR progress by team (% complete at current week of quarter vs. expected)**

The OKR framework document covers goal-setting in depth — this is the dashboard signal: each team's progress as a percentage of where it should be at this point in the quarter. A team that is 40% complete at week 8 of a 13-week quarter is behind pace; a team that is 90% complete at week 4 either sand-bagged or is about to declare victory on something that was not ambitious enough.

- Watch for: two or more teams simultaneously behind pace (suggests a cross-cutting dependency or a planning assumption that was wrong); a team consistently hitting 100% before week 10 (goals need to be reset)

**Distraction ratio (% of sprint capacity on unplanned work, trailing 4 weeks)**

Unplanned work is not inherently bad — incident response, urgent stakeholder requests, and production support are real work. The signal is the ratio. When more than 40% of sprint capacity is reactive for two or more consecutive sprints, the team has effectively lost the ability to execute on planned strategy.

- Target: <25% unplanned work as a sustained average
- Warning: >40% for two consecutive sprints; immediate attention: >60% for any single sprint

**Dependency blockers (count, owner, age)**

Cross-team dependencies that are blocking work. Track count, which team is blocked, which team is the blocker, and how long the block has been open. A blocker that has been open for three weeks without escalation is not being managed — it is being tolerated.

- Instrument: Jira blocker links with a dependency-tracking label, or a dedicated RAID log (see program management playbooks)
- Watch for: the same team showing up repeatedly as a blocker — this is a structural capacity or prioritization issue, not a one-time dependency problem

**Initiative milestone health (RAG by initiative)**

For the two to four major engineering initiatives in flight at any given time: are they on track, at risk, or off track? Not the detailed project plan — just the RAG status and the one-line reason. This belongs on the dashboard as a prompt to check the detailed tracking if anything is amber or red, not as a replacement for it.

---

### Domain 5: Cost and Capacity

**Question it answers:** Are we spending efficiently and do we have the capacity we think we have?

**Cloud spend vs. budget (trailing 30 days, by service/team)**

Not a FinOps replacement — that requires dedicated tooling and expertise. The dashboard signal is: are we on track against monthly budget, and which team or service is the outlier? A service that jumped 40% in cloud spend in a single week is either handling significantly more traffic (good, but check if it was expected) or has a misconfiguration (infrastructure leak, forgotten test environment, cost regression from a code change).

- Instrument: AWS Cost Explorer / GCP Billing / Azure Cost Management, with tag-based attribution by team or service
- Watch for: week-over-week spend increases >20% on any service; total spend trending ahead of monthly budget by week 2 of the month

**Cost per deployment (trailing 30 days)**

Unit economics for the delivery pipeline. Divide total CI/CD infrastructure cost by deployment count. This catches pipeline inefficiencies that aggregate spend does not surface — long build times, large artifact sizes, redundant test runs. If cost per deployment is rising while deployment frequency is flat, something in the pipeline is getting more expensive.

**Open headcount reqs and time-to-fill**

Already in Domain 3 as a capacity signal; it appears here as a cost signal because unfilled reqs typically mean the work is being absorbed by existing staff (overload risk) or deferred (strategic risk). Time-to-fill >60 days on a critical-path hire is a flag for the recruiting pipeline, not just the hiring manager.

---

## Dashboard Format

The dashboard should live in a system that refreshes automatically — not a slide deck or a manually updated spreadsheet. Grafana, Datadog dashboards, or a simple internal tool are all appropriate. The format that works:

**Top-level view:** Five rows, one per domain. Each row shows a RAG status (green/amber/red) and a one-line summary. If all five are green, the review takes 90 seconds.

**Drill-down per domain:** Clicking into an amber or red domain shows the underlying metrics for that domain — the specific team, service, or signal that triggered the status.

**Weekly diff:** Show the delta from last week, not just the current value. A metric that is amber but improving is a different conversation than one that is amber and has been declining for four weeks.

**Owner per metric:** Every metric has a named owner. The owner is responsible for the trend, not just reporting the number.

---

## RAG Status Definitions

Consistent RAG definitions prevent the dashboard from becoming a negotiation.

| Status | Meaning | Default response |
|--------|---------|-----------------|
| Green | At target or within 10% of target, trend flat or improving | No action required |
| Amber | >10% off target, or at target but trend declining for 2+ weeks | Owner is accountable for a plan within one week |
| Red | >25% off target, or amber for 2+ consecutive weeks without an improving trend | Escalation to HoE for direct attention; plan due in 48 hours |

The most common dashboard failure mode is treating amber as "informational." Amber that does not trigger an owner response becomes red; red that does not trigger HoE attention becomes an incident, a miss, or an attrition event.

---

## What to Exclude

The same exclusion principles from the scorecard apply here, with one addition:

**Individual engineer metrics.** The dashboard operates at team and system resolution, never individual contributor resolution. Using the dashboard to monitor individual output is both methodologically unsound (the metrics do not have the fidelity for that) and corrosive to the trust that makes the people signals useful. If you need to evaluate an individual's output, that belongs in a performance conversation with their manager, not on a dashboard.

**Metrics that require manual input.** If a metric cannot be pulled from a system, it will not stay current. A manually updated dashboard decays within two weeks — the cost of keeping it fresh exceeds the value it provides. Every metric on the operational dashboard must have an automated data source.

**Engagement survey scores.** Useful in a quarterly HR review; too noisy and infrequent for a weekly operational signal.

---

## Relationship to the Engineering Health Scorecard

The scorecard and the dashboard share some underlying data sources but serve different purposes and audiences.

| | Operational Dashboard | Engineering Health Scorecard |
|---|---|---|
| **Audience** | Head of Engineering | VP Eng, CTO, board, M&A diligence |
| **Cadence** | Weekly (reviewed Monday) | Monthly (full review); Quarterly (narrative) |
| **Resolution** | Per-team, per-service | Org-level with domain rollups |
| **Format** | RAG + drill-down | Narrative + tables |
| **Purpose** | Early warning; catch drift before it becomes a miss | Credibility artifact; convert org performance into a defensible story |
| **Maintenance** | Automated; no curation | Requires owner narrative per metric |

The scorecard is built from the dashboard. When a domain is amber or red on the dashboard for 4+ weeks, it surfaces as a scorecard item at the monthly or quarterly cadence with a narrative attached. The dashboard catches it; the scorecard tells the story.

---

## Further Reading

| Topic | Resource |
|---|---|
| DORA metrics and delivery performance | Forsgren, Humble & Kim, *Accelerate* (IT Revolution Press, 2018) |
| SLO-based reliability | Google SRE Book — [Service Level Objectives](https://sre.google/sre-book/service-level-objectives/) |
| Engineering org health and team sustainability | Skelton & Pais, *Team Topologies* (IT Revolution Press, 2019) |
