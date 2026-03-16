# 🟢 Beginner Learning Path

Start here if you're new to FHEVM. These examples cover the fundamentals with no prior knowledge assumed.

## Recommended Order

| Step | Example | What you'll learn |
|---|---|---|
| 1 | [EncryptSingleValue](../examples/basic/encrypt-single-value.md) | `fromExternal`, `allowThis`, `allow` — the core trio |
| 2 | [EncryptMultipleValues](../examples/basic/encrypt-multiple-values.md) | One proof for multiple encrypted inputs |
| 3 | [FHEAdd](../examples/basic/fhe-add.md) | Your first FHE arithmetic operation |
| 4 | [FHESub](../examples/basic/fhe-sub.md) | Subtraction on encrypted values |
| 5 | [FHECounter](../examples/basic/fhe-counter.md) | Persisting and updating encrypted state |
| 6 | [FHEEq](../examples/basic/fhe-eq.md) | Comparing encrypted values |
| 7 | [FHEIfThenElse](../examples/basic/fhe-if-then-else.md) | `FHE.select` — the branch-free conditional |
| 8 | [UserDecryptSingleValue](../examples/basic/user-decrypt-single.md) | Off-chain decryption flow |
| 9 | [UserDecryptMultipleValues](../examples/basic/user-decrypt-multiple.md) | Decrypting multiple results |
| 10 | [EncryptedAgeVerification](../examples/identity/encrypted-age-verification.md) | Real use case: threshold check |
| 11 | [ERC7984Example](../examples/openzeppelin/erc7984-example.md) | Minimal confidential token |

## After Completing Beginner

You'll understand how to:
- Encrypt inputs client-side and submit with proofs
- Store encrypted values on-chain with correct permissions
- Perform basic FHE arithmetic and comparisons
- Decrypt results off-chain

**Next step:** [Intermediate Path](intermediate.md)
