---
layout: post
title:  "Predicting NFL game outcomes using a very simple regression model"
date:   2015-11-24 17:00:00
categories: data-science regression scikit-learn
author: Neal Miller
---
I've been tinkering with predicting NFL game outcomes using some very simple machine learning techniques.
I thought I'd share my very first early stab at it - at the moment, it's an extremely over-simplified model, but it manages to not do completely horribly, and I also thought I'd share thoughts for future extensions I can make.

## The model

I get NFL game data using the [nflgame](http://pdoc.burntsushi.net/nflgame) Python module.
This module makes it very simple to get a detailed summary of each NFL game back to the 2009 season.
For this basic model, I only use the team-level statistics for each game.

Each week, I look back through the season and average the team-level stats provided for each team playing.
At the moment, I use only the season-long average, and don't weight more recent games in any special way.
For each game of the week, I construct a numpy array of average values for the home team and the away team.
From these arrays, I build up a large two-dimensional array (one row for each game) of `X` data, and a single array of `y` game results.
The `y` array is an array of boolean answers to the question "Did the home team win?".

Here's a list of potential parameters used for this model (again, these are averages for the season up to but not including the current game):

```
home_first_downs
home_passing_yds
home_penalty_cnt
home_penalty_yds
home_pos_time
home_punt_avg
home_punt_cnt
home_punt_yds
home_rushing_yds
home_total_yds
home_turnovers
home_score
home_opponentscore
away_first_downs
away_passing_yds
away_penalty_cnt
away_penalty_yds
away_pos_time
away_punt_avg
away_punt_cnt
away_punt_yds
away_rushing_yds
away_total_yds
away_turnovers
away_score
away_opponentscore
```

It's then a relatively simple matter of performing a logistic regression on the data.
For simplicity, when building a model after week 11 (for example), the training set is all data through week 10 of that season, and the test set is the week 11 data.

I performed the following logistic regression, using [scikit-learn](http://scikit-learn.org/stable/).

{% highlight python %}
from sklearn import linear_model

model = linear_model.LogisticRegression(penalty = 'l1', C = 0.001)
model.fit(X_train, y_train)
{% endhighlight %}

A couple of things are worth explanation:

1. `pentalty = 'l1'`: This indicates that the logistic regression fit should use the L1 norm for penalization.
This penalty causes the model to favor fewer features, in an attempt to avoid overfitting the data.
2. `C = 0.001`: The inverse of the regularization strength.  A smaller value of `C` results in a more stringent regularization penalty, and fewer resulting features.

The resulting model has `0`s for many of its coefficients (as a result of regularization), and can be used directly to predict the winner of future games.

## Results

If the above method is run each week of the 2015 season (starting with week 2) the resulting models are correct about 56% of the time, with a range from 29% (week 10, ouch) to 79% (week 8).
Obviously, a coin flip would give you a 50% prediction rate (and accounting for who's the home team would do slightly better), so this isn't amazing.
However, I thought it was worth comparing it to ESPN's "experts".

Here are the model's predictions vs. the "expert" predictions and the actual results from week 11:

| Game | Model pred. | "Experts" pred. | Actual winner |
|:------|-----:|-----:|----:|
| TEN vs. JAC | JAC | JAC | JAC | 
| DEN vs. CHI | CHI | CHI | DEN |
| OAK vs. DET | DET | OAK | DET |
| IND vs. ATL | ATL | ATL | IND |
| NYJ vs. HOU | HOU | NYJ | HOU |
|  TB vs. PHI | PHI | PHI | TB  |
|  GB vs. MIN | MIN | MIN | GB  |
| WAS vs. CAR | CAR | CAR | CAR |
| DAL vs. MIA | MIA | DAL | DAL |
| STL vs. BAL | BAL | BAL | BAL |
| CIN vs. ARI | ARI | ARI | ARI |
|  SF vs. SEA | SEA | SEA | SEA |
|  KC vs.  SD |  SD |  KC |  KC |
| BUF vs.  NE |  NE |  NE |  NE |

Coincidentally, both my simple model and the ESPN experts correctly predicted 8 of the 14 games, or 57%.
Diving in a bit more deeply, my model and the experts disagreed on 4 games:

* OAK/DET - My model was right.
* NYJ/HOU - My model was right.
* MIA/DAL - My model was wrong.  This one is completely understandable - The Dallas Cowbows' quarterback, Tony Romo, has been out for the past seven weeks with an injury.
However, he was back this week, and the team was much, much better.  My model knew nothing about Romo's return.
* SD/KC - My model completely blew this one.

In general, all three of the instances where my model disagreed with the experts (except MIA/DAL) picked the home team - I think in general, the model is overly favorable to the home team.

The resulting model, after week 11 of the 2015 season (this past weekend), has non-zero coefficients for the following parameters:

| Parameter | Coefficient |
|:----|---:|
| `home_passing_yds` | 0.005909538410341311 |
| `home_penalty_yds` | -0.0049323361289636122 |
| `home_pos_time` | 0.00028587196223593843 |
| `home_punt_yds` | -0.00065600775434796713 |
| `home_rushing_yds` | -0.00048070659425629718 |
| `home_score` | 0.013646685838077627 |
| `home_opponentscore` | -0.04872316045400267 |
| `away_passing_yds` | -0.0013947205856620652 |
| `away_pos_time` | -0.00018284691851858413 |
| `away_punt_yds` | 0.0019788838232865607 |
| `away_total_yds` | -0.00071304504849179547 |
| `away_score` | -0.028678438068529651 |
| `away_opponentscore` | 0.035755247261999162 |

This is probably still too many features to use for a model on as little data as we have, but it works for now.
Looking over this list, you can see that the model's estimate of the probability that the home team will win is *increased* by:

* The home team's average number of passing yards
* The home team's average time of posession
* The home team's average score
* The away team's average number of punt yards (most likely due to correlation with number of punts)
* The average score allowed by the away team.

The probability that the home team will win is *decreased* by:

* The home team's average number of yards of penalties
* The home team's average number of punt yards
* The average score allowed by the home team
* The away team's average number of passing yards
* The away team's average time of posession
* The away team's average total number of yards
* The away team's average score

Note that the features were not normalized, so the relative scales of these coefficients can't be meaningfully compared.
However, it is nice that the model passes the "smell test" - it behaves as expected in relation to the features in the model.

For fun, my model's predictions (with probability estimates) for week 12 are shown below:

| Game | Model pred. |
|:------:|-----:|
| PHI vs. DET | DET (56%) |
| CAR vs. DAL | CAR (52%) |
| CHI vs.  GB |  GB (63%) |
| BUF vs.  KC |  KC (63%) |
| STL vs. CIN | CIN (75%) |
|  NO vs. HOU | HOU (60%) |
|  TB vs. IND | IND (55%) |
|  SD vs. JAC | JAC (53%) |
| MIA vs. NYJ | NYJ (67%) |
| MIN vs. ATL | ATL (65%) |
| NYG vs. WAS | WAS (54%) |
| OAK vs. TEN | TEN (54%) |
| ARI vs.  SF | ARI (70%) |
| PIT vs. SEA | SEA (52%) |
|  NE vs. DEN |  NE (56%) |
| BAL vs. CLE | CLE (53%) |

I'm planning to throw the code used for this simply analysis up on GitHub once I've had a chance to clean it up a bit.

## Possible improvements
There are a few avenues I'd like to explore for improvement:
1. Consider player stats.  This might help catch instances like the MIA/DAL game mentioned above.
2. Weight team stats to favor more recent games.
3. Incorporate [Elo ratings](https://en.wikipedia.org/wiki/Elo_rating_system).  This would hopefully allow the model to be more responsive to strength of schedule as well as dramatic changes in team quality during a season.
4. Perform Monte Carlo simulations.  Ultimately, this is how prediction is really done, and it's vry necessary to predict things like scores and wins vs. the spread.  It's a much more complicated and involved method which involves constructing a model of each team and sampling ensembles of simulated matches between them.

As I said, this is a vastly oversimplified model and I wouldn't want to use it for anything besides as a toy.
But I do think it's interesting that such a simple model is able to do reasonably well, at least comparably to football "expert" predictions.