# Missing allowThis

**Severity:** Critical — breaks contract functionality silently

## The Problem

When a contract stores an encrypted handle but does not call `FHE.allowThis()`, the FHE coprocessor does not recognize the contract as authorized to use that handle in future transactions. Operations on the stored handle silently fail or produce wrong results.

## Wrong

```solidity
function store(externalEuint64 input, bytes calldata proof) external {
    euint64 val = FHE.fromExternal(input, proof);
    // ❌ Missing: FHE.allowThis(val)
    storedValue = val;
}

function increment() external {
    // ❌ This will fail — contract has no permission on storedValue
    euint64 newVal = FHE.add(storedValue, FHE.asEuint64(1));
}
```

## Correct

```solidity
function store(externalEuint64 input, bytes calldata proof) external {
    euint64 val = FHE.fromExternal(input, proof);
    FHE.allowThis(val);   // ✅ Contract retains rights to this handle
    FHE.allow(val, msg.sender);
    storedValue = val;
}

function increment() external {
    euint64 newVal = FHE.add(storedValue, FHE.asEuint64(1));
    FHE.allowThis(newVal);  // ✅ Also re-grant on the new handle
    storedValue = newVal;
}
```

## Rule

Every handle you intend to keep across transactions must have `FHE.allowThis()` called on it before the transaction ends.
