# Contributing to OTAS

Thanks for your interest in the **Open Tokenized Asset Standard (OTAS)**. This repository holds the community whitepaper and its supporting material, developed as an open draft under the **OTAS Lab at LF Decentralized Trust (LFDT)**.

It's a draft, shared for community review and refinement. All input is welcome — corrections, challenges to our assumptions, pointers to prior work, and proposals for new content.

OTAS is an open-source LFDT effort and welcomes **researchers, protocol developers, financial-markets practitioners, technical writers, and reviewers**. The quickest way in: read the whitepaper, browse the open questions in [Discussions](https://github.com/OpenTokenizedAssetStandard/whitepaper/discussions), and jump into the ones in your area of expertise.

New here? Start with these:

- 📖 **[Read the whitepaper (v0.1)](https://github.com/OpenTokenizedAssetStandard/whitepaper/blob/whitepaper-v0.1/whitepaper.md)**
- 👋 **[Introduce yourself in the welcome discussion](https://github.com/OpenTokenizedAssetStandard/whitepaper/discussions/7)**

---

## Code of Conduct

This project follows a [Code of Conduct](./CODE_OF_CONDUCT.md) based on the LF Decentralized Trust community standards. By participating, you agree to uphold it. Please be respectful, constructive, and considerate when engaging with other community members.

---

## Ways to contribute

Useful contributions include:

- Reviewing the whitepaper and giving feedback on specific sections
- Correcting factual errors, typos, broken links, or citations
- Engaging with the open research questions, or raising new ones
- Proposing new content: sections, examples, issuer archetypes, reference flows
- Improving repository tooling, docs, and workflows
- Sharing prior work, standards, or academic work we should reference

### Discussions vs. Issues — which to use?

Use **[Discussions](https://github.com/OpenTokenizedAssetStandard/whitepaper/discussions)** for open-ended questions, ideas, introductions, and debate on direction and open research questions.

Use **[Issues](https://github.com/OpenTokenizedAssetStandard/whitepaper/issues)** for specific, actionable items — corrections, section feedback, content proposals, and tooling tasks.

Not sure which fits? A discussion is always a good place to start, and it can move to an issue later if it turns into something actionable.

### Opening an issue

When you open an issue, you'll be offered a set of templates. Please pick the one that fits:

- **📝 Content feedback**: comment on or push back on a section
- **🔧 Correction**: factual error, typo, broken link, citation
- **❓ Open question**: engage with or raise a research question
- **➕ New content proposal**: propose new material
- **⚙️ Repository / tooling**: CI, workflows, build, or repo docs

Before opening, please search existing [issues](https://github.com/OpenTokenizedAssetStandard/whitepaper/issues) and [discussions](https://github.com/OpenTokenizedAssetStandard/whitepaper/discussions). Your point may already be under way, and adding to an existing thread keeps the conversation in one place.

---

## Making changes (pull request workflow)

Changes are proposed through pull requests from your own fork of the repository. The steps below walk through the whole process.

> **Please discuss before opening a pull request.** For anything beyond a small fix (typo, broken link, obvious correction), raise your idea first in a [Discussion](https://github.com/OpenTokenizedAssetStandard/whitepaper/discussions) or an [Issue](https://github.com/OpenTokenizedAssetStandard/whitepaper/issues), and get rough agreement on the approach before you start, it keeps direction aligned with the community and saves wasted effort.
>
> And if you're working on one of the verticals related to OTAS, we'd love to hear about it, please keep us posted on where we might collaborate or help contribute.

Throughout, replace `<your-username>` with your GitHub username, and treat `whitepaper-v0.1` as the current default development branch (unless an issue tells you otherwise).

### 1. Fork the repository

Open the repository and click **Fork** (top-right): 👉 https://github.com/OpenTokenizedAssetStandard/whitepaper

This creates your copy at `https://github.com/<your-username>/whitepaper`.

### 2. Clone your fork

```bash
git clone https://github.com/<your-username>/whitepaper.git
cd whitepaper
```

### 3. Add the `upstream` remote

So you can pull in changes made by others while you work:

```bash
git remote add upstream https://github.com/OpenTokenizedAssetStandard/whitepaper.git
```

### 4. Set your Git identity (first time only)

Your commits must match your DCO sign-off (see the DCO section). Use an email **verified on your GitHub account**:

```bash
git config user.name "Your Name"
git config user.email "your.email@example.com"
```

### 5. Create a branch

Start from the latest upstream state, and never commit to the default branch directly:

```bash
git fetch upstream
git checkout -b add-legal-finality upstream/whitepaper-v0.1
```

Use a short, descriptive branch name, e.g. `add-legal-finality`, `fix-broken-link`.

### 6. Make your change

Edit the files, keeping each change focused on **one topic**.

### 7. Commit with a DCO sign-off

Every commit must be signed off with `-s`:

```bash
git add .
git commit -s -m "Add legal-finality subsection to §6.1"
```

If the sign-off is missing or the identity doesn't match, the DCO check fails — see the [DCO section](#developer-certificate-of-origin-dco) to fix it.

> If upstream changes while you work, catch up with `git pull --rebase upstream whitepaper-v0.1`.

### 8. Push to your fork

```bash
git push origin add-legal-finality
```

### 9. Open the pull request

On GitHub, your fork shows a **Compare & pull request** button. Open the PR against the base repository (`OpenTokenizedAssetStandard/whitepaper`), targeting the `whitepaper-v0.1` branch. Fill in the template and link the issue — `Closes #123` or `Refs #123`.

### 10. Respond to review

Commit to the same branch and push again — the PR updates automatically. Sign off every commit:

```bash
git commit -s -m "Address review: clarify revocability boundary"
git push origin add-legal-finality
```

All checks (including DCO) must pass before a maintainer can merge.

### 11. After your PR is merged

```bash
git checkout whitepaper-v0.1
git branch -d add-legal-finality
```

> **Tip:** small, single-purpose PRs get reviewed and merged faster than large ones.

---

## Developer Certificate of Origin (DCO)

All commits **must be signed off**, certifying you have the right to contribute the change ([what the DCO is](https://developercertificate.org/)).

Add the `-s` flag when committing:

```bash
git commit -s -m "Your commit message"
```

The sign-off name and email must match your commit author identity, and the email should be verified on your GitHub account, or the check will fail. Forgot to sign off? Fix the last commit with `git commit --amend -s --no-edit` (add `--reset-author` if the identity was also wrong). The DCO check must be green before a PR can merge.

---

## Commit messages

- Write in the **imperative mood**: *"Add legal-finality subsection,"* not *"Added…"* (it should complete the sentence "If applied, this commit will…").
- Keep the subject line short (~50 characters) and add a body if the change needs explanation based on selected issue template.
- A [Conventional Commits](https://www.conventionalcommits.org/) prefix is welcome but not required — e.g. `docs:`, `fix:`, `ci:`.

---

## Licensing

By contributing, you agree that your contributions to the whitepaper are licensed under **CC-BY-4.0** (Creative Commons Attribution 4.0 International). You retain copyright to your contributions; the DCO sign-off certifies you have the right to submit them.

---

## Community & getting help

- **Questions or ideas?** Open a [discussion](https://github.com/OpenTokenizedAssetStandard/whitepaper/discussions).
- **Community calls.** OTAS holds regular open community calls where technical direction is discussed in public; everyone is welcome.
- **Stuck on a contribution?** Comment on the relevant issue or discussion and a maintainer will help.

---

Thank you for helping build OTAS. Every correction, question, and proposal moves the standard forward. 🚀