# Review Execution Contract

Status: Proposed canonical contract; accepted source boundaries and proposed
async shape are separated below.

Owner: Finance Backend for product lifecycle; AI Server for technical AI
execution
Consumers: Finance FE and AI Runtime
Related Jira: `RPL-81`, `RPL-50`, `RPL-68`

## Accepted source-level boundaries

The current verified contract accepts the following:

1. `ReviewExecution`, progress, and transient Review Result are temporary
   operational state. They are not automatically durable user records.
2. The Finance core loop is:

   ```text
   Question
     → ReviewExecution / ChecklistResult
     → transient Review Result
     → explicit user Journal save
     → optional immutable DecisionContextSnapshot
     → Retrospective
   ```

3. Phase 1 uses:

   ```text
   Static / Versioned Checklist
   + finance-harness-docs RAG
   + Cheap LLM Analysis Guidance
   ```

4. Stable RAG knowledge is methodology/analysis knowledge. It is not current
   financial evidence.
5. When no Financial Evidence Provider is available:

   ```text
   Fact = NOT_APPLICABLE
   Current-market Inference = NOT_APPLICABLE
   ```

6. A Phase 1 guidance sentence must not be presented as a current fact,
   unverified evidence, or a recommendation.
7. Finance Backend owns product lifecycle, policy validation, result meaning,
   and explicit durable-save semantics. AI Server owns RAG, embedding,
   retrieval, model routing, and model execution.

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
| Accepted | Phase 1 checklist + stable knowledge RAG + cheap guidance; no current evidence in Phase 1; transient Review lifecycle; REST/polling client boundary; Finance/AI ownership separation |
| Proposed | NATS JetStream request/result boundary for a later asynchronous Review Runtime |
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
- [Finance Review Engine architecture](https://github.com/ranikun-labs/finance-harness-docs/blob/main/04-backend-architecture/finance-review-engine-architecture.md)
- [Finance core-loop contract](https://github.com/ranikun-labs/finance-harness-docs/blob/main/04-backend-architecture/finance-core-loop-and-domain-contract.md)
- [Foundation service communication policy](https://github.com/ranikun-labs/harness-foundation-docs/blob/main/docs/architecture/backend-service-foundation/service-communication-policy.md)
