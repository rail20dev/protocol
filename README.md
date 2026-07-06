# RAIL20

**Zero-knowledge private payments on Base.**

RAIL20 is a zero-knowledge privacy layer for ERC-20 tokens on Base. Users shield tokens into a shared anonymity pool, transact privately using PLONK zero-knowledge proofs, and unshield back to a public address whenever they want.

- **Live on Base mainnet.** Contracts deployed and operational.
- **Privacy by default, transparency on demand.** Users keep selective-disclosure keys and can prove non-association with sanctioned addresses.
- **Built on audited primitives.** PLONK / BN254 / Poseidon — all standard, no novel cryptography.

## Quick Links

| Resource | Where |
|---|---|
| App | https://app.rail20.org |
| Docs site | https://docs.rail20.org |
| Twitter | https://x.com/railB20 |
| Verified contracts | [Basescan](https://basescan.org/address/0x1a9635E123a3fc7180701505630435BC83D1197B) |

## What's in this Repo

Public technical reference and integration documentation. The implementation source — contracts, circuits, backend, frontend — is maintained privately while the protocol stabilizes.

```
docs/
├── architecture.md     System design, ZK circuit, data flow
├── contracts.md        Deployed addresses, interfaces, verification
├── integration.md      How to integrate RAIL20 from another protocol
├── compliance.md       Compliance posture and selective-disclosure model
└── faq.md              Common questions
```

## How RAIL20 Works (60-second version)

1. **Shield** — deposit an ERC-20 into the vault. Receive a private note only you can spend.
2. **Send privately** — transfer notes between shielded addresses with sender, recipient, and amount hidden.
3. **Swap privately** — trade between tokens inside the shielded set via aggregated routing.
4. **Unshield** — withdraw to any public address. Use a fresh address to break linkability.

Every step is gated by a PLONK proof that the user generates locally. Nothing leaves the browser unencrypted, and the on-chain record reveals only commitments and nullifiers — no balances, no recipients, no amounts.

## Status

- Mainnet: live
- Audits: external review pending; RAIL20 is built on standard audited primitives for ZK verification (PLONK), Poseidon hashing, and the Merkle commitment tree over BN254
- Bug bounty: contact via Twitter DM

## License

Documentation in this repo is released under [MIT](LICENSE).
