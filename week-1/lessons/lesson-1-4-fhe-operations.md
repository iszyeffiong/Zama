# Lesson 1.4 — FHE Operations Reference

**Duration:** 60 minutes | **Format:** Reference + exercises

---

## Learning Goals

- Know every available FHE operation and its return type
- Understand the division limitation and why it exists
- Use `FHE.select` correctly as the branch-free conditional

---

## Arithmetic

```solidity
euint64 sum  = FHE.add(a, b);        // encrypted + encrypted
euint64 sum2 = FHE.add(a, uint64(5)); // encrypted + plaintext
euint64 diff = FHE.sub(a, b);
euint64 prod = FHE.mul(a, b);
euint64 quot = FHE.div(a, uint64(k)); // ⚠️ k must be PLAINTEXT
euint64 rem  = FHE.rem(a, uint64(k)); // ⚠️ k must be PLAINTEXT
```

> **Critical:** `FHE.div(encrypted, encrypted)` is **not supported**. The divisor must always be plaintext. This impacts AMM designs — we cover workarounds in Week 4.

## Comparison — returns ebool

```solidity
ebool eq = FHE.eq(a, b);   // a == b
ebool ne = FHE.ne(a, b);   // a != b
ebool lt = FHE.lt(a, b);   // a <  b
ebool le = FHE.le(a, b);   // a <= b
ebool gt = FHE.gt(a, b);   // a >  b
ebool ge = FHE.ge(a, b);   // a >= b
```

## Boolean & Bitwise

```solidity
ebool  andB = FHE.and(boolA, boolB);  // logical AND on ebool
ebool  orB  = FHE.or(boolA, boolB);
ebool  notB = FHE.not(boolA);

euint64 andI = FHE.and(a, b);  // bitwise AND on euintX
euint64 shl  = FHE.shl(a, uint64(k));
euint64 shr  = FHE.shr(a, uint64(k));
```

## Conditional Selection — the if/else replacement

```solidity
// result = condition ? trueVal : falseVal
euint64 result = FHE.select(condition, trueVal, falseVal);
```

Both branches are always computed. The encrypted condition picks the result.

```solidity
// ❌ WRONG — ebool cannot be used in if()
if (FHE.gt(balance, threshold)) { ... }

// ✅ CORRECT
ebool isAbove = FHE.gt(balance, threshold);
euint64 result = FHE.select(isAbove, balance, FHE.asEuint64(0));
```

## Type Conversion

```solidity
euint64 big   = FHE.asEuint64(smallHandle);  // upcast — safe
euint8  small = FHE.asEuint8(bigHandle);     // downcast — truncates!
```

---

## Exercise: Implement selectMax

Write a function that returns the larger of two encrypted values:

```solidity
function selectMax(
    externalEuint64 aExt, bytes calldata proofA,
    externalEuint64 bExt, bytes calldata proofB
) external returns (euint64 max) {
    // Your code here
}
```

Solution in [starter repo](../../resources/starter-repos.md).

---

## Full Operations Quick Reference

```solidity
// Arithmetic
FHE.add(a, b)          // euint + euint or euint + plaintext
FHE.sub(a, b)          // euint - euint or euint - plaintext
FHE.mul(a, b)          // euint * euint or euint * plaintext
FHE.div(a, k)          // euint / plaintext ONLY
FHE.rem(a, k)          // euint % plaintext ONLY
FHE.neg(a)             // negate: 0 - a

// Comparison → ebool
FHE.eq(a, b)   FHE.ne(a, b)
FHE.lt(a, b)   FHE.le(a, b)
FHE.gt(a, b)   FHE.ge(a, b)

// Boolean logic on ebool
FHE.and(a, b)   FHE.or(a, b)
FHE.not(a)      FHE.xor(a, b)

// Bitwise on euintX
FHE.and(a, b)   FHE.or(a, b)   FHE.xor(a, b)   FHE.not(a)
FHE.shl(a, k)   FHE.shr(a, k)  // shift by plaintext k
FHE.rotl(a, k)  FHE.rotr(a, k) // rotate by plaintext k

// Conditional
FHE.select(condition, trueVal, falseVal)  // ebool condition

// Type conversion
FHE.asEuint8(v)   FHE.asEuint16(v)
FHE.asEuint32(v)  FHE.asEuint64(v)
FHE.asEbool(v)

// From user input
FHE.fromExternal(externalHandle, inputProof)
```

---

## Common Mistakes with Operations

### Mistake 1 — Using ebool in require()
```solidity
// ❌ ebool is NOT a Solidity bool
require(FHE.gt(balance, FHE.asEuint64(0)), "Zero balance");

// ✅ Use FHE.select for conditional logic
euint64 result = FHE.select(FHE.gt(balance, FHE.asEuint64(0)), balance, FHE.asEuint64(0));
```

### Mistake 2 — Forgetting permissions on operation results
```solidity
// ❌ result has no permissions
euint64 result = FHE.add(a, b);
stored = result;

// ✅ Always grant before storing
euint64 result = FHE.add(a, b);
FHE.allowThis(result);
FHE.allow(result, msg.sender);
stored = result;
```

### Mistake 3 — Dividing by encrypted value
```solidity
// ❌ Not supported
FHE.div(amount, encryptedRate);

// ✅ Divisor must be plaintext
FHE.div(amount, uint64(100));
```

---

📖 **Next:** [Lesson 1.5 — Decryption Patterns](lesson-1-5-decryption.md)
