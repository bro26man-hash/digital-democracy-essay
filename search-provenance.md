# Search Provenance & Synthesis Summary

> This document summarizes the GitHub exploration that informed the essay outline in `essay-outline.md`. The full outline (already in that file) was synthesized from four parallel searches:

## 1. Repository Search — "Blockchain Voting"

| Project | Stars | Language | Key Idea |
|---|---|---|---|
| **mehtaAnsh/BlockChainVoting** | 450 | JavaScript/Solidity | Full-stack e-voting dApp: candidate registration, voter auth via email, on-chain vote casting, IPFS media storage |
| **cardano-foundation/jormungandr** | 368 | Rust | Privacy-focused voting blockchain node — formal methods and peer-reviewed cryptography |
| **yfgeek/BlockVotes** | 283 | PHP | Ring-signature anonymization for voters — process transparency vs. voter privacy |
| **BuildOnViction/victionchain** | 182 | Go | Proof-of-Stake voting consensus — governance mechanism as the consensus layer |
| **KashifCh-eth/blockchain-voting-system-** | 46 | JavaScript | Lightweight reference implementation — minimum viable architecture for on-chain elections |

## 2. Repository Search — "DAO Governance"

| Project | Stars | Language | Key Idea |
|---|---|---|---|
| **DA0-DA0/dao-contracts** | 217 | Rust (CosmWasm) | Modular: voting-power + proposal + treasury modules, any-to-any composability, audited by Oak Security |
| **ensdomains/governance-contracts** | 159 | JavaScript | ENS DAO — delegation voting, airdrop infra, real-world adversarial testbed |
| **decentraland/governance** | 49 | TypeScript | Snapshot off-chain hashing + on-chain enactment; multi-strategy voting (ERC-20/721 multipliers) |
| **gov4git/gov4git** | 216 | Go | Git-based off-chain governance — asks: *does governance need a blockchain at all?* |
| **Joystream/pioneer** | 43 | TypeScript | Governance for a decentralized streaming protocol — content moderation, upgrades, revenue |

## 3. Code Search — On-Chain Voting Contract Implementations

Key patterns surfaced:
- **EIP-2612 (Permit) + EIP-712** — Signature-based gasless voting; domain-separated structured data prevents cross-chain replay
- **Token-deposit-weighted voting** — Voters must lock tokens for a period to gain voting power (TerraBioDAO pattern); partial flash-loan mitigation
- **Staking for voting power** — Time-weighted staking accumulates voting power; `whenNotPaused` emergency stop mechanism
- **Quadratic voting** — sqrt(balance) formula; LeapDAO's production deployment at Volt Germany using Optimized Sparse Merkle Trees
- **zykm-SNARK tallying** — BlockVote's path to private yet verifiable on-chain vote counts
- **Commit-reveal schemes** — Hash-then-reveal to prevent vote buying and coercion
- **Sybil-resistant voting** — Implementation guide addressing the foundational one-person-one-vote problem
- **DAAC (Decentralized Autonomous Art Collective)** — Governance applied to cultural/creative communities, not just DeFi

## 4. Issue Search — Problems with Decentralized Governance

223 total results; the most substantive debates:

| Issue | Repo | Comments | Core Problem |
|---|---|---|---|
| **#519** | gnolang/gno | 6 | Three-body governance design (Evaluation DAO + Decentralists DAO + Chain Gov); "skin in the game" lost after Interchain Security; vote distribution is an unsolved political problem |
| **#270** | helium/HIP | 257 | Governance as political battlefield — 257 comments on revoking a manufacturer's approval; enforcement harder than enactment; closed as `invalid`/`stale` without resolution |
| **#132** | stacksgov/pm | 128 | Code of Conduct debate — freedom of speech vs. enforcement scope; "Can't Be Evil" ethos vs. practical moderation; process stalled since Feb 2021 |
| **#2** | Fushuma/FIP | 19 | Treasury DAO mechanism — who gets to vote? How is power allocated? Whale capture prevention; economic assumptions as contested as code |
| **#2** | GamaEdtech/solana-governance-program | 5 | On-chain vs. off-chain vs. hybrid governance; staking vs. quadratic voting; still open since Jan 2025 — no convergence |
| **#760** | filecoin-project/community | — | Skrynka masternode whitepaper: random-beacon quorum selection; Sybil-deduplication attack; quorum-held per-replica encryption keys; bounded decay at ~1.3×10⁻³ |
| **#187** | code-423n4/2021-04-vader-findings | — | Flash-loan governance attack — temporary balance inflation to dominate votes; past-block balance checkpoints as mitigation |
| **#76** | Web3-Risk-Logic-Analysis | — | Single Boolean operator (OR vs AND) can hollow out quorum protections — "a single logical error is a constitutional crisis" |

## Key Insight

The hardest problems in digital democracy are **not cryptographic but political**: quorum thresholds, power distribution, decision enforcement, and privacy-transparency reconciliation have no technical fix. The path forward requires better institutions, not just better cryptography.

See `essay-outline.md` for the full structured essay with code examples, architecture diagrams, and detailed reference links.