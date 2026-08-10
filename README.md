# Open Tokenized Asset Standard (OTAS): Whitepaper

[![LFDT Lab](https://img.shields.io/badge/LF%20Decentralized%20Trust-Lab-blue)](https://www.lfdecentralizedtrust.org/)  [![Status: Draft v0.1](https://img.shields.io/badge/status-draft%20v0.1-orange)](./whitepaper.md)  [![License: CC BY 4.0](https://img.shields.io/badge/license-CC--BY--4.0-lightgrey)](./LICENSE)

The **OTAS community whitepaper** explores whether a small set of composable token primitives can serve as a convergence layer for post-trade clearing, settlement, and compliance across heterogeneous distributed ledgers.

OTAS is a lab under [LF Decentralized Trust (LFDT)](https://www.lfdecentralizedtrust.org/).

## The idea in brief

Tokenized assets are proliferating across incompatible ledgers, so the same instrument tokenized by different institutions becomes several incompatible objects, each with its own token schema, identity model, compliance enforcement, and metadata. Liquidity is fragmented and compliance work is duplicated.

OTAS explores a thin, protocol-agnostic **interface layer** that standardizes the shared *functions* at the seams between systems (settlement, identity, compliance, and asset metadata), rather than the internal representation of the asset or a new ledger.

## Read the whitepaper

**[OTAS Whitepaper v0.1 (draft for review)](./whitepaper.md)**

It surveys existing token standards across the EVM, Sui, Solana, and Hyperledger ecosystems, proposes a composable primitive/behavior/event model, and develops four layers (Settlement, Identity, Compliance, and Asset Metadata) alongside a comparison of bank, financial-firm, and sovereign issuers, set against a fast-evolving regulatory landscape.

> This is a draft community whitepaper published for review and feedback. Its contents are subject to change and do not constitute legal, regulatory, financial, or investment advice. See the disclaimer in the whitepaper for details.

## Getting involved

OTAS is an open source LFDT effort and welcomes researchers, protocol developers financial-markets practitioners, technical writers, and reviewers. See the [contribution guidelines](./CONTRIBUTING.md) for how to get started, and the [Code of Conduct](./CODE_OF_CONDUCT.md) for our community standards. All commits must be signed off under the Developer Certificate of Origin (DCO).

### Join the discussion

The whitepaper is an open draft that ends with open questions rather than a finished specification, and community input is the primary driver of each revision. Browse the **[Discussions](https://github.com/OpenTokenizedAssetStandard/whitepaper/discussions)** for open questions and direction (look for the `[Open Question]` prefix), and check the **[open issues](https://github.com/OpenTokenizedAssetStandard/whitepaper/issues)**. Weigh in where it fits your expertise: propose an answer, challenge an assumption, or point us to prior work.

New to the project? **[Introduce yourself in the welcome discussion](https://github.com/OpenTokenizedAssetStandard/whitepaper/discussions/7)**.

## License

The whitepaper in this repository is licensed under [CC-BY-4.0](./LICENSE) (Creative Commons Attribution 4.0 International).
