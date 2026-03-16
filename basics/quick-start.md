# Quick Start — Scaffold in Seconds

The `create-fhevm-example` CLI scaffolds a fully configured FHEVM project in one command — Hardhat, encrypted types, FHE keys, network config, and tests all included.

## Requirements

- Node.js **v20.0.0** or higher (we recommend the latest LTS)
- No global installs needed — `npx` works out of the box

## Bootstrap a New Project

```bash
# Interactive wizard — recommended for first time
npx create-fhevm-example

# Or specify an example directly
npx create-fhevm-example blind-auction
```

## Three Modes

### 1. Single Example Mode

Perfect for learning one FHE concept step by step.

```bash
npx create-fhevm-example blind-auction
```

Creates a focused project with one contract and full test suite.

### 2. Category Bundle Mode

Download an entire domain at once.

```bash
npx create-fhevm-example --category advanced
```

| Category | Count | What's included |
|---|---|---|
| `basic` | 11 | Counters, encryption, decryption fundamentals |
| `concepts` | 8 | Access control, handles, proofs, anti-patterns |
| `gaming` | 4 | Poker, lottery, rock-paper-scissors, blackjack |
| `openzeppelin` | 5 | ERC7984, wrappers, swaps, vesting |
| `advanced` | 6 | Blind auction, escrow, voting, KYC, payroll, order book |

### 3. Smart Injection Mode — upgrade an existing project

Already have a Hardhat project? Add FHE in one command:

```bash
cd my-existing-project
npx create-fhevm-example --add
```

This will:
1. Detect your Hardhat configuration
2. Add `@fhevm/solidity` and `@fhevm/hardhat-plugin`
3. Update `hardhat.config.ts` with FHE imports
4. Inject a sample contract and test of your choice

## CLI Options

| Option | Description |
|---|---|
| `--example <name>` | Create a single example project |
| `--category <name>` | Create a category bundle |
| `--add` | Inject FHE into an existing Hardhat project |
| `--target <dir>` | Target directory for `--add` mode |
| `--output <dir>` | Output directory |
| `--help` | Show help |

## Project Structure

Every scaffolded project follows this layout:

```
my-fhevm-project/
├── contracts/
│   └── MyContract.sol       # Your FHE-enabled contract
├── test/
│   ├── types.ts             # Shared type definitions
│   └── MyContract.ts        # Mocha/Chai test suite
├── hardhat.config.ts        # Pre-configured for FHEVM
├── package.json
├── .env.example
└── README.md
```

## Get Running

```bash
npm install
npm run compile
npm run test
```

## Networks

Projects come pre-configured for:

| Network | Description |
|---|---|
| Local (Hardhat) | FHEVM mock — fast local testing, no real FHE |
| Zama Devnet | Sepolia-based testnet with real FHE coprocessor |

Add more networks in `hardhat.config.ts`.

## FAQ

**Does this work with existing Hardhat projects?**
Yes — use `--add` inside your project directory.

**Can I use this offline?**
Yes for scaffolding. Only `npm install` needs internet to download dependencies.

**Do I need to install anything globally?**
No — `npx` handles everything.


---

📝 **Ready to test your knowledge?** → [Take the Quiz](../quizzes/quick-start-quiz.md)
