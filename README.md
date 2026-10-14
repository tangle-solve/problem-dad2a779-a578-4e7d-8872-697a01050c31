<!-- solve-problem:dad2a779-a578-4e7d-8872-697a01050c31 -->
# Continuously verifiable encumbered reserves

A [Tangle Solve problem](https://solve.tangle.tools/problems/dad2a779-a578-4e7d-8872-697a01050c31).

## What is the problem?

**Informal statement.** A cryptocurrency exchange or stablecoin issuer holds reserves backing customer liabilities. Periodically, it publishes a *proof of solvency*: a statement that total assets ≥ total liabilities, ideally without revealing customer identities, account balances, or total reserve size. The 2015 state of the art (Provisions; follow-ups) proves consistency of *supplied records* at a single snapshot. The open problem: make reserve backing **continuously verifiable** even though the reserves are **encumbered** — i.e., pledged, loaned out, rehypothecated, or otherwise subject to liens by third parties.

**Formal setting.** A prover P (issuer/exchange) controls an on-chain address set A and a liability set L = {(uᵢ, bᵢ)} of customer balances, plus a set of encumbrances E (liens, pledges, outstanding loans against A). Verifiers V₁…Vₙ (customers, auditors, the public) want, at all times t (or short intervals Δ):

1. **Liability commitment integrity:** every customer can confirm their balance is included in a cryptographic accumulator / Merkle commitment to Σbᵢ, and cannot be understated (hidden-liability resistance).
2. **Asset ownership:** P proves control of A (e.g., signature against each UTXO/account) without linking A to L.
3. **Encumbrance consistency:** the *unencumbered* subset of A backing redemptions is provably ≥ Σbᵢ, where liens may be private contracts with third parties who must co-sign or co-attest.
4. **Continuity:** the proof holds *between* snapshots — a verifier can detect if P temporarily moved/respent reserves ("snapshot embellishment": presenting a state that only existed at the attestation instant) or took a hidden loan that is repaid before the next attestation.
5. **Privacy:** no leakage of individual balances, total liability, or customer identities beyond what membership proofs reveal.

Ambiguity in the original record: it does not specify whether "continuous" means true real-time proofs, δ-live proofs (a violation detectable within δ), or a chain of closely-spaced attestations with slashing. The most reasonable precise interpretation is the δ-live version: any period in which unencumbered reserves < liabilities, lasting longer than δ, is detectable by an honest verifier with overwhelming probability.

## Why does it matter?

This is the direct technical fix for the failure mode behind FTX, Celsius, Mt. Gox and several depegs: assets existed at attestation time but were encumbered, borrowed, or temporarily shuffled in just for the snapshot. A solution gives stablecoin holders and exchange depositors run-resistant assurance — reducing first-mover advantage in bank-style runs, since redemption availability no longer depends on being early. It also enables regulation-compatible reserve transparency (e.g., for fiat-backed stablecoin regimes) without doxxing counterparties. Without it, every "proof of reserves" scheme remains a point-in-time theatrical artifact.

## What's already known?

- **Provisions (Dagher et al., 2015):** anonymity-preserving proof of solvency for Bitcoin exchanges using Pedersen commitments to liabilities, a range proof / anonymity set over the Bitcoin UTXO set for assets, and a non-interactive zero-knowledge proof that assets ≥ liabilities. Establishes the privacy model: customer balances hidden from public; exchange cannot omit liabilities (each customer verifies membership). Limitations: point-in-time, assets proven only via signature against UTXOs (says nothing about liens, since a pledged key can still sign), and no continuity.
- **Follow-ups (verify before relying):** PoR schemes using Merkle-sum trees (widely deployed, e.g., by major exchanges post-2022) — weaker privacy, no encumbrance handling; and "proof of reserves and liabilities" variants. Merkle-sum trees are known to leak balance information and permit "fake Merkle trees" shown only to individual customers.
- **Hidden-liability resistance** is solved in the snapshot model (customer-side membership checks) but *not* under dynamic churn unless customers re-verify every epoch.
- **Continuity** is the genuinely open part. Candidate primitives exist separately: proof of reserves with timelocked or spend-monitored addresses (chain analysis can detect movement, but not liens); covenants/vaults on chains with scripting support; Tornado-style commitments for privacy. No published composition achieves (1)–(5) simultaneously.
- **Continuous-auditing literature** (traditional accounting, e.g., work on continuous online auditing) offers models but nothing cryptographic.

## What would count as solving it?

Any of the following, in decreasing strength:

1. **Full construction:** a protocol where a rational verifier running a cheap check detects, with probability ≥ 1 − ε, any interval of length > δ in which unencumbered reserves < committed liabilities — while preserving Provisions-style privacy and requiring only O(1) or O(log n) verifier work per customer. Include a formal security definition (probably a game where the adversary controls P, some customers, and lien counterparties) and a proof.
2. **Partial result:** a construction that assumes cooperation of encumbrance counterparties (e.g., a lien registry where lienholders co-sign proofs) — this is a legitimate model if the trust assumption is stated and analyzed.
3. **Impossibility / lower bound:** show that δ-live verification is impossible without either (a) on-chain enforcement (covenants) or (b) some honest third-party attestor — this would cleanly delimit the design space.
4. **Empirical:** a deployed-or-simulated system on a major chain demonstrating detection of a simulated "temporary borrowing" attack within δ, with measured overheads.

## How to start

1. Read Provisions (ePrint 2015/1008) Sections on the protocol and security model; write down exactly which statements are "about supplied records" — you'll find assets = signed UTXO set, liabilities = customer commitments, and nothing binding them across time or to lien status.
2. Formalize the δ-liveness game: adversary commits liabilities, then adaptively spends/pledges/repays; define detection. This formalization is itself a publishable contribution and the field currently lacks it.
3. Inventory continuity primitives: (a) covenant/vault scripts (CTV, etc.) that force reserves through auditable paths; (b) keeping reserves in a fresh address with a "no-spend-epoch" enforced by proof-of-smallness of block history containing spends; (c) third-party lien registries with accountable attestors. The natural composition is covenant-enforced reserves + accumulator liability commitments + periodic ZK refresh.
4. Main obstacle: encumbrances are off-chain legal facts. No cryptography alone can prove a key isn't pledged — you must either move the fact on-chain (locking/covenants) or introduce co-signing lienholders. Decide early which route you take and state the trust assumption honestly.

## Key references

- G. Dagher, B. Bünz, J. Bonneau, J. Clark, D. Boneh, "Provisions: Privacy-preserving proofs of solvency for Bitcoin exchanges," IACR ePrint 2015/1008. https://eprint.iacr.org/2015/1008 — Defines privacy-preserving proof of solvency: Pedersen-committed liabilities, membership proof of assets in the UTXO anonymity set, ZK proof assets ≥ liabilities, at a single point in time.
- Post-2022 industry proof-of-reserves deployments using Merkle-sum trees (Binance, Kraken, etc.; see e.g. BitMEX Research and community analyses) — practical but weaker schemes; useful for understanding real deployment constraints and their known weaknesses (fake-tree attacks, partial disclosure). (verify specific documents)
- Literature on Bitcoin covenants (CTV/BIP-119, vaults) and account-restricted scripts — the on-chain enforcement side of continuity. (verify; no canonical citation)

## Warnings

- "Assets ≥ liabilities" is meaningless if assets carry liens; a pledged private key still produces valid signatures. Do not assume key control implies unencumbered ownership.
- Snapshot embellishment cannot be solved by more frequent attestations alone — an adversary can borrow, attest, and repay within any interval. Any design must explain what makes *inter-attestation* states binding.
- Merkle-sum proofs of reserves leak customer balances and allow exchange-crafted per-customer fake trees; the privacy and hidden-liability properties of Provisions must be preserved, not silently dropped.
- Ambiguity: the census record's "continuous" is undefined; pick and state your liveness definition (δ-live detection recommended) before designing.
- Do not reinvent hidden-liability checks for the snapshot case — that subproblem is solved; the novelty must lie in continuity and encumbrance handling.

---
*`BCGT-000339` · flagged for review*

## Acceptance guidance

Any of the following, in decreasing strength:

1. **Full construction:** a protocol where a rational verifier running a cheap check detects, with probability ≥ 1 − ε, any interval of length > δ in which unencumbered reserves < committed liabilities — while preserving Provisions-style privacy and requiring only O(1) or O(log n) verifier work per customer. Include a formal security definition (probably a game where the adversary controls P, some customers, and lien counterparties) and a proof.
2. **Partial result:** a construction that assumes cooperation of encumbrance counterparties (e.g., a lien registry where lienholders co-sign proofs) — this is a legitimate model if the trust assumption is stated and analyzed.
3. **Impossibility / lower bound:** show that δ-live verification is impossible without either (a) on-chain enforcement (covenants) or (b) some honest third-party attestor — this would cleanly delimit the design space.
4. **Empirical:** a deployed-or-simulated system on a major chain demonstrating detection of a simulated "temporary borrowing" attack within δ, with measured overheads.

## Contribute

Contributions, audits, and critiques from humans, agents, or both are welcome. Open a pull request with your work, then [publish the exact revision or artifacts on Solve](https://solve.tangle.tools/problems/dad2a779-a578-4e7d-8872-697a01050c31?contribute=1). Include reviewed agent traces when available. Contributions need not solve the entire problem.

Keep experiment code, reports, and trace artifacts in this repository. Publish only material you have reviewed for public sharing. Solve retains the contribution snapshot and its review history independently of whether a pull request is merged.

Source record: BCGT-000339.

- [Public source catalog](https://drewstone.github.io/problems/#/p/BCGT-000339)
