<!---
---
layout: post
comments: true
title:  "Predicting Outcomes of Professional LoL Matches Based on Team Compositions - Part 1"
date:   2916-08-26 14:00:00 +0100
categories: jekyll update
---
-->
In this post, I describe a little project where I attempt to use an [Artificial Neural Network](https://en.wikipedia.org/wiki/Artificial_neural_network) (ANN)
to predict the outcomes of professional League of Legends (LoL) matches based on the team compositions selected by two opposing teams of five professional players each.
LoL is a popular video game with a large [eSports](https://en.wikipedia.org/wiki/ESports) scene. Every LoL match starts with a "Picks & Bans" phase,
where teams ban a few "champions", and then select one champion from the remaining pool of unbanned champions for each of their players to play in that game.
Different champions have different strengths and weaknesses. This means that certain champions can be strong or weak individually against certain other champions
that the opposing players may select, and certain champions can also work well or work badly together with other champions on the same team. We'll define the team
composition of a team in a match as the set of five champions selected by that team (note that this means that we're not actually looking at which human players are
playing in the team, just which characters they play in the video game). The goal is to use Machine Learning (in this case, a Neural Network, but other approaches may be
equally good or even better) to predict which team is going to win based on the selected team compositions of two opposing teams.

Why are we doing this? Partially it's just a fun project to see to what extent this is possible. However, I also think this could provide useful information to players,
coaches and/or analysts of professional teams when preparing for matches or reviewing previously played matches.

This post provides a non-technical description of an initial test / proof of concept of this idea, and the results. This should hopefully be understandable without any
prior knowledge of AI / Machine Learning, or the game League of Legends. The next post (part 2) will be longer, contain a lot more details, and require some understanding
of Machine Learning.

# Goal
---

So, the plan is to build a system that will take input that looks, for example, as follows:

* Blue team picked Shen, Nocturne, Karthus, Ashe, Tahm Kench
* Red team picked Riven, Shaco, Yasuo, Graves, Blitzcrank

and we expect it to provide output that looks, for example, as follows:

* Blue team win probability estimated at 0.83 (= 83%)
* Red team win probability estimated at 0.17 (= 17%)

Now, we're not interested in building a system where we come up with a set of rules ourselves that determines which of two team compositions is going to beat the other,
so the best (and coolest) option left is to use Machine Learning. The basic idea is to show a computer program a bunch of data that contains the input and output described
above, and have it detect patterns and "learn" from that data. An ANN is a specific implementation of such a technique, which has been successful so far in many domains
(such as recognizing whether there is a dog or a cat on an image). It typically requires a large amount of data (many examples of team compositions and indications of which
composition is the strongest). This raises an important question: 

# What Data Do We Use?
---

There are three possibilities that I considered:

1. **Live Server data:** Team compositions and match outcomes from the live server (non-professional matches). With millions of players, there is definitely
enough data available here. I did not choose for this option though, because I don't think non-professional matches are representative of the gameplay in
professional matches.
2. **Manually annotated data:** A list of opposing pairs of team compositions, annotated by human experts to indicate which team composition in every pair
would beat the other. As far as I'm aware, no such dataset exists, so this is not really an option currently.
3. **Professional games data:** A collection of professional matches that have been played in the past, containing at least data on which champions were selected
and which team won. This is also not perfect, because even professional players frequently make mistakes during gameplay and can lose matches in which they selected
the better team composition. Thanks to Clayton Thorrez (also known as [EsportsDataScience](https://twitter.com/EsportsDS)), I was able to get access to such a dataset 
describing 11,167 different professional matches that have been played between 18-06-2011 and 03-08-2016. This is the option I went for, because it seemed better than 
the first option, and more feasible than the second.

# Analysis of the Data
---

So, we have a set of professional LoL matches that have been played in the past (to be precise, 11,162 matches after getting rid of matches for which the result or
champion picks were not recorded). We want to train our model by showing it each of these matches, so that it can hopefully learn something interesting about combinations
of champion picks that result in an increased win percentage against certain other combinations of champion picks. There are a few issues that we need to take into account.

1. **Strengths and weaknesses of champions change over time:**
Riot Games (the developer of LoL) regularly updates the game with new patches. These patches typically modify a number of champions, for example by reducing the damage
that a spell deals of a champion that is perceived to be too strong. Sometimes there are also larger overhauls that can completely change how a champion works, or changes
to global systems of the game that entirely change how the game is played. Patches can also introduce new champions, which can cause existing champions to become stronger 
or weaker if the new champion is strong with or against the old champion. For this reason, I extended the input of the system with the value of the patch on which a match
was played. This should hopefully enable the system to recognize if, for example, a certain champion was only really strong a few years ago.

2. **Teams with stronger players may win even with a weaker team composition:**
Selecting the team compositions is only the first part of every match. It is still followed by the "real" game, which typically consists of 20-40 minutes (but sometimes
as much as 80 minutes) of gameplay, in which the players control the champions they selected for their team compositions. It is very well possible that a team with better
human players picks a team composition that is not favored to win against the opposing team composition, but still wins due to superior skill (or simply a bit of luck).
This is important for two reasons. The first reason is that it means there is a lot of *noise* in our training data. There will be matches in the training data where a
a weaker team composition beat a stronger team composition, and we will show it to our model in an attempt to teach it which team compositions are strong; from such matches
it will learn exactly the opposite of what it's supposed to learn! The second reason is that we need to take this into account when evaluating how well the trained model
performs. We certainly can't expect it to predict the outcomes of 100% of the matches correctly based only on selected team compositions. Even if we assume that the model 
can perfectly classify which of two team compositions should beat the other 100% of the time (which is a very unreasonable assumption), that still wouldn't be enough to
achieve 100% accuracy on real data where sometimes the "incorrect" team ends up winning.   
 
	In an attempt to handle this issue, I extended the input of the system to also include an estimate of each of the team's overall skill in the form of 
[TrueSkill rating](https://www.microsoft.com/en-us/research/project/trueskill-ranking-system/). This can help the model to sometimes explain a result that would otherwise
be inexplicable based only on team compositions. This is not a perfect solution either, since the TrueSkill rating itself is just an estimate of team skill, but it could
help.

3. **Players Make Mistakes:**
This point is closely related to the point above. In addition to teams with stronger human players winning games with weaker team compositions, as described above, it
is also possible that teams with equally skilled or weaker human players **and** weaker team compositions still win. Even though we're talking about professional LoL
players, they're still human and they still (frequently) make mistakes. More skilled players and stronger team compositions are favored to win more games on average,
but weaker players with weaker team compositions still can win matches in which their stronger opponents just happen to underperform and make a bit more mistakes than
they usually do. The implications of this issue are the same as described above; we have noisy data, and there is an upper bound (lower than 100%) on the prediction accuracy
that we can expect to obtain.

# Results
---

The dataset as described above was randomly split into a *training set* consisting of 90% of the data, and a *validation set* consisting of the remaining 10% of the data. The
training set was used for training the model, and the validation set is used to determine how well the trained model performs by comparing its predictions on that set to the
actual outcomes of those games. This distinction is important to make, because evaluating the performance of a trained model on the same data on which it was trained can
produce misleading results; maybe the model simply learned the outcomes of all those matches by heart!

On this validation set, the trained model was able to correctly predict the outcomes of 62.6% of the matches (699 correct predictions, 417 incorrect predictions). On the
complete set (training + validation), this percentage is slightly higher at 63.2%, but this includes matches that were also previously shown to the model for training.
A more detailed description of results, with many cool plots and other numbers, will be included in the more detailed part 2. Now, the real question is: *how good are these
results?*

As was discussed in the analysis of the data, we know that having the better team composition does not guarantee a win. This means that 100% accuracy is definitely impossible.
It is difficult to estimate what kind of accuracy should be possible though. To what extent does having a stronger team composition increase a team's chances of winning? It
would be interesting to get opinions from domain experts on this, but for now the best I can do is comparing to the prediction accuracy of some other methods:

1. **Coin Flip:** A coin flip would give us 50% prediction accuracy. Luckily we're doing better than that, but obviously this doesn't mean anything.

2. **Always predict Blue to win:** This would give us a prediction accuracy somewhere in the range of 54% to 56% according to
[EsportsDataScience](https://www.reddit.com/r/leagueoflegends/comments/4njrbm/blue_vs_red_advantage_over_6_years_and_across_all/). It's good to see we're
doing better than that, which means the model at least learned *something* useful.

3. **Predict based on TrueSkill Rating:** Remember how I included an estimate of each team's skill in a match, based on TrueSkill Rating, in the input of the model? A
simple predictor could be built to compare the TrueSkill Ratings of two teams directly, and predict the team with the higher TrueSkill Rating to win. Such a predictor
gets an accuracy of 60.9% on the complete set, and 46.7% on the smaller subset that was randomly selected for validation. This tells us two things. First off, with 63.2% 
we're doing at least a little bit better than simply predicting the team with the highest TrueSkill Rating to win. The difference is only a bit over 2% though, which is not 
a lot. Secondly, the trained model performs *much* better on the smaller, randomly selected validation set. The poor performance of predicting based on TrueSkill Rating 
may mean that there were relatively many "upsets" (with weaker players beating stronger players) in this set. The significantly better performance (62.6%) of the trained
model on these games provides some evidence that it has learned something about what good team compositions look like, and can apparantly correctly predict the outcomes 
of these upsets based on the selected team compositions.

4. **Better Machine Learning Model(s):** I still have many ideas to improve the chosen model, or other Machine Learning techniques to use instead of an ANN, which may
or may not lead to a better performance. These will be described in more detail in part 2. 

I do think that the 62-63% accuracy obtained with this model is a bit low, but I don't think it is too bad either considering the factors described in the analysis
of the data above. I may look into improving this in the future if I find time, and will write new posts if any interesting results do show up.

# Predictions for NA and EU LCS 2016 Summer Playoffs
---

I have also taken a look specifically at some of the most recently played games - games from the NA and EU LCS 2016 Summer Playoffs - to see what the model predicts for
those matches. Note that this is just a small sample size of games, and does not really have any statistical meaning at all... but it's still fun to look at. I entered the 
input (team sides, champion picks) for these games manually, so please let me know if you notice any mistakes! Correct predictions are in boldface.

**EU LCS 2016 Summer Playoffs**

| Game (Blue Team vs. Red Team - Game #)| Blue Team Picks									| Red Team Picks									| Predicted Winner		| Actual Winner		|
| :-----------------------------------:	| -------------------------------------------------	| -------------------------------------------------	| :-------------------:	| :---------------:	|
| Giants vs. Unicorns of Love - Game 1	| Gnar, Gragas, Vladimir, Jhin, Karma				| Kennen, Rek'sai, Ryze, Ashe, Braum				| Giants				| Unicorns of Love	|
| Unicorns of Love vs. Giants - Game 2	| Irelia, Graves, Kassadin, Ashe, Alistar			| Trundle, Elise, Ekko, Sivir, Taric				| **Giants**			| Giants			|
| Unicorns of Love vs. Giants - Game 3	| Gnar, Rek'sai, Vladimir, Corki, Bard				| Irelia, Elise, Cassiopeia, Sivir, Tahm Kench		| **Unicorns of Love**	| Unicorns of Love	|
| Unicorns of Love vs. Giants - Game 4	| Irelia, Rek'sai, Kassadin, Lucian, Braum			| Gnar, Elise, Ryze, Sivir, Alistar					| Giants				| Unicorns of Love	|
|										|													|													|						|					|
| H2K vs. Fnatic - Game 1				| Ekko, Elise, Vladimir, Tristana, Taric			| Gangplank, Nidalee, Lissandra, Sivir, Braum		| Fnatic				| H2K				|
| Fnatic vs. H2K - Game 2				| Gnar, Graves, Lissandra, Lucian, Braum			| Gangplank, Nidalee, Vladimir, Sivir, Taric		| **H2K**				| H2K				|
| H2K vs. Fnatic - Game 3				| Shen, Hecarim, Vladimir, Sivir, Taric				| Gangplank, Nidalee, Cassiopeia, Lucian, Trundle	| Fnatic				| H2K				|
|										|													|													|						|					|
| H2K vs. Splyce - Game 1				| Ekko, Rek'sai, Taliyah, Lucian, Taric				| Gnar, Gragas, Vladimir, Ashe, Tahm Kench			| **H2K**				| H2K				|
| H2K vs. Splyce - Game 2				| Ekko, Rek'sai, Vladimir, Ashe, Taric				| Gnar, Gragas, Taliyah, Jhin, Tahm Kench			| H2K					| Splyce			|
| H2K vs. Splyce - Game 3				| Gangplank, Rek'sai, Vladimir, Ashe, Tahm Kench	| Gnar, Gragas, Taliyah, Jhin, Braum				| **H2K**				| H2K				|
| H2K vs. Splyce - Game 4				| Gangplank, Rek'sai, Vladimir, Ashe, Tahm Kench	| Gnar, Gragas, Malzahar, Sivir, Karma				| **Splyce**			| Splyce			|
| H2K vs. Splyce - Game 5				| Gangplank, Rek'sai, Vladimir, Lucian, Karma		| Gnar, Gragas, Malzahar, Sivir, Tahm Kench			| **Splyce**			| Splyce			|
|										|													|													|						|					|
| G2 vs. Unicorns of Love - Game 1		| Gangplank, Rek'sai, Vladimir, Jhin, Tahm Kench	| Gnar, Elise, Anivia, Ashe, Braum					| Unicorns of Love		| G2				|
| Unicorns of Love vs. G2 - Game 2		| Shen, Gragas, Kassadin, Lucian, Taric				| Gnar, Rek'sai, Lissandra, Sivir, Trundle			| G2					| Unicorns of Love	|
| Unicorns of Love vs. G2 - Game 3		| Shen, Gragas, LeBlanc, Lucian, Taric				| Gnar, Rek'sai, Taliyah, Sivir, Soraka				| **G2**				| G2				|
| G2 vs. Unicorns of Love - Game 4		| Gnar, Elise, Malzahar, Tristana, Braum			| Irelia, Gragas, Vladimir, Sivir, Taric			| Unicorns of Love		| G2				|
|										|													|													|						|					|
| Unicorns of Love vs. H2K - Game 1		| Gangplank, Rek'sai, Vladimir, Ashe, Tahm Kench	| Shen, Gragas, Cassiopeia, Sivir, Karma			| **H2K**				| H2K				|
| Unicorns of Love vs. H2K - Game 2		| Gnar, Rek'sai, Vladimir, Lucian, Trundle			| Shen, Gragas, Cassiopeia, Ashe, Tahm Kench		| H2K					| Unicorns of Love	|
| Unicorns of Love vs. H2K - Game 3		| Gnar, Rek'sai, Viktor, Lucian, Trundle			| Shen, Elise, Vladimir, Tristana, Braum			| Unicorns of Love		| H2K				|
| Unicorns of Love vs. H2K - Game 4		| Kennen, Gragas, LeBlanc, Lucian, Braum			| Gnar, Nidalee, Taliyah, Sivir, Taric				| **H2K**				| H2K				|
|										|													|													|						|					|
| Splyce vs. G2 - Game 1				| Shen, Hecarim, Syndra, Sivir, Tahm Kench			| Gangplank, Gragas, Ekko, Lucian, Trundle			| **G2**				| G2				|
| G2 vs. Splyce - Game 2				| Gnar, Rek'sai, Ekko, Lucian, Soraka				| Shen, Gragas, Vladimir, Sivir, Karma				| G2					| Splyce			|
| G2 vs. Splyce - Game 3				| Gnar, Gragas, Vladimir, Sivir, Tahm Kench			| Shen, Rek'sai, Malzahar, Caitlyn, Karma			| **G2**				| G2				|
| G2 vs. Splyce - Game 4				| Shen, Elise, Vladimir, Jhin, Braum				| Fiora, Rek'sai, Ryze, Ashe, Tahm Kench			| **G2**				| G2				|

<br>
**NA LCS 2016 Summer Playoffs**

| Game (Blue Team vs. Red Team - Game #)| Blue Team Picks									| Red Team Picks									| Predicted Winner		| Actual Winner		|
| :-----------------------------------:	| -------------------------------------------------	| -------------------------------------------------	| :-------------------:	| :---------------:	|
| Cloud 9 vs. Team EnVyUs - Game 1		| Gnar, Rek'sai, Syndra, Ashe, Tahm Kench			| Kennen, Gragas, Lissandra, Sivir, Braum			| Cloud 9				| Team EnVyUs		|
| Cloud 9 vs. Team EnVyUs - Game 2		| Gnar, Rek'sai, Syndra, Ashe, Thresh				| Gangplank, Gragas, Lissandra, Jhin, Braum			| **Cloud 9**			| Cloud 9			|
| Cloud 9 vs. Team EnVyUs - Game 3		| Gnar, Gragas, Cassiopeia, Jhin, Tahm Kench		| Kennen, Rek'sai, Jayce, Ashe, Thresh				| **Cloud 9**			| Cloud 9			|
| Team EnVyUs vs. Cloud 9 - Game 4		| Lissandra, Lee Sin, Pantheon, Jhin, Karma			| Gnar, Gragas, LeBlanc, Ashe, Thresh				| **Cloud 9**			| Cloud 9			|
|										|													|													|						|					|
| CLG vs. Team Liquid - Game 1			| Gnar, Gragas, Syndra, Jhin, Karma					| Irelia, Rek'sai, LeBlanc, Ashe, Tahm Kench		| Team Liquid			| CLG				|
| CLG vs. Team Liquid - Game 2			| Gnar, Rek'sai, Viktor, Ashe, Bard					| Irelia, Gragas, Cassiopeia, Jhin, Braum			| **CLG**				| CLG				|
| CLG vs. Team Liquid - Game 3			| Gnar, Gragas, Viktor, Ezreal, Bard				| Irelia, Rek'sai, Cassiopeia, Ashe, Braum			| **Team Liquid**		| Team Liquid		|
| CLG vs. Team Liquid - Game 4			| Gnar, Olaf, Aurelion Sol, Sivir, Bard				| Irelia, Gragas, Cassiopeia, Ashe, Braum			| **CLG**				| CLG				|
|										|													|													|						|					|
| Immortals vs. Cloud 9 - Game 1		| Rumble, Elise, Taliyah, Jhin, Trundle				| Gangplank, Rek'sai, LeBlanc, Sivir, Braum			| Cloud 9				| Immortals			|
| Cloud 9 vs. Immortals - Game 2		| Gangplank, Gragas, Syndra, Sivir, Braum			| Riven, Rek'sai, LeBlanc, Jhin, Trundle			| Immortals				| Cloud 9			|
| Immortals vs. Cloud 9 - Game 3		| Kennen, Rek'sai, Viktor, Ashe, Taric				| Gnar, Gragas, Zilean, Sivir, Trundle				| **Cloud 9**			| Cloud 9			|
| Cloud 9 vs. Immortals - Game 4		| Ekko, Rek'sai, Lissandra, Jhin, Tahm Kench		| Gangplank, Elise, Taliyah, Ashe, Taric			| Cloud 9				| Immortals			|
| Immortals vs. Cloud 9 - Game 5		| Lissandra, Rek'sai, Taliyah, Jhin, Trundle		| Ekko, Gragas, Syndra, Ashe, Tahm Kench			| Immortals				| Cloud 9			|
|										|													|													|						|					|
| Team SoloMid vs. CLG - Game 1			| Gangplank, Rek'sai, Cassiopeia, Jhin, Trundle		| Gnar, Gragas, Taliyah, Sivir, Soraka				| **Team SoloMid**		| Team SoloMid		|
| CLG vs. Team SoloMid - Game 2			| Ekko, Gragas, Syndra, Caitlyn, Karma				| Gnar, Rek'sai, Lissandra, Sivir, Braum			| **Team SoloMid**		| Team SoloMid		|
| Team SoloMid vs. CLG - Game 3			| Gangplank, Rek'sai, Lissandra, Sivir, Braum		| Ekko, Gragas, Kassadin, Jhin, Karma				| **Team SoloMid**		| Team SoloMid		|
|										|													|													|						|					|
| Immortals vs. CLG - Game 1			| Gnar, Elise, Karma, Ashe, Trundle					| Shen, Rek'sai, Syndra, Jhin, Braum				| Immortals				| CLG				|
| CLG vs. Immortals - Game 2			| Shen, Rek'sai, Malzahar, Kalista, Alistar			| Ekko, Gragas, Vladimir, Sivir, Trundle			| **Immortals**			| Immortals			|
| CLG vs. Immortals - Game 3			| Shen, Rek'sai, Kassadin, Jhin, Braum				| Ekko, Gragas, Vladimir, Ezreal, Karma				| CLG					| Immortals			|
| CLG vs. Immortals - Game 4			| Shen, Rek'sai, Vel'Koz, Jhin, Braum				| Gnar, Elise, Cassopeia, Sivir, Trundle			| Immortals				| CLG				|
| Immortals vs. CLG - Game 5			| Ekko, Gragas, Vladimir, Ezreal, Taric				| Shen, Rek'sai, Malzahar, Sivir, Braum				| CLG					| Immortals			|
|										|													|													|						|					|
| Team SoloMid vs. Cloud 9 - Game 1		| Ekko, Gragas, Vladimir, Jhin, Tahm Kench			| Shen, Rek'sai, Cassiopeia, Ashe, Trundle			| **Cloud 9**			| Cloud 9			|
| Cloud 9 vs. Team SoloMid - Game 2		| Gnar, Gragas, Vel'Koz, Ashe, Trundle				| Ekko, Rek'sai, Cassiopeia, Jhin, Braum			| Cloud 9				| Team SoloMid		|
| Team SoloMid vs. Cloud 9 - Game 3		| Ekko, Rek'sai, Cassiopeia, Jhin, Bard				| Gnar, Zac, Vel'Koz, Ashe, Trundle					| **Team SoloMid**		| Team SoloMid		|
| Cloud 9 vs. Team SoloMid - Gamae 4	| Gnar, Rek'sai, Taliyah, Jhin, Braum				| Shen, Gragas, Vladimir, Lucian, Trundle			| **Team SoloMid**		| Team SoloMid		|

<br>

In total for these NA and EU LCS 2016 Summer Playoffs games, the model correctly predicted 55.10% of the games. As mentioned before, this sample size is not big enough to really mean anything.
It should also be noted that these matches were played on patch 6.15, whereas the data used for training contains very few matches from that patch. This patch in particular changed how the game
is played professionally quite drastically (the changes were so big that this patch is one of the reasons for some 
[recent drama](https://esports.yahoo.com/not-brief-timeline-current-league-000000257.html) in the LoL eSports scene). Such big changes are likely to negatively affect the model's predictive
power when the training data does not contain a sufficient number of matches from that patch. Of course, it is likely that the model was actually correct in terms of team composition strength
in more (or less) than exactly 55.10% of the games, because most games are really decided in the gameplay after champion selection. Whether some of these incorrect predictions actually do make
sense looking only at team compositions is up to domain experts to determine.

A much more detailed description of the techniques used, and a proper analysis of these (and more) results, will be included in the next post (part 2). That post will be a bit more heavy on
Data Science / Machine Learning. Please let me know what you think of the post, and if you like this kind of content!

# Acknowledgement
---
Thanks to Clayton Thorrez for providing me access to his dataset of professional LoL matches.