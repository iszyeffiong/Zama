# Lesson 3.3 — Identity & Compliance Systems

**Duration:** 60 minutes | **Format:** Architecture + code walkthroughs

---

## Learning Goals

- Build an encrypted identity registry storing multiple attributes
- Implement threshold checks (age verification, score checks) without revealing values
- Combine multiple compliance conditions with FHE.and boolean logic
- Design a KYC-gated system where compliance is provable without identity exposure

---

## 1. The Problem with Public Identity On-Chain

Traditional on-chain identity systems require publishing personal data — date of birth, nationality, credit score — to verify eligibility. This creates:
- Privacy violations (permanent public record)
- Regulatory risk (GDPR, data protection laws)
- Honeypots (aggregated identity data is a target)

FHE solves this: store encrypted attributes, prove eligibility from them, reveal nothing.

---

## 2. Encrypted Identity Registry

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.24;

import {FHE, euint8, euint16, ebool, externalEuint8, externalEuint16, externalEbool} from "@fhevm/solidity/lib/FHE.sol";
import {ZamaEthereumConfig} from "@fhevm/solidity/config/ZamaConfig.sol";

contract IdentityRegistry is ZamaEthereumConfig {
    struct Identity {
        euint8  age;          // encrypted age
        euint16 creditScore;  // encrypted credit score 0-850
        euint8  countryCode;  // encrypted ISO country code
        ebool   isVerified;   // encrypted KYC status
    }

    mapping(address => Identity) private identities;
    mapping(address => bool)     public  registered;
    address public               issuer;  // trusted identity issuer

    modifier onlyIssuer() {
        require(msg.sender == issuer, "Not issuer");
        _;
    }

    constructor() {
        issuer = msg.sender;
    }

    /// @notice Issuer registers an identity with encrypted attributes
    function register(
        address subject,
        externalEuint8  ageExt,         bytes calldata ageProof,
        externalEuint16 scoreExt,       bytes calldata scoreProof,
        externalEuint8  countryExt,     bytes calldata countryProof,
        externalEbool   verifiedExt,    bytes calldata verifiedProof
    ) external onlyIssuer {
        Identity storage id = identities[subject];

        id.age         = FHE.fromExternal(ageExt,      ageProof);
        id.creditScore = FHE.fromExternal(scoreExt,    scoreProof);
        id.countryCode = FHE.fromExternal(countryExt,  countryProof);
        id.isVerified  = FHE.fromExternal(verifiedExt, verifiedProof);

        // Contract needs access to compute checks
        FHE.allowThis(id.age);
        FHE.allowThis(id.creditScore);
        FHE.allowThis(id.countryCode);
        FHE.allowThis(id.isVerified);

        // Subject can decrypt their own attributes
        FHE.allow(id.age,         subject);
        FHE.allow(id.creditScore, subject);
        FHE.allow(id.countryCode, subject);
        FHE.allow(id.isVerified,  subject);

        registered[subject] = true;
    }

    /// @notice Check if user is 18+ — returns encrypted bool
    function isAdult(address subject) external view returns (ebool) {
        require(registered[subject], "Not registered");
        return FHE.ge(identities[subject].age, FHE.asEuint8(18));
    }

    /// @notice Check if credit score is above threshold
    function hasCreditScore(address subject, uint16 minimum) external view returns (ebool) {
        require(registered[subject], "Not registered");
        return FHE.ge(identities[subject].creditScore, FHE.asEuint16(minimum));
    }

    /// @notice Combined eligibility: verified AND adult AND credit score >= 650
    function isEligibleForLoan(address subject) external view returns (ebool) {
        require(registered[subject], "Not registered");
        Identity storage id = identities[subject];

        ebool adult   = FHE.ge(id.age, FHE.asEuint8(18));
        ebool score   = FHE.ge(id.creditScore, FHE.asEuint16(650));
        ebool eligible = FHE.and(FHE.and(id.isVerified, adult), score);

        return eligible;
    }
}
```

---

## 3. Encrypted Age Verification — Standalone Pattern

A common pattern: prove age threshold without revealing the exact age:

```solidity
contract AgeGatedService is ZamaEthereumConfig {
    IdentityRegistry public registry;

    constructor(address _registry) {
        registry = IdentityRegistry(_registry);
    }

    mapping(address => euint8) private accessLevels;

    function requestAccess(address user) external {
        // Get encrypted age check from registry
        ebool isAdult  = registry.isAdult(user);
        ebool isSenior = FHE.ge(registry.identities(user).age, FHE.asEuint8(65));

        // Branch-free access level assignment
        // 0=no access, 1=adult access, 2=senior access
        euint8 adultLevel  = FHE.select(isAdult,  FHE.asEuint8(1), FHE.asEuint8(0));
        euint8 accessLevel = FHE.select(isSenior, FHE.asEuint8(2), adultLevel);

        FHE.allowThis(accessLevel);
        FHE.allow(accessLevel, user);
        accessLevels[user] = accessLevel;
    }
}
```

---

## 4. Compliance Rules — Combining Multiple Checks

```solidity
contract ComplianceEngine is ZamaEthereumConfig {
    IdentityRegistry public registry;

    // Compliance check: KYC verified AND not sanctioned AND country is allowed
    function checkCompliance(address user, uint8 allowedCountry) external view returns (ebool) {
        require(registry.registered(user), "Not registered");

        ebool isKYC       = registry.identities(user).isVerified;
        ebool rightCountry = FHE.eq(
            registry.identities(user).countryCode,
            FHE.asEuint8(allowedCountry)
        );

        return FHE.and(isKYC, rightCountry);
    }
}
```

---

## 5. KYC-Gated Token Transfer

Using the registry in a token contract:

```solidity
contract KYCToken is ZamaEthereumConfig {
    mapping(address => euint64) private balances;
    IdentityRegistry public registry;

    function transfer(address to, externalEuint64 amtExt, bytes calldata proof) external {
        euint64 amount = FHE.fromExternal(amtExt, proof);

        // Both sender and recipient must be KYC verified
        ebool senderKYC    = registry.identities(msg.sender).isVerified;
        ebool recipientKYC = registry.identities(to).isVerified;
        ebool bothKYC      = FHE.and(senderKYC, recipientKYC);

        // Sender must have enough balance
        ebool hasEnough    = FHE.ge(balances[msg.sender], amount);
        ebool canTransfer  = FHE.and(bothKYC, hasEnough);

        // Branch-free transfer
        euint64 newSender    = FHE.select(canTransfer, FHE.sub(balances[msg.sender], amount), balances[msg.sender]);
        euint64 newRecipient = FHE.select(canTransfer, FHE.add(balances[to], amount), balances[to]);

        FHE.allowThis(newSender);    FHE.allow(newSender, msg.sender);
        FHE.allowThis(newRecipient); FHE.allow(newRecipient, to);

        balances[msg.sender] = newSender;
        balances[to]         = newRecipient;
    }
}
```

---

## 6. Privacy Properties

What this system achieves:
- The issuer knows the attributes (they registered them)
- The subject can read their own attributes
- Third-party contracts can run compliance checks (returns ebool — pass/fail)
- Nobody else learns the actual attribute values
- On-chain, only encrypted handles are visible

What this system does NOT achieve:
- Protection from the issuer — they saw plaintext values at registration
- Protection from a compromised coprocessor
- Anonymity — on-chain addresses are still public

---

## Hands-On Exercise (15 min)

Extend IdentityRegistry to support:
1. An `updateCreditScore` function — issuer only, updates the score
2. A `selfReport` function — subject updates their own age (but issuer must countersign off-chain)
3. A `revokeIdentity` function — issuer only, sets isVerified to false branch-free

---

📖 **Next:** [Lesson 3.4 — OpenZeppelin Confidential & ERC7984](lesson-3-4-erc7984.md)
