# ELO Rating for Squash

## Project Overview
Using the ELO rating system and past few decades of squash match data, I created ratings that are superior to the current simplistic ratings used by the Professional Squash Association (PSA). Code for this project is available in the [GitHub repository for this project](https://github.com/Lovkush-A/squash_elo).

## Problem Statement
There are two big problems in the current way squash ratings are determined by PSA. 

1. A player gets rating points depending on which tournment they play in and which round they reach in the tournament. There is zero weight put on who you actually play against. E.g. it ought to be more impressive to beat the world's best player in Round 1, then to reach a quarter-final by beating low ranked players (because you had a lucky draw).

2. The current rating points are not meaningful. It is revealing that ratings point are never mentioned in squash broadcasts.

The proposed strategy is to use the ELO rating system.
The underlying ideas behind ELO are straightforward:
* Use the difference between ratings of players to compute an expected score for an individual match.
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

By construction, ELO gaurantees to fix problem 1 because the ratings are completely based on who you beat, rather than which round of a tournament you reach.

However, the second problem is not guaranteed to be fixed by ELO, and will require empirical verification. This leads on to the metrics used to evaluate ELO.

## Metrics
We evaluate ELO by computing the following 'calibration metric' for various values of `p`:

* In all the matches in which the ELO rating system predicts that the higher-rated player has probability `p` of winning, what is the true rate at which the higher-rated player actually wins.

For example, if the ELO rating system believes that the higher rated player has 70% odds of winning, then we should observe the higher rated player winning approximately 70% of the time.

These calibration metrics provide a measure of how well (or not) a rating system can fix problem 2.


## Data Exploration
The raw data I used contains the male match history from the past few decades. It includes players' names, their seed for the tournament, who won the match, the scores of the games in the match (usually) and some other minor details.

Here are a few entries from the dataframe to illustrate:

| tournament_index |          round |                                           players |                       result |
|-----------------:|---------------:|--------------------------------------------------:|-----------------------------:|
|                0 | Quarter-finals |     [1] Tayyab Aslam (PAK) bt Farhan Hashmi (PAK) | 7-11, 11-9, 11-5, 11-5 (32m) |
|                0 | Quarter-finals | [7] Israr Ahmed (PAK) bt [9/16] Waqas Mehboob ... |       11-3, 11-3, 11-8 (23m) |
|                0 | Quarter-finals |  [4] Amaad Fareed (PAK) bt [5] Farhan Zaman (PAK) |      11-8, 11-7, 12-10 (25m) |

Not all of the data in this dataframe was needed for the project and there was a little bit of dirty data. Here are the cleaning and feature extraction steps taken:
* Dropping columns that were not useful for the analysis (`round` and `tournament_index`).
* Extracting the score in games from the score in points provided in the `result` column. In retrospect, this was not needed for the project, but it is still useful if I want to extend the project to make use of games scores. See possible improvements below.
* Keeping only those rows in which one player beat another player and dropping all other rows.
  * The vast majority of rows dropped were because a player automatically gets through in a particular round. E.g. if there are 48 players in a tournament, then in Round 1, players seeded 1 to 16 will automatically get through to Round 1 without having to beat anybody.
  * There were exactly two other rows that got dropped. One is recorded as "No shows" and the other is recorded as "Final not played due to unsafe court conditions."
* Parsing the `players` column of the raw data.
  * Extract the name (and seed and country) of the winner and loser of the match.
  * This was done using regex. If you are interested, the regex patterns used are in the `parse_player_entry` function in the [processing notebook](https://github.com/Lovkush-A/squash_elo/blob/main/notebooks/01-la-processing.ipynb).

At the end of this exploration, cleaning and feature extraction, the key information we end up with is a dataframe with two columns (name of winner and name of loser) where each row is a single match, and it is ordered chronologically.
## Data Visualisation
As far as I know, there are no data visualisations that help with the next task of calculating ELO ratings. If you run the [processing notebook](https://github.com/Lovkush-A/squash_elo/blob/main/notebooks/01-la-processing.ipynb), you will see some visuals (e.g. showing distribution of players' win percentages) but they have no influence on the next steps.

## Data Preprocessing
This has already been discussed in the data exploration section above.

## Implementation
The code to calculcate the ELO rating systems is in the [analysis notebook](https://github.com/Lovkush-A/squash_elo/blob/main/notebooks/02-la-analysis.ipynb).

Surprisingly, there were minimal complications in the implementation of the ELO rating system. I just had to write functions that calculate how to update ELO ratings based on a single match, then loop through all matches and update ELO ratings one-by-one.

One implementation detail worth noting is how I chose an initial value for the hyper-parameter `K` (see problem statement section). The choice was based on the values used in chess (the most famous place ELO ratings are used) and they are given in the [wikipedia article on ELO rating system](https://en.wikipedia.org/wiki/Elo_rating_system#Mathematical_details).

## Refinement
The refinement process was straightforward: I simply tried various values for the hyper-parameter `K` and looked at how their values in the 'calibration metrics' compared. The first solution tried used `K=32` and the final solution uses `K=100`.

## Results

Here is table of 'calibration metrics' for `K=32`:

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
But I was not 100% sure of this, so I did some (basic and manual) hyperparameter tuning, creating a function that loops through several values of `K` (namely 10, 50, 100, 200 and 500).

Manually looking at the tables shows that `K` = 100 is the best value out of these. Here are the values for the calibration metrics we get:

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

You can see that the predictions are well callibrated!

## Reflection
I personally find these results astonishing. By using a straightforward rule to update players' ratings based on their old ratings, we are able to get a meaningful rating system that provides callibrated odds on who will win a match.

I was not expecting this at all! Before doing the project, I was anticipating having to tweak the algorithm (e.g. by including some 'domain knowledge' somehow), but the ELO rating system just worked out-of-the-box. It is always nice when things simply work out!

## Improvement
There are numerous ways this project could be improved or extended. Here are just a few possibilities:

* Automatically update ratings as new results come in, by scraping live match data.
* Create a web app with these squash ratings. Allow users to see predictions for upcoming matches and tournaments, and explore past data.
* Currently we only use the final result of the match. However, we should be able to use the score in games to improve the ratings: winning 3-0 is more impressive than winning 3-2 and that should be reflected in the update rule. The way to do this is to have more refined definition of 'true score'. Right now true score is 1 for victory, but can imagine having true score of 0.9 for winning 3-0 and true score of 0.7 for winning 3-2. This would require some tuning.
* Creating a general purpose ELO-rating package that people can use to create ELO ratings and predictions for any sports or competitions they are interested in.
