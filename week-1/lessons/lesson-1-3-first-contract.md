# Lesson 1.3 — Your First Encrypted Contract

**Duration:** 90 minutes | **Format:** Live coding / hands-on lab

---

## Learning Goals

- Set up a FHEVM Hardhat project from scratch
- Write a complete encrypted contract end-to-end
- Write TypeScript tests that encrypt inputs and decrypt outputs
- Run tests locally using the FHEVM mock

---

## Part 1 — Environment Setup (20 min)

### Scaffold your project

```bash
npx create-fhevm-example encrypt-single-value
cd encrypt-single-value
npm install
npm run compile
npm run test
```

All tests should pass. You now have a working FHEVM environment.

### What was installed

- `@fhevm/solidity` — encrypted types and FHE operations for Solidity
- `@fhevm/hardhat-plugin` — local mock coprocessor for testing
- Pre-configured `hardhat.config.ts` with FHE network settings

---

## Part 2 — Anatomy of an Encrypted Contract (30 min)

Let's build `EncryptedCounter.sol` from scratch:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.24;

import {FHE, euint64} from "@fhevm/solidity/lib/FHE.sol";
import {ZamaEthereumConfig} from "@fhevm/solidity/config/ZamaConfig.sol";

contract EncryptedCounter is ZamaEthereumConfig {
    // euint64 stores a HANDLE — not the number itself
    euint64 private counter;
    address public owner;

    constructor() {
        owner = msg.sender;
        // Initialize counter to 0
        counter = FHE.asEuint64(0);
        FHE.allowThis(counter);  // Contract must have permission to use its own state
    }

    function increment() external {
        euint64 one = FHE.asEuint64(1);
        euint64 newCounter = FHE.add(counter, one);  // New handle created!

        // Re-grant on the NEW handle — permissions don't transfer
        FHE.allowThis(newCounter);
        counter = newCounter;
    }

    function decrement() external {
        euint64 one = FHE.asEuint64(1);
        euint64 newCounter = FHE.sub(counter, one);
        FHE.allowThis(newCounter);
        counter = newCounter;
    }

    // Grant owner decrypt rights
    function allowOwnerRead() external {
        require(msg.sender == owner, "Not owner");
        FHE.allow(counter, owner);
    }

    // Returns handle — NOT a number. Decrypt off-chain.
    function getCounter() external view returns (euint64) {
        return counter;
    }
}
```

### Key observations

1. `counter` holds a handle, not a number
2. Every `FHE.add` / `FHE.sub` creates a new handle → must `allowThis` again
3. `getCounter()` returns a handle — the client decrypts it off-chain
4. Permissions are explicit and per-handle

---

## Part 3 — Writing Tests (30 min)

```typescript
import { ethers, fhevm } from "hardhat";
import { FhevmType } from "@fhevm/hardhat-plugin";
import { expect } from "chai";

describe("EncryptedCounter", function () {
  let contract: any;
  let owner: any;
  let contractAddress: string;

  beforeEach(async () => {
    [owner] = await ethers.getSigners();
    const Factory = await ethers.getContractFactory("EncryptedCounter");
    contract = await Factory.deploy();
    contractAddress = await contract.getAddress();
  });

  it("starts at zero", async () => {
    await contract.allowOwnerRead();
    const handle = await contract.getCounter();

    // Decrypt off-chain using FHEVM SDK
    const value = await fhevm.userDecryptEuint(
      FhevmType.euint64,
      handle,
      contractAddress,
      owner
    );
    expect(value).to.equal(0n);
  });

  it("increments correctly", async () => {
    await contract.increment();
    await contract.increment();
    await contract.increment();
    await contract.allowOwnerRead();

    const handle = await contract.getCounter();
    const value = await fhevm.userDecryptEuint(
      FhevmType.euint64, handle, contractAddress, owner
    );
    expect(value).to.equal(3n);
  });

  it("decrements correctly", async () => {
    await contract.increment();
    await contract.increment();
    await contract.decrement();
    await contract.allowOwnerRead();

    const handle = await contract.getCounter();
    const value = await fhevm.userDecryptEuint(
      FhevmType.euint64, handle, contractAddress, owner
    );
    expect(value).to.equal(1n);
  });
});
```

---

## Part 4 — Submitting Encrypted Inputs (10 min)

What if we want users to increment by a custom encrypted amount?

```solidity
function incrementBy(externalEuint64 amountExt, bytes calldata proof) external {
    euint64 amount = FHE.fromExternal(amountExt, proof);
    euint64 newCounter = FHE.add(counter, amount);
    FHE.allowThis(newCounter);
    counter = newCounter;
}
```

```typescript
it("increments by encrypted amount", async () => {
  // Encrypt the amount client-side, bound to THIS contract + THIS sender
  const encrypted = await fhevm
    .createEncryptedInput(contractAddress, owner.address)
    .add64(5n)
    .encrypt();

  await contract.incrementBy(encrypted.handles[0], encrypted.inputProof);
  await contract.allowOwnerRead();

  const handle = await contract.getCounter();
  const value = await fhevm.userDecryptEuint(
    FhevmType.euint64, handle, contractAddress, owner
  );
  expect(value).to.equal(5n);
});
```

---

## Common Mistakes to Watch For

| Mistake | Symptom | Fix |
|---|---|---|
| Forget `allowThis` after `FHE.add` | Next operation fails silently | Always re-grant on new handles |
| Return plaintext from encrypted value | Wrong type / revert | Return `euint64` handle, decrypt client-side |
| Wrong contract address in proof | `fromExternal` reverts | Use exact deployed contract address |

---

📖 **Next:** [Lesson 1.4 — FHE Operations Reference](lesson-1-4-fhe-operations.md)
