# Week 3 Homework — ConfidentialAuction

**Estimated time:** 4–5 hours | **Due:** Before Week 4 starts

---

## Overview

Build a production-quality sealed-bid auction where bids stay encrypted throughout the auction, the highest bid is tracked branch-free, and the winner is revealed after the deadline.

---

## Specification

```solidity
interface IConfidentialAuction {
    function placeBid(externalEuint64 encryptedBid, bytes calldata proof) external;
    function revealWinner() external;         // only after deadline
    function claimRefund() external;          // non-winners reclaim bids
    function getMyBid() external view returns (euint64);
    function getHighestBid() external view returns (euint64);
}
```

### Requirements

1. Bids are encrypted with `euint64`
2. Highest bid is tracked with `FHE.gt` + `FHE.select` — no branching
3. Each bidder can update their bid (only their latest bid counts)
4. After deadline, `revealWinner` calls `makePubliclyDecryptable(highestBid)`
5. Non-winners can call `claimRefund` (track encrypted bid amounts per address)
6. Bidder can always decrypt their own bid with `FHE.allow`

---

## Grading Criteria

| Criteria | Points |
|---|---|
| Compiles and deploys | 10 |
| Branch-free highest bid tracking (`FHE.gt` + `FHE.select`) | 25 |
| Each bidder can view their own bid | 10 |
| `revealWinner` correctly uses `makePubliclyDecryptable` | 15 |
| Deadline enforcement | 10 |
| Bid update logic (latest bid replaces previous) | 10 |
| All required tests pass (min 10) | 15 |
| Code quality | 5 |
| **Total** | **100** |

### Bonus (up to 25 points)
- Implement Dutch auction variant with descending price (15 pts)
- Add a minimum reserve price stored as `euint64` (10 pts)

---

## Required Tests

```typescript
it("accepts a valid encrypted bid")
it("tracks highest bid correctly after multiple bids")
it("lower bid does not replace highest bid")
it("bidder can read their own bid")
it("bidder cannot read another's bid")
it("revealWinner fails before deadline")
it("revealWinner succeeds after deadline")
it("highest bid is publicly decryptable after reveal")
it("bidder can update their bid")
it("refund claimable by non-winner after reveal")
```
