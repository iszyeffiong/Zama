# View Functions on Encrypted Values

**Severity:** Medium — API confusion, unexpected client behavior

## The Problem

A view function that returns an encrypted value returns a **handle** (a uint256 pointer), not a plaintext number. Callers who expect a number will be surprised.

## Wrong

```solidity
// ❌ Misleading return type — returns a handle, not a balance
function getBalance() external view returns (uint64) {
    return uint64(euint64.unwrap(balances[msg.sender]));
}
```

## Correct

```solidity
// ✅ Explicit — clearly returns an encrypted handle
function getBalance() external view returns (euint64) {
    return balances[msg.sender];
}
```

Then decrypt off-chain:

```typescript
const handle = await contract.getBalance();
const balance = await fhevm.userDecryptEuint(
  FhevmType.euint64, handle, contractAddress, userSigner
);
```

## Rule

Decryption always happens **off-chain**. View functions can return handles; never try to return plaintext from an encrypted value in Solidity.


---

📝 **Ready to test your knowledge?** → [Take the Anti-Patterns Quiz](../quizzes/anti-patterns-quiz.md)
