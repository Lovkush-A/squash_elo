# ELO Rating for Squash

## Project Overview
Using the ELO rating system and past few decades of squash match data, I created ratings that are superior to the current simplistic ratings used by the Professional Squash Association (PSA). Code for this project is available in the [GitHub repository for this project](https://github.com/Lovkush-A/squash_elo).

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

The surprising thing about ELO is how simple it is, given the results it can achieve.

The underlying ideas are straightforward:
* Use the difference between ratings to compute an expected score for an individual match.
* Changes in ratings after an individual match are proportional to the difference between the true score and the expected score.

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
* If a player has not yet played any matches, we assign them a default starting rating of 1500.
    * This is arbitrary, since ELO ratings only depend on differences in ratings.

And that's it!


## The data
The raw data I used contains the male match history from the past few decades. It includes players' names, their seed for the tournmanet, who won the match, the scores of the games in the match (usually) and some other minor details.

## Cleaning
There were several parts to cleaning the raw data.
* Dropping columns that were not useful for the analysis (`round` and `tournament_index`).
* Extracting the score in games from the score in points provided in the results column. In retrospect, this was not needed, but it is still useful if I want to extend the project to make use of games scores. See possible improvements below.
* Keeping only those rows in which one player beat another player and dropping all other rows.
  * The vast majority of rows dropped were because a player automatically gets through in a particular round. E.g. if there are 48 players in a tournament, then in Round 1, players seeded 1 to 16 will automatically get through to Round 1 without having to beat anybody.
  * There were exactly two other rows that got deleted. One is recorded as "No shows" and the other is recorded as "Final not played due to unsafe court conditions."
* Parsing the `players` column of the raw data.
  * Extract the name (and seed and country) of the winner and loser of the match.
  * This was done using regex.

## Analysis
The analysis consisted of looping through all the matches (chronologically) and updating ratings one match at a time, and then computing the calibration metrics described above.

I first tried doing this when `K` is 32, the value used in chess. Here are the calibration metrics this achieves:

| Predicted probability of higher </br>rated player winning.</br> Rounded to nearest 0.05 | Number of predictions | Observed fraction of matches that </br>higher rated player won |
|----------------------------------------------------------------------------------------:|----------------------:|---------------------------------------------------------------:|
|                                                                                    0.50 |                  7386 |                                                       0.466152 |
|                                                                                    0.55 |                 11801 |                                                       0.566647 |
|                                                                                    0.60 |                  9542 |                                                       0.657514 |
|                                                                                    0.65 |                  8010 |                                                       0.746067 |
|                                                                                    0.70 |                  6759 |                                                       0.803817 |
|                                                                                    0.75 |                  5772 |                                                       0.867983 |
|                                                                                    0.80 |                  4967 |                                                       0.900946 |
|                                                                                    0.85 |                  4062 |                                                       0.932546 |
|                                                                                    0.90 |                  3128 |                                                       0.953005 |
|                                                                                    0.95 |                  2112 |                                                       0.974905 |
|                                                                                    1.00 |                   369 |                                                       0.981030 |

The pattern here is that the predicted odds of winning are underconfident: if the ELO rating thinks that the higher-rated player has 80% odds of winning, they actually win 90% of the time.
My instinct was that the ELO rating was not updating quick enough based on the data, i.e. that `K` is too small.
But I was not 100% sure of this, so I did some (basic and manual) hyperparamter tuning, creating a function that loops through several values of `K` (namely 10, 50, 100, 200 and 500).

Manually looking at the tables shows that `K` = 100 is the best value out of these, and it with this hyper-parameter that I achieved the calibration results given in the overview.

## Possible improvements
There are numerous ways this project could be improved or extended. Here are three possibilities:

* Automatically update ratings as new results come in, by scraping live match data.
* Create a web app with these squash ratings. Allow users to see predictions for upcoming matches and tournaments, and explore past data.
* Currently we only use the final result of the match. However, we should be able to use the score in games to improve the ratings: winning 3-0 is more impressive than winning 3-2 and that should be reflected in the update rule. The way to do this is to have more refined definition of 'true score'. Right now true score is 1 for victory, but can imagine having true score of 0.9 for winning 3-0 and true score of 0.7 for winning 3-2. This would require some tuning.
* Creating a general purpose ELO-rating package that people can use to create ELO ratings and predictions for any sports or competitions they are interested in.
