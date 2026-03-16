# Lesson 2.3 — Handle Lifecycle Mastery

**Duration:** 90 minutes | **Format:** Lab — trace, break, fix

---

## Learning Goals

- Trace handle creation and permissions across a multi-step contract
- Identify exactly which line breaks a handle lifecycle
- Write a contract that correctly manages 5+ handles simultaneously
- Understand symbolic execution and why derived handles are independent

---

## 1. Symbolic Execution — How Handles Are Derived

Every FHE operation performs **symbolic execution**: it records the computation as a new node in the FHE computation graph, returning a new handle that represents "the result of this operation."

```solidity
euint64 a = FHE.asEuint64(10);   // Handle A: represents plaintext 10
euint64 b = FHE.asEuint64(20);   // Handle B: represents plaintext 20
euint64 c = FHE.add(a, b);       // Handle C: represents A+B (derived)
euint64 d = FHE.add(c, a);       // Handle D: represents C+A (further derived)
```

Handles A, B, C, D are all independent. Changing permissions on A has zero effect on C or D.

---

## 2. Full Lifecycle Example — Step by Step

```solidity
contract LifecycleDemo is ZamaEthereumConfig {
    euint64 public balance;      // stored handle
    euint64 public lastDeposit;  // stored handle

    function deposit(externalEuint64 amtExt, bytes calldata proof) external {
        // STEP 1: Validate and create handle from user input
        euint64 amt = FHE.fromExternal(amtExt, proof);
        // amt = new handle, no permissions yet

        // STEP 2: Compute new balance (creates another new handle)
        euint64 newBalance = FHE.add(balance, amt);
        // newBalance = new handle, no permissions yet

        // STEP 3: Grant permissions on EACH handle we want to keep
        FHE.allowThis(amt);              // contract can reuse deposit amount
        FHE.allow(amt, msg.sender);      // user can see their deposit

        FHE.allowThis(newBalance);       // contract can use balance next tx
        FHE.allow(newBalance, msg.sender);

        // STEP 4: Store
        balance = newBalance;
        lastDeposit = amt;
    }
}
```

**Trace each handle:**

| Handle | Created by | allowThis? | allow(user)? | Stored? |
|---|---|---|---|---|
| `amt` | `fromExternal` | ✅ | ✅ | `lastDeposit` |
| `newBalance` | `FHE.add` | ✅ | ✅ | `balance` |

---

## 3. The Broken Lifecycle — Find the Bug

```solidity
// ❌ This contract has 3 bugs. Can you find them all?
contract BrokenVault is ZamaEthereumConfig {
    euint64 public balance;

    function deposit(externalEuint64 amt, bytes calldata proof) external {
        euint64 amount = FHE.fromExternal(amt, proof);
        balance = FHE.add(balance, amount);     // Bug 1
        FHE.allowThis(balance);                  // Bug 2
        FHE.allow(amount, msg.sender);           // Bug 3
    }
}
```

**Bug 1:** `FHE.add(balance, amount)` creates a new handle but it's assigned directly to `balance` without `allowThis` first — any subsequent read of `balance` from storage fails because the handle has no permissions at the time of storage.

**Bug 2:** `FHE.allowThis(balance)` is called *after* `balance = ...`. This is too late — the handle is already stored without permission. Always grant before storing.

**Bug 3:** `FHE.allow(amount, msg.sender)` grants the user decrypt rights to the *deposit amount*, not the balance. If the user wants to see their balance, the grant should be on the new balance handle.

**Fixed version:**
```solidity
function deposit(externalEuint64 amt, bytes calldata proof) external {
    euint64 amount = FHE.fromExternal(amt, proof);
    euint64 newBalance = FHE.add(balance, amount);

    FHE.allowThis(newBalance);             // Grant BEFORE storing
    FHE.allow(newBalance, msg.sender);     // Grant on the BALANCE, not deposit amt

    balance = newBalance;                  // Store AFTER granting
}
```

---

## 4. Multi-Handle Contract — The Payroll Example

Managing several encrypted fields simultaneously:

```solidity
contract ConfidentialPayroll is ZamaEthereumConfig {
    struct Employee {
        euint64 salary;
        euint64 bonus;
        euint64 totalPaid;
        ebool   isActive;
    }

    mapping(address => Employee) private employees;

    function enroll(
        externalEuint64 salaryExt, bytes calldata salaryProof,
        externalEuint64 bonusExt,  bytes calldata bonusProof
    ) external onlyOwner {
        euint64 salary = FHE.fromExternal(salaryExt, salaryProof);
        euint64 bonus  = FHE.fromExternal(bonusExt, bonusProof);
        euint64 zero   = FHE.asEuint64(0);
        ebool   active = FHE.asEbool(true);

        // Grant permissions on ALL four handles
        FHE.allowThis(salary);
        FHE.allowThis(bonus);
        FHE.allowThis(zero);
        FHE.allowThis(active);

        // Employee can see their own data
        address emp = msg.sender;
        FHE.allow(salary, emp);
        FHE.allow(bonus, emp);
        FHE.allow(zero, emp);
        FHE.allow(active, emp);

        employees[emp] = Employee(salary, bonus, zero, active);
    }

    function payCycle(address employee) external onlyOwner {
        Employee storage e = employees[employee];

        euint64 payment    = FHE.add(e.salary, e.bonus);
        euint64 newTotal   = FHE.add(e.totalPaid, payment);

        // Re-grant on EVERY new handle
        FHE.allowThis(payment);
        FHE.allowThis(newTotal);
        FHE.allow(newTotal, employee);

        e.totalPaid = newTotal;
    }
}
```

---

## Lab Exercise (40 min)

Open `exercises/2-3-lifecycle.sol` in the starter repo.

The contract has 6 intentional lifecycle bugs. Using only what you've learned:

1. Read the contract carefully
2. Mark every line where a handle is created
3. Verify `allowThis` is called before every storage operation
4. Verify `allow(user)` is called for every user who needs decrypt access
5. Fix all 6 bugs
6. Run the test suite — all tests should pass

---

📖 **Next:** [Lesson 2.4 — Anti-Patterns & Pitfalls](lesson-2-4-anti-patterns.md)
