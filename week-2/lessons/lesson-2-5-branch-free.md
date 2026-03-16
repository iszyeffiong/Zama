# Lesson 2.5 — Branch-Free Programming

**Duration:** 60 minutes | **Format:** Pattern library + hands-on exercises

---

## Learning Goals

- Translate any conditional Solidity pattern into its branch-free FHE equivalent
- Chain multiple FHE.select calls for complex multi-condition logic
- Use boolean combinators (FHE.and, FHE.or, FHE.not) for compound conditions
- Understand why branch-free is a privacy guarantee, not just a workaround

---

## 1. Why Branch-Free is a Privacy Feature

In a standard smart contract, a revert leaks information about private state:

```solidity
// BAD — this revert tells the world "balance < withdrawAmount"
if (balance >= withdrawAmount) {
    balance -= withdrawAmount;
} else {
    revert("Insufficient funds");
}
```

An attacker probes by sending increasing withdrawal amounts until the revert stops — extracting the exact balance without ever decrypting it. This is a side-channel attack via revert.

Branch-free programming closes this leak entirely:

```solidity
// GOOD — no revert, no information leak, always succeeds
ebool hasEnough = FHE.ge(balance, withdrawAmount);
euint64 newBalance = FHE.select(
    hasEnough,
    FHE.sub(balance, withdrawAmount),
    balance
);
```

---

## 2. The Core Pattern

```
// Plaintext conditional
result = condition ? trueValue : falseValue;

// FHE equivalent — both branches always computed
euint64 result = FHE.select(condition, trueValue, falseValue);
```

Key properties:
- condition is always ebool
- Both trueValue and falseValue are ALWAYS computed
- Encrypted condition picks which becomes the result
- No information leaks about which branch ran

---

## 3. Pattern Library

### Safe subtraction — no underflow, no revert

```solidity
ebool canSubtract = FHE.ge(a, b);
euint64 result = FHE.select(canSubtract, FHE.sub(a, b), FHE.asEuint64(0));
```

### Maximum of two encrypted values

```solidity
ebool aIsLarger = FHE.gt(a, b);
euint64 max = FHE.select(aIsLarger, a, b);
```

### Minimum of two encrypted values

```solidity
ebool aIsSmaller = FHE.lt(a, b);
euint64 min = FHE.select(aIsSmaller, a, b);
```

### Conditional state update — only update if condition true

```solidity
// Only replace stored bid if new bid is higher
ebool isHigher = FHE.gt(newBid, highestBid);
euint64 updatedHighest = FHE.select(isHigher, newBid, highestBid);
```

### AND — both conditions must be true

```solidity
ebool isKYC    = registry.isKYC(user);
ebool isAdult  = FHE.ge(ages[user], FHE.asEuint8(18));
ebool eligible = FHE.and(isKYC, isAdult);
euint64 amount = FHE.select(eligible, requested, FHE.asEuint64(0));
```

### OR — either condition must be true

```solidity
ebool isPremium     = FHE.eq(tier[user], FHE.asEuint8(2));
ebool isWhitelisted = whitelist[user];
ebool hasAccess     = FHE.or(isPremium, isWhitelisted);
```

### NOT — invert a condition

```solidity
ebool isSanctioned = sanctionsList[user];
ebool isClean      = FHE.not(isSanctioned);
```

### Three-way selection — chained selects

```solidity
// Grade: 1=A (>=90), 2=B (>=70), 3=C (below 70)
ebool isA = FHE.ge(score, FHE.asEuint8(90));
ebool isB = FHE.ge(score, FHE.asEuint8(70));

euint8 bOrC  = FHE.select(isB, FHE.asEuint8(2), FHE.asEuint8(3));
euint8 grade = FHE.select(isA, FHE.asEuint8(1), bOrC);
```

---

## 4. Full Example — Compliant Transfer

The most important real-world branch-free pattern. Used in every compliant ERC7984 token:

```solidity
function transfer(address to, externalEuint64 amountExt, bytes calldata proof) external {
    euint64 amount = FHE.fromExternal(amountExt, proof);

    // Build compound condition
    ebool senderHasEnough    = FHE.ge(balances[msg.sender], amount);
    ebool recipientKYC       = registry.isKYC(to);
    ebool recipientNotBanned = FHE.not(banned[to]);
    ebool recipientEligible  = FHE.and(recipientKYC, recipientNotBanned);
    ebool canTransfer        = FHE.and(senderHasEnough, recipientEligible);

    // Compute both outcomes branch-free
    euint64 newSenderBal = FHE.select(
        canTransfer, FHE.sub(balances[msg.sender], amount), balances[msg.sender]
    );
    euint64 newRecipientBal = FHE.select(
        canTransfer, FHE.add(balances[to], amount), balances[to]
    );

    // Grant permissions on every new handle
    FHE.allowThis(newSenderBal);    FHE.allow(newSenderBal, msg.sender);
    FHE.allowThis(newRecipientBal); FHE.allow(newRecipientBal, to);

    balances[msg.sender] = newSenderBal;
    balances[to]         = newRecipientBal;
}
```

If conditions pass: balances update. If any fails: both stay unchanged. Transaction always succeeds. No revert. No leak.

---

## 5. Gas Implications

Branch-free code always computes both branches — more expensive than a conditional revert on the false path. This is the deliberate privacy tradeoff.

Optimisation tip: put cheapest comparisons (eq, ge) first in your FHE.and chains. Avoid expensive operations (mul, div) on both sides of a select unless necessary.

---

## 6. Common Mistakes

### Mistake 1 — Using ebool in require()
```solidity
// WRONG — ebool is not a Solidity bool
require(FHE.ge(balance, amount), "Insufficient");

// CORRECT
euint64 result = FHE.select(FHE.ge(balance, amount), FHE.sub(balance, amount), balance);
```

### Mistake 2 — Forgetting to re-grant after FHE.select
```solidity
// WRONG — newVal has no permissions
euint64 newVal = FHE.select(condition, a, b);
stored = newVal;

// CORRECT
euint64 newVal = FHE.select(condition, a, b);
FHE.allowThis(newVal);
FHE.allow(newVal, msg.sender);
stored = newVal;
```

---

## Hands-On Exercises (20 min)

Translate these to branch-free FHE:

**Exercise 1 — Capped value:**
Write cappedValue(euint64 value, uint64 cap) that returns value if <= cap, else returns cap.

**Exercise 2 — Update high score:**
Write updateHighScore(euint64 newScore) that only updates stored euint64 highScore if newScore is higher.

**Exercise 3 — Triple condition transfer:**
Transfer only if: sender has enough AND recipient is not blocked AND amount > 0. Use FHE.and chains — no if/else.

Solutions in exercises/2-5-branch-free-solutions.sol in the starter repo.

---

📋 Week 2 complete! Take the [Week 2 Quiz](../../quizzes/week-2-quiz.md) then submit [Week 2 Homework](../homework/week-2-homework.md)
