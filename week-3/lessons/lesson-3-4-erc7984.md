# Lesson 3.4 — OpenZeppelin Confidential & ERC7984

**Duration:** 60 minutes | **Format:** Standard walkthrough + integration examples

---

## Learning Goals

- Understand ERC7984 as the confidential equivalent of ERC20
- Use OpenZeppelin Confidential library contracts
- Implement KYC-restricted, observer-access, and wrapper token patterns
- Know when to use ERC7984 vs plain euint64 balances

---

## 1. Why ERC7984 Exists

You could build a confidential token from scratch using `mapping(address => euint64)` and manual FHE operations. But you would need to reinvent:
- Allowance management (encrypted spending limits)
- Safe transfer validation
- Operator patterns
- Standard interface for wallets and DApps to interact with

ERC7984 is the standardised answer — the ERC20 of confidential tokens.

---

## 2. ERC20 vs ERC7984 Side by Side

```solidity
// ERC20 — everything public
function transfer(address to, uint256 amount) external returns (bool);
function balanceOf(address account) external view returns (uint256);
function approve(address spender, uint256 amount) external returns (bool);

// ERC7984 — amounts and balances encrypted
function transfer(address to, externalEuint64 amount, bytes calldata proof) external;
function balanceOf(address account) external view returns (euint64);  // returns handle
function approve(address spender, externalEuint64 amount, bytes calldata proof) external;
```

Key differences:
- `balanceOf` returns a `euint64` handle — decrypt off-chain
- `transfer` takes an encrypted amount with input proof
- `approve` takes an encrypted allowance with input proof
- Events emit encrypted handles — amounts are never public

---

## 3. Installation

```bash
npm install @openzeppelin/confidential-contracts
```

```solidity
import {ERC7984} from "@openzeppelin/confidential-contracts/token/ERC7984/ERC7984.sol";
import {ERC7984Restricted} from "@openzeppelin/confidential-contracts/token/ERC7984/ERC7984Restricted.sol";
```

---

## 4. Minimal ERC7984 Token

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.24;

import {ERC7984}           from "@openzeppelin/confidential-contracts/token/ERC7984/ERC7984.sol";
import {FHE, euint64, externalEuint64} from "@fhevm/solidity/lib/FHE.sol";
import {ZamaEthereumConfig} from "@fhevm/solidity/config/ZamaConfig.sol";

contract ConfidentialToken is ERC7984, ZamaEthereumConfig {
    address public owner;

    constructor() ERC7984("ConfToken", "CTK") {
        owner = msg.sender;
    }

    /// @notice Mint encrypted tokens to an address
    function mint(
        address to,
        externalEuint64 encryptedAmount,
        bytes calldata proof
    ) external {
        require(msg.sender == owner, "Not owner");
        euint64 amount = FHE.fromExternal(encryptedAmount, proof);
        _mint(to, amount);
        // _mint internally handles allowThis and allow(to)
    }
}
```

Usage client-side:
```typescript
// Mint 1000 tokens to Alice
const encrypted = await fhevm
  .createEncryptedInput(tokenAddress, deployer.address)
  .add64(1000n)
  .encrypt();

await token.mint(alice.address, encrypted.handles[0], encrypted.inputProof);

// Alice reads her balance
const handle  = await token.balanceOf(alice.address);
const balance = await fhevm.userDecryptEuint(FhevmType.euint64, handle, tokenAddress, alice);
console.log("Alice balance:", balance); // 1000n
```

---

## 5. KYC-Restricted Token

```solidity
import {ERC7984Restricted} from "@openzeppelin/confidential-contracts/token/ERC7984/ERC7984Restricted.sol";

contract KYCToken is ERC7984Restricted, ZamaEthereumConfig {
    mapping(address => bool) public kyc;
    address public compliance;

    constructor() ERC7984Restricted("KYCToken", "KYC") {
        compliance = msg.sender;
    }

    function approveKYC(address account) external {
        require(msg.sender == compliance, "Not compliance");
        kyc[account] = true;
    }

    function revokeKYC(address account) external {
        require(msg.sender == compliance, "Not compliance");
        kyc[account] = false;
    }

    // Hook called before every transfer — revert-based compliance
    function _beforeTokenTransfer(
        address from,
        address to,
        euint64 /* amount */
    ) internal override {
        if (from != address(0)) require(kyc[from], "Sender not KYC");
        if (to   != address(0)) require(kyc[to],   "Recipient not KYC");
    }

    function mint(address to, externalEuint64 amtExt, bytes calldata proof) external {
        require(msg.sender == compliance, "Not compliance");
        require(kyc[to], "Recipient not KYC");
        _mint(to, FHE.fromExternal(amtExt, proof));
    }
}
```

---

## 6. Observer Access — Audit Without Ownership

Observer access lets a compliance officer or auditor read balances without being able to spend them:

```solidity
import {ERC7984ObserverAccess} from "@openzeppelin/confidential-contracts/token/ERC7984/extensions/ERC7984ObserverAccess.sol";

contract AuditableToken is ERC7984ObserverAccess, ZamaEthereumConfig {
    constructor() ERC7984ObserverAccess("AuditToken", "AUD") {}

    // Account holder adds an observer
    function addMyObserver(address observer) external {
        _addObserver(msg.sender, observer);
        // Observer will automatically receive FHE.allow on future balance updates
    }

    function removeMyObserver(address observer) external {
        _removeObserver(msg.sender, observer);
        // Future balance updates will NOT grant observer access
        // Old handles they already received remain decryptable
    }
}
```

---

## 7. ERC20 ↔ ERC7984 Wrapper

Wrap a public ERC20 into a confidential ERC7984, and unwrap back:

```solidity
import {ERC7984ERC20Wrapper} from "@openzeppelin/confidential-contracts/token/ERC7984/ERC7984ERC20Wrapper.sol";

contract ConfidentialUSDC is ERC7984ERC20Wrapper, ZamaEthereumConfig {
    constructor(IERC20 _usdc)
        ERC7984ERC20Wrapper(_usdc, "Confidential USDC", "cUSDC")
    {}
}
```

Usage:
```typescript
// Wrap 100 USDC into 100 cUSDC
await usdc.approve(wrapperAddress, 100);
await wrapper.depositFor(alice.address, 100);

// Alice now has an encrypted cUSDC balance
// To unwrap, Alice submits an encrypted withdrawal amount
const encrypted = await fhevm.createEncryptedInput(wrapperAddress, alice.address)
  .add64(50n).encrypt();
await wrapper.connect(alice).withdrawTo(alice.address, encrypted.handles[0], encrypted.inputProof);
```

---

## 8. When to Use ERC7984 vs Manual euint64

| Situation | Use |
|---|---|
| Building a token for wallets/DApps to integrate | ERC7984 |
| Simple internal balance tracking in a larger contract | Manual euint64 mapping |
| Need transfer events and allowance management | ERC7984 |
| Only the contract itself reads balances | Manual euint64 mapping |
| Want KYC/compliance hooks | ERC7984Restricted |
| Want auditor access | ERC7984ObserverAccess |

---

## Hands-On Exercise (15 min)

Build a ConfidentialPaymentSplitter:
- Inherits from ERC7984
- Owner deposits tokens to the contract
- Owner calls split(address[] recipients, externalEuint64[] amounts) to distribute
- Each recipient can read and withdraw their allocated amount
- All amounts stay encrypted throughout

---

📖 **Next:** [Lesson 3.5 — Cross-Contract FHE Flows](lesson-3-5-cross-contract.md)
