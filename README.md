# Finance Harness Architecture

Finance Harness Architecture is the canonical repository for Finance-specific
system architecture, architecture decisions, cross-repository contracts,
runtime boundaries, and data/lifecycle ownership.

This repository records decisions and boundaries. It does not claim that a
component is implemented, deployed, or production-supported unless the owning
repository provides that evidence.

## Repository purpose

This repository owns:

- Finance system architecture and current/target boundary descriptions;
- Finance-specific ADRs and the deferred-decision register;
- human-readable integration contracts between Finance and shared/runtime
  consumers;
- Finance data and lifecycle ownership rules.

## Canonical hierarchy

```text
harness-foundation-docs
        ↓
finance-harness-architecture
        ↓
finance-harness-fe / finance-harness-be / AI Runtime
```

`harness-foundation-docs` is the Architecture Constitution. This repository
applies that constitution to Finance and must not redefine its shared rules.
The Finance implementation repositories and AI Runtime consume this
repository's merged canonical state; they do not silently redefine it.

Separately:

```text
finance-harness-docs
= Finance Analysis Knowledge / Safety / Product Knowledge / RAG Corpus
```

`finance-harness-docs` is the knowledge and policy corpus. It is not the
canonical home for new Finance system-architecture decisions created here.

## Decision authority and canonicalization

Architecture Decision Owner: **Finance Main**

The canonicalization path is:

```text
Finance Main Decision
→ Architecture PR
→ Independent Review
→ Merge
→ Merge SHA Canonical
→ FE / BE / AI Handoff
```

Until this bootstrap PR is merged, the ADRs in this repository are `Proposed`
and their `Canonical Merge SHA` is `Pending until merge`.

## Source precedence

When sources disagree, use this order:

1. `ranikun-labs/harness-foundation-docs`;
2. the verified Finance cross-repository canonical sources;
3. this repository;
4. executable implementation repositories;
5. Jira;
6. Confluence projections.

Architecture documents in `finance-harness-docs/04-backend-architecture/`
remain historical and implementation-oriented references during this
bootstrap. They are referenced, not migrated or duplicated here.

## Document map

| Area | Canonical documents |
|---|---|
| System boundary | [`architecture/system-context.md`](./architecture/system-context.md), [`architecture/runtime-topology.md`](./architecture/runtime-topology.md) |
| Service ownership | [`architecture/ownership-map.md`](./architecture/ownership-map.md) |
| Architecture decisions | [`adr/`](./adr/) |
| Cross-repository contracts | [`contracts/`](./contracts/) |
| Data lifecycle | [`data/lifecycle-and-ownership.md`](./data/lifecycle-and-ownership.md) |
| Decision inventory | [`decision-log.md`](./decision-log.md) |

## Non-goals

This repository does not contain:

- product implementation;
- a redefinition of Shared Platform, Shared Gateway, or Shared Identity;
- the Finance RAG corpus or stable analysis knowledge;
- credential or secret storage;
- guessed runtime hosts, authentication values, provider choices, or other
  deferred values;
- OpenAPI, AsyncAPI, schema-registry, code-generation, CI, or documentation
  site tooling;
- actual runtime activation or production rollout.

## Current bootstrap state

The repository was empty and unborn before this bootstrap. The initial base
commit only establishes a `main` ref so that the architecture changes can be
reviewed in a pull request. The architecture commits are scoped to the files
listed in the bootstrap plan; unrelated repository-local harness state is not
part of the PR.

