# Hugging Face: Jobs, Hub Artifacts and a ZeroGPU Demo

## Overview
Use the Hugging Face hackathon coupon (about $20 of credit for HF Jobs and ZeroGPU) to run compute jobs on HF infrastructure, publish datasets/models/results to the Hub, and optionally ship a Gradio demo Space for the 2-minute demo. It works well alongside Modal: HF Jobs suit short, script-shaped runs and evals near Hub data, and Spaces give judges a live link.

## What's Needed From User
- Devin secret `HF_TOKEN` (write scope) for an account with the coupon redeemed (huggingface.co/coupons/claim/HuggingFace-AIxScienceHack).
- HF namespace to publish to (username or org).
- Which part is needed: (a) run a job, (b) publish artifacts, (c) build a demo Space. Any combination.

## Procedure
1. Load `.agents/skills/hf-cli/SKILL.md` if present. Otherwise run `hf skills add` and load it. For Spaces also run `hf skills add huggingface-zerogpu` (or `huggingface-spaces`); for fine-tuning, `hf skills add huggingface-llm-trainer`.
2. Verify auth (`hf auth whoami`) and credit (`hf jobs hardware` lists flavours and prices).
3. Jobs:
   - Write a self-contained UV script `jobs/<slug>.py` with PEP 723 inline dependencies.
   - Dry run on `cpu-basic` or `t4-small` with tiny inputs.
   - Launch: `hf jobs uv run --flavor <flavor> --timeout 2h --secrets HF_TOKEN -d jobs/<slug>.py [args]`.
   - Monitor with `hf jobs ps`, `hf jobs logs <id>`, `hf jobs inspect <id>`.
   - The job filesystem is ephemeral. The script must push outputs to the Hub (a dataset repo such as `<ns>/<project>-results`, or a model repo) before exiting.
4. Artifacts: publish processed datasets with `hf upload <ns>/<name> <path> --repo-type dataset` and models with `hf upload <ns>/<name> <path>`. Add a README card covering data provenance and licence (required by the hackathon rules on crediting and consent).
5. Demo Space (ZeroGPU requires a PRO account on the owner; otherwise use CPU hardware or call a Modal endpoint from the Space):
   - Create `space/app.py` (Gradio) and `space/requirements.txt`. Decorate GPU functions with `@spaces.GPU(duration=...)`. Load models at module import, outside the decorated function.
   - Create the Space with `hf repos create <ns>/<space> --repo-type space --space-sdk gradio` and upload with `hf upload <ns>/<space> space/ --repo-type space`.
   - Open the Space URL and confirm it builds and returns a correct output on an example input. Screenshot it.
6. Record job ids, Hub URLs and the Space URL in `RESULTS.md`. Commit the scripts, push, and open a PR.

## Specifications
- Every job used for a reported number has a job id and a Hub location for its outputs.
- Published Hub repos have a README card with source and licence.
- A demo Space, if built, loads and returns a result on an example (screenshot in the PR).
- Validation: `hf jobs inspect <id>` shows COMPLETED for the reported runs, and the Hub URLs resolve.

## Advice and Pointers
- Flavour names differ from Modal: e.g. `t4-small`, `l4x1`, `a10g-small`, `l40sx1`, `a100-large`, `h200`. Check `hf jobs hardware` for current prices.
- The coupon is small, so put long or big training runs on Modal ($150) and use HF Jobs for evals, data processing and moderate fine-tunes.
- ZeroGPU does not support `torch.compile`. Keep `duration` tight to save quota.
- For the Polaron track, large microscopy stacks are better kept on a Modal Volume. Push only derived KPIs/embeddings to the Hub.

## Forbidden Actions
- Do not publish data the team has no right to share (check the track's data terms; Polaron/Serova data may be private). Use private repos when unsure.
- Do not print or upload `HF_TOKEN`.
