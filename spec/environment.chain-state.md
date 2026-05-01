# environment.chain_state — v0.1-draft

**Status:** Draft. Sibling to environment.market_state (agent-intent/verifiable-intent PR #9, v0.5.6-draft at 68f4db4) and environment.wallet_state (agent-intent/verifiable-intent PR #22, v0.6.3-draft at 3c92a54).

**Family integration posture.** Family-wide sections — §4.6 Freshness, §4.7 Algorithm Agility, §4.8 Field Scope Declaration, §5.5 Family Composition, §6.8 JWKS Caching and Key Rotation, Appendix C Register Discipline — currently live upstream in the verifiable-intent PRs. They are referenced by section number in this draft rather than duplicated. A coordinated v0.2 round in this repository (environment-charter) will lift those family-wide sections in here verbatim alongside market_state and wallet_state spec/schema pairs, restoring the byte-identical-prose discipline established between PRs #9 and #22.

**Subject:** Chain identified by CAIP-2 namespace.

**Operational state attested:** liveness, finality progress, and RPC-provider consensus at a known time.

---

## 1. Notational Conventions

The key words MUST, MUST NOT, REQUIRED, SHOULD, SHOULD NOT, RECOMMENDED, MAY, and OPTIONAL in this document are to be interpreted as described in BCP 14 (RFC 2119, RFC 8174) when, and only when, they appear in all capitals, as shown here.

## 2. Motivation and Scope

### 2.1 Motivation

Autonomous financial agents make decisions whose correctness depends on assumptions about chain-level liveness that are typically left implicit. An agent that signs a transaction assumes the destination chain is producing blocks; an agent that observes a settlement assumes finality is advancing; an agent that depends on RPC reads assumes its providers are not diverging. These assumptions fail silently in conditions that are operationally common — chain halts, finality stalls, RPC fork divergence, validator outages — and the failure mode is silent because the agent has no environmental primitive that reports the state of the chain itself.

environment.chain_state provides that primitive. It produces signed, time-bounded attestations that a named chain (by CAIP-2 identifier) is in a known operational state, with multi-axis freshness obligations exposed to the consumer. It composes with environment.market_state and environment.wallet_state under family-level conventions.

### 2.2 Sibling Relationship

This specification is the third member of the environment.* primitive family. The sibling relationship to environment.market_state and environment.wallet_state is structural: same fail-closed idiom, same lifecycle model, same composition model, same family-wide discipline on §4.6 / §4.7 / §4.8 / §5.5 / §6.8 / Appendix C.

The TOCTOU class of harm that motivates environment.market_state (an agent with a valid VI credential executing into a halted market) and environment.wallet_state (an agent executing into a wallet that no longer holds the required balance) extends to chain liveness: an agent with a valid VI credential whose conditions were checked at L2 can still execute against a chain that has halted, stalled finality, or fragmented across RPC providers between L2 issuance and L3 verification time. Chain state is environmental, not transactional. Per-attestation freshness on market_state and wallet_state does not absorb chain-level liveness failures, because their attestations describe market sessions and wallet contents, not chain operational state.

Standards alignment note: at v0.1-draft no upstream standards body has named this primitive. Visa's Apr 29, 2026 nine-chain settlement expansion and the broader multi-chain agentic-payment surface (Mastercard VI, Google AP2, Coinbase x402, ERC-8004) make a chain-liveness primitive operationally necessary; the draft is positioned for adoption rather than ratification.

### 2.3 Scope

In scope:

- Attestation that a chain is live, degraded, stalled, or unknown
- Multi-axis freshness exposure (chain liveness, finality lag, RPC consensus)
- Family-level composition with environment.market_state and environment.wallet_state under §5.5
- Fail-closed semantics on UNKNOWN and on every freshness-axis violation

Out of scope for v0.1:

- Specific finality models per chain family (deterministic vs probabilistic) — issuer-declared, see §6.9
- Bridge-state or cross-domain message attestation — separate primitive
- MEV / mempool conditions — separate primitive

## 3. Attestation Interface (abstract)

This section follows the family precedent established at PR #22 v0.2 (provider-neutrality contract): the spec defines an abstract attestation interface; a reference implementation appears in §7.

A conformant chain_state attestation issuer MUST:

- Expose an HTTPS POST endpoint returning a signed attestation
- Sign attestations using the algorithm declared in §4.7
- Publish signing keys discoverable via the trusted_jwks URL named in the constraint
- Carry the seven core attestation claims defined in §4.1

A second conformant implementation needs to emit the seven core claims and publish the appropriate key-discovery document. That is the bar.

## 4. Specification

### 4.1 Attestation core claims

Every chain_state attestation MUST carry the following seven core claims:

| Claim | Type | Description |
|---|---|---|
| `iss` | string | Issuer identity (hostname). Verifier matches against constraint `expected_issuer`. |
| `sub` | string | CAIP-2 chain identifier. Verifier matches against constraint `chain_id`. |
| `iat` | integer | Issued-at, Unix seconds. |
| `exp` | integer | Expires-at, Unix seconds. |
| `status` | string | One of `LIVE` / `DEGRADED` / `STALLED` / `UNKNOWN`. See §4.2. |
| `freshness` | object | Multi-axis freshness obligations per §4.6. |
| `kid` | string | Key identifier. Verifier matches against constraint `expected_kid`. |

The following claims are OPTIONAL and, when present, provide consumer convenience:

| Claim | Type | Description |
|---|---|---|
| `tip_height` | integer | Block height at most-recent chain-liveness observation. Useful for consumer-side staleness cross-check. |
| `finalized_height` | integer | Most recently observed finalized block height. Semantics chain-family-specific, see §6.9. |
| `block_timestamp` | integer | Chain-tip block timestamp at attestation time, Unix seconds. When present, drives `max_cross_chain_skew_seconds` evaluation under §6.9; otherwise verifier falls back to `iat`. |

### 4.2 Status field

The `status` claim MUST be one of `LIVE`, `DEGRADED`, `STALLED`, or `UNKNOWN`.

- **LIVE.** The chain is producing blocks at the issuer's expected cadence, finality is advancing within the issuer's finality-lag threshold, and RPC consensus holds at or above the issuer's quorum threshold.
- **DEGRADED.** At least one liveness signal is off: block production is slowed, finality lag exceeds threshold, or RPC consensus has fallen below quorum but providers are not in fork divergence. Consumers MAY proceed with caution; issuers SHOULD include sufficient detail in `freshness` for consumers to make their own determination.
- **STALLED.** The chain is not producing new blocks, or finality has not advanced past the issuer-defined threshold. Consumers SHOULD NOT submit time-sensitive transactions to a STALLED chain.
- **UNKNOWN.** Pre-publication state, or telemetry insufficient to determine status. Issuers MUST emit UNKNOWN rather than guess. Consumers MUST treat UNKNOWN as a fail-closed condition for any constraint with `expected_status: LIVE`.

### 4.3 Subject identification

The subject of a chain_state attestation is identified by the attestation's `sub` claim, which MUST be a CAIP-2-conformant namespace string. The verifier MUST match `sub` against the constraint's `chain_id` field.

CAIP-2 examples: `eip155:1`, `eip155:8453`, `solana:5eykt4UsFv8P8NJdTREpY1vzqKqZKvdp`, `bip122:000000000019d6689c085ae165831e93`, `cosmos:cosmoshub-4`.

The issuer SHOULD attest only to chains for which it operates direct telemetry. Issuers MUST NOT relay attestations from third parties without re-signing under their own key.

### 4.4 TTL

`iat` and `exp` are Unix-seconds timestamps. Verifiers MUST reject attestations where the current consumer-clock time is later than `exp`. Issuers SHOULD set TTLs no longer than the most stringent freshness-axis bound declared at §4.6.

For production chains (Ethereum mainnet, Base, Solana, etc.), recommended TTL is 30–120 seconds. Issuers MAY use longer TTLs for chains with slower block cadence (e.g., Bitcoin), but SHOULD document the rationale.

### 4.5 Joint composition example

The following non-normative example shows a single mandate carrying environment.market_state, environment.wallet_state, and environment.chain_state constraints in conjunction (NYSE must be open, settlement wallet must hold ≥1 USDC, and Base must be live):

```json
{
  "constraints": [
    {
      "type": "environment.market_state",
      "mic": "XNYS",
      "expected_status": "OPEN",
      "max_attestation_age": 60,
      "...": "..."
    },
    {
      "type": "environment.wallet_state",
      "subject_wallet": "0xabc...",
      "required_condition_hashes": ["0xc938..."],
      "max_attestation_age": 300,
      "...": "..."
    },
    {
      "type": "environment.chain_state",
      "chain_id": "eip155:8453",
      "expected_status": "LIVE",
      "max_attestation_age": 60,
      "...": "..."
    }
  ]
}
```

Per §5.5 the family is a conjunction; failure on any member fails the mandate's environment-gate.

### 4.6 Freshness

The family-wide §4.6 (Freshness), as established in PR #9 and PR #22, applies. The `max_attestation_age` constraint field is REQUIRED, no default, family-wide-trust-root-agnostic. Verifiers MUST reject mandates where `max_attestation_age` is absent or malformed.

In addition to family-wide `max_attestation_age`, chain_state attestations expose three per-axis freshness signals inside the attestation's `freshness` object. These are issuer-side telemetry obligations exposed to the consumer for visibility, not consumer-side policy bounds:

#### 4.6.1 chain_liveness

`freshness.chain_liveness.last_observed_at` declares the most recent RPC poll confirming tip-height advance. `freshness.chain_liveness.max_age_seconds` is the issuer-declared maximum acceptable age for this signal.

#### 4.6.2 finality_lag

`freshness.finality_lag.last_observed_at` declares the most recent finality measurement. The semantics of "finalized" depend on the chain family — see §6.9.

#### 4.6.3 rpc_consensus

`freshness.rpc_consensus.last_observed_at` declares the most recent multi-provider consensus check. `freshness.rpc_consensus.providers_polled` and `freshness.rpc_consensus.providers_agreeing` give the consumer the inputs to compute consensus ratio independently of the issuer's status determination.

The relationship between family-wide `max_attestation_age` and per-axis `max_age_seconds`: `max_attestation_age` bounds the consumer-policy TOCTOU window between the attestation's `iat` and the consumer's verification time; per-axis `max_age_seconds` declares the issuer's internal telemetry obligations between input observation and attestation signing. The two serve different principals and MUST be independently configurable. This is the same structural separation §4.6 establishes for environment.market_state and environment.wallet_state.

### 4.7 Algorithm Agility

The family-wide §4.7 (Algorithm Agility), as established in PR #9 and PR #22, applies. Per-type MUST-implement declaration plus SHOULD/MAY extension set, verifiers negotiate per constraint instance, RFC 8725 §3.1 hook.

For environment.chain_state v0.1:

| Algorithm | Status |
|---|---|
| Ed25519 (RFC 8032) | MUST-implement |
| ES256 (P-256) | SHOULD |
| Ed448 / ES384 / ES512 | MAY |

Ed25519 as MUST-implement matches the environment.market_state precedent and the Headless Oracle reference-implementation signing stack. ES256 in the SHOULD set permits composition with the JOSE/JWS stack used by environment.wallet_state and the wider VI JWT family.

Verifiers MUST reject attestations signed under unsupported algorithms — fail-closed check, not silent downgrade — per family §4.7.

### 4.8 Field Scope Declaration

Per family-wide §4.8, every constraint field declares its scope under one of three categories: family-wide-trust-root-agnostic, per-type-trust-root-mechanism-bound, or per-type-evaluation-mechanism-bound. The §4.8 preamble extends to JWT claims where operationally relevant.

| Field | Scope | Reference |
|---|---|---|
| `max_attestation_age` | family-wide-trust-root-agnostic | §4.6 |
| `attestation_url` | family-wide-trust-root-agnostic | §3 |
| `trusted_jwks` | per-type-trust-root-mechanism-bound | §6.3 |
| `expected_kid` | per-type-trust-root-mechanism-bound | §6.3 |
| `expected_issuer` | per-type-trust-root-mechanism-bound | §6.3 |
| `stale_cache_fallback_permitted` | per-type-trust-root-mechanism-bound | §6.8 |
| `chain_id` | per-type-evaluation-mechanism-bound | §5.5 |
| `expected_status` | per-type-evaluation-mechanism-bound | §4.2 |
| `min_finality_depth` | per-type-evaluation-mechanism-bound | §6.9 |
| `max_cross_chain_skew_seconds` | per-type-evaluation-mechanism-bound | §6.9 |
| `block_timestamp` (JWT claim) | per-type-evaluation-mechanism-bound | §6.9 |

**Rationale.** `max_attestation_age` and `attestation_url` are family-wide-trust-root-agnostic on the same basis the family settled at v0.5.1: identical semantic role across types, with per-type variance localised at neighbouring fields. `trusted_jwks`, `expected_kid`, `expected_issuer`, and `stale_cache_fallback_permitted` are per-type-trust-root-mechanism-bound because they bind to the JWKS-discovery mechanism this primitive uses for key resolution; a chain_state attestation issuer using a different key-discovery scheme (e.g., RFC 8615 well-known key registry) would name analogous but mechanism-specific fields. `chain_id` is per-type-evaluation-mechanism-bound and serves as the per-member distinguishing field for §5.5: equality-check against the attestation's `sub` claim is the evaluation primitive specific to chain-state. `expected_status`, `min_finality_depth`, `max_cross_chain_skew_seconds`, and the OPTIONAL `block_timestamp` JWT claim are per-type-evaluation-mechanism-bound because they describe the chain-reading evaluation specific to this primitive — environment.market_state reads exchange-session state, environment.wallet_state reads on-chain wallet contents, environment.chain_state reads chain-level liveness and finality.

## 5. Lifecycle Integration

### 5.1 Layer mapping

(Family-wide; identical to environment.market_state §5.1 and environment.wallet_state §5.1 as established in PR #9 and PR #22.)

### 5.2 Execution order

The family-wide §5.2 (Execution order) applies. environment.* constraints MUST be checked before transactional constraints. Agents MUST short-circuit on first environment.* failure during pre-L3 self-gating; verifier-side L3-acceptance evaluation is governed by §5.5's completeness rule (forward-pointer per family-wide §5.2 v0.5.4 update).

### 5.5 Family Composition

The family-wide §5.5 (Family Composition) applies, including conjunction semantics, mixed pass/fail handling, L3 execution gate and completeness rule, per-member diagnostic output, and the agent-phase / verifier-phase bridge sentence.

Per-member disambiguation (§5.5 Block 2) for environment.chain_state: the per-member distinguishing field is **`chain_id`** (REQUIRED per §4 schema, no fallback). When two environment.chain_state constraints appear in the same mandate (a multi-chain mandate, e.g., Base and Solana must both be LIVE), the violations list MUST carry each constraint's array index and the human-readable `chain_id` value. This is the chain-state analog of environment.market_state's MIC-with-attestation-URL-fallback and environment.wallet_state's `subject_wallet`-no-fallback patterns. No fallback is needed because `chain_id` is REQUIRED at the constraint level and always present.

## 6. Security Considerations

### 6.3 Trust binding (key-host, not endpoint)

The family-wide §6.3 trust-root binding pattern applies. Trust binds to the JWKS host (`trusted_jwks` URL) plus the key identifier (`expected_kid`); verifier policy enforces an allowlist on `trusted_jwks` hostnames. JWKS URL migration is a new-mandate event per family precedent.

### 6.5 Constraint Stripping

Family-wide §6.5 applies. Verifiers MUST reject mandates whose L2 signature does not validate over the full constraint list, and MUST NOT accept subset-signed mandates.

### 6.8 JWKS Caching and Key Rotation

Family-wide §6.8 applies without modification. The `stale_cache_fallback_permitted` field follows the family pattern (OPTIONAL boolean, default false, MUST NOT be true for strict-freshness deployments).

### 6.9 Cross-Chain Temporal Consistency

§6.9 introduces chain-state-specific temporal mechanics. The structure is adopted from PR #22 v0.6 (commit feb3292) and localized for the multi-chain-state-attestations-in-one-mandate case.

#### 6.9.1 Adjacency to §6.7

(§6.7 of the family addresses verifier↔issuer wall-clock skew. This section addresses attested-block↔chain-tip skew across multi-chain mandates and within-chain reorg posture — adjacent but distinct.)

#### 6.9.2 Orthogonal axes

Cross-chain wall-clock coordination (`max_cross_chain_skew_seconds`) and within-chain finality remediation (reorg re-verification SHOULD, §6.9.4) are orthogonal axes: the former bounds tip-time divergence across chains at evaluation time; the latter addresses attested-block reorganization on a single chain post-acceptance at settlement time. Multi-chain mandates surface each axis independently per participating chain.

#### 6.9.3 Finality semantics

The semantics of "finalized" depend on the chain family:

- **Deterministic-finality chains** (Cosmos SDK, Solana via TowerBFT, Tendermint-family): `finalized_height` is the most recent height known to be finalized by consensus.
- **Probabilistic-finality chains** (Bitcoin, classic Nakamoto-style): `finalized_height` is the height at which the issuer considers blocks economically final, using the issuer's declared confirmation depth. Issuers SHOULD document confirmation-depth policy out of band.
- **Hybrid chains** (post-merge Ethereum, Base via fault proofs): issuer SHOULD use the strongest available finality signal (Ethereum's `finalized` block tag, Base's challenge-window-elapsed condition).

`min_finality_depth` is verified by the verifier's independent chain read at verification time, not by an additional JWT claim from the issuer. This preserves the §3 abstract-interface contract and keeps the trust model clean — the issuer cannot become the attacker's target for finality manipulation.

#### 6.9.4 Reorg remediation

Verifiers SHOULD re-verify chain-state attestations at application-layer settlement time, scoped to the VI verifier's L3-acceptance logic (excluding chain-client, RPC-transport, and cryptographic-library behaviour). This is an advisory SHOULD, not a synchronous chain-finality mandate; the spec does not own chain consensus.

#### 6.9.5 Measurement basis for max_cross_chain_skew_seconds

Verifiers MUST use the attestation's `block_timestamp` claim when present, falling back to `iat` otherwise. Issuers SHOULD include `block_timestamp` on chains where it exists so the tighter cross-chain dimension is available to verifiers that need it. Non-block-time chains (XRPL, certain Cosmos variants) participate via the `iat` fallback, which `max_attestation_age` already governs on the per-attestation side.

When comparing timestamps for cross-chain skew evaluation, verifiers MUST canonicalize to integer Unix seconds, truncating any sub-second precision before comparison. Verifier-side truncation enforces deterministic cross-chain comparison semantics; issuer-side sub-second precision outside the `max_cross_chain_skew_seconds` evaluation context remains permitted.

## 7. Reference Implementation

The Headless Oracle reference implementation will publish chain_state attestations at `/v5/chain` (planned) on the same Ed25519 signing stack used by the environment.market_state reference implementation. Concrete endpoint details, canonical payload specification, and live test vectors will land in §7 of v0.2 alongside the family-wide prose adoption.

## 8. Open Questions for Working Group

1. **`block_timestamp` REQUIRED vs OPTIONAL.** Argument for REQUIRED: forces issuers to declare a chain-anchored timestamp, makes `max_cross_chain_skew_seconds` evaluation tighter. Argument against: not all chains expose a single block-timestamp notion, OPTIONAL preserves the §3 provider-neutrality contract for chains where this doesn't apply. v0.1 leaves OPTIONAL.

2. **Status enum cardinality.** LIVE / DEGRADED / STALLED / UNKNOWN parallels market_state's OPEN/CLOSED/HALTED/UNKNOWN. Whether DEGRADED-vs-STALLED distinction is operationally useful at the protocol level or should collapse into a single "not-LIVE" non-UNKNOWN state is open. v0.1 keeps four-element enum.

3. **`min_finality_depth` measurement.** v0.1 places the measurement on the verifier (independent chain read) per §6.9.3 to preserve trust-model cleanliness. Alternative would be requiring the attestation to carry a finality-depth claim; rejected for v0.1 because it makes the issuer the trust root for finality, which is an attack surface.

4. **Recommended TTL ceiling.** Family precedent on environment.market_state and environment.wallet_state is informational note rather than normative ceiling. v0.1 follows precedent.

## 9. Changelog

- v0.1-draft (2026-05-01): initial draft. Schema, type-local prose, four open questions for review. Family-wide sections (§4.6, §4.7, §4.8, §5.5, §6.8, Appendix C) referenced from upstream PRs #9 and #22; verbatim adoption deferred to coordinated v0.2 round in environment-charter.
