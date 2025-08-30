
# Proposing Enduring Solutions for Solana Account Data Storage: A Comprehensive Analysis and Architectural Blueprint

## 1. The Solana State Bloat Imperative: A Crisis of Scale

### 1.1. The Account Model and Its On-Chain Footprint

Solana's architectural design prioritizes **high throughput** through parallel transaction processing, a fundamental choice directly tied to its account model.¹ Unlike monolithic blockchains that process transactions sequentially, Solana's "SeaLevel" runtime isolates account data, enabling multiple contracts to execute concurrently without global locks to prevent state collisions.¹ This approach, while driving Solana's exceptional speed and scalability, creates a systemic challenge: **every piece of account data**, regardless of size or access frequency, must be stored fully on-chain and replicated indefinitely across all validator nodes.

The consequence is a **rapidly expanding blockchain state**, known as "state bloat." This is not merely an inconvenience but an accelerating, systemic threat to the network's long-term viability. The architectural trade-off for speed and concurrency directly correlates with the on-chain data replication burden. As network activity and account numbers grow, this burden escalates non-linearly, exerting immense pressure on validators' hardware and operational capabilities. Addressing this foundational trade-off requires a strategic, enduring solution.

#### Key Architectural Trade-Offs

- **Advantage**: Parallel processing enables 1,000+ TPS.
- **Drawback**: Full on-chain replication of all data.
- **Impact**: Exponential growth in storage requirements.

### 1.2. Quantitative Analysis of State Growth and Validator Costs

Solana's state bloat is well-documented and quantifiable. As of mid-2025, the live state of all accounts stands at approximately **500 GB**² — the actively used, in-memory portion for near-instant transaction processing. However, the full, unpruned ledger, encompassing every transaction and state change since genesis, exceeds **400 TB**². This complete historical record is maintained by a small number of centralized archive nodes for data availability and developer tooling.³ The full ledger grows at a staggering rate, with estimates of several petabytes per year at full design capacity.² Archive nodes face a more concrete growth of about **80 TB per year**.³

This immense data footprint translates to significant hardware and operational costs for validators. Recommended specifications are demanding:

- **RAM**: At least 384 GB, with RPC servers needing 512 GB to 1 TB for indexes and quick data serving.²
- **Storage**: Two separate, enterprise-grade NVMe SSDs, each 3.84 TB or more, to separate accounts and ledger databases for optimized read/write operations.²
- **Costs**: Upfront hardware ranges from $15,000 to $50,000+, with monthly operational costs of $500 to $1,000 per validator.³ Archive node maintenance is an order of magnitude higher at ~$3,000/month.³

These high costs create barriers to entry for independent validators, leading to stake consolidation among well-capitalized institutions.⁶ This introduces centralization risks, including:

- **Geographic Concentration**: 68% of staked SOL delegated to European validators.⁶
- **Infrastructure Fragility**: Over-reliance on major cloud providers like AWS and Google Cloud, creating single points of failure.⁶
- **Security Implications**: Regional outages or exploits could compromise the network's security and censorship resistance.⁶

The distinction between pruning validators and centralized archive nodes is crucial, as the high number of pruning validators masks underlying data availability issues.

#### State Metrics and Hardware Requirements Overview

| Feature                  | Validator Node          | Archive Node            |
|--------------------------|-------------------------|-------------------------|
| **Current Live State Size** | ~500 GB              | ~500 GB                 |
| **Current Unpruned Ledger** | ~2 TB (Pruned)       | ~400+ TB                |
| **Ledger Growth Rate**   | Varies with pruning     | ~80 TB/year             |
| **Recommended RAM**      | 384+ GB                 | 512 GB - 1 TB           |
| **Recommended Storage**  | 2x 3.84 TB NVMe SSD     | 400+ TB NVMe SSD        |
| **Approximate Installation Cost** | $15,000+         | $45,000+                |
| **Approximate Monthly Cost** | $500 - $1,000        | $3,000                  |

### 1.3. The Developer and User Experience: The Real-World Impact of Rent

State bloat manifests practically through Solana's **rent mechanism** — a refundable SOL deposit to prevent account garbage collection.⁷ Rent is proportional to data size: a simple 16-byte account requires ~0.001 SOL, while complex programs incur higher costs.⁷ For instance, deploying a 59 KB "hello world" program costs ~0.41 SOL to be rent-exempt.⁸

While designed to encourage efficient storage and compensate validators,⁷ the rent system has become a **negative user experience factor**. Complaints on Reddit and Stack Exchange reveal confusion and frustration, particularly among developers in developing economies where small SOL amounts are significant.⁹ Users fear valuable token accounts being "purged" if wallet balances drop to zero, creating uncertainty and psychological barriers to adoption.⁹

The core issue: Rent is an imperfect fix for deeper problems — the absence of a low-cost, protocol-level solution for dormant accounts and perpetual full-replica state expansion.

#### Rent Mechanism Challenges

- **Economic Barrier**: High costs for developers and users.
- **User Confusion**: Risk of data loss without clear understanding.
- **Adoption Hurdle**: Psychological uncertainty in emerging markets.

## 2. A Comparative View: Solana vs. Ethereum State Management

### 2.1. Architectural Differences: Monolithic vs. Multi-Layer

The state bloat problem on Solana is best understood in comparison to Ethereum's approach to scaling. Solana operates as a monolithic, single-sharded blockchain where all transactions and their associated state updates occur on a single layer.1 This design philosophy, which keeps all liquidity and data in a unified state, avoids the liquidity fragmentation and user complexity that plague multi-layered networks.11 In contrast, Ethereum has pursued a multi-layered scaling strategy, relying on Layer 2 (L2) networks like Arbitrum and Optimism to process the vast majority of transactions off-chain.1 Only a minimal proof of these transactions, typically a "data blob," is submitted back to the Ethereum mainnet for final settlement and security.1 This represents a fundamental divergence in design philosophies. Solana aims for simplicity by keeping all activity on a single, high-throughput layer, while Ethereum embraces complexity to offload its computational and storage burden.
The proposed SIMD-0341 solution, which leverages state compression and off-chain data indexing, signals a significant strategic shift for Solana. By pushing account data to off-chain indexers and a cheaper, on-chain ledger, Solana is, in effect, moving closer to Ethereum's multi-layered design. This approach, while a pragmatic solution, challenges Solana's core design ethos and pushes it away from its simplicity [Query].

### 2.2. State Growth and Bottlenecks: A Data-Driven Comparison

The architectural differences between the two networks are reflected in their respective state growth metrics. Ethereum's state, while still a concern for its validator community, grows at a comparatively modest rate of approximately 2.62 GiB per month.12 This is an order of magnitude smaller than the raw ledger data growth on Solana, which is estimated to be several terabytes per month.3 The primary reason for this is Ethereum's lower transaction volume (30 TPS versus Solana's 1,000+ TPS) and the offloading of data to L2 networks, which dramatically reduces the state burden on the mainnet.11
However, both networks face the same fundamental problem: the perpetual growth of on-chain state. The crucial difference is that Solana's problem is more acute and urgent due to its design for massive throughput. Ethereum's long-term research into solutions like "statelessness" and "state expiry" protocols demonstrates a shared recognition of this challenge across the blockchain industry.13 This highlights a convergence in blockchain design philosophies. Despite their different starting points, both Solana and Ethereum are being forced to grapple with the unsustainability of a full-state, fully-replicated model at scale. Therefore, the architectural solutions proposed for Solana's state bloat problem can be viewed as a unique, high-performance take on these shared industry challenges.

## 3. The State of the Art: A Critical Review of Existing State Compression

### 3.1. Technical Mechanics of Concurrent Merkle Trees (cNFTs)

Solana has already implemented a form of state compression, primarily for high-volume use cases like Non-Fungible Tokens (NFTs).15 This solution, which is enabled by a "concurrent Merkle tree," is a cryptographic data structure that allows for multiple, rapid changes to the tree in the same block without invalidating a proof.16 The core mechanism involves storing a "fingerprint" or root hash of the data on-chain, while the actual, uncompressed data is written to the cheaper, append-only "ledger state".16
This is a clever and effective approach that significantly reduces the storage burden on the expensive "account state." To mint a compressed NFT, a program passes the metadata to the State Compression Program, which hashes it and emits it as an event to a "Noop program".19 The hash is then stored in the Merkle tree account, and the full metadata is logged to the ledger state where it exists forever, accessible for indexing.19 This process has yielded dramatic cost savings, with a tree for 16,384 nodes costing only ~0.222 SOL to create, and the cost of minting one million compressed NFTs estimated at approximately $247.80.15 This is an astounding reduction compared to the millions of dollars it would cost to mint the same number of NFTs on other chains.15

### 3.2. Successes and Limitations: Why a New Solution is Required

The existing state compression has proven to be a transformative solution for specific, high-volume use cases. Its success in the NFT space is undeniable, enabling projects like DRiP to mint over 68 million NFTs to a user base of 750,000 at a total cost of around $12,000.15 This has effectively lowered the barrier to entry for developers and users and unlocked new business models that rely on abundance and utility, rather than scarcity.15
However, this solution is not a panacea for Solana's state bloat problem. As articulated in the user query, it introduces significant trade-offs that prevent it from being a truly "enduring" solution [Query]. The primary limitation is that the uncompressed data, while existing in the ledger state, is not directly accessible by on-chain programs through Cross-Program Invocation (CPI) calls [Query]. This harms data interoperability and breaks the composability that is central to Solana's design, forcing developers to rely on third-party RPC providers to index and serve the data from the ledger.19 This creates a critical vulnerability, as the Solana protocol does not guarantee that these RPC providers will faithfully store the data, and there is no on-chain mechanism to enforce data availability or prevent censorship.20 The current compression model, therefore, solves the problem of on-chain storage cost by creating a new, potentially more dangerous problem of centralized data availability and vendor lock-in.
This is a stop-gap measure, a tactical fix that addresses one symptom (cost) by introducing a new dependency on centralized infrastructure. The report maintains that a truly enduring solution must solve the storage problem without compromising the network's core tenets of decentralization, composability, and censorship resistance.

## 4. Architectural Proposal I: A Protocol-Level Archival and Expiry System

4.1. Technical Blueprint for a Two-Tiered State Model

An enduring solution must build upon existing network behavior while addressing its shortcomings. This architectural blueprint proposes a formal, protocol-level archival and expiry system that decentralizes the existing, ad-hoc roles of pruning validators and centralized archive nodes. The system would create a two-tiered state model: an active state and a new archival state.
The active state would function similarly to the current Solana account model, where frequently accessed accounts are replicated across all validator nodes for fast finality and seamless composability. A new, protocol-level rule would define a period of inactivity (e.g., 30-90 days), after which accounts are considered "dormant." Dormant accounts would be moved from the high-cost active state to a lower-cost archival state through a formal process known as state expiry.14
The archival state would be a new, verifiable, and distributed off-chain storage network. Instead of being replicated across all validators, the data for dormant accounts would be moved to a network of specialized "archival nodes" or a decentralized storage protocol like Arweave.21 A small, on-chain "pointer" or a succinct cryptographic proof, such as a Merkle proof, would remain in the active state, attesting to the existence and integrity of the archival data.21 This approach leverages the same core concept as existing compression but generalizes it to all account data, and, crucially, decentralizes the archival role with protocol-level guarantees.

4.2. Economic Incentives for Validators and the State Archival Bounty

To ensure the long-term viability of this two-tiered model, new economic incentives would need to be introduced. The protocol would be designed to provide a "state archival bounty" or a dedicated portion of network emissions to compensate archival nodes for the immense cost of storing and serving historical data.3 This is distinct from the current rent system, which is a one-time refundable deposit, and would create a continuous revenue stream for a new class of network participants. The incentives would be structured to attract a diverse set of operators specializing in data storage, thereby further decentralizing the network's data availability layer.6 This approach diversifies the network's economic model beyond block production and aligns with ongoing discussions about programmatic, market-based emissions and the Validator Admission Ticket (VAT) proposed in
SIMD-0326.22

4.3. Migration Strategy and Backward Compatibility

Implementing this protocol would require a phased rollout via the Solana Improvement Document (SIMD) process.25 The implementation would not break existing applications immediately but would introduce new instructions and features for managing accounts in the two-tiered system. The core technical challenge would be designing a "rehydration" mechanism. This mechanism would allow a user or program to initiate a transaction that moves a dormant account back from the archival state into the active state. This would require the user to provide the on-chain proof and the off-chain data as transaction parameters, and the network would re-instantiate the full account on-chain for a fee. This approach provides a graceful migration path for existing applications while future-proofing the network against perpetual state growth.

## 5. Architectural Proposal II: A Generalized Verifiable Off-Chain Data Protocol

5.1. Leveraging ZK-Proofs for On-Chain Integrity

A more radical and future-proof solution is to fundamentally redefine the Solana account model by introducing a generalized verifiable off-chain data protocol. Instead of a dual-tiered state, this proposal suggests that all large, mutable data should be stored off-chain from the outset.
The on-chain account would no longer hold the full data blob but would instead store a small, static, cryptographic proof that attests to the integrity of the off-chain data. The use of Zero-Knowledge (ZK) proofs, a technology actively being developed in the Solana ecosystem 26, would be a cornerstone of this model. A ZK-proof is a powerful primitive that can cryptographically prove the validity of a statement about off-chain data without revealing the data itself.28 This is more robust than a simple hash or Merkle root, as it can be used to verify complex claims about the data, such as a proof of ownership or a state transition. The raw, uncompressed account data would be stored on a decentralized, globally available storage network like Arweave or IPFS.21

5.2. Redefining Data Interoperability with On-Chain Proofs

This model directly addresses the primary weakness of the existing state compression: the breaking of CPI calls and data interoperability [Query]. In this new paradigm, the ZK-proof becomes a new, first-class data primitive. When a program needs to interact with off-chain data, a user would provide the raw data and its corresponding ZK-proof as transaction parameters. The program would then use a new CPI call to a native ZK-proof verification program, such as the ZkE1Gama1Proof11111111111111111111111111111 program, to verify the proof against the small, on-chain attestation.26
This new model would be inherently composable. A program would no longer need to read and write to a large account. Instead, it would simply verify a proof, which is a computationally cheap operation. This would allow for complex, cross-program interactions to be verified on-chain without the need to ever store massive amounts of mutable data. This approach effectively makes state bloat a non-issue by design, as the on-chain "state" becomes a small, static proof rather than a large, perpetually expanding data set.

5.3. Developer Tooling and User Experience Enhancements

The primary challenge of this solution is developer complexity. Working with ZK-proofs and off-chain data is not trivial. To ensure widespread adoption, a new generation of SDKs and developer tools would need to be created to abstract away these complexities. These tools would automate the process of off-chain data storage, ZK-proof generation, and transaction construction, making the experience seamless for the developer. For the end-user, the experience would be largely the same as the current model, but with a crucial difference: they would only have to pay a minimal, one-time fee to create their small, verifiable account, with no ongoing rent costs. This is the ultimate "enduring" solution, as it solves the problem at its core and future-proofs the network.

## 6. A Multi-Criteria Analysis of Proposed Solutions

### 6.1. The Trade-Off Matrix: Feasibility, Cost, and Decentralization

An objective assessment of the proposed solutions requires a systematic comparison against the current model and the existing state compression. Each approach involves a unique set of trade-offs that must be carefully considered.

| Evaluation Criteria          | Current Solana Model      | Existing State Compression | Proposal I: Protocol-Level Archival | Proposal II: Verifiable Off-Chain |
|------------------------------|---------------------------|----------------------------|-------------------------------------|-----------------------------------|
| Storage Cost Reduction       | None; all data on-chain.  | High for specific use cases (e.g., cNFTs). | High for dormant accounts.          | Extremely high; all large data is off-chain. |
| Validator Hardware Reduction | None; burden is increasing. | Low; validators still store ledger. | High for validators; burden shifts to archival nodes. | Extremely high; on-chain state is minimal. |
| Data Interoperability        | Seamless CPI calls.       | Broken; on-chain programs cannot read data. | Full for active accounts; requires rehydration for dormant. | Full; new CPI primitive is based on proof verification. |
| Data Availability & Reliance  | Fully decentralized.      | Centralized; relies on RPC indexers. | Decentralized; relies on new, incentivized archival network. | Decentralized; relies on protocols like Arweave. |
| Developer Onboarding         | Low; a standard account model. | Medium; requires specific libraries and off-chain indexing. | Medium; new APIs for state expiry and rehydration. | High; requires new SDKs to abstract ZK-proofs. |
| Implementation Timeline      | N/A                       | Implemented (for NFTs).    | Medium-term (1-3 years) via SIMD process. | Long-term (3-5+ years) via a new account model. |
| User Experience Impact       | High rent costs.          | Low cost; new centralization risks for data. | Low cost; requires rehydration transaction for dormant data. | Minimal cost; seamless experience with new SDKs. |

### 6.2. Short-Term Gains vs. Long-Term Resilience

The analysis in the trade-off matrix indicates a clear distinction between the existing state compression and the two proposed architectural blueprints. The current compression solution is a tactical, short-term fix. It delivers immediate, dramatic cost reductions for specific applications like NFTs but does so at the expense of core network principles like decentralized data availability and composability. Its reliance on centralized RPC indexers is a significant centralization vector that could be a point of failure or censorship.
In contrast, both Proposal I and Proposal II represent strategic, long-term solutions. Proposal I, a protocol-level archival and expiry system, is the most feasible and least disruptive "enduring" solution. It formalizes an existing behavior of the network, creating new economic incentives and a clear path to managing dormant accounts without compromising on-chain interoperability. Proposal II, a generalized verifiable off-chain data protocol, is the ultimate "enduring" solution. While its implementation is more complex and would require a fundamental re-architecture of the network, it would solve the state bloat problem permanently by design.
The path forward, therefore, should be a multi-pronged approach. Solana should continue to leverage its existing state compression for high-volume, low-value use cases where the trade-offs are acceptable. Simultaneously, it must prioritize the technical and governance work required to propose and implement a protocol-level archival and expiry system. This would be a multi-year effort executed through the SIMD process and would require collaboration from core teams like Anza and Jump.25

### 6.3. The Path to Implementation via the SIMD Governance Process

Any major change to the Solana protocol requires formalization through the Solana Improvement Document (SIMD) process.25 This process is collaborative and iterative, involving a proposal, community discussion, and a non-binding validator vote.25 The core development teams, such as Anza and Jump, play a significant role in proposing and implementing these changes.25
For Proposal I, the path to implementation would involve drafting a detailed SIMD that outlines the two-tiered state model, the rules for state expiry, the economic incentives for archival nodes, and a clear migration strategy. This proposal would be submitted to the solana-improvement-documents repository and undergo a period of public review and debate on forums and in the Solana Discord.25 Final adoption would be contingent on a two-thirds supermajority vote from validators, signaling their sentiment for the change, followed by their decision to adopt the new software.25

## 7. Conclusion and Final Recommendations

### 7.1. Summary of Key Findings

The analysis presented in this report confirms that Solana's account data storage model is a critical, accelerating problem with systemic implications for network decentralization, security, and user experience. The unpruned ledger, already over 400 TB in size, places a prohibitive burden on validators and leads to stake consolidation among well-capitalized entities. While the existing state compression has proven its value for specific applications like NFTs, it is not a complete or enduring solution, as it compromises on-chain interoperability and relies on centralized third parties for data availability.

### 7.2. Strategic Recommendations

The following recommendations are provided to guide Solana's strategic response:

1. **Acknowledge and Differentiate**: Acknowledge the current state compression's value for niche applications that do not require on-chain data composability or guaranteed data availability. However, it must be recognized that this solution is a tactical fix, not a strategic one.

2. **Prioritize the Feasible**: Prioritize the development and formal proposal of a generalized, protocol-level archival and expiry system (Proposal I). This solution is the most feasible and least disruptive "enduring" fix, as it aligns with the network's current operational reality and formalizes the role of archive nodes, thereby decentralizing the data availability layer.

3. **Initiate Long-Term Research**: Simultaneously, initiate long-term research and development for a generalized verifiable off-chain data protocol (Proposal II). This represents the ultimate "enduring" solution that would fundamentally future-proof the network by decoupling on-chain state from large, off-chain data and leveraging new cryptographic primitives like ZK-proofs. This would be a multi-year effort that redefines the Solana account model.

### 7.3. Call to Action

Solana is at a critical inflection point where a strategic decision on state management will define its path to capturing the multi-trillion-dollar opportunity in tokenizing real-world assets and building a new generation of consumer applications.^30 By addressing the state bloat imperative with a clear, multi-phased approach, the network can secure its future as a high-performance, decentralized, and scalable blockchain.

## Works cited

1. **Ethereum vs. Solana for Developers: A Technical Comparison** | Guides - GoldRush, accessed August 30, 2025, <https://goldrush.dev/guides/ethereum-vs-solana-for-developers-a-technical-comparison/>
2. **Solana Full Node: Complete Guide** - GetBlock.io, accessed August 30, 2025, <https://getblock.io/blog/solana-full-node-complete-guide/>
3. **Solana Archive Node Guidelines** - GetBlock.io, accessed August 30, 2025, <https://getblock.io/blog/solana-archive-node-guidelines/>
4. **Solana Archive Node: Complete guide** - NOWNodes Blog, accessed August 30, 2025, <https://nownodes.io/blog/solana-archive-node-complete-guide/>
5. **How to Host a Solana Validator Node: Hardware & Setup Guide** - ServerMania, accessed August 30, 2025, <https://www.servermania.com/kb/articles/how-to-host-solana-validator-node>
6. **Solana's Centralization Paradox and Its Economic Impact** | by Toba - Medium, accessed August 30, 2025, <https://medium.com/@tobs.x/solanas-centralization-paradox-and-its-economic-impact-cdd357abd9bc>
7. **What is Rent on Solana and How to Calculate it** | QuickNode Guides, accessed August 30, 2025, <https://www.quicknode.com/guides/solana-development/getting-started/understanding-rent-on-solana>
8. **Program rent** : r/solana - Reddit, accessed August 30, 2025, <https://www.reddit.com/r/solana/comments/r432am/program_rent/>
9. **Solana Rent** : r/Tangem - Reddit, accessed August 30, 2025, <https://www.reddit.com/r/Tangem/comments/1b5fakr/solana_rent/>
10. **How Accounts and rent works in solana?**, accessed August 30, 2025, <https://solana.stackexchange.com/questions/18913/how-accounts-and-rent-works-in-solana>
11. **Solana vs Ethereum: Which is Better in 2025? | Pros and Cons** - Backpack Learn, accessed August 30, 2025, <https://learn.backpack.exchange/articles/solana-vs-ethereum>
12. **How to Raise the Gas Limit, Part 1: State Growth** - Paradigm, accessed August 30, 2025, <https://www.paradigm.xyz/2024/03/how-to-raise-the-gas-limit-1>
13. **A Protocol Design View on Statelessness** - Economics - Ethereum Research, accessed August 30, 2025, <https://ethresear.ch/t/a-protocol-design-view-on-statelessness/22060>
14. **Statelessness series — Part1: State Expiry & History Expiry** | by Chaisomsri - Medium, accessed August 30, 2025, <https://medium.com/@chaisomsri96/statelessness-series-part1-state-expiry-history-expiry-2bbd5835b329>
15. **State Compression Unlocked 'Cambrian Explosion for Digital Assets'** - Solana, accessed August 30, 2025, <https://solana.com/en/news/state-compression-update-december-2023>
16. **State Compression** | Solana - Solana Documentation, accessed August 30, 2025, <https://edge.docs.solana.com/learn/state-compression>
17. **State Compression** - Solana, accessed August 30, 2025, <https://solana.com/ja/developers/guides/advanced/state-compression>
18. **Generalized State Compression objectives** | Solana, accessed August 30, 2025, <https://solana.com/ru/developers/courses/state-compression/generalized-state-compression>
19. **Compressed NFTs** - Solana, accessed August 30, 2025, <https://solana.com/developers/courses/state-compression/compressed-nfts>
20. **Scalability with State Archival on Stellar vs. Solana's Avocado**, accessed August 30, 2025, <https://stellar.org/blog/developers/introducing-state-archival-part-2-scalability-vs-solana-s-avocado>
21. **On-Chain vs. Off-Chain Storage: Understanding Blockchain Data Solutions** - DX Talks, accessed August 30, 2025, <https://www.dxtalks.com/blog/news-2/on-chain-vs-off-chain-blockchain-storage-805>
22. **SIMD-0326: Proposal for the New Alpenglow Consensus Protocol** - Governance, accessed August 30, 2025, <https://forum.solana.com/t/simd-0326-proposal-for-the-new-alpenglow-consensus-protocol/4236>
23. **Alpenglow Vote Could Redefine Solana's Protocol** - Cointribune, accessed August 30, 2025, <https://www.cointribune.com/en/alpenglow-vote-could-redefine-solanas-protocol/>
24. **Proposal For Introducing a Programmatic, Market-Based Emission Mechanism Based on Staking Participation Rate** - Solana Developer Forums, accessed August 30, 2025, <https://forum.solana.com/t/proposal-for-introducing-a-programmatic-market-based-emission-mechanism-based-on-staking-participation-rate/3294>
25. **Understanding Solana Improvement Documents (SIMDs): A Deep Dive into Protocol Governance and Comparison with Ethereum's EIPs** - DeFi Prime, accessed August 30, 2025, <https://defiprime.com/solana-simds>
26. **Solana ZK ElGamal Proof Program** - Agave Validator Documentation, accessed August 30, 2025, <https://docs.anza.xyz/runtime/zk-elgamal-proof>
27. **Best ZK Platforms On Solana: Top Zero-Knowledge Proof Solutions**, accessed August 30, 2025, <https://solanacompass.com/projects/category/research/zk-proofs>
28. **Zk Proofs to Verify Off-chain Data** | Space and Time - SpaceandTime.io, accessed August 30, 2025, <https://www.spaceandtime.io/blog/zk-proofs-to-verify-off-chain-data>
29. **Using Governance to Prioritize SIMD Acceptance** · solana-foundation solana-improvement-documents · Discussion #227 - GitHub, accessed August 30, 2025, <https://github.com/solana-foundation/solana-improvement-documents/discussions/227>
30. **Solana Ecosystem Report (H1 2025) — Earnings & Growth** - Helius, accessed August 30, 2025, <https://www.helius.dev/blog/solana-ecosystem-report-h1-2025>
31. **Solana H1 2025 Report: DeFi, RWAs & Inst. Growth** - QuickNode Blog, accessed August 30, 2025, <https://blog.quicknode.com/solana-ecosystem-report-h1-2025/>
