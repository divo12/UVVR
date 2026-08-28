---
name: uvvr
description: Interactively identify an AI system, clarify what its user wants evaluated, and create a readable eval contract package for partly unverifiable work. Use when a user wants to turn fuzzy AI behavior into criteria, verifiers, stress cases, and coding-agent instructions. Do not use for ordinary unit tests with known assertions, directly grading one answer, or general RL/RLVR explanation without an eval-design request.
---

# UVVR

Design and materialize the evaluation; do not execute it unless the user separately asks.

Do not pretend subjective quality became objective. Convert the original outcome into the narrowest useful evaluable claim, state what the conversion loses, and preserve what remains unverifiable.

## Phase 1 — Identify and confirm the system

Inspect user-provided repositories, documentation, examples, prompts, traces, and existing evals before asking about facts that can be discovered directly. Do not modify anything yet.

Present a short **system card**:

- system and purpose;
- trigger or input;
- model/agent and available actions;
- environment and important constraints;
- produced output or trajectory;
- user or downstream decision the output serves;
- evidence currently observable to an evaluator;
- observed facts versus assumptions.

Ask one confirmation question: **“Is this the system you want to evaluate? What is wrong or missing?”** Stop and wait. If corrected, update the card and confirm it again before proceeding.

## Phase 2 — Light requirements drill

Ask one question per turn. Ask at most six questions total, including one follow-up that probes an example, hidden assumption, or boundary from an earlier answer. Stop early once the contract can be designed.

Choose the next unresolved question in this order:

1. What decision will this eval support: release, regression, model selection, monitoring, or something else?
2. Give one concrete successful output and one unacceptable output. What makes the difference consequential?
3. What evidence can the evaluator actually inspect, and what behavior remains off-screen?
4. Which failures must block acceptance, and which quality differences should only score or escalate?
5. Which alternative outputs should count as valid even when they differ from a reference?
6. Where should the eval package live, and how will the coding agent or team use it?

Do not use ambiguity percentages, challenge modes, questionnaires, or exhaustive discovery. Do not ask a question already answered by the system card or prior replies.

Before writing files, summarize the intended eval in five lines: evaluated object, decision, success, blockers, and evidence. Ask **“Should I build the eval package from this?”** Stop and wait; incorporate any correction before proceeding.

## Phase 3 — Apply the UVVR framework

### Transform the claim

Record:

```text
original_claim: the outcome the user actually cares about
evaluable_claim: the narrower claim the proposed evidence can support
capability_lost: what the transformation no longer measures
transfer_check: how improvement will be checked on the original task
```

Useful transformations include a known answer, reference-conditioned replication, executable end state, temporal snapshots, round trip, metamorphic relation, differential comparison, simulation, provenance check, human review, or delayed outcome. A perfect proxy label does not prove original-task quality.

### Assign verification strength

| Level | Signal |
|---|---|
| V4 | Formal, deterministic, or executable check against a stated contract |
| V3 | Grounded environment state, transition, transaction, or measured outcome |
| V2 | Independent reference, evidence, provenance, or known latent-variable anchor |
| V1 | Structured semantic proxy: rubric, learned verifier, or model judge |

V4–V1 are usable verifier classes. Put preferences, contested values, and delayed outcomes with no current verifier in the **unverifiable remainder (V0)** instead of inventing a score.

Keep level separate from role:

- `block`: reliable failure makes the output ineligible;
- `score`: supplies partial credit or diagnosis;
- `audit`: monitors without deciding this run;
- `escalate`: sends uncertainty to a human or stronger process.

### Build one criterion map

Use one row per atomic, decision-relevant property:

| Criterion | Level | Evidence | Verifier / what pass establishes | Role | Blind spot |
|---|---|---|---|---|---|

Reject “good,” “safe,” or “high quality” until decomposed into observable properties. Use the strongest feasible evidence and deterministic checks before semantic proxies. Never call an LLM judge, rubric, vote, or reward model ground truth.

Keep hard gates separate from partial-credit signals. Missing evidence or verifier failure produces `unknown` and usually `escalate`, not automatic evaluated-system failure. State any explicitly chosen fail-closed policy.

### Stress the verifier

Create four concrete cases from the user's domain:

1. known-good output that must pass;
2. plausible bad output a superficial verifier might accept;
3. valid alternative unlike the reference;
4. likely reward hack or evidence-tampering attempt.

For each case state the artifact/input, expected criterion results, and why. Add another case only for a named risk. Keep hidden holdouts, graders, reward code, and labels unreadable and unwritable by the evaluated policy.

## Phase 4 — Write the eval package

Default to `evals/<system-slug>/` inside the user's project. If that path already contains user-authored files, preserve them and ask before overwriting; otherwise choose a new slug.

Create only these four Markdown files:

```text
evals/<system-slug>/
├── README.md
├── contract.md
├── cases.md
└── runbook.md
```

### `README.md`

Include the confirmed system card, eval purpose, package status, file map, and this ready-to-use coding-agent handoff adapted to the project:

```text
Read every file in this eval folder. Use contract.md as the decision authority,
cases.md as verifier self-tests, and runbook.md as the evidence and execution guide.
Do not change the contract or expected case outcomes while grading. Do not infer
missing evidence. Return the result shape defined in runbook.md with artifact paths.
```

### `contract.md`

Include the original-to-evaluable claim transformation, the criterion map, block/score/escalation policy, what `pass`, `fail`, and `unknown` mean, trust boundaries, and the owned unverifiable remainder.

### `cases.md`

Include the four concrete stress cases. They must be specific enough for a coding agent to turn into fixtures or manual checks; do not leave generic placeholders. Add a `fixtures/` directory only when runnable non-Markdown artifacts are actually required.

### `runbook.md`

Specify evidence locations, verifier order, concrete commands when already available, expected artifacts, stopping conditions, and recovery for missing evidence or verifier errors. End with this result shape:

```yaml
status: success | warning | error
decision: accept | reject | escalate | no_decision
summary: one-line outcome
criteria: {}
evidence: []
next_actions: []
artifacts: []
```

Do not create verifier scripts, dependencies, manifests, result directories, or hidden datasets until the user asks or the confirmed system already requires them.

## Completion

Finish only when the system card was confirmed, the light drill resolved decision-relevant ambiguity, all four files exist and agree, each criterion has evidence and a blind spot, the four cases are concrete, and the unverifiable remainder has an owner.

Report the created file paths and the exact next handoff. If material evidence is missing, still write the useful package with `unknown` and its owner rather than fabricating certainty.
