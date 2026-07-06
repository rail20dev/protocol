# Smart Contracts

Deployed on **Base mainnet** (chain ID 8453). Each shielded pool is its own contract, verifiable on Basescan.

## Addresses

| Contract | Address | Purpose |
|---|---|---|
| ETH Pool | [`0x99205B045Fcf5ff2689ad6B038abF3dd35b79ddE`](https://basescan.org/address/0x99205B045Fcf5ff2689ad6B038abF3dd35b79ddE) | Shielded pool for native ETH - Poseidon Merkle tree, proof verification, note custody |
| USDC Pool | [`0x764FF96EabEeF2b197f845dD8D39441Ac68FDE11`](https://basescan.org/address/0x764FF96EabEeF2b197f845dD8D39441Ac68FDE11) | Shielded pool for USDC ([`0x8335…2913`](https://basescan.org/address/0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913)) |
| Fee recipient | [`0xd962c7a67B74EEe8D418189f7917A18577A4775d`](https://basescan.org/address/0xd962c7a67B74EEe8D418189f7917A18577A4775d) | Collects protocol fees on withdrawals |

RAIL20 runs **one dedicated pool contract per token** rather than a single shared vault. Each pool holds only its own asset and maintains its own commitment tree, which keeps the anonymity set per-asset and the contract surface minimal.

## Pool Interface (key functions)

Each pool exposes a UTXO-style shielded-transaction interface. A single transaction spends input notes and creates up to **two** output notes (recipient + change).

```solidity
// Shield tokens into the pool -> create private note(s)
function deposit(uint256[] calldata inputNullifiers, Commitment[] calldata outputs, Proof calldata proof) external payable;

// Shielded transaction: spend note(s), create up to 2 outputs, optionally withdraw to a public address
function transact(
    Proof calldata proof,
    uint256 root,
    uint256[] calldata nullifiers,   // spent notes
    Commitment[2] calldata outputs,  // outputs[0] = recipient, outputs[1] = change
    uint256 publicAmount,            // net amount leaving the pool (0 for internal transfer)
    address recipient,               // public recipient when withdrawing
    uint256 fee,
    address feeRecipient
) external;
```

On-chain, a transaction reveals only **commitments** (new notes) and **nullifiers** (spent notes). Amounts, senders, and recipients are never exposed. The circuit enforces a maximum of two outputs per transaction.

## Supported Tokens

Each supported token has its own pool. Currently live:

- **ETH** (native)
- **USDC** (Base)

## Relayer & Fees

Withdrawals and shielded transactions are broadcast by a **relayer** so users pay no gas and their own wallet never appears as the transaction sender. The fee is a flat component plus **0.35%** of the amount, paid to the fee recipient. Flat fee: `0.00025 ETH` (ETH pool) or `1 USDC` (USDC pool).

## Verifying Contracts

All pool source is verified on Basescan. To independently verify:

1. Visit the Basescan contract page for a pool address above.
2. Click **Contract → Read Contract** to inspect state (Merkle root, nullifier status).
3. Click **Contract → Code** to view the verified Solidity source.

The proof verifier is generated from a standard universal setup and can be reproduced from the public ceremony output.
