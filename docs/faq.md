# FAQ

## What is RAIL20?

A zero-knowledge privacy layer for tokens on Base, Robinhood Chain, and Arbitrum. Shield tokens into a private pool, transact without exposing balances or counterparties, and unshield to any public address.

## How is privacy achieved?

Every transaction is gated by a Groth16 zero-knowledge proof built by the relayer from your signature. The on-chain record contains only commitments and nullifiers - no amounts, no addresses, no balances. Observers can't link who sent what to whom.

## Is this a mixer?

No. RAIL20 uses a per-token commitment set with selective-disclosure view keys and Private Proofs of Innocence. Unlike mixers, each note retains a provable history that the owner can selectively disclose. See [compliance.md](compliance.md).

## What if I switch devices — do I lose my notes?

No. Your private balance is **reconstructed from on-chain events** using your wallet signature: account and note-encryption keys are derived deterministically from a single signed message, so any device that can produce that signature can rebuild your notes by scanning the pool. Nothing critical is stored only in one browser.

The one thing you must keep is **access to the wallet key that produces the signature** — lose that and, like any self-custodied asset, the funds are unrecoverable. There is no server-side account and no master key.

## Do I need ETH for gas?

Not necessarily. You can submit through the relayer that pays the ETH gas and charges a small markup in the token you're transacting. This also breaks the link between your wallet and the transaction.

## What tokens are supported?

On Base: native **ETH** and **USDC**. On Robinhood Chain: native **ETH** and **USDG** (Global Dollar). On Arbitrum: native **ETH** and **USDC** (native Circle). Each token has its own pool; additional tokens can be enabled by governance.

## Which networks?

Base (chain ID 8453), Robinhood Chain (chain ID 4663, an Arbitrum Orbit L2), and Arbitrum (chain ID 42161).

## Is the code audited?

RAIL20 builds on well-established, audited cryptographic primitives - Groth16 proof verification, Poseidon hashing, and a Merkle-tree commitment scheme over the BN254 curve. These are industry-standard building blocks with no novel cryptography. A dedicated external audit of the RAIL20 contracts is pending.

## How do I integrate RAIL20?

See [integration.md](integration.md), or reach out via [@rail20](https://x.com/rail20).

## Where are the contracts?

Deployed on Base, Robinhood Chain, and Arbitrum. See [contracts.md](contracts.md) for all addresses and explorer links.

## Can I stake $RAIL20?

Yes. Staking and governance are live on Robinhood Chain at [staking.rail20.org](https://staking.rail20.org). Stake $RAIL20 to earn a share of protocol fees paid in ETH and to gain voting power over on-chain governance. The staking and governance contracts are a fork of the audited Railgun governance system (identifiers renamed, logic unchanged). See the Staking & Governance section in [contracts.md](contracts.md#staking--governance-robinhood-chain-4663).

## How are staking rewards paid?

In ETH, not in more $RAIL20. Fees collected from private operations (send, swap, bridge) accrue to the protocol treasury and are distributed to stakers by voting-power share. There is no token emission and no burn, so the supply stays fixed and rewards track real protocol usage.

## How does governance work?

Any staker with voting power can submit a proposal. It must gather 500,000 $RAIL20 of sponsorship within 30 days to reach a vote, then pass a community vote (quorum 2,000,000, Yes over No). Passed proposals execute on-chain through the governance delegator. Unlocking a stake starts a 30-day cooldown before it can be claimed, which protects quorum from sudden withdrawals during open votes.

