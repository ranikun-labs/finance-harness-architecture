# ADR-004: Production Activation Gates

Status: Accepted
Decision Owner: Finance Main
Date: 2026-08-12
Related Jira: RPL-81
Related Repositories: `harness-foundation-docs`, `finance-harness-architecture`, `finance-harness-fe`, `finance-harness-be`, `platform-services`, AI Runtime
Depends On: Foundation ADR-0017 migration gates; Journal, Identity, and Review contracts
Upstream Gate: G0/RPL-52; G1/RPL-71 + RPL-72; G2/RPL-53; G3/RPL-54; G4/RPL-55; RPL-27; RPL-50
Supersedes: None
Canonical Merge SHA: 07b598375a6ce4c84c3137d7c4a9b082c1c7e2c8

## 1. Context

The existence of a UI, port, endpoint, adapter seam, or accepted architecture
decision does not prove that a Finance production user flow is safe or
available. Finance therefore needs explicit activation gates that preserve the
difference between implementation readiness, upstream contract readiness,
runtime support, and production release.

The current Foundation ADR-0017 sequence is the upstream ordering for Finance
integration. The gates below record that ordering without claiming completion
of any gate.

## 2. Decision

Production activation is opt-in per runtime capability and requires all
upstream contracts and verification evidence named by that capability.

| Capability | Implementation evidence may exist | Production activation gate |
|---|---|---|
| Journal Create | Remote Finance BE main contains the merged RPL-50 S2 Journal Persistence + Create API; FE has a Journal Create port/adapter | Raw read coverage, identity ownership path, reliability/idempotency verification, cross-repository evidence, and an explicit production activation decision are complete. Current FE production activation remains off. |
| Shared Identity consumer | `CurrentUserPort` and adapter seam are defined; local/test adapter may exist | Accepted upstream Identity/Gateway consumer contract, production adapter validation, ownership/security tests, and platform cutover evidence are complete. |
| Review UI | FE review surfaces and fixtures may exist | Real Review Execution contract, safety validation, runtime availability, truthful partial/failure handling, and production AI activation evidence are complete. |
| Review async transport | A logical NATS boundary is documented | Accepted stream/subject/delivery/retry/retention contract, idempotent producer/consumer behavior, observability, replay/recovery, and runtime support are complete. |

The Foundation physicalization sequence remains an upstream gate:

```text
G0 / RPL-52
  → G1 / RPL-71 + RPL-72
  → G2 / RPL-53
  → G3 / RPL-54
  → G4 / RPL-55
  → RPL-27
  → RPL-50
  → Finance Shared Gateway / Identity E2E
```

In ADR-0017, G0 is completed history; G1 is the RPL-71/RPL-72 inert-runtime
foundation scope; G2, G3, and G4 remain planned migration gates; RPL-27 is
sequenced after those gates; and RPL-50 is planned after G4. The sequence is
an activation dependency, not a statement that those runtime steps are
complete in this repository.

The Finance Journal production activation chain is:

```text
Journal Create
  → Raw List / Detail
  → FE Raw Adapter
  → Reliability / Idempotency
  → Shared Identity G5
  → Cross-repository E2E
  → Production Create ON
```

The current reviewed state of each stage is:

| Stage | State | Evidence or remaining gate |
|---|---|---|
| Journal Create | Current implementation; production OFF | BE main at [RPL-50 merge SHA](https://github.com/ranikun-labs/finance-harness-be/commit/8b38a65650bca8e0d96dbb6a9f4b1a13f84400d5) contains Journal persistence and `POST /finance/journals`; this is not production activation. |
| Raw List / Detail | Required next gate | Current BE S2 evidence is create-only; the Finance build plan requires Journal list/detail read coverage. |
| FE Raw Adapter | Required after raw read contract | Current FE master has Journal UI and a Create adapter seam; a production raw List/Detail adapter is not evidenced in the reviewed tree. |
| Reliability / Idempotency | Required next gate | The BE concurrent-create test demonstrates distinct concurrent creates, not retransmission idempotency; duplicate-request behavior remains a required verification. |
| Shared Identity G5 | Upstream/deferred Finance activation gate | `CurrentUserPort` and a local adapter exist, but production Identity integration and the upstream shared-platform cutover are not complete. G5 is named here as the Finance activation gate; it is not added to the Foundation ADR-0017 sequence. |
| Cross-repository E2E | Required gate | Must follow the upstream Foundation sequence and verify FE, BE, Gateway/Identity, ownership, and runtime boundaries together; completion is not evidenced here. |
| Production Create ON | Deferred final activation | FE production Create remains off; the FE source lists production persistence and Production Journal Create activation as follow-up work. |

The following distinctions are invariants:

```text
Create/Persistence merged != Production activation
FE editor merged        != Production activation
Idempotency alone       != Production activation
Shared Identity seam    != Production Identity integration complete
```

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
  service contract, raw read path, ownership path, reliability/idempotency,
  cross-repository evidence, and activation evidence are complete.
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
- [Foundation ADR-0017 migration sequence](https://github.com/ranikun-labs/harness-foundation-docs/blob/69fceb267094756913f400f8b4045590bd0e8ac5/docs/adr/ADR-0017-shared-platform-gateway-identity-physicalization.md)
- [Finance FE production activation boundary](https://github.com/ranikun-labs/finance-harness-fe/blob/d0ed81489d75e2e714d96d02ae6cb8dd2c86fd78/README.md)
- [Finance BE RPL-50 S2 merge](https://github.com/ranikun-labs/finance-harness-be/commit/8b38a65650bca8e0d96dbb6a9f4b1a13f84400d5)
- [Finance MVP build and verification plan](https://github.com/ranikun-labs/finance-harness-docs/blob/5326977691b420f573a469c0bb7635de26fe2aba/04-backend-architecture/finance-mvp-build-and-verification-plan.md)
