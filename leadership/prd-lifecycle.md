# PRD Lifecycle Management

> **Leadership context:** The PRD is the most abused document in product engineering. It gets frozen at kick-off, accumulates stale assumptions, and becomes something engineers route around rather than reference. The engineering leader's job is not to own the PRD — that's the PM's job — but to set the standard for how it evolves, enforce its use as a working document, and make sure the team is building against current understanding, not the snapshot from last quarter's planning session.

## Purpose

This playbook defines how a PRD is structured, how it evolves over the life of a product, and what "done" means at each stage. It applies the continuous discovery model: the PRD tracks evolving product understanding rather than locking scope. One living PRD per product; version history is in git, not in a frozen "v1.3 Final (2)" document.

---

## The One Living PRD Principle

The most common PRD failure mode is treating it as a contract. You write it during discovery, lock it at kick-off, and then run a separate change-control process for every deviation. This produces:
- Engineers building against stale requirements because updating the PRD is bureaucratic
- PMs maintaining two sources of truth (the PRD and the Jira description where the real decisions happen)
- Post-mortems where "we built what the PRD said" is treated as an absolution

The one living PRD model inverts this. The document always reflects current intent. The history is in version control. The changelog (see below) is how you communicate what changed and why — not a separate document, a section of the same one.

**The test:** Any engineer joining the team mid-project should be able to read the PRD and get accurate context about what is being built, why, and for whom. If that isn't true, the document has drifted.

---

## PRD Structure

Every PRD has five required sections. Sections 1–3 are set during discovery and rarely change. Sections 4–5 evolve continuously.

### Section 1 — Problem Statement

One paragraph. What is the user unable to do, or doing poorly, because this product does not exist? Written as a job-to-be-done: *"When [situation], users want to [goal], but [obstacle] prevents them. The cost of this is [outcome]."*

Avoid solution language in this section. "Users need a dashboard" is not a problem statement.

### Section 2 — User Job and Outcome Table

| User type | Job to be done | Success looks like | Current workaround |
|-----------|----------------|-------------------|-------------------|
| (e.g.) Product manager | Monitor experiment results across multiple flag variants without switching tools | One view, no manual aggregation, results available within 24h of flag creation | Exporting from LaunchDarkly, joining in a spreadsheet |

Fill one row per distinct user type with a meaningfully different job. Two or three rows is normal. More than four typically indicates unclear product scope, not genuine user diversity.

### Section 3 — Personas

Two or three personas, maximum. Each persona is a named, specific person — not a demographic. Use Cooper's persona format: a name, a role, one sentence on what they're trying to accomplish in their work, and one sentence on where this product fits in their day.

Personas are not market segments. A persona is "Maria, a senior PM who runs 8–10 concurrent experiments and needs to brief the CPO weekly on business impact." Not "data-driven PMs at mid-market SaaS companies."

### Section 4 — Hypothesis and Bets

The specific bets this product makes. Written as falsifiable hypotheses: *"We believe [action] will produce [outcome] for [persona]. We will know this worked when [metric] moves by [amount] within [timeframe]."*

This section is updated when bets change. If a bet is invalidated by discovery or user research, it is not deleted — it is struck through with a one-line explanation and a changelog entry (see Section 5).

### Section 5 — Changelog

Append-only log of substantive changes to this document. Format:

```
## Changelog

### 2026-03-15 — Scope reduction: dropped async notifications
**What changed:** Removed async email notification feature from v1 scope.
**Why:** User interviews (n=8) showed in-app visibility was sufficient for the primary persona. Async adds two weeks; the bet doesn't require it.
**Impact:** Engineers on notification service unblocked for other work. Revisit in v2 if retention data shows re-engagement gap.

### 2026-02-28 — Hypothesis updated: engagement metric changed
**What changed:** Success metric changed from DAU to experiment-start rate.
**Why:** DAU is affected by marketing campaigns we don't control. Experiment-start rate isolates product value.
**Impact:** None to scope; affects how the data team instruments the event.
```

---

## Two-Tier Update Process

Not all changes to the PRD are equal. Use two tiers to reduce change-control friction for minor updates while keeping major changes visible.

### Tier 1 — Minor (update and log, no approval required)

- Clarifications that don't change scope or success criteria
- Updated metrics or targets based on baseline data
- Persona refinements from user interviews
- Removing features deferred to a later version (log the reason; if the cut affects team allocation or delivery commitments, use Tier 2 instead)
- Copy edits and format fixes

**Process:** Edit the document, add a changelog entry, notify the team in the project communication channel.

### Tier 2 — Major (requires PM + EM alignment before update)

- Changes to the core problem statement
- Adding new user types with distinct jobs
- Changing a success metric after development has begun
- Scope additions that affect timeline or team allocation
- Invalidating a hypothesis (the bet was wrong)

**Process:** PM and EM discuss async or sync (depending on urgency), align on the change and its rationale, then update the document together. Add a changelog entry. If the change affects delivery commitments, communicate upstream before updating the document.

---

## Lifecycle Stages

The PRD exists across four stages. What's expected in each:

### Discovery

The PRD is a working hypothesis. Sections 1–3 are drafts. Section 4 has at least one hypothesis. Section 5 is empty.

**Gate to Alignment:** Problem statement is crisp, job-outcome table has ≥1 validated row, at least one persona has been confirmed with a real user, and the PM can articulate the primary bet in one sentence.

### Alignment

The PRD is being reviewed and socialized. Engineering is scoping, design is exploring. The document is updated frequently as assumptions surface.

**Gate to Delivery:** PM and EM have signed off on Section 4. Engineering has estimated. Dependencies are identified. No open "TBD" in Sections 1–3.

### Delivery

The PRD is a reference document. Minor updates are expected as implementation surfaces edge cases. Major changes require Tier 2 process.

**In-flight discipline:** If an engineer can't reconcile a decision with the PRD, that's a signal the PRD is wrong — update it, don't paper over it in the code.

**Gate to Shipped:** Feature is in production, instrumentation is live, the team can observe the success metrics in Section 4.

### Shipped

The PRD is archived (read-only). The outcomes are documented in a brief post-ship retrospective appended to the changelog: what the metrics showed, whether the bets were validated, and what was learned.

---

## Ownership and Collaboration

**PM owns the PRD.** The PM is the author, the final decision-maker on content, and the person responsible for keeping it current.

**EM is the accountability partner.** The EM's job is to flag when the document has drifted from reality, push back on hypotheses that aren't falsifiable, and ensure engineers can reference the PRD to resolve implementation ambiguity.

**Engineers read and flag.** Any engineer who can't reconcile a decision with the PRD should flag it to the PM within one working day — not route around it.

**Design owns the persona depth.** Design research should feed Sections 2 and 3. If personas are being written without user contact, that's a process gap to fix, not a PRD problem.

---

## Common Failure Modes

**The frozen PRD.** The document is correct at kick-off and stale six weeks later. Engineers stop consulting it. Fix: make PRD review a standing agenda item in the weekly team sync — five minutes to ask "is this still accurate?"

**The PRD-as-contract.** The PM treats deviation from the PRD as a process violation. Engineers treat "it's in the PRD" as a reason not to raise concerns. Fix: Tier 2 process should take hours, not days. The cost of a slow change process is undocumented drift.

**The solution PRD.** The document describes what will be built, not why. Section 1 says "Build a real-time dashboard" instead of describing the user's problem. Fix: ban solution language from Sections 1–3. If the problem statement could have been written after the solution was decided, it was.

**The thousand-page PRD.** The document grows to cover every edge case and specification, duplicating what should live in design files and tickets. Fix: the PRD answers why and for whom. It does not answer how. Technical specs, API contracts, and edge case handling live in linked documents.

**The invisible changelog.** Changes happen but aren't logged. Two months in, no one remembers why the scope changed. Fix: make the changelog append-only and treat entries the same way you treat commit messages — they're permanent, they explain reasoning, and they're written for someone who wasn't in the room.

---

## Background and Motivation

This lifecycle model was developed from the feature lifecycle and roadmapping process revision at Community Tech Alliance (2025–2026). I designed and drove the process changes; the team adopted them. The revision shifted emphasis from scope-locked product specifications to outcome-oriented living documents — a change that reduced planning debt and improved IC alignment with the problems the engineering org was actually trying to solve.

---

## Further reading: demonstration artifacts

> **Demonstration sandbox:** [lifting-logbook](https://github.com/brownm09/lifting-logbook)
> is a personal-project monorepo, not a production system at scale. The artifact linked
> here illustrates the technique; production-scale application of the same technique is
> documented in [ORIGINS.md](../ORIGINS.md) where applicable.

The artifacts below illustrate the techniques described in this playbook against the demonstration sandbox. See [LINKING.md](../LINKING.md) for the full convention. Citation links pin to commit [`413f8a6`](https://github.com/brownm09/lifting-logbook/tree/413f8a62f43f12fa200be3e3307da7ef72c7b446) per the LINKING.md SHA-pinning rule. Where an artifact is intended to evolve as the stack does, a `main` link is provided alongside.

### On the proposal lifecycle state machine

- **Proposal lifecycle and automation** — [`docs/proposals/README.md`](https://github.com/brownm09/lifting-logbook/blob/main/docs/proposals/README.md) (live state). Documents the `draft → accepted → shipped → declined` state machine and the `/propose` skill that automates proposal creation, issue linking, and ROADMAP entry. The four statuses map structurally onto the playbook's four lifecycle stages (Discovery → Alignment → Delivery → Shipped). The key difference from this playbook's model: the proposal lifecycle is scope-definition, not outcome-tracking — there is no equivalent of the Hypothesis and Bets section or falsifiable success metrics. The proposals are the gate-to-Delivery artifact; the PRD extends that into outcome accountability.
- **Lifecycle tooling configuration** — citation: [`.claude/propose.json` at `413f8a6`](https://github.com/brownm09/lifting-logbook/blob/413f8a62f43f12fa200be3e3307da7ef72c7b446/.claude/propose.json). The skill configuration file that wires the `/propose` skill to the repo's proposals directory, GitHub project, milestones, and epics. Demonstrates that lifecycle tooling can be operationalized at the individual-project level — the state machine is not just documented but enforced by the scaffolding that creates every proposal.

### On a worked end-to-end proposal

- **On-call readiness proposal** — citation: [`docs/proposals/2026-05-08-on-call-readiness.md` at `413f8a6`](https://github.com/brownm09/lifting-logbook/blob/413f8a62f43f12fa200be3e3307da7ef72c7b446/docs/proposals/2026-05-08-on-call-readiness.md). Shows a proposal moving through the full lifecycle in a single file: problem statement, proposed solution with three scoped work items, acceptance criteria, linked GitHub issue, ROADMAP milestone placement, and a final `Status: shipped`. The one-living-document structure — where the file accumulates through discovery, alignment, and delivery without being duplicated or forked — is the direct analog to this playbook's one-living-PRD principle.

---

These artifacts are not exhaustive. Per [LINKING.md](../LINKING.md), additional cross-references are added only where they add evaluative power — not as breadth for its own sake.

---

## References

- [Clayton Christensen, Taddy Hall, Karen Dillon, and David Duncan — "Know Your Customers' 'Jobs to Be Done'" (*Harvard Business Review*, September 2016)](https://hbr.org/2016/09/know-your-customers-jobs-to-be-done) — The canonical HBR treatment of the Jobs to Be Done framework. Establishes that customers hire products to accomplish specific outcomes, not to consume features. The job-outcome table format in §2 (job + "success looks like") is a direct application.
- [Alan Cooper — *The Inmates Are Running the Asylum* (Sams, 1998)](https://www.amazon.com/Inmates-Are-Running-Asylum-Products/dp/0672326140) — Origin of Goal-Directed Design and user personas as a product design tool. The guidance to keep personas to two or three reflects Cooper's observation that more personas typically indicate unclear product scope rather than genuine user diversity.
- [Marty Cagan — *Inspired: How to Create Tech Products Customers Love*, 2nd ed. (Wiley, 2018)](https://www.svpg.com/books/inspired-how-to-create-tech-products-customers-love-2nd-edition/) — Establishes continuous discovery and outcome-oriented product thinking. The "one living PRD" design and the rejection of per-version document freezes aligns with Cagan's product-team model, where the document tracks evolving product understanding rather than locking scope.
