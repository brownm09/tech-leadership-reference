# Managing Up and Across Playbook

## Leadership Context

Engineering leaders who are technically strong and operationally disciplined often stall at the senior manager or director level for one reason: they treat the relationships above and beside them as administrative overhead rather than as the actual work. Managing up is not politics — it is how you get your team the resources, air cover, and strategic alignment they need to operate. Managing across is not coordination — it is how you prevent your org from becoming a local optimization machine that creates drag for every team that depends on you.

Both relationships are skills. They have mechanics, failure modes, and a practice regimen. This playbook covers both.

## Background and Motivation

This framework was developed from the cross-functional program work at ActBlue Technical Services (2022–2025), where I owned a multi-year PCI environment deprecation program that required sustained alignment across engineering, legal, finance, product, and account operations — and simultaneous management of a platform directorate with six teams and approximately fifty engineers. The challenges of managing up (maintaining executive visibility and trust through a multi-year program with shifting compliance deadlines) and across (coordinating four to five of those teams on the PCI program directly, along with external audit timelines and non-technical stakeholders with different incentives) were parallel and interdependent. Mismanaging either created cascading problems in the other.

---

## Part 1: Managing Up

### The Trust Account Model

Your relationship with your skip-level (the person above your manager) and with your manager is a trust account. You make deposits through: delivering on commitments, surfacing problems early rather than late, communicating clearly without over-engineering every message, and making their job easier. You make withdrawals every time they are surprised by something they should have known, every time they have to re-explain your team's work to someone else because they did not fully understand it, and every time a decision that should have stayed at your level lands on their desk.

The goal is not a high balance — it is a steady deposit habit. Most of the withdrawals in a healthy relationship are small and expected. A single large surprise withdrawal can take months of deposits to recover from.

### Building the Exec Relationship

Do not wait for formal check-ins to build the relationship with your skip. The formal check-in is where you report on things; the relationship is built in the informal moments — a brief Slack message when something important lands, a 5-minute walk between meetings, a direct note after a difficult decision.

Three practices that build exec relationships reliably:

1. **Lead with the conclusion.** Executives do not have time to reconstruct your reasoning. State the conclusion first, then offer the reasoning. "We should delay the migration by one quarter. Here's why: [two sentences]." Not: "So I've been looking at the migration timeline and there are a few factors to consider..."

2. **Name trade-offs, not problems.** A problem requires someone else to solve it. A trade-off requires a decision. "The payment processor migration is on track, but accelerating it to meet Q2 creates risk to the fraud detection refactor in the same sprint. I can handle either, but not both at the original timeline. Which do you want me to protect?" This frames you as the person managing the tension, not surfacing it.

   In practice: during the PCI deprecation at ActBlue, an external compliance deadline shifted six weeks mid-program. The framing to the VP of Engineering was not "the timeline changed" — it was: "We can hit the new compliance date, but it means the payment processor integration slips to Q3. The compliance date is non-negotiable from a regulatory standpoint; the integration date is a business decision. Which risk do you want me to protect?" That framing put the decision in the right hands and produced a resolution in the same conversation.

3. **Give them something to say.** Your skip will talk about your team's work in contexts you are not in. Give them the two or three talking points that make your team's work legible to that audience. If they have to reconstruct your team's value proposition from memory, the version they communicate will not be the version you would choose.

### Skip-Level Relationships

Your skip-level is not an ally or an authority — they are a stakeholder. Manage the relationship accordingly:

- **Request time when you have something to bring**, not just to check in. Skips who routinely have 30-minute 1:1s with every sub-org leader will have those compressed or canceled. Come with a question, a decision that needs them, or an update that is genuinely useful to them.
- **Do not go around your manager.** If you have a concern about your manager, address it with your manager first. Going directly to the skip without exhausting the direct conversation is a relationship withdrawal against both accounts simultaneously.
- **Know their priorities.** What is the skip-level worried about right now? What is the board asking about? What metric are they under pressure on? Your team's work is more legible when it is framed in terms of what is already on their mind.

### Absorb vs. Escalate

Most decisions that could go up should not. The escalation test is: does this decision require authority, relationships, or information that I genuinely do not have? If yes, escalate. If it requires time, discomfort, or a hard trade-off that I am trying to avoid, do not escalate — absorb it.

Common decisions that should be absorbed, not escalated:
- Whether to delay a feature to address technical debt (you have the information to make this call)
- Whether to extend a sprint due to unexpected complexity (same)
- Personnel performance issues that are in early stages (handle them at your level until they require formal process)
- Cross-team prioritization conflicts that you can resolve directly with the other manager

Common decisions that warrant escalation:
- A commitment that will be missed, when the miss affects a stakeholder relationship your manager owns
- A personnel situation where HR process is required
- A strategic trade-off where two or more senior leaders' priorities are in direct conflict and only someone above them can adjudicate
- Any decision that, if wrong, would be irreversible and whose consequences extend beyond your scope

When you escalate, escalate with a recommendation. "I don't know what to do here" is a request for your manager to do your job. "Here's what I recommend, here's the alternative, here's the risk of each, and here's my ask" is an escalation that respects their time and positions you correctly.

### Protecting Your Team's Capacity

One of the clearest signals of a leader who manages up well is that their team does not feel the chaos from above. Cross-functional requests, executive whims, and urgent-but-not-important asks arrive constantly. Your job is to be the filter.

Practical mechanics:
- **Run a weekly intake process.** New requests go into a queue rather than directly to the team. Review the queue weekly against the roadmap and capacity. Most of the urgent requests will have resolved themselves by the time you review them.
- **Name the trade-off explicitly.** When a new request genuinely must be prioritized, do not just absorb it silently — communicate what moves. "We can take this on, but it means the fraud detection refactor moves to Q3. Is that the trade you want?" This keeps leadership accountable for the trade-offs they are implicitly making when they add scope.
- **Do not let your team be the cushion.** Sprint extensions, weekend work, and heroics are sometimes necessary. They are never a substitute for adequate scope management at the leadership level.

---

## Part 2: Managing Across

### The Peer Manager Relationship

Your peer engineering managers are not competitors, but they are not automatically allies either. Peer relationships are built on reciprocity: you help them when they need it, you are honest with them when their work creates problems for you, and you do not go over or around them in a way that makes them look bad to their team.

Common failure modes in peer manager relationships:
- **Scope creep without negotiation.** Your team starts owning work that bleeds into another team's domain without an explicit conversation.
- **Public disagreement.** Raising concerns about a peer's team or decisions in a forum where their team is present, rather than directly.
- **Asymmetric information.** Being in a planning conversation that affects a peer's team without including them, then presenting the decision as fait accompli.

Practices that maintain healthy peer relationships:
- **Direct channels first.** If a peer's team creates a problem for yours, message the manager directly before raising it in a shared forum. This is almost always faster and preserves the relationship.
- **Shared language for cross-team issues.** When two teams have a dependency or a conflict, agree on a shared framing before it goes to leadership. "Team A needs X from Team B by Y date for Z reason" is more useful than two competing stories about the same situation.
- **Credit generously.** When cross-team work goes well, make the other manager's contribution visible to their manager and your shared skip. This costs you nothing and builds significant trust.

### Cross-Functional Partnership: Product

The engineering/product relationship is the most consequential cross-functional relationship in most tech orgs, and the most frequently damaged.

The structural tension: product manages outcomes (what gets built and when), engineering manages execution (how and at what cost). In a healthy relationship, these two functions inform each other continuously. In an unhealthy one, product delivers requirements and engineering delivers dates, and both parties are surprised when neither is met.

Mechanics that work:
- **Joint roadmap ownership.** Engineering should have a seat in roadmap planning, not just roadmap review. The cost of a feature is not just the sprint points — it is the opportunity cost, the technical debt incurred, and the on-call implications. Engineering is the only party who can accurately represent those costs.
- **Early technical input on specs.** When product is writing a PRD, engineering input during the spec phase (not after) prevents the "we built the wrong thing" problem. A two-hour technical consultation at spec time is worth ten sprint days of rework.
- **No surprises on timeline.** When a commitment is at risk, tell the product manager before it slips, not after. The earlier the warning, the more options they have to adjust scope, timeline, or priority.

### Cross-Functional Partnership: Legal and Compliance

Legal and compliance are often treated as obstacles. They are more accurately described as early-warning systems who have organizational authority to block things and prefer not to.

The key insight: legal and compliance teams almost always have a workable path — they just need to understand what you are trying to do and why before they can help you find it. Going to them with a completed design and asking for approval is a losing approach. Going to them at the problem statement stage and asking "what constraints should we design around?" produces a very different conversation.

For compliance-adjacent programs (PCI, GDPR, SOC 2): own the relationship, not just the deliverables. Know your compliance lead by name. Know their concerns and timeline pressures. When the audit is approaching, they should be calling you, not the other way around.

In practice: during the PCI deprecation at ActBlue, rather than arriving at the compliance team with a completed scope reduction design and asking for sign-off, the conversation started with: "We are planning to decommission the legacy cardholder data environment. What constraints do we need to design around?" That question produced a requirement about data retention timing that would have required costly late-stage rework if surfaced at the approval stage instead of the design stage.

### Cross-Functional Partnership: Finance

Finance cares about two things: accuracy and surprise avoidance. Your job in the finance relationship is to give them accurate numbers and early warning when those numbers are changing.

The headcount request, the infrastructure cost estimate, and the vendor contract renewal are all documents that finance will scrutinize. The ones that survive scrutiny have: a clear business case (what problem does this solve), a cost model (not just the sticker price, but the total cost of the decision), and a plan for what happens if the approved number is not available.

Do not go to finance with a number you made up and hope they do not ask questions. They will ask questions. Go with a range, a methodology, and an acknowledgment of the assumptions baked in.

### Dependency Navigation

When your team has a dependency on another team (or another team has a dependency on yours), the default failure mode is that both teams treat the dependency as the other team's problem. The dependency sits unresolved until it blocks someone, at which point both teams are surprised and resentful.

The practice:
- **Name the dependency in writing at the time it is created.** Who owns what, by when, and what blocks it. Not in a meeting — in a RAID log, a ticket, or a shared document that both managers can see.
- **Weekly dependency check.** During whatever stand-up or sync runs across the teams, explicitly review blockers and dependencies. Treat an unresolved dependency as a blocker, not a nice-to-have follow-up.
- **Escalate early, not late.** If a dependency is not moving and the deadline is real, escalate before the deadline — not after it. The earlier the escalation, the more options exist for resolution.

### When to Escalate a Cross-Team Issue vs. Solve It Yourself

Escalate when:
- The other manager and you have reached an impasse and the decision requires authority above both of you
- The timeline impact of the unresolved issue will affect a commitment that leadership owns
- The issue involves a resource or priority that two orgs are competing for and only one person can adjudicate

Solve it yourself when:
- You have not yet tried a direct conversation with the peer manager
- The issue can be resolved by changing scope, timeline, or priority at the team level without affecting shared commitments
- Escalation would involve burning relationship capital with shared leadership for a problem that is solvable locally

Most cross-team issues that get escalated did not need to be. Most that were never escalated needed to be.

---

## Anti-Patterns

**The Status Theater Update.** Sending detailed weekly updates to your manager that communicate a lot of information and very little understanding. Updates should be curated: here is what matters, here is what changed, here is what I need.

**Managing Down Instead of Up.** Spending all available energy on team-facing work and treating the exec relationship as secondary. This is comfortable and feels productive; it is also a cap on how much your team can accomplish, because the resourcing, prioritization, and air cover they need comes from the relationship you are not investing in.

**The Peer Escalation Bypass.** Going to your shared manager about a peer's team before trying to resolve it directly. Even if your concern is valid, this approach damages the peer relationship and signals to leadership that you cannot handle lateral problems.

**Alignment Theater.** Scheduling cross-functional syncs that do not surface the real tensions. The real tensions get resolved in the hallway (or Slack) by the people who are actually willing to name them, and the sync becomes a performance of alignment. Run syncs where the agenda explicitly includes: what is blocking us, what do we disagree about, what needs a decision.

**The Invisible Trade-Off.** Absorbing a cross-functional request without naming what moved to accommodate it. When you absorb scope silently, you give leadership no information about the cost, and the same pattern repeats. Name the trade-off every time.
