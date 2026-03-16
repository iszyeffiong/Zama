# ✅ Quiz — FHE Operations

---

**Q1. Which of these operations is NOT supported in FHEVM?**

- A) `FHE.add(encryptedA, encryptedB)`
- B) `FHE.mul(encryptedA, plaintextK)`
- C) `FHE.div(encryptedA, encryptedB)`
- D) `FHE.sub(encryptedA, encryptedB)`

<details>
<summary>Show Answer</summary>

✅ **C — `FHE.div(encryptedA, encryptedB)`**

FHEVM only supports `FHE.div(encrypted, plaintext)`. Dividing by an encrypted value is not supported.

</details>

---

**Q2. What does `FHE.select(condition, trueVal, falseVal)` return when `condition` is encrypted-false?**

- A) `trueVal`
- B) `falseVal`
- C) Zero
- D) It reverts

<details>
<summary>Show Answer</summary>

✅ **B — `falseVal`**

`FHE.select` is the branch-free ternary: `condition ? trueVal : falseVal`. Both are always computed; the encrypted condition picks which is returned.

</details>

---

**Q3. What is the return type of `FHE.gt(a, b)`?**

- A) `bool`
- B) `uint8`
- C) `ebool`
- D) `euint64`

<details>
<summary>Show Answer</summary>

✅ **C — `ebool`**

All comparison operations return `ebool` — an encrypted boolean. It cannot be used directly in a Solidity `if()`.

</details>

---

**Q4. You need to check if an encrypted balance is zero. Which operation is correct?**

- A) `if (balance == 0)`
- B) `ebool isZero = FHE.eq(balance, uint64(0))`
- C) `require(balance == FHE.asEuint64(0))`
- D) `bool isZero = FHE.decode(balance) == 0`

<details>
<summary>Show Answer</summary>

✅ **B — `ebool isZero = FHE.eq(balance, uint64(0))`**

Mixed comparisons (encrypted vs plaintext) are supported. The result is an `ebool` which you then use with `FHE.select`.

</details>

---

**Q5. `FHE.div(FHE.asEuint64(7), 2)` — what is the result?**

- A) `3.5`
- B) `4` (rounded up)
- C) `3` (truncated)
- D) Reverts due to odd division

<details>
<summary>Show Answer</summary>

✅ **C — `3` (truncated)**

FHE division is integer division — it truncates, just like Solidity. There is no rounding.

</details>

---

**Q6. Which operation would you use to implement "transfer zero if non-compliant, else transfer full amount"?**

- A) `FHE.and(isCompliant, amount)`
- B) `FHE.select(isCompliant, amount, FHE.asEuint64(0))`
- C) `if (isCompliant) return amount;`
- D) `FHE.mul(isCompliant, amount)`

<details>
<summary>Show Answer</summary>

✅ **B — `FHE.select(isCompliant, amount, FHE.asEuint64(0))`**

`FHE.select` is the correct pattern for all encrypted conditionals. It evaluates both branches and returns the one matching the encrypted condition.

</details>

---

🎉 **All done?** Move on to [Anti-Patterns & Pitfalls](../anti-patterns/README.md).
