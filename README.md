# ProofDock — proofdock-contracts

> Credential registry and verifiable status proofs.

![ProofDock Smart contracts](banner.png)

## About the project

ProofDock is a credential registry built on Stellar/Soroban. Issuers (universities, training bodies, employers, licensing authorities) register credentials, and anyone — an employer, a regulator, the holder — can later check whether a credential is still valid and obtain a verifiable status proof, without having to phone or trust the issuer's own website. The chain is the neutral source of truth for credential status; everything else is built around it.

**Who it is for:** Issuers, credential holders and verifiers (employers, schools, regulators).

**How the pieces fit together:**

| Repository | Responsibility |
|---|---|
| `proofdock-contracts` | On-chain Soroban state and authorization — the source of truth |
| `proofdock-backend` | Off-chain indexing, read models and operational APIs |
| `proofdock-app` | User-facing web application |

Typical flow:

1. An issuer registers or updates a credential's status on-chain (authorized by the issuer's Stellar account).
2. The backend indexes the resulting contract state into a fast read model.
3. A verifier opens the web app, looks a credential up and gets a status proof backed by public chain state.

## This repository: Smart contracts

The **contracts** repository is the on-chain core of ProofDock. It holds the minimum durable state the product needs and enforces who is allowed to change it. Anything that must be trustworthy and publicly verifiable lives here; everything else lives in the app and backend.

### What is included today

- A Soroban smart contract (`ProofDockContract`, Rust, `#![no_std]`) in a Cargo workspace (`soroban-sdk` 28).
- `initialize(admin)` — sets the administrator; requires that address to authorize the call and stores it under the `ADMIN` key.
- `record(actor, value)` — writes an `i128` value under the `VALUE` key; requires `actor` to authorize the call.
- `read()` — returns the stored value (defaults to `0` if nothing has been recorded).
- A unit test (`records_value`) that initializes the contract, records `42` and reads it back.
- Size- and safety-focused release profile: `opt-level = "z"`, LTO, `overflow-checks = true`, `panic = "abort"`, stripped symbols.

> The generic `value` model is a placeholder. The architecture notes call for replacing it with a ProofDock-specific state machine (see *Roadmap*) before any mainnet deployment.

### Tech stack

Rust · Soroban SDK 28 · Stellar CLI

### Getting started

```bash
# run the contract unit tests
cargo test

# build the WASM contract
stellar contract build

# formatting check (same as CI)
cargo fmt --all -- --check
```

The same three commands are available as `make test`, `make build` and `make fmt`.

## Roadmap

- Replace the generic `i128` value with the ProofDock domain model.
- Add granular authorization (admin vs. authorized actors) and emitted events for the backend to index.
- Expand tests to cover unauthorized access and edge cases, then get an independent audit before mainnet.

## Maintainer

`@ollypee22`

## Status

**v0.1.0 development baseline — not audited and not production-ready.**

## Stellar alignment

The project uses Stellar/Soroban where on-chain state is the source of truth or where deterministic settlement is valuable. Off-chain services are kept out of consensus-critical logic.

## License

Apache-2.0
