# 3. API contract

**Owner:** Noor. **Client consumer:** Jordan. **Contract reviewer:** Maya.

**Assignment reference:** [Group application planning requirements](../requirements.md). This document is a worked CampusKit example of the API deliverable. The Markdown examples are sufficient for the classroom walkthrough.

The machine-readable companion is [api/openapi.yaml](../api/openapi.yaml), using **OpenAPI 3.0.3**. It is a machine-checkable description of this proposed interface, not an implementation. OpenAPI is an optional extension in the Week 3 lesson, not a mandatory extra assignment. [JSON fixtures](../api/examples/) make the examples reusable for client mocks and later contract tests. The rules below explain behavior that a schema alone cannot enforce. [S4]

The sample local server URL is `http://localhost:3000`; no server is supplied or running. `https://campuskit.example.invalid` is a deliberately non-live documentation address. These are not demo deployment links.

## Conventions

- Business resources are plural nouns under `/api/v1`. The identity provider owns login/logout flows; the application does not invent password endpoints.
- Business requests require `Authorization: Bearer <access-token>`. Only `/healthz` is public.
- Request and response bodies use JSON. UUIDs identify items, loans, and retry keys. Server timestamp strings are RFC 3339 UTC instants with whole seconds and a `Z` suffix; persistence uses the same precision.
- Malformed JSON, malformed path UUIDs, and unknown/repeated query parameters receive **400 BAD_REQUEST**. Parsed but invalid JSON fields or invalid values of supported query parameters receive **422 VALIDATION_ERROR**, following the Week 3 field-validation lesson.
- The server rejects undeclared writable fields with **422**. It does not accept client-supplied member IDs or timestamps. It reports all safely detectable field errors together rather than making the client discover them one request at a time.
- Single-resource success bodies are the resource itself. Collection bodies use `items` and `page`; errors always use the error envelope below.
- Catalogue order is `name ASC, id ASC`. Personal loan order is `checkedOutAt DESC, id DESC`. Offset paging is not a snapshot: concurrent changes can shift later pages.
- `limit` is an integer from 1 through 100, default 20; `offset` is an integer from 0 through 10,000, default 0. `page.total` counts the filtered, authorized collection.
- The sample design allows 60 business requests per authenticated member per minute. Exceeding that allowance returns **429** with `Retry-After` in seconds; health checks are excluded. This is a proposed policy, not a measured capacity.

**Example-specific choices:** the lecture shows a `data`/`pagination` collection envelope, including page-number examples. CampusKit consistently uses `items`/`page` with limit/offset instead; `items` means collection entries even for the loans list. Fixed ordering keeps v1 small. Flat `itemId` references avoid duplicating item data, and a well-formed missing reference returns 404. These are documented alternatives, not corrections to the lecture. [S4]

## Endpoint inventory

| Method and path | Purpose / inputs | Success |
| --- | --- | --- |
| `GET /healthz` | Public process liveness; no dependency or configuration disclosure | 200: `{"status":"ok"}` |
| `GET /api/v1/items` | Catalogue; optional `available=true` or `false`, `limit`, `offset` | 200: item collection |
| `GET /api/v1/items/{itemId}` | One catalogue item by UUID | 200: item |
| `POST /api/v1/loans` | Borrow with `itemId` and `clientRequestId` | 201 + `Location`: new loan; 200: recognized retry |
| `GET /api/v1/loans` | Caller's loans; optional `status=active` or `returned`, `limit`, `offset` | 200: loan collection |
| `GET /api/v1/loans/{loanId}` | One loan owned by the caller | 200: loan |
| `PATCH /api/v1/loans/{loanId}` | Record return using only `{"status":"returned"}` | 200: returned loan, including repeated returns |

There is no `DELETE /loans`: a returned loan remains part of the history and the retry ledger. We choose methods based on the resource lifecycle, not to demonstrate every HTTP verb.

## Resource shapes

| Schema | Required fields | Source of truth |
| --- | --- | --- |
| Item | `id`, `name`, `category`, `description`, `available` | Seeded inventory; availability derived from active loans |
| Loan | `id`, `itemId`, `status`, `checkedOutAt`, `dueAt`, `returnedAt` | Committed loan row; `returnedAt` may be null |
| Collection | `items`, `page` containing `limit`, `offset`, `total` | Authorized, filtered query |
| Error | `error` containing `code`, `message`, `details`, `requestId` | Server; `details` is an array and may be empty |

An item category is `camera`, `audio`, or `accessory`. A loan status is `active` or `returned`. Neither `memberId` nor an identity-provider subject is exposed in these response schemas.

## Worked borrow request and receipt

These are **illustrative messages**, not captured network traffic. Dates are synthetic transaction dates, not assignment deadlines.

```http
POST /api/v1/loans
Authorization: Bearer <access-token>
Content-Type: application/json
```

```json
{
  "itemId": "11111111-1111-4111-8111-111111111111",
  "clientRequestId": "44444444-4444-4444-8444-444444444444"
}
```

```http
HTTP/1.1 201 Created
Content-Type: application/json
Location: /api/v1/loans/33333333-3333-4333-8333-333333333333
```

```json
{
  "id": "33333333-3333-4333-8333-333333333333",
  "itemId": "11111111-1111-4111-8111-111111111111",
  "status": "active",
  "checkedOutAt": "2026-09-29T18:00:00Z",
  "dueAt": "2026-10-06T18:00:00Z",
  "returnedAt": null
}
```

**What the server decided:** the member, loan ID, checkout time, seven-day due time, and current state. A client cannot extend the due date or borrow as another member by adding fields.

### The retry contract

| Repeated request | Result | Effect |
| --- | --- | --- |
| Same member, same key, same item | 200 and the existing loan's current state | No new loan, even if the original loan has since been returned |
| Same member, same key, different item | 409 `IDEMPOTENCY_CONFLICT` | No change |
| New key, item already has an active loan | 409 `ITEM_UNAVAILABLE` | No change |

`clientRequestId` is unique **within one member**, retained with the loan, and generated once for a borrow intent. A different member using the same UUID does not get access to the first member's loan. POST is not automatically idempotent; these explicit application and database rules make this operation retry-safe.

The UI disables duplicate submissions while a request is in flight. If the connection drops, it explains that the outcome is uncertain and offers a retry using the retained key or a check of My Loans. A new borrowing action after a return uses a new key.

## List and return examples

```http
GET /api/v1/items?available=true&limit=20&offset=0
Authorization: Bearer <access-token>
```

```json
{
  "items": [
    {
      "id": "11111111-1111-4111-8111-111111111111",
      "name": "Club camera 01",
      "category": "camera",
      "description": "Entry-level camera with a carry case.",
      "available": true
    }
  ],
  "page": {"limit": 20, "offset": 0, "total": 1}
}
```

This is the catalogue **before** the example loan. After checkout, that item is no longer in the `available=true` result until it is returned. An empty valid result is **200** with `items: []`; a dependency failure is not an empty result.

```http
PATCH /api/v1/loans/33333333-3333-4333-8333-333333333333
Authorization: Bearer <access-token>
Content-Type: application/json
```

```json
{"status": "returned"}
```

The response is **200** with the same loan ID, `status: "returned"`, and a server-supplied `returnedAt`, for example `2026-09-30T18:00:00Z`. Repeating the request returns that original return time. Attempting to set `status: "active"`, change `itemId`, or provide `returnedAt` returns **422**.

## Errors are part of the interface

```json
{
  "error": {
    "code": "ITEM_UNAVAILABLE",
    "message": "This item is already checked out.",
    "details": [],
    "requestId": "req-demo-conflict"
  }
}
```

| Status | Meaning in this design | Client response |
| --- | --- | --- |
| 400 | Malformed JSON, malformed path UUID, unknown/repeated query parameter | Correct the malformed request; preserve user input |
| 401 | Missing, invalid, or expired token | Prompt sign-in; do not pretend the collection is empty |
| 403 | Valid identity lacks membership, or member does not own the loan | Explain lack of access; do not expose loan contents |
| 404 | Well-formed ID does not identify an item/loan | Explain that the record was not found |
| 409 | Active-item or retry-key conflict | Refresh availability or explain the key conflict; do not blindly resubmit |
| 415 | Write request is not JSON | Correct the client's content type |
| 422 | Parsed JSON has invalid/missing/unknown fields, or a supported query parameter has an invalid value | Show all field problems together; preserve input |
| 429 | Rate limit exceeded | Respect `Retry-After`; do not loop |
| 503 | A required dependency is unavailable | Show temporary unavailability; a write may still need reconciliation |
| 500 | Unexpected internal failure | Show safe message and request ID; no stack trace or secrets |

401 responses include `WWW-Authenticate: Bearer`. A 422 response uses code `VALIDATION_ERROR` and `details`, for example `{"field":"itemId","message":"Must be a UUID."}`. The correlation `requestId` helps support find a log entry; it is not the borrow deduplication key. Expected client errors do not become 500s simply because they reached a common error handler.

For this teaching design, another member's **known** loan ID returns 403, whereas a missing ID returns 404. That reveals existence, so a privacy-sensitive system might deliberately use 404 for both. If the team changes that policy, it must update the contract and tests together, not leave each endpoint to decide.

## Client integration without waiting for the backend

Jordan can build against the JSON fixtures while Noor implements the service. The mock must reproduce errors, loading, empty lists, 200 replay versus 201 creation, and returned timestamps, not only a happy-path screenshot. Fixtures describe examples; they are not the source of truth for live availability.

The My Loans view resolves each `itemId` from a cached, paginated catalogue query rather than making a new request for every row. This is practical for the explicitly bounded 200-item demo catalogue; a larger application would revisit the response shape or add a batch lookup.

Once an application exists, a smoke request could be made as follows. Supply a real local base URL and a synthetic user's short-lived token **outside** source control; do not paste the token into documentation or an AI conversation.

```bash
curl --fail-with-body "$BASE_URL/api/v1/items?available=true" \
  -H "Authorization: Bearer $ACCESS_TOKEN"
```

This package deliberately does not include a token or run that command against a live service.

## Planned acceptance and contract tests

| Test | Setup / action | Required evidence |
| --- | --- | --- |
| AT-01 | Missing, expired, wrong-audience, or invalid-signature token | 401; no business data returned |
| AT-02 | Valid token whose subject has no member record | 403; no auto-enrollment |
| AT-03 | Valid catalogue query; valid empty query; invalid limit; unknown/repeated parameter | 200 with correct paging; 200 empty; 422 for invalid value; 400 for unknown/repeated parameter |
| AT-04 | Borrow an available item; refresh and restart the service | 201 + Location; exactly one persistent loan |
| AT-05 | Two members race to borrow the same available item | One 201, one 409; exactly one active loan in PostgreSQL |
| AT-06 | Repeat or concurrently retry the same member/key/item | One new loan; replay is 200 with the same ID |
| AT-07 | Reuse a member/key for a different item | 409; no second loan |
| AT-08 | List own loans; get/patch another member's known loan | No foreign loans in list; 403 for both direct requests |
| AT-09 | Return a loan twice; replay original borrow after return | Same original return time; item available; replay is 200 returned loan, not a new loan |
| AT-10 | Submit writable member/timestamp fields, bad enum, malformed JSON, or non-JSON body | 422 for invalid fields; 400 for malformed JSON; 415 for unsupported media type |
| AT-11 | Dependency outage, rate limit, unexpected server error | Safe 503/429/500 envelope; `Retry-After` on 429; no fake success |
| AT-12 | Complete core flow by keyboard, at 390 px, including an uncertain write | Visible focus, usable controls, understandable error/retry state |
| AT-13 | View a known item's detail; request a missing item; supply a malformed path UUID | 200 with fields matching the catalogue; 404 for well-formed missing ID; 400 for malformed path ID |

**Test layers:** unit tests cover validation/state decisions; integration tests run the real API with disposable PostgreSQL; browser tests exercise the user journey; schema checks guard OpenAPI/example consistency. A mock-only test cannot prove AT-05, persistence, or authorization.

### Versioning and change control

Version 1 is a reviewed baseline. Noor updates OpenAPI first for interface changes; Jordan and Maya review consumer and test impact in the same PR. Removing a field, changing its meaning/type, or adding a required request field needs an explicit migration/versioning decision. Optional features do not silently appear in `/v1` without documentation.

This Markdown edition does not include a validation script or application runtime. The planned checks distinguish schema/example consistency from actual API, database, and browser behavior. Application tests remain planned until implemented and executed with recorded results; reading a schema or mock does not make a runtime test pass.

[S4]: ../sources.md#s4-week-3-api-design-and-documentation
