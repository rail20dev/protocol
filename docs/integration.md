# Integration Guide

How to integrate RAIL20 privacy primitives into your protocol, agent, or dApp.

## Overview

RAIL20 exposes three integration surfaces:

1. **Direct contract calls** — shield/send/unshield via the Vault ABI
2. **Broadcaster API** — submit proofs for relay (user doesn't pay gas directly)
3. **SDK (coming soon)** — high-level TypeScript wrapper for proof generation + submission

## Integration Pattern: Agent Treasury

For autonomous agents that need private token management:

```
1. Agent receives funds to its public EOA
2. Agent calls vault.shield() → tokens enter the privacy pool
3. Agent generates ZK proofs locally for internal transfers
4. Agent calls vault.unshield() to a fresh address when spending publicly
```

Each step breaks the on-chain link. An observer sees deposits into the pool and withdrawals from the pool — but cannot connect the two.

## Integration Pattern: Private Swap

For protocols that want to offer private trading:

```
1. User shields tokens into RAIL20 Vault
2. User requests a private swap (token A → token B)
3. PrivateSwapRouter atomically: unshield → DEX swap → re-shield
4. User receives shielded token B — no public trace of the trade
```

## Key Integration Considerations

| Consideration | Detail |
|---|---|
| Proof generation | Client-side (WASM). ~2-4s on modern hardware. |
| Broadcaster relay | Users don't need ETH for gas if using a broadcaster. Broadcaster charges a markup in the transacted token. |
| Finality | Base L2 finality (~2s block time). Proofs are verified on-chain in the same block. |
| Nullifier freshness | Check `vault.isSpent(nullifierHash)` before submitting to avoid wasted gas. |
| Token support | Only registered tokens can be shielded. Query `vault.registeredTokens(address)`. |

## Selective Disclosure

Integrators can request proof of non-association (compliance):

- Users hold view keys that reveal specific transactions to authorized parties
- Private Proofs of Innocence (POI) demonstrate funds don't trace to sanctioned addresses
- No bulk surveillance — disclosure is per-user, per-request, opt-in

## Contact

For integration support: [@railB20](https://x.com/railB20) on Twitter.
