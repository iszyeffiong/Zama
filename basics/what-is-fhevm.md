# What is FHEVM?

FHEVM (Fully Homomorphic Encryption Virtual Machine) enables **computations on encrypted data directly on-chain**. Built by [Zama](https://zama.ai), it allows smart contracts to process sensitive values  balances, bids, votes, identity attributes  without ever revealing them in plaintext.

## Key Properties

- **Data stays encrypted**  sensitive values are never exposed on-chain, not even to the contract itself
- **Full arithmetic on ciphertext**  add, subtract, compare, and branch on encrypted values
- **User-controlled decryption**  only explicitly authorized parties can read results
- **Compatible with standard tooling**  works with Hardhat, Solidity, and TypeScript tests

## How It Works

FHEVM introduces a **FHE coprocessor**  an off-chain service that stores ciphertexts and executes FHE operations. Smart contracts submit operation requests; the coprocessor computes on the encrypted data and returns new encrypted results.

```
User (off-chain)                Contract (on-chain)         FHE Coprocessor
     |                               |                            |
     |-- encrypt(value) -----------> |                            |
     |-- submit(ciphertext, proof) -> |                            |
     |                               |-- FHE.add(a, b) ---------->|
     |                               |<-- new encrypted handle ---|
     |                               |-- store handle             |
     |<-- return handle -------------|                            |
     |-- decrypt(handle) off-chain                               |
```

## What FHEVM Enables

| Use Case | Traditional | With FHEVM |
|---|---|---|
| Token balances | Public on-chain | Encrypted, user-visible only |
| Auction bids | Revealed at submission | Sealed until reveal phase |
| Voting | Public or off-chain | On-chain, private tallying |
| KYC checks | Reveal identity | Prove compliance without exposure |
| DeFi trades | Public trade size | Encrypted swap amounts |

## Relationship to Zama's Stack

- **TFHE-rs**  the underlying Rust FHE library
- **fhevm-solidity**  the Solidity library (`@fhevm/solidity`)
- **FHEVM**  the EVM-compatible network with FHE coprocessor
- **Hardhat plugin** (`@fhevm/hardhat-plugin`)  local mock for testing


---

📝 **Ready to test your knowledge?** → [Take the Quiz](../quizzes/what-is-fhevm-quiz.md)
