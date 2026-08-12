# Finance Architecture Decision Log

This is the inventory for the Finance Architecture canonical repository. It
records the state, lineage, and canonical merge SHA of each decision without
promoting a deferred value to a contract.

## ADR inventory

| ID | Title | Status | Jira | Related repositories | Canonical Merge SHA | Superseded |
|---|---|---|---|---|---|---|
| ADR-001 | Journal Lifecycle | Accepted | RPL-81 | `harness-foundation-docs`, `finance-harness-docs`, `finance-harness-fe`, `finance-harness-be` | 07b598375a6ce4c84c3137d7c4a9b082c1c7e2c8 | No |
| ADR-002 | Shared Identity Consumer Boundary | Accepted | RPL-81 | `harness-foundation-docs`, `finance-harness-docs`, `finance-harness-fe`, `finance-harness-be` | 07b598375a6ce4c84c3137d7c4a9b082c1c7e2c8 | No |
| ADR-003 | Review Engine / AI Runtime Boundary | Accepted | RPL-81 | `harness-foundation-docs`, `finance-harness-docs`, `finance-harness-fe`, `finance-harness-be`, AI Runtime | 07b598375a6ce4c84c3137d7c4a9b082c1c7e2c8 | No |
| ADR-004 | Production Activation Gates | Accepted | RPL-81 | `harness-foundation-docs`, `finance-harness-fe`, `finance-harness-be`, Shared Gateway / Identity, AI Runtime | 07b598375a6ce4c84c3137d7c4a9b082c1c7e2c8 | No |

Finance ADR numbering starts at `ADR-001` in this repository. It is a
Finance-specific namespace and does not collide with Foundation ADR-0011
through ADR-0017.

## Decision lineage

| ADR | Source / lineage | Current meaning |
|---|---|---|
| ADR-001 | New RPL-81 Finance Main decision with an explicit delta from the merged Finance lifecycle semantics in `finance-core-loop-and-domain-contract.md`, Sections 8–11. | Proposed lifecycle decision; not a previously accepted normalization. Upon acceptance/merge it supersedes only the prior AnalysisResult durability/reference semantics, not the whole historical document. |
| ADR-002 | Finance consumer boundary constrained by Foundation identity, gateway, and data-ownership decisions. | Proposed application seam; transport and platform values remain deferred unless upstream accepted. |
| ADR-003 | New RPL-81 Finance runtime composition constrained by Foundation AI/communication boundaries, ADR-001, and current Finance product requirements. | Proposed async composition; no pre-existing verified remote Review Engine architecture is claimed. |
| ADR-004 | New RPL-81 activation decision aligned to the current Foundation ADR-0017 sequence and current FE/BE evidence. | Proposed production gates; implementation evidence does not activate a runtime. |

## Existing merged source constraints

The current merged Finance core-loop source at
[`5326977691b420f573a469c0bb7635de26fe2aba`](https://github.com/ranikun-labs/finance-harness-docs/blob/5326977691b420f573a469c0bb7635de26fe2aba/04-backend-architecture/finance-core-loop-and-domain-contract.md)
records the previous lifecycle: `AnalysisResult` is a separate resource,
completed results are immutable, and Journal references the completed result
after an explicit save. ADR-001 records the proposed RPL-81 delta instead of
claiming that this source already contains the new transient/durable model.

The Foundation sources continue to constrain service communication, shared AI,
data ownership, and platform migration sequencing. The current Finance FE/BE
repositories provide implementation evidence for the activation gates, but
their branch or commit state is not a substitute for an Architecture merge
SHA.

## Existing Finance contract constraints

The following non-lifecycle constraints remain part of the current Finance
contract surface:

1. The Finance core loop is `검토 → 판단 기록 → 복기 ↺ 다음 검토 개선`.
2. `POST /finance/journals` is a Finance service-local product API, not the
   final published Shared Gateway route.
3. Journal time uses the user's local wall-clock `occurredAt` plus an IANA
   `timeZone`; the FE does not pre-convert it to UTC, and the BE resolves the
   local date-time and zone for persistence time semantics.
4. Journal IDs are backend-application-owned UUIDv7 values; the FE consumes an
   opaque nonblank ID.
5. Client payloads do not carry `userId`, `identityUserId`, or `actorId` as an
   ownership authority.
6. Finance consumes Shared Identity through `CurrentUserPort` and an opaque,
   stable `identityUserId`; it does not access Shared Identity databases or
   Redis directly or copy another product's auth model.
7. Finance Backend does not execute LLM, embedding, vector retrieval, or direct
   vector-database access as product-side implementation.
8. NATS is not part of Journal Create S1/S2 and requires a concrete Review
   Runtime use case and an accepted follow-up contract.

## Proposed Finance decisions under RPL-81

The following decisions are proposed in this PR:

1. `ReviewExecution` is transient operational state, while Journal is a
   durable user-owned reasoning record.
2. An explicit user save creates an immutable `DecisionContextSnapshot`.
3. A `Retrospective` is a separate timestamped durable record.
4. The invariant is:

   ```text
   Transient AI Review
   !=
   Durable User Record
   ```

5. Finance consumes Shared Identity through `CurrentUserPort` and an opaque,
   stable `identityUserId`; transport details remain separate and deferred.
6. Finance Backend does not execute LLM, embedding, vector retrieval, or direct
   vector-database access as product-side implementation.
7. Review Phase 1 is proposed as Static/Versioned Checklist +
   `finance-harness-docs` RAG + Cheap LLM Analysis Guidance. Without a
   Financial Evidence Provider, Fact and Current-market Inference are
   `NOT_APPLICABLE`.
8. NATS is not part of Journal Create S1/S2. A later Review Runtime may use the
   proposed logical boundary only after its exact contract and activation gates
   are accepted.
9. Finance starts as a product boundary with a modular-monolith preference;
   this repository does not authorize pre-splitting or runtime scaffolding.

## Contract inventory

| Contract | Status | Owner / consumer | Scope |
|---|---|---|---|
| [`Journal Create`](./contracts/journal-create.md) | Accepted canonical record; current service-local contract evidence | Finance Backend / Finance FE | Service-local `POST /finance/journals`, local time semantics, ownership and ID boundary |
| [`Identity Consumer`](./contracts/identity-consumer.md) | Accepted canonical record; semantic boundary recorded | Finance Application / Shared Identity adapter | `CurrentUserPort` and opaque stable `identityUserId`; transport details separated |
| [`Review Execution`](./contracts/review-execution.md) | Accepted canonical record; upstream constraints and Finance proposal separated | Finance Backend / FE / AI Server | Phase 1 processing, proposed lifecycle, async target, RAG/evidence separation |

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

The following four tracked documents under
`finance-harness-docs/04-backend-architecture/` were checked at Finance
`main` SHA `5326977691b420f573a469c0bb7635de26fe2aba` and are preserved:

| Existing document | Reference role |
|---|---|
| `finance-backend-architecture.md` | Domain ownership, identity boundary, Finance storage, and AI boundary reference |
| `finance-core-loop-and-domain-contract.md` | Merged product/domain contract and previous AnalysisResult/Journal reference lifecycle input |
| `finance-mvp-build-and-verification-plan.md` | Build ordering, explicit save/read coverage, idempotency verification, and release-boundary reference |
| `finance-mvp-implementation-scope.md` | V1 implementation scope, sequencing, and non-goal reference |

This bootstrap uses `Reference, don't migrate.` No existing document is moved,
deleted, rewritten, or copied wholesale. The new ADRs do not claim that all
historical semantics in those documents agree with the RPL-81 proposal. A
future migration/deprecation path is a follow-up decision, not a hidden change
in this PR.

## Merge SHA policy

`Canonical Merge SHA` is
`07b598375a6ce4c84c3137d7c4a9b082c1c7e2c8` for ADR-001 through ADR-004,
established by the initial canonicalization PR #1. This metadata
reconciliation PR must not replace that value with its own merge SHA. Future
ADRs retain `Pending until merge` until their own canonical merge.
