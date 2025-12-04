# **PQHD — Post-Quantum Hierarchical Deterministic Wallet**

**An Open Standard for Quantum-Resistant Bitcoin Custody**

**Specification Version:** v1.0.0
**Status:** Implementation Ready. Peer Review Requested.
**Author:** rosiea
**Contact:** [PQRosie@proton.me](mailto:PQRosie@proton.me)
**Date:** November 2025
**Licence:** Apache License 2.0 — Copyright 2025 rosiea

---

## **Summary**

The Post-Quantum Hierarchical Deterministic Wallet (PQHD) is a quantum-resistant Bitcoin custody standard that eliminates classical key possession as a spending condition. Instead of relying on seeds or ECDSA keys—which can be stolen, cloned, harvested, or broken by quantum attack—PQHD enforces a deterministic, multi-predicate model where all authority comes from post-quantum ML-DSA keys and a strictly verifiable execution context.

PQHD defines a unified custody predicate:

```
valid_for_signing =
      valid_tick
  AND valid_consent
  AND valid_policy
  AND valid_device
  AND valid_quorum
  AND valid_ledger
  AND valid_psbt
```

A signature is only permitted when all seven conditions are true.
If any condition fails, the system enters mandatory fail-closed behaviour and cannot produce a signature. Classical private keys provide zero spending authority after Secure Import.

### **Core Security Mechanisms**

**1. Temporal Authority — `valid_tick`**
PQHD uses signed EpochTicks from the Bitcoin-anchored Epoch Clock. Ticks must be fresh (≤900 seconds), canonically encoded, monotonic, and correctly referenced. This removes reliance on system clocks and eliminates replay attacks by making time a cryptographic condition.

**2. Intent Binding — `valid_consent`**
User intent is captured in a canonical ConsentProof, signed with ML-DSA-65 and bound to:
• a specific transaction (`bundle_hash`)
• a specific session (`exporter_hash`)
• a precise tick window
• a specific device and role
Consent cannot be reused, forwarded, or replayed.

**3. Deterministic Policy Enforcement — `valid_policy`**
All authorization rules—roles, thresholds, limits, destinations, time windows—are evaluated deterministically using canonical structures and EpochTicks. No nondeterministic logic is allowed.

**4. Runtime Integrity — `valid_device`**
Device trust comes exclusively from PQVL attestation. A device is valid only when `drift_state == "NONE"`. Malware, unsafe runtimes, or modified environments immediately block signing.

**5. Deterministic Structure — `valid_psbt` & `valid_ledger`**
PSBTs are canonicalised (sorted, witness-stripped, deterministically encoded) so all devices compute the same `bundle_hash`.
The Local Merkle Ledger records monotonic, append-only events; any divergence freezes the ledger.

**6. Multisig Determinism — `valid_quorum`**
All participants independently evaluate the same predicates. Each device must reach the same decision, preventing coordinator manipulation or cross-device mismatches.

### **Architecture & Compatibility**

PQHD uses ML-DSA-65 for internal signatures and cSHAKE256 for domain-separated deterministic key derivation. Compatibility secp256k1 keys are derived only for script satisfaction.
On-chain behaviour remains fully classical—PQHD produces standard Bitcoin transactions using standard ECDSA/Schnorr signatures.
No consensus changes are required.

Recovery uses ML-KEM-encrypted capsules, tick-bound delays, guardian thresholds, and full predicate enforcement. Secure Import uses dual proof-of-possession to safely migrate classical keys into PQHD without granting them authority.

### **Result**

PQHD ensures that Bitcoin can only be spent when fresh time, explicit intent, deterministic policy, runtime integrity, multisig quorum, ledger continuity, and canonical PSBT equivalence all align. No single compromised assumption—key theft, device compromise, stale state, or coordinator tampering—is sufficient.

This provides quantum-safe, deterministic, replay-proof Bitcoin custody fully compatible with today’s network.

---

# **INDEX**

### **[ABSTRACT](#abstract)**

### **[PROBLEM STATEMENT](#problem-statement)**

---

## **1. PURPOSE AND SCOPE (NORMATIVE)**

* [1.1 Purpose](#11-purpose)
* [1.1A Trustless Temporal Authority (INFORMATIVE)](#11a-trustless-temporal-authority-informative)
* [1.2 Scope](#12-scope)
* [1.3 Relationship to PQSF](#13-relationship-to-pqsf)
* [1.3A Normative Dependencies (NORMATIVE)](#13a-normative-dependencies-normative)
* [1.4 Relationship to PQVL and Epoch Clock](#14-relationship-to-pqvl-and-epoch-clock)
* [1.5 Relationship to-pqai](#15-relationship-to-pqai)
* [1.6 Version Compatibility (NORMATIVE)](#16-version-compatibility-normative)

---

## **2. ARCHITECTURE OVERVIEW (NORMATIVE)**

* [Section 2](#2-architecture-overview-normative)

---

## **3. CRYPTOGRAPHIC PRIMITIVES (NORMATIVE)**

* [3.1 Signatures](#31-signatures)
* [3.2 Hashing & Derivation](#32-hashing--derivation)
* [3.3 Encoding](#33-encoding)
* [3.4 Domain Separation](#34-domain-separation)

---

## **4. DETERMINISTIC KEY HIERARCHY (NORMATIVE)**

* [4.1 Child Key Derivation](#41-child-key-derivation)
* [4.2 Key Classes](#42-key-classes)
* [4.3 Canonical Derivation Paths](#43-canonical-derivation-paths)
* [4.4 Tick-Bound Derivation](#44-tick-bound-derivation)
* [4.5 Device-Bound Key Revocation Semantics (NORMATIVE)](#45-device-bound-key-revocation-semantics-normative)

---

## **5. TEMPORAL AUTHORITY — EPOCH TICK INTEGRATION (NORMATIVE)**

* [5.1 Tick Verification Requirements](#51-tick-verification-requirements)
* [5.2 Canonical EpochTick Structure (NORMATIVE)](#52-canonical-epochtick-structure-normative)

---

## **6. POLICY ENFORCER — POLICY ENFORCEMENT (NORMATIVE)**

* [6.1 Predicate Model](#61-predicate-model)
* [6.2 Policy Object Structure](#62-policy-object-structure)
* [6.3 Time-Based Predicates](#63-time-based-predicates)
* [6.4 Role & Threshold Predicates](#64-role--threshold-predicates)
* [6.5 Destination Policies](#65-destination-policies)
* [6.6 Device Predicates](#66-device-predicates)
* [6.7 Ledger Predicates](#67-ledger-predicates)
* [6.8 PSBT Predicates](#68-psbt-predicates)
* [6.9 Secure Import Predicates](#69-secure-import-predicates)
* [6.10 Anomaly Detection](#610-anomaly-detection)
* [6.11 Custom Predicates](#611-custom-predicates)
* [6.12 Failure Semantics](#612-failure-semantics)
* [6.13 Optional Hardening as Predicate Extensions (NORMATIVE)](#613-optional-hardening-as-predicate-extensions-normative)

---

## **7. CONSENTPROOF (NORMATIVE)**

* [7.1 Structure](#71-structure)
* [7.1.1 Canonical ConsentProof Structure (NORMATIVE)](#711-canonical-consentproof-structure-normative)
* [7.2 Consent Binding](#72-consent-binding)
* [7.3 Expiry](#73-expiry)
* [7.4 Failure Semantics](#74-failure-semantics)

---

## **8. TRANSPORT INTEGRATION (NORMATIVE)**

* [8.1 Exporter Binding](#81-exporter-binding)
* [8.2 TLSE-EMP Requirements](#82-tlse-emp-requirements)
* [8.3 STP Requirements](#83-stp-requirements)
* [8.4 Offline Transport](#84-offline-transport)
* [8.5 Stealth Mode](#85-stealth-mode)
* [8.6 Multi-Device Rules](#86-multi-device-rules)
* [8.7 Replay Protection](#87-replay-protection)
* [8.8 Failure Semantics](#88-failure-semantics)

---

## **9. PSBT INTEGRITY & SIGNING WORKFLOW (NORMATIVE)**

* [9.1 PSBT Canonicalisation](#91-psbt-canonicalisation)
* [9.2 Bundle Hash](#92-bundle-hash)
* [9.3 Structural Validation](#93-structural-validation)
* [9.4 Role & Quorum Validation](#94-role--quorum-validation)
* [9.5 Device Attestation](#95-device-attestation)
* [9.6 Multisig Coordination](#96-multisig-coordination)
* [9.7 Eligibility Predicate](#97-eligibility-predicate)
 * [9.8 Merkle Ledger State Verification (NORMATIVE)](https://www.google.com/search?q=%2398-merkle-ledger-state-verification-normative)
* [9.9 Post-Signing Checks](#99-post-signing-checks)
* [9.10 Failure Semantics](#910-failure-semantics)
* [9.11 Destination Compatibility (NORMATIVE)](#911-destination-compatibility-normative)

---

## **10. RECOVERY CAPSULES, CONTINUITY & DEVICE REPLACEMENT (NORMATIVE)**

* [Section 10](#10-recovery-capsules-continuity--device-replacement-normative)

---

## **11. MULTISIG MODEL (NORMATIVE)**

* [11.1 Overview](#111-overview)
* [11.1 Bitcoin-Facing Compatibility Keys](#111-bitcoin-facing-compatibility-keys)
* [11.2 Role Definitions](#112-role-definitions)
* [11.3 Thresholds](#113-thresholds)
* [11.4 Participant Model](#114-participant-model)
* [11.5 Coordination Protocol](#115-coordination-protocol)
* [11.6 Workflow Determinism](#116-workflow-determinism)
* [11.7 Signing Rules](#117-signing-rules)
* [11.8 Role Rotation](#118-role-rotation)
* [11.9 Forbidden Behaviours](#119-forbidden-behaviours)
* [11.10 Finalisation](#1110-finalisation)

---

## **12. DEVICE ATTESTATION & RUNTIME INTEGRITY (NORMATIVE)**

* [12.1 Device Identity](#121-device-identity)
* [12.2 Attestation Envelope](#122-attestation-envelope)
* [12.2.1 Canonical AttestationEnvelope Structure (NORMATIVE)](#1221-canonical-attestationenvelope-structure-normative)
* [12.3 PQVL Integration](#123-pqvl-integration)
* [12.4 Code Path Determinism](#124-code-path-determinism)
* [12.5 Enclave Requirements](#125-enclave-requirements)
* [12.6 Consent Binding](#126-consent-binding)
* [12.7 Drift Detection](#127-drift-detection)
* [12.8 Multisig Attestation](#128-multisig-attestation)
* [12.9 Time & Attestation Binding](#129-time--attestation-binding)
* [12.10 Forbidden Behaviours](#1210-forbidden-behaviours)
* [12.11 Re-Attestation](#1211-re-attestation)
* [12.12 Ledger Commit Requirements](#1212-ledger-commit-requirements)

---

## **13. STEALTH MODE, OFFLINE MODE & AIR-GAPPED MODE (NORMATIVE)**

* [13.1 Stealth Mode](#131-stealth-mode)
* [13.2 Offline Mode](#132-offline-mode)
* [13.3 Air-Gapped Mode](#133-air-gapped-mode)
* [13.4 Forbidden Behaviours](#134-forbidden-behaviours)
* [13.5 Mode Transitions](#135-mode-transitions)

---

  * [ANNEX A — Cryptographic Parameter Set](https://www.google.com/search?q=%23annex-a--cryptographic-parameter-set)
  * [ANNEX B — Canonical Encoding Specifications](https://www.google.com/search?q=%23annex-b--canonical-encoding-specifications)
  * [ANNEX C — PQHD Error Codes](https://www.google.com/search?q=%23annex-c--pqhd-error-codes)
  * [ANNEX D — Key Derivation Paths](https://www.google.com/search?q=%23annex-d--key-derivation-paths)
  * [ANNEX E — Message Format Specification](https://www.google.com/search?q=%23annex-e--message-format-specification)
  * [ANNEX F — Policy Language Grammar](https://www.google.com/search?q=%23annex-f--policy-language-grammar)
  * [ANNEX G — Transaction Finalization Flow](https://www.google.com/search?q=%23annex-g--transaction-finalization-flow)
  * [ANNEX H — Recovery Protocol Details](https://www.google.com/search?q=%23annex-h--recovery-protocol-details)
  * [ANNEX I — Security Audit Checklist](https://www.google.com/search?q=%23annex-i--security-audit-checklist)
  * [ANNEX J — HD Seed Adoption](https://www.google.com/search?q=%23annex-j--hd-seed-adoption)
  * [ANNEX K — Delegated Authority](https://www.google.com/search?q=%23annex-k--delegated-authority)
  * [ANNEX L — KeyMail](https://www.google.com/search?q=%23annex-l--keymail)
  * [ANNEX M — Universal Secret Derivation](https://www.google.com/search?q=%23annex-m--universal-secret-derivation)
  * [ANNEX N — Credential Vault](https://www.google.com/search?q=%23annex-n--credential-vault)
  * [ANNEX O — Service-Scoped API Key Derivation](https://www.google.com/search?q=%23annex-o--service-scoped-api-key-derivation)
  * [ANNEX P — Verified Identity & KYC Credentials](https://www.google.com/search?q=%23annex-p--verified-identity-kyc-credentials)
  * [ANNEX Q — Delegated Identity](https://www.google.com/search?q=%23annex-q--delegated-identity)
  * [ANNEX R — Delegated Spending](https://www.google.com/search?q=%23annex-r--delegated-spending)
  * [ANNEX S — Multi-Chain Deterministic Interoperability](https://www.google.com/search?q=%23annex-s--multi-chain-deterministic-interoperability)
  * [ANNEX T — Cross-Chain & Layer-2 Integration](https://www.google.com/search?q=%23annex-t--cross-chain-layer-2-integration)
  * [ANNEX U — Deterministic Derivation Security](https://www.google.com/search?q=%23annex-u--deterministic-derivation-security)
  * [ANNEX V — Normative Ecosystem Dependencies](https://www.google.com/search?q=%23annex-v--normative-ecosystem-dependencies)

## **[GLOSSARY](#glossary-canonical)**

## **[ACKNOWLEDGEMENTS](#acknowledgements-informative)**

-----

# **ABSTRACT**

The Post-Quantum Hierarchical Deterministic Wallet (PQHD) eliminates classical-key possession as a spend condition. After Secure Import, compromise of any classical ECDSA keys or legacy seeds does not grant spending authority. All authorisation derives solely from PQHD’s post-quantum key hierarchy and the unified custody predicate, in which a signature may be produced only when all conditions evaluate to true:

```
valid_for_signing =
valid_tick
AND valid_consent
AND valid_policy
AND valid_device
AND valid_quorum
AND valid_ledger
AND valid_psbt
```

If any predicate (valid_tick, valid_consent, valid_policy, valid_device,
valid_quorum, valid_ledger, valid_psbt) evaluates to false, PQHD MUST set:

```
valid_for_signing = false
```

and MUST enter fail-closed behaviour without exception.

PQHD derives ML-DSA key hierarchies from a dedicated PQHD root and treats classical key material as non-authoritative. Classical keys cannot satisfy any predicate or influence multisig authority.

Replay attacks are structurally eliminated. Every consent, PSBT, session, device attestation, and signing attempt is bound to fresh, monotonic EpochTicks, canonical structure, exporter-bound session data, and the exact transaction bundle hash, preventing reuse across time windows or sessions.

Internally, PQHD integrates verifiable time from the Epoch Clock, intent binding through ConsentProof, deterministic rule evaluation through the Policy Enforcer, runtime integrity through PQVL, and deterministic transaction assembly via canonical CBOR or JCS JSON encoding. All cryptographic operations use standardised primitives including ML-DSA-65 signatures and cSHAKE256-based domain-separated key derivation. These components operate under strict fail-closed semantics, eliminating attack classes involving stale time, replayed consent, policy drift, runtime compromise, coordinator manipulation, multisig divergence, ledger rollback, or classical ECDSA key recovery.

Despite these internal guarantees, all on-chain behaviour remains fully classical. PQHD produces standard Bitcoin transactions using secp256k1 public keys and classical ECDSA or Schnorr signatures. Bitcoin-facing keys are deterministic compatibility keys derived only for script construction and witness satisfaction; no PQHD seeds, ML-DSA keys, predicate results, or internal state appear on-chain.

PQHD does not require any changes to Bitcoin consensus. It is fully compatible with current signature rules and produces standard classical transactions. The only scenario that would require a Bitcoin consensus change is a hypothetical quantum computer capable of deriving a secp256k1 private key within the 10 minute mempool window. That scenario cannot be solved by any wallet and is therefore outside the scope of PQHD. Until such a scenario exists, PQHD protects funds against all quantum enabled attacks that can be solved at the wallet layer.

PQHD provides reproducible, deterministic behaviour across devices, programming languages and operating environments, including offline, air gapped, multisig, enterprise and sovereign deployments. The result is a custody architecture in which Bitcoin remains protected even under complete classical key compromise, runtime compromise, replay attempts or system level manipulation, without requiring any changes to Bitcoin consensus.

---


# **PROBLEM STATEMENT**

Possession of a private key is no longer a reliable foundation for Bitcoin security. Classical wallets rely on assumptions that no longer hold under modern threat conditions: private keys can be stolen, leaked, cloned, harvested from memory, misused by malware, or recovered by quantum attack. In these models, any party who obtains a seed or private key can produce a valid signature, regardless of time, device integrity, policy state, or user intent.

Runtime compromise further undermines classical custody. Malicious processes, altered environments, injected libraries, or compromised coordinators can cause a device to produce signatures the user did not intend. Without a verifiable model of runtime integrity, wallets cannot determine whether the signer is behaving correctly.

Replay and stale-state attacks remain feasible because classical wallets depend on system clocks, unverified timestamps, reusable signatures, and unbound PSBTs. An adversary can reuse outdated intentions, resubmit transactions, or manipulate coordinator flows if time and intent are not cryptographically bound.

Cross-device multisig introduces its own failure modes. Devices frequently build PSBTs differently, produce divergent hashes, or evaluate inconsistent spending rules. Without deterministic PSBT construction and canonical encoding, multisig participants can disagree on the structure of the transaction they are asked to sign. Similarly, ledger rollback, state drift, and inconsistent policy evaluation can cause multisig to desynchronise or be manipulated by a malicious coordinator.

These weaknesses share a common cause: classical wallets treat time, intent, policy, quorum, runtime integrity, and transaction structure as external assumptions rather than verifiable cryptographic conditions. As a result, any one assumption can be exploited to bypass intended controls.

PQHD addresses these systemic failures by replacing key possession with a deterministic, multi-predicate custody model. Signing requires fresh verifiable time, explicit cryptographic intent, deterministic policy evaluation, attested runtime integrity, multisig quorum correctness, continuous ledger history, and canonical PSBT equivalence. Replay is eliminated through tick-bound windows, session-bound consent, and strict monotonicity. Runtime compromise is addressed through mandatory PQVL attestation. Classical key compromise is rendered irrelevant because classical keys never grant authority. All on-chain behaviour remains fully classical, ensuring compatibility with existing Bitcoin consensus rules.

The result is a custody model in which no single assumption—keys, devices, coordinators, time sources, or scripts—can be exploited to produce an unauthorised signature.

---

# **1. PURPOSE AND SCOPE (NORMATIVE)**

## **1.1 Purpose**

PQHD provides deterministic rules ensuring that Bitcoin can only be spent when:

• time is verified through signed EpochTicks,
• user intent is captured through canonical ConsentProof,
• authorisation rules are satisfied precisely,
• device runtime integrity is validated,
• multisig roles and thresholds are met,
• the PSBT is structurally sound and identical across devices,
• the wallet’s state is continuous and monotonic,
• keys are derived deterministically and safely.

These mechanisms ensure that signatures cannot be produced under stale time, forged intent, compromised runtime, inconsistent PSBTs, or policy-violating conditions.

## **1.1A Trustless Temporal Authority (INFORMATIVE)**
PQHD relies on the Epoch Clock for temporal authority. Time validation never
depends on central servers, cloud infrastructure, DNS, NTP, or any local system
clock. All security properties derive exclusively from:

• the Bitcoin-inscribed Epoch Clock profile,
• ML-DSA-65 signature verification,
• deterministic CBOR/JCS canonical encoding, and
• mirror consensus across independent servers.

Implementations may verify, self-host, or fork mirror infrastructure without
permission or coordination from any operator. All temporal checks are performed
locally using signed EpochTicks and the canonical profile reference.

The canonical Epoch Clock v2.0 profile referenced by PQHD is:

profile_ref = "ordinal:439d7ab1972803dd984bf7d5f05af6d9f369cf52197440e6dda1d9a2ef59b6ebi0"

(Informative) Non-authoritative public explorers:

https://ordinals.com/inscription/439d7ab1972803dd984bf7d5f05af6d9f369cf52197440e6dda1d9a2ef59b6ebi0
https://bestinslot.xyz/ordinals/inscription/439d7ab1972803dd984bf7d5f05af6d9f369cf52197440e6dda1d9a2ef59b6ebi0
https://www.ord.io/439d7ab1972803dd984bf7d5f05af6d9f369cf52197440e6dda1d9a2ef59b6ebi0

Explorer URLs are non-authoritative. Implementations MUST validate the canonical
profile using the on-chain inscription referenced by profile_ref.

## **1.2 Scope**

PQHD defines:

• deterministic key hierarchy (PQHD-DF),
• tick-bound authorisation windows,
• canonical PSBT construction and validation,
• multisig and role semantics,
• recovery capsules and Secure Import flows,
• PQVL-based device integrity validation,
• deterministic Merkle ledger construction,
• behaviour under offline, air-gapped, and Stealth Mode operation,
• optional annex modules (A–T).

PQHD does not define Bitcoin consensus rules, PSBT base semantics, or UI/UX design.

---

# **WHAT THIS SPECIFICATION COVERS**

1. **Custody Predicate Model**
   The complete set of conditions determining whether signing is permitted.

2. **Deterministic Key Hierarchy**
   Domain-separated cSHAKE256 derivations for all key classes.

3. **Temporal Authority**
   Integration of EpochTicks for freshness, monotonicity, and replay resistance.

4. **Intent Binding**
   Canonical ConsentProof with tick windows, exporter_hash, and device identity fields.

5. **Policy Enforcement**
   Deterministic rule evaluation for thresholds, roles, time windows, and destination rules.

6. **Runtime Integrity**
   PQVL attestation ensuring device integrity before authorisation.

7. **PSBT Integrity**
   Canonicalisation, bundle_hash calculation, and cross-device deterministic equality.

8. **Ledger Model**
   Deterministic, Merkle-secured, monotonic event history.

9. **Transport Integration**
   Replay-resistant, exporter-bound communication using TLSE-EMP or STP.

10. **Multisig Workflow**
    Deterministic cross-device evaluation and signing.

11. **Recovery & Secure Import**
    Deterministic, time-bound, and integrity-checked recovery mechanisms.

12. **Annex Modules**
    Optional components that do not affect custody predicates.

---

# **1.3 Relationship to PQSF**

PQHD relies on PQSF for:

• **EpochTick definitions and validation rules**,
• **ConsentProof structure and semantics**,
• **Policy Enforcer architecture**,
• **deterministic transport requirements**,
• **canonical object encoding**,
• **Merkle ledger structure**,
• **shared predicate semantics**.

PQHD implements wallet-level behaviour consistent with these foundations.

### Pseudocode (Informative)

```
// Use PQSF’s canonical tick validation as a dependency
function pqhd_validate_tick_with_pqsf(tick, last_valid_tick):
    if not PQSF.verify_tick_signature(tick):
        return false
    if not PQSF.verify_tick_canonical_encoding(tick):
        return false
    if PQSF.is_tick_stale(tick, window_seconds = 900):
        return false
    if tick.t < last_valid_tick:
        return false
    return true
```

---

# **1.3A Normative Dependencies (NORMATIVE)**

PQHD implementations MUST satisfy the following external normative dependencies:

1. **PQSF v1.0.0** — defines the unified custody predicate structure:
   valid_tick, valid_consent, valid_policy, valid_device/valid_runtime,
   valid_quorum, valid_ledger, and valid_psbt.

2. **Epoch Clock v2.0.0** — provides the authoritative, signed temporal model.
   PQHD MUST enforce EpochTick signature validity, canonical encoding, freshness
   ≤900 seconds, monotonicity, and correct profile_ref.

3. **PQVL v1.0.0** — provides canonical runtime and device integrity. PQHD MUST
   enforce:
       valid_device := (drift_state == "NONE")

4. **PQAI v1.0.0** (optional) — defines SafePrompt and AI runtime verification.
   These mechanisms MAY assist intent validation but MUST NOT weaken any PQHD
   predicate.

Failure of any dependency predicate MUST result in:
valid_for_signing = false
with no fallback or override permitted.

---

# **1.4 Relationship to PQVL and Epoch Clock**

Device integrity in PQHD is determined exclusively by PQVL.
A device is considered valid for signing only when:

```
valid_device = (PQVL.drift_state == "NONE")
```

Temporal authority is derived exclusively from Epoch Clock v2.0.0.
Every tick MUST satisfy:

• ML-DSA-65 signature validity,
• canonical encoding,
• freshness ≤ 900 seconds,
• monotonicity relative to the last valid tick,
• matching profile_ref:

```
ordinal:439d7ab1972803dd984bf7d5f05af6d9f369cf52197440e6dda1d9a2ef59b6ebi0
```

No system clock or local time source may be used for authorisation.

### Pseudocode (Informative)

```
// Combine Epoch Clock and PQVL into a simple device + time predicate
function pqhd_valid_device_and_time(attestation, tick, last_tick):
    if attestation.drift_state != "NONE":
        return false

    if not verify_ml_dsa_65(epoch_profile_pubkey, canonical(tick), tick.sig):
        return false
    if tick.profile_ref != canonical_profile_ref:
        return false
    if tick.t < last_tick:
        return false
    if tick_age_seconds(tick) > 900:
        return false

    return true
```

---

# **1.5 Relationship to PQAI**

PQAI MAY support high-assurance intent confirmation through SafePrompt, provided:

• it does not override custody predicates,
• it operates using tick-fresh, canonical input,
• it respects device integrity and temporal validity,
• it complements rather than modifies policy rules.

### Pseudocode (Informative)

```
function pqhd_allow_safe_prompt(safe_prompt_context):
    if not safe_prompt_context.valid_tick:
        return false
    if not safe_prompt_context.valid_device:   // via PQVL
        return false
    if not safe_prompt_context.valid_consent:  // normal ConsentProof rules
        return false

    return true
```

---

# **1.6 Version Compatibility (NORMATIVE)**

PQHD v1.0.0 requires the following minimum ecosystem versions:

• PQSF v1.0.0 or later  
• Epoch Clock v2.0.0 or later  
• PQVL v1.0.0 or later  
• PQAI v1.0.0 or later (if AI/SafePrompt features are enabled)

A PQHD implementation MUST NOT claim conformance unless all required dependency
versions meet or exceed these minima.

---

# **2. ARCHITECTURE OVERVIEW (NORMATIVE)**

PQHD consists of deterministic layers:

1. **Key Hierarchy Layer**
   cSHAKE256-based PQHD-DF derivation.

2. **Temporal Layer**
   EpochTick validation for all time-dependent behaviour.

3. **Intent Layer**
   ConsentProof capture and validation.

4. **Policy Layer**
   Deterministic evaluation of authorisation rules.

5. **Device Layer**
   PQVL attestation for runtime integrity.

6. **Encoding Layer**
   Deterministic CBOR or JCS JSON for all signed/hashed structures.

7. **Ledger Layer**
   Deterministic Merkle model ensuring continuous, monotonic wallet state.

8. **Execution Layer**
   Canonical PSBT processing and signature generation.

All layers MUST behave identically across platforms, programming languages, and signing devices.

### Pseudocode (Informative)

```
// High-level “full pipeline” evaluation of the PQHD predicates
function pqhd_evaluate_all_predicates(ctx):
    predicates = {}

    predicates.valid_tick    = validate_tick(ctx.tick, ctx.last_tick)
    predicates.valid_consent = validate_consent(ctx.consent, ctx.tick, ctx.session)
    predicates.valid_policy  = evaluate_policy(ctx.policy, ctx)
    predicates.valid_device  = validate_device_attestation(ctx.attestation)
    predicates.valid_quorum  = validate_quorum(ctx.multisig_state)
    predicates.valid_ledger  = validate_ledger_state(ctx.ledger, ctx.tick)
    predicates.valid_psbt    = validate_psbt_structure(ctx.psbt, ctx.consent)

    ctx.valid_for_signing = (
        predicates.valid_tick    and
        predicates.valid_consent and
        predicates.valid_policy  and
        predicates.valid_device  and
        predicates.valid_quorum  and
        predicates.valid_ledger  and
        predicates.valid_psbt
    )

    return ctx
```

---

# **3. CRYPTOGRAPHIC PRIMITIVES (NORMATIVE)**

## **3.1 Signatures**

PQHD uses:

• **ML-DSA-65** for all internal PQHD signatures, including ConsentProof, ledger entries, recovery capsules, delegated authority, governance, and device attestation.
• **ECDSA** only for validating classical Proof-of-Possession during Secure Import.

### Pseudocode (Informative)

```
// Wrapper around the required ML-DSA-65 signature verification
function verify_pq_signature(pubkey_pq, payload_bytes, signature_pq):
    // payload_bytes MUST be the canonical encoding (CBOR/JCS) of the object
    return MLDSA65.verify(pubkey_pq, payload_bytes, signature_pq)

function sign_pq(private_key_pq, payload_bytes):
    // behaviour: ML-DSA-65 sign; returns signature bytes
    return MLDSA65.sign(private_key_pq, payload_bytes)
```

## **3.2 Hashing & Derivation**

• SHAKE256-256 — hashing
• cSHAKE256 — deterministic key derivation (PQHD-DF)
• ML-KEM-768/1024 — optional encrypted payloads

### Pseudocode (Informative)

```
// Canonical SHAKE-based hashing, always 256 bits
function shake256_256(bytes):
    return SHAKE256_256(bytes)

// Generic cSHAKE-based derivation with explicit domain separation
function cshake256_derive(key, domain_string, info_bytes):
    return cSHAKE256(key, domain = domain_string, info = info_bytes)
```

## **3.3 Encoding**

All PQHD data structures MUST use deterministic encoding:

• deterministic CBOR (RFC 8949 §4.2) or
• JCS JSON (RFC 8785)

### Pseudocode (Informative)

```
// Canonical encoder abstraction
function canonical_encode(obj):
    if ENCODING_MODE == "CBOR":
        return deterministic_cbor_encode(obj)
    else if ENCODING_MODE == "JCS_JSON":
        return jcs_json_encode(obj)

// Hashing a canonical representation
function hash_canonical(obj):
    return shake256_256(canonical_encode(obj))
```

## **3.4 Domain Separation**

All cSHAKE256 derivations MUST use explicit domain strings such as:

```
PQHD-Child-Key-consent
PQHD-Child-Key-governance
PQHD-Child-Key-recovery
PQHD-Device-Attestation
PQHD-BundleHash
PQHD-SecureImport
```

Domain separation ensures cross-class isolation.

### Pseudocode (Informative)

```
// Structured domain derivation helper
function pqhd_derive_with_domain(root_key, label, info_bytes):
    domain = "PQHD-" + label
    return cshake256_derive(root_key, domain, info_bytes)
```

---

# **4. DETERMINISTIC KEY HIERARCHY (NORMATIVE)**

PQHD uses a deterministic root:

```
root_key = SHAKE256-256(seed)
```

The seed is 256-bit CSPRNG entropy generated once and never exported.

### Pseudocode (Informative)

```
// Initialise a PQHD root from 256-bit entropy
function pqhd_init_root(seed_256_bits):
    // seed_256_bits must be 32 random bytes
    return shake256_256(seed_256_bits)
```

## **4.1 Child Key Derivation**

Child keys MUST be derived using:

```
child_key = cSHAKE256(
    root_key,
    domain = "PQHD-Child-Key-" || class_id,
    info   = derivation_path || context_bytes
)
```

The derivation MUST be deterministic across devices and independent of runtime conditions.

### Pseudocode (Informative)

```
// Deterministic child-key derivation for any key class
function pqhd_derive_child(root_key, class_id, derivation_path, context_bytes):
    domain = "PQHD-Child-Key-" + class_id
    info   = concat(utf8(derivation_path), context_bytes)
    return cshake256_derive(root_key, domain, info)
```

## **4.2 Key Classes**

Mandatory key classes include:

• governance
• audit
• identity
• consent
• delegation
• vault (DEK/KEK)
• device-bound
• attestation
• recovery
• chain (Annex S)

Additional classes may be defined in annex modules.

## **4.3 Canonical Derivation Paths**

Paths MUST be deterministic and canonical. Examples:

```
/consent/0/0
/device-bound/deviceA/0
/recovery/guardianA/0
/chain/eth/0
```

### Pseudocode (Informative)

```
// Helper to build a canonical string path from components
function build_path(segments):
    // segments is an array like ["consent","0","0"]
    return "/" + join("/", segments)
```

## **4.4 Tick-Bound Derivation**

Time-sensitive keys MUST incorporate the Epoch Tick value:

• consent keys
• session keys
• recovery delay keys
• Secure Import state keys
• device-runtime keys

This ensures resistance to temporal replay.

### Pseudocode (Informative)

```
// Derive a key that is explicitly bound to a tick t
function derive_tick_bound_key(root_key, class_id, base_path, tick):
    // tick is an EpochTick object
    tick_bytes = uint64_to_bytes(tick.t)
    path = base_path
    domain = "PQHD-Child-Key-" + class_id
    info = concat(utf8(path), tick_bytes)
    return cshake256_derive(root_key, domain, info)
```

## **4.5 Device-Bound Key Revocation Semantics (NORMATIVE)**

Device-Bound Keys are deterministically derivable via PQHD-DF and are therefore, in principle, recomputable from the PQHD root. However, derivability MUST NOT imply authority. PQHD MUST treat Device-Bound Keys associated with lost, destroyed, compromised, or decommissioned devices as permanently revoked, regardless of their derivability.

1. Permanent Revocation  
When a device is marked as lost, destroyed, compromised, or decommissioned:
• its Device-Bound Key MUST be treated as permanently revoked;  
• it MUST NOT be reused or re-associated with new hardware;  
• it MUST NOT satisfy any part of valid_device or valid_quorum.

2. No Rebinding  
Even though revoked Device-Bound Keys may be derivable from the root, implementations MUST NOT:
• re-attest a revoked key,  
• re-enrol it under new hardware,  
• include it in any role or quorum configuration,  
• accept it during or after recovery.

3. Replacement Devices  
Replacement devices MUST:
• derive a fresh Device-Bound Key using a new derivation path or context;  
• provide a fresh PQVL AttestationEnvelope satisfying all requirements in §12 and Annex B.

A replacement device MUST NOT be treated as continuity of a revoked device.

4. Ledger Recording  
Revocation and subsequent new-device enrolment MUST generate governance events in the Local Merkle Ledger:
• device_revoked  
• device_enrolled

5. PQVL Integration (NORMATIVE)  
A device presenting a PQVL AttestationEnvelope whose device_id corresponds to a revoked Device-Bound Key MUST be treated as:

    drift_state = "CRITICAL"

This MUST force:

    valid_device = false  
    valid_quorum = false  
    valid_for_signing = false

Revoked devices MUST NOT regain authority through attestation.

---

# **5. TEMPORAL AUTHORITY — EPOCH TICK INTEGRATION (NORMATIVE)**

All temporal validation in PQHD uses the canonical Epoch Clock profile defined
in §1.1A. Implementations MUST NOT substitute any other profile_ref unless a
profile migration event is recorded in the ledger as defined in Annex A.

PQHD uses EpochTicks for all time-based authorisations.

An operation MAY proceed only if:

```
valid_tick = true
```

### Pseudocode (Informative)

```
// High-level tick validation for PQHD
function validate_tick(tick, last_valid_tick):
    // 1. Check signature is valid
    if not verify_ml_dsa_65(epoch_profile_pubkey, canonical_encode(tick_without_sig(tick)), tick.sig):
        return false

    // 2. Check profile_ref
    if tick.profile_ref != canonical_profile_ref:
        return false

    // 3. Check monotonicity
    if tick.t < last_valid_tick:
        return false

    // 4. Check freshness (<= 900 seconds)
    if tick_age_seconds(tick) > 900:
        return false

    return true
```

## **5.1 Tick Verification Requirements**

PQHD MUST verify:

1. **ML-DSA-65 signature**
2. **canonical encoding**
3. **tick freshness ≤ 900 seconds**
4. **monotonicity** (tick_current ≥ last_valid_tick)
5. **profile_ref correctness**
6. **mirror consensus** when online (≥2 matching ticks)

If any condition is false:

```
valid_tick = false
valid_for_signing = false
```

No override is permitted.

---

# **5.2 Canonical EpochTick Structure (NORMATIVE)**

PQHD relies on the canonical EpochTick object defined in Annex A.1 for all temporal validation:

EpochTick = {
t: uint, ; Strict Unix Time
profile_ref: tstr, ; canonical Epoch Clock profile reference
alg: tstr, ; MUST be "ML-DSA-65"
sig: bstr ; ML-DSA-65 signature
}

Requirements:

t MUST represent Strict Unix Time (seconds since 1970-01-01T00:00:00Z, ignoring leap seconds).

profile_ref MUST match the canonical Epoch Clock v2.0.0 profile defined in Annex A.1:

ordinal:439d7ab1972803dd984bf7d5f05af6d9f369cf52197440e6dda1d9a2ef59b6ebi0

sig MUST be a valid ML-DSA-65 signature over the canonical payload.

Object MUST be encoded using deterministic CBOR or JCS JSON.

Pseudocode (Informative)
```
// Canonicalise exactly the fields of EpochTick for signing/verifying
function canonical_tick_payload(tick):
    payload = {
        t:           tick.t,
        profile_ref: tick.profile_ref,
        alg:         tick.alg
        // sig excluded from payload when computing/validating signature
    }
    return canonical_encode(payload)
```
---

# **6. POLICY ENFORCER — POLICY ENFORCEMENT (NORMATIVE)**

The Policy Enforcer evaluates all authorisation rules deterministically.
Policy evaluation contributes the predicate:

```
valid_policy = true | false
```

Policy rules MUST NOT contain nondeterministic logic. All conditions MUST be expressible through canonical, reproducible structures.

## **6.1 Predicate Model**

```
valid_policy =
      satisfy_roles
  AND satisfy_thresholds
  AND satisfy_destinations
  AND satisfy_limits
  AND satisfy_time_windows
  AND satisfy_device_attestation
  AND satisfy_ledger_continuity
  AND satisfy_psbt_integrity
  AND satisfy_recovery_rules
  AND satisfy_custom_predicates
```

If ANY component evaluates to false, then:

```
valid_policy = false
```

### Pseudocode (Informative)

```
// Deterministic policy evaluation across all sub-predicates
function evaluate_policy(policy, ctx):
    if not satisfy_roles(policy, ctx):              return false
    if not satisfy_thresholds(policy, ctx):         return false
    if not satisfy_destinations(policy, ctx):       return false
    if not satisfy_limits(policy, ctx):             return false
    if not satisfy_time_windows(policy, ctx):       return false
    if not satisfy_device_attestation(policy, ctx): return false
    if not satisfy_ledger_continuity(policy, ctx):  return false
    if not satisfy_psbt_integrity(policy, ctx):     return false
    if not satisfy_recovery_rules(policy, ctx):     return false
    if not satisfy_custom_predicates(policy, ctx):  return false

    return true
```

## **6.2 Policy Object Structure**

A policy MUST be canonicalised before evaluation.

Required fields include:

• roles
• thresholds
• allowlists / denylists
• time windows
• spending limits
• cooldown configuration
• destination requirements
• recovery rules
• optional deterministic custom predicates

### Pseudocode (Informative)

```
// Canonical policy hashing (e.g. for policy_hash used elsewhere)
function compute_policy_hash(policy):
    return hash_canonical(policy)  // SHAKE256-256(canonical(policy))
```

## **6.3 Time-Based Predicates**

Time-based rules MUST be evaluated using the EpochTick. Examples:

• minimum delay between actions
• expiry windows
• session-duration constraints
• recovery delay enforcement

System time MUST NOT be used.

### Pseudocode (Informative)

```
// Example: enforce a minimum delay and an expiry window using EpochTicks
function satisfy_time_windows(policy, ctx):
    tick_value = ctx.tick.t

    if policy.min_delay_from_tick is not null:
        if tick_value < ctx.reference_tick.t + policy.min_delay_from_tick:
            return false

    if policy.expiry_tick is not null:
        if tick_value > policy.expiry_tick:
            return false

    return true
```

---

# **6.3.1 EXTENDABLE TIMELOCK POLICY (NORMATIVE)**

*(See also PROBLEM STATEMENT)*

The Extendable Timelock Policy provides a deterministic, time-based mechanism for controlled delay during specified threat conditions.
When a threat is detected but cannot be immediately resolved, the wallet enters a **timelocked state** and signing is not permitted. The policy ensures fail-closed behaviour without requiring permanent lockout.

Threat detection is performed by the Policy Enforcer or a defined anomaly-detection module as specified in §6.2.

A timelock contributes directly to:

```
valid_policy = true | false
```

A timelock MUST block signing whenever `current_tick < unlock_at` or when extension rules apply. No user, administrator, or guardian MAY bypass or shorten an active timelock.

---

## **TimelockState Structure**

```
TimelockState {
  threat_id:        tstr,
  created_at:       uint,   ; EpochTick.t
  unlock_at:        uint,   ; EpochTick.t when signing may resume
  status:           "active" | "released" | "blocked",
  extension_count:  uint
}
```

Timelock state MUST be canonical, deterministic, and stored in wallet state.

---

## **Threat Conditions That Trigger Timelock**

Timelock activation is permitted only for temporary or uncertain threat conditions, including:

* rate-limit anomalies
* tick inconsistencies
* device-attestation irregularities
* consent-pattern anomalies
* suspicious behaviour or anomaly-detection triggers

These conditions signal uncertainty without confirmed compromise.

---

## **Timelock Creation and Extension Rules**

### **A. Threat active AND no existing timelock**

A new `TimelockState` MUST be created:

```
status = "active"
created_at = current_tick.t
unlock_at = current_tick.t + timelock_duration
extension_count = 0
```

Signing MUST be denied.

---

### **B. Threat active AND existing timelock expired**

If `current_tick.t ≥ unlock_at` and the threat remains active:

* timelock MUST extend,
* `extension_count` MUST increment,
* `unlock_at` MUST update to:

```
unlock_at = current_tick.t + timelock_extension_duration
```

Signing MUST continue to be denied.

---

### **C. Threat cleared AND timelock expired**

If the threat has cleared and `current_tick.t ≥ unlock_at`:

```
status = "released"
```

Signing MAY proceed once all other predicates evaluate to true.

---

### **D. Current tick < unlock_at**

Signing MUST remain denied.

---

## **Maximum Extensions**

If `extension_count ≥ max_extensions`:

* If `auto_release = true`, the timelock MUST release automatically.
* If `auto_release = false`, timelock MUST transition to:

```
status = "blocked"
```

A blocked timelock requires manual recovery or guardian procedures as specified in §10 (Recovery Capsules).

---

## **Policy Integration**

The timelock contributes to policy evaluation:

```
valid_policy =
      base_policy_predicates
  AND timelock_condition_satisfied
```

Where:

```
timelock_condition_satisfied =
      (status == "released")
  OR  (status == "active" AND current_tick.t ≥ unlock_at AND threat_cleared)
```

If neither condition holds:

```
valid_policy = false
```

---

## **Fail-Closed Semantics**

During any active or extended timelock:

```
valid_policy = false
valid_for_signing = false
```

No alternate predicate path or manual override is permitted.

---

## **Informative Example — Timelock Check**

```pseudo
function evaluate_timelock(tl, current_tick):
    if tl.status == "active" and current_tick < tl.unlock_at:
        return false
    if tl.status == "blocked":
        return false
    return true
```
---

## **6.4 Role & Threshold Predicates**

Roles MUST be deterministic and ordered.
Threshold predicates MUST confirm that the required number of authorised roles have valid signatures.

### Pseudocode (Informative)

```
// Check that required roles are present and thresholds met
function satisfy_roles(policy, ctx):
    // ctx.signers is a map role -> set of signer_ids
    for role in policy.roles:
        if role not in ctx.signers:
            return false
    return true

function satisfy_thresholds(policy, ctx):
    for role, required_count in policy.thresholds.items():
        actual = count(ctx.signers[role])
        if actual < required_count:
            return false
    return true
```

## **6.5 Destination Policies**

If a destination is subject to allowlist rules:
• It MUST appear in the allowlist.

If a destination is subject to denylist rules:
• It MUST NOT appear in the denylist.

These checks MUST be deterministic.

### Pseudocode (Informative)

```
// Deterministic allow/deny check on PSBT outputs
function satisfy_destinations(policy, ctx):
    dests = ctx.psbt_output_destinations   // array of destination identifiers

    for dest in dests:
        if policy.allowlist and dest not in policy.allowlist:
            return false
        if dest in policy.denylist:
            return false

    return true
```

## **6.6 Device Predicates**

Policy MUST confirm that:

```
PQVL.drift_state == "NONE"
```

If drift_state is not NONE, valid_policy MUST become false.

### Pseudocode (Informative)

```
// Mirror the global valid_device requirement inside policy evaluation
function satisfy_device_attestation(policy, ctx):
    return (ctx.attestation.drift_state == "NONE")
```

## **6.7 Ledger Predicates**

Policy MUST require:

• monotonic ledger,
• correct prev_hash,
• no freeze state unless explicitly required for recovery,
• correct ordering relative to ticks.

### Pseudocode (Informative)

```
// Require that the device’s local ledger view is consistent
function satisfy_ledger_continuity(policy, ctx):
    if ctx.ledger.frozen and not policy.allow_frozen_for_recovery:
        return false

    if not ctx.ledger.tick_monotonic:
        return false

    if ctx.ledger.prev_hash != ctx.expected_prev_hash:
        return false

    return true
```

## **6.8 PSBT Predicates**

The canonical PSBT MUST be structurally valid and MUST match the bundle_hash bound inside the active ConsentProof.

A mismatch MUST set:

```
valid_policy = false
```

### Pseudocode (Informative)

```
// Policy layer re-checks structural integrity and consent binding
function satisfy_psbt_integrity(policy, ctx):
    if not ctx.psbt_structurally_valid:
        return false
    if ctx.bundle_hash != ctx.consent.bundle_hash:
        return false
    return true
```

## **6.9 Secure Import Predicates**

When Secure Import occurs, policy MUST require:

• tick freshness,
• dual Proof-of-Possession,
• appropriate role authority,
• any configured guardian approvals.

### Pseudocode (Informative)

```
// High-level import-specific policy predicate
function satisfy_recovery_rules(policy, ctx):
    if not ctx.is_secure_import:
        return true  // not applicable

    if not ctx.import_classical_pop_valid:
        return false
    if not ctx.import_pq_pop_valid:
        return false
    if policy.guardian_threshold is not null:
        if ctx.guardian_approvals < policy.guardian_threshold:
            return false

    return true
```

## **6.10 Anomaly Detection**

Anomaly logic MUST be deterministic if enabled.
Heuristic or environment-driven logic MUST NOT be used.

### Pseudocode (Informative)

```
// Placeholder for deterministic custom predicates
function satisfy_custom_predicates(policy, ctx):
    // Custom predicates must be pure functions of canonical inputs
    for rule in policy.custom_predicates:
        if not rule.evaluate(ctx):
            return false
    return true
```

## **6.11 Custom Predicates**

Custom rules MUST be deterministic, canonical, and reproducible across devices.

## **6.12 Failure Semantics**

If any policy rule fails:

• the wallet MUST fail-closed,
• no signature MAY be produced,
• a ledger event documenting the failure MUST be appended (if possible).

### Pseudocode (Informative)

```
// Fail-closed semantics for policy violations
function policy_fail_closed(ctx, error_code):
    ctx.valid_for_signing = false
    append_ledger_event("policy_failure", ctx.tick, { error: error_code })
    return false
```

# **6.13 Optional Hardening as Predicate Extensions (NORMATIVE)**

Implementations MAY define optional hardening mechanisms such as external cosigners, mandatory device-bound conditions, guardian co-authorisation, or extended time-locks.

These mechanisms MUST comply with the following rules:

1. Predicate Integration  
Optional hardening MUST be represented exclusively as additional constraints inside existing predicates, such as:
    • valid_policy  
    • valid_quorum  

Optional hardening MUST NOT define new predicates or override existing ones.

2. No Bypass  
Optional hardening MUST NOT permit signing when:
    valid_for_signing = false

3. Strengthening Only  
Optional hardening MAY increase the number of required conditions but MAY NOT relax or replace any existing condition.

4. Logical Structure Enforcement (NORMATIVE)  
All optional hardening MUST use conjunctive (AND) logic.  
Disjunctive (OR) logic — where satisfying a hardening condition could bypass a failing mandatory predicate — is expressly forbidden.

---

# **7. CONSENTPROOF (NORMATIVE)**

ConsentProof binds user intent to a specific transaction, under a specific tick window, on a specific device, in a specific session.

A ConsentProof MUST be canonical, signed with ML-DSA-65, and MUST be bound to:

• bundle_hash
• exporter_hash
• tick_issued
• tick_expiry
• device_id
• role
• quorum_index

## **7.1 Structure**

```
ConsentProof {
  action,
  intent_hash,
  bundle_hash,
  tick_issued,
  tick_expiry,
  exporter_hash,
  device_id,
  role,
  quorum_index,
  signature_pq
}
```

## **7.1.1 Canonical ConsentProof Structure (NORMATIVE)**

A ConsentProof captures user intent bound to a specific PSBT, session, device, and tick window. The canonical structure is defined in Annex C.1 and is restated here:

ConsentProof = {
action: tstr,
intent_hash: bstr,
bundle_hash: bstr,
tick_issued: uint,
tick_expiry: uint,
exporter_hash: bstr,
device_id: tstr,
role: tstr,
quorum_index: uint,
signature_pq: bstr ; ML-DSA-65
}

### Pseudocode (Informative)

// Build and sign a canonical ConsentProof object
function build_consent_proof(action, intent_bytes, bundle_hash, tick, session, device_id, role, quorum_index, signing_key):
    cp = {
        action:        action,
        intent_hash:   shake256_256(intent_bytes),
        bundle_hash:   bundle_hash,
        tick_issued:   tick.t,
        tick_expiry:   tick.t + 900,  // example window
        exporter_hash: session.exporter_hash,
        device_id:     device_id,
        role:          role,
        quorum_index:  quorum_index
    }
    bytes = canonical_encode(cp)
    cp.signature_pq = sign_pq(signing_key, bytes)
    return cp

## **7.2 Consent Binding**

ConsentProof binds intent to:

1. a specific PSBT (via bundle_hash)
2. a specific session (via exporter_hash)
3. a specific tick window (tick_issued → tick_expiry)
4. a specific device
5. a specific participant role

## **7.3 Expiry**

If:

```
tick_current < tick_issued
OR
tick_current > tick_expiry
```

then:

```
valid_consent = false
```

Consent MUST NOT be reused outside its tick window.

### Pseudocode (Informative)

```
// Validate that consent is within its allowed tick window
function validate_consent_window(consent, current_tick):
    if current_tick.t < consent.tick_issued:
        return false
    if current_tick.t > consent.tick_expiry:
        return false
    return true
```

## **7.4 Failure Semantics**

Invalid consent MUST block signing and SHOULD produce a ledger entry documenting the failure.

### Pseudocode (Informative)

```
// Full consent validation including exporter and bundle binding
function validate_consent(consent, current_tick, session, bundle_hash):
    if not validate_consent_window(consent, current_tick):
        append_ledger_event("consent_expired", current_tick, { consent: consent })
        return false

    if consent.exporter_hash != session.exporter_hash:
        append_ledger_event("consent_exporter_mismatch", current_tick, { consent: consent })
        return false

    if consent.bundle_hash != bundle_hash:
        append_ledger_event("consent_bundle_mismatch", current_tick, { consent: consent })
        return false

    // Verify signature
    bytes = canonical_encode(consent_without_signature(consent))
    if not verify_pq_signature(consent_pubkey(consent), bytes, consent.signature_pq):
        append_ledger_event("consent_sig_invalid", current_tick, { consent: consent })
        return false

    return true
```

---

# **8. TRANSPORT INTEGRATION (NORMATIVE)**

PQHD uses PQSF transports:

• **TLSE-EMP** for deterministic TLS-enforced sessions
• **STP** for DNS-free, CA-free sovereign transport

Transport-layer binding ensures replay resistance and session specificity.

## **8.1 Exporter Binding**

All ConsentProof objects MUST satisfy:

```
ConsentProof.exporter_hash == session.exporter_hash
```

Session exporter_hash MUST be fresh and deterministic.

### Pseudocode (Informative)

```
// Check exporter binding between session and ConsentProof
function validate_exporter_binding(consent, session):
    return (consent.exporter_hash == session.exporter_hash)
```

## **8.2 TLSE-EMP Requirements**

TLSE-EMP MUST provide:

• PQ-safe handshake
• exporter binding
• canonical message framing
• deterministic transcript

## **8.3 STP Requirements**

STP MUST provide:

• DNS-free operation
• deterministic message formatting
• STP identity binding
• strong replay resistance

## **8.4 Offline Transport**

Offline environments MAY use:

• QR
• USB
• serial links

These transports MUST preserve canonical encoding and MUST enforce tick freshness before signing.

### Pseudocode (Informative)

```
// Example offline envelope validation
function validate_offline_envelope(envelope, current_tick):
    // envelope.payload_bytes is canonical-encoded
    if tick_age_seconds(current_tick) > 900:
        return false
    if not verify_pq_signature(envelope.sender_pubkey, envelope.payload_bytes, envelope.signature_pq):
        return false
    return true
```

## **8.5 Stealth Mode**

Stealth Mode MUST use:

• STP-only
• no DNS
• no TLSE-EMP
• strict tick reuse ≤ 900 seconds

Reconciliation MUST occur on exit.

## **8.6 Multi-Device Rules**

Every participating device MUST independently evaluate:

• valid_tick
• valid_consent
• valid_policy
• valid_device
• valid_quorum
• valid_ledger
• valid_psbt

### Pseudocode (Informative)

```
// Each device independently re-evaluates the full predicate set
function device_evaluate_predicates(device_ctx):
    device_ctx = pqhd_evaluate_all_predicates(device_ctx)
    return device_ctx.valid_for_signing
```

## **8.7 Replay Protection**

Transport envelopes MUST include:

• tick
• exporter_hash
• canonical encoding

Envelope replay MUST be rejected.

### Pseudocode (Informative)

```
// Detect replay by combining exporter_hash and tick
function is_replay(envelope, replay_cache):
    key = concat(envelope.exporter_hash, uint64_to_bytes(envelope.tick.t))
    if key in replay_cache:
        return true
    replay_cache.add(key)
    return false
```

## **8.8 Failure Semantics**

Any transport-layer mismatch MUST:

• invalidate consent,
• invalidate the session,
• fail-closed.

---

# **9. PSBT INTEGRITY & SIGNING WORKFLOW (NORMATIVE)**

Signing MUST be deterministic.

The canonical PSBT is constructed through a deterministic pipeline, resulting in:

```
bundle_hash = SHAKE256-256(canonical_psbt)
```

All devices MUST compute the identical canonical_psbt and identical bundle_hash.

## **9.1 PSBT Canonicalisation**

Canonicalisation includes:

• lexicographic ordering of inputs
• lexicographic ordering of outputs
• witness stripping
• sorted script descriptors
• canonical encoding (CBOR/JCS)

### Pseudocode (Informative)

```
// Canonicalise a PSBT before hashing/broadcast
function canonicalise_psbt(psbt):
    psbt_no_witness = strip_witness(psbt)
    psbt_no_witness.inputs  = sort_inputs_lex(psbt_no_witness.inputs)
    psbt_no_witness.outputs = sort_outputs_lex(psbt_no_witness.outputs)
    psbt_no_witness.scripts = sort_scripts_lex(psbt_no_witness.scripts)
    return canonical_encode(psbt_no_witness)
```

## **9.2 Bundle Hash**

```
bundle_hash = SHAKE256-256(canonical_psbt)
```

ConsentProof MUST incorporate this bundle_hash.

A mismatch MUST result in:

```
valid_psbt = false
```

### Pseudocode (Informative)

```
// Compute bundle hash and compare against ConsentProof
function compute_bundle_hash(canonical_psbt_bytes):
    return shake256_256(canonical_psbt_bytes)

function validate_bundle_hash(psbt, consent):
    canonical_psbt = canonicalise_psbt(psbt)
    bh = compute_bundle_hash(canonical_psbt)
    return (bh == consent.bundle_hash)
```

## **9.3 Structural Validation**

The PSBT MUST have:

• correct inputs,
• correct outputs,
• correct script types,
• correct fee calculation,
• no malleation,
• no extraneous metadata.

### Pseudocode (Informative)

```
// Example structural PSBT checks (simplified)
function validate_psbt_structure(psbt):
    if psbt.inputs.length == 0:  return false
    if psbt.outputs.length == 0: return false

    if not check_fee_bounds(psbt):
        return false

    if has_unknown_fields(psbt):
        return false

    return true
```

## **9.4 Role & Quorum Validation**

Required roles MUST contribute signatures as defined by policy.

## **9.5 Device Attestation**

Before signing, PQVL attestation MUST confirm:

```
drift_state == "NONE"
```

## **9.6 Multisig Coordination**

Every device MUST compute:

• identical canonical_psbt
• identical bundle_hash
• identical predicate results

### Pseudocode (Informative)

```
// Multisig device signing workflow
function multisig_device_sign(device_ctx, psbt):
    canonical_psbt = canonicalise_psbt(psbt)
    bundle_hash    = compute_bundle_hash(canonical_psbt)

    if not validate_bundle_hash(psbt, device_ctx.consent):
        return error("E_BUNDLE_HASH_MISMATCH")

    device_ctx.bundle_hash = bundle_hash
    device_ctx = pqhd_evaluate_all_predicates(device_ctx)

    if not device_ctx.valid_for_signing:
        return error("E_PREDICATES_FAILED")

    signing_key = derive_tick_bound_key(
        device_ctx.root_key,
        "consent",
        device_ctx.role_path,
        device_ctx.tick
    )

    signature = sign_psbt(canonical_psbt, signing_key)
    return signature
```

## **9.7 Eligibility Predicate**

Signing MAY occur only if:

```
valid_for_signing = true
```

9.8 Signature Production (NORMATIVE)

Once valid_for_signing = true, PQHD MUST perform a two-layer signing process, separating:

Internal post-quantum custody authorisation, and

External classical Bitcoin signatures used for consensus.

9.8.1 Internal PQ SignIntent (custody proof)

Before producing any consensus-valid signature, PQHD MUST construct a canonical SignIntent object that captures the full internal context of the spend:

• canonical PSBT bundle_hash,
• current EpochTick,
• ConsentProof identifier,
• device identity and runtime attestation reference,
• active policy_hash,
• multisig role/quorum information.

PQHD MUST sign this SignIntent using an ML-DSA-65 child key derived through PQHD-DF and MUST record the resulting signature inside the local PQHD ledger.

This ML-DSA-65 signature:

• is authoritative for custody,
• is internal to PQHD,
• MUST NOT appear in any Bitcoin PSBT, scriptSig, witness, annex, Taproot leaf, or any other consensus-reachable field.

9.8.2 External classical Bitcoin signatures (consensus layer)

After the internal SignIntent has been produced and recorded, PQHD MUST:

derive or select the appropriate classical secp256k1 compatibility key(s) associated with the inputs;

generate deterministic ECDSA or Schnorr signatures over the canonical PSBT input hashes;

attach these classical signatures to the PSBT in the standard Bitcoin witness and/or scriptSig fields.

PQHD MUST NOT emit ML-DSA-65 signatures in any structure interpreted by Bitcoin consensus rules.

From the Bitcoin network’s perspective, PQHD-constructed transactions MUST be indistinguishable from standard classical wallets.

Pseudocode (Informative)
function pqhd_produce_signatures(ctx, psbt):
    assert ctx.valid_for_signing == true

    canonical_psbt = canonicalise_psbt(psbt)
    psbt_hash      = hash(canonical_psbt)

    // 1. Internal PQ SignIntent
    sign_intent = {
        psbt_hash:    psbt_hash,
        tick:         ctx.tick.t,
        consent_id:   ctx.consent.id,
        device_id:    ctx.device.id,
        policy_hash:  ctx.policy.hash,
        quorum_slice: ctx.quorum_slice_id
    }

    sign_intent.signature_pq =
        sign_ml_dsa(ctx.role_pq_key, canonical(sign_intent))

    ledger_append_sign_intent(ctx.ledger, sign_intent)

    // 2. Classical Bitcoin signatures for consensus
    classical_key = ctx.classical_signing_key
    btc_sigs      = btc_sign_psbt_inputs(canonical_psbt, classical_key)

    psbt_with_sigs = attach_btc_signatures(canonical_psbt, btc_sigs)
    return (psbt_with_sigs, sign_intent)


## **9.9 Post-Signing Checks**

After signing:

• recompute bundle_hash
• verify deterministic structure
• update ledger

### Pseudocode (Informative)

```
// Post-signing verification and ledger commit
function post_signing_update(ctx, psbt, signature):
    canonical_psbt = canonicalise_psbt(psbt)
    new_bundle_hash = compute_bundle_hash(canonical_psbt)

    if new_bundle_hash != ctx.bundle_hash:
        return error("E_PSBT_CANONICALIZATION_FAILED")

    append_ledger_event("sign", ctx.tick, {
        bundle_hash: new_bundle_hash,
        role: ctx.role,
        device_id: ctx.device_id
    })

    return true
```

## **9.10 Failure Semantics**

If any condition fails, the signer MUST fail-closed.

## **9.11 Destination Compatibility (NORMATIVE)**

PQHD does not impose restrictions on the destination scripts or address formats of PSBT outputs. A PQHD-controlled spend may send to any consensus-valid Bitcoin output, including classical wallets, legacy ECDSA-based addresses, native SegWit outputs, Taproot outputs, or non-PQHD multisig scripts. Destination addresses are treated as opaque data for the purpose of custody; PQHD enforces authority over the originating inputs only.

PQHD MUST always emit standard Bitcoin transactions whose scriptSig and/or witness fields contain only secp256k1-based ECDSA or Schnorr signatures. All post-quantum ML-DSA-65 signatures are strictly internal to PQHD (e.g., SignIntent, ConsentProof, EpochTick, AttestationEnvelope, ledger entries) and MUST NOT appear in or be required by any Bitcoin consensus-interpreted structure.

Policy rules may optionally apply allowlists, denylists, or destination constraints, but the PQHD custody model does not require the recipient to use PQHD, post-quantum key material, or any specific script type. All destination script forms accepted by Bitcoin consensus are valid outputs for PQHD-constructed transactions.

This requirement ensures PQHD remains fully interoperable with the existing Bitcoin ecosystem and does not mandate any changes to consensus or address formats.

---

# **10. RECOVERY CAPSULES, CONTINUITY & DEVICE REPLACEMENT (NORMATIVE)**

Recovery is a governance action and MUST be secured under the same unified custody predicate as spending. A recovery operation MAY proceed only when all of the following evaluate to true:

    valid_tick
AND valid_consent
AND valid_policy
AND valid_device
AND valid_quorum
AND valid_ledger
AND valid_psbt     // when PSBT-based artefacts are used

Recovery MUST NOT introduce a weaker authorisation model or alternate predicate path.
All recovery actions MUST generate governance entries in the Local Merkle Ledger.

10.1 Capsule Structure
RecoveryCapsule = {
  capsule_id,
  guardian_set,
  threshold,
  encrypted_material,
  capsule_hash,
  recovery_delay_seconds,
  creation_tick,
  valid_from_tick,
  signature_pq
}
Requirements:
* encrypted_material MUST use ML-KEM
* signature_pq MUST use ML-DSA-65
* MUST be canonically encoded
* creation_tick and valid_from_tick MUST derive from EpochTicks

10.2 Guardian Sets & Thresholds
1 ≤ threshold ≤ len(guardian_set)
Guardian set and threshold MUST be canonical, policy-defined, and recorded in the ledger.

10.3 Recovery Delay Windows
Activation is permitted only when:
tick_current ≥ creation_tick + recovery_delay_seconds
EpochTicks MUST be used. System time MUST NOT be used.

10.4 Guardian Approval
GuardianApproval = {
  capsule_id,
  guardian_id,
  approval_tick,
  approval_sig
}
Threshold approvals MUST:
* be ML-DSA-signed;
* use valid EpochTicks;
* be evaluated under all predicates.

10.5 Root Reconstruction via Recovery Capsules
Recovery Capsules MAY encapsulate sufficient deterministic material to reconstruct the PQHD root.
Reconstruction is permitted only when:
* the capsule is canonical and hash-valid;
* threshold approvals satisfy policy;
* ledger continuity is verified;
* the full predicate set evaluates to true;
* runtime integrity (valid_device) is satisfied;
* reconstruction occurs inside the secure vault.
Ledger Continuity Requirement (NORMATIVE)
Root reconstruction MUST validate ledger continuity. If reconciliation fails due to:
* prev_hash mismatch,
* Merkle root divergence,
* non-monotonic tick sequence,
the operation MUST fail-closed with:
E_LEDGER_DIVERGENCE
Recovery MUST NOT rebase, rewrite, or discard past ledger history.

10.6 Deterministic Key Hierarchy Restoration
After root reconstruction, the following MUST be regenerated deterministically and match previous values exactly:
* Governance Keys
* Custody Keys
* Consent & Delegation Keys
* Vault DEK/KEK
* Recovery Keys
* Chain/L2 keys (Annex S/T)
* Policy derivation keys
Continuity MUST preserve multisig roles, thresholds, and policy behaviour.

10.7 Independence From Original Devices
All previously enrolled devices MUST be treated as revoked unless individually re-enrolled and re-attested.
PQHD MUST NOT attempt to restore or reactivate any prior Device-Bound Key.

10.8 Post-Recovery Device Re-Enrolment
Replacement devices MUST:
* derive new Device-Bound Keys;
* present a fresh PQVL AttestationEnvelope;
* be incorporated into policy via governance updates;
* produce ledger events.
No device is eligible until attestation and governance updates complete.

10.9 Device Loss Without Account Recovery
If only one device is lost and the wallet is otherwise intact:
1. Mark device as revoked (ledger event).
2. Enrol a new device with a fresh key.
3. Require PQVL attestation.
4. Update policy and quorum.
The PQHD root and key hierarchy remain unchanged.

10.10 Replacing an Attested Device in a Multisig
A replacement device MUST:
* derive a new Device-Bound Key;
* pass PQVL attestation;
* be added to the quorum via governance;
* generate ledger events;
* NOT modify L2 custody keys or policy except for device-role mapping.

10.11 Policy Continuity Guarantee
After recovery:
* all prior policies remain intact;
* Governance Keys retain full authority;
* newly enrolled devices MAY sign under prior policies;
* all modifications MUST generate ledger events.

10.12 Security Semantics
1. Possession of the PQHD root or Governance Keys provides full authority; optional hardening MAY add constraints but MUST NOT override predicates.
2. Recovery Capsules MUST NOT expose root plaintext.
3. Full predicate evaluation is mandatory for all recovery actions.
4. All actions MUST be auditable via the Merkle ledger.

10.13 Forbidden Behaviours
Implementations MUST NOT:
* perform unilateral or policy-bypassing recovery
* bypass delay windows
* resurrect revoked Device-Bound Keys
* rewrite ledger history
* ignore any predicate failures
* create alternate recovery-only predicates

---


# **11. MULTISIG MODEL (NORMATIVE)**

Multisig behaviour MUST be deterministic across devices.

## **11.1 Overview**

A multisig configuration comprises:

• participants
• roles
• thresholds
• device-role bindings
• tick-bound keys

## **11.1 Bitcoin-Facing Compatibility Keys**

PQHD multisig scripts presented to the Bitcoin network MUST use standard secp256k1 public keys and MUST produce classical ECDSA or Schnorr signatures compatible with existing Bitcoin consensus rules. These Bitcoin-facing public keys are deterministic compatibility keys derived from the PQHD root solely for script construction and witness generation.

Spending authority derives exclusively from PQHD’s ML-DSA-65 tick-bound child keys and the unified custody predicate. Compatibility ECDSA keys:

MUST NOT grant spending authority,

MUST NOT be derived from or depend on classical seeds,

MUST exist only to satisfy Bitcoin script and witness formats,

MUST remain isolated from all PQHD internal predicate logic.

Miners and validating nodes observe only classical multisig structures and standard secp256k1 signatures. PQHD internal seeds, ML-DSA keys, predicate evaluations, attestation data, and tick state MUST NOT appear on-chain.

## **11.2 Role Definitions**

Mandatory roles:

• primary
• secondary
• guardian

Optional roles:

• auditor
• operator
• observer
• federation

## **11.3 Thresholds**

Thresholds MUST be deterministic per role.

## **11.4 Participant Model**

Each participant MUST have:

• a device-bound key
• attested device integrity
• deterministic role assignment

## **11.5 Coordination Protocol**

Each participant MUST:

• fetch tick
• validate PSBT
• validate consent
• validate policy
• validate ledger
• validate attestation
• produce identical predicate results

### Pseudocode (Informative)

```
// Per-participant multisig coordination step
function multisig_participant_step(ctx, psbt):
    ctx.tick = get_fresh_tick()
    ctx.psbt = psbt
    ctx = pqhd_evaluate_all_predicates(ctx)
    return ctx.valid_for_signing
```

## **11.6 Workflow Determinism**

Devices MUST arrive at identical decisions.
Any mismatch MUST fail-closed.

## **11.7 Signing Rules**

Sign ONLY with:

• tick-bound child key
• correct role
• valid_for_signing == true

## **11.8 Role Rotation**

Role rotation MUST be deterministic, tick-bound, and ledger-anchored.

### Pseudocode (Informative)

```
// Tick-bound role rotation
function rotate_role(current_role_binding, new_roles, tick):
    // current_role_binding and new_roles are canonical maps: device_id -> role
    event = {
        old_roles:  current_role_binding,
        new_roles:  new_roles,
        tick_value: tick.t
    }
    append_ledger_event("role_rotation", tick, event)
    return new_roles
```

## **11.9 Forbidden Behaviours**

• stale ticks
• signature reuse
• mid-session role modification

## **11.10 Finalisation**

All signatures MUST be validated cross-device.

---

# **12. DEVICE ATTESTATION & RUNTIME INTEGRITY (NORMATIVE)**

PQHD integrates PQVL for runtime integrity.

## **12.1 Device Identity**

Each device MUST derive a device-bound key through PQHD-DF.

## **12.2 Attestation Envelope**

```
DeviceAttestation = {
  device_id,
  device_key,
  measurement_hash,
  enclave_state,
  pqvl_probes,
  signature_pq
}
```

## **12.2.1 Canonical AttestationEnvelope Structure (NORMATIVE)**

Devices MUST provide a canonical, signed runtime integrity object. The canonical PQVL AttestationEnvelope structure is defined in Annex B.1, with ProbeResult defined in Annex B.3, and is restated here:

AttestationEnvelope = {
envelope_id: tstr,
device_id: tstr,
measurement_hash: bstr,
drift_state: tstr, ; "NONE" | "WARNING" | "CRITICAL"
probes: [* ProbeResult],
tick: EpochTick,
signature_pq: bstr ; ML-DSA-65
}

Required semantics:

drift_state MUST be evaluated as:

"NONE" → environment valid

"WARNING" → non-fatal deviation

"CRITICAL" → integrity failure

PQHD MUST compute:

valid_device = (drift_state == "NONE")

Envelope MUST be encoded canonically.

Tick MUST satisfy PQHD tick validation rules and the EpochTick requirements in Annex A.1.

All required probes MUST match expected canonical values.

ProbeResult Structure (NORMATIVE, see Annex B.3)
ProbeResult = {
probe_id: tstr,
result: tstr,
expected: tstr,
signature_pq: bstr
}

### Pseudocode (Informative)

// PQHD-side validation of PQVL attestation envelope
function validate_device_attestation(att_env, last_tick):
    // 1. Canonical encoding check
    if canonical_encode(att_env) != att_env.raw_bytes:
        return false

    // 2. Signature verification
    bytes = canonical_encode(att_env_without_sig(att_env))
    if not verify_pq_signature(pqvl_attest_pubkey(att_env.device_id), bytes, att_env.signature_pq):
        return false

    // 3. Tick validation (using Epoch Clock rules from §5.2 / Annex A.1)
    if not validate_tick(att_env.tick, last_tick):
        return false

    // 4. Drift state
    if att_env.drift_state != "NONE":
        return false

    // 5. Probe checks (Annex B.3)
    for probe in att_env.probes:
        if probe.result != probe.expected:
            return false

    return true

## **12.3 PQVL Integration**

Device validity is:

```
valid_device = (PQVL.drift_state == "NONE")
```

## **12.4 Code Path Determinism**

Signer MUST use a canonical, reproducible code path.

## **12.5 Enclave Requirements**

If used, enclaves MUST enforce code integrity.

## **12.6 Consent Binding**

ConsentProof.device_id MUST match the attested device.

## **12.7 Drift Detection**

Any drift MUST invalidate valid_device.

## **12.8 Multisig Attestation**

Participants MUST validate each other’s attestation.

## **12.9 Time & Attestation Binding**

Attestation MUST include a fresh EpochTick.

## **12.10 Forbidden Behaviours**

• signing without attestation
• dynamic code injection

## **12.11 Re-Attestation**

Required when:

• tick expires
• policy changes
• ledger diverges

## **12.12 Ledger Commit Requirements**

Attestations MUST record appropriate ledger events.

### Pseudocode (Informative)

```
// Attach attestation event to ledger
function record_attestation_event(att_env):
    append_ledger_event("device_attested", att_env.tick, {
        device_id: att_env.device_id,
        drift_state: att_env.drift_state,
        measurement_hash: att_env.measurement_hash
    })
```

---

# **13. STEALTH MODE, OFFLINE MODE & AIR-GAPPED MODE (NORMATIVE)**

These modes support environments where network connectivity is restricted or unavailable.
All behaviour MUST remain deterministic and MUST respect the custody predicate.

## **13.1 Stealth Mode**

Stealth Mode is designed for DNS-free, CA-free, low-visibility environments.

Rules:

• MUST use **STP-only** transport
• MUST NOT use DNS
• MUST NOT use TLSE-EMP
• EpochTicks MAY be reused for **≤900 seconds**
• profile rotation MUST NOT occur
• ledger updates MUST continue normally
• exit requires reconciliation (below)

### 13.1.1 Exit Requirements

To exit Stealth Mode, the device MUST:

1. Validate ≥2 matching ticks from mirrors
2. Validate profile_ref
3. Validate profile lineage
4. Reconcile ledger roots with peers
5. Re-attest device integrity (PQVL)

Signing MAY resume only after the exit workflow completes.

### Pseudocode (Informative)

```
// Stealth Mode exit workflow
function exit_stealth_mode(ctx):
    ticks = fetch_ticks_from_mirrors(count = 2)
    if not all_ticks_match(ticks):
        return error("E_TICK_DIVERGENCE")

    if not validate_profile_lineage(ticks[0].profile_ref):
        return error("E_PROFILE_MISMATCH")

    if not reconcile_ledger_roots(ctx):
        return error("E_LEDGER_DIVERGENCE")

    attestation = PQVL.get_attestation(ctx.device_id)
    if not validate_device_attestation(attestation, ctx.last_tick):
        return error("E_DEVICE_ATTESTATION_FAILED")

    ctx.mode = "normal"
    return true
```

---

## **13.2 Offline Mode**

Offline clients MUST:

• rely solely on cached ticks ≤900 seconds old
• store PSBTs canonically for QR/USB export
• require fresh tick and attestation before signing once online
• treat all cached state as untrusted once reconnected

System/local time MUST NOT be used.

### Pseudocode (Informative)

```
// Check whether an offline client may continue using its cached tick
function offline_can_proceed(cached_tick, current_elapsed_seconds):
    if current_elapsed_seconds > 900:
        return false
    return true
```

---

## **13.3 Air-Gapped Mode**

Air-gapped signers MUST:

• operate without network stack
• transfer PSBTs using QR, USB, or serial
• validate tick freshness prior to signing
• require re-attestation and re-ticking on reconnect

---

## **13.4 Forbidden Behaviours**

All restricted modes MUST forbid:

• synthetic ticks
• local/system time fallback
• unverified PSBT modifications
• role reassignment without full reconciliation

---

## **13.5 Mode Transitions**

Transitioning between modes MUST require:

1. fresh EpochTicks
2. ledger reconciliation
3. PQVL re-attestation
4. canonical policy refresh
5. consent regeneration if active

Signing MUST NOT continue across mode boundaries without these steps.

### Pseudocode (Informative)

```
// Generic mode transition guard
function transition_mode(ctx, new_mode):
    new_tick = get_fresh_tick()
    if not validate_tick(new_tick, ctx.last_tick):
        return error("E_TICK_INVALID")

    if not reconcile_ledger_roots(ctx):
        return error("E_LEDGER_DIVERGENCE")

    att = PQVL.get_attestation(ctx.device_id)
    if not validate_device_attestation(att, ctx.last_tick):
        return error("E_DEVICE_ATTESTATION_FAILED")

    ctx.policy = refresh_policy_snapshot(ctx)
    ctx.consent = null  // require regeneration

    ctx.mode = new_mode
    ctx.last_tick = new_tick
    return true
```

---

# **14. SECURE IMPORT (NORMATIVE)**

Secure Import is the only authorised method for migrating classical ECDSA keys or seeds into PQHD.

## **14.1 Threat Model**

Attackers may possess classical keys.
Secure Import prevents misuse by requiring:

• dual Proof-of-Possession,
• fresh ticks,
• deterministic mapping to PQHD keys,
• policy approval,
• optional guardian approval,
• PQVL-validated runtime integrity.

## **14.2 Migration Envelope**

```
SecureImportEnvelope = {
  legacy_pubkey,
  legacy_signature,
  pqhd_pubkey,
  pq_signature,
  migration_tick,
  exporter_hash,
  migration_policy,
  signature_pq
}
```

Rules:

• legacy_signature MUST prove control of classical key
• pq_signature MUST be ML-DSA-65
• canonical encoding required
• migration_tick MUST be fresh

### Pseudocode (Informative)

```
// High-level validation of a Secure Import envelope
function validate_secure_import_envelope(env, current_tick, session):
    if not validate_tick(env.migration_tick, current_tick.t):
        return error("E_TICK_INVALID")

    if env.exporter_hash != session.exporter_hash:
        return error("E_EXPORTER_MISMATCH")

    if not verify_ecdsa_signature(env.legacy_pubkey, env.legacy_challenge, env.legacy_signature):
        return error("E_IMPORT_CLASSICAL_SIG_INVALID")

    if not verify_pq_signature(env.pqhd_pubkey, env.pq_challenge_bytes, env.pq_signature):
        return error("E_IMPORT_PQ_SIG_INVALID")

    bytes = canonical_encode(env_without_sig(env))
    if not verify_pq_signature(import_policy_pubkey, bytes, env.signature_pq):
        return error("E_IMPORT_MISMATCH")

    return true
```

## **14.3 Legacy Proof-of-Possession**

ECDSA PoP MUST verify successfully.
This demonstrates legitimate control over the classical key.

## **14.4 PQHD Proof-of-Identity**

pq_signature binds the PQHD identity key to the import process.

## **14.5 Tick Requirements**

The migration_tick MUST satisfy:

• freshness ≤ 900 seconds
• canonical encoding
• ML-DSA-65 signature validity
• profile_ref correctness
• monotonicity

## **14.6 Consent Requirements**

```
ConsentProof.action = "secure_import"
```

Consent MUST be tick-fresh and exporter-bound.

## **14.7 Policy Requirements**

Policy MUST approve the import.
This may include:

• specific roles,
• limits on key classes allowed to import,
• guardian thresholds.

## **14.8 Guardian Approval (Optional)**

Policies MAY require guardian signatures for high-value migrations.

## **14.9 Deterministic Mapping**

PQHD MUST derive a mapped key using deterministic derivation:

```
mapped_key = PQHD-DF("secure-import", legacy_pubkey || index)
```

Mapping MUST be:

• deterministic
• reproducible
• domain-separated

### Pseudocode (Informative)

```
// Deterministically map a legacy public key into the PQHD keyspace
function derive_secure_import_key(root_key, legacy_pubkey, index):
    context = concat(legacy_pubkey, uint32_to_bytes(index))
    return pqhd_derive_with_domain(root_key, "SecureImport", context)
```

## **14.10 Activation**

Secure Import completes only when:

```
valid_for_signing = true
```

and all import-specific predicates succeed.

## **14.11 Post-Import Restrictions**

Classical keys MUST NOT be reused for any signing purpose.

## **14.12 Ledger Commit**

The event:

```
secure_import_completed
```

MUST be written to the ledger.

## **14.13 Failure Semantics**

Any invalid predicate MUST cause fail-closed behaviour.

## **14.14 Runtime Integrity Requirements**

PQVL MUST confirm drift_state == "NONE" throughout import.

---

# **15. IDENTITY & CREDENTIAL EXTENSIONS (NORMATIVE–OPTIONAL)**

Identity and credential features are defined in annexes and do NOT influence the custody predicate.
They MAY be implemented independently without affecting PQHD signing requirements.

These features include:

• Credential Vault (Annex N)
• Service-scoped API keys (Annex O)
• Verified Identity & KYC Credentials (Annex P)
• Delegated Identity (Annex Q)

All such modules MUST remain isolated from signing authority.

---

# **16. LEDGER RULES & MERKLE CONSTRUCTION (NORMATIVE)**

The ledger provides an append-only, deterministic record of wallet state.

## **16.1 Ledger Entry Structure**

```
LedgerEntry = {
  event,
  epoch_tick,
  entry_tick,
  payload,
  signature_pq,
  prev_hash
}
```

Rules:

• MUST be canonicalised
• MUST be signed with ML-DSA-65
• prev_hash MUST match previous Merkle root

### Pseudocode (Informative)

```
// Build and sign a ledger entry
function build_ledger_entry(event, tick, payload, prev_root, signing_key):
    entry = {
        event:       event,
        epoch_tick:  tick,
        entry_tick:  tick.t,
        payload:     payload,
        prev_hash:   prev_root
    }
    bytes = canonical_encode(entry)
    entry.signature_pq = sign_pq(signing_key, bytes)
    return entry
```

## **16.2 Merkle Construction**

```
leaf_hash = SHAKE256-256(0x00 || canonical_entry)
node_hash = SHAKE256-256(0x01 || left || right)
```

Merkle construction MUST be deterministic.

### Pseudocode (Informative)

```
// Deterministic Merkle-tree update
function ledger_leaf_hash(entry):
    return shake256_256(concat(0x00, canonical_encode(entry)))

function merkle_node_hash(left, right):
    return shake256_256(concat(0x01, left, right))

function append_to_merkle_tree(root, entry):
    leaf = ledger_leaf_hash(entry)
    // Implementation-specific tree structure omitted
    new_root = merkle_append(root, leaf)  // deterministic
    return new_root
```

## **16.3 Append Rules**

Before appending:

• tick MUST be fresh
• prev_hash MUST match current Merkle root
• ledger MUST NOT be frozen
• canonical encoding MUST pass

### Pseudocode (Informative)

```
// Append entry if and only if ledger invariants hold
function append_ledger_entry(ledger, entry, current_tick):
    if ledger.frozen:
        return error("E_LEDGER_FROZEN")

    if not validate_tick(current_tick, ledger.last_tick):
        return error("E_TICK_INVALID")

    if entry.prev_hash != ledger.root:
        ledger.frozen = true
        return error("E_LEDGER_MISMATCH")

    new_root = append_to_merkle_tree(ledger.root, entry)
    ledger.entries.append(entry)
    ledger.root = new_root
    ledger.last_tick = current_tick.t

    return true
```

## **16.4 Freeze Rules**

The ledger MUST freeze if:

• tick rollback detected
• prev_hash mismatch
• Merkle mismatch
• PQVL drift
• profile mismatch
• tick reuse >900 seconds

## **16.5 Cross-Device Reconciliation**

Devices MUST:

1. exchange Merkle roots
2. exchange inclusion proofs
3. reconstruct authoritative history
4. validate monotonic tick sequence
5. ensure canonical structure

### Pseudocode (Informative)

```
// Simplified reconciliation skeleton
function reconcile_ledger_roots(ctx):
    peer_roots = fetch_peer_ledger_roots()
    all_roots = peer_roots + [ctx.ledger.root]
    if not all_equal(all_roots):
        return false
    // Optional: exchange proofs and rebuild history
    return true
```

## **16.6 Ledger Event Types**

Events MAY include:

• tick events
• consent events
• signing
• device attestation
• recovery events
• Secure Import
• governance actions

## **16.7 Ordering**

Ticks MUST be monotonic across the ledger.

## **16.8 State Limits**

Retention MAY be configurable, but critical events MUST remain.

## **16.9 Export/Import**

Exports MUST use canonical bundles with signatures.

## **16.10 Forbidden Behaviours**

• reordering entries
• overwriting entries
• unsigned entries
• ledger modification outside append path

---

# **17. ERROR CODES & FAILURE MODES (NORMATIVE)**

PQHD MUST implement explicit deterministic error codes.

## **17.1 Tick Errors**

• E_TICK_INVALID
• E_TICK_EXPIRED
• E_TICK_ROLLBACK

## **17.2 Consent Errors**

• E_CONSENT_EXPIRED
• E_CONSENT_REVOKED
• E_CONSENT_EXPORTER_MISMATCH

## **17.3 Policy Errors**

• E_POLICY_FAILED
• E_POLICY_THRESHOLD_FAILED
• E_POLICY_DESTINATION_DENIED

## **17.4 Device & Attestation Errors**

• E_DEVICE_ATTESTATION_FAILED
• E_DEVICE_DRIFT_DETECTED
• E_DEVICE_INTEGRITY_FAILED

## **17.5 PSBT Errors**

• E_PSBT_INVALID
• E_PSBT_CANONICALIZATION_FAILED
• E_BUNDLE_HASH_MISMATCH

## **17.6 Quorum Errors**

• E_QUORUM_NOT_MET
• E_ROLE_INVALID

## **17.7 Ledger Errors**

• E_LEDGER_MISMATCH
• E_LEDGER_FROZEN
• E_LEDGER_DIVERGENCE

## **17.8 Secure Import Errors**

• E_IMPORT_CLASSICAL_SIG_INVALID
• E_IMPORT_PQ_SIG_INVALID
• E_IMPORT_MISMATCH

## **17.9 Recovery Errors**

• E_RECOVERY_THRESHOLD_UNMET
• E_RECOVERY_DELAY_ACTIVE
• E_RECOVERY_CAPSULE_INVALID

## **17.10 Transport Errors**

• E_EXPORTER_MISMATCH
• E_TRANSPORT_INVALID

### **FAIL_CLOSED (Authoritative)**

If ANY predicate fails, the system MUST:

• block signing,
• freeze ledger append,
• invalidate the session,
• require fresh tick + attestation before recovery.

### Pseudocode (Informative)

```
// Centralised fail-closed handler
function fail_closed(ctx, error_code):
    ctx.valid_for_signing = false
    ctx.session_valid = false
    ctx.ledger.frozen = true
    append_ledger_event("fail_closed", ctx.tick, { error: error_code })
    return error(error_code)
```

---

# **18. IMPLEMENTATION GUIDANCE (INFORMATIVE)**

Recommended engineering practices:

• modular architecture
• reproducible build processes
• canonical encoding fuzzing
• drift detection testing
• stack separation between policy, consent, and signing
• robust ledger integrity verification
• strict STP/TLSE-EMP usage

• External Service Mocks (INFORMATIVE):
Implementations MAY provide local mock providers for the Epoch Clock and PQVL attestation layers during development. These mocks SHOULD emit deterministic EpochTick and AttestationEnvelope objects conforming to the structures and validation rules defined in §5.2, §12.2.1, Annex A.1 (Temporal Authority — EpochTick Structure), Annex B.1 (Runtime Integrity — AttestationEnvelope), and Annex C.1 (ConsentProof Structure).

---

# **19. SECURITY CONSIDERATIONS (INFORMATIVE)**

PQHD ensures:

• quantum-safe signatures
• replay-resistant tick windows
• deterministic key derivation
• verified runtime integrity
• deterministic transaction assembly
• monotonic ledger state
• fail-closed behaviour under uncertainty

---

# **20. PRIVACY CONSIDERATIONS (INFORMATIVE)**

PQHD avoids exposing identity or behavioural metadata:

• no PII in ledger
• no device fingerprint leakage
• no destination leakage beyond PSBT
• no correlatable session identifiers
• no external timestamp usage

---

# **21. IMPROVEMENTS OVER CLASSICAL WALLETS (INFORMATIVE)**

PQHD prevents:

• key-theft signing
• PSBT malleation
• replay attacks
• stale-time signature reuse
• nondeterministic multisig
• classical ECDSA collapse

---

# **22. BACKWARDS COMPATIBILITY (INFORMATIVE)**

PQHD:

• requires no Bitcoin consensus changes
• uses ordinary PSBTs
• is compatible with air-gapped workflows
• is compatible with classical wallets via Secure Import

---

# **23. CONFORMANCE REQUIREMENTS (NORMATIVE)**

Three conformance levels:

### **L1 — Minimal PQHD**

Supports:
• deterministic key derivation
• EpochTick validation
• canonical PSBT
• basic ledger

### **L2 — Full PQHD**

Adds:
• Policy Enforcer
• PQVL device attestation
• multisig
• recovery capsules
• Secure Import

### **L3 — Hardened Governance**

Adds:
• guardian governance
• delegated authority
• identity/credential modules
• advanced audit requirements

A conformant implementation MUST meet the requirements of its claimed level.

---

# **24. SECURITY GUARANTEES UNDER CLASSICAL-KEY COMPROMISE (NORMATIVE)**

This section defines the mandatory security properties that apply when classical ECDSA private keys, seeds, or multisig key material associated with legacy wallet outputs are compromised, leaked, or broken by quantum attack. These guarantees apply **after Secure Import** and apply to all PQHD-controlled UTXOs.

---

## **24.1 Classical Keys Provide No Spending Authority**

After Secure Import completes, PQHD MUST ensure that:

1. Classical ECDSA keys (including seeds, xprv, derived private keys, or any classical multisig signer material) **MUST NOT** be used for PSBT signing under any circumstance.

2. Classical private keys **MUST NOT** be queried, accessed, or referenced during the PQHD signing pipeline.

3. Compromise of any classical private key **MUST NOT** satisfy any component of the unified custody predicate:

```
valid_for_signing =
      valid_tick
  AND valid_consent
  AND valid_policy
  AND valid_device
  AND valid_quorum
  AND valid_ledger
  AND valid_psbt
```

4. PQHD MUST treat classical keys as non-authoritative and MUST NOT grant spending authority on the basis of any classical signature.

Classical key compromise MUST NOT influence any predicate outcome.

---

## **24.2 All Spending Authority Derives From PQHD ML-DSA Keys**

PQHD MUST only produce Bitcoin-signing witnesses using tick-bound ML-DSA child keys derived through PQHD-DF. These keys MUST be derived using:

```
child_key = cSHAKE256(
    root_key,
    domain = "PQHD-Child-Key-" || class_id,
    info   = derivation_path || tick_bytes || context_bytes
)
```

The signing key MUST NOT exist unless all predicates in `valid_for_signing` evaluate to true.

An attacker who possesses classical private keys MUST be unable to derive the ML-DSA child key because they lack:

* the PQHD root_key
* a valid EpochTick
* the session exporter_hash
* the canonicalised PSBT bytes
* the active policy snapshot
* a valid PQVL attestation
* the monotonic Merkle ledger state
* the correct tick-bound derivation context

PQHD MUST reject any attempt to sign without these conditions.

### **Pseudocode (Informative)**

```
function derive_pqhd_signing_key(ctx):
    if not ctx.valid_for_signing:
        return error("E_PREDICATES_FAILED")

    info = concat(
        utf8(ctx.role_path),
        uint64_to_bytes(ctx.tick.t),
        ctx.policy_hash,
        ctx.exporter_hash
    )

    return cshake256_derive(ctx.root_key, "PQHD-Child-Key-consent", info)
```

An attacker possessing classical keys cannot supply the required inputs to produce an equivalent signing key.

---

## **24.3 Miners Observe Only Standard Bitcoin Multisig Witnesses**

PQHD MUST produce on-chain transactions that use:

* a standard script path or P2WSH witness, and
* the ML-DSA-derived public keys exported by PQHD, formatted according to the wallet’s configured script policy.

Miners MUST NOT require knowledge of PQHD predicates, ticks, attestation, consent windows, or ledger state. Miners validate only the Bitcoin script and witness.

PQHD MUST enforce all custody predicates **before** any signature is produced.

Therefore:

* Classical ECDSA signatures submitted by an attacker MUST NOT satisfy the script path exported by PQHD.
* PQHD-derived ML-DSA keys MUST be the only valid public keys able to satisfy the spending condition after Secure Import.

## **24.3.1 On-Chain Signature and Script Format**

PQHD signatures used internally for custody enforcement are ML-DSA-65 and MUST NOT be exposed on-chain. When producing a Bitcoin transaction, PQHD MUST synthesize a valid classical secp256k1 signature over the canonical PSBT using a deterministic Bitcoin-compatible keypair derived from the PQHD root. This synthesis process MUST maintain full compliance with Bitcoin consensus rules and MUST NOT weaken any component of the custody predicate.

The Bitcoin network observes only standard script paths and witnesses:

secp256k1 public keys embedded in P2WSH, P2TR, or other consensus-valid scripts, and

classical ECDSA or Schnorr signatures satisfying those scripts.

PQHD’s internal ML-DSA keys, PQHD seed material, predicate evaluations, device attestation, and temporal state remain entirely internal to the custody model and are not visible on-chain. Compatibility keys are used solely to satisfy Bitcoin’s script validation requirements and do not provide independent authority to spend.

---

## **24.4 Classical-Key Compromise MUST NOT Permit Replay**

Replay MUST be structurally prevented by PQHD through:

* ConsentProof `tick_issued` / `tick_expiry` windows
* EpochTick freshness ≤ 900 seconds
* tick monotonicity validation
* exporter_hash session binding
* deterministic PSBT canonicalisation and bundle_hash verification
* PQVL attestation (`drift_state == "NONE"`)
* ledger monotonicity and prev_hash continuity
* tick-bound key derivation

Replay of any historical PSBT or classical signature MUST fail because one or more of:

1. `bundle_hash` does not match canonical_psbt
2. tick is expired or non-monotonic
3. the consent window is closed
4. device attestation is stale or invalid
5. ledger state does not match expected `prev_hash`

---

## **24.5 Classical-Key Compromise Is Not a Spend Condition**

PQHD MUST guarantee:

```
Compromise of classical ECDSA keys, xprvs, seeds, or any
legacy multisig material MUST NOT grant spending authority
over UTXOs controlled by PQHD.
```

Only the following constitute valid spending authority:

* tick-fresh ML-DSA child keys
* valid ConsentProof
* valid PQVL device attestation
* valid policy state
* valid quorum
* valid canonical PSBT
* valid ledger continuity

No classical key MAY substitute for any PQHD predicate.

---

## **24.6 Quantum-Attack Resilience**

PQHD MUST preserve spending safety under total classical-key collapse, including:

* quantum recovery of ECDSA seeds
* exposure of all classical multisig private keys
* full reconstruction of legacy key paths

PQHD MUST guarantee that:

1. Classical signatures are never accepted for spending after Secure Import.
2. ML-DSA signatures cannot be forged using classical-key material.
3. ML-DSA child key derivation cannot be computed without satisfying **all seven** custody predicates.

---

## **24.7 Required Conclusion**

A conformant implementation MUST guarantee:

```
Classical-key compromise alone is insufficient to spend PQHD-controlled Bitcoin.
All active spending authority derives exclusively from the PQHD predicate model
and tick-bound ML-DSA child keys.
```

No exceptions are permitted.

---

# **25. Retention Time Lock (RTL)**

Retention Time Lock (RTL) is an optional operational mechanism that minimises the time a Bitcoin transaction is visible in the mempool before confirmation. RTL uses a future-dated timelock (via `nLockTime` or `OP_CHECKLOCKTIMEVERIFY`) that prevents the transaction from becoming valid until a specified activation point. PQHD signers only authorise spends that include this future-dated timelock. The wallet broadcasts the fully signed transaction shortly before the timelock becomes valid, creating a controlled **Retention Window** (typically 30–90 seconds).

RTL is a visibility-minimisation technique. It does not modify Bitcoin consensus and does not provide cryptographic quantum resistance by itself. An optional **Time-Lock Extension (TLE)** allows safely shifting the activation point when the Retention Window becomes too tight.

---

## **25.1 RTL Overview**

An RTL-protected spend behaves as follows:

1. The transaction cannot be mined until its RTL timelock has expired.
2. PQHD signers only sign transactions containing a valid future-dated RTL timelock.
3. Wallet broadcasts only within the configured Retention Window.
4. The transaction is visible in the mempool only during this window.
5. If the first eligible block is missed, the transaction proceeds as a normal unconfirmed transaction unless the wallet regenerates a fresh RTL attempt.
6. RTL provides operational visibility minimisation, not cryptographic protection.

---

## **25.2 RTL Script and Transaction Structure (Normative)**

Implementations MUST use standard Bitcoin timelocks.

### **25.2.1 Transaction-level RTL (`nLockTime`)**

Bitcoin interprets:

* **Height-based timelocks** when `nLockTime < 500_000_000`.
* **Time-based timelocks** when `nLockTime ≥ 500_000_000` (Unix timestamp).

Height-based RTL:
`nLockTime = lock_height`
Minimum granularity: **1 block**

Time-based RTL:
`nLockTime = lock_time`
Minimum granularity: **1 second**

### **25.2.2 Script-level RTL (CLTV)**

```
<locktime> OP_CHECKLOCKTIMEVERIFY OP_DROP
<multisig script>
```

### **25.2.3 Signer Requirements**

PQHD signers MUST:

* verify the RTL timelock is present and future-dated;
* reject the spend if the RTL timelock is missing or invalid;
* reject transactions that deviate from the RTL template.

---

## **25.3 RTL Signing Policy (Normative)**

### **Determine activation point**

Inputs:
`current_height`, `current_time`.

Height-based RTL:
`rtl_height = current_height + 1`

Time-based RTL:
`rtl_time = current_time + TARGET_INTERVAL_SECONDS`

### **Embed RTL parameters**

`nLockTime = rtl_height` or `rtl_time`

### **Signature production**

Signers MUST refuse to sign if:

* RTL parameters are missing,
* RTL parameters are earlier than current_height or current_time,
* the transaction does not match the RTL spend template.

### **Broadcast timing**

Wallet MUST broadcast only when:

Height-based:
`current_height >= rtl_height − SAFETY_BLOCKS`

Time-based:
`current_time >= rtl_time − SAFETY_SECONDS`

Recommended defaults:

* Normal Mode: `SAFETY_SECONDS = 60–90`
* Tight Mode: `SAFETY_SECONDS = 30` with sufficient fee rate

### **Fail-closed**

Any RTL validation failure MUST cause signers to reject the spend.

---

# **25.4 Quantum-Hardened RTL (Optional)**

Quantum-Hardened RTL adds a per-attempt secret `S`, derived off-chain and committed on-chain via:

```
OP_HASH160 <C> OP_EQUALVERIFY
```

This prevents cloned or modified RTL transactions observed in the mempool from being replayed or redirected without the correct secret `S`.

Wallets MAY substitute SHA256 for SHAKE256 if quantum hardness is not required; only the Script commitment `C = HASH160(S)` is consensus-visible.

---

## **25.4.1 Overview**

A Quantum-Hardened RTL spend is valid only when:

1. the RTL timing predicate is satisfied;
2. the chosen Taproot script path includes a hashlock on a per-attempt secret `S`;
3. the witness reveals `S` such that `HASH160(S) == C`; 
Bitcoin Script requires the preimage S to be revealed inside the same spending transaction; no separate reveal transaction is used or required. The secret S appears only in the witness of the RTL spend when the transaction is broadcast.
4. all multisig and PQHD predicates are satisfied.

---

## **25.4.2 Per-Attempt Secret Derivation (Wallet-Level, Normative)**

Inputs:

* `master_secret`
* `utxo_id` (`"txid:vout"`)
* `attempt_counter`
* `H_target`
* `bundle_hash` (optional)

Derivation:

```
context = canonical_encode({
    "version": 1,
    "utxo_id": utxo_id,
    "attempt_counter": attempt_counter,
    "H_target": H_target,
    "bundle_hash": bundle_hash
})

raw = SHAKE256(master_secret || context)   // or SHA256(...)
S   = raw[0:32]
C   = HASH160(S)
```

Requirements:

* `attempt_counter` MUST be unique per attempt.
* `(S, C)` MUST NOT be reused.
* Only `C` is committed on-chain.
* `S` MUST appear only in the witness.
* `canonical_encode` MUST be deterministic.

---

## **25.4.3 Locking Script Template (Leaf A)**

```
<H_target> OP_CHECKLOCKTIMEVERIFY OP_DROP
OP_HASH160 <C> OP_EQUALVERIFY
<pub1> <pub2> <pub3> 3 OP_CHECKMULTISIG
```

Leaf A MUST be a Taproot leaf.
Recovery leaf SHOULD use a later `H_recovery` and distinct quorum.
`control_block` MUST follow BIP-341.

---

## **25.4.4 Quantum-Hardened RTL Spend Procedure (Normative)**

1. **Select UTXO and attempt id**
   Increment `attempt_counter`.

2. **Derive (S, C)**
   As in §25.4.2.

3. **Construct spend transaction**

   * `nLockTime = H_target`
   * Input sequence < `0xFFFFFFFF`
   * Outputs per policy

4. **Build Taproot script-path witness**

```
""
<sig1>
<sig2>
<sig3>
<S>
<scriptA_bytes>
<control_block_A>
```

5. **Apply RTL broadcast timing**
   Wallet MUST broadcast only inside the Retention Window.

6. **Expiry**
   If unconfirmed by `H_target + 1`, the wallet MUST NOT rebroadcast and MUST derive a new `(S′, C′)` for subsequent attempts.

---

## **25.4.5 Security Considerations (Informative)**

* RTL restricts mempool exposure to seconds.
* Even if classical keys are broken, attacker cannot satisfy `HASH160(S) == C`.
* New `(S, C)` per attempt prevents replay.
* Recovery leaf SHOULD exist for long-term spendability.

---

# **25.5 Decoy Multisig Key Policy (Optional)**

## **25.5.1 Abstract**

The Decoy Multisig Key Policy obfuscates multisig topology by randomising key ordering in Script and randomising which keys sign each spend. All keys MUST be real keys with corresponding private keys. Roles (`core`, `guardian`, `decoy`, `backup`) are wallet-level metadata.

---

## **25.5.2 Model**

Given:

```
M <pub1> <pub2> ... <pubN> N OP_CHECKMULTISIG
```

* All keys are real.
* `M` is the enforced threshold.
* Script is unaware of roles; it accepts any `M` valid signatures.

---

## **25.5.3 Deterministic Key Ordering (Wallet-Level, Normative)**

Inputs:

* `master_secret`
* `utxo_id`
* `leaf_id`
* `pubkeys`

Key ordering seed:

```
seed = HMAC-SHA256(master_secret,
                   "key_order" || utxo_id || leaf_id)
```

Shuffle:

```
ordered_pubkeys = deterministic_shuffle(pubkeys, seed)
```

Ordered keys MUST be used in Script.

---

## **25.5.4 Signer Selection Per Spend (Wallet-Level, Normative)**

Signer selection MUST use the **unsigned transaction hash**.

1. Compute hash:

```
tx_hash = SHA256(canonical_serialize(unsigned_tx))
```

2. Derive seed:

```
signer_seed = HMAC-SHA256(master_secret,
                          "signers" || utxo_id || tx_hash)
```

3. Shuffle:

```
shuffled = deterministic_shuffle(all_keys, signer_seed)
```

4. Signers:

```
signing_keys = shuffled[:M]
```

Wallet SHOULD ensure all keys are selected periodically.

---

## **25.5.5 Security and Privacy Notes (Informative)**

* All keys MUST be real.
* Threshold remains `M`.
* Randomisation prevents inference of wallet topology.
* Determinism ensures full restorability.

---

# **25.6 Visibility Timeline (Informative)**

Without RTL:
Transactions may remain in mempool ≈1 block interval or longer.

With RTL:
Transaction becomes visible only within the Retention Window.

---

# **25.7 Retention Window and Fee Requirements (Normative)**

`SAFETY_SECONDS` configurable:

* Normal Mode: 60–90
* Tight Mode: 30

If the first eligible block is missed, wallet MAY:

* treat the transaction as a normal unconfirmed transaction, or
* generate a new RTL attempt with updated `H_target` and new `(S′, C′)`.

---

# **25.7A Time-Lock Extension (TLE) (Normative, Optional)**

Requirements:

* Extension MUST occur before the original timelock becomes valid.
* `new_lock > old_lock` MUST hold.
* The original RTL PSBT MUST be discarded.
* PQHD predicates MUST remain satisfied.

---

# **25.8 Relationship to Quantum Safety (Informative)**

RTL is operational.
Quantum resistance derives from PQHD and, optionally, per-attempt secret `S`.

---

# **25.9 RTL in Classical Wallets (Informative)**

Classical wallets may use RTL with:

* standard timelocks,
* delayed broadcast,
* local transaction retention.

RTL reduces mempool visibility but does not itself resist quantum attacks.

---

# **25.10 Error Codes (Normative)**

Implementations SHOULD include:

* `E_TLE_INVALID`
* `E_TLE_TOO_EARLY`
* `E_TLE_DELTA_VIOLATION`
* `E_RTL_POLICY_VIOLATION`
* `E_QRTL_SECRET_REUSE`

---

# **ANNEX A — Temporal Authority (Epoch Clock) (NORMATIVE)**


This annex defines the canonical temporal authority used by PQHD.
All time-bound predicates MUST rely exclusively on EpochTicks.

---

## **A.1 Canonical EpochTick Structure**

```
EpochTick = {
  t:              uint,   ; Strict Unix Time
  profile_ref:    tstr,   ; canonical Epoch Clock profile reference
  alg:            tstr,   ; MUST be "ML-DSA-65"
  sig:            bstr    ; ML-DSA-65 signature
}
```

### Requirements:

* **t** MUST represent Strict Unix Time (seconds since 1970-01-01T00:00:00Z, ignoring leap seconds).
* **profile_ref** MUST equal the canonical profile reference:

```
ordinal:439d7ab1972803dd984bf7d5f05af6d9f369cf52197440e6dda1d9a2ef59b6ebi0
```

* **sig** MUST be a valid ML-DSA-65 signature over the canonical payload.
* Object MUST be encoded using **deterministic CBOR or JCS JSON**.

---

## **A.2 Tick Validation Rules**

PQHD MUST validate the following conditions before any signing or predicate evaluation:

1. **Signature Validity**
   ML-DSA-65 signature MUST verify over the canonical tick payload.

2. **Canonical Encoding**
   Tick MUST be deterministically encoded.

3. **Freshness (≤900 seconds)**

   ```
   tick_current - EpochTick.t ≤ 900
   ```

4. **Monotonicity**

   ```
   EpochTick.t ≥ last_valid_tick
   ```

5. **Profile Correctness**

   ```
   EpochTick.profile_ref == canonical_profile_ref
   ```

6. **Mirror Consensus (online mode)**
   Devices MUST obtain ≥2 matching ticks.

Any failure MUST set:

```
valid_tick = false
```

---

## **A.3 Tick Reuse Rules**

A validated tick MAY be reused for **up to 900 seconds** in:

* offline mode
* air-gapped mode
* Stealth Mode

After 900 seconds, **all time-dependent operations MUST halt** until a fresh tick is validated.

Local/system time MUST NOT be used for fallback.

---

## **A.4 Tick-Based Predicate Semantics**

All of the following MUST use EpochTicks:

* ConsentProof tick_issued / tick_expiry
* policy time windows
* recovery_delay_seconds
* multisig coordination
* Secure Import timing
* delegated authority validity windows
* delegated spending windows
* any other temporal rule in PQHD

System time MUST NOT appear in any PQHD rule.

---

## **A.5 Tick Validation Pseudocode (Informative)**

```
function validate_tick(tick, last_valid_tick):
    if not verify_ml_dsa_65(tick.sig, canonical(tick)):
        return E_TICK_INVALID

    if tick.profile_ref != canonical_profile_ref:
        return E_TICK_PROFILE_MISMATCH

    if tick.t < last_valid_tick:
        return E_TICK_ROLLBACK

    if (current_wall_clock - tick.t) > 900:
        return E_TICK_EXPIRED

    return OK
```

Note: **current_wall_clock** may measure only elapsed seconds; absolute time MUST come from EpochTicks.

---

## **A.6 Deterministic Integration**

EpochTicks MUST appear canonically inside:

* ledger entries
* recovery capsules
* delegated authority objects
* delegated spending credentials
* identity/KYC credentials
* KeyMail envelopes
* SafePrompt envelopes
* ConsentProof windows
* all tick-bound predicate evaluations

Tick ordering MUST remain strictly monotonic.

---

# **ANNEX B — Runtime Integrity (PQVL) (NORMATIVE)**

PQHD depends on cryptographic runtime attestation to ensure the device performing signing is in a trusted state.
A signing operation MAY proceed only when:

```
valid_device = (drift_state == "NONE")
```

No other source of device trust is permitted.

---

# **B.1 AttestationEnvelope Structure**

Devices MUST present a canonical, PQVL-compliant attestation envelope:

```
AttestationEnvelope = {
  envelope_id:      tstr,
  device_id:        tstr,
  measurement_hash: bstr,
  drift_state:      tstr,        ; "NONE" | "WARNING" | "CRITICAL"
  probes:           [* ProbeResult],
  tick:             EpochTick,
  signature_pq:     bstr         ; ML-DSA-65
}
```

### Requirements:

* MUST be encoded with deterministic CBOR or JCS JSON
* MUST include a fresh EpochTick
* signature_pq MUST be a valid ML-DSA-65 signature
* drift_state MUST be one of: `"NONE"`, `"WARNING"`, `"CRITICAL"`

---

# **B.2 Drift State Semantics**

### **"NONE"**

All integrity probes match expected values. Device is valid.

### **"WARNING"**

Non-fatal deviation. Device validity MAY be downgraded by policy, but:

```
valid_device = false
```

for custody purposes.

### **"CRITICAL"**

Required probe mismatch, measurement tampering, or integrity breach:

```
valid_device = false
```

AND the wallet MUST fail-closed.

---

# **B.3 ProbeResult Structure**

Each probe reports a single integrity measurement:

```
ProbeResult = {
  probe_id:     tstr,
  result:       tstr,
  expected:     tstr,
  signature_pq: bstr
}
```

Probe validation MUST be deterministic.

Probe classes include:

* system_state
* process_state
* integrity_state
* policy_state

Implementations MAY include additional probes, but required probes MUST appear.

---

# **B.4 Attestation Freshness**

A device’s attestation is valid only if:

* tick.age ≤ 900 seconds
* tick signature is valid
* tick profile_ref is canonical
* tick is monotonic relative to previous attestation ticks
* drift_state == "NONE"

If freshness fails, the device MUST be treated as invalid.

---

# **B.5 Device Predicate**

This predicate MUST be enforced for **every** sensitive operation:

```
valid_device = (drift_state == "NONE")
```

This predicate applies to:

* PSBT signing
* multisig workflows
* Secure Import
* recovery
* delegated authority actions
* identity/credential operations
* operational mode transitions (offline/stealth)
* any action requiring ConsentProof

There are **no exceptions**.

---

# **B.6 Attestation Verification Rules**

PQHD MUST verify:

1. **Canonical Encoding**

   ```
   canonical(attestation)
   ```

2. **Signature Validity**

   ```
   verify_ml_dsa_65(attestation.signature_pq, canonical(attestation_without_signature))
   ```

3. **Tick Integrity**

   * ML-DSA-65 signature
   * canonical encoding
   * freshness ≤ 900s
   * monotonicity
   * profile_ref correctness

4. **Probe Validation**

   ```
   probe.result == probe.expected
   ```

Any failing condition MUST produce:

```
valid_device = false
```

and MUST block signing.

---

# **B.7 Multisig Attestation**

For multisig:

* Each device MUST validate all other participants’ AttestationEnvelopes.
* Each device MUST compute **valid_device** independently.
* If ANY device is invalid:

```
valid_quorum = false
valid_for_signing = false
```

Signatures MUST NOT be produced.

---

# **B.8 Attestation Pseudocode (Informative)**

```
function validate_attestation(env):
    if not canonical(env):
        return E_DEVICE_ATTESTATION_FAILED

    if not verify_ml_dsa_65(env.signature_pq, canonical(env)):
        return E_DEVICE_ATTESTATION_FAILED

    if env.drift_state != "NONE":
        return E_DEVICE_DRIFT_DETECTED

    if not validate_tick(env.tick):
        return E_DEVICE_ATTESTATION_FAILED

    for probe in env.probes:
        if probe.result != probe.expected:
            return E_DEVICE_INTEGRITY_FAILED

    return OK
```

---

# **B.9 Restricted Mode Requirements**

In **Stealth**, **Offline**, or **Air-Gapped** modes:

* drift_state MUST remain "NONE"
* attestation MUST bind to a tick
* fresh attestation MUST be obtained when returning to online mode
* local time MUST NOT be used

Failure in any restricted mode MUST force fail-closed behaviour.

---

# **B.10 Ledger Requirements**

Ledger MUST record:

* device registration events
* attestation events
* drift detection
* integrity failures

All entries MUST observe deterministic encoding and monotonic tick ordering.

---

# **B.11 Forbidden Behaviours**

Implementations MUST NOT:

* infer trust from minimal device identity
* sign without valid attestation
* use stale attestation
* override drift_state
* substitute heuristics for required probes
* ignore probe mismatches
* rely on non-canonical encoding

---

# **ANNEX C — Core PQSF & PQAI Structures (NORMATIVE)**

PQHD depends on a set of foundational structures defined in PQSF and PQAI.
These are normative requirements, not optional guidance.

Nothing here overrides PQSF or PQAI; it is included to make PQHD self-sufficient.

---

# **C.1 ConsentProof Structure (PQSF)**

ConsentProof binds explicit user intent to a specific PSBT, device, session, and tick window.

```
ConsentProof = {
  action:        tstr,
  intent_hash:   bstr,
  bundle_hash:   bstr,
  tick_issued:   uint,
  tick_expiry:   uint,
  exporter_hash: bstr,
  device_id:     tstr,
  role:          tstr,
  quorum_index:  uint,
  signature_pq:  bstr    ; ML-DSA-65
}
```

## Requirements:

* MUST use deterministic CBOR or JCS JSON
* MUST bind intent to a specific PSBT via **bundle_hash**
* MUST bind to a session via **exporter_hash**
* MUST bind to a device via **device_id**
* MUST be tick-fresh:

  ```
  tick_issued ≤ tick_current ≤ tick_expiry
  ```
* MUST be signed with ML-DSA-65

Any violation MUST result in:

```
valid_consent = false
```

---

# **C.2 ConsentProof Validation Rules**

PQHD MUST enforce:

1. **Tick Window**

   ```
   tick_current ≥ tick_issued
   tick_current ≤ tick_expiry
   ```

2. **Exporter Binding**

   ```
   ConsentProof.exporter_hash == session.exporter_hash
   ```

3. **Bundle Hash Binding**
   MUST equal the canonical PSBT's bundle_hash.

4. **Signature Validity (ML-DSA-65)**
   MUST be verified over the canonical encoding.

Failure in ANY rule MUST set:

```
valid_consent = false
```

and block signing.

---

# **C.3 Canonical Encoding Rules (PQSF)**

All PQHD structures consumed by PQSF MUST use **deterministic** encoding.

Permitted formats:

* **JCS JSON (RFC 8785)**
* **Deterministic CBOR (RFC 8949 §4.2)**

Canonical encoding applies to:

* ConsentProof
* Policy objects
* Ledger entries
* Delegate/identity/credential objects
* Recovery capsules
* Chain/L2 derivation context maps
* SafePrompt envelopes
* PSBT canonical encoding

If encoding is non-canonical, the structure MUST be rejected.

---

# **C.4 Ledger Entry Structure (PQSF)**

PQHD uses a Merkle-anchored ledger with strictly canonical entries.

```
LedgerEntry = {
  event:        tstr,
  epoch_tick:   EpochTick,
  entry_tick:   uint,
  payload:      { * tstr => any },
  signature_pq: bstr,
  prev_hash:    bstr
}
```

## Requirements:

* MUST be canonically encoded
* MUST be signed with ML-DSA-65
* MUST reference the validated EpochTick
* prev_hash MUST match the previous Merkle root
* entry_tick MUST be monotonic
* MUST follow append-only semantics

Failure MUST cause ledger freeze.

---

# **C.5 Policy Enforcer — Core Semantics (PQSF)**

The Policy Enforcer evaluates deterministic rules with no nondeterministic inputs.

The predicate is:

```
valid_policy = f(policy, tick, consent, device, ledger, psbt)
```

### Requirements:

1. **Deterministic Evaluation**
   Same inputs MUST yield identical results across implementations.

2. **Canonical Policy Format**

   ```
   policy_hash = SHAKE256-256(canonical(policy))
   ```

3. **Roles and Thresholds**
   MUST be deterministic and identical across devices.

4. **Amount, Limit, and Destination Rules**
   MUST enforce allowlists, denylists, daily limits, cooldowns, etc.

5. **Temporal Semantics**
   MUST use EpochTicks only.

6. **Recovery Rules**
   MUST use tick-bound windows deterministically.

---

# **C.6 Cryptographic Primitives Required (PQSF)**

PQHD MUST use the following primitives:

* **ML-DSA-65** — all signatures
* **SHAKE256-256** — hashing
* **cSHAKE256** — deterministic key derivation
* **ML-KEM (768/1024)** — for encrypted payloads (optional modules)

These MUST be used with deterministic, domain-separated context strings.

---

# **C.7 Mandatory Predicate Model (PQSF)**

PQHD inherits the full predicate set:

```
valid_tick
AND valid_consent
AND valid_policy
AND valid_runtime      ; from PQVL
AND valid_quorum
AND valid_ledger
AND valid_structure / valid_psbt
```

The PQHD signing predicate:

```
valid_for_signing
```

MUST evaluate TRUE only when **all** above are true.

There are no overrides or bypasses.

---

# **C.8 SafePrompt Structure (PQAI)**

SafePrompt is used for high-risk natural-language confirmations.

```
SafePrompt = {
  prompt_id:     tstr,
  content_hash:  bstr,
  action:        tstr,
  consent_id:    tstr,
  tick_issued:   uint,
  expiry_tick:   uint,
  exporter_hash: bstr / null,
  signature_pq:  bstr
}
```

### Requirements:

* MUST be canonical
* MUST bind prompt content to content_hash
* MUST use tick windows
* MUST be signed with ML-DSA-65
* MUST bind to ConsentProof via consent_id
* MUST respect PQHD fail-closed semantics

---

# **C.9 SafePrompt Restrictions (PQAI)**

SafePrompt MUST NOT:

* override custody predicates
* bypass signer thresholds
* operate under drift_state ≠ "NONE"
* operate outside tick_issued/tick_expiry
* operate when consent or exporter_hash is invalid

SafePrompt MAY be used only when all PQHD predicates are valid.

---

# **C.10 Universal Secret Derivation Reference (PQSF)**

PQHD may use deterministic, non-custodial secrets for identity/credential purposes:

```
PQHD-Secret:<context>
```

Derived using cSHAKE256 with canonical context parameters.

These secrets MUST be isolated from all custody keys and MUST NOT influence PQHD signing.

---

# **C.11 Integration Requirements**

PQHD MUST:

* accept these structures exactly as defined
* reject any modified or extended versions
* enforce all canonical encoding rules
* verify all required signatures
* apply all predicate rules deterministically

These structures are to be treated as authoritative within PQHD.

---

# **ANNEX D — Reference Test Vectors (INFORMATIVE)**

Annex D provides deterministic test vectors allowing independent implementations to validate correctness.
All vectors MUST be encoded using deterministic CBOR or JCS JSON.
Any difference in canonical encoding MUST be treated as an interoperability failure.

---

## **D.1 Key Derivation Test Vectors**

### **D.1.1 Root Key Derivation**

**Input seed (256-bit CSPRNG):**

```
9a7f3e2d1c8b6a5f4e3d2c1b0a9f8e7d6c5b4a3f2e1d0c9b8a7f6e5d4c3b2a1
```

**Operation:**

```
root_key = SHAKE256-256(seed)
```

**Expected output:**

```
root_key = 0x8c3f7a2e14b318fdd9e005ea91a6b4dfe44ef772aa35fedeb495f1ad313aa660
```

---

### **D.1.2 Governance Key Derivation**

**Inputs:**

```
root_key: 0x8c3f7a2e14b318fd....aa660
class_id: "governance"
derivation_path: "/0/0/0"
```

**Operation:**

```
child_key = cSHAKE256(
    root_key,
    domain = "PQHD-Child-Key-governance",
    info   = "/0/0/0"
)
```

**Expected output:**

```
child_key = 0x5d2a8f1c09cf416f4eefbb72e2440ba09e222ae36a2d59ed667d30bef44fc310
```

---

### **D.1.3 Tick-Bound Consent Key**

**Inputs:**

```
class_id: "consent"
path: "/consent/0/0/1"
tick: 1730000000
```

**Operation:**

```
child_key_tick = cSHAKE256(
    root_key,
    domain = "PQHD-Child-Key-consent",
    info   = "/consent/0/0/1" || 1730000000
)
```

**Expected output:**

```
child_key_tick = 0xaa59f02b88a443e19ae2da4f4450bd23bf1e922664c38c44fb57abe0d323a81e
```

---

### **D.1.4 Device-Bound Key**

**Inputs:**

```
class_id: "device-bound"
path: "device-12345"
```

**Expected output:**

```
device_key = 0x91ed4420d4b8b71fd91e1dd55c2689ad411fb40e8cb29e5da0db6cd70c357093
```

---

## **D.2 PSBT and Bundle Hash Vectors**

### **D.2.1 Canonical PSBT Example**

A non-canonical PSBT example:

```
inputs:
 - txid: 01...ff
   vout: 0
outputs:
 - address: bc1qexample...
   amount: 10000
```

Canonicalised PSBT MUST be deterministically encoded.

**Expected bundle_hash:**

```
0x3ea941bf1f2924ddc9d7129983f54f897c10382ad82642b14d3d07f77d51c81f
```

Deviation MUST produce:

```
E_PSBT_CANONICALIZATION_FAILED
```

---

### **D.2.2 Invalid PSBT**

If inputs/outputs reordered or fields malleated:

```
valid_psbt = false
error = E_PSBT_CANONICALIZATION_FAILED
```

---

## **D.3 ConsentProof Test Vectors**

### **D.3.1 Valid ConsentProof**

```
{
  "action": "spend",
  "intent_hash": SHAKE256("transfer 10000 sats"),
  "bundle_hash": 0x3ea941bf...81f,
  "tick_issued": 1730000000,
  "tick_expiry": 1730000300,
  "exporter_hash": 0xabc12345f00ba9e3...,
  "device_id": "device-12345",
  "role": "primary",
  "quorum_index": 0,
  "signature_pq": <ML-DSA-65 sig>
}
```

Expected:

```
valid_consent = true
```

---

### **D.3.2 Expired ConsentProof**

If tick_current > tick_expiry:

```
valid_consent = false
error = E_CONSENT_EXPIRED
```

---

### **D.3.3 Exporter Mismatch**

If exporter_hash mismatches session:

```
valid_consent = false
error = E_CONSENT_EXPORTER_MISMATCH
```

---

## **D.4 Recovery Capsule Vectors**

### **D.4.1 Example Capsule**

```
{
  "capsule_id": "capsule-001",
  "guardian_set": ["g1","g2","g3"],
  "threshold": 2,
  "recovery_delay_seconds": 86400,
  "creation_tick": 1730000000,
  "encrypted_material": <ML-KEM payload>,
  "capsule_hash": 0xa1b2c3d4e5...
}
```

### **D.4.2 Guardian Approval**

```
{
  "guardian_id": "g1",
  "approval_tick": 1730000500,
  "approval_sig": <ML-DSA-65 sig>
}
```

---

### **D.4.3 Threshold Not Met**

If approvals < threshold:

```
E_RECOVERY_THRESHOLD_UNMET
```

---

## **D.5 Tick Validation Vectors**

### **D.5.1 Valid Tick**

Tick signature valid, age ≤900s.

### **D.5.2 Tick Rollback**

If tick_current < last_valid_tick:

```
E_TICK_ROLLBACK
```

### **D.5.3 Mirror Divergence**

If mirror ticks disagree:

```
E_TICK_DIVERGENCE
```

---

## **D.6 Ledger Vectors**

### **D.6.1 Example Ledger Entry**

```
{
  "event": "sign",
  "epoch_tick": {...},
  "entry_tick": 1730000012,
  "payload": { "bundle_hash": "<hash>", "role": "primary" },
  "signature_pq": "<sig>",
  "prev_hash": "<root>"
}
```

### **D.6.2 Merkle Root Calculation**

```
leaf_hash = SHAKE256-256(0x00 || canonical_entry)
node_hash = SHAKE256-256(0x01 || left || right)
```

---

## **D.7 Drift Detection Vectors**

If PQVL.drift_state != "NONE":

```
valid_device = false
error = E_DEVICE_DRIFT_DETECTED
```

---

# **ANNEX E — Implementation Snippets (INFORMATIVE)**

This annex provides deterministic pseudocode patterns for implementers.
All pseudocode is language-agnostic.

---

## **E.1 PSBT Canonicalisation**

```
function canonicalise_psbt(psbt):
    psbt = strip_witnesses(psbt)
    psbt.inputs  = sort_inputs(psbt.inputs)
    psbt.outputs = sort_outputs(psbt.outputs)
    psbt.fields  = sort_kv_pairs(psbt.fields)
    return encode_canonical(psbt)   ; deterministic CBOR or JCS
```

---

## **E.2 Bundle Hash**

```
bundle_hash = SHAKE256-256(canonical_psbt)
```

---

## **E.3 Predicate Evaluation**

```
function evaluate_predicates(state):
    if not valid_tick(state):     return false
    if not valid_consent(state):  return false
    if not valid_policy(state):   return false
    if not valid_device(state):   return false
    if not valid_quorum(state):   return false
    if not valid_ledger(state):   return false
    if not valid_psbt(state):     return false
    return true
```

---

## **E.4 Tick Validation**

```
tick = epoch_clock.get_current_tick()

if not tick.verify_signature(profile):
    return E_TICK_INVALID
if tick.age() > 900:
    return E_TICK_EXPIRED
if tick.value < last_valid_tick:
    return E_TICK_ROLLBACK
```

---

## **E.5 PQVL Attestation Wrapper**

```
function validate_device():
    att = pqvl.get_attestation()
    if not att.signature_valid():       return E_DEVICE_ATTESTATION_FAILED
    if att.integrity_state != "VALID":  return E_DEVICE_INTEGRITY_FAILED
    if att.age() > 900:                 return E_DEVICE_ATTESTATION_FAILED
    return OK
```

---

## **E.6 Recovery Capsule Validation**

```
function validate_capsule(c):
    if c.threshold > len(c.guardian_set):
        return E_RECOVERY_INVALID
    if current_tick < c.valid_from_tick:
        return E_RECOVERY_DELAY_ACTIVE
    if not verify_signature(c.signature_pq):
        return E_RECOVERY_INVALID
    if not verify_capsule_hash(c):
        return E_RECOVERY_INVALID
```

---

## **E.7 Ledger Append**

```
function append_ledger(entry):
    canonical_entry = encode_canonical(entry)
    leaf = SHAKE256-256(0x00 || canonical_entry)
    new_root = merkle_append(ledger.root, leaf)
    entry.prev_hash = ledger.root
    entry.signature_pq = sign(entry)
    ledger.entries.append(entry)
    ledger.root = new_root
```

---

## **E.8 ConsentProof Construction**

```
function build_consent(action, intent_bytes):
    tick = get_fresh_tick()
    cp = {
        action: action,
        intent_hash: SHAKE256_256(intent_bytes),
        tick_issued: tick.value,
        tick_expiry: tick.value + 900,
        exporter_hash: session.exporter_hash,
        device_id: device.identifier
    }
    cp.signature_pq = sign(cp)
    return cp
```

---

## **E.9 Multisig Signing Skeleton**

```
function multisig_sign(psbt, role, index):
    canonical = canonicalise_psbt(psbt)
    bundle = SHAKE256_256(canonical)

    if not evaluate_predicates(state):
        fail_closed()

    signing_key = derive_key(role, index, tick)
    sig = sign_psbt(canonical, signing_key)
    return sig
```

## **E.9 External Service Interface Stubs (INFORMATIVE)**

The wallet’s signing pipeline assumes the existence of external providers that supply valid EpochTick and AttestationEnvelope objects. Implementations MAY define these providers using simple interfaces:

interface EpochClockClient {
    get_fresh_tick() -> EpochTick
}

interface PQVLClient {
    get_attestation(device_id) -> AttestationEnvelope
}


For testing, implementations MAY use in-memory or deterministic mock implementations that return canonical objects matching Annex A and Annex B. These mocks MUST NOT alter any PQHD custody predicates and MUST follow the same canonical encoding rules as production providers.

---

# **ANNEX F — Interoperability Test Suite (INFORMATIVE)**

This suite ensures different implementations produce identical outputs under all deterministic rules.

---

## **F.1 Cross-Platform Key Derivation Tests**

### **F.1.1 Root Key Matching**

Multiple implementations MUST derive identical root_key from the same seed.

### **F.1.2 Child Key Matching**

Test paths:

```
/consent/0/0
/governance/0/1
/device-bound/deviceA/0
/recovery/guardian1/0
```

---

## **F.2 Tick-Bound Derivation Matching**

All devices MUST derive the same output for:

```
PQHD-DF("consent", "/consent/0/0/1" || tick)
```

---

## **F.3 PSBT Canonicalisation Tests**

### **F.3.1 Canonical PSBT Equality**

All implementations must generate identical canonical_psbt bytes.

### **F.3.2 Bundle Hash Equality**

```
bundle_hash = SHAKE256-256(canonical_psbt)
```

MUST match exactly.

### **F.3.3 Malformed PSBT Handling**

Invalid ordering MUST produce:

```
E_PSBT_CANONICALIZATION_FAILED
```

---

## **F.4 ConsentProof Interoperability**

* ML-DSA signature verification MUST match
* exporter_hash mismatch MUST produce error
* tick expiry MUST cause invalidation

---

## **F.5 Multisig Interoperability**

Tests MUST verify:

* identical role mapping
* identical threshold evaluation
* identical predicate results
* identical bundle_hash values

---

## **F.6 Tick Validation Interoperability**

### **Mirror divergence**

All devices MUST detect mismatched ticks.

### **Tick rollback**

MUST produce:

```
E_TICK_ROLLBACK
```

---

## **F.7 Ledger Interoperability**

Tests include:

* Merkle root reconstruction
* append failure on mismatched prev_hash
* freeze behaviour on rollback

---

## **F.8 Secure Import Interop Tests**

* classical PoP failure MUST produce `E_IMPORT_CLASSICAL_SIG_INVALID`
* ML-DSA PoP failure MUST produce `E_IMPORT_PQ_SIG_INVALID`
* mapping mismatch MUST produce `E_IMPORT_MISMATCH`

---

# **ANNEX G — Deployment Configuration Examples (INFORMATIVE)**

This annex provides deterministic, implementation-ready examples for deploying PQHD across multiple environments.
Each example illustrates valid predicate evaluation, PSBT integrity, temporal rules, and runtime requirements.

Nothing in this annex overrides normative sections.

---

## **G.1 Personal Wallet Configurations**

### **G.1.1 Two-Device Personal PQHD Wallet**

Use case: A user wants increased safety using a mobile device + hardware signer.

**Devices:**

* deviceA (mobile signer) — role: primary
* deviceB (hardware signer) — role: secondary

**Policy:**

```
roles: ["primary", "secondary"]
thresholds:
  primary:   1
  secondary: 1
allowlist: []
denylist: []
min_delay_seconds: 0
max_spend_period: 600
daily_limit_sats: 1000000
cooldown_seconds: 0
guardian_delay_seconds: 0
required_devices: ["deviceA"]
required_attestors: []
anomaly_rules: {}
custom_predicates: {}
```

Notes:

* Both devices MUST sign.
* Good for moderate-value personal custody.
* Uses full predicate model.

---

### **G.1.2 Single-Device PQHD Wallet**

Use case: convenience wallet with full PQHD predicate enforcement.

**Device:**

* deviceA — role: primary

**Policy:**

```
roles: ["primary"]
thresholds:
  primary: 1
allowlist: []
min_delay_seconds: 0
max_spend_period: 300
daily_limit_sats: 500000
cooldown_seconds: 0
```

* PQVL, EpochTick, PSBT rule enforcement still required.
* Valid for non-high-value custody.

---

## **G.2 Enterprise Deployments**

### **G.2.1 Enterprise 3-Tier Role Model**

**Roles:**

* primary
* secondary
* guardian

**Participants:**

* A, B — primary
* C, D — secondary
* E, F — guardian

**Policy:**

```
thresholds:
  primary:   1
  secondary: 1
  guardian:  1
min_delay_seconds: 3600
cooldown_seconds: 7200
daily_limit_sats: 100000000
guardian_delay_seconds: 86400
required_devices: ["A", "C"]
```

Notes:

* High-value enterprise wallets.
* Multiple layers of approval.

---

### **G.2.2 Federated Institution Example**

**Use case:** Multiple institutions share treasury custody.

**Policy:**

```
roles: ["federation"]
thresholds:
  federation: 3
cooldown_seconds: 1800
daily_limit_sats: 50000000
```

* No role mixing.
* Ensures identical predicate evaluation across institutions.

---

## **G.3 High-Security / Sovereign Configurations**

### **G.3.1 Multi-Jurisdiction Custody**

Roles:

* primary (jurisdiction 1)
* secondary (jurisdiction 2, 3)
* guardian (offline entity)

**Policy:**

```
thresholds:
  primary:   1
  secondary: 2
  guardian:  1
daily_limit_sats: 200000000
guardian_delay_seconds: 172800
min_delay_seconds: 43200
max_spend_period: 86400
```

* Designed for regional resilience.
* Guardian ensures mandatory cool-down periods.

---

### **G.3.2 Air-Gapped Cold Storage**

Device:

* deviceA (air-gapped)

**Policy:**

```
roles: ["primary"]
thresholds:
  primary: 1
max_spend_period: 43200
daily_limit_sats: 1000000
```

* MUST follow strict tick reuse rules.
* PSBT transferred via QR/USB.

---

## **G.4 Sovereign Deployments**

**Transport:** STP-only
**Tick Source:** sovereign mirror cluster

**Policy:**

```
roles: ["primary", "guardian"]
thresholds:
  primary: 1
  guardian: 1
min_delay_seconds: 3600
guardian_delay_seconds: 259200
```

* No DNS or CA dependencies.
* Designed for national or institutional resilience.

---

## **G.5 Deployment Notes**

* Coordinators MUST be treated as untrusted.
* All deployments MUST follow the unified predicate model.
* Offline/air-gapped systems MUST respect tick freshness.
* Ledger reconciliation MUST be performed regularly in multi-device systems.

---

# **ANNEX H — Recommended Repository Layout (INFORMATIVE)**

This annex defines a reproducible, auditable repository structure for PQHD implementations.

---

## **H.1 Directory Structure**

```
/pqhd/
    README.md
    LICENSE
    SPEC/
    REF/
    TESTS/
    DOCS/
    TOOLS/
    bindings/
    examples/
```

### PURPOSE:

* **SPEC/** — canonical PQHD spec & annexes
* **REF/** — reference implementations and canonical scripts
* **TESTS/** — deterministic interoperability and fuzz test cases
* **DOCS/** — diagrams, flowcharts
* **TOOLS/** — deterministic encoders, ledger validators
* **bindings/** — language wrappers (Rust, Go, JS/WASM, Python, C++, etc.)

---

## **H.2 SPEC Directory**

```
SPEC/
    PQHD_v1.0.0.md
    Annex_A_TemporalAuthority.md
    Annex_B_RuntimeIntegrity.md
    Annex_C_CoreStructures.md
    ...
```

### Rules:

* MUST include versioned and hashed canonical spec
* MUST include full annexes
* MUST track changes in **CHANGELOG.md**

---

## **H.3 REF Directory**

Provides minimal, deterministic reference implementations:

```
REF/
    psbt/
    epoch/
    pqhd/
```

Examples include:

* canonical PSBT normaliser
* EpochTick verifier
* consent builder
* deterministic ledger model

Requirements:

* MUST produce byte-identical outputs across languages
* MUST avoid dynamic non-deterministic dependencies

---

## **H.4 TESTS Directory**

Contains full cross-language, cross-runtime test suites:

* key derivation
* bundle_hash equality
* tick validation
* multisig matching
* recovery capsule tests
* ledger reconciliation

---

## **H.5 DOCS Directory**

Contains:

* architecture diagrams
* multisig explainer
* canonical encoding rules
* ledger overview
* recovery process documentation

None of these documents are normative.

---

## **H.6 TOOLS Directory**

Deterministic developer utilities:

* **verify_tick.py**
* **verify_psbt.py**
* **ledger_reconcile.py**
* **verify_attestation.py**

All tools MUST be deterministic & reproducible.

---

## **H.7 OPTIONAL Example Wallet**

```
/wallet/
    core/
    interfaces/
    ui/
    storage/
```

Shows how components map to PQHD modules.

---

## **H.8 Metadata Files**

* **CONTRIBUTING.md**
* **CODE_OF_CONDUCT.md**
* **SECURITY.md**
* **CHANGELOG.md**

Security guidelines MUST include reproducible-build requirements.

---

# **ANNEX I — Operational Recovery Guidance (INFORMATIVE)**

This annex describes deterministic procedures for recovering from any fail-closed state.

---

## **I.1 Fail-Closed Recovery Principles**

If PQHD enters FAIL_CLOSED:

* ALL signing MUST stop
* ledger append MUST halt
* sessions MUST be invalidated
* PQVL attestation MUST be refreshed
* a fresh EpochTick MUST be obtained
* cached state MUST be treated as untrusted

No predicate failures may be bypassed.

---

## **I.2 Recovering From Tick Errors**

### **I.2.1 Stale Tick (E_TICK_EXPIRED)**

Steps:

1. Fetch fresh tick
2. Validate signature + profile_ref
3. Validate monotonicity
4. Retry operation

---

### **I.2.2 Tick Rollback (E_TICK_ROLLBACK)**

Steps:

1. Fetch ≥2 mirror ticks
2. Validate monotonicity
3. If rollback persists → freeze ledger
4. Manual operator intervention required

---

## **I.3 Recovering From Consent Errors**

### **I.3.1 Expired Consent**

* User MUST regenerate ConsentProof
* Old consent MUST NOT be reused

### **I.3.2 Exporter Mismatch**

* Reset session
* Generate new exporter_hash
* Request new consent

---

## **I.4 Recovering From PSBT Errors**

### **I.4.1 Canonicalization Failure**

* Reconstruct PSBT from intent
* Re-canonicalise
* Re-check bundle_hash

### **I.4.2 Bundle Hash Mismatch**

* Abort signing
* Regenerate canonical PSBT
* If mismatch persists → freeze ledger

---

## **I.5 Recovering From Device Attestation Errors**

### **I.5.1 Drift Detected**

* Re-attest device
* If mismatch persists → mark device untrusted

### **I.5.2 Integrity Failure**

* Re-validate OS and enclave
* Re-attest
* If failure persists → decommission device

---

## **I.6 Ledger Recovery**

### **I.6.1 Divergence**

* Exchange Merkle roots
* Exchange proofs
* Rebuild ledger to authoritative state

### **I.6.2 Ledger Frozen**

* Identify underlying cause
* Fix cause
* Reconcile ledger
* Unfreeze after monotonic state restored

---

## **I.7 Policy Errors**

### **I.7.1 Threshold Failure**

* Ensure all required devices present
* Ensure valid PQVL attestation

### **I.7.2 Destination Denied**

* User MUST adjust destination
* OR governance rotates policy

---

## **I.8 Governance Errors**

* Validate PQVL state
* Verify governance signatures
* Compare expected vs recorded governance parameters
* Reconcile ledger
* Align policy snapshots

---

## **I.9 Secure Import Errors**

* Recompute classical PoP
* Recompute PQ PoP
* Recompute mapping
* Validate ledger state

---

## **I.10 Full Reset Procedure**

1. fresh EpochTick
2. full PQVL re-attestation
3. rebuild in-memory caches
4. re-canonicalise PSBT
5. re-derive tick-bound keys
6. recompute ledger root
7. re-evaluate predicates

---

## **I.11 Multi-Device Recovery**

Devices must:

* independently validate attestation
* reconcile ledger roots
* refresh EpochTick
* refresh policy snapshot

---

## **I.12 Storage & Backup Recovery Notes**

PQHD recovery uses:

* seed backup
* guardian capsules
* exported ledger bundles

---

## **I.13 Optional Recovery Hardening Profiles (NORMATIVE–OPTIONAL)**

Implementations MAY define optional recovery-hardening profiles that increase, but never decrease, the required authority for recovery.

The following mechanisms MAY be supported:

1. External Cosigner Requirement  
Recovery MAY require an external cosigner key that is:
    • independent of the PQHD root, and  
    • not derivable from PQHD-DF.

Such requirements MUST be enforced as additional AND-conditions inside valid_quorum or valid_policy.

2. Mandatory Device-Bound Predicate  
Recovery MAY require the presence of specific hardware-bound keys (e.g., enclave-sealed keys) that are not reconstructible from the PQHD root.  
These conditions MUST be enforced as AND-conditions inside existing predicates.

3. Guardian-Coauthorised Recovery  
High-value recovery MAY require guardian devices or guardian quorums to participate in approval.  
All such requirements MUST be defined in policy and MUST generate ledger events.

4. Time-Locked Recovery  
Policies MAY define additional tick-bound delay windows for recovery using EpochTicks.  
These MUST be evaluated through valid_tick and valid_policy.

Predicate Binding Rules  
All optional hardening MUST:
    • be enforced as AND-conditions inside existing predicates;  
    • be reflected in canonical policy snapshots;  
    • be recorded as governance events in the ledger;  
    • NEVER introduce alternate authorisation paths;  
    • NEVER permit signing when any mandatory predicate is false.

---

ANNEX J — HD Seed Adoption & External-Seed Wallet Creation (NORMATIVE–OPTIONAL)
This annex defines how PQHD can adopt an external HD seed before PQHD initialisation.

J.1 Purpose
Supports workflows such as:
* importing BIP39 seed
* adopting SLIP39 shards
* adopting xprv/xpub root
* using raw 256-bit deterministic seed
All under deterministic, PQ-safe conditions.

J.2 Preconditions
PQHD MAY adopt an external seed only if:
* PQHD is not initialised
* external seed is provided in allowed formats
* seed has never been used in PQHD
* seed does not control classical Bitcoin assets
* user explicitly authorises via: ConsentProof.action = "pqhd.adopt_external_seed"
* 
* fresh EpochTick available
* PQVL drift_state == "NONE"
Pseudocode (Informative)
// Check preconditions for HD seed adoption
function can_adopt_external_seed(ctx, seed_metadata):
    if ctx.pqhd_initialised:
        return false
    if seed_metadata.controls_classical_utxos:
        return false
    if seed_metadata.previously_used_in_pqhd:
        return false
    if ctx.attestation.drift_state != "NONE":
        return false
    if ctx.consent.action != "pqhd.adopt_external_seed":
        return false
    if not validate_tick(ctx.tick, ctx.last_tick):
        return false
    return true

J.3 Canonical Seed Preparation
* BIP39 → derive seed bytes via PBKDF2
* SLIP39 → reconstruct shard → derive seed
* xprv → decode extended key → extract seed
* raw bytes → used as-is
All MUST be canonicalised before hashing.
Pseudocode (Informative)
// Normalise external seed into canonical bytes
function canonicalise_external_seed(source):
    if source.type == "BIP39":
        return bip39_to_seed(source.mnemonic, source.passphrase)
    if source.type == "SLIP39":
        return slip39_to_seed(source.shares)
    if source.type == "XPRV":
        return decode_xprv_to_seed(source.xprv)
    if source.type == "RAW_256":
        return source.bytes  // must be 32 bytes
    // behaviour not defined for other formats
    return error("unsupported_seed_format")

J.4 Trial Root Derivation
external_seed_canonical = SHAKE256-256(canonical_bytes)
trial_root = SHAKE256-256(external_seed_canonical)
Pseudocode (Informative)
// Compute a trial root from a canonical external seed
function derive_trial_root_from_external_seed(external_seed_bytes):
    external_seed_canonical = shake256_256(external_seed_bytes)
    trial_root = shake256_256(external_seed_canonical)
    return trial_root

J.5 Future-Use Check
Wallet MUST ensure:
* no existing PQHD ledger
* no existing PQHD device-bound keys
* no prior use of this seed in PQHD context
If conflict exists → MUST use Secure Import instead.
Pseudocode (Informative)
// Ensure we’re not overwriting an existing PQHD instance
function check_no_prior_pqhd_state(ctx, trial_root_fingerprint):
    if ctx.ledger.entries.length > 0:
        return false
    if ctx.device_keys_exist:
        return false
    if trial_root_fingerprint in ctx.known_root_fingerprints:
        return false
    return true

J.6 Consent Requirements
ConsentProof MUST be:
* canonical
* ML-DSA-65 signed
* exporter-bound
* tick-fresh

J.7 Tick & Attestation Validation
Adoption MUST occur under:
* fresh EpochTick
* drift_state == "NONE"

J.8 Ledger Initialisation
PQHD MUST create an initial ledger entry:
{
  event: "pqhd_seed_adopted",
  root_fingerprint: SHAKE256-256(trial_root)[0:16],
  seed_type: "external_hd_seed",
  tick: <tick>,
  signature_pq: <sig>
}
Pseudocode (Informative)
// Initialise ledger with a seed-adoption event
function initialise_ledger_with_seed(ctx, trial_root, tick, signing_key, seed_type):
    fingerprint = slice(shake256_256(trial_root), 0, 16)
    payload = {
        root_fingerprint: fingerprint,
        seed_type:        seed_type
    }
    entry = build_ledger_entry("pqhd_seed_adopted", tick, payload, prev_root = zero_hash(), signing_key)
    append_ledger_entry(ctx.ledger, entry, tick)
    return fingerprint

J.9 Finalisation
If all predicates succeed:
pqhd_root = trial_root
Then:
* derive device-bound keys
* initialise PQHD policy engine
* perform reconciliation
Pseudocode (Informative)
// Full adoption flow
function adopt_external_seed(ctx, seed_source):
    if not can_adopt_external_seed(ctx, seed_source.metadata):
        return error("E_POLICY_FAILED")

    seed_bytes   = canonicalise_external_seed(seed_source)
    trial_root   = derive_trial_root_from_external_seed(seed_bytes)
    fingerprint  = slice(shake256_256(trial_root), 0, 16)

    if not check_no_prior_pqhd_state(ctx, fingerprint):
        return error("E_IMPORT_MISMATCH")  // must use Secure Import instead

    // record event
    initialise_ledger_with_seed(ctx, trial_root, ctx.tick, ctx.local_sign_key, "external_hd_seed")

    // commit
    ctx.pqhd_root = trial_root
    ctx.pqhd_initialised = true
    derive_device_bound_keys(ctx)
    init_policy_engine(ctx)

    return true

J.10 Forbidden Behaviours
MUST NOT:
* adopt seed after PQHD initialisation
* replace existing root
* adopt seed controlling classical UTXOs
* bypass consent or tick validation
* adopt under PQVL drift

J.11 Recovery & Backup Notes
Adoption allows deterministic recovery using:
* original seed
* ledger export
* recovery capsules


ANNEX K — Delegated Authority (POA / Trustee Keys) (NORMATIVE–OPTIONAL)
Delegated authority allows a principal to give another party strictly limited, deterministic, revocable authority to perform specific wallet actions.
A delegated action MAY proceed only if the full PQHD custody predicate AND the delegated predicate both evaluate to true:
valid_for_signing_delegated =
      valid_tick
  AND valid_consent
  AND valid_policy
  AND valid_device
  AND valid_quorum
  AND valid_ledger
  AND valid_psbt
  AND valid_delegation
Delegation MUST NOT weaken any PQHD custody predicate.

K.1 Purpose
Delegated authority supports:
* estate planning
* emergency access
* operational company flows
* automation agents
* time-bound authority windows
* revocable trustee arrangements
Delegation is always:
* deterministic,
* tick-bound,
* scope-limited,
* revocable,
* attested.

K.2 Key Classes
Two optional key classes:
* poa (Power-of-Attorney keys)
* trustee (long-term custodial authority keys)
K.2.1 POA Key Derivation
poa_key = PQHD-DF(
    class_id = "poa",
    derivation_path = principal_pubkey || delegate_label || index
)
K.2.2 Trustee Key Derivation
trustee_key = PQHD-DF(
    class_id = "trustee",
    derivation_path = trust_id || trustee_label || index
)
Both MUST be deterministic and canonically encoded.
Pseudocode (Informative)
// Derive a POA or trustee key
function derive_poa_key(root_key, principal_pub, delegate_label, index):
    path   = concat(principal_pub, utf8(delegate_label), uint32_to_bytes(index))
    return pqhd_derive_with_domain(root_key, "Child-Key-poa", path)

function derive_trustee_key(root_key, trust_id, trustee_label, index):
    path   = concat(utf8(trust_id), utf8(trustee_label), uint32_to_bytes(index))
    return pqhd_derive_with_domain(root_key, "Child-Key-trustee", path)

K.3 POA Delegation Object
POA_Delegation = {
  delegation_id: tstr,
  principal_pub: bstr,
  delegate_pub:  bstr,
  scope:         { * tstr => any },
  limits:        { * tstr => uint },
  tick_issued:   uint,
  expiry_tick:   uint,
  revocable:     bool,
  policy_ref:    tstr / null,
  signature_pq:  bstr
}
Requirements:
* canonical CBOR or JCS JSON
* ML-DSA-65 signature
* explicit scopes & limits
* MUST include tick_issued and expiry_tick
* MUST specify principal_pub and delegate_pub

K.4 Trustee Delegation Object
Trustee_Delegation = {
  delegation_id: tstr,
  trust_id:      tstr,
  principal_pub: bstr,
  trustee_pub:   bstr,
  scope:         { * tstr => any },
  limits:        { * tstr => uint },
  tick_issued:   uint,
  expiry_tick:   uint / null,
  revocable:     bool,
  policy_ref:    tstr / null,
  signature_pq:  bstr
}
Expiry may be null for long-lived trustee arrangements.

K.5 Delegation Predicate
valid_delegation =
      delegation_object_canonical
  AND delegation_signature_valid
  AND tick_current ≥ tick_issued
  AND tick_current ≤ expiry_tick (or expiry_tick == null)
  AND scope_allows(action)
  AND limits_satisfied(action)
  AND policy_ref_active (if present)
MUST evaluate deterministically.
Pseudocode (Informative)
// Evaluate a generic delegation predicate
function evaluate_delegation(delegation, current_tick, action, amount, active_policies):
    // 1. Canonical encoding
    if canonical_encode(delegation_without_sig(delegation)) != delegation.raw_bytes:
        return false

    // 2. Signature
    if not verify_pq_signature(delegation.principal_pub,
                               delegation.raw_bytes,
                               delegation.signature_pq):
        return false

    // 3. Tick window
    if current_tick.t < delegation.tick_issued:
        return false
    if delegation.expiry_tick is not null and current_tick.t > delegation.expiry_tick:
        return false

    // 4. Scope
    if not scope_allows_action(delegation.scope, action):
        return false

    // 5. Limits
    if not limits_allow_amount(delegation.limits, action, amount):
        return false

    // 6. Policy reference
    if delegation.policy_ref is not null:
        if not policy_ref_is_active(delegation.policy_ref, active_policies):
            return false

    return true

K.6 Consent Binding
Delegated actions MUST use:
ConsentProof.action = "poa.<action>"
or
ConsentProof.action = "trustee.<action>"
ConsentProof MUST bind:
* delegation_id
* role & scope
* bundle_hash (if applicable)
* exporter_hash

K.7 Ledger Integration
Ledger MUST record:
* delegation_created
* delegation_revoked
* delegation_used
* delegation_expired
* delegation_invalid
Payloads MUST NOT contain sensitive identity information.
Pseudocode (Informative)
// Example: record delegation usage
function record_delegation_use(ctx, delegation, action, amount):
    payload = {
        delegation_id: delegation.delegation_id,
        action:        action,
        amount:        amount
    }
    append_ledger_event("delegation_used", ctx.tick, payload)

K.8 Revocation
DelegationRevocation = {
  delegation_id: tstr,
  tick:          EpochTick,
  signature_pq:  bstr
}
Revocation MUST:
* be tick-fresh
* be ML-DSA signed
* invalidate delegation immediately
Pseudocode (Informative)
// Revoke a delegation
function revoke_delegation(ctx, delegation, revocation_key):
    rev = {
        delegation_id: delegation.delegation_id,
        tick:          ctx.tick
    }
    bytes = canonical_encode(rev)
    rev.signature_pq = sign_pq(revocation_key, bytes)

    append_ledger_event("delegation_revoked", ctx.tick, { delegation_id: delegation.delegation_id })
    mark_delegation_inactive(delegation.delegation_id)

    return rev

K.9 Forbidden Behaviours
* delegation MUST NOT bypass any PQHD predicate
* MUST NOT reduce thresholds
* MUST NOT override ledger continuity
* MUST NOT use expired delegation
* MUST NOT allow delegate to reassign roles

K.10 Implementation Guidance
Recommended:
* POA = short-lived delegation
* Trustee = structural, long-lived
* clear UI visibility of scopes and limits
* KeyMail (Annex L) for high-risk delegated actions


ANNEX L — KeyMail Out-of-Band Confirmation (NORMATIVE–OPTIONAL)
KeyMail provides deterministic, cryptographically verifiable OOB confirmation for high-risk wallet operations.
KeyMail MUST NOT override custody predicates — it is a secondary verification channel.

L.1 Purpose
Supports secure confirmation for:
* large Bitcoin spends
* Secure Import
* multisig governance
* delegated actions
* recovery triggers
KeyMail ensures:
* two-device confirmation
* canonicalised prompts
* tick-fresh authority
* exporter-bound context

L.2 KeyMail Envelope Structure
KeyMailEnvelope = {
  km_id:          tstr,
  consent_ref:    tstr,
  action:         tstr,
  content_hash:   bstr,
  epoch_tick:     EpochTick,
  exporter_hash:  bstr,
  device_id:      tstr,
  signature_pq:   bstr    ; ML-DSA-65
}
Requirements:
* canonical CBOR/JCS
* ML-DSA-65 signature
* content_hash MUST be deterministic
* MUST bind to exporter_hash
* MUST include a fresh EpochTick

L.3 OOB Confirmation Predicate
KeyMail MUST enforce:
valid_oob =
      signature_valid
  AND tick_fresh
  AND exporter_hash_match
  AND device_attested
  AND consent_ref_valid
If any condition fails:
valid_policy = false
valid_for_signing = false
Pseudocode (Informative)
// Validate a KeyMail envelope
function validate_keymail_envelope(env, ctx):
    // 1. Canonical + signature
    bytes = canonical_encode(env_without_sig(env))
    if not verify_pq_signature(keymail_pubkey(env.device_id), bytes, env.signature_pq):
        return false

    // 2. Tick
    if not validate_tick(env.epoch_tick, ctx.last_tick):
        return false

    // 3. Exporter binding
    if env.exporter_hash != ctx.session.exporter_hash:
        return false

    // 4. Device attestation
    if ctx.attestation.drift_state != "NONE":
        return false

    // 5. Consent link
    if not consent_exists_with_id(env.consent_ref):
        return false

    return true

L.4 Consent Binding
KeyMailEnvelope MUST bind to ConsentProof via:
consent_ref = ConsentProof.intent_hash
This prevents mismatched actions.

L.5 Tick Requirements
EpochTick inside KeyMailEnvelope MUST satisfy:
* signature validity
* freshness ≤900 seconds
* profile_ref correctness
* monotonicity

L.6 Device Attestation Requirements
The confirming device MUST satisfy:
drift_state == "NONE"
all other PQVL requirements apply.

L.7 Ledger Integration
Ledger MUST record:
oob_confirmation
with minimally required metadata.
Pseudocode (Informative)
// Record an out-of-band confirmation
function record_keymail_confirmation(ctx, env):
    payload = {
        km_id:       env.km_id,
        consent_ref: env.consent_ref,
        action:      env.action
    }
    append_ledger_event("oob_confirmation", ctx.tick, payload)

L.8 Common Use Cases
* approval of high-value spends
* verifying delegated spending/authority
* confirming Secure Import
* verifying governance actions

L.9 Forbidden Behaviours
KeyMail MUST NOT:
* bypass policy
* accept stale ticks
* accept mismatched exporter_hash
* accept non-canonical envelopes
* accept un-attested devices

L.10 Implementation Guidance
* support mobile-to-hardware signer OOB flows
* enable QR-based offline confirmations
* maintain canonical encoding end-to-end

---

# **ANNEX M — Universal Secret Derivation (NORMATIVE–OPTIONAL)**

This annex defines deterministic non-custodial secrets used for:

* password generation
* API tokens
* identity attributes
* service-scoped secrets

Never used for Bitcoin signing.

---

## **M.1 Purpose**

Provides a PQ-safe mechanism for generating deterministic secrets bound to:

* root_key
* domain string
* canonical context parameters

---

## **M.2 Derivation Function**

```
Secret = cSHAKE256(
    root_key,
    domain = "PQHD-Secret:" || context,
    info   = canonical(context_parameters)
)
```

Requirements:

* deterministic and cross-device identical
* domain-separated
* MUST NOT mix entropy after initialization
* context parameters MUST be canonical

---

## **M.3 Context Parameters**

Examples:

```
{ "service": "example.com" }
{ "username": "alice" }
{ "purpose": "password" }
```

Forbidden:

* timestamps
* random bytes
* non-canonical maps
* device identifiers

---

## **M.4 Separation From Custody Keys**

Secrets MUST NOT be used for:

* PSBT signing
* governance
* recovery
* delegated authority
* Secure Import

---

## **M.5 Typical Use Cases**

* password manager vaults
* application access tokens
* L2 identity attributes (Annex T)
* KYC selective disclosure (Annex P)

---

## **M.6 Ledger Interaction**

Secret derivations MUST NOT write to the ledger.

---

## **M.7 Security Properties**

* unlinkable across services
* cannot reveal root_key
* safe under loss/theft
* identical across devices

---

## **M.8 Forbidden Behaviours**

* nondeterministic parameters
* mixing entropy after wallet initialization
* using secrets as signing keys

---

ANNEX N — Credential Vault (NORMATIVE–OPTIONAL)
A deterministic, encrypted vault for non-custodial secrets (from Annex M). Must NOT affect signing behaviour.

N.1 Purpose
Stores:
* passwords
* service API keys
* identity attributes
* encrypted metadata
All entries deterministic, encrypted, and portable.

N.2 Vault Keys
vault_DEK = PQHD-DF("vault", "/vault/dek")
vault_KEK = PQHD-DF("vault", "/vault/kek")
* DEK encrypts credentials
* KEK encrypts the DEK for export
* both MUST be deterministic
Pseudocode (Informative)
// Derive vault DEK/KEK from pqhd_root
function derive_vault_keys(pqhd_root):
    dek = pqhd_derive_with_domain(pqhd_root, "Child-Key-vault", utf8("/vault/dek"))
    kek = pqhd_derive_with_domain(pqhd_root, "Child-Key-vault", utf8("/vault/kek"))
    return (dek, kek)

N.3 Credential Object
Credential = {
  credential_id:      tstr,
  service:            tstr,
  username:           tstr / null,
  derived_secret:     bstr,
  metadata:           { * tstr => any } / null,
  last_rotated_tick:  uint,
  expiry_tick:        uint / null,
  signature_pq:       bstr
}
Requirements:
* canonical encoding
* encrypted with vault_DEK
* derived_secret MUST come from Annex M

N.4 Vault Storage Format
VaultEntry = {
  encrypted_payload:  bstr,
  credential_hash:    bstr,
  signature_pq:       bstr
}
Entries MUST be lexicographically ordered.

N.5 Vault Operations
Create
* derive secret
* canonicalise credential
* encrypt with DEK
* append to vault
Retrieve
* decrypt with DEK
* verify signature
* verify credential_hash
Rotate
* derive new secret
* set last_rotated_tick
* re-encrypt
Delete
* remove entry
* optionally record anonymised ledger event
Pseudocode (Informative)
// Create and store a credential in the vault
function vault_create_credential(vault, dek, signing_key, service, username, secret, tick):
    cred = {
        credential_id:      generate_credential_id(service, username),
        service:            service,
        username:           username,
        derived_secret:     secret,
        metadata:           null,
        last_rotated_tick:  tick.t,
        expiry_tick:        null
    }
    cred_hash = hash_canonical(cred)
    bytes     = canonical_encode(cred)
    cred.signature_pq = sign_pq(signing_key, bytes)

    encrypted_payload = encrypt_symmetric(dek, bytes)

    entry = {
        encrypted_payload: encrypted_payload,
        credential_hash:   cred_hash,
        signature_pq:      cred.signature_pq
    }

    vault.entries.append(entry)
    sort_vault_entries_lex(vault.entries)  // deterministic order
    return cred.credential_id
// Retrieve and verify credential
function vault_get_credential(vault, dek, credential_id):
    entry = find_vault_entry(vault, credential_id)
    if entry is null:
        return null

    bytes = decrypt_symmetric(dek, entry.encrypted_payload)
    cred  = decode_canonical(bytes)

    if hash_canonical(cred) != entry.credential_hash:
        return error("vault_integrity_failed")

    if not verify_pq_signature(vault_signing_pubkey(), bytes, entry.signature_pq):
        return error("vault_signature_invalid")

    return cred

N.6 Ledger Interaction
Ledger MUST NOT store:
* service names
* usernames
* secrets
* metadata
May store anonymised events.

N.7 Security & Privacy
* unlinkable across services
* deterministic across devices
* tamper evident
* encrypted end-to-end

N.8 Forbidden Behaviours
* unencrypted secrets
* bypassing PQVL attestation
* non-canonical credential objects
* using vault material as signing keys


ANNEX O — Service-Scoped API Key Derivation (NORMATIVE–OPTIONAL)
This annex defines deterministic, privacy-preserving API keys that do not interact with PQHD custody semantics.
API keys MUST NOT be used for PSBT signing, governance, recovery, or any custody predicate.

O.1 Purpose
Enables deterministic, unlinkable API keys for:
* service authentication
* application integrations
* automated tooling
* internal services
* per-account or per-scope credentials
With no exposure of PQHD custody keys.

O.2 Derivation Function
API keys are derived using Annex M (Universal Secret Derivation):
api_key = Secret(
  context = "api",
  context_parameters = {
      service: <service_identifier>,
      account: <optional>,
      scope:   <optional>,
      purpose: "service-api"
  }
)
Requirements:
* deterministic
* context_parameters MUST be canonical
* no entropy after initialization
* MUST NOT include device identifiers
Pseudocode (Informative)
// Derive a service-scoped API key using Universal Secret derivation
function derive_api_key(pqhd_root, service, account, scope):
    context = "api"
    params = {
        service: service,
        account: account,
        scope:   scope,
        purpose: "service-api"
    }
    return universal_secret(pqhd_root, context, params)

O.3 Material Format
API key output MAY be encoded as:
* base64url
* base58
* hex
* deterministic CBOR
Implementations MUST NOT alter underlying bytes.

O.4 Service Binding
Every API key MUST bind to a canonical:
{ "service": "<identifier>" }

O.5 Scope & Permissions
Scopes MAY be included deterministically.

O.6 Rotation Rules
Rotations MUST be deterministic:
api_key_v3 = Secret(
   context = "api",
   context_parameters = {
       service: "example.com",
       scope:   "read-only",
       rotation: 3
   }
)
Rotation MUST NOT break determinism.

O.7 Storage Rules
Recommended:
* store API keys in the Credential Vault (Annex N)
* do not store plaintext keys
* use KEK/DEK encryption for vault exports

O.8 Ledger Interaction
Ledger MUST NOT store:
* API keys
* service identifiers
* scopes
* rotation counters
May record anonymized:
api_key_rotated

O.9 Forbidden Behaviours
MUST NOT:
* derive API keys using entropy
* embed API keys in PSBTs
* treat API keys as PQHD custody keys
* bypass PQVL attestation for vault operations


ANNEX P — Verified Identity & KYC Credentials (NORMATIVE–OPTIONAL)
Identity flows do not affect PQHD custody. This annex defines deterministic identity and selective disclosure mechanisms for regulated or institutional contexts.

P.1 Purpose
Provides deterministic identity tools for:
* age or jurisdiction checks
* compliance-bound workflows
* institution-issued “know-your-customer” attributes
* privacy-preserving disclosure
All without putting identity into the ledger.

P.2 KYCCredential Structure
KYCCredential = {
  cred_id:       tstr,
  issuer_id:     tstr,
  subject_id:    tstr,
  attributes:    { * tstr => any },
  issued_at:     uint,
  expires_at:    uint / null,
  tick:          EpochTick,
  issuer_sig_pq: bstr
}
Requirements:
* canonical encoding
* ML-DSA-65 signature by issuer
* attributes MUST be explicitly user-approved
* subject_id MUST be pseudonymous

P.3 Selective Disclosure
KYCDisclosure = {
  cred_id:      tstr,
  disclosed:    { * tstr => any },
  tick:         EpochTick,
  consent_ref:  tstr,
  signature_pq: bstr
}
Requirements:
* canonical encoding
* ML-DSA-65 signature
* MUST bind to ConsentProof via consent_ref
* tick MUST be fresh
Verifier MUST confirm:
* disclosed fields match original credential
* issuer signature is valid
* tick is fresh
* ConsentProof is valid
Pseudocode (Informative)
// Verify a KYC disclosure against a stored KYCCredential
function verify_kyc_disclosure(credential, disclosure, current_tick, consent):
    if disclosure.cred_id != credential.cred_id:
        return false

    // Tick validity
    if not validate_tick(disclosure.tick, current_tick.t):
        return false

    // Consent binding
    if disclosure.consent_ref != consent.intent_hash:
        return false

    // Signature on disclosure
    bytes = canonical_encode(disclosure_without_sig(disclosure))
    if not verify_pq_signature(credential.issuer_id_pubkey, bytes, disclosure.signature_pq):
        return false

    // Attribute consistency
    for key, value in disclosure.disclosed.items():
        if credential.attributes[key] != value:
            return false

    // Expiry
    if credential.expires_at is not null and current_tick.t > credential.expires_at:
        return false

    return true

P.4 Verification Rules
Verifier MUST:
1. canonicalise disclosure
2. verify ML-DSA signature
3. enforce tick window
4. validate ConsentProof
5. cross-check disclosed attributes

P.5 Privacy Guarantees
* only disclosed fields visible
* no global correlation
* subject_id pseudonymous
* ledger MUST NOT store identity attributes

P.6 Vault Integration
KYCCredential objects MAY be encrypted and stored inside the Credential Vault (Annex N).
Vault MUST NOT expose:
* issuer_id
* subject_id
* attribute maps

P.7 Consent Binding
Identity actions MUST use:
ConsentProof.action = "identity.disclose"
Consent MUST be tick-fresh.

P.8 Ledger Interaction
Ledger MAY record:
kyc_disclosure_issued
MUST NOT contain identity data.

P.9 Forbidden Behaviours
MUST NOT:
* embed identity in PSBTs
* bypass consent
* leak attributes into ledger
* accept stale ticks
* accept invalid ConsentProof


ANNEX Q — Delegated Identity (NORMATIVE–OPTIONAL)
Delegated Identity allows a principal to grant limited identity capabilities to another device or entity. It does NOT allow signing authority unless combined with Annex K.

Q.1 Purpose
Supports:
* helper devices
* care-giver or assistant roles
* service login delegation
* organisational identity
* limited per-scope functionality

Q.2 DelegatedIdentity Structure
DelegatedIdentity = {
  delegation_id:  tstr,
  subject_id:     tstr,
  delegate_id:    tstr,
  scopes:         [* tstr],
  limits:         { * tstr => uint },
  valid_from:     uint,
  valid_until:    uint,
  tick:           EpochTick,
  consent_ref:    tstr,
  signature_pq:   bstr
}
Requirements:
* canonical encoding
* ML-DSA-65 signature
* scopes MUST be explicit
* limits MUST be deterministic
* tick MUST be fresh
* MUST bind to ConsentProof via consent_ref

Q.3 Verification Rules
Verifier MUST ensure:
* canonical encoding
* signature_pq valid
* tick fresh and monotonic
* ConsentProof valid
* requested action ∈ scopes
* limits not exceeded
* valid_from ≤ tick_current ≤ valid_until
Pseudocode (Informative)
// Evaluate a delegated identity claim
function verify_delegated_identity(identity, current_tick, action, consent):
    if not validate_tick(identity.tick, current_tick.t):
        return false

    if identity.consent_ref != consent.intent_hash:
        return false

    bytes = canonical_encode(identity_without_sig(identity))
    if not verify_pq_signature(subject_pubkey(identity.subject_id), bytes, identity.signature_pq):
        return false

    if current_tick.t < identity.valid_from:
        return false
    if current_tick.t > identity.valid_until:
        return false

    if action not in identity.scopes:
        return false

    // limit checking is deployment-specific; placeholder
    // for each scope, ensure usage not exceeding limits
    return true

Q.4 Consent Binding
Delegation MUST use:
ConsentProof.action = "identity.delegate"

Q.5 Time Requirements
Delegation is valid only when:
valid_from ≤ tick_current ≤ valid_until
Outside this window → invalid.

Q.6 Ledger Interaction
Ledger MAY record pseudonymous:
* delegation_id_created
* delegation_id_expired
* delegation_id_revoked
Must not include subject_id, delegate_id, or attributes.

Q.7 Revocation
DelegationRevocation = {
  delegation_id: tstr,
  tick:          EpochTick,
  signature_pq:  bstr
}
Revocation MUST be immediate.

Q.8 Forbidden Behaviours
* delegated identity MUST NOT bypass PQHD custody
* MUST NOT leak identity attributes
* MUST NOT accept stale ticks
* MUST NOT override drift_state


ANNEX R — Delegated Spending (NORMATIVE–OPTIONAL)
Delegated Spending enables controlled, revocable spending authority with strict limits, merchant controls, and tick-bound windows.
It NEVER bypasses PQHD custody predicates.

R.1 Purpose
Supports:
* family allowances
* prepaid spending limits
* employee expense controls
* subscription-style spending
* delegated merchant channels

R.2 DelegatedPaymentCredential Structure
DelegatedPaymentCredential = {
  delegation_id:     tstr,
  payer_id:          tstr,
  delegate_id:       tstr,
  allowed_merchants: [* tstr] / null,
  spend_limit_minor: uint,
  currency:          tstr,
  per_tx_limit:      uint / null,
  interval_limit:    uint / null,
  valid_from:        uint,
  valid_until:       uint,
  tick:              EpochTick,
  consent_ref:       tstr,
  signature_pq:      bstr
}
Requirements:
* canonical encoding
* ML-DSA-65 signature
* tick freshness
* explicit limits and scopes

R.3 Delegated Payment Transaction
DelegatedPayment = {
  delegation_id: tstr,
  intent:        PaymentIntent,
  delegate_tick: EpochTick,
  signature_pq:  bstr
}
Where PaymentIntent may include:
* amount
* merchant_id
* memo
Delegate signs using delegated key.

R.4 Verification Rules
Delegate payment valid only if:
1. canonical credential
2. signature_pq valid
3. fresh EpochTick
4. ConsentProof valid
5. merchant allowed (if applicable)
6. amount ≤ per_tx_limit
7. cumulative interval ≤ interval_limit
8. total spends ≤ spend_limit_minor
9. drift_state == "NONE"
10. ledger monotonic
Pseudocode (Informative)
// Validate a single delegated payment against a payment credential
function verify_delegated_payment(cred, payment, current_tick, consent, runtime_state, ledger_view):
    // 1. Runtime and tick
    if runtime_state.drift_state != "NONE":
        return false
    if not validate_tick(payment.delegate_tick, current_tick.t):
        return false

    // 2. Credential linkage
    if payment.delegation_id != cred.delegation_id:
        return false

    // 3. Credential time window
    if current_tick.t < cred.valid_from:
        return false
    if current_tick.t > cred.valid_until:
        return false

    // 4. Consent binding
    if cred.consent_ref != consent.intent_hash:
        return false

    // 5. Merchant constraints
    if cred.allowed_merchants is not null:
        if payment.intent.merchant_id not in cred.allowed_merchants:
            return false

    // 6. Amount constraints
    if cred.per_tx_limit is not null:
        if payment.intent.amount > cred.per_tx_limit:
            return false

    spent_in_interval = compute_interval_spend(cred, ledger_view, current_tick)
    if cred.interval_limit is not null and spent_in_interval + payment.intent.amount > cred.interval_limit:
        return false

    total_spent = compute_total_spend(cred, ledger_view)
    if total_spent + payment.intent.amount > cred.spend_limit_minor:
        return false

    // 7. Signature over payment
    bytes = canonical_encode(payment_without_sig(payment))
    if not verify_pq_signature(delegate_pubkey(cred.delegate_id), bytes, payment.signature_pq):
        return false

    return true

R.5 Consent Binding
Delegated payments MUST use:
ConsentProof.action = "payment.execute"
Consent MUST bind to:
* delegation_id
* PaymentIntent hash
* bundle_hash (if PSBT-based)
* exporter_hash

R.6 Ledger Interaction
Ledger MAY record pseudonymous:
delegated_payment_created
delegated_payment_used
delegated_payment_revoked
MUST NOT contain:
* merchant names
* payer/delegate identity
* transaction metadata

R.7 Revocation
DelegatedPaymentRevocation = {
  delegation_id: tstr,
  tick:          EpochTick,
  signature_pq:  bstr
}
Revocation MUST immediately disable delegated spending.

R.8 Forbidden Behaviours
MUST NOT:
* bypass custody predicates
* reuse expired credentials
* ignore limits
* override ledger ordering
* accept actions under drift_state ≠ "NONE"

R.9 Implementation Guidance
Recommended:
* UI surfacing of spending limits
* QR-based offline delegated payments
* KeyMail for high-value delegated spending approvals

---

ANNEX S — Multi-Chain Deterministic Interoperability (NORMATIVE–OPTIONAL)
This annex defines deterministic, isolated key derivation for non-Bitcoin ecosystems. These keys MUST remain completely separate from PQHD custody keys.
They MAY be used for:
* Ethereum accounts
* Solana/Ed25519 keys
* Cosmos/SLIP10 systems
* Polkadot/SR25519
* any standardized chain derivation
They MUST NOT be used for Bitcoin spending authority.

S.1 Purpose
Multi-chain derivation enables:
* one seed → many ecosystems
* deterministic cross-platform compatibility
* safe export to external wallets
* user-controlled chain segregation
* zero cross-chain key reuse
All without affecting Bitcoin custody semantics.

S.1A Security Boundary for Non-Bitcoin Chains (INFORMATIVE)

PQHD may derive deterministic keys for external chains (e.g. Ethereum,
Solana, Cosmos, Polkadot), but PQHD-level custody guarantees do not extend
beyond Bitcoin unless those chains implement equivalent predicate enforcement.

For external chains:

• PQHD controls only deterministic key derivation and optional public-key export.
• PQHD does not control or interpret transaction structures for those chains.
• PQHD predicates (valid_tick, valid_consent, valid_policy, valid_device,
  valid_quorum, valid_ledger, valid_psbt) are not enforced for external chain
  transactions.
• PQVL runtime integrity is not enforced by external chain wallets.
• EpochTick temporal authority is not applied to non-Bitcoin transactions.
• ConsentProof, exporter binding, canonical transaction equivalence, and
  ledger-anchored governance are not supported on those chains.

As a result, exporting PQHD-derived keys to an external wallet provides
deterministic key generation but not PQHD-level spending security. Achieving
equivalent protection on other chains would require full PQHD integration on
those chains, including PQVL attestation, EpochTick semantics, ConsentProof,
policy engines, deterministic transaction structures, and multisig predicate
enforcement.

Without these protections, exported chain keys remain vulnerable to external
risks outside the PQHD security boundary, including:

• runtime malware producing arbitrary signatures,
• replay attacks using stale or forged authorisation,
• policy violations such as sending to unapproved destinations, and
• transaction malleation or structural attacks specific to the external chain.

These risks exist because external chain clients typically do not implement
PQVL runtime integrity, EpochTick temporal authority, canonical intent binding,
or PQHD’s policy and ledger predicates.

S.2 ChainDescriptor Structure
ChainDescriptor = {
  chain_id:        tstr,
  standard:        tstr,        ; "BIP44", "SLIP10", "SOL-ED25519", etc.
  derivation_root: tstr,        ; e.g. "m/44'/60'/0'/0"
  address_format:  tstr,        ; "eth_checksum", "sol_base58", etc.
  sig_type:        tstr,        ; "ecdsa-secp256k1", "ed25519", etc.
  metadata:        { * tstr => any }
}
MUST be canonical.
Pseudocode (Informative)
// Normalise a ChainDescriptor into canonical bytes for hashing or KDF context
function canonicalise_chain_descriptor(desc):
    obj = {
        chain_id:        desc.chain_id,
        standard:        desc.standard,
        derivation_root: desc.derivation_root,
        address_format:  desc.address_format,
        sig_type:        desc.sig_type,
        metadata:        desc.metadata
    }
    return canonical_encode(obj)

S.3 Chain Master Derivation
Every supported chain MUST derive from an isolated namespace:
chain_master = PQHD-DF(
    class_id = "chain",
    derivation_path = "/chain/" || chain_id
)
Ensures:
* deterministic
* no cross-chain collisions
* no leakage into custody keyspace
Pseudocode (Informative)
// Derive a chain master key for a given chain_id
function derive_chain_master(pqhd_root, chain_id):
    path = "/chain/" + chain_id
    return pqhd_derive_with_domain(pqhd_root, "Child-Key-chain", utf8(path))

S.4 Child Key Derivation
chain_key = cSHAKE256(
    chain_master,
    domain = "PQHD-Chain:" || chain_id,
    info   = canonical({
        standard:        chain_descriptor.standard,
        derivation_root: chain_descriptor.derivation_root,
        path:            path
    })
)
Chain keys MUST:
* match chain’s expected key format (ECDSA/Ed25519/SR25519)
* be canonical and deterministic
* be independent of runtime state
Pseudocode (Informative)
// Derive a specific chain key (e.g. account/path) from a chain_master
function derive_chain_key(chain_master, chain_descriptor, path):
    info_obj = {
        standard:        chain_descriptor.standard,
        derivation_root: chain_descriptor.derivation_root,
        path:            path
    }
    info_bytes = canonical_encode(info_obj)
    domain     = "PQHD-Chain:" + chain_descriptor.chain_id
    return cshake256_derive(chain_master, domain, info_bytes)

S.5 Supported Chains (Examples)
Ethereum (ETH)
chain_id:        "eth"
standard:        "BIP44"
derivation_root: "m/44'/60'/0'/0"
address_format:  "eth_checksum"
sig_type:        "ecdsa-secp256k1"
Solana (SOL)
chain_id:        "sol"
standard:        "SOL-ED25519"
derivation_root: "m/44'/501'/0'"
address_format:  "sol_base58"
sig_type:        "ed25519"
Cosmos (ATOM)
chain_id:        "cosmos"
standard:        "SLIP10"
derivation_root: "m/44'/118'/0'/0/0"
address_format:  "bech32"
sig_type:        "secp256k1"
Polkadot (DOT)
chain_id:        "dot"
standard:        "SR25519-DERIVATION"
derivation_root: "//hard/soft/path"
address_format:  "ss58"
sig_type:        "sr25519"
Pseudocode (Informative)
// Convenience helper: derive a specific account key for a named chain
function derive_chain_account_key(pqhd_root, chain_id, path):
    desc  = lookup_chain_descriptor(chain_id)  // implementation-specific registry
    master = derive_chain_master(pqhd_root, chain_id)
    return derive_chain_key(master, desc, path)

S.6 Exporting Chain Keys
chain_export = {
  chain_id,
  address,
  public_key,
  exported_at_tick,
  signature_pq
}
Requirements:
* MUST NOT export private keys
* MUST NOT expose chain_master
* exported_at_tick MUST be fresh
Pseudocode (Informative)
// Export a chain address/public key without revealing private material
function build_chain_export(pqhd_root, chain_id, path, current_tick, signing_key):
    desc   = lookup_chain_descriptor(chain_id)
    master = derive_chain_master(pqhd_root, chain_id)
    chain_priv = derive_chain_key(master, desc, path)
    (pub, addr) = derive_public_and_address(chain_priv, desc)

    export_obj = {
        chain_id:        chain_id,
        address:         addr,
        public_key:      pub,
        exported_at_tick: current_tick.t
    }

    bytes = canonical_encode(export_obj)
    export_obj.signature_pq = sign_pq(signing_key, bytes)

    return export_obj

S.7 Isolation Rules
Chain keys MUST be isolated from:
* PQHD custody keys
* Secure Import
* Policy Enforcer logic
* multisig roles
* recovery capsules
* ledger events
Chain keys are external only.

S.8 Ledger Interaction
Ledger MAY record:
chain_key_exported
MUST NOT include:
* chain private keys
* addresses
* derivation paths
Pseudocode (Informative)
// Record that a chain key/address has been exported, without leaking details
function record_chain_export(ctx, chain_id):
    payload = { chain_id: chain_id }
    append_ledger_event("chain_key_exported", ctx.tick, payload)

S.9 Forbidden Behaviours
MUST NOT:
* derive chain keys nondeterministically
* reuse PQHD custody keys for chains
* embed chain metadata in PSBTs
* treat chain keys as Bitcoin signing authority


ANNEX T — Cross-Chain & Layer-2 Integration (NORMATIVE–OPTIONAL)
Annex T defines deterministic namespaces for Layer-2 ecosystems and smart-contract platforms, ensuring these integrations do NOT impact Bitcoin custody.

T.1 Purpose
Supports:
* Lightning Network
* L2 rollups (Optimism, Arbitrum, zkSync, StarkNet)
* DeFi protocol integrations
* sidechains
* LSP (Lightning Service Provider) flows
WITHOUT affecting:
* Bitcoin policy
* custody keys
* PQVL predicates
* ledger rules
* multisig semantics

T.2 L2 Namespace Hierarchy
l2_master = PQHD-DF(
    class_id = "chain",
    derivation_path = "/chain/" || chain_id || "/l2/" || l2_id
)
Ensures each L2 ecosystem is properly sandboxed.
Pseudocode (Informative)
// Derive an L2 master for <chain_id, l2_id>
function derive_l2_master(pqhd_root, chain_id, l2_id):
    path = "/chain/" + chain_id + "/l2/" + l2_id
    return pqhd_derive_with_domain(pqhd_root, "Child-Key-chain", utf8(path))

T.3 Child L2 Key Derivation
l2_key = cSHAKE256(
    l2_master,
    domain = "PQHD-L2:" || chain_id || ":" || l2_id,
    info   = canonical({
        path:    l2_path,
        context: l2_context
    })
)
Context MUST be deterministic.
Pseudocode (Informative)
// Derive a specific L2 key within an L2 namespace
function derive_l2_key(l2_master, chain_id, l2_id, l2_path, l2_context):
    info_obj = {
        path:    l2_path,
        context: l2_context
    }
    info_bytes = canonical_encode(info_obj)
    domain     = "PQHD-L2:" + chain_id + ":" + l2_id
    return cshake256_derive(l2_master, domain, info_bytes)

T.4 Lightning Network Integration
ln_master = PQHD-DF("chain", "/chain/btc/l2/ln")
Commitment key derivation:
commitment_key(i) = cSHAKE256(
    ln_master,
    domain = "PQHD-L2:btc:ln",
    info   = canonical({
        channel_index: c,
        commitment_index: i
    })
)
Lightning keys MUST NOT be used for on-chain Bitcoin signing.
Pseudocode (Informative)
// Derive Lightning commitment keys deterministically
function derive_ln_master(pqhd_root):
    path = "/chain/btc/l2/ln"
    return pqhd_derive_with_domain(pqhd_root, "Child-Key-chain", utf8(path))

function derive_ln_commitment_key(ln_master, channel_index, commitment_index):
    info_obj = {
        channel_index:   channel_index,
        commitment_index: commitment_index
    }
    info_bytes = canonical_encode(info_obj)
    domain     = "PQHD-L2:btc:ln"
    return cshake256_derive(ln_master, domain, info_bytes)

T.5 Rollup & Smart-Contract Integrations
rollup_master = PQHD-DF(
    class_id="chain",
    derivation_path="/chain/" || chain_id || "/l2/rollup:" || rollup_name
)
Child keys:
rollup_key = cSHAKE256(
    rollup_master,
    domain = "PQHD-L2:" || chain_id || ":rollup:" || rollup_name,
    info = canonical({
        path,
        contract,
        function,
        context
    })
)
Supports:
* EVM contract accounts
* StarkNet felt-based keys
* zk-rollup proving keys
* account abstraction identities
Pseudocode (Informative)
// Derive a rollup-specific key for a given contract/function/context
function derive_rollup_master(pqhd_root, chain_id, rollup_name):
    path = "/chain/" + chain_id + "/l2/rollup:" + rollup_name
    return pqhd_derive_with_domain(pqhd_root, "Child-Key-chain", utf8(path))

function derive_rollup_key(rollup_master, chain_id, rollup_name, path, contract, fn, context):
    info_obj = {
        path:     path,
        contract: contract,
        function: fn,
        context:  context
    }
    info_bytes = canonical_encode(info_obj)
    domain     = "PQHD-L2:" + chain_id + ":rollup:" + rollup_name
    return cshake256_derive(rollup_master, domain, info_bytes)

T.6 DeFi Integrations (Informative)
Examples:
* AMM liquidity position keys
* staking keys
* internal depository account keys
* DEX interaction keys
All MUST be deterministic and sandboxed.

T.7 Lightning Service Provider (LSP) Integration
lsp_key = PQHD-DF(
    class_id = "chain",
    derivation_path = "/chain/btc/l2/lsp:" || provider_id
)
Used for LSP authentication ONLY.
Pseudocode (Informative)
// Derive authentication keys for an LSP relationship
function derive_lsp_key(pqhd_root, provider_id):
    path = "/chain/btc/l2/lsp:" + provider_id
    return pqhd_derive_with_domain(pqhd_root, "Child-Key-chain", utf8(path))

T.8 Export Policies
l2_export = {
  chain_id,
  l2_id,
  address,
  public_key,
  exported_at_tick,
  signature_pq
}
Private keys MUST NOT be exported. exported_at_tick MUST be fresh.
Pseudocode (Informative)
// Export an L2 public identity (no private key)
function build_l2_export(pqhd_root, chain_id, l2_id, l2_path, l2_context, current_tick, signing_key):
    l2_master = derive_l2_master(pqhd_root, chain_id, l2_id)
    l2_priv   = derive_l2_key(l2_master, chain_id, l2_id, l2_path, l2_context)
    (pub, addr) = derive_l2_public_and_address(l2_priv, chain_id, l2_id)

    obj = {
        chain_id:        chain_id,
        l2_id:           l2_id,
        address:         addr,
        public_key:      pub,
        exported_at_tick: current_tick.t
    }
    bytes = canonical_encode(obj)
    obj.signature_pq = sign_pq(signing_key, bytes)

    return obj

T.9 Ledger Interaction
Ledger MUST NOT include:
* chain metadata
* addresses
* L2 identifiers
* public keys
* contract references
May include:
l2_key_exported
Pseudocode (Informative)
// Record export of an L2 key/address without leaking details
function record_l2_export(ctx, chain_id, l2_id):
    payload = {
        chain_id: chain_id,
        l2_id:    l2_id
    }
    append_ledger_event("l2_key_exported", ctx.tick, payload)

T.10 Forbidden Behaviours
MUST NOT:
* reuse L2 keys for Bitcoin custody
* derive L2 keys with nondeterministic input
* embed L2 keys in ledger events
* weaken the custody predicate
* leak chain identifiers in ledger payloads

---

# **ANNEX U — Deterministic Derivation Security (NORMATIVE–OPTIONAL)**


## **U.1 Deterministic Derivation Security**

PQHD’s cSHAKE256 deterministic derivation ensures:

* all devices produce identical keys,
* multisig remains cross-device consistent,
* recovery and air-gapped workflows work without extra entropy,
* accidental key reuse is prevented through explicit domain separation.

Deterministic derivation is fundamental for:

* offline security,
* reproducibility,
* safe multisig,
* independent validation.

---

## **U.2 Predicate Completeness**

Security arises from requiring **all seven predicates** to be true:

```
valid_for_signing =
      valid_tick
  AND valid_consent
  AND valid_policy
  AND valid_device
  AND valid_quorum
  AND valid_ledger
  AND valid_psbt
```

Each predicate blocks an entire class of attacks:

* **valid_tick** → stale ticks, replay, timestamp substitution
* **valid_consent** → forged or stale intent
* **valid_policy** → bypassing role/limit/destination rules
* **valid_device** → compromised runtime or malware
* **valid_quorum** → insufficient roles or missing devices
* **valid_ledger** → rollback, forked state, coordinator tampering
* **valid_psbt** → malleation, structure drift, or mismatched outputs

Attacker must break **all seven simultaneously** — designed to be infeasible.

---

## **U.3 Replay Resistance**

Replay attacks are eliminated by combining:

* tick freshness ≤900s
* tick monotonicity
* ConsentProof tick_issued / tick_expiry
* exporter_hash session binding
* bundle_hash binding to PSBT
* ledger continuity
* tick-bound derivations for consent and session keys

A leaked PSBT cannot be reused because:

* the tick expires,
* consent expires,
* exporter_hash mismatches new session,
* bundle_hash mismatches canonical PSBT,
* ledger advances.

---

## **U.4 Runtime Integrity & Drift Defense**

PQVL provides runtime-integrity guarantees:

* binary integrity
* enclave integrity (if present)
* correct OS/process state
* environment coherence
* deterministic, tamper-evident measurement_hash

Any drift produces:

```
valid_device = false
```

and PQHD enters fail-closed.

This prevents:

* malware-injected signing
* runtime patching
* shadow signing from compromised devices
* corrupted interpreters
* misconfigured runtime environments

---

## **U.5 Secure Import Soundness**

Secure Import prevents attackers who possess classical seeds or keys from bypassing PQHD.

It enforces:

* dual Proof-of-Possession (classical + ML-DSA-65)
* tick freshness
* deterministic mapping to PQHD keys
* optional guardian approval
* policy-controlled migration
* ledger anchoring for auditability

A classical key collapse does **not** compromise PQHD.

---

## **U.6 Recovery Capsule Safety**

Recovery Capsules are secure because:

* **ML-KEM** ensures confidentiality of recovery material,
* **ML-DSA** ensures authenticity,
* threshold approvals prevent single-party takeover,
* tick-bound delays prevent rushed recovery,
* ledger anchoring prevents replay or misuse,
* role-based recovery prevents unauthorised transitions.

Attackers cannot accelerate, replay, or steal recovery keys.

---

## **U.7 Ledger Immutability & Anti-Rollback**

Anti-rollback properties come from:

* tick monotonicity,
* prev_hash continuity,
* deterministic Merkle roots,
* append-only semantics,
* freeze rules on mismatch.

Any deviation (rollback, mismatch, or tick rewind) freezes the ledger.

---

## **U.8 Multi-Device Determinism**

PQHD ensures that every device:

* canonicalises the PSBT identically,
* computes the same bundle_hash,
* derives the same tick-bound role keys,
* evaluates all predicates deterministically.

This blocks:

* coordinator manipulation,
* cross-device divergence,
* inconsistent signing results,
* malicious client-side transforms.

---

## **U.9 Temporal Authority Resilience**

Epoch Clock secures time by eliminating:

* NTP poisoning
* system clock manipulation
* timezone attacks
* user clock drift
* leap-second ambiguity

Time becomes a cryptographically signed, monotonic authority.

---

## **U.10 Cross-Chain Isolation**

Annex S/T key domains ensure:

* no reuse of custody keys on other chains,
* no signature cross-contamination,
* deterministic but isolated derivations,
* zero influence on Bitcoin security.

Multi-chain capability never weakens PQHD custody.

---

## **U.11 Quantum Resistance**

PQHD is quantum-resistant because:

* **ML-DSA-65** signatures
* **SHAKE256** hashing
* **cSHAKE256** domain-separated derivation
* **ML-KEM** for encrypted payloads

Classical ECDSA signatures are never accepted for Bitcoin spending.

Even a full ECDSA break does **not** compromise PQHD.

---

## **U.12 Why PQHD Must Fail-Closed**

Fail-closed ensures that **any ambiguous or unsafe condition halts signing**.

Triggered by:

* drift
* stale tick
* rollback
* invalid consent
* bundle_hash mismatch
* quorum failure
* ledger mismatch
* policy mismatch

This prevents attackers from exploiting edge cases or partial failures.

---

ANNEX V — Normative Ecosystem Dependencies (NORMATIVE)
This annex makes explicit all external primitives and behaviours PQHD relies on from the wider PQ ecosystem:
* Epoch Clock (temporal authority)
* PQVL (runtime and device integrity)
* PQSF (consent, transport, policy, ledger)
* PQAI (SafePrompt and AI runtime)
* PQ umbrella invariants (unified predicates and profile alignment)
All rules in this annex are normative for PQHD implementations. Where a structure is already defined in another annex or section, this annex binds PQHD to its behavioural requirements and provides pseudocode to remove ambiguity for implementers.

V.1 Scope
1. PQHD MUST implement wallet behaviour in a way that is compatible with:
    * EpochTick semantics (Annex A).
    * AttestationEnvelope semantics (PQVL).
    * ConsentProof, policy, and ledger semantics (PQSF).
    * SafePrompt semantics and AI runtime rules (PQAI).
    * The unified predicate model defined in the PQ ecosystem.
2. PQHD MAY be implemented without separate copies of the PQVL, PQSF, PQAI, or PQ umbrella documents, provided that all behaviours specified in this annex are implemented as written.

V.2 Epoch Clock Dependencies
V.2.1 Canonical Tick Requirements
Annex A defines the canonical EpochTick structure. PQHD MUST additionally enforce:
1. Signature and encoding
    * tick.alg MUST be "ML-DSA-65".
    * tick.sig MUST be a valid ML-DSA-65 signature over the canonical encoding.
    * Canonical encoding MUST be either:
        * deterministic CBOR; or
        * canonical JSON (JCS-style).
2. Freshness
    * A tick MUST be considered valid only if: 0 <= (current_wall_clock() - tick.t) <= 900
    * 
    * current_wall_clock() MAY use system time only to measure elapsed seconds.
    * Absolute time (the value of t) MUST be trusted only when provided by EpochTicks.
3. Monotonicity
    * For each wallet instance, ticks MUST be strictly monotonic: tick.t > last_valid_tick.t
    * 
    * Non-monotonic ticks MUST be rejected and MUST cause valid_tick = false.
4. Profile reference
    * tick.profile_ref MUST match the canonical Epoch Clock profile reference configured for PQHD.
    * PQHD MUST NOT accept ticks that reference unknown, deprecated, or untrusted profiles.
V.2.2 Mirror Behaviour
When PQHD relies on multiple Epoch Clock mirrors:
1. PQHD MUST query ≥2 independent mirrors for a tick.
2. PQHD MUST reject:
    * sets of ticks with mismatched profile_ref;
    * sets in which no consensus can be reached on {t, profile_ref, alg, sig} for the current window.
3. PQHD SHOULD treat a tick as valid only if at least two mirrors return identical canonical tick objects.
Pseudocode (Informative) — Mirror Consensus
function pqhd_get_consensus_tick(mirrors, last_valid_tick, profile_ref):
    candidates = []

    for m in mirrors:
        tick = m.get_current_tick()
        if not validate_epoch_tick(tick, last_valid_tick, profile_ref):
            continue
        candidates.append(tick)

    // group by canonical payload (excluding sig if implementation differs)
    groups = group_by_canonical_payload(candidates)

    consensus_group = max(groups, key=group_size)

    if size(consensus_group) < 2:
        return (null, "E_TICK_NO_CONSENSUS")

    // return any representative from the consensus group
    return (consensus_group[0], null)
V.2.3 Tick Reuse Rules and Modes
PQHD MUST implement tick reuse according to mode:
* Online mode:
    * A tick MAY be reused for multiple operations within a ≤900-second window.
* Offline mode:
    * An implementation MAY configure a stricter reuse window or require a fresh tick per operation.
* Stealth or high-risk mode:
    * Implementations SHOULD require a fresh tick for each operation.
Local/system time MAY be used only to measure elapsed time since tick.t. PQHD MUST NOT derive absolute time from local clocks for predicate evaluation.
Pseudocode (Informative) — Tick Validation
function validate_epoch_tick(tick, last_valid_tick, profile_ref):
    if tick.alg != "ML-DSA-65":
        return false

    if not verify_ml_dsa_signature(tick):
        return false

    if not is_canonical_encoding(tick):
        return false

    if tick.profile_ref != profile_ref:
        return false

    if last_valid_tick != null and tick.t <= last_valid_tick.t:
        return false

    // freshness: strictly bounded
    delta = current_wall_clock() - tick.t
    if delta < 0 or delta > 900:
        return false

    return true
V.2.4 Profile Lineage
1. Epoch Clock profiles are immutable.
2. If PQHD migrates from one profile to another, it MUST:
    * validate the new profile’s lineage against the canonical Epoch Clock lineage rules; and
    * store a ledger entry documenting the migration.
3. PQHD MUST NOT mix ticks from different profiles within a single wallet state unless the migration event is explicitly recorded and future predicates reference the correct profile.

V.3 PQVL Dependencies — Runtime and Device Integrity
PQHD depends on PQVL to provide attestations of device/runtime integrity.
V.3.1 Canonical AttestationEnvelope
PQHD MUST treat an AttestationEnvelope as having at least the following canonical fields:
AttestationEnvelope = {
  "envelope_id":      bstr,
  "device_id":        tstr,
  "measurement_hash": bstr,
  "drift_state":      tstr,   // "NONE" | "WARNING" | "CRITICAL"
  "probes":           [ *ProbeResult ],
  "tick":             uint,   // EpochTick.t at issuance
  "signature_pq":     bstr    // ML-DSA-65 or equivalent PQ signature
}
* measurement_hash MUST summarise the measured runtime state used for integrity checks.
* probes MUST contain a set of ProbeResult objects defined below.
* tick MUST reference a valid EpochTick as per Annex A and §V.2.
* signature_pq MUST verify under the device / runtime attestation public key.
AttestationEnvelope MUST be deterministically encoded (canonical JSON or deterministic CBOR).
V.3.2 Drift Semantics
drift_state semantics are:
* "NONE" — device/runtime is healthy; PQHD MAY proceed with signing.
* "WARNING" — device/runtime anomaly detected; PQHD MUST treat the device as invalid for custody unless an explicit local policy says otherwise.
* "CRITICAL" — severe drift or compromise detected; PQHD MUST:
    * treat valid_device = false; and
    * refuse to sign; and
    * optionally mark the wallet state as frozen until a valid attestation is obtained.
V.3.3 Probe Model
PQVL defines ProbeResult as:
ProbeResult = {
  "class":     tstr,   // "system_state" | "process_state" | "integrity_state" | "policy_state" | ...
  "name":      tstr,
  "expected":  tstr / int / bool / bstr,
  "observed":  tstr / int / bool / bstr,
  "result":    bool
}
PQHD MUST require at least:
* system_state
* process_state
* integrity_state
policy_state SHOULD be required if local policy modules are configured.
For any required probe:
* result MUST be true; otherwise the device is invalid.
V.3.4 Attestation Freshness
PQHD MUST enforce:
1. AttestationEnvelope’s tick MUST satisfy the same validation rules as in §V.2.
2. The elapsed time between the attestation tick and the current tick MUST be within a bounded window (implementation-defined, but typically ≤900 seconds).
3. If:
    * attestation tick is expired; or
    * drift_state != "NONE"; or
    * required probes are missing or failed,
4. then valid_device MUST be set to false.
Pseudocode (Informative) — Device Validation
function pqhd_valid_device(envelope, current_tick, required_probe_classes, max_age_seconds):
    if not verify_ml_dsa_signature(envelope.signature_pq, envelope):
        return false

    // tick freshness and monotonicity
    if current_tick.t < envelope.tick:
        return false

    if (current_tick.t - envelope.tick) > max_age_seconds:
        return false

    // drift state
    if envelope.drift_state == "CRITICAL":
        return false
    if envelope.drift_state == "WARNING":
        // hardened default: treat as invalid
        return false

    // required probes
    for cls in required_probe_classes:
        probe = find_probe(envelope.probes, cls)
        if probe == null or probe.result != true:
            return false

    return true

V.4 PQSF Dependencies — Consent, Transport, Policy, Ledger
PQHD reuses PQSF primitives for consent, transport binding, policy evaluation, and ledger semantics.
V.4.1 ConsentProof Behaviour
PQHD’s canonical ConsentProof structure is defined in Annex C. This annex adds the following behavioural requirements from PQSF:
1. tick_issued and tick_expiry MUST refer to EpochTicks validated as per §V.2.
2. exporter_hash MUST derive from the current transport session, as defined in §V.4.2.
3. bundle_hash MUST bind the consent to a specific bundle of wallet artefacts (e.g., PSBT, prompts, auxiliary data).
4. device_id MUST correspond to the device for which attestation is being used (from PQVL).
5. role and quorum_index MUST be consistent with the wallet’s quorum configuration.
Violation of any of these bindings MUST result in valid_consent = false.
Pseudocode (Informative) — Consent Validation
function pqhd_valid_consent(consent, current_tick, session_exporter_hash, expected_bundle_hash):
    // exporter binding
    if consent.exporter_hash != session_exporter_hash:
        return (false, "E_CONSENT_EXPORTER_MISMATCH")

    // bundle binding
    if consent.bundle_hash != expected_bundle_hash:
        return (false, "E_CONSENT_BUNDLE_MISMATCH")

    // tick window
    if consent.tick_issued > current_tick.t:
        return (false, "E_CONSENT_INVALID_WINDOW")

    if current_tick.t > consent.tick_expiry:
        return (false, "E_CONSENT_EXPIRED")

    // signature
    if not verify_ml_dsa_signature(consent.sig, canonicalise(consent)):
        return (false, "E_CONSENT_SIG_INVALID")

    return (true, null)
V.4.2 ExporterHash & Deterministic Transport
PQHD MUST:
1. Use TLSE-EMP and/or STP as transport layers when exporter-bound semantics are required.
2. Derive exporter_hash from a deterministic exporter function on the handshake transcript:
exporter_hash = HKDF-Extract-And-Expand(transcript_secret, "PQSF-EXPORTER", context_info)
1. Reject:
    * non-deterministic TLS sessions that cannot produce a stable exporter; and
    * any ConsentProof, SafePrompt, or capsule whose exporter_hash does not match the active session’s exporter.
V.4.3 Policy Enforcer Core Rules
PQHD MUST implement a deterministic policy engine consistent with PQSF:
1. Policies MUST have a canonical, serialised form that can be hashed.
2. Policy evaluation MUST be:
    * pure (no hidden side effects),
    * deterministic across implementations, and
    * bound to EpochTicks for time windows.
3. Allowlists/denylists MUST be evaluated in a stable order.
4. Custom predicates MUST be pure functions over canonical inputs.
Pseudocode (Informative) — Policy Evaluation
function pqhd_valid_policy(policy, psbt, context, current_tick):
    // 1. Canonical hash
    if hash(canonicalise(policy)) != policy.hash:
        return (false, "E_POLICY_HASH_MISMATCH")

    // 2. Time window
    if current_tick.t < policy.tick_start or current_tick.t > policy.tick_end:
        return (false, "E_POLICY_TIME_WINDOW")

    // 3. Thresholds and lists
    if not eval_thresholds(policy.thresholds, context):
        return (false, "E_POLICY_THRESHOLD")

    if is_denied(policy.denylist, psbt, context):
        return (false, "E_POLICY_DENYLIST")

    if not is_allowed(policy.allowlist, psbt, context):
        return (false, "E_POLICY_ALLOWLIST")

    // 4. Custom predicates (pure)
    for pred in policy.custom_predicates:
        if not pred(psbt, context, current_tick):
            return (false, "E_POLICY_CUSTOM")

    return (true, null)
V.4.4 Ledger and Merkle Construction
PQHD’s LedgerEntry structure and rules are defined in §16. This annex adds ecosystem-level constraints:
1. Every ledger entry MUST:
    * include a canonical EpochTick; and
    * be signed with ML-DSA-65 or equivalent PQ signature.
2. Ledger entries MUST form a hash-linked chain via prev_hash. Any mismatch between prev_hash and the current ledger root MUST set ledger.frozen = true.
3. Ticks used in ledger entries MUST be monotonic.
Pseudocode (Informative) — Ledger Append
function pqhd_append_ledger(ledger, entry, current_tick):
    if ledger.frozen:
        return error("E_LEDGER_FROZEN")

    if not validate_epoch_tick(current_tick, ledger.last_tick, ledger.profile_ref):
        ledger.frozen = true
        return error("E_TICK_INVALID")

    if entry.prev_hash != ledger.root:
        ledger.frozen = true
        return error("E_LEDGER_MISMATCH")

    // canonicalise and sign
    entry.tick = current_tick.t
    entry.hash = hash(canonicalise(entry))
    entry.sig  = sign_ml_dsa(ledger.signing_key, entry.hash)

    ledger.root      = merkle_append(ledger.root, entry.hash)
    ledger.last_tick = current_tick

    ledger.entries.append(entry)

    return ok()
V.4.5 Encrypted-Before-Transport (EBT)
PQHD MUST enforce Encrypted-Before-Transport for all sensitive artefacts:
* consent bundles,
* SafePrompt payloads and associated content,
* recovery material and capsules,
* ledger export bundles,
* KeyMail-like out-of-band messages (where applicable).
Rules:
1. Sensitive artefacts MUST be encrypted using ML-KEM-1024 (HPKE) and/or AEAD (AES-256-GCM or XChaCha20-Poly1305) before they leave the local boundary.
2. Plaintext artefacts MUST NOT be transmitted over the network, even inside TLS, except where explicitly allowed for non-sensitive metadata.

V.5 PQAI Dependencies — SafePrompt and AI Runtime
V.5.1 SafePrompt Canonical Structure
PQHD MUST treat SafePrompt as:
SafePrompt = {
  "prompt_id":     tstr,
  "content_hash":  bstr,
  "action":        tstr,
  "consent_id":    tstr,  // reference to ConsentProof
  "tick_issued":   uint,
  "expiry_tick":   uint,
  "exporter_hash": bstr,
  "signature_pq":  bstr
}
* content_hash MUST be a hash of the canonical prompt content.
* action SHOULD describe the high-risk or policy-bound operation (e.g., "WALLET_SPEND").
* consent_id MUST reference a valid ConsentProof where required.
* exporter_hash MUST match the current transport session exporter.
* tick_issued and expiry_tick MUST be bound to EpochTicks as per §V.2.
V.5.2 AI Drift and Runtime Requirements
When PQHD delegates analysis or decision support to an AI runtime:
1. The AI runtime MUST present a valid AttestationEnvelope (PQVL).
2. PQHD MUST confirm:
    * fresh EpochTick for AI runtime;
    * drift_state != "CRITICAL"; and
    * any additional AI-specific probe requirements.
3. If AI drift or misalignment is detected, PQHD MUST:
    * treat AI advice as non-authoritative; and
    * ensure no predicate is weakened by AI output.
V.5.3 Consent Binding for AI Actions
1. For any SafePrompt tied to a high-risk action (e.g. spending, recovery, policy modification):
    * SafePrompt.consent_id MUST reference a valid ConsentProof; and
    * that ConsentProof MUST satisfy §V.4.1.
2. AI advice MUST remain advisory only:
    * AI output MAY help construct PSBTs or suggest policy decisions.
    * AI output MUST NOT bypass, weaken, or circumvent:
        * valid_consent,
        * valid_policy,
        * valid_device,
        * valid_ledger,
        * valid_quorum.
Pseudocode (Informative) — SafePrompt Validation
function pqhd_valid_safe_prompt(prompt, current_tick, session_exporter_hash, consent_lookup):
    if prompt.exporter_hash != session_exporter_hash:
        return (false, "E_PROMPT_EXPORTER_MISMATCH")

    if current_tick.t < prompt.tick_issued or current_tick.t > prompt.expiry_tick:
        return (false, "E_PROMPT_EXPIRED")

    // signature
    if not verify_ml_dsa_signature(prompt.signature_pq, canonicalise(prompt)):
        return (false, "E_PROMPT_SIG_INVALID")

    // consent binding (where required)
    consent = consent_lookup(prompt.consent_id)
    if consent == null:
        return (false, "E_PROMPT_CONSENT_REQUIRED")

    (ok, err) = pqhd_valid_consent(consent, current_tick, session_exporter_hash, derive_bundle_hash_from_prompt(prompt))
    if not ok:
        return (false, "E_PROMPT_CONSENT_INVALID")

    return (true, null)

V.6 PQ Umbrella Dependencies — Unified Predicates and Profiles
V.6.1 Unified Predicate Model
PQHD’s custody predicate MUST be consistent with the PQ ecosystem’s unified predicate set.
The unified signing predicate is:
valid_for_signing =
      valid_tick
  AND valid_consent
  AND valid_policy
  AND valid_device
  AND valid_runtime
  AND valid_quorum
  AND valid_ledger
  AND valid_psbt
  AND valid_structure
Where:
* valid_tick — EpochTick validated as per §V.2.
* valid_consent — ConsentProof validated as per §V.4.1.
* valid_policy — policy engine approval as per §V.4.3.
* valid_device — device integrity as per §V.3.
* valid_runtime — runtime integrity; when PQVL is used, this MUST include valid_device.
* valid_quorum — quorum rules satisfied (threshold of devices / roles).
* valid_ledger — ledger rules satisfied as per §V.4.4.
* valid_psbt — PSBT passes structural and policy checks.
* valid_structure — all objects (ticks, consents, prompts, ledger entries) are canonical and hash-stable.
Pseudocode (Informative) — Unified Predicate
function pqhd_valid_for_signing(state, psbt, context):
    if not state.valid_tick:
        return false
    if not state.valid_consent:
        return false
    if not state.valid_policy:
        return false
    if not state.valid_device:
        return false
    if not state.valid_runtime:
        return false
    if not state.valid_quorum:
        return false
    if not state.valid_ledger:
        return false
    if not state.valid_psbt:
        return false
    if not state.valid_structure:
        return false

    return true
V.6.2 Global Profile Consistency
PQHD MUST ensure that:
1. All modules it depends on (temporal authority, device attestation, consent, AI runtime) reference a single, consistent Epoch Clock profile_ref.
2. Profile mismatches between:
    * ticks,
    * attestation envelopes,
    * consent proofs,
    * SafePrompt objects,
3. MUST cause fail-closed behaviour (valid_for_signing = false).
4. Any migration between profiles MUST:
    * be recorded in the ledger; and
    * be accompanied by explicit configuration updates so that future objects reference the new profile.

---

# **GLOSSARY (CANONICAL)**

GLOSSARY 
AAI — Attested AI AI behaviour validated under PQAI + PQVL + Epoch Clock. Used in high-assurance environments where model integrity and behavioural freshness must be cryptographically verifiable.
AICW — AI Communication Wrapper Secure envelope format used for deterministic communication between AI systems or between AI and PQ-secure components, ensuring canonical encoding and session-bound delivery.
Alignment Freshness The requirement that an AI ModelProfile and its behavioural fingerprints must be validated under a tick not older than the alignment_window. Ensures AI behaviour is recent, non-stale, and unmodified.
Alignment Tick EpochTick bound to the last successful AI alignment or fingerprint event. Used to determine expiry of behavioural fingerprints and model profiles.
Allowlist A canonical list of permitted destinations, roles, or actions within policy evaluation. Deterministic and fully encoded.
AttestationEnvelope (PQVL) Canonical PQVL object containing certifiable runtime integrity data: measurement hashes, probe results, drift_state, EpochTick, and ML-DSA-65 signature.
Baseline The expected canonical set of integrity or behavioural values used by PQVL or PQAI to detect drift.
BDC — Bluep Derived Credential Deterministically derived credential object used in PQ-based identity storage or service authentication.
Bundle Hash (PQHD) SHAKE256-256 hash of a canonicalised PSBT. All devices MUST compute the identical bundle_hash for the same PSBT.
Canonical Encoding Encoding that MUST produce byte-identical output across implementations. PQHD uses deterministic CBOR (RFC 8949 §4.2) or JCS JSON (RFC 8785).
Canonical Fingerprint (PQAI) Deterministic fingerprint of AI behaviour used for alignment and drift detection, encoded before hashing.
ChainDescriptor (PQHD Annex S) Deterministic map describing how a non-Bitcoin chain’s keys are derived (BIP44 path, standard, signature scheme, and address format).
Child Profile (Epoch Clock) A profile whose lineage references a previous profile via parent_profile_ref. Anchored on Bitcoin as part of Epoch Clock versioning.
ClockLock (PQSF) Deterministic mechanism binding operations to a fresh EpochTick, ensuring replay resistance and tick-bound authorisation.
Commitment Key (L2) Deterministic L2 key derived for Lightning channels or similar systems, isolated from Bitcoin custody operations.
ConsentProof Canonical, tick-bound, exporter-bound structure that captures user intent for sensitive operations.
Context Parameters Canonical parameters used in deterministic derivation functions. MUST NOT contain random or nondeterministic data.
DelegatedIdentity Credential granting identity-scoped permissions with tick-bound windows and explicit scopes (Annex Q).
DelegatedPaymentCredential Credential granting delegated spending rights within strict deterministic limits (Annex R).
Device Attestation (PQVL) Cryptographic attestation describing the system_state, process_state, and integrity_state of a device.
DeviceIdentity_PQVL Identity constructed from PQVL attestation keys and measurement hashes.
DeviceIdentity_Minimal Non-authoritative fallback identifier derived via PQHD-DF. MAY be used as a label, NEVER for trust.
Drift State (PQVL/PQAI) Runtime or behavioural divergence classification: NONE, WARNING, or CRITICAL.
ECDSA-P256 Classical signature scheme used only for transitional interoperability during Secure Import.
EmergencyTick Tick issued under emergency governance rules, still subject to canonical Epoch Clock constraints.
Epoch Clock Cryptographic temporal authority providing signed, monotonic EpochTicks with deterministic semantics.
EpochTick Authoritative signed time object containing Strict Unix Time, profile_ref, algorithm identifier, and ML-DSA-65 signature.
Exporter Hash Session-bound identifier derived from TLSE-EMP or STP, binding consent and transport messages to a specific session.
Fail-Closed Mandatory behaviour requiring PQHD to halt signing and freeze ledger when any predicate is invalid.
Fingerprint (PQAI) Deterministic set of behavioural probes used to validate AI model stability and detect drift.
Fingerprint Hash SHAKE256-256 hash of canonical fingerprint bytes.
Governance Key Deterministic key class used for governance-level signing (e.g., policy rotations, emergency operations).
Guardian Role capable of approving recovery events or high-risk governance actions.
HD Seed Adoption Optional procedure (Annex J) allowing PQHD to adopt an external seed before initialisation.
Intent Hash SHAKE256-256 digest of user intent, canonicalised for deterministic signing.
KeyMail Out-of-band confirmation system for high-risk actions, using canonical envelopes and ML-DSA-65 signatures.
KYCCredential (Annex P) Canonical identity attribute credential used for selective disclosure and compliance workflows.
KYCDisclosure (Annex P) Selective disclosure object revealing only authorised attributes.
Ledger Deterministic, Merkle-anchored event log of wallet operations, strictly append-only.
Ledger Freeze State where ledger stops accepting new entries due to tick, hash, or attestation mismatch.
Lineage (Profile) Chain of parent_profile_ref → child_profile_ref references used to validate Epoch Clock profile history.
L2 Namespace (Annex T) Deterministic namespace for deriving Layer-2 keys safely isolated from custody operations.
Merkle Node SHAKE256-256 hash representing an interior Merkle tree node.
Merkle Root Root hash of a deterministic Merkle tree representing current ledger state.
Minimal Identity Non-authoritative device identity derived outside PQVL. MAY label a device, MUST NOT grant trust.
Mirrors (Epoch Clock) Independent servers publishing validated ticks and profiles. Clients require ≥2 matching ticks.
ModelProfile (PQAI) Canonical representation of an AI model’s identity, configuration, and fingerprints.
Monotonicity Temporal requirement that each tick MUST be greater than or equal to the previous tick used.
Multisig Predicate Deterministic evaluation of thresholds, roles, tick validation, device integrity, and ledger continuity before signing.
Parallel Device Attestation Simultaneous evaluation of PQVL attestation from multiple devices in multisig workflows.
Parent Profile Ancestor profile in Epoch Clock lineage.
Password Vault (Annex N) Encrypted deterministic vault for credential storage.
Payload Hash Canonicalised SHAKE256-256 hash used inside ledger entries or policy/consent structures.
Policy Enforcer Deterministic rule engine that evaluates wallet policy controls (thresholds, limits, allowlists, time windows).
Policy Hash SHAKE256-256 digest of canonical policy representation.
ProbeResult (PQVL) Runtime integrity measurement including expected vs actual value and signature.
Profile Ref Canonical string referencing the Epoch Clock profile inscription.
PSBT (Canonical) Deterministically normalised PSBT used to compute bundle_hash and ensure cross-device consistency.
PQAI Post-Quantum Artificial Intelligence framework defining deterministic AI integrity, fingerprints, and drift rules.
PQHD Post-Quantum Hierarchical Deterministic Wallet specification.
PQKEM Post-quantum KEM (e.g., ML-KEM) used for encrypted payloads.
PQLedger The deterministic Merkle ledger used in PQSF/PQHD for recording state.
PQSF Post-Quantum Security Framework defining transports, consent, policy, encoding, and predicate models.
PQVL Post-Quantum Verification Layer providing runtime integrity enforcement.
Recovery Capsule Deterministic envelope containing encrypted recovery material with threshold approval and tick-bound validity.
Replay Window Temporal bound (≤900 seconds) for legitimate tick reuse.
Role Binding Deterministic mapping between device identity and multisig role.
Runtime State State verified by PQVL attestation, including OS, processes, integrity, and policy signals.
SafePrompt Canonical natural-language prompt object used for high-risk confirmations, requiring tick freshness and consent binding.
Secure Import Protocol for migrating classical ECDSA keys into PQHD using dual Proof-of-Possession and tick-bound validation.
Selective Disclosure Mechanism for revealing a subset of stored identity attributes.
Session Exporter Session-specific PRF output used to bind consent and transport messages.
Signature (PQ) ML-DSA-65 post-quantum signature used for all PQHD internal structures.
Sovereign Transport Protocol (STP) DNS-independent deterministic transport protocol with exporter binding and canonical messaging.
Stealth Mode Privacy-focused mode disabling DNS and TLSE-EMP, using STP-only and strict tick reuse rules.
Tick Cache Local storage of the most recent verified tick, valid for ≤900 seconds.
Tick Expiry Condition where a tick exceeds the allowed freshness window.
Tick Freshness Time elapsed since EpochTick issuance, MUST be ≤900 seconds for signing.
Tick Monotonicity Requirement that each new tick MUST be ≥ the last validated tick.
Tick Reuse Allowed only within the defined 900-second window.
TLSE-EMP Deterministic TLS profile with exporter binding used for PQSF transport.
Travel Rule Data Optional compliance metadata for regulated payment flows (not stored in PQHD ledger).
Universal Secret (Annex M) Deterministic non-custodial secret derived from PQHD root using cSHAKE256.
Vault Key DEK/KEK keys used for encrypted credential vaults.
Verifier Any system validating cryptographic artefacts and predicates.
Warning Drift Non-fatal drift state indicating possible upcoming inconsistencies; CRITICAL drift immediately invalidates device integrity.


# **ACKNOWLEDGEMENTS (INFORMATIVE)**

This specification acknowledges foundational contributions by:

**Peter Shor** — whose algorithm demonstrated the need for post-quantum-safe signatures.
**Pieter Wuille** — for HD wallet design and deterministic Bitcoin key management.
**Greg Maxwell** — for multisig, PSBT semantics, and Bitcoin script foundations.
**Ralph Merkle** — for Merkle tree constructions used for deterministic ledger state.
**Guido Bertoni, Joan Daemen, Michaël Peeters, Gilles Van Assche** — for Keccak, the basis of SHAKE-family hashing used throughout PQHD.

If you find this work useful and want to support it, you can do so here:
bc1q380874ggwuavgldrsyqzzn9zmvvldkrs8aygkw
