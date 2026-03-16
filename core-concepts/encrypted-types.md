# Encrypted Types

FHEVM introduces a family of encrypted Solidity types. These replace their plaintext counterparts wherever privacy is needed.

## Type Reference

| Type | Plaintext equivalent | Description |
|---|---|---|
| `ebool` | `bool` | Encrypted boolean |
| `euint8` | `uint8` | Encrypted 8-bit unsigned integer |
| `euint16` | `uint16` | Encrypted 16-bit unsigned integer |
| `euint32` | `uint32` | Encrypted 32-bit unsigned integer |
| `euint64` | `uint64` | Encrypted 64-bit unsigned integer |
| `euint128` | `uint128` | Encrypted 128-bit unsigned integer |
| `euint256` | `uint256` | Encrypted 256-bit unsigned integer |
| `eaddress` | `address` | Encrypted address |

## Important: Types are Handles

These types are **opaque 256-bit handles**  references to ciphertexts stored by the FHE coprocessor. They are not the values themselves.

```solidity
euint64 balance = FHE.asEuint64(100);
// balance holds a handle (uint256 pointer), not the number 100
```

All FHE operations take handles as input and return new handles as output.

## Importing Types

```solidity
import {FHE, euint64, euint8, ebool, externalEuint64} from "@fhevm/solidity/lib/FHE.sol";
import {ZamaEthereumConfig} from "@fhevm/solidity/config/ZamaConfig.sol";
```

## External Types

When receiving encrypted inputs from users, use the `external` variants:

| External type | Used for |
|---|---|
| `externalEuint8` | User-submitted encrypted uint8 |
| `externalEuint16` | User-submitted encrypted uint16 |
| `externalEuint32` | User-submitted encrypted uint32 |
| `externalEuint64` | User-submitted encrypted uint64 |
| `externalEbool` | User-submitted encrypted bool |

```solidity
function submit(externalEuint64 encryptedValue, bytes calldata inputProof) external {
    euint64 value = FHE.fromExternal(encryptedValue, inputProof);
    // Now value is a verified, usable handle
}
```


---

📝 **Ready to test your knowledge?** → [Take the Core Concepts Quiz](../quizzes/core-concepts-quiz.md)
