# environment.* charter

Charter and governance log for the `environment.*` constraint family on the
[Mastercard Verifiable Intent](https://github.com/agent-intent/verifiable-intent)
specification.

## Co-stewards

- **Michael Msebenzi** ([Headless Oracle](https://headlessoracle.com)) —
  reference implementation for `environment.market_state`
  ([RFC #9](https://github.com/agent-intent/verifiable-intent/pull/9))
- **Douglas Borthwick** ([InsumerAPI](https://insumermodel.com)) —
  reference implementation for `environment.wallet_state`
  ([RFC #22](https://github.com/agent-intent/verifiable-intent/pull/22))

## Status

Charter v0.1 in drafting. The pre-drafting outline ([`charter-outline-v0_3.md`](./charter-outline-v0_3.md))
records the convergence state as of April 27, 2026. v0.1 will be co-bylined
on commit.

## Family

The `environment.*` constraint family addresses temporal drift between
credential issuance and credential evaluation in autonomous agent execution.
Currently realized as:

- `environment.market_state` — cryptographically-signed attestation that an
  exchange (XNYS, XNAS, etc.) is OPEN, CLOSED, or HALTED at time of execution
- `environment.wallet_state` — cryptographically-signed attestation that a
  wallet satisfies caller-specified conditions across 33 chains at time of
  execution

The membership criterion and admission process for new constraint types are
specified in the charter.

## Background

The architectural argument for the family is given in the essay
[*The Trust Primitive: Why Agent Commerce Needs Environment-State Attestation*](https://github.com/headlessoracle/essays/blob/main/environment-state-attestation.md)
(Msebenzi, April 28, 2026).
