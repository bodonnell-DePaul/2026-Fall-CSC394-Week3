# 1. Project brief

> **Week 3 submission - proposed design, not built software.** CampusKit, its users, its team, and its data are fictional. All backlog items and application tests in this example are planned.

**Read alongside:** [Group application planning requirements](../requirements.md). This brief supplies CampusKit's project-specific context; the linked document explains what students must include in their architecture, API, and delivery plans.

## Problem and target users

A fictional student media club lends cameras, microphones, and accessories from a shared cabinet. Its spreadsheet does not reliably show what is available, who has borrowed an item, or whether a return has been recorded. Members sometimes arrive expecting an item that someone else has already borrowed.

**CampusKit** is a small self-service lending ledger. A club member signs in, browses equipment, borrows an available item, sees their own loans, and records a return. The application prevents two members from holding an active loan for the same physical item.

The primary users are **members of one club**. For this course project, the catalogue and approved demo members are seeded by the team. CampusKit does not unlock the cabinet, prove physical possession, or verify that a reported return actually happened. A real lending operation would need a separate check-in policy.

**Success for the MVP:** two synthetic members can complete the borrow/return workflow in a shared deployed environment, with persistent data, understandable failures, and enforced ownership. We are not claiming that the MVP exists at Week 3.

## Team and accountability

All names below are invented. One person owns each responsibility; pairing does not remove ownership.

| Fictional member | Primary roles | Additional ownership |
| --- | --- | --- |
| Maya Chen | Project manager + QA lead | Requirements, scope, acceptance criteria, presentation |
| Eli Brooks | Technical lead + DevOps lead | Architecture, cloud, CI/CD, repository and collaboration tools |
| Jordan Singh | Frontend lead | Accessible interaction, technical manual, user-facing copy |
| Noor Patel | Backend lead | API contract, data model, transactions, authentication integration |

Every member must be able to explain the whole system. A different teammate reviews each change. This four-person allocation follows the small-team ownership pattern discussed in Week 2; it is not a required roster for other teams. [Course sources: S1, S3](../sources.md)

## Smallest useful end-to-end journey

1. Member A signs in using an approved synthetic account.
2. A browses the catalogue and opens the detail page for a camera.
3. A selects **Borrow**; the API records one active loan and returns a receipt.
4. Refreshing the page still shows the loan. Another member cannot borrow the same item while that loan is active.
5. A records a return; the same loan becomes returned and the item becomes available again.

This is the vertical slice: **browser -> API -> database -> visible result**. A collection of disconnected screens is not the slice.

## Scope and priorities

| MoSCoW priority | Commitment | Reason |
| --- | --- | --- |
| Must | Sign in; browse and view items; borrow; see personal active/history loans; return | Completes the useful workflow |
| Must | Server validation, ownership checks, persistent data, safe retries, keyboard access, tests, and deployability | These protect the workflow; they are not polish to remove when late |
| Should | Search the catalogue by name | Helpful, but availability filtering is sufficient for the MVP |
| Could | Export personal loan history as CSV; a bounded read-only MCP catalogue lookup | Only after the complete MVP is stable and there is demonstrated value |
| Won't this quarter | Reservations, renewals, payments/fines, email reminders, public registration, inventory editing, multiple clubs, native mobile apps, AI borrowing actions | Keeps the team focused on a deliverable product |

The **current v1 contract does not include** name search, CSV export, or MCP operations. Those require a separately reviewed contract change if selected. Ordinary UI actions call the HTTP API directly; they do not need an LLM. [S3]

## Requirements and acceptance

CampusKit's project-specific outcomes, business rules, quality targets, and acceptance cases are documented in the worked architecture, API contract, and delivery plan. The generic [planning requirements](../requirements.md) explain how students should keep those three deliverables complete and consistent.

An HTTP response alone is not sufficient acceptance evidence: the persistent state and the user's visible result must agree. Runtime evidence remains planned, not completed.

## Assumptions and open decisions

- The four-person team is comfortable with TypeScript and basic web development. It will not learn several unfamiliar infrastructure frameworks simultaneously.
- One club has at most 200 seeded items for the demo. Each database item represents one physical unit; quantities are not pooled.
- A loan is due exactly seven 24-hour periods after checkout. The server stores UTC timestamps; the client labels localized display times.
- A Week 4 authentication spike will confirm that the managed identity provider can issue an API-audience access token and map its subject to a seeded member.
- The club's real-world return policy, actual hosting budget, and any use of real student records would need approval outside this fictional project.

## AI disclosure

The original exemplar was drafted with GitHub Copilot and Claude Opus 5.5 for course analysis, design, and API documentation; this Markdown revision and the generic student planning requirements were prepared with GitHub Copilot. Document and contract review is separate from application testing: no application has been implemented, deployed, or runtime-tested. Students should replace this disclosure with an accurate 1-3 sentence account of their own assistance and verification and remain able to explain every decision.

[S3]: ../sources.md#s3-week-2-planning-workflow-and-mcp
