# Smart Contracts

Deployed on **Base** (chain ID 8453) and **Robinhood Chain** (chain ID 4663, an Arbitrum Orbit L2). Each shielded pool is its own contract per token per chain.

## Addresses

### Base (chain 8453)

| Contract | Address | Purpose |
|---|---|---|
| ETH Pool | [`0x99205B045Fcf5ff2689ad6B038abF3dd35b79ddE`](https://basescan.org/address/0x99205B045Fcf5ff2689ad6B038abF3dd35b79ddE) | Shielded pool for native ETH - Poseidon Merkle tree, proof verification, note custody |
| USDC Pool | [`0x764FF96EabEeF2b197f845dD8D39441Ac68FDE11`](https://basescan.org/address/0x764FF96EabEeF2b197f845dD8D39441Ac68FDE11) | Shielded pool for USDC ([`0x8335…2913`](https://basescan.org/address/0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913)) |

### Robinhood Chain (chain 4663)

| Contract | Address | Purpose |
|---|---|---|
| ETH Pool | `0xBf6a26CEF8f6251B46Cc09D03A0f12Df5972d163` | Shielded pool for native ETH on Robinhood |
| USDG Pool | `0x04F8d8E4C3f23fE2BfB80c6C46b7d7dfE6f853F0` | Shielded pool for USDG — Global Dollar (`0x5fc5360D0400a0Fd4f2af552ADD042D716F1d168`) |

### Shared

| Contract | Address | Purpose |
|---|---|---|
| Fee recipient | [`0xd962c7a67B74EEe8D418189f7917A18577A4775d`](https://basescan.org/address/0xd962c7a67B74EEe8D418189f7917A18577A4775d) | Collects protocol fees on withdrawals (both chains) |
| Relayer | `0x691685db9D8dB916f305Dc9fB01D23FEfA8101dA` | Broadcasts proofs on both chains |

RAIL20 runs **one dedicated pool contract per token** rather than a single shared vault. Each pool holds only its own asset and maintains its own commitment tree, which keeps the anonymity set per-asset and the contract surface minimal.

## Staking & Governance (Robinhood Chain 4663)

The RAIL20 staking and governance system is a fork of the audited [Railgun governance contracts](https://github.com/Railgun-Privacy/contract) (identifiers renamed only; logic unchanged). Stakers lock $RAIL20 to earn a share of protocol fees paid in ETH and to gain voting power over on-chain governance.

| Contract | Address | Purpose |
|---|---|---|
| $RAIL20 token | [`0xda917699d4C66941bE575fb53044E5df71bbe460`](https://robinhoodchain.blockscout.com/address/0xda917699d4C66941bE575fb53044E5df71bbe460) | Governance and staking token |
| Staking | [`0x3149A3876DabFb98979818f81Cd17c6cdB3d0f67`](https://robinhoodchain.blockscout.com/address/0x3149A3876DabFb98979818f81Cd17c6cdB3d0f67) | Per-position stake with 30-day unlock cooldown, daily snapshots |
| Voting | [`0xcE0dA52e0879F8Aa8347E2B800E638BF97d5e97d`](https://robinhoodchain.blockscout.com/address/0xcE0dA52e0879F8Aa8347E2B800E638BF97d5e97d) | Proposal lifecycle: create, sponsor, callVote, vote, execute |
| Delegator | [`0x74C47d01D283A50a4349B20e8562a91505929b1A`](https://robinhoodchain.blockscout.com/address/0x74C47d01D283A50a4349B20e8562a91505929b1A) | Permissioned executor; owner is Voting |
| Treasury (proxy) | [`0x16171E1c5E9a4461E4Ff4E16EC2AdE1cE60D9E74`](https://robinhoodchain.blockscout.com/address/0x16171E1c5E9a4461E4Ff4E16EC2AdE1cE60D9E74) | Holds protocol fees; admin is Delegator |
| GovernorRewards (proxy) | [`0x8623e9a582A42B069005b7719B3eFdF7B65D0c61`](https://robinhoodchain.blockscout.com/address/0x8623e9a582A42B069005b7719B3eFdF7B65D0c61) | Distributes treasury funds to stakers by voting power |
| ProxyAdmin | [`0xC612F631398551e2A35dF949E3CA137b7cD986e7`](https://robinhoodchain.blockscout.com/address/0xC612F631398551e2A35dF949E3CA137b7cD986e7) | Upgrade admin for Treasury and GovernorRewards; owner is Delegator |
| Reward token (WETH) | [`0x0Bd7D308f8E1639FAb988df18A8011f41EAcAD73`](https://robinhoodchain.blockscout.com/address/0x0Bd7D308f8E1639FAb988df18A8011f41EAcAD73) | Rewards are distributed in WETH |

**Governance parameters** (identical to upstream): sponsorship threshold `500,000` $RAIL20 within a 30-day window, voting quorum `2,000,000`, 30-day unlock cooldown, snapshot interval 1 day, reward distribution interval 14 days.

**Ownership chain:** Treasury, ProxyAdmin, and GovernorRewards are all owned or admin-controlled by Delegator; Delegator is owned by Voting. All privileged actions execute only through passed governance proposals.

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

Each supported token has its own pool per chain. Currently live:

- **Base:** ETH (native), USDC
- **Robinhood Chain:** ETH (native), USDG (Global Dollar)

## Relayer & Fees

Withdrawals and shielded transactions are broadcast by a **relayer** so users pay no gas and their own wallet never appears as the transaction sender. The fee is a flat component plus **0.35%** of the amount, paid to the fee recipient. Flat fee per pool: `0.00025 ETH` (Base ETH), `1 USDC` (Base USDC), `0.00002 ETH` (Robinhood ETH), `0.5 USDG` (Robinhood USDG). Deposits (shielding) pay no protocol fee.

## Verifying Contracts

All pool source is verified on Basescan. To independently verify:

1. Visit the Basescan contract page for a pool address above.
2. Click **Contract → Read Contract** to inspect state (Merkle root, nullifier status).
3. Click **Contract → Code** to view the verified Solidity source.

The proof verifier is generated from a standard universal setup and can be reproduced from the public ceremony output.
