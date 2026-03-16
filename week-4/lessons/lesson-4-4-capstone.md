# Lesson 4.4 — Capstone Project Briefing

**Duration:** 60 minutes | **Format:** Brief + Q&A + planning session

---

## Learning Goals

- Understand the full capstone requirements and grading criteria
- Choose the right project track for your skills and interests
- Plan your architecture before writing a single line of code
- Know what production-ready means in the context of this bootcamp

---

## 1. What the Capstone Is

The capstone is your chance to build something real. It is not a homework exercise — it is a complete, tested, deployable confidential application that you design from scratch.

The best capstone projects:
- Solve a genuine privacy problem that FHE uniquely addresses
- Apply everything from all 4 weeks
- Have clean architecture that another developer could extend
- Include tests that would catch real bugs
- Are documented well enough to be deployed by someone who didn't write it

---

## 2. Choosing Your Track

### Track A — Confidential DeFi

Best for: developers interested in finance, AMMs, or trading.

Strong project ideas:
- **Confidential lending protocol** — encrypted collateral and debt amounts; liquidation check branch-free via FHE.lt
- **Private yield aggregator** — encrypted allocation weights; rebalance without revealing strategy
- **Confidential stablecoin** — encrypted collateral ratios; stability checks via FHE.ge

What makes it strong: a real economic mechanism (liquidation, yield, stability) implemented fully branch-free.

### Track B — Confidential Identity

Best for: developers interested in compliance, KYC, credentials.

Strong project ideas:
- **Soulbound credential system** — encrypted attributes issued by authorities; on-chain eligibility without exposure
- **Private DAO** — membership stakes encrypted; governance power hidden from other members
- **Encrypted attestation registry** — third-party claims stored and queried privately

What makes it strong: a real compliance use case (regulatory, financial, access control) fully implemented.

### Track C — Confidential Gaming

Best for: developers interested in game theory, interactive contracts.

Strong project ideas:
- **Encrypted poker** — full 5-card hand evaluation branch-free
- **Hidden information strategy game** — each player has private state; moves computed on encrypted board
- **Confidential prediction market** — positions hidden until resolution

What makes it strong: non-trivial game logic implemented entirely without revealing private game state.

### Track D — Open Track

Build anything meaningful with FHEVM. Must include at least 3 distinct FHE operations and a real privacy justification.

---

## 3. Architecture Planning Template

Before writing code, fill in this template:

```
Project name: ___________
Track: ___________
Core privacy property: ___________
(What information stays hidden, from whom, for how long?)

Main contracts:
1. ___________  — role: ___________
2. ___________  — role: ___________

Key FHE operations used:
- FHE.___  for ___________
- FHE.___  for ___________
- FHE.___  for ___________

Permission flows:
- Who calls allowThis: ___________
- Who calls allow(user): ___________
- Any cross-contract flows requiring allowTransient: ___________

Potential FHE division issue? Yes / No
If yes, how handled: ___________

Main test scenarios (at least 5):
1. ___________
2. ___________
3. ___________
4. ___________
5. ___________
```

Complete this template before your first line of Solidity. Students who skip this step consistently spend more time debugging architecture than code.

---

## 4. What Production-Ready Means

| Not production-ready | Production-ready |
|---|---|
| Tests only cover happy path | Tests cover happy path, edge cases, and permission boundaries |
| No NatSpec | NatSpec on all public functions |
| No README | README with architecture diagram and deployment guide |
| Contract compiles but has anti-patterns | No anti-patterns; passes the Lesson 4.3 checklist |
| Hardcoded addresses | Configurable via constructor |
| No deployment script | Working deploy.ts for Zama Devnet |

---

## 5. Grading Walkthrough

The grading rubric (full details in [Week 4 Capstone](../homework/week-4-capstone.md)):

**15 pts — Compiles and tests pass**
The baseline. If tests don't pass, nothing else matters.

**15 pts — Meaningful FHE use**
Three or more distinct FHE operations that serve a real privacy purpose. Using `FHE.asEuint64(0)` three times does not count.

**20 pts — Correct permission design**
Every handle: `allowThis` before storage, `allow(user)` for all relevant users, `allowTransient` for cross-contract flows. Evaluated by reading the contract, not just running tests.

**15 pts — Branch-free logic throughout**
No `if(ebool)`, no `require(ebool)`. All conditional encrypted logic uses `FHE.select`. Evaluated by code review.

**15 pts — Test quality**
15+ test cases. Must include permission boundary tests (user A cannot read user B's data) and at least one anti-pattern test (demonstrate you know what breaks and handle it).

**10 pts — README and architecture**
A developer who has never seen your project can understand what it does, why FHE is used, and how to deploy it. An architecture diagram is required.

**10 pts — Code quality**
Clean Solidity, NatSpec, no magic numbers, no dead code, consistent naming.

---

## 6. Common Capstone Mistakes

**Mistake 1 — Too small in scope**
A single-function contract that only does `FHE.add` is not a capstone. Aim for at least 3 contracts or 8+ functions.

**Mistake 2 — FHE where it's not needed**
Encrypting data that has no privacy value wastes gas and complexity. Every encrypted value should have a clear justification: "this is hidden because..."

**Mistake 3 — Skipping test edge cases**
The most revealing tests are the ones that would catch real bugs: what happens when balance is exactly zero? What happens when two users try to claim the same resource?

**Mistake 4 — Missing deployment guide**
If the grader can't deploy your contract to Devnet in under 10 minutes, you lose the deployment bonus.

**Mistake 5 — No architecture explanation of FHE tradeoffs**
Every FHEVM project makes tradeoffs (public vs encrypted reserves, revert vs branch-free). Document them and explain why you chose what you chose.

---

## 7. Timeline

| Day | Activity |
|---|---|
| Thursday (today) | Choose track, complete architecture template |
| Thursday evening | Set up project repo, scaffold with create-fhevm-example |
| Friday | Write main contract(s) |
| Saturday | Write tests — aim for 15 before touching README |
| Sunday | README, NatSpec, deployment script, final review |
| Sunday night | Submit repo link |

---

## Office Hours

Instructors are available during:
- Friday: 4pm–6pm (Discord)
- Saturday: 10am–12pm (Discord)
- Sunday: 2pm–4pm (Discord — last chance before submission)

Post your architecture template in Discord before Friday office hours for early feedback.

---

📋 **Ready to build?** → [Read the full Capstone specification](../homework/week-4-capstone.md)
