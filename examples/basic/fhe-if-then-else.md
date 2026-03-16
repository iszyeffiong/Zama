# FHEIfThenElse

**Difficulty:** 🟢 Beginner | **Concept:** Branch-free conditional using FHE.select

## What It Demonstrates

`FHE.select` is the FHEVM equivalent of a ternary operator. Both branches are always evaluated — the encrypted condition picks which result is returned without revealing which path was taken.

## Contract

```solidity
contract FHEIfThenElse is ZamaEthereumConfig {
    function selectMax(
        externalEuint64 aExt,
        externalEuint64 bExt,
        bytes calldata proofA,
        bytes calldata proofB
    ) external returns (euint64 max) {
        euint64 a = FHE.fromExternal(aExt, proofA);
        euint64 b = FHE.fromExternal(bExt, proofB);

        // If a >= b, return a; otherwise return b
        ebool aIsGreater = FHE.ge(a, b);
        max = FHE.select(aIsGreater, a, b);

        FHE.allowThis(max);
        FHE.allow(max, msg.sender);
    }
}
```

## Key Points

- `ebool` cannot be used in a Solidity `if()` — always use `FHE.select`
- Both `a` and `b` are computed; only the selected result is returned
- Grant permissions on the result handle, not on the input handles


---

📝 **Ready to test your knowledge?** → [Take the Basic Examples Quiz](../../quizzes/examples-basic-quiz.md)
