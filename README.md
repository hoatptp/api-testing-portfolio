# API Testing Portfolio

A hands-on API testing portfolio using [restful-api.dev](https://restful-api.dev) — a free public REST API backed by a real database with full data persistence.

---

## Tools

- **Postman** — writing and running test collections
- **Collection Runner** — executing collections and exporting test run reports

---

## Project Structure

```
api-testing-portfolio/
├── README.md
└── restful-api-dev/
    ├── collections/
    │   ├── unit_tests.json          ← Test each endpoint individually
    │   ├── integration_tests.json   ← Test end-to-end flows
    │   └── contract_tests.json      ← Verify response structure & data types
    └── reports/
        ├── unit_tests_postman_test_run.json
        ├── integration_tests_postman_test_run.json
        └── contract_tests_postman_test_run.json
```

---

## Test Coverage

### Unit Tests (`unit_tests.json`)
Tests each endpoint in isolation, asserting detailed behavior:
- Correct status codes (200, 404)
- Response body values match request
- Required fields exist
- Response time under 2000ms
- Both positive and negative cases for all 7 public endpoints

| Request | Type | Cases |
|---------|------|-------|
| GET /objects | Positive | Status, array, not empty, each object has id and name |
| GET /objects?id=3&id=5&id=10 | Positive | Returns exactly requested IDs, no extras |
| GET /objects/{id} | Positive | Status, ID match, name is non-empty string |
| GET /objects/99999999 | Negative | 404, error field returned |
| POST /objects | Positive | Status, ID created, name/price match, createdAt present |
| POST /objects (missing name) | Negative | API behavior — accepts request without name field |
| PUT /objects/{id} | Positive | Status, name/price/year updated, updatedAt present, ID unchanged |
| PUT /objects/99999999 | Negative | 404, error field returned |
| PATCH /objects/{id} | Positive | Status, name patched, data field intact, updatedAt present |
| PATCH /objects/99999999 | Negative | 404, error field returned |
| DELETE /objects/{id} | Positive | Status, message contains deleted ID |
| DELETE /objects/99999999 | Negative | 404, error field returned |

### Integration Tests (`integration_tests.json`)
Tests end-to-end flows with real data persistence across multiple requests:

| Flow | Description |
|------|-------------|
| Flow 1 | Full CRUD: POST → GET verify → PUT → GET verify → PATCH → GET verify → DELETE → GET 404 |
| Flow 2 | POST then GET by ID → verify data matches exactly |
| Flow 3 (Negative) | DELETE → PUT on deleted object → verify 404 |
| Flow 4 (Negative) | DELETE → DELETE again → verify 404 |

### Contract Tests (`contract_tests.json`)
Verifies response structure remains consistent with the API spec:
- Field names match spec (`id`, `name`, `data`, `createdAt`, `updatedAt`, `message`)
- Correct data types (`id` is string, `data` is object or null, timestamps are valid)
- No unexpected fields outside the spec
- `createdAt` only appears in POST responses
- `updatedAt` only appears in PUT/PATCH responses

---

## API Under Test

**Base URL:** `https://api.restful-api.dev`

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | /objects | Get all objects |
| GET | /objects?id=3&id=5 | Get multiple objects by IDs |
| GET | /objects/{id} | Get a single object |
| POST | /objects | Create a new object |
| PUT | /objects/{id} | Full update of an object |
| PATCH | /objects/{id} | Partial update of an object |
| DELETE | /objects/{id} | Delete an object |

---

## Findings

Discrepancies found between API documentation and actual behavior:

| # | Finding | Expected | Actual |
|---|---------|----------|--------|
| 1 | `createdAt` format | ISO 8601 string e.g. `"2022-11-21T20:06:23.986Z"` | Unix timestamp number e.g. `1776277391093` |
| 2 | `name` field validation | Required — should return 400 if missing | Not enforced — returns 200 and creates object without `name` |

---

## How to Run

1. Import a collection file from `collections/` into Postman
2. Open **Collection Runner**
3. Select the collection → click **Run**
4. After the run completes, click **Export Results** to save the report as JSON to `reports/`
