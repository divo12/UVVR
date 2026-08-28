# Research Agent Dogfood Run 01

## Run

- Harness: Arceus `services/flow-tester` underlying `run_agent_task`
- Agent: `browser-use` 0.11.13
- Model: Chorus Azure deployment `gpt-5.4-mini`
- Start page: Google
- Step budget: 15
- Credential values: not copied or logged

## Research task

> Should AI product teams evaluating open-ended agent work prioritize deterministic or environment-state verifiers before LLM-as-a-judge rubrics? Compare evidence, limitations, and failure modes; give a practical recommendation; cite exact URLs; include counterevidence.

## Harness outcome

```text
run_agent_task ok:       true
agent is_successful:     false
harness judge verdict:   PASS
source depth:            mostly search snippets; one full source page
```

The agent completed a formatted report after encountering DuckDuckGo and OpenReview verification pages. Its own limitations section disclosed that most sources were not fully opened.

## Final report

### Recommendation

Prioritize deterministic or environment-state verifiers for mechanically checkable outcomes, then use LLM rubric graders only for residual semantic criteria.

### Key findings

1. Files, tool outputs, and environment state can support direct checks that do not depend on natural-language quality judgments.
2. The sources surveyed favor a hybrid stack: deterministic checks for structured properties and LLM judges for nuance.
3. LLM rubric verification remains noisy in agentic scenarios.
4. Environment-aware judging can acquire evidence that a transcript-only judge cannot see.
5. Neither rule-based nor learned verifiers generalize universally.

### Evidence cited by the agent

| Claim | Source |
|---|---|
| Rubric-graded environments can inspect deliverables and environment state | https://openreview.net/forum?id=ayA2tJNDET |
| Some criteria require multi-file artifacts and tool-mediated environment state | https://kdd-eval-workshop.github.io/agenticai-evaluation-kdd2026/assets/papers/34_Verifying_Agents_in_Rubric_Graded_Environments.pdf |
| LLM rubric verification in agentic scenarios remains noisy | https://arxiv.org/abs/2606.29920 |
| A production framework combines deterministic checks and rubric graders | https://www.hebbia.com/blog/evaluating-ai-agents-a-hybrid-deterministic-and-rubric-based-framework |
| Observability-agent grading resolves environment facts before semantic judging | https://deepwiki.com/grafana/o11y-bench/4.3-llm-judge-and-rubric-evaluation |
| Deterministic checks should be preferred where available | https://agentpatterns.ai/verification/meta-evaluate-llm-judge-rubric-verification |
| Agent-as-a-Judge can improve on transcript-only LLM judging | https://aj-bench.github.io/ |
| AJ-Bench evaluates environment-aware judges across search, data, and GUI tasks | https://arxiv.org/abs/2604.18240 |

### Counterevidence included

- Many open-ended properties cannot be checked deterministically.
- Hybrid systems are more realistic than an either/or choice.
- Rule-based verifiers also fail outside narrow domains.
- Rubric graders remain necessary for coherence, completeness, and domain appropriateness.

### Limitations disclosed by the agent

- It relied on search-result snippets for several sources.
- It did not fully open every cited source.
- Deterministic checks establish only the properties they encode.
- LLM judges remain probabilistic proxies rather than ground truth.

## Outcome-contract evaluation

| Criterion | Result | Evidence |
|---|---|---|
| Output and citation syntax parse | Pass | Required sections and URLs are present |
| Cited sources resolve | Fail | Six sources were independently accessible; OpenReview returned a browser challenge, and the workshop PDF could not be independently fetched by the validator. This is a hard gate, so unresolved citations cannot be downgraded to a warning |
| Quoted passages match sources | Not applicable | The report used paraphrases rather than direct quotations |
| Quantities are correct | Pass with limited scope | RuVerBench's 2,458 instances and AJ-Bench's 155 tasks/516 trajectories match their primary pages; most other claims were qualitative |
| Material claims are supported | Fail | Main hybrid recommendation is supported by Hebbia, o11y-bench, RuVerBench, and AJ-Bench; other material claims lack opened source passages. This is a hard gate |
| Source quality fits the claim | Warning | Mixes primary papers with a company blog, generated repository wiki, and an inaccessible secondary page |
| Evidence and counterevidence covered | Partial | Includes useful counterarguments but misses broader reward-hacking, verifier-noise, and rubric-overoptimization literature already present in the UVVR research report |
| Recommendation follows evidence | Pass | The hybrid recommendation is more cautious than an either/or conclusion |
| Tool use respects policy | Pass with caveat | No secret was exposed; the browser agent spent several steps on CAPTCHA and verification pages and wrote a temporary plan file inside its isolated session |
| Helps the user decide | Provisional | Directionally useful, but requires stronger source validation before acting as a gold research output |

## Source validation notes

- RuVerBench is a primary arXiv source and supports the claim that LLM rubric verification remains noisy in agentic scenarios.
- AJ-Bench is a primary paper/project page and supports environment-aware Agent-as-a-Judge outperforming LLM-as-a-Judge baselines while retaining open challenges.
- Hebbia directly documents a hybrid deterministic-plus-rubric production pattern.
- The Grafana o11y-bench page documents deterministic state verification followed by grounded rubric judging.
- OpenReview content was blocked by a verification page during independent validation.
- The workshop PDF and AgentPatterns page were not independently accessible to the validator.

## Verdict

**Plausible bad / calibration case.**

The answer is fluent, correctly structured, directionally supported, and honest about its limitations. It is not a gold output because several material claims depend on snippets or inaccessible sources.

```text
Hard gates: FAIL
Reason: unresolved citations and material claims without opened supporting passages
```

The harness's PASS conflicts with `is_successful: false` and does not penalize insufficient source depth. UVVR should require source-open evidence and treat this run as a grader false positive until those checks exist.

## Upgrade path

Run the same task with source receipts that record whether each page was opened, the exact supporting passage, retrieval time, and claim mapping. A report should not pass the source-support gate using search snippets alone.
