# Per-epoch training time, three architectures

Measured on the same laptop GPU used for the main VGG-19-BN reproduction
(see `../environment.md`), while the real `mitra-reproduction` VGG-19-BN
seed-1 job (160+40 epochs) was running elsewhere on the same GPU. Full
narrative in `STATUS_AND_TRAINING_REPORT.md`.

## Which file is which

| File | Architecture | Condition | Mean epoch time |
|---|---|---|---|
| `resnet18_epochs.jsonl` | ResNet-18 | **Contended** — ran concurrently with the VGG-19-BN job | 100.84s (SD 0.13s) |
| `resnet18_epochs_clean.jsonl` | ResNet-18 | **Clean** — measured after VGG-19-BN had exited | 47.71s (SD 0.082s) |
| `mobilenet_v3_small_epochs.jsonl` | MobileNetV3-Small | Clean — measured after VGG-19-BN had exited | 7.08s (SD 0.16s) |
| `gpu_health_resnet18.log` | — | GPU telemetry during the **contended** ResNet-18 run | — |
| `gpu_health_resnet18_clean.log` | — | GPU telemetry during the **clean** ResNet-18 run | — |
| `gpu_health_mobilenet.log` | — | GPU telemetry during the MobileNetV3-Small run | — |

The contended run is ~2.1x slower (GPU power pinned at its cap, SM clock
reduced from ~1972MHz to 1627-1845MHz — compute/power-budget sharing between
two concurrent CUDA contexts, not thermal throttling: temperature stayed at
68-72C). It is retained here, not deleted, because it is a real measurement
and the contention effect itself is a useful data point; the clean file is
the one that should be used for any extrapolation (10-seed schedules etc.).

VGG-19-BN's own clean per-epoch time (68.68s baseline, ~68.65s recovery) is
reported in `STATUS_AND_TRAINING_REPORT.md`; its own epoch-level JSON lines
are in `../results/seed1/epochs/` (combine with `../provenance/file_timestamps.csv`
for exact per-epoch wall-clock seconds, field `seconds` in each
`baseline_NNN.json` / `pruned30_recovered_NNN.json`).
