# 2. Threat model

> **BTH is a research prototype, not a medical device.**

## Assets

1. The patient's physical safety: torque, stiffness, speed and session load.
2. The integrity of the clinician's authority: the envelope they set.
3. The audit trail: the ability to reconstruct who and what decided.

## Trust boundaries

| Component | Trusted? | Why |
|---|---|---|
| Firmware ceilings and the human stop | Trusted, outermost | Independent of all software |
| Gate, monitor, fixed protocol, step-down ladder, features, quality checks | Trusted | Deterministic, reviewed, tested |
| Clinician (signed envelope, signed instructions) | Trusted to **narrow** | An instruction cannot widen; widening needs a new signed envelope |
| Proposer model | **Untrusted** | Can be wrong, sycophantic, drifted or steered |
| Quarantined parser model | **Untrusted** | Reads attacker-controllable text; its output is limited to three typed fields |
| Free text (notes, patient messages) | **Untrusted** | Injection channel |
| Sensor data | Untrusted for quality | Flatline, dropout, saturation, noise, drift and staleness are all possible |

## Threats, controls and residual risk

Control IDs refer to [03_control_families.md](03_control_families.md).

| Threat | Controls | Residual risk |
|---|---|---|
| The model proposes an unsafe value | Typed schema (A1), premise check (D1), envelope intersected with firmware ceilings, rate limits (A8), fail-closed gate (A5, A6), override in severe overload, independent escalation on a patient report | Errors inside the envelope that the monitor's rules do not describe. The severe-overload threshold is a placeholder |
| Prompt injection through notes | Dual-model split (A3): raw text never reaches the proposer; the parser's output is validated to three fields; note-derived claims can only **raise** caution; a deterministic raising-only scan of the raw note | A note can raise caution spuriously, and a fabricated *raising* claim is accepted. Lowering caution needs structured input |
| Model output forging authority (envelope fields, features, version) | No prose field, extra fields forbidden, version pin (E1), envelope signature | The real adapter must take the version from provider metadata; the prototype also checks the model's echo |
| Hallucinated premises | Premise codes checked against deterministic predicates (D1, D3) | Premises the predicates cannot check pass through the quarantined fields |
| Manipulating the plan through data | Plan committed and hashed before any untrusted data is read (A2) | None found in testing |
| Silent model swap or upgrade | Pin against provider metadata, registry and envelope; mismatch or missing metadata is a denial (E1) | The pin is only as good as the provenance record |
| Clinician sycophancy or an illogical instruction | Deterministic validator; narrowing-only overlay; confirmation never treated as a certificate (D1) | Clinician error *within* the envelope is not caught. This is the automation-bias problem the meter and training address |
| Envelope tampering or expiry | Signature; expiry steps the ladder down (A9, F3) | The prototype uses a symmetric signature, so whoever can verify can sign. Production needs asymmetric signatures with the private key only on the clinician side |
| Audit-log tampering | Hash chain over every entry, verifiable; redaction keeps the chain valid (E5) | A chain proves integrity relative to its head. The head must be anchored externally to detect truncation |
| Bad sensor data driving decisions | Quality flags hold model actions and escalate; no baseline or unconfirmed confounds means fixed protocol only (C3 to C6) | Failure modes not in the flag list |
| A software bug bypassing the gate | The actuation port accepts only gate-issued approved commands; firmware ceilings (A7) | In-process code cannot fully prevent a determined bypass. Firmware is the real backstop |
| Delegation creep and deskilling | Expiring envelopes; meter reports acceptance, verification and catch rates per situation (F3) | The meter only reports. Acting on it is a human decision |

## Out of scope

Network attacks on the backend (it runs locally with token authentication; a networked deployment is not designed yet), supply-chain attacks on dependencies
(versions are pinned, hashes are not), physical attacks, and poisoning of any fine-tuning data (control E4).
These are listed so that nobody reads the table above as complete.
