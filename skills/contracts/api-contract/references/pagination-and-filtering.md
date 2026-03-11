# Pagination and Filtering

> Zalando RESTful API Guidelines — pagination and query design reference.
> Source: https://opensource.zalando.com/restful-api-guidelines/ (CC-BY-4.0, Zalando SE)

## Pagination is Mandatory [#159]

MUST support pagination for all list endpoints that may return more than a single page of results. Never return unbounded collections.

## Prefer Cursor-Based Pagination [#160]

SHOULD prefer **cursor-based** pagination over offset-based:

- **Cursor-based** (recommended): Uses an opaque cursor token. Stable under concurrent inserts/deletes. Client sends `cursor` parameter.
- **Offset-based** (discouraged): Uses `offset` + `limit`. Unstable — results can shift between pages when data changes.

### Cursor Pagination Parameters

```yaml
parameters:
  - name: cursor
    in: query
    schema:
      type: string
    description: Opaque cursor from a previous response's pagination links
  - name: limit
    in: query
    schema:
      type: integer
      format: int32
      minimum: 1
      maximum: 100
      default: 20
    description: Maximum number of items to return
```

## Pagination Response Page Object [#248]

SHOULD use the standard pagination response page object. Wrap collection responses in a page object:

```yaml
OrderPage:
  type: object
  required:
    - items
  properties:
    items:
      type: array
      items:
        $ref: '#/components/schemas/Order'
    cursors:
      type: object
      properties:
        before:
          type: string
          description: Cursor pointing to the start of the page
        after:
          type: string
          description: Cursor pointing to the end of the page
    has_more:
      type: boolean
      description: Whether more items exist beyond this page
```

## Pagination Links [#161]

SHOULD use pagination links. Include `self`, `next`, and optionally `prev` links:

```yaml
_links:
  type: object
  properties:
    self:
      type: string
      format: uri
    next:
      type: string
      format: uri
    prev:
      type: string
      format: uri
```

## Avoid Total Count [#254]

SHOULD avoid a total result count — computing total counts is expensive for large datasets and often inaccurate. Use `has_more` boolean instead.

## Simple Query Languages [#236]

SHOULD design simple query languages using query parameters:

- **Filtering**: Use property-based filters: `?status=OPEN&customer_id=abc123`
- **Sorting**: Use `sort` parameter: `?sort=created_at` or `?sort=-created_at` (descending)
- **Field selection**: Use `fields` parameter: `?fields=id,status,total`
- **Embedding**: Use `embed` parameter: `?embed=items,customer`

## Complex Query Languages [#237]

SHOULD design complex query languages using JSON — for advanced filtering beyond simple query parameters, accept a JSON body in a POST to a search endpoint:

```
POST /orders:search
Content-Type: application/json

{
  "filter": {
    "status": ["OPEN", "PROCESSING"],
    "created_at": {"after": "2024-01-01"}
  },
  "sort": [{"field": "created_at", "direction": "desc"}],
  "cursor": "abc123",
  "limit": 20
}
```

## Conventional Query Parameters [#137]

MUST stick to conventional query parameters:

| Parameter | Type | Description |
|-----------|------|-------------|
| `q` | `string` | Free-text search query |
| `sort` | `string` | Sort field(s), prefix `-` for descending |
| `cursor` | `string` | Opaque pagination cursor |
| `limit` | `integer` | Page size (default 20, max 100) |
| `offset` | `integer` | Offset-based pagination (discouraged) |
| `fields` | `string` | Sparse fieldset / partial response |
| `embed` | `string` | Expand sub-resources inline |
| `expand` | `string` | Alias for embed |
