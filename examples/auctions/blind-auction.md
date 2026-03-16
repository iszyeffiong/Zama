# BlindAuction

**Difficulty:** 🔴 Advanced | **Concept:** Sealed-bid auction with encrypted bids and public reveal

## What It Demonstrates

A complete sealed-bid auction where all bids remain private until the auction ends. The highest bid is tracked on-chain using a branch-free comparison — without revealing individual bid amounts.

## Key FHE Operations

- `FHE.fromExternal` — validate and store encrypted bids
- `FHE.gt` — compare incoming bid to current highest
- `FHE.select` — branch-free update of highest bid
- `FHE.makePubliclyDecryptable` — reveal winner after deadline

## Contract

```solidity
contract BlindAuction is ZamaEthereumConfig {
    mapping(address => euint64) public bids;
    euint64 public highestBid;
    uint256 public deadline;

    function placeBid(externalEuint64 encryptedBid, bytes calldata proof) external {
        require(block.timestamp < deadline, "Auction ended");

        euint64 bid = FHE.fromExternal(encryptedBid, proof);

        // Branch-free: no information leaks about relative bid values
        ebool isHigher = FHE.gt(bid, highestBid);
        euint64 newHighest = FHE.select(isHigher, bid, highestBid);

        FHE.allowThis(bid);
        FHE.allow(bid, msg.sender);
        FHE.allowThis(newHighest);

        bids[msg.sender] = bid;
        highestBid = newHighest;
    }

    function reveal() external {
        require(block.timestamp >= deadline, "Auction not ended");
        FHE.makePubliclyDecryptable(highestBid);
    }
}
```

## Test Cases

- ✓ Places bid and updates highest bid correctly
- ✓ Lower bid does not displace highest bid
- ✓ Public decrypt works after reveal
- ✓ Reverts if bid placed after deadline
- ✓ Public decrypt reverts before reveal

```bash
npx create-fhevm-example blind-auction
```
