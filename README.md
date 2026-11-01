# LLM4Rec Midterm — sweettv TV Program Recommender

Goal: recommend the top-5 TV programs per user for the Kaggle competition
`sweettv-tv-program-recommender`, scored with MAP@5. HSE LLM4Rec exam project:
every number and table in this repository is produced by `midterm.ipynb`
(one exception: the first LightGBM reranker version, private 0.43412, was run from a Claude
chat cell that was later deleted and is not reproducible; it is listed in `kaggle_scores.md`).

## Final submission

**`outputs/submission_lgbm_v0.csv`**: simple LightGBM LambdaRank reranker over each user's own-history candidates (24 history-only features, binary labels). Kaggle private MAP@5 **0.43240** (public 0.45010). The best returned file is `submission_lgbm_v2.csv` (five-seed full reranker, private 0.43614).

The best hand-set ranker without a learned model is `outputs/submission_gt_th50_g1_tau18.csv` (private 0.42853).

- Notebook Sections 1-14: matching, validation folds, baselines (popularity, ALS, SVD, content-based, hybrid) and the hand-set own-history ranker.
- Sections 15-18 (written by Claude Code): LightGBM reranker, variants (seed bagging, 50 popular candidates, grid, slot-match, rank_xendcg), simple v0 and rank blend. Output files: `submission_lgbm_agent.csv`, `_v0`, `_v1`, `_v2`, `_v3`, `_blend`.
- All 24 returned Kaggle scores: [`kaggle_scores.md`](kaggle_scores.md).
- AI usage and session exports: see report section 6.

## Reproduce

1. Python 3.13. Install dependencies: `pip install -r requirements.txt`.
2. Download the competition data with your own Kaggle credentials
   (`kaggle competitions download sweettv-tv-program-recommender -p data --unzip`)
   and keep these exact file names in `data/` (rename after download if needed):
   `dataset11-30.csv`, `export_arh_11-20-final.csv`, `export_arh_21-30-final.csv`,
   `export_arh_31-42-final.csv`, `submission.csv`. The `data/` folder is not committed.
3. Execute the whole notebook once, top to bottom:
   `python3 -m nbconvert --to notebook --execute --inplace midterm.ipynb`
   LightGBM 4.6.0 is needed for sections 15-18 (it is in `requirements.txt`; add it if missing).
   Measured runtime: 768 s for sections 0-13 on the machine listed in section 13
   (Apple M4, peak memory 8.65 GiB). The later cells (9e-9i, 10c/10d, 14) reuse the
   in-memory objects and add little time (section 9g alone takes 4.1 s). Sections 15-18 (LightGBM) add about 3,500 s
   (146 s, 2,005 s, 9 s and 1,364 s).
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
returned Kaggle scores, so the hand-set recipe was chosen by the returned private score,
not by GT-1. See the report for the limits of this choice.

LightGBM rerankers (sections 15-18), two-fold user cross-fit, MAP@5 on GT* (top-5 by
airings watched for at least 25% of their length) and GT-1:

| Method | A GT* | A GT-1 | B GT* | B GT-1 |
|---|---|---|---|---|
| Hand-set ranker (tau 18) | 0.4205 | 0.2637 | 0.4214 | 0.2714 |
| LightGBM, 1 seed (`lgbm_agent`) | 0.4342 | 0.2809 | 0.4340 | 0.2862 |
| LightGBM, 5-seed average (`lgbm_v2`) | 0.4358 | 0.2822 | 0.4363 | 0.2872 |
| LightGBM, grid-best parameters (`lgbm_v1`) | 0.4350 | 0.2813 | 0.4377 | 0.2873 |
| LightGBM, 50 popular candidates (`lgbm_v3`) | 0.4368 | 0.2833 | 0.4344 | 0.2870 |
| Simple v0 (`lgbm_v0`, designated final) | 0.4260 | 0.2769 | 0.4313 | 0.2849 |
| Rank blend (`lgbm_blend`) | 0.4360 | 0.2805 | 0.4385 | 0.2882 |

Kaggle private scores: `lgbm_v0` 0.43240 (designated final, highest public score 0.45010),
`lgbm_v2` 0.43614 (best private), `lgbm_v1` 0.43558, `lgbm_blend` 0.43526,
`lgbm_agent` 0.43419, `lgbm_v3` 0.43365, against 0.42853 for the hand-set ranker.

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
| `submission_gt_th50_g1_tau18.csv` | best hand-set ranker: th50_g1 recipe with tau=18 weeks, private 0.42853 | `518d8753` |
| `submission_lgbm_agent.csv` | section 15: LightGBM LambdaRank reranker, 1 seed, private 0.43419 | `894eb2ca` |
| `submission_lgbm_v0.csv` | section 17: simple reranker, own-history candidates, binary labels, **designated final**, private 0.43240 | `94ce2520` |
| `submission_lgbm_v1.csv` | section 16: grid-best parameters, private 0.43558 | `8970fd3f` |
| `submission_lgbm_v2.csv` | section 16: five-seed average of the section 15 configuration, private 0.43614 (best returned) | `602510d2` |
| `submission_lgbm_v3.csv` | section 16: 50 popular candidates, private 0.43365 | `a39fc057` |
| `submission_lgbm_blend.csv` | section 18: within-user rank blend of section 15, v0 and v1, private 0.43526 | `442ea26d` |

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

- **15-18 (Claude Code, run by the student):** LightGBM LambdaRank reranker over own-history
  candidates plus popular unseen programmes with history-only features and graded labels (15),
  variants evaluated by cross-fit and paired bootstrap (16), the simple v0 design (17) and a
  rank blend (18). The seven reranker files returned 0.43240 to 0.43614 on the private
  leaderboard, all above the hand-set 0.42853, and validation does not separate them.
  `lgbm_v0` is designated final because it has the highest public score of the existing files
  (private scores were visible when it was chosen; `lgbm_v2` has the best private score).

## Notebook layout

0 setup, 1-5 data/EDA/matching, 6 baselines, 7 ALS + analysis, 8 CBF, 9 hybrid,
9b repeat ranker, 9c exposure form + personalised exposure, 9d proxy calibration,
9e manual rounds (student), 9f recency decay, 9g finer tau and SVD fill,
9h-9i student cells (SVD-fill check, tau18/tau24 files), 10 case study
(10c/10d table by the student), 11 submissions, 12 limitations and conclusions,
13 environment and runtime, 14 final-file integrity check (written before sections 15-18, so
its manifest does not list the lgbm files), 15-18 LightGBM rerankers.

## Case study users

Fixed users used for qualitative comparison across all methods:
`13571199046623737884` (heavy user, most labelled airings in the Fold A history among
users with a non-empty GT-1) and `8150816946177670084` (light user, one labelled airing).

## Disclosure

Sections 9e, 9h, 9i, the case-study table cell 10c/10d and the integrity-check cell 14
were added by hand after the OpenCode session ended (code suggested in a separate Claude
chat) and are not in the session export. The Claude chat itself is not in the session
export either.

Sections 15-18 were written by Claude Code from a prompt drafted in the Claude chat and run
by the student (`session2.json`, a processed transcript with reasoning removed and tool results
cut at 4,000 characters, not a native export). The first reranker version was written in the
Claude chat and deleted later. The idea of a LightGBM reranker comes from reading another
student's public README; no code or results were copied.
