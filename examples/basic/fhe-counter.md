# FHECounter

**Difficulty:** 🟢 Beginner | **Concept:** Encrypted counter using FHE.add and FHE.sub

## What It Demonstrates

How to persist and update encrypted state across multiple transactions. Each operation creates a new handle — you must re-grant `allowThis` on every new handle.

## Contract

```solidity
contract FHECounter is ZamaEthereumConfig {
    euint64 private counter;

    constructor() {
        counter = FHE.asEuint64(0);
        FHE.allowThis(counter);
    }

    function increment() external {
        euint64 one = FHE.asEuint64(1);
        euint64 newCounter = FHE.add(counter, one);
        FHE.allowThis(newCounter);   // Re-grant on the NEW handle
        counter = newCounter;
    }

    function decrement() external {
        euint64 one = FHE.asEuint64(1);
        euint64 newCounter = FHE.sub(counter, one);
        FHE.allowThis(newCounter);
        counter = newCounter;
    }

    function grantRead(address user) external {
        FHE.allow(counter, user);
    }

    function getCounter() external view returns (euint64) {
        return counter;
    }
}
```

## Key Points

- `FHE.add` returns a **new** handle — the old one is unchanged
- You must call `FHE.allowThis` on the new handle before storing
- Permissions do not transfer from old handle to new one automatically


---

📝 **Ready to test your knowledge?** → [Take the Basic Examples Quiz](../../quizzes/examples-basic-quiz.md)
