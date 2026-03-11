# HTTP Methods and Status Codes

> Zalando RESTful API Guidelines — methods and status codes reference.
> Source: https://opensource.zalando.com/restful-api-guidelines/ (CC-BY-4.0, Zalando SE)

## HTTP Method Semantics [#148]

MUST use HTTP methods correctly. The table below defines the contract each method carries.

| Method | Safe | Idempotent | Cacheable | Request Body | Typical Use |
|--------|------|------------|-----------|--------------|-------------|
| GET | Yes | Yes | Yes | No | Retrieve resource(s) |
| HEAD | Yes | Yes | Yes | No | Check existence, metadata |
| POST | No | No* | No | Yes | Create resource, trigger action |
| PUT | No | Yes | No | Yes | Full replacement of resource |
| PATCH | No | No* | No | Yes | Partial update (merge-patch) |
| DELETE | No | Yes | No | Optional | Remove resource |
| OPTIONS | Yes | Yes | No | No | Discover allowed methods |

*POST and PATCH can be made idempotent — see [#229].

## Method Properties [#149]

MUST fulfill common method properties:

- **GET**: MUST be safe (no side effects). MUST be idempotent. Response cacheable.
- **POST**: Creates new resources (returns `201` with `Location` header) or triggers actions (returns `200`/`202`/`204`). Not idempotent by default.
- **PUT**: Full resource replacement. MUST be idempotent. Returns `200` or `204`.
- **PATCH**: Partial update using `application/merge-patch+json` [RFC 7396]. Returns `200` or `204`.
- **DELETE**: MUST be idempotent. Returns `200` or `204`. Subsequent calls return `204` (not `404`).

## Idempotency [#229][#231]

SHOULD consider designing POST and PATCH idempotent:

- Use **`Idempotency-Key` header** [#230] — client-generated UUID sent with the request. Server stores the key and returns the same response on replay.
- SHOULD use **secondary key** (natural business key) for idempotent POST [#231] — e.g., a unique `order_reference` to prevent duplicate order creation.

## Asynchronous Requests [#253]

MAY support asynchronous request processing:

- Return `202 Accepted` with a `Location` header pointing to a status resource.
- The status resource returns `200` while processing and `303 See Other` (with `Location` to the result) when complete.

## Request Parameter Format [#154]

MUST define collection format of header and query parameters:

- Use `explode: true` with `style: form` for query arrays: `?status=OPEN&status=CLOSED`
- Document whether multi-value means AND or OR semantics.

## Implicit Filtering [#226]

MUST document implicit response filtering:

- If an endpoint silently excludes data (e.g., soft-deleted items), document this in the operation description.

## Status Code Selection

### Success Codes [#150][#151][#220][#243]

MUST specify success and error responses for every operation. Use the **most specific** HTTP status code.

| Code | When to Use |
|------|-------------|
| `200 OK` | GET (with body), PUT/PATCH (with updated body), POST action |
| `201 Created` | POST that creates a resource. MUST include `Location` header |
| `202 Accepted` | Async operation accepted for processing |
| `204 No Content` | PUT/PATCH/DELETE with no response body |
| `207 Multi-Status` | Batch/bulk operations [#152] — response body contains per-item status |

### Redirect Codes [#251]

SHOULD not use redirection codes — prefer explicit resource URLs.

### Client Error Codes

| Code | When to Use |
|------|-------------|
| `400 Bad Request` | Malformed request syntax, invalid parameters |
| `401 Unauthorized` | Missing or invalid authentication |
| `403 Forbidden` | Authenticated but not authorized |
| `404 Not Found` | Resource does not exist |
| `405 Method Not Allowed` | HTTP method not supported on this path |
| `406 Not Acceptable` | Cannot produce requested content type |
| `409 Conflict` | Request conflicts with current resource state |
| `412 Precondition Failed` | ETag mismatch (optimistic locking) |
| `415 Unsupported Media Type` | Request body content type not supported |
| `422 Unprocessable Entity` | Syntactically valid but semantically incorrect |
| `429 Too Many Requests` | Rate limit exceeded [#153] — MUST include `Retry-After` header |

### Server Error Codes

| Code | When to Use |
|------|-------------|
| `500 Internal Server Error` | Unexpected server failure |
| `501 Not Implemented` | Method recognized but not implemented |
| `503 Service Unavailable` | Temporary overload or maintenance. Include `Retry-After` |

## Error Response Format [#176][#177]

MUST use **RFC 9457 Problem Detail** (`application/problem+json`) for all error responses:

```yaml
content:
  application/problem+json:
    schema:
      $ref: '#/components/schemas/Problem'
```

See [data-formats-and-common-objects.md](./data-formats-and-common-objects.md) for the Problem schema.

MUST NOT expose stack traces in error responses [#177].

## Method-to-Status-Code Matrix

| | 200 | 201 | 202 | 204 | 207 | 400 | 401 | 403 | 404 | 409 | 412 | 422 | 429 |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| **GET** | ✓ | | | | | ✓ | ✓ | ✓ | ✓ | | | | ✓ |
| **POST** (create) | | ✓ | ✓ | | | ✓ | ✓ | ✓ | ✓ | ✓ | | ✓ | ✓ |
| **POST** (action) | ✓ | | ✓ | ✓ | | ✓ | ✓ | ✓ | ✓ | ✓ | | ✓ | ✓ |
| **PUT** | ✓ | | | ✓ | | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| **PATCH** | ✓ | | | ✓ | | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| **DELETE** | ✓ | | ✓ | ✓ | | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | | ✓ |
| **Batch** | | | | | ✓ | ✓ | ✓ | ✓ | | | | | ✓ |
