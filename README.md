# PQHD -- Post-Quantum Hierarchical Deterministic Custody

* **Specification Version:** 1.2.0
* **Status:** Implementation Ready
* **Date:** 2026
* **Author:** rosiea
* **Contact:** [PQRosie@proton.me](mailto:PQRosie@proton.me)
* **Licence:** Apache License 2.0 — Copyright 2026 rosiea
* **PQ Ecosystem:** CORE — Post-quantum custody architecture for Bitcoin.


---

**Problem:** Key possession is treated as sufficient authority. Custody architectures lack deterministic pre-signing governance, time-bounded policy enforcement, and quantum-transition discipline.

**Solution:** PQHD separates key capability from spending authority. Every custody operation requires predicate evaluation before signing is permitted. Post-quantum by default. Fail-closed by design.

PQHD defines custody policy only. It does not enforce.

Part of the [PQ Ecosystem](https://github.com/rosieRRRRR/pq-ecosystem).

---

## **Summary**

PQHD defines custody structures and signing constraints for post-quantum key management.

It specifies hierarchical key derivation, descriptor-based custody policies, quorum requirements, and signing conditions, while explicitly separating key possession from authorisation.

PQHD does not approve signing actions. All decisions about whether a signature or spend is permitted are evaluated by PQSEC.

---

## Index

1. [Scope and Custody Boundary](#1-scope-and-custody-boundary)
2. [Non-Goals and Separation of Concerns](#2-non-goals-and-separation-of-concerns)
3. [Threat Model](#3-threat-model)
4. [Trust Assumptions](#4-trust-assumptions)
   - [4A. Quantum Safety Scope and Chain Primitive Dependencies](#4a-quantum-safety-scope-and-chain-primitive-dependencies-normative)
5. [Architecture Overview](#5-architecture-overview)
   - [5A. Explicit Dependencies](#5a-explicit-dependencies)
6. [Conformance Keywords](#6-conformance-keywords)
7. [Custody Authority Model](#7-custody-authority-model)
8. [Unified Custody Predicate](#8-unified-custody-predicate)
   - [8.1 Predicate Naming Discipline](#81-predicate-naming-discipline)
9. [Custody Tiers](#9-custody-tiers)
   - [9.1 Baseline Custody](#91-baseline-custody)
   - [9.2 Enterprise Custody](#92-enterprise-custody)
10. [Key Generation and Derivation](#10-key-generation-and-derivation)
    - [10.0 Derivation Function and Domain Separation](#100-derivation-function-and-domain-separation-normative)
    - [10.1 Root Key Generation](#101-root-key-generation)
    - [10.2 Key Class Derivation](#102-key-class-derivation-normative)
    - [10.3 Key Classes](#103-key-classes)
    - [10.4 Key Derivation Rules](#104-key-derivation-rules)
11. [Bitcoin Compatibility Keys](#11-bitcoin-compatibility-keys)
    - [11.1 secp256k1 Scalar Derivation](#111-secp256k1-scalar-derivation-normative)
    - [11.2 Bitcoin Key Authority](#112-bitcoin-key-authority)
12. [Canonical PSBT Handling](#12-canonical-psbt-handling)
    - [12.1 PSBT Structural Requirements](#121-psbt-structural-requirements)
    - [12.2 PSBT Canonicalisation](#122-psbt-canonicalisation)
    - [12.3 PSBT Equivalence](#123-psbt-equivalence)
13. [Dual-Control Signing Requirement](#13-dual-control-signing-requirement)
    - [13.1 Principle](#131-principle)
    - [13.2 EnforcementOutcome Compatibility](#132-enforcementoutcome-compatibility)
    - [13.3 Signing Component Requirements](#133-signing-component-requirements)
    - [13.4 PQSEC Evaluation Request](#134-pqsec-evaluation-request)
    - [13A. Signature Preimage Rule](#13a-signature-preimage-rule-normative)
    - [13.5 Signing Flow](#135-signing-flow)
14. [Custody Hardware Profile](#14-custody-hardware-profile)
15. [Multisignature and Quorum Authority](#15-multisignature-and-quorum-authority)
    - [15.1 Quorum Requirements](#151-quorum-requirements)
    - [15.2 M-of-N Custody](#152-m-of-n-custody)
    - [15.3 Partial Signing Protocols](#153-partial-signing-protocols)
16. [Ledger Authority](#16-ledger-authority)
17. [Recovery Governance](#17-recovery-governance)
    - [17.1 Guardian Set](#171-guardian-set)
    - [17.2 Recovery Intent](#172-recovery-intent)
    - [17.3 Recovery Approvals](#173-recovery-approvals)
    - [17.4 Recovery Activation Conditions](#174-recovery-activation-conditions)
    - [17.5 Recovery Constraints](#175-recovery-constraints)
18. [Device Lifecycle Management](#18-device-lifecycle-management)
    - [18.1 Device Enrollment](#181-device-enrollment)
    - [18.2 Device Revocation](#182-device-revocation)
19. [Epoch Clock Handling](#19-epoch-clock-handling)
20. [Error Handling](#20-error-handling)
    - [20.1 Error Code Mapping](#201-error-code-mapping)
    - [20.2 Error Propagation](#202-error-propagation)
21. [Dependency Boundaries](#21-dependency-boundaries)
    - [21.1 Optional Evidence Producers](#211-optional-evidence-producers)
    - [21.2 Enforcement Boundary Discipline](#212-enforcement-boundary-discipline)
22. [Execution Integration](#22-execution-integration)
    - [22.1 ZET / ZEB Integration](#221-zet--zeb-integration)
  - [22.2 SEAL Integration (Normative)](#222-seal-integration-normative)
23. [Failure Semantics](#23-failure-semantics)
24. [Bootstrap Trust Anchors](#24-bootstrap-trust-anchors)
    - [24.1 Initial Trust Establishment](#241-initial-trust-establishment)
    - [24.2 Bootstrap Mode](#242-bootstrap-mode)
    - [24.3 Bootstrap Failure Handling](#243-bootstrap-failure-handling)
25. [Conformance](#25-conformance)
26. [Transcript-Bound Signing](#26-transcript-bound-signing)
27. [Canonical PSBT Hash](#27-canonical-psbt-hash)

**Annexes**

- [Annex A - Conformance Determination (Informative)](#annex-a---conformance-determination-informative)
- [Annex B - Signing Authorisation Flow (Informative)](#annex-b---signing-authorisation-flow-informative)
- [Annex C - EnforcementOutcome Acceptance Rules (Normative)](#annex-c---enforcementoutcome-acceptance-rules-normative)
- [Annex D - Deterministic Key Derivation (Reference)](#annex-d---deterministic-key-derivation-reference)
- [Annex E - Bitcoin secp256k1 Scalar Derivation (Reference)](#annex-e---bitcoin-secp256k1-scalar-derivation-reference)
- [Annex F - Canonical PSBT Hashing and Equivalence (Normative)](#annex-f---canonical-psbt-hashing-and-equivalence-normative)
- [Annex G - Custody Predicate Assembly (Informative)](#annex-g---custody-predicate-assembly-informative)
- [Annex H - Dual-Control Signer (Reference)](#annex-h---dual-control-signer-reference)
- [Annex I - Untrusted M-of-N Coordinator (Reference)](#annex-i---untrusted-m-of-n-coordinator-reference)
- [Annex J - Custody Lifecycle Extensions (Normative)](#annex-j---custody-lifecycle-extensions-delegation--guardian-recovery--safemode-normative)
  - [J.7 DelegationConstraint Scope Vocabulary (Normative)](#j7-delegationconstraint-scope-vocabulary-normative)
- [Annex K - Recovery Activation Sequence (Informative)](#annex-k---recovery-activation-sequence-informative)
- [Annex L - Device Enrollment and Revocation (Informative)](#annex-l---device-enrollment-and-revocation-informative)
- [Annex M - Custody Tier Qualification Matrix (Informative)](#annex-m---custody-tier-qualification-matrix-informative)
- [Annex N - Migration Guidance (Informative)](#annex-n---migration-guidance-informative)
- [Annex O - Signing Attempt Record (Informative)](#annex-o---signing-attempt-record-informative)
- [Annex P - Troubleshooting (Informative)](#annex-p---troubleshooting-informative)
- [Annex Q - Security Checklist (Informative)](#annex-q---security-checklist-informative)
- [Annex R - Payment Endpoint Keys (Normative)](#annex-r---payment-endpoint-keys-normative)
- [Annex S - State-Transition and Execution Integration (Mixed: Normative + Informative)](#annex-s---state-transition-and-execution-integration-mixed-normative--informative)
- [Annex T - Recipient Whitelisting (Optional, Normative)](#annex-t---recipient-whitelisting-optional-normative)
  - [T.X Global Agent Spending Budget (Normative)](#tx-global-agent-spending-budget-normative)
- [Annex U - Fee, Change, and Output Manifest Policy (Optional, Normative)](#annex-u---fee-change-and-output-manifest-policy-optional-normative)
- [Annex V - PSBT Integrity Binding (Optional, Normative)](#annex-v---psbt-integrity-binding-optional-normative)
- [Annex W - Address Reuse Prohibition (Optional, Normative)](#annex-w---address-reuse-prohibition-optional-normative)
- [Annex X - Secure Session Resumption (Optional, Normative)](#annex-x---secure-session-resumption-optional-normative)
- [Annex Y - UTXO Exposure Discipline (Optional, Normative)](#annex-y---utxo-exposure-discipline-optional-normative)
- [Annex Z - Display Integrity and User Approval Binding (Optional, Normative)](#annex-z---display-integrity-and-user-approval-binding-optional-normative)
- [Annex AA - Hash-Only Audit Logging (Optional, Normative)](#annex-aa---hash-only-audit-logging-optional-normative)
- [Annex AB - Emergency Freeze and Authority Lock (Optional, Normative)](#annex-ab---emergency-freeze-and-authority-lock-optional-normative)
- [Annex AC - Common Implementation Mistakes (Informative)](#annex-ac---common-implementation-mistakes-informative)

[Glossary](#glossary-canonical)
[Changelog](#changelog)
[Acknowledgements](#acknowledgements-informative)

---

## **Non-Normative Overview - For Explanation and Orientation Only**

**This section is NOT part of the conformance surface.
It is provided for explanatory and onboarding purposes only.**

### Plain Summary

PQHD specifies what must be true before Bitcoin signing is allowed. It does not sign, enforce, or execute transactions. Instead, it defines deterministic custody rules and hands them to PQSEC, which evaluates whether a given operation is authorised.

A PQHD-conformant system can safely refuse to sign even when valid keys are present, because authority is conditional, not implicit.

### What PQHD Is / Is Not

| PQHD IS                            | PQHD IS NOT             |
| ---------------------------------- | ----------------------- |
| A custody policy definition layer  | An enforcement engine   |
| A deterministic key hierarchy spec | A signing executor      |
| A quorum and role framework        | A single-key wallet     |
| Predicate-driven by design         | Trust-based or implicit |

### Canonical Flow (Single Line)

Custody Intent → PQHD Predicate Set → PQSEC Evaluation → Dual-Control Signing (if allowed)

### Why This Exists

Key-based authority collapses under compromise and quantum attack. PQHD exists to replace implicit authority with explicit, auditable predicates evaluated by an external enforcement core. This allows custody systems to fail closed, preserve safety under partial compromise, and evolve cryptography without rewriting authority assumptions.

---

## **1. Scope and Custody Boundary**

PQHD defines **Bitcoin custody authority policy only**.

PQHD normatively defines:

* custody authority rules and tier qualification
* deterministic post-quantum key hierarchy requirements
* custody predicate composition
* canonical PSBT equivalence requirements
* multisignature quorum authority models
* custody recovery governance constraints
* signing authorisation and delegation requirements
* custody-specific conformance requirements

### Custody Boundary

PQHD defines **what custody authority requires**.

### Enforcement Boundary

PQHD **does not**:

* evaluate predicates
* enforce refusal or escalation
* perform lockout, backoff, or retry logic
* issue or validate runtime attestations
* issue or validate time artefacts
* perform freshness or monotonicity enforcement
* execute Bitcoin scripts or broadcast transactions
* establish transport sessions

All such behaviour is defined exclusively by **PQSEC** and producing specifications.

Any implementation performing predicate evaluation, refusal, escalation, freshness enforcement, or runtime evaluation inside PQHD is **non-conformant**.

---

## **2. Non-Goals and Separation of Concerns**

PQHD does not define:

* predicate evaluation logic
* enforcement engines
* runtime integrity probe generation
* attestation semantics
* time anchoring or issuance
* transaction broadcast or mempool strategy
* Bitcoin execution hardening
* transport protocols or session establishment
* AI behaviour, inference, or alignment
* user interface behaviour or workflows

Custody authority defined by PQHD has **no operational meaning** without PQSEC enforcement.

PQHD grants no authority by key possession. Authority is conditional and refusal-based.

---

## **3. Threat Model**

PQHD assumes adversaries may:

* obtain classical Bitcoin private keys or seeds
* compromise individual signing devices
* replay, reorder, or suppress messages
* attempt rollback or omission of wallet state
* coerce or compromise a subset of participants
* possess future cryptographically relevant quantum computation capability

PQHD does **not** assume trusted clocks, trusted networks, trusted coordinators, trusted runtimes, or secrecy of classical key material.

---

## **4. Trust Assumptions**

PQHD operates under the following trust assumptions:

* custody authority derives only from deterministic predicate satisfaction
* possession of Bitcoin keys does not imply authority
* cryptographic verification is performed locally
* canonical encoding eliminates structural ambiguity
* time artefacts, runtime evidence, and enforcement outcomes are external inputs

---

## **4A. Quantum Safety Scope and Chain Primitive Dependencies (Normative)**

PQHD defines quantum-safe custody governance for off-chain policy and enforcement inputs.

All authoritative PQHD custody artefacts (e.g. policy profiles, consent proofs, recovery packages, enforcement inputs) MUST be:

- signed and verified using a post-quantum signature scheme permitted by the applicable suite profile, and
- hashed and signed over PQSF-canonical bytes.

However, if the underlying on-chain spending authorisation mechanism relies on non-post-quantum primitives (e.g. currently deployed ECDSA or Schnorr signature schemes), PQHD MUST NOT claim end-to-end quantum-safe settlement.

In such deployments, PQHD MAY claim:

- quantum-safe custody governance and refusal semantics (off-chain), and
- exposure-minimisation strategies for execution and broadcast where defined by applicable execution-layer protocols,

but MUST treat migration of on-chain signature primitives to post-quantum alternatives as an external dependency.

---

## **5. Architecture Overview**

PQHD defines a custody authority layer governing whether Bitcoin signatures may be produced.

PQHD consists of:

* a deterministic post-quantum root key hierarchy
* custody tiers and declarative custody predicates
* dual-control signing requirements
* canonical PSBT equivalence rules
* multisignature quorum models
* deterministic ledger authority rules
* recovery governance artefacts

PQHD does **not** produce enforcement decisions.
**PQSEC produces the EnforcementOutcome for each custody attempt.**

---

## 5A. Explicit Dependencies

| Specification | Minimum Version | Purpose |
|---------------|-----------------|---------|
| PQSEC | ≥ 2.0.3 | Enforcement of custody predicates and runtime integrity |
| PQSF | ≥ 2.0.3 | Canonical encoding and hashing/derivation parameter binding |
| Epoch Clock | ≥ 2.1.0 | Time artefacts and staleness model (§9A), mirror signing, v3 schema presence |
| ZET/ZEB | ≥ 1.2.0 | Execution boundary (when executing Bitcoin transactions) |

Runtime integrity evidence is consumed exclusively via PQSEC predicates.

Implementations MAY evaluate using earlier versions, but MUST NOT claim conformance while below the stated minimums.

PQHD defines custody policy only. All enforcement is performed by PQSEC.

---

## **6. Conformance Keywords**

The key words MUST, MUST NOT, REQUIRED, SHALL, SHALL NOT, SHOULD, SHOULD NOT, RECOMMENDED, MAY, and OPTIONAL are to be interpreted as described in RFC 2119.

---

## **7. Custody Authority Model**

1. Custody authority MUST NOT derive from possession of a single key.
2. Custody authority MUST derive from simultaneous satisfaction of all required custody predicates.
3. Custody authority MUST be evaluated deterministically.
4. Any ambiguity, missing input, or validation failure MUST result in refusal.

No degraded or partial authority state is permitted for Authoritative operations.

---

## **8. Unified Custody Predicate**

Custody authority for Bitcoin signing is defined by the following declarative predicate composition:

```
valid_for_signing =
    valid_tick
AND valid_consent
AND valid_policy
AND valid_runtime
AND valid_quorum
AND valid_ledger
AND valid_psbt
```

PQHD defines **predicate composition only**.

Predicate production, validation, freshness, monotonicity, refusal, escalation, and lockout semantics are defined externally by PQSEC and producing specifications.

### **8.1 Predicate Naming Discipline**

PQHD does not introduce independent freshness, versioning, or execution predicates. All freshness and monotonicity requirements are enforced via valid_tick and artefact expiry semantics. All policy integrity, versioning, and rollback protection are enforced via valid_policy. Execution-layer behaviour (SEAL, ZEB) does not modify custody authority predicates and MUST NOT introduce parallel authorisation signals.

---

## **9. Custody Tiers**

PQHD defines two custody tiers that qualify the security posture under which custody authority claims MAY be made. Only Baseline and Enterprise tiers exist. There is no reduced, schema-only, or transitional tier. Implementations that do not satisfy Baseline requirements MUST NOT claim PQHD conformance (see Annex N for migration guidance).

### **9.1 Baseline Custody**

Baseline Custody MUST satisfy all of the following:

* multi-device signing quorum
* deterministic custody policy constraints
* canonical PSBT equivalence
* ledger continuity requirements
* dual-control signing enforced via PQSEC

A single compromised device MUST be insufficient to authorise a spend.

Baseline Custody MAY be used for high-value self-custody and professional custody environments.

---

### **9.2 Enterprise Custody**

Enterprise Custody MUST satisfy all Baseline Custody requirements plus:

* quorum diversity constraints across administrative or trust domains
* guardian participation in recovery
* deterministic recovery governance with enforced delay
* comprehensive auditability

Enterprise Custody is intended for institutional, fiduciary, and sovereign environments.

---

## **10. Key Generation and Derivation**

### **10.0 Derivation Function and Domain Separation (Normative)**

All PQHD key derivations use SHAKE256-256 (32-byte output).

Domain separation MUST be implemented by hashing the concatenation of:

1. a UTF-8 domain label (ASCII),
2. a single 0x00 separator byte,
3. the canonical CBOR encoding of the context map.

Formally:

```
derive(domain_label, context_cbor) = SHAKE256-256(domain_label || 0x00 || context_cbor)
```

`context_cbor` MUST be PQSF deterministic CBOR.

Implementations MUST NOT use cSHAKE256 in PQHD.

---

### **10.1 Root Key Generation**

A single post-quantum root key MUST be generated at wallet creation:

```
root_key = SHAKE256-256("PQHD-RootKey" || 0x00 || seed)
```

The `seed` bytes MUST be treated as opaque entropy input. No canonical encoding is applied to `seed`.

Where:

* `seed` is high-entropy random material (minimum 256 bits)
* `seed` MUST be generated using a cryptographically secure RNG
* `seed` MUST NOT be reused across wallets
* `SHAKE256-256` output length MUST be exactly 32 bytes

Loss of the root key without recovery governance results in permanent loss of custody.

---

### **10.2 Key Class Derivation (Normative)**

For any derived key class, define a deterministic context map:

```
context = {
  "key_class": <tstr>,
  ... key-class-specific fields ...
}
context_cbor = CBOR_ENCODE_DETERMINISTIC(context)
```

Then derive:

```
child_key = SHAKE256-256("PQHD-Key:" || key_class || 0x00 || context_cbor)
```

Where:
- `"PQHD-Key:" || key_class` is a UTF-8 string domain label,
- `context_cbor` is PQSF deterministic CBOR,
- output is exactly 32 bytes.

Derivation MUST be deterministic and reproducible across implementations.

---

### **10.3 Key Classes**

PQHD defines the following key classes:

* **custody** - multisignature custody keys
* **recovery** - guardian and recovery keys
* **identity** - device identity binding keys
* **ephemeral** - per-operation keys (MUST NOT be used for custody)

Key derivability MUST NOT imply custody authority.

---

### **10.4 Key Derivation Rules**

1. All keys MUST derive from PQHD root material.
2. Key classes MUST be isolated by purpose.
3. Two implementations given identical inputs MUST derive identical keys.
4. Ephemeral keys MUST NOT be reused across operations.

---

## **11. Bitcoin Compatibility Keys**

### **11.1 secp256k1 Scalar Derivation (Normative)**

To derive a secp256k1 private scalar from `root_key`, use rejection sampling
with SHAKE256-256 and explicit domain separation.

Let `n` be the secp256k1 curve order (`0xFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFEBAAEDCE6AF48A03BBFD25E8CD0364141`).

For counter values `i = 0, 1, 2, ...`:

```
candidate_bytes = SHAKE256-256("PQHD-Key:secp256k1" || 0x00 || root_key || i32be(i))
candidate = int(candidate_bytes)
```

Accept the first `candidate` such that:

```
1 <= candidate < n
```

`i32be(i)` is the 4-byte big-endian encoding of `i`.

This procedure is deterministic given identical inputs. The expected number of iterations is 1 (probability of rejection is approximately 2^-128).

---

### **11.2 Bitcoin Key Authority**

1. Bitcoin keys MUST NOT grant custody authority by themselves.
2. Possession of Bitcoin private keys is necessary but not sufficient for custody.
3. Custody authority requires satisfaction of the unified custody predicate.

---

## **12. Canonical PSBT Handling**

### **12.1 PSBT Structural Requirements**

PSBTs submitted for custody signing MUST satisfy:

1. PSBT version MUST be 2.
2. All inputs MUST include `witness_utxo`.
3. Inputs MUST NOT include `non_witness_utxo`.
4. Unknown or proprietary fields MUST cause validation failure.

---

### **12.2 PSBT Canonicalisation**

PSBTs MUST be canonicalised deterministically prior to signing.

Canonicalisation MUST ensure:

* deterministic field ordering: PSBT key-value records within each map (global, per-input, per-output) MUST be sorted by key type (ascending numeric order of the key type byte), then by key data (lexicographic byte order). This ordering follows BIP-174's requirement that "key-value records within each map must be sorted by key."
* identical semantic representation across signers
* rejection of unknown or proprietary PSBT fields (key types not defined in BIP-174/BIP-370)
* removal of duplicate keys within the same map
* PSBT v2 (BIP-370) structural requirements when applicable

---

### **12.3 PSBT Equivalence**

1. All signing participants MUST evaluate identical canonical PSBTs.
2. PSBT equivalence MUST be enforced prior to signing.
3. Any divergence MUST result in refusal.

---

## **13. Dual-Control Signing Requirement**

### **13.1 Principle**

Bitcoin signing MUST require **both**:

* possession of the relevant signing key material, and
* a valid **PQSEC EnforcementOutcome** authorising the operation.

Possession of keys alone MUST NOT permit signing.

### 13.1A Clarification: Dual-Control vs. Multi-Factor Authentication (Informative)

The dual-control signing requirement defined in §13.1 may superficially resemble multi-factor authentication (MFA) because multiple independent conditions must be satisfied before a signature is produced.

However, PQHD does not implement traditional two-factor authentication.

In conventional MFA, multiple credentials belonging to the same principal (e.g., password + TOTP) are combined to authenticate identity. In PQHD, the two required elements are categorically different:

1. **Cryptographic Capability** -- possession of the private key material.
2. **Externalised Authority** -- a valid, time-bound EnforcementOutcome produced by PQSEC after deterministic policy evaluation.

The EnforcementOutcome is not a credential and does not authenticate identity. It is a control-plane artefact that represents the result of independent policy adjudication.

This architecture enforces a strict separation between key possession and authority to act. Possession of a valid private key is necessary but never sufficient for signing. Authority derives exclusively from predicate satisfaction as evaluated by PQSEC.

This design is more accurately described as **capability--authority decoupling** rather than multi-factor authentication.

---

### **13.2 EnforcementOutcome Compatibility**

PQHD signing components MUST accept PQSEC-native EnforcementOutcome artefacts as authoritative.

Where an execution boundary requires a boolean `allowed`, implementations MUST map:

```
allowed = (decision == "ALLOW")
```

Implementations MUST preserve and verify the following binding fields:

* `decision_id`
* `intent_hash`
* `session_id`
* `exporter_hash`
* `issued_tick`
* `expiry_tick`

If `decision == "FAIL_CLOSED_LOCKED"`, signing MUST be refused and surfaced as a lockout condition.

**Signing components MUST implement the EnforcementOutcome acceptance and validation requirements defined in Annex C in full.**

---

### **13.3 Signing Component Requirements**

1. Signing components MUST refuse to produce signatures without a valid EnforcementOutcome.
2. EnforcementOutcome expiry MUST be enforced locally.
3. Any mismatch in binding fields MUST cause refusal.
4. EnforcementOutcome artefacts MUST be single-use per signing attempt.

---

### **13.4 PQSEC Evaluation Request**

To obtain an EnforcementOutcome, a PQHD custody operation MUST submit to PQSEC:

1. Operation identifier and class
2. Canonical PSBT bytes (or bundle_hash)
3. Session binding (session_id, exporter_hash for Authoritative operations)
4. All required custody artefacts:
   * EpochTick (JCS canonical JSON bytes)
   * ConsentProof
   * Policy artefacts
   * Runtime integrity evidence (as evaluated by PQSEC)
   * Quorum state
   * Ledger state

PQSEC evaluates the unified custody predicate and returns an EnforcementOutcome.

PQHD signing components MUST NOT produce signatures without a valid EnforcementOutcome satisfying Annex C.

---

## **13A. Signature Preimage Rule (Normative)**

For any PQHD artefact in this specification that contains a `signature: bstr`
field, the signature MUST be computed over the deterministic CBOR encoding
of the artefact with the `signature` field omitted.

Verification MUST reconstruct the same canonical bytes.

This rule applies to (non-exhaustive):
- CustodyHardwareProfile (§14)
- DeviceEnrollment (§18.1)
- DeviceRevocation (§18.2)
- DelegationConstraint (Annex J.1)
- GuardianSet (Annex J.2)
- RecoveryIntent (Annex J.3)
- RecoveryApproval (Annex J.4)
- SafeModeState (Annex J.5)
- PaymentEndpointKey (Annex R)

---

### **13.5 Signing Flow**

```
User → Create PSBT
    ↓
PQHD → Assemble custody predicates
    ↓
PQSEC → Evaluate predicates → EnforcementOutcome
    ↓
Signing Component → Verify EnforcementOutcome → Produce signature
    ↓
Aggregate signatures → Execute via ZET → Broadcast via ZEB
```

---

## **14. Custody Hardware Profile**

PQHD defines a CustodyHardwareProfile artefact describing hardware requirements.

```
CustodyHardwareProfile = {
  profile_id: tstr,
  device_class: tstr,
  isolation_properties: [* tstr],
  attestation_requirements: [* tstr],
  suite_profile: tstr,
  signature: bstr
}
```

Rules:

1. CustodyHardwareProfile defines requirements only.
2. Evaluation and enforcement are performed by PQSEC.
3. Absence of a CustodyHardwareProfile MUST NOT grant authority.

---

## **14A. Biometric Matching Boundary (Normative)**

### 14A.1 Scope

Biometric authentication MAY be used as an input to custody policy. Biometric signals are evidence only and MUST NOT grant custody authority independently.

Biometric validation does not satisfy any custody predicate unless explicitly consumed and evaluated by PQSEC under policy.

### 14A.2 Holder Execution Boundary Requirement

Biometric template storage and biometric matching MUST occur exclusively within:

* the Holder Execution Boundary, or
* a hardware-backed secure enclave contained within the Holder Execution Boundary.

Raw biometric templates MUST NOT be transmitted outside the Holder Execution Boundary or exported in reconstructable form.

### 14A.3 Template Protection

Biometric templates MUST:

* be stored in hardware-isolated memory or equivalent enclave,
* be encrypted at rest,
* be non-exportable in raw form,
* be non-reconstructable from any exported artefact.

No raw biometric data MAY appear in any PQHD artefact or ReceiptEnvelope.

### 14A.4 Matching Result Tokenisation

Where biometric verification is required by policy, the output MUST be a signed, scope-bound, time-bound token.

The token MUST:

* bind to `sid`,
* bind to `decision_id` (for Authoritative operations),
* include `issued_tick`,
* include `expiry_tick`,
* be canonically encoded,
* be signed under a device identity key.

Biometric match tokens MUST NOT be reusable across distinct `decision_id` values.

### 14A.5 Authority Boundary

Biometric verification:

* does not grant authority,
* does not bypass dual-control signing,
* does not replace ConsentProof or EnforcementOutcome requirements.

All enforcement decisions remain exclusively within PQSEC.

---

## **15. Multisignature and Quorum Authority**

### **15.1 Quorum Requirements**

1. Quorum thresholds MUST be explicit and deterministic.
2. Partial quorum satisfaction is forbidden.
3. Coordinators MUST be treated as untrusted.

---

### **15.2 M-of-N Custody**

For M-of-N custody:

1. M signatures MUST be collected from N signers.
2. Each signer MUST independently validate:

   * canonical PSBT
   * EnforcementOutcome validity
   * custody predicate completeness
3. Coordinators MUST NOT force or bypass signatures.

---

### **15.3 Partial Signing Protocols**

**PSBT Distribution**

1. Coordinator constructs canonical PSBT.
2. PSBT is distributed to all N signers.

**Signature Collection**

1. Signers return signatures asynchronously.
2. Coordinator collects until M signatures are obtained.
3. Timeout MAY be enforced; failure results in refusal.

---

## **16. Ledger Authority**

1. Custody-relevant events MUST be recorded in an append-only ledger.
2. Ledger continuity MUST be monotonic.
3. Ledger divergence MUST invalidate custody authority.

Ledger structure, reconciliation, and enforcement semantics are defined by **PQSF** and **PQSEC**.

---

## **17. RECOVERY GOVERNANCE**

Recovery is an explicit, governed custody operation. Recovery MUST NOT bypass custody predicates, MUST NOT grant authority by itself, and MUST remain fully subject to PQSEC enforcement.

PQHD recovery governance is expressed exclusively using the **custody lifecycle extension artefacts** defined in **Annex J**:

* **GuardianSet**
* **RecoveryIntent**
* **RecoveryApproval**
* **SafeModeState** (when applicable)

PQHD defines the required structures and constraints only.
All evaluation, delay enforcement, quorum validation, refusal, and lockout semantics are performed exclusively by **PQSEC**.

Recovery activation is an **Authoritative operation** and therefore requires a valid **PQSEC EnforcementOutcome** prior to any recovery signing or custody mutation.

---

### **17.1 Guardian Set**

A recovery-capable custody domain MUST define a **GuardianSet**.

A GuardianSet specifies:

* the set of eligible guardians,
* the approval threshold required for recovery,
* the cryptographic suite used for verification.

GuardianSet creation and modification SHOULD be recorded in the custody ledger.

A GuardianSet grants no authority by itself and MUST NOT enable recovery without satisfying all required predicates as evaluated by PQSEC.

---

### **17.2 Recovery Intent**

A recovery operation MUST be initiated using a **RecoveryIntent** artefact.

RecoveryIntent:

* explicitly declares a recovery attempt,
* binds the attempt to a specific GuardianSet,
* defines an enforced activation delay,
* MUST be time-bound and canonically signed,
* SHOULD be recorded in the custody ledger.

RecoveryIntent does not grant authority and MUST NOT permit signing or key activation without PQSEC approval.

---

### **17.3 Recovery Approvals**

Guardians approve recovery using **RecoveryApproval** artefacts bound to the `recovery_id`.

Rules:

1. Each guardian MAY issue at most one RecoveryApproval per recovery_id.
2. RecoveryApproval artefacts MUST be canonically encoded and signed.
3. Approvals SHOULD be recorded in the custody ledger.
4. Approval presence alone MUST NOT permit recovery.

---

### **17.4 Recovery Activation Conditions**

Recovery MAY proceed only when **PQSEC** determines that all of the following conditions are satisfied:

1. `recovery_delay_elapsed` evaluates to true.
2. `valid_guardian_quorum` evaluates to true.
3. SafeMode restrictions are satisfied when SafeMode is active.
4. All other custody predicates required for the recovery operation class evaluate to true.

If any required predicate is missing, invalid, ambiguous, expired, or fails evaluation, recovery MUST be refused.

---

### **17.5 Recovery Constraints**

1. Recovery MUST be deterministic and auditable.
2. Recovery MUST NOT reduce quorum strength, policy constraints, or custody guarantees without explicit governance approval.
3. Recovery MUST NOT introduce any signing path that bypasses PQSEC EnforcementOutcome gating.
4. Recovery MUST fail closed on ambiguity, missing artefacts, or verification failure.

Recovery exists to restore custody under governance control, not to provide an emergency bypass or degraded authority mode.

---

## **18. Device Lifecycle Management**

### **18.1 Device Enrollment**

Devices participating in custody quorum MUST be explicitly enrolled.

```
DeviceEnrollment = {
  device_id: tstr,
  device_pubkey: bstr,
  device_capabilities: [* tstr],
  enrolled_at_tick: uint,
  enrolled_by: tstr,
  signature: bstr
}
```

Rules:

1. Enrollment MUST be recorded in the custody ledger.
2. Device identity keys MUST be derived deterministically.
3. Enrollment does not grant authority without predicate satisfaction.

---

### **18.2 Device Revocation**

```
DeviceRevocation = {
  device_id: tstr,
  revoked_at_tick: uint,
  reason: tstr,
  revoked_by: tstr,
  signature: bstr
}
```

Rules:

1. Revocation MUST be recorded in the custody ledger.
2. Revoked devices MUST NOT contribute to quorum.
3. Revocation MUST take effect immediately upon enforcement by PQSEC.

---

## **19. Epoch Clock Handling**

1. PQHD MUST consume time artefacts via PQSEC.
2. PQHD MUST NOT transform, re-encode, hash, or reinterpret Epoch Clock artefacts.
3. Epoch Clock ticks MUST be treated as opaque canonical JSON byte sequences.
4. Freshness, monotonicity, and validity are enforced exclusively by PQSEC.

---

## **20. Error Handling**

### **20.1 Error Code Mapping**

PQHD MUST map custody failures to PQSEC error codes, including but not limited to:

* PSBT structure invalid → `E_PSBT_NON_CANONICAL`
* quorum insufficient → `E_QUORUM_INSUFFICIENT`
* ledger divergence → `E_LEDGER_DIVERGENCE`
* device not enrolled → `E_DEVICE_UNENROLLED`
* recovery delay not elapsed → `E_RECOVERY_TOO_EARLY`
* EnforcementOutcome expired → `E_OUTCOME_EXPIRED`
* EnforcementOutcome replay → `E_OUTCOME_REPLAYED`
* SEAL submission evidence missing → `E_SUBMISSION_EVIDENCE_MISSING`
* SEAL submission evidence invalid → `E_SUBMISSION_EVIDENCE_INVALID`
* SEAL execution leak detected → `E_EXECUTION_LEAK_DETECTED`
* SEAL timeout → `E_SEAL_TIMEOUT`
* SEAL replay detected → `E_SEAL_REPLAY_DETECTED`

---

### **20.2 Error Propagation**

PQHD MUST NOT define new error codes.
All refusals MUST use PQSEC error vocabulary.

---

## **21. Dependency Boundaries**

1. PQHD MUST delegate all enforcement decisions to PQSEC.
2. PQHD MUST consume runtime integrity evidence only through PQSEC evaluation.
3. PQHD MUST consume canonical encoding rules via PQSF.
4. PQHD MUST consume temporal authority via Epoch Clock through PQSEC.
5. PQHD MUST NOT perform enforcement, refusal, escalation, lockout, or backoff.

---

### **21.1 Optional Evidence Producers**

PQHD MAY be deployed with additional non-authoritative evidence producers, including operator-state systems.

Such evidence:

* MUST NOT grant authority
* MUST be evaluated exclusively by PQSEC
* MAY be required by policy for specified operation classes

Examples include operator-state and coercion-resistance systems.

---

### **21.2 Enforcement Boundary Discipline**

A PQHD-conformant deployment involves multiple components with distinct responsibilities. No component other than PQSEC may perform enforcement, and no component may exceed its defined responsibility. The following responsibility decomposition is normative.

PQSEC is the enforcement core. PQSEC evaluates predicates, produces EnforcementOutcome artefacts, enforces lockout and backoff, enforces freshness and monotonicity, and enforces ledger continuity. PQSEC MUST NOT construct transactions, derive keys, interact with users, or manage Bitcoin network communication.

The Adapter (or execution bridge) translates between PQHD custody semantics and external systems such as Bitcoin nodes, PSBT coordinators, and hardware signers. The Adapter MUST NOT evaluate predicates, produce enforcement decisions, override or suppress EnforcementOutcome results, cache or reuse EnforcementOutcome artefacts, or perform retry logic for refused operations. The Adapter receives EnforcementOutcome from PQSEC and either executes the authorised operation atomically or surfaces the refusal. There is no intermediate state.

The Coordinator (in M-of-N quorum deployments) distributes PSBTs and collects partial signatures from multiple signers. The Coordinator is untrusted. The Coordinator MUST NOT evaluate predicates, produce enforcement decisions, select which signers participate based on expected outcome, suppress or delay EnforcementOutcome delivery to individual signers, or aggregate signatures prior to each signer independently verifying the EnforcementOutcome per Annex C.

The User Interface (wallet application, CLI, or API consumer) presents custody state and collects user intent. The UI MUST NOT evaluate predicates, produce enforcement decisions, interpret EnforcementOutcome semantics beyond displaying the result to the user, construct ConsentProof artefacts without user interaction, or retry refused operations without fresh user intent. The UI surfaces the enforcement result and collects new user intent when required. The UI does not participate in the enforcement decision.

Any component performing predicate evaluation, enforcement gating, or EnforcementOutcome production outside PQSEC is non-conformant. Any component suppressing, caching, reinterpreting, or overriding an EnforcementOutcome is non-conformant.

---

## **22. Execution Integration**

1. Authorised execution MUST occur only after PQSEC approval.
2. Execution systems MUST NOT influence custody predicate evaluation.
3. Execution MUST consume EnforcementOutcome as the sole authority gate.
4. Execution MUST be atomic: execute or refuse.

---

### **22.1 ZET / ZEB Integration**

* ZET provides execution boundary semantics.
* ZEB provides Bitcoin broadcast and observation profiles.
* ZEB MUST treat EnforcementOutcome as the sole execution gate.
* Decision replay protection MUST be enforced by execution layers.

---

### **22.2 SEAL Integration (Normative)**

SEAL is an optional, policy-defined execution profile for encrypted, direct-to-Submission-Endpoint transaction delivery.

When `execution_profile = "sealed"`, the transaction MUST be submitted via SEAL.

**Rules:**

1. The `intent_hash` in `EnforcementOutcome` MUST bind to the `template_hash` of the sealed transaction.
2. The wallet MUST persist `SubmissionEvidence` and verify it before any state transition.
3. If `SubmissionEvidence` is missing, invalid, or mismatched → `E_SUBMISSION_EVIDENCE_INVALID`
4. If `template_hash` does not match `intent_hash` → `E_PSBT_INTEGRITY_VIOLATION`
5. If the transaction appears in the public mempool before confirmation → `E_EXECUTION_LEAK_DETECTED`
6. Any recovery action (RBF, public broadcast) requires explicit user authorization and constitutes a new execution attempt.

**Authority Boundary:**

SEAL provides execution-layer confidentiality and accountability. It does not grant authority, modify custody predicates, or bypass PQSEC enforcement.

---

## **23. Failure Semantics**

1. Any custody predicate failure MUST result in refusal.
2. Partial authority MUST NOT be granted.
3. No override or fallback is permitted for Authoritative operations.
4. Repeated failures MAY result in PQSEC lockout.
5. If execution fails under `sealed` profile: the wallet MUST transition to `FAILED` state; no automatic retry, resubmit, or public broadcast is permitted; any recovery action (RBF, public broadcast) requires explicit user or policy authorization; the new action constitutes a new execution attempt with fresh `intent_hash`, `submission_id`, and `EnforcementOutcome`.

---

## **24. Bootstrap Trust Anchors**

### **24.1 Initial Trust Establishment**

Implementations MUST be provisioned with:

1. Governance public keys
2. Bootstrap CryptoSuiteProfile
3. Pinned Epoch Clock profile reference

Bootstrap establishes verification capability only.

---

### **24.2 Bootstrap Mode**

While in BOOTSTRAP mode:

1. No Authoritative operations are permitted.
2. Policy mutation and signing are forbidden.
3. Bootstrap exits only after first valid Epoch Tick is accepted.

---

### **24.3 Bootstrap Failure Handling**

Failure MUST result in fail-closed behaviour.
No fallback trust sources are permitted.

---

## **25. Conformance**

An implementation is PQHD conformant if it:

* defines custody predicates exactly as specified
* enforces deterministic key derivation
* enforces canonical PSBT handling
* enforces dual-control signing
* enforces quorum requirements (multi-device; single compromised device insufficient to authorise a spend)
* enforces recovery governance where supported
* delegates all enforcement to PQSEC
* claims only Baseline or Enterprise tier, with all tier requirements satisfied

Implementations that do not satisfy Baseline requirements MUST NOT claim PQHD conformance. There is no reduced, schema-only, or transitional conformance tier. Implementations MAY use PQHD canonical structures for interoperability during migration, but MUST NOT claim PQHD conformance or PQHD Custody until Baseline requirements are satisfied (see Annex N).

---

# **ANNEX A - Conformance Determination (Informative)**

Conformance to PQHD is determined exclusively through:

- Deterministic PSBT construction rules defined in this specification.
- Correct binding and verification of EnforcementOutcome artefacts.
- Proper refusal behaviour enforced by PQSEC (e.g., replay detection, expiry, integrity violations).
- Emission of appropriate refusal codes when custody invariants are violated.

No additional checklist is required. Implementations MUST rely on PQSEC enforcement semantics and canonical encoding rules defined in PQSF to determine correctness.

Custody conformance is behavioural and verifiable, not declarative.

---

# **ANNEX B - Signing Authorisation Flow (Informative)**

This annex illustrates the custody and signing flow. It is explanatory and does not define additional conformance requirements beyond the core specification.

```
1. Construct canonical PSBT
     ↓
2. Derive intent_hash from canonical PSBT bytes
     ↓
3. Assemble custody artefacts:
     - EpochTick bytes (JCS JSON)
     - ConsentProof
     - Policy artefacts
     - Runtime integrity evidence
     - Quorum state
     - Ledger state
     ↓
4. Submit operation + artefacts to PQSEC
     ↓
5. PQSEC evaluates required predicates and produces EnforcementOutcome
     ↓
6. Each signer verifies EnforcementOutcome binding + replay safety
     ↓
7. Each signer signs PSBT input(s) (dual-control)
     ↓
8. Aggregate M-of-N signatures
     ↓
9. If execution_profile = "sealed":
     a. Construct S1 (encrypted submission with template_hash)
     b. Submit to Submission Endpoint
     c. On valid SubmissionEvidence, transition to SUBMITTED
     d. Monitor for confirmation or exposure
     e. On exposure or timeout → FAILED, require explicit recovery
     f. On confirmation → CONFIRMED
10. If execution_profile = "public" or "observed":
     a. Execute via ZET boundary, broadcast via ZEB
```

No signature may be produced unless a valid EnforcementOutcome authorises the attempt.

---

# **ANNEX C - EnforcementOutcome Acceptance Rules (Normative)**

This annex defines the minimum required checks a PQHD signing component MUST apply to an EnforcementOutcome prior to signature emission.

## C.1 Required Fields

A signing component MUST refuse unless all required fields are present:

* `decision`
* `decision_id`
* `operation_id`
* `operation_class`
* `intent_hash`
* `session_id`
* `issued_tick`
* `expiry_tick`

For Authoritative operations, `exporter_hash` MUST be present and non-null.

## C.2 Decision Gate

Signing MUST be refused unless:

* `decision == "ALLOW"`

If `decision == "DENY"` or `decision == "FAIL_CLOSED_LOCKED"`, signing MUST be refused.

## C.3 Binding Requirements

For any signing attempt, the signing component MUST verify:

1. `intent_hash` equals the computed `bundle_hash` for the canonical PSBT associated with the attempt.
2. `session_id` matches the active session identifier for the attempt.
3. If `operation_class == "Authoritative"`, `exporter_hash` matches the active session exporter hash.
4. `issued_tick` and `expiry_tick` are present and `current_tick < expiry_tick`.
5. If `execution_profile = "sealed"`, the signing component MUST verify that `submission_id` is present and unique, `template_hash` matches `intent_hash`, `miner_id` is pinned and trusted, and `expires_at` is enforced. Any mismatch → `E_SUBMISSION_EVIDENCE_INVALID`.

Any mismatch MUST cause refusal.

## C.4 Replay Protection

A signing component MUST enforce single-use of EnforcementOutcome at the decision_id level:

* A given `decision_id` MUST be accepted at most once for signature emission.
* Reuse MUST be refused.

For `sealed` execution profile, a signing component MUST additionally enforce single-use of `submission_id` at the wallet level. Reuse → `E_SEAL_REPLAY_DETECTED`.

Decision replay protection MUST be persistent across process restarts within the signing domain.

### C.4A Replay Store Retention (Normative)

A signing component MAY prune replay records for `decision_id` values whose
corresponding `expiry_tick` is strictly less than the current verified tick.

Pruning MUST NOT remove any `decision_id` that is still within its validity
window.

If replay store state is lost or cannot be proven intact, the signing
component MUST fail closed for Authoritative signing until replay integrity
is re-established (for example by restoring from a trusted backup or
reinitialising the signing domain under fresh policy).

## C.5 Signature Verification

If the EnforcementOutcome includes a `signature` field and the active policy requires it, the signing component MUST verify that signature under the referenced suite profile.

If signature verification is required and fails, signing MUST be refused.

## C.6 Compatibility Mapping

Where an execution boundary expects `allowed: bool`, implementers MAY map:

* `allowed = (decision == "ALLOW")`

This mapping MUST NOT bypass the required checks above.

## C.7 Consolidated Acceptance Algorithm (Informative)

The following pseudocode summarises the required checks from C.1–C.6 as a single verification path:

```python
def verify_enforcement_outcome(outcome, expected_session_id, expected_intent_hash,
                                expected_exporter_hash, current_tick, replay_store,
                                policy_requires_signature):
    # C.1 Required fields
    REQUIRED = ["decision", "decision_id", "operation_id", "operation_class",
                "intent_hash", "session_id", "issued_tick", "expiry_tick"]
    for field in REQUIRED:
        if field not in outcome:
            return refuse("E_OUTCOME_STRUCTURE_INVALID")

    # C.2 Decision gate
    if outcome["decision"] != "ALLOW":
        return refuse("E_OUTCOME_NOT_ALLOW")

    # C.3 Binding requirements
    if outcome["session_id"] != expected_session_id:
        return refuse("E_SESSION_MISMATCH")
    if outcome["intent_hash"] != expected_intent_hash:
        return refuse("E_INTENT_HASH_MISMATCH")
    if current_tick >= outcome["expiry_tick"]:
        return refuse("E_OUTCOME_EXPIRED")
    if outcome["operation_class"] == "Authoritative":
        if outcome.get("exporter_hash") != expected_exporter_hash:
            return refuse("E_EXPORTER_HASH_MISMATCH")

    # C.4 Replay protection
    if outcome["decision_id"] in replay_store:
        return refuse("E_OUTCOME_REPLAYED")
    replay_store.add(outcome["decision_id"])

    # C.5 Signature verification
    if policy_requires_signature:
        if not verify_signature(outcome):
            return refuse("E_OUTCOME_SIGNATURE_INVALID")

    return accept()
```

This algorithm is informative. Implementations MUST satisfy the normative requirements in C.1–C.6 regardless of implementation structure.

---

# **ANNEX D - Deterministic Key Derivation (Reference)**

This annex provides reference pseudocode for deterministic derivation. It is non-authoritative but intended to be implementable.

```python
# Reference pseudocode (SHAKE-only). Matches §10.0--§10.2.

from hashlib import shake_256

def shake256_256(data: bytes) -> bytes:
    return shake_256(data).digest(32)

def derive_root_key(seed: bytes) -> bytes:
    return shake256_256(b"PQHD-RootKey\x00" + seed)

def derive_child_key(root_key: bytes, key_class: str, context_cbor: bytes) -> bytes:
    domain = ("PQHD-Key:" + key_class).encode("utf-8")
    return shake256_256(domain + b"\x00" + context_cbor)
```

---

# **ANNEX E - Bitcoin secp256k1 Scalar Derivation (Reference)**

```python
# Reference pseudocode (SHAKE-only). Matches §11.1.

from hashlib import shake_256

SECP256K1_N = int(
    "FFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFEBAAEDCE6AF48A03BBFD25E8CD0364141", 16
)

def shake256_256(data: bytes) -> bytes:
    return shake_256(data).digest(32)

def i32be(i: int) -> bytes:
    return i.to_bytes(4, "big")

def derive_secp256k1_scalar(root_key: bytes) -> int:
    i = 0
    while True:
        candidate_bytes = shake256_256(b"PQHD-Key:secp256k1\x00" + root_key + i32be(i))
        candidate = int.from_bytes(candidate_bytes, "big")
        if 1 <= candidate < SECP256K1_N:
            return candidate
        i += 1
```

---

# **ANNEX F - Canonical PSBT Hashing and Equivalence (Normative)**

This annex defines how `bundle_hash` (intent_hash for signing) MUST be computed.

## **F.1 Canonical PSBT Bytes**

Implementations MUST define a canonical PSBT byte form for hashing such that:

1. Given semantically identical PSBTs, independent implementations produce byte-identical canonical bytes.
2. Unknown/proprietary fields MUST cause refusal.
3. Canonical bytes MUST be stable across platforms.

Implementations claiming interoperability MUST declare a named PSBT canonicalisation profile or publish a complete canonicalisation rule-set as part of their conformance statement.

If a deployment uses a named profile (for example, a fixed profile in the ecosystem), that profile MAY define the canonical PSBT byte form. If no profile is declared, the deployment MUST document and enforce its canonicalisation rules.

## **F.2 Hash Function**

`bundle_hash` MUST be computed as:

```
bundle_hash = SHAKE256-256(canonical_psbt_bytes)
```

Output length MUST be exactly 32 bytes.

## **F.3 Equivalence Rule**

Two PSBTs are equivalent iff their `bundle_hash` values are byte-identical.

A signer MUST refuse to sign if the PSBT presented differs from the PSBT whose `bundle_hash` is bound into the EnforcementOutcome.

---

# **ANNEX G - Custody Predicate Assembly (Informative)**

PQHD predicate assembly is descriptive. PQSEC performs evaluation.

A typical custody signing attempt submits the following artefact set to PQSEC:

* EpochTick (as JCS canonical JSON bytes)
* ConsentProof
* Policy artefacts
* Runtime integrity evidence
* Quorum state snapshot
* Ledger state snapshot
* canonical PSBT bytes (or their hash)

This annex does not define enforcement semantics.

---

# **ANNEX H - Dual-Control Signer (Reference)**

```python
class DualControlSigner:
    """
    Reference dual-control signer: requires BOTH key material and
    an acceptable EnforcementOutcome per Annex C.
    """
    def __init__(self, signing_key):
        self.signing_key = signing_key
        self.used_decision_ids = set()

    def sign(self, psbt, input_index, outcome, current_tick):
        if not self._accept_outcome(psbt, outcome, current_tick):
            return None
        sig = self._produce_signature(psbt, input_index)
        self.used_decision_ids.add(outcome["decision_id"])
        return sig

    def _accept_outcome(self, psbt, outcome, current_tick):
        # Minimal checks; real implementations MUST implement Annex C fully.
        if outcome.get("decision") != "ALLOW":
            return False
        if current_tick >= outcome.get("expiry_tick", 0):
            return False
        if outcome.get("decision_id") in self.used_decision_ids:
            return False
        if outcome.get("intent_hash") != psbt.bundle_hash:
            return False
        return True

    def _produce_signature(self, psbt, input_index):
        return b"signature_bytes"
```

---

# **ANNEX I - Untrusted M-of-N Coordinator (Reference)**

```python
class QuorumCoordinator:
    """
    Coordinator is untrusted. Signers enforce Annex C independently.
    """
    def __init__(self, m: int, n: int):
        self.m = m
        self.n = n
        self.signers = []

    def add_signer(self, signer):
        if len(self.signers) >= self.n:
            raise ValueError("Too many signers")
        self.signers.append(signer)

    def collect(self, psbt, outcome, current_tick):
        sigs = []
        for signer in self.signers:
            sig = signer.sign(psbt, 0, outcome, current_tick)
            if sig is not None:
                sigs.append(sig)
            if len(sigs) >= self.m:
                return True, sigs
        return False, sigs
```

---

# **ANNEX J - Custody Lifecycle Extensions (Delegation / Guardian Recovery / SafeMode) (Normative)**

This annex defines artefact schemas used to restrict or condition custody authority. These artefacts grant no authority by themselves. Enforcement is performed exclusively by PQSEC.

## J.1 DelegationConstraint

```
DelegationConstraint = {
  delegation_id: tstr,
  delegator_id: tstr,
  delegate_id: tstr,
  scope: [* tstr],
  issued_tick: uint,
  expiry_tick: uint,
  revocation_ref: tstr / null,
  suite_profile: tstr,
  signature: bstr
}
```

Rules:

1. DelegationConstraint MUST be canonically encoded and signed.
2. DelegationConstraint MUST be time-bounded.
3. DelegationConstraint MUST be revocable via `revocation_ref` where used.
4. DelegationConstraint MUST NOT outlive delegator authority.
5. DelegationConstraint MUST NOT bypass quorum, SafeMode, recovery, consent, policy, runtime, or ledger predicates.
6. Absence or failure of a required DelegationConstraint MUST result in refusal.

Enforcement predicate: `valid_delegation` (PQSEC).

## J.2 GuardianSet

```
GuardianSet = {
  guardian_set_id: tstr,
  guardians: [* tstr],
  threshold: uint,
  issued_tick: uint,
  suite_profile: tstr,
  signature: bstr
}
```

Rules:

1. `threshold` MUST be <= number of guardians.
2. GuardianSet updates SHOULD be ledger-recorded.
3. Signature MUST verify under suite_profile.

## J.3 RecoveryIntent

```
RecoveryIntent = {
  recovery_id: tstr,
  guardian_set_id: tstr,
  requested_by: tstr,
  issued_tick: uint,
  expiry_tick: uint,
  activation_delay: uint,
  suite_profile: tstr,
  signature: bstr
}
```

Rules:

1. RecoveryIntent MUST be explicit and ledger-recorded.
2. `activation_delay` MUST be enforced using verified ticks.
3. Signature MUST verify under suite_profile.
4. `expiry_tick` MUST be present. Consuming enforcement specifications MUST treat RecoveryIntent as invalid when `current_tick >= expiry_tick`.

**Predicate Definition (Informative, Enforcement-Targeted)**

Consuming enforcement specifications evaluate `recovery_delay_elapsed` as:

```
recovery_delay_elapsed =
  current_tick >= (recovery_intent.issued_tick + recovery_intent.activation_delay)
```

If `recovery_delay_elapsed` is FALSE, recovery activation MUST be refused.

## J.4 RecoveryApproval

```
RecoveryApproval = {
  recovery_id: tstr,
  guardian_id: tstr,
  approved_tick: uint,
  expiry_tick: uint,
  suite_profile: tstr,
  signature: bstr
}
```

Rules:

1. Each guardian MAY approve at most once per recovery_id.
2. Approvals SHOULD be ledger-recorded.
3. Signature MUST verify under suite_profile.
4. `expiry_tick` MUST be present. Consuming enforcement specifications MUST treat RecoveryApproval as invalid when `current_tick >= expiry_tick`.

Enforcement predicates (PQSEC):

* `valid_guardian_quorum`
* `recovery_delay_elapsed`

## J.5 SafeModeState

```
SafeModeState = {
  state: "ACTIVE" / "INACTIVE",
  activated_by: tstr,
  activated_tick: uint,
  suite_profile: tstr,
  signature: bstr
}
```

Rules:

1. SafeMode activation MUST be explicit and ledger-recorded.
2. When SafeMode is ACTIVE, irreversible operations MUST be refused as defined by policy and enforced by PQSEC.
3. Signature MUST verify under suite_profile.

Enforcement predicate: `safe_mode_active` (PQSEC).

## J.6 RecoveryCancellation

```
RecoveryCancellation = {
  recovery_id: tstr,
  cancelled_by: tstr,
  cancelled_tick: uint,
  suite_profile: tstr,
  signature: bstr
}
```

Rules:

1. RecoveryCancellation MUST be canonically encoded and signed.
2. When a valid RecoveryCancellation exists for `recovery_id`, consuming enforcement specifications MUST evaluate recovery as invalid regardless of delay or approvals.
3. Cancellation grants no authority and does not enable recovery.
4. RecoveryCancellation SHOULD be ledger-recorded.

---

## J.7 DelegationConstraint Scope Vocabulary (Normative)

DelegationConstraint.scope tokens MUST be stable, lowercase ASCII tokens
with no whitespace, no path separators, and no quotes.

### J.7.1 Token Format

Scope tokens follow the format: `<domain>:<resource>:<action>`

Where:
- `<domain>` is the governance domain
- `<resource>` is the specific resource or wildcard
- `<action>` is the permitted action

### J.7.2 Reserved Scope Tokens

| Token | Meaning |
|-------|---------|
| `custody:btc:spend` | May initiate Bitcoin spend operations |
| `custody:btc:receive` | May generate receive addresses |
| `custody:*:view` | May view custody state (read-only) |
| `tool:<tool_id>:invoke` | May invoke the specified tool |
| `tool:*:invoke` | May invoke any tool in the Tool Capability Profile |
| `gateway:<service_id>:call` | May use gateway adapter for the specified service |
| `gateway:*:call` | May use any registered gateway adapter |
| `session:*:create` | May create new sessions (within delegation bounds) |
| `session:*:resume` | May resume existing sessions |

### J.7.3 Scope Token Rules

1. The wildcard `*` applies to the resource segment only. It MUST NOT appear in domain or action segments.
2. Unknown scope tokens MUST be refused.
3. The effective authority surface is the intersection of:
   - DelegationConstraint.scope tokens, and
   - the active Tool Capability Profile.
   A scope token has no effect if the Tool Capability Profile does not permit the corresponding tool_id or gateway action.
4. Scope tokens are evidence, not authority. PQSEC evaluates them alongside all other predicates.
5. Scope token invalidity MUST be refused using an AE-registered refusal code. Default mapping: `E_DELEGATION_INVALID`.

### J.7.4 Authority Boundary

This section defines a vocabulary for scope tokens. It does not grant authority, modify enforcement semantics, or create new predicate types. All enforcement remains exclusively within PQSEC.

---

# **ANNEX K - Recovery Activation Sequence (Informative)**

```
1. Create GuardianSet
2. Create RecoveryIntent (ledger-recorded)
3. Wait activation_delay
4. Collect RecoveryApproval artefacts
5. PQSEC evaluates:
   - recovery_delay_elapsed
   - valid_guardian_quorum
   - custody predicates for recovery operation class
6. PQSEC issues EnforcementOutcome
7. Recovery signing may proceed under dual-control
```

---

# **ANNEX L - Device Enrollment and Revocation (Informative)**

Devices MUST be enrolled to participate in quorum. Revocation removes eligibility.

Recommended ledger events:

* device_enrolled
* device_revoked

Enrollment and revocation are Authoritative operations and therefore require PQSEC EnforcementOutcome.

---

# **ANNEX M - Custody Tier Qualification Matrix (Informative)**

| Requirement                  | Baseline | Enterprise |
| ---------------------------- | -------- | ---------- |
| Multi-device quorum          | REQUIRED | REQUIRED   |
| Dual-control signing         | REQUIRED | REQUIRED   |
| Ledger continuity            | REQUIRED | REQUIRED   |
| Recovery governance          | OPTIONAL | REQUIRED   |
| Quorum diversity constraints | OPTIONAL | REQUIRED   |

Only Baseline and Enterprise tiers exist. There is no schema-only or reduced tier. Implementations that do not satisfy Baseline requirements MUST NOT claim PQHD conformance (see Annex N for migration guidance).

---

# **ANNEX N - Migration Guidance (Informative)**

## N.1 Purpose

This annex provides guidance for implementations migrating from key-only Bitcoin wallets toward PQHD Baseline Custody conformance. Migration is expected to be incremental. Implementations that have not yet reached Baseline conformance MUST NOT claim PQHD conformance but MAY use PQHD canonical structures (key derivation, PSBT canonicalisation, EnforcementOutcome schemas) for interoperability purposes during the migration period.

## N.2 Phased Migration Path

Migration from key-only wallets SHOULD proceed in phases. Each phase adds a conformance requirement without regressing prior phases. An implementation MAY ship intermediate phases to production provided it does not claim PQHD Custody.

Phase 1: Canonical PSBT validation and hashing.
Implement PSBT v2 structural requirements (Section 12.1). Enforce deterministic canonicalisation (Section 12.2). Compute bundle_hash using SHAKE256-256 (Section 27). Reject unknown or proprietary PSBT fields. At the end of this phase, the implementation can produce deterministic PSBT hashes that agree with any PQHD-conformant signer.

Phase 2: Dual-control signing gate.
Integrate PQSEC evaluation and require a valid EnforcementOutcome before producing any signature (Section 13). Implement Annex C acceptance rules in full, including decision gate, binding verification, expiry enforcement, and single-use replay protection. At the end of this phase, the implementation cannot produce a signature without external enforcement authorisation.

Phase 3: Multi-device quorum.
Deploy at least two independent signing devices. Implement M-of-N quorum collection (Section 15). Ensure each signer independently verifies the EnforcementOutcome per Annex C. No single compromised device may authorise a spend. At the end of this phase, the implementation satisfies the core Baseline requirement that a single compromised device MUST be insufficient to authorise a spend.

Phase 4: Ledger continuity.
Implement the deterministic, append-only, tamper-evident ledger required by Section 16. Record all custody-relevant events. Enforce monotonic state continuity as a predicate input to PQSEC. At the end of this phase, the implementation can detect state rollback, omission, and replay at the custody layer.

Phase 5: Recovery governance (Enterprise only).
Implement GuardianSet, RecoveryIntent, RecoveryApproval schemas (Section 17, Annex J). Enforce quorum-gated recovery with deterministic delay. This phase is REQUIRED only for Enterprise Custody and OPTIONAL for Baseline.

## N.3 Interoperability Without Conformance

Implementations that use PQHD canonical structures (key derivation paths, PSBT hashing, EnforcementOutcome schemas) without satisfying Baseline requirements operate outside the PQHD conformance boundary. Such implementations:

1. MUST NOT describe themselves as PQHD-conformant.
2. MUST NOT describe themselves as providing PQHD Custody.
3. MUST NOT use the term "PQHD" in user-facing product descriptions, tier badges, or compliance claims.
4. MAY reference PQHD in technical documentation to describe schema compatibility (for example, "uses PQHD-compatible PSBT canonicalisation").

This distinction exists because PQHD Custody is a security claim. The claim has meaning only when the full predicate set, including multi-device quorum, is satisfied. A single-device deployment that uses PQHD schemas is not PQHD Custody and MUST NOT be represented as such.

## N.4 Migration Completion

Migration is complete when the implementation can demonstrate all normative requirements defined in this specification for the target tier. Baseline conformance requires Phases 1 through 4. Enterprise conformance requires Phases 1 through 5.

---

# **ANNEX O - Signing Attempt Record (Informative)**

Recommended ledger payload fields for a signing attempt:

* decision_id
* bundle_hash (intent_hash)
* operation_class
* signer_ids (or count)
* outcome decision
* refusal error_code (if refused)

---

# **ANNEX P - Troubleshooting (Informative)**

**Signing refused**

* Check EnforcementOutcome decision, expiry_tick, and binding to bundle_hash/session/exporter.

**Quorum not met**

* Confirm signer availability and enrollment status.

**Recovery not activating**

* Confirm delay elapsed and guardian approvals meet threshold.

---

# **ANNEX Q - Security Checklist (Informative)**

* ☐ PSBT canonicalisation enforced; unknown fields refused
* ☐ bundle_hash computed deterministically (SHAKE256-256, 32 bytes)
* ☐ EnforcementOutcome required for signing; decision_id replay-protected
* ☐ exporter_hash required and checked for Authoritative operations
* ☐ Multi-device quorum enforced; single compromised device insufficient to authorise a spend
* ☐ Ledger continuity enforced
* ☐ Recovery is delayed and quorum-gated (Enterprise)
* ☐ No degraded, override, or single-device signing path exists
* ☐ No schema-only or reduced tier claimed as PQHD conformance
* ☐ (If sealed profile) Refusal when `SubmissionEvidence` is missing or invalid
* ☐ (If sealed profile) Enforcement of `submission_id` replay protection
* ☐ (If sealed profile) Detection and refusal on public mempool exposure
* ☐ (If sealed profile) Explicit authorization required for recovery actions

---

# ANNEX R - Payment Endpoint Keys (Normative)

## R.1 Scope

This annex defines Payment Endpoint Keys (PEK) as a joint PQHD + PQSEC extension
used to qualify regulated payment endpoint operations.

A PaymentEndpointKey is NOT custody authority. It does not permit signing,
does not bypass custody predicates, and does not change PQHD custody tier
requirements.

PEK provides a jurisdiction-scoped, revocable eligibility signal that MAY be
required by PQSEC policy for specified regulated operation classes.

## R.2 PaymentEndpointKey Structure

PaymentEndpointKey = {
  pek_id: tstr,
  jurisdiction: tstr,
  scope: [* tstr],
  issued_tick: uint,
  expiry_tick: uint,
  suite_profile: tstr,
  signature: bstr
}

## R.3 Field Semantics

- pek_id
  Unique identifier for this PaymentEndpointKey instance.

- jurisdiction
  A jurisdiction identifier. Interpretation is defined by consuming policy
  (for example ISO-3166-1 alpha-2, legal region code, or a policy namespace).

- scope
  A bounded list of tokens defining what regulated endpoint operations this
  key applies to (for example "payments:endpoint:create", "payments:endpoint:update",
  "payments:endpoint:activate", "payments:endpoint:deactivate").

- issued_tick, expiry_tick
  Verifiable time bounds. Freshness and validity are enforced by PQSEC using
  Epoch Clock ticks.

- suite_profile, signature
  Cryptographic suite reference and signature over the canonical payload with
  signature omitted.

## R.4 Canonical Encoding Requirements

1. PaymentEndpointKey MUST be canonically encoded under PQSF deterministic CBOR rules.
2. signature MUST be computed over the canonical CBOR encoding with signature omitted.
3. Re-encoding a decoded PaymentEndpointKey MUST produce byte-identical output.
4. Non-canonical encodings MUST be rejected.

## R.5 Rules

1. PEK is NOT custody authority.
   - Presence of a valid PaymentEndpointKey MUST NOT satisfy any PQHD custody predicate.
   - PEK MUST NOT be treated as permission to sign or spend Bitcoin.

2. PEK MUST be jurisdiction-scoped.
   - PaymentEndpointKey.jurisdiction MUST be present and MUST be evaluated by PQSEC
     against the operation jurisdiction context when required by policy.

3. PEK MUST be revocable.
   - A deployment claiming support for PaymentEndpointKey MUST define an authenticated
     revocation mechanism enforced by PQSEC (for example signed revocation artefacts
     or a signed denylist).
   - Revocation discovery MUST cause valid_payment_endpoint to evaluate to false for
     affected pek_id values.

## R.6 Authority Boundary

1. PaymentEndpointKey MUST NOT grant authority, admit operations, or bypass refusal.
2. Failure to provide a valid PaymentEndpointKey MUST NOT affect non-regulated custody
   operations unless required by PQSEC policy for the given operation class.
3. All enforcement decisions remain exclusively defined and executed by PQSEC.

## R.7 PQSEC Predicate Hook

Consuming enforcement specifications (PQSEC) MAY define and require a predicate
named `valid_payment_endpoint` for regulated operation classes.

Failure of `valid_payment_endpoint` MUST result in refusal for the affected
regulated operations. This annex defines no enforcement behaviour.

---

## **26. Transcript-Bound Signing**

If policy requires transcript binding for signing operations:

**Required receipts:**

1. A `pqsf.message` with:
   - `class="EXECUTE"`
   - `sid` matching the session
   - `action_id` for this signing operation
   - `psbt_hash` in the payload or refs

2. A `pqsf.transcript_commitment` with:
   - Same `sid`
   - Same `action_id` (if `scope="ACTION"`)
   - Valid signature
   - Not expired

**Refusal conditions:**

| Condition | Refusal Code |
|-----------|--------------|
| `pqsf.message` missing | `E_TRANSCRIPT_BINDING_MISSING` |
| `pqsf.transcript_commitment` missing | `E_TRANSCRIPT_BINDING_MISSING` |
| Invalid receipts | `E_TRANSCRIPT_BINDING_INVALID` |
| `sid` mismatch | `E_TRANSCRIPT_BINDING_INVALID` |
| `action_id` mismatch | `E_TRANSCRIPT_BINDING_INVALID` |
| Supervision evidence missing | `E_SUPERVISION_REQUIRED` |

---

## **27. Canonical PSBT Hash**

PQHD uses a canonical PSBT hash for intent binding and equivalence checking.

**Hash computation:**

```
psbt_hash = bundle_hash = SHAKE256-256(canonical_psbt_bytes)
```

Output length MUST be exactly 32 bytes.

**Canonicalization procedure:**

1. Serialise the PSBT in BIP-174 format.
2. Within each map (global, per-input, per-output):
   - Sort key-value pairs lexicographically by key bytes
3. Forbid duplicate keys (reject if found)
4. Exclude unknown proprietary keys (key types 0xFC and above unless explicitly permitted)
5. Require `witness_utxo` per input. `non_witness_utxo` MUST NOT be present
6. Require `scriptPubKey` and `amount` per output

For complete specification of canonical PSBT hashing and equivalence rules, see **Annex F**.

Refusal code: `E_PSBT_NON_CANONICAL`

---

# ANNEX S - State-Transition and Execution Integration (Mixed: Normative + Informative)

### S.1 State-Transition Classification (Normative)

All custody operations MUST declare a `state_mutation_class`:

| Operation | Classification |
|-----------|----------------|
| Balance query | `read_only` |
| Address derivation | `read_only` |
| PSBT construction | `low_risk_mutation` |
| PSBT signing | `high_risk_mutation` |
| Transaction broadcast | `high_risk_mutation` |
| Policy update | `authority_mutation` |
| Freeze/unfreeze | `authority_mutation` |

**PQSEC Annex AA integration:**

- `authority_mutation` operations are evaluated under PQSEC state-transition safety rules (PQSEC Annex AA)
- Continuity predicates and authority mutation constraints are enforced by PQSEC

### S.2 Execution Profile Selection (Informative)

PQHD defines custody authority only. How transactions are actually submitted to the network is an execution concern.

**Profile options:**

| Profile | Description | When to use |
|---------|-------------|-------------|
| `public` | Standard mempool broadcast | Low-value, non-sensitive |
| `observed` | Broadcast with monitoring | Medium-value |
| `sealed` | Encrypted submission via SEAL | High-value, privacy-sensitive, quantum-aware |

`sealed` profile requires `SubmissionEvidence` validation and explicit recovery authorization on failure.

**Important:** SEAL is recommended but NEVER required. PQHD conformance does not require SEAL availability.

### S.3 Profile-Governed Execution (Normative)

If policy requires execution profile governance:

1. The execution profile MUST be declared before signing.
2. PQSEC Annex AC (Execution Profile Enforcement) compliance is required when execution profile governance is enabled by policy.
3. Profile evidence MUST be available for audit.
4. This does NOT change custody predicates-signing authority remains unchanged.

### S.4 Execution-Layer Protection (Informative)

PQHD defines custody authority only. Transaction execution and broadcast protection are defined by separate specifications:

**Recommended execution profile:**
- **SEAL** - Execution-layer mitigation providing encrypted transaction submission and mempool exposure elimination

Execution profile selection does NOT change PQHD custody predicates. Authorisation semantics remain identical regardless of broadcast method.

**PQHD does not consume execution evidence and does not evaluate execution outcomes; all execution-related evidence is evaluated exclusively by PQSEC.**

For pre-contract authorization patterns, see the BPC specification. PQHD Annex R defines Payment Endpoint Keys only.

---

# ANNEX T - Recipient Whitelisting (Optional, Normative)

PQHD defines the structures and required checks only. All refusal, override, lockout, and enforcement decisions are evaluated exclusively by PQSEC. Any "refuse" language in this annex describes PQSEC predicate outcomes, not behaviour performed by PQHD.

### T.1 Purpose

Prevent accidental or adversarial transfers to unapproved destinations by maintaining a recipient allowlist.

### T.2 Definitions

| Term | Definition |
|------|------------|
| Recipient Identity | Canonical identifier derived from address, script, or descriptor |
| Allowlisted | Approved for transfers in local store |
| Unknown | Not in allowlist |
| Override Path | Policy mechanism permitting transfer to unknown recipient |

### T.3 Recipient Canonicalisation (Normative)

For each output (excluding change), derive:

1. `address_string` - the bech32/bech32m/base58 address
2. `script_pubkey_hash = SHAKE256-256(scriptPubKey_bytes)`
3. `descriptor_hash = SHAKE256-256(output_descriptor)` if applicable

**Binding:**
- Include `network` (mainnet, testnet, signet)
- Include `output_type` (p2wpkh, p2tr, etc.)
- Exclude change outputs from recipient checks

### T.4 AllowlistEntry Structure

**AllowlistEntry (local storage, deterministic CBOR map):**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `v` | uint | Yes | Schema version |
| `recipient_id` | bstr (32 bytes) | Yes | `SHAKE256-256(CBOR_ENCODE_DETERMINISTIC({ "address": address_string, "network": network }))` |
| `identity_type` | tstr | Yes | `"address"` or `"descriptor"` |
| `network` | tstr | Yes | `"mainnet"`, `"testnet"`, `"signet"` |
| `label` | tstr | Yes | Human-readable label |
| `created_at` | uint | Yes | When entry was created |
| `limits` | map | No | Per-recipient limits |
| `supervision` | tstr | Yes | Minimum supervision for this recipient |
| `notes` | tstr | No | Optional notes |

**Limits map:**

| Field | Type | Description |
|-------|------|-------------|
| `max_per_tx_sats` | uint | Maximum per transaction |
| `max_daily_sats` | uint | Maximum per 24-hour period |
| `max_pending_txs` | uint | Maximum unconfirmed transactions |

**Storage:** AllowlistEntry is local only. It MUST NOT be broadcast or shared without explicit user consent.

### T.5 Policy Modes

| Mode | Behaviour |
|------|-----------|
| `ALLOWLIST_ONLY` | Unknown recipients → fail closed |
| `ALLOWLIST_PREFERRED` | Unknown recipients → require override |
| `OFF` | Allowlist disabled (not recommended) |

### T.6 Predicate Inputs (Normative)

Before signing any transaction, the following predicate inputs MUST be assembled:

1. Canonicalise all non-change outputs per T.3.
2. For each canonicalised recipient:
   - If in allowlist: include limits and supervision evidence
   - If not in allowlist: include policy mode evidence
3. Include limit compliance evidence (if matched)
4. Include supervision evidence

### T.7 Override Paths

For `ALLOWLIST_PREFERRED` mode, overrides are permitted via:

| Override Type | Requirements |
|---------------|--------------|
| Human approval | `HUMAN_APPROVE` bound to `(sid, action_id, recipient_id, amount)` |
| Time delay | Configurable delay period before execution |
| Amount limit | Below configurable threshold |

### T.8 PQSEC Integration

| Refusal Code | Condition |
|--------------|-----------|
| `E_RECIPIENT_NOT_ALLOWLISTED` | Recipient unknown and mode is `ALLOWLIST_ONLY` |
| `E_RECIPIENT_LIMIT_EXCEEDED` | Transaction exceeds recipient limits |
| `E_RECIPIENT_SUPERVISION_REQUIRED` | Required supervision not satisfied |

### T.9 UTXO Exposure Discipline Note (Informative)

Recipient whitelisting integrates with UTXO exposure:

| Label | Meaning |
|-------|---------|
| `CLEAN` | No exposure concerns |
| `UNKNOWN` | Exposure status unknown |
| `RESTRICTED` | Known exposure, restricted use |

## T.X Global Agent Spending Budget (Normative)

Recipient-specific ceilings (T.5–T.8) are insufficient for autonomous agents. Delegations governing agents MAY include a global spend ceiling that applies across all recipients.

### T.X.1 Budget Fields (Optional)

A DelegationConstraint MAY include:

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `max_total_sats` | uint / null | No | Maximum total spend for delegation lifetime |
| `max_per_tick_sats` | uint / null | No | Maximum spend per Epoch Clock tick |
| `max_per_tx_sats` | uint / null | No | Maximum spend per single transaction |
| `max_total_sats_window_ticks` | uint / null | No | Rolling window in ticks for total budget (null = lifetime) |
| `heartbeat_tick_window` | uint / null | No | Maximum allowed tick drift without fresh verification |

### T.X.2 Enforcement

1. Budget counters are enforcement state and MUST be evaluated by PQSEC.
2. If `spent_total + proposed_amount > max_total_sats`, the operation MUST be refused.
3. If aggregate spend within the current tick exceeds `max_per_tick_sats`, the operation MUST be refused.
4. If `heartbeat_tick_window` is defined and `current_tick - last_verified_tick` exceeds this window, Authoritative spend MUST fail closed.
5. If `proposed_amount > max_per_tx_sats`, the operation MUST be refused.
6. If the budget enforcement state is unavailable or cannot be proven intact, Authoritative spend operations MUST fail closed.
7. Budget state MUST persist across sessions and power cycles.
8. Budget state MUST NOT be resettable by the agent. Budget reset requires a new DelegationConstraint with holder authorization.
9. Budget exhaustion MUST NOT trigger automatic delegation renewal.

Budget exhaustion MUST be refused by PQSEC using existing AE-registered refusal codes (policy constraint failure) unless dedicated codes are registered in Annex AE.

### T.X.3 Budget Override

The holder MAY override budget limits via holder-authorized consent bound to the specific transaction. Budget overrides are one-time and do not modify the budget constraints.

### T.X.4 Authority Boundary

This section defines spending budget fields and enforcement rules. It does not grant authority. All enforcement remains exclusively within PQSEC.

---

# ANNEX U - Fee, Change, and Output Manifest Policy (Optional, Normative)

PQHD defines the structures and required checks only. All refusal, override, lockout, and enforcement decisions are evaluated exclusively by PQSEC. Any "refuse" language in this annex describes PQSEC predicate outcomes, not behaviour performed by PQHD.

### U.1 Purpose

Prevent fee manipulation, malicious change outputs, and hidden output injection by enforcing explicit bounds and manifests.

### U.2 Fee Policy Bounds (Normative)

**Fee policy parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `max_fee_rate` | uint | Maximum fee rate in sat/vB |
| `max_absolute_fee` | uint | Maximum total fee in satoshis |
| `min_fee_rate` | uint | Minimum fee rate (for relay) |
| `rbf_allowed` | bool | Whether RBF is permitted |

**Enforcement (via PQSEC):**

1. Compute actual fee rate and absolute fee
2. `valid_fee_policy` MUST evaluate to FALSE if `fee_rate > max_fee_rate`
3. `valid_fee_policy` MUST evaluate to FALSE if `absolute_fee > max_absolute_fee`
4. `valid_fee_policy` SHOULD evaluate to FALSE if `fee_rate < min_fee_rate` (warning, may fail relay)
5. `valid_fee_policy` MUST evaluate to FALSE if RBF signaled when `rbf_allowed = false`

**Override:** Human approval (`HUMAN_APPROVE`) MAY override fee bounds, bound to `(sid, action_id, fee_rate, absolute_fee)`.

### U.3 Change Output Validation (Normative)

Change outputs are identified by:
- Matching wallet descriptor
- Matching scriptPubKey pattern

**Constraints:**

| Constraint | Requirement |
|------------|-------------|
| Script type | MUST match policy-permitted types |
| Address reuse | MUST NOT reuse addresses (see Annex W) |
| Dust | Amount MUST be ≥ dust threshold |
| Count | MUST be ≤ policy maximum (default: 1) |

### U.4 Output Manifest (Normative)

Before signing, construct an output manifest:

**OutputManifest (deterministic CBOR map):**

| Field | Type | Description |
|-------|------|-------------|
| `v` | uint | Schema version |
| `outputs` | array of OutputEntry | Ordered list of outputs |
| `total_output_sats` | uint | Sum of all output amounts |
| `fee_sats` | uint | Transaction fee |

**OutputEntry (deterministic CBOR map):**

| Field | Type | Description |
|-------|------|-------------|
| `index` | uint | Output index in transaction |
| `classification` | tstr | `"recipient"`, `"change"`, `"op_return"` |
| `amount_sats` | uint | Output amount |
| `destination` | tstr | Address or descriptor |
| `script_type` | tstr | Output script type |

**Integrity:**

1. Compute `output_manifest_hash = SHAKE256-256(canonical_manifest_bytes)`
2. Bind signing approval to `output_manifest_hash`
3. At signing time, recompute and verify match
4. Mismatch → `valid_output_manifest` MUST evaluate to FALSE

**Multi-recipient requirement:**

Transactions with multiple non-change recipients MUST require `HUMAN_APPROVE` bound to the full `output_manifest_hash`.

### U.5 Refusal Codes

| Code | Condition |
|------|-----------|
| `E_FEE_POLICY_VIOLATION` | Fee exceeds bounds |
| `E_CHANGE_OUTPUT_INVALID` | Change output fails validation |
| `E_OUTPUT_MANIFEST_MISMATCH` | Manifest doesn't match transaction |
| `E_RBF_PROHIBITED` | RBF signaled when not allowed |

---

# ANNEX V - PSBT Integrity Binding (Optional, Normative)

PQHD defines the structures and required checks only. All refusal, override, lockout, and enforcement decisions are evaluated exclusively by PQSEC. Any "refuse" language in this annex describes PQSEC predicate outcomes, not behaviour performed by PQHD.

### V.1 Purpose

Prevent PSBT modification between construction and signing, ensuring the transaction signed is exactly the transaction approved.

### V.2 Commitment (Normative)

After PSBT construction and before approval:

```
psbt_commitment = SHAKE256-256(canonical_psbt_bytes)
```

Store `psbt_commitment` in session state, bound to `(sid, action_id)`.

### V.3 Signing-Time Verification (Normative)

At signing time:

1. Recompute `SHAKE256-256(canonical_psbt_bytes)`
2. Compare against stored `psbt_commitment`
3. If mismatch → `valid_psbt_integrity` MUST evaluate to FALSE

Refusal code: `E_PSBT_INTEGRITY_VIOLATION`

### V.4 Metadata Binding (Normative)

The commitment MUST cover:

| Element | Requirement |
|---------|-------------|
| Input amounts | Bound in PSBT |
| Derivation paths | Bound per input |
| Sighash flags | Bound per input |
| Output scripts | Bound |
| Output amounts | Bound |

Any modification to these elements after commitment MUST fail closed.

### V.5 Partial Signature Protection (Normative)

If partial signatures are collected:

1. Each partial signature is bound to the committed `psbt_hash`
2. Partial signatures MUST NOT be reusable on modified PSBTs
3. PSBT modification invalidates all prior partial signatures

### V.6 Refusal Codes

| Code | Condition |
|------|-----------|
| `E_PSBT_INTEGRITY_VIOLATION` | PSBT modified since commitment |
| `E_PSBT_METADATA_TAMPERED` | Specific metadata field changed |

---

# ANNEX W - Address Reuse Prohibition (Optional, Normative)

PQHD defines the structures and required checks only. All refusal, override, lockout, and enforcement decisions are evaluated exclusively by PQSEC. Any "refuse" language in this annex describes PQSEC predicate outcomes, not behaviour performed by PQHD.

### W.1 Purpose

Prevent address reuse for quantum safety and privacy.

### W.2 Reuse Detection (Normative)

Maintain records of:
- Used addresses (by string)
- Used scriptPubKeys (by hash)
- Used descriptor instances

### W.3 Rules (Normative)

| Context | Rule |
|---------|------|
| Recipient reuse | `valid_address_reuse` MUST evaluate to FALSE unless override granted |
| Change reuse | NEVER permitted, no override |
| Override binding | `(sid, action_id, reused_address, reason)` |

### W.4 Quantum-Aware Mode (Normative)

When `quantum_aware_mode = true`:

- Address reuse is prohibited entirely
- No overrides are permitted
- Change to previously-used addresses is blocked
- Receiving to previously-used addresses triggers warning

### W.5 Refusal Codes

| Code | Condition |
|------|-----------|
| `E_ADDRESS_REUSE_DETECTED` | Reuse found, override available |
| `E_ADDRESS_REUSE_PROHIBITED` | Reuse found, no override (quantum mode) |

---

# ANNEX X - Secure Session Resumption (Optional, Normative)

PQHD defines the structures and required checks only. All refusal, override, lockout, and enforcement decisions are evaluated exclusively by PQSEC. Any "refuse" language in this annex describes PQSEC predicate outcomes, not behaviour performed by PQHD.

### X.1 Purpose

Allow session resumption without restoring authority.

### X.2 Resume Token (Normative)

```
resume_token = SHAKE256-256(sid || exporter_hash || last_ctr || policy_profile_hash || expires_at)
```

| Field | Description |
|-------|-------------|
| `sid` | Original session identifier |
| `exporter_hash` | Original transport exporter |
| `last_ctr` | Last message counter |
| `policy_profile_hash` | Policy at session end |
| `expires_at` | Resume expiry |

### X.3 Resumption Rules (Normative)

1. Token MUST verify (recompute and compare)
2. New `pqsf.session_scope` MUST be issued
3. `ctr` MUST increment from `last_ctr`
4. Supervision requirements MUST be re-evaluated

### X.4 Prohibited Restoration (Normative)

Resumption MUST NOT restore:

| Prohibited | Reason |
|------------|--------|
| Prior approvals | Expired with session |
| Prior transcripts | New session = new transcript |
| Prior authority | Authority is not persistent |
| Prior tool access | Must be re-established |

### X.5 Refusal Code

| Code | Condition |
|------|-----------|
| `E_SESSION_RESUMPTION_INVALID` | Token invalid or rules violated |

---

# ANNEX Y - UTXO Exposure Discipline (Optional, Normative)

PQHD defines the structures and required checks only. All refusal, override, lockout, and enforcement decisions are evaluated exclusively by PQSEC. Any "refuse" language in this annex describes PQSEC predicate outcomes, not behaviour performed by PQHD.

### Y.1 Purpose

Manage UTXO exposure risk without mandating specific algorithms.

### Y.2 UTXO Classification (Normative)

| Classification | Description |
|----------------|-------------|
| `DORMANT` | Never spent from, public key not exposed |
| `ACTIVE` | Recently used, normal exposure |
| `RESTRICTED` | Known exposure concerns |
| `QUARANTINE` | Do not use without explicit approval |

Classification is local only.

### Y.3 Selection Constraints (Normative)

| Constraint | Rule |
|------------|------|
| `DORMANT` selection | Requires `HUMAN_APPROVE` |
| `RESTRICTED` to unknown recipient | Not permitted |
| `QUARANTINE` mixing | Not permitted |
| `DORMANT` + `ACTIVE` mixing | SHOULD require approval |

### Y.4 Consolidation Rules (Normative)

1. Consolidation MUST require approval
2. Consolidation to non-allowlisted address → fail closed
3. `DORMANT` UTXO consolidation → `HUMAN_APPROVE`
4. Consolidation MUST be user-initiated only

### Y.5 Quantum-Aware Mode (Normative)

Exposed UTXOs (public key visible on-chain) MUST require explicit override for selection.

### Y.6 Override Binding (Normative)

Overrides MUST bind to:
- `sid`
- `action_id`
- `utxo_refs` (array of outpoints)
- `recipient_id`

### Y.7 Refusal Codes

| Code | Condition |
|------|-----------|
| `E_DORMANT_UTXO_SELECTION_DENIED` | Dormant UTXO without approval |
| `E_RESTRICTED_UTXO_RECIPIENT_MISMATCH` | Restricted to wrong recipient |
| `E_QUARANTINE_UTXO_MIXING_PROHIBITED` | Quarantine mixing attempted |
| `E_CONSOLIDATION_APPROVAL_REQUIRED` | Consolidation without approval |

---

# ANNEX Z - Display Integrity and User Approval Binding (Optional, Normative)

PQHD defines the structures and required checks only. All refusal, override, lockout, and enforcement decisions are evaluated exclusively by PQSEC. Any "refuse" language in this annex describes PQSEC predicate outcomes, not behaviour performed by PQHD.

### Z.1 Purpose

Prevent UI deception by cryptographically binding displayed transaction details to approval and signing.

### Z.2 Display Manifest (Normative)

**DisplayManifest (deterministic CBOR map):**

| Field | Type | Description |
|-------|------|-------------|
| `v` | uint | Schema version |
| `network` | tstr | Bitcoin network |
| `psbt_commitment` | bstr (32) | Hash of PSBT |
| `output_manifest_hash` | bstr (32) | Hash of output manifest |
| `displayed_at` | uint | Tick when shown to user |
| `outputs` | array of DisplayOutput | What was displayed |
| `fee_rate` | uint | Displayed fee rate |
| `absolute_fee` | uint | Displayed absolute fee |

**DisplayOutput:**

| Field | Type | Description |
|-------|------|-------------|
| `classification` | tstr | `"recipient"`, `"change"` |
| `amount_display` | tstr | Human-readable amount |
| `destination_display` | tstr | Full address (no truncation) |
| `script_type` | tstr | Output type |

**No truncation:** Addresses MUST be displayed in full. Truncated display is insufficient.

### Z.3 Approval Binding (Normative)

```
display_manifest_hash = SHAKE256-256(canonical_display_manifest_bytes)
```

Approval MUST bind to:
- `sid`
- `action_id`
- `display_manifest_hash`
- `displayed_at`
- `expires_at`

For `sealed` operations, the `DisplayManifest` MUST additionally include `execution_profile = "sealed"`, `miner_id` (if known), and `expires_at`. The approval MUST additionally bind to `submission_id` and `expires_at`. Mismatch → `E_UI_APPROVAL_NOT_BOUND`.

### Z.4 Freshness Constraints (Normative)

Approval MUST be given within `freshness_window` of `displayed_at`.

`freshness_window` is a policy parameter. RECOMMENDED default: 60 seconds. Deployments MAY configure a different value via custody policy. The configured value MUST be enforced by PQSEC.

### Z.5 Signing-Time Verification (Normative)

At signing time:

1. Recompute `display_manifest_hash`
2. Verify it matches approval binding
3. Verify `psbt_commitment` matches
4. Mismatch → `valid_display_integrity` MUST evaluate to FALSE

### Z.6 Multi-Recipient Requirements (Normative)

Transactions with multiple non-change recipients MUST:
- Display all recipients
- Require `HUMAN_APPROVE`
- Bind approval to complete manifest

### Z.7 Destination Display Requirements (Normative)

Must provide verifiable destination identity:
- Full address string
- QR code (optional, supplementary)
- No truncation

Insufficient display fidelity → fail closed.

### Z.8 Refusal Codes

| Code | Condition |
|------|-----------|
| `E_DISPLAY_MANIFEST_MISSING` | No manifest present |
| `E_DISPLAY_MANIFEST_HASH_MISMATCH` | Manifest changed |
| `E_UI_APPROVAL_NOT_BOUND` | Approval not bound to manifest |
| `E_UI_APPROVAL_EXPIRED` | Approval too old |
| `E_UI_RENDER_UNVERIFIABLE` | Display insufficient |

### Z.9 Scope Clarification (Informative)

Display integrity does NOT address:
- Compromised operating system
- Screen overlays
- Hardware supply chain attacks
- Social engineering

These are deployment concerns, not protocol concerns.

---

# ANNEX AA - Hash-Only Audit Logging (Optional, Normative)

PQHD defines the structures and required checks only. All refusal, override, lockout, and enforcement decisions are evaluated exclusively by PQSEC. Any "refuse" language in this annex describes PQSEC predicate outcomes, not behaviour performed by PQHD.

### AA.1 Purpose

Enable auditability without surveillance by logging only hashes and metadata.

### AA.2 Constraints (Normative)

Audit logs MUST NOT store:

| Prohibited | Reason |
|------------|--------|
| Payloads | Contains transaction details |
| Addresses | Identifies recipients |
| Descriptors | Reveals wallet structure |
| Cleartext recipients | Privacy violation |
| Cleartext policies | May contain sensitive rules |

### AA.3 Allowed Fields (Normative)

| Field | Type | Description |
|-------|------|-------------|
| `sid` | bstr (16) | Session identifier |
| `action_id` | bstr (16) | Action identifier |
| `outcome` | tstr | `ALLOW`, `DENY`, `UNAVAILABLE` |
| `refusal_code` | tstr | Error code if refused |
| `receipt_hashes` | array of bstr | Hashes of evidence used |
| `policy_profile_hash` | bstr (32) | Policy hash |
| `issued_tick` | uint | When logged |

**Rule:** Hash where feasible. Never store cleartext of sensitive data.

**SEAL operations:** For `sealed` operations, the audit log MAY additionally include `submission_id`, `miner_id`, `submission_evidence_hash`, and `execution_profile = "sealed"`. The audit log MUST NOT include `template_hash` (already bound to `intent_hash`), `ciphertext`, `kem_ciphertext`, `nonce`, `tag` (confidential), or raw SEAL payloads.

### AA.4 Integrity (Normative)

Audit log MUST be append-only.

Tampering detection:
- Maintain running hash chain
- Periodic checkpoints

### AA.5 Refusal Code

| Code | Condition |
|------|-----------|
| `E_AUDIT_LOG_COMPROMISED` | Integrity check failed |

---

# ANNEX AB - Emergency Freeze and Authority Lock (Optional, Normative)

PQHD defines the structures and required checks only. All refusal, override, lockout, and enforcement decisions are evaluated exclusively by PQSEC. Any "refuse" language in this annex describes PQSEC predicate outcomes, not behaviour performed by PQHD.

### AB.1 Purpose

Provide immediate authority suspension mechanism.

### AB.2 Authority Lock State (Normative)

| State | Description |
|-------|-------------|
| `UNLOCKED` | Normal operation |
| `FROZEN` | All authority-bearing actions rejected |

When `AuthorityLockState = FROZEN`:
- All `REQUEST_AUTHORITY` → `E_AUTHORITY_FROZEN`
- All `EXECUTE` → `E_AUTHORITY_FROZEN`
- `COORDINATION` messages permitted

### AB.3 Trigger Mechanisms (Normative)

Freeze MAY be triggered by:

| Mechanism | Description |
|-----------|-------------|
| Trusted UI | Direct user action in trusted interface |
| Threshold approval | Multiple authorised humans |
| Policy trigger | Automatic based on policy rules |

### AB.4 Freeze Token (Normative)

```
freeze_input = deterministic_cbor_encode({
  "root_secret": root_secret,
  "freeze_salt": freeze_salt,
  "issued_tick": issued_tick
})
freeze_token = SHAKE256-256(freeze_input)
```

| Field | Type | Description |
|-------|------|-------------|
| `root_secret` | bstr | Wallet root secret material |
| `freeze_salt` | bstr | Unique salt for this freeze |
| `issued_tick` | uint | When freeze was initiated |

The input MUST be deterministic CBOR (PQSF encoding rules). Raw concatenation MUST NOT be used.

Freeze token proves freeze was initiated by legitimate authority.

### AB.5 Effects (Normative)

Upon freeze:

1. All active sessions invalidated
2. All transcript commitments invalidated
3. All pending approvals discarded
4. New session scope issuance blocked

### AB.6 Recovery (Normative)

Recovery from `FROZEN` requires:

1. New session establishment
2. Full re-authorisation under current policy
3. Policy re-approval if required
4. Quantum lock compliance verification

### AB.7 Persistence (Normative)

Freeze state MUST persist across:
- Application restarts
- Device reboots
- Crashes

Freeze is NOT automatically cleared by restart.

### AB.8 Audit (Normative)

Record freeze events:

| Field | Description |
|-------|-------------|
| `issued_tick` | When freeze occurred |
| `trigger_mechanism` | How freeze was triggered |
| `freeze_token_hash` | `SHAKE256-256(freeze_token)` |
| `authority_lock_state` | New state |

### AB.9 Refusal Codes

| Code | Condition |
|------|-----------|
| `E_AUTHORITY_FROZEN` | Action attempted while frozen |
| `E_FREEZE_RECOVERY_REQUIRED` | Must complete recovery |
| `E_FREEZE_TOKEN_INVALID` | Invalid freeze/unfreeze token |

---

# **ANNEX AC - Common Implementation Mistakes (Informative)**

This annex catalogues antipatterns observed in custody system implementations. Each entry describes the mistake, why it violates PQHD, and the correct approach. This annex is informative and does not define additional conformance requirements, but implementations exhibiting these patterns will fail conformance testing.

## AC.1 Signing Without EnforcementOutcome

Mistake: The signing component produces a Bitcoin signature when it holds valid key material, without first receiving and verifying a PQSEC EnforcementOutcome.

Why this violates PQHD: Section 13.1 requires both key material and a valid EnforcementOutcome. Key possession alone MUST NOT permit signing. This is the foundational principle of PQHD: key possession is not authority.

Correct approach: The signing component MUST refuse to produce any signature unless it has received an EnforcementOutcome with decision == "ALLOW", verified all binding fields per Annex C, confirmed the decision has not expired, and confirmed the decision_id has not been used before.

## AC.2 Reusing EnforcementOutcome Across Signing Attempts

Mistake: The implementation caches a successful EnforcementOutcome and reuses it for subsequent signing attempts or for different PSBTs.

Why this violates PQHD: Annex C Section C.4 requires single-use enforcement at the decision_id level. Each signing attempt requires a fresh EnforcementOutcome bound to the specific canonical PSBT being signed.

Correct approach: Track all consumed decision_id values. Refuse any EnforcementOutcome whose decision_id has been seen before. Each new signing attempt requires a new PQSEC evaluation request and a new EnforcementOutcome.

## AC.3 Coordinator Filtering Signers

Mistake: In an M-of-N deployment, the coordinator selects which signers receive the PSBT and EnforcementOutcome based on expected signing behaviour, availability prediction, or performance optimisation.

Why this violates PQHD: The coordinator is untrusted (Annex I). Signer selection by the coordinator introduces a trust assumption that the coordinator will not suppress quorum. All eligible signers MUST receive the PSBT and EnforcementOutcome. Each signer independently decides whether to sign based on its own Annex C verification.

Correct approach: The coordinator distributes the PSBT and EnforcementOutcome to all enrolled signers. The coordinator collects responses. The coordinator does not decide which signers participate.

## AC.4 Adapter Retrying Refused Operations

Mistake: The adapter (execution bridge) receives a DENY or FAIL_CLOSED_LOCKED EnforcementOutcome and automatically retries the PQSEC evaluation request without new user intent.

Why this violates PQHD: A refusal is an enforcement decision. Automatic retry without fresh user intent and fresh consent is an attempt to circumvent enforcement. Section 21.2 explicitly prohibits the adapter from performing retry logic for refused operations.

Correct approach: Surface the refusal to the user interface. Collect new user intent. Generate a new ConsentProof. Submit a new PQSEC evaluation request. The user, not the adapter, decides whether to retry.

## AC.5 UI Interpreting EnforcementOutcome Semantics

Mistake: The user interface receives an EnforcementOutcome and applies its own logic to determine whether to proceed. For example, the UI might treat a DENY as retriable based on the refusal code, or downgrade a FAIL_CLOSED_LOCKED to a user-friendly warning.

Why this violates PQHD: The UI does not participate in the enforcement decision. Section 21.2 requires the UI to surface the enforcement result without reinterpretation. FAIL_CLOSED_LOCKED is a lockout condition (Section 13.2) and MUST be surfaced as such.

Correct approach: Display the decision and refusal code to the user. Do not apply conditional logic to enforcement results. Do not offer retry for FAIL_CLOSED_LOCKED. If the decision is DENY, the UI may collect new user intent for a fresh attempt, but the UI does not decide whether the refusal is soft or hard.

## AC.6 Single-Device Deployment Claiming PQHD Conformance

Mistake: An implementation uses PQHD key derivation, PSBT canonicalisation, and EnforcementOutcome schemas but deploys on a single device without multi-device quorum, and claims PQHD conformance or PQHD Custody.

Why this violates PQHD: Section 9.1 requires multi-device quorum for Baseline Custody. Section 25 requires multi-device quorum for conformance. A single compromised device MUST be insufficient to authorise a spend. There is no reduced or transitional tier.

Correct approach: Either deploy multi-device quorum and claim Baseline conformance, or use PQHD canonical structures for interoperability without claiming conformance (see Annex N, Section N.3).

## AC.7 Fallback Signing Path

Mistake: The implementation provides a "fallback" or "emergency" signing path that bypasses PQSEC evaluation when the enforcement core is unavailable. This is sometimes implemented as a local override, an admin key, or a timeout-based bypass.

Why this violates PQHD: Section 23 states that no override or fallback is permitted for Authoritative operations. Section 7 (Custody Authority Model) requires that any ambiguity, missing input, or validation failure results in refusal. An unavailable enforcement core is a missing input.

Correct approach: If PQSEC is unavailable, signing is refused. The system is inert until enforcement is restored. Emergency access is handled through recovery governance (Section 17), which itself requires PQSEC enforcement, guardian quorum, and enforced delay. There is no shortcut.

## AC.8 Constructing ConsentProof Without User Interaction

Mistake: The adapter or an automated component generates ConsentProof artefacts programmatically without actual user interaction, in order to automate custody operations.

Why this violates PQHD: ConsentProof (Glossary) binds explicit user intent to a specific action. Programmatic generation without user interaction is a fabricated consent artefact. PQSEC evaluates valid_consent as a predicate; if the artefact is structurally valid but was not produced by genuine user intent, the system has bypassed consent by construction.

Correct approach: ConsentProof MUST be produced only in response to explicit user action. The user must see the operation details (amount, recipient, fee) and explicitly approve. Automated signing requires delegation governance, not fabricated consent.

---

## **GLOSSARY (CANONICAL)**

**Bundle Hash**
A deterministic hash of the canonical PSBT, computed using SHAKE256-256 (32 bytes), that commits to the complete transaction structure approved for signing. In PQHD flows, the bundle hash is the `intent_hash` bound into an EnforcementOutcome.

**ConsentProof**
A cryptographically signed, time-bounded artefact that binds explicit user intent to a specific action, canonical transaction structure, session binding, exporter binding (for Authoritative operations), and an EpochTick window.

**Epoch Clock**
A Bitcoin-anchored temporal authority that produces signed EpochTicks used as the sole source of time for PQHD custody decisions. Epoch Clock artefacts are consumed via PQSEC.

**EpochTick**
A signed, monotonic time artefact issued under an Epoch Clock profile that binds custody actions to verifiable time. Freshness and monotonicity are enforced by PQSEC.

**EnforcementOutcome**
An authoritative decision artefact produced by PQSEC indicating whether a custody operation is permitted, refused, or locked out. A valid EnforcementOutcome is required before any Authoritative signing or custody mutation may proceed.

**Fail-Closed**
Mandatory behaviour in which any predicate failure, ambiguity, missing input, or verification failure halts custody operations and prevents signature emission.

**GuardianSet**
A signed declaration of the guardian set and approval threshold used for recovery governance. GuardianSet grants no authority by itself and is enforced by PQSEC.

**Ledger**
A deterministic, append-only, tamper-evident structure recording all PQHD custody-relevant events and enforcing monotonic wallet state continuity.

**ML-DSA-65**
A post-quantum digital signature algorithm used by producing specifications for PQHD-related artefacts, including ConsentProofs, policy artefacts, ledger entries, recovery governance artefacts, and EnforcementOutcome signatures where required by policy.

**PaymentEndpointKey (PEK)**
A signed, time-bounded, jurisdiction-scoped, revocable eligibility artefact for regulated payment endpoint operations. A PEK does not grant custody authority, does not permit signing, and is enforced only via PQSEC predicates (for example valid_payment_endpoint).

**PQHD**
Post-Quantum Hierarchical Deterministic Custody. An open specification defining deterministic, predicate-based custody authority policy for Bitcoin.

**PQSEC**
Post-Quantum Security & Enforcement Core. The enforcement specification that evaluates custody predicates defined by PQHD and produces authoritative EnforcementOutcome decisions.

**PQSF**
Post-Quantum Canonical Format. A supporting specification defining canonical encoding, cryptographic suite profiles, transport binding, and deterministic security primitives consumed by PQHD.

**PSBT**
Partially Signed Bitcoin Transaction. A Bitcoin transaction coordination format used for multisignature signing without exposing private keys.

**RecoveryApproval**
A signed guardian approval artefact bound to a RecoveryIntent. RecoveryApproval grants no authority and is enforced by PQSEC.

**RecoveryIntent**
A signed artefact declaring a recovery attempt, including the applicable GuardianSet and enforced activation delay. RecoveryIntent grants no authority and is enforced by PQSEC.

**SafeMode**
A restrictive custody operating state that can refuse or constrain irreversible operations. SafeMode does not grant authority and is enforced by PQSEC.

**SafeModeState**
A signed artefact declaring SafeMode ACTIVE or INACTIVE for a custody domain. SafeModeState grants no authority and is enforced by PQSEC.

**Unified Custody Predicate**
The conjunction of all custody predicates that MUST simultaneously evaluate to true, as determined by PQSEC, before a Bitcoin signature may be produced.

**ZEB**
Zero-Exposure Execution Boundary. A Bitcoin execution and observation protocol that executes transactions only after PQSEC authorization. See ZEB specification for broadcast semantics.

**ZET**
Zero-Exposure Transactions. An execution boundary model ensuring atomic execute-or-refuse semantics after external enforcement approval.

---

## Changelog

### Version 1.2.0

* **Authority separation finalised**
  Formally decoupled Bitcoin signing authority from key possession. Private keys are necessary inputs but never sufficient for authorisation.

* **External enforcement locked**
  All predicate evaluation, refusal, freshness, monotonicity, escalation, and lockout semantics are delegated exclusively to PQSEC. PQHD defines policy and structure only.

* **Unified custody predicate stabilised**
  Canonicalised the custody predicate composition for signing (`valid_tick`, `valid_consent`, `valid_policy`, `valid_runtime`, `valid_quorum`, `valid_ledger`, `valid_psbt`) with fail-closed semantics on any ambiguity.

* **Custody tiers clarified**
  Finalised **Baseline** and **Enterprise** custody tiers with explicit qualification requirements. Removed implicit or reduced authority modes from conformance claims.

* **Deterministic key hierarchy hardened**
  Standardised post-quantum root and child key derivation with strict domain separation. Explicitly prohibited authority inference from key derivability.

* **Canonical PSBT discipline enforced**
  Introduced strict PSBT v2 canonicalisation, equivalence rules, and deterministic intent hashing to prevent malleability and signer divergence.

* **Dual-control signing mandated**
  Required a valid, single-use PQSEC EnforcementOutcome for every signing attempt. Defined binding and replay-protection requirements normatively.

* **Recovery governance formalised**
  Finalised guardian-based recovery with enforced delay, quorum approval, SafeMode integration, and full PQSEC enforcement. Recovery cannot bypass custody predicates.

* **Execution boundary integration clarified**
  Defined non-authoritative integration with ZET and ZEB. Execution method selection does not modify custody authority semantics.

* **Auditability without surveillance**
  Added hash-only audit logging and explicit prohibition of payload, address, and descriptor leakage.

---

## Funding (Non-Normative)

`bc1q380874ggwuavgldrsyqzzn9zmvvldkrs8aygkw`


