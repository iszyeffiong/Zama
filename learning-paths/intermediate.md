# 🟡 Intermediate Learning Path

Requires completing the [Beginner Path](beginner.md) or equivalent familiarity with basic FHE concepts.

## Recommended Order

| Step | Example | What you'll learn |
|---|---|---|
| 1 | [HandleGeneration](../examples/basic/handle-generation.md) | Handles as symbolic references |
| 2 | [HandleLifecycle](../examples/basic/handle-lifecycle.md) | Persisting and reusing across calls |
| 3 | [InputProofsExplained](../examples/basic/input-proofs-explained.md) | Deep dive into proof binding |
| 4 | [PublicDecryptSingleValue](../examples/basic/public-decrypt-single.md) | When to use public decryption |
| 5 | [AntiPatternMissingAllowThis](../examples/basic/handle-generation.md) | Why `allowThis` is non-negotiable |
| 6 | [AntiPatternMissingUserAllow](../examples/basic/encrypt-single-value.md) | Why `allow(user)` matters |
| 7 | [AntiPatternViewOnEncrypted](../examples/basic/user-decrypt-single.md) | View functions return handles |
| 8 | [AccessControlGrants](../examples/identity/access-control-grants.md) | Controlling who can decrypt |
| 9 | [ComplianceRules](../examples/identity/compliance-rules.md) | Combining checks with `FHE.and()` |
| 10 | [DutchAuction](../examples/auctions/dutch-auction.md) | Real use case: descending price auction |
| 11 | [FHEWordle](../examples/games/fhe-wordle.md) | Branch-free letter comparison |
| 12 | [IdentityRegistry](../examples/identity/identity-registry.md) | Multi-attribute encrypted identity |
| 13 | [TransientAccessControl](../examples/identity/transient-access-control.md) | Cross-contract permissions |
| 14 | [ERC7984KycRestricted](../examples/openzeppelin/erc7984-kyc-restricted.md) | KYC-gated confidential token |

## After Completing Intermediate

You'll understand how to:
- Design correct handle lifecycles across multiple transactions
- Work with access control and permission delegation
- Write compliance-enforcing contracts without revealing private attributes
- Build cross-contract FHE flows with `allowTransient`

**Next step:** [Advanced Path](advanced.md)
