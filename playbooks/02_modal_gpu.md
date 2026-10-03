# Run a GPU Experiment on Modal

## Overview
Run a training, inference or evaluation job on Modal GPUs (the hackathon gives $150 credit per builder) and bring the result back into the repo as a reviewable PR. Use this for anything heavier than a quick CPU script: fine-tuning a protein language model, a NN speedrun, segmentation or embedding of microscopy images, batched inference with a foundation model.

## What's Needed From User
- Repo with the code or data-loading entry point (ideally already set up with the "Hackathon Repo Setup" playbook).
- Devin secrets `MODAL_TOKEN_ID` and `MODAL_TOKEN_SECRET`.
- The experiment: what to run, the metric, and a budget (GPU type and maximum wall-clock or dollar spend). If none is given, use `L40S` and 30 minutes.
- Data location: HF dataset id, URL, or local files to upload.

## Procedure
1. If `.agents/skills/modal/SKILL.md` exists, load it. Read the relevant pages from `https://modal.com/llms.txt` (append `.md` to doc URLs for plain text) rather than relying on memory, because the Modal API changes often.
2. Authenticate: `modal token set --token-id "$MODAL_TOKEN_ID" --token-secret "$MODAL_TOKEN_SECRET"`, then `modal profile current`.
3. Create a branch: `exp/<short-slug>`.
4. Write `modal_app/<slug>.py`:
   - Define a `modal.Image` with pinned versions (`.uv_pip_install(...)` or `.pip_install(...)`).
   - Use `@app.function(gpu="<TYPE>", timeout=<seconds>, volumes={"/vol": vol}, secrets=[...])`. The default timeout is 5 minutes, so always set it explicitly.
   - Store data and checkpoints on a named Volume (`modal.Volume.from_name("<team>-<slug>", create_if_missing=True)`) and call `vol.commit()` after writing.
   - Pass tokens like `HF_TOKEN` with `modal.Secret.from_dict(...)` or a named Modal secret (`modal secret create`). Never hard-code them.
   - Have a `@app.local_entrypoint()` that runs the job and writes `results/<slug>.json` locally.
5. Do a dry run first: tiny data, a few steps, cheapest GPU (`T4` or `L4`). Fix errors before scaling up.
6. Launch the real run. Use `modal run modal_app/<slug>.py` if it fits within the session; for long jobs use `modal run --detach ...`, note the app id, and check back with `modal app logs <app-id>`.
7. Use `.map()` / `.starmap()` over a list of configs for sweeps or seeds instead of a Python loop. Run at least 3 seeds when a metric is close to a baseline.
8. Pull artifacts back with `modal volume get <vol> <path> results/` (small files only) and write `results/<slug>.json` with the metric(s), config, GPU type, wall-clock, estimated cost and Modal app URL.
9. Write `RESULTS.md` (or a section in it) covering: hypothesis, setup, result vs baseline with uncertainty (std over seeds or a CI), what went wrong, and the next step.
10. Commit code and small result files, push, and open a PR titled `[exp] <slug>: <headline metric>`.

## Specifications
- The PR contains a runnable `modal_app/<slug>.py`, `results/<slug>.json` and an updated `RESULTS.md`.
- The headline number has a stated baseline and an uncertainty estimate (or an explicit note that only one seed was run).
- GPU type, runtime and approximate cost are recorded.
- Validation: the dry run succeeded, and the real run's Modal app shows a completed state (link in the PR).

## Advice and Pointers
- GPU choice: `T4`/`L4` for debugging, `L40S` (48 GB) for most inference and fine-tuning, `A100-80GB`/`H100` only when memory or speed requires it. `H100` may be upgraded to H200 at no extra cost; use `H100!` to pin it when benchmarking speed.
- Cache model weights on the Volume so retries don't re-download them.
- Use `modal shell --volume <vol>` to inspect files interactively.
- If the job will outlive the session, `--detach` is essential; otherwise the app stops when the client disconnects.
- Credits are shared across a team member's runs. Stop stray apps with `modal app stop <app-id>`.

## Forbidden Actions
- Do not run GPU training on the Devin VM.
- Do not launch multi-GPU (`:4`, `:8`) or B200/H200 runs without the user's approval.
- Do not report a metric from the dry run as the result.
