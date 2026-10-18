# 🐦 Decoy Canary Wallet

> **A Zcash-based wallet security prototype that uses shielded canary outputs to detect suspicious activity, trigger a protective freeze, and require legitimate recovery.**

[![Status](https://img.shields.io/badge/status-MVP%20in%20development-orange)]()
[![Zcash](https://img.shields.io/badge/Zcash-shielded%20payments-yellow)]()
[![Zero--Knowledge](https://img.shields.io/badge/ZK-Halo%202-blue)]()

## Overview

The **Decoy Canary Wallet** is a security-focused wallet prototype designed around a simple idea:

> **If an attacker touches a decoy, the wallet knows that something is wrong.**

The wallet creates a legitimate payment alongside wallet-controlled **canary outputs**. These canary outputs are not intended to provide additional cryptographic anonymity. Instead, they act as **tripwires**.

If a canary is unexpectedly spent:

```text
Real payment + canary outputs
             │
             ▼
       Attacker probes
             │
             ▼
      Canary is touched
             │
             ▼
       CANARY TRIGGER
             │
             ▼
      Protected / Frozen
             │
             ▼
      Legitimate recovery
```

The project explores how this security mechanism can be combined with **Zcash shielded transactions and zero-knowledge proofs**.

---

## Why?

Blockchain transactions can provide strong cryptographic guarantees while still creating difficult security problems for users.

A compromised wallet may allow an attacker to attempt to spend or interact with wallet-controlled funds.

Traditional security mechanisms often detect compromise after the attacker has already taken meaningful action.

The Decoy Canary Wallet explores a different approach:

> **Give the attacker something they should never touch, and use that interaction as an early-warning signal.**

---

## Core MVP

The MVP intentionally focuses on five components:

- **Real payment**
- **Canary outputs**
- **Canary/tripwire detection**
- **Protective wallet state**
- **Two-party recovery**

The MVP does **not** attempt to implement every possible wallet-security feature.

### Demo flow

1. Create a shielded payment containing the intended payment and wallet-controlled canary outputs.
2. Simulate or perform an attacker interaction with a canary.
3. Detect the canary interaction.
4. Place the wallet into a protected state.
5. Require legitimate recovery authorization.
6. Restore normal wallet operation after successful recovery.

---

## Zcash Integration

The project is designed around the current Zcash shielded transaction architecture.

The application layer is responsible for:

- identifying wallet-controlled canary outputs;
- maintaining the canary registry;
- monitoring for unexpected canary spends;
- triggering the protected state;
- managing recovery authorization.

Zcash is responsible for the underlying shielded transaction and cryptographic validity mechanisms.

Conceptually:

```text
┌─────────────────────────────────────┐
│        DECOY CANARY WALLET          │
│                                     │
│  Payment Manager                    │
│       │                             │
│       ├── Real payment              │
│       │                             │
│       └── Canary outputs            │
│                    │                │
│                    ▼                │
│             Canary Registry         │
│                    │                │
│                    ▼                │
│              Canary Monitor         │
│                    │                │
│              ┌─────┴─────┐          │
│              │           │          │
│           Normal      Canary hit    │
│              │           │          │
│              ▼           ▼          │
│           Continue     Freeze       │
│                          │           │
│                          ▼           │
│                 Two-party recovery  │
└──────────────────────────┬──────────┘
                           │
                           ▼
                 ┌──────────────────┐
                 │      ZCASH       │
                 │                  │
                 │ Shielded payment │
                 │ Commitments      │
                 │ Nullifiers       │
                 │ ZK proofs        │
                 │ Halo 2           │
                 └──────────────────┘
```

### Zero-knowledge component

The project does **not** attempt to implement a new ZK proving system.

Instead, it uses the ZK machinery provided by the Zcash protocol.

The current Zcash shielded architecture uses the Orchard protocol's action/proof machinery, with Halo 2 used for the relevant zero-knowledge proofs.

The goal is to understand and demonstrate how the existing Zcash privacy infrastructure can be incorporated into a security-oriented wallet design.

---

## Threat Model

The MVP focuses on a **blind attacker**.

We assume that:

- the attacker cannot distinguish the real target from the canary outputs;
- outputs are probed in random order;
- the attacker does not initially know which output is the real target;
- interaction with a canary is treated as evidence of compromise.

Let:

- `n` = number of canary outputs
- `k` = number of canary interactions required to trigger protection

For the MVP, the preferred configuration is:

```text
k = 1
```

A single canary interaction triggers the protective state.

### Probability model

Under the simplified blind-attacker model, the probability calculations are **illustrative model results, not guarantees of real-world security**.

Real-world effectiveness depends on factors including:

- how the attacker obtains information;
- whether the attacker can distinguish outputs;
- wallet implementation;
- transaction metadata;
- monitoring reliability;
- key compromise;
- timing and network behavior.

---

## Important Security Limitation

The canary mechanism is **not a replacement for cryptographic key security**.

If an attacker already controls the wallet's spending authority and knows the legitimate target, the attacker may be able to bypass the application-level protection.

Therefore:

> **The canary mechanism is primarily designed to detect a blind or partially informed attacker probing wallet-controlled outputs.**

It should not be interpreted as protection against complete compromise of the underlying spending authority.

---

## Protective State

When a canary is detected, the wallet transitions from:

```text
ACTIVE
   │
   │ canary detected
   ▼
PROTECTED
   │
   │ legitimate recovery
   ▼
RECOVERING
   │
   ▼
ACTIVE
```

While protected, ordinary wallet operations are blocked by the application.

This is an **application-level protection mechanism**. It is not a claim that Zcash consensus itself freezes the wallet.

---

## Recovery

The MVP uses a two-party authorization model for legitimate recovery.

Conceptually:

```text
              CANARY DETECTED
                     │
                     ▼
                  FROZEN
                     │
                     ▼
             Recovery requested
                     │
              ┌──────┴──────┐
              │             │
          Party A        Party B
              │             │
              └──────┬──────┘
                     │
                Both approve
                     │
                     ▼
                Recovery
                     │
                     ▼
                  ACTIVE
```

The exact transaction-level implementation is still being evaluated against the capabilities of the selected Zcash wallet stack.

---

## Current Status

**MVP: In development**

### Locked

- [x] Core concept
- [x] Real payment
- [x] Canary outputs
- [x] Canary/tripwire mechanism
- [x] Protective freeze
- [x] Two-party recovery concept
- [x] Blind-attacker model
- [x] Explicit security limitations

### Implementation

- [ ] Finalize Zcash wallet/client stack
- [ ] Implement shielded transaction creation
- [ ] Implement canary registry
- [ ] Implement canary monitoring
- [ ] Implement protected wallet state
- [ ] Implement recovery workflow
- [ ] Test Zcash integration
- [ ] Build end-to-end demo

---

## Roadmap

### Phase 1 — Zcash foundation

- Shielded transaction integration
- Wallet/key management
- Note and nullifier tracking
- Zcash test environment

### Phase 2 — Canary mechanism

- Create canary outputs
- Maintain canary registry
- Detect canary spends
- Trigger protective state

### Phase 3 — Recovery

- Implement two-party authorization
- Build recovery transaction workflow
- Test failure/recovery scenarios

### Phase 4 — Demo

- Normal payment
- Attacker interaction
- Canary detection
- Wallet freeze
- Legitimate recovery

### Future extensions

The following are intentionally **outside the MVP**:

- Decoy recipients
- Decoy amounts
- Automatic decoy replacement
- Repair/checkpoint mechanisms
- Escalation/tiered responses
- Owner-added decoys

---

## Architecture Principle

The project deliberately separates **Zcash privacy mechanisms** from the **application-level security mechanism**.

```text
Zcash
─────
Privacy
Shielded transactions
Commitments
Nullifiers
Zero-knowledge proofs
Halo 2

Our application
───────────────
Canary designation
Monitoring
Tripwire
Protected state
Recovery workflow
```

This distinction prevents the project from incorrectly claiming that canary outputs are a native Zcash privacy feature.

---

## What This Project Does NOT Claim

The project does **not** claim that:

- canary outputs provide additional cryptographic anonymity by themselves;
- Zcash consensus freezes compromised wallets;
- the canary mechanism defeats an attacker who already knows the real target;
- application-level freezing can stop an attacker who independently controls the underlying spending authority;
- the prototype is production-ready wallet security.

The goal is to demonstrate and investigate the concept honestly.

---

## Technology

The final implementation stack is being selected based on compatibility with the current Zcash shielded protocol.

Expected areas include:

- Zcash
- Shielded transactions
- Zero-knowledge proofs
- Halo 2
- Rust / Zcash libraries
- Wallet application layer
- Automated tests

---

## Competition

This project is being developed for **ZECATHON 2026**, with the goal of exploring a practical security mechanism on top of Zcash's privacy technology.

The competition submission will prioritize:

1. Working prototype
2. Meaningful Zcash integration
3. Clear privacy/security model
4. Reproducible demonstration
5. Honest limitations

---

## Disclaimer

This is a research/prototype project.

It is **not intended to secure real funds** and should not be used as a production wallet or relied upon for financial security.

Cryptographic and wallet-security claims should be independently reviewed before any production deployment.

---

## License

License to be determined.
