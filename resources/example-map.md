# Full Example Map

All 40+ examples across every category, with difficulty levels and key FHE concepts.

## Basic

| Example | Difficulty | Key Concept |
|---|---|---|
| EncryptSingleValue | 🟢 Beginner | `fromExternal`, `allowThis`, `allow` |
| EncryptMultipleValues | 🟢 Beginner | Single proof for multiple inputs |
| FHEAdd | 🟢 Beginner | `FHE.add` |
| FHESub | 🟢 Beginner | `FHE.sub` |
| FHECounter | 🟢 Beginner | Encrypted counter, persisted state |
| FHEEq | 🟢 Beginner | `FHE.eq` |
| FHEIfThenElse | 🟢 Beginner | `FHE.select` |
| UserDecryptSingleValue | 🟢 Beginner | Off-chain user decryption |
| UserDecryptMultipleValues | 🟢 Beginner | Batch user decryption |
| PublicDecryptSingleValue | 🟡 Intermediate | `makePubliclyDecryptable` |
| PublicDecryptMultipleValues | 🟡 Intermediate | Batch public decryption |
| HandleGeneration | 🟡 Intermediate | Derived handles, symbolic execution |
| HandleLifecycle | 🟡 Intermediate | Persisting handles across calls |
| InputProofsExplained | 🟡 Intermediate | Proof binding to contract + sender |
| AntiPatternMissingAllowThis | 🟡 Intermediate | Pitfall: missing `allowThis` |
| AntiPatternMissingUserAllow | 🟡 Intermediate | Pitfall: missing `allow(user)` |
| AntiPatternViewOnEncrypted | 🟡 Intermediate | Pitfall: view returns handle |

## Games

| Example | Difficulty | Key Concept |
|---|---|---|
| FHEWordle | 🟡 Intermediate | Branch-free letter comparison |
| Rock Paper Scissors | 🟢 Beginner | Commit-reveal with FHE |
| Encrypted Lottery | 🟡 Intermediate | Encrypted ticket selection |
| Encrypted Poker | 🔴 Advanced | Encrypted hand management |
| Encrypted Blackjack | 🔴 Advanced | Encrypted deck + score |

## Auctions

| Example | Difficulty | Key Concept |
|---|---|---|
| DutchAuction | 🟡 Intermediate | Encrypted reserve, descending price |
| BlindAuction | 🔴 Advanced | Sealed bids, branch-free tracking, public reveal |

## Advanced

| Example | Difficulty | Key Concept |
|---|---|---|
| HiddenVoting | 🔴 Advanced | Encrypted ballot tallying |
| PrivatePayroll | 🔴 Advanced | Confidential salary processing |
| EncryptedEscrow | 🔴 Advanced | Conditional encrypted release |
| PrivateKYC | 🔴 Advanced | KYC without identity exposure |
| PrivateOrderBook | 🔴 Advanced | Encrypted limit order matching |

## DeFi

| Example | Difficulty | Key Concept |
|---|---|---|
| FHEAMMSimple | 🟡 Intermediate | Constant product AMM, FHE division constraint |

## Identity & Compliance

| Example | Difficulty | Key Concept |
|---|---|---|
| EncryptedAgeVerification | 🟢 Beginner | `FHE.ge` threshold check |
| IdentityRegistry | 🟡 Intermediate | Multi-attribute encrypted identity |
| AccessControlGrants | 🟡 Intermediate | `FHE.allow` lifecycle |
| ComplianceRules | 🟡 Intermediate | `FHE.and` for policy checks |
| CompliantERC20 | 🔴 Advanced | Branch-free compliant transfers |
| TransientAccessControl | 🟡 Intermediate | `FHE.allowTransient` cross-contract |

## OpenZeppelin Confidential

| Example | Difficulty | Key Concept |
|---|---|---|
| ERC7984Example | 🟢 Beginner | Minimal confidential token |
| ERC7984KycRestricted | 🟡 Intermediate | KYC-gated token transfers |
| ERC7984ObserverAccessExample | 🟡 Intermediate | Audit observer opt-in |
| ERC7984ERC20WrapperExample | 🔴 Advanced | Public ↔ confidential wrapping |
| SwapERC7984ToERC7984 | 🟡 Intermediate | Confidential-to-confidential swap |
| SwapERC7984ToERC20 | 🔴 Advanced | Confidential-to-public swap |
| VestingWalletConfidentialExample | 🔴 Advanced | Confidential vesting + KYC + factory |
| WETHc | 🟡 Intermediate | Wrap native ETH confidentially |
| ERC7984Rwa | 🔴 Advanced | Confidential real-world asset token |
| ERC7821 Executor | 🔴 Advanced | Encrypted batch execution |
