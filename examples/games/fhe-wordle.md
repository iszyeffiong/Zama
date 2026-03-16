# FHEWordle

**Difficulty:** 🟡 Intermediate | **Concept:** Encrypted letter comparison with branch-free feedback

## What It Demonstrates

A privacy-preserving Wordle clone. The target word is stored encrypted on-chain. On each guess, the contract compares each letter using `FHE.eq` and produces encrypted feedback without ever revealing the answer.

## Key FHE Operations

- `FHE.eq`  compare each guess letter to the encrypted target letter
- `FHE.select`  branch-free feedback assignment (correct / present / absent)
- `FHE.and`, `FHE.or`  combine multiple conditions for "letter present" check

## Core Logic

```solidity
// For each of 5 positions
for (uint i = 0; i < 5; i++) {
    ebool isCorrect = FHE.eq(
        FHE.asEuint8(guess[i]),
        secretLetters[i]
    );

    // 2 = correct position, 1 = present elsewhere, 0 = absent
    feedback[i] = FHE.select(
        isCorrect,
        FHE.asEuint8(2),
        checkPresent(guess[i], secretLetters)  // further FHE ops
    );

    FHE.allowThis(feedback[i]);
    FHE.allow(feedback[i], msg.sender);
}
```

## Privacy Properties

- The target word is never revealed, even to the contract executor
- Feedback is encrypted  only the guessing player can see their results
- No information is leaked about positions that aren't guessed correctly

```bash
npx create-fhevm-example fhe-wordle
```
