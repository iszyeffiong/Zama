# ✅ Quiz  What is FHEVM?

Test your understanding before moving on.

---

**Q1. What does FHEVM stand for?**

- A) Fast Hashing Ethereum Virtual Machine
- B) Fully Homomorphic Encryption Virtual Machine
- C) Federated Hybrid Execution Virtual Module
- D) Functional Hash Encryption Verification Model

<details>
<summary>Show Answer</summary>

✅ **B  Fully Homomorphic Encryption Virtual Machine**

FHEVM is a system that allows smart contracts to compute on encrypted data without revealing it.

</details>

---

**Q2. What is the role of the FHE coprocessor?**

- A) It stores the Solidity source code
- B) It validates Ethereum transactions
- C) It stores ciphertexts and executes FHE operations off-chain
- D) It generates wallet private keys

<details>
<summary>Show Answer</summary>

✅ **C  It stores ciphertexts and executes FHE operations off-chain**

Smart contracts submit operation requests to the coprocessor, which computes on encrypted data and returns new encrypted handles.

</details>

---

**Q3. Which of the following is a valid FHEVM use case?**

- A) Making ERC20 token names private
- B) Hiding the Solidity contract bytecode
- C) Processing encrypted auction bids without revealing amounts
- D) Encrypting the gas price of a transaction

<details>
<summary>Show Answer</summary>

✅ **C  Processing encrypted auction bids without revealing amounts**

FHEVM is designed to keep sensitive values like bids, votes, balances, and identity attributes private on-chain.

</details>

---

**Q4. When does decryption happen in FHEVM?**

- A) Inside the smart contract during execution
- B) Always off-chain by an authorized party
- C) Automatically after every transaction
- D) Only by the contract owner

<details>
<summary>Show Answer</summary>

✅ **B  Always off-chain by an authorized party**

Contracts store and operate on handles. Decryption is done off-chain by whoever holds the correct permission.

</details>

---

🎉 **All done?** Head back to [What is FHEVM?](../basics/what-is-fhevm.md) if you need a refresher, or continue to [FHE 101](../basics/fhe-101.md).
