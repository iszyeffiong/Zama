# EncryptSingleValue

**Difficulty:** 🟢 Beginner | **Concept:** Store one encrypted value with permissions

## What It Demonstrates

The three essential steps for any FHEVM contract:
1. Validate the encrypted input with `FHE.fromExternal`
2. Grant the contract permission with `FHE.allowThis`
3. Grant the user decrypt rights with `FHE.allow`

## Contract

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.24;

import {FHE, euint64, externalEuint64} from "@fhevm/solidity/lib/FHE.sol";
import {ZamaEthereumConfig} from "@fhevm/solidity/config/ZamaConfig.sol";

contract EncryptSingleValue is ZamaEthereumConfig {
    euint64 private storedValue;

    function store(externalEuint64 encryptedValue, bytes calldata inputProof) external {
        euint64 val = FHE.fromExternal(encryptedValue, inputProof);
        FHE.allowThis(val);          // Contract can reuse this handle
        FHE.allow(val, msg.sender);  // User can decrypt
        storedValue = val;
    }

    function retrieve() external view returns (euint64) {
        return storedValue;  // Returns handle — decrypt off-chain
    }
}
```

## Test

```typescript
const encrypted = await fhevm
  .createEncryptedInput(contractAddress, user.address)
  .add64(42n)
  .encrypt();

await contract.connect(user).store(encrypted.handles[0], encrypted.inputProof);

const handle = await contract.retrieve();
const plaintext = await fhevm.userDecryptEuint(
  FhevmType.euint64, handle, contractAddress, user
);

expect(plaintext).to.equal(42n);
```

## Key Points

- Always call `FHE.allowThis` before storing a handle
- `retrieve()` returns a `euint64` handle, not a number
- Decryption happens off-chain using the FHEVM SDK


---

📝 **Ready to test your knowledge?** → [Take the Basic Examples Quiz](../../quizzes/examples-basic-quiz.md)
