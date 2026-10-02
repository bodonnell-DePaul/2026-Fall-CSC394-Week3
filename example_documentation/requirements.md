# Group application planning assignment

Create a product and technical plan for your group's application. Submit the following four Markdown documents.

Use the files in the [`example/`](example/) directory as examples of the expected content and level of detail. Adapt the plans to your application; do not copy the CampusKit decisions.

## 1. Product brief

**Output:** `01-project-brief.md`

Include:

- The problem your application addresses and its target users.
- The core user workflow and the value it provides.
- The MVP features and important features outside the current scope.
- The group members and their primary responsibilities.
- Key assumptions, constraints, and unresolved product questions.

Keep this section brief. It should provide enough context to understand the architecture, API, and delivery plan.

## 2. Architecture

**Output:** `02-architecture.md`

Include:

- A diagram showing the client, API, database, external services, and how they communicate.
- A short description of each major component.
- The main data entities and their relationships.
- How authentication, authorization, secrets, and persistent data will be handled.
- Important technical decisions, alternatives considered, and tradeoffs.
- Major technical risks or questions that require investigation.

The architecture must support your application's core user workflow.

## 3. API contract

**Output:** `03-api-contract.md`

Include:

- API conventions, including the base path, data format, authentication, and error format.
- An endpoint list covering the application's core workflow.
- For each endpoint: method, path, purpose, inputs, successful response, and expected errors.
- Example requests and responses using synthetic data.
- Important rules for validation, permissions, conflicts, retries, filtering, or pagination when applicable.
- A short list of planned API or acceptance tests.

The API must agree with the components, data, and security rules in the architecture.

## 4. Delivery plan

**Output:** `04-delivery-plan.md`

Include:

- The MVP scope and the smallest useful end-to-end workflow.
- Major milestones and the evidence required to complete each milestone.
- A prioritized backlog of user stories and technical tasks.
- For each backlog item: owner, acceptance criteria, dependencies, and target milestone.
- The team's Definition of Done.
- Plans for testing, code review, continuous integration, deployment, and rollback.
- The main project risks, their owners, and planned responses.

The plan must schedule integration early. Do not leave client, API, database, or deployment integration until the end.

## Submission check

Before submitting, confirm that:

- The four documents describe the same application and use the same terms.
- Every core workflow has supporting architecture, API endpoints, and assigned work.
- Every backlog item has one accountable owner and clear acceptance criteria.
- Examples use synthetic data and do not include secrets.
- Planned work is not described as completed work.
- Your group can explain and defend its decisions.
