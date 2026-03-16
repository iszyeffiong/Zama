# ✅ Week 4 Quiz — Production DeFi & Capstone Readiness

---

**Q1. Why does FHEAMMSimple use public reserves instead of encrypted reserves?**

- A) Encrypted reserves would make swaps slower
- B) `FHE.div` requires a plaintext denominator — encrypted reserves make the denominator encrypted
- C) ERC7984 requires public reserve tracking
- D) The Zama coprocessor cannot store more than 2 encrypted values per contract

<details>
<summary>Show Answer</summary>

✅ **B — `FHE.div` requires a plaintext denominator**

The constant product formula `amountOut = (reserveB × amountIn) / (reserveA + amountIn)` produces an encrypted denominator when `amountIn` is encrypted. Since `FHE.div(encrypted, encrypted)` is unsupported, reserves must stay public in this design.

</details>

---

**Q2. Which of these is the production workaround for fully confidential AMM reserves?**

- A) Use `FHE.rem` instead of `FHE.div`
- B) Store masked numerator/denominator; off-chain oracle decrypts the masked denominator; settle with proof
- C) Hash the reserves with keccak256 before storing
- D) Use `FHE.select` to approximate division

<details>
<summary>Show Answer</summary>

✅ **B — Masked fractions with off-chain oracle decryption**

This is the production pattern used by protocols like EPOOL. It keeps reserves confidential while enabling correct division.

</details>

---

**Q3. Which of these makes a capstone project production-ready?**

- A) It compiles without errors
- B) Full NatSpec docs, 15+ tests, no anti-patterns, deployment script, and architecture README
- C) It uses at least one `FHE.add`
- D) It is deployed on mainnet

<details>
<summary>Show Answer</summary>

✅ **B — Full NatSpec docs, 15+ tests, no anti-patterns, deployment script, and architecture README**

Production readiness means the project could be handed to another developer and immediately understood, deployed, and extended.

</details>

---

**Q4. Your capstone uses `FHE.div` to calculate interest. The denominator (interest rate) changes per user. How do you handle this?**

- A) Store the rate as `euint64` and use `FHE.div(principal, rate)`
- B) Store the rate as a public `uint64` per user and use `FHE.div(principal, rate)`
- C) Only support a fixed global rate stored as plaintext
- D) Approximate with repeated `FHE.sub` until zero

<details>
<summary>Show Answer</summary>

✅ **B — Store the rate as a public `uint64` per user and use `FHE.div(principal, rate)`**

The divisor must always be plaintext. User-specific rates can be stored publicly as `uint64` while the principal remains encrypted. This is a deliberate design tradeoff.

</details>

---

🎉 **Score 3/4 or higher?** You're ready to submit your [Capstone Project](../week-4/homework/week-4-capstone.md). Good luck!
