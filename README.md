# ELO Ratings for Squash

## Summary
The Professional Squash Association (PSA) uses a simple rating system to rate professional squash players. The aim of this project was to try using the more sophisticated (but still relatively simple) ELO rating system, which is most famously used to rate professional chess players.

There are two intended benefits of using ELO:
* a person gets rating points based on who they play against, rather than which round of a tournament they reach. It is more impressive to beat the world's best player in Round 1, then to get to the final of a tournament by having a lucky draw.
* the rating is predictive and interpretable. The difference between two players' ELO ratings should inform you of the odds of each player winning the match.

Using past few decades of male squash match results, I used the ELO rating system to rate the players. The table below shows that ELO is able to achieve the desired benefits.

| Predicted probability of higher rated player winning. Rounded to nearest 0.05 | Number of predictions | Observed fraction of matches that higher rated player won |
|----------------------------------------------------------------------------------------:|----------------------:|---------------------------------------------------------------:|
|                                                                                    0.50 |                  2566 |                                                       0.518316 |
|                                                                                    0.55 |                  5315 |                                                       0.559548 |
|                                                                                    0.60 |                  5292 |                                                       0.611678 |
|                                                                                    0.65 |                  5238 |                                                       0.668385 |
|                                                                                    0.70 |                  5401 |                                                       0.700796 |
|                                                                                    0.75 |                  5642 |                                                       0.755583 |
|                                                                                    0.80 |                  6024 |                                                       0.813081 |
|                                                                                    0.85 |                  6507 |                                                       0.849393 |
|                                                                                    0.90 |                  7258 |                                                       0.900248 |
|                                                                                    0.95 |                  8834 |                                                       0.937967 |
|                                                                                    1.00 |                  5831 |                                                       0.967930 |

## Repository Structure
* `data/`
    * `raw/`
        * `matches_male.csv`. Raw data containing all professional male squash match results for past few decades.
        * `tournaments_male.csv`. Raw data containing metadata for all professional male squash tournaments for past few decades. (Data not used for this project)
    * `processed/`
        * `matches_male.csv`. Processed match results data. See `01-la-processing.ipynb` for the processing that was done.
* `notebooks`
    * `01-la-processing.ipynb`. Notebook used to carry out processing and EDA of the raw data.
    * `02-la-analysis`. Notebook used to calculate and evaluate the ELO ratings using the processed match data.
* `.gitignore`
* `LICENSE`
* `README.md`
## Libraries used
* numpy
* pandas
* matplotlib
* seaborn
* re
* os
* tqdm