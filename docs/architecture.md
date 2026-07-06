# Architecture

Technical overview of RAIL20's components, zero-knowledge circuit, and data flow.

## System Overview

RAIL20 is built from small, single-purpose contracts rather than one monolithic vault. There is **one shielded pool contract per token**.

| Component | Role |
|---|---|
| **Token Pool** (one per asset) | Core contract for a single asset. Holds shielded balances, maintains a Poseidon Merkle tree of note commitments, verifies zk proofs, tracks spent nullifiers. |
| **Proof Verifier** | On-chain zk-SNARK verifier (BN254). Validates proofs submitted to a pool. |
| **Relayer** | Off-chain service that submits users' proofs on their behalf, so a user's own wallet is never linked to shielded activity and they pay no gas. |
| **Solver network** (external) | Cross-chain intent settlement (NEAR Intents / 1Click) used by in-app swap and bridge. Not a RAIL20 contract. |

## Data Flow

```
User (browser)
  │
  ├── 1. Generate zk proof locally (WASM prover)
  ▼
Relayer
  │
  ├── 2. Validate proof structure + nullifier freshness
  ├── 3. Submit tx to Base (relayer pays gas)
  ▼
Token Pool (on-chain)
  │
  ├── 4. Verify zk proof via the Verifier
  ├── 5. Check nullifiers not already spent
  ├── 6. Mark nullifiers as spent
  ├── 7. Insert new commitment(s) into the Merkle tree
  └── 8. Move tokens (shield / unshield)
```

## Shielded Transaction Model

RAIL20 uses a **UTXO note model**. Each private balance is a set of notes (commitments). A transaction spends one or more input notes and creates up to **two** output notes:

- `outputs[0]` — recipient note
- `outputs[1]` — change note back to the sender

This two-output structure covers private send (both outputs internal), withdrawal (net amount leaves the pool to a public recipient), and re-shielding after a swap. On-chain, only commitments and nullifiers appear — never amounts or addresses.

## ZK Circuit Design

| Parameter | Value |
|---|---|
| Proof system | zk-SNARK over a universal setup |
| Hash function | Poseidon |
| Curve | BN254 |
| Outputs per tx | 2 (recipient + change) |
| Tree | Poseidon Merkle tree, append-only |

## Merkle Tree & Nullifiers

An append-only Poseidon Merkle tree stores note commitments; a nullifier set marks spent notes without revealing which note was consumed.

```
commitment    = Poseidon(pubkey, amount, blinding)
nullifier     = Poseidon(commitment, merkleIndex, spendSignature)
```

## Swap & Bridge

Swap and bridge are handled in the app, not by an on-chain swap router:

- **Swap to private** — a private withdrawal funds a one-time **burner wallet**, which swaps through the cross-chain solver network and re-shields the output back into the pool as fresh private notes. A neutral burner sits between the pool and the swap, so there is no on-chain link between the user's wallet and the swapped funds.
- **Bridge** — a private withdrawal is routed to the solver's deposit address; the solver fulfills the intent cross-chain and delivers the output directly to the user's chosen recipient on the destination chain (Base, Arbitrum, BNB Chain).

## Security Model

- **zk soundness** — BN254, discrete-log hard. No valid proof without the private inputs.
- **Poseidon collision resistance** — no known collisions in the field.
- **Append-only nullifier set** — prevents double-spend without revealing the consumed note.
- **Non-custodial relayer** — the relayer never holds user funds or keys; it only broadcasts pre-proven transactions.
- **Client-side proof generation** — secrets never leave the user's browser. The burner-wallet key used for swaps is derived from the user's signature and also stays in the browser.
