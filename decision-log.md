# Finance Architecture Decision Log

This is the inventory for the Finance Architecture canonical repository. It
records the state of each decision without manufacturing a merge SHA or
promoting a deferred value to a contract.

## ADR inventory

| ID | Title | Status | Jira | Related repositories | Canonical Merge SHA | Superseded |
|---|---|---|---|---|---|---|
| ADR-001 | Journal Lifecycle | Proposed | `RPL-81` | `harness-foundation-docs`, `finance-harness-docs`, `finance-harness-fe`, `finance-harness-be` | Pending until merge | No |
| ADR-002 | Shared Identity Consumer Boundary | Proposed | `RPL-81` | `harness-foundation-docs`, `finance-harness-docs`, `finance-harness-fe`, `finance-harness-be` | Pending until merge | No |
| ADR-003 | Review Engine / AI Runtime Boundary | Proposed | `RPL-81` | `harness-foundation-docs`, `finance-harness-docs`, `finance-harness-fe`, `finance-harness-be`, AI Runtime | Pending until merge | No |
| ADR-004 | Production Activation Gates | Proposed | `RPL-81` | `harness-foundation-docs`, `finance-harness-fe`, `finance-harness-be`, Shared Gateway / Identity, AI Runtime | Pending until merge | No |

Finance ADR numbering starts at `ADR-001` in this repository. It is a
Finance-specific namespace and does not collide with Foundation ADR-0011
through ADR-0017.

## Confirmed decisions carried into this bootstrap

The following statements are recorded from previously verified sources; this
bootstrap does not invent a new implementation detail:

1. The Finance core loop is `검토 → 판단 기록 → 복기 ↺ 다음 검토 개선`.
2. A Journal is a user-owned durable reasoning record.
3. `ReviewExecution` is temporary operational state, not a durable user record.
4. `DecisionContextSnapshot` is an immutable, explicit-save snapshot.
5. `Retrospective` is a separate timestamped durable record.
6. `POST /finance/journals` is a Finance service-local product API, not the
   final published Shared Gateway route.
7. Journal time is sent as user local wall-clock `occurredAt` plus an IANA
   `timeZone`; the FE does not pre-convert it to UTC.
8. The Finance Backend resolves the local date-time and zone for persistence
   time semantics.
9. Journal IDs are backend-application-owned UUIDv7 values; the FE consumes an
   opaque nonblank ID.
10. Client payloads do not carry `userId`, `identityUserId`, or `actorId`.
11. Finance consumes shared identity through `CurrentUserPort` and an opaque,
    stable `identityUserId` semantic.
12. Finance does not access Shared Identity databases or Redis directly and
    does not copy another product's auth model.
13. Finance Backend does not execute LLM, embedding, vector retrieval, or
    direct vector-database access as product-side implementation.
14. Review Phase 1 is Static/Versioned Checklist + `finance-harness-docs` RAG +
    Cheap LLM Analysis Guidance; without a financial evidence provider, Fact
    and Current-market Inference are `NOT_APPLICABLE`.
15. NATS is not part of Journal Create S1/S2 and requires a concrete Review
    Runtime use case and an accepted follow-up contract.
16. Finance starts as a product boundary with a modular-monolith preference;
    this repository does not authorize pre-splitting or runtime scaffolding.

## Contract inventory

| Contract | Status | Owner / consumer | Scope |
|---|---|---|---|
| [`Journal Create`](./contracts/journal-create.md) | Proposed canonical record; source alignment confirmed | Finance Backend / Finance FE | Service-local `POST /finance/journals`, local time semantics, ownership and ID boundary |
| [`Identity Consumer`](./contracts/identity-consumer.md) | Proposed canonical record; semantic boundary confirmed | Finance Application / Shared Identity adapter | `CurrentUserPort` and opaque stable `identityUserId`; transport details separated |
| [`Review Execution`](./contracts/review-execution.md) | Proposed canonical record; phase/lifecycle boundary confirmed | Finance Backend / FE / AI Server | Phase 1 processing, result lifecycle, async target, RAG/evidence separation |

These are human-readable contracts. OpenAPI, AsyncAPI, protobuf, schema
registry, and code generation are outside this bootstrap.

## Deferred decision register

The following values are intentionally not decided here:

### Shared Identity and Gateway

- Shared Identity host;
- Gateway host;
- final Gateway Finance route;
- OAuth entrypoint;
- issuer;
- audience;
- JWKS endpoint;
- refresh/session mechanism;
- browser authentication transport;
- Identity Account schema;
- ProductClient persistence;
- Identity Redis semantics.

### Review transport and retention

- exact NATS stream;
- exact NATS subject;
- NATS retention;
- NATS retry policy;
- NATS delivery/acknowledgement policy;
- production retention.

### AI and evidence

- AI vendor;
- embedding vendor/model;
- vector database implementation;
- Market Data vendor;
- Evidence provider.

No host, URL, token, credential, provider, model, stream, subject, or retention
value may be inferred merely to make a diagram look complete.

Foundation-level deferred topics such as Commerce, a central Audit consumer,
Shared AI physicalization, Kubernetes adoption, and AWS region remain owned by
the Foundation decisions; this repository does not re-decide them.

## Existing `finance-harness-docs` architecture references

The following existing documents under
`finance-harness-docs/04-backend-architecture/` were checked and are preserved:

| Existing document | Reference role |
|---|---|
| `finance-backend-architecture.md` | Domain ownership, identity boundary, Finance storage, and AI boundary reference |
| `finance-review-engine-architecture.md` | Review lifecycle, phase boundary, and transport reference |
| `finance-core-loop-and-domain-contract.md` | Product/FE-BE domain contract reference |
| `finance-mvp-implementation-scope.md` | V1 build ordering and activation-boundary reference |

This bootstrap uses `Reference, don't migrate.` No existing document is moved,
deleted, rewritten, or copied wholesale. A future migration/deprecation path is
a follow-up decision, not a hidden change in this PR.

## Merge SHA policy

`Canonical Merge SHA` remains `Pending until merge` for every ADR until an
independent review is complete and this PR is merged. Neither the initial base
SHA nor a branch HEAD is a substitute for the canonical merge SHA.
