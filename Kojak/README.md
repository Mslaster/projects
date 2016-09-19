# Optimizing MLB Manager's Challenge   

For my capstone project I decided to tackle the issue of when to optimally use a manager's challenge in baseball.
Starting in 2014, Major League Baseball instituted a new rule where a manager had the right to "challenge" a call by an umpire (for certain types of calls). 
The call is then reviewed by a crew of umpires in New York City through multiple camera angles. If the manager is correct, the call is overturned. If the umpire is correct or the video is inconclusive, then the manager loses the ability to challenge for the remainder of the game. So, my goal was to model when a manager should optimally attempt to challenge a possible mistaken call.    

##Data Sources
I scraped replay data from MLB's [Baseball Savant website] (https://baseballsavant.mlb.com/replay) and combined it with the files available available on [Retrosheet](http://retrosheet.org/Replay.htm). With the exception of about 50 out of 3000 replays, the data was pretty consistent. 

## Units of Measurement   
First, I had to determine how exactly I'd capture what "success" or "failure" meant. To do this, I turned to the [Win Expectancy] (http://www.fangraphs.com/library/misc/we/) stat developed by Tom Tango. This essentially is a matrix to calculate the probability a team will win a game given the game state (i.e. score differential, innings, runners on base, etc.). So, I use the difference in WE to capture how "good" or "bad" a certain event is.    

## Formula    
To figure out whether to challenge, there are three things to take into account. 1. The potential upside if successful. 2. The downside if unsuccessful. 3. The probability of success. If the probability of success times the gain in the upside is greater than the probability of failure times the loss, then a manager should challenge. Otherwise he shouldn't. Next, I go through my calculations for each of the three factors.   

##1. Upside 
To calculate this, I simply use [Tom Tango's WE Matrix] (BigTable-Table 1.csv) to calculate the current WE given the game state. Then, I plug in the game state if the call is overturned. The increase in probability is the upside. 

##2. Downside   
Essentially, I'm trying to capture the "opportunity cost" of losing the ability to challenge for the remainder of the game. This is further complicated by the fact that, even if a manager loses a challenge, he can still request the umpire to initiate a subsequent one out of his own accord. So, I tried a few approaches. First, I tried to match each replay with Fangraph's [game logs] (http://www.fangraphs.com/plays.aspx?date=2016-09-18&team=Mets&dh=0&season=2016), which capture the WE after every at bat. I would select all lost challenges and find the differences between the WE after the lost challenge and the final WE (i.e. either 0 or 1). I assumed that, if there was in fact an opportunity cost, there would be a small net loss in the average change. However, the data wasn't neat enough to match up too easily, so the approach wasn't feasible.     

Instead, I took the average win percentage of a game after a team just lost a challenge (the win/loss result was obtained via the [mlbgame python package](https://pypi.python.org/pypi/mlbgame/)). The only thing is that this "swing" below .500 is not totally attributable to a lost challenge since I'm already biasing the sample to plays that were just challenged (which always follows an adverse call). So, to offset this, I calculated the net win percentage gain above .500 for successful challenges. I assumed that the greater dip below .500 for a lost challenge vs. the gain above .500 for a successful challenge is attributable to "opportunity cost." Furthermore, I assumed that this number decreases linearly with the number of outs remaining, since the lost "opportunities" are proportional to the number of outs. However, the number I got seemed to be too high, as a team losing a challenge at the beginning of a game would lose about 9% WE.     

So, I tried a third approach. I simply did the same as the second except limited my sample to the 1st inning and did not offset it against a positive "swing" from an overturned call (thus "rounding up"). I divided by the average number of outs remaining, which lead to my estimated figure: .03% decrease in WE per out remaining.    

Either way you slice it, there were a lot of different shifts in how a lost challenge affected the chances of winning from inning to inning, which I do not have an intuitive explanation for other than attribute it to small sample sizes. But, either way, I think it's safe to assume that the "cost" is very low. In fact, I managed to get this quote directly from Tom Tango himself via direct email, "There's not much cost to consider."

##3. Probability of Call Being Overturned
