---
title: "Ludii Pre-Release"
date: 2019-08-13 16:30:00 +0100
categories:
  - "Ludii" 
  - "General Game Playing" 
  - "Digital Ludeme Project"
  - "Games"
  - "Artificial Intelligence"
excerpt: "Ludii is a new general game-playing (GGP) system, of which an early pre-release version has been published today. The system comes with a wide range of built-in games, and may be
  interesting for game-playing enthusiasts to simply play games! Ludii's new game description language tends to lead to game descriptions that are significantly shorter
  than previous game description languages (such as Stanford's GDL), and they also tend to be much easier to read. On top of those advantages, the system can run most of
  these games significantly faster than older GGP systems. All of these advantages combined are expected to make the system interesting for many Artificial Intelligence
  researchers, in particular those active in General Game Playing. The pre-release version of Ludii, together with other resources (such as a User Guide) may be downloaded 
  from its website: [http://ludii.games/](http://ludii.games/)"
---

# Ludii
---

Ludii is a new general game-playing (GGP) system, of which an early pre-release version has been published today. The system comes with a wide range of built-in games, and may be
interesting for game-playing enthusiasts to simply play games! Ludii's new game description language tends to lead to game descriptions that are significantly shorter
than previous game description languages (such as Stanford's GDL), and they also tend to be much easier to read. On top of those advantages, the system can run most of
these games significantly faster than older GGP systems. All of these advantages combined are expected to make the system interesting for many Artificial Intelligence
researchers, in particular those active in General Game Playing. The pre-release version of Ludii, together with other resources (such as a User Guide) may be downloaded 
from its website: [http://ludii.games/](http://ludii.games/)

{% include figure image_path="/assets/post_ludii_prerelease/Hex.png" alt="Hex in Ludii" caption="Hex in Ludii." %}

# AI Controllers in Ludii
---

Ludii ships with a number of built-in AI controllers, which we aim to continue improving by following up on research as described in these three papers (all
experiments in these were carried out using earlier versions of Ludii-in-development):

- [Strategic Features for General Games](http://ceur-ws.org/Vol-2313/)
- [Biasing MCTS with Features for General Games](https://ieeexplore.ieee.org/document/8790141)
- [Learning Policies from Self-Play with Policy Gradients and MCTS Value Estimates](www.ieee-cog.org/proceedings/)

Ludii can also import custom AI implementations (currently only in Java). Instructions and examples for writing your own Ludii AI controller can be found in
[their own github repository](https://github.com/Ludeme/LudiiExampleAI).

# Building New Games for Ludii
---

Games in Ludii are written using "ludemes", which are keywords that correspond to a variety of common, high-level game concepts. An example description for
Tic-Tac-Toe is provided below. It still looks a bit "formal" or "mathematical", but is much shorter and more readable than, for instance, a GDL description 
of the same game. 

```
(game "Tic-Tac-Toe"
  (mode 2)
  (equipment {
    (board (square 3) (square))
	(disc P1)
	(cross P2)
  }
  )
  (rules
    (play (to (mover) (empty)))
	(end (line length:3) (result Mover Win))
  )
)
```

Modifying this game description to create a variant of Tic-Tac-Toe played on a 4x4 board would be trivial, only requiring the `(square 3)` part to be replaced by
`(square 4)`, and `(line length:3)` by `(line length:4)` (under the assumption that we'd want to require a line of 4 to win this game). Advanced users can also
specify the options to implement such a variant directly inside a single Ludii game description file. More instructions on how to model your own games for
Ludii, including an explanation of this *options* mechanism and more, can all be found in our 
[Ludii User Guide](https://www.ludii.games/LudiiUserGuide-0.2.0.pdf) (note: this link is for the pre-release version).

# Digital Ludeme Project
---

Ludii is in development as part of the ERC-funded [Digital Ludeme Project](/projects/DigitalLudemeProject/). This project aims to use computational techniques
to gain more insight into how traditional games were played throughout history, and how they may have spread temporally, geographically, and culturally.
This requires us to model all the games we wish to study (up to a thousand, or more including variants of games!) in a single, consistent, "mathematical" format.
This is exactly what we do in Ludii, with its *ludeme*-based format. The ability to efficiently run all of these games -- which, contrary to older GGP systems,
is offered to us by Ludii -- is also a prerequisite for the large-scale, AI-based evaluations of (reconstructions of) game rulesets that we aim to run in this project.