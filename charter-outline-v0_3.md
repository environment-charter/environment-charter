# Charter for the environment.* Constraint Family — Outline

**Co-stewards:** Michael Msebenzi (Headless Oracle), Douglas Borthwick (InsumerAPI)
**Status:** Outline v0.3 · April 27, 2026 · For convergence before drafting

---

## Preamble

This charter formalizes the institutional shape of the `environment.*` constraint family — a set of cryptographically-signed, fail-closed authorization constraints for autonomous agent execution that gate action on the verified state of external systems at the moment of execution. The family is currently realized as `environment.market_state` (RFC #9 on agent-intent/verifiable-intent, reference implementation Headless Oracle) and `environment.wallet_state` (RFC #22, reference implementation InsumerAPI). This document describes what the family is, who stewards it, how new constraint types may be admitted, and how the family relates to the Mastercard Verifiable Intent specification on which the constituent RFCs sit.

The charter is published at a neutral, jointly-controlled venue (proposed: a GitHub organization both stewards administer, e.g., `github.com/environment-charter`), with footer references in the descriptions of PR #9 and PR #22. It is not filed against the verifiable-intent repository as a PR. This venue choice preserves the artifact regardless of how the constituent RFCs are eventually merged, absorbed, or superseded by the maintainers of Verifiable Intent, and avoids the steward-asymmetry created by hosting on either steward's own domain.

---

## §1. Statement of purpose

The `environment.*` family addresses a class of agent-execution failure that none of the existing Verifiable Intent constraint types is structured to detect: temporal drift between credential issuance and credential evaluation. Identity, governance, payment, and user-control constraints are static — they were valid at issuance and any verifier with the correct keys can confirm so. Environment-state constraints are temporal — the credential was correct at issuance, and the world changed between issuance and execution. The family exists because this failure class requires its own primitive, its own trust roots, and its own composition discipline.

*To draft:* one to two paragraphs articulating the structural distinction between static and temporal failure modes, with reference to the architectural argument in "The Trust Primitive."

---

## §2. Membership criterion

A constraint type belongs in the `environment.*` family if and only if its failure mode is gating. A constraint whose violation is survivable, optional, or advisory is by definition not in the family. A constraint whose violation must halt execution to preserve safety is a candidate for admission.

The test, applied to a proposed type:

1. *Failure-mode test.* If the constraint evaluates to false at execution time, must the action halt? If the answer is "no, the action may proceed with the constraint marked as failed," the type is not in the family.
2. *Trust-root independence test.* Does the type's trust root sit outside the credential chain (e.g., an external oracle observing a system the credential issuer does not control)? Trust roots that are themselves the agent operator, the principal, or the credential issuer are out of scope.
3. *Reference-implementation test.* The proposer must demonstrate a fail-closed reference implementation operating against real external state. Specification-only proposals are not admissible.

This three-part test is normative. The membership criterion does not impose architectural restrictions on how the type is implemented within those bounds — the family is namespace-extensible by design.

---

## §3. Current state

| Type | RFC | Reference implementation | Status |
|------|-----|--------------------------|--------|
| `environment.market_state` | PR #9 (v0.5.6-draft, HEAD 68f4db4 as of April 25) | Headless Oracle, 28 global exchanges, headlessoracle.com | Active; family-wide normative sections landed |
| `environment.wallet_state` | PR #22 (v0.6.3-draft, 3c92a54 as of April 25) | InsumerAPI, 33 chains, api.insumermodel.com | Active; family-wide normative sections landed |

Family-wide normative sections shared byte-identically across both RFCs, jointly authored:

- §4.6 — `max_attestation_age` freshness primitive
- §4.7 — RFC 8725-aligned algorithm agility
- §4.8 — Field Scope Declaration with three-category taxonomy
- §5.5 — Family Composition with conjunction-with-completeness semantics
- §6.8 — JWKS caching and rotation discipline
- §6.9 — Cross-chain temporal consistency (where applicable)
- Appendix C — Register discipline

The family also propagates upstream changes from Verifiable Intent's transactional-constraint surface as those changes land. PR #23 (Mastercard, April 20) introduced field-alignment-v2 renames — VCT version suffixes, constraint-prefix collapse, allowed-list pluralisation. Both RFCs propagated the renames in their next patch revision: PR #9 v0.5.6-draft (commit 68f4db4) and PR #22 v0.6.3-draft (commit 3c92a54). Pure downstream propagation, no semantic change to the family's own constraint types, additive PR #23 fields out of scope. This pattern is named in §6 below as part of the family's reactive engagement with Verifiable Intent maintainers.

*To draft:* brief description of each family-wide section, with reference to the relevant RFC text.

---

## §4. Stewardship

Michael Msebenzi and Douglas Borthwick steward the family jointly. Stewardship is not ownership — the family is a public good, not a closed standard. Stewardship means:

- *Maintenance of family-wide normative sections.* When a section needs revision, both stewards review and converge before any RFC is updated. The §5.5 Block 3 convergence pattern is the operational model.
- *Review of proposed new types.* Third-party proposals for additional `environment.*` constraint types are evaluated by both stewards against the membership criterion. Both must concur for admission.
- *Advocacy in standards-body venues.* Either steward may engage IETF SPICE, RATS, WIMSE, or Web Bot Auth working groups on behalf of the family. Engagements are recorded in this charter's revision history.

Stewardship does not require unanimity on individual constraint-type drafting. The stewards may co-author, divide, or recuse on specific RFCs without affecting the family's institutional shape.

---

## §5. Extension protocol

A third-party implementer proposing a new `environment.*` constraint type follows this protocol:

1. *Failure-mode test demonstration.* Prose argument that the proposed type's failure mode is gating per §2.1.
2. *Reference implementation.* A working, fail-closed implementation operating against real external state. Specification-only proposals are returned without further review.
3. *Family-wide alignment.* Demonstration that the proposal aligns with the family-wide normative sections enumerated in §3, or a substantive argument for why the family-wide sections should be revised to accommodate the proposal.
4. *Co-steward review.* Both stewards review and concur, or both decline, or both request revision. A single declination does not block; the co-stewards converge before responding.

Proposals are tracked at github.com/agent-intent/verifiable-intent as RFCs against the Verifiable Intent repository, following the established pattern of PR #9 and PR #22.

*To draft:* worked example of how a proposal moves through this protocol, using a hypothetical type for illustration.

---

## §6. Relationship to Verifiable Intent

The `environment.*` family sits within the Mastercard-maintained Verifiable Intent specification at github.com/agent-intent/verifiable-intent. The constituent RFCs (PR #9, PR #22) are open against that repository. Maintainer decisions on those RFCs — merge, absorb, supersede, decline — affect the constituent specifications but do not affect this charter, which is canonical at the jointly-controlled neutral venue (see §1) and survives any specific RFC outcome.

The stewards engage Verifiable Intent maintainers reactively, not proactively. The git histories of PR #9 and PR #22 already document the constituent work; the maintainers may incorporate at their pace. The April 20 PR #23 propagation (PR #9 v0.5.6, PR #22 v0.6.3) is the worked example: when the maintainers land transactional-constraint changes, the family propagates the relevant renames into its own examples and prose cross-references in the next patch revision, without requiring family-wide coordination or semantic changes to the constituent RFCs.

If Verifiable Intent v1.0 freezes with `environment.*` constraint types adopted, the charter becomes documentation of the namespace's history and stewardship. If v1.0 freezes without them, the charter remains the canonical reference for implementers building against either RFC.

---

## §7. Operating cadence

The stewards have established an operating pattern that the charter documents but does not formalize as binding:

- *Coordinated revision rounds.* Substantive changes to one constituent RFC are mirrored in the other within roughly 24-72 hours.
- *Byte-identical family-wide normative prose.* Sections enumerated in §3 are authored once and reproduced byte-identically across both RFCs.
- *Lockstep release on shared bundles.* Family-wide changes are released as coordinated bundles across both RFCs, not as drift-then-reconciliation.
- *Drafter-reviewer cross-instance discipline.* For any family-wide section, one steward drafts and the other reviews substantively before commit.
- *Upstream-propagation handling.* Downstream propagation of Verifiable Intent transactional-constraint changes (e.g., PR #23 field-alignment-v2) is handled per-spec at the next patch revision and does not require family-wide coordination, because the changes touch examples and prose cross-references rather than the family's own normative surface.

This pattern is observed practice. Either steward may sit out a specific round of work without it constituting a relationship rupture or a charter violation.

---

## §8. Revision history

| Version | Date | Changes |
|---------|------|---------|
| v0.1 | 2026-04-XX | Initial charter, co-bylined |

---

## Open questions for convergence (not in final charter)

1. *§1 prose.* How much of the architectural argument from "The Trust Primitive" should be reproduced here vs. cited? Lean: cite the essay, summarize in 2-3 sentences only.
2. *§3 family-wide section descriptions.* Brief paragraph each, or just reference the RFC text? Lean: reference RFC text; the charter is not the spec.
3. *§5 worked example.* Hypothetical type, or real one we'd both endorse if proposed? If real, which?
4. *§6 venue language.* Is "reactive engagement" the right phrasing, or is there a less politically-loaded term?
5. *§7 binding question.* Confirm the "observed practice, not binding" framing matches your Q5 answer. Word it differently if it lands better.
6. *Charter venue (Q6 narrowed).* Douglas's pushback on the Apr 27 v0.2 framing is accepted: hosting on either steward's domain creates soft asymmetry. Two neutral options on the table: (a) a jointly-administered GitHub organization, e.g., `github.com/environment-charter`, with the charter at the org root and PR/issue history serving as governance log; (b) a neutral domain, e.g., `environment-charter.dev` or `environment-namespace.org`, with both stewards on DNS/registrar. Author preference: Option (a) — version-controlled, free hosting via GitHub Pages, charter-governance maps cleanly to PR/issue mechanics, no renewal-coordination overhead. Awaiting Douglas's vote.

---

*Outline ends. Drafting begins after convergence on structure and the six open questions above.*
