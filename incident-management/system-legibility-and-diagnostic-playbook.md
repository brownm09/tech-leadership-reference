# System Legibility and Diagnostic Judgment Playbook

## Purpose

This playbook addresses a specific failure mode in engineering organizations: systems that work
but cannot be diagnosed — by new engineers, by responders under pressure, or by anyone who
wasn't present when the system was built. It covers what system legibility means, how it decays,
and the concrete practices that preserve it.

The primary audience is engineering leaders and senior individual contributors responsible for
on-call programs, service ownership standards, and onboarding design. It complements the
[On-Call Restructuring Framework](on-call-restructuring-framework.md), which covers rotation
structure and incident coordination; this playbook covers the underlying diagnostic
infrastructure that makes a rotation sustainable.

---

## The Core Problem: Two Kinds of Legibility

System legibility in software operates at two levels simultaneously, and they decay
independently.

**Code legibility** is the property Fred Brooks called *conceptual integrity* in *The Mythical
Man-Month* (Addison-Wesley, 1975): a system behaves as if designed by a single coherent mind.
When conceptual integrity is absent, engineers cannot predict a system's behavior from
inspection. They test by poking and observing, which is slow and unreliable under pressure.

**Runtime legibility** is whether an engineer can understand what the system is doing *right
now*. Traditional monitoring — dashboards of predefined metrics — is a legibility theater: it
makes a system look understandable while hiding the state space that actually causes incidents.
High-cardinality, high-dimensionality telemetry is the only way to make a running system legible
against novel failure modes.[^1] A system that is only legible when it fails in anticipated ways
is not actually legible.

Both kinds of legibility require active maintenance. Neither is a one-time documentation
exercise.

---

## How Judgment Atrophies in Software Systems

Organizations build systems to scale coordination and reduce variance. Every system that absorbs
a decision also erodes the judgment capacity that could have made it. The longer a system runs
without challenge, the more its internal logic becomes invisible — to operators, to managers,
and eventually to the people who designed it.

### The Automation Trap

**ORMs and query planners** trade SQL legibility for development velocity. This is generally
worth it — until a query plan degrades under production load and the engineer debugging it has
never written a JOIN. The system worked; the human judgment needed to fix it when it stopped had
quietly atrophied. Michael Feathers describes the downstream result in *Working Effectively with
Legacy Code* (Prentice Hall, 2004): code that works but that no one can safely change, because
the understanding required to change it was never built or has since been lost.

**CI/CD pipelines** create a structurally similar problem at the deployment level. Green tests
are a legibility proxy: they assert correct behavior on the dimensions the test suite measures.
Engineers who trust the proxy lose the habit of reasoning about what the pipeline doesn't cover —
infrastructure state, downstream dependencies, traffic patterns. Automation bias migrates
judgment to the binary green/red output of a pipeline whose coverage is never inspected.

**AI-assisted coding** is the current frontier of this dynamic. The output is plausible and
often correct, which is exactly the condition under which judgment atrophies fastest. The risk
is not that AI-generated code is wrong; it is that engineers who ship it without building a
mental model of it have no basis for diagnosing it when it fails in a novel way.

### Normalization of Deviance

Diane Vaughan's analysis of the Challenger disaster in *The Challenger Launch Decision*
(University of Chicago Press, 1996) describes the institutional version of this failure: small
deviations from expected behavior are observed, then rationalized as acceptable. The decision
process looks correct by its own internal measures, while those measures quietly drift from what
they were designed to track. The same mechanism operates in software systems when alert
thresholds are tuned to eliminate noise rather than to reflect actual risk — and when no one
reviews the thresholds afterward.

---

## Interval Validation Mechanisms

Several engineering practices are legibility-preservation mechanisms wearing other labels. The
key insight is that legibility must be validated at the same cadence as the system changes, not
as a one-time setup activity.

### Architecture Decision Records (Per Decision)

ADRs — documented in Michael Nygard's 2011 post[^2] — force the decision-maker to articulate
why a system is the way it is, in terms a future engineer can evaluate. Without them,
architectural decisions accumulate as tacit knowledge in the heads of the engineers who made
them, then evaporate when those engineers leave. ADRs are the primary tool for keeping the
*why* of a system legible over time.

### Fitness Functions (Per Commit)

Fitness functions, from Neal Ford, Rebecca Parsons, and Patrick Kua's *Building Evolutionary
Architectures* (O'Reilly, 2017), are automated assertions over architectural properties:
dependency direction, module coupling, latency budgets, security posture. Legibility of
architectural intent needs to be mechanically enforced at the same cadence as code changes or
it drifts. A fitness function is a legibility check that runs in CI.

### Chaos Engineering and Game Days (Quarterly to Annually)

Chaos engineering and game days (developed formally at Netflix and described in Gregor Hohpe's
*The Software Architect Elevator*, O'Reilly, 2020) are explicit interval validation of
operational judgment. The point is not to find bugs — it is to verify that engineers can still
reason about and respond to failure modes, independent of whether the system is currently
failing. Every question a responder cannot answer during a game day is a gap in the runbook or
in the system's observability.

### Cross-Service On-Call Rotation (Ongoing)

Werner Vogels' 2006 articulation of "you build it, you run it" at Amazon[^3] was originally a
quality incentive, but its legibility effect is equally important: engineers who operate their
systems under pressure build mental models that design-time work does not produce. Rotation
across *unfamiliar* services is the stronger version — it surfaces the gap between documentation
and actual operational knowledge, which is exactly the gap that matters in an incident.

### Runbook Criteria Review (Annually)

Do not ask only "are we hitting our targets?" Ask "do our targets still track what we care
about?" Any alert threshold or runbook trigger condition that is adjusted to reduce alert
frequency without a corresponding investigation of the underlying signal is a legibility
regression. Criteria review should be an explicit annual step, separate from the incident
retrospective.

---

## Making Systems Diagnosable for Onboarding Engineers

The hardest version of the legibility problem eliminates the one thing most systems rely on —
implicit knowledge in the team. The right frame is: what artifacts does the system produce that
enable correct inference by someone with no priors?

### Layer 1: Service Catalog

An engineer who has never touched a service needs to answer four questions before they can
diagnose anything:

1. What does this service do and what does it own?
2. What does it depend on, and what depends on it?
3. What does healthy look like?
4. Who wrote it and who operates it?

A service catalog — with those four fields, kept current, and linked from the alerting system —
is the single highest-leverage investment for onboarding legibility. Spotify's Backstage[^4] is
the reference implementation, but the format matters less than the discipline: every service has
an entry, entries are owner-maintained, and alerts link to them.

A catalog is not a wiki. It is a machine-readable registry with a human-readable summary. Wikis
rot; catalogs with ownership fields have someone accountable for keeping them current.

### Layer 2: Distributed Tracing with Meaningful Context

Tracing is the only observability primitive that preserves causality across service boundaries.
For an onboarding engineer, this is the difference between "the checkout service is slow" and
"the checkout service is slow because a downstream call to the inventory service is timing out,
and here is the specific request path." The first requires system knowledge to diagnose; the
second is followable by anyone who can read a waterfall.

As Cindy Sridharan establishes in *Distributed Systems Observability* (O'Reilly, 2018), metrics
tell you something is wrong; traces tell you where and why.[^5] The practical requirements:

- Trace IDs must propagate through every service boundary, including async queues and caches
- Spans must carry enough context to identify the operation — not just a function name, but the
  relevant parameters (order ID, user ID, SKU) that make a trace instance interpretable
- Errors must attach structured context, not just a message string

### Layer 3: Runbooks Structured for Diagnosis

Most runbooks describe what to do after diagnosis. That is the wrong ordering for an engineer
new to the system. A useful runbook starts with symptoms and walks toward cause:

1. **Alert context.** What does this metric measure and what does it mean when it's elevated?
2. **First look.** The specific dashboard or query that shows current system state in one view.
   What healthy looks like on that view.
3. **Decision tree.** If you see A, proceed to step 4. If you see B, escalate to service owner.
4. **Known failure modes.** The three to five historical causes of this alert, with distinguishing
   symptoms.
5. **Remediation.** The action, its blast radius, and its reversibility.
6. **Escalation.** Role, not individual — who to call and what to tell them.

Michael Nygard's *Release It!* (Pragmatic Bookshelf, 2nd ed., 2018) establishes the
architectural prerequisite: systems must be designed with operational affordances — health
endpoints, structured error output, graceful degradation signals — that make the diagnostic path
navigable. A runbook is only as good as the system it describes; if the system does not expose
its own state cleanly, no runbook compensates.

### Layer 4: Error Messages Designed as Diagnostics

An error message is read by an engineer who does not know what caused it. Its job is to narrow
the search space. Good error messages answer: what was attempted, what failed, and what state
the system was in when it failed.

"Connection refused" is not a diagnostic. "Connection refused to postgres-primary.internal:5432
after 3 retries; last attempted at 14:23:07; upstream service returned HTTP 503 at 14:22:59 —
possible upstream availability issue" gives a new engineer a hypothesis and a timeline.

The Google SRE book (Beyer, Jones, Petoff, Murphy, O'Reilly, 2016) frames this as part of SLO
design: errors at service boundaries should distinguish user-visible impact from internal
degradation.[^6] That structuring requirement naturally improves error message quality because
it forces engineers to think about what information a responder needs.

### Layer 5: The Onboarding Engineer as Legibility Probe

The most effective validation of onboarding legibility is to deliberately give a new engineer a
staged incident and observe where they get stuck.

Run a structured shadow on-call exercise where the new engineer responds to a simulated incident
with an experienced engineer watching silently. Treat every question the new engineer asks as a
gap report: a missing runbook step, an untraced service boundary, a metric without a
description, an error message without context.

Matthew Skelton and Manuel Pais's *Team Topologies* (IT Revolution, 2019) frames this as a
cognitive load problem: if a new engineer cannot build a sufficient mental model of a service to
diagnose it within a bounded time window, the service's cognitive load exceeds what a single
person can hold, and the team structure needs to change to match.[^7] The shadow on-call
exercise surfaces this before the production incident does.

---

## What Doesn't Work

**Architecture diagrams maintained manually.** They are wrong within a month. Auto-generated
dependency maps from traces or service mesh telemetry are always current; hand-drawn diagrams
are not.

**"Ask the team" as an escalation path.** It works until the team turns over, which is exactly
when it is most needed. Escalation paths should name roles, not individuals.

**Onboarding buddy programs without legibility artifacts.** They transfer tacit knowledge, which
is valuable, but they do not fix the underlying gap. When the buddy leaves, the knowledge leaves
with them.

**Long wikis.** They encode what the author thinks a newcomer needs, filtered through what the
author has forgotten they know. They age immediately and are usually found too late.

---

## Summary

Legibility for new engineers is an architectural property that must be maintained at the same
cadence as the system itself. The governing question for any system is: *what artifacts does
this produce that enable correct inference by someone with no priors, under pressure?* Answer
that question concretely — with a service catalog entry, a trace, a runbook that starts with
symptoms, and error messages that point toward causes — and onboarding legibility follows as a
byproduct.

---

## References

[^1]: Majors, C., Fong-Jones, L., & Miranda, G. (2022). *Observability Engineering*. O'Reilly Media.

[^2]: Nygard, M. (2011, November 15). Documenting Architecture Decisions. *Cognitect Blog*. https://cognitect.com/blog/2011/11/15/documenting-architecture-decisions

[^3]: Vogels, W. (2006). All Things Distributed: *ACM Queue interview on Amazon's architecture*. https://queue.acm.org/detail.cfm?id=1142065

[^4]: Spotify Engineering. (2020). *Backstage: An open platform for building developer portals*. https://backstage.io

[^5]: Sridharan, C. (2018). *Distributed Systems Observability*. O'Reilly Media.

[^6]: Beyer, B., Jones, C., Petoff, J., & Murphy, R. (Eds.). (2016). *Site Reliability Engineering: How Google Runs Production Systems*. O'Reilly Media.

[^7]: Skelton, M., & Pais, M. (2019). *Team Topologies: Organizing Business and Technology Teams for Fast Flow*. IT Revolution Press.
