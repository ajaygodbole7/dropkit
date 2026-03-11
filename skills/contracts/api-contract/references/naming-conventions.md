# Naming Conventions

> Zalando RESTful API Guidelines — naming rules reference.
> Source: https://opensource.zalando.com/restful-api-guidelines/ (CC-BY-4.0, Zalando SE)

## URL Path Segments

| Rule | Level | Constraint |
|------|-------|------------|
| [#129] | MUST | Use **kebab-case** for path segments: `/order-items/{order-item-id}` |
| [#134] | MUST | **Pluralize** resource names: `/orders`, `/customers` |
| [#141] | MUST | Keep URLs **verb-free** — no `/getOrders` or `/createUser` |
| [#138] | MUST | **Avoid actions** — think about resources. Exceptions: truly non-resource actions can use POST to a verb path as last resort |
| [#135] | SHOULD | **Not use `/api`** as base path |
| [#136] | MUST | Use **normalized paths** — no empty segments (`//`), no trailing slash |
| [#228] | MUST | Use **URL-friendly resource identifiers**: `[a-zA-Z0-9:._\-/]*` |
| [#142] | MUST | Use **domain-specific resource names** — not generic names like `/items` |
| [#143] | MUST | Identify resources and sub-resources **via path segments**: `/orders/{order-id}/items/{item-id}` |
| [#145] | MAY | Consider using (non-)nested URLs — flatten when sub-resource has a globally unique ID |
| [#146] | SHOULD | **Limit number of resource types** — prefer fewer, well-modeled resources |
| [#147] | SHOULD | **Limit sub-resource levels** — max 3 levels after root: `/{resource}/{id}/{sub}/{id}/{sub}/{id}` |

### Resource Identity

| Rule | Level | Constraint |
|------|-------|------------|
| [#140] | SHOULD | Define **useful resources** with clear identity, lifecycle, and state transitions |
| [#139] | SHOULD | **Model complete business processes** — ensure the API covers full workflows |
| [#241] | MAY | **Expose compound keys** as resource identifiers using a separator (e.g., `sku_country`) |

## Query Parameters

| Rule | Level | Constraint |
|------|-------|------------|
| [#130] | MUST | Use **snake_case** for query parameters: `?order_status=OPEN&sort_by=created_at` |
| [#137] | MUST | Stick to **conventional query parameters**: `q`, `sort`, `cursor`, `limit`, `offset`, `fields`, `embed`, `expand` |

## JSON Property Names

| Rule | Level | Constraint |
|------|-------|------------|
| [#118] | MUST | Property names MUST be **snake_case** (never camelCase): `order_id`, `created_at` |
| [#120] | SHOULD | **Pluralize array names**: `items`, `line_items`, `addresses` |
| [#235] | SHOULD | Use **naming convention for date/time properties**: suffix `_at` for `date-time`, `_date` for `date`, `_duration` for durations, `_period` for periods |
| [#174] | MUST | Use **common field names and semantics**: `id`, `created_at`, `modified_at`, `type` |
| [#249] | MUST | Use the **common address fields**: follow the standard address schema |

## Enum Values

| Rule | Level | Constraint |
|------|-------|------------|
| [#240] | SHOULD | Declare enum values using **UPPER_SNAKE_CASE**: `ORDER_PLACED`, `PAYMENT_RECEIVED` |
| [#112] | SHOULD | Use **open-ended list of values** (`x-extensible-enum`) for evolvable enums that may grow over time |

## API Identifiers and Hostnames

| Rule | Level | Constraint |
|------|-------|------------|
| [#215] | MUST | Provide **API identifiers** — `x-api-id` as a UUID in `info` |
| [#103] | MUST | Write APIs using **U.S. English** |
| [#213] | MUST | Follow naming convention for **event type names**: `<organization>.<functional-domain>.<event-name>.<version>` |
