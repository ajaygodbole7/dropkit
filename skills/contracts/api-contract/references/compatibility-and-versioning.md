# Compatibility and Versioning

> Zalando RESTful API Guidelines — compatibility and versioning reference.
> Source: https://opensource.zalando.com/restful-api-guidelines/ (CC-BY-4.0, Zalando SE)

## No Breaking Changes [#106]

MUST NOT break backward compatibility of published APIs. Breaking changes include:

- Removing or renaming a field, endpoint, or enum value
- Changing a field's type or format
- Adding a new required field to request bodies
- Changing the semantics of an existing field
- Restricting previously allowed values
- Changing the response status code for a given scenario

## Compatible Extensions [#107]

SHOULD prefer compatible extensions:

- Adding optional fields to response bodies
- Adding new endpoints
- Adding new optional query parameters
- Adding new enum values (if using `x-extensible-enum`)
- Adding new optional headers

## Conservative Design [#109]

SHOULD design APIs conservatively — be strict in what you send, tolerant in what you accept (Postel's Law applied to APIs).

## Tolerant Reader [#108]

MUST prepare clients to accept compatible API extensions:

- Clients MUST ignore unknown fields in responses
- Clients MUST NOT depend on the order of fields
- Clients MUST be prepared for new enum values

## Open for Extension [#111]

MUST treat OpenAPI specification as open for extension by default — new properties may appear in responses at any time. Clients must handle this gracefully.

## Extensible Enums [#112]

SHOULD use open-ended list of values for enums that are expected to grow:

```yaml
status:
  type: string
  x-extensible-enum:
    - OPEN
    - CLOSED
    - CANCELLED
  description: >
    Order status. This is an extensible enum — new values may be added
    in the future. Clients must handle unknown values gracefully.
```

Use `x-extensible-enum` instead of `enum` for values that may evolve. Reserve `enum` for truly fixed sets (e.g., HTTP methods, boolean-like values).

## Avoid Versioning [#113]

SHOULD avoid versioning — use compatible extensions instead. Only version when a breaking change is truly unavoidable.

## Media Type Versioning [#114]

MUST use media type versioning if versioning is necessary:

```
Content-Type: application/vnd.example.order+json;version=2
Accept: application/vnd.example.order+json;version=2
```

## No URL Versioning [#115]

MUST NOT use URL versioning — never put `/v1/`, `/v2/` in URLs:

```
# Wrong
GET /v1/orders
GET /v2/orders

# Correct — use content negotiation
GET /orders
Accept: application/vnd.example.order+json;version=2
```

## Deprecation [#187][#185][#186][#188][#189][#190][#191]

### Reflect in Specification [#187]

MUST reflect deprecation in API specifications using the `deprecated: true` flag on operations and schemas.

### Client Approval Required [#185]

MUST obtain approval of clients before API shut down — never remove an API without coordinating with consumers.

### External Partner Consent [#186]

MUST collect external partner consent on deprecation time span — agree on sunset timeline with all external consumers.

### Monitor Deprecated Usage [#188]

MUST monitor usage of deprecated API scheduled for sunset — track call volume to ensure consumers are migrating.

### Deprecation Headers [#189]

SHOULD add `Deprecation` and `Sunset` headers to responses:

```
Deprecation: true
Sunset: Sat, 01 Mar 2025 00:00:00 GMT
```

### Monitor Headers [#190]

SHOULD add monitoring for `Deprecation` and `Sunset` headers — clients should monitor these headers and alert on upcoming sunsets.

### No New Usage [#191]

MUST NOT start using deprecated APIs — if an API is deprecated, do not build new integrations against it.
