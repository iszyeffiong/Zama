# Type Conversion

## Creating Encrypted Values from Plaintext

```solidity
euint8   v8   = FHE.asEuint8(value);
euint16  v16  = FHE.asEuint16(value);
euint32  v32  = FHE.asEuint32(value);
euint64  v64  = FHE.asEuint64(value);
euint128 v128 = FHE.asEuint128(value);
euint256 v256 = FHE.asEuint256(value);
ebool    vb   = FHE.asEbool(true);
```

## Casting Between Encrypted Types

```solidity
// Upcast (smaller → larger, always safe)
euint8  small  = FHE.asEuint8(42);
euint64 big    = FHE.asEuint64(small);   // safe upcast

// Downcast (larger → smaller, may truncate)
euint64 large  = FHE.asEuint64(1000);
euint8  narrow = FHE.asEuint8(large);    // truncates if > 255
```

> ⚠️ Downcasting truncates silently  there is no overflow check in FHE operations. Ensure values fit within the target type's range.

## Converting from User Input

User-submitted encrypted inputs always use `FHE.fromExternal()`:

```solidity
// externalEuint64 → euint64
euint64 val = FHE.fromExternal(encryptedInput, inputProof);
```


---

📝 **Ready to test your knowledge?** → [Take the FHE Operations Quiz](../quizzes/fhe-operations-quiz.md)
