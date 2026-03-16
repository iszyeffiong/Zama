# DutchAuction

**Difficulty:** 🟡 Intermediate | **Concept:** Descending price auction with encrypted reserve

## What It Demonstrates

A Dutch auction where the price decreases over time from a starting price. The reserve price (minimum acceptable price) is stored **encrypted**, so bidders cannot know whether their offer will clear.

## Design

```
startPrice (public) → decreases every block → currentPrice (public)
reservePrice (encrypted) → FHE.ge(currentPrice, reserve) → sale clears?
```

The key insight: the sale condition is evaluated with FHE without revealing the reserve.

## Key FHE Operations

- `FHE.ge(plaintextCurrentPrice, encryptedReserve)` — check if price is at or above reserve
- `FHE.select` — branch-free: either complete the sale or reject

```bash
npx create-fhevm-example dutch-auction
```
