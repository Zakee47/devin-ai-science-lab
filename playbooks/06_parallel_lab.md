# Run a Parallel Lab (Best Use of Devin challenge)

## Overview
Run the team's research like a lab group: this session acts as lab lead and launches 5 or more managed Devins in parallel, each testing a different idea for the track problem (a model, a method, an optimisation, an analysis). Each child ends in its own branch and PR with a result, and GPU work runs on Modal. The lead then compiles a leaderboard PR so the team can spend its time on direction and judgment. It meets the Cognition side challenge rules: at least 5 parallel sessions, each ending in a PR with a result, with GPU work on Modal.

## What's Needed From User
- Team repo, ideally already set up with the "Hackathon Repo Setup" playbook so sponsor skills are in `.agents/skills/`.
- Track and problem statement, plus the data location.
- The primary metric and how it is computed.
- 5 or more ideas, one line each. If the team gives fewer, propose the rest (optionally using the Paperclip literature playbook) and get them confirmed.
- Devin org secrets `MODAL_TOKEN_ID` and `MODAL_TOKEN_SECRET`, plus any others the ideas need (`HF_TOKEN`, `PAPERCLIP_API_KEY`, `AMASS_API_KEY`). Child sessions only see secrets saved in Devin, not ones pasted into chat.
- Optional: ACU limit per child (default 15) and a wall-clock deadline (default 3 hours).

## Procedure
1. Check prerequisites: confirm each needed secret exists and run `modal token set ... && modal profile current`. If Modal auth fails, stop and ask; children would fail the same way.
2. Build the shared harness on `main` before any fan-out, because comparable results matter more than any single result:
   - `eval/evaluate.py`: a fixed data split (seeded, saved in `eval/splits.json`), the primary metric, and the uncertainty method (bootstrap CI or std over 3 seeds).
   - `results/SCHEMA.md` and `results/baseline.json`: the simplest reasonable baseline, run once.
   - `lab/IDEAS.md`: a table of slug, hypothesis, method, GPU needed (y/n) and expected signal.
   Push to `main` (or merge a quick PR) so every child branches from the same harness.
3. Start one managed Devin per idea, all at once, with the "Parallel Lab Experiment" playbook attached. Look up its playbook id by listing the org's playbooks and finding macro `!lab_experiment`; if it is missing, fall back to pasting that playbook's steps into each prompt. Give each child:
   - the repo, branch `lab/<slug>`, idea text, metric, baseline value and deadline;
   - an ACU limit and the tags `parallel-lab` and `lab-<slug>`;
   - the rule: "GPU work runs on Modal via `modal_app/<slug>.py`; evaluate only with `eval/evaluate.py`; write `results/<slug>.json` in the schema; open a PR titled `[lab] <slug>: <headline>`".
4. Share the session links with the team right away. The team spends its time on direction while the children run.
5. Monitor the children together (wait for all, rather than polling each one). Answer a child's questions quickly. If a child is stuck or blows past its budget, message it once with a correction, then put it to sleep or terminate it and record why.
6. When a child's PR arrives, check it: the metric came from `eval/evaluate.py` on the frozen split, the Modal run link is present, and the uncertainty is reported. Send back any PR that changed the split or the metric.
7. Optionally run a second round: take the top 1-2 ideas, plus one combination of winners, and launch another round of children.
8. Compile `lab/LEADERBOARD.md` on branch `lab/leaderboard`: one row per idea (slug, PR link, metric ± uncertainty, delta vs baseline, GPU-hours/cost, verdict "promote / inconclusive / negative"), a short "what we learned" section that includes negative results, and a plot `lab/leaderboard.png`. Open a PR.
9. Report to the user with the leaderboard PR, all child PR links, the session links and total Modal spend.

## Specifications
- At least 5 child sessions ran in parallel, each tied to a distinct idea, branch and PR.
- Every child PR contains `results/<slug>.json` that follows the schema, an updated `RESULTS.md` section, and (for GPU ideas) a Modal app link.
- Every result is measured with the same `eval/evaluate.py` and split as the baseline.
- The leaderboard PR links every child PR and reports uncertainty, not just point estimates.
- Validation: re-run `eval/evaluate.py` on the best idea's saved predictions and confirm it matches the reported number.

## Advice and Pointers
- Good ideas differ in kind, not just hyperparameters. Mix a strong baseline tweak, a foundation-model approach, a data-centric idea, an uncertainty/calibration method and an analysis/interpretability idea.
- Example ideas for tracks 1-4: https://raw.githubusercontent.com/Zakee47/devin-ai-science-lab/main/docs/track-ideas.md
- Keep child prompts self-contained. Children can't see this session's files, only what is pushed to git.
- If Modal capacity is slow, cheap ideas can drop to `L4`/`L40S`; record the GPU type in results so wall-clock comparisons stay fair.

## Forbidden Actions
- Do not let children modify `eval/`, `eval/splits.json` or `results/baseline.json`.
- Do not merge child PRs into `main` without the team's approval. The leaderboard PR only links to them.
- Do not run GPU workloads on Devin VMs.
- Do not drop negative or inconclusive results from the leaderboard.
