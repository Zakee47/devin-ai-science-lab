# Parallel Lab idea starters by track

Use these as the 5+ ideas for `!parallel_lab`, or as a starting point for your own. Ideas should differ in kind, not just in hyperparameters.

## Polaron materials QC
- Suggested metric: batch accept/investigate/reject accuracy on held-out batches
- Ideas: classical microstructure KPIs (porosity, particle size distribution, tortuosity), self-supervised SEM embeddings with a distribution-shift test, anomaly detection vs the baseline batch, conformal accept/investigate/reject thresholds, and per-KPI attribution for the explanation.

## Serova peptide-HLA
- Suggested metric: Spearman correlation with measured peptide-HLA stability
- Ideas: a NetMHCstabpan-style baseline, ESM-2 embeddings with a regressor, fine-tuned protein LM with LoRA on Modal, structure-based features (predicted complex), and allele-held-out generalisation analysis.

## C3 maths/speedrun
- Suggested metric: wall-clock time to the target loss on a fixed GPU type
- Ideas: optimiser swap, architecture tweak, data ordering/curriculum, mixed precision/compile, and LR schedule search, all scored by time-to-target on the same GPU type (`H100!` to avoid auto-upgrade skew).

## Originator
- Suggested metric: detection precision/recall on planted reward hacks
- Ideas: one reward-hack detector per hacking pattern, a calibration probe, a falsification-test generator, and an eval harness ablation.

