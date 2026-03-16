# Lesson 2.2 — Access Control & Permissions

**Duration:** 60 minutes | **Format:** Reading + whiteboard exercises

---

## Learning Goals

- Master all three permission functions and know when to use each
- Design multi-user permission flows correctly
- Understand the revocation model and its limitations
- Implement observer/auditor access patterns

---

## 1. The Three Permission Functions

### `FHE.allowThis(handle)` — contract self-access

Grants the contract permission to use this handle in future transactions.

```solidity
euint64 result = FHE.add(a, b);
FHE.allowThis(result);   // Contract can use result in next tx
storedResult = result;
```

**When to call:** Always, before storing any handle you plan to use again.

**What happens without it:** The next transaction that tries to read or operate on the stored handle fails silently.

---

### `FHE.allow(handle, address)` — user decrypt access

Grants a specific address the right to decrypt this handle off-chain.

```solidity
FHE.allow(result, msg.sender);      // Caller can decrypt
FHE.allow(result, adminAddress);    // Admin can also decrypt
FHE.allow(result, address(0));      // ❌ Dangerous — do not do this
```

**When to call:** Whenever a user or authorized party needs to read the value.

**What happens without it:** The user receives a handle they cannot decrypt — `userDecryptEuint` will fail.

---

### `FHE.allowTransient(handle, contractAddress)` — one-transaction cross-contract

Grants another contract usage rights for the duration of the **current transaction only**.

```solidity
FHE.allowTransient(balances[msg.sender], swapContract);
ISwap(swapContract).execute(balances[msg.sender]);
// After this tx ends, swapContract loses access
```

**When to call:** When passing a handle to another contract in a multi-contract flow.

**What happens without it:** The receiving contract reverts with an authorization error.

---

## 2. The Full Permission Lifecycle

```
┌─────────────────────────────────────────────────────┐
│  FHE.fromExternal() or FHE.add() → new handle       │
│                                                      │
│  FHE.allowThis(handle)   ← Contract future access   │
│  FHE.allow(handle, user) ← User off-chain decrypt   │
│                                                      │
│  store in state / return to caller                  │
│                                                      │
│  Next tx: operate → new handle → re-grant → repeat  │
└─────────────────────────────────────────────────────┘
```

---

## 3. Multi-User Contracts

When multiple users each have their own encrypted state:

```solidity
contract MultiUserVault is ZamaEthereumConfig {
    mapping(address => euint64) private balances;

    function deposit(externalEuint64 amount, bytes calldata proof) external {
        euint64 amt = FHE.fromExternal(amount, proof);
        euint64 newBal = FHE.add(balances[msg.sender], amt);

        FHE.allowThis(newBal);
        FHE.allow(newBal, msg.sender);  // Only THIS user can read their balance

        balances[msg.sender] = newBal;
    }

    function getBalance() external view returns (euint64) {
        return balances[msg.sender];
        // Alice gets Alice's handle; Bob gets Bob's handle
        // Each can only decrypt their own
    }
}
```

---

## 4. Selective Access — Auditors and Observers

Grant a third party read access without giving them ownership:

```solidity
contract AuditableVault is ZamaEthereumConfig {
    mapping(address => euint64) private balances;
    address public auditor;

    function setAuditor(address _auditor) external onlyOwner {
        auditor = _auditor;
    }

    function deposit(externalEuint64 amount, bytes calldata proof) external {
        euint64 amt = FHE.fromExternal(amount, proof);
        euint64 newBal = FHE.add(balances[msg.sender], amt);

        FHE.allowThis(newBal);
        FHE.allow(newBal, msg.sender);  // User can read their own balance
        FHE.allow(newBal, auditor);     // Auditor can also read all balances

        balances[msg.sender] = newBal;
    }
}
```

---

## 5. Revoking Access

**Important:** You can stop granting new permissions, but you cannot invalidate handles already given.

```solidity
// You can stop granting allow on new handles going forward
function revokeAuditor() external onlyOwner {
    auditor = address(0);  // New deposits won't grant access to old auditor
    // ⚠️ But handles already granted to old auditor remain decryptable
}
```

**Design implication:** If you need revocable access, consider time-locked handles or rotating keys rather than relying on revocation.

---

## 6. Permission Checklist

Before shipping any FHEVM contract, verify every `euint` handle:

- [ ] Is `allowThis` called before storing?
- [ ] Is `allow(user)` called for every address that needs decrypt rights?
- [ ] For cross-contract flows, is `allowTransient` called before the external call?
- [ ] After every FHE operation (add, sub, select...), are permissions re-granted on the NEW handle?
- [ ] Is `makePubliclyDecryptable` only used for genuinely public reveals?

---

## Hands-On Exercise

Build a `SharedSecret` contract:
- Two users each submit an encrypted value
- The contract adds them together
- Both users can decrypt the result
- A third-party observer can also read the result if the owner grants access

---

📖 **Next:** [Lesson 2.3 — Handle Lifecycle Mastery](lesson-2-3-handle-lifecycle.md)
