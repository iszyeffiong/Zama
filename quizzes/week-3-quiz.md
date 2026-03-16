# ✅ Week 3 Quiz — Advanced Patterns & Real Use Cases

---

**Q1. In a BlindAuction, how is the highest bid tracked without revealing who is winning?**

- A) All bids are stored in plaintext and compared at reveal time
- B) `FHE.gt(newBid, highestBid)` + `FHE.select` updates highest bid branch-free
- C) Bids are hashed and the highest hash wins
- D) The contract owner manually tracks bids off-chain

<details>
<summary>Show Answer</summary>

✅ **B — `FHE.gt` + `FHE.select` updates highest bid branch-free**

`ebool isHigher = FHE.gt(newBid, highestBid)` then `FHE.select(isHigher, newBid, highestBid)` updates the running maximum without revealing relative bid values.

</details>

---

**Q2. What is the main difference between ERC20 and ERC7984?**

- A) ERC7984 only works on Zama Devnet
- B) ERC7984 balances and transfer amounts are `euint64` handles — encrypted
- C) ERC7984 supports NFTs; ERC20 does not
- D) ERC7984 requires KYC for all transfers

<details>
<summary>Show Answer</summary>

✅ **B — ERC7984 balances and transfer amounts are `euint64` handles — encrypted**

ERC7984 is the confidential token standard. All financial values are encrypted, so no one can inspect balances or transfer amounts on-chain.

</details>

---

**Q3. `FHE.allowTransient(handle, swapContract)` was called in TokenA. The swap function runs. After the transaction ends, can swapContract still use the handle?**

- A) Yes — `allowTransient` is permanent once called
- B) No — transient permission expires at end of transaction
- C) Yes — but only for read operations
- D) Only if swapContract called `FHE.allowThis(handle)`

<details>
<summary>Show Answer</summary>

✅ **B — No — transient permission expires at end of transaction**

That's the point of `allowTransient` — single-transaction access. After the tx, the permission is gone. Stored handles in swapContract that were re-granted with `allowThis` remain valid.

</details>

---

**Q4. A compliance check needs: user is KYC'd AND over 18 AND not sanctioned. Which code is correct?**

- A) `require(isKYC[u] && isAdult[u] && !isSanctioned[u])`
- B) `FHE.and(FHE.and(isKYC[u], isAdult[u]), FHE.not(isSanctioned[u]))`
- C) `FHE.select(isKYC[u], FHE.select(isAdult[u], FHE.not(isSanctioned[u]), false), false)`
- D) `FHE.or(isKYC[u], FHE.or(isAdult[u], isSanctioned[u]))`

<details>
<summary>Show Answer</summary>

✅ **B — `FHE.and(FHE.and(isKYC[u], isAdult[u]), FHE.not(isSanctioned[u]))`**

Boolean combinators on `ebool` values are the correct way to combine encrypted conditions. The result is a single `ebool` you can use with `FHE.select`.

</details>

---

🎉 **Score 3/4 or higher?** You're ready for [Week 4](../week-4/overview.md).
