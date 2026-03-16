# Conditional Selection

Because you cannot branch on an encrypted boolean, FHEVM uses `FHE.select` as the branch-free equivalent of `if/else`.

## Syntax

```solidity
euint64 result = FHE.select(condition, trueValue, falseValue);
// Equivalent to: result = condition ? trueValue : falseValue
```

Both branches are always computed. The encrypted condition determines which result is kept — without revealing which branch was taken.

## Examples

### Basic if/else replacement

```solidity
// ❌ WRONG
if (FHE.gt(balance, threshold)) {
    return balance;
} else {
    return FHE.asEuint64(0);
}

// ✅ CORRECT
ebool isAboveThreshold = FHE.gt(balance, threshold);
euint64 result = FHE.select(
    isAboveThreshold,
    balance,
    FHE.asEuint64(0)
);
```

### Blind auction — track highest bid

```solidity
function placeBid(externalEuint64 encryptedBid, bytes calldata proof) external {
    euint64 bid = FHE.fromExternal(encryptedBid, proof);

    ebool isHigher = FHE.gt(bid, highestBid);

    // Branch-free update: no information leaks about who is winning
    euint64 newHighest = FHE.select(isHigher, bid, highestBid);

    FHE.allowThis(bid);
    FHE.allowThis(newHighest);
    FHE.allow(bid, msg.sender);

    bids[msg.sender] = bid;
    highestBid = newHighest;
}
```

### Compliant transfer

```solidity
function transfer(address to, externalEuint64 amount, bytes calldata proof) external {
    euint64 transferAmount = FHE.fromExternal(amount, proof);

    // Only transfer if both sender and recipient are compliant
    ebool bothCompliant = FHE.and(isCompliant[msg.sender], isCompliant[to]);

    // If not compliant, transfer zero instead of reverting (privacy-preserving)
    euint64 actualAmount = FHE.select(bothCompliant, transferAmount, FHE.asEuint64(0));

    // Process actualAmount...
}
```

## Key Properties

- **Both branches are computed** — gas cost is constant regardless of condition
- **No information leakage** — which branch was taken is never revealed
- **Nested select is fine** — chain multiple `FHE.select` calls for complex logic
