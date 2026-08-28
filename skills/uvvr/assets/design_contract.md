# UVVR Design Contract Template

> UVVR_ADAPT every marked section for the confirmed system. Remove all
> `UVVR_ADAPT` markers before delivering the generated eval package.

## System and decision

UVVR_ADAPT: name the evaluated system, the exact object/run being evaluated,
the downstream decision, and the human owner.

## Claim transformation

```text
original_claim: UVVR_ADAPT
current_reward: UVVR_ADAPT
why_unverifiable: UVVR_ADAPT
evaluable_claim: UVVR_ADAPT
capability_lost: UVVR_ADAPT
transfer_check: UVVR_ADAPT
```

## Criterion map

UVVR_ADAPT: give every atomic criterion exactly one level and one role. Put
supporting anchors in Evidence instead of inventing composite levels.

| Criterion | Level | Evidence | Verifier / what pass establishes | Role | Blind spot |
|---|---|---|---|---|---|
| UVVR_ADAPT | V4/V3/V2/V1 | UVVR_ADAPT | UVVR_ADAPT | block/score/audit/escalate | UVVR_ADAPT |

## Decision policy

- `accept`: every `block` criterion passes and nothing requires escalation.
- `reject`: sufficient evidence shows an explicit `block` criterion failed.
- `escalate`: an `escalate` criterion fails or remains semantically uncertain.
- evaluator `error`: required input/evidence is incomplete, malformed, or the
  judge failed. The runbook must provide an exact collection/retry path; an
  evaluator error is not a product decision.

There is no `no_decision`. `evaluate.py` must reject incomplete collected input
before evaluation and return the missing-field checklist.

## Evidence authority and trust boundaries

UVVR_ADAPT: state which observed state, artifact, trace, reference, or human
label wins when sources conflict. Identify which evidence is candidate-controlled,
which assets are hidden/read-only, and what the judge may receive.

Do not treat a model verdict, rubric, vote, or reward model as ground truth.
Only sufficiently evidenced `block` failure can reject.

## Public examples

`evaluate.py` embeds four public self-tests:

- known-good → `accept`;
- plausible-bad → `reject`;
- valid-alternative → `accept`;
- reward-hack with contrary grounded evidence → `reject`.

UVVR_ADAPT: replace each with concrete domain input/output/evidence while keeping
expected decisions private from the evaluated agent.

## Human checklist and unverifiable remainder

UVVR_ADAPT: list the original-task outcomes that no current verifier establishes,
who inspects them, and what evidence that human needs. Mirror this list in
`evaluate.py` as `HUMAN_CHECKLIST`; never assign it a fabricated numeric score.

## Versioned components

UVVR_ADAPT: record task/input version, environment, deterministic check version,
judge command/model/rubric, reference pool or design contract version, and the
date this decision policy was approved.
