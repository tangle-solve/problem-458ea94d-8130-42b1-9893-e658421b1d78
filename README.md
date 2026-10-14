<!-- solve-problem:458ea94d-8130-42b1-9893-e658421b1d78 -->
# Free-option-resistant PBS under strategic failures

A [Tangle Solve problem](https://solve.tangle.tools/problems/458ea94d-8130-42b1-9893-e658421b1d78).

## What is the problem?

In Ethereum's Proposer-Builder Separation (PBS) protocol — and specifically in the enshrined design of EIP-7732 ("ePBS") — a proposer selects and signs a block header containing a builder's commitments (the builder's payment bid and a commitment to the payload), and the builder later reveals the full payload. The protocol problem: this structure creates a *free option*. Once the proposer has committed (or once the builder has committed its bid), the party with the residual move can withhold the payload or otherwise fail strategically, and the protocol's payment/fallback rules do not price this abandonment incentive — especially when either party holds **outside positions** contingent on block contents (e.g., the proposer runs MEV-extracting infrastructure, the builder has CEX hedges or long/short positions in tokens whose price depends on whether a block lands).

Formally: model an extensive-form game between proposer P and builder B. P has an outside payoff function that depends on payload content and landing time; B likewise. Protocol parameters: payment rule (when P pays B), slashing/collateral schedule (whether withheld or invalid payloads forfeit a deposit), fallback rules (local block, subsequent-builder auction, or no block), and timing. The open problem is to **design (and prove properties of) a payment/collateral/fallback mechanism such that no equilibrium involves strategic withholding**; i.e., the protocol is *free-option-resistant*: any gain from abandonment is dominated by protocol-enforced forfeitures, with outside positions correctly priced in. This is a mechanism-design problem, not an empirical one, though simulation of calibrated games is admissible supporting evidence.

Ambiguities to note: "outside positions" is not formally defined in the census record; the most reasonable reading is any payoff to a party contingent on block landing/content that is not captured by the protocol's bid/payment. Whether collateral is denominated in ETH or burnable stake is left open.

## Why does it matters?

Solving it would make ePBS (EIP-7732) safe to finalize: today, trustless payment lets a builder hold a free option to withhold a payload (e.g., to front-run a CEX hedge, or to grief a proposer), which is a core reason enshrined PBS has stalled in review. A provably free-option-resistant design would let Ethereum remove reliance on trusted intermediaries (relays, MEV-Boost) while pricing abandonment risk honestly, and would generalize to fair-exchange protocols wherever participants hold external contingent positions. If unsolved, PBS stays mediated by semi-trusted relays, and protocol-level builder markets carry unpriced tail risk.

## What's already known?

- **MEV-Boost / out-of-protocol PBS:** proposers delegate to relays; the known free-option pathology is the *builder withholding* problem — a committed builder can withhold the payload, forcing a missed slot, and existing designs make the proposer bear this cost. Conversely, in commit–reveal bid schemes, proposers gain a free option on the builder's price (see combinatorial/first-price auction analyses in the MEV literature). (verify exact attributions)
- **EIP-7732 (2024, Draft/Review):** enshrines the commitment structure — proposer signs header with builder bid and payload commitment; payload delivered in a separate attestation phase; payments are enforced in-protocol. It handles honest failure, but its payment rules do **not** charge for the option value created when a party profits from non-landing — this is the stated gap.
- **D'Amato et al. (2024, evolving):** analyses of PBS market structure and the builder's strategic incentives under ePBS-style timing; identifies the free option and sketches collateral/pricing directions but does not give a full mechanism with a no-withholding equilibrium proof. (verify current status)
- **Related tooling:** two-party fair exchange is classically impossible without a trusted third party or resource-burning (Cleve 1986-style impossibility); so any solution must either burn collateral, use repeated-game incentives, or exploit the chain itself as the trusted third party. Partial-payload / staged-reveal and slashing-based designs exist in the relayer literature.

No construction currently achieves a proven no-strategic-withholding equilibrium with outside positions, and no lower bound on required collateral size is known.

## What would count as solving it?

Any of the following, in increasing strength:

1. **Mechanism with proof:** an explicit payment/collateral/fallback rule for the EIP-7732 game and a theorem that, for any outside-position profiles (within a stated class, e.g., payoff bounded by V), no agent's best response involves strategic withholding — i.e., withholding is strictly dominated in every subgame-perfect equilibrium.
2. **Matching bound:** a lower bound showing the collateral/payment forfeiture must be at least some function of the maximum outside-position value V (a "price of the free option"), plus a mechanism achieving it — an optimal-pricing result.
3. **Impossibility result:** a proof that no protocol within a defined class (no trusted third party, bounded collateral) can eliminate the free option when outside positions are unobservable, which would itself resolve the problem negatively and redirect design toward repeated-game or reputation mechanisms.
4. **Experimental support (supplementary, not sufficient alone):** calibrated agent simulations showing withholding rates collapse under the proposed schedule versus MEV-Boost baselines.

## How to start

1. **Read EIP-7732 first**, focusing on the execution/consensus split, the payload-attestation phase, and exactly when payments become enforceable. Write down the extensive-form game tree with timing; identify precisely which histories admit a profitable deviation given an outside position of value V.
2. **Read D'Amato et al. (2024)** for the current formalization of builder incentives under ePBS; reuse their notation if possible so results compose.
3. **Model the option:** treat the builder's post-commitment choice as a real option whose value depends on outside-position volatility (CEX price moves within a slot ≈ 12 seconds). A natural first step is computing the option value under a price-diffusion model, then asking what collateral C makes withholding unprofitable: C ≥ option value. Check whether feasible C is bounded by what builders can plausibly stake.
4. **Main obstacle:** outside positions are unobservable to the protocol, so first-best (position-specific pricing) is unreachable; you must price against a worst case or bound V via collateral caps. The key open design lever is the fallback rule — how quickly and how well the proposer can substitute a local/secondary block determines the builder's withholding payoff.

## Key references

- **EIP-7732: Enshrined Proposer-Builder Separation.** https://eips.ethereum.org/EIPS/eip-7732 — Specifies the protocol whose payment/fallback structure creates the free option; defines the commit/attest timing game any analysis must use.
- **D'Amato et al. (2024, evolving)** — Analysis of PBS market structure and builder/proposer strategic incentives under ePBS; identifies the unpriced abandonment incentive. (verify full citation and venue; the census record does not give a complete reference)
- **Cleve, R. (1986), "Limits on the Security of Coin Flips when Half the Processors are Faulty" (and the fair-exchange literature building on it)** — Establishes the impossibility of unconditionally fair two-party exchange without a third party, bounding what any PBS payment design can achieve. (verify you cite the canonical fair-exchange impossibility you use)
- **MEV-Boost documentation / Flashbots research (mevboost.pics and Flashbots writings on builder withholding)** — Empirical and design context for out-of-protocol PBS and observed withholding/griefing behavior. (verify)

## Warnings

- The D'Amato et al. (2024) reference is marked "evolving" in the census record; locate and cite the actual current version rather than relying on this brief's characterization.
- Do not assume observability of outside positions — that is the crux; solutions that condition payments on "the builder's hedge" are vacuous.
- Distinguish honest failures (network loss, invalid-but-honest bugs) from strategic withholding: EIP-7732 explicitly tolerates the former, and a good mechanism must not slash honest failures or builders will exit.
- Beware double-counting the two distinct free options: the *proposer's* option on builder price (pre-commit) and the *builder's* withholding option (post-commit); fixing one can worsen the other.
- Repeated-game/reputation "solutions" only work if you can argue persistence of identity and stake across slots; state that as an assumption or prove it.

---
*`BCGT-000312` · flagged for review*

## Acceptance guidance

Any of the following, in increasing strength:

1. **Mechanism with proof:** an explicit payment/collateral/fallback rule for the EIP-7732 game and a theorem that, for any outside-position profiles (within a stated class, e.g., payoff bounded by V), no agent's best response involves strategic withholding — i.e., withholding is strictly dominated in every subgame-perfect equilibrium.
2. **Matching bound:** a lower bound showing the collateral/payment forfeiture must be at least some function of the maximum outside-position value V (a "price of the free option"), plus a mechanism achieving it — an optimal-pricing result.
3. **Impossibility result:** a proof that no protocol within a defined class (no trusted third party, bounded collateral) can eliminate the free option when outside positions are unobservable, which would itself resolve the problem negatively and redirect design toward repeated-game or reputation mechanisms.
4. **Experimental support (supplementary, not sufficient alone):** calibrated agent simulations showing withholding rates collapse under the proposed schedule versus MEV-Boost baselines.

## Contribute

Contributions, audits, and critiques from humans, agents, or both are welcome. Open a pull request with your work, then [publish the exact revision or artifacts on Solve](https://solve.tangle.tools/problems/458ea94d-8130-42b1-9893-e658421b1d78?contribute=1). Include reviewed agent traces when available. Contributions need not solve the entire problem.

Keep experiment code, reports, and trace artifacts in this repository. Publish only material you have reviewed for public sharing. Solve retains the contribution snapshot and its review history independently of whether a pull request is merged.

Source record: BCGT-000312.

- [Public source catalog](https://drewstone.github.io/problems/#/p/BCGT-000312)
