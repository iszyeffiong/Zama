# EncryptedAgeVerification

**Difficulty:** 🟢 Beginner | **Concept:** Threshold check without revealing the actual value

## What It Demonstrates

A contract that verifies a user is 18 or older  without ever learning or storing their actual age. Uses `FHE.ge` for the comparison and stores the `ebool` result.

## Contract

```solidity
contract EncryptedAgeVerification is ZamaEthereumConfig {
    mapping(address => euint8) private ages;
    mapping(address => ebool)  private isAdult;

    function submitAge(externalEuint8 encryptedAge, bytes calldata proof) external {
        euint8 age = FHE.fromExternal(encryptedAge, proof);
        FHE.allowThis(age);
        ages[msg.sender] = age;

        // age >= 18  result is encrypted, only user can verify
        ebool adult = FHE.ge(age, FHE.asEuint8(18));
        FHE.allowThis(adult);
        FHE.allow(adult, msg.sender);
        isAdult[msg.sender] = adult;
    }

    function checkAdult(address user) external view returns (ebool) {
        return isAdult[user];  // Returns encrypted boolean handle
    }
}
```

## Test Cases

- ✓ Age 25 → isAdult decrypts to true
- ✓ Age 16 → isAdult decrypts to false
- ✓ Age exactly 18 → isAdult decrypts to true (ge, not gt)
- ✓ Verifier cannot decrypt without explicit grant
- ✓ Public decrypt reverts before publishing

```bash
npx create-fhevm-example encrypted-age-verification
```
