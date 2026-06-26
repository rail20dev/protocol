# FAQ

## What is RAIL20?

A zero-knowledge privacy layer for ERC-20 tokens on Base. Shield tokens into a private pool, transact without exposing balances or counterparties, and unshield to any public address.

## How is privacy achieved?

Every transaction is gated by a PLONK zero-knowledge proof generated locally in your browser. The on-chain record contains only commitments and nullifiers — no amounts, no addresses, no balances. Observers can't link who sent what to whom.

## Is this a mixer?

No. RAIL20 uses a per-token commitment set with selective-disclosure view keys and Private Proofs of Innocence. Unlike mixers, each note retains a provable history that the owner can selectively disclose. See [compliance.md](compliance.md).

## What happens if I lose my note secret?

Your shielded funds become unrecoverable — that's the nature of self-custodied privacy. RAIL20 persists your notes locally and offers an optional encrypted cloud backup (the encryption key is derived from your wallet signature, so only you can decrypt it). As long as you keep at least one device or backup synced, a device swap won't lose your notes.

**Always keep at least one synced backup.**

## Do I need ETH for gas?

Not necessarily. You can submit through a broadcaster — a relayer that pays the ETH gas and charges a small markup in the token you're transacting. This also breaks the link between your wallet and the transaction.

## What tokens are supported?

Currently USDC and WETH on Base. Additional ERC-20s can be enabled by governance.

## Which networks?

Base mainnet (chain ID 8453).

## Is the code audited?

RAIL20 builds on audited primitives — PLONK verification, Poseidon hashing, and the Merkle commitment design from Railgun's audited stack. A dedicated external audit of the RAIL20 wrapper contracts is pending.

## How do I integrate RAIL20?

See [integration.md](integration.md), or reach out via [@railB20](https://x.com/railB20).

## Where are the contracts?

Deployed and verified on Base. See [contracts.md](contracts.md) for addresses and Basescan links.
