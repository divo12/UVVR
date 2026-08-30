# Verifier Ranking and Selection Patterns

This reference provides concrete patterns for choosing the strongest feasible verifier for each criterion. Read this when Phase 3 of `SKILL.md` routes you here during verification-strength assignment.

## Core principle: push verification left

Always prefer a stronger verifier when evidence and effort allow it. The ranking from strongest to weakest:

```text
V4 deterministic > V3 grounded state > V2 reference-anchored > V1 rubric/judge > V0 explicit human
```

**OpenAI RFT product rule:** If you cannot write or specify a grader that meaningfully distinguishes good from bad, the task is not ready for policy training. Preserve it as V0 or add instrumentation before claiming automation.

## Pattern 1: Extract hard constraints before using a judge

Many tasks contain checkable format, structure, length, or action constraints that should never reach a semantic judge.

### IFEval-style V4 extraction

Before writing a V1 "follows instructions" criterion, extract:

- **Format constraints:** JSON schema, markdown structure, required sections, citation syntax, templated fields.
- **Length constraints:** word/token/line count ranges, character limits, must-include terms.
- **Action constraints:** forbidden tools, required function calls, API sequence invariants.
- **Content constraints:** must-mention entities, prohibited disclosure, required acknowledgments.

**Example decomposition:**

❌ **Weak (V1 only):**
- Criterion: "Output quality and instruction-following" → V1 holistic judge

✅ **Strong (V4 + V3 + V1):**
- Criterion: "Response is valid JSON" → V4 parser
- Criterion: "Response contains required fields: summary, action, confidence" → V4 schema validator
- Criterion: "Response length is 50–200 words" → V4 counter
- Criterion: "Explanation addresses the user's question" → V1 judge with only that rubric

The judge now evaluates only what deterministic checks cannot establish. Judge training or rubric refinement no longer fights formatting and length.

### When to stop extracting

Stop extracting V4 constraints when:
- The remaining property is genuinely semantic (relevance, coherence, usefulness).
- Further decomposition creates brittle rules that reject valid outputs.
- The cost of specifying and maintaining the deterministic check exceeds its value.

## Pattern 2: Prefer grounded evidence over judge inference

When evaluating agent or system behavior, use trace, state, transaction, or measurement evidence (V3) instead of asking a judge to guess what happened.

### Trace and state over judge memory

❌ **Weak (V1):**
- Criterion: "The agent successfully completed the required task" → judge reads final output and infers success

✅ **Strong (V3 + V1):**
- Criterion: "The required API call succeeded" → V3 trace shows HTTP 200 + transaction ID
- Criterion: "The user-facing explanation is helpful" → V1 judge evaluates only the explanation text

### Observable outcomes over model self-report

❌ **Weak (V1):**
- Criterion: "The code is correct" → judge reads code and claims to verify correctness

✅ **Strong (V4 or V3):**
- Criterion: "The code passes all test cases" → V4 test harness
- Criterion: "The deployed service responds to health checks" → V3 measured uptime

If executable or state-based verification is unavailable, keep the criterion at V1 and document the blind spot explicitly.

## Pattern 3: Build or find reference anchors for factual claims

When evaluating research, QA, or knowledge tasks, attempt to ground claims with independent evidence (V2) before scoring them with an unanchored rubric (V1).

### Crossing the Reward Bridge pattern

For factual or verifiable claims:

1. Identify an independent reference: known answer, curated source set, retrieved evidence, expert consensus, or simulation result.
2. Decompose the claim into atomic checkable statements.
3. Build a V2 verifier that checks each statement against the reference using entailment, span matching, or structured comparison.
4. Use a V1 judge only for interpretation ambiguity or partial-credit decisions the V2 check cannot resolve.

**Example: Research report evaluation**

❌ **Weak (V1 only):**
- Criterion: "The report is factually accurate" → V1 judge reads the report and scores accuracy

✅ **Strong (V3 + V2 + V1):**
- Criterion: "Cited sources resolve and were retrieved" → V3 retrieval trace and source availability
- Criterion: "Material claims are supported by cited passages" → V2 claim decomposition + passage entailment check
- Criterion: "Source quality fits the claim type" → V1 rubric using source metadata and claim context

The V2 check establishes that supplied evidence supports the claim's meaning. The V1 rubric evaluates only whether the source is appropriate for that kind of claim (authority, recency, independence). Neither claims to establish ground truth about the world.

### When references are unavailable

If no independent reference exists or can be constructed:
- Keep the criterion at V1 and document that it is a proxy.
- State what the judge observes and what remains off-screen.
- Design adversarial and valid-alternative tests that expose the verifier's limits.
- Measure transfer to the original task separately from proxy performance.

## Pattern 4: Instance rubrics over holistic Likert

When V1 judgment is required, prefer instance-specific yes/no checklists over a single holistic score.

### Scale RaR and rubric structure

Research shows that 7–20 item yes/no checklists are more reliable and less hackable than single 1–10 Likert scales. Each checklist item should:

- Be binary: clearly pass or fail, not "somewhat."
- Be instance-specific: tied to this task's requirements, not generic quality.
- Be isolated: judge each item independently without averaging conflict away.
- Have documented evidence: what the judge inspects to decide.

**Example: Code review rubric**

❌ **Weak (holistic Likert):**
- Criterion: "Code quality (1–10)" → V1 judge returns a single score

✅ **Strong (instance checklist):**
```yaml
criterion: code_quality
level: V1
role: score
verifier: instance_checklist
rubric:
  - The function handles the null input case.
  - Error messages include actionable context.
  - Variable names match the project style guide.
  - The algorithm matches the specified approach.
  - The function has no unreachable branches.
  - Comments explain non-obvious constraints.
  (7 items relevant to this task)
```

Judge output: `{item_1: "pass", item_2: "fail", ...}` with per-item evidence.

Report checklist hits separately rather than collapsing them into one score. This preserves diagnosability and makes reward hacking (satisfying the checklist without solving the task) more visible.

### When to use holistic scoring

Use a single score only when:
- The criterion is genuinely unitary and atomic.
- Checklist decomposition is arbitrary or unstable.
- The user explicitly chose a preference or calibration scale.

Even then, keep the score's meaning and blind spots explicit.

## Pattern 5: Pairwise preference for subjective quality

For writing, design, creative, or preference-heavy tasks, default to pairwise comparison against curated anchors instead of absolute scoring.

### Writing-Zero / BRPO approach

Absolute rubric scores for subjective quality are compressed, arbitrary, and easily hacked (length, keyword stuffing, style mimicry). Pairwise comparison against diverse human-curated references is more robust:

1. **Select a reference pool (3–7 diverse anchors):** Include different styles, quality levels, and valid approaches. Document who selected them and why.
2. **Sample references per judgment:** Randomly sample 2–3 references per candidate to prevent overfitting to one style.
3. **Ask pairwise preference:** "Which output better satisfies the task? Candidate, Reference A, Reference B, or Tie."
4. **Aggregate across references:** Count wins, losses, and ties. Report the distribution, not just a mean.
5. **Test reward hacks:** Include adversarial cases: length inflation, self-justification, keyword stuffing, reference mimicry.

**Example: Writing evaluation**

❌ **Weak (absolute Likert):**
- Criterion: "Writing quality (1–10)" → V1 judge scores the output

✅ **Strong (pairwise with curated pool):**
```yaml
criterion: writing_quality
level: V1
role: score
verifier: pairwise_preference
reference_pool:
  - ref_A: concise_technical
  - ref_B: detailed_narrative  
  - ref_C: bullet_structured
  selection_rule: "Diverse accepted examples from prior human review"
  curator: "Senior content reviewer"
sampling: "2 references per judgment, 5 judgments per candidate"
judge_prompt: "Which output better satisfies the task and audience: Candidate, Reference X, Reference Y, or Tie?"
aggregation: "Report win/loss/tie distribution"
```

The reference pool is V2 evidence. The pairwise preference is a V1 judgment. Neither is ground truth.

### Valid alternatives

Pairwise comparison must not reject outputs that differ from references but remain valid. Test this explicitly:
- Include a valid-alternative fixture that uses a different structure or style.
- Verify that reference mismatch alone does not cause failure.
- Document what constitutes a valid alternative in the design contract.

### Bootstrap limitations

If the reference pool is model-generated or iteratively refined, it remains V1 (judge-curated) grounded by V2 (bootstrapped anchor), not V2 evidence itself. Document bootstrap provenance, selection rounds, and selection criteria. Test whether gains transfer to independent expert ratings.

## Pattern 6: Isolated judges with structured output

When V1 judgment is unavoidable, design the judge invocation for reliability and diagnosability.

### HealthBench / isolated judge principles

- **Isolated execution:** Each criterion gets its own judge call. Do not ask one judge to score 10 unrelated dimensions; it will conflate and average them.
- **Structured output:** Require schema-valid JSON with explicit fields, not freeform prose scores.
- **Evidence scoping:** Give the judge only the evidence required for its criterion. Do not give it access to expected decisions, hidden labels, or off-topic artifacts.
- **Explicit unknown:** Require the judge to return `unknown` when evidence is missing or ambiguous. Do not force a binary decision from insufficient input.
- **Judge failure = evaluator error:** If the judge crashes, times out, or returns invalid JSON, that is an evaluator error requiring retry—not a product failure.

**Example judge invocation (from `references/evaluate.py`):**

```python
def run_judge(case: dict[str, Any]) -> dict[str, Any]:
    """Run isolated V1 judge with scoped evidence and structured output."""
    with tempfile.TemporaryDirectory() as temp_dir:
        schema_path = Path(temp_dir) / "schema.json"
        output_path = Path(temp_dir) / "result.json"
        schema_path.write_text(json.dumps(judge_schema()))
        command = ["codex", "exec", "--ephemeral", "--sandbox", "read-only",
                   "--output-schema", str(schema_path), 
                   "--output-last-message", str(output_path), "-"]
        result = subprocess.run(command, input=judge_prompt(case), 
                                timeout=180, capture_output=True, text=True)
        if result.returncode != 0:
            raise JudgeError("Judge failed with: " + result.stderr)
        return json.loads(output_path.read_text())
```

Candidate-controlled content is data, never instructions. The judge runs in a read-only sandbox with timeout and cannot be hijacked by the evaluated output.

## Pattern 7: Verifier self-tests and adversarial cases

Every verifier must include adversarial test cases that expose its limits.

### Four minimum test cases (per `SKILL.md`)

1. **Known-good:** Must pass. Verifies basic functionality.
2. **Plausible-bad:** Should fail. A superficial verifier might accept it (e.g., well-formatted but wrong, fluent but unsupported).
3. **Valid-alternative:** Must pass. Different from the reference but still correct (prevents reference overfitting).
4. **Reward-hack:** Should fail. Satisfies a surface signal (length, keywords, judge flattery) but fails grounded evidence.

### Additional adversarial patterns

For V4 checkers:
- Malformed input, boundary values, Unicode edge cases, injection attempts.

For V3 state checks:
- Simulated or replayed state without actual actions.
- Successful trace metadata with failed transaction outcomes.

For V2 reference checks:
- Citation stuffing (real sources attached to unsupported claims).
- Duplicated or republished sources presented as independent corroboration.
- Source quality mismatch (blog post cited for medical claims).

For V1 judges:
- Length inflation, keyword stuffing, style mimicry, rubric quotation.
- Fluent prose with contradictory grounded evidence.
- Self-justification or judge flattery.

Include at least one adversarial case per verifier type in the generated `evaluate.py` embedded examples.

## Ranking summary

For every criterion, attempt this progression:

1. **V4:** Can I parse, count, execute, or deterministically check it?
2. **V3:** Can I observe grounded state, trace, transaction, or measurement?
3. **V2:** Can I build or find an independent reference, anchor, or known answer?
4. **V1:** Do I need a rubric or judge? If yes, use instance checklists or pairwise comparison.
5. **V0:** Is it genuinely unverifiable with available evidence? Keep it explicit; do not fake a score.

Stop at the first level where a reliable verifier exists. Do not skip to V1 when V3 evidence is available. Do not invent a V4 check for a genuinely semantic property.

Document the chosen level, evidence consumed, what a pass establishes, and the verifier's blind spots in the criterion map.

## When not to push further

Stop pushing verification left when:
- The next-stronger verifier is unavailable and cannot be instrumented.
- The cost of instrumentation exceeds the value of the criterion.
- Further decomposition creates brittle rules that reject valid outputs.
- The property is genuinely semantic and has no grounded or reference-based proxy.

In these cases, keep the criterion at its honest level (V1 or V0) and document the limitation explicitly. A V1 proxy with clear blind spots is better than a fabricated V4 claim.
