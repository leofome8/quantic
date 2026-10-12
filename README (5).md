# A calibrated fair-comparison audit of quantum kernels

Code, derived data and decision rules for the manuscript *A calibrated fair-comparison audit of
quantum kernels: what such a protocol can and cannot conclude* (Forero Mendoza, de Mello Junior,
Batista, Kohler, Hall Barbosa, Pacheco).

Everything here is a Jupyter notebook meant to run in Google Colab on CPU. Each notebook is
self-contained: it installs its dependencies, connects its data, runs one experiment and prints its
own results. Cells are numbered in execution order. No notebook depends on another notebook having
run first.

## Data

| Corpus | How to obtain | Subjects | Recordings |
|---|---|---|---|
| Saarbrücken Voice Database (SVD) | registration at `stimmdb.coli.uni-saarland.de` | 1,315 speakers | 15,766 |
| Parkinson (Oxford, UCI) | downloaded automatically by the notebooks | 32 subject codes | 195 |

`features_v5.pkl` holds the acoustic feature matrix derived from the SVD recordings, together with
speaker and session identifiers. It contains no audio. The raw recordings are not redistributed
here; obtain them from the database maintainers under their terms.

The Parkinson corpus is public and the notebooks fetch it from the UCI repository at run time, so
nothing needs to be placed by hand for that corpus.

## Notebooks

**Protocol validation**

| File | What it answers |
|---|---|
| `mc3_baselines.ipynb` | Five arms under identical selection and evaluation: an independently tuned RBF, the least-squares surrogate, random Fourier features, and the product and ZZ fidelity kernels. Separates the question of whether the gaussian kernel reproduces the quantum Gram geometry from the question of whether a competitively tuned classical model matches quantum performance. |
| `mc8_calibracao.ipynb` | Calibration of the decision rule itself against synthetic data with a controlled effect size: observed type I error against the nominal level, and power curves by sample size and prevalence. Generates its own data; needs no external corpus. |

**Audit trail**

These are the versions that produced the results reported in the manuscript, kept in the order they
were run so that each correction is visible.

| File | What it does |
|---|---|
| `audit_v5_multicorpus.ipynb` | Extends the surrogate-tracking claim beyond voice to a second corpus. |
| `audit_v6_analitico.ipynb` | Analytic form of the fidelity kernel at `reps=1`, graduated control, WDBC at low n. |
| `audit_v7_parkinson.ipynb` | Fixes the scaling artefact in the graduated control and adds the Parkinson corpus. |
| `audit_v8_parkinson.ipynb` | Replaces the invalid null (zeroed margin) with label permutation across the whole pipeline. |
| `audit_v11_replicacao.ipynb` | Tests whether the surrogate–quantum correlation replicates across corpora. |
| `audit_v13_parecer__1___1_.ipynb` | Answers the four blocking points of the first review: partial correlation controlling for bandwidth, fold as the unit of analysis, and the per-cell p-value problem. |
| `sweep_v2_dois_mapas__2_.ipynb` | Bandwidth sweep over two feature maps with the corrected residual diagnostic. |
| `sweep_v3_10folds (3).ipynb` | Same sweep at 10 speaker-disjoint folds. |
| `sweep_v4_criticas.ipynb` | Re-analysis at each map's own optimum, deconfounding, and projected kernels. |

## Decision rules

These were fixed in the analysis scripts before the runs they govern. They were not deposited in a
time-stamped public registry, so they are not pre-registered in the formal sense; this file and the
commit history are the record.

**Equivalence margin.** Two kernels are declared equivalent when the two-one-sided-tests procedure
rejects non-equivalence at ±0.01 AUC. The margin is a practical threshold, not a clinical one: it is
the smallest difference the authors considered worth reporting as a methodological finding.

**Detection floor.** For each corpus, labels are permuted across the entire pipeline (feature
selection, scaling and training all see only permuted labels), the advantage statistic is recomputed,
and the floor is the 0.95 quantile of that null distribution. It is the critical value of a one-sided
permutation test at 5%, so it controls the type I error rate. It is not a power calculation and it is
not a minimum relevant effect. An observed difference below the floor is not detectable by this
protocol, whatever its sign. Measured floors: 0.0075 (SVD), 0.0595 (Parkinson), 0.167 (synthetic).

**Unit of analysis.** The fold, never the grid cell. Grid cells sharing a fold are not independent,
so no p-value is reported per cell. Confidence intervals come from a percentile bootstrap over
folds.

**Validation.** Speaker-disjoint throughout, with `StratifiedGroupKFold` on the outer loop and on the
inner loop of any hyperparameter search. Mixing a speaker between the training and validation halves
of an inner search inflates the apparent advantage of whichever arm is not searched that way.

## Reproducing

Open a notebook in Colab, run cell 1 to install dependencies, run cell 3 to connect the data, then
run the remaining cells in order. The SVD notebooks ask for `features_v5.pkl`, either from a Google
Drive folder you name in the form or by manual upload; the cell verifies the file size and reports
what it found. Run times on Colab CPU are roughly 20 minutes per repetition for the SVD corpus and
15 seconds for Parkinson.

Results depend on the seeds declared inside each notebook. Figures in the manuscript were produced
from the runs recorded here.
