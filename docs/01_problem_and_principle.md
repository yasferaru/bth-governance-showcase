# 1. Problem and principle

> **BTH is a research prototype, not a medical device.**

## Why a language model in a closed-loop wearable at all

Patients differ more than averages show, and rehabilitation is long. The design goal for BTH includes a
per-patient assistant that watches every session, keeps track of the individual, adapts suggestions over time
and reports to both patient and clinician. A language model is a plausible engine for that: it can read
heterogeneous context, summarise, and explain.

It is also the wrong thing to trust with a motor. A language model can be wrong, sycophantic, inconsistent
between versions, and steered by the text it reads. In a wearable that applies torque to a weak arm, those
failure modes are physical.

Refusing to use such models does not remove the need they address. The question this work asks is narrower:
**what has to be true of the system around the model so that its failures cannot reach the patient, and the
clinician stays in charge?**

## The principle: the model proposes, a deterministic layer disposes

- The model has **no path to actuation** except one: a typed proposal from a closed set, checked by a
  deterministic gate.
- **Everything computable stays deterministic**: features, quality checks, limits, step-down rules, logging.
  The model is used only where judgement over context is the point, and even there it only proposes.
- **The firmware is the last line of defence**, with ceilings and a stop that no software can raise or remove.
- **The model never certifies its own output.** Neither does a clinician's confirmation: a confirmation can
  narrow what is allowed, but it is not evidence that an action is safe.

This follows the "action-selector" and "plan-then-execute" design patterns for securing LLM agents, the
pre-action authorisation pattern, and the AI Control framing of a safety case: safeguards that hold even if
the model misbehaves (sources in [03_control_families.md](03_control_families.md)).

## Delegation creep

Every assistant that is usually right invites people to stop checking it. Evidence from medicine points the
same way: in a randomised study of physicians who had completed AI-literacy training, deliberately wrong AI
recommendations still lowered diagnostic-reasoning scores (medRxiv preprint, NCT06963957), and endoscopists'
adenoma detection without AI fell after AI was introduced (Lancet Gastroenterol Hepatol 2025). Training in
general AI literacy did not prevent automation bias in that study.

So the system is built **against** delegation creep:

- An envelope the clinician sets always **expires**, and can always be stopped.
- The default autonomy tier is **observe**. Suggest and act tiers must be earned.
- A **delegation meter** reports, per clinical situation, how often suggestions are accepted unchanged, how often
  the clinician verified before accepting, and how many planted errors were caught. It reports; people decide.

## Accountability stays with the clinician

The clinician owns every decision. The design makes that ownership real rather than nominal:

- what the clinician **saw** is logged, not only what they clicked;
- suggestions are logged **even when ignored or unseen**, so a decision can be reconstructed later;
- clinician input can **narrow** the envelope but never widen it; widening requires a newly signed envelope;
- the patient-facing channel is separate and constrained, so that a shared "second brain" does not blur
  who decided what.

Fault attribution for a single case is honestly reported as "undetermined" when the counterfactual cannot be
observed. The system never asserts a single-case counterfactual, and it never uses the model to judge its own
suggestions.

## Limits of this framing

- This is a design and a simulation. It shows the mechanisms can work as specified; it does not show that they
  are sufficient.
- Who is legally at fault when a suggestion is accepted without verification is an open legal question.
- Whether task-specific AI combined with safety-specific training changes the safe level of delegation is an
  untested hypothesis, not a finding.
