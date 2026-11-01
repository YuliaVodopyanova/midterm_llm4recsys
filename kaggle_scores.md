# Kaggle returned scores

All 24 files submitted to `sweettv-tv-program-recommender` (MAP@5). Private score is the reported one; the public score is shown for completeness. Files 1-17 are the hand-set own-history ranker, 18-24 are LightGBM rerankers (notebook Sections 15-18, except #18). Files were designed after seeing earlier returned scores, so the best private score is an optimistic estimate.

| # | File (`outputs/submission_<name>.csv`) | Private | Public | What changed |
|---|---|---|---|---|
| 1 | `hybrid` | 0.24856 | 0.27887 | own-history seconds, hybrid weights 0/0/1/0 |
| 2 | `repeat` | 0.29642 | 0.32907 | x log1p(target-window airings) (R10) |
| 3 | `repeat2` | 0.32890 | 0.35686 | x personalised channel-by-hour exposure (E7) |
| 4 | `probe_days` | 0.36849 | 0.38739 | distinct days x log1p(airings) (E9) |
| 5 | `probe_frac25` | 0.38960 | 0.40570 | airings with overlap_frac >= 0.25 |
| 6 | `probe_frac` | 0.41211 | 0.42075 | airings with overlap_frac >= 0.5 |
| 7 | `probe_frac90` | 0.38706 | 0.39074 | airings with overlap_frac >= 0.9 |
| 8 | `best_gtstar` | 0.41417 | 0.43249 | theta 0.25, days^0.25, gamma 0.5 |
| 9 | `gt_th75` | 0.41665 | 0.43108 | same recipe, theta 0.75 |
| 10 | `gt_th50` | 0.42211 | 0.43659 | same recipe, theta 0.5 |
| 11 | `gt_th50_g1` | 0.42270 | 0.44159 | theta 0.5, gamma 1.0 |
| 12 | `gt_th50_g15` | 0.42179 | 0.44250 | theta 0.5, gamma 1.5 |
| 13 | `gt_th50_b50` | 0.42257 | 0.44223 | theta 0.5, gamma 1.0, days^0.5 |
| 14 | `gt_th50_g1_tau6` | 0.42430 | 0.43857 | recency decay tau = 6 weeks |
| 15 | `gt_th50_g1_tau12` | 0.42702 | 0.44344 | recency decay tau = 12 weeks |
| 16 | `gt_th50_g1_tau24` | 0.42824 | 0.44342 | recency decay tau = 24 weeks |
| 17 | `gt_th50_g1_tau18` | 0.42853 | 0.44375 | recency decay tau = 18 weeks (best hand-set ranker) |
| 18 | (first reranker version) | 0.43412 | 0.45057 | LightGBM reranker written in the Claude chat; code and file were later removed, not reproducible |
| 19 | `lgbm_agent` | 0.43419 | 0.44396 | Section 15, single seed, 30 popular candidates |
| 20 | `lgbm_v0` | 0.43240 | 0.45010 | Section 17, simple reranker, own-history candidates only, binary labels (DESIGNATED FINAL) |
| 21 | `lgbm_v1` | 0.43558 | 0.44641 | Section 16, grid-best parameters (15 leaves, 400 trees, lr 0.03, 50 per leaf) |
| 22 | `lgbm_v3` | 0.43365 | 0.44485 | Section 16, 50 popular candidates |
| 23 | `lgbm_blend` | 0.43526 | 0.44595 | Section 18, within-user rank blend of S15, v0, v1 |
| 24 | `lgbm_v2` | 0.43614 | 0.44292 | Section 16, five-seed average (seeds 42-46) of the Section 15 configuration (best returned score) |

Designated final submission: `outputs/submission_lgbm_v0.csv`, private 0.43240, public 0.45010 (sha256 prefix 94ce2520; run `shasum -a 256` on the committed file and paste the full hash here). Best returned file: `submission_lgbm_v2.csv`, private 0.43614. Best hand-set file: `submission_gt_th50_g1_tau18.csv`, private 0.42853.

The seven reranker scores lie within 0.0037 of each other and validation does not separate them; the designated file `lgbm_v0` has the highest public score among the existing files (0.45010) but not the highest private one; private scores were visible when it was chosen.
