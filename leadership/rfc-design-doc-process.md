# RFC and Design Doc Process

## Leadership Context

Every technical team makes architectural decisions — about service boundaries, data models, infrastructure choices, integration patterns, and operational practices. In small teams, these decisions happen in conversation and are documented after the fact, if at all. In larger teams or across teams, undocumented decisions are a recurring source of friction: engineers make incompatible assumptions, the person who made the original decision leaves, or a new engineer re-investigates and re-proposes a decision that was already made and for good reason.

The RFC (Request for Comments) process is the primary mechanism for making significant technical decisions in a structured, reviewable, and documented way. It is not a bureaucratic checkpoint — it is a forcing function for articulating a decision's rationale in writing before implementing it, making the decision visible to the people it will affect, and creating a searchable record that future engineers can find when they encounter the same question.

The relationship between an RFC and an Architectural Decision Record (ADR) is frequently confused. An RFC is a proposal: it describes a decision that has not yet been made, invites feedback, and documents the options considered. An ADR is a record: it captures a decision that has already been made, along with the context, rationale, and consequences. Not every RFC produces an ADR — some RFCs conclude with no change to the existing approach. Every significant ADR should be preceded by an RFC-equivalent process, though the process may be lighter for smaller decisions.

## Background and Motivation

This framework is grounded in technical decision-making across multiple contexts at ActBlue Technical Services (2022–2025) and Community Tech Alliance (2025–2026). At ActBlue, the scale of the platform directorate — six teams, significant cross-system dependencies, regulated infrastructure — required structured decision-making for architectural changes. The payment processor migration (Stripe), Heroku-to-Kubernetes migration, and PCI environment deprecation all involved decisions where the RFC process provided structure and created organizational alignment that ad-hoc decision-making would not have produced.

At CTA, the challenge was different: establishing a lightweight RFC process in a smaller team where the instinct was to decide in Slack and move on. The friction the RFC process resolves is not primarily size-related — it is documentation and asynchronous review discipline, which matters even in a team of five.

The RFC format in this document is adapted from the Rust RFC process and from the patterns described by Will Larson in his writing on engineering decision-making.[^1][^2]

## When to Use This

| Trigger | RFC Required? | Format |
|---|---|---|
| Choosing a new external service or infrastructure component | Yes | Full RFC |
| Changing a service's API contract in a backward-incompatible way | Yes | Full RFC |
| Introducing a new architectural pattern across multiple services | Yes | Full RFC |
| Changing an existing pattern within a single service | Maybe | Abbreviated RFC or ADR-only |
| Choosing between two implementation approaches within a ticket | No | PR description is sufficient |
| Documenting a decision already made without significant options analysis | No | ADR directly |

The key threshold question: **would a reasonable engineer on an adjacent team want to know about this decision before it is implemented?** If yes, an RFC is warranted. If the decision affects only the team making it and has limited surface area for future engineers, document the decision as an ADR without a full RFC process.

---

## Part 1: When an RFC Is Warranted

Beyond the table above, three factors push a decision toward the full RFC process:

**Irreversibility.** Decisions that are hard to undo — schema changes, external API contract commitments, infrastructure migrations — warrant more upfront review than easily-reversible decisions. The asymmetry in the cost of getting these decisions wrong justifies the asymmetry in the process.

**Blast radius.** Decisions that affect multiple teams, multiple services, or customer-facing behavior warrant broader review than decisions contained within a single team's scope. The RFC process creates a forcing function for identifying all the stakeholders who should have input before the decision is made.

**Novelty.** Decisions that establish a new pattern — the first time the team uses a particular data consistency strategy, the first time a new infrastructure component is introduced — warrant documentation of the options considered and the rationale for the choice. Future engineers encountering the same question should be able to find the prior analysis.

---

## Part 2: RFC Lifecycle

### Stage 1: Draft

The author produces a draft RFC using the document skeleton provided below. The draft should be complete enough for substantive review — not a rough sketch, but not a polished document that presents a single option without acknowledging trade-offs.

**Draft criteria:**
- Problem statement is specific and bounded (not "we should improve our database strategy")
- At least two options are presented and evaluated (including "do nothing")
- Author's recommendation is stated with reasoning
- Known risks and open questions are named

The draft is shared with a limited initial audience: the author's manager, the Staff IC if one exists, and one to two engineers from adjacent teams who will be affected. This pre-review surfaces obvious gaps before the RFC is opened to broader comment.

### Stage 2: Open for Comment

The RFC is published to the team or org's designated RFC channel or repository. The review window should be explicit and long enough for async review: a minimum of five business days for team-scoped decisions, ten business days for org-scoped decisions.

**Opening announcement format:**
```
RFC-[number]: [title]
Author: [name]
Review deadline: [date]
Scope: [team / multiple teams / org-wide]
Summary: [one sentence]
Link: [link to document]

I'm looking for feedback specifically on:
- [Question 1]
- [Question 2]
```

During the review window, the author should actively engage with comments — not to defend the proposal, but to clarify ambiguities, acknowledge concerns, and update the document when feedback surfaces gaps.

### Stage 3: Decision

At the end of the review window, the designated decision-maker (typically the author's manager for team-scoped decisions, a Director or VP for org-scoped decisions) makes a decision. The three outcomes:

- **Approve:** The RFC is accepted. Implementation may begin. The decision is recorded as an ADR.
- **Revise:** The RFC requires significant changes before a decision can be made. The author revises and the review window reopens.
- **Decline:** The RFC is not accepted. The "do nothing" option is chosen, or a different option from the RFC is selected. The decision is still recorded as an ADR — why not to change something is as valuable as why to change it.

The decision should be communicated in the same channel where the RFC was published, with a brief rationale. Engineers who commented on an RFC and received no decision communication are likely to comment on future RFCs with lower engagement.

### Stage 4: Decision Record

The RFC's decision is converted to an ADR (Architectural Decision Record). The ADR captures the final decision and its rationale; the RFC document is preserved as the record of the review process and the options considered.

**Relationship between RFC and ADR:**
- The RFC lives in the RFC archive (a directory or wiki section designated for in-progress and completed RFCs)
- The ADR lives in the codebase or repository it applies to (typically `/docs/adr/` or equivalent)
- The ADR should link back to the RFC for the full context; the RFC should link forward to the ADR for the final decision

---

## Part 3: Review Mechanics

### Who Has Blocking Authority vs. Advisory Voice

Not every comment on an RFC has the same weight. Making this explicit prevents the RFC process from becoming a consensus-seeking exercise that stalls on every objection.

**Blocking authority:** The decision-maker (manager or designated approver) has the only true blocking authority. An RFC cannot be blocked by a commenter who raises concerns — it can be blocked by the decision-maker who agrees those concerns are unresolved.

**Technical blocking signals:** A Staff or Principal IC who identifies a technical flaw in the proposed approach has a strong signal. Their concern should be addressed or explicitly acknowledged in the decision rationale. Ignoring a Staff IC's technical objection and approving the RFC anyway without explanation damages the RFC process's credibility.

**Advisory voice:** Everyone else. Advisory comments are valuable input to the decision — they are not veto power. The author and decision-maker should engage with advisory comments seriously, but "I disagree with this approach" from an advisory commenter does not block the decision.

### Async vs. Sync Review

Most RFC review should happen asynchronously. The review window exists to allow people to read, think, and respond on their own schedule — not to create a meeting. Reserve synchronous discussion for:

- RFCs where the problem space is genuinely complex and a conversation would produce better signal than written comments
- RFCs where the comment thread has reached an impasse that a structured conversation could resolve
- RFCs with significant cross-team implications where a joint session prevents misunderstanding

If a synchronous session is held, document the key points of the conversation and add them to the RFC before the review window closes. Engineers who could not attend should be able to read the RFC and understand the full context of the decision.

### The Decision-Maker's Responsibility

The decision-maker's job is not to find consensus. Consensus is valuable when it is natural; forced consensus — where the decision-maker delays the decision until objections go away — produces slow organizations and rewards the most persistent objectors. The decision-maker's job is to:

1. Confirm that the review window was sufficient
2. Confirm that significant objections were addressed or acknowledged
3. Make the decision with the available information
4. Communicate the decision and its rationale

Disagreement after the decision is made is a normal outcome. Engineers who disagree should be able to express that in the ADR's "dissenting views" section, which exists precisely to avoid the false consensus of a document that pretends the decision was unanimous.

---

## Part 4: The RFC-to-ADR Relationship

### What Goes in the RFC vs. the ADR

| Content | RFC | ADR |
|---|---|---|
| Problem statement | Full description | Brief reference |
| Options considered | Full analysis of all options | Summary of options, detail on chosen option |
| Review comments | Yes | No (archived in RFC) |
| Author's recommendation | Yes | N/A |
| Final decision | Decision recorded at end | Primary content |
| Consequences | Options for each alternative | Consequences of the decision as made |
| Dissenting views | Comments section | Explicit "dissenting views" section |
| Link to counterpart | Link to ADR (added after decision) | Link to RFC |

### ADR Decision-Record Footer Block

Add this block to the RFC document after the decision is made:

```markdown
---

## Decision Record

**Decision:** [Approve / Decline / Revise]
**Decision-maker:** [Name and title]
**Date:** [Date of decision]
**ADR:** [Link to the ADR document]

**Rationale:** [2–3 sentences summarizing why this decision was made]

**Significant objections considered:**
- [Objection]: [How it was addressed or why it was acknowledged but not determinative]
```

---

## Templates

### RFC Document Skeleton

```markdown
# RFC-[number]: [Title]

**Author:** [Name]
**Created:** [Date]
**Status:** Draft | In Review | Decided | Superseded
**Review deadline:** [Date]
**Decision-maker:** [Name or role]
**ADR:** [Link — added after decision]

---

## Summary

[One paragraph: the problem, the proposed solution, and why it matters.
Write this last — it should summarize the document, not introduce it.]

---

## Problem Statement

[What problem are we solving? Be specific. Describe the current state and
why it is insufficient. Include evidence where possible: error rates,
performance data, developer time estimates, incident patterns.]

---

## Goals

- [Specific, testable outcome 1]
- [Specific, testable outcome 2]

## Non-Goals

- [What this RFC is explicitly not trying to solve]
- [Scope boundary]

---

## Options Considered

### Option 1: [Name] (Recommended)

[Description of the option]

**Pros:**
- [Pro]
- [Pro]

**Cons:**
- [Con]
- [Con]

**Risk:** [Primary risk of this option]

### Option 2: [Name]

[Description]

**Pros / Cons / Risk** [same format]

### Option 3: Do Nothing

[What happens if we don't act. This option should always be included.
If "do nothing" is clearly worse than every proposed option, say why.]

---

## Recommendation

[Author's recommended option and 2–3 sentence rationale. Be direct.
"I recommend Option 1 because..." not "Option 1 may be worth considering."]

---

## Open Questions

- [Question that reviewers should specifically address]
- [Dependency or assumption that needs validation]

---

## Consequences

If the recommended option is adopted:
- [What changes]
- [What ongoing cost is introduced]
- [What future flexibility is constrained]

---

## References

- [Link to relevant prior RFC or ADR]
- [Link to external documentation]
```

### Reviewer Guidance Card

```
When reviewing an RFC:

1. Focus on the problem statement first. Is this the right problem to solve?
   Is it bounded enough to be actionable?

2. Evaluate the options analysis. Are the options genuinely distinct? Are the
   trade-offs accurately represented? Is there an important option that's missing?

3. Check the recommendation. Does the rationale follow from the analysis?
   Is the author's preference legible, or does the recommendation appear
   disconnected from the options as written?

4. Name your role in the comment. "I think this is wrong" is less useful than
   "As the team that owns the downstream service, we will need to [change X]
   if this RFC is adopted — is that accounted for in the cost estimate?"

5. If you have a blocking concern, say so explicitly. "This RFC should not
   proceed without resolving [X]." Don't bury a blocking concern in a list
   of advisory suggestions.

6. If you don't have a blocking concern but have a preference, say that too.
   "I'd prefer Option 2, but I can live with Option 1 if [condition]."
```

---

## Anti-Patterns

**The RFC as a Rubber Stamp.**
A process where RFCs are written after the decision has already been made and implemented, used only to create documentation. The RFC loses its value as a review mechanism. Engineers learn that commenting on RFCs has no effect and stop engaging.

**The Consensus Trap.**
Delaying decision-making until every objection is resolved. In a technically opinionated team, some objections will never be fully resolved — the decision-maker's job is to acknowledge them and decide anyway. A process that requires consensus to move forward produces paralysis on contested decisions.

**The Scope Creep RFC.**
An RFC that starts with a specific technical question and expands to include organizational philosophy, process reform, or questions that belong in a different forum. Scope-creep RFCs become long, unfocused, and hard to decide on. Keep the problem statement bounded.

**The Missing "Do Nothing" Option.**
An RFC that presents two implementation approaches without considering whether the change is necessary at all. "Do nothing" is always an option and should always be analyzed. An RFC that cannot make the case against "do nothing" may not have a sufficiently compelling problem statement.

**The Undocumented Decision.**
An RFC that concludes with a decision communicated verbally or in a Slack thread, without an ADR. The RFC process's value is captured in the ADR — a decision that exists only in memory or a Slack archive is effectively undocumented, regardless of how thorough the RFC was.

[^1]: The Rust RFC Process. https://github.com/rust-lang/rfcs — the canonical open-source RFC process from which many engineering RFC conventions derive.
[^2]: Larson, W. (2019). *An Elegant Puzzle: Systems of Engineering Management*. Stripe Press. Chapter 4 discusses architectural decision-making and documentation at the org level.
