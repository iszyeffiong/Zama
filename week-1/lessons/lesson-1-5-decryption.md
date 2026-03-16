# Lesson 1.5 — Decryption Patterns

**Duration:** 60 minutes | **Format:** Reading + hands-on

---

## Learning Goals

- Implement user decryption for private results
- Implement public decryption for shared reveals
- Understand when each pattern is appropriate

---

## Pattern 1 — User Decryption

The user privately decrypts their own result. Most common pattern.

**Contract:**
```solidity
mapping(address => euint64) private results;

function compute(externalEuint64 input, bytes calldata proof) external {
    euint64 val = FHE.fromExternal(input, proof);
    euint64 result = FHE.add(val, FHE.asEuint64(10));

    FHE.allowThis(result);
    FHE.allow(result, msg.sender);  // Grant decrypt rights

    results[msg.sender] = result;
}

function getResult() external view returns (euint64) {
    return results[msg.sender];
}
```

**Client:**
```typescript
const handle = await contract.getResult();
const plaintext = await fhevm.userDecryptEuint(
  FhevmType.euint64, handle, contractAddress, userSigner
);
```

---

## Pattern 2 — Public Decryption

Anyone can read the result after an on-chain reveal. Use for: auction winners, vote totals, game outcomes.

**Contract:**
```solidity
euint64 public winningBid;

function revealWinner() external onlyAfterDeadline {
    FHE.makePubliclyDecryptable(winningBid);  // Irreversible!
}
```

> ⚠️ `makePubliclyDecryptable` cannot be undone. Only use for values truly meant to be public.

---

## Pattern 3 — Selective Access

Grant decrypt rights to specific addresses (auditors, compliance officers, observers).

```solidity
function grantAuditorAccess(address auditor) external onlyOwner {
    FHE.allow(sensitiveHandle, auditor);
}
```

---

## Decryption Checklist

Before deploying, verify for every encrypted result:
- [ ] Is `FHE.allowThis` called if the contract stores it?
- [ ] Is `FHE.allow(handle, user)` called for every user who should read it?
- [ ] Is `makePubliclyDecryptable` only used for genuinely public values?
- [ ] Does the client use `userDecryptEuint` with the correct signer?

---

📋 **Week 1 complete!** → [Take the Week 1 Homework](../homework/week-1-homework.md)
