# Biomedical Evidence and Data with Amass

## Overview
Use Amass (the hackathon gives $500 of API credit per team) to pull cited, structured biomedical evidence: publications (BiomedCore), clinical trials (TrialCore), drugs and molecules (DrugCore), FDA/EMA authorisations (RegulatoryCore), and genes/targets with druggability and safety signals (GeneCore). It suits Track 3 (peptide-HLA and immunotherapy context), Track 2 (building grounded evals and benchmarks for bio agents), and any project that needs traceable citations in its demo.

## What's Needed From User
- Devin secret `AMASS_API_KEY` (`amass_...`, from platform.amass.tech > API Keys, after team onboarding at amass.tech/onboarding/london-ai-science-hackathon).
- The question or dataset to build (e.g. "HLA-A*02:01 neoantigen trials and their papers", "a benchmark of 200 drug-target facts with citations").

## Procedure
1. Load `.agents/skills/amass-api/SKILL.md` if present. Otherwise install it with `npx -y skills add amass-technologies/public-skills --skill amass-api -a codex -y --copy`. Also read the LLM quick reference: https://platform.amass.tech/documentation/for-ai-agents/llm-quick-reference
2. Smoke-test auth: `curl -s "https://api.amass.tech/api/v1/cores/biomedcore/records?query=<q>&limit=2" -H "Authorization: Bearer $AMASS_API_KEY"`.
3. Write a small client `tools/amass.py` with: bearer auth, reading from the `data` key, retry and exponential backoff on 429 using `Retry-After`, and on-disk JSON caching per request, so parallel sessions and reruns don't burn credit.
4. Pick Cores to match the question. Search broadly, then fetch details by Amass ID. Convert external ids (PMID/DOI/NCT/ChEMBL) through the `lookup` endpoints first.
5. Follow cross-core links (e.g. a trial's `referencesBiomedCore`) instead of matching records by title.
6. Save outputs to `data/amass/<slug>.jsonl` (one record per line with `amassId`, source ids and the fields used) and a human-readable `docs/evidence_<slug>.md` where every claim cites its Amass ID and PMID/NCT.
7. Commit the client, the data (if small and allowed) and the docs. Push and open a PR.

## Specifications
- Every fact used downstream links to an Amass ID plus a public id (PMID/DOI/NCT).
- The client respects the 60 requests/60 s limit and caches responses.
- Validation: re-fetch 3 random records by Amass ID and confirm the stored fields match.

## Advice and Pointers
- Responses are wrapped in `{"data": ...}`; errors use `{"error": {...}}`.
- Don't request `include=fulltext` unless needed; it makes responses much larger.
- The rate limit is per user+org, not per key, so parallel Devins share it. Use one cached client.
- Batch lookups can fail item by item. Check each item for `error`.
- `amass-biomedical-evidence-scout` and the other showcase skills run over the Amass MCP (OAuth). Use them only if an admin has connected `https://mcp.amass.tech/mcp` to Devin.

## Forbidden Actions
- Do not fabricate PMIDs, NCT ids or trial-paper links. Use only what the API returns.
- Do not commit `AMASS_API_KEY` or raw full-text dumps.
