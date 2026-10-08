# Open Tokenized Asset Standard (OTAS)

## Composable Token Primitives for Post-Trade Clearing, Settlement, and Compliance on Distributed Ledgers

**A Community Whitepaper (Draft for Review)**

Prepared as a contribution to the OTAS Lab under LF Decentralized Trust (LFDT)

---

> **Disclaimer**
>
> This is a draft community whitepaper published by the OTAS Lab under LF Decentralized Trust (LFDT) for review and feedback; its contents are subject to change without notice. It is provided "as is" for informational purposes only and does not constitute legal, regulatory, financial, or investment advice. The views expressed are those of individual contributors and do not necessarily represent their employers, LFDT, or the Linux Foundation. References to third-party projects, standards, or products are for comparison only and imply no endorsement; all trademarks belong to their respective owners. Nothing herein is an offer or recommendation regarding any security or digital asset, and the models described are conceptual proposals, not finished specifications.

---

## Table of Contents

1. Executive Summary
2. Introduction and Motivation
3. The Post-Trade Problem Space
4. A Comparative Survey of Existing Token Standards
5. Toward Composable Token Primitives: A Convergence Model
6. The Four Layers in Depth
   - 6.1 Settlement Networks
   - 6.2 Identity
   - 6.3 Compliance
   - 6.4 Asset Metadata
   - 6.5 Two Cross-Cutting Concerns: Lifecycle and Privacy
7. Issuer Archetypes: Banks, Financial Firms, and Sovereigns
8. Reference Flows
9. Relationship to Prior and Academic Work
10. Appendices (Glossary, Standards Index, How to Contribute)
11. References

---

## 1. Executive Summary

Tokenization has moved from pilots into production. Tokenized money market funds, deposits, repos, and sovereign bonds now exist across multiple distributed ledgers, and several settlement initiatives have demonstrated atomic delivery-versus-payment in central-bank and commercial-bank money. Yet the infrastructure remains fragmented. The same bond, tokenized by five institutions, becomes five incompatible objects on five ledgers, each with its own token schema, identity model, compliance enforcement, and metadata conventions. Liquidity is constrained, processes are duplicated, and final settlement still leans on legacy coordination layers.

This whitepaper explores whether a small set of **composable token primitives** (protocol-agnostic interface conventions rather than a new chain or a new monolithic token) can serve as a convergence layer for post-trade clearing, settlement, and compliance. The guiding heuristic, voiced repeatedly across the OTAS community, is to standardize the **lowest common denominator required for a shared function** (settlement, identity portability, collateral transfer, compliance signaling) and to deliberately not standardize the internal representation of the asset, which is legitimately institution- and jurisdiction-specific.

The paper does four things:

- **Surveys** the major on-chain token standards: the EVM/Ethereum family (ERC-20, ERC-721, ERC-1155, ERC-1400, ERC-3643/T-REX, ERC-4626), Sui's Move object model, Solana's SPL and Token-2022 extensions, permissioned enterprise models (Hyperledger Fabric Token SDK / Panurus, Besu, Corda), and the implementation-neutral InterWork Alliance Token Taxonomy Framework. It identifies where they converge and where they diverge.
- **Proposes** a primitive model organized as a token base type + composable behaviors + event schema, expressible across heterogeneous execution environments.
- **Develops the four layers** the OTAS community has prioritized (**Settlement Networks, Identity, Compliance, and Asset Metadata**) and adds two cross-cutting concerns (lifecycle/asset servicing and privacy).
- **Compares issuer archetypes** (banks, asset managers and other financial firms, and sovereigns/central banks), because the trust anchor behind a token (central-bank money, commercial-bank liability, fund NAV, sovereign credit) materially changes what a standard must accommodate.

The intended contribution is not a finished specification but a shared vocabulary and a set of open questions — tracked as issues in the project repository — that the community can converge on.

## 2. Introduction and Motivation

### 2.1 The fragmentation thesis

Global post-trade settlement infrastructure is fragmented along three axes at once:

1. **Incompatible ledger implementations:** account/balance models versus UTXO models versus object models; permissioned versus permissionless; differing finality and privacy guarantees.
2. **Jurisdictional compliance regimes:** FATF guidance and the Travel Rule, MiCA in the EU, the GENIUS Act and BSA/AML regime in the US, securities law, and Basel treatment of crypto exposures, each with different data-handling constraints.
3. **Asset-class-specific token schemas:** a tokenized deposit, a tokenized treasury fund share, a tokenized sovereign bond, and an IoT-linked asset each carry different lifecycle events, metadata, and control requirements.

The result is systemic friction precisely at the seam between programmable finance and institutional market microstructure. Each institution has, reasonably, built governed infrastructure for its own assets; the missing piece is a **common language at the boundaries between those systems**.

### 2.2 What an open standard is and is not for

A recurring framing in this space is the analogy to foundational protocols: FIX standardized trade negotiation, SWIFT standardized cross-institution messaging and coordination, and TCP/IP standardized how heterogeneous networks interoperate without forcing anyone to rebuild. The post-trade layer for tokenized assets has no equivalent. OTAS positions itself as an interface layer: a common language that governs the compliance and settlement seams between institutions that already operate their own governed infrastructure, rather than as a blockchain or a competing platform.

This whitepaper therefore treats the standard as **minimal by design**. \*Reference implementations are intended to demonstrate the interface, not to replicate the platform value (compliance lifecycle management, metadata propagation, multi-rail orchestration, institutional reporting) that production systems legitimately provide and differentiate on.

\* Reference implementations are a planned OTAS Lab deliverable; they do not yet exist at the time of writing.

### 2.3 Scope of this document

In scope: a comparative survey, a proposed primitive/behavior/event model, the four prioritized layers, an issuer-archetype comparison, reference flows, and open questions. Out of scope (consistent with the OTAS Lab charter): compliance workflow engines, metadata propagation logic, regulatory reporting pipelines, lifecycle orchestration, and platform integrations. Those are the value layer left to implementers. The illustrative reference environments referenced throughout are Ethereum/EVM and Sui, with Hyperledger Fabric and Besu noted as permissioned execution environments of interest; the intent is protocol-neutrality, not exclusivity.

## 3. The Post-Trade Problem Space

### 3.1 The post-trade lifecycle

"Post-trade" begins after a trade is agreed and spans:

- **Clearing:** confirming, matching, and novating obligations; computing net positions; managing counterparty risk (often via a central counterparty).
- **Settlement:** the final, irrevocable exchange of the asset for cash (delivery-versus-payment, "DvP") or of one currency for another (payment-versus-payment, "PvP").
- **Custody and safekeeping:** holding the asset and the authoritative record of ownership.
- **Asset servicing:** coupon and dividend payments, corporate actions, redemptions, maturity, tax processing, and reporting throughout the asset's life.

### 3.2 The frictions tokenization targets

Conventional settlement operates on multi-day cycles, time-zoned operating hours, and "give-before-you-get" exposure, with reconciliation across many intermediaries. DvP on a shared ledger can be **atomic**, meaning the asset and the cash leg either both settle or neither does, which eliminates or sharply reduces principal (Herstatt-type) risk by conditioning one final transfer on the other. Real-time, potentially 24/7 settlement also reshapes intraday liquidity: instant settlement can reduce the need for large buffers, while synchronized incoming/outgoing legs reduce funds sitting "in limbo." These are not theoretical: multiple wholesale initiatives are explicitly organized around making atomic settlement operational (Section 6.1).

### 3.3 Why the silos persist

The benefits above are mostly realized within a single ledger or consortium. The hard part is **between** them. Atomicity is naturally a single-ledger property; spanning two ledgers requires either a shared platform, a trusted coordinator, or a cross-chain protocol (hash-time-locks, escrow/notary schemes, or messaging bridges), each with its own trust and finality assumptions. And because each ledger encodes ownership, identity, and compliance differently, an asset cannot simply "move"; it must be re-represented. A convergence layer must therefore reconcile not just data formats but **execution-model differences** (Section 4) and **trust models** (Section 7).

## 4. A Comparative Survey of Existing Token Standards

This section surveys the principal on-chain token standards as the raw material for a convergence layer. The aim is a gap analysis, not a ranking; each standard is well-adapted to its environment.

### 4.1 The Ethereum / EVM family

The EVM ecosystem standardizes token interfaces via ERC specifications layered over an account/balance state model.

- **ERC-20 (fungible).** The baseline interface: `balanceOf`, `transfer`, `approve`, `transferFrom`, plus `Transfer`/`Approval` events. It is permissionless and unrestricted by design: it has no concept of eligibility, jurisdiction, or transfer rules. Enormous network effects and DeFi composability flow from it, but it cannot, alone, express the controls that regulated assets require.
- **ERC-721 (non-fungible).** Pairs a globally unique token ID with the contract address to represent unique items; the basis for representing uniquely identifiable instruments and certificates.
- **ERC-1155 (multi-token).** A single contract managing many fungible and non-fungible token types with batch operations, efficient for collections of related instruments.
- **ERC-1400 (security token).** Introduces partitions (tranches) and an off-chain-validation pattern: a transfer is checked via functions such as `canTransferByPartition`, which typically consults an external controller or oracle. It prioritizes operational flexibility and a familiar investor experience by keeping much eligibility logic off-chain.
- **ERC-3643 / T-REX (permissioned token).** An ERC-20-compatible suite that embeds compliance and identity enforcement on-chain. Transfers succeed only when both the parties' eligibility (via an on-chain identity, ONCHAINID) and the offering's global compliance rules pass. Core components include an **Identity Registry**, a **Trusted Issuers Registry**, a **Claim Topics Registry**, and a modular **Compliance** contract; the standard specifies `isVerified` (party eligibility) and `canTransfer` (global rules such as max-holders or per-investor caps), plus operational controls: forced transfers, partial/total freeze, pause, key-recovery, and batch operations. ONCHAINID itself builds on the ERC-734/735 key-and-claim model and stores references and hashes rather than raw PII. ERC-3643 reached Final EIP status and is stewarded by the ERC-3643 Association; it is the most widely adopted on-chain compliance pattern for regulated real-world assets.
- **ERC-4626 (tokenized vault).** Standardizes deposit/withdraw/share-accounting for yield-bearing vaults, relevant to tokenized funds and structured products.

**Takeaway.** The EVM family shows a clear progression from unrestricted fungibility (ERC-20) to on-chain, identity-bound compliance (ERC-3643). The two dominant security-token approaches differ primarily on **where eligibility is decided**: off-chain validation (ERC-1400) versus on-chain registries (ERC-3643). Both remain ERC-20-compatible, which is the practical anchor for interoperability.

### 4.2 Sui: the Move object model

Sui represents assets as **typed objects**, each with a unique ID, rather than as balances in a contract. Because objects are inherently unique, Sui needs no ERC-721 analogue. Its fungible-asset standards, and the mechanisms layered over them, are:

- **Coin\<T\> / Currency (open-loop).** Freely transferable, wrappable, storable fungible tokens, with metadata managed through a **Coin Registry** (a shared system object) that unifies metadata, supply tracking, and regulatory status. Supply models can be **fixed, burn-only, or uncontrolled**.
- **Address Balances (account-style accrual over the object model).** A canonical balance per `(address, coin type)` pair, addressing the operational friction of treating fungible value as a set of discrete `Coin<T>` objects that must be split before spending and merged after receipt. Funds sent with `coin::send_funds` or `balance::send_funds` accrue automatically into a single balance for that address and type, and are drawn back out as objects via `redeem_funds` when a call site needs a `Coin<T>`. Because the balance is addressable directly, gas can be paid from it with an empty gas-payment list. Address balances do not replace the object model; they are an optional account-style layer over it, and the two coexist for the same coin type.
- **Permissioned Asset Standard (PAS).** Sui's standard for assets whose movement must be conditioned on issuer or third-party approval, and the current successor to the closed-loop token for regulated and institutionally held assets. Ownership sits in a shared **`Account`** object (one per address, created permissionlessly, deterministically derived from a system **`Namespace`**), and each managed asset type carries a **`Policy<T>`** declaring which approvals each action requires across `send_funds`, `unlock_funds`, and `clawback_funds`. Every state change follows a **create -> approve -> resolve** request pattern: the account emits a `Request<SendFunds<T>>` (or the clawback/unlock equivalent), one or more packages stamp it via `request.approve(MyWitness())`, and resolution succeeds only when the collected approval witnesses, matched by `TypeName`, are exactly those the policy names. The request is a hot potato, so it must resolve within the same transaction or the transaction aborts. Two properties matter for a convergence layer. First, the guarantee is genuinely **closed-loop**: a managed asset cannot leave the system except through a request with matching approvals, which is why enabling `unlock_funds` is explicitly the escape hatch that voids all future transfer control. Second, PAS deliberately **does not decide policy**. It standardizes the approval-gating mechanism and leaves eligibility logic to the issuer's approval-witness code, while `clawback_funds` provides a native seizure/forced-transfer path gated by that same policy.
- **Token\<T\> (closed-loop).** The earlier restricted type (it has the `key` ability but not `store`), so it cannot be freely transferred, wrapped, or stored in arbitrary applications. Its permitted actions are governed by a **TokenPolicy** that attaches per-action rules (allow-list, deny-list, spend limits, verification requirements) and resolves each action via an `ActionRequest`. Its enduring fit is **isolated in-app value**: service-specific currencies, loyalty and reward points, and similar balances that never need to circulate beyond a single application's perimeter. For regulated assets that must interoperate with external counterparties, PAS supersedes it.

Control is **capability-based**: a `TreasuryCap` authorizes mint/burn and, under PAS, the one-time creation of a `Policy<T>` for a currency; a `PolicyCap<T>` authorizes changing which approvals an action requires; a `DenyCapV2` plus the system **DenyList** object enables regulated coins to block specific addresses and to trigger a **global pause**; a `MetadataCap` governs metadata mutability. Capabilities are themselves transferable objects, which makes delegation of issuance/administration explicit and auditable, but also means capability custody is security-critical.


### 4.3 Solana: SPL and Token-2022 extensions

Solana's original SPL Token program covers mint/transfer/burn. **Token-2022 (Token Extensions)** is a superset that adds optional, composable features configured at mint or account creation, including:

- **Confidential transfers:** ZK-encrypted balances and amounts.
- **Transfer hooks:** a program invoked on every transfer for custom validation (allow-lists, royalties, rules).
- **Permanent delegate:** an authority that can transfer or burn from any account of the mint, used for compliance clawback/freeze (e.g., regulated stablecoin issuers).
- **Non-transferable ("soulbound") tokens:** for credentials and identity-bound items.
- **Interest-bearing, transfer-fee, default-account-state, memo-required, pausable, permissioned-burn, metadata-pointer/native-metadata,** and **CPI-guard** extensions.

Two design properties matter for a convergence layer. First, **most extensions are immutable after creation**, so token design must be planned up front. Second, **some extensions are mutually exclusive** (for example, confidential transfers and transfer hooks cannot be combined, because a hook that reads the amount is incompatible with an encrypted amount). Solana thus pushes traditionally contract-level logic down into the asset itself as native, declarative features.

### 4.4 Permissioned and enterprise models

- **Hyperledger Fabric Token SDK (now evolving into the Panurus project).** A UTXO-based token model on permissioned Fabric: token transactions form a directed acyclic graph where each transaction consumes unspent outputs and creates new ones, and wallets track unspent outputs. It emphasizes **privacy** (selective disclosure / zero-knowledge techniques), **performance** at institutional scale, and **compliance/auditability**, the triad most relevant to regulated deployments.
- **Hyperledger Besu.** An Ethereum-compatible client that can run in permissioned configurations, allowing EVM/ERC tooling to be reused inside a controlled network, a natural bridge between public-chain standards and enterprise governance.
- **R3 Corda.** A UTXO-style model with a mature tokens SDK, widely used in regulated settlement contexts.

**Takeaway.** Permissioned environments often favor **UTXO/object-style** accounting (better for privacy, parallelism, and per-instance properties) over the account/balance model, reinforcing that a convergence layer must abstract over both accounting models.

### 4.5 Implementation-neutral meta-frameworks: the Token Taxonomy Framework

The **InterWork Alliance Token Taxonomy Framework (TTF)**, an initiative now under the Global Blockchain Business Council and originally incubated around the Enterprise Ethereum Alliance, is the most directly relevant prior art for composable primitives. It is an implementation-neutral meta-model that decomposes any token into:

- a **Token Base** (e.g., fungible vs. non-fungible; and "common" account/balance representation vs. "unique" UTXO representation),
- a set of **Behaviors** (divisible, transferable, mintable, burnable, pausable, delegable, and so on), and
- one or more **Property Sets** (metadata and data attributes),

combined into a shorthand **"Token Formula."** The TTF's explicit goal is a common language usable by technical and non-technical stakeholders and mappable to Ethereum, Fabric, Corda, and others. Its base-type distinction (account/balance vs. UTXO) and its behavior/property decomposition are a strong foundation that OTAS can build on and modernize for post-trade.

### 4.6 Comparative synthesis

| Dimension | Ethereum / EVM | Sui (Move) | Solana (Token-2022) | Fabric Token SDK / Panurus | TTF (meta-model) |
|:-:|:-:|:-:|:-:|:-:|:-:|
| **State model** | Account/balance | Object (typed, unique-ID); optional address balances | Account (mint + token accounts) | UTXO (DAG) | Abstracts both |
| **Fungible primitive** | ERC-20 | Coin\<T\>/Currency | SPL mint | UTXO token | Token Base (common) |
| **Unique primitive** | ERC-721/1155 | Native objects | NFT via metadata | UTXO token | Token Base (unique) |
| **Where compliance lives** | Registries + modular compliance (ERC-3643) | **PAS** `Policy<T>` + approval witnesses (legacy: TokenPolicy) | Mint **extensions** + transfer hooks | Chaincode + selective disclosure | Behaviors / property sets |
| **Identity binding** | ONCHAINID (ERC-734/735) | External, checked inside PAS approval witnesses | Account state / external | Org identity (MSP) | Property set |
| **Admin control** | Owner/Agent roles, freeze, forced transfer | Treasury/Policy/Deny caps, PAS clawback, global pause | Permanent delegate, pause | Issuer keys | "controllable/pausable" behaviors |
| **Privacy of amounts** | Public (or L2/ZK) | Public (or app-level) | Confidential transfers (ZK) | Native selective disclosure | Out of model |
| **Composability anchor** | ERC-20 compatibility | Capabilities, policies, approval requests | Extension stacking | UTXO graph | Token Formula |

**Gap analysis.** Across all of them, four functions recur but are expressed incompatibly: (1) who may hold/transfer (identity + eligibility), (2) under what rules (compliance), (3) how value finally changes hands (settlement/atomicity), and (4) what the token represents and how it is serviced (metadata + lifecycle). These four recurring functions are exactly the OTAS layers, and they are the right altitude for standardization: **the function is mutual across institutions; the internal asset representation is not.**

## 5. Toward Composable Token Primitives: A Convergence Model

### 5.1 Design principles

1. **Standardize functions, not assets.** Define the minimal interface for a shared function (settle, prove-eligibility, signal-compliance, describe-asset); leave internal representation to issuers.
2. **Protocol-agnostic.** Every primitive must be expressible in at least the account/balance, object, and UTXO models. The standard is an interface contract, not bytecode.
3. **Composable, not monolithic.** Favor small primitives that combine (TTF-style base + behaviors) over a single all-encompassing token, so issuers adopt only what they need.
4. **Capability- and event-oriented.** Make administrative power explicit (who can mint, freeze, force-transfer) and emit a **canonical event schema** so that off-chain systems on any rail can reconcile uniformly.
5. **Privacy-compatible.** The model must degrade gracefully where amounts/identities are encrypted (ZK), since regulated deployments will require it.
6. **No PII on-chain.** Identity and compliance primitives carry references, hashes, and verifiable attestations, never raw personal data.

### 5.2 A primitive / behavior / event sketch (for community refinement)

This is intentionally a strawman to be torn apart in review, expressed as neutral pseudo-interfaces.

**Token base types**

- `FungibleBase`: interchangeable units; account or UTXO representation.
- `UniqueBase`: uniquely identifiable instrument/instance.
- `FractionalUniqueBase`: a unique instrument with fungible fractional claims (common for bonds and funds).

**Composable behaviors** (each optional, each independently discoverable)

- Supply: `Mintable`, `Burnable`, `FixedSupply`, `BurnOnly`.
- Control: `Pausable`, `Freezable(account)`, `ForceTransferable` (issuer/agent), `Recoverable` (key loss).
- Eligibility: `Restricted`, meaning a transfer succeeds only if an attached **Eligibility Oracle** (see 6.2/6.3) returns true for both parties and the action.
- Settlement: `AtomicLeg`, letting the token participate as one leg of an all-or-nothing settlement instruction (see 6.1).
- Servicing: `IncomeBearing` (coupons/dividends/rebasing), `Redeemable`, `Maturing`, `CorporateActionable`.
- Confidential: `ConfidentialAmount`, with amounts encrypted while eligibility remains provable via ZK.

**Canonical event schema** (rail-neutral, so any indexer can reconcile)

- `Issued`, `Transferred`, `Burned`, `Frozen`/`Unfrozen`, `ForcedTransfer`, `Paused`/`Unpaused`, `EligibilityChecked(result)`, `SettlementProposed`/`Committed`/`Settled`/`Aborted`, `ServicingEvent(type)`, `MetadataAnchored(hash)`.

**Discovery.** A token should expose a machine-readable capability/behavior manifest (which behaviors it implements, which policy/registry it points to, which metadata anchor it uses), so a counterparty's system can determine compatibility before transacting, analogous to a Sui PAS `Policy<T>`, whose required approvals are readable before a request is submitted, and to ERC-3643's pre-trade `canTransfer` check.

### 5.3 Mapping the model onto environments

| Primitive/behavior | EVM expression | Sui expression | Solana expression | Fabric/UTXO expression |
|:-:|:-:|:-:|:-:|:-:|
| **FungibleBase** | ERC-20 | Coin\<T\>/Currency (object or address balance) | SPL mint | UTXO token |
| **Restricted** | ERC-3643 canTransfer/isVerified | PAS `Policy<T>` + approval witness | Transfer hook | Chaincode endorsement |
| **Freezable** | Agent freeze | DenyList + DenyCap; withheld PAS approval | Default-state / permanent delegate | Issuer policy |
| **ForceTransferable** | ERC-3643 forced transfer | PAS `clawback_funds` | Permanent delegate | Issuer-signed tx |
| **ConfidentialAmount** | L2s/ZK | App-level/ZK | Confidential transfer ext. | Native selective disclosure |
| **MetadataAnchored** | Token URI / registry | Coin Registry / object field | Metadata pointer | On-chain ref + off-chain store |

The point of the table is that **the same function has a home in every environment**, which is precisely what makes a thin interface standard feasible rather than utopian.

### 5.4 Modeling an existing token as an OTAS object: native vs. observed

The primitives in Section 5.2 describe a token as a base type plus behaviors plus a capability manifest. A token already deployed on a rail does not necessarily carry those OTAS labels, so before reference flows like those in Section 8 can treat it uniformly, its native facts have to be read in OTAS terms somehow. It may help to picture this as a spectrum, with two ends worth naming:

- **Native mapping.** The token is issued to conform, with its base type, behaviors, and capability manifest declared against OTAS conventions from the start (for example, an ERC-3643 deployment, or a Sui PAS `Policy<T>` whose required approvals are published as the token's compliance behavior). Here the OTAS description tends to originate with the issuer.
- **Observed mapping.** The token predates or ignores OTAS, and tooling infers a description by reading on-chain facts (interfaces implemented, registries and policies referenced, extensions enabled, capability objects held). Such a description is only as reliable as what the chain exposes: some properties are directly observable, others may not be confirmable from on-chain state alone.

Most real tokens will likely sit somewhere between these ends. Where a given case falls, and how much corroboration an observed description needs before a counterparty relies on it, is left open here. For the flows that follow, the point is only that they operate on the same OTAS object either way; what differs is where its description came from.

## 6. The Four Layers in Depth

The OTAS community has prioritized four layers. We treat each as a function to be standardized at its lowest common denominator, and we add two cross-cutting concerns.

### 6.1 Settlement Networks

**What it is.** The settlement layer answers: how does final, irrevocable value transfer happen, and against what settlement asset? The defining capability is **atomic settlement** (DvP for securities-vs-cash and PvP for currency-vs-currency), where linked legs either all complete or all revert.

**The settlement-asset question is primary.** Atomicity is only as strong as the cash leg. The same DvP is materially different depending on whether the cash token is:

- **Wholesale central-bank money / tokenized reserves (wCBDC):** the highest-quality settlement asset, carrying no commercial-bank credit risk (a direct central-bank liability).
- **Tokenized commercial-bank deposits:** programmable commercial-bank money; carries the issuing bank's credit risk but offers regulatory familiarity and is where much institutional DvP is expected to settle.
- **Regulated stablecoins / e-money tokens:** useful where structurally appropriate, with quality depending on backing and redemption mechanics.

**The landscape (illustrative).** A two-tier structure is emerging: a central-bank settlement layer (tokenized reserves) beneath a commercial-bank layer (tokenized deposits). Notable initiatives include the BIS **Project Agorá** (a unified-ledger prototype combining tokenized commercial-bank deposits with tokenized central-bank reserves for atomic, multi-currency cross-border settlement, convened with multiple central banks and 40+ financial institutions), earlier BIS/central-bank work such as **Helvetia**, **Jura**, **mBridge**, and **Dunbar**, and private/consortium rails such as **Partior**, **Fnality**, the **Regulated Liability/Settlement Network (RLN/RSN)**, **SIX Digital Exchange (SDX)** (where CHF tokens backed one-to-one by central-bank reserves settle tokenized securities), **Euroclear's D-FMI**, **DTCC's** digital efforts, and **J.P. Morgan's Kinexys** (deposit-token and repo settlement). Interoperability providers (e.g., Ownera) aim to connect these rails.

**What a standard should (and should not) do here.** It should define a **rail-neutral settlement instruction primitive**: an `AtomicLeg` that can be referenced by a settlement coordinator on any rail, with a canonical settlement event lifecycle, an explicit declaration of the settlement asset and its trust tier, and hooks for cross-ledger atomicity patterns (HTLC, escrow/notary, shared-ledger). It should **not** prescribe a single settlement venue, a single cash token, or a single cross-chain mechanism. The convergence value is a common description of a settlement leg that lets a tokenized bond on one rail and a tokenized deposit on another be coordinated without either party migrating infrastructure.

**Finality: technical and legal.** Atomic settlement, as described above, delivers **technical finality**: the ledger-level guarantee that linked legs either all complete or all revert. This is worth separating from **legal finality**: the point at which settlement is irrevocable and unconditional, good against an insolvent participant and recognized under the applicable legal framework. The two are not necessarily the same moment. A settlement system's rules should distinguish the point after which an instruction may no longer be revoked from the point at which settlement is final. Legal finality is a legally defined moment that depends on the governing jurisdiction and its insolvency law, is often established by a reasoned legal opinion, and is rarely harmonized across borders.

Technical finality is itself not uniform across rails. Some ledgers offer deterministic, single-block finality, while others are probabilistic, where a settled-looking transfer carries a residual reorganization risk that decays over time. A standard that speaks to settlement should therefore let both the finality model and the finality moment be declared, rather than assuming atomicity delivers either on its own.

To give these distinctions somewhere to attach, the settlement lifecycle can carry an explicit `Committed` state between proposal and final settlement: `SettlementProposed -> Committed -> Settled | Aborted`. Once the coordinator holds both legs' states, the instruction moves to `Committed`, meaning it is enqueued, will not be lost, and can no longer be revoked by a participant. The revocability boundary sits at the transition into `Committed`, which is a natural place for a rulebook to anchor legal finality; final settlement is then marked by `Settled`. Whether an instruction may still `Abort` after `Committed`, for example on an exceptional technical failure, is left to the rulebook. In all cases OTAS would provide the construct to express which moment governs and under which finality model, not the legal determination itself, which remains the operator's and the jurisdiction's responsibility.

**Open tension.** Instant atomic settlement requires pre-funding of both legs, which can strain liquidity in fragmented markets; the standard should make liquidity-saving mechanisms (netting windows, conditional/queued settlement) describable, not impossible.

### 6.2 Identity

**What it is.** The identity layer answers: who is the holder, and how is that proven across institutions without exposing personal data? Putting PII on-chain is a non-starter under GDPR and is in tension with BSA/AML data-handling expectations. The consensus direction is **portable, privacy-preserving credentials**, not on-chain identity databases.

**The building blocks.**

- **W3C Decentralized Identifiers (DIDs) v1.0:** subject-controlled identifiers that resolve to a DID Document (keys + service endpoints); method variety includes did:web, did:ethr, did:key, did:ion.
- **W3C Verifiable Credentials (VC) Data Model:** tamper-evident, cryptographically signed attestations (e.g., KYC status, accreditation, jurisdiction, beneficial-ownership), exchanged among an **Issuer -> Holder -> Verifier** triangle, supporting **selective disclosure** and **zero-knowledge presentations** (prove "over-18" or "accredited, EU" as a boolean without revealing the underlying data).
- **On-chain identity binding:** exemplified by **ONCHAINID** (ERC-734/735), which links wallets to an identity contract holding hashed references to claims from trusted issuers, queried at transfer time.
- **Bridging profiles:** OpenID4VC/OpenID4VP, Trust over IP (ToIP) layers, and **eIDAS 2.0 / EU Digital Identity Wallet**, which is bringing VCs into regulated workflows.

**What a standard should do here.** Define **reference data primitives and verifiable interfaces** so that an eligibility credential issued by one institution is readable and checkable by another, without exposing PII. Concretely: a canonical eligibility attestation schema (claim topics such as KYC-complete, jurisdiction, investor-class, sanctions-clear), a trusted-issuer discovery mechanism, and a verification interface (`isEligible(party, claimTopics) -> bool/proof`) that a `Restricted` token's transfer can call. The standard carries hashes/credential references and verification logic, never the credential contents.

**Why portability is the whole point.** Reusable KYC only scales if a receiving institution can verify a million presentations against a trust list without re-collecting documents. FATF guidance has moved toward accepting reusable, credential-based identity with appropriate assurance, which makes a portable attestation interface both useful and defensible.

### 6.3 Compliance

**What it is.** The compliance layer answers: under what rules may this specific transfer proceed, in this jurisdiction, for these parties? Where identity establishes who, compliance establishes whether this action is permitted now. The two are distinct: ERC-3643's split between `isVerified` (party eligibility) and `canTransfer` (global offering rules) is a useful conceptual template.

**The constraint surface.** A workable standard must be satisfiable under, at minimum, FATF member-state requirements and the **Travel Rule**, **MiCA** and the EU **AMLR** (with MiCA's transitional period concluding mid-2026), US **BSA/AML** and the **GENIUS Act** stablecoin framework, applicable **securities law**, and **Basel** treatment, without imposing a single internal architecture on any participant.

**Where compliance executes: three patterns observed.**

- **Registry + modular compliance contracts** (ERC-3643): rules are on-chain modules evaluated at transfer.
- **Per-action policy plus typed approvals** (Sui PAS): a `Policy<T>` declares which approval witnesses each action requires, and a request resolves only when the witnesses collected match that set exactly.
- **Transfer-time program hooks** (Solana transfer hooks; Fabric endorsement): custom logic invoked on movement.

**What a standard should do here.** Standardize the **compliance-signaling interface and result semantics**, not the rule engine. Concretely: a `canTransfer(from, to, amount, context) -> {allow | deny + reason-code}` contract with **standard, machine-readable reason codes** (e.g., `INELIGIBLE_RECEIVER`, `JURISDICTION_BLOCKED`, `HOLDER_CAP_EXCEEDED`, `LOCKUP_ACTIVE`, `SANCTIONS_HIT`, `TRAVEL_RULE_PENDING`), a **pre-settlement check** so failures are knowable before a settlement instruction is submitted, and a **Travel-Rule handshake hook** that references an off-chain VASP-to-VASP exchange (e.g., TRP/TAP-style protocols) rather than embedding identity data on-chain. Privacy-enhancing techniques (selective disclosure, ZK) should be first-class, so compliance can be proven without data exposure. The rules themselves, and how aggressively they are enforced, remain the implementer's value layer and jurisdictional responsibility.

Here `canTransfer` names an OTAS interface convention, not the ERC-3643 function of the same name. It fixes the query shape and the result semantics (allow, or deny with a reason code); enforcement binds to whichever native mechanism the rail provides, reusing the three patterns above: ERC-3643 registries and modular compliance on EVM, a Sui PAS `Policy<T>` resolved by approval witnesses, or a Solana transfer hook. The intended contribution is the common result vocabulary across those mechanisms, so a counterparty on any rail reads the same allow/deny plus reason code.

### 6.4 Asset Metadata

**What it is.** The metadata layer answers: what does this token represent, what are its terms, and how is that information kept authoritative and discoverable across rails? This is where the temptation to over-standardize is greatest and should be resisted: the envelope can be common even where the contents are asset- and jurisdiction-specific.

**A layered metadata model (proposed).**

- **Core descriptor (common):** asset class, issuer identity/DID, instrument identifier(s) (ISIN/CUSIP/FIGI where applicable), currency/denomination, decimals, jurisdiction, legal-wrapper reference, and the **capability/behavior manifest** (Section 5.2).
- **Lifecycle/terms (semi-common):** for debt, coupon schedule, day-count, maturity, redemption terms; for funds, NAV source, distribution policy; for IoT-linked assets, the referenced device/oracle and update cadence.
- **Extension (issuer-defined):** anything proprietary, carried as a typed extension so it does not break common parsers.

**Anchoring and authority.** On-chain expressions differ (EVM token URIs/registries, Solana's **metadata pointer / native metadata**, Sui's **Coin Registry** and object fields), so the standard should specify a **content-addressed anchor** (a hash/URI on-chain pointing to an off-chain canonical record) plus a **provenance/attestation** mechanism (who asserted this metadata, signed how) and an **update/event** convention (`MetadataAnchored(hash)`), so that consumers on any rail can verify authenticity and detect changes. The off-chain record format should be aligned with established financial data models where they exist (e.g., FIX/ISO 20022 semantics, FpML for derivatives terms) rather than inventing parallel vocabularies.

**Why this matters for convergence.** If two institutions tokenize the same bond, a shared core descriptor + authoritative anchor is what lets a third party recognize them as representations of the same underlying instrument, the precondition for shared liquidity and cross-rail settlement.

### 6.5 Two cross-cutting concerns

- **Lifecycle / asset servicing.** Coupons, dividends, corporate actions, redemptions, and maturity are where tokenization's automation promise is realized and where most operational risk lives. A `ServicingEvent(type, terms-ref)` primitive and standard event taxonomy let servicing be observed and reconciled uniformly across rails, even though execution stays platform-specific.
- **Privacy.** Across settlement, identity, and compliance, the same requirement recurs: prove a property (sufficient funds, eligibility, rule-compliance) without revealing the underlying data. Treating ZK/selective-disclosure as a cross-cutting capability, rather than bolting it onto one layer, keeps the primitives usable in the regulated, confidential deployments that institutions actually require.

## 7. Issuer Archetypes: Banks, Financial Firms, and Sovereigns

A token's design is shaped less by its asset class than by **who stands behind it**: the trust anchor. A standard that ignores this will fit no one. The comparison below is illustrative of the design space.

| | **Banks** (deposit/payment tokens) | **Financial firms** (funds, brokers, fintechs) | **Sovereigns / central banks** |
|:-:|:-:|:-:|:-:|
| **Representative examples** | J.P. Morgan Kinexys (deposit tokens, repos); Citi Token Services; tokenized deposits under MAS Project Guardian | BlackRock BUIDL; Franklin Templeton FOBXX/BENJI; Ondo OUSG/USDY/OGM; JPMorgan Asset Mgmt MONY/JLTXX | Wholesale CBDC / tokenized reserves (BIS Agorá, Helvetia, Jura); sovereign bond issuances; UBS/SDX & AIIB/Euroclear digital bonds |
| **Trust anchor** | Commercial-bank liability | Fund NAV / underlying collateral; broker obligation | Central-bank money / sovereign credit |
| **Credit risk** | Issuing bank's credit | Underlying assets + manager/structure | Effectively risk-free settlement asset (for reserves) |
| **Typical chain choice** | Permissioned or hybrid (e.g., Quorum-lineage, consortium rails) | Increasingly **public chains**, KYC-gated (Ethereum, Solana, others), often multi-chain | Permissioned shared/unified ledgers, central-bank-operated or consortium |
| **Control model** | Issuer/agent controls, freeze, deny-list | Transfer-agent allow-list; redemption gating | Central-bank governance; restricted participant set |
| **Compliance posture** | Bank's existing AML/KYC perimeter | Accredited/qualified-investor gating; transfer-agent of record | Policy/legal mandate; cross-jurisdiction coordination |
| **Settlement role** | Often the **cash leg** of DvP | Often the **asset leg** (the thing being delivered) | The **highest-tier cash leg**; monetary-policy interface |
| **What the standard must respect** | Programmable conditions on commercial-bank money; bank-controlled rules | Multi-chain portability; investor-eligibility portability; NAV/servicing metadata | Two-tier money structure; jurisdictional control over issuance, compliance, and monetary policy |

**Implications for the standard.**

1. **The cash leg and the asset leg are usually issued by different archetypes on different rails.** This is the core cross-archetype, cross-rail DvP problem the settlement primitive must serve (a fund's tokenized treasury delivered against a bank's tokenized deposit, or against tokenized reserves).
2. **Credit-tier must be explicit metadata.** Because the same DvP carries different risk depending on the cash leg's trust tier, the settlement-asset's tier belongs in the core descriptor, not buried in documentation.
3. **The standard cannot assume permissionless or permissioned.** Banks and central banks lean permissioned/consortium; asset managers increasingly issue on public chains. Protocol-neutrality is not optional.
4. **Eligibility portability has the highest cross-archetype value.** A buy-side institution verified once should be recognizable to a bank's deposit-token rail and a fund's public-chain token alike, which is exactly what the identity layer (6.2) targets.

## 8. Reference Flows

These narrative flows illustrate how the primitives compose. They are conceptual, not implementation commitments. Each flow assumes the tokens involved are already modeled as OTAS objects (Section 5.4), whether natively or by observation, and it names the OTAS interface exercised at each step. The interface is the common contract; enforcement of each step binds to whatever native mechanism the rail provides, as summarized in Section 5.3.

**Flow A: Cross-rail DvP of a tokenized sovereign bond against a tokenized deposit.**

1. Bond token (`FractionalUniqueBase`, `Restricted`, `IncomeBearing`) lives on Rail X; deposit token (`FungibleBase`, `Restricted`, `AtomicLeg`) lives on Rail Y. Both are modeled as OTAS objects per Section 5.4.
2. **Identity interface.** Both parties' wallets resolve eligibility via the identity layer, producing eligibility proofs, with no PII moving.
3. **Compliance interface (`canTransfer`).** Each token's OTAS `canTransfer` returns allow with no blocking reason codes: a pre-settlement check, so any blocking condition is knowable before settlement is proposed. On each rail this binds to the native mechanism (ERC-3643 registries, a Sui PAS `Policy<T>`/approval witnesses, or a Solana transfer hook).
4. **Settlement interface (`AtomicLeg` / `SettlementProposed`).** A settlement coordinator references both `AtomicLeg`s in a single `SettlementProposed`, declaring the cash-leg trust tier (commercial-bank deposit).
5. **Settlement interface (`Committed`).** Once the coordinator holds both legs' states, the instruction moves to `Committed`: enqueued, no longer revocable by either participant, and the point at which a rulebook can anchor legal finality.
6. **Settlement interface (`Settled` / `Aborted`).** Cross-ledger atomicity is achieved via the chosen mechanism (HTLC/escrow/shared-ledger); on success both legs emit `Settled`; on failure both `Aborted`. Principal risk is removed.

**Flow B: Coupon servicing across holders on a public chain.**

1. Bond token emits `ServicingEvent(coupon, terms-ref)` referencing the metadata anchor's coupon schedule.
2. Income is distributed to eligible holders (eligibility re-checked); confidential-amount variants prove correctness via ZK without revealing balances.
3. Indexers on any rail reconcile from the canonical event schema.

**Flow C: Sanctions/eligibility revocation.**

1. A holder's credential is revoked; the identity layer's verification now returns false.
2. The token's `Restricted` behavior blocks new transfers (`SANCTIONS_HIT`/`INELIGIBLE`); issuer/agent may invoke `Freezable`/`ForceTransferable` per policy, using whichever native control the rail provides (EVM agent freeze, Sui DenyList or PAS `clawback_funds`, Solana permanent delegate).

## 9. Relationship to Prior and Academic Work

This inquiry stands on substantial existing work and should cite and defer to it rather than duplicate it:

- **Standards bodies and consortia.** The InterWork Alliance **Token Taxonomy Framework** (base/behavior/property meta-model) is the most direct conceptual ancestor of "composable primitives." The **ERC** process (notably ERC-20, ERC-1400, ERC-3643/T-REX, ERC-4626) provides battle-tested interface patterns and a working on-chain compliance model. **W3C** DIDs and Verifiable Credentials, **OpenID4VC/VP**, **Trust over IP**, and **eIDAS 2.0** define the identity substrate. **ISO 20022**, **FIX**, and **FpML** define the financial-data semantics metadata should align with.
- **Central-bank and policy research.** BIS Innovation Hub work on the **unified ledger** and projects **Agorá, Helvetia, Jura, mBridge, and Dunbar**; the **CPMI-IOSCO Principles for Financial Market Infrastructures (PFMI)**, whose Principle 8 on settlement finality anchors the technical-versus-legal finality distinction (Section 6.1); CPMI/BIS analyses of **wholesale CBDC / tokenized reserves**; the **EBA report on tokenised deposits**; and **IMF** notes on tokenized finance and central-bank exploration of tokenized reserves collectively define the settlement-layer design space, the two-tier money structure, and the atomic-settlement evidence base.
- **Academic literature.** Peer-reviewed and preprint work on asset-backed/open-asset protocols, privacy-enhancing technologies for the FATF Travel Rule, zero-knowledge-based decentralized identity and verifiable data sharing, and the economics of tokenized settlement (e.g., WEF/industry estimates of underwriting-fee and spread reductions on tokenized bonds) inform the identity, compliance, and settlement layers.

## 10. Appendices

### 10.1 Glossary

- **Atomic settlement:** all-or-nothing execution of linked transfers; the basis of DvP/PvP.
- **Technical vs. legal finality:** technical finality is the ledger-level guarantee that linked legs all complete or all revert; legal finality is the jurisdiction-defined point at which settlement is irrevocable and unconditional, good against an insolvent participant.
- **Committed (settlement state):** a proposed lifecycle state in which the coordinator holds both legs' states, the instruction is enqueued and no longer revocable by a participant, and legal finality can be anchored, ahead of final `Settled`.
- **DvP / PvP:** delivery-versus-payment (asset vs. cash) / payment-versus-payment (currency vs. currency).
- **Tokenized deposit:** a commercial-bank deposit represented on a ledger; a bank liability.
- **Wholesale CBDC / tokenized reserves:** central-bank money on a ledger for interbank settlement.
- **DID / VC:** W3C Decentralized Identifier / Verifiable Credential.
- **Selective disclosure / ZK proof:** revealing only the minimum (or a boolean), proven cryptographically.
- **Travel Rule:** FATF requirement to transmit originator/beneficiary information between obliged entities.
- **UTXO / account / object model:** three on-chain accounting paradigms the standard must abstract over.
- **Capability:** a transferable authority object (e.g., Sui TreasuryCap) gating privileged actions.
- **Address balance:** on Sui, a canonical per-address, per-coin-type balance that funds accrue into automatically, offered as an account-style alternative to holding discrete `Coin<T>` objects.
- **Approval witness:** a zero-sized type whose presence proves that a given package approved an action; Sui PAS resolves a request only when the witnesses collected match the policy exactly.
- **Token Formula:** TTF shorthand combining a token base with behaviors and property sets.

### 10.2 Standards & projects index (for the survey)

EVM: ERC-20, ERC-721, ERC-1155, ERC-1400, ERC-3643/T-REX, ERC-4626, ERC-734/735 (ONCHAINID). Sui: Coin/Currency, Address Balances, Permissioned Asset Standard (PAS), Closed-Loop Token, Coin Registry, DenyList, capabilities. Solana: SPL Token, Token-2022 extensions. Enterprise: Hyperledger Fabric Token SDK / Panurus, Besu, Corda. Meta-model: InterWork Alliance TTF. Identity/compliance: W3C DID & VC, OpenID4VC/VP, ToIP, eIDAS 2.0, FATF Travel Rule, MiCA/AMLR, GENIUS Act. Settlement: BIS Agorá/Helvetia/Jura/mBridge/Dunbar, Partior, Fnality, RLN/RSN, SDX, Euroclear D-FMI, DTCC, J.P. Morgan Kinexys.

### 10.3 How to contribute

OTAS is an open source LFDT lab. This draft invites: researchers (literature surveys, gap analyses), protocol developers (Solidity, Move, Fabric chaincode prototypes), financial-markets practitioners (post-trade, settlement, ETF/bond lifecycle, regulatory context), technical writers, and reviewers.

The open questions identified during the drafting of this paper are tracked as GitHub issues in the project repository (label: `open-question`); community discussion and resolution of those questions is the primary input to the next revision of this document.

Contributions require a signed Developer Certificate of Origin (DCO). Specification material is intended for the Community Specification License 1.0 and reference code for Apache 2.0. Repository and lab documentation are linked in the References.

## 11. References

(Selected sources consulted for this draft; citations will be expanded and formalized in the companion Research Report.)

1. LF Decentralized Trust. OTAS Lab profile and charter. https://github.com/OpenTokenizedAssetStandard ; lab docs: https://lf-decentralized-trust-labs.github.io/labs/lfdt/otas.html
2. LF Decentralized Trust / Linux Foundation. "LF Decentralized Trust Announces 10 New Members; Adds Tokenized Assets Standard as a New Lab" (Apr 2026). https://www.linuxfoundation.org/press/lf-decentralized-trust-announces-10-new-members-adds-tokenized-assets-standard-as-a-new-lab
3. LFDT. "Open Standards for Tokenization: Why Interoperability Matters Now." https://www.lfdecentralizedtrust.org/blog/open-standards-for-tokenization-why-interoperability-matters-now
4. LFDT. "Panurus Joins LF Decentralized Trust" (Fabric Token SDK / UTXO model). https://www.lfdecentralizedtrust.org/blog/panurus-joins-lf-decentralized-trust-as-new-incubating-project ; Fabric Token SDK lab: https://lf-decentralized-trust-labs.github.io/labs/hyperledger/fabric-token-sdk.html
5. Ethereum. EIP-3643 (T-REX). https://eips.ethereum.org/EIPS/eip-3643 ; ERC-3643 Association: https://www.erc3643.org/
6. Tokeny. ERC-3643 overview and ERC-3643 vs ERC-1400. https://tokeny.com/erc3643/ ; https://tokeny.com/erc3643-vs-erc1400/
7. Chainalysis. "Introduction to ERC-3643." https://www.chainalysis.com/blog/introduction-to-erc-3643-ethereum-rwa-token-standard/
8. Sui. Coin/Currency Standard, Address Balances, Permissioned Asset Standard (PAS), Closed-Loop Token, Regulated Currencies & DenyList. https://docs.sui.io/standards/currency ; https://docs.sui.io/onchain-finance/asset-custody/address-balances/using-address-balances ; https://docs.sui.io/onchain-finance/pas/ ; PAS architecture: https://docs.sui.io/onchain-finance/pas/pas-architecture ; PAS workflows: https://docs.sui.io/onchain-finance/pas/pas-workflows ; https://docs.sui.io/onchain-finance/closed-loop-token/ ; https://docs.sui.io/guides/developer/coin/regulated
9. Solana. Token Extensions (Token-2022) docs. https://solana.com/docs/tokens/extensions ; Chainstack overview: https://docs.chainstack.com/docs/solana-token-extensions
10. InterWork Alliance. Token Taxonomy Framework. https://github.com/InterWorkAlliance/TokenTaxonomyFramework/blob/main/token-taxonomy.md ; https://www.gbbc.io/interwork-alliance/token-taxonomy-framework
11. BIS. Project Agorá. https://www.bis.org/about/bisih/topics/fmis/agora.htm
12. IMF. "Tokenized Finance" (IMF Notes 2026) and "Central Bank Exploration of Tokenized Reserves" (Note 2025/011). https://www.elibrary.imf.org/view/journals/068/2026/001/article-A001-en.xml ; https://www.imf.org/-/media/files/publications/ftn063/2025/english/ftnea2025011.pdf
13. EBA. "Report on Tokenised Deposits" (Dec 2024). https://www.eba.europa.eu/publications-and-media/publications
14. W3C. Decentralized Identifiers (DID) v1.0; Verifiable Credentials Data Model. (See W3C Recommendations, 2022.)
15. Academic/preprint: Privacy-enhancing technologies for the FATF Travel Rule; ZK-based decentralized identity & verifiable data sharing (arXiv:2510.09715; arXiv:2012.00136); IEEE "Open Asset Protocol on Blockchain."
16. Industry/issuer references: BlackRock BUIDL, Franklin Templeton FOBXX/BENJI, Ondo OUSG/OGM, J.P. Morgan Kinexys/MONY/JLTXX, Citi Token Services, UBS/SDX and AIIB/Euroclear digital bonds, MAS Project Guardian (compiled from public reporting).
17. CPMI-IOSCO. *Principles for Financial Market Infrastructures* (PFMI), Principle 8: Settlement finality. https://www.bis.org/cpmi/publ/d101a.pdf
