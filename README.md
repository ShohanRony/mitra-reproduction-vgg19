# Mitra, Schwalbe & Klein (2024) VGG-19-BN pruning reconstruction

**This is an independent reconstruction built from the paper's text, not the
authors' released code.** Mitra, Pallavi, Gesina Schwalbe, and Nadja Klein,
*"Investigating Calibration and Corruption Robustness of Post-hoc Pruned
Perception CNNs: An Image Classification Benchmark Study"* (CVPR Workshops,
2024). [arXiv](https://arxiv.org/abs/2405.20876). No author code for this
experiment was found during the source audit in `docs/sources.md`.

## What this is, in plain terms

- VGG-19-BN trained on CIFAR-10, then pruned 30% (global unstructured L1 on
  convolution weights) and recovery-trained, run for **3 independent seeds**.
- Evaluated on clean CIFAR-10 test data and all 15 CIFAR-10-C corruption
  types at all 5 severities.
- **3 seeds is a pilot, not a confirmatory study.** No p-value is reported
  anywhere in this repository. The one finding that is directionally
  consistent across all 3 seeds and all 6 clean+severity conditions
  (pruning makes calibration slightly worse) is reported as suggestive, not
  proven — see `results/cross_seed_report.md` and `VERIFICATION.md`.
- **The baseline model trains for 160 epochs; the pruned model trains for
  160 + 40 = 200 epochs.** There is no matched-training dense control run
  for 200 epochs without pruning, so this reconstruction cannot yet rule out
  "more total training" as the cause of the small calibration difference
  instead of pruning itself. That control is listed as planned future work
  in `docs/protocol.md`, not run here.
- **AI coding assistants were used to write code in this repository.** Every
  reported number was checked against the raw log/metric files it came
  from — see `VERIFICATION.md` for the line-by-line check.
- **Datasets and model checkpoints are not included** in this repository
  (see "Datasets and checkpoints" below). SHA-256 hashes are provided so
  anyone can verify they're using the same files.

## Results summary (3 seeds, mean ± SD)

| Metric | Baseline | Pruned 30% + recovery | Plain difference |
|---|---|---|---|
| Clean accuracy | 0.9372 ± 0.0010 | 0.9375 ± 0.0011 | +0.0003 (not consistent in sign across seeds — no detectable effect) |
| Clean ECE (equal-mass) | 0.0472 ± 0.0012 | 0.0480 ± 0.0007 | +0.0008 (pruned worse; consistent in all 3 seeds, not labeled significant) |
| Corruption mPC (severity 1 → 5) | 0.8678 → 0.5655 | 0.8670 → 0.5607 | small, inconsistent in sign at 3 of 5 severities |
| Pooled corruption ECE (all 5 severities) | lower at every severity | higher at every severity | consistent across all 3 seeds, all 5 severities (6/6 conditions) |

Full numbers, per-seed breakdowns, and statistics: `results/cross_seed_report.md`
and `results/summary.csv`. Full claim-by-claim check against the raw files:
`VERIFICATION.md`.

## Repository layout

```
src/            training/pruning/evaluation code, ECE metrics (src/metrics.py)
scripts/        environment checks, metric unit-test runner
tests/          unit tests for the metrics and pruning-mask logic
configs/        exact hyperparameters used (configs/vgg19_bn_pruning30.json)
results/        per-seed manifests, epoch logs, metrics, pruning stats, summary.csv
                (checkpoints and per-example predictions are NOT committed — see below)
timing/         per-epoch wall-clock time for VGG-19-BN / ResNet-18 / MobileNetV3-Small
provenance/     checksums.sha256, run_manifest.csv, file_timestamps.csv
docs/           protocol.md, deviations.md, assumptions.md, paper_setup.md, sources.md
environment.md  hardware / OS / CUDA / PyTorch versions actually used
VERIFICATION.md line-by-line check of every claim above against raw files
```

## Setup

```bash
python -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
git clone https://github.com/Eric-mingjie/rethinking-network-pruning external/rethinking-network-pruning
cd external/rethinking-network-pruning && git checkout 2ac473d70a09810df888e932bb394f225f9ed2d1 && cd ../..
```

Download CIFAR-10 (`torchvision.datasets.CIFAR10(..., download=True)`) and
CIFAR-10-C from https://github.com/hendrycks/robustness into `data/`.
Verify both against `provenance/checksums.sha256` before running anything.

## Reproducing a seed

```bash
python run_experiment.py --upstream external/rethinking-network-pruning \
  --upstream-revision 2ac473d70a09810df888e932bb394f225f9ed2d1 \
  --device cuda --seed 1 --out results/seed1 --cifar-c data/CIFAR-10-C
```

Repeat with `--seed 2 --out results/seed2` and `--seed 3 --out results/seed3`.
Each run needs ~4-5 GPU-hours on hardware comparable to `environment.md`.

## Running the evaluation only (no retraining)

The included checkpoints-not-uploaded can be re-evaluated against
`src/metrics.py` to reproduce the numbers in `results/seed*/metrics/` exactly,
provided the same determinism flags are set (`cudnn.deterministic = True`,
`torch.use_deterministic_algorithms(True)`, TF32 disabled, see
`environment.md`). This was done once against `results/seed1/checkpoints/baseline_last.pt`
(not committed; checksum in `provenance/checksums.sha256`) during packaging
and reproduced the recorded accuracy and ECE bit-for-bit.

## Datasets and checkpoints

Not included in this repository (CIFAR-10, CIFAR-10-C, and all `.pt`
checkpoints exceed the size and scope of a code/results repo). Download
sources:
- CIFAR-10: https://www.cs.toronto.edu/~kriz/cifar.html
- CIFAR-10-C: https://github.com/hendrycks/robustness

SHA-256 checksums for both, and for every checkpoint produced by this
reconstruction, are in `provenance/checksums.sha256`. Original filesystem
modification times for every result file (so the git commit date is not
mistaken for the measurement date) are in `provenance/file_timestamps.csv`.

## Relationship to other repos

- [`ShohanRony/green-ai-energy-phase1`](https://github.com/ShohanRony/green-ai-energy-phase1)
  is a **separate study** (MAC counts vs. measured energy) and shares no
  files with this repository.
- An earlier, unpublished harness (`energy-pilot`) reported 0.0447 J/image and
  active draw of 71-78W using `nvmlDeviceGetTotalEnergyConsumption`. On this
  GPU that counter over-reports active power by about 30% at every polling
  interval, and it also produces a separate polling artifact (20ms → 291W
  implied for an idle GPU). **Those numbers are superseded and should not be
  cited.** Corrected measurements, using `nvmlDeviceGetPowerUsage`, are
  published in [`ShohanRony/green-ai-energy-phase1`](https://github.com/ShohanRony/green-ai-energy-phase1).

## License

Code: MIT, see `LICENSE` (Copyright (c) 2026 Shohinur Pervez Shohan).
Results and data files (`results/`, `timing/`, `provenance/`, `configs/`):
CC BY 4.0, see `LICENSE-DATA`. The vendored upstream model/pruning code this
reconstruction depends on (`Eric-mingjie/rethinking-network-pruning`) is not
included in this repository and keeps its own MIT license — clone it
separately per the Setup section above. CIFAR-10 / CIFAR-10-C are not
included and retain their original dataset licenses/terms.
