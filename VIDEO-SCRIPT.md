# 🎬 Demo Video Script — FHEVM Dev Bootcamp

**Target length:** 4:30–5:00 minutes
**Format:** Screen recording with voiceover (Loom recommended)
**Screen:** GitBook open in browser throughout

---

## Pre-recording Setup

- Open the GitBook to the README page
- Have the browser zoomed to 110% for readability
- Have a terminal ready with the bootcamp repo open
- Practice the full run-through at least once before recording

---

## SCRIPT

---

### [0:00–0:20] — Hook & Introduction

*(On screen: GitBook README — "FHEVM Dev Bootcamp")*

> "What if your smart contracts could process sensitive data — auction bids, balances, votes, identity attributes — without ever revealing them on-chain?
>
> That's what Fully Homomorphic Encryption makes possible. And this bootcamp teaches developers how to build with it.
>
> I'm walking you through the FHEVM Dev Bootcamp — a complete 4-week curriculum that takes any Solidity developer from zero FHE knowledge to shipping production-ready confidential applications on Zama Protocol."

---

### [0:20–1:00] — Curriculum Structure Overview

*(On screen: Scroll through README → click Curriculum Map)*

> "The bootcamp is structured as four progressive weeks, each building on the last.

> Week 1 is foundations — students learn what the handle model is, write their first encrypted contract, and understand how decryption works.

> Week 2 is where things get deeper — input proofs, the full permission system, and all the common pitfalls that trip up FHEVM developers.

> Week 3 is real use cases — encrypted games, sealed-bid auctions, identity systems, and the ERC7984 confidential token standard.

> And Week 4 is the capstone — students build a complete, tested, production-ready DeFi protocol of their choice and ship it to Zama Devnet.

> The bootcamp is designed to work both as a live cohort — with 2 sessions per week, office hours, and peer review — and as a fully self-paced program."

*(On screen: Click Week 1 Overview, show the weekly schedule table)*

> "Each week has a clear milestone. For example, at the end of Week 1, every student will have submitted a working PrivateVault contract — an encrypted savings vault with deposit, withdraw, and threshold checks."

---

### [1:00–2:30] — Sample Lesson Walkthrough

*(On screen: Navigate to Lesson 1.2 — Encrypted Types & the Handle Model)*

> "Let me walk you through Lesson 1.2 — which is the most important lesson in the entire bootcamp.

> The central insight students need to grasp is this: when you write `euint64 balance` in Solidity, that variable does not store a number. It stores a 256-bit handle — an opaque pointer to a ciphertext stored by the FHE coprocessor off-chain.

> We teach this with a concrete demo."

*(On screen: Scroll to the code block showing handle = hex hash not 42)*

> "Students run this in their terminal — they create an encrypted 42, log it to the console, and instead of seeing 42, they see a hex hash. That moment of surprise is where the model clicks.

> From there we cover the handle lifecycle — how to create handles from user inputs, from plaintext constants, and from FHE operations. And critically — that every FHE operation creates a brand new handle with no permissions."

*(On screen: Scroll to the permissions code block)*

> "This is the golden rule we repeat throughout the bootcamp: create handle → grant allowThis so the contract can reuse it → grant allow(user) so they can decrypt → then store.

> The lesson ends with a hands-on exercise — students build a deposit function from scratch, run into the 'handle not authorized' error when they forget allowThis, and fix it themselves."

*(On screen: Navigate to Lesson 1.3 briefly)*

> "The follow-up lesson — 1.3 — takes them through a full 90-minute live coding lab building EncryptedCounter. By the end of Tuesday of Week 1, they've written and tested their first complete encrypted contract."

---

### [2:30–3:45] — Homework Design Philosophy

*(On screen: Navigate to Week 1 Homework — PrivateVault)*

> "Now let me talk about the homework design philosophy — because this is where the real learning happens.

> Every homework assignment produces a complete, deployable contract. No toy examples, no fill-in-the-blanks. Real code that could go to production.

> Week 1's homework is the PrivateVault — a single-owner encrypted savings vault. Let me show you why every requirement is deliberate."

*(On screen: Scroll through the specification)*

> "The deposit function is straightforward — it's practice for the basic store-and-grant pattern. But withdraw is where the lesson is. The requirement says: if the user withdraws more than their balance, don't revert — use FHE.select to transfer zero instead.

> Why? Because reverting on insufficient balance leaks information. An attacker could probe your balance by sending progressively larger withdrawals and watching for reverts. FHE.select is branch-free — both outcomes are computed, and the encrypted condition picks the result without leaking which path was taken.

> Students who really internalize that design decision are thinking like FHEVM developers."

*(On screen: Scroll to grading rubric table)*

> "The rubric is fully transparent — students see exactly what each requirement is worth before they start. 20 points for branch-free logic alone. Code quality counts for 5 — we care about clean, commented code from Day 1.

> And there are always bonus challenges — the Week 1 bonus asks students to add an auditor access grant function, which previews the access control patterns from Week 2."

*(On screen: Show test cases list)*

> "Every homework specifies the required test cases. Students must test edge cases — the overdraft scenario, the second user having an independent balance, and permissions boundary tests — not just the happy path."

---

### [3:45–4:20] — Production Readiness & Capstone

*(On screen: Navigate to Week 4 Capstone)*

> "By Week 4, students are ready to build the real thing. The capstone has four tracks — confidential DeFi, identity systems, gaming, or an open track.

> Each submission requires: at minimum 15 test cases, NatSpec documentation, a deployment script for Zama Devnet, and a README with an architecture diagram.

> The bonus track includes deploying live to Devnet and building a frontend — which is where participants go from bootcamp graduates to builders ready to ship."

*(On screen: Navigate back to README)*

> "The full curriculum — 4 weeks, 20 lessons, 4 homework assignments, 4 quizzes, and complete instructor notes — is structured as a GitBook with a SUMMARY.md that auto-generates navigation.

> Everything is ready to deploy. Upload the folder to GitBook, connect a GitHub repo, and you have a live learning platform in minutes."

---

### [4:20–4:50] — Close

*(On screen: README)*

> "To summarize: the FHEVM Dev Bootcamp is a production-ready 4-week curriculum with clear weekly milestones, fully specced homework with grading rubrics, complete instructor notes, and a coherent progression from encrypted hello-world to production DeFi.

> It works for live cohorts and self-paced learners. It's designed for Web3 developers who already know Solidity — and want to add on-chain privacy to everything they build.

> The bootcamp is ready to run. Thank you."

---

## Recording Tips

- Speak at a relaxed pace — 5 minutes feels short but goes fast when you're narrating
- Pause briefly before each new section — gives viewers time to follow your navigation
- Zoom in (`Ctrl +`) before showing code blocks
- Don't rush the Lesson 1.2 section — that's the heart of the demo
- If you stumble, pause and restart the sentence — edits are easy in Loom

## Recommended Loom Settings

- Record: Screen + camera (small bubble bottom-right)
- Resolution: 1080p
- Background: neutral — remove clutter behind you
- Microphone: use a headset or AirPods — built-in mic picks up room noise
