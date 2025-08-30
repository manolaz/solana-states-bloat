
# Proposing Enduring Solutions for Solana Account Data Storage: A Comprehensive Analysis and Architectural Blueprint

**Author:** Tristan Nguyen, Founder of AMOCA  
**Contact:** Discord @tristannguyen

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

Solana's state bloat problem is best understood through comparison with Ethereum's scaling approach. Solana operates as a **monolithic, single-sharded blockchain**, where all transactions and state updates occur on a single layer.¹ This design keeps all liquidity and data unified, avoiding the fragmentation and complexity of multi-layered networks.¹¹

In contrast, Ethereum pursues a **multi-layered scaling strategy**, relying on Layer 2 (L2) networks like Arbitrum and Optimism to process most transactions off-chain.¹ Only minimal proofs, such as "data blobs," are submitted to the Ethereum mainnet for settlement and security.¹

#### Key Architectural Divergences

- **Solana**: Simplicity through unified, high-throughput layer (1,000+ TPS).
- **Ethereum**: Complexity to offload computational and storage burdens via L2s.
- **Implication**: Solana's design prioritizes speed but amplifies state replication; Ethereum reduces mainnet load but introduces user complexity.

The proposed SIMD-0341 solution, leveraging state compression and off-chain indexing, marks a strategic shift for Solana toward Ethereum's multi-layered model. While pragmatic, this challenges Solana's core ethos of simplicity [Query].

### 2.2. State Growth and Bottlenecks: A Data-Driven Comparison

Architectural differences manifest in state growth metrics:

- **Ethereum's State Growth**: Modest at ~2.62 GiB per month.¹²
- **Solana's State Growth**: Several terabytes per month, an order of magnitude higher.³

#### Reasons for Disparity

- **Transaction Volume**: Ethereum ~30 TPS vs. Solana 1,000+ TPS.
- **Data Offloading**: Ethereum's L2s reduce mainnet burden; Solana replicates all data on-chain.¹¹

Despite differences, both networks confront **perpetual on-chain state growth**. Solana's issue is more acute due to its throughput design. Ethereum's research into "statelessness" and "state expiry" protocols reflects industry-wide recognition of this challenge.¹³ This convergence suggests Solana's solutions could be a high-performance adaptation of shared blockchain principles.

#### Shared Challenges

- Unsustainability of full-state, fully-replicated models at scale.
- Need for innovative storage and archival mechanisms.

## 3. The State of the Art: A Critical Review of Existing State Compression

### 3.1. Technical Mechanics of Concurrent Merkle Trees (cNFTs)

Solana has implemented **state compression** for high-volume use cases like NFTs via **concurrent Merkle trees** — cryptographic structures allowing rapid changes without invalidating proofs.¹⁵ This reduces storage by storing on-chain "fingerprints" (root hashes) while logging uncompressed data to the cheaper ledger state.¹⁶

#### Compression Process

- Metadata hashed and stored in Merkle tree account.
- Full data logged to ledger state for indexing.
- Enables efficient proofs for data integrity.

This yields dramatic savings: A 16,384-node tree costs ~0.222 SOL, minting 1 million compressed NFTs ~$247.80 — far below traditional costs.¹⁵

### 3.2. Successes and Limitations: Why a New Solution is Required

#### Successes

- **Cost Reduction**: Enables projects like DRiP to mint 68 million NFTs for ~$12,000.¹⁵
- **Scalability**: Unlocks abundance-based business models.
- **Adoption**: Lowers barriers for developers and users.

#### Limitations

- **Data Interoperability**: Uncompressed data inaccessible via on-chain CPI calls, breaking composability [Query].
- **Centralization Risk**: Relies on third-party RPC indexers for data serving, with no protocol guarantees for availability or censorship resistance.¹⁹ ²⁰
- **Vendor Lock-In**: Creates dependency on centralized infrastructure.

Existing compression is a **tactical fix** — it addresses costs but introduces new vulnerabilities. A truly enduring solution must preserve decentralization, composability, and censorship resistance without compromising core network principles.

## 4. Architectural Proposal: Adaptive State Management Protocol (ASMP) for Solana

### Executive Summary

The Adaptive State Management Protocol (ASMP) is a comprehensive solution to Solana's state bloat that introduces a **three-tiered hybrid architecture** combining intelligent state transitions, economic incentives, and verifiable off-chain storage while preserving Solana's core principles of speed, composability, and decentralization.

### Architecture Overview

#### Three-Tiered State Model

##### Tier 1: Hot State (On-Chain Active)

- Frequently accessed accounts (accessed within 7 days)
- Full validator replication for instant finality
- Current rent mechanism applies
- Size target: ~200-300 GB

##### Tier 2: Warm State (Hybrid Cache)

- Moderately active accounts (7-90 days since last access)
- Cached by subset of validators using predictive algorithms
- Cryptographic proofs stored on-chain
- Fast reactivation (1-2 block delays)

##### Tier 3: Cold State (Verifiable Off-Chain)

- Dormant accounts (90+ days inactive)
- Distributed across decentralized storage network
- Zero-knowledge proofs for integrity verification
- Economic incentives for storage providers

### Core Innovation: Predictive State Management

#### Intelligent Transition Engine

```typescript
interface StateTransitionPredictor {
  accountUsagePattern: UsagePattern;
  programDependencies: ProgramID[];
  seasonalityFactors: SeasonalData;
  crossAccountRelationships: AccountGraph;
  
  predictNextAccess(): AccessProbability;
  calculateOptimalTier(): StateTier;
  generatePreloadingSchedule(): PreloadSchedule;
}
```

#### Machine Learning-Based Prefetching

- Analyze historical access patterns to predict account usage
- Proactively move accounts between tiers before they're needed
- Reduce latency for "cold start" scenarios
- Optimize validator cache allocation

### Economic Design

#### Dynamic Rent Structure

```typescript
interface AdaptiveRentModel {
  baseTier: StateTier;
  accessFrequency: number;
  dataSize: number;
  strategicValue: number; // For critical infrastructure accounts
  
  calculateRent(): {
    hotStateCost: number;    // High cost, immediate access
    warmStateCost: number;   // Medium cost, cached access  
    coldStateCost: number;   // Low cost, retrieval delays
    transitionFees: number;  // Incentivize proper tier usage
  };
}
```

#### Decentralized Storage Incentives

- **Storage Mining**: Validators earn rewards for reliably storing cold state data
- **Retrieval Rewards**: Bonus emissions for fast data retrieval during reactivation
- **Proof-of-Storage**: Regular cryptographic challenges to verify data availability
- **Slashing Conditions**: Penalties for data unavailability or corruption

### Technical Implementation

#### Verifiable Off-Chain Storage

```typescript
interface ColdStateProof {
  merkleRoot: Hash;           // Root of account data Merkle tree
  zkProof: ZKProof;          // Zero-knowledge proof of data integrity
  storageCommitment: Hash;    // Commitment to distributed storage locations
  replicationFactor: number;  // Minimum copies across storage providers
  
  verifyIntegrity(): boolean;
  generateRetrievalProof(): RetrievalProof;
}
```

#### Composability Preservation

- **Virtual Account Interface**: Programs can call cold-state accounts transparently
- **Asynchronous Execution**: Delayed transaction execution for cold-state access
- **State Preloading**: Predictive loading based on transaction patterns
- **Cross-Program Invocation Buffering**: Queue CPI calls involving cold accounts

#### Reactivation Mechanisms

1. **Instant Reactivation**: Pay premium fee for immediate tier upgrade
2. **Scheduled Reactivation**: Request account warming during next maintenance window
3. **Bulk Reactivation**: Batch multiple accounts for cost efficiency
4. **Smart Reactivation**: Automatic tier adjustment based on usage patterns

### Decentralization Guarantees

#### Distributed Storage Network

- Multiple independent storage providers (Arweave, IPFS, custom protocols)
- Cryptoeconomic incentives prevent single points of failure
- Open participation model for storage providers
- Geographic distribution requirements

#### Validator Diversity

- Lower storage requirements enable more validators to participate
- Specialized roles: Hot-state validators, Warm-state cachers, Cold-state miners
- Dynamic validator sets based on network needs and economic incentives

#### Censorship Resistance

- Redundant storage across multiple providers
- Cryptographic proofs prevent data manipulation
- Economic penalties for censorship attempts
- Community governance for dispute resolution

### Migration Strategy

#### Phase 1: Infrastructure (6 months)

- Deploy three-tiered storage infrastructure
- Implement predictive algorithms and ML models
- Launch testnet with sample applications

#### Phase 2: Economic Integration (4 months)

- Introduce adaptive rent mechanisms
- Deploy storage mining rewards system
- Begin voluntary account migration

#### Phase 3: Protocol Integration (6 months)

- Implement core protocol changes via SIMD process
- Enable automatic state transitions
- Deploy composability preservation features

#### Phase 4: Network-Wide Rollout (6 months)

- Gradual migration of existing accounts
- Monitor performance and adjust parameters
- Full feature activation across mainnet

### Performance Characteristics

#### Expected Improvements

- **Storage Reduction**: 60-80% reduction in live state size
- **Validator Costs**: 40-60% reduction in hardware requirements
- **Network Throughput**: Maintained or improved due to smaller working set
- **Access Latency**:
  - Hot state: Current performance
  - Warm state: 1-2 block delay
  - Cold state: 5-30 second retrieval time

#### User Experience

- Transparent for most applications
- Optional performance hints for developers
- Predictive loading minimizes cold-start delays
- Economic incentives encourage proper state management

### Risk Mitigation

#### Technical Risks

- **Data Loss**: Multiple redundancy layers and cryptographic verification
- **Performance Degradation**: Predictive algorithms and intelligent caching
- **Complexity**: Gradual rollout with extensive testing and monitoring

#### Economic Risks

- **Storage Provider Centralization**: Open market with low barriers to entry
- **Rent Market Manipulation**: Dynamic pricing with circuit breakers
- **Migration Costs**: Subsidized transition periods and batching mechanisms

### Competitive Advantages

This solution addresses the limitations of existing approaches:

- **vs Current State Compression**: Maintains full composability and decentralization
- **vs Simple Archival**: Intelligent transitions reduce user friction
- **vs Pure Off-Chain**: Hybrid model preserves performance for active accounts
- **vs Ethereum's Approach**: Leverages Solana's unique parallel architecture

### Conclusion

The Adaptive State Management Protocol provides a path to sustainable state growth while preserving Solana's core value propositions. By intelligently managing state transitions, creating proper economic incentives, and maintaining decentralization guarantees, ASMP positions Solana to handle massive scale while remaining accessible to validators and developers.

The solution's phased approach allows for careful validation and community feedback, while the hybrid architecture provides flexibility to adapt as the ecosystem evolves. Most importantly, ASMP addresses the root cause of state bloat through protocol-level solutions rather than application-layer workarounds.

## 5. Architectural Proposal II: A Protocol-Level Archival and Expiry System

### 5.1. Technical Blueprint for a Two-Tiered State Model

This proposal builds on existing network behavior to create a **two-tiered state model**:

- **Active State**: Frequently accessed accounts replicated across validators for fast composability.
- **Archival State**: Dormant accounts (inactive >30-90 days) moved to decentralized off-chain storage.

#### Key Mechanisms

- **State Expiry**: Protocol-level rules move dormant accounts to archival state.¹⁴
- **Pointers**: On-chain Merkle proofs attest to off-chain data integrity.²¹
- **Decentralization**: Archival via specialized nodes or protocols like Arweave, incentivized by protocol.

This generalizes compression, decentralizing archival with guarantees.

### 5.2. Economic Incentives for Validators and the State Archival Bounty

To sustain the model:

- **State Archival Bounty**: Portion of emissions compensates archival nodes for storage/serving costs.³
- **Continuous Revenue**: Unlike one-time rent, provides ongoing incentives.
- **Diversification**: Attracts diverse operators, reducing centralization.⁶ Aligns with market-based emissions (e.g., SIMD-0326).²²

### 5.3. Migration Strategy and Backward Compatibility

#### Implementation via SIMD

- Phased rollout to avoid breaking changes.²⁵
- New instructions for state management.
- **Rehydration**: Users provide proof + data to restore dormant accounts on-chain for a fee.

Provides graceful migration, future-proofs against perpetual growth.

## 6. Architectural Proposal III: A Generalized Verifiable Off-Chain Data Protocol

### 6.1. Leveraging ZK-Proofs for On-Chain Integrity

This radical proposal redefines Solana's account model: **all large, mutable data stored off-chain from outset**.

- On-chain accounts hold small, static **ZK-proofs** attesting to off-chain data integrity.²⁶ ²⁸
- Raw data stored on decentralized networks like Arweave or IPFS.²¹
- ZK-proofs enable verification of complex claims (e.g., ownership, transitions) without revealing data.

Makes state bloat obsolete by design.

### 6.2. Redefining Data Interoperability with On-Chain Proofs

Addresses compression's weakness: **full composability via proof verification**.

- Users provide data + ZK-proof as transaction parameters.
- Programs use new CPI calls to verify proofs against on-chain attestations (e.g., ZkE1Gama1Proof program).²⁶
- Cheap verification enables complex cross-program interactions without storing large data.

### 6.3. Developer Tooling and User Experience Enhancements

#### Challenges

- **Complexity**: ZK-proofs and off-chain data require expertise.

#### Solutions

- **SDKs & Tools**: Automate storage, proof generation, transaction construction for seamless developer experience.
- **User Benefits**: Minimal one-time fees, no ongoing rent; experience similar to current model but with eliminated bloat risks.

Ultimate enduring solution: future-proofs by decoupling on-chain state from data.

## 7. A Multi-Criteria Analysis of Proposed Solutions

This section provides a comprehensive comparison of the proposed solutions, evaluating them across multiple criteria to determine the most viable path forward for addressing Solana's state bloat. The analysis considers technical feasibility, economic implications, decentralization guarantees, and implementation timelines to guide strategic decision-making.

### 7.1. The Trade-Off Matrix: Feasibility, Cost, and Decentralization

Objective comparison of solutions:

| Evaluation Criteria          | Current Solana Model      | Existing State Compression | ASMP: Adaptive State Management | Proposal II: Protocol-Level Archival | Proposal III: Verifiable Off-Chain |
|------------------------------|---------------------------|----------------------------|-------------------------------------|-----------------------------------|
| Storage Cost Reduction       | None; all data on-chain.  | High for specific use cases (e.g., cNFTs). | High (60-80% reduction in live state). | High for dormant accounts.          | Extremely high; all large data is off-chain. |
| Validator Hardware Reduction | None; burden is increasing. | Low; validators still store ledger. | High (40-60% reduction in hardware requirements). | High for validators; burden shifts to archival nodes. | Extremely high; on-chain state is minimal. |
| Data Interoperability        | Seamless CPI calls.       | Broken; on-chain programs cannot read data. | Full; preserves composability with virtual interfaces. | Full for active accounts; requires rehydration for dormant. | Full; new CPI primitive is based on proof verification. |
| Data Availability & Reliance  | Fully decentralized.      | Centralized; relies on RPC indexers. | Decentralized; distributed storage network with incentives. | Decentralized; relies on new, incentivized archival network. | Decentralized; relies on protocols like Arweave. |
| Developer Onboarding         | Low; a standard account model. | Medium; requires specific libraries and off-chain indexing. | Medium; new SDKs with intelligent transitions. | Medium; new APIs for state expiry and rehydration. | High; requires new SDKs to abstract ZK-proofs. |
| Implementation Timeline      | N/A                       | Implemented (for NFTs).    | Medium-term (2-3 years) phased rollout. | Medium-term (1-3 years) via SIMD process. | Long-term (3-5+ years) via a new account model. |
| User Experience Impact       | High rent costs.          | Low cost; new centralization risks for data. | Low cost; transparent with predictive loading. | Low cost; requires rehydration transaction for dormant data. | Minimal cost; seamless experience with new SDKs. |

**Key Insights from the Matrix:**

- **ASMP** emerges as the most balanced solution, offering high reductions in storage and hardware costs while maintaining full composability and decentralization.
- **Existing State Compression** provides immediate tactical benefits but falls short on long-term decentralization and interoperability.
- **Proposal II (Archival)** offers strong cost reductions for dormant data but requires rehydration, potentially impacting user experience.
- **Proposal III (Off-Chain)** achieves the highest decentralization and cost efficiency but demands significant developer tooling investments and has the longest timeline.

### 7.2. Short-Term Gains vs. Long-Term Resilience

#### Distinctions

- **Existing Compression**: Tactical fix with immediate cost savings but compromises decentralization and composability.
- **ASMP**: Comprehensive solution with three-tiered architecture, predictive management, and economic incentives.
- **Proposal II**: Strategic archival system with incentives.
- **Proposal III**: Ultimate fix, solving bloat permanently via ZK-proofs and off-chain data.

#### Path Forward

- Leverage compression for high-volume, low-value use cases.
- Prioritize ASMP via phased rollout for comprehensive resilience.
- Research Proposal III for long-term future-proofing.

### 7.3. The Path to Implementation via the SIMD Governance Process

Major changes require **SIMD process**: collaborative proposal, discussion, validator vote.²⁵

#### For ASMP

- Implement via phased rollout as outlined in the ASMP migration strategy.
- Submit to solana-improvement-documents repo for review.
- Adopt via two-thirds validator supermajority.²⁵

#### Governance Considerations

- **Community Consensus**: Requires broad stakeholder agreement on state management principles.
- **Validator Incentives**: Aligns with existing emission mechanisms to ensure participation.
- **Backward Compatibility**: Ensures seamless migration for existing applications and users.

### 7.4. Recommended Implementation Strategy

Based on the multi-criteria analysis, ASMP is recommended as the primary solution due to its comprehensive approach that balances immediate benefits with long-term sustainability. The phased rollout minimizes disruption while providing progressive improvements in scalability and cost-efficiency.

## 8. Conclusion and Final Recommendations

### 8.1. Summary of Key Findings

Solana's account model creates **accelerating state bloat** with systemic implications:

- Unpruned ledger >400 TB, growing petabytes/year.²
- High validator costs ($15K-$50K upfront, $500-$1K/month) create centralization barriers.⁶
- Rent system imperfect; existing compression tactical but compromises composability.

### 8.2. Strategic Recommendations

#### Immediate Actions

1. **Acknowledge Compression's Role**: Value for niche, high-volume apps (e.g., NFTs) but recognize as tactical fix.
2. **Prioritize ASMP**: Comprehensive three-tiered solution via phased rollout; addresses root causes with intelligent transitions and economic incentives.
3. **Leverage Existing Solutions**: Use archival and off-chain protocols for complementary approaches.
4. **Initiate Long-Term Research**: Explore advanced ZK-proofs for future-proofing.

#### Multi-Phased Approach

- **Short-Term**: Deploy ASMP infrastructure and predictive algorithms.
- **Medium-Term**: Implement ASMP phases (2-3 years).
- **Long-Term**: Integrate Proposal III for ultimate bloat elimination (3-5+ years).

### 8.3. Call to Action

At a critical inflection point, Solana must address state bloat to capture tokenization opportunities.³⁰ A clear, phased strategy secures high-performance, decentralized scalability.

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
