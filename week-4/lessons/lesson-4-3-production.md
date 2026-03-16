# Lesson 4.3 — Security, Gas & Production Readiness

**Duration:** 60 minutes | **Format:** Checklist-driven review + auditing techniques

---

## Learning Goals

- Identify the security risks unique to FHEVM contracts
- Understand gas costs and how to optimise FHE-heavy contracts
- Apply a production readiness checklist before deployment
- Know how to write NatSpec documentation for encrypted contracts

---

## 1. Security Risks Unique to FHEVM

### Risk 1 — Unauthorized Handle Access

Any contract can receive a handle — but only the FHE coprocessor enforces who can use it. If you grant overly broad permissions, any contract that receives the handle can operate on it.

```solidity
// DANGEROUS — grants any address decrypt rights
FHE.allow(sensitiveHandle, address(0));  // Don't do this

// SAFE — grant only to the intended address
FHE.allow(sensitiveHandle, specificUser);
```

### Risk 2 — Irreversible Public Decryption

`makePubliclyDecryptable` cannot be undone. Adding it to the wrong function (e.g., callable by anyone) permanently exposes private data.

```solidity
// DANGEROUS — anyone can reveal confidential data
function reveal() external {
    FHE.makePubliclyDecryptable(salaries[anyEmployee]);
}

// SAFE — gated by role and conditions
function revealAuctionResult() external {
    require(block.timestamp >= deadline, "Too early");
    require(msg.sender == owner, "Not owner");
    FHE.makePubliclyDecryptable(highestBid);
}
```

### Risk 3 — Handle Reuse Across Contexts

A handle from one user's input should never be mixed with another user's data flow without explicit re-authorization.

```solidity
// DANGEROUS — using Alice's handle in Bob's computation
function crossContaminate(euint64 aliceHandle) external {
    euint64 result = FHE.add(aliceHandle, balances[bob]);  // Alice's data enters Bob's flow
```

### Risk 4 — Stale Handle After Revocation

As covered in Lesson 2.2 — revoking permissions does not invalidate existing handles. Design assuming previously granted handles remain readable indefinitely.

### Risk 5 — Integer Overflow in FHE Operations

FHE arithmetic wraps on overflow — there is no automatic revert like Solidity's checked arithmetic. Always validate ranges before FHE operations:

```solidity
// If balance is near max uint64, adding a large amount wraps to a small number
// Validate: balance + amount <= type(uint64).max
// Do this check on plaintext inputs before encrypting
```

---

## 2. Gas Cost Model

FHE operations are significantly more expensive than plaintext operations:

| Operation | Approximate gas cost |
|---|---|
| `FHE.add(euint64, euint64)` | ~200,000 gas |
| `FHE.mul(euint64, euint64)` | ~400,000 gas |
| `FHE.eq(euint64, euint64)` | ~150,000 gas |
| `FHE.select(ebool, euint64, euint64)` | ~250,000 gas |
| `FHE.fromExternal` | ~100,000 gas |
| Plaintext `uint256` add | ~5 gas |

**Key optimisation principle:** Minimise the number of FHE operations in hot paths.

### Optimisation 1 — Move plaintext checks outside FHE

```solidity
// INEFFICIENT — FHE check on a value that could be validated in plaintext
ebool isValid = FHE.gt(encryptedAmount, FHE.asEuint64(0));

// EFFICIENT — validate min amount client-side before encrypting
// Only submit encrypted inputs that pass plaintext validation
require(plainAmount > 0, "Zero amount");  // revert before tx is submitted
```

### Optimisation 2 — Batch FHE operations

```solidity
// INEFFICIENT — separate allowThis calls
FHE.allowThis(a);
FHE.allowThis(b);
FHE.allowThis(c);

// EFFICIENT — same gas, but combine where computation allows
euint64 combined = FHE.add(a, b);
FHE.allowThis(combined);  // One permission call for the combined result
```

### Optimisation 3 — Avoid redundant FHE.eq chains

```solidity
// INEFFICIENT — 5 separate FHE.eq calls
ebool present = FHE.or(FHE.or(FHE.eq(g, s[0]), FHE.eq(g, s[1])), ...);

// In production: consider a Merkle commitment to the secret word
// and verify membership proof off-chain, reducing FHE ops to just exact match
```

---

## 3. NatSpec for Encrypted Contracts

Always document what is encrypted, who can decrypt, and what side-channel properties hold:

```solidity
/// @title ConfidentialAuction
/// @notice Sealed-bid auction — bids stay encrypted until reveal
/// @dev Privacy properties:
///   - Bid amounts are encrypted with euint64 — not visible on-chain
///   - Highest bid is tracked branch-free via FHE.select — no leak about current leader
///   - After reveal(), highestBid is publicly decryptable — irreversible
///   - Individual bids remain private even after reveal
/// @dev Trust assumptions:
///   - Zama FHE coprocessor is honest and available
///   - Input proofs bind bids to bidder address — replay protected
contract ConfidentialAuction is ZamaEthereumConfig {

    /// @notice Place an encrypted bid
    /// @param encryptedBid The encrypted bid amount (euint64 handle)
    /// @param proof Input proof binding the bid to this contract and msg.sender
    /// @dev Permission: bidder receives FHE.allow to decrypt their own bid
    /// @dev Privacy: bid amount never visible on-chain; highest bid updated branch-free
    function placeBid(externalEuint64 encryptedBid, bytes calldata proof) external { ... }
}
```

---

## 4. Production Readiness Checklist

Work through this checklist before deploying any FHEVM contract:

### Security
- [ ] Every `makePubliclyDecryptable` call is gated by role + timing conditions
- [ ] No `FHE.allow(handle, address(0))` calls
- [ ] Input validation on plaintext parameters before they become ciphertexts
- [ ] Reentrancy guards on functions that modify encrypted state
- [ ] Access control on all sensitive functions (onlyOwner, onlyIssuer, etc.)

### Correctness
- [ ] Every stored handle has `FHE.allowThis` called before storage
- [ ] Every user-facing result has `FHE.allow(handle, user)` called
- [ ] Every cross-contract handle pass has `FHE.allowTransient` called before the call
- [ ] No `if(ebool)` — all branches use `FHE.select`
- [ ] No `FHE.div(a, encryptedB)` — divisors are always plaintext
- [ ] Integer overflow impossible for all reasonable input ranges

### Testing
- [ ] All happy-path flows tested
- [ ] All anti-patterns tested (missing allowThis, missing allow, etc.)
- [ ] Permission boundary tests (user A cannot decrypt user B's data)
- [ ] Minimum 15 test cases for any contract above intermediate complexity
- [ ] Tests run cleanly with `npm test` from a fresh clone

### Documentation
- [ ] NatSpec on all public/external functions
- [ ] Privacy properties documented at contract level
- [ ] Trust assumptions stated
- [ ] Architecture diagram in README

### Deployment
- [ ] Deployment script works on Zama Devnet
- [ ] Constructor arguments documented
- [ ] Event names and signatures documented
- [ ] Emergency functions (pause, emergency stop) considered

---

## 5. Common Audit Findings in FHEVM Contracts

Based on patterns seen across community contracts:

| Finding | Frequency | Severity |
|---|---|---|
| Missing `allowThis` on stored handle | Very common | High |
| `makePubliclyDecryptable` callable by anyone | Common | Critical |
| No minimum amount validation before FHE ops | Common | Medium |
| Overflow possible in FHE arithmetic | Occasional | High |
| Stale handle assumptions after permission revoke | Occasional | Medium |
| Missing `allowTransient` in multi-contract flow | Common | High |

---

## Hands-On Exercise (15 min)

Audit the following contract and find all security issues:

```solidity
contract BrokenPayroll {
    mapping(address => euint64) private salaries;
    address public owner;

    function setSalary(address emp, externalEuint64 sal, bytes calldata proof) external {
        salaries[emp] = FHE.fromExternal(sal, proof);
    }

    function reveal(address emp) external {
        FHE.makePubliclyDecryptable(salaries[emp]);
    }

    function getSalary(address emp) external view returns (uint64) {
        return uint64(euint64.unwrap(salaries[emp]));
    }

    function payCycle(address emp) external {
        euint64 newTotal = FHE.add(salaries[emp], salaries[emp]);
        salaries[emp] = newTotal;
    }
}
```

Find: 5 issues — security, correctness, and privacy.

---

📖 **Next:** [Lesson 4.4 — Capstone Project Briefing](lesson-4-4-capstone.md)
