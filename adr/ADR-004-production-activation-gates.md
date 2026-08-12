# ADR-004: Production Activation Gates

Status: Proposed
Decision Owner: Finance Main
Date: 2026-08-12
Related Jira: `RPL-XXX`, `RPL-50`, `RPL-52`, `RPL-53`, `RPL-54`, `RPL-55`, `RPL-68`
Related Repositories: `harness-foundation-docs`, `finance-harness-architecture`, `finance-harness-fe`, `finance-harness-be`, `platform-services`, AI Runtime
Depends On: Foundation ADR-0017 migration gates; Journal, Identity, and Review contracts
Supersedes: None
Canonical Merge SHA: Pending until merge

## 1. Context

The existence of a UI, port, endpoint, adapter seam, or accepted architecture
decision does not prove that a Finance production user flow is safe or
available. Finance therefore needs explicit activation gates that preserve the
difference between implementation readiness, upstream contract readiness,
runtime support, and production release.

The current verified sequence is Foundation physicalization and validation
before Finance Backend production integration. The gates below record that
ordering without claiming completion of any gate.

## 2. Decision

Production activation is opt-in per runtime capability and requires all
upstream contracts and verification evidence named by that capability.

| Capability | Implementation evidence may exist | Production activation gate |
|---|---|---|
| Journal Create | FE has a client port/adapter and the service-local API contract is defined | Backend contract, identity ownership path, persistence, error/idempotency verification, and an explicit production activation decision are complete. Current FE production activation remains off. |
| Shared Identity consumer | `CurrentUserPort` and adapter seam are defined; local/test adapter may exist | Accepted upstream Identity/Gateway consumer contract, production adapter validation, ownership/security tests, and platform cutover evidence are complete. |
| Review UI | FE review surfaces and fixtures may exist | Real Review Execution contract, safety validation, runtime availability, truthful partial/failure handling, and production AI activation evidence are complete. |
| Review async transport | A logical NATS boundary is documented | Accepted stream/subject/delivery/retry/retention contract, idempotent producer/consumer behavior, observability, replay/recovery, and runtime support are complete. |

The Foundation physicalization sequence remains an upstream gate:

```text
RPL-52 / G1 Foundation approval
  → RPL-53 / G2 Gateway extraction
  → RPL-54 / G3 Identity extraction
  → RPL-55 / G4 Carelog cutover and regression
  → RPL-50 / Finance Backend and Shared Platform E2E
```

The sequence is an activation dependency, not a statement that those runtime
steps are complete in this repository.

## 3. Boundaries

- A Finance Architecture merge records a canonical decision; it does not
  activate a runtime.
- A service-local `POST /finance/journals` contract does not establish the
  final Shared Gateway route.
- A `CurrentUserPort` seam does not establish that a production Identity
  adapter, token contract, or Gateway cutover is complete.
- A Review UI or fixture does not establish that the AI Runtime, RAG, evidence,
  safety, or persistence path is production-ready.
- An accepted target topology does not establish deployment, support, or
  release.
- Only the owning service and the applicable upstream decision owners can close
  the relevant activation evidence; a downstream repository must not infer
  completion from a branch or UI state.

## 4. Consequences

Positive consequences:

- partial implementation cannot silently become a production dependency;
- the FE, BE, Platform, and AI Runtime handoff has explicit evidence needs;
- rollback and failure behavior are reviewed before cutover;
- a deferred upstream contract blocks activation without blocking honest local
  implementation work;
- reviewers can distinguish architecture correctness from release readiness.

Costs and trade-offs:

- production launch requires coordination across repositories and decision
  owners;
- some UI work can remain inactive while backend or shared-platform gates are
  open;
- gate evidence must be maintained as contracts evolve;
- a merge of this repository alone cannot close a runtime gate.

## 5. Deferred Decisions

- exact production activation checklist and approver for each capability;
- final Gateway and Identity host/route values;
- token transport and validation details;
- exact Review Runtime broker configuration and retention;
- AI/evidence provider selection and production SLOs;
- production data retention, rollback observation period, and release policy;
- the final Jira decomposition and evidence links for future implementation work.

These deferred items are gates to be decided by the responsible owners; they
are not permission to fill in guessed values in this repository.

## 6. Verification

- Verify the Journal Create production switch is explicitly off until the
  service contract, ownership path, persistence, and activation evidence are
  complete.
- Verify a local identity adapter cannot be selected in a production
  configuration.
- Verify production Identity activation is blocked until the upstream Gateway/
  Identity extraction and cutover sequence is complete.
- Verify Review UI availability is not used as evidence of a production AI
  runtime.
- Verify async Review activation has an accepted message contract and tested
  idempotent, retry, replay, partial-failure, and recovery behavior.
- Verify each closed gate links to owning-repository evidence and an accepted
  decision; a branch HEAD or architecture merge SHA alone is insufficient.
- Verify rollback paths preserve data ownership and do not introduce dual
  writers across service boundaries.

## 7. References

- [Runtime topology](../architecture/runtime-topology.md)
- [Journal Create contract](../contracts/journal-create.md)
- [Identity Consumer contract](../contracts/identity-consumer.md)
- [Review Execution contract](../contracts/review-execution.md)
- [Foundation ADR-0017 migration sequence](https://github.com/ranikun-labs/harness-foundation-docs/blob/main/docs/adr/ADR-0017-shared-platform-gateway-identity-physicalization.md)
- [Finance FE production activation boundary](https://github.com/ranikun-labs/finance-harness-fe/blob/master/README.md)
- [Finance MVP implementation scope](https://github.com/ranikun-labs/finance-harness-docs/blob/main/04-backend-architecture/finance-mvp-implementation-scope.md)
