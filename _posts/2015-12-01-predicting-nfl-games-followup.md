---
layout: post
title:  "NFL game prediction followup"
date:   2015-12-01 17:00:00
categories: data-science regression scikit-learn
author: Neal Miller
---

As I catch up from the long weekend, I don't have time for a real post, but just thought I'd do a quick follow-up to [last weeks' post]({% post_url 2015-11-24-predicting-nfl-games %}) about predicting NFL game outcomes.

Here are my model's predictions from last week, along with the actual results from the past weekend:

| Game | Model pred. | Actual |
|:------:|-----:|-----:|
| PHI vs. DET | DET (56%) | DET |
| CAR vs. DAL | CAR (52%) | CAR |
| CHI vs.  GB |  GB (63%) | CHI |
| BUF vs.  KC |  KC (63%) | KC  |
| STL vs. CIN | CIN (75%) | CIN |
|  NO vs. HOU | HOU (60%) | HOU |
|  TB vs. IND | IND (55%) | IND |
|  SD vs. JAC | JAC (53%) | SD  |
| MIA vs. NYJ | NYJ (67%) | NYJ |
| MIN vs. ATL | ATL (65%) | MIN |
| NYG vs. WAS | WAS (54%) | WAS |
| OAK vs. TEN | TEN (54%) | OAK |
| ARI vs.  SF | ARI (70%) | ARI |
| PIT vs. SEA | SEA (52%) | SEA |
|  NE vs. DEN |  NE (56%) | DEN |
| BAL vs. CLE | CLE (53%) | BAL |

Overall, the model correctly predicted 10/16 or 63% of the games this past weekend.
A general observation I'll point out is one I made last week: my model seems to overvalue home-field adantage.
In fact, my model chose the home team to win in 13 of the 16 games, and 5 of the 6 incorrect predictions were wins by the away team.

A couple of more specific observations about particular games:

1. The Denver win over New England was a surprise win in overtime, which I don't think many people saw coming -- I probably would have given NE a higher win probability than the 56% the model gave them.
Denver is a team in transition from an injured Peyton Manning (who hasn't played very well this season) to 4th-year backup Brock Osweiler, who has played very well.
2. Baltimore's win over Cleveland was a result of a freak [blocked field goal return for a touch down](http://www.nfl.com/videos/nfl-cant-miss-plays/0ap3000000592050/Can-t-Miss-Play-Will-the-Thrill-wins-it-at-the-buzzer) (the crazy "kick six").
It's hard to ding the model for a prediction that really, really should have been correct.