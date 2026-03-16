# Week 2 Homework — ConfidentialVoting

**Estimated time:** 3–4 hours | **Due:** Before Week 3 starts

---

## Overview

Build a `ConfidentialVoting` contract where voters cast encrypted ballots (yes/no), tallies accumulate privately, and results are revealed only after the voting period ends.

---

## Specification

```solidity
interface IConfidentialVoting {
    function castVote(externalEbool encryptedVote, bytes calldata proof) external;
    function endVoting() external;           // only owner, after deadline
    function getResults() external view returns (euint64 yesCount, euint64 noCount);
    function hasVoted(address voter) external view returns (bool);
}
```

### Requirements

1. Each address can vote only once — track with `mapping(address => bool) public hasVoted`
2. Votes are `ebool`: encrypted true = yes, false = no
3. Running tallies `votesYes` and `votesNo` are `euint64`, updated on each vote
4. After `endVoting()`, both tallies become publicly decryptable
5. Before `endVoting()`, only the contract can access tallies (`allowThis` only)
6. Correct input proof validation on each vote

---

## Grading Criteria

| Criteria | Points |
|---|---|
| Contract compiles | 10 |
| Correct vote accumulation using `FHE.add` | 20 |
| Branch-free yes/no split using `FHE.select` | 20 |
| Duplicate vote prevention | 10 |
| `endVoting` correctly calls `makePubliclyDecryptable` | 15 |
| Input proof validation working | 10 |
| All required tests pass (min 8 test cases) | 10 |
| Code quality and comments | 5 |
| **Total** | **100** |

### Bonus (up to 20 points)
- Add a time-lock: voting cannot end before a deadline block (10 pts)
- Add a `delegateVote(address to)` that transfers voting rights to another address (10 pts)

---

## Required Tests

```typescript
it("allows a valid encrypted vote")
it("prevents double voting")
it("accumulates yes votes correctly")
it("accumulates no votes correctly")
it("tallies are not publicly decryptable before endVoting")
it("tallies become publicly decryptable after endVoting")
it("rejects vote with wrong contract in proof")
it("rejects vote with wrong sender in proof")
```

---

## 📝 Before You Submit

Complete the Week 2 Quiz on Tally first:

👉 [Open Week 2 Quiz](https://forms.gle/8Weaz9AUBsFyPNgE9)

Paste your quiz score in your homework submission README.


