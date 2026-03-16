# Week 4 Capstone Project

**Estimated time:** 8–10 hours | **Final submission of the bootcamp**

---

## Overview

Design and build a complete, production-ready confidential application on FHEVM. This is your chance to demonstrate everything you've learned across 4 weeks.

---

## Choose Your Track

Pick one of the following tracks based on your interests:

### Track A — Confidential DeFi Protocol
Build a privacy-preserving DeFi application. Examples:
- Confidential AMM with encrypted swap amounts
- Private order book exchange
- Encrypted lending protocol (collateral and debt amounts hidden)

### Track B — Confidential Identity System
Build a privacy-preserving identity or compliance application. Examples:
- KYC-gated confidential token with observer access
- Encrypted credential registry
- Private DAO membership with voting rights

### Track C — Confidential Gaming Platform
Build a multi-player game with on-chain privacy. Examples:
- Encrypted poker or card game
- Private sealed-bid marketplace
- Confidential prediction market

### Track D — Open Track
Build anything that meaningfully uses FHEVM. Must include at least 3 distinct FHE operations and a real use case justification.

---

## Deliverables

Your submission must include:

### 1. Smart Contracts (`contracts/`)
- At least one main contract + any supporting contracts
- Full NatSpec documentation on all public functions
- No compiler warnings

### 2. Test Suite (`test/`)
- Minimum 15 test cases
- Must cover: happy path, edge cases, permission tests, and at least 2 known anti-patterns (show they are handled correctly)
- All tests must pass

### 3. README.md
- What your project does (1 paragraph)
- Architecture diagram (ASCII or image)
- Key FHE design decisions and tradeoffs
- How to deploy and run tests

### 4. Deployment Script (`scripts/deploy.ts`)
- Working deployment script for Zama Devnet

---

## Grading Criteria

| Criteria | Points |
|---|---|
| Project compiles and all tests pass | 15 |
| Meaningful use of FHE (3+ distinct operations) | 15 |
| Correct permission design (allowThis, allow, allowTransient) | 20 |
| Branch-free logic throughout (FHE.select, no if(ebool)) | 15 |
| Test coverage (quality + breadth) | 15 |
| README clarity and architecture explanation | 10 |
| Code quality, NatSpec, no anti-patterns | 10 |
| **Total** | **100** |

### Excellence Bonus (up to 30 points)
- Deployed and verified on Zama Devnet (10 pts)
- Frontend dApp that interacts with the contract (15 pts)
- Novel use of FHEVM not covered in bootcamp material (5 pts)

---

## Submission

Push to GitHub and submit:
1. Repo link
2. 2-minute Loom walkthrough explaining your design choices
3. (Bonus) Devnet contract address

---

## Past Capstone Examples for Inspiration

- `ConfidentialPayroll` — employer pays employees with hidden salary amounts
- `EncryptedEscrow` — multi-party escrow with hidden deposit amounts
- `PrivateOrderBook` — on-chain limit orders with encrypted prices
- `ConfidentialDAO` — DAO where member voting power is hidden
