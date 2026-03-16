# ✅ Quiz  Core Concepts

Covers: Encrypted Types, Handles, Input Proofs, Access Control, Decryption Patterns.

---

**Q1. What does a `euint64` variable actually store in Solidity?**

- A) A plaintext 64-bit number
- B) An AES-encrypted number
- C) An opaque 256-bit handle pointing to a ciphertext
- D) A hash of the encrypted value

<details>
<summary>Show Answer</summary>

✅ **C  An opaque 256-bit handle pointing to a ciphertext**

Encrypted types are handles  references to ciphertexts stored by the FHE coprocessor. They are not the values themselves.

</details>

---

**Q2. What happens if you store a `euint64` handle without calling `FHE.allowThis()`?**

- A) The handle is automatically re-authorized on the next transaction
- B) The contract will revert immediately
- C) Future operations on that handle will fail silently
- D) The user loses decrypt rights but the contract can still use it

<details>
<summary>Show Answer</summary>

✅ **C  Future operations on that handle will fail silently**

Without `allowThis`, the FHE coprocessor does not recognize the contract as authorized. This is the most common FHEVM mistake.

</details>

---

**Q3. You call `FHE.add(a, b)` which produces handle `c`. Do the permissions from `a` and `b` transfer to `c`?**

- A) Yes  permissions always propagate to derived handles
- B) Yes  but only `allowThis` propagates, not `allow(user)`
- C) No  you must explicitly grant permissions on every new handle
- D) Only if `a` and `b` have identical permissions

<details>
<summary>Show Answer</summary>

✅ **C  No  you must explicitly grant permissions on every new handle**

Every FHE operation creates a brand new handle. Permissions never propagate automatically. Always re-grant `allowThis` and `allow` on new handles.

</details>

---

**Q4. What does an input proof cryptographically bind?**

- A) The encrypted value to a specific block number
- B) The encrypted value to a specific contract address and sender address
- C) The encrypted value to the deployer's private key
- D) The encrypted value to the gas price of the transaction

<details>
<summary>Show Answer</summary>

✅ **B  The encrypted value to a specific contract address and sender address**

Input proofs prevent replay attacks. A proof generated for Contract A by Alice will be rejected if submitted to Contract B or by Bob.

</details>

---

**Q5. Which function is correct for a contract to pass a handle to another contract within the same transaction?**

- A) `FHE.allow(handle, otherContract)`
- B) `FHE.allowThis(handle)`
- C) `FHE.allowTransient(handle, otherContract)`
- D) `FHE.makePubliclyDecryptable(handle)`

<details>
<summary>Show Answer</summary>

✅ **C  `FHE.allowTransient(handle, otherContract)`**

`allowTransient` grants permission that lasts only for the current transaction  perfect for cross-contract flows.

</details>

---

**Q6. A view function returns a `euint64`. What does the caller receive?**

- A) The decrypted plaintext value
- B) An encrypted handle they must decrypt off-chain
- C) A revert, because view functions cannot return encrypted types
- D) Zero, because view functions cannot access the FHE coprocessor

<details>
<summary>Show Answer</summary>

✅ **B  An encrypted handle they must decrypt off-chain**

View functions return handles. Decryption always happens off-chain using the FHEVM SDK.

</details>

---

🎉 **All done?** Move on to [FHE Operations](../fhe-operations/arithmetic.md).
