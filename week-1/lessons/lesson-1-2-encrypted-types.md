# Lesson 1.2 — Encrypted Types & the Handle Model

**Duration:** 60 minutes | **Format:** Reading + Code walkthrough

---

## Learning Goals

- Know all FHEVM encrypted types and when to use each
- Understand that encrypted variables store handles, not values
- Understand handle lifecycle: creation → permission → storage → reuse

---

## 1. Encrypted Type Reference

| Solidity Type | Encrypted Equivalent | Use For |
|---|---|---|
| `bool` | `ebool` | Flags, conditions, compliance status |
| `uint8` | `euint8` | Small counters, age, ratings |
| `uint16` | `euint16` | Medium values, scores |
| `uint32` | `euint32` | Larger counters |
| `uint64` | `euint64` | Token amounts, balances ← most common |
| `uint128` | `euint128` | Large financial values |
| `uint256` | `euint256` | Maximum range |
| `address` | `eaddress` | Hidden recipients |

**Import:**
```solidity
import {FHE, euint64, euint8, ebool, externalEuint64} from "@fhevm/solidity/lib/FHE.sol";
import {ZamaEthereumConfig} from "@fhevm/solidity/config/ZamaConfig.sol";
```

---

## 2. The Handle Model — The Most Important Concept

> ⚠️ **This is the single most important thing to understand in FHEVM.**

When you write `euint64 balance`, the variable does **not** store a number. It stores a **256-bit handle** — an opaque pointer to a ciphertext stored by the FHE coprocessor.

```solidity
euint64 balance = FHE.asEuint64(100);
// balance = 0x3f7a...  (a handle, not 100)
```

**Consequences:**
- You cannot `require(balance > 0)` — balance is a handle, not a number
- You cannot read the value in a view function and return plaintext
- Every FHE operation creates a **new handle** — the old one is unchanged
- Handles have no permissions by default — you must grant them explicitly

---

## 3. Creating Handles

```solidity
// From a plaintext constant
euint64 zero = FHE.asEuint64(0);
euint8  flag = FHE.asEuint8(1);
ebool   yes  = FHE.asEbool(true);

// From a user-submitted encrypted input (requires proof)
euint64 userValue = FHE.fromExternal(encryptedInput, inputProof);

// From an FHE operation (derived handle)
euint64 result = FHE.add(a, b);   // new handle — a and b unchanged
```

---

## 4. Handle Permissions

Every handle starts with **no permissions**. You must explicitly grant:

```solidity
FHE.allowThis(handle);           // Contract can reuse this handle
FHE.allow(handle, userAddress);  // User can decrypt off-chain
FHE.allowTransient(handle, contractAddress); // One-tx cross-contract
```

**The golden rule:**
```
Create handle → allowThis (if storing) → allow(user) (if user needs to decrypt) → store/return
```

---

## 5. The Lifecycle in Practice

```solidity
contract PrivateBalance is ZamaEthereumConfig {
    euint64 private balance;

    function deposit(externalEuint64 amount, bytes calldata proof) external {
        // Step 1: Validate and convert input
        euint64 amt = FHE.fromExternal(amount, proof);

        // Step 2: Compute new balance (new handle!)
        euint64 newBalance = FHE.add(balance, amt);

        // Step 3: Grant permissions on the NEW handle
        FHE.allowThis(newBalance);           // Contract can use it later
        FHE.allow(newBalance, msg.sender);   // Owner can decrypt

        // Step 4: Store
        balance = newBalance;
    }

    function getBalance() external view returns (euint64) {
        return balance;  // Returns handle — decrypt off-chain
    }
}
```

---

## 6. External Types

When receiving encrypted inputs from users, use `external` variants:

```solidity
// In function parameters — user-submitted encrypted values
function store(externalEuint64 value, bytes calldata proof) external {
    euint64 val = FHE.fromExternal(value, proof);
    // val is now a verified internal handle
}
```

| External type | Internal type |
|---|---|
| `externalEuint64` | `euint64` |
| `externalEuint8` | `euint8` |
| `externalEbool` | `ebool` |

---

## Hands-On Exercise

Open the [Week 1 Starter Repo](../../resources/starter-repos.md) and complete `exercises/1-2-handles.ts`:

1. Create an encrypted value from plaintext
2. Add two encrypted values together
3. Verify the result handle is different from both inputs
4. Grant permissions and decrypt the result

---

📖 **Next:** [Lesson 1.3 — Your First Encrypted Contract](lesson-1-3-first-contract.md)
