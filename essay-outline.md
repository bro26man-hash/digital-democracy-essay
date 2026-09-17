# Digital Democracy: Blockchain Voting, DAO Governance, and On-Chain Decision-Making

## Introduction

Digital democracy promises to extend democratic participation beyond the nation-state and the polling place, letting communities make collective decisions through code. Two movements drive that vision: **blockchain-based voting systems**, which aim to make elections tamper-resistant and auditable, and **Decentralized Autonomous Organizations (DAOs)**, which turn governance into a continuous, on-chain process of proposals and votes. Blockchain's tamper-evident append-only logs and Turing-complete smart contracts seemed to offer the perfect infrastructure — and indeed, a vibrant ecosystem of projects has emerged around on-chain voting and DAO governance.

This essay surveys the notable projects building these systems, explains how on-chain voting contracts actually work under the hood, and maps the core controversies — many still unresolved — that researchers and practitioners are debating in open-source repositories and governance forums today.

---

## Part I — Key Projects in Blockchain Voting & DAO Governance

### 1. General-Purpose Blockchain Voting Apps

These are turnkey E-voting systems that record votes as immutable blockchain transactions:

| Project | Language | Stars | Notes |
|---|---|---|---|
| [mehtaAnsh/BlockChainVoting](https://github.com/mehtaAnsh/BlockChainVoting) | JavaScript | 450 | Most-starred general-purpose E-voting project; Next.js + Solidity + IPFS + MongoDB |
| [yfgeek/BlockVotes](https://github.com/yfgeek/BlockVotes) | PHP | 283 | E-voting using **ring signatures** for ballot secrecy |
| [KashifCh-eth/blockchain-voting-system-](https://github.com/KashifCh-eth/blockchain-voting-system-) | JavaScript | 46 | Standard blockchain voting dApp |

**BlockChainVoting** is the most-starred general-purpose E-voting project on GitHub. Built as a final-year polytechnic project, it uses Solidity/Web3 for the blockchain contract, Next.js + Semantic UI React for the front-end, MongoDB/ExpressJS/Node.js for the back-end, and IPFS for image storage. The workflow: an admin creates an election, adds candidates and voters, voters receive secure credentials via email, and votes are recorded on-chain with success/failure notifications. It illustrates the basic architecture of most blockchain voting dApps: an off-chain server manages identity and email, while the on-chain contract records votes.

**BlockVotes** distinguishes itself by using **ring signatures** to hide the voter's identity within a group of possible signers, providing ballot secrecy — a feature most simpler E-voting projects lack.

### 2. Privacy-Focused & Consensus-Level Voting

| Project | Language | Stars | Notes |
|---|---|---|---|
| [cardano-foundation/jormungandr](https://github.com/cardano-foundation/jormungandr) | Rust | 368 | Cardano Shelley node with **privacy-preserving voting**; formal verification to reduce election-software bugs |
| [BuildOnViction/victionchain](https://github.com/BuildOnViction/victionchain) | Go | 182 | Consensus *is* PoS voting — validators are elected by token-holder votes |

**Jormungandr** represents the privacy-first philosophy: Cardano's research-driven approach applies formal methods and zero-knowledge techniques to voting, treating election software as a security-critical system requiring mathematical guarantees rather than just audits. **Viction** takes a different approach — voting is embedded directly into the consensus layer. Validators are elected by token-holder votes, so the blockchain's block production *is* a voting process. This is the simplest possible model: voting *is* block production, but it's limited to validator election, not general referenda.

### 3. Identity-Based DAO Governance

| Project | Language | Stars | Notes |
|---|---|---|---|
| [ensdomains/governance-contracts](https://github.com/ensdomains/governance-contracts) | JavaScript | 159 | Token-weighted voting with delegation for the ENS DAO — real-world reference |
| [decentraland/governance](https://github.com/decentraland/governance) | TypeScript | 49 | Governance platform for the Decentraland DAO; virtual-world scale |
| [Joystream/pioneer](https://github.com/Joystream/pioneer) | TypeScript | 43 | Governance app for Joystream DAO |

**ENS** is a critical real-world reference: it has actually undergone governance crises, including the controversial transformation from DAO to multisig, and its contracts have been battle-tested. **Decentraland** governs a virtual world with millions of dollars in LAND tokens, illustrating how governance scales to complex, continuously evolving systems. **Joystream** adds a substrate-based (Rust/Polkadot) perspective, showing that DAO tooling isn't exclusively Ethereum.

### 4. Modular & Composable DAO Tooling

| Project | Language | Stars | Notes |
|---|---|---|---|
| [DA0-DA0/dao-contracts](https://github.com/DA0-DA0/dao-contracts) | Rust/WASM | 217 | Three-module architecture: voting-power, proposal, core-treasury — composable and upgradable |

The DA0-DAO project is the most architecturally ambitious project in this survey. Its core insight is that a DAO should be composed of **interchangeable modules**, not a monolithic contract:

1. **Voting power module** — manages how voting power is calculated (staked governance tokens, staked NFTs, membership credential, reputation score)
2. **Proposal module** — manages the proposal lifecycle (yes/no, multiple-choice, ranked-choice/Condorcet)
3. **Core module** — holds the DAO treasury

Each module type has a **standard trait interface** in Rust. As a result, any voting module can be used with any proposal module, and any proposal module with any voting module. This composability is a genuine innovation: it lets a DAO swap from token-weighted voting to quadratic voting without changing its proposal logic, or upgrade from yes/no to ranked-choice without touching the voting power module.

Open issues on that repo debate **quadratic voting** implementation, **sybil-proof stake delegation**, and how to handle delegation edge cases when voters change their delegation mid-proposal.

### 5. Production-Grade Governance References

- **Celo Governance.sol** (inside [celo-org/celo-monorepo](https://github.com/celo-org/celo-monorepo), ~805★) — ~1,700-line reference implementation for on-chain governance proposals, deployed on mainnet and audited. Features checkpoint-based voting power, timelocks, and ReentrancyGuard patterns.
- **TerraBioDAO Voting.sol** ([TerraBioDAO/dao-first-iteration](https://github.com/TerraBioDAO/dao-first-iteration)) — Modular adapter pattern separating Voting, Proposer, and Agora concerns; vote weight derived from token deposit + lock period via a Bank contract. Illustrates how DAO frameworks can compose specialized Modules (Bank, Agora, Voting) through slot-based addressing.

---

## Part II — How On-Chain Voting Code Actually Works

### Architectural Patterns

Reading across these repositories and their `Voting.sol` / `Governance.sol` / `voting.rs` files, three patterns dominate:

#### Pattern A: Native Protocol Voting (Jormungandr, Viction)

Vote recording is built into the **consensus layer** itself. Validators stake tokens and votes are signed staking-key transactions. Simplest model — voting *is* block production — but limited to validator election, not general referenda.

#### Pattern B: Smart Contract Voting (Celo, DA0-DAO, ENS, TerraBioDAO)

The dominant pattern for general-purpose governance. A smart contract stores:

1. **Voter registry** — eligibility tied to token balances, NFT ownership, or reputation.
2. **Proposal state machine** — draft → discussion → voting → timelock → execution.
3. **Vote tallying** — weighted by token balance, quadratic weight, or reputation; tallied on-chain.

**From Celo `Governance.sol`** (~1,700 lines, audited, mainnet-deployed):

```solidity
enum VoteValue { None, Abstain, No, Yes }
struct UpvoteRecord { uint256 proposalId; uint256 weight; }
struct VoteRecord  { uint256 proposalId; uint256 yesVotes; uint256 noVotes; uint256 abstainVotes; }
struct Voter       { UpvoteRecord upvote; uint256 mostRecentReferendumProposal; mapping(uint256 => VoteRecord) referendumVotes; }

contract Governance is IGovernance, Ownable, Initializable, ReentrancyGuard, UsingRegistry {
    // Checkpoint-based voting power: power = balanceAt(blockNumber)
    // Timelock: proposals must wait before execution
    // ReentrancyGuard: prevents reentrancy attacks on vote casting

    function propose(...) external payable returns (uint256) { ... }     // Deposit-gated proposal creation
    function upvote(uint256 proposalId, uint256 lesser, uint256 greater) external nonReentrant returns (bool) { ... }
    function vote(uint256 proposalId, uint256 index, VoteValue value) external nonReentrant returns (bool) { ... }
    function votePartially(...) external nonReentrant returns (bool) { ... }  // Split yes/no/abstain
    function revokeVotes() external nonReentrant returns (bool) { ... }
    function execute(uint256 proposalId, uint256 index) external nonReentrant returns (bool) { ... }
}
```

Key mechanics revealed by reading the full contract:

- **Checkpoint-based voting power** — derived from token balance at a specific block number, preventing mid-vote manipulation (e.g., flash-loan-acquired tokens can't be used if the checkpoint was taken earlier)
- **Three-stage proposal lifecycle** — **Queue** (upvote to prioritize) → **Referendum** (cast yes/no/abstain votes) → **Execution** (timelock expires, then execute). Each stage has independent clocks and expiration conditions.
- **Upvote queue ordering** — proposals in the queue are ordered by upvote weight using an `IntegerSortedLinkedList`, so the most-supported proposals get dequeued first when the time comes. Think of it as a continuous priority lane.
- **Deposit-gated proposal creation** — `propose()` requires `msg.value >= minDeposit`, with the deposit refunded upon dequeuing. This suppresses spam.
- **Vote revocation** — voters can revoke their upvote (queue stage) or referendum votes (referendum stage), and the contract updates totals accordingly. `revokeVotes()` iterates all dequeued proposals for the sender.
- **Participation-based quorum** — `_isProposalPassing()` checks not just yes-vs-no ratio but also a **participation baseline**: the proportion of total locked gold that voted. If participation drops below the baseline quorum factor, the proposal fails even with a yes majority. This prevents a small quorum from passing sweeping changes.
- **Constitution-specific thresholds** — different proposal *destinations* (addresses) and *function IDs* can have different passing thresholds via `setConstitution()`. Default is simple majority; specific functions (e.g., treasury transfers) can require supermajority.
- **ReentrancyGuard on every state-changing function** — standard DeFi security pattern applied to democratic processes.
- **Hotfix mechanism** — a privileged path for emergency upgrades: `approveHotfix` → `prepareHotfix` → `executeHotfix`, requiring both an approver AND a security council, with a time-window expiry. The hotfix is identified by a keccak256 hash of the transaction blob + salt.

**From TerraBioDAO `Voting.sol`** (adapter-style modular design):

```solidity
contract Voting is ProposerAdapter {
    enum ProposalType { CONSULTATION, VOTE_PARAMS }

    struct Consultation       { string title; string description; address initiater; }
    struct ProposedVoteParam  { bytes4 voteParamId; IAgora.Consensus consensus; uint32 votingPeriod; uint32 gracePeriod; uint32 threshold; uint32 adminValidationPeriod; }
    struct VotingProposal     { ProposalType proposalType; Consultation consultation; ProposedVoteParam voteParam; }

    mapping(bytes28 => VotingProposal) private _votingProposals;

    function submitVote(bytes32 proposalId, uint256 value, uint96 deposit, uint32 lockPeriod, uint96 advancedDeposit)
        external onlyMember
    {
        uint96 voteWeight = IBank(_slotAddress(Slot.BANK)).newCommitment(
            msg.sender, proposalId, deposit, lockPeriod, advancedDeposit);
        IAgora(_slotAddress(Slot.AGORA)).submitVote(proposalId, msg.sender, uint128(voteWeight), value);
    }
}
```

TerraBioDAO's design separates **Voting** from **Agora** (the tallying engine) and **Bank** (the deposit/lock module) via a slot-based addressing scheme. Vote weight is computed by the Bank — combining token deposit, lock period, and advanced deposits — then submitted to Agora for tallying. This is a clean example of **separation of concerns**: the Voting module handles proposal submission and vote casting, the Bank handles economic commitments, and Agora handles consensus logic. The `_executeProposal()` override shows how a passed VOTE_PARAMS proposal auto-applies new voting parameters, while a passed CONSULTATION proposal does nothing on-chain (it's advisory only).

**From DA0-DAO (Rust/CosmWasm), `packages/dao-voting/src/voting.rs`:**

```rust
pub trait Voting {
    fn vote(&mut self, ctx: &Ctx, proposal_id: u64, addr: String, vote: Vote) -> Result<()>;
}

pub fn get_voting_power_with_delegation(
    deps: Deps, address: &Addr, dao: &Addr,
    proposal_id: u64, proposal_height: u64,
) -> StdResult<VotingPowerWithDelegation> {
    // 1. Get individual voting power from the voting module
    let individual = get_voting_power(deps, address.clone(), dao, Some(proposal_height))?;
    // 2. Get delegated (unvoted) power from other members
    let udvp = delegation_module.query(UnvotedDelegatedVotingPower { ... })?;
    // 3. Sum both for total power on this proposal
    let total = individual.checked_add(udvp)?;
    Ok(VotingPowerWithDelegation { individual, total })
}
```

This delegation-aware design is crucial: a voter's total power includes both their own stake *plus* any stake delegated to them by others who haven't yet voted. The code handles this with a "fail-gracefully" pattern — if the delegation query fails, it assumes zero delegated power so votes can still be cast.

The **vote comparison logic** reveals a careful approach to threshold arithmetic:

```rust
pub fn compare_vote_count(votes: Uint128, cmp: VoteCmp, total_power: Uint128, passing_percentage: Decimal) -> bool {
    // Uses PRECISION_FACTOR = 10^9 for fixed-point arithmetic
    // NOTE: This function does NOT round up — by design, to avoid
    // reporting a proposal as both passed and rejected
    let votes = votes.full_mul(PRECISION_FACTOR);
    let total_power = total_power.full_mul(PRECISION_FACTOR);
    let threshold = total_power.multiply_ratio(passing_percentage.atomics(), ...);
    match cmp {
        VoteCmp::Greater => votes > threshold,
        VoteCmp::Geq => votes >= threshold,
    }
}
```

The precision handling is notable: the developers consciously chose *not* to round up to avoid a proposer simultaneously passing and failing a proposal — a subtle but real risk in fixed-point arithmetic on-chain.

**From generic `Voting.sol` implementations** (found across many repos):

```solidity
function vote(uint16 _choice) public duringPoll {
    uint256 dockTokens = dock.balanceOf(msg.sender);
    require(dockTokens > 0);
    // Replaces previous vote weight, prevents double-voting per poll
    if (numberOfVotes[msg.sender] > 0) {
        totalVotes[options[msg.sender]] = totalVotes[options[msg.sender]].sub(numberOfVotes[msg.sender]);
    }
    options[msg.sender] = _choice;
    numberOfVotes[msg.sender] = dockTokens;
    totalVotes[_choice] = totalVotes[_choice].add(dockTokens);
}
```

Common features across these contracts:
- **One-person-one-vote** enforcement via `numberOfVotes[msg.sender]` guards
- **Vote-changing** support (replaces previous vote rather than adding)
- **SafeMath** on all arithmetic to prevent overflow/underflow
- **State modifiers** (`duringPoll`, `onlyAuthorized`) to enforce election windows
- **IPFS-stored poll metadata** to keep proposal details off-chain but verifiable

#### Pattern C: Off-Chain Signed Voting with On-Chain Verification (ENS / Snapshot)

ENS and Snapshot use **off-chain signaling** — signed messages via web UI — to avoid gas costs, then settle finality on-chain. This drastically reduces participation barriers but introduces a trust assumption in the off-chain infrastructure.

### Cryptographic Voting Schemes

| Scheme | Example | How It Works | Tradeoff |
|---|---|---|---|
| Simple token-weighted | Celo, ENS | One token, one vote | Plutocratic; no privacy |
| Quadratic voting | DA0-DAO (debated) | Weight = √tokens_staked | Expensive to monopolize; complex to implement |
| Ring signatures | BlockVotes, Jormungandr | Voter indistinguishable from decoys | Ballot secrecy; high computational cost |
| zk-SNARKs / ZK proofs | ENS (Byzantium), Jormungandr | Prove voting eligibility without revealing vote | Strong privacy; trusted setup assumptions |

### What the Code Tells Us: Design Choices Are Political Choices

Every technical decision in these contracts is simultaneously a political decision:

| Code Choice | Political Implication |
|---|---|
| Token-weighted voting | Plutocratic — wealth translates directly to political power |
| Quadratic voting | Reduces plutocracy but makes voting computationally expensive and harder to reason about |
| Timelocks | Protects against flash-governance but slows emergency response |
| Off-chain signing (Snapshot) | Lowers participation barriers but trusts a centralized server |
| On-chain voting | Censorship-resistant but exposes vote choices to public scrutiny |
| Participation quorum | Prevents tiny quorums from passing sweeping changes but can deadlock governance |
| Delegatee-weighted voting | Enables representative democracy but opensSybil and violence-coercion vectors |

---

## Part III — Main Controversies & Open Debates

### 1. Token-Weight Plutocracy vs. "One Person, One Vote"

Most governance contracts size votes by token holdings, conflating financial stake with political legitimacy. In MakerDAO, Lido, and ENS, a small number of token holders (venture funds, exchanges, early investors) control disproportionate voting power. DA0-DAO explores composable voting modules as a way to mix membership, stake, and NFT-based power — but the default remains plutocratic. Community proposals push explicitly for non-token-weighted "one-person-one-vote" designs.

This debate is not merely theoretical. The **NEO governance crisis** (documented in [NEP #4411](https://github.com/neo-project/neo/issues/4411), "Moving toward a governance model that can actually execute", Jan 2026) is a cautionary tale. After years of part-time Council engagement — 21 members making decisions as a side job — the NEO Council became unable to execute even basic proposals. A detailed community analysis concluded: *"True governance and decision-making do not work when treated as a voluntary, part-time responsibility layered on top of already demanding roles."* The Council was well-suited for infrequent, high-level protocol decisions but incapable of continuous treasury management and strategy execution. The proposed fix: create a small, paid, full-time "Strategy & Treasury Board" elected by the Council, separating **oversight** ( Council) from **execution** (Board). This reveals a structural insight: token-weighted voting doesn't just create plutocracy — it can also create **governance paralysis** when too many token holders are part-time participants making complicated decisions.

### 2. Sybil Attacks & Identity Verification

Without verified real-world identities, anyone can create a hundred wallets and each gets voting power. This is the **Sybil problem**, and it is unsolved in general. Approaches under active debate:
- **Proof-of-personhood** (biometric or social-graph identity)
- **Quadratic voting** (makes Sybil attacks expensive but not impossible)
- **Reputation-based voting** (earned through participation, not purchased)
- **Soulbound tokens** (non-transferable identity tokens gating voting power)

Each introduces its own center of trust — the fundamental tension of digital democracy. The Cardano ecosystem is grappling with this live: in [SPO-Incentives issue #55](https://github.com/input-output-hk/spo-incentives/issues/55), "A Difficult Problem" (May 2026), ChangePool argues that Cardano's voting power centralization crisis stems from the inseparable link between ADA delegation and voting delegation. The analysis notes that *"Liquid delegation is the protocol's accountability mechanism and its primary anti-monopoly tool. But it only functions if pools need delegators — if operators depend on community-sourced stake to reach their optimal reward. Without that dependency, delegators have no leverage and the accountability channel collapses."* The essay proposes that identity confirmation within structured communities (e.g., WorkplaceDAO models) may be more effective than protocol-level solutions alone, but acknowledges that identity confirmation does not mitigate collusion risk.

### 3. Flash-Loan & Temporary-Voting-Power Attacks

An attacker can borrow governance tokens via a flash loan, capture a snapshot, vote through a malicious treasury proposal, and repay the loan — all in one transaction. This vector is documented in Web3 security analyses. **Defenses debated:** historical snapshots with holding periods, timelocks, vote locking, proposal thresholds, anti-flash-loan checks, and multi-sig execution safeguards. Celo's checkpoint-based power system is a direct response to this attack vector.

### 4. Voter Apathy & Democratic Legitimacy

Even in well-governed DAOs, typical voter turnout is catastrophically low — often under 5% of token holders. Time-locks, which improve security, also make voting feel distant and abstract. The question: does a decision made by 3% of holders truly represent the community? The NEO crisis illustrates this: even after organizing community meetings, publishing deliverables publicly, and creating a governance portal, the Council struggled to get enough votes for a simple pilot proposal for a European developer hub.

### 5. Smart Contract Security & Formal Verification

A bug in a governance contract can lead to **treasury drainage, proposal manipulation, or DAO seizure**. Many DAO governance contracts have never been formally audited — a common situation. Even audited code has historically contained critical bugs (the DAO DAO project has been audited by Oak Security multiple times, reflecting the recognition that continuous auditing is necessary, not sufficient). The debate: is formal verification (mathematically proving correctness) feasible for governance contracts, or is the state-space too complex? Cardano's Jormungandr project leans toward formal methods, while most Ethereum-based projects rely on traditional code audits.

### 6. On-Chain vs. Off-Chain Governance

ENS and Snapshot have popularized **off-chain voting** (signed messages, with on-chain finality). This drastically reduces gas costs and participation barriers, but introduces a trust assumption: the off-chain infrastructure (a website, server) could censor, manipulate, or go offline. The tradeoff is between **cost and censorship resistance** — different projects make different bets. The blockrewardsfunding DAO (Ethereum Foundation's funding mechanism) debated this explicitly in [issue #25](https://github.com/ethereum-funding/blockrewardsfunding/issues/25), "meta-DAO options" (2019), proposing to run multiple DAO models simultaneously — Moloch, Aragon, DAOstack, Colony, multisig, CLR matching — as a trial to see which governance structure actually works. No clear winner has emerged.

### 7. Privacy and Coercion

On-chain votes are public by default, enabling vote buying and coercion. Privacy-focused approaches — zero-knowledge proofs and ring signatures in BlockVotes and Jormungandr — attempt to separate *who* voted from *how* they voted. This adds complexity and new trust assumptions. The tension between **transparency** (a core blockchain value) and **ballot secrecy** (a core democratic value) remains unresolved. Cardano's "CPS-???? | Private Voting for DReps" proposal (CIP-1201, 2026) directly addresses this for delegation-based voting, acknowledging that on-chain transparency can be a governance vulnerability, not just a feature.

### 8. Timelocks: Safety vs. Agility

Timelocks enable community response and "cold-off" review, but they also slow emergency responses and create MEV/extractable-value windows that attackers can target. The NEO governance debate highlights this: the Council's part-time nature makes emergency response even slower, since decisions depend on getting enough part-time members online. No consensus exists on optimal durations or bypass mechanisms.

### 9. Governance Execution Gap: Oversight ≠ Management

A meta-controversy running through all these debates is the **execution gap**: governance mechanisms that are theoretically decentralized often produce centralized or ineffective outcomes in practice. The NEO, Cardano, and DAO DAO cases all point to the same structural insight — **oversight can remain decentralized, but execution cannot remain voluntary**. Strategy, treasury management, and operational decision-making must be treated as professional responsibilities with clear mandates, expectations, and accountability. This challenges the ideological core of decentralization: if the answer is always "more delegation," at what point does the system become indistinguishable from a traditional organization?

The TerraBioDAO design explicitly separates VOTE_PARAMS (on-chain executable) from CONSULTATION (off-chain advisory), which is a pragmatic acknowledgment that not all governance decisions need on-chain enforcement. But it also raises a question: who decides which proposals are "consultation" vs. "executable"? The answer — the DAO's admin key — is itself a centralization point.

### 10. The Precision Politics of Fixed-Point Arithmetic

A micro-controversy with macro implications: the DA0-DAO developers chose to *not* round up in their `compare_vote_count` function, specifically to prevent a proposal from being both passed and rejected. This is a beautiful example of how code-level decisions carry political weight — the choice of rounding direction determines governance outcomes in edge cases. Similar decisions appear everywhere: how to count abstentions (as participating or not?), how to handle delegated votes when a delegatee changes delegation mid-proposal, how to weight votes across different proposal types. These are not bugs; they are **constitutional choices** encoded in code.

---

## Conclusion

Digital democracy rests on a tightrope: code can make voting transparent, auditable, and borderless, but the same transparency and on-chain token mechanics introduce novel attack surfaces (flash loans, plutocracy, coercion, timelock MEV) that traditional systems largely avoided. Understanding the projects, the contract mechanics, and the live controversies is the first step toward designs that are at once **secure, inclusive, and genuinely democratic**.

The questions ahead are not merely technical:
- Is a democracy defined by one-person-one-vote, or by one-token-one-vote?
- Is ballot secrecy more important than transparent tallying?
- Is formal verification a prerequisite for democratic legitimacy?
- Can governance execution be professionalized without re-centralizing control?
- Do timelocks protect democracy or enable paralysis?
- When does a "consultation" become a "binding decision" — and who decides that?
- Is the precision of fixed-point arithmetic a constitutional matter that demands formal governance?

These questions require a synthesis of political theory, cryptography, game theory, and software engineering — and the essays, repos, and open issues of the digital democracy community are where that synthesis is being written.

---

## References

| Project / Issue | Link | Stars | Role |
|---|---|---|---|
| BlockChainVoting | [mehtaAnsh/BlockChainVoting](https://github.com/mehtaAnsh/BlockChainVoting) | 450 | Most-starred E-voting app; Next.js + Solidity + IPFS |
| BlockVotes | [yfgeek/BlockVotes](https://github.com/yfgeek/BlockVotes) | 283 | Ring-signature e-voting for ballot secrecy |
| Jormungandr | [cardano-foundation/jormungandr](https://github.com/cardano-foundation/jormungandr) | 368 | Rust privacy-preserving voting node; formal methods |
| Viction | [BuildOnViction/victionchain](https://github.com/BuildOnViction/victionchain) | 182 | Go PoS voting-consensus chain |
| DA0-DAO Contracts | [DA0-DA0/dao-contracts](https://github.com/DA0-DA0/dao-contracts) | 217 | Rust/WASM modular, composable DAO tooling; audited by Oak Security |
| ENS Governance | [ensdomains/governance-contracts](https://github.com/ensdomains/governance-contracts) | 159 | JS identity-based DAO governance |
| Celo Governance.sol | [celo-org/celo-monorepo](https://github.com/celo-org/celo-monorepo) | ~805 | Production-grade on-chain governance reference (audited, mainnet) |
| TerraBioDAO Voting.sol | [TerraBioDAO/dao-first-iteration](https://github.com/TerraBioDAO/dao-first-iteration) | — | Modular adapter pattern; Bank/Agora/Voting separation |
| Decentraland Governance | [decentraland/governance](https://github.com/decentraland/governance) | 49 | TypeScript virtual-world DAO |
| Joystream Pioneer | [Joystream/pioneer](https://github.com/Joystream/pioneer) | 43 | TypeScript governance app for Joystream DAO |
| NEO Governance Crisis | [neo-project/neo #4411](https://github.com/neo-project/neo/issues/4411) | — | "Moving toward a governance model that can actually execute" — NEP 4411 |
| Cardano SPO Incentives | [input-output-hk/spo-incentives #55](https://github.com/input-output-hk/spo-incentives/issues/55) | — | "A Difficult Problem" — voting power centralization and staking collapse |
| Meta-DAO Options | [ethereum-funding/blockrewardsfunding #25](https://github.com/ethereum-funding/blockrewardsfunding/issues/25) | — | "meta-DAO options" — comparing Moloch, Aragon, DAOstack, Colony models |
| Private Voting for DReps | [cardano-foundation/CIPs PR #1201](https://github.com/cardano-foundation/CIPs/pull/1201) | — | CPS proposal for private on-chain voting in Cardano delegation |
| Stacksgov PM | [stacksgov/pm #132](https://github.com/stacksgov/pm/issues/132) | — | Request for Comment: Stacks Code of Conduct — governance process debate (128 comments) |

_Research conducted via parallel GitHub searches: repository search for "blockchain voting" and "DAO governance"; code search for Solidity/Rust on-chain voting contract implementations; issue search for decentralized governance debates, security controversies, and execution gaps. Code analysis of Celo Governance.sol (~1,700 lines, SHA 2c5fb20) and TerraBioDAO Voting.sol (330 lines, SHA fbad62b)._
