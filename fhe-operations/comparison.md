# Comparison Operations

All comparison operations return `ebool`  encrypted booleans.

```solidity
ebool eq  = FHE.eq(a, b);   // a == b
ebool ne  = FHE.ne(a, b);   // a != b
ebool lt  = FHE.lt(a, b);   // a <  b
ebool le  = FHE.le(a, b);   // a <= b
ebool gt  = FHE.gt(a, b);   // a >  b
ebool ge  = FHE.ge(a, b);   // a >= b
```

## Important: You Cannot Branch on ebool

An `ebool` cannot be used in a Solidity `if()` statement. Use `FHE.select()` instead.

```solidity
// ❌ WRONG  compile error or undefined behavior
ebool isAdult = FHE.ge(age, FHE.asEuint8(18));
if (isAdult) { ... }

// ✅ CORRECT  branch-free selection
ebool isAdult = FHE.ge(age, FHE.asEuint8(18));
euint64 access = FHE.select(isAdult, FHE.asEuint64(1), FHE.asEuint64(0));
```

See [Conditional Selection](conditional-selection.md) for the full `FHE.select` pattern.

## Mixed Comparisons

You can compare encrypted values against plaintext constants:

```solidity
ebool isZero    = FHE.eq(balance, uint64(0));
ebool isOverCap = FHE.gt(amount, uint64(MAX_AMOUNT));
```


---

📝 **Ready to test your knowledge?** → [Take the FHE Operations Quiz](../quizzes/fhe-operations-quiz.md)
