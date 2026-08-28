# UVVR Engineering Rules

These instructions apply to coding agents working anywhere in this repository.

## Product boundaries

- This repository develops an instruction-first Codex skill for designing defensible evals for AI systems whose work is partly unverifiable.
- UVVR must distinguish deterministic or executable verification, grounded or anchored verification, calibrated semantic proxies, and human or delayed-outcome judgment.
- Preserve the V4–V0 terminology defined in the research artifacts. Never implement behavior that labels an LLM score, rubric score, majority vote, or learned reward model as ground truth.
- Preserve an explicit unverifiable remainder. Do not force every criterion into an automated score.
- Keep intended outcomes separate from forbidden outcomes. Safety, permission, privacy, and evidence-integrity failures are gates.
- The first product wedge is eval design for research agents. Customer support is the first out-of-domain contrast, not a second MVP domain.
- The first milestone is one instruction-only skill plus real examples and behavioral evidence. Do not add hooks, persistent modes, MCP, dashboards, hosted services, or multi-host adapters without an observed need.

## Required coding workflow

1. Restate the requested repository change and explicit non-goals.
2. Inspect `AGENTS.md`, the relevant files, referenced artifacts, and every consumer before editing.
3. Draw the smallest affected file flow and identify trust boundaries or hidden evaluation assets.
4. Define observable behavior, inputs, outputs, error cases, and file ownership before implementation.
5. For behavior changes, add or update the smallest test or realistic fixture that would fail without the change.
6. Implement the smallest change that makes the observable requirement pass.
7. Refactor only demonstrated duplication or measured complexity in the changed path.
8. Run every applicable validation command and inspect generated or modified artifacts.
9. Review the final diff for scope creep, duplicated guidance, leaked holdouts, secrets, and unrelated changes.
10. Report exact local verification evidence. Never describe a local command as CI evidence.

## Repository architecture

- `AGENTS.md` contains repository engineering rules for coding agents. It must not duplicate the future UVVR runtime workflow.
- `unverifiable-to-verifiable-rewards-research.md` is the detailed research source.
- `design/uvvr-core-contract.md` is the detailed domain-neutral design record behind the runtime skill.
- `skills/uvvr/SKILL.md` is the source of truth for UVVR invocation behavior.
- Put substantial conditional guidance in focused `skills/uvvr/references/` files only when the core skill needs progressive disclosure.
- Put deterministic helpers in `skills/uvvr/scripts/` only when repeated mechanics justify executable code.
- Keep public teaching examples separate from hidden benchmark and pilot cases.

## Skill implementation rules

- Follow the installed `skill-creator` instructions when creating or changing a skill.
- Assume Codex already understands general reasoning, research, and software practices. Include only UVVR-specific decisions, invariants, and failure modes.
- Keep `SKILL.md` compact and decision-oriented. Do not inline the full research report or copy the verifier checklist repeatedly.
- Use concise routing to focused references instead of loading every domain, transformation, and paper on every invocation.
- Keep automatic discovery precise. The description must identify when UVVR applies and avoid attracting ordinary testing, generic research, or unrelated RL questions.
- Do not create placeholder directories, manifests, commands, agents, assets, or scripts for hypothetical future use.
- Do not duplicate skill rules into `AGENTS.md`, README files, or host-specific mirrors.
- Add companion skills only after repeated dogfood or pilot sessions demonstrate a distinct trigger, input, and output contract.
- Package for Codex only after the instruction-only skill passes dogfood, benchmark, and external pilot gates.

## Product invariants to test

When changing UVVR behavior, preserve these observable invariants:

- Every evaluation criterion receives a V4, V3, V2, V1, or V0 classification.
- Every V0–V2 criterion names its evidence and blind spot.
- Hard gates remain separate from partial-credit rewards.
- The proposed verifier states what object it checks, what evidence it consumes, what remains off-screen, and what a pass establishes.
- The output includes known-good and plausible-bad verifier tests.
- Valid alternatives, adversarial cases, high-reward-tail audits, and original-task holdouts are considered where relevant.
- Proxy-task gains are not reported as original-task gains without independent evaluation.
- Policy improvement is separated from test-time search or system-scaffolding gains.
- The result reports verified, anchored, proxy, and human-only coverage separately.
- Keep configuration Pareto optimization out of the UVVR runtime skill; it belongs in a separate comparison skill.

## Dogfood and benchmark implementation

- Start with six cases: three research-agent tasks, one V0-heavy or impossible research task, one customer-support out-of-domain contrast, and one safety-sensitive research task.
- Blind review baseline and UVVR outputs with shuffled anonymous IDs and a versioned review checklist.
- Freeze the control prompt and ensure it contains no UVVR-specific vocabulary.
- Run treatment and control in fresh isolated workspaces with identical tools and permissions.
- Keep hidden references and holdouts outside the evaluated agent's readable paths.
- Keep graders, logs, benchmark assets, and reward calculations read-only to the evaluated agent.
- Run deterministic checks before semantic judges.
- Use a different judge model family from the evaluated model when practical and report disagreement.
- Require good-reference-pass and plausible-bad-fail self-tests before model spend.
- Preserve raw outputs for offline rescoring when graders change.
- Track completion rate, retries, pass@1, pass@3, cost per attempt, cost per successful task, latency p50/p95, uncertainty, and verifier disagreement.
- Compare only hard-policy-eligible configurations on the `(maximize success, minimize cost, minimize latency)` Pareto frontier.
- Do not scalarize success, cost, and latency unless the user supplies weights or an explicit product policy.
- The benchmark must be capable of falsifying UVVR's value and must publish limitations and negative results.

## Complexity and maintainability

- Prefer plain Markdown and standard-library validation before adding dependencies or runtime infrastructure.
- Do not add factories, registries, plugin layers, configuration, or abstractions for hypothetical hosts or future eval types.
- Keep one source of truth for each rule, schema, example, and benchmark label.
- Use small named files with one clear purpose. Avoid catch-all “utils,” “core,” or “framework” modules.
- Comments explain non-obvious constraints, trust boundaries, and reasons rather than restating mechanics.
- Do not reorganize the repository while implementing an unrelated behavior change.

## Tests

- Test observable behavior and decision invariants, not exact generated prose or heading order.
- Use realistic tasks with incomplete evidence, conflicting criteria, valid alternatives, and adversarial outputs.
- Include at least one case where the correct result preserves a V0 unverifiable remainder.
- Include at least one case where a proposed transformation has a perfect proxy label but weak original-task fidelity.
- Verify grader self-tests before running an expensive treatment/control matrix.
- Keep public examples, dogfood cases, hidden benchmark cases, and external pilot tasks as separate datasets.
- Treat benchmark contamination, control-treatment leakage, and writable evaluator assets as test failures.

## Verification commands

Run only commands whose artifacts exist at the current project stage.

```bash
# Current repository contract and research artifacts
test -f AGENTS.md
test -f unverifiable-to-verifiable-rewards-research.md
# Skill validation, once the skill exists
python3 ~/.codex/skills/.system/skill-creator/scripts/quick_validate.py skills/uvvr

# Plugin validation, only after a Codex plugin is intentionally added
python3 ~/.codex/skills/.system/plugin-creator/scripts/validate_plugin.py .
```

Never report a command that was not run, and never describe local verification as CI evidence.

## Repository hygiene

- Preserve unrelated user changes and local research artifacts.
- Do not commit credentials, private production data, hidden benchmark labels, caches, temporary model runs, or generated reports containing sensitive inputs.
- Do not expose hidden benchmark or pilot cases through public examples, skill references, logs, or prompts.
- Version task data, parsers, verifiers, rubrics, environments, thresholds, and reports together when they jointly define a result.
- Use one behavioral or architectural change per reviewable unit.
- Do not move the existing research report or HTML study files unless repository reorganization is explicitly requested.
