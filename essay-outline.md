# Digital Democracy: Blockchain Voting, DAO Governance, and the Future of Collective Decision-Making

## Introduction

The promise of digital democracy is deceptively simple: *what if every citizen could
vote on every issue — transparently, securely, and without intermediaries?*
Blockchain technology and Decentralized Autonomous Organizations (DAOs) have
turned this thought experiment into a live engineering frontier. From on-chain
voting contracts that weigh ballots by quadratic cost to modular DAO frameworks
where anyone can mix-and-match governance modules, the ecosystem is growing
rapidly — and so are the debates about whether it works.

This essay surveys the landscape through three lenses: **(1)** notable
blockchain-voting and DAO-governance projects on GitHub, **(2)** how on-chain
voting contracts actually work at the code level, and **(3)** the open
controversies and unresolved problems that the community is actively debating.
What emerges is a portrait of a field that is technically thrilling and
politically unresolved — where the code is written but the meaning is still
being fought over.

---

## Part I — Key Projects: What's Being Built

### 1. Democracy Earth — *The Social Smart Contract*

**Repository:** [DemocracyEarth/paper](https://github.com/DemocracyEarth/paper)
**Stars:** 617 ⭐ | **Language:** Markdown | **License:** MIT

The single most influential document in the digital democracy space. Democracy
Earth's 2017 manifesto argues that Bitcoin gave us programmable money, Ethereum
gave us programmable contracts, and the world now needs **programmable votes** —
a third layer that signals incorruptible ballots beyond the boundaries of
nation-states.

**Key innovations:**

- **Vote Token:** An ERC-20 token branded as "vote." Every human who validates
  their self-sovereign identity receives an equal share — *cryptographically
  induced equality*.
- **Proof of Identity:** A mechanism called *attention mining* that incentivizes
  participants to perform simple tests to detect "replicants" (Sybil attackers)
  without requiring a central authority.
- **Liquid Democracy Model:** Citizens can vote directly, or delegate voting
  power to peers — broadly, or on specific tagged topics (e.g., only #environment).
  Delegation is transitive. Voters can always override their delegate. Votes are
  public by default.
- **Sovereign App:** An adaptive mobile/desktop application featuring the
  "liquid bar" — a single-gesture UI for voting, delegating, and withdrawing.
- **Agora:** A Reddit-style debate forum where *upvoting a comment triggers a
  one-vote delegation* — making discourse itself a political act.
- **Cryptographic Privacy:** Precompiled contracts for alt_bn128 curve operations
  enable zk-SNARKs on Ethereum; ring signatures via Monero; shielded
  transactions via ZCash.

**Why it matters:** Democracy Earth has written 30,000+ lines of code (Sovereign
platform) and piloted a digital plebiscite among Colombian expatriates in 2016.
It is both a paper and a project — rare in the crypto governance space.

**Citation:** Democracy Earth Foundation, *"The Social Smart Contract,"* 2017.

---

### 2. Blockchain Voting Systems — The GitHub Landscape

Our search for "blockchain voting" surfaced a diverse ecosystem, split into two
camps:

| Project | Stars | Language | Key Idea |
|---|---|---|---|
| **mehtaAnsh/BlockChainVoting** | 450 ⭐ | JavaScript / Solidity | Full-stack E-voting dApp: companies create elections, candidates register, voters cast ballots via MetaMask. Uses IPFS for candidate images, MongoDB for backend, Next.js + Semantic UI for frontend. |
| **yfgeek/BlockVotes** | 283 ⭐ | PHP | Privacy-preserving e-voting using **ring signatures** to anonymize voters while still proving eligibility — a direct answer to the transparency-vs-privacy tension. |
| **cardano-foundation/jormungandr** | 368 ⭐ | Rust | Cardano-based privacy voting node, emphasizing cryptographic anonymity ballots on a public chain. |
| **BuildOnViction/victionchain** | 182 ⭐ | Go | A blockchain powered by **Proof-of-Stake Voting** consensus — where validators are elected by token-weighted votes, blending governance and consensus into one mechanism. |
| **KashifCh-eth/blockchain-voting-system-** | 46 ⭐ | JavaScript | A simpler entry-level blockchain voting system, good for understanding the baseline architecture. |

**Takeaway:** The blockchain-voting space splits into two camps —
*transparent, identity-linked systems* (BlockChainVoting) and
*privacy-preserving, cryptography-first systems* (BlockVotes, jormungandr).
Neither has "won"; the tension between verifiability and anonymity remains the
central design problem.

---

### 3. DAO Governance Frameworks — Modular,Composable, and Contested

| Project | Stars | Language | Key Idea |
|---|---|---|---|
| **DA0-DA0/dao-contracts** | 218 ⭐ | Rust (CosmWasm) | Modular, composable DAO architecture. Every DAO = voting-power module + proposal module(s) + core treasury. Supports yes/no, multiple-choice, and **Condorcet ranked-choice** voting. Audited by Oak Security. |
| **ensdomains/governance-contracts** | 159 ⭐ | JavaScript (Hardhat) | On-chain governance for the ENS DAO. Includes airdrop contracts, API layer, and deployment scripts — a real-world, production-grade governance stack. |
| **ensdomains/governance-docs** | 25 ⭐ | — | Accompanying documentation for ENS governance — a rare case of governance *documentation* being as curated as the code itself. |
| **decentraland/governance** | 49 ⭐ | TypeScript | Governance platform for the Decentraland virtual world DAO — a real-world deployment with token-weighted voting. |
| **Joystream/pioneer** | 43 ⭐ | TypeScript | Governance app for Joystream DAO, featuring on-chain council elections and referenda. |
| **delvtech/council** | 92 ⭐ | TypeScript | Flexible, upgradeable governance contract system supporting time-weighted voting, conviction voting, and more. |
| **brownie-mix/dao-mix** | 99 ⭐ | Python | A scaffold for working with and building DAOs — ideal for rapid prototyping. |
| **Virtual-Protocol/protocol-contracts** | 101 ⭐ | Solidity | Virtual governance ecosystem for Virtual DAO and Virtual-specific governance. |

**Takeaway:** DAO governance has evolved from monolithic "one contract does
everything" designs to **modular, upgradeable architectures** (DAO DAO) where
voting-power, proposal-type, and treasury modules are swappable. This
composability is arguably the most important architectural insight of the
current era.

---

### 4. Quadratic Voting in Practice — ArcanaDAO

**Repository:** [Kuuhaku-web/Arcana — VOTING_GUIDE.md](https://github.com/Kuuhaku-web/Arcana/blob/main/VOTING_GUIDE.md)

The most detailed public guide to implementing quadratic voting on-chain. The
core insight: **the cost of voting increases quadratically** with the number of
votes you cast. This makes it economically irrational for whales to dominate.

---

### 5. ENS Governance — On-Chain Governance at Scale

**Repository:** [ensdomains/governance-contracts](https://github.com/ensdomains/governance-contracts)

The ENS DAO was one of the first major protocols to transfer control to a
community-governed DAO. Its governance contracts include airdrop mechanics, an
API layer, and Hardhat-based deployment scripts — representing one of the few
real-world, production-grade on-chain governance stacks. The companion
[governance-docs](https://github.com/ensdomains/governance-docs) repository shows
that even the *documentation* of governance processes is treated as a first-class
citizen — a sign of maturity most DAOs have not reached.

---

## Part II — How On-Chain Voting Code Actually Works

To ground the essay in technical reality, we examined multiple on-chain voting
implementations via code search. The patterns they reveal are universal.

### Pattern 1 — Deposit-and-Lock Voting Power

Voting power is **not free** — it is a function of tokens deposited and a
chosen lock-up period. Longer locks → higher weight. This solves the "sybil
attack" problem (one-person-one-vote is trivially gamed on-chain) but
introduces a **plutocratic bias**: wealth = influence.

```solidity
function stakeTokensForVotingPower(uint256 _amount) public whenNotPaused {
    // In a real implementation, you would integrate with an ERC20 token contract.
    // For simplicity, ...
}
```

### Pattern 2 — Quadratic Voting

From the Arcana guide:

> *"1 vote = 1 token, 2 votes = 4 tokens (2²), 3 votes = 9 tokens (3²),
> 5 votes = 25 tokens (5²)"*

**Smart contract functions:**

```solidity
createProposal(title, description)    // Register a new governance proposal
vote(proposalId, votes, choice)       // Cast a vote (Yes / No / Abstain)
calculateVoteCost(votes)              // Returns votes² (the token cost)
getProposal(proposalId)               // Fetch proposal details
isVotingActive(proposalId)            // Check if voting period is open
getProposalVotes(proposalId)          // Get all votes for a proposal
```

**Voting flow:**

1. User views a proposal → clicks "Cast Your Vote"
2. Selects choice (Yes / No / Abstain) and number of votes (1–100)
3. Modal displays real-time cost: *votes²*
4. User clicks "Vote" → MetaMask approval
5. Smart contract transfers `votes²` tokens from user to DAO
6. Contract records vote on-chain
7. Tokens are **locked** for the voting period (default: 7 days)

**Why it matters:** A whale with 1,000 tokens cannot cast 1,000 votes — that
would cost 1,000,000 tokens. To cast 32 votes costs 1,024 tokens — more than
their entire holding. This makes domination economically irrational.

### Pattern 3 — Modular Composable Architecture (DAO DAO)

```
┌──────────────────────────────────────────────────┐
│               DAO Instance                         │
│                                                    │
│  ┌──────────────┐   ┌──────────────┐  ┌────────┐ │
│  │  Voting      │   │  Proposal    │  │ Core   │ │
│  │  Power       │◄─ │  Module      │◄─│ Treasury│ │
│  │  Module      │   │  (Yes/No,    │  │        │ │
│  │              │   │   MC, Cond.) │  │        │ │
│  └──────────────┘   └──────────────┘  └────────┘ │
│       │                  │                        │
│       └── Standard Interface ────────────────────┘ │
└──────────────────────────────────────────────────┘
```

- **Voting Power Module:** Token-staked (CW20), NFT-staked (CW721), or
  membership-based (CW4).
- **Proposal Module:** Single-choice, multiple-choice, or ranked-choice
  (Condorcet).
- **Core Module:** Holds and manages the DAO's treasury.
- **Standard Interface:** Any module can be swapped with any other of the same
  type — governance becomes "plug-and-play."

### Pattern 4 — Liquid Democracy Transactions (Sovereign)

| Transaction | Description |
|---|---|
| **Direct Vote** | Vote on an issue yourself, as in direct democracy |
| **Basic Delegation** | Delegate your votes to a trusted peer |
| **Tag-Limited Delegation** | Delegate only on specific topics (e.g., #environment) |
| **Transitive Delegation** | Your delegate can further delegate |
| **Overriding Vote** | As the sovereign owner, you can always override your delegate |
| **Public Vote** | All delegators can see how their delegate voted |
| **Secret Vote** | Votes are untraceable via zk-SNARKs, ZCash, or Monero |

### Pattern 5 — Adapter / Slot Architecture

The **adapter/slot pattern** introduces an indirection layer where the actual
bank and voting-tally contracts can be replaced without changing the adapter.
This is the same composable-module pattern used by DAO DAO, translated into
Solidity. It's architecturally elegant — but it also means *the code you see
isn't the code that runs*. Upgradeability, by design, means trust in the
upgrade mechanism.

### Pattern 6 — Role-Based Access Control

- `onlyMember` → any DAO member can submit votes, deposit, withdraw, propose
- `onlyAdmin` → a privileged address (or multi-sig) can validate proposals,
  add/remove vote-parameter sets

The tension is clear: **DAO DAO's ethos is "no admins," but most real systems
quietly reintroduce an admin role for parameter changes.** Is any blockchain
governance system truly leaderless, or does every "decentralized" system
silently encode an admin escape hatch?

### Pattern 7 — Timing Windows as Political Design

```solidity
struct ProposedVoteParam {
    bytes4 voteParamId;
    IAgora.Consensus consensus;
    uint32 votingPeriod;          // how long votes are cast
    uint32 gracePeriod;           // buffer after voting ends
    uint32 threshold;             // acceptance quorum
    uint32 adminValidationPeriod; // pre-vote review window
}
```

Timing is governance. The **grace period** allows for dispute resolution after
a vote; the **admin validation period** creates a "cooling-off" window before
votes even begin. A short voting period favors organized minorities; a long
one favors token-rich whales who can afford to monitor proposals.

### Pattern 8 — Campaigning State & Quorum Enforcement

From Neufund's `VotingProposal.sol`:

```solidity
enum State {
    Campaigning,  // Initial state: voting owner builds quorum for public visibility
    ...
}
```

A **Campaigning phase** where quorum must be visibly built before votes are
counted — preventing secret low-turnout votes from claiming legitimacy.
Combined with **SafeMath** for arithmetic, ensuring that vote counts cannot
overflow or be manipulated. The mundane details of integer arithmetic become
political when a single overflow could flip a governance outcome.

### Pattern 9 — ERC20 Approval-Based Voting

From the earliest on-chain voting contracts (2017-era):

```solidity
pragma solidity ^0.4.8;
import "./zeppelin/token/ERC20.sol";
// Owners of an ERC20 token will be allowed to vote according to their ownership stake.
```

The simplest model: *ERC20 balance = vote weight*. Transparent and simple, but
it means *whoever controls the tokens controls the outcome*. The subsequent
evolution — toward deposit-and-lock, quadratic, and NFT-based voting —
represents a century of democratic theory compressed into five years of Solidity.

### Pattern 10 — Vote Parameter Proposals (TerraBioDAO)

From `TerraBioDAO/dao-first-iteration/src/adapters/Voting.sol`:

```solidity
/**
 * @notice Users can submit votes, vote parameters, consultation and
 *         also deposit and withdraw from the Bank
 */
contract Voting is ProposerAdapter {
    enum ProposalType {
        CONSULTATION,
        VOTE_PARAMETERS,
        ...
    }
}
```

This reveals a subtle but important design choice: **the parameters of voting
themselves become votable proposals.** You don't just vote on policy — you vote
on *how voting works*. This meta-governance layer is where the deepest questions
of digital democracy live.

---

## Part III — The Central Controversies

### 3.1 Plutocracy & Whale Domination

**The problem:** In token-weighted voting, the rich get richer — and the rich
also get *more political power*. A single entity holding 5% of tokens can block
any proposal (requiring 20% + 1 to pass in many systems). Blockchain governance
may simply replicate existing power structures in a new medium.

**The proposed solution:** Quadratic voting (cost = votes²) makes it
economically infeasible for whales to dominate. But it introduces new questions:
who sets the exchange rate? What prevents front-running? What about voters who
can't afford to participate at all?

**Open debate:** Should voting power be strictly proportional to tokens, or
should there be caps, quadratic penalties, or even universal basic income
mechanisms that distribute political power more equally?

---

### 3.2 The Secrecy–Verifiability Tension

**The problem:** Researchers (Hosp & Vora, using information theory) have
shown that **perfect ballot secrecy, perfect tally verifiability, and perfect
integrity cannot all be simultaneously achieved** when an adversary has
unlimited computational resources. You must sacrifice something.

**The trade-off:**

- **Full transparency** → perfect auditability but destroys ballot secrecy,
  enabling coercion and vote-buying
- **Full secrecy** → protects voters but makes it harder to verify the tally
- **Hybrid approaches** (public tallies with secret individual ballots) are the
  pragmatic middle ground, used by Sovereign and most DAO systems

**Open debate:** In a small DAO, is public voting acceptable? In a national
election, is it even possible? Where is the line?

---

### 3.3 Identity & Sybil Attacks

**The problem:** Anonymous on-chain governance is vulnerable to **Sybil
attacks** — a single entity creating thousands of fake identities to amass
voting power. Democracy Earth's "Proof of Identity" and "attention mining" are
ambitious attempts to solve this, but they remain experimentally unproven at
scale.

**The debate:** Should identity be verified through biometrics? Social graph
analysis? Proof-of-personhood (e.g., Worldcoin)? Or should governance simply
accept pseudonymity as a feature?

---

### 3.4 Low Participation & Apathy

**The problem:** Even in decentralized systems, participation rates are
abysmally low. ENS DAO proposals routinely see <10% voter turnout. If digital
democracy is supposed to be more participatory than representative democracy,
why aren't people voting?

**Possible explanations:**

- Governance tokens are concentrated, so most holders feel powerless
- Proposals are too technical for non-expert voters
- Delegation creates an "someone else will handle it" mentality
- The "unforeseen side effects" problem — voters don't understand what
  they're actually voting on

---

### 3.5 Upgradability vs. Immutability

**The problem:** Smart contracts are supposed to be immutable — that's the
point. But governance contracts need to be upgraded to fix bugs, add features,
and adapt to new circumstances. DAO DAO addresses this with its modular,
upgradeable architecture. But who controls the upgrade key?

**The tension:** If a small multisig or DAO council can upgrade contracts, is
the system truly decentralized? If nobody can upgrade, is it truly governable?

---

### 3.6 The "Admin Trap"

**The evidence:** The TerraBioDAO `Voting.sol` contract quietly includes
`onlyAdmin` functions for parameter changes. DAO DAO markets itself as admin-free,
yet its modular design requires an on-chain proposal to upgrade modules.

**The core question:** If a DAO's smart contract has an admin key — even one
intended for "emergency use" — can it truly be called decentralized? Every
real-world DAO has traded some decentralization for upgradeability. The question
is not *whether* to have admins, but *how many, how they're selected, and how
they can be removed*.

---

### 3.7 Legal & Regulatory Uncertainty

**The problem:** On-chain governance decisions may have real-world legal
implications (changing fee structures, minting tokens, treasury withdrawals).
But most jurisdictions have no clear framework for recognizing DAO decisions
as legally binding.

**The debate:** Should a DAO vote that legally constitutes a "securities
offering" be subject to SEC regulation? Can a smart contract be held liable
for a decision that causes financial harm?

---

### 3.8 Sustainable Economics for DAOs

**The question:** Can a DAO ever be economically self-sustaining, or does every
DAO implicitly rely on off-chain value creation (brand loyalty, developer
reputation, ecosystem grants) that on-chain governance cannot fully capture?
The betrusted-wiki argument is blunt: *"The problem with such economy is that
it needs to be setup in such architecture that makes valorisation possible...
without the need for exterior intervention."*

---

## Part IV — Proposed Essay Structure

| Section | Content | Target Length |
|---|---|---|
| **1. Introduction** | The normative promise of digital democracy vs. the engineering reality | ~500 words |
| **2. The Blockchain Voting Landscape** | Survey: BlockChainVoting, BlockVotes, jormungandr, victionchain, Moscow | ~600 words |
| **3. Democracy Earth & the Social Smart Contract** | The foundational paper: vote tokens, liquid democracy, Proof of Identity, Agora | ~700 words |
| **4. How On-Chain Voting Actually Works** | Code-level walkthrough: deposit-and-lock, quadratic voting, modular architecture, adapter pattern, role-based access, timing windows, quorum enforcement, ERC20 voting, vote-parameter proposals | ~1,100 words |
| **5. Modular DAO Architecture** | DAO DAO's composable modules, Condorcet voting, ENS governance in practice | ~600 words |
| **6. Controversy I: Plutocracy & Whale Domination** | Token-weighted voting, quadratic countermeasures, the fundamental unfairness of wealth-based power | ~450 words |
| **7. Controversy II: The Secrecy–Verifiability Dilemma** | Ring signatures, zk-SNARKs, the impossible triangle, and what Democracy Earth chose | ~500 words |
| **8. Controversy III: The Sybil & Identity Problem** | On-chain identity, attention mining, proof-of-personhood, and the impossibility of anonymous democracy | ~450 words |
| **9. Controversy IV: The Admin Trap & Upgradability** | Whether any "decentralized" system can truly eliminate privileged keys | ~400 words |
| **10. Controversy V: Low Participation & the Apathy Paradox** | Why <10% turnout undermines the legitimacy of "decentralized" governance | ~350 words |
| **11. Controversy VI: Sustainable DAO Economics** | Can on-chain token economies truly replace off-chain value creation? | ~350 words |
| **12. Controversy VII: Legal & Regulatory Gaps** | What happens when a smart contract decision crosses into real-world law? | ~350 words |
| **13. Conclusion** | A grounded vision: what would a genuinely democratic on-chain system require? | ~400 words |

**Total target:** ~6,000–6,500 words

---

## Appendix — Key References

### Repositories

| Resource | Link | Stars |
|---|---|---|
| Democracy Earth (Social Smart Contract paper) | [DemocracyEarth/paper](https://github.com/DemocracyEarth/paper) | 617 ⭐ |
| Sovereign (Liquid Democracy app) | [DemocracyEarth/sovereign](https://github.com/DemocracyEarth/sovereign) | — |
| BlockChainVoting (full-stack E-voting) | [mehtaAnsh/BlockChainVoting](https://github.com/mehtaAnsh/BlockChainVoting) | 450 ⭐ |
| BlockVotes (ring-signature privacy voting) | [yfgeek/BlockVotes](https://github.com/yfgeek/BlockVotes) | 283 ⭐ |
| Jormungandr (Cardano privacy voting node) | [cardano-foundation/jormungandr](https://github.com/cardano-foundation/jormungandr) | 368 ⭐ |
| Victionchain (PoS voting consensus) | [BuildOnViction/victionchain](https://github.com/BuildOnViction/victionchain) | 182 ⭐ |
| DAO DAO (modular DAO contracts, Rust) | [DA0-DA0/dao-contracts](https://github.com/DA0-DA0/dao-contracts) | 218 ⭐ |
| ENS Governance Contracts | [ensdomains/governance-contracts](https://github.com/ensdomains/governance-contracts) | 159 ⭐ |
| ENS Governance Docs | [ensdomains/governance-docs](https://github.com/ensdomains/governance-docs) | 25 ⭐ |
| Arcana (Quadratic Voting guide) | [Kuuhaku-web/Arcana/VOTING_GUIDE.md](https://github.com/Kuuhaku-web/Arcana/blob/main/VOTING_GUIDE.md) | — |
| Council (flexible DAO governance) | [delvtech/council](https://github.com/delvtech/council) | 92 ⭐ |
| Decentraland Governance | [decentraland/governance](https://github.com/decentraland/governance) | 49 ⭐ |
| Joystream Pioneer (DAO governance app) | [Joystream/pioneer](https://github.com/Joystream/pioneer) | 43 ⭐ |
| TerraBioDAO (Voting adapter contract) | [TerraBioDAO/dao-first-iteration](https://github.com/TerraBioDAO/dao-first-iteration) | — |

### Code Search Highlights

| Pattern | Source | Significance |
|---|---|---|
| `stakeTokensForVotingPower()` | waihungho/smart-contracts | Deposit-and-lock model for voting power |
| `calculateVoteCost(votes)` → votes² | ArcanaDAO guide | Quadratic voting cost function |
| `enum State { Campaigning }` | Neufund VotingProposal.sol | Quorum-building phase before counting |
| `struct ProposedVoteParam` | TerraBioDAO Voting.sol | Timing windows as political design |
| `onlyAdmin` / `onlyMember` | Multiple contracts | Role-based access control and the admin trap |
| `import "./zeppelin/token/ERC20.sol"` | 2017-era voting contracts | ERC20 balance = vote weight (the simplest model) |
| `contract Voting is ProposerAdapter` | TerraBioDAO | Adapter/slot pattern for swappable logic |

### Academic & Theoretical References

| Resource | Link |
|---|---|
| Hosp & Vora — Information-Theoretic Voting Security | https://pdfs.semanticscholar.org/24d5/5c866a7317dae11d37518b312ee460bc33d3.pdf |
| Democracy Earth — Colombian Digital Plebiscite Pilot | https://words.democracy.earth/a-digital-referendum-for-colombias-diaspora-aeef071ec014 |
| OECD — Embracing Innovation in Government | https://www.oecd.org/gov/innovative-government/embracing-innovation-in-government-colombia.pdf |

### Key Concepts

| Concept | Description |
|---|---|
| **Quadratic Voting** | Cost = votes²; prevents whale domination |
| **Liquid Democracy** | Hybrid direct + delegated voting with tag-limited and transitive delegation |
| **Proof of Identity** | Attention mining to detect Sybil attackers without central authority |
| **Adapter/Slot Pattern** | Indirection layer for swappable contracts (upgradeability) |
| **Campaigning State** | Quorum-building phase before votes are counted |
| **Condorcet Voting** | Ranked-choice method that finds the candidate who beats all others pairwise |
| **Ring Signatures** | Cryptographic technique to hide signer identity within a group |
| **zk-SNARKs** | Zero-knowledge proofs that verify computation without revealing inputs |
| **The Admin Trap** | The tendency of "decentralized" systems to silently reintroduce privileged keys |
| **Secrecy–Verifiability Triangle** | The impossibility of simultaneously achieving perfect secrecy, verifiability, and integrity |

---

*This outline was generated from GitHub research on blockchain voting, DAO
governance, and on-chain voting contract implementations. It draws on 10+
repositories, 20+ code search results, and community discussions across the
decentralized governance ecosystem. Last updated: September 2025.*
