# Arithmetic Operations

```solidity
euint64 sum  = FHE.add(a, b);   // a + b
euint64 diff = FHE.sub(a, b);   // a - b
euint64 prod = FHE.mul(a, b);   // a * b
euint64 quot = FHE.div(a, k);   // a / k  ← k must be PLAINTEXT
euint64 rem  = FHE.rem(a, k);   // a % k  ← k must be PLAINTEXT
```

Mixed encrypted/plaintext operations are supported for add, sub, and mul:

```solidity
euint64 result = FHE.add(encryptedValue, uint64(10));  // encrypted + plaintext
euint64 result = FHE.mul(encryptedValue, uint64(2));   // encrypted * plaintext
```

## Critical: Division Limitation

> ⚠️ `FHE.div(encrypted, encrypted)` is **not supported**. The divisor must always be a plaintext value.

```solidity
// ❌ NOT SUPPORTED
euint64 result = FHE.div(numerator, encryptedDenominator);

// ✅ SUPPORTED
euint64 result = FHE.div(numerator, uint64(1000));
```

This limitation affects algorithms that need encrypted denominators (e.g., AMM constant product formula). See the [Confidential AMM](../examples/defi/fhe-amm-simple.md) for workaround patterns.

## Integer Division Truncation

FHE division truncates, just like Solidity integer division:

```
FHE.div(FHE.asEuint64(7), 2) → 3 (not 3.5)
```

In AMMs or other financial contracts, very small input amounts may produce zero output. Always enforce minimum amounts in production.


---

📝 **Ready to test your knowledge?** → [Take the FHE Operations Quiz](../quizzes/fhe-operations-quiz.md)
