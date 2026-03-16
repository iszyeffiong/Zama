# Week 1 Homework — PrivateVault

**Released:** End of Day 5 | **Due:** Before Week 2 starts | **Estimated time:** 3–4 hours

---

## Overview

Build a `PrivateVault` contract — a single-owner encrypted savings vault where only the owner knows their balance.

This homework consolidates everything from Week 1:
- Encrypted state management
- Handle lifecycle (create → allowThis → allow → store)
- FHE arithmetic (add, sub)
- FHE comparison (ge) with branch-free logic
- User decryption pattern

---

## Specification

### Contract: `PrivateVault.sol`

The vault must implement the following interface:

```solidity
interface IPrivateVault {
    // Deposit an encrypted amount into the vault
    function deposit(externalEuint64 amount, bytes calldata proof) external;

    // Withdraw an encrypted amount (must not exceed balance)
    function withdraw(externalEuint64 amount, bytes calldata proof) external;

    // Returns the caller's encrypted balance handle
    function getBalance() external view returns (euint64);

    // Returns encrypted bool: is caller's balance >= threshold?
    function isAboveThreshold(uint64 threshold) external view returns (ebool);
}
```

### Requirements

1. **Each user has their own private balance** — `mapping(address => euint64)`
2. **Deposit** adds to the caller's encrypted balance
3. **Withdraw** subtracts from the balance, but **only if balance >= amount** — use `FHE.select` to handle this branch-free (do not revert if insufficient — instead transfer zero)
4. **getBalance** returns the caller's encrypted handle (the caller must already have been granted `FHE.allow`)
5. **isAboveThreshold** computes `FHE.ge(balance, FHE.asEuint64(threshold))` without revealing the balance
6. All handles must have correct `allowThis` and `allow` grants

---

## Starter Code

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.24;

import {FHE, euint64, ebool, externalEuint64} from "@fhevm/solidity/lib/FHE.sol";
import {ZamaEthereumConfig} from "@fhevm/solidity/config/ZamaConfig.sol";

contract PrivateVault is ZamaEthereumConfig {
    mapping(address => euint64) private balances;

    function deposit(externalEuint64 amount, bytes calldata proof) external {
        // TODO: implement
    }

    function withdraw(externalEuint64 amount, bytes calldata proof) external {
        // TODO: implement — use FHE.select for branch-free safe withdrawal
    }

    function getBalance() external view returns (euint64) {
        // TODO: implement
    }

    function isAboveThreshold(uint64 threshold) external view returns (ebool) {
        // TODO: implement
    }
}
```

---

## Required Tests

Your test file must cover all of the following. Each test is worth points (see grading below).

```typescript
describe("PrivateVault", () => {
  it("starts with zero balance")
  it("deposit increases balance correctly")
  it("two deposits accumulate correctly")
  it("withdraw reduces balance correctly")
  it("withdraw of exact balance results in zero")
  it("withdraw more than balance results in zero balance change") // FHE.select behavior
  it("isAboveThreshold returns true when balance is above")
  it("isAboveThreshold returns false when balance is below")
  it("user cannot decrypt another user's balance") // permissions test
  it("second user has independent balance")
})
```

---

## Grading Criteria

| Criteria | Points |
|---|---|
| Contract compiles without errors | 10 |
| `deposit` correctly adds to balance with proper permissions | 15 |
| `withdraw` uses `FHE.select` for branch-free logic | 20 |
| `withdraw` does not reduce balance below zero | 10 |
| `getBalance` returns correct handle with proper permissions | 10 |
| `isAboveThreshold` correctly uses `FHE.ge` | 10 |
| All 10 required test cases pass | 20 |
| Code quality: comments, clear naming, no unused variables | 5 |
| **Total** | **100** |

### Bonus (up to 15 extra points)
- Add an `emergencyReset` function that sets balance to zero (5 pts)
- Add an `allowAuditor(address)` function so owner can grant a third party read access (10 pts)

---

## Submission

Push your solution to a GitHub repo and submit the link. Your repo must contain:

```
contracts/PrivateVault.sol
test/PrivateVault.ts
README.md  (brief explanation of your design choices)
```

---

## Hints

> Stuck on the branch-free withdraw? Think: `newBalance = FHE.select(hasEnough, FHE.sub(balance, amount), balance)`

> Remember: after every `FHE.add` / `FHE.sub` / `FHE.select`, you get a new handle. Grant `allowThis` and `allow` on the **new** handle before storing.

> `isAboveThreshold` doesn't need `allowThis` if you're not storing the result — but the caller needs to have been granted `allow` on the balance for it to work.

---

✅ **[Take the Week 1 Quiz](https://forms.gle/Q7o3oJXBw253x1xw9)** before submitting your homework.

---

## 📝 Before You Submit

Complete the Week 1 Quiz on Tally first:

👉 [Open Week 1 Quiz](https://forms.gle/Q7o3oJXBw253x1xw9)

Paste your quiz score in your homework submission README.


