# Lesson 2.4 — Anti-Patterns & Pitfalls

**Duration:** 60 minutes | **Format:** Show the wrong code first, then fix it

---

## Learning Goals

- Recognise all 7 FHEVM anti-patterns on sight
- Know exactly what symptom each produces
- Apply the correct fix immediately

---

## Anti-Pattern 1 — Missing `allowThis`

**Symptom:** Contract works on first deploy, fails silently on subsequent calls.

```solidity
// ❌ WRONG
function store(externalEuint64 input, bytes calldata proof) external {
    euint64 val = FHE.fromExternal(input, proof);
    storedValue = val;   // No allowThis — next tx cannot use storedValue
}

// ✅ CORRECT
function store(externalEuint64 input, bytes calldata proof) external {
    euint64 val = FHE.fromExternal(input, proof);
    FHE.allowThis(val);   // Grant BEFORE storing
    storedValue = val;
}
```

---

## Anti-Pattern 2 — Missing `allow(user)`

**Symptom:** User receives a handle but decryption fails with "not authorized."

```solidity
// ❌ WRONG
function compute(externalEuint64 input, bytes calldata proof) external {
    euint64 result = FHE.add(FHE.fromExternal(input, proof), FHE.asEuint64(10));
    FHE.allowThis(result);
    results[msg.sender] = result;
    // User gets the handle but cannot decrypt it
}

// ✅ CORRECT
function compute(externalEuint64 input, bytes calldata proof) external {
    euint64 result = FHE.add(FHE.fromExternal(input, proof), FHE.asEuint64(10));
    FHE.allowThis(result);
    FHE.allow(result, msg.sender);   // Now user can decrypt
    results[msg.sender] = result;
}
```

---

## Anti-Pattern 3 — Branching on `ebool`

**Symptom:** Compile error or undefined runtime behavior.

```solidity
// ❌ WRONG — ebool is not bool
ebool isAdult = FHE.ge(age, FHE.asEuint8(18));
if (isAdult) { grantAccess(); }   // Does not compile

// ✅ CORRECT — branch-free selection
ebool isAdult = FHE.ge(age, FHE.asEuint8(18));
euint64 access = FHE.select(isAdult, FHE.asEuint64(1), FHE.asEuint64(0));
```

---

## Anti-Pattern 4 — View Functions Returning Plaintext

**Symptom:** Caller expects a number, receives a meaningless large integer.

```solidity
// ❌ WRONG — returns a handle cast to uint64, not the actual value
function getBalance() external view returns (uint64) {
    return uint64(euint64.unwrap(balances[msg.sender]));
}

// ✅ CORRECT — return the handle; decrypt off-chain
function getBalance() external view returns (euint64) {
    return balances[msg.sender];
}
```

Client-side:
```typescript
const handle = await contract.getBalance();
const balance = await fhevm.userDecryptEuint(FhevmType.euint64, handle, contractAddress, signer);
```

---

## Anti-Pattern 5 — Wrong Input Proof Binding

**Symptom:** `FHE.fromExternal` reverts unexpectedly.

```typescript
// ❌ WRONG — proof bound to wrong address
const encrypted = await fhevm
  .createEncryptedInput(WRONG_ADDRESS, user.address)
  .add64(value)
  .encrypt();

// ✅ CORRECT — exact deployed contract address
const encrypted = await fhevm
  .createEncryptedInput(CONTRACT_ADDRESS, user.address)
  .add64(value)
  .encrypt();
```

---

## Anti-Pattern 6 — Dividing by Encrypted Value

**Symptom:** Unsupported operation error or unexpected revert.

```solidity
// ❌ NOT SUPPORTED
euint64 result = FHE.div(numerator, encryptedDenominator);

// ✅ SUPPORTED — denominator must be plaintext
euint64 result = FHE.div(numerator, uint64(1000));
```

---

## Anti-Pattern 7 — Missing `allowTransient` in Cross-Contract Calls

**Symptom:** Cross-contract call reverts with authorization error even though `allowThis` is set.

```solidity
// ❌ WRONG — Contract B has no permission to use the handle
function swap(address contractB) external {
    ISwap(contractB).execute(balances[msg.sender]);   // Reverts

// ✅ CORRECT
function swap(address contractB) external {
    FHE.allowTransient(balances[msg.sender], contractB);  // Grant first
    ISwap(contractB).execute(balances[msg.sender]);        // Now works
}
```

---

## Anti-Pattern Reference Table

| # | Anti-Pattern | Symptom | Fix |
|---|---|---|---|
| 1 | Missing `allowThis` | Silent failure on reuse | `allowThis` before storing |
| 2 | Missing `allow(user)` | Decrypt fails | `allow(user)` after computing |
| 3 | Branch on `ebool` | Compile error | Use `FHE.select` |
| 4 | View returns plaintext | Wrong value | Return `euint64` handle |
| 5 | Wrong proof binding | `fromExternal` reverts | Use exact addresses |
| 6 | Divide by encrypted | Unsupported op | Divisor must be plaintext |
| 7 | Missing `allowTransient` | Cross-contract revert | `allowTransient` before call |

---

## Exercise

Open `exercises/2-4-antipatterns.sol`. Seven functions each contain one of the above anti-patterns. Identify and fix all seven without looking at the reference table.

---

📖 **Next:** [Lesson 2.5 — Branch-Free Programming](lesson-2-5-branch-free.md)
