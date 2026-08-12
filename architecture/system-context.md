# Finance Harness System Context

Status: Accepted canonical view; logical boundary only.

This document distinguishes observed repository facts, the target logical
architecture, and decisions that remain deferred. A component shown as a
target or boundary is not thereby declared production-deployed.

## Context boundary

```text
                         ┌──────────────────────────────┐
                         │ Finance Client / FE           │
                         │ finance-harness-fe            │
                         └──────────────┬───────────────┘
                                        │ external product request
                                        ▼
                         ┌──────────────────────────────┐
                         │ Shared Gateway                │
                         │ external ingress boundary    │
                         └──────────────┬───────────────┘
                                        │ trusted identity context
                                        ▼
                         ┌──────────────────────────────┐
                         │ Finance Backend               │
                         │ product API and lifecycle     │
                         └───────┬───────────────┬──────┘
                                 │               │
                    owned state │               │ review runtime contract
                                 ▼               ▼
                 ┌────────────────────┐  ┌──────────────────────┐
                 │ PostgreSQL /       │  │ NATS JetStream       │
                 │ Finance storage    │  │ async transport      │
                 └────────────────────┘  └──────────┬───────────┘
                                                     │
                                                     ▼
                                          ┌──────────────────────┐
                                          │ AI Server             │
                                          │ RAG / retrieval /     │
                                          │ model execution       │
                                          └──────┬─────────┬─────┘
                                                 │         │
                          stable knowledge       │         │ future evidence
                                                 ▼         ▼
                                  ┌──────────────────┐  ┌──────────────────┐
                                  │ finance-harness- │  │ Evidence / Market │
                                  │ docs             │  │ Data Provider     │
                                  │ stable corpus    │  │ future boundary   │
                                  └──────────────────┘  └──────────────────┘

                         ┌──────────────────────────────┐
                         │ Shared Identity              │
                         │ stable identity consumer     │
                         └──────────────┬───────────────┘
                                        │ CurrentUserPort
                                        └──────────────► Finance Backend
```

The diagram is a boundary model. It intentionally does not name a host,
published Gateway route, authentication header, broker subject, or provider.

## Current state

| Boundary or component | Current evidence | State represented here |
|---|---|---|
| Finance FE | The reviewed `master` contains the P0 Finance UI, Journal UI, and Journal Create adapter seam. Its README keeps backend/API integration and Production Journal Create activation in follow-up scope. | Current client implementation; production activation is not implied. |
| Finance Backend | The reviewed `main` contains the merged RPL-50 S2 Journal Persistence + Create API, including Finance-owned persistence and `POST /finance/journals`. | Current backend foundation and Journal Create implementation; raw reads, idempotency, and production activation remain separate gates. |
| Shared Gateway / Shared Identity | The Foundation has accepted constrained physicalization, but extraction and cutover are not complete. | Logical shared boundary and target; not a Finance production dependency claim. |
| AI Server | AI/RAG responsibility is separated by the Finance and Foundation architecture sources. | Target runtime boundary; no Finance production runtime is asserted. |
| `finance-harness-docs` | The repository contains stable analysis, safety, and product knowledge. | Knowledge source and future ingestion input, not system-architecture authority. |
| PostgreSQL / Finance-owned storage | Foundation and Finance sources define logical database ownership and prohibit cross-service database access. | Ownership boundary; physical provisioning and schema are out of scope. |
| NATS JetStream | Foundation marks the technology as deferred until a concrete asynchronous use case. | Review-runtime target option; not current Journal transport. |
| Evidence / Market Data Provider | No provider is accepted in the verified sources. | Future boundary only. |

## Current reviewed runtime reality

The current remote repository evidence separates implementation from
activation:

| Capability | Current state |
|---|---|
| Backend foundation | Implemented where evidenced by the current Finance BE `main`; the repository is not an empty implementation target. |
| Journal Persistence / Create | Implemented in the merged BE S2 change, including `POST /finance/journals`; this does not turn production Create on. |
| Journal Detail / List | Not yet implemented in the reviewed BE S2 state; required for the raw read gate. |
| Reliability / Idempotency | Not yet evidenced as a retransmission contract; concurrent-create coverage is not idempotency. |
| FE production Create | Not activated; the FE production adapter seam exists, while production persistence/Create activation remains follow-up work. |
| Shared Identity production adapter | Not complete; `CurrentUserPort` and local/test seams do not prove production Identity integration. |
| Review NATS runtime | Not implemented or activated; only the logical async target is recorded. |
| AI runtime integration | Not implemented or activated; no provider, model, embedding, or vector implementation is selected here. |

## Target logical boundary

The target Finance system separates responsibilities as follows:

- Finance FE owns presentation, interaction, and client transport adaptation.
- Finance Backend owns the Finance Product API, domain lifecycle, user-owned
  durable state, review orchestration, and the Finance-owned persistence
  boundary.
- Shared Identity owns authentication and stable identity. Finance consumes it
  through an adapter seam and does not own its account, credential, token,
  session, or storage model.
- Shared Gateway owns the external ingress/security boundary. Its final
  Finance route and deployed topology are not fixed here.
- AI Server owns Finance knowledge ingestion, RAG, embedding, retrieval, model
  routing, and LLM execution. Product policy, workflow, domain context, and
  durable product meaning remain Finance-owned.
- `finance-harness-docs` owns stable analysis knowledge and safety/product
  knowledge used as corpus input.
- NATS JetStream is the intended asynchronous boundary for a later Review
  Runtime phase, subject to a separately accepted contract.
- A future Evidence / Market Data Provider may supply current facts only after
  a provider contract and activation gate are accepted.

## Explicit boundary rules

1. Finance does not access Shared Identity or another service's database
   directly.
2. Finance does not send client-supplied user identifiers as an ownership
   authority.
3. Stable RAG knowledge is not represented as current financial evidence.
4. A temporary AI Review is not automatically a durable user record.
5. Target topology, implementation readiness, runtime support, and production
   activation are separate states.

## Deferred context

The following remain intentionally unspecified in this context view:

- Shared Gateway and Shared Identity hosts and final published Finance route;
- OAuth entrypoint, issuer, audience, JWKS, browser transport, and
  refresh/session details;
- exact NATS stream, subject, retention, retry, and delivery policy;
- AI, embedding, vector, market-data, and evidence providers;
- production retention and physical storage configuration.

The complete register is in [`decision-log.md`](../decision-log.md).

## References

- [`harness-foundation-docs` ADR-0013](https://github.com/ranikun-labs/harness-foundation-docs/blob/69fceb267094756913f400f8b4045590bd0e8ac5/docs/adr/ADR-0013-target-deployment-and-data-boundaries.md)
- [`harness-foundation-docs` ADR-0015](https://github.com/ranikun-labs/harness-foundation-docs/blob/69fceb267094756913f400f8b4045590bd0e8ac5/docs/adr/ADR-0015-platform-communication-messaging-scaling.md)
- [`harness-foundation-docs` ADR-0017](https://github.com/ranikun-labs/harness-foundation-docs/blob/69fceb267094756913f400f8b4045590bd0e8ac5/docs/adr/ADR-0017-shared-platform-gateway-identity-physicalization.md)
- [`finance-harness-docs` backend architecture](https://github.com/ranikun-labs/finance-harness-docs/blob/5326977691b420f573a469c0bb7635de26fe2aba/04-backend-architecture/finance-backend-architecture.md)
- [`finance-harness-fe` reviewed master](https://github.com/ranikun-labs/finance-harness-fe/blob/d0ed81489d75e2e714d96d02ae6cb8dd2c86fd78/README.md)
- [`finance-harness-be` reviewed main](https://github.com/ranikun-labs/finance-harness-be/commit/8b38a65650bca8e0d96dbb6a9f4b1a13f84400d5)
