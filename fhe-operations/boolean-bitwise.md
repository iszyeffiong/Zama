# Boolean & Bitwise Operations

## Boolean Logic (on ebool)

```solidity
ebool andResult = FHE.and(a, b);   // a && b
ebool orResult  = FHE.or(a, b);    // a || b
ebool notResult = FHE.not(a);      // !a
ebool xorResult = FHE.xor(a, b);   // a ^ b
```

Useful for combining compliance conditions:

```solidity
ebool isCompliant = FHE.and(
    FHE.and(isKYC[user], isAdult[user]),
    FHE.not(isSanctioned[user])
);
```

## Bitwise Operations (on euintX)

```solidity
euint64 andBits = FHE.and(a, b);   // bitwise AND
euint64 orBits  = FHE.or(a, b);    // bitwise OR
euint64 xorBits = FHE.xor(a, b);   // bitwise XOR
euint64 notBits = FHE.not(a);      // bitwise NOT
```

## Shift & Rotation

```solidity
euint64 shlResult  = FHE.shl(a, k);    // left shift by plaintext k
euint64 shrResult  = FHE.shr(a, k);    // right shift by plaintext k
euint64 rotlResult = FHE.rotl(a, k);   // rotate left by plaintext k
euint64 rotrResult = FHE.rotr(a, k);   // rotate right by plaintext k
```

> The shift/rotation amount `k` must be a **plaintext** value.
