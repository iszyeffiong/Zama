# 🔴 Advanced Learning Path

Production-ready patterns. Requires solid understanding of intermediate concepts.

## Recommended Order

| Step | Example | What you'll learn |
|---|---|---|
| 1 | [BlindAuction](../examples/auctions/blind-auction.md) | Sealed bids + branch-free highest-bid tracking |
| 2 | [CompliantERC20](../examples/identity/compliant-erc20.md) | `FHE.select()` for compliant transfers |
| 3 | [HiddenVoting](../examples/advanced/hidden-voting.md) | Encrypted ballot tallying |
| 4 | [PrivateKYC](../examples/advanced/private-kyc.md) | KYC without identity exposure |
| 5 | [EncryptedEscrow](../examples/advanced/encrypted-escrow.md) | Conditional encrypted release |
| 6 | [PrivatePayroll](../examples/advanced/private-payroll.md) | Confidential salary processing |
| 7 | [PrivateOrderBook](../examples/advanced/private-order-book.md) | Encrypted limit order matching |
| 8 | [FHEAMMSimple](../examples/defi/fhe-amm-simple.md) | Confidential AMM + FHE division constraints |
| 9 | [ERC7984ERC20WrapperExample](../examples/openzeppelin/erc7984-erc20-wrapper.md) | Cross-standard token wrapping |
| 10 | [SwapERC7984ToERC20](../examples/openzeppelin/swap-confidential-to-public.md) | Confidential-to-public swap |
| 11 | [SwapERC7984ToERC7984](../examples/openzeppelin/swap-confidential-to-confidential.md) | Full confidential swap |
| 12 | [VestingWalletConfidentialExample](../examples/openzeppelin/vesting-wallet.md) | Confidential vesting + KYC + factory |

## After Completing Advanced

You'll be able to:
- Architect full privacy-preserving DeFi protocols
- Design compliant token systems with ERC7984
- Handle complex multi-contract FHE flows
- Make informed tradeoffs around FHE constraints (division limitation, gas costs)
