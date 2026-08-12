# Review Execution Contract

Status: Proposed canonical contract; previous remote reality, upstream
constraints, and the RPL-81 proposal are separated below.

Owner: Finance Backend for product lifecycle; AI Server for technical AI
execution
Consumers: Finance FE and AI Runtime
Related Jira: RPL-81
Implementation Tracking: RPL-50, RPL-68

## Previous merged Finance reality

The merged Finance core-loop contract records the previous product lifecycle:

```text
Question
  → ChecklistResult
  → AnalysisRequest
  → AnalysisResult
  → explicit Journal save
```

Its AnalysisResult section defines the completed result as immutable and its
Journal section defines Journal as holding a reference to that completed
result. ADR-001 proposes a new lifecycle under RPL-81; this document must not
present the new lifecycle as an already accepted historical contract.

## Accepted upstream constraints

The current upstream constraints are:

- Foundation Shared AI and communication boundaries keep technical AI
  execution, RAG, retrieval, embedding, and model execution outside the
  Finance Backend product implementation.
- Stable RAG knowledge is methodology/analysis knowledge. It is not current
  financial evidence.
- When no Financial Evidence Provider is available:

  ```text
  Fact = NOT_APPLICABLE
  Current-market Inference = NOT_APPLICABLE
  ```

- A guidance sentence must not be presented as a current fact, unverified
  evidence, or a recommendation.
- NATS is not part of Journal Create S1/S2; any later broker use requires a
  concrete Review Runtime use case and an accepted follow-up contract.

## Proposed Finance contract under RPL-81

ADR-001 proposes that `ReviewExecution` and its transient result are temporary
operational state, that Journal is a durable user-owned reasoning record, that
an explicit save creates an immutable `DecisionContextSnapshot`, and that a
`Retrospective` is a separate timestamped durable record. Completing a Review
does not automatically save the full output to Journal.

The proposed Finance Phase 1 composition is:

```text
Static / Versioned Checklist
+ finance-harness-docs RAG
+ Cheap LLM Analysis Guidance
```

Finance Backend owns product lifecycle, policy validation, result meaning, and
explicit durable-save semantics. AI Server owns RAG, embedding, retrieval,
model routing, and model execution.

## Phase 1 client-facing transport

Phase 1 uses a Finance product-facing REST and polling shape:

```text
Frontend
  ↓ REST / Polling
Finance Backend
```

The exact endpoint, DTO, polling interval, and retention values are not fixed by
this bootstrap. The client observes product-level states such as available,
partial, blocked, failed, or completed output only when the corresponding safety
and validation boundary passes.

## Proposed asynchronous runtime boundary

For a later Review Runtime phase, the proposed logical flow is:

```text
Frontend
  ↓ REST / Polling
Finance Backend
  ↓ request message
NATS JetStream
  ↓
AI Server
  ↓ result message
NATS JetStream
  ↓
Finance Backend
  ↓ REST / Polling
Frontend
```

NATS is not part of Journal Create S1/S2. The proposed boundary does not mean
that NATS, the AI Server, or a production consumer is currently deployed.

## Ownership and safety rules

- Finance Backend selects the Review workflow and the applicable knowledge or
  evidence contract.
- AI Server must preserve provenance and must not turn stable RAG retrieval
  into current-market fact.
- Current evidence, when a future provider is accepted, is normalized into an
  evidence boundary with source and time semantics before grounded inference.
- Finance PolicyGuard and Output Validator remain required before a result is
  shown as complete.
- Provider or runtime failure may yield a truthful partial result; unsafe or
  unvalidated output is not complete.
- Completing a Review does not save the full output into a Journal.

## Accepted / proposed / deferred matrix

| State | Items |
|---|---|
| Accepted upstream constraints | Foundation AI/communication boundaries; stable-knowledge versus current-evidence distinction; `NOT_APPLICABLE` behavior when no evidence provider exists; no NATS dependency for Journal Create S1/S2 |
| Proposed under RPL-81 | ReviewExecution/transient-result lifecycle; explicit Journal save; DecisionContextSnapshot and Retrospective boundaries; Phase 1 composition; NATS JetStream request/result composition |
| Deferred | Exact stream, subject, retention, retry, acknowledgement/delivery, dead-letter and replay policy; async payload/schema; timeout values; AI vendor; embedding vendor/model; vector DB; Market Data/Evidence provider; production retention and runtime support |

## Failure and compatibility minimums

Before the proposed async boundary is activated, the owning contracts must
define:

- immutable request/result correlation;
- idempotent processing and duplicate handling;
- bounded retry and failure classification;
- acknowledgement and recovery behavior;
- compatibility/versioning behavior;
- retention, replay, and operational observability;
- partial-result and safety-validation semantics.

This document does not choose values for those fields.

## References

- [ADR-003 Review Engine / AI Runtime Boundary](../adr/ADR-003-review-engine-ai-runtime-boundary.md)
- [ADR-001 Journal Lifecycle](../adr/ADR-001-journal-lifecycle.md)
- [ADR-004 Production Activation Gates](../adr/ADR-004-production-activation-gates.md)
- [Runtime topology](../architecture/runtime-topology.md)
- [Finance core-loop contract](https://github.com/ranikun-labs/finance-harness-docs/blob/5326977691b420f573a469c0bb7635de26fe2aba/04-backend-architecture/finance-core-loop-and-domain-contract.md)
- [Finance MVP build and verification plan](https://github.com/ranikun-labs/finance-harness-docs/blob/5326977691b420f573a469c0bb7635de26fe2aba/04-backend-architecture/finance-mvp-build-and-verification-plan.md)
- [Foundation service communication policy](https://github.com/ranikun-labs/harness-foundation-docs/blob/69fceb267094756913f400f8b4045590bd0e8ac5/docs/architecture/backend-service-foundation/service-communication-policy.md)
- [Foundation ADR-0015](https://github.com/ranikun-labs/harness-foundation-docs/blob/69fceb267094756913f400f8b4045590bd0e8ac5/docs/adr/ADR-0015-platform-communication-messaging-scaling.md)
