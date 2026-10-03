# Parallel Lab Experiment (child session)

## Overview
Run one experiment as one member of a Parallel Lab: test a single idea against the team's frozen evaluation harness, run GPU work on Modal, and finish with a branch and PR that holds a comparable, honest result. The lab lead session normally starts this playbook and supplies the inputs below.

## What's Needed From User
- Repo, idea slug and branch name (`lab/<slug>`).
- Idea description and hypothesis.
- Primary metric, baseline value (from `results/baseline.json`) and deadline/ACU budget.
- Devin secrets `MODAL_TOKEN_ID`/`MODAL_TOKEN_SECRET` (plus any tool keys the idea needs).

## Procedure
1. Clone the repo, check out `main`, then create `lab/<slug>`. Read `AGENTS.md`, `results/SCHEMA.md`, `eval/evaluate.py` and any skill in `.agents/skills/` relevant to the idea (e.g. `modal`, `hf-cli`, `paperclip`).
2. Write a 5-line plan at the top of `lab/<slug>/NOTES.md`: hypothesis, change, expected effect, compute plan, stop condition.
3. Implement the idea in `lab/<slug>/` (and `modal_app/<slug>.py` for GPU work). Keep changes out of `eval/` and the shared split.
4. Smoke-test on a tiny subset (Modal `T4`/`L4` for GPU code) so a full run won't crash.
5. Run the full experiment on Modal: explicit `gpu=` and `timeout=`, a named Volume for data/checkpoints, `--detach` for anything over ~20 min, and 3 seeds when the expected effect is small.
6. Score only with `eval/evaluate.py` on the frozen split. Save predictions to the Volume or `lab/<slug>/preds.*` (if small).
7. Write `results/<slug>.json` per the schema (metric, uncertainty, seeds, GPU type, wall-clock, estimated cost, Modal app URL, commit SHA). Append a section to `RESULTS.md`: result vs baseline, uncertainty, verdict (promote / inconclusive / negative), and one next step.
8. Push and open a PR titled `[lab] <slug>: <metric> vs baseline <value>`. Its description should hold the hypothesis, result table, Modal link and caveats. Report the PR link to the lab lead (parent session).

## Specifications
- One idea, one branch, one PR.
- The reported metric comes from `eval/evaluate.py` on the frozen split, with uncertainty or an explicit single-run caveat.
- `results/<slug>.json` validates against `results/SCHEMA.md`.
- Validation: a fresh `python eval/evaluate.py --preds <saved preds>` reproduces the reported number.

## Advice and Pointers
- A clean negative result is a valid outcome. Report it and stop instead of tuning until the deadline.
- If the idea is unworkable as specified, message the parent with a one-line alternative before changing scope.
- Stop when the stop condition hits or about 80% of the budget is used, then write up what you have.

## Forbidden Actions
- Do not edit `eval/`, `eval/splits.json` or `results/baseline.json`, and do not touch other ideas' directories.
- Do not train on the evaluation split or tune on held-out data.
- Do not merge into `main`.
- Do not run GPU work on the Devin VM.
