# 8. What is not validated

> **BTH is a research prototype, not a medical device.**

Everything below is stated plainly so that nobody mistakes a working value for a validated one. **No value is
shown here**: the point is that each one is unvalidated, not what it currently is.

## 8.1 How values are tracked

Every numeric configuration value in the governance package is registered with its source and a status:

| Status | Meaning | Count |
|---|---|---|
| DOC | Taken from a design document | 12 |
| PLACEHOLDER | Chosen by the author because no source gives a value. **Blocks live use** | 64 |
| NEW_UNVALIDATED | Derived by the author from stated assumptions, with a named validation test still to run | 13 |
| TUNED_FOR_TESTS | Chosen so that a test passes. Always reported next to the documented alternative | 1 |
| SIMULATION_ONLY | Assumption used only for a reporting figure | 1 |

A lint test fails the build if a NEW_UNVALIDATED value appears without its label. The readiness gate refuses live
start while any PLACEHOLDER remains. Even the DOC values are design-document values, not clinically validated
ones.

## 8.2 Unvalidated values by area

The full per-parameter register is kept in the private repository. Here it is summarised by area, so that the
scale of what is unvalidated is visible without exposing the structure of the pipeline.

| Area | Values | Status | Who must supply or validate them |
|---|---|---|---|
| Step-down triggers (drift, overload, patient-report escalation, deferral) | 21 | 20 PLACEHOLDER, 1 NEW_UNVALIDATED | Clinical lead, ML owner |
| Signal quality (staleness, dropout, flatline, saturation, jumps, drift) | 25 | PLACEHOLDER | Signal-processing, hardware and backend owners |
| State-index features | 7 | PLACEHOLDER | State-index owner, clinical lead |
| Fixed protocol | 6 | PLACEHOLDER | Clinical lead |
| Monitor rule weights | 2 | PLACEHOLDER (chosen knowing the planted errors) | Clinical lead, ML owner |
| Premise check | 1 | PLACEHOLDER | Clinical lead |
| Firmware ceilings (torque, current, speed, stiffness rate) | 8 | NEW_UNVALIDATED | Hardware and firmware team (motor datasheet, IEC 80601-2-78, measured joint speeds) |
| Exercise prescription | 4 | NEW_UNVALIDATED | Clinical lead |
| Data retention | 2 | PLACEHOLDER | Data-protection professional |
| Sampling rate | 1 | PLACEHOLDER (disagrees with the design document) | Hardware and backend owners |
| Experiment setting | 1 | TUNED_FOR_TESTS | Reported next to the documented alternative |
| Delegation-meter reporting assumption | 1 | SIMULATION_ONLY | None (reporting figure only) |

The roles in the last column are roles to fill, not people already on the project.

## 8.3 Prerequisites before any human use

The system refuses live start, and cannot reach the suggest or act tier, while any of these is pending. All seven
are pending.

| Prerequisite | Owner |
|---|---|
| Signing-key custody and named signers for envelopes, instructions, restores and approvals | Clinical lead and a security owner |
| Data policy: retention periods, lawful basis, consent wording; a signed data-protection impact assessment | Data-protection professional |
| Clinical sign-off of the fixed protocol, all thresholds, the session-plan phases and the deviation of the state index from the backend's model | Clinical lead |
| Rubric content and weights; allowed change types and acceptance criteria in the change control plan | Clinical lead and a regulatory owner |
| Ethics approval and a study protocol | Ethics committee |
| Firmware ceilings (torque, speed, stiffness) from the motor datasheet, and the firmware rate limit | Hardware and firmware team |
| Any randomisation of actuation, including micro-randomisation (first choice: none, shadow observation only) | Ethics committee, clinical lead and a risk owner under IEC 80601-2-78 |

## 8.4 Open questions

1. **Severe overload is not defined** in the design documents. Until it is, the gate's override fails closed to
   the documented overload boundary, which pre-empts most of the monitor's overload rules.
2. **Sampling rate.** The software design document recommends a higher skin-conductance rate for real-time
   feedback than the backend currently streams. The governance layer assumes the backend's rate.
3. **Gravity support.** At the documented stiffness range, the impedance term is an order of magnitude below the
   shoulder's gravity torque. Whether BTH adds gravity compensation, or supports only part of the weight, is not
   in the documents.
4. **Torque and speed ceilings.** The working values clip a large share of normal simulated movement
   ([05](05_simulator_and_credibility.md)). Real values need the motor datasheet and measured joint speeds.
5. **Flagged patients** (confirmed atrial fibrillation or beta-blockers): the default is the fixed protocol with
   the state index in shadow. Five further variants exist only as hooks. Moving a flagged patient to
   index-driven modulation needs a pre-registered check that is not built.
6. **The state index at rest, the watchdog input and the meaning of "low stiffness"** are decisions for the
   design-document owner ([05](05_simulator_and_credibility.md)).
7. **The prototype uses symmetric signatures.** Production needs asymmetric ones, with the private key only on
   the clinician side.
8. **The audit chain's head is not anchored externally** yet.
9. **Backend changes still required:** frame-age checks, treating
   missing demographics as unknown, held-out validation before registering a model, and a safe default model.
10. **The frontend has no override or stop** yet.

From the delegation research notes (research questions, not blockers):

11. Which delegation metrics predict outcomes, and do they differ by clinical situation?
12. Does safety-specific training beat generic AI-literacy training at catching planted errors?
13. How small can a randomised safe set be while still giving estimable effects at BTH's likely sample sizes?
14. What is the legal position when a suggestion is accepted without verification?
15. How should the patient-facing channel be limited, and does patient feedback change clinician delegation?

## 8.5 Simulator decisions waiting on the owner

The simulator's own to-do list holds 14 decisions about the ground-truth documents, 7 about working values and
flagged patients, 5 about data and permissions, and 6 kinds of real recordings that would settle what
simulation cannot (healthy-volunteer exosuit sessions, the same in post-stroke patients, flagged patients,
manually edited heartbeat annotations, measured joint speeds and torques, and a pilot comparing modulation
directions inside a clinician-approved envelope).
