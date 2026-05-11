# Blameless Post-Mortem Framework

## Leadership Context

The post-mortem is the most widely misused artifact in engineering operations. In principle, it is a structured process for learning from an incident in a way that makes the next incident less likely. In practice, it is often used as a documentation checkbox — something that gets written after a significant outage and filed in a wiki where nobody reads it, or as a mechanism for assigning responsibility in a way that is politically palatable but organizationally counterproductive.

This document is distinct from the [Disaster Recovery Fire Drill Template](../disaster-recovery/fire-drill-template.md), which covers proactive resilience testing. The post-mortem is a reactive artifact — it is written after a real incident has occurred. The fire drill produces evidence that systems work as designed. The post-mortem produces learning when they did not.

The "blameless" framing requires more precision than it usually receives. Blameless does not mean consequence-free. It means the analysis is directed at systems, processes, and conditions rather than at individuals. An engineer who made an error that contributed to an incident did so within a system that allowed that error to happen. The post-mortem's job is to understand the system, not to identify the human as the root cause. If the human is the cause, the system created the conditions in which the human cause was possible — and that system is what the post-mortem should analyze.

## Background and Motivation

This framework is grounded in incident response operations at ActBlue Technical Services (2022–2025), where I was responsible for post-mortem culture across a platform directorate handling high-availability payments infrastructure. The payments context — PCI compliance, SLA obligations, and voter data sensitivity — created genuine organizational consequences for how incidents were analyzed and documented. The blameless framing was not aspirational in that context; it was operationally necessary. Attribution-focused post-mortems in a regulated environment produce two outcomes: engineers who are afraid to make changes, and documentation that obscures more than it reveals.

The five-part structure in this framework is an adaptation of the Google SRE post-mortem format[^1], modified for team-level use in contexts without a dedicated SRE function.

## When to Use This

| Trigger | What This Framework Provides |
|---|---|
| Any incident that crossed the severity threshold for a post-mortem (typically: customer impact > N minutes, SLO breach, data loss, or security event) | A structured document and facilitation process |
| Recurring incidents of the same type | A root cause framework for identifying systemic causes rather than treating recurrences as isolated events |
| Near-misses where no impact occurred but failure was likely | An abbreviated post-mortem process (the "postmortem-lite" variant) |
| Establishing a post-mortem culture in a team that doesn't have one | A starting protocol and facilitation guide |

---

## Part 1: The Five-Part Post-Mortem Document

The post-mortem document is written before the post-mortem meeting, not during it. The on-call owner (or whoever has the best incident context) drafts the document as a starting point. The meeting is for review, correction, and collaborative root cause analysis — not for first-draft synthesis.

### 1. Incident Summary

A three-sentence summary that answers: what happened, when it happened, and who was affected. This section is read by people who were not involved in the incident. It should be intelligible to a non-technical stakeholder.

```
On [date], [system or service] experienced [impact description] from [start time] to [end time].
Approximately [N users / $X revenue / N% of requests] were affected.
The incident was resolved when [brief resolution description].
```

### 2. Impact Assessment

The specific, quantified impact. This is not the place for hedging language ("the impact was potentially significant"). If the impact is unknown, say so and why.

| Dimension | Impact |
|---|---|
| Duration | [Start time to end time, total minutes] |
| User impact | [Number of affected users, or % of traffic] |
| Revenue impact | [$X if calculable, or "not calculable" with reason] |
| SLO impact | [Which SLO was breached, by how much] |
| Data impact | [Any data loss, corruption, or privacy implications] |
| Secondary systems | [Other systems affected by the incident or its remediation] |

### 3. Incident Timeline

A timestamped log of what happened, in chronological order. The timeline is not a narrative — it is a factual record. Each entry should have a timestamp, an actor (person or system), and an action or observation.

Format:
```
HH:MM  [Actor]   [Action or observation]
14:32  Alerting  PagerDuty alert: payment service error rate > 5% (threshold: 1%)
14:34  @alice    Acknowledged alert, began investigation
14:41  @alice    Identified elevated error rate correlating with recent deploy
14:43  @bob      Initiated rollback of deployment abc123
14:51  @bob      Rollback complete; error rate returning to baseline
15:04  Alerting  Alert resolved; error rate < 0.5%
```

The timeline should be assembled from alert logs, Slack history, runbook entries, and on-call notes — not from memory after the fact. Encourage on-call engineers to maintain a running timeline during the incident.

### 4. Root Cause Analysis

This is the core analytical work of the post-mortem. Two methodologies apply here; use one or both depending on the complexity of the incident.

**The 5 Whys.** Iterative questioning that traces a symptom to its systemic root.

Example:
```
Why did the payment service return errors?
  → The database connection pool was exhausted.
Why was the connection pool exhausted?
  → A recent deploy changed the query pattern in a way that held connections longer.
Why did the query pattern change reach production?
  → The change was tested on a small dataset that didn't reproduce the connection pressure.
Why was the test dataset small?
  → We don't have a process for synthetic load testing before payment service deploys.
Why don't we have that process?
  → It was flagged as a gap eighteen months ago and never prioritized.
Root cause: Absence of load testing protocol for payment service deploys.
```

**Contributing Factors.** A list of conditions that allowed the incident to occur or made it worse — distinct from the root cause, which is the primary cause.

```
Contributing factors:
- Deploy was on Friday afternoon, reducing availability of experienced on-call responders
- Monitoring alert threshold was set to 5% error rate; earlier detection at 1% might have
  shortened the impact window
- Runbook for this service had not been updated in six months and did not reflect the
  current rollback procedure
```

### 5. Action Items

The action items are the primary output of the post-mortem. A post-mortem without action items is a documentation exercise. A post-mortem with action items that are never completed is worse — it produces a false sense of closure.

Each action item must have:
- A specific, testable description of what will change
- An owner (a single person, not a team)
- A due date
- A severity (P1: address before next deploy; P2: address within 30 days; P3: backlog)

| Action Item | Owner | Due | Priority |
|---|---|---|---|
| Add load test to payment service deploy pipeline | @alice | [date] | P1 |
| Update on-call runbook with current rollback procedure | @bob | [date] | P2 |
| Review and lower alert thresholds for payment error rate | @carol | [date] | P2 |
| Conduct post-mortem retro on Friday deploy policy | @manager | [date] | P3 |

---

## Part 2: Blameless Facilitation

The post-mortem meeting is where the blameless framing is most at risk. Even in organizations with explicit blameless policies, the group dynamics of a post-mortem meeting can produce attribution — an engineer who is embarrassed by their role in an incident may volunteer blame preemptively, and other participants may accept it because it provides a clean narrative.

The facilitator's job is to keep the analysis directed at systems rather than people.

### Facilitation Principles

**Separate the person from the action.** When an engineer says "I made a mistake," redirect: "What in the system made that action possible? What would have had to be true for a different engineer to avoid the same outcome?"

**Name contributing factors, not culpable individuals.** When the timeline shows that an individual made a decision that contributed to the incident, the post-mortem records the decision and the context — not the individual's name as the cause. "A deploy was initiated without load testing" is a system observation. "Alice initiated the deploy without load testing" is an attribution.

**Ask what would have to change, not what should have been done differently.** "What would have to be true for this incident to not have happened?" produces system changes. "What should [person] have done differently?" produces individual accountability without systemic learning.

**Read the contributing factors list aloud and ask: did we miss any?** This prevents the post-mortem from closing around a single root cause when the reality is multi-causal. Complex systems fail through the combination of small gaps, not a single catastrophic error.

### The Facilitator Role

The facilitator should not be the on-call owner from the incident. The on-call owner is a primary contributor to the timeline and root cause analysis; they should not also be managing the meeting dynamics. Assign facilitation to a manager, a senior IC from an adjacent team, or a designated post-mortem facilitator if your team has one.

**Facilitator responsibilities:**
- Open the meeting by reading the blameless principle aloud (see template)
- Drive through each section of the post-mortem document for review and correction
- Redirect attribution language toward system analysis
- Ensure every action item has an owner and a date before the meeting ends
- Close the meeting with explicit acknowledgment of the work the on-call team did

---

## Part 3: The Action Item Tracking Problem

Post-mortem action items are the highest-mortality artifact in engineering operations. They are created with good intentions, assigned to owners, and then deprioritized in the next sprint, the one after that, and the one after that. Months later, the same incident type recurs and the post-mortem produces the same action items.

### Why Action Items Die

- They are in a wiki, not a ticket system. Engineers do not live in wikis.
- They are assigned to individuals who do not have the authority to prioritize them.
- They require cross-team coordination that nobody owns.
- The sprint is full and the post-mortem AIs have no deadline enforcement.

### What Actually Works

**Create tickets immediately.** At the end of the post-mortem meeting, post-mortem action items become tickets in whatever system the team uses for sprint work. Not wiki tasks, not a checkbox in the post-mortem doc — actual backlog items.

**P1 items block the next deploy.** If an action item is classified P1, it should block the next production deploy for the service that had the incident. This is a strong policy that will generate pushback. It is also the only mechanism that reliably ensures critical post-mortem AIs get completed.

**Manager reviews open post-mortem AIs monthly.** The manager — not the on-call owner — owns the review of outstanding post-mortem action items. This removes the conflict of interest (the on-call owner may be reluctant to surface that their AIs are not complete) and signals organizational commitment to the follow-through.

**The aging post-mortem is a signal.** If a post-mortem has open P2 action items more than sixty days past their due date, something is wrong with the prioritization process, the ownership assignment, or the clarity of what "done" means. Treat aged AIs as a process problem, not an individual accountability problem.

---

## Part 4: The Post-Mortem-Lite for Near-Misses

Not every incident that warrants learning warrants the full five-part process. Near-misses — situations where no customer impact occurred but failure was close — produce valuable learning at lower cost than a full post-mortem.

The post-mortem-lite covers three questions:

1. **What almost happened?** (Brief incident summary — one to two sentences)
2. **Why didn't it happen?** (What prevented the customer impact — the catch or the luck)
3. **What one thing would make this more robust?** (Single action item with owner and date)

Run the post-mortem-lite as a short Slack thread or a fifteen-minute sync, not a formal meeting. The value is capturing the near-miss signal while it is fresh — not producing a comprehensive analysis.

---

## Templates

### Post-Mortem Document Skeleton

```markdown
# Post-Mortem: [Incident Name or ID]

**Date:** [Date of incident]
**Severity:** [P0 / P1 / P2]
**Status:** [Draft / In Review / Complete]
**Document owner:** [Name of primary author]
**Review date:** [Date of post-mortem meeting]

---

## Incident Summary

[3 sentences: what happened, when, who was affected]

---

## Impact Assessment

| Dimension | Impact |
|---|---|
| Duration | |
| User impact | |
| Revenue impact | |
| SLO impact | |
| Data impact | |
| Secondary systems | |

---

## Incident Timeline

| Time | Actor | Action / Observation |
|---|---|---|
| | | |

---

## Root Cause Analysis

### Primary Root Cause
[One sentence: the systemic condition that enabled this incident]

### 5 Whys
Why [symptom]?
  →
Why [answer]?
  →
Why [answer]?
  →
[Continue until you reach a systemic cause]
Root cause: [statement]

### Contributing Factors
- [Condition that allowed the incident to occur or worsen]
- [Condition]
- [Condition]

---

## Action Items

| Action Item | Owner | Due Date | Priority |
|---|---|---|---|
| | | | |

---

## Lessons Learned

[Optional: 1–3 generalizable observations from this incident — something
other teams or future post-mortems should know]
```

### Facilitator Opening Statement

```
Before we start, I want to name the purpose of this meeting. We're here to
understand what happened in a way that makes the next incident less likely.
We're analyzing systems, not people. If you made a decision that contributed
to this incident, we want to understand what the system made possible —
not assign responsibility for the outcome.

We will leave this meeting with: a shared understanding of the timeline,
an agreed root cause, and action items with owners and dates.

Let's start with the timeline.
```

### AI Tracking Table (for sprint system integration)

```
Post-mortem AI tracking table — created [date], incident [ID]

| Ticket | Action Item | Owner | Due | Priority | Status |
|---|---|---|---|---|---|
| [JIRA-XXX] | [Description] | [Name] | [Date] | P1/P2/P3 | Open |
```

---

## Anti-Patterns

**The Shame Spiral.**
A post-mortem meeting where the on-call owner spends the first ten minutes apologizing and the rest of the meeting defending their decisions. The facilitator's job is to interrupt this pattern before it begins. An opening statement that names the blameless frame explicitly reduces the probability of the shame spiral.

**The Single Point of Failure Attribution.**
A post-mortem that concludes "the engineer should have tested this before deploying" and stops there. This is almost never the root cause — it is the proximate cause. The root cause is whatever in the system made inadequate testing possible: no automated test gate, insufficient review coverage, no deploy checklist, inadequate staging environment fidelity. Stop at the person only when you cannot find the system.

**The Orphaned Action Item.**
A post-mortem action item without a single named owner. Teams, committees, and "we" do not own action items. If the action requires coordination across multiple owners, one of them is the lead and the others are contributors.

**The Filed and Forgotten.**
A post-mortem document written, reviewed, and posted to a wiki — with no mechanism for ensuring the action items are ever completed. The post-mortem is only as valuable as the systemic changes it produces.

**The Retrospective Creep.**
A post-mortem meeting that evolves into a general retrospective on engineering practices, team dynamics, or historical grievances. Keep the post-mortem scoped to the specific incident. A good post-mortem surfaces opportunities for improvement that belong in the team's regular retro process — it does not try to run that retro in the same session.

[^1]: Beyer, B., Jones, C., Petoff, J., & Murphy, N. R. (2016). *Site Reliability Engineering: How Google Runs Production Systems*. O'Reilly Media. Chapter 15 covers the post-mortem culture and format that this framework adapts.
