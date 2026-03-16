# Lesson 3.5 — Cross-Contract FHE Flows

**Duration:** 60 minutes | **Format:** Architecture + multi-contract examples

---

## Learning Goals

- Understand why cross-contract FHE requires explicit permission management
- Use allowTransient correctly for handle passing between contracts
- Implement a confidential token swap pattern
- Avoid the most common multi-contract FHE bugs

---

## 1. Why Cross-Contract FHE is Different

In a standard Solidity call, passing a value between contracts is trivial:
```solidity
IToken(tokenAddr).transfer(to, amount); // amount is public — no problem
```

With FHE, passing a `euint64` handle between contracts requires explicit authorization. The receiving contract must have permission to use that handle — and permission is not inherited from the calling contract.

**Without allowTransient:**
```solidity
// ContractA calls ContractB passing a handle
ISwap(swapAddr).execute(balances[msg.sender]);
// ContractB tries to use the handle → REVERTS: "not authorized"
```

**With allowTransient:**
```solidity
FHE.allowTransient(balances[msg.sender], swapAddr); // Grant before calling
ISwap(swapAddr).execute(balances[msg.sender]);       // Now works
```

---

## 2. allowTransient Semantics

```solidity
FHE.allowTransient(handle, targetContract);
```

- Grants `targetContract` the right to use `handle` **within this transaction only**
- After the transaction ends, the permission expires automatically
- The target contract should call `FHE.allowThis(handle)` if it wants to store the handle for future use

---

## 3. ERC7984 Token Swap — Confidential to Confidential

Two confidential tokens exchanged atomically, with all amounts staying encrypted:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.24;

import {FHE, euint64, ebool, externalEuint64} from "@fhevm/solidity/lib/FHE.sol";
import {ZamaEthereumConfig} from "@fhevm/solidity/config/ZamaConfig.sol";

interface IConfidentialToken {
    function transferFrom(address from, address to, euint64 amount) external;
    function balanceOf(address account) external view returns (euint64);
}

contract ConfidentialSwap is ZamaEthereumConfig {

    IConfidentialToken public tokenA;
    IConfidentialToken public tokenB;

    constructor(address _tokenA, address _tokenB) {
        tokenA = IConfidentialToken(_tokenA);
        tokenB = IConfidentialToken(_tokenB);
    }

    /// @notice Swap encrypted amountA of TokenA for encrypted amountB of TokenB
    /// Both users must have pre-approved this contract as an operator on their tokens
    function swap(
        address counterparty,
        externalEuint64 amountAExt, bytes calldata proofA,
        externalEuint64 amountBExt, bytes calldata proofB
    ) external {
        address initiator = msg.sender;

        euint64 amountA = FHE.fromExternal(amountAExt, proofA);
        euint64 amountB = FHE.fromExternal(amountBExt, proofB);

        // Grant transient permission to each token contract for this tx
        FHE.allowTransient(amountA, address(tokenA));
        FHE.allowTransient(amountB, address(tokenB));

        // Atomic swap: initiator sends TokenA, receives TokenB
        tokenA.transferFrom(initiator,    counterparty, amountA);
        tokenB.transferFrom(counterparty, initiator,    amountB);
    }
}
```

---

## 4. Multi-Hop Flow — Three Contracts

Sometimes a handle needs to travel through several contracts in one transaction:

```solidity
// Contract A — Token
function sendToRouter(address router, euint64 amount) external {
    FHE.allowTransient(amount, router);
    IRouter(router).route(amount, msg.sender);
}

// Contract B — Router
function route(euint64 amount, address originator) external {
    // Router re-grants transient permission to the vault
    FHE.allowTransient(amount, address(vault));
    vault.deposit(amount, originator);
}

// Contract C — Vault
function deposit(euint64 amount, address owner) external {
    // Vault wants to keep this handle for future use
    FHE.allowThis(amount);          // Now vault can use it next tx
    FHE.allow(amount, owner);       // Owner can decrypt
    balances[owner] = amount;
}
```

Each hop requires its own `allowTransient` call before passing to the next contract.

---

## 5. Common Bugs in Cross-Contract Flows

### Bug 1 — Forgetting allowTransient

```solidity
// WRONG
IVault(vault).deposit(balances[msg.sender]);  // Vault cannot use the handle

// CORRECT
FHE.allowTransient(balances[msg.sender], vault);
IVault(vault).deposit(balances[msg.sender]);
```

### Bug 2 — Wrong direction for allowTransient

```solidity
// WRONG — granting allowTransient in the receiving contract instead of the caller
contract Vault {
    function deposit(euint64 amount) external {
        FHE.allowTransient(amount, address(this));  // Too late — already here
```

`allowTransient` must be called by the **sending** contract before the call.

### Bug 3 — Missing allowThis in receiving contract

```solidity
// WRONG — vault receives handle but can't use it next tx
contract Vault {
    function deposit(euint64 amount) external {
        balances[msg.sender] = amount;  // No allowThis — next tx read fails
    }
}

// CORRECT
contract Vault {
    function deposit(euint64 amount) external {
        FHE.allowThis(amount);           // Vault retains access
        FHE.allow(amount, msg.sender);   // Owner can decrypt
        balances[msg.sender] = amount;
    }
}
```

### Bug 4 — ERC7984 swap missing operator approval

Before a swap contract can call `transferFrom` on an ERC7984 token, the token holder must approve the swap contract as an operator:

```typescript
// Client-side: Alice approves the swap contract first
const approvalAmount = await fhevm.createEncryptedInput(tokenAAddress, alice.address)
  .add64(1000n).encrypt();
await tokenA.connect(alice).approve(swapAddress, approvalAmount.handles[0], approvalAmount.inputProof);

// Then Alice can swap
await swap.connect(alice).swap(bob.address, ...);
```

---

## 6. Transient vs Permanent Access — Decision Guide

| Scenario | Use |
|---|---|
| Passing handle to another contract this tx | `allowTransient` |
| User decrypting a result off-chain | `allow(handle, user)` |
| Contract storing handle for future txs | `allowThis` |
| Granting auditor permanent read access | `allow(handle, auditor)` |
| One-shot delegation across contracts | `allowTransient` |

---

## 7. Testing Multi-Contract Flows

```typescript
it("swap completes atomically", async () => {
    // Setup: mint tokens to alice and bob
    // Alice mints 100 TokenA, Bob mints 50 TokenB
    // ...

    // Alice approves swap contract
    const aliceApproval = await fhevm.createEncryptedInput(tokenAAddr, alice.address)
      .add64(100n).encrypt();
    await tokenA.connect(alice).approve(swapAddr, aliceApproval.handles[0], aliceApproval.inputProof);

    // Bob approves swap contract
    const bobApproval = await fhevm.createEncryptedInput(tokenBAddr, bob.address)
      .add64(50n).encrypt();
    await tokenB.connect(bob).approve(swapAddr, bobApproval.handles[0], bobApproval.inputProof);

    // Execute swap
    const amtA = await fhevm.createEncryptedInput(swapAddr, alice.address).add64(100n).encrypt();
    const amtB = await fhevm.createEncryptedInput(swapAddr, alice.address).add64(50n).encrypt();
    await swap.connect(alice).swap(bob.address,
        amtA.handles[0], amtA.inputProof,
        amtB.handles[0], amtB.inputProof
    );

    // Verify: Alice now has 50 TokenB, Bob has 100 TokenA
    const aliceB = await tokenB.balanceOf(alice.address);
    const bobA   = await tokenA.balanceOf(bob.address);

    const aliceBDecrypted = await fhevm.userDecryptEuint(FhevmType.euint64, aliceB, tokenBAddr, alice);
    const bobADecrypted   = await fhevm.userDecryptEuint(FhevmType.euint64, bobA,   tokenAAddr, bob);

    expect(aliceBDecrypted).to.equal(50n);
    expect(bobADecrypted).to.equal(100n);
});
```

---

## Hands-On Exercise (15 min)

Build a ConfidentialEscrow using two contracts:
- `Escrow.sol` — holds an encrypted amount deposited by Alice
- `EscrowRelease.sol` — authorized by owner to release to Bob

The release contract must use `allowTransient` to pass the handle from the escrow to Bob's balance.

---

📋 **Week 3 complete!** → [Take the Week 3 Quiz](../../quizzes/week-3-quiz.md) then submit [Week 3 Homework](../homework/week-3-homework.md)
