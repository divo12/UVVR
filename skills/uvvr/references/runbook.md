# Runbook authoring reference

Read this reference when Phase 4 creates `evals/<system-slug>/runbook.md`.
The generated runbook is an operational instruction for a coding agent: it must
make every required `evaluate.py` input collectable and every check runnable.
It is not another criterion contract and must not restate `design_contract.md`.
Read it together with `references/evaluate.py`; the generated runbook and
evaluator must be designed as one interface from the same criterion map.

## Required outcome

A coding agent with only the generated eval folder and access to the confirmed
system must be able to:

1. identify the exact system run to evaluate;
2. collect every required raw input, output, trace, state, reference, and image;
3. build schema-valid run JSON without seeing private expected decisions;
4. run the collection checklist;
5. list and execute individual evaluator functions;
6. run embedded examples without model spend;
7. run a complete real evaluation, including the configured V1 judge;
8. interpret `accept`, `reject`, `escalate`, and evaluator `error`;
9. repair missing/malformed evidence and rerun without modifying raw output.

Do not ship a runbook that leaves a required criterion dependent on evidence the
coding agent cannot obtain. If evidence is unavailable, redesign the criterion,
add instrumentation, or assign a human collection step before completing UVVR.

## Generated runbook structure

Use these sections, omitting only sections that truly do not apply.

### 1. System, evaluated object, and authority

State:

- system name and input → actions → output flow;
- exact run/artifact being judged;
- decision supported by the eval;
- path to `design_contract.md` and `evaluate.py`;
- authoritative evidence when specifications, model prose, and observed behavior
  disagree;
- human owner for the unverifiable remainder.

### 2. Prerequisites

List concrete prerequisites rather than generic setup:

- repository/workspace path;
- runtime and version;
- agent/system command or API;
- authentication/environment variables by name only—never values;
- browser/service/dependency state;
- judge CLI availability and approved model;
- writable artifact directory and `UVVR_EVIDENCE_ROOT` when images are used.

Give a read-only preflight command for every prerequisite when practical.
Time-bound dependency installation, container builds, health checks, live system
runs, and judge calls. Document how to distinguish system failure from a busy or
broken local runtime, and do not let abandoned setup processes continue in the
background.

Before writing this section, actually run the safe preflights and inspect the
repository's runtime declarations (`Dockerfile`, package/requirements files,
version files, service README). Do not prescribe the bare `python3` on a host whose
observed version cannot install the pinned dependencies. Choose a compatible,
available path and record the observed version/evidence behind that choice.

If the service is not guaranteed to be running, include a concrete cold-start
path (native, container, or approved remote), health probe, cleanup command, and
timeout for every setup/run step. Naming an endpoint without explaining how to
make it reachable is incomplete.

A generated runbook fails acceptance if it contains bare long-running commands
such as `docker build`, dependency/browser installation, service/agent execution,
or `curl` without an explicit bound. Use a platform-available timeout wrapper or
`subprocess.run(..., timeout=...)` for builds/installs and `--max-time` for curl.

Execute the chosen preflight/health path when safe. If shared infrastructure or
missing authority prevents it, mark the generated package `draft`/incomplete and
report the exact blocker; do not claim the runbook path is proven.

### 3. Case JSON field map

Copy the exact schema from the adapted `evaluate.py` module docstring. For every
field provide:

| Field | Source | Collection moment | Normalization | Used by checks |
|---|---|---|---|---|

At minimum explain `id`, `input`, `output`, and `evidence`. List every name in
`REQUIRED_OUTPUT_FIELDS` and `REQUIRED_EVIDENCE_FIELDS`. If a field contains a
path, state its allowed root and who intentionally selected it.

For every output/artifact, prove that the selected invocation path actually
returns or persists it. Inspect wrapper routes, serializers, redaction, truncation,
and context-size filters. If a convenient MCP/API wrapper removes an artifact
required by `evaluate.py`, the runbook must use a lower-level path or add explicit
instrumentation rather than claiming the field is collectable.

Private example keys such as `expected_decision` and `judge_fixture` must never
appear in real-run JSON.

### 4. Criterion-to-evidence checklist

For every row of `design_contract.md`, map:

- exact check name from `python evaluate.py --list-checks`;
- evidence object and field;
- collection command/action;
- what a valid evidence reference looks like;
- what the collector must do if the evidence is absent or contradictory.

V4/V3 criteria need direct functions. V2 anchors need provenance/version and a
clear relationship to the claim. V1 criteria need the rubric inputs and judge
attachments. V0 concerns belong in `--checklist` for human inspection and are
not converted into automatic scores.

### 5. Collection procedure

Write an ordered procedure tailored to the system.

Before the run:

- freeze the task/input version;
- record the exact command/configuration;
- collect read-only references and anchors;
- create an artifact directory under the approved evidence root.

During the run:

- preserve the raw input actually supplied;
- capture actions/traces/state transitions and errors;
- capture required intermediate/final artifacts;
- do not let the evaluated agent read expected decisions or hidden evidence.

After the run:

- preserve raw output before interpretation;
- collect final state/measurement/screenshots;
- build the run JSON using only observed evidence;
- retain precise references such as trace index, artifact path, state query, or
  source span rather than prose like “looks correct.”

When semantic interpretation is required, pass raw evidence to `evaluate.py`;
do not pre-label the criterion in collected input unless the deterministic input
contract explicitly requires an instrumented status.

If the system has multiple invocation paths, name which one is valid for the full
eval and which support only partial/debug checks. Verify artifact parity from code,
not documentation alone.

### 6. Completeness gate

The runbook must make this command actionable:

```bash
python evaluate.py --checklist
```

Before a full eval, the coding agent checks every returned item and every required
field. Incomplete input is an evaluator `error`, not `no_decision`. Provide the
exact recollection/retry step for each possible missing field.

Every criterion that participates in the complete result—including `score` and
`escalate` criteria—must be conclusive. `unknown` is allowed only as an individual
check result that instructs recollection/retry; the full evaluator must return
`error` until all criteria have judgeable evidence.

If a required field can never be collected in the actual environment, stop and
change the design contract/evaluator before running model comparisons.

### 7. Verify examples

Document:

```bash
python evaluate.py
```

All four adapted public examples must pass without calling an external judge.
Explain their expected decisions and the failure each example guards against.

### 8. Run individual functions

Document both discovery and invocation:

```bash
python evaluate.py --list-checks
python evaluate.py --check CHECK_NAME /absolute/path/to/run.json
```

Give one concrete command for every deterministic function and V1 criterion.
Explain whether the command spends judge tokens. Use individual checks as a
debugging checklist; do not cherry-pick only favorable checks for the final call.

Each individual deterministic check must run with only its own documented fields;
do not require screenshot/design/judge inputs to debug a URL or schema check.
Judge checks may require their complete rubric-specific evidence.
Deterministic check functions must not rerun global/private-field validation;
`evaluate(..., allow_judge_fixture=True)` owns fixture trust for embedded examples.

### 9. Run the complete eval

Document:

```bash
python evaluate.py /absolute/path/to/run.json
python evaluate.py /absolute/path/to/runs.json
```

Show safe stdout redirection for retained results. State that `evaluate.py`,
`design_contract.md`, judge rubric, and raw output must remain unchanged during a
run. If any changes, version them and rerun all examples and cases.

### 10. Judge-based criteria

For each V1 criterion state:

- configured check name and rubric;
- evidence fields and approved images passed to the judge;
- `JUDGE_COMMAND`, model-selection variable, timeout, and sandbox boundary;
- expected structured output values;
- retry/owner when the judge command errors or returns invalid JSON;
- why the judge is a proxy rather than ground truth.

Judge failure is evaluator `error`; retry or repair it. Do not silently replace it
with a pass, rejection, or `no_decision`.

Before delivery, run at least one real configured judge invocation using complete
non-private evidence. A shim/fixed fixture proves decision plumbing only and does
not prove CLI flags, authentication, image attachment, schema parsing, or timeout.
Do not cite a shim as satisfaction of this gate.

### 11. Human checklist

Document:

```bash
python evaluate.py --checklist
```

Explain every `HUMAN_CHECKLIST` item, the human owner, evidence they inspect, and
where they record their conclusion. These concerns remain explicit V0 remainder;
they do not become a fake automated score merely to make the script return.

### 12. Interpretation and recovery

Use these meanings:

- `accept`: all blocker evidence passes and nothing requires escalation;
- `reject`: sufficient evidence shows an explicit blocker failed;
- `escalate`: a complete V1/high-consequence criterion needs human decision;
- evaluator `error`: collection, configuration, or judge execution is incomplete.

For every error include root-cause hint, exact safe retry, expected corrected
artifact, and stop condition. Preserve the original run for offline rescoring.
If setup is blocked by shared infrastructure contention, stop the scoped setup
process, preserve its logs, and retry only after the runtime is available; never
convert setup failure into evaluated-system evidence.

### 13. Ready-to-paste coding-agent instruction

End the generated runbook with a project-specific instruction equivalent to:

```text
Read README.md, design_contract.md, runbook.md, and evaluate.py. Do not change
the contract, evaluator, raw system output, or expected examples. Follow the
runbook to collect a complete run JSON. Run --checklist, run every name returned
by --list-checks individually, then run the complete evaluator. Return commands,
JSON results, evidence paths, evaluator errors/retries, and the human checklist.
```

## Runbook acceptance checklist

Before delivery, verify:

- every evaluator-required field appears in the field map;
- every criterion maps to a callable check and collectable evidence;
- actual system invocation commands are concrete;
- runtime/setup commands match versions observed during generation;
- setup, system, and judge commands have explicit timeouts and cleanup;
- no bare long-running build/install/health/run command remains unbounded;
- the selected full-run path is proven to expose every required artifact;
- individual-check commands cover all listed checks;
- deterministic individual checks require only their own evidence;
- embedded private fixture allowance reaches every check without being re-rejected;
- judge prerequisites and failure recovery are explicit;
- image paths are constrained by `UVVR_EVIDENCE_ROOT`;
- examples run without judge spend;
- one real configured judge call succeeds with structured output;
- the generated status remains draft/incomplete when either the real system path
  or real judge path could not be executed;
- full evaluation has no `no_decision` branch;
- incomplete input and judge failure return evaluator `error` with next actions;
- no private labels, credentials, or `UVVR_ADAPT` markers appear in the generated
  package.
