# Digital Democracy: An Essay Outline

## Introduction

The promise of digital democracy is deceptively simple: use technology to make governance more transparent, inclusive, and resistant to corruption. From blockchain-based e-voting systems to decentralized autonomous organizations (DAOs), a growing ecosystem of projects claims to be building the tools for a more direct and participatory political future. Yet as the debate around Ethereum's Diamond Standard (EIP-2535) and Cardano's Voltaire governance proposal (CIP-1694) show, the path from `code on a blockchain` to `legitimate self-governance` is fraught with unresolved tensions around centralization, voter apathy, security, and the very meaning of a "vote." This essay surveys the key projects, examines how on-chain voting contracts actually work at the code level, maps the principal controversies that define the field today, and draws on live community debates to ground every claim in evidence.

---

## Part I — Key Projects in Blockchain Voting and DAO Governance

### 1. Blockchain E-Voting Systems

#### mehtaAnsh/BlockChainVoting (450 ⭐)
- **What it is:** A full-stack blockchain-based e-voting system created as a final-year academic project. Built with Solidity/Web3 smart contracts, a Next.js & Semantic UI React front-end, and a MongoDB/ExpressJS back-end, with IPFS for media storage.
- **Why it matters:** Illustrates the *basic mechanics* of on-chain voting — candidate registration, voter authentication via email, secure vote casting, and automated result announcement. It demonstrates how a traditional election workflow can be mapped onto a blockchain, while also exposing the limitations (e.g., reliance on off-chain email for voter identity, test-net Ether from faucets).
- **Key takeaway:** The "voting" part is trivially simple on-chain; the hard part is *binding real identity to a wallet* without reintroducing the trusted intermediaries blockchain was meant to eliminate.

#### yfgeek/BlockVotes (283 ⭐)
- **What it is:** An e-voting system based on blockchain using *ring signatures* for privacy.
- **Why it matters:** Addresses the anonymity problem — ring signatures allow a voter to sign a vote without revealing which key in a group produced the signature, echoing concepts from Monero. This is a step toward the secret ballot on a public ledger. It demonstrates that privacy-preserving voting is not just a theoretical concern but a live engineering challenge with real implementations.
- **Trade-off:** Ring signatures increase transaction size and verification cost. Privacy on-chain is never free — it comes at the price of scalability and auditability.

#### cardano-foundation/jormungandr (368 ⭐)
- **What it is:** A privacy-focused voting blockchain node written in Rust, from the Cardano ecosystem.
- **Why it matters:** Shows that formal, research-driven blockchain projects (Cardano's peer-reviewed approach) are also exploring voting use-cases, not just DeFi. The Rust implementation signals a shift toward formally verifiable voting protocols — where the correctness of the voting contract can be mathematically proven rather than merely tested.

### 2. DAO Governance Platforms

#### DA0-DA0/dao-contracts (218 ⭐)
- **What it is:** A collection of composable, modular, upgradable smart contracts for building DAOs on the WebAssembly (Cosmos) ecosystem. Written in Rust using the CosmWasm framework. Audited by Oak Security.
- **Architecture — Three Modular Layers:**
  1. **Voting Power Module** — Determines *who gets to vote*. Supports staked governance tokens (CW20), staked NFTs (CW721), or simple membership (CW4). The `dao-voting-cw20-staked` contract (470 lines of Rust) explicitly implements an `ACTIVE_THRESHOLD` mechanism — meaning voting power is not just based on token holdings but on *active participation* (staking). The code checks `if let ActiveThreshold::Percentage { percent } = active_threshold` and validates that `percent > 0 && percent <= 100`, ensuring that a minimum percentage of the total supply must be actively staked for a vote to be considered legitimate.
  2. **Proposal Module** — Determines *how decisions are made*. The `dao-proposal-single` contract (1,224 lines of Rust) implements a complete proposal lifecycle: `Pending → Active → Passed/Failed → Queued → Executed`. Key functions include `handle_delegate_vote_override` (for delegated voting), `validate_voting_period` (temporal bounds), and `Threshold`-based quorum logic. It supports `SingleChoiceProposeMsg` for yes/no referenda and integrates with pre-propose modules (`dao-pre-propose-single`) that gate proposal creation.
  3. **Core Module** — Holds the DAO treasury and enforces the outcome of proposals.
- **Why it matters:** The modular design means any voting module can pair with any proposal module — a kind of "lego" approach to governance. This is the most explicit attempt to *parameterize* democracy itself: you can swap in different voting rules and watch how outcomes change. The code-level separation of concerns (voting power vs. proposal logic vs. treasury) is an architectural answer to the philosophical question: *is democracy a single process or a market of components?*
- **Philosophy:** The project's manifesto is blunt: "Our institutions grew rapidly after 1970, but their priorities shifted from growth to protectionism. We're fighting this." It frames DAOs as the answer to ossified institutional inertia.

#### ensdomains/governance-contracts (159 ⭐)
- **What it is:** The smart contracts powering the ENS (Ethereum Name Service) DAO, deployed via Hardhat.
- **Why it matters:** ENS is one of the most successful real-world DAOs, governing a critical piece of Ethereum infrastructure (the .eth naming system). Its contracts show how token-weighted voting operates in practice — including airdrop distributions, proposal lifecycle management, and execution via timelocks.

#### decentraland/governance & Joystream/pioneer
- **What they are:** Governance front-ends for Decentraland (a virtual world DAO) and Joystream (a content streaming DAO).
- **Why they matter:** Demonstrate that DAO governance isn't just about *protocol-level* decisions — it extends to content moderation, treasury allocation for virtual land, and community-driven curation. The `pioneer` app shows that DAO governance can be packaged as a consumer product, not just a developer tool.

---

## Part II — How On-Chain Voting Contracts Actually Work

### Anatomy of a Governor Contract: GovernorAlpha (TrustToken / Compound Fork)

The canonical on-chain governance pattern is captured in **GovernorAlpha.sol**, a 464-line Solidity contract (MIT-licensed, originally from Compound Labs). Its structure reveals the core mechanics — and the core tensions — of digital democracy:

```sol
contract GovernorAlpha is UpgradeableClaimable {
    uint public votingPeriod;          // Duration of voting, measured in blocks
    ITimelock public timelock;         // Delayed execution — proposals don't act immediately
    IVoteToken public trustToken;      // The governance token (ERC-20)
    IVoteToken public stkTRU;          // Staked voting token (boosts influence)
    address public guardian;           // Emergency override — a "break glass" account
    mapping(uint => Proposal) public proposals;
    mapping(address => uint) public latestProposalIds;
}
```

**Key mechanisms:**

1. **Proposal Lifecycle:** A proposal moves through *Pending → Active (voting period) → Succeeded/Failed → Queued (timelock) → Executed*. This multi-stage process is designed to prevent flash-vote attacks and give token holders time to react.

2. **Quorum Requirement:** `quorumVotes()` defines the minimum number of votes needed before a proposal can even be considered. Without a quorum, the vote is meaningless — a direct echo of real-world absenteeism thresholds.

3. **Delegation:** Token holders can delegate their voting power to representatives (or vote directly). This is the on-chain analogue of representative democracy — you can either participate directly or lend your voice to someone you trust.

4. **Timelock:** Even after a proposal passes, there is a *mandatory delay* before it can be executed. This is a critical security feature — it gives the community time to object, exit, or propose a counter-measure. It is the on-chain equivalent of "delayed enactment" in legislative process.

5. **Guardian Role:** A single `guardian` address can *cancel* a proposal or *execute* one without waiting for the timelock. This is the naked centralization — the emergency brake. Critics point to this as evidence that even the most sophisticated on-chain governance ultimately relies on a trusted human custodian.

6. **Upgradeability via Proxy Pattern:** The contract is `UpgradeableClaimable`, meaning the implementation logic can be swapped out by the admin. This creates a paradox: a contract designed to be *immutable and trustless* is itself *upgradeable*, which means its rules can change under its users.

### The DAO DAO Modular Pattern: Code-Level Deep Dive

By contrast, the DA0-DA0 framework *severs* the upgradeability question from the voting logic itself. Let's look at what the actual code reveals:

#### The Voting Power Contract (`dao-voting-cw20-staked/src/contract.rs`, 470 lines)

```rust
pub fn instantiate(
    deps: DepsMut, env: Env, info: MessageInfo, msg: InstantiateMsg,
) -> Result<Response, ContractError> {
    set_contract_version(deps.storage, CONTRACT_NAME, CONTRACT_VERSION)?;
    DAO.save(deps.storage, &info.sender)?;
    if let Some(active_threshold) = msg.active_threshold.as_ref() {
        if let ActiveThreshold::Percentage { percent } = active_threshold {
            if *percent > Decimal::percent(100) || *percent <= Decimal::percent(0) {
                return Err(ContractError::InvalidActivePercentage {});
            }
        }
        ACTIVE_THRESHOLD.save(deps.storage, active_threshold)?;
    }
    // ... sets up staking contract, token info, and reply handlers
}
```

Key observations from the code:
- **Active threshold as a first-class citizen:** Unlike GovernorAlpha, where quorum is a static parameter, DAO DAO makes *active participation* a required condition. Voting power is not derived from passive token holding but from *deliberate, economic commitment* (staking). This is a fundamental design choice: it costs something to vote, which should reduce sybil attacks but also raises barriers to entry.
- **Precision arithmetic:** The `PRECISION_FACTOR: u128 = 10u128.pow(9)` constant shows that the contract uses fixed-point arithmetic for percentage calculations — a subtle but critical detail. On-chain voting requires deterministic math; floating-point operations are not allowed on most blockchains. This means every threshold calculation is an approximation, and the precision of that approximation can subtly affect outcomes.
- **Reply-based architecture:** The `INSTANTIATE_TOKEN_REPLY_ID` and `INSTANTIATE_STAKING_REPLY_ID` constants show that the contract uses *cross-contract replies* — it instantiates other contracts (the staking contract, the token contract) and handles their responses asynchronously. This means the voting contract's behavior depends on the *successful execution of other contracts*, introducing a layering of dependencies that is absent in monolithic governor contracts.

#### The Proposal Contract (`dao-proposal-single/src/contract.rs`, 1,224 lines)

```rust
use dao_voting::voting::{
    get_total_power, get_voting_power, get_voting_power_with_delegation, validate_voting_period,
    Vote, Votes,
};
use dao_voting::threshold::Threshold;
use dao_voting::veto::{VetoConfig, VetoError};
```

Key observations:
- **Delegation as a core primitive:** The import of `handle_delegate_vote_override` shows that delegation is not an afterthought but a first-class mechanism built into the voting module. The contract explicitly handles cases where a delegate overrides their own vote — a level of sophistication that GovernorAlpha's simple delegation mapping does not provide.
- **Veto configuration:** The `VetoConfig` and `VetoError` types suggest that DAO DAO supports *veto mechanisms* — a feature inspired by constitutional systems where certain actors (or none) can block proposals. This is a governance design choice encoded in the contract: who has the power to say "no"?
- **Module hooks:** The `new_proposal_hooks`, `proposal_completed_hooks`, and `proposal_status_changed_hooks` imports reveal a *hook system* — external contracts can react to proposal events. This is the modular architecture in action: governance is not just the proposal vote, but the *ecosystem of responses* that the vote triggers.
- **Version migration:** The `v1_state` module (`v1_duration_to_v2`, `v1_expiration_to_v2`, etc.) shows that the contract has undergone *on-chain upgrades* — v1 logic was migrated to v2. This is the upgradeability paradox made concrete: the contract *itself* was evolved, meaning its rules changed under its users.

### The Architectural Divergence: Monolithic vs. Modular

This architectural contrast — **monolithic governor (GovernorAlpha) vs. composable modules (DAO DAO)** — mirrors a deeper philosophical split in digital democracy:

- **Monolithic approach:** Governance is a single authoritative process, like a parliamentary system. The rules are in one place, the execution is atomic, and the accountability is clear — but so is the centralization risk.
- **Modular approach:** Governance emerges from interoperable components, like a market of ideas. The rules are distributed, the execution is composable, and the flexibility is maximized — but so is the diffusion of accountability. When there's no single "Governor," who is responsible when things go wrong?

---

## Part III — The Main Controversies

### 1. The Upgradeability Paradox: Can a Contract Be Both Immutable and Governed?

**The debate (EIP-2535 Diamond Standard, 179 comments):**
The Diamond pattern allows a contract to exceed Ethereum's 24KB size limit by splitting logic into "facets" that can be added, removed, or replaced by a controlling "diamond owner." The comments on EIP-2535 reveal a deep schism:

- **Proponents** (led by author `mudgen`) argue that diamonds are simply practical — real-world contracts *need* to evolve, and the ability to upgrade is a feature, not a bug. They point to existing usage (Enjin's ERC-1155, Caesar's Triumph) as evidence that upgradeable architectures are already mainstream.

- **Critics** (notably `leonardoalt`) retort: *"How is this standard any different from the centralized owned upgradeable smart contracts out there? Why not a standard that abstracts upgrades being opt-in only by default?"* And further: *"To me Uniswap is great exactly because it's not upgradeable/centralized."*

**The DAO DAO case study:** The v1-to-v2 migration in `dao-proposal-single` is the upgradeability paradox played out in practice. The contract *was* upgraded — its rules *did* change under its users. The migration code (`v1_duration_to_v2`, `v1_expiration_to_v2`, `v1_status_to_v2`, `v1_threshold_to_v2`, `v1_votes_to_v2`) shows that the community accepted this change. But acceptance is not the same as legitimacy. The question remains: *did the voters consent to the change, or did the developers impose it?*

**The core tension:** If a governance contract can be upgraded by its admin, then the "rules" of the system are never truly fixed. The vault that holds democracy's treasury can itself be changed. This is the *upgradeability paradox*: the tool designed to make governance more flexible can also make it more vulnerable to capture.

### 2. The Quorum Problem: Who Counts as "The People"?

Both GovernorAlpha and DAO DAO contracts implement quorum requirements — a minimum participation threshold before a vote is valid. But quorum creates a fundamental dilemma:

- **Voter apathy vs. legitimacy.** If only 5% of token holders vote, is a proposal with 51% approval "legitimate"? In traditional elections, low turnout delegitimizes results. On-chain, the code doesn't care — it just checks the raw number.
- **Plutocratic concentration.** In token-weighted systems, a single whale can meet the quorum alone. This is sometimes called "governance through gold" — governance-by-wealth. DAO DAO's staking-based voting modules (`ACTIVE_THRESHOLD`) partially address this (require *active* participation, not just passive holding), but the fundamental problem remains: economic power translates directly into political power. The `PRECISION_FACTOR` constant in the code reminds us that even the calculation of "active participation" is an approximation — and approximations can be gamed.
- **Delegation illusions.** Delegation is meant to be a remedy — token holders who don't want to vote directly can delegate to experts. But in practice, delegation often follows *social reputation* rather than *policy expertise*, creating de facto power structures that mirror real-world politics (and its patron-client networks). The DAO DAO code's `handle_delegate_vote_override` function shows that delegation is not a simple transfer of power — it's a *reversible* action, which means delegates can be pressured, bought, or replaced.

### 3. The Gnolang GNO Debate: Governance as a Design Problem

**The debate (gnolang/gno#519, 6 comments, 3 👀):**
User `piux2` opened a detailed, three-part proposal for the GNO chain's governance, which reveals a crucial insight: **governance design is fundamentally about conflict resolution, not just vote counting.**

The proposal identifies three distinct governance domains:
1. **Evaluation DAO** — Manages community contribution assessment (categorizing work, quantifying work, qualifying submissions, deciding winners, sizing rewards, changing rules).
2. **Decentralists DAO** — Manages Cosmos Hub improvement proposals (approving initiatives, approving budgets, qualifying implementations, deciding winning deliveries, distributing funds, changing rules).
3. **GNO Chain Governance** — Manages chain-level decisions (parameter changes, chain upgrades, changing DAO rules themselves).

Key observations from the debate:
- **Two types of decisions require two types of voting:** The proposal explicitly distinguishes between *approval voting* (yes/no for proposals) and *score voting* (ranking/rating for selection). This is a sophisticated recognition that different governance questions need different democratic mechanisms — a nuance absent from most on-chain systems that default to simple majority.
- **The "skin in the game" problem:** The proposal notes that if GNO adopts Interchain Security (ICS), it "removes the incentive and skin in the game of token delegation and bonding from the POS token. Therefore, we should introduce new incentives and skin in the game for those governance tokens." This is a profound insight: *economic security and governance legitimacy are intertwined.* If validators don't risk capital, their votes become cheap signals. The GNO community is essentially asking: *can you have meaningful governance without meaningful economic stakes?*
- **The two-tier model:** The proposal considers both "one vote per person" (simple but vulnerable to Sybil attacks) and "token-weighted voting" (secure but plutocratic). It also floats "privilege-based" voting weight, which raises the question: *who gets to define privilege?* This is the identity problem recast as a governance design question.

### 4. The "Global Brain" Vision and Its Discontents

**The debate (Tribler/tribler#7064, 13 comments, 13 📝 label):**
The Tribler project's "Global Brain" roadmap is perhaps the most ambitious vision for digital democracy — and the most candid about its own challenges. The issue, labeled as a "memo" (Tribler's label for "stuff that can't be solved"), lays out a roadmap that spans:

- **Decentralized search** (a "decentralized Google")
- **Misinformation resistance** (a "web-of-trust which works" with "zero-trust architecture")
- **Incentive alignment** ("People contribute, instead of freeride")
- **Scalability** ("Works for 2 people and 2 billion people")
- **Self-governance** ("owned by both everybody and nobody")
- **Self-funding** (Taproot-based DAO with FROST threshold signatures, retroactive funding)

The roadmap explicitly cites the Wikipedia wisdom: *"a more distributed form of decision-making would decrease the power of governments, corporations or political leaders, thus increasing democratic participation and reducing the dangers of totalitarian control."*

But the "memo" label is telling — Tribler acknowledges that some of these challenges are *not solvable* with current technology. The scientific challenges table is candid:

| Challenge | The Unresolved Problem |
|---|---|
| Content search | Can you create a decentralized Google? |
| Misinformation | Can you build a web-of-trust that actually works? |
| Incentive alignment | Can you prevent free-riding at scale? |
| Decentralization + Scalability | Does the system degrade gracefully from 2 to 2 billion users? |
| Self-governance | Can something be "owned by both everybody and nobody"? |
| Self-funding | Can democratic voting on proposals actually fund a sustainable ecosystem? |

This is the dark secret of digital democracy: **the most ambitious projects are also the most aware of their own impossibility.** The global brain vision doesn't hide its difficulties — it *labels them as unsolvable* and moves forward anyway. This is either heroic optimism or reckless hubris, depending on your perspective.

### 5. The Cardano Voltaire Question: Can On-Chain Governance Ever Be "Good Enough"?

**The debate (CIP-1694, 303 comments, 55 reactions):**
Cardano's Voltaire phase aims to transition the network from a *founder-governed* system to a *community-governed* one through on-chain mechanisms. CIP-1694 is the first concrete proposal, and it has generated enormous discussion:

- **DReps (Delegated Representatives):** The proposal introduces a two-tier system where voters can either vote directly or delegate to *DReps* — professional governance participants. This is explicitly modeled on representative democracy. But who accredits DReps? The proposal itself? A separate on-chain registry? Social consensus? The Cardano community is still debating this, and the answer will determine whether Voltaire is *direct* democracy (everyone votes) or *representative* democracy (experts vote for you).
- **Committee governance:** The proposal includes "Committee" roles (Conflict Resolution Committee, etc.) that can intervene in disputes. Critics argue this reintroduces a *quasi-constitutional* authority — a "committee of wise persons" — that is structurally indistinguishable from the off-chain governance Cardano claims to be replacing. This is the authoritarian hereditary principle in democratic clothing: *who watches the watchers?*
- **Threshold settings:** The specific parameters (e.g., what percentage constitutes a "yes" majority, how many DReps are needed for quorum) are politically loaded. A 3% threshold for constitutional amendments is very different from a 50% + 1 threshold for routine spending. The CIP's authors acknowledge this, but the community is still debating whether *any* fixed parameter set can fairly represent a diverse global community.

**The reaction split (21 👍 vs. 3 👎)** suggests broad consensus on *principles* but deep disagreement on *parameters* — the classic problem of constitutional design.

### 6. The Identity Problem: One Person, One Vote vs. One Token, One Vote

Perhaps the deepest controversy is the most fundamental: **what is the unit of democratic personhood?**

- **Token-weighted voting** (1 token = 1 vote) is the default in most DAOs. It aligns political influence with economic stake — you can only be harmed by bad decisions if you've invested capital. Critics call this "plutocracy with a veneer of democracy."
- **One-person-one-vote** requires some form of identity verification (KYC, soulbound tokens, quadratic voting). This is politically egalitarian but technically invasive — how do you prove you're a unique human without creating a surveillance infrastructure?
- **Quadratic voting** (the cost of votes increases quadratically) is a compromise — it allows anyone to buy votes, but the marginal cost discourages concentration. But it requires a known cost function, which means a known token economy, which means *governance over the voting mechanism itself*.

None of these systems are ideologically neutral. Each one encodes a different theory of *what democracy is for* — protection of property, expression of popular will, or something else entirely. The BlockVotes ring signature approach adds a *fourth dimension*: maybe the question isn't *who gets to vote* but *whether anyone can trace how you voted*. Privacy is not just an anonymity feature — it's a *democratic infrastructure*. Without it, voting becomes surveillance.

---

## Part IV — Synthesis and Open Questions

1. **Can modular governance (DAO DAO's approach) solve the monolithic contract problem, or does it just diffuse accountability?** When there's no single "Governor," who is responsible when things go wrong? The DAO DAO code's hook system and module separation suggest that *responsibility is architectural* — but architecture is not agency.

2. **Is the upgradeability paradox solvable through "optimistic governance" — where upgrades are assumed valid unless challenged within a time window?** Or does any upgrade mechanism inherently undermine the immutability that makes blockchain trustworthy? The DAO DAO v1-to-v2 migration shows that upgrades *have happened* — but the community's consent to those upgrades was never formally recorded on-chain.

3. **Does voter apathy in DAOs reflect a rational response to low stakes, or a structural flaw in token-weighted systems?** Would quadratic voting, quadratic funding, or conviction voting change the calculus — or just create new forms of gaming? The GNO governance proposal's "skin in the game" analysis suggests that *economic stakes are necessary but not sufficient* — you need stakes that are *meaningful* relative to the decision at hand.

4. **Is the move from off-chain governance (forum discussions, signal-breaking) to on-chain governance (smart contract execution) inherently *radicalizing* — making compromises harder because "the code doesn't negotiate"?** The Tribler "Global Brain" memo's label — "stuff that can't be solved" — suggests that some governance challenges are *inherently off-chain* and that forcing them on-chain doesn't solve them, it *codifies them*.

5. **Can digital democracy ever achieve *deliberation* — the kind of reasoned, context-sensitive discourse that characterizes the best democratic moments — or is it inevitably reduced to *tabulation* — the mere counting of preferences?** The GNO proposal's distinction between approval voting and score voting is a step toward deliberation — it recognizes that *how* you vote matters, not just *whether* you vote. But on-chain systems are fundamentally *tabulation* engines. Can a blockchain be a forum?

6. **Is privacy (BlockVotes' ring signatures, Cardano's DRep anonymity) compatible with *accountability*?** A secret ballot prevents coercion but also prevents verification. If no one can trace how you voted, no one can prove that the outcome was "fair." Is a verifiable but transparent election more democratic than a private but unverifiable one?

---

## Sources & References

### Repositories
- **BlockChainVoting** — https://github.com/mehtaAnsh/BlockChainVoting
- **BlockVotes** — https://github.com/yfgeek/BlockVotes
- **jormungandr** — https://github.com/cardano-foundation/jormungandr
- **DAO DAO Contracts** — https://github.com/DA0-DA0/dao-contracts
  - Voting module: `contracts/voting/dao-voting-cw20-staked/src/contract.rs`
  - Proposal module: `contracts/proposal/dao-proposal-single/src/contract.rs`
- **ENS Governance Contracts** — https://github.com/ensdomains/governance-contracts
- **Decentraland Governance** — https://github.com/decentraland/governance
- **Joystream Pioneer** — https://github.com/Joystream/pioneer

### On-Chain Governance Classics
- **GovernorAlpha.sol** — https://github.com/trusttoken/contracts-pre22/blob/main/contracts/governance/GovernorAlpha.sol

### Issues & Debates
- **gnolang/gno#519** — Evaluation DAO, Decentralist DAO, and GNO Chain Governance Proposal — https://github.com/gnolang/gno/issues/519
- **Tribler/tribler#7064** — The Global Brain: Roadmap and Scientific Challenges — https://github.com/Tribler/tribler/issues/7064

### Standards & Proposals
- **EIP-2535: Diamonds Standard** — https://github.com/ethereum/EIPs/issues/2535
- **CIP-1694: Voltaire On-Chain Governance** — https://github.com/cardano-foundation/CIPs/pull/380

---

*This outline is a living document. As the projects evolve and the debates continue, the conversation about digital democracy is written not in stone, but in code — and code, as we've seen, is always open to revision. The MFA story of the `dao-proposal-single` contract (1,224 lines, two major versions, migration hooks) is a reminder that governance is not a product — it's a process. And processes, by definition, are never finished.*