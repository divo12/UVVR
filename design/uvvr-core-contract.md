# UVVR Core Contract

Status: design source of truth for the first instruction-only skill.

## Purpose

UVVR helps a user turn an AI system's intended behavior into an honest, testable evaluation design.

It does not make subjective goals objectively true. It identifies what can be checked, chooses the strongest feasible verifier, assigns that signal an appropriate decision role, and preserves what remains uncertain or human-judged.

## Non-goals

- Run the eval or judge the AI system unless the user separately requests execution.
- Treat an LLM judge, rubric, preference, or majority vote as ground truth.
- Force every criterion into an automated score.
- Prescribe research-specific, coding-specific, medical, or other domain rules in the core skill.
- Generate executable verifier code and run it without an explicit trust boundary and authorization.
- Collapse all evaluation dimensions into one unexplained score.

## Two independent axes

UVVR must never infer one axis from the other.

### Verification strength

| Level | Meaning |
|---|---|
| V4 | Formal, deterministic, or executable verification against a specified contract |
| V3 | Grounded environment state, transition, transaction, or measured outcome |
| V2 | Independent reference, evidence, provenance, or known latent-variable anchor |
| V1 | Structured semantic proxy such as a rubric, learned verifier, or model judge |
| V0 | Human preference, expert judgment, contested value, or delayed real-world outcome |

The level describes how the signal is established, not how important it is.

### Decision role

| Role | Meaning |
|---|---|
| `block` | A sufficiently trustworthy failure prevents acceptance or release |
| `score` | The signal contributes diagnostic or quality information without independently blocking |
| `audit` | The signal is monitored for drift, investigation, or later policy decisions |
| `escalate` | The result requires human, expert, or higher-trust review |

The role depends on consequence, verifier reliability, and user policy—not on V-level alone.

Examples:

- A V4 formatting check can be a non-blocking score.
- A V3 privacy violation can block immediately.
- A V1 safety judgment can escalate instead of making an irreversible decision.
- A V0 long-term outcome can be an audit signal used to update future evals.

## Activation contract

Use UVVR when the user wants to:

- design evals for an AI model, agent, workflow, or product;
- turn an open-ended or partly unverifiable goal into checkable evaluation criteria;
- decide which verifiers, rubrics, environment checks, references, or human reviews belong in an eval stack;
- define grader calibration, hidden audits, holdouts, or verification coverage.

Do not activate UVVR for:

- ordinary unit-test implementation with already specified assertions;
- general explanations of RL, RLHF, or RLVR without an eval-design request;
- directly grading one answer when the user did not ask to design the evaluation;
- broad product planning unrelated to evaluation.

## Input contract

### Minimum input

- AI system or workflow being evaluated.
- Intended outcome or decision the system should support.
- At least one representative task or usage example.

### Useful optional input

- Excellent, acceptable, failed, unsafe, or disputed outputs.
- Available artifacts, references, tools, traces, environment state, and delayed outcomes.
- Forbidden outcomes and risk tolerance.
- Cost, latency, privacy, compliance, and operational constraints.
- Existing evals, graders, benchmarks, or production incidents.

### Missing input policy

Ask only questions whose answers could change:

- the evaluated object;
- criterion decomposition;
- verification strength;
- decision role;
- available evidence;
- trust boundary;
- acceptance or escalation policy.

If useful work is still possible, produce a provisional design and label assumptions rather than blocking on exhaustive discovery.

## Workflow contract

Use six reasoning actions. They are workflow phases, not executable tools.

1. `inspect`: identify the evaluated object, outcome, context, evidence, environment, and off-screen behavior.
2. `decompose`: split intent into atomic success criteria and forbidden outcomes.
3. `route`: assign each criterion a V-level and candidate verifier without yet deciding its product consequence.
4. `assign`: choose `block`, `score`, `audit`, or `escalate` using consequence and verifier reliability.
5. `stress`: design positive, plausible-bad, valid-alternative, adversarial, metamorphic, and high-reward-tail tests.
6. `report`: produce the eval design, verification ledger, unresolved remainder, and next actions.

Skip phases whose result is already explicit in user-provided material. Do not perform ceremonial steps.

## Criterion contract

Every criterion must define:

```yaml
id: stable_name
property: what must be true
evaluated_object: answer | artifact | step | action | state | trajectory | outcome
criticality: low | medium | high | critical
available_evidence: []
verification_level: V4 | V3 | V2 | V1 | V0
verifier_id: stable_name
decision_role: block | score | audit | escalate
pass_means: precise claim established by pass
fail_means: precise claim established by fail
unknown_means: why the verifier could not decide
false_positive_cost: consequence of accepting bad behavior
false_negative_cost: consequence of rejecting good behavior
blind_spots: []
owner: decision owner or reviewer
```

Rules:

- Keep criteria atomic enough that one failure has one interpretable meaning.
- Separate success properties from forbidden outcomes.
- Do not use vague properties such as “good,” “high quality,” or “safe” without decomposition.
- Preserve `unknown`; do not force missing evidence into pass or fail.
- A criterion can have multiple verifiers, but each verifier must have a distinct purpose.

## Verifier contract

Every proposed verifier must define:

```yaml
id: stable_name
checks: criterion_id
type: formal | executable | state | anchor | semantic_proxy | human | delayed_outcome
inputs: []
procedure: reproducible evaluation method
output: pass | fail | unknown | score
normalization: parsing, equivalence, and malformed-input rules
trust_boundary: what the evaluated system can read or modify
versioned_components: []
cost_and_latency: estimate or measurement
known_failure_modes: []
calibration_required: true | false
```

A verifier description must state:

- what it observes;
- what evidence it consumes;
- what remains off-screen;
- what pass, fail, and unknown establish;
- whether the evaluated policy can influence or tamper with it.

## Evidence contract

Evidence is domain-specific; its integrity requirements are general.

Every evidence item should identify:

```yaml
id: stable_name
kind: output | artifact | trace | state | reference | measurement | human_label
origin: where it came from
observed_at: relevant time or cutoff
locator: path, record ID, URL, event ID, or equivalent
integrity: hash, signature, immutable snapshot, or stated limitation
independence: relationship to other evidence
access_scope: what the policy and verifier may read
supports: criterion IDs
```

Domain adapters can specialize this contract. For example, a research evaluator may use source spans; a coding evaluator may use tests and repository state; a support evaluator may use transaction records. These specializations belong in examples or focused references, not the core workflow.

## Decision-role contract

Assign a signal to `block` only when:

- the criterion is acceptance-critical;
- verifier false-positive and false-negative behavior is understood enough for the consequence;
- the verifier has the required evidence;
- failure is not merely verifier uncertainty;
- an override or escalation path exists when appropriate.

Use `score` when the signal is useful but should not independently decide acceptance.

Use `audit` when the signal is exploratory, delayed, drift-oriented, or insufficiently calibrated for immediate decisions.

Use `escalate` when the criterion is critical but available automation is too weak, ambiguous, contested, or high-risk.

## Verdict separation contract

Never collapse these layers implicitly:

```yaml
execution_status: completed | interrupted | error
task_compliance: pass | fail | unknown
criterion_results:
  criterion_id: pass | fail | unknown | score
verifier_status:
  verifier_id: success | error | uncalibrated
acceptance_decision: accept | reject | escalate | no_decision
```

- `execution_status` says whether the evaluated run finished, not whether its work was good.
- `task_compliance` says whether explicit task requirements were followed, not whether every factual or quality criterion passed.
- `criterion_results` preserve independent evaluation dimensions.
- `verifier_status` distinguishes evaluated-system failure from grader or evidence failure.
- `acceptance_decision` applies the user-owned decision policy to available results.

A completed run can be rejected. A correct claim can coexist with an instruction-format failure. A verifier error should produce `unknown` or escalation, not an automatic policy failure unless the user explicitly chose fail-closed behavior.

## Pareto optimization contract

When an eval compares models, prompts, tools, search budgets, retry policies, or agent harnesses, treat selection as constrained multi-objective optimization:

```text
maximize task success
minimize cost
minimize latency
```

Apply acceptance-critical `block` criteria first. Configurations that violate the chosen hard policy are infeasible and do not become attractive merely because they are cheap or fast.

For every evaluated configuration record:

```yaml
configuration_id: stable_name
task_success_definition: explicit acceptance policy or metric
task_success_rate: estimate
pass_at_1: estimate
pass_at_k: estimate when retries or search are part of the system
cost_per_attempt: distribution
cost_per_success: distribution
latency_ms:
  p50: value
  p95: value
  p99: value when relevant
sample_count: n
uncertainty: confidence interval or repeated-run variance
hard_policy_eligible: true | false
```

Configuration A dominates B only when:

```text
success(A) >= success(B)
cost(A)    <= cost(B)
latency(A) <= latency(B)
```

and at least one inequality is strict. Use uncertainty when deciding whether apparent dominance is credible.

Report all non-dominated configurations as the Pareto frontier. Do not collapse the objectives into one weighted score unless the user supplies the weights or a product policy defines them.

Measurement rules:

- If a configuration has zero successes, cost per success is undefined or infinite, never zero.
- Report latency across all attempts and successful attempts separately when failures or timeouts differ materially.
- Count timeouts as failed attempts and report their timeout threshold; do not drop them from latency statistics.
- Use the same task distribution, acceptance policy, tool permissions, and accounting boundaries across configurations.
- Include model, tool, retrieval, verifier, and retry costs inside the declared cost boundary, or explicitly list exclusions.

Choose an operating point with explicit constraints, for example:

- minimum acceptable task-success rate;
- maximum cost per successful task;
- p95 latency service-level objective;
- minimum safety or policy-gate performance.

Report policy-only pass@1 separately from gains produced by retries, best-of-N search, larger tool budgets, or a stronger harness. Those are different configurations, not free model improvement.

## Calibration contract

Each blocking or scoring verifier needs the smallest meaningful calibration set:

- known-good example;
- plausible-bad example that a superficial verifier might accept;
- valid alternative unlike the reference;
- malformed or missing-evidence example;
- adversarial example targeting the verifier;
- metamorphic or invariant variant when applicable;
- high-reward-tail output after search or optimization.

Measure or estimate:

- false-positive rate;
- false-negative rate;
- unknown or abstention rate;
- disagreement with independent human, expert, or verifier labels;
- sensitivity to formatting, ordering, length, and irrelevant content;
- drift across policy and verifier versions.

## Transformation contract

When direct verification is unavailable, UVVR may propose:

- known-answer construction;
- round-trip consistency;
- metamorphic relations;
- differential comparison;
- self-play or adversarial games;
- simulation or digital environment;
- reference or evidence anchoring;
- held-out-future or downstream-outcome prediction.

For every transformation, state separately:

1. Why the transformed task has a reliable label.
2. Why performance may transfer to the original capability.
3. How original-task transfer will be tested.
4. Which shortcuts or spurious signals the transformation may introduce.

A perfect proxy label does not establish original-task fidelity.

## Test-case contract

An eval design should select only relevant case families:

- positive or ordinary use;
- negative, refusal, or abstention;
- boundary or threshold;
- adversarial or exploit-seeking;
- counterfactual;
- invariance or metamorphic;
- out-of-distribution;
- regression from a real incident.

Every case must record provenance, targeted criteria, expected evidence, forbidden outcomes, visibility to the policy, and whether it belongs to examples, development, calibration, benchmark, holdout, or external audit data.

## Trust-boundary contract

For an executable eval or benchmark, specify:

- policy-readable assets;
- hidden references and holdouts;
- writable environment state;
- read-only graders, reward code, logs, and labels;
- secrets and credentials;
- sandbox and resource limits;
- trace and evidence retention;
- treatment/control isolation;
- model-judge independence where practical.

If hidden labels or verifier assets are exposed or writable, mark the result invalid rather than merely lower confidence.

## Output contract

Every UVVR response begins with:

```yaml
status: ready | needs_input | blocked
summary: one-line statement of what can and cannot be verified
next_actions: []
artifacts: []
```

Then provide only sections useful to the task:

1. Scope: system, evaluated object, intended outcome, non-goals, assumptions.
2. Criterion map: criterion contracts in a compact table.
3. Verifier stack: verifier and evidence contracts.
4. Decision policy: block, score, audit, and escalation rules.
5. Calibration and cases: minimum tests and hidden audits.
6. Holdout and integrity: transfer checks, isolation, versioning, and attribution.
7. Verification ledger: coverage by V-level and decision role, without implying aggregate importance.
8. Pareto frontier: non-dominated success/cost/latency configurations and the selected operating constraints.
9. Unverifiable remainder: missing evidence, contested values, delayed outcomes, and human ownership.

When the eval is executed, report execution status, task compliance, criterion results, verifier status, and acceptance decision separately.

Do not emit empty boilerplate sections.

## Recovery contract

| Condition | Response |
|---|---|
| Missing but non-critical input | Produce a provisional design with explicit assumptions |
| Missing material evidence | `needs_input`; name the evidence and why it changes the design |
| Conflicting requirements | `blocked`; name the conflict and decision owner |
| No faithful automated verifier | Preserve V0 or V1 and recommend human review or delayed measurement |
| Unsafe verifier execution | Stop; specify sandbox, permission, or isolation required |
| Contaminated holdout or writable grader | Invalidate the affected result and require a clean run |
| Verifier disagreement | Return `unknown` or escalate; do not average away the conflict silently |

Every blocked or failed path includes a root-cause hint, safe retry, and stop condition.

## Completion contract

An eval design is ready only when:

- the evaluated object and intended outcome are explicit;
- every criterion defines property, evidence, V-level, verifier, decision role, and blind spots;
- pass, fail, and unknown have distinct meanings;
- hard decisions are supported by adequate verifier reliability or escalation;
- relevant known-good, plausible-bad, valid-alternative, adversarial, and transfer tests exist;
- hidden audits and holdouts are separated from examples and development data;
- trust boundaries and versioned components are named;
- policy improvement is separated from search, retries, tools, and other system gains;
- compared configurations report task success, cost, latency, repeated-run uncertainty, and the non-dominated frontier;
- the unverifiable remainder and its owner are explicit.

If these conditions cannot be met, return a useful provisional design rather than fabricating completeness.

## Skill context contract

The first `skills/uvvr/SKILL.md` should contain:

- activation boundaries;
- the two-axis model;
- the six workflow actions;
- essential criterion, verifier, output, recovery, and completion rules;
- a link to this contract when full field definitions are needed.

It should not contain:

- the full research literature;
- research-agent citation rules;
- every transformation example;
- benchmark implementation detail;
- plugin or host-specific behavior.

## Behavioral validation contract

Test the future skill on at least:

- an open-ended research or synthesis system;
- an environment-changing agent;
- a naturally executable task;
- a preference-heavy creative task;
- a high-stakes task requiring escalation;
- a task where a perfect proxy has weak original-task transfer.

Compare a baseline agent and UVVR-assisted agent using shuffled outputs. Test decisions and invariants, not exact prose or heading order.

Required observable behaviors:

- no domain-specific hardcoding in the core workflow;
- no inference from V-level to decision role;
- no learned judge labeled as ground truth;
- no unsupported binary decision when evidence is unknown;
- no proxy gain reported as original-task gain;
- clear next action when the design is provisional or blocked.

## Deferred decisions

Do not decide these until behavior testing demonstrates a need:

- machine-readable JSON Schema;
- executable source-receipt or verifier scripts;
- companion audit or calibration skills;
- Codex plugin manifest;
- hooks, persistent modes, MCP, hosted service, or dashboard;
- additional host adapters.
