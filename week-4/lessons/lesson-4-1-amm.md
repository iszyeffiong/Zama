# Lesson 4.1 — Confidential AMM Design

**Duration:** 60 minutes | **Format:** Architecture deep dive + full contract

---

## Learning Goals

- Understand the constant product formula and why FHE makes it challenging
- Implement a working AMM with encrypted swap amounts and public reserves
- Know the FHE division limitation and two production workarounds
- Analyse the privacy tradeoffs in AMM design

---

## 1. What is an AMM?

An Automated Market Maker is a decentralised exchange that uses a mathematical formula to price assets. Instead of order books, AMMs use liquidity pools.

The most common formula — constant product:

```
x * y = k
```

Where `x` and `y` are token reserves and `k` is a constant. When you swap token A for token B:

```
amountOut = (reserveB * amountIn) / (reserveA + amountIn)
```

---

## 2. The FHE Division Problem

In a confidential AMM, users want `amountIn` to be encrypted (hide trade size). But look at the denominator:

```
denominator = reserveA + amountIn
```

If `amountIn` is `euint64`, the denominator becomes `euint64` — encrypted. But:

```solidity
FHE.div(numerator, encryptedDenominator);  // NOT SUPPORTED
```

**FHEVM only supports `FHE.div(encrypted, plaintext)`.**

This is not a bug — it is a fundamental constraint of the current FHE scheme. Encrypted division requires a different mathematical approach entirely.

---

## 3. Solution 1 — Public Reserves (This Lesson)

Keep `reserveA` and `reserveB` as public `uint64`. Only swap amounts are encrypted.

Privacy achieved:
- Trade size hidden (amountIn, amountOut encrypted)
- Exact exchange rate hidden (computed from hidden amounts)

Privacy NOT achieved:
- Pool ratio visible (reserveA/reserveB is public)
- Large trades still affect reserves visibly

This is sufficient for many use cases and is the approach we implement here.

---

## 4. Full Confidential AMM Contract

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.24;

import {FHE, euint64, externalEuint64} from "@fhevm/solidity/lib/FHE.sol";
import {ZamaEthereumConfig} from "@fhevm/solidity/config/ZamaConfig.sol";

contract ConfidentialAMM is ZamaEthereumConfig {
    // Public reserves — required for FHE.div plaintext denominator
    uint64 public reserveA;
    uint64 public reserveB;
    address public owner;
    bool    public initialized;

    event LiquidityAdded(uint64 amountA, uint64 amountB);
    event SwapExecuted(address indexed user, bool aForB);

    error NotOwner();
    error AlreadyInitialized();
    error NotInitialized();
    error ZeroLiquidity();
    error ZeroDenominator();

    constructor() {
        owner = msg.sender;
    }

    // ----------------------------------------------------------------
    // Liquidity Management
    // ----------------------------------------------------------------

    function addLiquidity(uint64 amountA, uint64 amountB) external {
        if (msg.sender != owner)  revert NotOwner();
        if (initialized)          revert AlreadyInitialized();
        if (amountA == 0 || amountB == 0) revert ZeroLiquidity();

        reserveA    = amountA;
        reserveB    = amountB;
        initialized = true;

        emit LiquidityAdded(amountA, amountB);
    }

    // ----------------------------------------------------------------
    // Swap: Token A → Token B
    // ----------------------------------------------------------------

    /// @notice Swap an encrypted amount of A for B
    /// @dev Formula: amountOut ≈ (reserveB * amountIn) / reserveA
    ///      Simplified: ignores +amountIn in denominator for educational clarity
    function swapAForB(
        externalEuint64 amountInExt,
        bytes calldata  inputProof
    ) external returns (euint64 amountOut) {
        if (!initialized) revert NotInitialized();
        if (reserveA == 0) revert ZeroDenominator();

        // Step 1: Validate encrypted input
        euint64 amountIn = FHE.fromExternal(amountInExt, inputProof);

        // Step 2: Compute output — encrypted * plaintext = encrypted
        // amountOut = (reserveB * amountIn) / reserveA
        euint64 numerator = FHE.mul(amountIn, reserveB);
        amountOut         = FHE.div(numerator, reserveA);

        // Step 3: Grant permissions
        FHE.allowThis(amountOut);
        FHE.allow(amountOut, msg.sender);

        emit SwapExecuted(msg.sender, true);
    }

    /// @notice Swap an encrypted amount of B for A
    function swapBForA(
        externalEuint64 amountInExt,
        bytes calldata  inputProof
    ) external returns (euint64 amountOut) {
        if (!initialized) revert NotInitialized();
        if (reserveB == 0) revert ZeroDenominator();

        euint64 amountIn  = FHE.fromExternal(amountInExt, inputProof);
        euint64 numerator = FHE.mul(amountIn, reserveA);
        amountOut         = FHE.div(numerator, reserveB);

        FHE.allowThis(amountOut);
        FHE.allow(amountOut, msg.sender);

        emit SwapExecuted(msg.sender, false);
    }

    // ----------------------------------------------------------------
    // Full formula swap (plaintext amounts — for testing)
    // ----------------------------------------------------------------

    /// @notice Swap with plaintext amount — demonstrates full constant product formula
    function swapExact(uint64 amountIn, bool aForB) external returns (euint64 amountOut) {
        if (!initialized) revert NotInitialized();

        uint64 reserveIn  = aForB ? reserveA : reserveB;
        uint64 reserveOut = aForB ? reserveB : reserveA;

        // Full constant product: out = (reserveOut * in) / (reserveIn + in)
        uint64 numerator   = reserveOut * amountIn;
        uint64 denominator = reserveIn + amountIn;
        uint64 out         = numerator / denominator;

        // Update reserves
        if (aForB) { reserveA += amountIn; reserveB -= out; }
        else       { reserveB += amountIn; reserveA -= out; }

        amountOut = FHE.asEuint64(out);
        FHE.allowThis(amountOut);
        FHE.allow(amountOut, msg.sender);

        emit SwapExecuted(msg.sender, aForB);
    }

    // ----------------------------------------------------------------
    // View helpers
    // ----------------------------------------------------------------

    function getReserves() external view returns (uint64, uint64) {
        return (reserveA, reserveB);
    }

    function getExpectedOutput(uint64 amountIn, bool aForB) external view returns (uint64) {
        if (!initialized) return 0;
        uint64 rIn  = aForB ? reserveA : reserveB;
        uint64 rOut = aForB ? reserveB : reserveA;
        return (rOut * amountIn) / (rIn + amountIn);
    }
}
```

---

## 5. Integer Division Pitfall

```
Pool: 1,000,000 A : 500,000 B
Swap: 1 A
Formula: (500,000 * 1) / (1,000,000 + 1) = 0.499... → truncates to 0
```

Very small swaps yield zero output. Always enforce a minimum swap amount:

```solidity
// Add to swapAForB:
ebool isNonZero = FHE.gt(amountIn, FHE.asEuint64(0));
// Or enforce client-side: reject inputs below a minimum
```

---

## 6. Solution 2 — Masked Fractions (Production Pattern)

For fully confidential reserves in production:

1. Store reserves as encrypted values
2. When a swap arrives, compute a masked version of the denominator
3. An off-chain oracle decrypts only the masked denominator
4. Oracle posts the decrypted denominator + a ZK proof of correct decryption
5. Contract settles the swap using the proven plaintext denominator

This is the approach used by EPOOL and advanced confidential DeFi protocols. It adds oracle dependency and latency but achieves full reserve privacy.

---

## Hands-On Exercise (15 min)

Extend ConfidentialAMM with:
1. A `removeLiquidity(uint64 amountA, uint64 amountB)` function — owner only
2. A price impact estimate: `getExpectedOutputWithImpact(uint64 amountIn, bool aForB)` that returns the full constant product result including the amountIn in the denominator
3. A minimum output check: revert if expected output is zero

---

📖 **Next:** [Lesson 4.2 — Private Order Books & Payroll](lesson-4-2-advanced-defi.md)
