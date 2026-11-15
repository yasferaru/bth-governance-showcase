# 3. Control families A to F

> **BTH is a research prototype, not a medical device.**

The controls come from a working document compiled on 12 November 2026, "Software safety controls for the BTH
LLM". Each control there carries an evidence tag, kept here:

- **V**: source opened and the figures re-read.
- **S**: source reached only as a secondary summary; check the original before relying on it.
- **D**: own design requirement, derived from code review or stress tests. Not a literature claim.

Literature findings are described in words; no literature figure is presented as a BTH result. Test counts are from the
governance test suite (373 tests, all passing on the last full run, 12 November 2026).

"Built" means implemented in the governance package and exercised in simulation. Nothing here has run with a
real model, real hardware or a patient.

## A. Constraining what the model can do

| ID | Control | Why | Evidence | Built | Tested by |
|---|---|---|---|---|---|
| A1 | Typed, schema-validated action space: one action from a closed set, coded reasons, no prose | Of the published agent-security design patterns, action selection gives the strongest guarantee, because the agent can only do what the fixed actions allow (*Design Patterns for Securing LLM Agents*) | V | Yes. Five actions: hold, set stiffness, recommend pause, request clinician, abstain. Extra actions exist but are disabled | Schema and property tests (27) |
| A2 | Plan-then-execute: commit the session plan before reading untrusted data | Injected data must not be able to add actions | V (same paper) | Yes. The plan is hashed at session start | 7 tests |
| A3 | Dual model: the privileged proposer never sees untrusted text; a quarantined model reads it without tools | The quarantined model can still be manipulated, so its output is handled symbolically | V (same paper) | Yes | Adversarial injection tests (with A4: 48) |
| A4 | Context minimisation: process each untrusted item in isolation; drop raw text after use | Limits what an injected item can influence | V (same paper) | Yes | See A3 |
| A5 | Deterministic pre-action authorisation returning ALLOW, DENY or ESCALATE, with a logged record | A synchronous, declarative check before execution. In one agent testbed a permissive policy let social engineering succeed often, a restrictive one never (*Before the Tool Call*, Open Agent Passport); that result does not generalise beyond its testbed | V | Yes | Gate and audit tests (with A6: 67) |
| A6 | Fail closed: unparseable, unknown, out of range, wrong version or any exception means DENY | "Unknown" must never mean "allowed" | D | Yes. Property test: any exception in the gate yields DENY | See A5 |
| A7 | Hard limits in firmware, independent of backend and model | Software can lower limits, never raise them | D | Specified only, with a reference model in Python. No firmware exists. Ceilings are working values, not validated | 7 tests on the reference model |
| A8 | Rate limits on every output, in the gate and again in firmware | A stress test showed the documented system could move stiffness across its whole range in one tick | D | Yes, in the gate, the fixed protocol and trusted editing | 2 tests |
| A9 | Autonomy tiers (observe, suggest, act within envelope); clinician-set envelopes that expire and are logged | Keeps authority with the overseeing clinician; compare the guardrailed g-AMIE design | D (g-AMIE: V) | Yes. Default tier is observe; override or expiry returns to observe | Step-down tests |

## B. Independent oversight of the model

| ID | Control | Why | Evidence | Built | Tested by |
|---|---|---|---|---|---|
| B1 | Trusted monitoring: a simpler trusted component scores each proposed action | In the AI Control code-backdoor setting, trusted monitoring improved safety over no monitoring at a small usefulness cost (Greenblatt et al., *AI Control*, arXiv:2312.06942) | V | Yes, rule-based | Monitor tests (with B2, B3, B6: 14) |
| B2 | Defer to trusted above a suspicion threshold; report the safety/usefulness curve | The defer rate is a choice on a trade-off curve, and should be shown as one | V (same paper) | Yes. Fallback is the fixed protocol. The curve is reported from simulation ([06_results.md](06_results.md)) | See B1 |
| B3 | Trusted editing: clamp the action to the nearest safe value | Keeps more usefulness than deferring outright | V (same paper) | Yes. Untrusted monitoring is deliberately skipped | See B1 |
| B4 | Safety case structured as "control": safeguards hold even if the model misbehaves | Inability and trustworthiness arguments are not available for this model (Clymer et al., safety cases) | V | Documented | Documentation lint |
| B5 | Layered input and output classifiers | Useful, never the only layer | V (Constitutional Classifiers) | Interface stub only | 2 tests |
| B6 | Do not rely on stated reasoning as a monitor; treat pre-deployment tests as weaker evidence | Reasoning text is often not controllable or faithful (Chen et al.); models can recognise evaluations (secondary summary) | V / S | Yes. The monitor's input type excludes reasoning text; post-deployment rolling statistics exist as hooks | Architecture test on monitor inputs |
| B7 | Frontier-lab threshold frameworks: numeric triggers, periodic risk report, external review | Soft commitments are the lesson (GovAI analysis of lab frameworks) | V | Yes. A periodic risk report is generated; the external reviewer line is blank | 3 tests (with B6) |

## C. Honest uncertainty and sensor trust

| ID | Control | Why | Evidence | Built | Tested by |
|---|---|---|---|---|---|
| C1 | Abstention is a first-class action; coverage is reported | Explicit abstention options increase safe abstention (conformal and MedAbstain work) | V | Yes | Abstention tests |
| C2 | Calibration monitoring; step down when calibration drifts | Online recalibration work shows drift is detectable and correctable (Feng et al.) | V | Yes. Rolling slope and intercept; drift caps the tier at suggest | Uncertainty tests |
| C3 | Cold start: "no baseline yet" state | A stress test showed the documented normaliser returned zero for its first samples, hiding a real step | D | Yes. Only the fixed protocol until a baseline exists | Sensor-trust tests |
| C4 | Stale-data check: reject old frames and escalate | The documented backend never checked frame age | D | Yes, here. The backend still needs the change | Sensor-trust tests |
| C5 | Quality flag on every frame; flatline and dropout detection; clipped z-scores | A stress test showed a tiny blip after a flat signal driving the state index to its maximum | D | Yes, with a regression test for that case | Sensor-trust tests |
| C6 | Confounds: unknown arrhythmia or beta-blocker status must not mean "safe" | The documented rubric treated missing demographics as "no arrhythmia"; an AF-like rhythm can mask true arousal | D | Yes. Unknown status is quarantined until a clinician confirms | Sensor-trust tests |
| C7 | The watchdog must catch slow overload, not only abrupt overload | The controls document reported a slow ramp evading a short-window watchdog | D | Yes: raw-signal bounds plus a rate-of-rise rule. The original evasion finding was **not reproduced** in our simulator, so the rule is defence in depth | Sensor-trust tests (C1 to C7: 19) |

## D. Sycophancy, hallucination and clinician input

| ID | Control | Why | Evidence | Built | Tested by |
|---|---|---|---|---|---|
| D1 | Clinician input is a constraint on the envelope, never evidence that an action is safe; premise check before acting | Models complied with illogical medical requests in a 2025 npj Digital Medicine study; prompting and fine-tuning reduced but did not remove this | V | Yes. A validator refuses illogical, unsafe or widening instructions; the model must state a premise code that deterministic predicates check | Adversarial clinician-input and premise tests (with D3: 23) |
| D2 | Keep personalisation memory minimal; test agreement drift | Interaction context and stored user profiles raised model agreeableness (MIT and Penn State study) | V | Yes. Role-separated memory; a drift test with a sycophantic positive control | 1 test |
| D3 | Adversarial-hallucination red team: inject fabricated details | Prompt mitigation reduced but did not remove adversarial hallucination in physician-validated vignettes (Communications Medicine 2025) | V | Yes, as planted errors | See D1 |
| D4 | Physician-written rubric evaluation | HealthBench shows rubric evaluation at scale; BTH needs its own rubric | V | Template and scorer only. **The clinician must write the content** | 1 test |

## E. Lifecycle, drift and change control

| ID | Control | Why | Evidence | Built | Tested by |
|---|---|---|---|---|---|
| E1 | Pin the model version; no silent upgrades | Model behaviour can change markedly between versions (Chen, Zaharia and Zou) | V | Yes. Version from provider metadata; the model's echo is a cross-check | Change-control tests (with E2, E3: 10) |
| E2 | Predetermined change control plan: what may change, how it is validated, who signs off | FDA final guidance on PCCPs, December 2024 | V | Yes, as a document and an enforcement check. Allowed change types are pending the clinician and a regulatory owner | See E1 |
| E3 | Shadow re-validation before any change goes live; log every suggestion | Silently deployed models performed differently from retrospective estimates (DEPLOYR) | D | Yes | See E1 |
| E4 | Data-poisoning defences for any fine-tuning | Small amounts of poisoned data can produce harmful models that pass benchmarks | S | Documentation only. No training happens here | None |
| E5 | Authenticated, logged changes; tamper-evident log | The documented backend accepted unauthenticated model changes | D | Hash-chained log on our side. **The backend change is still required** | Audit tests |
| E6 | Fitting hygiene: held-out validation and explicit promotion | The documented fitter registered models on in-sample results | D | **Backend change required**; not built here | None |
| E7 | Choose the default model by safety behaviour | The documented default was not chosen for safety | D | **Backend change required** | None |

## F. Human factors and governance

| ID | Control | Why | Evidence | Built | Tested by |
|---|---|---|---|---|---|
| F1 | Human oversight by design: understand limits, stay aware of automation bias, override, stop | EU AI Act Art. 14(4) | V | Ports only. **The frontend has no override or stop yet** | None |
| F2 | Use correct regulatory dates | Dates changed with the 2026 Digital Omnibus | V (secondary legal summary) | Documented; to be verified against the Official Journal | Documentation lint |
| F3 | Delegation-creep and deskilling guards | Exposure to AI can erode the clinician's own skill | D | Yes. Expiring envelopes; the meter reports renewals and acceptance | Meter tests (12, with isolation tests) |
| F4 | Reporting standards; pre-specified endpoints | DECIDE-AI and TRIPOD-LLM | V | Yes. Endpoints: unsafe-action pass-through, abstention coverage, time to step-down, override rate | 1 test |

## Proposed, not built

- **Clinician training:** a mandatory, safety-specific curriculum. Not implemented, not tested.
- **External reviewer** for the periodic risk report: not appointed.
