# Missing allow(user)

**Severity:** High — user receives a handle they cannot decrypt

## The Problem

A contract grants `FHE.allowThis()` but forgets `FHE.allow(user)`. The user gets a handle back but decryption fails because the coprocessor has not authorized their address.

## Wrong

```solidity
function compute(externalEuint64 input, bytes calldata proof) external {
    euint64 result = FHE.add(FHE.fromExternal(input, proof), FHE.asEuint64(1));
    FHE.allowThis(result);
    // ❌ Missing: FHE.allow(result, msg.sender)
    results[msg.sender] = result;
}
```

## Correct

```solidity
function compute(externalEuint64 input, bytes calldata proof) external {
    euint64 result = FHE.add(FHE.fromExternal(input, proof), FHE.asEuint64(1));
    FHE.allowThis(result);
    FHE.allow(result, msg.sender);  // ✅ User can now decrypt
    results[msg.sender] = result;
}
```

## Note on Revocation

Granting and then revoking access does **not** invalidate a handle the user already received. If `FHE.allow(result, user)` was called in a previous transaction, the user can still decrypt that specific handle even after revocation.
