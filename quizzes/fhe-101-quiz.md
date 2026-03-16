# ✅ Quiz — FHE 101

---

**Q1. What is the best plain-language analogy for FHE?**

- A) A transparent box where anyone can see inside
- B) A locked box that can do math without being opened
- C) A shared spreadsheet with password protection
- D) A hash function that irreversibly encodes data

<details>
<summary>Show Answer</summary>

✅ **B — A locked box that can do math without being opened**

You encrypt a value (put it in the box), the blockchain computes on it, and only authorized parties can open the box to see the result.

</details>

---

**Q2. Which function lets a user decrypt an encrypted result?**

- A) `FHE.allowThis(value)`
- B) `FHE.allowTransient(value, contract)`
- C) `FHE.allow(value, userAddress)`
- D) `FHE.makePubliclyDecryptable(value)`

<details>
<summary>Show Answer</summary>

✅ **C — `FHE.allow(value, userAddress)`**

This grants a specific address the right to decrypt that handle off-chain.

</details>

---

**Q3. What is `FHE.allowTransient` used for?**

- A) Permanently storing a handle
- B) Granting one-transaction permission to another contract
- C) Making a value publicly readable
- D) Creating an encrypted value from plaintext

<details>
<summary>Show Answer</summary>

✅ **B — Granting one-transaction permission to another contract**

`allowTransient` is used in multi-contract flows where one contract needs to pass a handle to another for a single transaction only.

</details>

---

**Q4. When is it safe to use `FHE.makePubliclyDecryptable`?**

- A) Always — it's the standard way to return values
- B) Only when the value is intentionally meant to be revealed publicly
- C) Only for values under 64 bits
- D) Only inside constructor functions

<details>
<summary>Show Answer</summary>

✅ **B — Only when the value is intentionally meant to be revealed publicly**

`makePubliclyDecryptable` is irreversible. Use it only for things like auction winners or vote tallies that are meant to be public after a reveal phase.

</details>

---

**Q5. Which of these actions is NOT possible in FHEVM?**

- A) Adding two encrypted values
- B) Comparing an encrypted value to a plaintext threshold
- C) Using an `ebool` directly in a Solidity `if()` statement
- D) Granting decrypt rights to a specific address

<details>
<summary>Show Answer</summary>

✅ **C — Using an `ebool` directly in a Solidity `if()` statement**

`ebool` is an encrypted boolean — it cannot be evaluated by Solidity's control flow. Use `FHE.select()` instead.

</details>

---

🎉 **All done?** Continue to [Quick Start](../basics/quick-start.md) or review [FHE 101](../basics/fhe-101.md).
