# Access Control & Permissions

The FHE permission system controls who can use and decrypt a ciphertext. Every handle starts with no permissions — you must explicitly grant them.

## The Three Permission Functions

### `FHE.allowThis(handle)`

Grants the **contract itself** the right to use this handle in future transactions.

```solidity
euint64 result = FHE.add(a, b);
FHE.allowThis(result);   // Contract can use result in later calls
storedResult = result;
```

Without this, the next transaction that tries to use `storedResult` will fail.

### `FHE.allow(handle, address)`

Grants a **specific address** the right to decrypt this handle.

```solidity
FHE.allow(result, msg.sender);    // User can decrypt their own result
FHE.allow(result, adminAddress);  // Admin can also decrypt
```

### `FHE.allowTransient(handle, contractAddress)`

Grants **one-transaction** usage rights to another contract. Useful for cross-contract flows.

```solidity
FHE.allowTransient(tokenHandle, swapContractAddress);
ISwap(swapContractAddress).execute(tokenHandle);
// After this transaction, swapContract loses access
```

## Granting Access After the Fact

Permissions can be granted at any time, not just at creation:

```solidity
function grantAccess(address recipient) external onlyOwner {
    FHE.allow(storedHandle, recipient);
}
```

## Revoking Access

Revoking does **not** invalidate previously obtained ciphertexts. If a user already holds a decryptable handle, they can still decrypt it after revocation.

```solidity
// Even after revoking, handles the user already obtained remain decryptable
function revokeAccess(address user) external onlyOwner {
    // This prevents future grants only — cannot retroactively revoke
}
```

## Multi-Contract Flows

When passing handles between contracts, use `allowTransient`:

```solidity
// Contract A — token
function transferTo(address swapContract) external {
    FHE.allowTransient(balances[msg.sender], swapContract);
    ISwap(swapContract).receiveToken(balances[msg.sender]);
}

// Contract B — swap
function receiveToken(euint64 amount) external {
    // Can use amount here because of allowTransient
    // After this transaction ends, access expires
    FHE.allowThis(amount);  // Re-grant for contract B's own future use
}
```

## Checklist

Before every FHE operation, ask:
- [ ] Did I call `FHE.allowThis()` on every handle I want to store or reuse?
- [ ] Did I call `FHE.allow(handle, user)` for every user who should be able to decrypt?
- [ ] Did I call `FHE.allowTransient()` before passing a handle to another contract?
- [ ] Did I re-grant permissions on **new** handles produced by FHE operations?


---

📝 **Ready to test your knowledge?** → [Take the Core Concepts Quiz](../quizzes/core-concepts-quiz.md)
