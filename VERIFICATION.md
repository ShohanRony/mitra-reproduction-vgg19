# Verification of proposal §2.3 / §2.5 claims against raw outputs

Checked against the files in this repository (not re-derived, not rounded).
Every value below was read directly from the cited file.

| Claim in proposal | Value found | Source file | Status |
|---|---|---|---|
| Model / data | VGG-19-BN, CIFAR-10 | `results/seed1/manifest.json` (`configuration.architecture`, `model_parameters: 20040522`), confirmed identical in seed2/seed3 | MATCH |
| Pruning | Global unstructured L1 on conv weights, 30% | `results/seed1/pruning.json`: `"method": "global_L1_conv_only_fixed_mask"`, `"target_fraction": 0.3`, `"achieved_fraction": 0.3` | MATCH |
| Training | 160 baseline epochs + 40 recovery epochs | `results/seed1/manifest.json` `configuration.baseline_epochs: 160`, `recovery_epochs: 40`; `results/seed1/epochs/` contains `baseline_001..160.json` and `pruned30_recovered_001..040.json` | MATCH |
| Optimizer | SGD, momentum 0.9, weight decay 1e-4, batch 64, LR 0.1 | `results/seed1/manifest.json` `configuration.momentum: 0.9`, `weight_decay: 0.0001`, `batch_size: 64`, `baseline_lr_one_based: "1-79:0.1;..."` | MATCH |
| Seeds | 3, all run to completion | `results/seed{1,2,3}/manifest.json` all show `status: "completed_reconstruction_clean_and_corruption"`, no `failure.json` present in any seed dir | MATCH |
| Baseline clean accuracy | 0.9372 ± 0.0010 | `results/cross_seed_report.md` line 28 | MATCH |
| Pruned clean accuracy | 0.9375 ± 0.0011 | `results/cross_seed_report.md` line 28 | MATCH |
| Baseline clean ECE (equal-mass) | 0.0472 ± 0.0012 | `results/cross_seed_report.md` line 29 | MATCH |
| Pruned clean ECE (equal-mass) | **0.0480 ± 0.0007** (not stated in the proposal table) | `results/cross_seed_report.md` line 29 | FOUND — see note below |
| Pruned clean ECE higher than baseline in all 3 seeds | Per-seed deltas +0.00078, +0.00120, +0.00035 — all positive | `results/cross_seed_report.md` lines 37-40 | MATCH |
| Corruption mPC | 0.8678 down to 0.5655 across severities | `results/cross_seed_report.md` table, baseline mPC column, severities 1 and 5 | MATCH |
| Pooled corruption ECE | Pruned higher at every severity, all 3 seeds (6 of 6 conditions) | `results/cross_seed_report.md` lines 62-69 ("Sign is consistent ... across all 3 seeds at every single severity, and also in the clean condition — 6 out of 6") | MATCH |
| Corruption evaluation | All 15 CIFAR-10-C types x 5 severities | `results/seed1/manifest.json` `cifar_c_sha256` lists 15 corruption `.npy` files + `labels.npy`; `results/cross_seed_report.md` reports 5 severities | MATCH |
| Provenance | SHA-256 hashes and commit references for every run | `results/seed{1,2,3}/manifest.json`: `upstream.revision` (commit hash), `upstream.model_source_sha256`, `dataset_sha256`, `cifar_c_sha256` all present per seed | MATCH (with one gap — see below) |
| Per-epoch time (clean), VGG-19-BN | 68.68s | `timing/STATUS_AND_TRAINING_REPORT.md` ("Baseline avg 68.68s/epoch"); corroborated by `results/seed1/epochs/baseline_*.json` `seconds` field (isolated epochs ~66-69s) | MATCH |
| Per-epoch time (clean), ResNet-18 | 47.71s (SD 0.082) | `timing/resnet18_epochs_clean.jsonl` — mean 47.706s, computed SD ≈0.082s over 5 epochs | MATCH |
| Per-epoch time (clean), MobileNetV3-Small | 7.08s (SD 0.16) | `timing/mobilenet_v3_small_epochs.jsonl` — mean 7.080s, computed SD ≈0.16s over 5 epochs | MATCH |

## Pruned clean ECE — the value the proposal table leaves blank

**0.0480 ± 0.0007**, compared to baseline's **0.0472 ± 0.0012** — a plain
difference of **+0.0008** (pruned higher). This is the magnitude behind the
"pruned ECE higher than baseline in all 3 seeds" row; per the paired t-test
in `results/cross_seed_report.md` (t(2) = 3.18), it does not reach the
strict df=2 critical value of 4.303 at n=3. It is reported here as a
directionally consistent observation, not a significant one.

## Gap found during verification

**Provenance does not include a SHA-256 of `run_experiment.py` itself at run
time** — only the upstream *model source* files are hashed in each
`manifest.json`. This means the exact harness-code version used to produce
seed1-3 cannot be cryptographically verified after the fact, only inferred
circumstantially (timestamps, a working preflight check, and the fact that
the only diff from the originally-committed script is a single one-line
import fix). Full detail in `docs/deviations.md` item 1. This is a real gap,
reported honestly rather than resolved by assumption; it does not change any
reported number.

## Tally

- MATCH: 15
- MISMATCH: 0
- NOT FOUND: 0
- Additional finding (value not in proposal table, located and reported above): 1 (pruned clean ECE)
- Provenance gap disclosed (does not affect any number above): 1 (run-time harness-code hash)
