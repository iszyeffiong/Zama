# Handles & Lifecycle

A **handle** is an opaque 256-bit reference to an encrypted value stored by the FHE coprocessor. Understanding the handle lifecycle is the most important skill for writing correct FHEVM contracts.

## Creating Handles

```solidity
// From a user-submitted encrypted input (requires proof)
euint64 value = FHE.fromExternal(encryptedInput, inputProof);

// From a plaintext constant
euint64 zero = FHE.asEuint64(0);

// From an FHE operation — creates a derived handle
euint64 result = FHE.add(a, b);
```

Every FHE operation creates a **new, derived handle**. The original handles remain valid.

## Storing Handles

```solidity
contract Example is ZamaEthereumConfig {
    euint64 public storedBalance;

    function store(externalEuint64 input, bytes calldata proof) external {
        euint64 val = FHE.fromExternal(input, proof);

        // CRITICAL: grant allowThis before storing
        FHE.allowThis(val);          // Contract can reuse this handle later
        FHE.allow(val, msg.sender);  // User can decrypt

        storedBalance = val;
    }
}
```

> ⚠️ Without `FHE.allowThis(val)`, the contract cannot use the stored handle in future transactions. This is the most common FHEVM mistake.

## Reusing Stored Handles

```solidity
function increment() external {
    euint64 one = FHE.asEuint64(1);
    euint64 newBalance = FHE.add(storedBalance, one);  // New handle created

    // Must re-grant on the NEW handle
    FHE.allowThis(newBalance);
    FHE.allow(newBalance, msg.sender);

    storedBalance = newBalance;
}
```

Permissions on the old handle do **not** carry over to the new one.

## Lifecycle Summary

```
1. Create handle (fromExternal / asEuint / FHE operation)
2. Grant FHE.allowThis()  ← required to store or reuse
3. Grant FHE.allow(user)  ← required for user to decrypt
4. Store in state or return
5. On next use: operate → new handle → re-grant → repeat
```

## Transient Handles

Handles created with `FHE.allowTransient` are available within a single transaction:

```solidity
FHE.allowTransient(handle, otherContract);
// otherContract can now use handle in this tx only
```

Cached (stored) handles outlive transient permissions. A handle stored in a mapping remains decryptable even after the `allowTransient` that created it expires.
