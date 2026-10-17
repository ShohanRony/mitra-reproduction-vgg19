# Deviations, gaps, and honest provenance limitations

## 1. No cryptographic proof that seed1-3 results were produced with the fixed `run_experiment.py`

`run_experiment.py` had a bug: `importlib.import_module("models")` resolved
to the upstream repo's empty top-level `models` package instead of
`models.cifar`, so `vgg19_bn` was never found (fixed in the commit
"Fix models import to models.cifar in run_experiment.py"). No run manifest or
log records the SHA-256 of `run_experiment.py` itself at run time — only the
upstream *model source* files are hashed (see `upstream.model_source_sha256`
in each `results/seed*/manifest.json`), not the harness script that imports
them.

What supports (but does not cryptographically prove) that seed1-3 used the
fixed code:
- `results/laptop-preflight/preflight.json` (a real forward/backward pass,
  `status: "passed_forward_backward_only"`) could only have succeeded if the
  import bug was already fixed, since `vgg19_bn` must resolve to build the
  model.
- File timestamps in `provenance/file_timestamps.csv` show the preflight
  (2026-09-20 14:17 +0600) predates `results/seed1/results.json`
  (2026-09-20 19:57 +0600).
- The only diff between the two originally-committed `run_experiment.py`
  versions and the version added in this repo's third commit is the single
  one-line import fix (verified with `git diff` at package time).

This is circumstantial, not a hash match. If exact run-time source identity
matters, it cannot be established retroactively — stated plainly here rather
than papered over.

## 2. No matched-training dense control (200-epoch baseline)

See `docs/protocol.md` — the pruned arm gets 40 more epochs of training than
the baseline arm, and that confound is not controlled for in this
reconstruction. Planned as future work.

## 3. Unresolved ambiguities inherited from the paper text

`docs/assumptions.md` lists every point where the paper's text underspecified
the protocol (ECE estimator conflict between the stated equation and the
experimental section, LR boundary indexing, checkpoint selection, etc.) and
states the explicit decision made here. None of these were resolved by
consulting author code, because no author-released code for this experiment
was found (`docs/sources.md`).

## 4. Commit history predates the successful training runs

This repository's git history begins with 2 commits from 2026-09-20, both
from an earlier attempt that was blocked by a missing GPU/CUDA environment
(see `REPORT.md` and `docs/resumed_status.md` for that attempt's honest
"no numerical result" status). The actual successful seed1/2/3 training runs
happened later the same day on different hardware and were originally
uncommitted working-tree files. This repository adds them in a dedicated
commit; `provenance/file_timestamps.csv` records each result file's
*original* filesystem modification time (September 2026) so the commit date
is not mistaken for the measurement date.

## 5. Corruption evaluation report for seed 1 alone is explicitly insufficient

`results/seed1/corruption_eval_report.md` is a single-seed report generated
before seeds 2 and 3 completed; it is retained for provenance but
`results/cross_seed_report.md` explicitly states that single-seed report was
"insufficient" for any directional claim, and is the authoritative summary.
