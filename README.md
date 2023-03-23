# Predicting-Winability-In-LoL-At-15

Our exploratory data analysis on this dataset can be found at https://sean1572.github.io/LOLMatchAnaysis/. 

League of legends isn't always the most amusing game to play, espically when game times can take anywhere from 20 mins to an hour. Luckily the developers added the ablity to surrender early at 15mins (soon to be 10!) but how would you know when to quit? What if the match you thought was unwinable, could be winnable? Can a model tell you when its best to quit?

# Framing the Problem

We are going to create a binary classifier to predict by the 15 minute mark if a game is winnable or likely to lose. We will use the data from https://oracleselixir.com/ which contains a large dataset for pro play League of Legends games. The output of this model will be either win for if the game is predicted to be a winnable game or lose if the game is predicted to be a losing game (that way players don't have play a 45 minute long game of league just to lose when they could have quit at 15 minutes and got into a winning game and have more fun!). 

To evaluate the model we are going to use accuracy. Frist off, the dataset itself contains data for a pro play game for both the losing and winning team. This means the dataset contains roughly 50% win and 50% lose classes, leading to low class imbalance (accuracy is a poor mertic for class imbalance)!. Secondly, we don't nessarily care about what kinds of mistakes the model makes, we just want it to make fewer mistakes. Namely, if we falsely say a game will win, then the player will play the game and end with a lost which is a lot of time wasted for the player. If the model tells the player the game is lost but could have been won, the player ends up with a lost on thier match history. Thus its important for the model to be as accurate as possible for the best possible experience for a player. So any mistake is equally bad. So since we don't nessarily want to control for any particluar error and class imbalance is negliable, accuracy is a decent choice of evaluation. 

In the ideal use case of this model, where the user won't end up losing the game by carefully inputting all the nessarily data into the model then running it while the games plays out in the background, the user will likely have the model predict at the end of 15 minutes. The dataset given has serval statistics for what happens at the end of 15 minutes, while other statistics are aggreates for the whole game. Obviously we do not have access to aggreates statstics for the entire game if only the frist 15 minutes have been played so we will remove those.

The features we have access to from the dataset then the champions each team bannen, the champions thier players played, if the team was playing on the blue side of the map, and all the varied at 15 stats and at 10 stats (namely 'goldat10', 'xpat10', 'csat10', 'opp_goldat10','opp_xpat10', 'opp_csat10', 'golddiffat10', 'xpdiffat10','csdiffat10', 'killsat10', 'assistsat10', 'deathsat10','opp_killsat10', 'opp_assistsat10', 'opp_deathsat10', 'goldat15','xpat15', 'csat15', 'opp_goldat15', 'opp_xpat15', 'opp_csat15', 'golddiffat15', 'xpdiffat15', 'csdiffat15', 'killsat15','assistsat15','deathsat15', 'opp_killsat15', 'opp_assistsat15',
'opp_deathsat15').` But does the player have access to this data before a match is played out? For the most part, yes! When a player hits tab on the game, the player can see most of these stats while the game is being played (or at least be able to add it up or compute it for the data provided at 10 and 15 minutes). They player would know which side they are on, who got banned, and what champions each role is playing. Everything expect any stats about enemy XP and Gold, and team XP. So for convinence we will stick to only looking at what side is being played on, the bans, the champions a team is playing and all at 10 or at 15 stats that don't include gold or xp as those are the stats a user has access to at the time. The computing the diffrences can be automated and in this ideal senerio, we will assume that a user will note all these statistics while in game (which realisitically would probably cause the game to be lost since the player would have to spend serval minutes pausing to write down everything at 10 and 15 minutes into the game but we will overlook that point for now since techically the data is accessible to the player)

# Baseline Model

For this classification work, we will use a KNeighborsClassifier with 5 neighbors to predict on the result of the match. For the features, we will use all the features described above which we will repeat here again:

 - 'onblueside', ordinal, boolean, True if On Blue Side of Map! Since its a boolean its techically already one_hot_encoded
 - ['ban1', 'ban2', 'ban3', 'ban4', 'ban5'] Ordinal, the names of the champions that team banned from anyone using. These datapoints were one_hot_encoded prior to feeding into the model.
- ['top', 'jng', 'mid', 'bot', 'sup'], Ordinal, each column contains the name of the champion that role in the team played (each rol is the column name!) These were also one hot encoded.

Note the folllowing features all are quantiative and follow the same format. X_at10 or X_at15 means a measurement of X at 10 or 15 minutes respectively. Xdiffat10 or Xdiffat15 is just the diffrence between the enemy team and current team in that stat. Any stat starting with opp is describing how far the enemeny team is in that stat.

- ['csat10', 'opp_csat10','csdiffat10','csat15', 'opp_csat15',
       'csdiffat15',], quanitative, keeps track of the number of creatures each team has killed by now
- ['killsat10','opp_killsat10','killsat15','opp_killsat15',], quanitative, number of champion kills the team has
- ['assistsat10','opp_assistsat10', 'opp_assistsat15','assistsat15'], quanitative, number of champion assits the team has
- ['assistsat10','opp_assistsat10', 'opp_assistsat15','assistsat15'], quanitative, number of champion assits the team has

- ['deathsat10', 'opp_deathsat10', 'deathsat15','opp_deathsat15'] quanitative, number of deaths the team has


When training a model on this data, the model does pretty decently for a baseline. We acheive a training accuracy of 0.77 and a testing acuracy of about 0.66. This model accuracy is okay however the model is very overfit to our training data based on the massive diffrence between testing and training. This means that if we attempted to use the model on unseen data, it will have far worse performance. Thus the model is not very good at generalizing to our data set and is not as good of a model in that case. 

# Final Model

We added a few features and improvements to the model, namely improved one hot encoding to put any infrequently used champions into a category of thier own, computing how fast a team grew between the 10 and 15 minute point, a KDA (kills deaths assits stat computed as (Kills + assits)/death) and a custom function I created called computing the snowballing effect which takes the growth rates of all the at10 stats and 15stats, mutliplies all the states for the current team and the enemy team, and then subtracts this overallgrowth rate from team A to team B. The idea of this stat is to try and give a guess as to how much more quickly one team is growing over the other (the faster a team grows, the furter they get ahead, the more likely that team is to "snowball" and get so far ahead they win). 

I believe these features are good becuase they reduce some complexity and give a sense of how the game is going to be in the future

- Frist off most champions are played infrequently for each role. That means there are a few champions that work really well in that role and most champions outside of that are not very well suited for it. So encoding every champion played is a lot of noisy data. It much more imporant to know if the champion was off meta or one of the few more important champions in each role. So limiting the number of champions with an infrequent one hot encoding is useful.

- We have a lot of features in our data and alot of it can be put together in pretty common ways in the world of league of legends stats. A very simple one many players use as a single point statsistic to show the skill of a team is KDA. So we can have 3 serpeate columns or combine them all together in a single KDA statstics, hopefully reducing the complexity of the model and reduce overfitting. 

- Then with growth rate and snowballing, we hopefully get some insight into how quickly the teams are growing. A faster growing team will likely stay ahead and get furter ahead in a game unless something drastic changes. So knowing how well a team is getting ahead is a great way to estaimate what thier future performance is going to be like. 

Now we take these features and see what works best for the model. Frist I designed mutliple models with combinations of these above computed features (for now I didn't remove the old features and replaced them with the new computed features, just getting a small baseline). To determine what model performed well I used cross validation with a k-flod of 5 to get a massive array of accuracies for each validation dataset. I then choose the model with the best adverage validation score then procedded to try removing the baseline features with the engineered features. 

In the end, the model which performed the best was KDA and Snowballing!  This model kept all the original raw data and added KDA and snowballing as extra features while one hot encoding all champion data without limiting the number of one hot columns. I then experiment with replacing the original raw data with KDa and snowballing to see if the model performed better this those aggerate statsitics (we want to try to reduce complexity based on the baseline model after all). So what I did is create a system for which I could toggle if the input to the transformations would be persered or replaced by the engineered features then test diffrent permutations of that data. I found that keeping all kill death, assits data but removing all the data to compute snowballing slightly increased validation accuracy and so my find model was a model that computed KDA, keeping all extra data about the kills, deaths, and assits, and computed snowballing from the other at10stats. 

Next we needed to do hyperparameter tuning. Since we are still using a KNeighborsClassifier, we need to tune for the ideal number of neighbors. Like selecting a model, I created many models with the hyperparameters via manually running cross validation score. From 1 neighbor to 100, the best cross validation avergae accuracy occured with a neighbor of 45.

So our best model to recap one-hot encodes the champion data as normal, gets the KDA at 10 and 15 for both the current and emnemt team, and computes the snowballing affect dropping the gold earned feature. This transformed data is passed into a KNeighborsClassifier with 45 neighbors. Now we can evaluate the model on the test dataand find that our overall training accuracy is 0.73 and our overall testing accuracy is 0.71. 

Thus this model is much more generalizable over the previous model and has a higher testing accuracy in turn by 0.05 accuracy points. This improvement means that the model will be more reliable on unseen data and thus is a better model overall. 


# Fairness Analysis

Now lets see if the model treats some games unfairly over other games. In this case, lets look at longer games vs shorter games. Now game length was never a factor in the prediction so one would expect the model wouldn't nessarily have worse perforamnce over time. To test this lets frist define our groups more spefifically

- GROUP A: Games with longer than adverage playtime (the adverage was found by looking at all gamelengths from our original dataset)
- Group B: Games with shorter than or equal to adverage playtime

Since we used accuracy for all our evluations so far we used accuracy.

To test to see if the two distrubtons have diffrent accuracies, we use a permutation test

- Null hypothesis: Group A and Group B have the same accuracies and any differences is due to random chance
- Alterantive hypothesis: Games with longer gamelegnth have a greater accuracy than games with a shorter gamelith. 

Our signicant value is 0.05. We got a p-value of 0.0 so we reject the null. This doesn't mean the alterantive hypothesis is true, it just implies the null hyptheiss MAY not nessarily hold true. We cannot say anything defintive from a hypothesisi test afterall. 
