# ADR-002: Shared Identity Consumer Boundary

Status: Proposed
Decision Owner: Finance Main
Date: 2026-08-12
Related Jira: `RPL-XXX`, `RPL-50`
Related Repositories: `harness-foundation-docs`, `finance-harness-docs`, `finance-harness-fe`, `finance-harness-be`, `platform-services`
Depends On: Foundation identity, gateway, and data-ownership decisions
Supersedes: None
Canonical Merge SHA: Pending until merge

## 1. Context

Finance is a consumer of Shared Identity, not an owner or copy of the shared
authentication system. The Finance Application needs one semantic value to
scope product data and authorization: an opaque, stable identity user
identifier.

The application must remain independent of whether that semantic arrives from a
local development adapter, a future shared adapter, or a trusted deployment
boundary. Transport details are therefore separated from the application seam.

The verified Foundation identity token contract is still Draft. Finance records
its transport-dependent details as Deferred rather than treating that Draft as
an accepted production contract.

## 2. Decision

Finance exposes one application seam:

```text
CurrentUserPort
  → identityUserId
```

`identityUserId` is an opaque, stable identifier owned by Shared Identity.
Finance may store it as an external ownership reference, but it must not
interpret its construction, replace it with email or login name, or treat it as
an account schema.

The consumer boundary is:

```text
Shared Identity
  → accepted identity consumer contract / adapter
  → CurrentUserPort
  → Finance Application and ownership checks
```

Development and test environments may use a dedicated local adapter that
returns a stable contract-shaped value. A production adapter is not complete
until the upstream identity and gateway contracts, validation, and activation
gates are accepted.

## 3. Boundaries

Finance may consume:

- the current principal's opaque stable `identityUserId` semantic;
- the identity contract required to validate the current principal after an
  upstream transport decision is accepted;
- an explicit, bounded management response when a future use case requires
  current identity state.

Finance must not:

- access `identity_db`, Identity PostgreSQL tables, or Identity Redis directly;
- copy another product's Auth or JWT model;
- accept `userId`, `identityUserId`, or `actorId` from a client as the owner
  authority;
- copy organization, role, public-ID, credential, token, or session models into
  Finance;
- use a production fixed-user fallback;
- make ordinary product requests depend on a synchronous Identity lookup when
  locally validated trusted context is sufficient.

Finance authorization remains Finance-owned. A valid identity does not by
itself grant access to another user's Journal or product data.

## 4. Consequences

Positive consequences:

- Finance domain code is insulated from authentication transport changes;
- ownership checks have one explicit application boundary;
- shared identity data and Finance product data retain separate ownership;
- local development can preserve the production semantic without pretending to
  be production authentication;
- replacing an adapter does not require rewriting Finance domain entities.

Costs and trade-offs:

- the adapter must fail closed when the current identity is unavailable or
  untrusted;
- production activation depends on upstream identity/gateway contract work;
- local adapters need environment controls so they cannot silently cross into
  a production path;
- identity lifecycle changes require an explicit Finance retention and local
  reference policy rather than a cross-service delete cascade.

## 5. Deferred Decisions

The following are intentionally not fixed by this ADR:

- Shared Identity host;
- Gateway host and final published Finance route;
- OAuth entrypoint;
- token issuer;
- token audience;
- JWKS endpoint and key-rotation details;
- browser authentication transport;
- refresh/session mechanism;
- Identity Account schema;
- ProductClient persistence;
- Identity Redis semantics;
- exact gateway-to-application trusted-context fields and service-auth
  mechanism;
- production adapter implementation and cutover evidence.

The application contract is valid without guessing any of those transport or
platform values.

## 6. Verification

- Verify every Finance user-owned command/query obtains the current identity
  only through `CurrentUserPort`.
- Verify client request DTOs contain no owner identity field used for
  authorization.
- Verify Finance configuration and code have no direct Identity database or
  Redis connection.
- Verify local/test identity adapters are environment-bounded and cannot be
  used as production authentication.
- Verify the production adapter validates only an accepted upstream identity
  contract and fails closed on missing or invalid context.
- Verify Finance data access compares the current opaque identity with the
  Finance-owned owner reference.
- Verify shared Identity changes do not introduce cross-service foreign keys,
  joins, or deletion cascades.

## 7. References

- [Identity Consumer contract](../contracts/identity-consumer.md)
- [Ownership map](../architecture/ownership-map.md)
- [Foundation identity token contract](https://github.com/ranikun-labs/harness-foundation-docs/blob/main/docs/contracts/backend-service-foundation/identity-token-contract.md)
- [Foundation ADR-0015](https://github.com/ranikun-labs/harness-foundation-docs/blob/main/docs/adr/ADR-0015-platform-communication-messaging-scaling.md)
- [Foundation ADR-0017](https://github.com/ranikun-labs/harness-foundation-docs/blob/main/docs/adr/ADR-0017-shared-platform-gateway-identity-physicalization.md)
- [Finance backend architecture identity boundary](https://github.com/ranikun-labs/finance-harness-docs/blob/main/04-backend-architecture/finance-backend-architecture.md)
