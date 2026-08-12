# Finance Runtime Topology

Status: Proposed canonical view; logical target separated from current
implementation evidence.

## Current implementation boundary

The Journal Create contract is a Finance service-local product API consumed by
the FE adapter:

```text
Finance FE
  ↓ service-local REST contract
Finance Backend
```

This does not declare a final Shared Gateway route or a production user flow.
The FE has a client port/adapter and the backend contract is recorded in the
verified Finance planning sources; backend deployment and production activation
remain separate gates.

For the initial Journal Create phases, there is no NATS dependency:

```text
Journal Create S1/S2
Finance FE ──REST──► Finance Backend
                     ✕ NATS
```

The cross-service identity semantic is also not a per-request direct database
lookup. Finance consumes the current principal through `CurrentUserPort`; the
transport and platform adapter remain separate contracts.

## Review Runtime target

The logical asynchronous Review Runtime target is:

```text
Frontend
  ↓ REST / Polling
Finance Backend
  ↓ NATS JetStream boundary
AI Server
  ↓ NATS result boundary
Finance Backend
  ↓ REST / Polling
Frontend
```

This is a target Review Runtime shape, not a declaration that the broker,
consumer, or AI Server is currently deployed. NATS adoption is gated by the
Foundation trigger and a Review-specific accepted contract.

## Runtime phases

| Phase | Transport shape | What is known | What is not claimed |
|---|---|---|---|
| Journal Create S1/S2 | REST service-local API | `POST /finance/journals`, local wall-clock plus IANA zone, backend-owned Journal ID | Final Gateway route, production activation, or backend deployment |
| Review Phase 1 | REST + polling | Static/versioned checklist, `finance-harness-docs` RAG, and cheap LLM guidance; no current financial evidence | NATS, current-market facts, or production AI runtime |
| Later Review async phase | REST/polling at the client with NATS between Finance Backend and AI Server | Async boundary can decouple execution and result handling after contract acceptance | Stream, subject, retry, retention, delivery, and provider values |
| Production activation | Depends on accepted upstream contracts and runtime evidence | Activation is a separate decision gate | Any automatic promotion from UI or adapter existence |

## Ownership along the flow

```text
Frontend
  = interaction and client transport

Finance Backend
  = product API, ownership checks, lifecycle, orchestration,
    durable product state, and user-visible result contract

NATS JetStream
  = asynchronous Review Runtime transport only

AI Server
  = knowledge ingestion, RAG, embedding, retrieval, model routing,
    and model execution
```

Finance Backend must not use a direct LLM SDK, product-side embedding, product-
side vector retrieval, or direct vector-database access as a substitute for the
AI Runtime boundary.

## Failure and state boundary

- The client observes Review progress through the product contract; internal
  provider or broker details are not promoted to the client contract.
- Review execution is assumed to be temporary operational state. A completed
  execution does not create a Journal automatically.
- A provider or runtime partial failure may leave a `PARTIAL` result when the
  remaining sections pass the Finance safety boundary.
- A result that fails PolicyGuard or Output Validation is not presented as a
  completed user-visible result.
- Retry, deduplication, replay, retention, and delivery behavior must be
  specified before production asynchronous adoption.

## Deferred topology values

No value is selected here for:

- Gateway host or final Finance route;
- identity transport, issuer, audience, JWKS, or browser auth mechanism;
- NATS stream, subject, retention, retry, acknowledgement, or delivery policy;
- AI vendor, embedding model/vendor, vector database, or current-evidence
  provider;
- production retention and deployment infrastructure.

## References

- [`harness-foundation-docs` ADR-0015](https://github.com/ranikun-labs/harness-foundation-docs/blob/main/docs/adr/ADR-0015-platform-communication-messaging-scaling.md)
- [`harness-foundation-docs` ADR-0017](https://github.com/ranikun-labs/harness-foundation-docs/blob/main/docs/adr/ADR-0017-shared-platform-gateway-identity-physicalization.md)
- [`finance-harness-docs` Review Engine architecture](https://github.com/ranikun-labs/finance-harness-docs/blob/main/04-backend-architecture/finance-review-engine-architecture.md)
- [`finance-harness-docs` core loop contract](https://github.com/ranikun-labs/finance-harness-docs/blob/main/04-backend-architecture/finance-core-loop-and-domain-contract.md)
