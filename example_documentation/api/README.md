# CampusKit API contract

This folder holds the proposed Week 3 contract for CampusKit. No application implements it yet, and neither server URL in the contract is live.

| File | Purpose |
| --- | --- |
| [`openapi.yaml`](openapi.yaml) | OpenAPI 3.0.3 description of seven operations: public `GET /healthz` and six `/api/v1` business operations |
| [`examples/loan-create.json`](examples/loan-create.json) | Request body for `POST /api/v1/loans` |
| [`examples/loan-created.json`](examples/loan-created.json) | 201 response to that request; also the 200 replay while the loan is active |
| [`examples/loan-return-request.json`](examples/loan-return-request.json) | Request body for `PATCH /api/v1/loans/{loanId}` |
| [`examples/loan-returned.json`](examples/loan-returned.json) | 200 response to the first or a repeated return; also the 200 borrow replay after the return |
| [`examples/items-page.json`](examples/items-page.json) | 200 response to `GET /api/v1/items?available=true&limit=20&offset=0`, before the example loan |
| [`examples/loans-page.json`](examples/loans-page.json) | 200 response to `GET /api/v1/loans` while the example loan is active |
| [`examples/error-conflict.json`](examples/error-conflict.json) | 409 `ITEM_UNAVAILABLE` from `POST /api/v1/loans` |
| [`examples/error-validation.json`](examples/error-validation.json) | 422 `VALIDATION_ERROR` from `POST /api/v1/loans` |

The shared 400 `BAD_REQUEST` example (`components/examples/BadRequest`, an unknown query parameter) is inline in `openapi.yaml` only; it has no JSON file.

Rules that a schema cannot express, such as the check order, retry behaviour, and the 403 policy, are in the operation descriptions. The [API contract chapter](../example/03-api-contract.md) explains why.

## Keeping the files in sync

`openapi.yaml` repeats each fixture file under `components/examples`, so the contract renders on its own. Each of those entries names its file in `x-example-file`. Change the YAML and the JSON file in the same pull request. Noor owns the contract; Jordan and Maya review changes.

## Synthetic data

Every ID, time, and name here is made up. None of it is real member data, real equipment, or captured traffic.

| Value | Meaning |
| --- | --- |
| `11111111-1111-4111-8111-111111111111` | Item: Club camera 01 |
| `33333333-3333-4333-8333-333333333333` | Example loan: checked out 2026-09-29T18:00:00Z, due 2026-10-06T18:00:00Z, returned 2026-09-30T18:00:00Z |
| `44444444-4444-4444-8444-444444444444` | `clientRequestId` of the example borrow |
| `55555555-5555-4555-8555-555555555555` | An earlier, returned loan of the same camera |
| `66666666-6666-4666-8666-666666666666` | A second item ID, used only in the `IDEMPOTENCY_CONFLICT` example |
| `req-demo-...` | Illustrative `requestId` values |

## Using the fixtures

The client's mock API can serve these fixtures now. Once a local API exists, the same files can be sent with cURL, for example `curl -X POST "$BASE_URL/api/v1/loans" -H "Content-Type: application/json" -H "Authorization: Bearer $ACCESS_TOKEN" --data @api/examples/loan-create.json`. Keep real tokens out of files, screenshots, and shared shell history.

The core classroom walkthrough is in the [Markdown API contract](../example/03-api-contract.md); the generic [planning requirements](../requirements.md) describe what students must document in their own API plans. This folder is optional machine-readable supporting material. The Markdown edition has no export/validation tooling or application runtime; schema and runtime checks are future implementation tasks, not claimed results.
