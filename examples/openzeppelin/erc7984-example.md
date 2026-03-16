# ERC7984Example — Minimal Confidential Token

**Difficulty:** 🟢 Beginner | **Concept:** Confidential mint + transfer

## What It Demonstrates

The simplest possible confidential token: encrypted balances, encrypted minting, encrypted transfers. No compliance, no wrappers — just the core ERC7984 lifecycle.

## Contract

```solidity
import {ERC7984} from "@openzeppelin/confidential-contracts/token/ERC7984/ERC7984.sol";

contract ConfidentialToken is ERC7984, ZamaEthereumConfig {
    address public owner;

    constructor() ERC7984("ConfToken", "CTK") {
        owner = msg.sender;
    }

    function mint(
        address to,
        externalEuint64 encryptedAmount,
        bytes calldata proof
    ) external {
        require(msg.sender == owner, "Not owner");
        euint64 amount = FHE.fromExternal(encryptedAmount, proof);
        _mint(to, amount);
    }
}
```

## Key Points

- `_mint` handles `allowThis` and `allow` internally via ERC7984 base
- `balanceOf` returns a `euint64` handle — decrypt off-chain
- Transfers encrypt the amount in the transaction

```bash
npx create-fhevm-example erc7984-example
```
