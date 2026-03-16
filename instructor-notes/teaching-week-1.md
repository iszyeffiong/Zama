# Teaching Week 1 — Instructor Notes

## Pre-Session Checklist

- [ ] Confirm all students have Node.js v20+ installed
- [ ] Test `npx create-fhevm-example encrypt-single-value` in your environment
- [ ] Prepare a backup repo in case npm is slow during live session

---

## Lesson 1.2 — The Handle Model (Most Important)

This is the single most important lesson. If students leave without understanding that `euint64` stores a handle — not a number — they will struggle for the entire bootcamp.

**Concrete demo to run live:**
```typescript
const counter = FHE.asEuint64(42);
console.log(counter); // prints 0x3f7a... — not 42!
```

The moment students see a hex hash instead of 42, the model clicks.

**Whiteboard exercise:** Draw the flow and ask students to fill in the blanks:
```
FHE.add(a, b) → c
               ├── FHE.allowThis(c)   ← Who needs this?
               └── FHE.allow(c, ?)    ← Who needs this?
```

---

## Lesson 1.3 — First Contract Lab

Budget 25 min for setup — it always takes longer in a group. Have a pre-scaffolded fallback repo at a short URL.

**Most common Day 1 error:**
```
Error: handle not authorized
```
Always means `allowThis` was not called. Turn this into a teaching moment — explain why before fixing.

---

## Homework Tips

When students ask "should I revert if insufficient funds for withdraw?" — the answer is **no**. Use `FHE.select`. Reverting would leak information about the balance.
