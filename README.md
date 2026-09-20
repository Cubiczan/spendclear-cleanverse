# SpendClear

> **Verified agent payments. Clean money, clean identity, on-chain audit trail.**

SpendClear is an on-chain payment governor for autonomous agents, built on **Monad testnet** and integrated with **Cleanverse Verified Identity (CVI)** and **Cleanverse Verified Assets (CVA)**. Every agent payment routes through CVA-verified stablecoins to CVI-verified recipients, with programmable spend controls and an immutable audit trail.

## The Problem

Autonomous agents can initiate payments — but today there is no on-chain mechanism to ensure:
- The **stablecoin being spent** has clean origination (no mixed/laundered funds)
- The **recipient is a verified identity** (not a freshly created sybil wallet)
- **Spend limits** are enforced per agent mandate
- Every decision has an **immutable audit trail** for compliance reporting

## The Solution

SpendClear sits between an agent and a payment. Before any transfer executes, it enforces four checks:

1. **CVI Identity Check** — the recipient wallet must hold a valid CVI identity token
2. **CVA Asset Check** — the stablecoin must be a CVA-verified asset (clean origination, full traceability)
3. **Mandate Spend Limit** — the amount must be within the agent's approved mandate cap
4. **Audit Log** — every attempt (approved or denied) is recorded on-chain with full context

## CVI · CVA Integration Points

| Cleanverse Capability | How SpendClear Uses It |
|---|---|
| **CVI (Verified Identity)** | `require(cvi.hasIdentity(recipient))` — gates payments to verified wallets only |
| **CVA (Verified Assets)** | `require(cva.isVerified(token))` — only CVA-clean stablecoins can be transferred |
| **Agent Skill Framework** | SpendClear implements the mandate-execution pattern: principal verification, counterparty validation, spend controls |
| **CCP Protocol** | Audit log events are structured for Travel Rule data extraction and compliance reporting |

## Deployed Contracts (Monad Testnet)

| Contract | Address | Explorer |
|---|---|---|
| SpendClear | [`0xe97B6F...20aB`](https://testnet.monadvision.com/address/0xe97B6FF1864EC55fe7DA49BEc72786e00DA620aB) | [View](https://testnet.monadvision.com/address/0xe97B6FF1864EC55fe7DA49BEc72786e00DA620aB) |
| MockCVI | [`0x321798...8283`](https://testnet.monadvision.com/address/0x321798e7A122c9f981059Be3c6b4679ad7668283) | [View](https://testnet.monadvision.com/address/0x321798e7A122c9f981059Be3c6b4679ad7668283) |
| MockCVA | [`0xc9D40e...5484`](https://testnet.monadvision.com/address/0xc9D40ef967d0BAD2444E96BAa784f60caa4D5484) | [View](https://testnet.monadvision.com/address/0xc9D40ef967d0BAD2444E96BAa784f60caa4D5484) |
| MockUSDC | [`0xe4Eb14...bDF9`](https://testnet.monadvision.com/address/0xe4Eb146e84d77510B9f1345a217892CA88F5bDF9) | [View](https://testnet.monadvision.com/address/0xe4Eb146e84d77510B9f1345a217892CA88F5bDF9) |

## Deployed Chain

- **Monad Testnet** (Chain ID: 10143)
  - RPC: `https://testnet-rpc.monad.xyz`
  - Explorer: `https://testnet.monadvision.com`

## Smart Contracts

| Contract | Purpose |
|---|---|
| `SpendClear` | Core governor: CVI/CVA checks, spend limits, audit trail |
| `ICVI` | Interface for Cleanverse Verified Identity tokens |
| `ICVA` | Interface for Cleanverse Verified Assets registry |
| `MockCVI` | Test implementation for demo |
| `MockCVA` | Test implementation for demo |

## Architecture

```
  Agent                              SpendClear                          Recipient
  ─────                              ──────────                          ─────────
  initiatePayment()  ──────▶  ┌──────────────────────┐
  (token, amount,              │ 1. CVI identity check │
   recipient,                  │ 2. CVA asset check    │
   mandateId)                 │ 3. Spend limit check  │
                               │ 4. Transfer execution │
                               │ 5. Audit log emit     │
                               └──────────┬───────────┘
                                          │
                               APPROVED ──▶  CVA-verified stablecoin
                               DENIED   ──▶  PaymentReverted event
```

## Prior Work Disclosure

SpendClear is new, built during the Cleanverse Build hackathon (Aug 8-9, 2026).

It draws on architectural patterns from [MetaboSpend](https://github.com/icohangar-ops/metabospend), an agent spend governor (78 tests, MIT, built Jul 30 – Aug 2, 2026). No code is shared — only the conceptual model of deny-first gates upstream of payment execution.

## Propagation notes (wave B)

- **Row 6 (dual-authority governor) — reversed.** The on-chain side of the
  pair exists — `contracts/SpendClear.sol` carries the mandate authority
  (`createMandate`, `executePayment`, caps, deny-first `_deny` audit) — but
  the repository contains no off-chain payment-execution client to gate: the
  web app is a mock-wallet demo (`web/app/page.tsx` `DEMO_WALLETS`, no web3
  dependency), and `executePayment` has no in-repository caller. A governor
  here would be a decision function with no caller. Reopens when a real
  execution client lands that submits `executePayment` through a pre-check
  the same audit trail records.
- **Row 7 (Sentinel-style circuit breaker) — reversed.** The reversal
  condition fires: Sentinel needs a dense, rolling action stream with
  correlated-failure structure, while SpendClear's action surface is one
  mandate-gated payment per call — too sparse for autocorrelation-based
  halting. Reopens if the desk gains a high-frequency autonomous action
  stream.
- **Row 8 (attestation-threshold minting) — reversed.** The row's own
  condition (a minting/issuance function to gate with 3-of-N verifier
  attestation) does not hold: `contracts/SpendClear.sol` has no mint path —
  it pays out existing tokens against mandates. There is nothing to
  threshold-gate. Reopens if a reserve-backed mint function is added to the
  contract.

## License

MIT — see [LICENSE](LICENSE). Copyright (c) 2026 Shyam Desigan.
