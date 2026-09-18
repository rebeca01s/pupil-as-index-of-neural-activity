# Pupil as an index of neural activity

Does pupil size track the population state of mouse primary visual cortex (VISp), and how much of that link is just locomotion?

Data: **Allen Visual Coding – Neuropixels** ([DANDI:000021](https://dandiarchive.org/dandiset/000021)), read by streaming — the session files are several GB and are never downloaded in full.

**Scope.** This is a learning project: a first hands-on pass at reading NWB files through the Python API and at working with spike data, so the emphasis is on building the pipeline and its controls correctly rather than on a new finding. It starts from one session and is then extended to every session in the dataset that has eye tracking.

## Result

Spontaneous activity (grey screen, constant luminance), 25 sessions, one per animal:

| Coupling with the VISp state (Spearman, max \|r\| within ±3 s) | Median | Significant |
|---|---|---|
| pupil | 0.403 | 14 / 25 |
| pupil, running speed regressed out | 0.315 | 14 / 25 |
| running speed | 0.200 | 15 / 25 |

Removing running speed lowers the pupil-state coupling systematically (Wilcoxon, p = 1.4e-4) but modestly — about 39 % of the shared variance is attributable to locomotion. An out-of-sample decoder agrees: the pupil reconstructs the state slightly better than running speed does (r = 0.282 vs 0.203) and keeps most of it once running is removed (0.238, p = 0.12).

**The pupil is a partial but genuine index of the VISp population state, above locomotion.**

This corrects the single-session result: the session analysed first happened to have the strongest running-state coupling in the whole dataset (|r| = 0.758), so there the pupil-state link did collapse once running was controlled for (0.285 → 0.071).

## Contents

| File | |
|---|---|
| `Pupila_estado_VISp_individual_analysis.ipynb` | Single session: alignment, PCA state, cross-correlation with a circular null, control for running speed, decoding with a lag sweep |
| `Pupila_estado_VISp2_group_analysis.ipynb` | Same pipeline wrapped per session, swept over every session with eye tracking, plus the group statistics |
| `tutorial_estado_neuronal_dandi000021.ipynb` | Walkthrough from spikes to population state: raster, firing rates, PCA, state trajectories |
| `sessions_with_pupil_000021.csv` | The 26 sessions of DANDI:000021 that contain pupil tracking |
| `group_spontaneous_results.csv` | One row of metrics per session — the data behind the group figures |

## Reproduce

Python 3.11+ and an internet connection; there is nothing to download first.

```bash
python -m venv .venv
source .venv/bin/activate        # Windows: .venv\Scripts\activate
pip install -r requirements.txt
jupyter lab
```

The single-session notebook runs in a few minutes. The group sweep is the slow part: roughly 2–3 minutes per session, so about an hour for all 26. Results accumulate in `group_spontaneous_results.csv` with incremental caching, so the sweep can be stopped and resumed; `MAX_SESSIONS` limits how many run at a time and `RUN_GRU = False` drops the GRU for a faster pass.

## Method

50 ms bins, 100 ms gaussian smoothing, Allen Institute spike-sorting QC thresholds, and a 3-component PCA of the VISp firing rates fitted on the first 70 % of bins only. Significance comes from a circular-shift null (2000 permutations), which preserves each signal's autocorrelation and breaks only their temporal relationship — necessary because two slow signals correlate spuriously under any test that assumes independent samples. Locomotion is controlled for by regressing contemporaneous running speed out of both signals.

## Caveats

- The running control is linear and contemporaneous, so delayed or non-linear effects of locomotion may remain.
- Per-session couplings are modest (~0.3); the result rests on consistency across sessions, not on effect size.
- PC1 explains a median 12 % of the variance, so it is a partial summary of the population state.
- One dataset, one recording modality, one behavioural regime.

## License and citation

Code under Apache 2.0 (see `LICENSE`). The data belong to the Allen Institute and are distributed through DANDI under CC-BY 4.0. If you reuse this work, cite the dataset:

> Siegle, J. H., Jia, X., Durand, S., et al. (2021). Survey of spiking in the mouse visual system reveals functional hierarchy. *Nature*, 592, 86–92. DANDI:000021
