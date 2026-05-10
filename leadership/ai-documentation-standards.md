# AI-Assisted Documentation Standards

**Scope:** How to ensure AI-generated documentation is auditable, verifiable, and trustworthy
**Applies to:** Any project where Claude Code or another AI assistant contributes to architectural
documentation, design documents, ADRs, playbooks, or technical recommendations

---

## Background and Motivation

These standards were developed from the Claude Code and Claude integration initiative at Community Tech Alliance (2025–2026). I designed the phased rollout plan with separate use cases for engineers and managers and developed a structured evaluation rubric for measuring impact. The auditability requirements here emerged from the recognition that AI-generated documentation lacks the same provenance chain as human-authored documentation — and that for regulated environments and portfolio-quality work, that gap needs to be closed explicitly.

---

## Core Principle: AI Recommendations Must Be Sourced

When an AI assistant produces documentation that names a technology, pattern, protocol, or
compliance framework as a recommendation, that recommendation must be traceable to a primary
source. Unsourced AI output is unverifiable — a reader cannot distinguish a well-grounded
recommendation from a confident-sounding hallucination.

This is not primarily a hallucination-prevention rule. It is an intellectual integrity rule:
the same standard that applies to a staff engineer writing a design document applies to
AI-generated output that carries the same weight.

---

## What Must Be Cited

### Technology and framework choices
Link to the official documentation for every tool selected. The official docs are the
authoritative source for capability claims, configuration, and limitations.

**Examples:**
- Choosing NestJS → link to [docs.nestjs.com](https://docs.nestjs.com)
- Choosing Prisma → link to [prisma.io/docs](https://www.prisma.io/docs/getting-started)
- Choosing Turborepo → link to [turbo.build/repo/docs](https://turbo.build/repo/docs)

### Protocol and specification choices
Link to the normative specification for every protocol or standard referenced.

**Examples:**
- OAuth 2.0 → [RFC 6749](https://datatracker.ietf.org/doc/html/rfc6749)
- PKCE → [RFC 7636](https://datatracker.ietf.org/doc/html/rfc7636)
- GraphQL → [spec.graphql.org](https://spec.graphql.org/October2021/)
- OIDC → [openid.net/specs/openid-connect-core-1_0.html](https://openid.net/specs/openid-connect-core-1_0.html)

### Architectural patterns
Link to the original paper, article, or book chapter that introduced or best defines the
pattern. Do not cite secondary summaries or Wikipedia.

**Examples:**
- Hexagonal Architecture → [Cockburn (2005)](https://alistair.cockburn.us/hexagonal-architecture/)
- Clean Architecture / Dependency Rule → [Uncle Bob (2012)](https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html)
- Dependency Injection → [Fowler (2004)](https://martinfowler.com/articles/injection.html)

### Compliance and regulatory references
Link to the primary regulatory or standards body source. Do not cite compliance vendor
summaries or blog posts as the authoritative source.

**Examples:**
- GDPR Article 17 → [EUR-Lex — GDPR (Regulation 2016/679), Article 17](https://eur-lex.europa.eu/legal-content/EN/TXT/?uri=CELEX:32016R0679)
- HIPAA Security Rule → [hhs.gov/hipaa/for-professionals/security](https://www.hhs.gov/hipaa/for-professionals/security/index.html)

---

## Where Citations Go

### In ADRs
A `## References` section at the bottom of every ADR. Each entry: a linked title, and
one sentence describing what decision in the ADR it supports.

```markdown
## References

- [RFC 6749 — OAuth 2.0](https://datatracker.ietf.org/doc/html/rfc6749) — The authorisation
  code flow used by Clerk and Auth0; defines the token exchange this ADR relies on.
- [Clerk — Documentation](https://clerk.com/docs) — Official docs for the primary identity
  provider chosen in the Decision section.
```

### In playbooks
A `## References` section at the bottom of any playbook that cites a specific framework,
methodology, or standard. Playbooks that contain only internal process guidance with no
external dependencies do not require citations.

### In AI responses
When Claude recommends a technology in a response (not only in committed documentation),
it should include the official documentation link inline. This makes the recommendation
immediately verifiable without requiring a follow-up search.

---

## What Does Not Require a Citation

- Internal cross-references to other ADRs, issues, or PRs in the same project
- Decisions that are entirely internal (e.g., a naming convention with no external basis)
- Widely understood facts that do not assert specific capability claims

---

## Consolidated Reference Index

For projects with multiple ADRs, maintain a consolidated reference index (e.g.,
`docs/adr-references.md`) that groups all sources by domain. This serves as:
- A single place to audit coverage ("are all our technology choices sourced?")
- A reading list for new contributors onboarding to the architecture
- A starting point for due-diligence reviews (security, ARB, compliance audits)

Update the index whenever an ADR's references change.

---

## Enforcing This Standard in Prompts

When opening a session in which documentation will be written or updated, include this in
the opening brief:

> All ADRs and design documents produced this session must include a ## References section
> with links to primary sources for every technology, pattern, and specification cited.
> See docs/adr-references.md for the established pattern.

For projects using CLAUDE.md, encode this as a standing instruction in the
**Documentation and Citations** section so it applies automatically every session.

---

## Further reading: demonstration artifacts

> **Demonstration sandbox:** [lifting-logbook](https://github.com/brownm09/lifting-logbook)
> is a personal-project monorepo, not a production system at scale. The artifact linked
> here illustrates the technique; production-scale application of the same technique is
> documented in [ORIGINS.md](../ORIGINS.md) where applicable.

The artifacts below illustrate the techniques described in this document against the demonstration sandbox. See [LINKING.md](../LINKING.md) for the full convention. Citation links pin to commit [`413f8a6`](https://github.com/brownm09/lifting-logbook/tree/413f8a62f43f12fa200be3e3307da7ef72c7b446) per the LINKING.md SHA-pinning rule.

### On ADR citation practice

- **Worked ADR with a full References section** — citation: [ADR-016: Cycle Planning Agent](https://github.com/brownm09/lifting-logbook/blob/413f8a62f43f12fa200be3e3307da7ef72c7b446/docs/adr/ADR-016-cycle-planning-agent.md). A production ADR whose `## References` section demonstrates this document's citation requirements: the Anthropic Tool Use guide (for the technology chosen), the Anthropic Node SDK (for the integration library), the Hexagonal Architecture primary source (for the architectural pattern governing the port boundary), and prior ADRs in the same series (for internal cross-references). Each entry includes the one-sentence explanation of what decision it supports — the format this document prescribes for ADR References sections.

### On AI attribution and workflow provenance

- **Workflow-level attribution via `/propose`** — citation: [`.claude/propose.json` at `413f8a6`](https://github.com/brownm09/lifting-logbook/blob/413f8a62f43f12fa200be3e3307da7ef72c7b446/.claude/propose.json). The skill configuration file for the `/propose` command, which scaffolds proposals with AI assistance, creates linked GitHub issues, and adds ROADMAP entries. Every commit produced by this skill carries the `Co-Authored-By: Claude Sonnet 4.6 <noreply@anthropic.com>` footer, implementing attribution at the workflow level rather than relying on per-session author discipline. This demonstrates the document's core premise: provenance is a workflow property, not a post-hoc annotation.

**What is missing:** The repo does not yet have a `docs/adr-references.md` consolidated reference index (as this document recommends in the "Consolidated Reference Index" section). That is a candidate for a future round once the ADR series reaches the scale where per-domain grouping adds navigation value.

---

These artifacts are not exhaustive. Per [LINKING.md](../LINKING.md), additional cross-references are added only where they add evaluative power — not as breadth for its own sake.
