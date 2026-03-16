# Decryption Patterns

Decryption in FHEVM always happens **off-chain**. Contracts store and operate on handles; users decrypt results using the FHEVM SDK.

## User Decryption

The most common pattern. The contract grants `FHE.allow(handle, user)`, and the user decrypts off-chain.

### Contract side

```solidity
function compute(externalEuint64 input, bytes calldata proof) external {
    euint64 val = FHE.fromExternal(input, proof);
    euint64 result = FHE.add(val, FHE.asEuint64(10));

    FHE.allowThis(result);
    FHE.allow(result, msg.sender);  // Grant decrypt rights

    results[msg.sender] = result;
}

function getResult() external view returns (euint64) {
    return results[msg.sender];  // Returns handle — NOT plaintext
}
```

### Client side

```typescript
import { fhevm } from "hardhat";
import { FhevmType } from "@fhevm/hardhat-plugin";

// Get the handle
const handle = await contract.getResult();

// Decrypt off-chain
const plaintext = await fhevm.userDecryptEuint(
  FhevmType.euint64,
  handle,
  contractAddress,
  userSigner
);

console.log("Result:", plaintext.toString());
```

## Public Decryption

Used when a value should be revealed to everyone after an on-chain condition — auction ends, vote closes, game resolves.

### Contract side

```solidity
euint64 public winningBid;

function revealWinner() external onlyAfterDeadline {
    // Makes value readable by anyone — irreversible
    FHE.makePubliclyDecryptable(winningBid);
}
```

### Client side

```typescript
// Anyone can now decrypt
const plaintext = await fhevm.publicDecryptEuint(
  FhevmType.euint64,
  handle,
  contractAddress
);
```

> ⚠️ `makePubliclyDecryptable` is permanent and irreversible. Only call it when you genuinely intend to reveal a value publicly.

## Decrypting Multiple Values

```solidity
// Contract stores multiple handles
mapping(address => euint64[5]) public letterFeedback;
```

```typescript
// Client decrypts each handle
const feedback = await Promise.all(
  handles.map(handle =>
    fhevm.userDecryptEuint(FhevmType.euint64, handle, contractAddress, user)
  )
);
```

## Common Mistake: Expecting Plaintext from View Functions

```solidity
// ❌ This does NOT return a readable number
function getBalance() external view returns (uint64) {
    return uint64(euint64.unwrap(balances[msg.sender]));  // Returns handle, not balance
}

// ✅ Return the encrypted handle — decrypt off-chain
function getBalance() external view returns (euint64) {
    return balances[msg.sender];
}
```
