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
  ├── 1. Sign the RAIL20 message (derives account keys locally)
  ├── 2. POST { signature, pool, amount, recipient } to the relayer
  ▼
Relayer (api.rail20.org)
  │
  ├── 3. Derive keys from signature, scan pool for the user's notes
  ├── 4. Build the zk-SNARK proof (2 inputs, 2 outputs)
  ├── 5. Submit tx to the pool's chain (Base, Robinhood, or Arbitrum; relayer pays gas)
  ▼
Token Pool (on-chain)
  │
  ├── 6. Verify zk proof via the Verifier
  ├── 7. Check nullifiers not already spent
  ├── 8. Mark nullifiers as spent
  ├── 9. Insert new commitment(s) into the Merkle tree
  └── 10. Move tokens (shield / unshield)
```

## Shielded Transaction Model

RAIL20 uses a **UTXO note model**. Each private balance is a set of notes (commitments). A transaction spends one or more input notes and creates up to **two** output notes:

- `outputs[0]` - recipient note
- `outputs[1]` - change note back to the sender

This two-output structure covers private send (both outputs internal), withdrawal (net amount leaves the pool to a public recipient), and re-shielding after a swap. On-chain, only commitments and nullifiers appear - never amounts or addresses.

## ZK Circuit Design

| Parameter | Value |
|---|---|
| Proof system | Groth16 (zk-SNARK) |
| Inputs per tx | 2 |
| Outputs per tx | 2 (recipient + change) |
| Hash function | Poseidon |
| Curve | BN254 |
| Merkle tree depth | 26 levels |
| Note encryption | AES-256-GCM (key derived from wallet signature) |
| Tree | Poseidon Merkle tree, append-only |

## Merkle Tree & Nullifiers

An append-only Poseidon Merkle tree stores note commitments; a nullifier set marks spent notes without revealing which note was consumed.

```
commitment    = Poseidon(amount, pubkey, blinding, mintAddress)
nullifier     = Poseidon(commitment, merklePath, signature)  // signature = Poseidon(privKey, commitment, merklePath)
```

## Swap & Bridge

Swap and bridge are handled in the app:

- **Same-chain swap to private** - a private withdrawal funds a one-time **burner wallet**, which swaps **on-chain through Uniswap V3** (ETH ↔ USDC on Base, ETH ↔ USDG on Robinhood, ETH ↔ USDC on Arbitrum) in a single transaction, then re-shields the output back into the pool as fresh private notes. A neutral burner sits between the pool and the swap, so there is no on-chain link between the user's wallet and the swapped funds. (Earlier versions routed same-chain swaps through the cross-chain solver, which stalled for minutes; moving to an on-chain DEX cut this to ~30s.)
- **Cross-chain bridge** - a private withdrawal is routed to a solver/router deposit address (NEAR Intents 1Click, or a direct Relay/Across router between pool chains); it fulfills the intent cross-chain and delivers the output directly to the user's chosen recipient on the destination chain (between pool chains Base, Robinhood, and Arbitrum, plus out to Ethereum and BNB Chain).

## Security Model

- **zk soundness** - BN254, discrete-log hard. No valid proof without the private inputs.
- **Poseidon collision resistance** - no known collisions in the field.
- **Append-only nullifier set** - prevents double-spend without revealing the consumed note.
- **Non-custodial relayer** - the relayer never holds user funds or keys; it only broadcasts pre-proven transactions.
- **Relayer-side proving** - the relayer builds proofs from the user's signature, so it can see the note amounts and recipients it relays; it cannot move funds without the signature, and on-chain observers still learn nothing. Self-hosting a relayer removes this trust.
