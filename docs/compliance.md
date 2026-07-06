# Compliance

RAIL20's design follows a single principle: **privacy by default, transparency on demand.**

Privacy tools should protect honest users without becoming a haven for illicit funds. RAIL20 pairs a shielded pool with a selective-disclosure model: users hold viewing keys that let them prove - to a counterparty, auditor, or regulator - exactly what they choose, including non-association with sanctioned addresses, without deanonymizing the rest of their activity.

## Private Proofs of Innocence (POI)

Users can cryptographically prove that their shielded funds do **not** trace back to a known list of sanctioned or flagged addresses - without revealing their full transaction history.

This lets honest users demonstrate clean provenance while keeping their financial activity private. Funds that cannot produce a valid POI are isolated from the compliant set.

## Selective Disclosure

Each user holds **view keys** that can reveal specific transactions to an authorized party - an auditor, a counterparty, a regulator - on a per-transaction, opt-in basis.

- No master key exists that can decrypt all activity
- Disclosure is initiated by the user, not imposed globally
- An auditor sees only what the user chooses to share

## Why This Isn't a Mixer

| Mixer | RAIL20 |
|---|---|
| Fixed denominations | Arbitrary amounts |
| Pooled fungible liquidity | Per-token commitment set |
| No disclosure mechanism | View keys + POI |
| No provenance proofs | Private Proofs of Innocence |

The shielded set is per-token and commitment-based, not a fungible liquidity pool. There is no "anonymity set laundering" argument because each note retains a provable, selectively-disclosable history.

## Front-End Screening

The official RAIL20 front-end can screen addresses against sanctions lists before allowing a shield operation. Integrators are encouraged to apply the same screening at their own entry points.

## Summary

RAIL20 gives users real privacy for legitimate financial activity - treasury management, payroll, agent-to-agent payments, RWA settlement - while preserving the tools regulators and auditors need to verify compliance when legally required.
