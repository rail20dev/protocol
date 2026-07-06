# Architecture

Technical overview of RAIL20's components, ZK circuit, and data flow.

## System Overview

| Component | Role |
|---|---|
| **RAIL20Vault** | Core contract. Holds shielded tokens, manages Poseidon Merkle tree, verifies ZK proofs, tracks spent nullifiers. |
| **PlonkVerifier** | On-chain PLONK verifier (BN254). Validates ZK proofs submitted to the Vault. |
| **PrivateSwapRouter** | Aggregated private swaps: unshield → swap → re-shield in one atomic tx. |
| **UsernameRegistry** | Maps `.rail20` names to shielded addresses for human-readable recipients. |
| **Broadcaster** | Off-chain relayer network. Submits proofs on behalf of users so their EOA is never linked to shielded activity. |

## Data Flow

```
User (browser)
  │
  ├── 1. Generate ZK proof locally (WASM PLONK prover)
  ▼
Broadcaster (relayer)
  │
  ├── 2. Validate proof structure + nullifier freshness
  ├── 3. Submit tx to Base (broadcaster pays gas)
  ▼
RAIL20Vault (on-chain)
  │
  ├── 4. Verify PLONK proof via PlonkVerifier
  ├── 5. Check nullifier not already spent
  ├── 6. Mark nullifier as spent
  ├── 7. Insert new commitment(s) into Merkle tree
  └── 8. Transfer tokens (shield/unshield/swap)
```

## ZK Circuit Design

Single unified circuit compiled with circom 2.x, verified via PLONK (universal trusted setup).

| Parameter | Value |
|---|---|
| Proof system | PLONK (universal setup) |
| Trusted setup | Hermez PTAU (2^14, 54 contributors) |
| Constraints | 5,073 non-linear + 5,409 linear |
| Public inputs | 6 (root, nullifierHash, commitments, txHash, mode) |
| Merkle depth | 16 (65,536 max commitments) |
| Hash function | Poseidon |
| Curve | BN254 |
| Proof size | ~450 bytes (24 uint256) |
| On-chain verify gas | ~250,000 |

### Why PLONK over Groth16?

PLONK uses a universal trusted setup — no Phase 2 ceremony, no new toxic waste when the circuit changes. Tradeoff: ~450 byte proof (vs 96) and ~20K more gas. Negligible on Base L2.

## Merkle Tree

16-level Poseidon append-only tree. Spent notes are tracked via a nullifier mapping — the tree itself never shrinks.

```
commitment = Poseidon(nullifier, secret, amount)
nullifierHash = Poseidon(nullifier)
pathElements[16]  // sibling hashes for Merkle proof
pathIndices[16]   // 0=left, 1=right at each level
```

## Security Model

- **PLONK soundness** — BN254 curve, discrete-log hard. No forged proofs without private inputs.
- **Poseidon collision resistance** — no known collisions in the field.
- **Append-only nullifier set** — prevents double-spend without revealing which note was consumed.
- **Broadcaster non-custodial** — relayer never holds user funds or keys; it only submits pre-signed proofs.
- **Client-side proof generation** — secrets (nullifier, secret scalar) never leave the user's browser.
