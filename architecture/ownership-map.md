# Finance Ownership Map

Status: Proposed canonical ownership view.

The map separates logical responsibility from implementation status. Ownership
does not imply that the owner has already deployed the runtime.

## Repository and runtime ownership

| Owner / boundary | Owns | Does not own | Current/target state |
|---|---|---|---|
| Finance FE (`finance-harness-fe`) | Presentation, interaction, client transport, local form/time-zone capture, and rendering of product states | Finance domain truth, identity authority, direct LLM/RAG/vector access, or production activation policy | Existing UI and adapter seams; backend/runtime integration remains gated |
| Finance Backend (`finance-harness-be`) | Finance Product API, domain lifecycle, ownership checks, Journal and Review contracts, durable product state, orchestration, and Finance safety/result boundaries | Shared Identity account data, shared auth storage, AI provider execution, shared vector storage, or another service's database | Product service target; remote implementation repository was empty in preflight |
| Shared Identity | Authentication, stable identity, account/credential/token/session responsibility | Finance Profile, Journal ownership semantics, Finance authorization, or Finance tables | Shared logical boundary; physicalization accepted with constraints, extraction/cutover not complete |
| Shared Gateway | External ingress, routing boundary, and trusted authentication-context edge | Finance domain logic, durable Finance state, and final Finance API contract | Target shared boundary; final host/route is deferred |
| AI Server | `finance-harness-docs` ingestion, RAG, embedding, retrieval, model routing, LLM/model execution, and runtime execution concerns | Finance product workflow, Journal meaning, user ownership, Finance policy source, or durable Journal storage | Target runtime boundary; no Finance production deployment asserted |
| `finance-harness-docs` | Stable analysis knowledge, safety knowledge, product knowledge, and RAG corpus authoring | New cross-repository system architecture canonical decisions | Existing knowledge corpus; consumed through an explicit ingestion/projection boundary |
| NATS JetStream | Async Review Runtime transport after adoption | Journal S1/S2 transport, durable product ownership, AI provider choice, or business state source of truth | Deferred technology/target boundary |
| Finance-owned PostgreSQL/storage | Durable Finance product records and Finance-owned migrations when implemented | Shared Identity/AI tables, cross-service joins, or cross-service foreign keys | Logical ownership target; physical schema/provisioning is not defined here |

## Responsibility flow

```text
FE interaction
  → Finance Backend product command/query
  → CurrentUserPort identity semantic
  → Finance-owned ownership and lifecycle rules
  → Finance-owned durable or temporary state
  → AI Server only through the Review Runtime contract
```

The direction is intentional: the AI Runtime executes technical AI capability,
while Finance retains product meaning and the final user-visible result
responsibility.

## Data access rules

1. A service reads or mutates another service's data through an explicit API,
   token claim, event, or projection—not direct SQL.
2. Finance uses only an opaque stable `identityUserId` semantic for external
   identity. It does not copy account, organization, role, credential, token,
   or session models.
3. Finance-owned durable records are scoped by the current identity obtained
   from `CurrentUserPort`; a client-supplied owner identifier is not trusted.
4. Stable knowledge and current financial evidence are separate source classes.
5. Temporary Review state is not promoted to durable user state without an
   explicit user save operation.

## Source of truth summary

| Information | Source of truth |
|---|---|
| Finance domain state | Finance Backend and Finance-owned storage |
| Current identity | Shared Identity, consumed through the Finance identity adapter seam |
| Stable analysis knowledge | `finance-harness-docs` canonical corpus |
| Current financial evidence | A future accepted Evidence Provider contract; none is selected now |
| AI execution details | AI Server runtime boundary |
| Async delivery state | The adopted Review Runtime transport contract, when accepted |

## References

- [`harness-foundation-docs` repository/service boundaries](https://github.com/ranikun-labs/harness-foundation-docs/blob/main/docs/architecture/repository-service-boundaries.md)
- [`harness-foundation-docs` ADR-0013](https://github.com/ranikun-labs/harness-foundation-docs/blob/main/docs/adr/ADR-0013-target-deployment-and-data-boundaries.md)
- [`harness-foundation-docs` identity token contract](https://github.com/ranikun-labs/harness-foundation-docs/blob/main/docs/contracts/backend-service-foundation/identity-token-contract.md)
- [`finance-harness-docs` backend architecture](https://github.com/ranikun-labs/finance-harness-docs/blob/main/04-backend-architecture/finance-backend-architecture.md)

