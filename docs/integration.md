# Integration Guide

How to integrate RAIL20 privacy primitives into your protocol, agent, or dApp.

## Overview

RAIL20 exposes these integration surfaces:

1. **CLI** (`@rail20/cli`, live on npm) - the fastest path. `npm i -g @rail20/cli`, then `rail20 deposit / send / swap / bridge / recover / balance`, with `--chain base|robinhood|arbitrum`. Ideal for agents and scripts.
2. **Relayer API** - submit proofs for relay so the user doesn't pay gas directly (`https://api.rail20.org`). Use this to embed RAIL20 into an existing runtime.
3. **Direct contract calls** - shield / transact / unshield against a token pool contract.
4. **SDK (coming soon)** - high-level TypeScript wrapper for proof generation + submission.

Each supported asset has its own pool contract (see [Contracts](contracts.md)).

## Integration Pattern: Agent Treasury

For autonomous agents that need private token management:

```
1. Agent receives funds to its public EOA
2. Agent deposits into the token pool → funds enter the privacy pool as notes
3. Agent signs; the relayer builds zk proofs and broadcasts internal transfers
4. Agent withdraws to a fresh address when spending publicly
```

Each step breaks the on-chain link. An observer sees deposits into the pool and withdrawals from the pool - but cannot connect the two.

## Integration Pattern: Private Swap & Bridge

For protocols that want to offer private trading or cross-chain movement:

```
Same-chain swap (stays private):
1. User holds a private balance in a pool
2. A private withdrawal funds a one-time burner wallet
3. The burner swaps on-chain via Uniswap V3 (one transaction, no solver wait)
4. The output is re-shielded back into the pool as private notes

Bridge (private → any chain):
1. A private withdrawal funds a burner, which routes to a solver/router deposit address
   (NEAR Intents 1Click for Arbitrum/Ethereum/BNB, or a direct Relay/Across router for Base ↔ Robinhood)
2. The solver/router fulfills the intent cross-chain
3. The output asset is delivered to the recipient on the destination chain
```

A neutral burner and an external solver/router network sit between the pool and the swap, so the trade has no public trace back to the user.

## Key Integration Considerations

| Consideration | Detail |
|---|---|
| Proof generation | Relayer-side from the user's signature (a few seconds). Self-hosting the relayer keeps proving fully in your control. |
| Relayer | Users don't need ETH for gas if using the relayer. Fee is a flat amount plus 0.35% in the transacted token. |
| Finality | Base L2 finality (~2s block time). Proofs verify on-chain in the same block. |
| Outputs per tx | Maximum of two (recipient + change). |
| Cross-chain settle | Bridge/swap settlement takes ~1-6 min; poll intent status until confirmed. |
| Token support | Each supported token has its own pool per chain. Live: ETH + USDC on Base, ETH + USDG on Robinhood Chain, ETH + USDC on Arbitrum. |

## Selective Disclosure

Integrators can request proof of non-association (compliance):

- Users hold view keys that reveal specific transactions to authorized parties
- Private Proofs of Innocence (POI) demonstrate funds don't trace to sanctioned addresses
- No bulk surveillance - disclosure is per-user, per-request, opt-in

## Contact

For integration support: [@railB20](https://x.com/railB20) on Twitter.
