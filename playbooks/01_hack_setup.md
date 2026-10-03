# Hackathon Repo Setup: Sponsor Tools + Agent Skills

## Overview
Get a team repo ready for the London AI x Science Hackathon so that every Devin session working on it, including parallel child sessions, can use the sponsor tools (Modal, Hugging Face, Paperclip by GXL, Amass) without setting them up again. Several sponsors ship official agent skills (`SKILL.md`). Devin reads skills committed under `.agents/skills/`, so installing them once in the repo is enough for every future session.

## What's Needed From User
- GitHub repo for the project (new and empty is fine; the rules say no commits before the event).
- Track (1 C3 Open Maths, 2 Originator, 3 Serova peptide-HLA, 4 Polaron materials QC) and a one-line problem statement.
- Devin secrets for the tools the team will use (Settings > Secrets, saved org-wide so child sessions inherit them):
  - Modal: `MODAL_TOKEN_ID`, `MODAL_TOKEN_SECRET`. Create at modal.com/settings/tokens after redeeming the $150 credit.
  - Hugging Face: `HF_TOKEN` with write scope, from huggingface.co/settings/tokens, after redeeming the hackathon coupon.
  - Paperclip: `PAPERCLIP_API_KEY` (`gxl_...`), from paperclip.gxl.ai/keys.
  - Amass: `AMASS_API_KEY` (`amass_...`), from platform.amass.tech > API Keys, after team onboarding at amass.tech/onboarding/london-ai-science-hackathon.
- Skip any tool the team won't use. Do not add DeepMind/Antigravity or Anthropic credentials here; those are used outside Devin.

## Procedure
1. Clone the repo. Create the branch `setup/sponsor-skills`.
2. Install the Python CLIs: `pip install -U modal "huggingface_hub[cli]"`. Use `uv` if the repo already uses it.
3. Install the official agent skills into the repo (all of them write to `.agents/skills/`):
   - Modal: `modal skills install -y --no-docs`. Drop `--no-docs` only if the team wants the offline docs mirror committed.
   - Hugging Face: `hf skills add` (installs `hf-cli`). Then add the workflow skills the track needs, e.g. `hf skills add huggingface-zerogpu`, `hf skills add huggingface-llm-trainer`, `hf skills add huggingface-datasets`, `hf skills add huggingface-trackio`. `hf skills list` shows the full list.
   - Amass: `npx -y skills add amass-technologies/public-skills --skill amass-api -a codex -y --copy` (`-a codex` targets `.agents/skills/`; `'*'` would write into ~40 agent folders)
   - Paperclip: install the CLI with `curl -fsSL https://paperclip.gxl.ai/install.sh -o /tmp/pc.sh && bash /tmp/pc.sh </dev/null`. The sign-in step needs a TTY and is expected to fail here; the API key replaces it. Then write the skill: `mkdir -p .agents/skills/paperclip && paperclip skill > .agents/skills/paperclip/SKILL.md`
4. Delete any `.claude/skills` symlinks the installers created, unless the team also uses Claude Code.
5. Run a smoke check for each tool whose secret exists, binding the secret to the command:
   - `modal token set --token-id "$MODAL_TOKEN_ID" --token-secret "$MODAL_TOKEN_SECRET" && modal profile current`
   - `hf auth whoami`
   - `paperclip search -s arxiv "neural network training speedrun" -n 3`
   - `curl -s "https://api.amass.tech/api/v1/cores/biomedcore/records?query=peptide%20HLA&limit=2" -H "Authorization: Bearer $AMASS_API_KEY" | head -c 400`
6. Write `AGENTS.md` at the repo root (under 2 KB) covering: the track and problem statement, which sponsor tools are available and which skill covers each, the rule "GPU work runs on Modal (or HF Jobs), never on the Devin VM", and where results go (`results/`, `RESULTS.md`).
7. Add `.gitignore` entries for data caches and checkpoints (`data/raw/`, `*.pt`, `*.ckpt`, `wandb/`, `.modal/`).
8. Commit, push and open a PR titled "Set up sponsor agent skills". The PR description should include a table of tool / skill path / smoke-check result.

## Specifications
- `.agents/skills/` contains a `SKILL.md` for every tool the team enabled.
- Every enabled tool's smoke check passes, or the PR clearly lists the missing secret.
- No secret values appear in any committed file, log excerpt or PR text.
- `AGENTS.md` exists and is short.

## Advice and Pointers
- The Modal and HF skills tell the agent to read live docs (`https://modal.com/llms.txt`, `hf <cmd> --help`), so these installs are small and stay current.
- `hf skills add` with no name installs only `hf-cli`; workflow skills must be named explicitly.
- HF Jobs only need a positive credit balance (the coupon counts). Hosting a ZeroGPU Space requires a PRO account. Calling an existing ZeroGPU Space works on free accounts, with a small daily quota.
- Paperclip's `search` requires `-s <source>`, e.g. `-s pmc,biorxiv,arxiv`.
- Paperclip and Amass also offer remote MCP servers (`https://paperclip.gxl.ai/mcp` with an `X-API-Key` header; `https://mcp.amass.tech/mcp` with OAuth). An org admin can add them under Settings > MCP Marketplace > Add custom MCP. The CLI and API paths above work without that.

## Forbidden Actions
- Do not commit secrets or write them into `AGENTS.md` or skill files.
- Do not store DeepMind or Anthropic credentials in Devin.
- Do not commit large datasets or model weights; use a Modal Volume or a HF dataset/model repo.
