# Indexer Integration Guide

This document defines event-schema expectations for downstream indexers and
analytics systems consuming the Credit contract.

## Semver and Event Compatibility

- Existing event payloads are preserved as **v1** (no breaking field removal or
  type changes).
- Enriched payloads are emitted as **v2** on separate topics.
- Indexers can migrate incrementally by consuming both families in parallel.

## Topics and Payloads

### Lifecycle events

- `("credit", "opened")`, `("credit", "suspend")`, `("credit", "closed")`,
  `("credit", "default")`, `("credit", "reinstate")` -> `CreditLineEvent` (v1)
- `("credit", "opened_v2")`, `("credit", "suspend_v2")`,
  `("credit", "closed_v2")`, `("credit", "default_v2")`,
  `("credit", "reinstate_v2")` -> `CreditLineEventV2` (v2)

`CreditLineEventV2` adds:

- `timestamp: u64`
- `actor: Address` (caller identity for lifecycle transition)
- `amount: i128` (currently `0` for lifecycle transitions)

### Draw events

- `("credit", "drawn")` -> `DrawnEvent` (v1)
- `("credit", "drawn_v2")` -> `DrawnEventV2` (v2)

`DrawnEventV2` adds identifier fields:

- `recipient: Address`
- `reserve_source: Address`

### Repayment events

- `("credit", "repay")` -> `RepaymentEvent` (v1)
- `("credit", "repay_v2")` -> `RepaymentEventV2` (v2)

`RepaymentEventV2` adds:

- `payer: Address`

### Risk update events

- `("credit", "risk_upd")` -> `RiskParametersUpdatedEvent` (v1)
- `("credit", "risk_upd_v2")` -> `RiskParametersUpdatedEventV2` (v2)

`RiskParametersUpdatedEventV2` adds:

- `timestamp: u64`
- `actor: Address` (admin that performed update)

## Indexer Migration Strategy

1. Keep v1 parsers active for backward compatibility.
2. Add v2 parsers and store both payload versions.
3. Prefer v2 fields for analytics dimensions requiring actor/timestamp/source.
4. Deprecate v1 ingestion only after consumer parity is confirmed.
