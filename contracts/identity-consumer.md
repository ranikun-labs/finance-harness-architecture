# Identity Consumer Contract

Status: Accepted canonical semantic contract; transport details remain
deferred.

Owner: Finance Application
Provider boundary: Shared Identity
Related Jira: `RPL-81`, `RPL-50`

## Purpose

Finance needs the identity semantic required to scope Finance-owned data. It
does not need to own or replicate the identity system's account, credential,
token, session, or persistence model.

## Application contract

```text
CurrentUserPort
  → current principal
  → identityUserId: opaque stable identifier
```

The returned `identityUserId`:

- is assigned and governed by Shared Identity;
- is opaque to Finance;
- is stable for the identity contract's lifetime;
- is used as an external ownership reference for Finance data;
- is not replaced by email, login name, a product-local copy, or a client value.

Finance authorization then applies its own product ownership and policy checks:

```text
CurrentUserPort
  → identityUserId
  → Finance-owned resource lookup
  → owner match and product authorization
```

## Consumer rules

Finance must not:

- accept `userId`, `identityUserId`, or `actorId` from a client as the current
  user;
- access Shared Identity databases, `identity_db`, Identity Redis, or internal
  tables directly;
- copy another product's Auth or JWT model;
- copy organization, role, public-ID, credential, token, or session semantics;
- use a fixed production user fallback;
- make routine product requests perform a synchronous Identity lookup when the
  accepted deployment contract provides locally validated trusted context.

The Finance Application consumes only the semantic it needs. Network, token,
and browser transport are not part of this application contract.

## Adapter lifecycle

| Adapter stage | Allowed meaning |
|---|---|
| Local development/test adapter | Supplies a stable contract-shaped identity for local execution; not production authentication |
| Production identity adapter | Future adapter behind `CurrentUserPort`, enabled only after upstream identity/gateway contract and activation evidence are accepted |
| Missing or invalid identity | Fail closed; do not guess an owner or continue with a fixed user |

## Transport separation

This document does not define or select:

- token format or claims;
- issuer, audience, or JWKS values;
- OAuth entrypoint;
- browser authentication transport;
- refresh/session mechanism;
- final Gateway route or trusted-context field names;
- service-to-service authentication mechanism.

Those details belong to an accepted Shared Platform contract and must not be
inferred from this semantic consumer contract.

## References

- [ADR-002 Shared Identity Consumer Boundary](../adr/ADR-002-shared-identity-consumer-boundary.md)
- [Ownership map](../architecture/ownership-map.md)
- [Foundation identity token contract](https://github.com/ranikun-labs/harness-foundation-docs/blob/main/docs/contracts/backend-service-foundation/identity-token-contract.md)
- [Foundation ADR-0015](https://github.com/ranikun-labs/harness-foundation-docs/blob/main/docs/adr/ADR-0015-platform-communication-messaging-scaling.md)
- [Finance backend identity boundary reference](https://github.com/ranikun-labs/finance-harness-docs/blob/main/04-backend-architecture/finance-backend-architecture.md)
