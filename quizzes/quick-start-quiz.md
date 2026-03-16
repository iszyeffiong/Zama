# ✅ Quiz — Quick Start

---

**Q1. What is the minimum Node.js version required for `create-fhevm-example`?**

- A) v16.0.0
- B) v18.0.0
- C) v20.0.0
- D) v22.0.0

<details>
<summary>Show Answer</summary>

✅ **C — v20.0.0**

Node.js v20 or higher is required. The latest LTS version is recommended.

</details>

---

**Q2. Which command scaffolds a single blind auction example project?**

- A) `npx create-fhevm-example --add blind-auction`
- B) `npx create-fhevm-example --category blind-auction`
- C) `npx create-fhevm-example blind-auction`
- D) `npm init fhevm blind-auction`

<details>
<summary>Show Answer</summary>

✅ **C — `npx create-fhevm-example blind-auction`**

Pass the example name directly for single example mode.

</details>

---

**Q3. What does the `--add` flag do?**

- A) Adds a new example to an existing GitBook
- B) Injects FHE dependencies into an existing Hardhat project
- C) Adds a new network to the config
- D) Appends test cases to an existing test file

<details>
<summary>Show Answer</summary>

✅ **B — Injects FHE dependencies into an existing Hardhat project**

Running `npx create-fhevm-example --add` inside a Hardhat project adds `@fhevm/solidity`, updates `hardhat.config.ts`, and injects a sample contract.

</details>

---

**Q4. Which category contains examples for Poker, Lottery, and Blackjack?**

- A) `advanced`
- B) `openzeppelin`
- C) `concepts`
- D) `gaming`

<details>
<summary>Show Answer</summary>

✅ **D — `gaming`**

The `gaming` category bundles 4 examples: Poker, Lottery, Rock Paper Scissors, and Blackjack.

</details>

---

**Q5. Do you need to install `create-fhevm-example` globally before using it?**

- A) Yes — `npm install -g create-fhevm-example` is required
- B) No — `npx` runs it directly without a global install
- C) Yes — but only on Windows
- D) No — it comes bundled with Hardhat

<details>
<summary>Show Answer</summary>

✅ **B — No — `npx` runs it directly without a global install**

`npx` (included with Node.js) handles everything. No global installation needed.

</details>

---

🎉 **All done?** Move on to [Core Concepts](../core-concepts/encrypted-types.md).
