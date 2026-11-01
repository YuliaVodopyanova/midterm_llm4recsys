# LLM4Rec Midterm — sweettv TV Program Recommender

Goal: recommend the top-5 TV programs per user for the Kaggle competition
`sweettv-tv-program-recommender`, scored with MAP@5. HSE LLM4Rec exam project:
every number and table in this repository is produced by `midterm.ipynb`.

## Reproduce

1. Python 3.13. Install dependencies: `pip install -r requirements.txt` (or `uv sync`).
2. Download the competition data with your own Kaggle credentials
   (`kaggle competitions download sweettv-tv-program-recommender -p data --unzip`)
   and keep these exact file names in `data/` — rename after download if needed:
   `dataset11-30.csv`, `export_arh_11-20-final.csv`, `export_arh_21-30-final.csv`,
   `export_arh_31-42-final.csv`, `submission.csv`.
3. Execute the whole notebook once, top to bottom:
   `python3 -m nbconvert --to notebook --execute --inplace midterm.ipynb`
   Measured runtime: 732.0 s on the machine listed in section 13 (the full run that includes sections 9e and 9f).
4. No network access is needed after step 2. The matching step caches
   `data/interim/matched_pairs.parquet` and `data/interim/interactions.parquet`.

## Results (MAP@5, Fold A = weeks 11-20 -> 21-30, Fold B = weeks 11-18 -> 19-28)

| Method | Fold A GT-1 | Fold A GT-2 | Fold B GT-1 | Fold B GT-2 |
|---|---|---|---|---|
| Random (avg 5 seeds) | 0.000506 | 0.003538 | 0.000410 | 0.003508 |
| Popularity-by-users | 0.177103 | 0.553070 | 0.188676 | 0.552938 |
| Popularity-by-seconds | 0.180193 | 0.541744 | 0.192483 | 0.541780 |
| Personal history [diagnostic] | 0.301359 | 0.510529 | 0.303652 | 0.504927 |
| Best ALS (alpha_t=0.01, f=32) | 0.122533 | 0.368034 | 0.126070 | 0.365795 |
| Best SVD (k=32) | 0.221413 | 0.507973 | 0.225958 | 0.509013 |
| Own-history + personalised exposure (section 9c, E7, adopted) | 0.318652 | 0.605568 | 0.325666 | 0.602004 |
| Own-history upper bound (diagnostic, not a method) | 0.699757 | — | 0.681307 | — |

## Files in `outputs/` (sha256 prefix)

| File | What it is | sha256[:8] |
|---|---|---|
| `submission_mf.csv` | SVD k=32 on full history (weeks 11-30) | `c9f84e23` |
| `submission_cbf.csv` | CBF-lite (TF-IDF over show metadata) only | `9c0c56fb` |
| `submission_hybrid.csv` | tuned hybrid = own-history with popularity fallback | `2550c033` |
| `submission_repeat.csv` | R10 repeat ranker | `cd9f833d` |
| `submission_mf_recheck.csv` | section-9b regeneration of `submission_mf.csv` (zero differing rows) | `c9f84e23` |
| `submission_repeat2.csv` | adopted section-9c variant (E7: chhw, gamma=0.25) | `3267a04f` |
| `submission_probe_days.csv` | probe: E9 ndays x log1p(airings) | `cf246700` |
| `submission_probe_frac.csv` | probe: E10 frac>=0.5 count x log1p(airings) | `273c8069` |
| `submission_probe_frac25.csv` | probe: frac>=0.25 count x log1p(airings) | `6c876c1a` |
| `submission_probe_frac75.csv` | probe: frac>=0.75 count x log1p(airings) | `84696a32` |
| `submission_probe_frac90.csv` | probe: frac>=0.90 count x log1p(airings) | `444096cd` |
| `submission_best_gtstar.csv` | section-9d pick tuned on GT*: days^0.25 x log1p(airings) x chhw exposure | `c2805b0e` |
| `submission_gt_th50.csv` | section-9e manual round (student): theta=0.50, gamma=0.5 | `cbae38cb` |
| `submission_gt_th60.csv` | section-9e manual round (student): theta=0.60, gamma=0.5 | `fcbd0b74` |
| `submission_gt_th75.csv` | section-9e manual round (student): theta=0.75, gamma=0.5 | `258caceb` |
| `submission_gt_th50_g1.csv` | section-9e manual round (student): theta=0.50, gamma=1.0 — best returned private score | `d5d7b1a2` |
| `submission_gt_th50_g15.csv` | section-9e manual round (student): theta=0.50, gamma=1.5 | `fb6902c6` |
| `submission_gt_th50_b50.csv` | section-9e manual round (student): theta=0.50, beta=0.5 | `e1fa63b8` |
| `submission_gt_th50_g1_tau12.csv` | section-9f: th50_g1 recipe with recency decay tau=12 weeks | `966fb76e` |
| `submission_gt_th50_g1_tau6.csv` | section-9f: th50_g1 recipe with recency decay tau=6 weeks | `0a36f231` |

These four probe files were created to calibrate the validation ground truth
(proxy) against the returned Kaggle private score, since the local GT-1 ranks the
submitted files in the opposite order to Kaggle.

The first four files were uploaded to Kaggle before this commit and were not
modified afterwards: the full notebook run rewrites them byte-identically, and
section 9b rechecks `submission_mf.csv` against a fresh regeneration (same sha,
zero differing rows). `submission_probe_frac25.csv`, `submission_probe_frac75.csv`,
`submission_probe_frac90.csv`, and `submission_best_gtstar.csv` are new in this
commit; they are probe candidates, not replacements for the uploaded ones.

Sections 9e and 9f are later additions. Section 9e (the six `submission_gt_th*.csv` files) was
added by the student by hand after the assistant session — it is the student's work, not the
assistant's. It selects theta on the returned Kaggle private score instead of on GT*, because
GT* is itself built with theta=0.25, so picking theta by GT* would be circular. Section 9f adds
an optional recency decay `w = exp(-age_weeks / tau)` to the own-history hit count; with the
section-9e recipe held fixed, mild decay (`tau=12`, `tau=6` weeks) improves both validation
folds, while `tau=3` and `tau=1.5` are worse than the unweighted count on both folds. The two
folds disagree on which mild `tau` is best, so the effect is reported as small and not fully
settled, and `tau=None` stays the default — which leaves every earlier number in the notebook
unchanged.

## Cleanup done in this step (stale numbers removed from prints)

- Fold A and Fold B comparison tables in section 7 printed five hard-coded
  baseline rows; they are now recomputed from the stored recommendation dicts.
- Part 0 item 1 restated a cross-check result as literal numbers; it is now a
  markdown pointer to the matching cell that computes it.
- Part 0 item 2 printed literal row/user/show counts; all values are recomputed.
- Section 5 printed the slide AP@5 expectations as literals next to asserted
  computations; the print now uses the computed values.
- Section 7 debug/independent-check cells quoted baseline numbers (random,
  popularity, personal history, ALS base) as literals; they are recomputed or
  replaced by pointers to the tables that produce them.
- The matching cell reported peak memory as "KB" although `ru_maxrss` is bytes on
  macOS, and printed an unexplained `False` from a cross-check that compared
  mismatched index keys; the unit is now normalised and the check verifies that
  vectorized pairs exist and all overlaps are positive.
- A local `rows` variable in the diagnostics cell shadowed the section-6 results
  list; it was renamed.

## Notebook layout
0 setup, 1-5 data/EDA/matching, 6 baselines, 7 ALS + analysis, 8 CBF, 9 hybrid,
9b repeat ranker, 9c exposure form + personalised exposure, 9e manual final round (added by
the student), 9f recency decay of history hits, 10 case study and
report tables, 11 submissions, 12 limitations and conclusions, 13 environment.

## Case study users (section 7)
Fixed users used for qualitative comparison across all methods: 13571199046623737884 (heavy user) and 8150816946177670084 (light user).
