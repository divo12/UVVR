---
name: uvvr
description: Interactively identify an AI system, clarify what its user wants evaluated, and create a self-contained executable eval package for partly unverifiable work. Use when a user wants to turn fuzzy AI behavior into criteria, deterministic verifiers, judge-based checks, examples, and coding-agent instructions. Do not use for ordinary unit tests with known assertions, directly grading one answer, or general RL/RLVR explanation without an eval-design request.
---

# UVVR

Design and materialize the evaluation; do not execute it unless the user separately asks.

Do not pretend subjective quality became objective. Convert the original outcome into the narrowest useful evaluable claim, state what the conversion loses, and preserve what remains unverifiable.

## Phase 1 — Identify and confirm the system

Inspect user-provided repositories, documentation, examples, prompts, traces, and existing evals before asking about facts that can be discovered directly. Do not modify anything yet.

Present a short **system card**:

- system purpose and a plain-language input → actions → output description;
- trigger or input;
- model/agent, available actions, environment, and important constraints;
- current reward or good/bad judgment, who supplies it, and why it is not directly verifiable;
- user or downstream decision the output serves;
- evidence currently observable to an evaluator;
- observed facts versus assumptions.

Describe the system and its current unverifiable reward in a short paragraph, then ask: **“Is this the system you want to evaluate, and is that how success is currently judged? What is wrong or missing?”** Stop and wait. If corrected, update the card and confirm it again before proceeding.

## Phase 2 — Light requirements drill

Ask one question per turn and at most three questions total. Combine related uncertainties and stop early once the contract can be designed. A follow-up counts toward the limit and is justified only when proceeding would produce the wrong acceptance decision.

Choose the next unresolved question in this order:

1. What decision should the eval make, and what is one clear pass/fail example?
2. Which single failure must block, what may warn or escalate, and which different outputs remain valid?
3. Which evidence is authoritative when specifications and observed behavior disagree? Ask only if repository evidence cannot settle it.

Do not ask where to store the package; use the Phase 4 default. Do not use ambiguity percentages, challenge modes, questionnaires, or exhaustive discovery. Do not ask about facts discoverable from the system, repository, or prior replies; state a reversible assumption instead.

After the last necessary answer, summarize the intended eval in five lines: evaluated object, decision, success, blockers, and evidence. State assumptions and proceed to Phase 3 without another confirmation round; the user may correct them at any time.

## Phase 3 — Apply the UVVR framework

### Transform the claim

Record:

```text
original_claim: the outcome the user actually cares about
current_reward: what currently labels or signals success
why_unverifiable: why that reward cannot directly prove the original claim
evaluable_claim: the narrower claim the proposed evidence can support
capability_lost: what the transformation no longer measures
transfer_check: how improvement will be checked on the original task
```

Useful transformations include a known answer, reference-conditioned replication, executable end state, temporal snapshots, round trip, metamorphic relation, differential comparison, simulation, provenance check, human review, or delayed outcome. A perfect proxy label does not prove original-task quality.

For subjective rewards, first render or otherwise expose an inspectable outcome and keep executable eligibility checks separate from quality. When absolute rubric scores are compressed or arbitrary, prefer relative comparison against several human-curated references. Record who selected the pool, its inclusion rule and coverage, the sampled references, and the comparison judge. This remains a V1 judgment grounded by V2 anchors—not ground truth.

Before combining reward components, remove signals that duplicate one another or have already saturated. Keep reward changes separate from prompt, tool, retry, and action-space changes so improvement is attributed correctly.

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

Assign exactly one V-level and one role to each row. Use the level of the deciding signal; put supporting anchors in `Evidence` instead of writing composite levels such as `V3 + V4`. Roles are only `block`, `score`, `audit`, or `escalate`; warning is a result status, not a role.

Reject “good,” “safe,” or “high quality” until decomposed into observable properties. Use the strongest feasible evidence and deterministic checks before semantic proxies. Never call an LLM judge, rubric, vote, or reward model ground truth.

Keep hard gates separate from partial-credit signals. `reject` requires sufficient evidence that an explicit `block` criterion failed. Wrong-target runs, missing evidence, malformed input, or verifier failure are evaluator errors with an exact collection/retry path—not product decisions. Do not emit `no_decision`.

### Stress the verifier

Create four concrete cases from the user's domain:

1. known-good output that must pass;
2. plausible bad output a superficial verifier might accept;
3. valid alternative unlike the reference;
4. likely reward hack or evidence-tampering attempt.

For each case state the artifact/input, expected criterion results, and why. Add another case only for a named risk. Keep hidden holdouts, graders, reward code, and labels unreadable and unwritable by the evaluated policy.

Embed these four public cases in `evaluate.py` as executable examples with expected decisions. The evaluated agent receives only each case's task input, never its expected decision or private evidence.

## Phase 4 — Write the eval package

Default to `evals/<system-slug>/` inside the user's project. If that path already contains user-authored files, preserve them and ask before overwriting; otherwise choose a new slug.

Create only these files:

```text
evals/<system-slug>/
├── README.md
├── design_contract.md
├── runbook.md
└── evaluate.py
```

Copy `assets/design_contract.md` into the eval folder and adapt every `UVVR_ADAPT` section. Before authoring `runbook.md` or `evaluate.py`, read `references/runbook.md` and `references/evaluate.py` completely. Use those references as required behavioral blueprints, then write system-specific files from the confirmed criteria and actual codebase. Do not ship a renamed generic template. Remove every `UVVR_ADAPT` marker from the generated package; the skill asset/reference retain theirs for future invocations.

### `README.md`

Include the confirmed system card and an **Underlying unverifiable reward** section explaining the current signal, who supplies it, what it rewards, why it cannot prove success, and its known failure modes. Then include the eval purpose, package status, file map, and this ready-to-use coding-agent handoff adapted to the project:

```text
Read every file in this eval folder. First understand the system and underlying
unverifiable reward in README.md. Use design_contract.md as the decision authority and
runbook.md as the evidence-collection guide, then read the module docstring and
examples in evaluate.py. Follow the runbook to collect complete evidence without
changing the contract or expected outcomes. Run the checklist, every individual
check, and then the complete evaluation.
```

### `design_contract.md`

Start from the copied asset. Include the Phase 3 claim transformation, criterion map, decision/error policy, trust boundaries, versioned components, human checklist, and owned unverifiable remainder, including reference-pool provenance and sampling when used.

### `evaluate.py`

Author this from `references/evaluate.py` as the self-contained executable eval. Preserve its safety and CLI contracts while replacing its example configuration and checks with the confirmed system's real functions. Use only the Python standard library unless the confirmed system already requires a dependency.

- Start with a module docstring describing the exact task input and run evidence a coding agent must collect for every case, evidence locations, commands, trust boundaries, and both invocations.
- Define editable constants for the judge CLI command, model, timeout, and required input fields. Use an argument list with `subprocess.run`; never `shell=True`.
- Put the four concrete public examples and their expected decisions in `EXAMPLE_CASES`.
- Implement one small deterministic function per V4/V3 criterion.
- Put all V1 rubric text, evidence selection, structured-output schema, and isolated judge invocation in `run_judge`. A command such as `codex exec --ephemeral --sandbox read-only` is a judge, not ground truth. Judge failure returns evaluator `error` with a retry path.
- Implement `evaluate(case) -> result`; only sufficiently evidenced failed `block` criteria may reject. V0 stays in the reported remainder and is never scored.
- Expose `--checklist`, `--list-checks`, and `--check NAME <run.json>` so the coding agent can collect inputs and test every deterministic or judge criterion individually.
- `python evaluate.py` runs the embedded examples without model spend by using fixed judge fixtures. `python evaluate.py <run.json>` evaluates one case or a JSON list and invokes the configured judge only when V1 evidence exists.
- Validate all inputs and keep raw evidence references in the result. Make sampling reproducible when references affect a score.

Do not weaken the reference's guarded subprocess, input-validation, private-fixture, image-root, completeness-error, or decision-aggregation behavior. Change one only when the confirmed contract requires a stricter rule and record why in `design_contract.md`.

### `runbook.md`

Author this from `references/runbook.md`. It must give the coding agent concrete system commands, a field-by-field input map, criterion-to-evidence collection, completeness gate, individual-check commands, full-eval commands, human checklist, judge prerequisites, owners, and exact recovery. It must make every required input collectable and must not duplicate the criterion contract.

Print this result shape as JSON:

```yaml
status: success | warning | error
decision: accept | reject | escalate  # absent when status is error
summary: one-line outcome
criteria: {}
evidence: []
human_checklist: []
next_actions: []
artifacts: []
```

Do not create runners, adapters, manifests, result directories, or hidden datasets around `evaluate.py` until an observed need requires them.

## Completion

Finish only when the system card and underlying reward were confirmed, the light drill resolved decision-relevant ambiguity, all four files agree, no `UVVR_ADAPT` marker remains in the generated package, every required input has a runbook collection path, every listed check runs individually, the four embedded examples pass with `python evaluate.py`, a complete test case runs end to end, and the unverifiable remainder has an owner.

Report the created file paths and exact next handoff. If material evidence cannot be collected, revise the criterion or add instrumentation before completing the package; never fabricate certainty or emit `no_decision`.
