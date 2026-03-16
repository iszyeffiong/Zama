# FHEVM Dev Hub

## Getting Started
* [Introduction](README.md)
* [What is FHEVM?](basics/what-is-fhevm.md)
* [FHE 101 — Plain Language Primer](basics/fhe-101.md)
* [Quick Start — Scaffold in Seconds](basics/quick-start.md)

## Core Concepts
* [Encrypted Types](core-concepts/encrypted-types.md)
* [Handles & Lifecycle](core-concepts/handles-and-lifecycle.md)
* [Input Proofs](core-concepts/input-proofs.md)
* [Access Control & Permissions](core-concepts/access-control.md)
* [Decryption Patterns](core-concepts/decryption-patterns.md)

## FHE Operations Reference
* [Arithmetic](fhe-operations/arithmetic.md)
* [Comparison](fhe-operations/comparison.md)
* [Boolean & Bitwise](fhe-operations/boolean-bitwise.md)
* [Conditional Selection](fhe-operations/conditional-selection.md)
* [Type Conversion](fhe-operations/type-conversion.md)

## Anti-Patterns & Pitfalls
* [Overview](anti-patterns/README.md)
* [Missing allowThis](anti-patterns/missing-allow-this.md)
* [Missing allow(user)](anti-patterns/missing-allow-user.md)
* [View Functions on Encrypted Values](anti-patterns/view-on-encrypted.md)
* [Branching on Encrypted Booleans](anti-patterns/branching-on-ebool.md)
* [Wrong Input Proof Binding](anti-patterns/wrong-proof-binding.md)
* [Dividing by Encrypted Value](anti-patterns/encrypted-division.md)
* [Missing allowTransient](anti-patterns/missing-allow-transient.md)

## Learning Paths
* [Beginner](learning-paths/beginner.md)
* [Intermediate](learning-paths/intermediate.md)
* [Advanced](learning-paths/advanced.md)

## Examples

### Basic
* [EncryptSingleValue](examples/basic/encrypt-single-value.md)
* [EncryptMultipleValues](examples/basic/encrypt-multiple-values.md)
* [FHEAdd](examples/basic/fhe-add.md)
* [FHESub](examples/basic/fhe-sub.md)
* [FHECounter](examples/basic/fhe-counter.md)
* [FHEEq](examples/basic/fhe-eq.md)
* [FHEIfThenElse](examples/basic/fhe-if-then-else.md)
* [UserDecryptSingleValue](examples/basic/user-decrypt-single.md)
* [UserDecryptMultipleValues](examples/basic/user-decrypt-multiple.md)
* [PublicDecryptSingleValue](examples/basic/public-decrypt-single.md)
* [PublicDecryptMultipleValues](examples/basic/public-decrypt-multiple.md)
* [HandleGeneration](examples/basic/handle-generation.md)
* [HandleLifecycle](examples/basic/handle-lifecycle.md)
* [InputProofsExplained](examples/basic/input-proofs-explained.md)

### Games
* [FHEWordle](examples/games/fhe-wordle.md)
* [Rock Paper Scissors](examples/games/rock-paper-scissors.md)
* [Encrypted Lottery](examples/games/encrypted-lottery.md)
* [Encrypted Poker](examples/games/encrypted-poker.md)
* [Encrypted Blackjack](examples/games/encrypted-blackjack.md)

### Auctions
* [BlindAuction](examples/auctions/blind-auction.md)
* [DutchAuction](examples/auctions/dutch-auction.md)

### Advanced
* [Hidden Voting](examples/advanced/hidden-voting.md)
* [Private Payroll](examples/advanced/private-payroll.md)
* [Encrypted Escrow](examples/advanced/encrypted-escrow.md)
* [Private KYC](examples/advanced/private-kyc.md)
* [Private Order Book](examples/advanced/private-order-book.md)

### DeFi
* [Confidential AMM](examples/defi/fhe-amm-simple.md)

### Identity & Compliance
* [EncryptedAgeVerification](examples/identity/encrypted-age-verification.md)
* [IdentityRegistry](examples/identity/identity-registry.md)
* [AccessControlGrants](examples/identity/access-control-grants.md)
* [ComplianceRules](examples/identity/compliance-rules.md)
* [CompliantERC20](examples/identity/compliant-erc20.md)
* [TransientAccessControl](examples/identity/transient-access-control.md)

### OpenZeppelin Confidential
* [ERC7984 Standard](examples/openzeppelin/erc7984.md)
* [ERC7984Example](examples/openzeppelin/erc7984-example.md)
* [ERC7984KycRestricted](examples/openzeppelin/erc7984-kyc-restricted.md)
* [ERC7984ObserverAccessExample](examples/openzeppelin/erc7984-observer-access.md)
* [ERC7984ERC20WrapperExample](examples/openzeppelin/erc7984-erc20-wrapper.md)
* [SwapERC7984ToERC7984](examples/openzeppelin/swap-confidential-to-confidential.md)
* [SwapERC7984ToERC20](examples/openzeppelin/swap-confidential-to-public.md)
* [VestingWalletConfidentialExample](examples/openzeppelin/vesting-wallet.md)
* [WETHc — Confidential ETH Wrapper](examples/openzeppelin/wethc.md)
* [ERC7984Rwa](examples/openzeppelin/erc7984-rwa.md)
* [ERC7821 Executor](examples/openzeppelin/erc7821-executor.md)

## Reference
* [Full Example Map](resources/example-map.md)
* [Resources & Links](resources/links.md)

## Quizzes
* [What is FHEVM?](quizzes/what-is-fhevm-quiz.md)
* [FHE 101](quizzes/fhe-101-quiz.md)
* [Quick Start](quizzes/quick-start-quiz.md)
* [Core Concepts](quizzes/core-concepts-quiz.md)
* [FHE Operations](quizzes/fhe-operations-quiz.md)
* [Anti-Patterns & Pitfalls](quizzes/anti-patterns-quiz.md)
* [Basic Examples](quizzes/examples-basic-quiz.md)
* [Advanced Examples](quizzes/examples-advanced-quiz.md)
