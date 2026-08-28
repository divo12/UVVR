# Research Agent Runs 02–06

## Setup

- Same decision question and Azure `gpt-5.4-mini` deployment for every run.
- Runs 02–05: 30-step browser budget, 15-minute timeout.
- Run 06: 40-step budget, 20-minute timeout, persistent `sources.md` evidence ledger.
- Exact-passage requirement; snippets and inaccessible sources excluded.
- Raw results: [`raw/`](raw/).

## Results

| Run | Trace steps | Agent completion | Harness judge | Task compliance | Acceptance under this task | Main failure |
|---:|---:|---:|---:|---:|---:|---|
| 02 | 25 | Fail | Fail | Fail | Reject | Search URL cited; evidence summaries were not exact opened-source passages |
| 03 | 21 | Pass | Pass | Fail | Reject | GroundEval and OpenART text labeled “exact” was stitched or paraphrased rather than verbatim spans |
| 04 | 25 | Pass | Pass | Fail | Reject | GroundEval/AJ-Bench wording labeled “exact” was not found verbatim in the cited primary pages |
| 05 | 25 | Pass | Pass | Fail | Reject | Final evidence relied on two independent sources despite the explicit four-source requirement |
| 06 | 19 | Pass | Fail | Fail | Reject | Four independent sources were logged, but the SWE-bench field labeled “exact passage” used ellipses |

## Aggregate

```text
Agent-reported success: 4/5
Harness-judge pass:      3/5
Task-compliance pass:    0/5
Acceptance decision:     0 accept, 5 reject
```

Increasing the budget improved source opening and apparent task completion. It did not satisfy this run's explicit four-source and exact-passage requirements.

These rejections are task-specific decisions, not universal UVVR policy. A correct paraphrase can pass semantic claim-support evaluation when the task does not require a direct quotation. Here, the outputs explicitly labeled text as exact passages.

## Pareto observation

The current harness cannot produce a defensible success/cost/latency frontier because raw runs do not record model cost or end-to-end latency as structured fields.

The larger 30–40-step configurations increased agent-reported completion but did not improve task-compliance pass rate above 0/5. Without cost and latency measurements, no formal dominance claim is possible; this is a benchmark instrumentation gap to fix before comparing budgets.

## Observed failure modes

1. Search snippets or search URLs substituted for opened sources.
2. Paraphrases were placed inside quotation marks and called exact passages.
3. A paper and its repository were treated as independent evidence.
4. The agent lost source details while switching tabs.
5. More steps sometimes produced tab-switching loops rather than better evidence.
6. Agent completion status, harness-judge verdict, and contract verdict disagreed.
7. The source ledger improved source counting but did not enforce verbatim fidelity.

## Source validation examples

- GroundEval's primary abstract supports trajectory and evidence-path verification, but not every quote produced by the agent was verbatim: https://arxiv.org/abs/2606.22737
- JADE supports the rigor/flexibility tradeoff and evidence-gated claim evaluation: https://arxiv.org/abs/2602.06486
- OpenART supports persistent shared-state environments and fixed objectives under changing state: https://arxiv.org/abs/2608.00677
- The expert-knowledge study reports only 68% and 64% SME agreement with LLM judges in its two domains: https://arxiv.org/abs/2410.20266
- RuVerBench reports substantial remaining noise in LLM rubric verification for agentic outputs: https://arxiv.org/abs/2606.29920

## Root cause

The browser agent stores evidence as natural-language memory. The final writer can paraphrase that memory while believing it copied the source exactly. Prompting it to maintain `sources.md` improves retention but does not prove the recorded span exists.

## Next harness change

Add a source-receipt boundary outside the model:

```text
receipt = {
  canonical_url,
  retrieved_at,
  source_hash,
  exact_passage,
  passage_start,
  passage_end
}
```

Before a report is eligible:

1. Fetch each canonical URL independently.
2. Normalize the page text deterministically.
3. Require every declared direct quotation to be an exact substring or validated ordered omission; evaluate paraphrases for semantic support instead.
4. Group paper/project mirrors as one source.
5. Reject claims whose receipt is missing or invalid.

This is a verifier improvement, not another prompt instruction.
