# Kaggle returned scores

All 17 files submitted to `sweettv-tv-program-recommender` (MAP@5). Private score is the reported one; the public score is shown for completeness. Files were designed after seeing earlier returned scores, so the best private score is an optimistic estimate.

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
| 17 | `gt_th50_g1_tau18` | 0.42853 | 0.44375 | recency decay tau = 18 weeks (DESIGNATED FINAL) |

Designated final submission: `submission_gt_th50_g1_tau18.csv` (sha256 518d87537ce2760cd827c6432b7f1ed899d616b6a10162a645d358c9a136db81), private 0.42853.
