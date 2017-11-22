# Project McNulty 
## Predicting MLB First-Ballot Hall of Famers

For this [project](HOFPredictions.ipynb), I built an algorithm to predict which retired Major League Baseball players will be voted into the Hall of Fame on their first year eligible. A motivating use case to this would be if one were an MLB merchandising platform with Hall of Fame voting coming up, you would want to predict whether a given player will be voted in when considering inventory of memorabilia of that player. The company would want to minimize both false positives (you don’t want a bunch of un-sellable jerseys of non-hall of famers lying around) as well as false-negatives (you want to make sure you can keep up with an upswing in demand for a newly-minted hall of famer’s jerseys). 

For simplicity I narrowed the scope of the project in two ways. First, I focused only on players who were primarily known for their hitting, since the “classical” stats for hitting are more representative than those for either pitching or fielding. Additionally, I limited it to looking at only the first opportunity since it can be difficult to replicate the rationale for a voter to choose not choose a player one year and change his mind the next.

Here are my presentation [slides](MLBHof.pdf), using Vladimir Guerrero as a toy example (spoiler alert: my Random Forest model was accurately conservative in predicting his coming up short in 2017...but Naive Bayes did not fare as well)

## Features 

Ultimately, the goal of this algorithm would be to make a “second order” prediction. Meaning, I’m not using their playing history to directly predict whether they will be voted in; rather, I’m predicting what the voters will think. So, I’m not exactly trying to capture the players’ underlying value as much as the perception of that value. 

With this in mind, I first compiled a bunch of the “classical” counting stats from the [Lahman’s baseball database](http://www.seanlahman.com/baseball-archive/statistics/). This includes things like total homers, rbi, doubles, etc. Additionally, I used those to engineer career batting average and slugging percentage.

Next, I determined a player’s primary position by choosing the most frequently played position.

I wanted to account for the era in which they played, but it was difficult to come up with a simple feature to reflect that. So, recognizing that the changes in voting rules can naturally create 3 different “epochs,” I binned each player into one of those 3. 

Additionally, I used a catch-all metric from Fangraphs called, [“Wins Above Replacement.”](https://www.fangraphs.com/library/misc/war/) Essentially, this is an advanced stat that captures how many wins a player added to his team’s total throughout the course of his career (beyond a certain baseline). This might seem a bit strange, since this stat only came about over the last two decades. However, my rationale was that it would numerically capture aspects about the players’ ability that was more intuitively known by the voters. This proved to be one of the best features.

Finally, I recognized that there are several players whose career numbers very likely warrant a first-ballot induction, but failed due to the taint of scandal (whether steroids in the case of Barry Bonds or gambling like Pete Rose). To further account for this, I manually added a binary variable to indicate this for the handful of more high-profile players who fit into this group.

## Modeling

I iteratively put the data into a bunch of classification algorithms and cross-validated. Since it was a heavily unbalanced class, (only about 5% are inducted on the first ballot), I chose to optimize for Area Under the Curve, which accounts for false positive/negative tradeoffs and would be more telling than simple classification. I tried a range of classification algorithms on it and looked for best cross validation scores. The two top-performing were Random Forests and Naive Bayes. Though Random Forests’ success is of little surprise, Naive Bayes’ certainly was. I believe one reason is possibly because, due do its sensitivity to prior estimates, it handled the high-achieving scandal-tainted players really well. It was able to “pick up” on the scandal indicator quite nicely. 

## Further Considerations

There’s a myriad of ways to expand on features and feature engineering to enhance the quality of the predictions. One can use more of the robust collection of modern stats. Or, one can create some features that measure playoff success or represent special awards (like MVP, rookie of the year, etc.). Also, you can create features to capture players’ performance in their primes. 

Additionally, there’s definitely more room to fine-tune the algorithms. Specifically, as Naive Bayes pre-supposes a Gaussian distribution, trying some feature transformations on the data to fit that might improve performance. This is especially true considering that elite baseball stats probably have some skew.

However, recognizing that it is still a very small class of those getting inducted on their first opportunity (around 50 in total), there is probably a clear upper limit as to how well an algorithm can perform.
