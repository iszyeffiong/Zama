# Lesson 4.2 — Private Order Books & Payroll

**Duration:** 60 minutes | **Format:** Two complete system walkthroughs

---

## Learning Goals

- Design a private limit order book with encrypted prices and sizes
- Build a confidential payroll system with encrypted salaries
- Handle complex multi-user encrypted state management
- Apply all permission, branch-free, and lifecycle patterns at scale

---

## 1. Private Order Book

An order book exchange matches buyers and sellers. Traditional on-chain order books expose all prices and sizes — enabling front-running, sandwich attacks, and strategic manipulation. A private order book keeps prices and sizes encrypted until matching occurs.

### Design

```
Bid Order: { encrypted price, encrypted size, bidder address }
Ask Order: { encrypted price, encrypted size, asker address }

Match condition: askPrice <= bidPrice (branch-free check)
Fill size:       min(bidSize, askSize) (branch-free)
```

### Implementation

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.24;

import {FHE, euint64, ebool, externalEuint64} from "@fhevm/solidity/lib/FHE.sol";
import {ZamaEthereumConfig} from "@fhevm/solidity/config/ZamaConfig.sol";

contract PrivateOrderBook is ZamaEthereumConfig {
    struct Order {
        euint64 price;
        euint64 size;
        address trader;
        bool    active;
    }

    Order[] public bids;
    Order[] public asks;

    event OrderPlaced(uint256 indexed orderId, bool isBid);
    event OrderMatched(uint256 bidId, uint256 askId);

    function placeBid(
        externalEuint64 priceExt, bytes calldata priceProof,
        externalEuint64 sizeExt,  bytes calldata sizeProof
    ) external returns (uint256 orderId) {
        euint64 price = FHE.fromExternal(priceExt, priceProof);
        euint64 size  = FHE.fromExternal(sizeExt,  sizeProof);

        FHE.allowThis(price); FHE.allowThis(size);
        FHE.allow(price, msg.sender); FHE.allow(size, msg.sender);

        orderId = bids.length;
        bids.push(Order(price, size, msg.sender, true));

        emit OrderPlaced(orderId, true);
    }

    function placeAsk(
        externalEuint64 priceExt, bytes calldata priceProof,
        externalEuint64 sizeExt,  bytes calldata sizeProof
    ) external returns (uint256 orderId) {
        euint64 price = FHE.fromExternal(priceExt, priceProof);
        euint64 size  = FHE.fromExternal(sizeExt,  sizeProof);

        FHE.allowThis(price); FHE.allowThis(size);
        FHE.allow(price, msg.sender); FHE.allow(size, msg.sender);

        orderId = asks.length;
        asks.push(Order(price, size, msg.sender, true));

        emit OrderPlaced(orderId, false);
    }

    /// @notice Attempt to match a bid against an ask
    /// Both orders partially or fully fill — branch-free
    function match(uint256 bidId, uint256 askId) external {
        Order storage bid = bids[bidId];
        Order storage ask = asks[askId];

        require(bid.active && ask.active, "Order not active");

        // Match condition: bid price >= ask price (buyer willing to pay at least ask price)
        ebool priceMatch = FHE.ge(bid.price, ask.price);

        // Fill size = min(bid.size, ask.size)
        ebool bidIsSmaller = FHE.le(bid.size, ask.size);
        euint64 fillSize   = FHE.select(bidIsSmaller, bid.size, ask.size);

        // Branch-free: only fill if prices match, else fill zero
        euint64 actualFill = FHE.select(priceMatch, fillSize, FHE.asEuint64(0));

        // Update remaining sizes
        euint64 newBidSize = FHE.sub(bid.size, actualFill);
        euint64 newAskSize = FHE.sub(ask.size, actualFill);

        FHE.allowThis(newBidSize); FHE.allow(newBidSize, bid.trader);
        FHE.allowThis(newAskSize); FHE.allow(newAskSize, ask.trader);
        FHE.allowThis(actualFill);
        FHE.allow(actualFill, bid.trader);
        FHE.allow(actualFill, ask.trader);

        bid.size = newBidSize;
        ask.size = newAskSize;

        emit OrderMatched(bidId, askId);
    }
}
```

---

## 2. Confidential Payroll System

A payroll system where:
- Salaries are encrypted — only the employee and employer know them
- Total payroll can be computed without revealing individual salaries
- Disbursement is verifiable without exposing amounts

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.24;

import {FHE, euint64, ebool, externalEuint64} from "@fhevm/solidity/lib/FHE.sol";
import {ZamaEthereumConfig} from "@fhevm/solidity/config/ZamaConfig.sol";

contract ConfidentialPayroll is ZamaEthereumConfig {
    address public employer;

    struct Employee {
        euint64 salary;       // monthly base salary
        euint64 bonus;        // one-time bonus
        euint64 totalPaid;    // running total disbursed
        bool    active;
    }

    mapping(address => Employee) private employees;
    address[] public employeeList;

    euint64 public totalPayrollBudget;  // total allocated, encrypted

    constructor() {
        employer = msg.sender;
    }

    modifier onlyEmployer() {
        require(msg.sender == employer, "Not employer");
        _;
    }

    // ----------------------------------------------------------------
    // Employee Management
    // ----------------------------------------------------------------

    function enroll(
        address employee,
        externalEuint64 salaryExt, bytes calldata salaryProof,
        externalEuint64 bonusExt,  bytes calldata bonusProof
    ) external onlyEmployer {
        euint64 salary = FHE.fromExternal(salaryExt, salaryProof);
        euint64 bonus  = FHE.fromExternal(bonusExt,  bonusProof);
        euint64 zero   = FHE.asEuint64(0);

        FHE.allowThis(salary); FHE.allow(salary, employee); FHE.allow(salary, employer);
        FHE.allowThis(bonus);  FHE.allow(bonus, employee);  FHE.allow(bonus, employer);
        FHE.allowThis(zero);   FHE.allow(zero, employee);

        employees[employee] = Employee(salary, bonus, zero, true);
        employeeList.push(employee);

        // Update total payroll budget
        euint64 compensation  = FHE.add(salary, bonus);
        euint64 newBudget     = FHE.add(totalPayrollBudget, compensation);
        FHE.allowThis(newBudget); FHE.allow(newBudget, employer);
        totalPayrollBudget = newBudget;
    }

    // ----------------------------------------------------------------
    // Pay Cycle
    // ----------------------------------------------------------------

    /// @notice Pay an employee their monthly salary
    function paySalary(address employee) external onlyEmployer {
        Employee storage emp = employees[employee];
        require(emp.active, "Not active");

        euint64 newTotal = FHE.add(emp.totalPaid, emp.salary);
        FHE.allowThis(newTotal);
        FHE.allow(newTotal, employee);
        FHE.allow(newTotal, employer);

        emp.totalPaid = newTotal;
    }

    /// @notice Pay bonus — one time, then zero it out
    function payBonus(address employee) external onlyEmployer {
        Employee storage emp = employees[employee];
        require(emp.active, "Not active");

        euint64 newTotal = FHE.add(emp.totalPaid, emp.bonus);
        FHE.allowThis(newTotal); FHE.allow(newTotal, employee); FHE.allow(newTotal, employer);

        // Zero out bonus after payment
        euint64 zeroed = FHE.asEuint64(0);
        FHE.allowThis(zeroed); FHE.allow(zeroed, employee);

        emp.totalPaid = newTotal;
        emp.bonus     = zeroed;
    }

    /// @notice Employee reads their own compensation data
    function getMyCompensation() external view returns (euint64 salary, euint64 paid) {
        Employee storage emp = employees[msg.sender];
        require(emp.active, "Not an employee");
        return (emp.salary, emp.totalPaid);
    }

    /// @notice Employer reads total payroll budget
    function getTotalBudget() external view returns (euint64) {
        return totalPayrollBudget;
    }
}
```

---

## 3. Privacy Analysis

### Order Book Privacy
- Order prices and sizes are encrypted — no front-running
- Match outcome (actualFill) is encrypted — only counterparties know the fill size
- The fact that a match was attempted is public (event emitted)
- Whether the match succeeded is NOT public — actualFill could be zero branch-free

### Payroll Privacy
- Individual salaries are encrypted — coworkers cannot compare salaries
- Total budget is encrypted — competitive salary intelligence is hidden
- Employee knows their own salary and total paid
- Employer knows all salaries (they set them)

---

## Hands-On Exercise (15 min)

Extend ConfidentialPayroll with:
1. A `terminate(address employee)` function — sets active=false and zeros salary branch-free
2. A `raiseSalary(address employee, externalEuint64 increment, bytes calldata proof)` function — adds increment to current salary branch-free
3. A `getHeadcount()` function — returns the number of active employees (this can be public — it doesn't reveal individual salaries)

---

📖 **Next:** [Lesson 4.3 — Security, Gas & Production Readiness](lesson-4-3-production.md)
