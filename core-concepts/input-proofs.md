# Input Proofs

Input proofs are cryptographic attestations that **bind an encrypted input to a specific contract and sender**. They are required whenever a user submits an encrypted value to a contract.

## Why They Exist

Without input proofs, an attacker could replay a ciphertext originally intended for Contract A against Contract B, or submit someone else's encrypted value as their own.

A proof proves: *"This ciphertext was encrypted specifically for `contractAddress` by `senderAddress`."*

## How to Use Input Proofs On-Chain

```solidity
function placeBid(externalEuint64 encryptedBid, bytes calldata inputProof) external {
    // FHE.fromExternal validates that the proof matches:
    // - address(this) — this specific contract
    // - msg.sender — this specific caller
    euint64 bid = FHE.fromExternal(encryptedBid, inputProof);
    // bid is now a verified, usable handle
}
```

## How to Generate Input Proofs Off-Chain

```typescript
import { fhevm } from "hardhat";

const encrypted = await fhevm
  .createEncryptedInput(contractAddress, userAddress)  // must match exactly
  .add64(bidAmount)
  .encrypt();

// Submit to contract
await contract.connect(user).placeBid(
  encrypted.handles[0],   // externalEuint64
  encrypted.inputProof    // bytes
);
```

## Multiple Values, One Proof

You can encrypt multiple values with a single proof:

```typescript
const encrypted = await fhevm
  .createEncryptedInput(contractAddress, userAddress)
  .add64(value1)
  .add64(value2)
  .add8(flag)
  .encrypt();

// encrypted.handles[0] = first euint64
// encrypted.handles[1] = second euint64
// encrypted.handles[2] = euint8
// encrypted.inputProof = single proof for all three
```

## What Proofs Prevent

| Attack | Without proofs | With proofs |
|---|---|---|
| Cross-contract replay | Ciphertext works on any contract | Rejected — wrong contract |
| Cross-user replay | Alice's input submittable by Bob | Rejected — wrong sender |
| Forged inputs | Any ciphertext accepted | Rejected — invalid proof |

## Common Pitfall

Generating the proof for the wrong contract or user address will cause `FHE.fromExternal` to revert:

```typescript
// ❌ WRONG — proof bound to wrong address
const encrypted = await fhevm
  .createEncryptedInput(wrongContractAddress, userAddress)
  .add64(value)
  .encrypt();
// This will revert when submitted to the real contract

// ✅ CORRECT
const encrypted = await fhevm
  .createEncryptedInput(CONTRACT_ADDRESS, user.address)
  .add64(value)
  .encrypt();
```
