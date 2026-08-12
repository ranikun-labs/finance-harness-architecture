# ADR-003: Review Engine / AI Runtime Boundary

Status: Proposed
Decision Owner: Finance Main
Date: 2026-08-12
Related Jira: RPL-81
Related Repositories: `harness-foundation-docs`, `finance-harness-docs`, `finance-harness-fe`, `finance-harness-be`, AI Runtime
Depends On: Foundation communication and shared-AI boundaries; ADR-001 Journal Lifecycle; current Finance product requirements
Implementation Tracking: RPL-50, RPL-68
Supersedes: None
Canonical Merge SHA: Pending until merge

## 1. Context

Finance needs useful review guidance while preserving a hard separation between
stable analysis knowledge, current financial evidence, product responsibility,
and technical model execution. A single product-side LLM flow would couple
Finance to provider details and make it easy to present retrieved knowledge as
current fact.

This is a Proposed Finance-specific runtime composition under RPL-81. The
current remote Finance documents provide product and build constraints, but do
not provide an accepted remote Review Engine architecture with this async
composition. This ADR therefore records a proposal constrained by upstream
Foundation decisions, ADR-001, and current Finance requirements; it does not
claim that an AI Server or broker runtime is already verified or deployed.

## 2. Decision

### Accepted upstream constraints

The following are constraints carried from the current Foundation sources and
the merged Finance product sources:

- Foundation communication rules govern service-to-service transport and do
  not authorize a broker topology without a concrete use case and accepted
  contract.
- The Foundation Shared AI boundary keeps ingestion, retrieval, embedding, and
  model execution outside the Finance Backend product boundary.
- The merged Finance core-loop contract and build plan are source evidence for
  the existing product lifecycle and explicit-save flow; ADR-001 records the
  proposed lifecycle delta separately.

### Proposed Finance runtime composition

The proposed AI Server responsibility is the technical AI execution boundary:

- `finance-harness-docs` ingestion;
- RAG and embedding;
- vector retrieval;
- model routing;
- LLM/model execution;
- provider/runtime execution concerns, subject to the owning runtime contract.

Finance Backend owns the product boundary:

- ReviewExecution lifecycle and Finance domain state;
- product question, checklist, context, and workflow semantics;
- Finance policy and output validation;
- selection of the knowledge/evidence inputs;
- the user-visible product result and explicit Journal save meaning.

Finance Backend must not use a direct LLM SDK, product-side Spring AI execution,
embedding, vector retrieval, or direct Vector DB access as a substitute for the
AI Runtime boundary.

The proposed Finance Phase 1 composition is:

```text
Static / Versioned Checklist
+ finance-harness-docs RAG
+ Cheap LLM Analysis Guidance
```

If no Financial Evidence Provider is available, then:

```text
Fact = NOT_APPLICABLE
Current-market Inference = NOT_APPLICABLE
```

Stable knowledge retrieved from the corpus must not be presented as a current
financial fact. Guidance is not a recommendation or a replacement for user
judgment.

For a later asynchronous Review Runtime, the proposed logical boundary is:

```text
Frontend
  → REST / Polling
  → Finance Backend
  → NATS JetStream
  → AI Server
  → NATS Result
  → Finance Backend
  → Frontend
```

The FE remains on its product-facing REST/polling contract. This is a target
integration boundary, not proof of current broker or AI runtime support.

## 3. Boundaries

| Boundary | Finance Backend | AI Server | `finance-harness-docs` |
|---|---|---|---|
| Product workflow | Owns | Does not own | Does not own |
| Review lifecycle | Owns | Executes within contract | Does not own |
| Stable analysis knowledge | Selects/consumes | Ingests/retrieves | Authors and versions |
| Current financial evidence | Owns product meaning and evidence contract | May consume normalized input | Does not provide current fact merely by being retrieved |
| Embedding/vector/model execution | Must not implement directly | Owns runtime capability | Is an input corpus, not the runtime |
| Durable Journal meaning | Owns | Does not own | Does not own |

The AI Runtime does not decide Finance domain policy, user ownership, Journal
retention, or whether generated content becomes durable. Finance does not own
the AI provider's credentials, model routing implementation, or vector store.

## 4. Consequences

Positive consequences:

- RAG methodology and current evidence remain different source classes;
- provider/runtime changes are isolated behind an execution boundary;
- Phase 1 can deliver checklist and verification guidance without inventing
  current-market facts;
- Finance retains responsibility for safety validation and user-visible
  meaning;
- the later asynchronous shape can evolve without adding NATS to Journal
  Create S1/S2.

Costs and trade-offs:

- the system needs explicit provenance and validation when evidence is added;
- an async boundary adds correlation, retry, duplicate, and partial-result
  handling work;
- the first phase provides guidance rather than current factual analysis;
- AI Runtime and Finance Backend must coordinate compatibility without sharing a
  database or silently expanding each other's ownership.

## 5. Deferred Decisions

- exact NATS stream;
- exact NATS subject;
- NATS retention;
- retry policy;
- delivery, acknowledgement, and dead-letter policy;
- AI vendor;
- embedding vendor/model;
- vector database implementation;
- current financial Evidence Provider and Market Data vendor;
- Phase 2/3 activation, pricing, and deep-review escalation rules;
- production AI deployment, support, and retention details.

Until these decisions are accepted with their contracts, the target async path
must not be described as an active production runtime.

## 6. Verification

- Verify Phase 1 output contains no fabricated current Fact or unsupported
  Current-market Inference when no evidence provider is available.
- Verify RAG corpus content is labeled as stable analysis knowledge and is not
  treated as mutable current financial evidence.
- Verify Finance Backend has no direct LLM SDK, embedding, vector retrieval, or
  direct Vector DB dependency.
- Verify ReviewExecution and transient results remain separate from automatic
  Journal persistence.
- Verify every later asynchronous message has an accepted owner, idempotency,
  retry, delivery, retention, and compatibility contract before production
  activation.
- Verify provider/runtime failures can produce a truthful partial or blocked
  state without presenting an unvalidated result as complete.

## 7. References

- [Review Execution contract](../contracts/review-execution.md)
- [Runtime topology](../architecture/runtime-topology.md)
- [Finance core-loop and domain contract](https://github.com/ranikun-labs/finance-harness-docs/blob/5326977691b420f573a469c0bb7635de26fe2aba/04-backend-architecture/finance-core-loop-and-domain-contract.md)
- [Finance MVP build and verification plan](https://github.com/ranikun-labs/finance-harness-docs/blob/5326977691b420f573a469c0bb7635de26fe2aba/04-backend-architecture/finance-mvp-build-and-verification-plan.md)
- [Finance backend architecture](https://github.com/ranikun-labs/finance-harness-docs/blob/5326977691b420f573a469c0bb7635de26fe2aba/04-backend-architecture/finance-backend-architecture.md)
- [Foundation ADR-0015](https://github.com/ranikun-labs/harness-foundation-docs/blob/69fceb267094756913f400f8b4045590bd0e8ac5/docs/adr/ADR-0015-platform-communication-messaging-scaling.md)
- [Foundation ADR-0017](https://github.com/ranikun-labs/harness-foundation-docs/blob/69fceb267094756913f400f8b4045590bd0e8ac5/docs/adr/ADR-0017-shared-platform-gateway-identity-physicalization.md)
