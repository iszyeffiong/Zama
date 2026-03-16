# Wrong Input Proof Binding

**Severity:** High — transaction reverts unexpectedly

## The Problem

Input proofs are cryptographically bound to a specific `contractAddress` and `userAddress`. Generating a proof for the wrong address causes `FHE.fromExternal()` to revert.

## Wrong

```typescript
// ❌ Proof generated for wrong contract address
const encrypted = await fhevm
  .createEncryptedInput(WRONG_CONTRACT_ADDRESS, user.address)
  .add64(value)
  .encrypt();

await contract.submit(encrypted.handles[0], encrypted.inputProof);
// Reverts: proof does not match this contract
```

## Correct

```typescript
// ✅ Use the exact contract and user addresses
const encrypted = await fhevm
  .createEncryptedInput(CONTRACT_ADDRESS, user.address)
  .add64(value)
  .encrypt();

await contract.connect(user).submit(encrypted.handles[0], encrypted.inputProof);
```

## Common Causes

- Hardcoding a proxy address instead of the implementation address
- Using the deployer address instead of the actual caller (`user.address`)
- Generating the proof before the contract is deployed (address not yet known)
