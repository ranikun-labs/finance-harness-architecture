# Data Lifecycle and Ownership

Status: Proposed canonical data-boundary view.

This document records ownership and lifecycle semantics without inventing a
database schema, table name, migration, or provider implementation. Finance
services own their data; cross-service access uses an accepted contract rather
than direct database access.

## Ownership matrix

| Data concept | Owner | Durable / transient | Mutation authority | Source of truth | Cross-service access rule |
|---|---|---|---|---|---|
| Journal | Finance Backend; user-authored content | Durable | Explicit user save/update through Finance product contract, subject to user-data policy | Finance-owned product state | Finance API with `CurrentUserPort` ownership check; no Shared Identity or AI database access |
| ReviewExecution | Finance Backend product lifecycle with AI Runtime execution participation | Transient operational state | Finance orchestration and runtime adapters within the Review contract | Finance Review Execution state for product lifecycle | REST/polling and, if adopted later, async Review contract; no direct client/provider database access |
| DecisionContextSnapshot | Finance Backend; user-owned immutable snapshot | Durable, immutable | Created only by explicit user save/opt-in; no later mutation | Snapshot captured by Finance at Journal save time | Finance API/read model; selected values/provenance only, not a direct reference to temporary runtime state |
| Retrospective | Finance Backend; user-authored record attached to a Journal | Durable, timestamped, separate from Journal | Explicit user retrospective operation through Finance contract | Finance-owned product state | Finance API with Journal ownership check; no cross-service cascade or automatic market-result linkage |
| Stable Knowledge | `finance-harness-docs` authoring boundary; AI Runtime ingestion/projection consumer | Durable and versioned at the corpus; runtime projection may be refreshed | Knowledge repository owners author; ingestion pipeline projects accepted revisions | `finance-harness-docs` canonical corpus | AI Runtime consumes through ingestion/projection; Finance must not treat it as current financial evidence |
| Current Evidence | Finance Evidence Layer consuming a future accepted provider boundary | Transient/freshness-bound by default; durable retention is not yet decided | Provider source plus Finance normalization/orchestration within an accepted Evidence contract | Future accepted source/provider; none selected now | Finance/AI consume normalized evidence bundles; never direct FE-to-provider or RAG substitution |
| Identity | Shared Identity | Durable in its owning service; Finance reference is not an identity copy | Shared Identity only | Shared Identity | Finance consumes opaque stable `identityUserId` through `CurrentUserPort`; no direct database/Redis/table access |

## Lifecycle distinctions

```text
ReviewExecution
  = temporary operational serving and processing state

Journal
  = user-owned durable reasoning record

DecisionContextSnapshot
  = explicit-save immutable value copy

Retrospective
  = separate timestamped durable user record
```

Completion of a ReviewExecution does not create a Journal. A Decision Context
snapshot is not a mutable pointer to Redis, a broker message, or a full AI
transcript. If current evidence or stable knowledge materially informs a saved
snapshot, the required provenance is copied according to the accepted Finance
contract rather than extending the temporary object's lifetime by accident.

## Ownership invariants

- Finance-owned storage is the source of truth for Finance product state.
- Shared Identity owns identity accounts and authentication state; Finance owns
  only its product-side opaque reference.
- `finance-harness-docs` owns stable knowledge authoring and versioning.
- Current Evidence is distinct from Stable Knowledge and remains unavailable as
  a production source until an Evidence Provider contract is accepted.
- A service must not mutate another service's database, create cross-service
  foreign keys, perform OLTP cross-service joins, or cascade deletion across a
  service boundary.
- Retention, deletion, and lifecycle transitions are owned by the data owner.
- Durable data is minimized; temporary execution, prompt, retrieval, and
  provider details are not automatically promoted to durable user records.

## Data flow

```text
Shared Identity
  → opaque identityUserId
  → CurrentUserPort
  → Finance-owned ownership check
  → Journal / ReviewExecution / Snapshot / Retrospective

finance-harness-docs
  → AI Runtime ingestion / RAG
  → Review guidance (stable knowledge classification)

Future Evidence Provider
  → Evidence normalization / provenance
  → Review phase that accepts current evidence
  → explicit snapshot value copy when the user saves context
```

## Physical persistence boundary

The Foundation target allows a Finance-owned logical database within a shared
physical PostgreSQL cluster, but physical co-location does not merge ownership.
This repository does not define schemas, tables, indexes, migrations,
partitioning, Redis keys, broker retention, or deployment configuration.

ReviewExecution and other short-lived state may use a runtime state mechanism
when a later implementation decision accepts it. Durable Journal and user-owned
records must remain under the Finance data owner and user-data policy.

## References

- [ADR-001 Journal Lifecycle](../adr/ADR-001-journal-lifecycle.md)
- [ADR-002 Identity Consumer Boundary](../adr/ADR-002-shared-identity-consumer-boundary.md)
- [ADR-003 Review Engine / AI Runtime Boundary](../adr/ADR-003-review-engine-ai-runtime-boundary.md)
- [Ownership map](../architecture/ownership-map.md)
- [Foundation ADR-0013](https://github.com/ranikun-labs/harness-foundation-docs/blob/main/docs/adr/ADR-0013-target-deployment-and-data-boundaries.md)
- [Foundation database ownership policy](https://github.com/ranikun-labs/harness-foundation-docs/blob/main/docs/architecture/backend-service-foundation/database-ownership-and-reference-policy.md)
- [Finance backend architecture lifecycle boundary](https://github.com/ranikun-labs/finance-harness-docs/blob/5326977691b420f573a469c0bb7635de26fe2aba/04-backend-architecture/finance-backend-architecture.md)
- [Finance MVP build and verification plan](https://github.com/ranikun-labs/finance-harness-docs/blob/5326977691b420f573a469c0bb7635de26fe2aba/04-backend-architecture/finance-mvp-build-and-verification-plan.md)
