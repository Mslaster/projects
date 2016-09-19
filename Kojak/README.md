# Optimizing MLB Manager's Challenge   

For my capstone project I decided to tackle the issue of when to optimally use a manager's challenge in baseball.
Starting in 2014, Major League Baseball instituted a new rule where a manager had the right to "challenge" a call by an umpire (for certain types of calls). 
The call is then reviewed by a crew of umpires in New York City through multiple camera angles. If the manager is correct, the call is overturned. If the umpire is correct or the video is inconclusive, then the manager loses the ability to challenge for the remainder of the game. So, my goal was to model when a manager should optimally attempt to challenge a possible mistaken call.   

## Units of Measurement   
First, I had to determine how exactly I'd capture what "success" or "failure" meant. To do this, I turned to the [Win Expectancy] (http://www.fangraphs.com/library/misc/we/) stat developed by Tom Tango. This essentially is a matrix to calculate the probability a team will win a game given the game state (i.e. score differential, innings, runners on base, etc.). So, I use the difference in WE to capture how "good" or "bad" a certain event is.    

## Formula    
To figure out whether to challenge, there are three things to take into account. 1. The potential upside if successful. 2. The downside if unsuccessful. 3. The probability of success. If the probability of success times the gain in the upside is greater than the probability of failure times the loss, then a manager should challenge. Otherwise he shouldn't. Next, I go through my calculations for each of the three factors.   

##1. Upside 
To calculate this, I simply use [Tom Tango's WE Matrix] (BigTable-Table 1.csv) to calculate the current WE given the game state. Then, I plug in the game state if the call is overturned. The increase in probability is the upside. 
