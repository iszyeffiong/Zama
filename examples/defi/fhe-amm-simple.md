# Confidential AMM (FHEAMMSimple)

**Difficulty:** 🟡 Intermediate | **Concept:** Constant product formula with encrypted swap amounts

## What It Demonstrates

An educational Automated Market Maker (AMM) with encrypted swap inputs and outputs. Demonstrates the FHE division constraint and how to work around it.

## Design Tradeoff

The constant product formula is: `amountOut = (reserveB × amountIn) / (reserveA + amountIn)`

When `amountIn` is encrypted, the denominator `(reserveA + amountIn)` becomes encrypted  but `FHE.div` requires a **plaintext divisor**.

**This example uses public reserves** so the denominator stays plaintext:

| What's public | What's private |
|---|---|
| Pool reserves (reserveA, reserveB) | Swap input amounts |
| Pool ratio / price | Swap output amounts |
| Trade direction | Exact trade size |

## Contract

```solidity
contract FHEAMMSimple is ZamaEthereumConfig {
    uint64 public reserveA;   // Public  required for FHE.div
    uint64 public reserveB;
    bool   public initialized;

    function addLiquidity(uint64 amountA, uint64 amountB) external onlyOwner {
        reserveA = amountA;
        reserveB = amountB;
        initialized = true;
    }

    function swapAForB(
        externalEuint64 amountInExt,
        bytes calldata inputProof
    ) external returns (euint64 amountOut) {
        euint64 amountIn = FHE.fromExternal(amountInExt, inputProof);

        // FHE.mul: encrypted × plaintext = encrypted ✅
        euint64 numerator = FHE.mul(amountIn, reserveB);

        // FHE.div: encrypted / plaintext = encrypted ✅
        // FHE.div(encrypted, encrypted) is NOT SUPPORTED
        amountOut = FHE.div(numerator, reserveA);

        FHE.allowThis(amountOut);
        FHE.allow(amountOut, msg.sender);

        emit SwapExecuted(msg.sender, true);
    }
}
```

## Pitfalls Demonstrated

| Pitfall | Description |
|---|---|
| `FHE.div` limitation | Cannot divide by encrypted denominator |
| Integer truncation | Tiny swaps may yield zero output |
| Missing `allowThis` | Breaks subsequent use of `amountOut` |
| Uninitialized pool | Must call `addLiquidity` first |

## Production Workaround  Masked Fractions

For fully confidential reserves in production AMMs:
1. Store a masked numerator/denominator pair
2. Off-chain oracle decrypts only the masked denominator
3. Swap settles with a proof of correct decryption

This approach (used by EPOOL) keeps reserves private at the cost of added complexity.

## Scaffold

```bash
npx create-fhevm-example fhe-amm-simple
```


---

📝 **Ready to test your knowledge?** → [Take the Advanced Examples Quiz](../../quizzes/examples-advanced-quiz.md)
