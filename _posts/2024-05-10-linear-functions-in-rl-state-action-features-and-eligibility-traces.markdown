---
title: "Linear Functions in RL, State-Action Features, and Eligibility Traces"
date: 2024-05-20 10:30:00 +0100
categories:
  - "Reinforcement Learning" 
  - "State-Action Features" 
  - "Linear Function Approximation"
  - "Eligibility Traces"
excerpt: The standard textbook on Reinforcement Learning, by Sutton and Barto, extensively covers how state feature vectors may
  be constructed for linear state value functions. However, there is very little explanation of the extension to state-action
  feature vectors, which are required for learning state-action value functions. In this post, I aim to fill this gap.
---

**Target audience**: people learning about Reinforcement Learning (RL); already familiar with the basics of value function learning (for
state value functions \\(\hat{v}(s)\\) as well as state-action value functions \\(\hat{q}(s, a)\\)); also familiar with (linear) function
approximation for state value functions, but looking to learn to extend this to state-action value functions. Assumed knowledge roughly
corresponds to chapters 6, 9, and (for the final section of my post) 12 of 
[the second edition of Sutton and Barto's textbook on Reinforcement Learning](http://incompleteideas.net/book/the-book-2nd.html).

# State Feature Vectors
---
Linear state value functions look like \\(\hat{v}(s) = \mathbf{x}(s)^{\top}\mathbf{w} = \sum_{i=1}^d x_i(s) \times w_i\\), where:

- \\(\mathbf{x}(s)\\) denotes a \\(d\\)-dimensional feature vector representing a state \\(s\\).
- \\(\mathbf{w}\\) denotes a \\(d\\)-dimensional weight vector: these are the parameters that we will be learning.
- \\(x_i(s)\\) denotes the \\(i^{th}\\) feature from the vector \\(\mathbf{x}(s)\\).
- \\(w_i\\) denotes the \\(i^{th}\\) weight from the vector \\(\mathbf{w}\\).

Note that such a linear function cannot ever learn to look at multiple features in combination. It simply looks at and learns a weight
for each feature independently. For example, if our features in [Mountain Car](https://gymnasium.farama.org/environments/classic_control/mountain_car/)
are just the position and the velocity of the car, we will never be able to get anywhere close to learning the optimal value function, as the optimal 
value function in this environment is a nonlinear function of these two state properties. Section 9.5 of 
[the second edition of Sutton and Barto's textbook on Reinforcement Learning](http://incompleteideas.net/book/the-book-2nd.html) provides extensive
detail on a variety of techniques that may be used to construct more informative feature vectors for state value functions.

## State-Action Feature Vectors
---
If we want to learn linear state-action value functions, we will need feature vectors \\(\mathbf{x}(s, a)\\) that provide information on *state-action pairs* \\((s, a)\\),
rather than vectors that only describe states. Chapter 10 of the textbook briefly mentions this fact, but otherwise focuses solely on learning algorithms, with no further
explanation of how such feature vectors may be constructed. This is a gap that I aim to fill in this post.

There are often relatively obvious features that could be used to describe states, such as *position* and *velocity* in Mountain Car 
(which may be extended using techniques such as Tile Encodings, Polynomials, Radial Basis Functions, etc.). This is often not the case for actions. The only straightforward
featurisation of actions we usually have is a *one-hot encoding*: for an environment with \\(n\\) different actions, we can construct a vector that contains a value of
\\(1\\) at the index corresponding to the action \\(a\\) we wish to represent, and values of \\(0\\) for all other entries. The first idea we will then consider is to
construct state-action feature vectors by concatenating such a representation to the representation we would have used for the state. For example,<sup>[1](#bias)</sup> if
\\(\mathbf{x}(s) = \begin{bmatrix} 1 & x_1 & x_2 & \dots & x_d \end{bmatrix}\\) would have been our feature vector for a state \\(s\\), we might consider using
\\(\mathbf{x}(s, a) = \begin{bmatrix} 1 & x_1 & x_2 & \dots & x_d & 0 & 1 & 0 & 0 & \dots & 0 \end{bmatrix}\\) as feature vector for \\((s, a=2)\\) (I put the \\(1\\) entry
in the second slot of the "action part" of the vector for this example). **Important:** you should not actually do this, it won't work when learning linear functions! The
reason that this won't work is because, as described in the previous section, a linear function looks at each feature independently. Looking at state features and action
features independently means that it can, at best, learn what "generally good" or "generally bad" actions are over the whole state space. For any given single state, it
will never be able to learn that a certain action is good in some situations, and bad in others.

There is a good solution to this, which we may think of in two different ways (which sound different at first, but are mathematically equivalent). Perhaps the easiest
way to think of the solution is that we will not just learn a single linear function (vector of weights), but learn \\(n\\) functions (\\(n\\) different vectors of weights)
for environments with \\(n\\) actions. Then, our feature vectors can revert to simply being state feature vectors, but we pick a different vector of weights to multiply
(in a dot product) with our feature vector depending on the action for which we wish to make a prediction. For example, if we wish to estimate \\(\hat{q}(s, 3)\\) (for the
third action), we compute it as \\(\hat{q}(s, 3) = \mathbf{x}(s)^{\top}\mathbf{w}_{(3)}\\), where the subscript \\((3)\\) for the weight vector indicates that we take the
specific set of weights tailored towards that particular action.

A different way of thinking of what is mathematically the same solution (but might look a bit different in code) is that we multiply the size of our state
feature vector with the number of actions, but always zero out most of it, such that different segments of the vector are used for different actions. More precisely,
you would put together \\(n\\) copies of the state feature vector \\(\mathbf{x}(s)\\), essentially giving you a much bigger vector with \\(n\\) equal "segments."
For an input pair \\((s, a)\\), you would then preserve all the feature values in the \\(a^{th}\\) segment, and set everything else to \\(0\)).

TODO: afterstates

## Eligibility Traces for Linear Functions of State-Action Pairs
---


# Footnotes
---
<a name="bias">[1]</a>: Note that it's good to include an always-\\(1\\) feature in any feature vector, whether it be one for states or state-action pairs, 
for the bias/intercept term.