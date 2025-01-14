---
layout: post
title: Is Bryce Harper a Good Investment for the Phillies? (part 2)
subtitle: I'm biased so I asked the data
tags: baseball Sabermetrics
published: true
---

## How good is Bryce Harper?

My main objective in evaluating Bryce Harper's value to the Phillies was analyzing his offensively ability, paying special attention to his past run production. I was much less concerned with his defensive ability because, as an outfielder, he is unlikely to change the score of a given game on the field. Of course, a truly terrible outfielder who misses routine fly balls and can't make a decent throw to the plate will cost their team some runs, but considering Harper is not a liability in the outfield, I chose to focus on his hitting.

As I outlined in Part 1 of this project, the key offensive stats I looked at were run production, OBP, and SLG. Considering Harper has been in the league for only 7 years plus one year in the Minor League, I wanted to get a sense of his average run production before trying to make any assumptions about his future production. 

![]({{site.baseurl}}/img/bhRunsggplot.png)

The first thing I noticed about this graph was his inconsistency in production. In 2014 when he was just 22, Harper only played 100 games due to a knee injury that required surgery. It seems like the surgery was not just successful but also gave him a combination of motivation and hitting super powers, because in 2015 he came back to lead the league in runs and HRs, lead the entire MLB in OBP and SLG, make the All Star team, win the NL [MVP award](https://en.wikipedia.org/wiki/Major_League_Baseball_Most_Valuable_Player_Award), and win the NL [Silver Slugger Award](https://en.wikipedia.org/wiki/Silver_Slugger_Award).

While these two seasons are definitely critical for understanding Harper as an offensive player, they make his stats pretty challenging to predict, on top of the dubiousness of having so little data to start with. For example, when I took a look at the relationship between runs scored, OBP and SLG, I got this:

![]({{site.baseurl}}/img/bhHittingStatsgg.png)

Statistically, the results of the Moneyball model were unusable due to their lack of statistical significance and low R-squared values. 

```
Call:
lm(formula = R ~ OBP + SLG, data = BryceHarperHitting)

Residuals:
      1       2       3       4       5       6       7       8 
 21.967  -8.841  15.452   2.472 -23.931  -8.209  20.318 -19.227 

Coefficients:
            Estimate Std. Error t value Pr(>|t|)
(Intercept)   -27.12      90.39  -0.300    0.776
OBP           -20.87     486.36  -0.043    0.967
SLG           234.59     248.60   0.944    0.389

Residual standard error: 21.11 on 5 degrees of freedom
Multiple R-squared:  0.481,	Adjusted R-squared:  0.2734 
F-statistic: 2.317 on 2 and 5 DF,  p-value: 0.1941
```

Based on the graphical visualization above, I was not surprised by these results and decided to see if there were other significant statistical relationships I could find to predict Harper's run production. From here, I tried many different combinations of the dependent variables to see if any could make a statistically valid prediction about Harper's run production. I will spare you the details and simply leave you with what I found to be the most successful model I could find. (NB: I said “most” successful. It is still not a great model owing in large part to the lack of data, but workable for my purposes nonetheless.)

**Model 2: OBP + SLG + G**

When I graphed Harper's runs over time, it was obvious that the season he had knee surgery he scored the fewest runs on account of the injury and amount of time spent on the bench. Therefore, I added games played to the model. I chose games because, unlike OBP and SLG, it is not a rate and therefore seemed less likely to be too closely related to the other variables.

```
lm <- lm(R ~ OBP + SLG + G, data = BryceHarperHitting)
summary(lm)

Call:
lm(formula = R ~ OBP + SLG + G, data = BryceHarperHitting)

Residuals:
      1       2       3       4       5       6       7       8 
 1.5634  6.9580  3.8655 -5.6864 -4.8989 -2.4194 -0.3697  0.9874 

Coefficients:
             Estimate Std. Error t value Pr(>|t|)   
(Intercept)  -52.2981    24.5241  -2.133  0.09992 . 
OBP         -378.7796   138.2093  -2.741  0.05187 . 
SLG          343.8607    68.2622   5.037  0.00730 **
G              0.8304     0.1030   8.065  0.00128 **
---
Signif. codes:  0 ‘***’ 0.001 ‘**’ 0.01 ‘*’ 0.05 ‘.’ 0.1 ‘ ’ 1

Residual standard error: 5.681 on 4 degrees of freedom
Multiple R-squared:  0.9699,	Adjusted R-squared:  0.9474 
F-statistic:    43 on 3 and 4 DF,  p-value: 0.001679
```

This result showed a big improvement from my initial model, but I wanted to do some more analysis to see how viable this model was.

#### Analysis

The first thing I checked for was if there was collinearity between variables.

```
cov(BryceHarperHitting$SLG, BryceHarperHitting$OBP)
> 0.003
cov(BryceHarperHitting$SLG, BryceHarperHitting$G)
> 0.40
cov(BryceHarperHitting$G, BryceHarperHitting$OBP)
> 0.30
```
Although games had a slight positive covariance with OBP and SLG, I felt that it was small enough (and much smaller than other variables) to continue my analysis by inspecting the model’s residuals.

Using the olsrr package, I ran the following:

```
ols_plot_resid_qq(lm)
```
![]({{site.baseurl}}/img/QQ0.png)

Here we see that our Q-Q plot is fairly linear although not perfectly aligned.

```
ols_plot_resid_hist(lm)
```
![]({{site.baseurl}}/img/ResHistlm0.png)

Additionally, this histogram shows that the mean of the residuals appears to be 0 but that the data might be skewed. I checked the summary and tested for this and found that the distribution was approximately symmetric with its mean at 0.

```
summary(lm6$residuals)
   Min.  1st Qu.  Median  Mean   3rd Qu.  Max. 
-5.6864 -3.0392  0.3089  0.0000  2.1389  6.9580 

skewness(lm$residuals)
> 0.1321866
```

Finally, I calculated the standard deviation and variance of the residuals, which were fairly high, but relatively lower than other models I tested:
```
var(lm$residuals)
> 18.44
sd(lm$residuals)
> 4.29
```



#### Prediction

At this point, I felt that I could responsibly run the model to see what predictions it would make. Assembling the necessary equation from the regression above, we have:

```
Runs = -52.30 + (-378.78)*OBP + 343.86*SLG + 0.83*G
```
Using 2018 numbers, we get the following prediction for 2019:

```
Runs = -52.30 + (-378.78)*0.393 + 343.86*0.496 + 0.83*159
Runs = 101
```

This feels very high but not unreasonable given his past production when he plays nearly all of the games in a season. Baseball-Reference has [their projection](https://www.baseball-reference.com/players/h/harpebr03.shtml) at 90 runs. Based on Harper's projected PAs, it seems that Baseball-Reference is assuming he will play about 140 games which is about his career average. Similarly, [MLB Fantasy Rankings](https://www.mlb.com/news/fantasy-baseball-rankings-2019-player-preview-c295284374) has him producing 94 runs.

If we reevaluate the equation with 140 games rather than 159 like last season, our new run projection is 86 which is very close to his MLB career average of 87 runs per season.

To give Harper the best shot at impressing me with his potential contribution to the Phillies, let's assume that my model is approximately correct and Harper will produce 101 runs and play nearly every game in 2019. Recalling from Part 1 of this post, the Phillies' run production equation was: 

```
RS = -771.67 + 2706.01*OBP + 1512.56*SLG
```
Also recall that the Phillies needed to make room in their lineup and on the field for Harper. Based on recently projected lineups for 2019, Harper is expected to hit third, right ahead of Rhys Hoskins and take Nick Williams' place as I expected. [According to Baseball-Reference](https://www.baseball-reference.com/players/w/willini01.shtml), Williams is projected to produce 56 runs in the 2019 season over 458 plate appearances (compared to Harper who they project will score 90 runs over 597 plate appearances). Williams' projection is based on the assumption that he will see slightly more action than his last two seasons in the Majors with the Phillies. Although we know this will not be the case as long as Harper stays healthy, it does make it easier to predict how removing Williams and adding Harper will change the Phillies' winning potential. 

As I determined in my first post, I predicted that the Phillies would score 672 runs if they kept their general lineup the same as in 2018. While Harper isn't the only change to the Phillies' roster, he is by far the most significant and the only player likely to have a real impact on the Phillies' season. For that reason, I am holding everything else equal and only changing Williams' and Harper's stats.

```
Original Phillies 2019 Projected Runs = 672
Harper Runs 2019 - Williams Runs 2019 = 101 - 56 = 45
New Phillies 2019 Projected Runs = 717
```
Now let's see how this will impact their winning potential using the model equation from Part 1:

```
Wins = 81.36 + 0.097*(717 - 688) = 84
```
Keeping in mind that my model seems to be a bit bullish on Harper's potential and that the threshold for making the playoffs is 95 wins, I am more confident in my initial feelings that investing so much of the team's salary cap on Bryce Harper was not the right move. To be fair, the results of this model suggest that Harper can single-handedly add four wins to the Phillies' season which is impressive. But I don't think it's enough because adding Harper doesn't address the other half of equation (lowering runs allowed) the way improving pitching would.

### Conclusion

Throughout the course of this project, I have been acutely aware that trying to predict how good a player will be in the future based on less than 10 years of data is very difficult. The beauty of Moneyball, in my opinion, is that the A's were successful because they were able to make a lot of low risk, low commitment gambles and specifically avoided risking a lot of money on blockbuster players. As an insurance underwriter, this concept makes a lot of sense to me.

My issue with signing players to enormous contracts is that either a) they're too young to know how good they will be in the short term or b) they've been around too long and are past their prime. By signing Harper for 13 years, he manages to fall into both categories.


**The Cautionary Tale of Albert Pujols**

When it became clear that Bryce Harper was going to sign a long term contract that wouldn't be up until he was in his mid 30s, my mind went to Albert Pujols. In 2011, he signed a 10 year/$240M contract, which is still one of the [top 10 contracts in MLB history](https://www.si.com/mlb/2019/02/19/manny-machado-largest-contracts-mlb-history). At the time, he was 31 and on the back end of a truly [incredible career](https://www.baseball-reference.com/players/p/pujolal01.shtml). Although Pujols' superstar credentials supported a substantial contract, there was a lot of criticism around the Angels locking him into a contract into his 40s. Since then, it has been clear that Pujols is not the player he was in his prime

![]({{site.baseurl}}/img/APruns.png)

I know that Harper is not Pujols. If any comparison is to be made however, Pujols has had a more impressive career considering the number of times he has been an MVP, and All Star, and won both the Silver Slugger and [Gold Glove](https://en.wikipedia.org/wiki/Rawlings_Gold_Glove_Award) Awards. To be fair, Pujols came into the Majors towards the end of the [Steroids Era](http://www.espn.com/mlb/topics/_/page/the-steroids-era). He was never suspended for steroid use, but the way the game was played in the end of the 90s and early 2000s is different than it is played now since the emphasis on power hitting fueled by performance enhancers has dissipated.

By 2010, the Steroids Era was a nearly a decade in the past, which makes the nosedive that Pujols' run production took particularly concerning for the long-term prospects of even the best players. Whether the deterioration in run production is a function of age or the decreased motivation that comes with incredible job security, sustaining numbers that warrant a $22M to $30M salary for 13 years is not feasible.

My final thoughts are this: The salary market for blockbuster players has continuously ballooned to make contracts like the one Harper signed possible. Whether or not this is a good thing for the sport is another matter, but what it means is that teams will have to keep making difficult payroll decisions to get and keep top talent. After doing this project, I maintain that having one "superstar" player is not a panacea for the Phillies' decade of mediocrity. Looking back at their [2008 roster](https://www.baseball-reference.com/teams/PHI/2008-roster.shtml) when they won the World Series, their most productive hitter was Ryan Howard who made $10M that year and scored 105 runs. Chase Utley and Brad Lidge who were both All Stars that year and heroes of the 2008 season in the hearts and minds of Phillies fans were each making less than $10M. And of course, the team was under the direction of [Charlie Manuel](https://www.baseball-reference.com/managers/manuech01.shtml), the winningest manager in Phillies franchise history. 

The 2008 team had chemistry, and I worry that having one player (especially one who is very young) getting paid way more than everyone else will not help the team dynamic in the locker room and on the field. Although that is just my opinion, after doing this project I feel more confident that signing Bryce Harper will not produce significantly better results than if management had sought to build a more well-rounded team. My prediction of the 2019 season is that the Phillies will be a little better than .500 and not make the playoffs. Now let's see what happens.
