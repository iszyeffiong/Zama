# Starter Repositories

## Week 1 — Foundations

```bash
# Scaffold the Week 1 starter project
npx create-fhevm-example encrypt-single-value
```

Contains:
- `contracts/EncryptSingleValue.sol` — reference contract
- `test/EncryptSingleValue.ts` — reference tests
- `exercises/` — empty files for lesson exercises

## Week 2 — Concepts

```bash
npx create-fhevm-example --category concepts
```

Contains all access control, handle lifecycle, input proof, and anti-pattern examples.

## Week 3 — Advanced

```bash
npx create-fhevm-example --category gaming
npx create-fhevm-example blind-auction
```

## Week 4 — DeFi

```bash
npx create-fhevm-example --category advanced
npx create-fhevm-example fhe-amm-simple
```

## Homework Starter Templates

Each homework file includes a starter contract skeleton. Copy it into `contracts/` in a new Hardhat project:

```bash
npx create-fhevm-example encrypt-single-value --output my-week1-hw
cd my-week1-hw
# Replace contracts/EncryptSingleValue.sol with the PrivateVault starter from the homework page
```

## Solution Repositories

Solution repos are provided to instructors only. Contact the bootcamp organizers for access. Do not share solutions publicly before homework deadlines.
