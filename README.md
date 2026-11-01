# LLM4Rec Midterm — sweettv TV Program Recommender

Goal: recommend the top-5 TV programs per user for the Kaggle competition
`sweettv-tv-program-recommender`, scored with MAP@5. HSE LLM4Rec exam project:
every number and table in this repository is produced by `midterm.ipynb`.

## Final submission

`outputs/submission_gt_th50_g1_tau18.csv` is the file whose returned Kaggle private
score (0.42853) is reported in the PDF report. It is produced by `write_sub9e` with
`theta=0.50` (an airing counts only if the user watched at least half of it), `log1p`
of the program's airings in the target window, a personalised channel-by-hour
exposure factor (`gamma=1.0`), `days^0.25`, and a recency decay of the own-history
hits with `tau = 18` weeks. sha256 prefix `518d8753`.

All scored files and their private/public scores are listed in `kaggle_scores.md`.

## Reproduce

1. Python 3.13. Install dependencies: `pip install -r requirements.txt`.
2. Download the competition data with your own Kaggle credentials
   (`kaggle competitions download sweettv-tv-program-recommender -p data --unzip`)
   and keep these exact file names in `data/` (rename after download if needed):
   `dataset11-30.csv`, `export_arh_11-20-final.csv`, `export_arh_21-30-final.csv`,
   `export_arh_31-42-final.csv`, `submission.csv`. The `data/` folder is not committed.
3. Execute the whole notebook once, top to bottom:
   `python3 -m nbconvert --to notebook --execute --inplace midterm.ipynb`
   Measured runtime: 768 s for sections 0-13 on the machine listed in section 13
   (Apple M4, peak memory 8.65 GiB). The later cells (9e-9i, 10c/10d, 14) reuse the
   in-memory objects and add little time (section 9g alone takes 4.1 s).
4. No network access is needed after step 2. The matching step caches
   `data/interim/matched_pairs.parquet` and `data/interim/interactions.parquet`.
5. Submission files are written to `outputs/`. Files that already exist are never
   overwritten by the later sections.

## Results (MAP@5, Fold A = weeks 11-20 -> 21-30, Fold B = weeks 11-18 -> 19-28)

| Method | Fold A GT-1 | Fold A GT-2 | Fold B GT-1 | Fold B GT-2 |
|---|---|---|---|---|
| Random (avg 5 seeds) | 0.000506 | 0.003538 | 0.000410 | 0.003508 |
| Popularity-by-users | 0.177103 | 0.553070 | 0.188676 | 0.552938 |
| Popularity-by-seconds | 0.180193 | 0.541744 | 0.192483 | 0.541780 |
| Personal history [diagnostic] | 0.301359 | 0.510529 | 0.303652 | 0.504927 |
| Best ALS (alpha_t=0.01, f=32) | 0.122533 | 0.368034 | 0.126070 | 0.365795 |
| Best SVD (k=32) | 0.221413 | 0.507973 | 0.225958 | 0.509013 |
| Own-history + personalised exposure (section 9c, E7, early candidate) | 0.318652 | 0.605568 | 0.325666 | 0.602004 |
| Final recipe (theta 0.5, tau 18; GT-1 is not its selection criterion) | 0.263719 | — | 0.271354 | — |
| Own-history upper bound (diagnostic, not a method) | 0.699757 | — | 0.681307 | — |

GT-1 (top-5 by watched seconds) ranks the later variants in the opposite order to the
returned Kaggle scores, so the final recipe was chosen by the returned private score,
not by GT-1. See the report for the limits of this choice.

## Files in `outputs/` (sha256 prefix)

| File | What it is | sha256[:8] |
|---|---|---|
| `submission_mf.csv` | SVD k=32 on full history (weeks 11-30) | `c9f84e23` |
| `submission_cbf.csv` | CBF-lite (TF-IDF over show metadata) only | `9c0c56fb` |
| `submission_hybrid.csv` | tuned hybrid = own-history with popularity fallback | `2550c033` |
| `submission_repeat.csv` | R10 repeat ranker | `cd9f833d` |
| `submission_repeat2.csv` | section-9c variant (E7: chhw, gamma=0.25) | `3267a04f` |
| `submission_probe_days.csv` | probe: E9 ndays x log1p(airings) | `cf246700` |
| `submission_probe_frac.csv` | probe: E10 frac>=0.5 count x log1p(airings) | `273c8069` |
| `submission_probe_frac25.csv` | probe: frac>=0.25 count x log1p(airings) | `6c876c1a` |
| `submission_probe_frac75.csv` | probe: frac>=0.75 count x log1p(airings) | `84696a32` |
| `submission_probe_frac90.csv` | probe: frac>=0.90 count x log1p(airings) | `444096cd` |
| `submission_best_gtstar.csv` | section-9d pick tuned on GT*: days^0.25 x log1p(airings) x chhw exposure | `c2805b0e` |
| `submission_gt_th50.csv` | section-9e manual round (student): theta=0.50, gamma=0.5 | `cbae38cb` |
| `submission_gt_th60.csv` | section-9e manual round (student): theta=0.60, gamma=0.5 | `fcbd0b74` |
| `submission_gt_th75.csv` | section-9e manual round (student): theta=0.75, gamma=0.5 | `258caceb` |
| `submission_gt_th50_g1.csv` | section-9e manual round (student): theta=0.50, gamma=1.0 | `d5d7b1a2` |
| `submission_gt_th50_g15.csv` | section-9e manual round (student): theta=0.50, gamma=1.5 | `fb6902c6` |
| `submission_gt_th50_b50.csv` | section-9e manual round (student): theta=0.50, beta=0.5 | `e1fa63b8` |
| `submission_gt_th50_g1_tau6.csv` | section 9f: th50_g1 recipe with recency decay tau=6 weeks | `0a36f231` |
| `submission_gt_th50_g1_tau12.csv` | section 9f: th50_g1 recipe with recency decay tau=12 weeks | `966fb76e` |
| `submission_gt_th50_g1_tau24.csv` | student-run cell: th50_g1 recipe with tau=24 weeks | `ff5eb700` |
| `submission_gt_th50_g1_tau18.csv` | **designated final**: th50_g1 recipe with tau=18 weeks, private 0.42853 | `518d8753` |

The probe files were created to calibrate the validation ground truth (proxy)
against the returned Kaggle private score, since the local GT-1 ranks the submitted
files in the opposite order to Kaggle. Files that were uploaded to Kaggle were not
modified afterwards: the full notebook run rewrites them byte-identically, and
section 9b rechecks `submission_mf.csv` against a fresh regeneration (same sha,
zero differing rows).

## What the later sections do

Sections 9e-9i are later additions.

- **9e (student, by hand):** the six `submission_gt_th*.csv` files. Theta is selected on
  the returned Kaggle private score instead of on GT*, because GT* is itself built with
  theta=0.25, so picking theta by GT* would be circular.
- **9f:** an optional recency decay `w = exp(-age_weeks / tau)` of the own-history hit
  count. With the 9e recipe held fixed, mild decay (tau 6 and 12 weeks) improves both
  validation folds and strong decay (tau 3 and 1.5 weeks) is worse than the unweighted
  count on both folds. `tau=None` stays the default and leaves every earlier number unchanged.
- **9g (agent) and 9h-9i (student):** a finer tau grid and an SVD fill for users with fewer
  than five own-history candidates. Every tau from 9 to 36 weeks beats the unweighted
  count on GT* on both folds, the folds disagree on the best tau, the SVD fill does not
  beat the popularity fill, and the final file uses tau = 18 weeks, chosen because it
  had the best returned private score (the validation cannot separate tau 12, 18 and 24).

## Notebook layout

0 setup, 1-5 data/EDA/matching, 6 baselines, 7 ALS + analysis, 8 CBF, 9 hybrid,
9b repeat ranker, 9c exposure form + personalised exposure, 9d proxy calibration,
9e manual rounds (student), 9f recency decay, 9g finer tau and SVD fill,
9h-9i student cells (SVD-fill check, tau18/tau24 files), 10 case study
(10c/10d table by the student), 11 submissions, 12 limitations and conclusions,
13 environment and runtime, 14 final-file integrity check.

## Case study users

Fixed users used for qualitative comparison across all methods:
`13571199046623737884` (heavy user, most labelled airings in the Fold A history among
users with a non-empty GT-1) and `8150816946177670084` (light user, one labelled airing).

## Disclosure

Sections 9e, 9h, 9i, the case-study table cell 10c/10d and the integrity-check cell 14
were added by hand after the OpenCode session ended (code suggested in a separate Claude
chat) and are not in the session export. The Claude chat itself is not in the session
export either.
