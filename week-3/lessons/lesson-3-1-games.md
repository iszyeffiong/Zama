# Lesson 3.1 — Encrypted Games

**Duration:** 60 minutes | **Format:** Concept + guided code walkthrough

---

## Learning Goals

- Understand why on-chain games need FHE privacy
- Build an encrypted game using FHE.eq for hidden comparison
- Use FHE.select for branch-free outcome resolution
- Design encrypted game state that reveals results only to the right players

---

## 1. Why Games Need On-Chain Privacy

Traditional on-chain games have a front-running problem. In Rock Paper Scissors:

```
Alice submits: "scissors" (public transaction)
Bob sees Alice's move BEFORE submitting his
Bob submits: "rock" — guaranteed win
```

The classic fix is commit-reveal — hash your move, commit the hash, reveal later. This requires two transactions, a waiting period, and the value is eventually public anyway.

With FHE, moves stay encrypted throughout. Both players submit encrypted moves. The contract compares them directly on ciphertext. Neither player ever sees the other's move.

---

## 2. Rock Paper Scissors — Full Contract

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.24;

import {FHE, euint8, ebool, externalEuint8} from "@fhevm/solidity/lib/FHE.sol";
import {ZamaEthereumConfig} from "@fhevm/solidity/config/ZamaConfig.sol";

// Moves: 1=Rock, 2=Paper, 3=Scissors
// Result: 1=alice wins, 2=bob wins, 3=draw
contract EncryptedRPS is ZamaEthereumConfig {
    euint8  private moveAlice;
    euint8  private moveBob;
    address public  alice;
    address public  bob;
    bool    public  aliceCommitted;
    bool    public  bobCommitted;
    euint8  private result;
    bool    public  resolved;

    constructor(address _alice, address _bob) {
        alice = _alice;
        bob   = _bob;
    }

    function submitMove(externalEuint8 encryptedMove, bytes calldata proof) external {
        require(msg.sender == alice || msg.sender == bob, "Not a player");
        require(!resolved, "Game over");

        euint8 move = FHE.fromExternal(encryptedMove, proof);
        FHE.allowThis(move);

        if (msg.sender == alice) {
            require(!aliceCommitted, "Already submitted");
            FHE.allow(move, alice);
            moveAlice = move;
            aliceCommitted = true;
        } else {
            require(!bobCommitted, "Already submitted");
            FHE.allow(move, bob);
            moveBob = move;
            bobCommitted = true;
        }

        if (aliceCommitted && bobCommitted) {
            _resolve();
        }
    }

    function _resolve() internal {
        ebool isDraw        = FHE.eq(moveAlice, moveBob);
        ebool alicePaper    = FHE.eq(moveAlice, FHE.asEuint8(2));
        ebool aliceScissors = FHE.eq(moveAlice, FHE.asEuint8(3));
        ebool aliceRock     = FHE.eq(moveAlice, FHE.asEuint8(1));
        ebool bobRock       = FHE.eq(moveBob,   FHE.asEuint8(1));
        ebool bobPaper      = FHE.eq(moveBob,   FHE.asEuint8(2));
        ebool bobScissors   = FHE.eq(moveBob,   FHE.asEuint8(3));

        // Alice wins: Paper beats Rock, Scissors beats Paper, Rock beats Scissors
        ebool aliceWins = FHE.or(
            FHE.or(FHE.and(alicePaper, bobRock), FHE.and(aliceScissors, bobPaper)),
            FHE.and(aliceRock, bobScissors)
        );

        // Branch-free result selection
        euint8 drawOrBob = FHE.select(isDraw, FHE.asEuint8(3), FHE.asEuint8(2));
        result           = FHE.select(aliceWins, FHE.asEuint8(1), drawOrBob);

        FHE.allowThis(result);
        FHE.allow(result, alice);
        FHE.allow(result, bob);
        resolved = true;
    }

    function getResult() external view returns (euint8) {
        require(resolved, "Game not resolved");
        return result;
    }
}
```

---

## 3. Key Design Decisions

### Why keep the result encrypted?

Even after resolution, both players see the same encrypted result handle. Each decrypts it off-chain with their own key. A public observer watching on-chain sees nothing — not the moves, not the outcome.

### Why FHE.or chains instead of a lookup table?

FHE does not support array indexing on encrypted values. Every conditional must be built from FHE.eq, FHE.and, FHE.or combinations. Verbose, but explicit and correct.

### What about invalid moves (e.g., value=5)?

Validate client-side before encrypting. Input proofs bind the ciphertext to the user — if they encrypt an invalid value, the game resolves incorrectly for them. You can also add a plaintext range check on the proof metadata.

---

## 4. FHEWordle Pattern

A 5-letter word stored encrypted on-chain. Each guess gets per-letter encrypted feedback computed branch-free:

```solidity
function guess(uint8[5] calldata guessLetters) external returns (euint8[5] memory feedback) {
    require(guessCount[msg.sender] < 6, "No guesses left");

    for (uint i = 0; i < 5; i++) {
        euint8 g = FHE.asEuint8(guessLetters[i]);

        // Exact match at this position
        ebool exact = FHE.eq(g, secretLetters[i]);

        // Present anywhere in the word
        ebool present = FHE.or(
            FHE.or(FHE.eq(g, secretLetters[0]), FHE.eq(g, secretLetters[1])),
            FHE.or(FHE.or(FHE.eq(g, secretLetters[2]), FHE.eq(g, secretLetters[3])),
                   FHE.eq(g, secretLetters[4]))
        );

        // 0=absent, 1=present elsewhere, 2=exact position
        euint8 presentOrAbsent = FHE.select(present, FHE.asEuint8(1), FHE.asEuint8(0));
        feedback[i]            = FHE.select(exact,   FHE.asEuint8(2), presentOrAbsent);

        FHE.allowThis(feedback[i]);
        FHE.allow(feedback[i], msg.sender);
    }

    guessCount[msg.sender]++;
}
```

---

## 5. Encrypted Lottery Pattern

```solidity
function claim() external {
    require(drawn, "Not drawn yet");

    // Branch-free win check — no revert, no leak
    ebool  isWinner = FHE.eq(tickets[msg.sender], winningNumber);
    euint64 prize   = FHE.select(isWinner, jackpot, FHE.asEuint64(0));

    FHE.allowThis(prize);
    FHE.allow(prize, msg.sender);
    claimable[msg.sender] = prize;
}
```

The user decrypts prize off-chain. If zero, they did not win. No information leaks to observers.

---

## Hands-On Exercise (15 min)

Extend the RPS contract to support best-of-3:
- Track wins for each player as euint8
- After each round, increment winner's count branch-free with FHE.select
- Game ends when either player reaches 2 wins

---

📖 **Next:** [Lesson 3.2 — Sealed Bid Auctions](lesson-3-2-auctions.md)
