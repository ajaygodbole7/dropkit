# Hypermedia and Performance

> Zalando RESTful API Guidelines — hypermedia and performance reference.
> Source: https://opensource.zalando.com/restful-api-guidelines/ (CC-BY-4.0, Zalando SE)

## REST Maturity Level

### Level 2 Required [#162]

MUST use REST maturity level 2 — proper use of HTTP methods and status codes. All APIs must:

- Use correct HTTP verbs (GET for reads, POST for creates, etc.)
- Return appropriate HTTP status codes
- Use standard content types

### Level 3 Optional [#163]

MAY use REST maturity level 3 (HATEOAS) — hypermedia-driven navigation is allowed but not required.

## Hypertext Controls [#164]

MUST use common hypertext controls. When providing links, use a consistent format:

```yaml
_links:
  type: object
  properties:
    self:
      type: string
      format: uri
      description: Link to this resource
    next:
      type: string
      format: uri
      description: Link to the next page
    related_resource:
      type: string
      format: uri
      description: Link to a related resource
```

## Simple Hypertext Controls [#165]

SHOULD use simple hypertext controls for pagination and self-references — at minimum, provide `self` links on resources and `next`/`prev` on paginated collections.

## Absolute URIs [#217]

MUST use full, absolute URI for resource identification — all links must be complete URIs including scheme and host:

```
# Correct
"self": "https://api.example.com/orders/abc123"

# Wrong — relative reference
"self": "/orders/abc123"
```

## No Link Headers with JSON [#166]

MUST NOT use link headers with JSON entities — embed links in the JSON response body, not in HTTP `Link` headers.

## Performance

### Reduce Bandwidth [#155]

SHOULD reduce bandwidth needs and improve responsiveness:

- Support partial responses (`fields` parameter)
- Support conditional requests (ETag/If-None-Match)
- Use compression

### Gzip Compression [#156]

SHOULD use gzip compression — support `Accept-Encoding: gzip` and `Content-Encoding: gzip` for responses.

### Partial Responses [#157]

SHOULD support partial responses via filtering — allow clients to request specific fields:

```
GET /orders/abc123?fields=id,status,total
```

Define a `fields` query parameter:

```yaml
parameters:
  - name: fields
    in: query
    schema:
      type: string
    description: Comma-separated list of fields to include in the response
    example: id,status,total
```

### Sub-Resource Embedding [#158]

SHOULD allow optional embedding of sub-resources — let clients expand related resources inline to reduce round trips:

```
GET /orders/abc123?embed=items,customer
```

### Caching [#227]

MUST document cacheable GET, HEAD, and POST endpoints:

- Annotate cacheable endpoints with `Cache-Control` header in the response
- Define appropriate `max-age` or `no-cache` directives
- Use `ETag` and `If-None-Match` for conditional requests [#182]

```yaml
headers:
  Cache-Control:
    schema:
      type: string
    example: "max-age=300, public"
  ETag:
    schema:
      type: string
    example: '"33a64df551425fcc55e4d42a148795d9f25f89d4"'
```

## HTTP Headers

### Standard Headers [#133]

MAY use standard headers as defined by IANA.

### Header Naming [#132]

SHOULD use kebab-case with uppercase separate words for HTTP headers: `Content-Type`, `Cache-Control`, `X-Flow-ID`.

### Content Headers [#178]

MUST use `Content-*` headers correctly:

- `Content-Type` MUST be set on requests with body and all responses with body
- `Content-Encoding` for compression
- `Content-Language` when content is localized

### Location Header [#180]

SHOULD use `Location` header instead of `Content-Location` for pointing to created resources.

### Content-Location [#179]

MAY use `Content-Location` header for pointing to the actual content location.

### Prefer Header [#181]

MAY consider supporting `Prefer` header for processing preferences:

- `Prefer: return=minimal` — return only essential fields
- `Prefer: return=representation` — return the full resource
- `Prefer: respond-async` — request asynchronous processing

### ETag Support [#182]

MAY consider supporting ETag together with `If-Match`/`If-None-Match` headers for optimistic locking and conditional requests.

### Idempotency-Key [#230]

MAY consider supporting `Idempotency-Key` header for safe retries of non-idempotent operations (POST, PATCH).

## Remote References [#234]

MUST only use durable and immutable remote references — any `$ref` pointing to external resources must reference a specific, versioned, immutable URL.

## API Operation

### Publish Specification [#192]

MUST publish OpenAPI specification for APIs — make the spec available to consumers.

### Monitor Usage [#193]

SHOULD monitor API usage — track metrics like request volume, latency, and error rates.
