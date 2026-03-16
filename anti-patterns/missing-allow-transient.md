# Missing allowTransient in Multi-Contract Flows

**Severity:** High — cross-contract calls revert

## The Problem

When Contract A calls Contract B and passes an encrypted handle, Contract B needs explicit permission to use that handle within the transaction. Without `FHE.allowTransient()`, the call reverts.

## Wrong

```solidity
// Contract A — token
function swap(address swapContract) external {
    // ❌ No permission granted — swapContract cannot use the handle
    ISwap(swapContract).execute(balances[msg.sender]);
}
```

## Correct

```solidity
// Contract A — token
function swap(address swapContract) external {
    // ✅ Grant one-transaction permission before calling
    FHE.allowTransient(balances[msg.sender], swapContract);
    ISwap(swapContract).execute(balances[msg.sender]);
}

// Contract B — swap
function execute(euint64 amount) external {
    // amount is usable here because of allowTransient
    euint64 result = FHE.mul(amount, FHE.asEuint64(2));
    FHE.allowThis(result);  // Re-grant for contract B's own future use
    ...
}
```

## ERC7984 Swap Pitfalls

When swapping ERC7984 tokens:
1. Operator approval must be set before the swap contract can spend on your behalf
2. Each token contract needs its own `allowTransient` call
3. Missing either causes a revert

```solidity
// ❌ Missing allowTransient for toToken
FHE.allowTransient(handle, address(fromToken));
// FHE.allowTransient(handle, address(toToken));  // Missing!
swapContract.swap(fromToken, toToken, handle);  // Reverts
```


---

📝 **Ready to test your knowledge?** → [Take the Anti-Patterns Quiz](../quizzes/anti-patterns-quiz.md)
