# RAIL20

**Zero-knowledge private payments on Base.**

RAIL20 is a zero-knowledge privacy layer for tokens on Base. Users shield tokens into a per-token anonymity pool, transact privately using Groth16 zero-knowledge proofs, and unshield back to a public address whenever they want.

RAIL20 pools are ERC-20-compatible. Because [B20](https://docs.base.org/base-chain/specs/upgrades/beryl/b20) (Base's native token standard, part of the Beryl upgrade) is a superset of ERC-20, a B20 pool drops in with no contract change once B20 activates on Base mainnet. Live today: native ETH and USDC.

- **Live on Base mainnet.** Contracts deployed and operational.
- **Privacy by default, transparency on demand.** Users keep selective-disclosure keys and can prove non-association with sanctioned addresses.
- **Built on audited primitives.** Groth16 / BN254 / Poseidon - all standard, no novel cryptography.

## Quick Links

| Resource | Where |
|---|---|
| App | https://app.rail20.org |
| Docs site | https://docs.rail20.org |
| Twitter | https://x.com/railB20 |
| Verified contracts | [Basescan](https://basescan.org/address/0x764FF96EabEeF2b197f845dD8D39441Ac68FDE11) |

## What's in this Repo

Public technical reference and integration documentation. The implementation source - contracts, circuits, backend, frontend - is maintained privately while the protocol stabilizes.

```
docs/
├── architecture.md     System design, ZK circuit, data flow
├── contracts.md        Deployed addresses, interfaces, verification
├── integration.md      How to integrate RAIL20 from another protocol
├── compliance.md       Compliance posture and selective-disclosure model
└── faq.md              Common questions
```

## How RAIL20 Works (60-second version)

1. **Shield** - deposit an ERC-20 into its token pool. Receive a private note only you can spend.
2. **Send privately** - transfer notes between shielded addresses with sender, recipient, and amount hidden.
3. **Swap privately** - swap ETH into private USDC (or vice versa) through a one-time burner wallet and a cross-chain solver network; the output is re-shielded so it stays private.
4. **Bridge privately** - send value from your private balance to a recipient on another chain (Base, Arbitrum, BNB Chain).
5. **Unshield** - withdraw to any public address. Use a fresh address to break linkability.

Every step is gated by a zk proof the relayer builds from the user's signature. Nothing leaves the browser unencrypted, and the on-chain record reveals only commitments and nullifiers - no balances, no recipients, no amounts.

## Status

- Mainnet: live
- Audits: external review pending; RAIL20 is built on standard audited primitives for ZK verification (Groth16), Poseidon hashing, and the Merkle commitment tree over BN254
- Bug bounty: contact via Twitter DM

## License

Documentation in this repo is released under [MIT](LICENSE).
