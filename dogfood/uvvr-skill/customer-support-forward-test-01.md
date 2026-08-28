# UVVR Skill Forward Test 01

## Input

Design evals for a customer-support agent that can look up orders, issue refunds, update tickets, and write customer responses. Evaluate correct resolution, privacy, refund policy, communication, cost, latency, and cases requiring human approval.

The independent evaluator received only `skills/uvvr/SKILL.md`; it was forbidden from reading the research example, core design, dogfood runs, or prior conversation.

## Result

The skill generalized without research-specific rules. It produced:

- V4 blockers for order facts, refund policy, approval, and private-data protection;
- V3 state verification for ticket and refund consistency;
- V2 case-oracle verification for valid resolution alternatives;
- V1 scoring for clarity, completeness, and tone;
- V0 audit outcomes for customer satisfaction and reopen rate;
- known-good, plausible-bad, valid-alternative, adversarial, and hidden cases;
- read-only grader and holdout trust boundaries;
- separate automatic, human-approved, and delayed outcomes;
- a non-dominated success/cost/latency Pareto frontier.

## Invariant review

| Invariant | Result |
|---|---|
| No research/citation hardcoding | Pass |
| V-level separate from decision role | Pass |
| Learned judge not ground truth | Pass |
| Unknown and escalation preserved | Pass |
| Hard blockers cannot be offset by tone/cost | Pass |
| Policy gain separated from retries/harness | Pass |
| Pareto frontier retained without scalarization | Pass |

## Observed defect

The output returned `status: ready` with empty `next_actions` even though the user supplied no success floor, cost ceiling, or latency SLA. It described cost/latency pass/fail relative to a “declared budget” that did not exist.

## Fix

The skill now requires an unresolved operating-point constraint to appear in `next_actions`. Without product constraints it reports the Pareto frontier but does not invent thresholds or choose one configuration.

## Verdict

Pass after the focused Pareto-selection correction. No supporting files, scripts, or plugin scaffolding are justified by this test.

## Regression confirmation

A fresh agent tested a coding-agent comparison with no success floor, cost ceiling, or latency SLA. The updated skill:

- returned a usable frontier design;
- listed the missing product constraints in `next_actions`;
- excluded security-failing configurations before Pareto comparison;
- refused to choose one winner or weighted score without explicit policy.
