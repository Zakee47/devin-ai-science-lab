# Parallel Lab: Devin playbooks for the London AI x Science Hackathon

**Landing page: https://zakee47.github.io/devin-ai-science-lab/**

These playbooks let your team run Devin as a research lab. Five or more Devin sessions run in parallel, each tests a different idea for your track problem, and each one ends in a PR with a measured result. GPU work runs on Modal.

## Install everything in one click

Open the landing page and click **Add all 7 playbooks to Devin**. A Devin session starts and saves every playbook from [`playbooks/manifest.json`](playbooks/manifest.json) into your org, macros included.

To add a single playbook instead, use that playbook's **Create in Devin** button on the landing page.

## Playbooks

| Macro | Playbook | Use it to |
|---|---|---|
| `!parallel_lab` | [Run a Parallel Lab](playbooks/06_parallel_lab.md) | Build a shared eval and baseline, launch 5+ child Devins, then open a leaderboard PR |
| `!lab_experiment` | [Parallel Lab Experiment](playbooks/07_lab_experiment.md) | Run a single child experiment: one idea, one branch, one PR |
| `!hack_setup` | [Hackathon Repo Setup](playbooks/01_hack_setup.md) | Install the sponsors' agent skills into `.agents/skills/` and smoke-test each tool |
| `!modal_gpu` | [GPU Experiment on Modal](playbooks/02_modal_gpu.md) | Run GPU jobs on Modal and bring results back as a PR |
| `!hf_jobs` | [Hugging Face Jobs + ZeroGPU](playbooks/03_hf_jobs.md) | Run HF Jobs, publish to the Hub and build a Gradio demo Space |
| `!paperclip_lit` | [Paperclip literature map](playbooks/04_paperclip.md) | Map the literature, find baselines and scope a reproduction target |
| `!amass_evidence` | [Amass biomedical evidence](playbooks/05_amass.md) | Collect cited papers, trials, drugs and targets |

Idea starters for each track: [docs/track-ideas.md](docs/track-ideas.md)

## Secrets (Devin Settings, then Secrets)

Child sessions can only use secrets saved in Devin, so add these before you launch parallel sessions. Secrets pasted into chat won't reach them.

- `MODAL_TOKEN_ID` and `MODAL_TOKEN_SECRET`: required.
- `HF_TOKEN`, `PAPERCLIP_API_KEY`, `AMASS_API_KEY`: optional, add the ones your ideas use.
