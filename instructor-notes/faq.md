# Common Student Questions — FAQ

## Week 1
**"Why does euint64 store a hex number?"** — It is a handle (pointer to a ciphertext), not the value. Review Lesson 1.2.

**"Why call allowThis every time after FHE.add?"** — Every FHE op creates a new handle with no permissions. Permissions never inherit.

**"Can I use euint64 in require()?"** — No. Get an ebool via FHE.gt/eq, then use FHE.select for logic.

## Week 2
**"My proof is being rejected on fromExternal."** — Check you used the exact deployed contract address and caller address when generating the proof.

**"I revoked access but user can still decrypt old results."** — By design. Revoking is forward-looking only.

**"When to use allowTransient vs allow?"** — allowTransient for same-tx cross-contract passing. allow for user off-chain decryption.

## Week 3
**"Cross-contract call reverts with authorization error."** — Missing FHE.allowTransient(handle, contractB) before the external call.

**"After makePubliclyDecryptable, can I hide the value again?"** — No — it is permanent and irreversible.

## Week 4
**"Why can't I do FHE.div(principal, encryptedRate)?"** — FHE.div only supports plaintext divisors. Store per-user rates as public uint64.

**"How much does a FHE operation cost in gas?"** — Significantly more than plaintext ops. See Zama docs for benchmarks. Minimize FHE ops in hot paths.

## General
**"Where can I get help after the bootcamp?"** — Zama Discord: https://discord.gg/zama
