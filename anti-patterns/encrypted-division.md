# Dividing by an Encrypted Value

**Severity:** Critical  unsupported operation

## The Problem

`FHE.div(encrypted, encrypted)` is not supported by FHEVM. The divisor must always be a plaintext value.

## Wrong

```solidity
// ❌ NOT SUPPORTED
euint64 result = FHE.div(numerator, encryptedDenominator);
```

## Correct

```solidity
// ✅ Divisor must be plaintext
euint64 result = FHE.div(numerator, uint64(1000));
```

## Implications for AMMs

The constant product formula requires: `amountOut = (reserveB × amountIn) / (reserveA + amountIn)`

When `amountIn` is encrypted, the denominator becomes encrypted  making FHE.div impossible.

**Workaround 1  Public reserves (simple):**
Keep reserves public so the denominator is always plaintext. Swap amounts remain private.

**Workaround 2  Masked fractions (production):**
Store a masked numerator/denominator. An off-chain oracle decrypts the masked denominator, then the swap settles with a proof of correct decryption.

See [Confidential AMM](../examples/defi/fhe-amm-simple.md) for a full implementation of Workaround 1.


---

📝 **Ready to test your knowledge?** → [Take the Anti-Patterns Quiz](../quizzes/anti-patterns-quiz.md)
