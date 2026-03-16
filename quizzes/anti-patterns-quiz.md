# ✅ Quiz — Anti-Patterns & Pitfalls

---

**Q1. You store a handle with `storedValue = val` but forget `FHE.allowThis(val)`. What happens next transaction?**

- A) The contract automatically re-authorizes the handle
- B) The contract reverts with "Unauthorized handle"
- C) Operations on `storedValue` fail — the contract has no permission to use it
- D) Nothing — `allowThis` is optional for stored values

<details>
<summary>Show Answer</summary>

✅ **C — Operations on `storedValue` fail — the contract has no permission to use it**

Without `allowThis`, the FHE coprocessor rejects the contract's use of that handle in future transactions.

</details>

---

**Q2. A user calls your contract, gets a handle back, but decryption fails. What is the most likely cause?**

- A) The user's wallet doesn't support FHEVM
- B) `FHE.allow(handle, msg.sender)` was never called
- C) The handle was created with the wrong encrypted type
- D) The contract wasn't deployed on Zama Devnet

<details>
<summary>Show Answer</summary>

✅ **B — `FHE.allow(handle, msg.sender)` was never called**

Decryption requires explicit permission. If the contract never called `FHE.allow(handle, user)`, the user cannot decrypt.

</details>

---

**Q3. Which line of code contains an anti-pattern?**

```solidity
euint64 result = FHE.add(a, b);
FHE.allowThis(result);
FHE.allow(result, msg.sender);
if (result > FHE.asEuint64(100)) { doSomething(); }
```

- A) Line 1 — `FHE.add` is not valid
- B) Line 2 — `allowThis` should come after `allow`
- C) Line 4 — cannot use `euint64` in an `if()` condition
- D) There is no anti-pattern here

<details>
<summary>Show Answer</summary>

✅ **C — Line 4 — cannot use `euint64` in an `if()` condition**

`euint64` and `ebool` cannot be used in Solidity control flow. Use `FHE.select()` for branch-free logic.

</details>

---

**Q4. Contract A calls Contract B passing a `euint64` handle. The call reverts. What is missing?**

- A) `FHE.allowThis(handle)` in Contract A
- B) `FHE.allowTransient(handle, address(contractB))` before the call
- C) `FHE.makePubliclyDecryptable(handle)` in Contract A
- D) `FHE.allow(handle, address(contractB))` after the call

<details>
<summary>Show Answer</summary>

✅ **B — `FHE.allowTransient(handle, address(contractB))` before the call**

Cross-contract handle passing requires `allowTransient` to be called before the external call so Contract B has permission within the transaction.

</details>

---

**Q5. You generate an input proof like this:**
```typescript
fhevm.createEncryptedInput(wrongAddress, user.address).add64(100).encrypt()
```
**What happens when the contract calls `FHE.fromExternal`?**

- A) It succeeds — the proof is valid regardless of contract address
- B) It reverts — the proof is bound to `wrongAddress`, not `address(this)`
- C) It succeeds but returns a corrupted handle
- D) It emits a warning event but continues

<details>
<summary>Show Answer</summary>

✅ **B — It reverts — the proof is bound to `wrongAddress`, not `address(this)`**

Input proofs are cryptographically tied to both the contract address and the sender. Using the wrong address causes `fromExternal` to revert.

</details>

---

**Q6. Why can't you use `FHE.div(numerator, encryptedDenominator)` in an AMM?**

- A) FHE division always returns zero for encrypted inputs
- B) FHEVM only supports `FHE.div(encrypted, plaintext)` — encrypted denominators are unsupported
- C) Division is not supported in FHEVM at all
- D) AMM contracts require a special license for division

<details>
<summary>Show Answer</summary>

✅ **B — FHEVM only supports `FHE.div(encrypted, plaintext)` — encrypted denominators are unsupported**

This is a core FHE constraint. Workarounds include keeping reserves public (simple AMM) or using masked fractions with off-chain oracles (production AMM).

</details>

---

🎉 **All done?** Move on to [Learning Paths](../learning-paths/beginner.md).
