# OWS-Dash — Dash Provider for the Open Wallet Standard

> A zero-trust, WASM-compatible Dash signer and provider implementing the Open Wallet Standard (OWS) with native InstantSend support for AI agents and multi-chain wallets.

---

## Overview

`ows-dash` brings Dash to the Open Wallet Standard ecosystem. It provides a Rust crat that implements the OWS signer and provider traits for Dash, plus a TypeScript/JavaScript wrapper for use in Node.js and browser environments.

Key capabilities:

- **Zero-Trust Signing** — private keys never leave the local signer; PSBT-based signing flow
- **InstantSend** — automatic IS flag handling with `islock` confirmation polling
- **CAIP-2 Compliant** — registered as `dash:mainnet` and `dash:testnet`
- **BIP-44 / Coin Type 5** — standard HD derivation for Dash
- **DAPI + Insight Fallback** — resilient broadcast via Dash Platform API with automatic fallback
- **AI Agent Ready** — designed for LangChain and autonomous agent frameworks

---

## Architecture

### Signer (Zero-Trust Model)

The signer follows the OWS zero-trust model: keys are decrypted locally and never transmitted. The PSBT flow is:

```
Agent / Wallet
     │
     ▼
[OWS Core] ──PSBT──▶ [ows-dash-signer]
                            │
                    decrypt key locally
                            │
                    sign PSBT inputs
                            │
                      set IS flags
                            │
                     ◀──signed tx──
     │
     ▼
[ows-dash-provider]
     │
  broadcast
  via DAPI
     │
  poll for
  islock event
     │
  return "Final"
```

### InstantSend

The signer automatically sets the correct transaction version and service flags required for InstantSend. After broadcast, the provider subscribes to the `islock` event from DAPI. The `SignAndSendResult` exposes:

- `isLock: boolean` — whether IS Lock has been received
- `pollIsLock(): Promise<boolean>` — manual poll for IS Lock status
- `status: "Pending" | "Broadcast" | "Final"` — overall finality state

### Network Provider

| Method | Primary | Fallback |
|--------|---------|---------|
| Broadcast TX | DAPI (gRPC) | Insight REST API |
| Subscribe islock | DAPI WebSocket | Polling via Insight |
| UTXOs | DAPI | Insight |

---

## CAIP-2 Chain IDs

| Network | CAIP-2 ID |
|---------|-----------|
| Mainnet | `dash:mainnet` |
| Testnet | `dash:testnet` |

---

## BIP-44 Derivation

Dash uses **Coin Type 5** per the BIP-44 specification.

```
m / 44' / 5' / account' / change / index
```

| Address Type | Support |
|---|---|
| P2PKH (standard) | ✅ |
| P2SH (multisig / escrow) | ✅ |

---

## Definition of Done

- [ ] All unit tests pass for BIP-44 derivation and PSBT signing
- [ ] Successful mainnet transaction broadcast via DAPI
- [ ] Transaction confirmed as "Instant" (IS Lock) by a third-party explorer
- [ ] TypeScript wrapper published as an npm package
- [ ] Upstream PR prepared for `open-wallet-standard/core`
- [ ] Documentation includes AI Agent / LangChain quick start guide

---

## Contributing

1. Fork the repository
2. Create a feature branch: `git checkout -b feat/your-feature`
3. Ensure all tests pass: `cargo test --workspace`
4. Submit a Pull Request with a clear description

Please follow the existing code style. Run `cargo fmt` and `cargo clippy` before submitting.

---

## License

Apache-2.0 — see [LICENSE](LICENSE) for details.

---

- [BIP-44 Multi-Account Hierarchy](https://github.com/bitcoin/bips/blob/master/bip-0044.mediawiki)
- [Dash InstantSend](https://docs.dash.org/en/stable/introduction/features.html#instantsend)
