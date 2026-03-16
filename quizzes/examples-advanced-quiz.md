# ✅ Quiz  Advanced Examples

Covers: BlindAuction, HiddenVoting, AMM, ERC7984.

---

**Q1. In `BlindAuction`, how is the highest bid tracked without revealing individual bids?**

- A) All bids are stored in plaintext and compared at reveal time
- B) `FHE.gt` + `FHE.select` updates the highest bid in a branch-free way
- C) Bids are hashed and compared using `keccak256`
- D) The auctioneer manually tracks bids off-chain

<details>
<summary>Show Answer</summary>

✅ **B  `FHE.gt` + `FHE.select` updates the highest bid in a branch-free way**

`ebool isHigher = FHE.gt(newBid, highestBid)` then `FHE.select(isHigher, newBid, highestBid)` updates the highest bid without revealing which bid is currently winning.

</details>

---

**Q2. In `HiddenVoting`, a voter submits `1` for Yes and `0` for No. How are totals accumulated?**

- A) Votes are stored and tallied at the end using a loop
- B) `FHE.add(votesFor, vote)` and `FHE.add(votesAgainst, FHE.sub(1, vote))` on every submission
- C) Votes are hashed and decoded at reveal time
- D) A trusted oracle tallies the votes off-chain

<details>
<summary>Show Answer</summary>

✅ **B  `FHE.add` increments the running totals on every vote submission**

Adding `vote` (1 or 0) to `votesFor` and `1 - vote` to `votesAgainst` keeps both tallies updated without revealing individual votes.

</details>

---

**Q3. Why does `FHEAMMSimple` use public reserves instead of encrypted reserves?**

- A) Encrypted reserves would make swaps slower
- B) FHE.div requires a plaintext denominator  encrypted reserves would make the denominator encrypted
- C) Public reserves are required by the ERC7984 standard
- D) The Zama Devnet doesn't support encrypted state variables

<details>
<summary>Show Answer</summary>

✅ **B  FHE.div requires a plaintext denominator  encrypted reserves would make the denominator encrypted**

The constant product formula `amountOut = (reserveB × amountIn) / (reserveA + amountIn)` has an encrypted denominator when `amountIn` is encrypted. Since `FHE.div(encrypted, encrypted)` is not supported, reserves must be public.

</details>

---

**Q4. What is the main difference between ERC20 and ERC7984?**

- A) ERC7984 supports NFTs; ERC20 does not
- B) ERC7984 balances and transfer amounts are encrypted; ERC20 are public
- C) ERC7984 uses proof-of-stake; ERC20 uses proof-of-work
- D) ERC7984 is only usable on Zama Devnet

<details>
<summary>Show Answer</summary>

✅ **B  ERC7984 balances and transfer amounts are encrypted; ERC20 are public**

ERC7984 is the confidential token standard for FHEVM. All financial values are `euint64` handles instead of public `uint256` values.

</details>

---

**Q5. In `VestingWalletConfidentialExample`, what gates the claim function?**

- A) A timelock encoded in the encrypted vesting schedule
- B) A KYC check (`require(kyc[msg.sender])`) plus the vesting release condition
- C) A multisig approval from 3 of 5 owners
- D) A public decryption of the vested amount

<details>
<summary>Show Answer</summary>

✅ **B  A KYC check plus the vesting release condition**

The claim function requires both that the caller is KYC-verified and that tokens have vested, combining compliance and time-based logic.

</details>

---

🎉 **All done?** Head to the [Full Example Map](../resources/example-map.md) to explore what's next.
