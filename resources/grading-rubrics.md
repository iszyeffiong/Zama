# Grading Rubrics

## Overview

All homework is graded out of 100 points. Bonus points are available on every assignment.

| Assignment | Due | Points | Bonus |
|---|---|---|---|
| Week 1 — PrivateVault | End of Week 1 | 100 | +15 |
| Week 2 — ConfidentialVoting | End of Week 2 | 100 | +20 |
| Week 3 — ConfidentialAuction | End of Week 3 | 100 | +25 |
| Week 4 — Capstone | End of Week 4 | 100 | +30 |

---

## Week 1 — PrivateVault Rubric

| Criteria | Points |
|---|---|
| Contract compiles without errors | 10 |
| `deposit` correctly adds to balance with proper permissions | 15 |
| `withdraw` uses `FHE.select` for branch-free logic | 20 |
| `withdraw` does not reduce balance below zero | 10 |
| `getBalance` returns correct handle with proper permissions | 10 |
| `isAboveThreshold` correctly uses `FHE.ge` | 10 |
| All 10 required test cases pass | 20 |
| Code quality: comments, naming, no unused variables | 5 |
| **Total** | **100** |

**Bonus:** emergencyReset (+5), allowAuditor (+10)

---

## Week 2 — ConfidentialVoting Rubric

| Criteria | Points |
|---|---|
| Contract compiles | 10 |
| Correct vote accumulation using `FHE.add` | 20 |
| Branch-free yes/no split using `FHE.select` | 20 |
| Duplicate vote prevention | 10 |
| `endVoting` correctly calls `makePubliclyDecryptable` | 15 |
| Input proof validation working | 10 |
| All 8 required test cases pass | 10 |
| Code quality and comments | 5 |
| **Total** | **100** |

**Bonus:** time-lock (+10), delegateVote (+10)

---

## Week 3 — ConfidentialAuction Rubric

| Criteria | Points |
|---|---|
| Compiles and deploys | 10 |
| Branch-free highest bid tracking (`FHE.gt` + `FHE.select`) | 25 |
| Each bidder can view their own bid | 10 |
| `revealWinner` correctly uses `makePubliclyDecryptable` | 15 |
| Deadline enforcement | 10 |
| Bid update logic | 10 |
| All 10 required test cases pass | 15 |
| Code quality | 5 |
| **Total** | **100** |

**Bonus:** Dutch auction variant (+15), encrypted reserve price (+10)

---

## Week 4 — Capstone Rubric

| Criteria | Points |
|---|---|
| Project compiles and all tests pass | 15 |
| Meaningful use of FHE (3+ distinct operations) | 15 |
| Correct permission design throughout | 20 |
| Branch-free logic throughout | 15 |
| Test coverage (15+ tests, quality + breadth) | 15 |
| README clarity and architecture explanation | 10 |
| Code quality, NatSpec, no anti-patterns | 10 |
| **Total** | **100** |

**Bonus:** Deployed on Devnet (+10), frontend dApp (+15), novel FHE use (+5)

---

## Grading Standards

**Full credit** — requirement is met exactly as specified, tests pass, code is clean.

**Partial credit (50–75%)** — requirement is partially met, or tests mostly pass with minor issues.

**Minimal credit (25–49%)** — significant gaps but student demonstrates understanding of the concept.

**No credit** — requirement is absent or fundamentally misunderstood.

## Resubmission Policy

Students may resubmit once per assignment within 48 hours of receiving feedback. Maximum score on resubmission is 85.
