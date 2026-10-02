# 4. Delivery plan

**Planning / scope owner:** Maya. **Release owner:** Eli. **Status:** proposed work; issue identifiers below are example labels, not links to existing GitHub issues.

**Assignment reference:** [Group application planning requirements](../requirements.md). This document is a worked CampusKit example of the delivery-plan deliverable. Its US/EN work items connect CampusKit's design to the AT tests in the API contract.

## Planning assumptions

The sample team budgets six focused project hours per person each week: 24 team-hours, with roughly six reserved for review, integration, and unexpected work. This is a planning assumption, not a course workload rule. T-shirt sizes are relative, not promises about hours.

Use **one-week iterations in Weeks 3-10**, consistent with Week 2's planning lesson. At each review, update the next week's commitment using completed evidence rather than a percentage-done guess. Week 11 is the final presentation. Week numbers below follow the Fall syllabus; official calendar dates and submission instructions come from D2L. [S1, S3]

## Milestones and exit evidence

The roadmap is maintained directly in the table below: agree the design, prove a persistent slice, complete the shared MVP, strengthen operations and quality, then rehearse and hand off.

| Week | Deliverable / goal | Exit evidence, not just activity |
| --- | --- | --- |
| 3 | Architecture, API, and delivery plan | Linked documents; reviewed schemas/examples; owners, dependencies, risks, and bounded spikes |
| 4 | Architecture, persistence, auth, integration | Spike records; migrations and seed plan; token/member checks; thin deployed skeleton; concept-demo script |
| 5 | **Concept demo** plus CI/testing | Member signs in, borrows, and sees a persistent loan; a conflict/error is demonstrated; tests and reviewed build recorded |
| 6 | **Midterm check-in demo** in a shared cloud environment | Full borrow/return MVP; own-loan access enforced; client/API integration; deploy and rollback smoke checks |
| 7 | Operations and peer feedback | Structured logs, alert, cost review, deployment automation, restore rehearsal, contribution evidence |
| 8 | Security, performance, and debt work | Negative tests, measured latency report, documented debt decisions; no speculative feature expansion |
| 9 | **Preview demo** | Rehearsal-quality full flow; known bugs and recovery plan; documentation matches the release |
| 10 | Presentation and handoff preparation | Release candidate, runbook, clean setup walkthrough, contribution/reflection drafts |
| 11 | **Final demo, presentation, and documentation** | Running release; final manual and evidence links; peer assessment and individual reflections |

The Week 3 submission describes future CI, cloud, and hardening work without pretending those later skills are already implemented. Early skeleton deployment is a risk-reduction choice, not a claim that the course moves its cloud lecture to Week 4.

## Proposed user-story backlog

Every row becomes an issue with one assignee, priority, size, sprint, acceptance checklist, and dependency links. **All are initially proposed/Backlog.** Dependencies refer to the enabling tasks below.

| ID / priority | User story and acceptance criterion | Owner / size | Target and dependency |
| --- | --- | --- | --- |
| US-01 / Must | As a club member, I can sign in so my loans belong to me. Valid identity works; invalid tokens and unprovisioned subjects are denied (AT-01/02) | Noor / M | W4; EN-02, EN-03 |
| US-02 / Must | As a member, I can browse available equipment so I do not plan around a borrowed item. Filtering/paging and empty/error states satisfy AT-03 | Jordan / M | W4; EN-01, EN-02 |
| US-03 / Must | As a member, I can inspect an item so I choose the right unit. Detail matches the list; missing IDs and malformed paths satisfy AT-13 | Jordan / S | W5; US-02 |
| US-04 / Must | As a member, I can borrow an item so I can use it. Persistence, conflict, and same-key retry satisfy AT-04 through AT-07 | Noor / M | W5; US-01, EN-02 |
| US-05 / Must | As a member, I can see my active and returned loans so I can track my responsibilities. No other member's loan is listed or exposed (AT-08) | Noor / M | W5; US-04 |
| US-06 / Must | As a member, I can record a return so the item can be borrowed again. Repeat returns preserve the timestamp; old borrow retries do not reborrow (AT-09) | Noor / M | W6; US-05 |
| US-07 / Must | As a keyboard/mobile user, I can complete the same workflow so access is not mouse-only. All AT-12 checks pass | Jordan / M | W6; US-02 through US-06 |
| US-08 / Should | As a member, I can search by item name so I find equipment faster. Results respect availability and paging; contract reviewed first | Jordan / S | W8 only if MVP is stable |
| US-09 / Could | As a member, I can export my history so I can keep a personal record. Export includes only my loans and is safe for spreadsheet import | Maya / S | W8 only if MVP is stable |
| US-10 / Could | As a member using an approved assistant, I can look up available items so I can plan a visit. Read-only, bounded, authorized results; failures stay visible | Eli / M | W8 spike only; US-02 and permission review |

US-08 through US-10 are **not release commitments** and are absent from the current OpenAPI contract. If an item grows to XL, split it before commitment. Do not solve schedule pressure by dropping authentication, review, tests, or truthful error handling.

## Enabling tasks and dependencies

These are engineering tasks, not a substitute for user stories.

| ID | Task and output | Owner / size | Target / dependency |
| --- | --- | --- | --- |
| EN-00 | Establish app lint/format scripts; record routing-doc review and a local health-endpoint cURL check | Eli / S | W3; Noor pairs on routing/check |
| EN-01 | Review the requirements baseline, v1 contract, examples, and requirement-to-test traceability | Maya / S | W3 |
| EN-02 | Migrations, synthetic seed, constraints; prove lending race/retry rule | Noor / M | W4; EN-01 |
| EN-03 | Identity-provider token/member spike; record chosen configuration | Eli / S | W4; EN-01 |
| EN-04 | Deploy thin web service + persistent DB; record cost estimate | Eli / S | W4 |
| EN-05 | CI gates, pull-request rules, and repeatable release artifact | Eli / M | W5; EN-04 |
| EN-06 | Automate AT-01 through AT-11 and AT-13 across unit/API/database layers; Jordan owns AT-12 in US-07 | Maya / M | W5-W6; split into small test issues alongside stories |
| EN-07 | Setup, API, operations, and demo manual maintained from working builds | Jordan / S | W5-W10, updated each sprint |
| EN-08 | Logs, alerts, billing check, deployment automation, restore rehearsal | Eli / M | W7; EN-04, EN-05 |
| EN-09 | Authorization regression, accessibility and performance evidence | Maya / M | W8; MVP and EN-06 |
| EN-10 | Rehearsal, release notes, presentation and contribution evidence | Maya / S | W9-W11; working release |

**Critical dependency chain:** EN-01 -> EN-02/03 -> US-01/04 -> US-05/06 -> integrated MVP. In parallel, Jordan builds client views against reviewed fixtures and Eli removes hosting risk. Maya pairs on tests from the beginning instead of receiving all testing work at the end.

## Sample issue ready for the task board

**US-04 - Borrow one available item**  
Owner: Noor. Reviewer: Maya. Priority: Must. Size: M. Sprint: Week 5. Dependencies: EN-02, EN-03, US-01.

As a signed-in club member, I want to borrow an available item so I can use it and the club can track it.

**Acceptance concerns:** availability, safe retries, validation, truthful failures, ownership, persistence, and concurrency. This issue must satisfy the relevant negative cases as well as the visible happy path.

- [ ] Request body matches `LoanCreate`; identity is taken from the validated token.
- [ ] First valid borrow returns 201, Location, and a schema-valid loan.
- [ ] Refresh/restart proves that the committed loan persists.
- [ ] Two competing borrowers create exactly one active loan.
- [ ] Same-key retries return the same loan ID; different-item key reuse returns 409.
- [ ] Invalid, unauthorized, and dependency-failure paths have the documented responses.
- [ ] UI exposes loading, success, conflict, and uncertain-write recovery.
- [ ] Linked tests, PR review, API docs, and demo evidence satisfy Definition of Done.

**Evidence to attach later:** PR URL, CI run, test names/results, release SHA, and a short demo note. There are no fabricated evidence links in this sample.

## First two iteration commitments

| Iteration | Commitment | Exit / adaptation decision |
| --- | --- | --- |
| Week 3 | Finalize this package, review the contract and fixtures, create issue/board structure, assign spikes; plan EN-00 tooling evidence | Team can explain boundaries and errors; no unresolved ownerless Must-have; a real team also records the separate weekly tooling homework |
| Week 4 | Prove token/member integration, persistent schema and concurrency rules; deploy thin skeleton; build browse UI against fixtures | If a spike fails, time-box an approved alternative and replan immediately; do not hide the failure until the concept demo |

Weeks 5-6 prioritize the complete vertical slice. Maya limits new commitments if Noor is the bottleneck and moves pairing/testing work to Eli or Maya while preserving one accountable owner per issue.

The concept demo depends on both US-04 and US-05: the member must see the persisted loan after refresh, not only a transient checkout toast. Jordan pairs on their client integration while Noor owns the end-to-end acceptance criteria. The return feature, US-06, completes the MVP in Week 6.

## Working agreements and Definition of Done

Use a GitHub Projects board with **Backlog -> To Do -> In Progress -> In Review -> Done**. Limit In Progress work to one or two cards per member. Update cards daily; surface a blocker after 30 minutes rather than disappearing for a week. Hold planning at the start, two or three brief check-ins, and a review/retrospective each week.

Use short feature branches and pull requests into protected `main`; no direct pushes. One teammate must approve. A changed contract and its client, tests, and documentation travel together.

**Week 3 design-ready checklist:** scope and assumptions stated; components/data/requests agree; examples and errors are defined; every Must-have has an owner and acceptance evidence; risks and next spikes are visible; AI use is disclosed.

**Later implementation Definition of Done:**

- Acceptance criteria and relevant negative cases pass.
- Lint, type checks, unit/API tests, and the appropriate database/browser checks pass.
- A teammate has reviewed the change; no known regression was introduced.
- OpenAPI, setup/runbook, and user documentation match the behavior.
- The change is integrated in the shared environment when that environment exists.
- The issue records evidence and known limitations, and the author can explain the code.

Moving a card to Done is a claim about evidence, not how long someone worked.

## Risk register

Each risk is open at this design milestone. Likelihood/impact are qualitative estimates; triggers tell us when to act.

| Risk / likelihood-impact | Trigger | Mitigation / fallback | Owner |
| --- | --- | --- | --- |
| Identity integration stalls / medium-high | No valid API-audience token by end W4 spike | Use official SDK and synthetic tenant; time-box approved provider alternative; never bypass auth publicly | Eli |
| Concurrent borrowing corrupts state / medium-high | Parallel spike produces two active loans | Transaction + partial unique index; test against PostgreSQL, not only mocks; keep constraint in migrations | Noor |
| Client/API drift / high-high | Fixture, schema, or status expectations differ in PR | Review contract first; validate examples; integration tests for replay, return, and errors | Maya |
| Data disappears on deploy / medium-high | Seeded loan vanishes after restart | Managed persistent DB; migrations, backups, and restore rehearsal; never store live state in process memory | Eli |
| Scope expansion delays MVP / high-high | A Could-have starts before Must acceptance is demonstrated | Defer CSV/MCP/search; maintain a visible cut line; choose smaller UI rather than weaker quality gates | Maya |
| Key member becomes unavailable / medium-high | Missed check-ins or blocked task for two days | Pair and document; secondary reviewer knows the area; rebalance work and notify instructor promptly | Maya |
| Credentials or member data leak / medium-high | Secret scan/review finds unsafe material | Synthetic fixtures only; approved secret storage; redact logs; stop sharing and rotate any exposed credential | Noor |
| Cloud cost or outage threatens demo / medium-high | Estimate exceeds budget or shared deployment fails | Review pricing W4; monitor spend; retain local seeded setup and prior release; rehearse fallback honestly | Eli |

## Testing, CI/CD, and release plan

**Week 5 pull-request gates:** install locked dependencies; lint; type-check; unit tests; OpenAPI/example validation; API tests against a disposable PostgreSQL instance. Use synthetic accounts or controlled token-verifier fixtures for isolated tests; also run a separate real-provider integration check so a mocked verifier does not masquerade as an authentication test.

**Week 6 integration gates:** browser smoke test for browse -> borrow -> My Loans -> return, plus the negative ownership scenario. Link the deployed commit and the test run. Main-branch releases build an immutable artifact, apply reviewed backward-compatible migrations, deploy, and run health plus authenticated catalogue/workflow smoke checks.

**Rollback:** retain the last known-good application release. Roll it back only when compatible with the current schema. An application rollback does not undo a destructive migration; those need a separately reviewed recovery plan. Never test backup restoration by overwriting the shared database: restore into a disposable environment and record elapsed time.

**Week 7 operational baseline:** request IDs and safe structured logs; alert on sustained 5xx errors; observable health and latency; cost alert; scripted deployment; a backup/restore record. Week 8 adds measured performance and targeted security/technical-debt work rather than an unplanned architecture rewrite.

## Demonstration and fallback plan

The **Week 5 concept demo** demonstrates sign-in, catalogue, persistent borrow, and a meaningful failure. The **Week 6/full-flow demo** adds personal history, return, and cross-member denial:

1. Show the requirement and architecture, then sign in as synthetic member A.
2. Borrow the available camera, refresh My Loans, and show persistence.
3. Use member B's stale view to trigger 409; explain the database invariant.
4. Retry A's original request key and show the same loan ID, not a duplicate.
5. Show B cannot access or return A's loan. A returns it; a repeat return preserves the timestamp.
6. Connect the behavior to the test run, release SHA, known risks, and next milestone.

If cloud hosting fails, try the rehearsed local app with seeded PostgreSQL and the approved identity provider. If internet/identity service is also unavailable, label a recording or screenshots honestly as a fallback, not a live demonstration. The syllabus's functioning-demo requirement still applies; having a recording does not waive it. [S1]

## Handoff and contribution evidence

By Weeks 10-11, the project manual includes setup, architecture, API, migrations/seeds, deployment, rollback/restore, monitoring, known limits, and demo instructions. Each member links issues, reviewed PRs, tests, and owned decisions in an individual contribution note and participates in peer assessment. Do not infer contribution from commit counts alone.

[S1]: ../sources.md#s1-fall-2026-syllabus
[S1, S3]: ../sources.md#source-authority
