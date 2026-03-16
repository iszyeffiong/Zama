# Lesson 3.2 — Sealed Bid Auctions

**Duration:** 60 minutes | **Format:** Deep dive + full contract walkthrough

---

## Learning Goals

- Design a sealed-bid auction where bids stay private throughout
- Implement branch-free highest-bid tracking with FHE.gt + FHE.select
- Understand the reveal phase and when to use makePubliclyDecryptable
- Handle edge cases: bid updates, refunds, and deadline enforcement

---

## 1. The Sealed-Bid Problem

In a public auction, everyone can see every bid. The last bidder always wins by bidding one unit more — making the auction trivially gameable. In a sealed-bid auction, all bids are submitted privately and revealed simultaneously.

On-chain, this is hard without FHE:
- Commit-reveal: two transactions, front-running risk at reveal
- ZKP: expensive proofs, limited to specific predicates
- FHE: bids stay encrypted on-chain until the reveal phase

---

## 2. Blind Auction — Full Contract

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.24;

import {FHE, euint64, ebool, externalEuint64} from "@fhevm/solidity/lib/FHE.sol";
import {ZamaEthereumConfig} from "@fhevm/solidity/config/ZamaConfig.sol";

contract BlindAuction is ZamaEthereumConfig {
    address public  owner;
    uint256 public  deadline;
    bool    public  revealed;

    euint64 public  highestBid;
    address public  highestBidder;

    mapping(address => euint64) public bids;

    event BidPlaced(address indexed bidder);
    event AuctionRevealed();

    error AuctionEnded();
    error AuctionNotEnded();
    error AlreadyRevealed();

    constructor(uint256 _deadline) {
        owner    = msg.sender;
        deadline = _deadline;

        // Initialize highestBid to zero
        highestBid = FHE.asEuint64(0);
        FHE.allowThis(highestBid);
    }

    /// @notice Place or update an encrypted bid
    function placeBid(externalEuint64 encryptedBid, bytes calldata proof) external {
        if (block.timestamp >= deadline) revert AuctionEnded();

        euint64 bid = FHE.fromExternal(encryptedBid, proof);

        // Branch-free highest bid update
        ebool  isHigher    = FHE.gt(bid, highestBid);
        euint64 newHighest = FHE.select(isHigher, bid, highestBid);

        // Permissions on the bid itself
        FHE.allowThis(bid);
        FHE.allow(bid, msg.sender);      // bidder can decrypt their own bid
        FHE.allow(bid, owner);           // owner can audit after reveal

        // Permissions on new highest
        FHE.allowThis(newHighest);

        bids[msg.sender] = bid;
        highestBid       = newHighest;

        // Note: we track highestBidder using a pattern below
        emit BidPlaced(msg.sender);
    }

    /// @notice Reveal the winning bid publicly after deadline
    function reveal() external {
        if (block.timestamp < deadline) revert AuctionNotEnded();
        if (revealed) revert AlreadyRevealed();

        FHE.makePubliclyDecryptable(highestBid);
        revealed = true;

        emit AuctionRevealed();
    }

    /// @notice Bidder retrieves their own bid handle to decrypt off-chain
    function getMyBid() external view returns (euint64) {
        return bids[msg.sender];
    }
}
```

---

## 3. The Highest-Bidder Tracking Challenge

The contract above tracks the highest bid amount but not the highest bidder. Tracking the bidder is harder because it requires conditional address assignment — which FHE does not support natively on `address` types.

**Pattern 1 — Public flag (small privacy tradeoff):**
```solidity
mapping(address => ebool) public isCurrentHighest;

function placeBid(externalEuint64 encryptedBid, bytes calldata proof) external {
    euint64 bid     = FHE.fromExternal(encryptedBid, proof);
    ebool isHigher  = FHE.gt(bid, highestBid);

    // Store encrypted flag per bidder — bidder can decrypt to know if they're winning
    ebool flag = isHigher;
    FHE.allowThis(flag);
    FHE.allow(flag, msg.sender);
    isCurrentHighest[msg.sender] = flag;

    euint64 newHighest = FHE.select(isHigher, bid, highestBid);
    FHE.allowThis(newHighest);
    highestBid = newHighest;
    FHE.allowThis(bid); FHE.allow(bid, msg.sender);
    bids[msg.sender] = bid;
}
```

**Pattern 2 — Reveal via public decryption + off-chain matching:**
After `reveal()`, anyone can decrypt `highestBid`. Each bidder decrypts their own bid. The one whose plaintext matches wins. This is the standard approach for production auctions.

---

## 4. Dutch Auction — Descending Price with Encrypted Reserve

In a Dutch auction, the price starts high and drops every block. A buyer accepts when the price reaches their valuation. The reserve (minimum acceptable price) is kept encrypted:

```solidity
contract DutchAuction is ZamaEthereumConfig {
    uint256 public startPrice;
    uint256 public startTime;
    uint256 public priceDropPerBlock;
    euint64 private reservePrice;  // encrypted minimum
    bool    public sold;

    function currentPrice() public view returns (uint256) {
        uint256 elapsed  = block.number - startTime;
        uint256 drop     = elapsed * priceDropPerBlock;
        return startPrice > drop ? startPrice - drop : 0;
    }

    function buy() external payable {
        require(!sold, "Already sold");

        uint64 price = uint64(currentPrice());

        // Branch-free: only complete sale if currentPrice >= reservePrice
        ebool meetsReserve = FHE.ge(FHE.asEuint64(price), reservePrice);

        // We cannot conditionally transfer ETH in branch-free FHE,
        // so we record the sale and use a settlement pattern:
        euint8 saleStatus = FHE.select(meetsReserve, FHE.asEuint8(1), FHE.asEuint8(0));
        FHE.allowThis(saleStatus);
        FHE.allow(saleStatus, owner);
        FHE.allow(saleStatus, msg.sender);

        pendingSale[msg.sender] = saleStatus;
    }
}
```

---

## 5. Test Patterns for Auctions

```typescript
describe("BlindAuction", () => {
    it("accepts encrypted bids and updates highest", async () => {
        // Alice bids 100
        const alice100 = await fhevm.createEncryptedInput(addr, alice.address).add64(100n).encrypt();
        await auction.connect(alice).placeBid(alice100.handles[0], alice100.inputProof);

        // Bob bids 200 — should become new highest
        const bob200 = await fhevm.createEncryptedInput(addr, bob.address).add64(200n).encrypt();
        await auction.connect(bob).placeBid(bob200.handles[0], bob200.inputProof);

        // After reveal, highestBid should decrypt to 200
        await time.increase(3601);
        await auction.reveal();

        const handle = await auction.highestBid();
        // Public decrypt — anyone can read after reveal
        const winning = await fhevm.publicDecryptEuint(FhevmType.euint64, handle, addr);
        expect(winning).to.equal(200n);
    });

    it("lower bid does not replace highest", async () => {
        const high = await fhevm.createEncryptedInput(addr, alice.address).add64(500n).encrypt();
        await auction.connect(alice).placeBid(high.handles[0], high.inputProof);

        const low = await fhevm.createEncryptedInput(addr, bob.address).add64(50n).encrypt();
        await auction.connect(bob).placeBid(low.handles[0], low.inputProof);

        await time.increase(3601);
        await auction.reveal();

        const handle  = await auction.highestBid();
        const winning = await fhevm.publicDecryptEuint(FhevmType.euint64, handle, addr);
        expect(winning).to.equal(500n);
    });

    it("reveal fails before deadline", async () => {
        await expect(auction.reveal()).to.be.revertedWithCustomError(auction, "AuctionNotEnded");
    });
});
```

---

## Hands-On Exercise (15 min)

Add a minimum bid increment to the BlindAuction:
- New bids must be at least 10 units higher than the current highest
- Use FHE.ge(bid, FHE.add(highestBid, FHE.asEuint64(10))) for the condition
- Bids that don't meet the increment are stored but don't replace the highest

---

📖 **Next:** [Lesson 3.3 — Identity & Compliance Systems](lesson-3-3-identity.md)
