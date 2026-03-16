# Anti-Patterns & Pitfalls

This section documents the most common mistakes when writing FHEVM contracts, with clear wrong/correct code pairs for each.

## Quick Reference

| # | Pitfall | Symptom |
|---|---|---|
| 1 | [Missing `allowThis`](missing-allow-this.md) | Stored handles fail silently on reuse |
| 2 | [Missing `allow(user)`](missing-allow-user.md) | User holds a handle they cannot decrypt |
| 3 | [View on encrypted](view-on-encrypted.md) | Callers expect plaintext, get a handle |
| 4 | [Branching on `ebool`](branching-on-ebool.md) | Compile error or unexpected behavior |
| 5 | [Wrong proof binding](wrong-proof-binding.md) | `fromExternal` reverts unexpectedly |
| 6 | [Dividing by encrypted](encrypted-division.md) | Unsupported operation error |
| 7 | [Missing `allowTransient`](missing-allow-transient.md) | Cross-contract call reverts |
