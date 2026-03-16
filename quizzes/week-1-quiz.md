# ✅ Week 1 Quiz — FHE Foundations & First Contracts

Complete this quiz before submitting your Week 1 homework.

---

**Q1. What does a `euint64` variable actually store in Solidity?**

- A) A plaintext 64-bit integer
- B) An AES-encrypted integer
- C) A 256-bit opaque handle pointing to a ciphertext in the FHE coprocessor
- D) A hash of the encrypted value

<details>
<summary>Show Answer</summary>

✅ **C — A 256-bit opaque handle pointing to a ciphertext in the FHE coprocessor**

Encrypted types in FHEVM are handles — references to ciphertexts stored off-chain. The number itself is never stored in the EVM state.

</details>

---

**Q2. You write `euint64 newBal = FHE.add(balance, amt)`. Do permissions from `balance` transfer to `newBal`?**

- A) Yes — permissions always propagate to derived handles
- B) Yes — but only `allowThis` propagates, not `allow(user)`
- C) No — `newBal` is a brand new handle with no permissions
- D) Only if `balance` and `amt` have identical permissions

<details>
<summary>Show Answer</summary>

✅ **C — `newBal` is a brand new handle with no permissions**

Every FHE operation creates a new handle. You must explicitly call `FHE.allowThis(newBal)` and `FHE.allow(newBal, user)` on every new handle before storing or returning it.

</details>

---

**Q3. Which line is WRONG in this contract?**

```solidity
function deposit(externalEuint64 amt, bytes calldata proof) external {
    euint64 amount = FHE.fromExternal(amt, proof);   // Line A
    euint64 newBal = FHE.add(balance, amount);       // Line B
    balance = newBal;                                // Line C
    FHE.allowThis(newBal);                           // Line D
}
```

- A) Line A
- B) Line B
- C) Line C — storing before granting `allowThis`
- D) Line D

<details>
<summary>Show Answer</summary>

✅ **C — Storing before granting `allowThis`**

`FHE.allowThis` must be called **before** storing the handle. Line C stores `newBal` into `balance` without permissions, so future operations on `balance` will fail. The fix: move `FHE.allowThis(newBal)` to before `balance = newBal`.

</details>

---

**Q4. What does `FHE.select(condition, trueVal, falseVal)` do when `condition` is encrypted-false?**

- A) Reverts
- B) Returns `trueVal`
- C) Returns `falseVal`
- D) Returns zero

<details>
<summary>Show Answer</summary>

✅ **C — Returns `falseVal`**

`FHE.select` is the branch-free ternary. Both branches are always computed. The encrypted condition determines which result is returned without revealing which branch was taken.

</details>

---

**Q5. A user calls `getBalance()` which returns `euint64`. They try to `console.log` it and see a large number like `0x3f7a...`. What happened?**

- A) The balance is actually that large number
- B) The balance was incorrectly encrypted
- C) They received a handle — they must decrypt it off-chain using the FHEVM SDK
- D) The contract forgot to call `FHE.allow`

<details>
<summary>Show Answer</summary>

✅ **C — They received a handle — they must decrypt it off-chain using the FHEVM SDK**

`getBalance()` returns a handle (a uint256 pointer), not the actual value. Use `fhevm.userDecryptEuint(FhevmType.euint64, handle, contractAddress, signer)` to decrypt.

</details>

---

**Q6. Which `FHE.div` call is valid in FHEVM?**

- A) `FHE.div(encryptedA, encryptedB)`
- B) `FHE.div(encryptedA, uint64(1000))`
- C) `FHE.div(uint64(1000), encryptedB)`
- D) `FHE.div(encryptedA, FHE.asEuint64(1000))`

<details>
<summary>Show Answer</summary>

✅ **B — `FHE.div(encryptedA, uint64(1000))`**

FHEVM only supports dividing an encrypted value by a **plaintext** value. Dividing by another encrypted value is not supported.

</details>

---

**Q7. What is the purpose of `FHE.makePubliclyDecryptable(handle)`?**

- A) Grants all users `FHE.allow` on the handle
- B) Permanently marks the handle as publicly readable by anyone — irreversible
- C) Sends the decrypted value as an event log
- D) Transfers ownership of the handle to the contract owner

<details>
<summary>Show Answer</summary>

✅ **B — Permanently marks the handle as publicly readable by anyone — irreversible**

Use this only for values that are genuinely meant to be revealed publicly (e.g., auction winner, vote tally). It cannot be undone.

</details>

---

🎉 **Score 6/7 or higher?** You're ready to submit your [Week 1 Homework](../week-1/homework/week-1-homework.md).

📖 **Score below 6?** Review [Lesson 1.2](../week-1/lessons/lesson-1-2-encrypted-types.md) and [Lesson 1.3](../week-1/lessons/lesson-1-3-first-contract.md) before submitting.
