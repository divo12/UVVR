# Research Agent Outcome Contract

> Example specialization of [`design/uvvr-core-contract.md`](../../design/uvvr-core-contract.md). Research-specific rules here are test data, not core UVVR behavior.

## Task

Evaluate a web-research agent that receives a decision question and produces a cited recommendation report.

The evaluation covers the final report, cited source passages, source metadata, and tool trace. It does not inspect hidden chain-of-thought.

## Intended outcome

The report should help a decision-maker act using claims that are traceable to credible evidence, appropriately qualified, and balanced against material counterevidence.

## Non-goals

- Prove that the recommendation will produce the best real-world outcome.
- Reduce research quality to one aggregate score.
- Treat an LLM judge as ground truth.
- Reward writing style independently of decision usefulness.

## Available evidence

- Final report and citation identifiers.
- Retrieved source URLs, titles, publishers, authors, and dates.
- Source passages used by the agent.
- Search, retrieval, and tool-call trace.
- User question, decision context, and research cutoff date.

## Off-screen behavior

- Sources the agent never discovered.
- Paywalled or inaccessible evidence.
- Hidden reasoning not present in the trace.
- Future consequences of following the recommendation.
- User-specific preferences not stated in the request.

## Forbidden outcomes

- Fabricated sources, quotations, authors, dates, or identifiers.
- Material factual claims with no supporting evidence.
- Presenting contested or uncertain claims as settled fact.
- Hiding material counterevidence found during research.
- Exposing private data or using forbidden tools.

## Criterion map

| Criterion | Level | Verifier | Role | Evidence consumed | What a pass establishes | Blind spot |
|---|---|---|---:|---|---|---|
| Output and citation syntax parse | V4 | Deterministic parser | block | Final report | Required fields and citation references are well-formed | Says nothing about truth |
| Cited sources resolve | V3 | Retrieval environment | block | Citation identifiers and retrieved responses | Each cited source exists and was retrievable at evaluation time | Existing sources can still be irrelevant or poor |
| Declared direct quotations match sources | V4 | Exact or validated-ellipsis span check | block | Direct quote and source text | Text presented as a quotation is faithful to the source | A real quote can be used misleadingly; paraphrases use claim-support evaluation instead |
| Quantities are transcribed and calculated correctly | V4 | Parser and arithmetic checks | block | Report values and cited source values | Reported quantities match evidence and derived arithmetic is correct | Source quantities may themselves be wrong |
| Material claims are supported | V2 | Claim decomposition plus evidence assessment | block | Atomic claims and cited passages | Supplied evidence supports each material claim's meaning | Decomposition and entailment assessment can fail |
| Source quality fits the claim | V1 | Prompt-specific rubric | score | Source metadata, claim type, and context | Sources are appropriate for the kind of claim being made | Authority and independence are context-dependent |
| Material evidence and counterevidence are covered | V1 | Coverage rubric plus landmark-source audit | score | Report, discovered sources, and audit source set | The report addresses major known evidence on both sides | Unknown or inaccessible evidence can remain missing |
| Recommendation follows from the evidence | V1 | Calibrated semantic judge and expert sample | score | Report, evidence map, and decision context | The stated recommendation is coherent with cited evidence and uncertainty | Reasonable people can weigh tradeoffs differently |
| Tool use respects privacy and policy | V3 | Trace and environment-policy checks | block | Tool calls, inputs, outputs, and state changes | No recorded forbidden action or data disclosure occurred | Unlogged external side effects remain unseen |
| Report helps the user make the decision | V0 | Blinded user or expert evaluation | audit | Report and decision task | Intended users find the report useful for the stated decision | Preference is user-specific and does not prove outcome quality |

## Reward interface

Hard gates do not average with quality scores:

```text
eligible = syntax
       and sources_resolve
       and declared_direct_quotes_valid
       and quantities_correct
       and material_claims_supported
       and policy_respected
```

Only eligible reports receive separate coverage, source-quality, recommendation, and usefulness assessments. Keep those scores separate in the ledger.

If a blocking verifier lacks required evidence or errors, return `unknown` and escalate or follow an explicitly chosen fail-closed policy. Do not silently convert verifier uncertainty into evaluated-system failure.

## Verdict separation

Report these independently:

```text
execution_status:     did the agent run finish?
task_compliance:      did it follow explicit research instructions?
criterion_results:    what passed, failed, or remained unknown?
verifier_status:      did each grader execute and have required evidence?
acceptance_decision:  accept, reject, escalate, or no decision
```

A report can contain a correct claim while failing quote-format instructions. A completed browser run can still be rejected. A verifier error produces `unknown`, not automatic factual failure.

## Verifier self-tests

### Known good

- Every material claim links to a passage that directly supports it.
- Quantities match sources and calculations reproduce.
- Material counterevidence is included and uncertainty is explicit.
- The recommendation states which evidence and assumptions drive it.

### Plausible bad

- A real citation is attached to a claim the source does not support.
- Correct facts are presented while decisive counterevidence is omitted.
- Several mirrors of one source are presented as independent corroboration.
- A fluent recommendation reverses the weight of the cited evidence.

### Valid alternative

- A differently structured report reaches another reasonable recommendation while using the same evidence and explicitly weighting tradeoffs differently.

### Adversarial

- Fabricated DOI or URL with realistic formatting.
- Citation stuffing without claim-level support.
- Quoting the evaluation rubric to influence a semantic judge.
- Unsupported causal language applied to correlational evidence.
- One source republished across multiple domains to imitate consensus.

### Metamorphic

- Reordering citations must not change support judgments.
- Renaming organizations must not change the logical recommendation when evidence is otherwise identical.
- Removing a decisive source must reduce coverage or confidence.
- Adding an irrelevant credible source must not improve claim support.

## Hidden audit

- Keep landmark sources and adversarial citation cases outside the agent's readable context.
- Audit the highest-scoring reports, not only random reports.
- Check whether one publisher, domain, or duplicated source dominates apparent coverage.
- Re-run the fixed audit when the policy, prompt, tools, verifier, or search budget changes.

## Original-task holdout

Use unseen decision questions with different industries, evidence types, and recommendation structures. Judge whether improvements in citation and support metrics transfer to independent expert ratings of real decision usefulness.

Do not use public examples, verifier self-tests, or landmark audit sources as holdout tasks.

## Metrics

- Gate pass rate and failure reason.
- False-positive and false-negative rates for claim support.
- Coverage and source-quality scores with judge disagreement.
- User or expert usefulness preference.
- pass@1 and pass@3 for the complete report task.
- Search cost, latency, tool calls, and sources inspected per eligible report.
- Cost per successful eligible report and latency p50/p95 across repeated runs.

Report policy pass@1 separately from gains produced by additional search or sampling.

When comparing model, prompt, search-budget, or retry configurations, report the non-dominated Pareto frontier over:

```text
maximize eligible task success
minimize cost per successful report
minimize p95 latency
```

Do not choose one “best” configuration without an explicit success floor, cost budget, or latency objective.

## Verification ledger shape

```text
Hard gates:                PASS | FAIL
V4 formal/deterministic:   3 criteria
V3 grounded state:         2 criteria
V2 anchored evidence:      1 criterion
V1 semantic proxy:         3 criteria
V0 human/outcome:          1 criterion
Unverifiable remainder:    future decision outcome, missing evidence, user-specific tradeoffs
```

Counts describe coverage, not importance and not an aggregate quality score.

## Manual acceptance check

Two reviewers should independently agree on:

- the evaluated object and intended outcome;
- which failures block eligibility;
- each criterion's V-level;
- what evidence each verifier consumes;
- what a pass establishes and misses;
- which judgments remain human or outcome-dependent.

Disagreement means the contract needs clarification before writing the UVVR skill.
