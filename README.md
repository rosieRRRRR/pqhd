# **PQHD — Post-Quantum Hierarchical Deterministic Wallet**

* **Specification Version:** 1.1.0
* **Status:** Public beta
* **Date:** 2026
**Author:** rosiea
**Contact:** [PQRosie@proton.me](mailto:PQRosie@proton.me)
**Licence:** Apache License 2.0 — Copyright 2026 rosiea

---

## **Summary**

PQHD defines custody authority policy for Bitcoin, decoupling authority from key possession. Traditional wallets treat keys as authority; PQHD makes authority conditional on explicit predicates: time, consent, policy, runtime integrity, quorum, ledger continuity, and PSBT equivalence. These predicates are **defined by PQHD** and **evaluated externally by PQSEC**.

PQHD enables custody systems that remain secure under classical private-key compromise and anticipated quantum attack by ensuring that possession of cryptographic material alone is insufficient to authorise a spend. Multi-device quorum, deterministic key hierarchies, canonical transaction handling, and governed recovery replace implicit trust with explicit, verifiable conditions.

PQHD defines custody policy only. All enforcement, refusal, escalation, lockout, and freshness semantics are performed exclusively by PQSEC.

**Key Properties:**
Predicate-driven custody · Key possession ≠ authority · Multi-device quorum · Deterministic hierarchies · Canonical PSBT handling · Recovery governance · External enforcement

---

## Index

1. [Summary](#summary)
2. [Non-Normative Overview](#non-normative-overview--for-explanation-and-orientation-only)
3. [Scope and Custody Boundary](#scope-and-custody-boundary)
4. [Non-Goals and Separation of Concerns](#non-goals-and-separation-of-concerns)
5. [Threat Model](#threat-model)
6. [Trust Assumptions](#trust-assumptions)
7. [Architecture Overview](#architecture-overview)
8. [Explicit Dependencies](#explicit-dependencies)
9. [Custody Authority Model](#custody-authority-model)
10. [Unified Custody Predicate](#unified-custody-predicate)
11. [Custody Tiers](#custody-tiers)
12. [Root Key Generation](#root-key-generation)
13. [Child Key Derivation](#child-key-derivation)
14. [Key Classes](#key-classes)
15. [Key Derivation Rules](#key-derivation-rules)
16. [Bitcoin Compatibility Keys](#bitcoin-compatibility-keys)
17. [Canonical PSBT Handling](#canonical-psbt-handling)
18. [Dual-Control Signing Requirement](#dual-control-signing-requirement)
19. [PQSEC Evaluation Request](#pqsec-evaluation-request)
20. [Signing Flow](#signing-flow)
21. [Custody Hardware Profile](#custody-hardware-profile)
22. [Multisignature and Quorum Authority](#multisignature-and-quorum-authority)
23. [Ledger Authority](#ledger-authority)
24. [Recovery Governance](#recovery-governance)
25. [Device Lifecycle Management](#device-lifecycle-management)
26. [Epoch Clock Handling](#epoch-clock-handling)
27. [Error Handling](#error-handling)
28. [Dependency Boundaries](#dependency-boundaries)
29. [Execution Integration](#execution-integration)
30. [Failure Semantics](#failure-semantics)
31. [Bootstrap Trust Anchors](#bootstrap-trust-anchors)
32. [Conformance](#conformance)
33. [Annexes](#annexes)
34. [Changelog](#changelog)

---

## **Non-Normative Overview — For Explanation and Orientation Only**

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
| PQSEC | ≥ 2.0.2 | Enforcement of custody predicates |
| PQSF | ≥ 2.0.2 | Canonical encoding for all custody artefacts |
| Epoch Clock | ≥ 2.0.0 | Time-bounded consent and policy artefacts |
| ZET/ZEB | ≥ 1.2.0 | Execution boundary (when executing Bitcoin transactions) |
| PQEH | ≥ 2.1.1 | Post-quantum execution hardening (when claiming zero-exposure) |
| PQVL | ≥ 1.0.3 | Runtime attestation (when valid_runtime is required) |

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

Predicate production, validation, freshness, monotonicity, refusal, escalation, and lockout semantics are defined externally by **PQSEC** and producing specifications.

---

## **9. Custody Tiers**

PQHD defines custody tiers that qualify the security posture under which custody authority claims MAY be made.

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

### **9.3 Transactional Profile**

**(Schema Conformant, Non-Custodial)**

The Transactional Profile exists for migration and low-value operational use only.

1. Implements PQHD canonical structures.
2. Lacks multi-device quorum guarantees.
3. MUST NOT be described as providing PQHD Custody.
4. MUST be treated as non-custodial under the PQHD threat model.

Only **Baseline** and **Enterprise** tiers MAY claim PQHD Custody.

---

## **10.1 Root Key Generation**

A single post-quantum root key MUST be generated at wallet creation:

```
root_key = SHAKE256-256(seed)
```

Where:

* `seed` is high-entropy random material (minimum 256 bits)
* `seed` MUST be generated using a cryptographically secure RNG
* `seed` MUST NOT be reused across wallets
* `SHAKE256-256` output length MUST be exactly 32 bytes

Loss of the root key without recovery governance results in permanent loss of custody.

---

### **10.2 Child Key Derivation**

All child keys MUST be deterministically derived from the root key using domain separation:

```
child_key = cSHAKE256(
  root_key,
  domain = "PQHD-Key:<key_class>",
  input  = canonical({
    purpose: tstr,
    index: uint,
    network: "mainnet" / "testnet"
  })
)
```

Derivation MUST be deterministic and reproducible across implementations.

---

### **10.3 Key Classes**

PQHD defines the following key classes:

* **custody** — multisignature custody keys
* **recovery** — guardian and recovery keys
* **identity** — device identity binding keys
* **ephemeral** — per-operation keys (MUST NOT be used for custody)

Key derivability MUST NOT imply custody authority.

---

### **10.4 Key Derivation Rules**

1. All keys MUST derive from PQHD root material.
2. Key classes MUST be isolated by purpose.
3. Two implementations given identical inputs MUST derive identical keys.
4. Ephemeral keys MUST NOT be reused across operations.

---

## **11. Bitcoin Compatibility Keys**

### **11.1 secp256k1 Key Derivation**

Bitcoin-compatible secp256k1 keys MUST be deterministically derived from PQHD root material using rejection sampling to ensure valid scalar range.

The derived scalar MUST be in the interval `[1, N−1]`, where `N` is the secp256k1 curve order.

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

* deterministic field ordering
* identical semantic representation across signers
* rejection of unknown or non-canonical fields

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

**Signing components MUST implement the EnforcementOutcome acceptance and validation requirements defined in Annex B in full.**

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
   * Runtime evidence (PQVL artefacts, when valid_runtime required)
   * Quorum state
   * Ledger state

PQSEC evaluates the unified custody predicate and returns an EnforcementOutcome.

PQHD signing components MUST NOT produce signatures without a valid EnforcementOutcome satisfying Annex B.

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

---

## **14. Custody Hardware Profile**


The new **13.4 PQSEC Evaluation Request** sits between Signing Component Requirements and Signing Flow, which makes logical sense: requirements → how to request → the flow.
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

PQHD recovery governance is expressed exclusively using the **custody lifecycle extension artefacts** defined in **Annex I**:

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

* PSBT structure invalid → `E_PSBT_NONCANONICAL`
* quorum insufficient → `E_QUORUM_INSUFFICIENT`
* ledger divergence → `E_LEDGER_DIVERGENCE`
* device not enrolled → `E_DEVICE_UNENROLLED`
* recovery delay not elapsed → `E_RECOVERY_TOO_EARLY`
* EnforcementOutcome expired → `E_OUTCOME_EXPIRED`
* EnforcementOutcome replay → `E_OUTCOME_REPLAYED`

---

### **20.2 Error Propagation**

PQHD MUST NOT define new error codes.
All refusals MUST use PQSEC error vocabulary.

---

## **21. Dependency Boundaries**

1. PQHD MUST delegate all enforcement decisions to PQSEC.
2. PQHD MUST consume runtime integrity evidence via PQVL only through PQSEC evaluation.
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
* Decision replay protection and burn discipline MUST be enforced by execution layers.

---

### **22.2 PQEH Integration**

* PQEH provides post-quantum execution hardening patterns.
* Revelation or execution steps MUST NOT occur prior to PQSEC approval.
* Execution hardening MUST NOT alter custody authority semantics.

---

### **22.3 Relationship to BIP-360 (Informative)**

BIP-360 describes a Bitcoin output construction that removes the Taproot key-path and relies exclusively on script-path spending, reducing long-range public-key exposure.

PQHD does not depend on BIP-360 and does not require any specific Bitcoin output type. Custody authority defined by PQHD is independent of execution construction and remains valid under all Bitcoin-current-consensus output forms.

Where BIP-360 or similar script-path-only constructions are used, they MAY be combined with PQHD custody and PQEH execution-time hardening to reduce exposure during execution. Such usage affects execution mechanics only and MUST NOT alter custody predicate semantics or enforcement boundaries.

PQHD makes no claims regarding consensus adoption, miner policy, or deployment status of BIP-360.

---

## **23. Failure Semantics**

1. Any custody predicate failure MUST result in refusal.
2. Partial authority MUST NOT be granted.
3. No override or fallback is permitted for Authoritative operations.
4. Repeated failures MAY result in PQSEC lockout.

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
* enforces quorum requirements
* enforces recovery governance where supported
* delegates all enforcement to PQSEC

---

# **ANNEX A — Signing Authorisation Flow (Informative)**

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
     - Runtime evidence (PQVL artefacts)
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
9. Execute via ZET boundary, broadcast via ZEB (if configured)
```

No signature may be produced unless a valid EnforcementOutcome authorises the attempt.

---

# **ANNEX B — EnforcementOutcome Acceptance Rules (Normative)**

This annex defines the minimum required checks a PQHD signing component MUST apply to an EnforcementOutcome prior to signature emission.

## B.1 Required Fields

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

## B.2 Decision Gate

Signing MUST be refused unless:

* `decision == "ALLOW"`

If `decision == "DENY"` or `decision == "FAIL_CLOSED_LOCKED"`, signing MUST be refused.

## B.3 Binding Requirements

For any signing attempt, the signing component MUST verify:

1. `intent_hash` equals the computed `bundle_hash` for the canonical PSBT associated with the attempt.
2. `session_id` matches the active session identifier for the attempt.
3. If `operation_class == "Authoritative"`, `exporter_hash` matches the active session exporter hash.
4. `issued_tick` and `expiry_tick` are present and `current_tick < expiry_tick`.

Any mismatch MUST cause refusal.

## B.4 Replay Protection

A signing component MUST enforce single-use of EnforcementOutcome at the decision_id level:

* A given `decision_id` MUST be accepted at most once for signature emission.
* Reuse MUST be refused.

Decision replay protection MUST be persistent across process restarts within the signing domain.

## B.5 Signature Verification

If the EnforcementOutcome includes a `signature` field and the active policy requires it, the signing component MUST verify that signature under the referenced suite profile.

If signature verification is required and fails, signing MUST be refused.

## B.6 Compatibility Mapping

Where an execution boundary expects `allowed: bool`, implementers MAY map:

* `allowed = (decision == "ALLOW")`

This mapping MUST NOT bypass the required checks above.

---

# **ANNEX C — Deterministic Key Derivation (Reference)**

This annex provides reference pseudocode for deterministic derivation. It is non-authoritative but intended to be implementable.

```python
from hashlib import shake_256

def derive_root_key(seed: bytes) -> bytes:
    # root_key is 32 bytes
    return shake_256(seed).digest(32)

def derive_child_key(root_key: bytes, key_class: str, context_cbor: bytes) -> bytes:
    # Domain separation string MUST be stable and unambiguous
    domain = f"PQHD-Key:{key_class}".encode("utf-8")
    return shake_256(root_key + domain + context_cbor).digest(32)
```

---

# **ANNEX D — Bitcoin secp256k1 Scalar Derivation (Reference)**

```python
from hashlib import shake_256

SECP256K1_N = int(
    "FFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFEBAAEDCE6AF48A03BBFD25E8CD0364141", 16
)

def derive_secp256k1_scalar(root_key: bytes, context: bytes) -> int:
    counter = 0
    while True:
        data = root_key + context + counter.to_bytes(4, "big")
        candidate = int.from_bytes(shake_256(data).digest(32), "big")
        if 1 <= candidate < SECP256K1_N:
            return candidate
        counter += 1
```

---

# **ANNEX E — Canonical PSBT Hashing and Equivalence (Normative)**

This annex defines how `bundle_hash` (intent_hash for signing) MUST be computed.

## **E.1 Canonical PSBT Bytes**

Implementations MUST define a canonical PSBT byte form for hashing such that:

1. Given semantically identical PSBTs, independent implementations produce byte-identical canonical bytes.
2. Unknown/proprietary fields MUST cause refusal.
3. Canonical bytes MUST be stable across platforms.

Implementations claiming interoperability MUST declare a named PSBT canonicalisation profile or publish a complete canonicalisation rule-set as part of their conformance statement.

If a deployment uses a named profile (for example, a fixed profile in the ecosystem), that profile MAY define the canonical PSBT byte form. If no profile is declared, the deployment MUST document and enforce its canonicalisation rules.

## **E.2 Hash Function**

`bundle_hash` MUST be computed as:

```
bundle_hash = SHAKE256-256(canonical_psbt_bytes)
```

Output length MUST be exactly 32 bytes.

## **E.3 Equivalence Rule**

Two PSBTs are equivalent iff their `bundle_hash` values are byte-identical.

A signer MUST refuse to sign if the PSBT presented differs from the PSBT whose `bundle_hash` is bound into the EnforcementOutcome.

---

# **ANNEX F — Custody Predicate Assembly (Informative)**

PQHD predicate assembly is descriptive. PQSEC performs evaluation.

A typical custody signing attempt submits the following artefact set to PQSEC:

* EpochTick (as JCS canonical JSON bytes)
* ConsentProof
* Policy artefacts
* Runtime evidence artefacts (PQVL)
* Quorum state snapshot
* Ledger state snapshot
* canonical PSBT bytes (or their hash)

This annex does not define enforcement semantics.

---

# **ANNEX G — Dual-Control Signer (Reference)**

```python
class DualControlSigner:
    """
    Reference dual-control signer: requires BOTH key material and
    an acceptable EnforcementOutcome per Annex B.
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
        # Minimal checks; real implementations MUST implement Annex B fully.
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

# **ANNEX H — Untrusted M-of-N Coordinator (Reference)**

```python
class QuorumCoordinator:
    """
    Coordinator is untrusted. Signers enforce Annex B independently.
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

# **ANNEX I — Custody Lifecycle Extensions (Delegation / Guardian Recovery / SafeMode) (Normative)**

This annex defines artefact schemas used to restrict or condition custody authority. These artefacts grant no authority by themselves. Enforcement is performed exclusively by PQSEC.

## I.1 DelegationConstraint

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

## I.2 GuardianSet

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

## I.3 RecoveryIntent

```
RecoveryIntent = {
  recovery_id: tstr,
  guardian_set_id: tstr,
  requested_by: tstr,
  issued_tick: uint,
  activation_delay: uint,
  suite_profile: tstr,
  signature: bstr
}
```

Rules:

1. RecoveryIntent MUST be explicit and ledger-recorded.
2. `activation_delay` MUST be enforced using verified ticks.
3. Signature MUST verify under suite_profile.

## I.4 RecoveryApproval

```
RecoveryApproval = {
  recovery_id: tstr,
  guardian_id: tstr,
  approved_tick: uint,
  suite_profile: tstr,
  signature: bstr
}
```

Rules:

1. Each guardian MAY approve at most once per recovery_id.
2. Approvals SHOULD be ledger-recorded.
3. Signature MUST verify under suite_profile.

Enforcement predicates (PQSEC):

* `valid_guardian_quorum`
* `recovery_delay_elapsed`

## I.5 SafeModeState

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

---

# **ANNEX J — Recovery Activation Sequence (Informative)**

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

# **ANNEX K — Device Enrollment and Revocation (Informative)**

Devices MUST be enrolled to participate in quorum. Revocation removes eligibility.

Recommended ledger events:

* device_enrolled
* device_revoked

Enrollment and revocation are Authoritative operations and therefore require PQSEC EnforcementOutcome.

---

# **ANNEX L — Custody Tier Qualification Matrix (Informative)**

| Requirement                  | Baseline | Enterprise | Transactional  |
| ---------------------------- | -------- | ---------- | -------------- |
| Multi-device quorum          | REQUIRED | REQUIRED   | NOT REQUIRED   |
| Dual-control signing         | REQUIRED | REQUIRED   | REQUIRED       |
| Ledger continuity            | REQUIRED | REQUIRED   | OPTIONAL       |
| Recovery governance          | OPTIONAL | REQUIRED   | NOT AVAILABLE  |
| Quorum diversity constraints | OPTIONAL | REQUIRED   | NOT APPLICABLE |

---

# **ANNEX M — Signing Attempt Record (Informative)**

Recommended ledger payload fields for a signing attempt:

* decision_id
* bundle_hash (intent_hash)
* operation_class
* signer_ids (or count)
* outcome decision
* refusal error_code (if refused)

---

# **ANNEX N — Migration Guidance (Informative)**

Migration from key-only wallets SHOULD proceed in phases:

1. Canonical PSBT validation and hashing
2. Dual-control signing gate
3. Multi-device quorum
4. Ledger continuity
5. Recovery governance (Enterprise)

---

# **ANNEX O — Troubleshooting (Informative)**

**Signing refused**

* Check EnforcementOutcome decision, expiry_tick, and binding to bundle_hash/session/exporter.

**Quorum not met**

* Confirm signer availability and enrollment status.

**Recovery not activating**

* Confirm delay elapsed and guardian approvals meet threshold.

---

# **ANNEX P — Security Checklist (Informative)**

* ☐ PSBT canonicalisation enforced; unknown fields refused
* ☐ bundle_hash computed deterministically (SHAKE256-256, 32 bytes)
* ☐ EnforcementOutcome required for signing; decision_id replay-protected
* ☐ exporter_hash required and checked for Authoritative operations
* ☐ Multi-device quorum required for Baseline/Enterprise
* ☐ Ledger continuity required for Baseline/Enterprise
* ☐ Recovery is delayed and quorum-gated
* ☐ No degraded or override signing path exists

---

# **ANNEX Q — Conformance Checklist (Informative)**

An implementation claiming PQHD v1.1.0 conformance SHOULD be able to demonstrate:

* Refusal when any required predicate is missing or false
* Refusal on expired or replayed EnforcementOutcome
* Refusal on PSBT non-canonical structure or unknown fields
* Deterministic bundle_hash agreement across independent implementations
* Deterministic key derivation agreement across independent implementations
* Recovery refusal prior to delay/quorum satisfaction

---

# ANNEX R — Payment Endpoint Keys (Normative)

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

## **GLOSSARY (CANONICAL)**

**BIP-360**
A proposed Bitcoin output construction that removes the Taproot key-path and enforces script-path-only spending. Referenced in PQHD for execution-context clarity only; PQHD does not depend on BIP-360 and does not modify custody authority semantics.

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
Post-Quantum Hierarchical Deterministic Wallet. An open specification defining deterministic, predicate-based custody authority policy for Bitcoin.

**PQSEC**
Post-Quantum Security & Enforcement Core. The enforcement specification that evaluates custody predicates defined by PQHD and produces authoritative EnforcementOutcome decisions.

**PQSF**
Post-Quantum Security Framework. A supporting specification defining canonical encoding, cryptographic suite profiles, transport binding, and deterministic security primitives consumed by PQHD.

**PQVL**
Post-Quantum Verification Layer. A runtime integrity evidence specification that produces non-authoritative attestation artefacts consumed by PQSEC.

**PSBT**
Partially Signed Bitcoin Transaction. A Bitcoin transaction coordination format used for multisignature signing without exposing private keys.

**PQEH**
Post-Quantum Execution Hardening. Execution-time hardening patterns that reduce mempool exposure and broadcast-window risk without altering custody authority semantics.

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
Zero-Exposure Broadcast. A Bitcoin broadcast and observation profile that executes transactions only after PQSEC authorisation and enforces exposure detection and burn discipline.

**ZET**
Zero-Exposure Transactions. An execution boundary model ensuring atomic execute-or-refuse semantics after external enforcement approval.

---

Changelog
Version 1.1.0 (Current)
Separation of Authority: Formally decoupled authority from private key possession; keys are now defined as "predicates" consumed by the enforcement layer.

Enforcement Delegation: Moved all refusal, freshness, and monotonicity semantics to PQSEC, narrowing PQHD’s scope to policy definition and key hierarchy management.

Custody Tiers: Formalized the qualification criteria for Baseline vs. Enterprise custody tiers to address different institutional threat models.

PSBT Discipline: Introduced strict PSBT (Partially Signed Bitcoin Transaction) canonicalization and equivalence requirements to prevent malleability-based attacks.

---

## **ACKNOWLEDGEMENTS (INFORMATIVE)**

This specification builds on decades of work in cryptography, distributed systems, and the design and operation of the Bitcoin protocol.

The author acknowledges the foundational contributions of the following individuals and communities, whose work made the concepts formalised in PQHD possible:

* **Satoshi Nakamoto** — for the original design of Bitcoin and its trust-minimised consensus model.
* **Hal Finney** — for early Bitcoin implementation, advocacy, and applied cryptographic insight.
* **Adam Back** — for Hashcash and foundational work on proof-of-work systems.
* **Pieter Wuille** — for hierarchical deterministic wallets, SegWit, Taproot, Miniscript, and PSBT semantics.
* **Greg Maxwell** — for extensive contributions to Bitcoin security, multisignature design, and adversarial analysis.
* **Andrew Poelstra** — for Miniscript, script composability, and formal reasoning about Bitcoin spending policies.
* **Ralph Merkle** — for Merkle trees enabling tamper-evident state commitment.
* **Whitfield Diffie** and **Martin Hellman** — for public-key cryptography.
* **Peter Shor** — for demonstrating the impact of quantum computation on classical cryptography.
* **Daniel J. Bernstein** — for cryptographic engineering, constant-time design, and adversarial robustness.
* **The NIST Post-Quantum Cryptography Project** — for standardising post-quantum cryptographic primitives suitable for long-lived security systems.
* **Contributors to BIP-360 discussions** — for advancing script-path-only Taproot constructions that inform execution-layer exposure reduction, independent of custody authority.

Acknowledgement is also due to the Bitcoin Core developers, contributors to Bitcoin Improvement Proposals, and independent reviewers who have continually challenged assumptions around key custody, operational security, and failure modes.

If you find this work useful and wish to support continued development and public availability of the PQHD specification, donations are welcome:

**Bitcoin:**
bc1q380874ggwuavgldrsyqzzn9zmvvldkrs8aygkw
