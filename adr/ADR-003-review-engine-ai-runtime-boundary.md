# ADR-003: Review Engine / AI Runtime Boundary

Status: Proposed
Decision Owner: Finance Main
Date: 2026-08-12
Related Jira: `RPL-81`, `RPL-50`, `RPL-68`
Related Repositories: `harness-foundation-docs`, `finance-harness-docs`, `finance-harness-fe`, `finance-harness-be`, AI Runtime
Depends On: Foundation communication and shared-AI boundaries; Finance Review Engine contract
Supersedes: None
Canonical Merge SHA: Pending until merge

## 1. Context

Finance needs useful review guidance while preserving a hard separation between
stable analysis knowledge, current financial evidence, product responsibility,
and technical model execution. A single product-side LLM flow would couple
Finance to provider details and make it easy to present retrieved knowledge as
current fact.

The verified Finance Review Engine decision already adopts a progressive hybrid
shape. This ADR records the ownership and runtime boundary in the Finance
Architecture repository without selecting a provider or claiming an AI Server
deployment.

## 2. Decision

AI Server owns the technical AI execution boundary:

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

Phase 1 is:

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
Finance Backend
  → NATS JetStream
  → AI Server
  → NATS result
  → Finance Backend
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
- [Finance Review Engine architecture](https://github.com/ranikun-labs/finance-harness-docs/blob/main/04-backend-architecture/finance-review-engine-architecture.md)
- [Finance backend architecture](https://github.com/ranikun-labs/finance-harness-docs/blob/main/04-backend-architecture/finance-backend-architecture.md)
- [Foundation ADR-0015](https://github.com/ranikun-labs/harness-foundation-docs/blob/main/docs/adr/ADR-0015-platform-communication-messaging-scaling.md)
- [Foundation ADR-0017](https://github.com/ranikun-labs/harness-foundation-docs/blob/main/docs/adr/ADR-0017-shared-platform-gateway-identity-physicalization.md)
