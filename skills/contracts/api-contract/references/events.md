# Events

> Zalando RESTful API Guidelines — event rules reference (Chapters 19-21).
> Source: https://opensource.zalando.com/restful-api-guidelines/ (CC-BY-4.0, Zalando SE)

These rules apply when an API domain includes asynchronous event schemas (e.g., for event-driven architectures, webhooks, or message queues). Event schemas are treated as API contracts with the same rigor as REST endpoints.

## Chapter 19: Event Basics — Event Types

### Define Events Per API Guidelines [#208]

MUST define events compliant with overall API guidelines — event schemas follow the same naming, format, and compatibility rules as REST resources.

### Events Are Part of the Interface [#194]

MUST treat events as part of the service interface — event types are first-class API contracts, not implementation details. They must be designed, documented, and versioned.

### Schema Availability [#195]

MUST make event schema available for review — publish event schemas alongside REST API specifications.

### Event Type Registration [#197]

MUST specify and register events as event types — each distinct event must be declared as a named event type in a central registry or schema catalog.

### Event Type Naming [#213]

MUST follow naming convention for event type names:

```
<organization>.<functional-domain>.<event-name>.<version>
```

Example: `org.order-management.order-placed.v1`

- Use kebab-case for segments
- Include version as last segment

### Event Type Ownership [#207]

MUST indicate ownership of event types — every event type must have a clearly identified owning team or service.

### Compatibility Mode [#245]

MUST carefully define the compatibility mode — specify whether the event type schema uses `forward` or `full` compatibility. This determines what changes are considered breaking.

### OpenAPI Schema Conformance [#196]

MUST ensure event schema conforms to OpenAPI schema object — event payloads are defined using the same JSON Schema / OpenAPI schema format as REST request/response bodies.

### Avoid additionalProperties [#210]

SHOULD avoid `additionalProperties` in event type schemas — unlike REST responses (where unknown fields are tolerated), event schemas should be explicit about their structure.

### Semantic Versioning [#246]

MUST use semantic versioning of event type schemas — follow `MAJOR.MINOR.PATCH` for event type versions. Breaking changes require a new major version.

## Chapter 20: Event Basics — Event Categories

### Event Category Conformance [#198]

MUST ensure that events conform to an event category. The two standard categories are:

1. **General events** — signal steps in business processes (e.g., `order-placed`, `payment-received`)
2. **Data change events** — signal mutations to resources (e.g., `order-created`, `order-updated`, `order-deleted`)

### Mandatory Event Metadata [#247]

MUST provide mandatory event metadata. Every event must include:

| Field | Type | Description |
|-------|------|-------------|
| `metadata.eid` | `string(uuid)` | Unique event identifier |
| `metadata.occurred_at` | `string(date-time)` | When the event occurred |
| `metadata.event_type` | `string` | Registered event type name |
| `metadata.flow_id` | `string` | Correlation ID for tracing |
| `metadata.partition` | `string` | Partition key for ordering |

### Unique Event Identifiers [#211]

MUST provide unique event identifiers — the `metadata.eid` field must be a globally unique UUID for deduplication.

### General Events for Business Steps [#201]

MUST use general events to signal steps in business processes:

```yaml
OrderPlacedEvent:
  type: object
  required:
    - metadata
    - order_id
    - status
  properties:
    metadata:
      $ref: '#/components/schemas/EventMetadata'
    order_id:
      type: string
    status:
      type: string
      x-extensible-enum:
        - PLACED
        - CONFIRMED
```

### General Event Ordering [#203]

SHOULD provide explicit event ordering for general events — include a sequence number or logical timestamp to enable consumers to process events in order.

### Data Change Events [#202]

MUST use data change events to signal mutations:

- `data_op` field indicates `CREATE`, `UPDATE`, or `DELETE`
- `data` field contains the resource snapshot (for CREATE/UPDATE) or the resource ID (for DELETE)
- `data_type` field identifies the resource type

```yaml
OrderDataChangeEvent:
  type: object
  required:
    - metadata
    - data_op
    - data_type
    - data
  properties:
    metadata:
      $ref: '#/components/schemas/EventMetadata'
    data_op:
      type: string
      enum: [CREATE, UPDATE, DELETE]
    data_type:
      type: string
      example: order
    data:
      $ref: '#/components/schemas/Order'
```

### Data Change Event Ordering [#242]

MUST provide explicit event ordering for data change events — include a monotonically increasing sequence number or use the event store's built-in ordering.

### Hash Partition Strategy [#204]

SHOULD use the hash partition strategy for data change events — partition by the resource ID to ensure all changes to the same resource land on the same partition (guaranteeing per-resource ordering).

## Chapter 21: Event Design

### Sensitive Data [#200]

SHOULD avoid writing sensitive data to events — PII, credentials, and other sensitive data should be referenced by ID rather than embedded in event payloads.

### Duplicate Robustness [#214]

MUST be robust against duplicates when consuming events — consumers must implement idempotent processing using the `metadata.eid` for deduplication.

### Idempotent Out-of-Order Processing [#212]

SHOULD design for idempotent out-of-order processing — consumers should handle events arriving out of order gracefully. Use event ordering fields to detect and reconcile out-of-order delivery.

### Useful Business Resources [#199]

MUST ensure that events define useful business resources — event payloads should contain enough information for consumers to act without additional API calls.

### Match API Resources [#205]

SHOULD ensure that data change events match the API's resources — the `data` payload should mirror the corresponding REST resource schema.

### Backward Compatibility [#209]

MUST maintain backwards compatibility for events — the same rules as REST API compatibility apply. Never remove fields, change types, or alter semantics in a non-major version.
