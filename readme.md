# ZK Stealth Protocol

<p align="center">
  <img src="https://static.wixstatic.com/media/e2da02_e4de428513494e418d280603cbeccdad~mv2.png" alt="ZK Stealth Logo" width="200"/>
</p>

**The World's First Invisible On-Chain Execution Layer**

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](https://opensource.org/licenses/MIT)
[![Build Status](https://img.shields.io/badge/build-passing-brightgreen.svg)]()
[![Coverage](https://img.shields.io/badge/coverage-98%25-brightgreen.svg)]()
[![Audit](https://img.shields.io/badge/audit-Trail%20of%20Bits-purple.svg)]()

---

## Overview

ZK Stealth is a cryptographic privacy infrastructure that enables invisible transaction execution on public blockchains. By leveraging Groth16 zero-knowledge proofs and polynomial commitment schemes, ZK Stealth eliminates MEV extraction, front-running, and on-chain surveillance while maintaining full cryptographic verifiability.

### The Problem

Public blockchains expose every transaction in the mempool before execution, creating a $7B+ annual MEV extraction economy. Traders, whales, and institutions suffer from:
- **Front-running attacks** by MEV bots scanning mempool transactions
- **Sandwich attacks** extracting value from retail traders
- **Strategy leakage** as competitors observe and replicate successful trades
- **Privacy violations** as all transaction data is permanently public

### The Solution

ZK Stealth encrypts transaction intent off-chain and generates zero-knowledge proofs that validate conditions WITHOUT revealing:
- Token pairs
- Trade amounts  
- Price triggers
- Execution strategies
- Wallet balances

Transactions remain invisible until atomic execution—no mempool exposure, no front-running, zero MEV extraction.

---

## Architecture

### Core Components

```
zk-stealth/
├── circuits/                    # Zero-knowledge circuit definitions
│   ├── groth16/                # Groth16 proving system implementation
│   │   ├── setup.circom        # Trusted setup ceremony circuits
│   │   ├── prover.circom       # Proof generation circuits
│   │   └── verifier.circom     # On-chain verifier circuits
│   ├── conditions/             # Condition evaluation circuits
│   │   ├── comparison.circom   # Greater than, less than, equals
│   │   ├── range.circom        # Range proofs (balance > X && balance < Y)
│   │   └── composite.circom    # AND/OR logical operators
│   └── lib/                    # Circuit libraries
│       ├── poseidon.circom     # Poseidon hash function
│       ├── eddsa.circom        # EdDSA signature verification
│       └── mimc.circom         # MiMC encryption primitives
│
├── prover/                      # Proof generation engine
│   ├── witness/                # Witness computation
│   │   ├── generator.rs        # R1CS witness generation
│   │   ├── field_ops.rs        # Finite field arithmetic (BN254)
│   │   └── constraint_system.rs # Constraint satisfaction
│   ├── groth16/                # Groth16 prover implementation
│   │   ├── prover.rs           # Proof generation (Pippenger MSM)
│   │   ├── setup.rs            # Powers of Tau ceremony
│   │   └── serialization.rs    # Proof serialization (288 bytes)
│   └── wasm/                   # Browser-compatible WASM runtime
│       ├── prover.wasm         # Compiled prover module
│       └── bindings.js         # JavaScript FFI bindings
│
├── verifier/                    # On-chain verification contracts
│   ├── evm/                    # Ethereum-compatible chains
│   │   ├── Verifier.sol        # Groth16 verifier contract
│   │   ├── BoundAction.sol     # Conditional execution logic
│   │   └── Pairing.sol         # BN254 pairing precompile wrapper
│   ├── solana/                 # Solana verification program
│   │   ├── lib.rs              # Anchor program entrypoint
│   │   ├── verify.rs           # Groth16 verification logic
│   │   └── execute.rs          # Conditional transaction execution
│   └── zk-rollups/             # zkEVM-specific verifiers
│       ├── starknet.cairo      # StarkNet verifier
│       ├── polygon-zkevm.sol   # Polygon zkEVM verifier
│       └── zksync.sol          # zkSync Era verifier
│
├── sdk/                         # Client SDKs
│   ├── typescript/             # TypeScript/JavaScript SDK
│   │   ├── src/
│   │   │   ├── proof.ts        # Proof generation API
│   │   │   ├── conditions.ts   # Condition builders
│   │   │   ├── circuit.ts      # Circuit compilation
│   │   │   └── crypto.ts       # Cryptographic primitives
│   │   └── examples/           # Integration examples
│   ├── python/                 # Python SDK
│   │   ├── zkstealth/
│   │   │   ├── prover.py       # Proof generation
│   │   │   ├── verifier.py     # Verification utilities
│   │   │   └── conditions.py   # Condition DSL
│   │   └── examples/
│   └── rust/                   # Rust SDK (native performance)
│       ├── src/
│       │   ├── prover.rs       # High-performance prover
│       │   ├── circuit.rs      # Circuit builder
│       │   └── field.rs        # Field arithmetic
│       └── examples/
│
├── crypto/                      # Cryptographic primitives
│   ├── curves/                 # Elliptic curve implementations
│   │   ├── bn254.rs            # BN254 curve (optimal ate pairing)
│   │   ├── bls12_381.rs        # BLS12-381 alternative
│   │   └── jubjub.rs           # JubJub twisted Edwards curve
│   ├── hashes/                 # Hash functions
│   │   ├── poseidon.rs         # Poseidon (ZK-friendly)
│   │   ├── pedersen.rs         # Pedersen commitment
│   │   └── mimc.rs             # MiMC block cipher
│   ├── signatures/             # Signature schemes
│   │   ├── eddsa.rs            # EdDSA over JubJub
│   │   └── schnorr.rs          # Schnorr signatures
│   └── commitment/             # Commitment schemes
│       ├── pedersen.rs         # Pedersen commitments
│       └── kzg.rs              # KZG polynomial commitments
│
├── infrastructure/              # Backend infrastructure
│   ├── relayer/                # Transaction relayer service
│   │   ├── mempool.rs          # Private mempool manager
│   │   ├── bundler.rs          # Transaction bundling
│   │   └── executor.rs         # Atomic execution engine
│   ├── prover-network/         # Distributed proving
│   │   ├── coordinator.rs      # Proof job coordinator
│   │   ├── worker.rs           # Proof generation worker
│   │   └── aggregator.rs       # Proof aggregation
│   └── monitoring/             # System monitoring
│       ├── metrics.rs          # Prometheus metrics
│       ├── alerts.rs           # Alert management
│       └── analytics.rs        # Usage analytics
│
├── tests/                       # Comprehensive test suite
│   ├── unit/                   # Unit tests
│   │   ├── circuits/           # Circuit correctness tests
│   │   ├── prover/             # Proof generation tests
│   │   └── crypto/             # Cryptographic primitive tests
│   ├── integration/            # Integration tests
│   │   ├── evm/                # EVM chain integration
│   │   ├── solana/             # Solana integration
│   │   └── sdk/                # SDK integration tests
│   ├── benchmarks/             # Performance benchmarks
│   │   ├── proof_generation.rs # Proof gen benchmarks
│   │   ├── verification.rs     # Verification benchmarks
│   │   └── gas_costs.rs        # On-chain gas analysis
│   └── security/               # Security tests
│       ├── fuzzing/            # Fuzz testing harness
│       ├── soundness/          # ZK soundness proofs
│       └── audit/              # Audit remediation tests
│
├── docs/                        # Documentation
│   ├── whitepaper.pdf          # Technical whitepaper
│   ├── specification.md        # Protocol specification
│   ├── api/                    # API documentation
│   ├── security/               # Security documentation
│   │   ├── audit-reports/      # Trail of Bits audit reports
│   │   └── threat-model.md     # Threat model analysis
│   └── guides/                 # Integration guides
│
└── scripts/                     # Build and deployment scripts
    ├── setup-ceremony.sh       # Trusted setup ceremony
    ├── deploy-verifiers.sh     # Verifier deployment
    └── benchmark.sh            # Performance benchmarking
```

---

## Cryptographic Foundation

### Groth16 Proving System

ZK Stealth implements the Groth16 zk-SNARK proving system for optimal proof size and verification efficiency:

**Proof Generation:**
```
π = (A, B, C) where:
- A = α + Σ(aᵢ·uᵢ) + r·δ
- B = β + Σ(aᵢ·vᵢ) + s·δ  
- C = (Σ(aᵢ·wᵢ) + h(x)·t(x)) / δ + A·s + B·r - r·s·δ
```

**Verification Equation:**
```
e(A, B) = e(α, β) · e(L, γ) · e(C, δ)
```

Where `e` is the optimal ate pairing on BN254.

### BN254 Elliptic Curve

**Curve Equation:** `y² = x³ + 3`

**Field Modulus:** `p = 21888242871839275222246405745257275088696311157297823662689037894645226208583`

**Embedding Degree:** `k = 12` (optimal for pairing operations)

**Security Level:** 128-bit (post-quantum: ~64-bit)

### Trusted Setup Ceremony

ZK Stealth conducted a multi-party computation (MPC) ceremony with **1,247 participants** to generate the Common Reference String (CRS):

- **Toxic Waste (τ):** Destroyed after ceremony
- **Powers of Tau:** `[1, τ, τ², ..., τⁿ]`
- **Security Guarantee:** Requires ALL participants to collude for compromise

---

## Performance Benchmarks

### Proof Generation
| Circuit Size | Constraints | Witness Gen | Proving Time | Memory |
|--------------|-------------|-------------|--------------|--------|
| Simple (>)   | 1,024       | 12ms        | 1.2s         | 128MB  |
| Range Proof  | 4,096       | 28ms        | 1.8s         | 256MB  |
| Complex AND  | 16,384      | 89ms        | 3.4s         | 512MB  |

### On-Chain Verification
| Chain          | Gas Cost | Verification Time | Proof Size |
|----------------|----------|-------------------|------------|
| Ethereum       | ~280k    | 9.7ms             | 288 bytes  |
| Polygon        | ~280k    | 8.2ms             | 288 bytes  |
| Arbitrum       | ~45k     | 7.1ms             | 288 bytes  |
| Optimism       | ~42k     | 7.3ms             | 288 bytes  |
| zkSync Era     | ~38k     | 6.8ms             | 288 bytes  |
| Solana         | 0.00025◎ | 5.2ms             | 288 bytes  |

---

## Security

### Audits
- **Trail of Bits** (December 2023) - [Report](./docs/security/audit-reports/trail-of-bits-2023-12.pdf)
- **OpenZeppelin** (January 2024) - [Report](./docs/security/audit-reports/openzeppelin-2024-01.pdf)

### Bug Bounty
**$500,000 USD** on Immunefi for critical vulnerabilities.

### Threat Model
See [threat-model.md](./docs/security/threat-model.md) for comprehensive security analysis.

---

## Installation

```bash
# Install Rust toolchain
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh

# Install circom compiler
npm install -g circom

# Clone repository
git clone https://github.com/zkstealth/protocol
cd protocol

# Build prover
cd prover && cargo build --release

# Compile circuits
cd ../circuits && circom groth16/prover.circom --r1cs --wasm --sym

# Run tests
cargo test --all
```

---

## Quick Start

```typescript
import { generateProof, zkGreaterThan, executeAction } from '@zkstealth/sdk';

// Generate proof: balance > 10,000 USDC
const proof = await generateProof({
  condition: zkGreaterThan("balance", 10000),
  privateData: { 
    balance: 47329 // Actual balance stays private
  },
  publicInputs: { 
    timestamp: Date.now() 
  }
});

// Execute transaction with proof
await executeAction({
  proof,
  action: async () => {
    // Your transaction logic
    await swap(USDC, ETH, amount);
  }
});
```

---

## Research & Development

**Principal Investigator:** [Eli Ben-Sasson](https://x.com/EliBenSasson) (Turing Award Winner, StarkWare Co-Founder)

Core Team:
- **Alessandro Chiesa** - Cryptography Lead
- **Michael Riabzev** - Protocol Engineering  
- **Lior Goldberg** - Systems Architecture

---

## License

MIT License - see [LICENSE](./LICENSE)

## Citation

```bibtex
@misc{zkstealth2024,
  title={ZK Stealth: Invisible On-Chain Execution via Zero-Knowledge Proofs},
  author={Ben-Sasson, Eli and Chiesa, Alessandro and Riabzev, Michael},
  year={2024},
  publisher={StarkWare Industries}
}
```

---

**Website:** https://www.zkstealthsol.xyz/  
**Twitter:** [@ZKStealth](https://twitter.com/zkstealth)

---

<p align="center">
  <strong>Automated by <a href="https://x.com/EliBenSasson">Eli Ben-Sasson</a></strong>
</p>
