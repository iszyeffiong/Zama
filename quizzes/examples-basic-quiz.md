# ✅ Quiz  Basic Examples

---

**Q1. In `EncryptSingleValue`, what is the correct order of operations after `FHE.fromExternal`?**

- A) Store → `allowThis` → `allow(user)`
- B) `allowThis` → `allow(user)` → Store
- C) `allow(user)` → Store → `allowThis`
- D) Order doesn't matter

<details>
<summary>Show Answer</summary>

✅ **B  `allowThis` → `allow(user)` → Store**

Always grant permissions before storing the handle. If you store first, the handle is in state but unauthorized for future use.

</details>

---

**Q2. In `FHECounter`, after calling `FHE.add(counter, one)`, do you need to call `FHE.allowThis` again?**

- A) No  the existing `allowThis` on `counter` carries over
- B) Yes  `FHE.add` creates a new handle that has no permissions
- C) Only if the counter exceeds 100
- D) Only on the first increment

<details>
<summary>Show Answer</summary>

✅ **B  Yes  `FHE.add` creates a new handle that has no permissions**

Every FHE operation produces a brand new handle. You must re-grant `allowThis` (and `allow` if needed) on every new handle before storing it.

</details>

---

**Q3. `FHEIfThenElse` uses `FHE.select(condition, a, b)`. Are both `a` and `b` computed?**

- A) No  only the branch matching the condition is computed
- B) Yes  both branches are always evaluated
- C) Only `a` is computed; `b` is a fallback
- D) It depends on the gas limit

<details>
<summary>Show Answer</summary>

✅ **B  Yes  both branches are always evaluated**

`FHE.select` is branch-free. Both values are computed and the encrypted condition selects which result to return, without revealing which was chosen.

</details>

---

**Q4. In `UserDecryptSingleValue`, what does the client need to decrypt a handle?**

- A) The contract's private key
- B) The original plaintext that was encrypted
- C) The handle, contract address, and a signer with `FHE.allow` permission
- D) Only the handle  no additional info needed

<details>
<summary>Show Answer</summary>

✅ **C  The handle, contract address, and a signer with `FHE.allow` permission**

The FHEVM SDK requires the handle, contract address, and a signer that was previously granted `FHE.allow` by the contract.

</details>

---

🎉 **All done?** Explore [Games Examples](../examples/games/fhe-wordle.md) or jump to [Auctions](../examples/auctions/blind-auction.md).
