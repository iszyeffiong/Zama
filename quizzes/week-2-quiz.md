# ✅ Week 2 Quiz — Access Control, Proofs & Patterns

---

**Q1. Alice generates an input proof for ContractA. Bob submits it to ContractB. What happens?**

- A) It succeeds — proofs are reusable across contracts
- B) ContractB reverts — the proof is bound to ContractA's address
- C) It succeeds but the handle is corrupted
- D) It succeeds if Bob is the same address as Alice

<details>
<summary>Show Answer</summary>

✅ **B — ContractB reverts — the proof is bound to ContractA's address**

Input proofs cryptographically bind the ciphertext to a specific `contractAddress` and `senderAddress`. Submitting to a different contract causes `FHE.fromExternal` to revert.

</details>

---

**Q2. What does `FHE.allowTransient(handle, contractB)` do?**

- A) Permanently grants ContractB permission to use the handle
- B) Grants ContractB permission for the current transaction only
- C) Allows ContractB to make the handle publicly decryptable
- D) Transfers handle ownership to ContractB

<details>
<summary>Show Answer</summary>

✅ **B — Grants ContractB permission for the current transaction only**

`allowTransient` is designed for cross-contract flows. The permission expires at the end of the transaction.

</details>

---

**Q3. You revoke a user's access by not calling `FHE.allow` on new handles. Can they still decrypt handles they previously received?**

- A) No — revoking access invalidates all previous handles
- B) Yes — handles they already received remain decryptable
- C) Only if the contract calls `FHE.revoke(handle, user)`
- D) Only within 24 hours of original grant

<details>
<summary>Show Answer</summary>

✅ **B — Yes — handles they already received remain decryptable**

Revoking is forward-looking only. You stop granting new permissions, but previously granted handles cannot be invalidated. Design your permission flow with this in mind.

</details>

---

**Q4. Which pattern correctly implements "transfer zero if non-compliant, full amount otherwise"?**

- A) `if (isCompliant[msg.sender]) { transfer(amount); }`
- B) `require(isCompliant[msg.sender], "Not compliant"); transfer(amount);`
- C) `euint64 actual = FHE.select(isCompliant[msg.sender], amount, FHE.asEuint64(0));`
- D) `euint64 actual = FHE.and(isCompliant[msg.sender], amount);`

<details>
<summary>Show Answer</summary>

✅ **C — `FHE.select(isCompliant[msg.sender], amount, FHE.asEuint64(0))`**

This is the branch-free pattern. `isCompliant` is `ebool`, which cannot be used in `if()`. `FHE.select` evaluates both outcomes and picks based on the encrypted condition.

</details>

---

**Q5. Contract A calls Contract B passing a `euint64` handle. The call reverts. What is most likely missing?**

- A) `FHE.allowThis(handle)` in Contract A
- B) `FHE.allow(handle, address(contractB))` after the call
- C) `FHE.allowTransient(handle, address(contractB))` before the call
- D) `FHE.makePubliclyDecryptable(handle)` before the call

<details>
<summary>Show Answer</summary>

✅ **C — `FHE.allowTransient(handle, address(contractB))` before the call**

Cross-contract handle passing requires `allowTransient` called in Contract A before the external call so Contract B has permission within the same transaction.

</details>

---

**Q6. Which of these is NOT a valid FHE anti-pattern?**

- A) Storing a handle without calling `FHE.allowThis`
- B) Using `FHE.select` instead of an `if` statement
- C) Calling `FHE.allow(handle, user)` after `FHE.fromExternal`
- D) Using `ebool` in a Solidity `require()` statement

<details>
<summary>Show Answer</summary>

✅ **C — Calling `FHE.allow(handle, user)` after `FHE.fromExternal`**

This is the **correct** pattern, not an anti-pattern. After validating an input with `fromExternal`, granting `allow` to the user is expected and necessary for them to decrypt their result.

</details>

---

🎉 **Score 5/6 or higher?** You're ready for [Week 3](../week-3/overview.md).
