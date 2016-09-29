<!---
---
layout: post
comments: true
title:  "Predicting Outcomes of Professional LoL Matches Based on Team Compositions - Part 2"
date:   2916-08-26 14:00:00 +0100
categories: jekyll update
---
-->
My [previous post]({% post_url 2016-08-31-predicting-outcomes-of-professional-lol-matches-part1 %}) provides a non-technical description of a proof of concept for using an Artificial
Neural Network (ANN) to predict the outcomes of professional League of Legends (LoL) matches, based on selected team compositions. This post contains a more formal, technical, and
detailed description of the process resulting in that proof of concept. This post assumes familiarity with Machine Learning, and at least basic knowledge of how an ANN works. The
[previous post]({% post_url 2016-08-31-predicting-outcomes-of-professional-lol-matches-part1 %}) is recommended reading for some more context on the domain (professional LoL matches).

# Data
---

The dataset that I have available (kindly provided to me by [Clayton Thorrez](https://twitter.com/EsportsDS)) contains data on 11,167 different professional LoL matches that have been played 
between 18-06-2011 and 03-08-2016. For every match, there are a lot of different features, such as the names of the two teams that played against each other in that match, names of the players
in each team, names of the champions played by each player, duration of the match, etc. This data was gathered from many different sources. There are many inconsistencies (for instance, different 
ways to spell the same team/player/champion name) and missing values (in particular for matches from the first couple of years).

# Goal
---
The goal of the project is to train an ANN to output estimates of win probabilities for two teams (Blue and Red) based on the champions selected by those two teams. Note that using an ANN to do
this is part of the goal too, because I wanted to get a bit more experience with a [particular library for ANNs](https://lasagne.readthedocs.io/en/latest/). This does not necessarily mean that I 
have any reason to believe that an ANN outperforms other kinds of models for this task, and for optimal results I'd definitely recommend to also try other techniques (such as SVMs, Random Forests, 
and combinations of multiple models). The idea is that a team could use such a model to get some insight into which out of two potential team compositions \\(T_1\\) and \\(T_2\\) would perform better 
against an expected opposing team composition \\(T_{opp}\\).

As described in part 1, the input of the model is intended to look as follows:

* Blue team picked Shen, Nocturne, Karthus, Ashe, Tahm Kench
* Red team picked Riven, Shaco, Yasuo, Graves, Blitzcrank

and the output should look like:

* Blue team win probability estimated at 0.83 (= 83%)
* Red team win probability estimated at 0.17 (= 17%)

# Network Input
---

The dataset contains 131 unique champion names (after preprocessing the data by comparing champion names to a manually constructed list of known misspellings and replacing them by the correct
names). The initial version of the input layer of the network was created to consist of 262 nodes; for each team (Blue and Red), one node for each champion. Then, a single LoL match is input
into the network by setting an input node's value to 1 if the corresponding champion was selected by the corresponding team, and a 0 otherwise. There are a few things I'd like to note for this
setup:

1. By modelling the input in this way, we ignore the fact that there are pairs of input nodes that should be very closely related. For every champion, there are two input nodes; one for Blue
side, and one for Red side. So, when we have some different team compositions on Blue side in which a certain champion (say, Alistar) works very well, the model does not learn anything at all
from this about picking the same champion on the Red side. One possible way to address this could be to duplicate our dataset, and switch our team compositions around for this duplicate. This
is not guaranteed to be correct, because there are some asymmetries on the LoL map which could affect whether or not a certain composition is strong on a certain side of the map. In practice,
I expect these effects would be fairly small and extending the dataset in this way could be beneficial.

2. In LoL, there are five "positions" to which players (and their selected champions) are assigned. These positions are Top, Jungle, Mid, Marksman and Support. By modelling the input as described
above, we ignore the positions to which champions are assigned, and therefore ignore potentially important information. A naive way to address this would be to multiply the size of our input vector
by 5; one input node per side per champion per position. Because many champions have only ever been played in one or two positions ever, it would likely be better to only create one input node
per side per champion per position in which that champion is actually ever played. 

3. Many champions have similarities to each other. For instance, there are many "tanky" champions, which are typically able to absorb large amounts of damage, or "poke" champions, which are
able to frequently deal damage from a large range. Xerath and Varus are two examples of poke champions. When training the model by showing it an example of a team composition with Xerath that
won, it would be nice if the model could also learn from that example that Varus would likely be a good fit in the same composition. Theoretically a network is indeed able to do this, by
adjusting weights of a hidden node that already has strong connections to both Xerath and Varus. This only works if the model has already seen enough example compositions with Xerath, and enough
example compositions with Varus, such that it could increase the weights from the Xerath and Varus input nodes to that hidden node. In practice, it is much more likely that the network does
not detect many such patterns, and gets stuck in a local minimum earlier. I have some ideas that I may try in the future to address this. The first idea is to initialize the network's weights by 
training them first on data from the game's live server (non-professional matches), before training on the dataset of professional matches. The dataset of unprofessional games is significantly
larger, and may result in a decent initialization of weights. A second idea is to extract additional features for every champion, such as health values, attack range, physical and magical damage
output, etc. More advanced techniques for learning features, such as clustering algorithms and Restricted Boltzmann Machines, can also be considered.

Additionally, the input layer has been extended with two more nodes; one taking the value of the season in which a game was played, and one taking the value of the patch on which a game was played.
Both values can be viewed as measures of time, indicating when the game was played. This is important to take into account, because the game is frequently changed, meaning that champions that
were strong at a certain point in time may be very weak now. The values for both nodes are normalized to lie in \\([0, 1]\\), based on the minimum and maximum observed values for each feature.

# Extending Input with TrueSkill Rating
---

A pair of team compositions alone is unlikely to be a good predictor of the outcome of a match. The main reason for this is that human players can make many mistakes during gameplay, and lose
games even with a stronger team composition based on overall gameplay skill, luck, preparation, etc. This is described in more detail in the previous post. This is an important problem, because
it means we have a lot of noise in our data. There will be many matches in the dataset where the weaker team composition won, and those matches are also used for training the model. In an attempt
to help the model explain those "incorrect" match outcomes, I extended the input to include [TrueSkill rating](https://www.microsoft.com/en-us/research/project/trueskill-ranking-system/)
estimates of the overall skill of both teams in a match. For every match, this rating is computed based on all the previous matches in the dataset. To incorporate these rating estimates in the
input, the input layer is extended with four nodes. For each team, there is an input node with the mean of the TrueSkill rating estimate, and an input node with the standard deviation of the
TrueSkill rating.

Including these estimates of overall team skill in the input can have two effects. The first is the intended effect, that it can hopefully help the model during the training phase to explain
"strange" results where a team with a stronger team composition lost. The second effect is that the model will no longer only make predictions based on selected team compositions (as was
originally intended), but also based on the perceived skill of teams of human players. This is not a problem in the example use case described in the **Goal** section, which was: *The idea is 
that a team could use such a model to get some insight into which out of two potential team compositions \\(T_1\\) and \\(T_2\\) would perform better against an expected opposing team composition 
\\(T_{opp}\\).* When using the trained model in such a way, the TrueSkill values would be the same for all different inputs for which outputs are compared to each other.

# Network Structure
---

The remainder of the network has been structured as follows. The input layer is followed by two hidden layers, of 150 nodes each. These hidden layers use the 
[tanh](https://lasagne.readthedocs.io/en/latest/modules/nonlinearities.html#lasagne.nonlinearities.tanh) activation function. The second hidden layer is followed by the output layer, consisting 
of two [softmax](https://lasagne.readthedocs.io/en/latest/modules/nonlinearities.html#lasagne.nonlinearities.softmax) nodes. The output of the first output node is interpreted as the probability
of the Blue team winning, and the output of the second node is interpreted as the probability of the Red team winning. Every layer is fully connected to the following layer. The input layer
has been implemented to have 10% [dropout](https://lasagne.readthedocs.io/en/latest/modules/layers/noise.html#lasagne.layers.DropoutLayer), and both hidden layers have 35-40% dropout. This setup
is based on some quick experiments and manual tuning, but can likely be improved.

Dropout in particular is very important to avoid overfitting. Without dropout (especially also with larger hidden layers), there is a risk of the network simply learning the outcomes of every
match in the training set by heart.

Two different loss functions have been tested to use in the training phase; [cross-entropy](https://lasagne.readthedocs.io/en/latest/modules/objectives.html#lasagne.objectives.categorical_crossentropy)
and [hinge loss](https://lasagne.readthedocs.io/en/latest/modules/objectives.html#lasagne.objectives.multiclass_hinge_loss). Cross-entropy punishes large mistakes more heavily than small mistakes,
whereas hinge loss does not. Let \\(E = 1 - p\\) denote the error of a prediction, where \\(p\\) is the probability of winning assigned by the model to the team that actually won. The hinge loss
function increases linearly as \\(E\\) increases, whereas cross-validation increases more than linearly. This means that a network trained with cross-entropy as a loss function has a tendency
to make more predictions close to 0.5 for both teams, whereas a network trained with hinge loss as a loss function has a tendency to make more predictions with probabilities far away from 0.5.

# Results
---

The performance of the network has been shortly evaluated by training it on 90% of the data (training set), and validating it on 10% of the data (validation set). For better confidence bounds
on the results, it would obviously be better to do \\(K\\)-Fold Cross-Validation (say, \\(K = 5\\) or \\(K = 10\\)). Because the training phase takes a decent amount of time, and this project
is just a quick proof of concept I worked on in my spare time, I decided to stick to only a single evaluation with 90% training and 10% validation.

The following figure depicts the training and validation loss over 2000 training epochs with **hinge loss**:
<img src="/assets/post_lol_teamcomps_nn/hingeloss_150_150_dropout01_04_04_delta07_seed12345.png">
This figure depicts the training and validation loss over 8000 training epochs with **cross-entropy**:
<img src="/assets/post_lol_teamcomps_nn/cross_entropy_150_150_dropout01_035_035.png">
The loss values for these two plots are not directly comparable, because they use different loss functions. The important thing to note is that, in both cases, the validation loss is consistently
lower than the training loss. The network does not appear to be overfitting in either case. For comparison, here is a figure without dropout in the training phase for the hidden layers:
<img src="/assets/post_lol_teamcomps_nn/no_hiddens_dropout.png">
This figure clearly shows that the network starts overfitting when there is no dropout in the hidden layers.

The model trained with the hinge loss function achieves 62.6% accuracy when predicting the outcomes of matches in the validation set (and 63.2% on training + validation set). Training with
cross-entropy as a loss function appears to perform slightly worse with 62.4% accuracy on training + validation data, but it's very close. These accuracy numbers are not spectacular, but
it should be noted that the prediction task is very difficult. We are dealing with professional matches, in which both teams try to create strong team compositions. In cases where teams don't
make huge mistakes, and aren't limited to heavily in terms of the champions that their human players feel comfortable playing, this typically results in two team compositions that are at least
close in strength, and should both have a decent chance of winning. There is also a lot of noisy training data, where a weaker team composition ended up winning due to superior play. I personally
still expect improvement to be possible (and mention many ideas throughout the post), but would be surprised if it is possible to get significantly higher than, say, 70%.

The following figure depicts the ROC curve for predicting Blue side wins (as in, a True Positive Rate of 0.6 and False Positive Rate of 0.4 means that 60% of Blue side wins are correctly
predicted, and 40% of Red side wins are incorrectly predicted) on the validation set. The area under the curve is 0.65 which, just like the accuracy, is not spectacular, but not horrible either
considering the complexity of the domain.
<img src="/assets/post_lol_teamcomps_nn/ROC_hingeloss.png">