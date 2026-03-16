# Branching on Encrypted Booleans

**Severity:** Critical — compile error or incorrect logic

## The Problem

`ebool` is an encrypted boolean — it cannot be used in a Solidity `if()` statement. Attempting to do so will either fail to compile or produce undefined behavior.

## Wrong

```solidity
ebool isAdult = FHE.ge(age, FHE.asEuint8(18));
if (isAdult) { ... }  // ❌ Does not compile — ebool is not bool
```

## Correct

Use `FHE.select()` for all conditional logic on encrypted values:

```solidity
ebool isAdult = FHE.ge(age, FHE.asEuint8(18));

// ✅ Branch-free: compute both outcomes, select encrypted result
euint64 accessLevel = FHE.select(
    isAdult,
    FHE.asEuint64(1),  // adult access
    FHE.asEuint64(0)   // no access
);
```

## Why Branch-Free?

FHE.select evaluates both branches. This is intentional — it prevents timing or gas-based side channels that could leak which branch was taken.
