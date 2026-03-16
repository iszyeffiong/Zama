# Lesson 2.1 — Input Proofs Deep Dive

**Duration:** 60 minutes | **Format:** Reading + hands-on exercises

---

## Learning Goals

- Understand exactly what an input proof proves and why it is required
- Generate input proofs client-side for single and multiple values
- Know what happens when a proof is invalid and how to debug it
- Understand the replay attack that proofs prevent

---

## 1. Why Input Proofs Exist

When a user submits an encrypted value to a contract, how does the contract know:
- This ciphertext was encrypted by **this specific user** (not replayed from someone else)?
- This ciphertext was intended for **this specific contract** (not stolen from another contract)?

Without input proofs, an attacker could:
1. Watch Alice submit an encrypted bid of 1000 to AuctionA
2. Copy that ciphertext and submit it to AuctionB pretending to be Alice
3. Or replay Alice's ciphertext as their own bid

**Input proofs are cryptographic attestations that bind a ciphertext to a specific `(contractAddress, senderAddress)` pair.**

---

## 2. What a Proof Contains

When you call `fhevm.createEncryptedInput(contractAddress, senderAddress)`, the SDK:

1. Generates a fresh ciphertext for your value
2. Signs it with a commitment to `contractAddress` and `senderAddress`
3. Returns the ciphertext handle and a proof bytes payload

The contract verifies this during `FHE.fromExternal(encryptedInput, inputProof)`:
- Does the proof match `address(this)`?
- Does the proof match `msg.sender`?
- Is the cryptographic signature valid?

If any check fails — revert.

---

## 3. Generating Proofs Client-Side

### Single value

```typescript
import { fhevm } from "hardhat";

const encrypted = await fhevm
  .createEncryptedInput(contractAddress, userAddress)
  .add64(bidAmount)          // encrypt a uint64
  .encrypt();

// encrypted.handles[0]  = externalEuint64
// encrypted.inputProof  = bytes proof
await contract.connect(user).placeBid(encrypted.handles[0], encrypted.inputProof);
```

### Multiple values — one proof

```typescript
const encrypted = await fhevm
  .createEncryptedInput(contractAddress, userAddress)
  .add64(salary)      // handles[0]
  .add64(bonus)       // handles[1]
  .add8(grade)        // handles[2]
  .addBool(isActive)  // handles[3]
  .encrypt();

// All four values share ONE inputProof
await contract.submitPayroll(
  encrypted.handles[0],
  encrypted.handles[1],
  encrypted.handles[2],
  encrypted.handles[3],
  encrypted.inputProof
);
```

This is more efficient than generating four separate proofs.

---

## 4. Using Proofs On-Chain

```solidity
function placeBid(
    externalEuint64 encryptedBid,
    bytes calldata inputProof
) external {
    // FHE.fromExternal validates the proof against address(this) and msg.sender
    // Reverts if the proof is invalid, stale, or bound to a different address
    euint64 bid = FHE.fromExternal(encryptedBid, inputProof);

    FHE.allowThis(bid);
    FHE.allow(bid, msg.sender);
    bids[msg.sender] = bid;
}
```

---

## 5. Common Proof Errors and Fixes

| Error | Cause | Fix |
|---|---|---|
| `fromExternal` reverts | Wrong contract address in proof | Use exact deployed contract address |
| `fromExternal` reverts | Wrong sender address in proof | Use `user.address`, not `deployer.address` |
| `fromExternal` reverts | Stale proof (contract redeployed) | Regenerate proof after each deploy |
| Proof rejected in test | Using proxy address | Use implementation address |

### Debugging tip

```typescript
// Always log these before submitting:
console.log("Contract address:", contractAddress);
console.log("Sender address:", user.address);
// These must match exactly what was used in createEncryptedInput
```

---

## 6. Proof Binding is Per-Sender

```typescript
// Alice generates a proof for herself
const aliceEncrypted = await fhevm
  .createEncryptedInput(contractAddress, alice.address)
  .add64(500n)
  .encrypt();

// Bob tries to submit Alice's proof as his own bid
await contract.connect(bob).placeBid(
  aliceEncrypted.handles[0],
  aliceEncrypted.inputProof
);
// ❌ REVERTS — proof is bound to alice.address, not bob.address
```

---

## Hands-On Exercise

In the [Week 2 starter repo](../../resources/starter-repos.md), open `exercises/2-1-proofs.ts`:

1. Generate a proof for the correct contract and sender — verify it works
2. Intentionally use the wrong contract address — observe the revert
3. Have User B submit User A's proof — observe the revert
4. Encrypt 3 values with a single proof — verify all three work in the contract

---

📖 **Next:** [Lesson 2.2 — Access Control & Permissions](lesson-2-2-access-control.md)
