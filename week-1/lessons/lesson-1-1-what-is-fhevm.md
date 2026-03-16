# Lesson 1.1 — What is FHEVM?

**Duration:** 45 minutes | **Format:** Reading + Discussion

---

## Learning Goals

- Understand what Fully Homomorphic Encryption is at a conceptual level
- Know why on-chain privacy matters and what current alternatives lack
- Understand FHEVM's architecture: EVM + FHE coprocessor

---

## 1. The Privacy Problem in Web3

Every transaction on a public blockchain is visible to everyone. For many use cases — auctions, voting, salaries, medical records, financial positions — this is a fundamental blocker.

Existing approaches and their limits:

| Approach | How it works | Limitation |
|---|---|---|
| Zero-knowledge proofs (ZKP) | Prove a statement without revealing data | Cannot compute on hidden state across calls |
| Commit-reveal | Hash the value, reveal later | Front-running risk; data eventually public |
| Off-chain computation | Process data off-chain, post result | Requires trust in off-chain party |
| Trusted hardware (TEE) | Run code in secure enclave | Hardware vulnerabilities; trust assumptions |

**FHE solves this differently:** compute directly on encrypted data, on-chain, with no trusted third party.

---

## 2. FHE in Plain Language

> Think of a locked box that can do math. You put a number in the box, hand it to the blockchain, and the blockchain adds, subtracts, and compares — all without ever opening the box. Only you (or someone you authorize) can open it.

Formally: Fully Homomorphic Encryption allows arbitrary computation on ciphertexts such that the result, when decrypted, equals the result of the same computation on the plaintexts.

```
encrypt(a) + encrypt(b) = encrypt(a + b)
```

---

## 3. FHEVM Architecture

FHEVM combines a standard EVM with a **FHE coprocessor** — an off-chain service that stores ciphertexts and executes FHE operations.

```
┌─────────────────────────────────────────────────┐
│                   User (off-chain)               │
│  1. Encrypt value (bound to contract + address)  │
│  2. Submit tx with ciphertext + input proof      │
└───────────────────┬─────────────────────────────┘
                    │
┌───────────────────▼─────────────────────────────┐
│              Smart Contract (EVM)                │
│  3. Validate input proof via FHE.fromExternal()  │
│  4. Request FHE operations (add, compare, etc.)  │
│  5. Receive new encrypted handles                │
│  6. Grant permissions: allowThis / allow         │
└───────────────────┬─────────────────────────────┘
                    │
┌───────────────────▼─────────────────────────────┐
│            FHE Coprocessor (off-chain)           │
│  - Stores all ciphertexts                        │
│  - Executes FHE operations                       │
│  - Returns new encrypted handles                 │
└─────────────────────────────────────────────────┘
```

---

## 4. What FHEVM Enables

| Use Case | Without FHEVM | With FHEVM |
|---|---|---|
| Token balances | Public on-chain | Encrypted, user-visible only |
| Auction bids | Revealed at submission | Sealed until reveal phase |
| Voting | Public or off-chain | On-chain, private tallying |
| KYC checks | Reveal identity | Prove compliance without exposure |
| DeFi trades | Public trade size | Encrypted swap amounts |
| Salary/payroll | Public amounts | Confidential disbursement |

---

## 5. Key Terms

| Term | Definition |
|---|---|
| **Ciphertext** | An encrypted value |
| **Handle** | A 256-bit pointer to a ciphertext stored by the coprocessor |
| **Input proof** | Cryptographic proof binding a ciphertext to a contract + sender |
| **FHE coprocessor** | Off-chain service that stores and operates on ciphertexts |
| **euint64** | An encrypted 64-bit unsigned integer (a handle type in Solidity) |

---

## Discussion Questions

1. What Web3 use case would benefit most from on-chain privacy in your view?
2. What are the trust assumptions in FHEVM compared to a ZKP-based system?
3. Why does the FHE coprocessor need to be off-chain rather than in the EVM?

---

📖 **Next:** [Lesson 1.2 — Encrypted Types & the Handle Model](lesson-1-2-encrypted-types.md)
