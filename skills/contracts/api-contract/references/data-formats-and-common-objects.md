# Data Formats and Common Objects

> Zalando RESTful API Guidelines — data formats and reusable schemas reference.
> Source: https://opensource.zalando.com/restful-api-guidelines/ (CC-BY-4.0, Zalando SE)

## Standard Data Formats [#238]

MUST use standard data formats. Every `type` + `format` pair must use a recognized OpenAPI / JSON Schema format.

## Number and Integer Formats [#171]

MUST define a `format` for every `number` and `integer` property:

| Type | Format | Range / Precision |
|------|--------|-------------------|
| `integer` | `int32` | -2^31 to 2^31-1 |
| `integer` | `int64` | -2^63 to 2^63-1 |
| `number` | `float` | IEEE 754 single |
| `number` | `double` | IEEE 754 double |
| `number` | `decimal` | Arbitrary precision (use for money, rates) |

## Date and Time [#169][#255]

MUST use ISO 8601 / RFC 3339 standard formats:

| Format | OpenAPI `format` | Example | When to use [#255] |
|--------|------------------|---------|---------------------|
| Date-time | `date-time` | `2024-03-11T14:30:00Z` | Events with time-of-day relevance |
| Date | `date` | `2024-03-11` | Calendar dates, birthdays, deadlines |

SHOULD select the appropriate one — prefer `date` when time is irrelevant [#255].

### Duration and Interval [#127]

SHOULD use standard formats for duration and interval:

- Duration: ISO 8601 duration string, e.g. `P3D`, `PT2H30M`
- Interval: ISO 8601 interval, e.g. `2024-01-01/2024-03-31`

## Country, Language, Currency [#170]

MUST use standard formats:

| Property | Standard | Format | Example |
|----------|----------|--------|---------|
| Country | ISO 3166-1 alpha-2 | `iso-3166-alpha-2` | `DE`, `US` |
| Language | ISO 639-1 | `iso-639-1` | `de`, `en` |
| Currency | ISO 4217 | `iso-4217` | `EUR`, `USD` |

## Content Negotiation [#244]

SHOULD use content negotiation if clients may choose from different resource representations (e.g., different media types or languages).

## UUIDs [#144]

SHOULD only use UUIDs if necessary — prefer shorter, URL-friendly IDs when uniqueness can be guaranteed at the application level. When used, format as `uuid`.

## JSON Payload Rules

### JSON as Default [#167]

MUST use JSON as the payload data interchange format for both request and response bodies.

### Non-JSON Media Types [#168]

MAY pass non-JSON media types using data-specific standard formats (e.g., `image/png`, `application/pdf`) — but JSON remains the default.

### Standard Media Types [#172]

SHOULD use standard media types: `application/json` for resources, `application/problem+json` for errors, `application/merge-patch+json` for PATCH.

### Single Schema for Read/Write [#252]

SHOULD design a single resource schema for both reading and writing. Use `readOnly` and `writeOnly` annotations to differentiate.

### Unicode Awareness [#250]

SHOULD be aware that some services may not fully support JSON/unicode — document encoding expectations.

## Null and Absence Semantics [#123]

MUST use same semantics for null and absent properties — both mean "field has no value." Do not assign different meanings to null vs. omitted.

## Boolean Properties [#122]

MUST NOT use null for boolean properties — boolean properties must always be `true` or `false`, never `null`. Mark them as `type: boolean` without nullable.

## Empty Arrays [#124]

SHOULD NOT use null for empty arrays — return `[]` instead of `null` for empty collections.

## Top-Level Objects [#110]

MUST always return JSON objects as top-level data structures — never return bare arrays at the top level. Wrap in an object:

```yaml
# Correct
OrderList:
  type: object
  properties:
    items:
      type: array
      items:
        $ref: '#/components/schemas/Order'

# Wrong — bare array
# type: array
# items:
#   $ref: '#/components/schemas/Order'
```

## Maps [#216]

SHOULD define maps using `additionalProperties`:

```yaml
Labels:
  type: object
  additionalProperties:
    type: string
  example:
    environment: production
    team: checkout
```

## Common Field Names [#174]

MUST use the standard common field names with their defined semantics:

| Field | Type | Description |
|-------|------|-------------|
| `id` | `string` | Unique resource identifier |
| `created_at` | `string(date-time)` | Creation timestamp |
| `modified_at` | `string(date-time)` | Last modification timestamp |
| `type` | `string` | Resource type discriminator |
| `etag` | `string` | Entity tag for optimistic locking |

## Money Object [#173]

MUST use the common Money object for all monetary values. Schema from Zalando `models/money-1.0.0.yaml`:

```yaml
Money:
  type: object
  required:
    - amount
    - currency
  properties:
    amount:
      type: number
      format: decimal
      description: >
        The amount describes unit and subunit of the currency in a single value,
        where the integer part (digits before the decimal point) is for the
        major unit and fractional part (digits after the decimal point) is for
        the minor unit.
      example: 99.95
    currency:
      type: string
      format: iso-4217
      description: 3 letter currency code as defined by ISO-4217
      example: EUR
```

## Address Object [#249]

MUST use the common address fields. Minimal address schema:

```yaml
Address:
  type: object
  properties:
    address_line_1:
      type: string
      description: Street address, P.O. box, company name
    address_line_2:
      type: string
      description: Apartment, suite, unit, building, floor
    city:
      type: string
    state:
      type: string
      description: State, province, region
    zip_code:
      type: string
    country:
      type: string
      format: iso-3166-alpha-2
      description: ISO 3166-1 alpha-2 country code
      example: DE
```

## Problem Object [#176]

MUST use RFC 9457 Problem Detail for all error responses. Schema from Zalando `models/problem-1.0.1.yaml`:

```yaml
Problem:
  type: object
  properties:
    type:
      type: string
      format: uri-reference
      description: >
        A URI reference that uniquely identifies the problem type only in the
        context of the provided API. Opposed to the specification in RFC-9457,
        it is neither recommended to be dereferenceable and point to a
        human-readable documentation nor globally unique for the problem type.
      default: 'about:blank'
      example: '/some/uri-reference'
    title:
      type: string
      description: >
        A short summary of the problem type. Written in English and readable
        for engineers, usually not suited for non technical stakeholders and
        not localized.
      example: some title for the error situation
    status:
      type: integer
      format: int32
      description: >
        The HTTP status code generated by the origin server for this occurrence
        of the problem.
      minimum: 100
      maximum: 600
      exclusiveMaximum: true
    detail:
      type: string
      description: >
        A human readable explanation specific to this occurrence of the
        problem that is helpful to locate the problem and give advice on how
        to proceed. Written in English and readable for engineers, usually not
        suited for non technical stakeholders and not localized.
      example: some description for the error situation
    instance:
      type: string
      format: uri-reference
      description: >
        A URI reference that identifies the specific occurrence of the problem,
        e.g. by adding a fragment identifier or sub-path to the problem type.
        May be used to locate the root of this problem in the source code.
      example: '/some/uri-reference#specific-occurrence-context'
```
