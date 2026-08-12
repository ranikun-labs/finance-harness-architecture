# ADR-001: Journal Lifecycle

Status: Proposed
Decision Owner: Finance Main
Date: 2026-08-12
Related Jira: RPL-81
Related Repositories: `harness-foundation-docs`, `finance-harness-docs`, `finance-harness-fe`, `finance-harness-be`
Depends On: Foundation data ownership and service-boundary decisions; the merged Finance core-loop contract is the previous-reality input for this proposal
Supersedes: None
Canonical Merge SHA: Pending until merge

## 1. Context

Finance's core loop is:

```text
검토
→ 판단 기록
→ 복기
↺ 다음 검토 개선
```

The review experience produces operational information before the user decides
what to retain as a personal reasoning record. Treating every intermediate AI
output as a Journal would blur product responsibility, increase sensitive-data
retention, and make a temporary execution look like a user-authored decision.

This is a new Proposed Finance Architecture decision under RPL-81. It is not a
normalization of an already accepted lifecycle decision, and it does not create
a new storage schema.

### Previous Canonical Reality (merged Finance contract)

The merged Finance contract at [`finance-core-loop-and-domain-contract.md`]
(Finance `main`,
[`5326977691b420f573a469c0bb7635de26fe2aba`](https://github.com/ranikun-labs/finance-harness-docs/blob/5326977691b420f573a469c0bb7635de26fe2aba/04-backend-architecture/finance-core-loop-and-domain-contract.md))
establishes the prior lifecycle semantics:

- Section 3 models `Question → ChecklistResult → AnalysisRequest →
  AnalysisResult`, followed by an explicit Journal save only after a completed
  result.
- Section 8 defines `AnalysisResult` as a separate resource whose completed
  result is immutable and which Journal references.
- Section 9 defines Journal as holding user-authored information and a
  completed `AnalysisResult` reference, with saving requested explicitly.
- Section 11 repeats the relationship as `AnalysisResult (immutable when
  completed)` and `Journal → user-authored information + completed
  AnalysisResult reference`.

Together, those sections establish the previous durable AnalysisResult and
Journal-reference model. They do not establish the ReviewExecution,
DecisionContextSnapshot, and Retrospective lifecycle proposed below.

## 2. Decision

Finance proposes the following target lifecycle under RPL-81:

| Concept | Meaning | Lifecycle |
|---|---|---|
| Journal | User-owned durable reasoning record containing the user's saved action, reasoning, and related Finance data | Created only by an explicit user save operation; retained and accessed through Finance-owned product APIs |
| ReviewExecution | Temporary operational state for a review request, progress, and transient result serving | May expire or be deleted under a later retention decision; it is not a user-owned durable record |
| DecisionContextSnapshot | Immutable, user-selected copy of the context material to the decision at save time | Created only when the user explicitly enables/saves context; no later mutation of the snapshot |
| Retrospective | Separate timestamped durable record for reviewing a prior Journal | Created and changed separately from the original Journal; it does not rewrite the original decision |

The invariant is:

```text
Transient AI Review
!=
Durable User Record
```

The complete AI Review output, prompt, transcript, or temporary retrieval
reference is not automatically saved into a Journal. When a user explicitly
saves a Decision Context, Finance copies only the required context values and
provenance into the immutable snapshot boundary.

## 3. Boundaries

### Decision delta and supersession semantics

This proposal differs from the previous merged Finance contract: the review
operation is transient, while the Journal is the user's durable reasoning
record; an immutable saved context is a separate snapshot rather than a
durable `AnalysisResult` reference; and a Retrospective is a separate durable
record.

Upon acceptance and merge, this ADR supersedes only the prior
`AnalysisResult` durability/reference lifecycle semantics in Sections 8–11 of
the merged core-loop contract. It does not supersede that entire document or
unrelated domain, API, safety, or product decisions in the existing Finance
documents. Migration or deprecation treatment for the historical document is a
deferred follow-up.

- Finance Backend owns the Journal, ReviewExecution, DecisionContextSnapshot,
  and Retrospective product lifecycle semantics.
- The user is the authority for creating the durable reasoning record and for
  opting into a Decision Context snapshot.
- AI Runtime output is an operational input to the product flow. It does not
  become a durable user record by completing execution.
- `finance-harness-docs` remains a stable knowledge source. Its retrieved
  content is not silently copied wholesale into Journal storage.
- Shared Identity supplies the opaque owner identity through the Finance
  identity adapter seam; it does not own Journal or Retrospective rows.
- A local same-service relationship may be enforced by Finance persistence,
  while cross-service foreign keys, joins, and deletion cascades remain
  prohibited by Foundation policy.

## 4. Consequences

Positive consequences:

- user-authored reasoning remains distinguishable from generated guidance;
- retention and deletion can differ between operational and durable data;
- retrospective records preserve history without mutating the original
  decision;
- AI, Redis, or broker implementation changes do not redefine the Journal
  contract;
- an explicit-save boundary gives FE, BE, and AI Runtime a testable handoff.

Costs and trade-offs:

- the product must implement an explicit save decision instead of treating a
  completed review as a Journal shortcut;
- snapshot selection and provenance need validation at save time;
- temporary execution cleanup and durable-data policy require separate
  operational decisions;
- users may see a useful review without any durable record if they do not save
  it, which is intentional.

## 5. Deferred Decisions

- exact ReviewExecution, progress, and transient-result retention period;
- deletion and user-data-policy details for each durable record;
- the complete Journal and Retrospective API/DTO contract beyond the Journal
  Create contract;
- idempotency-key representation and persistence mechanics for explicit save;
- physical database schema, indexes, migrations, and deployment topology;
- whether any additional evidence or knowledge provenance is required in a
  future snapshot revision.

None of these deferred items changes the lifecycle invariant or authorizes
automatic Review-to-Journal persistence.

## 6. Verification

- Verify that Review completion does not create a Journal without an explicit
  user save command.
- Verify that a saved Decision Context is immutable after creation.
- Verify that a Retrospective is timestamped and stored as a separate durable
  record without rewriting the original Journal.
- Verify ownership using the current opaque identity obtained through
  `CurrentUserPort`, not a client-supplied owner value.
- Verify that temporary AI output, prompts, transcripts, and retrieval
  references are not automatically persisted as durable user records.
- Verify retention, deletion, and cross-service data access against the
  accepted Finance User Data Policy and Foundation data-ownership policy before
  production activation.

## 7. References

- [Journal Create contract](../contracts/journal-create.md)
- [Lifecycle and ownership map](../data/lifecycle-and-ownership.md)
- [Finance backend architecture reference](https://github.com/ranikun-labs/finance-harness-docs/blob/5326977691b420f573a469c0bb7635de26fe2aba/04-backend-architecture/finance-backend-architecture.md)
- [Finance core-loop and domain contract reference](https://github.com/ranikun-labs/finance-harness-docs/blob/5326977691b420f573a469c0bb7635de26fe2aba/04-backend-architecture/finance-core-loop-and-domain-contract.md)
- [Finance MVP build and verification plan](https://github.com/ranikun-labs/finance-harness-docs/blob/5326977691b420f573a469c0bb7635de26fe2aba/04-backend-architecture/finance-mvp-build-and-verification-plan.md)
- [Foundation ADR-0013](https://github.com/ranikun-labs/harness-foundation-docs/blob/69fceb267094756913f400f8b4045590bd0e8ac5/docs/adr/ADR-0013-target-deployment-and-data-boundaries.md)
