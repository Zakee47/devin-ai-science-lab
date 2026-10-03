# Literature Grounding and Paper Reproduction Scoping with Paperclip (GXL)

## Overview
Use Paperclip by GXL (free unlimited credits at the hackathon) to ground a project in the literature: find prior methods and baselines, pull exact numbers and settings from papers, and scope a "reproduce, then push past it" target. Paperclip exposes 11M+ papers (PMC, bioRxiv, medRxiv, arXiv), FDA documents, clinical trials, patents and UniProt/PDB/ChEMBL through a filesystem-style CLI. Its output feeds an experiment plan, which suits the Parallel Lab playbook.

## What's Needed From User
- Devin secret `PAPERCLIP_API_KEY` (create at paperclip.gxl.ai/keys).
- The research question or the paper to reproduce (title, DOI, arXiv id, or a description).
- Optional: how many candidate methods to return (default 5-8).

## Procedure
1. Install the CLI if `paperclip` is missing: `curl -fsSL https://paperclip.gxl.ai/install.sh -o /tmp/pc.sh && bash /tmp/pc.sh </dev/null`. The browser sign-in step fails without a TTY; that is expected. Export `PAPERCLIP_API_KEY` for every `paperclip` command.
2. Load `.agents/skills/paperclip/SKILL.md`, or run `paperclip skill` and read it. It is the authoritative command guide.
3. Search broadly with an explicit source, e.g. `paperclip search -s pmc,biorxiv,arxiv "<question>" -n 30`. Note the returned search id (`s_...`).
4. Use `paperclip map --from <s_id> "<extraction question>"` to read results in parallel and extract the same fields from each: method, dataset, metric and value, compute, code link.
5. Deep-read the 2-3 most relevant papers: `paperclip cat /papers/<id>/meta.json`, `paperclip grep -i "<term>" /papers/<id>/content.lines`, and the figures/supplements for exact hyperparameters.
6. For protein/drug tracks, pull structured data with `paperclip sql` or `/proteins/` (UniProt, PDB, ChEMBL), e.g. HLA allele sequences or known binders.
7. Write `docs/literature.md` containing: a comparison table (method, data, metric, reported value, code available, paper path/DOI); the reproduction target (one table/figure, its exact number, and the settings needed); and 5+ concrete "push past it" ideas, each with a hypothesis and a cheap first experiment.
8. Commit, push and open a PR titled "Literature map: <topic>".

## Specifications
- Every number in `docs/literature.md` cites its source (DOI or Paperclip path) and the section/table it came from.
- The reproduction target names a specific table or figure, a numeric value, and the dataset/split.
- The ideas list is directly usable as input to the Parallel Lab playbook (one line per idea, with a metric).
- Validation: spot-check the two most important numbers by grepping the source paper's `content.lines`.

## Advice and Pointers
- `search` fails without `-s`. Common sources: `pmc`, `biorxiv`, `medrxiv`, `arxiv`, `abstracts`, `trials`, `fda`, `proteins`.
- Use `paperclip bash '<pipeline>'` for pipes on the server side.
- `paperclip git init <topic>` creates a sticky paper collection, which helps keep a team's reading list in one place.
- Track hints: C3 maths, search arXiv for Tao et al. problem lists and speedrun records; Serova, NetMHCstabpan and peptide-HLA stability benchmarks; Polaron, electrode microstructure characterisation and SEM-based QC; Originator, reward hacking and calibration evals for science agents.

## Forbidden Actions
- Do not invent citations or numbers. If Paperclip can't find it, say so.
- Do not paste API keys into commands that get logged in committed files.
