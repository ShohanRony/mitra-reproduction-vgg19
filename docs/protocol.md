# What was reproduced, and what differs

This is an independent reconstruction of the weight-pruning experiment in
Mitra, Schwalbe & Klein (2024), *"Investigating Calibration and Corruption
Robustness of Post-hoc Pruned Perception CNNs: An Image Classification
Benchmark Study"* (CVPR Workshops 2024) — see `docs/sources.md` for the exact
text accessed — not a run of the authors' own code or checkpoints. The
authors have not (as far as this search could determine) published their
experiment code; see `docs/sources.md` item 6.

## What matches the paper's stated protocol

- Architecture: VGG-19-BN (16 conv layers), from the third-party
  `Eric-mingjie/rethinking-network-pruning` repository's `cifar/weight-level`
  implementation (see `docs/paper_setup.md` and the upstream README it
  points to), not torchvision's ImageNet VGG.
- Data: CIFAR-10 (50,000 train / 10,000 test), CIFAR-10-C, all 15 standard
  corruptions, all 5 severities.
- Pruning: global unstructured L1 magnitude pruning on convolution weights
  only, target fraction 0.3 (see `results/seed*/pruning.json`).
- Training: SGD, momentum 0.9, weight decay 1e-4, batch size 64; baseline 160
  epochs with LR 0.1 (epochs 1-79) / 0.01 (80-119) / 0.001 (120-160); recovery
  40 epochs at constant LR 0.001 on clean training data.
  Note on LR boundary indexing: `docs/assumptions.md` flags that this is a
  one-based boundary (epoch 80 is the first at 0.01); a zero-based
  implementation could shift the schedule by one epoch and was not ruled out
  as the author convention.
- Evaluation: top-label ECE, both equal-mass (10 bins, primary) and
  equal-width (10 bins, secondary) estimators; mean-per-corruption accuracy
  (mPC) by severity, not normalized mCE, matching the paper's stated metric.
- Repetitions: 3 independently trained seeds (1, 2, 3), each with fresh
  weight init, data order, and augmentation randomness; mean ± SD reported.

## What is an explicit reconstruction decision, not a verified author choice

See `docs/assumptions.md` for the full list — normalization constants, crop
padding, flip probability, Nesterov momentum, checkpoint-selection
(last-epoch vs best), optimizer-reset-on-prune, and the equal-mass tie
handling were all not independently specified in the accessible paper text
and were decided by this reconstruction, not copied from author code.

## Known gap: no matched-training dense control

The baseline arm trains for 160 epochs; the pruned arm trains for
160 (pretrain) + 40 (recovery) = 200 total epochs. There is no dense
(unpruned) control trained for the same 200-epoch budget, so the ECE-
degradation finding in `results/cross_seed_report.md` cannot currently
distinguish "pruning degrades calibration" from "200 epochs of training
degrades calibration relative to 160, regardless of pruning." This is listed
as planned future work, not resolved here.

## Statistical framing

3 seeds is the paper's own stated minimum for a mean ± SD design. It is not a
well-powered sample. No p-value is reported anywhere in this repository for
that reason; `results/cross_seed_report.md` reports paired t-statistics
alongside the n=3 caveat explicitly, and treats the one finding that is
directionally consistent across all 6 clean+severity comparisons (pruned ECE
higher than baseline ECE) as suggestive, not confirmed.
