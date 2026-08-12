# Journal Create Contract

Status: Accepted canonical cross-repository contract; FE/BE alignment was
confirmed in preflight.

Owner: Finance Backend
Consumer: Finance FE
Related Jira: `RPL-81`, `RPL-50`, `RPL-68`

## Scope

This is the current Finance service-local Product API contract. It is not the
final published Shared Gateway route.

```text
POST /finance/journals
```

The endpoint represents an explicit user save of a Journal. It does not create
a Journal automatically when a ReviewExecution completes.

## Request

The request is a tagged union. The following fields are the currently aligned
Finance FE/BE shape:

### Investment Journal

```json
{
  "type": "investment",
  "assetName": "...",
  "occurredAt": "YYYY-MM-DDThh:mm",
  "timeZone": "IANA/Zone_ID",
  "action": "...",
  "reasoning": "...",
  "emotion": "..."
}
```

`emotion` is optional and is omitted when it is not selected.

### Study Journal

```json
{
  "type": "study",
  "title": "...",
  "occurredAt": "YYYY-MM-DDThh:mm",
  "timeZone": "IANA/Zone_ID",
  "keyContent": "...",
  "openQuestions": ["..."]
}
```

The examples show the contract shape; the allowed business values remain owned
by the Finance domain contract and implementation validation.

## Time semantics

- `occurredAt` is the user's local wall-clock date and time.
- `timeZone` is the user's IANA Zone ID.
- The FE does not pre-convert `occurredAt` to UTC.
- The Backend resolves the local date-time with the supplied zone and determines
  the persistence Instant according to the Backend time policy.
- A time-zone value is part of the request semantics, not presentation-only
  metadata.

## Identity and ownership

The payload must not contain:

- `userId`;
- `identityUserId`;
- `actorId`.

The Backend obtains the current opaque identity through `CurrentUserPort` and
enforces Finance ownership there. A client-provided identifier must not become
an authorization input.

## Success response

On successful creation:

```text
201 Created
Location: /finance/journals/{journalId}
```

```json
{
  "journalId": "..."
}
```

`journalId` is a UUIDv7 generated and owned by the Backend Application. The FE
consumes it as an opaque, nonblank identifier and does not construct or
interpret its value.

## Boundary and activation notes

- The API is service-local. A final Gateway route is deferred.
- The FE adapter must treat only the canonical `201 Created` response as a
  successful create result and must not expose backend error details as domain
  truth.
- Production Journal Create activation is a separate gate. The existence of
  the FE adapter or this contract does not activate the production user flow.
- Persistence schema, error payload, authentication transport, idempotency
  representation, and deployment route are not invented by this document.

## References

- [ADR-001 Journal Lifecycle](../adr/ADR-001-journal-lifecycle.md)
- [ADR-002 Identity Consumer Boundary](../adr/ADR-002-shared-identity-consumer-boundary.md)
- [ADR-004 Production Activation Gates](../adr/ADR-004-production-activation-gates.md)
- [Finance FE request adapter](https://github.com/ranikun-labs/finance-harness-fe/blob/master/src/features/journal-new/adapter/journalCreateRequest.ts)
- [Finance FE HTTP adapter](https://github.com/ranikun-labs/finance-harness-fe/blob/master/src/features/journal-new/adapter/HttpJournalCreatePort.ts)
- [Finance FE local-time behavior](https://github.com/ranikun-labs/finance-harness-fe/blob/master/src/lib/date.ts)
- [Finance core-loop contract reference](https://github.com/ranikun-labs/finance-harness-docs/blob/main/04-backend-architecture/finance-core-loop-and-domain-contract.md)
