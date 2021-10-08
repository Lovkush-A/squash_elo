# ELO Rating for Squash

## Project Overview
Using the ELO rating system and past few decades of squash match data, I created ratings that are superior to the current simplistic ratings used by the Professional Squash Association (PSA).

The two main improvements ELO ratings provide over PSA ratings are:
* In ELO, a person gets rating points based on who they play against, rather than which round of a tournament they reach. It is more impressive to beat the world's best player in Round 1, then to get to the quarter-final of a tournament by having a lucky draw.
* In ELO, the rating is predictive and interpretable. The difference between two players' ELO ratings provides a simple way of estimating the odds of the higher rated player beating the lower rated player. In PSA's system, the rating values have little useful information; it is revealing that the ratings are never shown or commentated on during squash broadcasts.

The first improvement is guaranteed by how the ELO rating system works, see explanation below.

The second improvement is not guaranteed but requires empirical verification. This is done by computing the following 'calibration metric' for various values of `p`:

* In all the matches in which the ELO rating system predicts that the higher-rated player has probability `p` of winning, what is the true rate at which the higher-rated player actually wins.

After doing some hyper-parameter tuning, I was able to achieve the following values for these metrics:

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

In the rest of this blog, I will describe how I achieved these remarkable results!

## ELO

The surprising thing about ELO is how simple it is, given what it can achieve.

The underlying ideas are straightforward:
* use the difference between ratings to compute an expected score
* changes in ratings are proportional to the difference between the true score and the expected score.

The full details are as follows:

* If two players have ratings `r1` and `r2` respectively, then the predicted odds of player 1 beating player 2 is `p(r1, r2) = 1 / (1 + 10**((r2 - r1) / 400))`
    * Note there is nothing fundamentally important about the numbers 10 or 400 here. These are simply the numbers used in chess, the most famous use case of the ELO rating system.
    * And yes, this is just (a scaled version) of the sigmoid function!
* If Player 1 with rating `r1` beats Player 2 with rating `r2`, then the ratings are updated as follows:
    * The change in rating is `delta = K*(1 - p(r1, r2))`.
        * The variable `K` is a hyper-parameter. The larger `K` is, the bigger the changes in ratings will be.
        * The quantity `1-p(r1, r2)` is the difference between the true score of 1 (representing victory) and predicted score of `p(r1, r2)`.
    * Player 1's new rating is `r1+delta`
    * Player 2's new rating is `r2-delta`
* If a player has no matches, we assign them a default starting rating of 1500.
    * This is arbitrary, since ELO ratings only depend on differences in ratings.

And that's it!


## The data
The raw data I used contains the male match history from the past few decades. It includes players' names, their seed for the tournmanet, who won the match, the scores of the games in the match (usually) and some other minor details.

## Cleaning
There were several parts to cleaning the raw data.
* 

## Analysis
First attempt with 32. then hyper-parameter tuning.

## Possible improvements
* Automatically update ratings as new results come in, by scraping live match data.
* Create a web app with these squash ratings. Allow users to see predictions for upcoming matches and tournaments, and explore past data.
* Currently we only use the final result of the match. However, we should be able to use the score in games to improve the ratings: winning 3-0 is more impressive than winning 3-2 and that should be reflected in the update rule. The way to do this is to have more refined definition of 'true score'. Right now true score is 1 for victory, but can imagine having true score of 0.9 for winning 3-0 and true score of 0.7 for winning 3-2. This would require some tuning.

