---
layout: page
title: The MaastCTS2 General Video Game AI Agent
date: 2016-07-01 00:00:00 +0100
description: >
  For my Master's thesis, I implemented an MCTS-based General Video Game AI agent named MaastCTS2. The basic idea of General Video Game AI is to implement an agent
  that has to be able to play any kind of video game (think of simple 2D games like Space Invaders or PacMan), without knowing in advance which games it is going to play.
  In addition to my Master's thesis, this project also resulted in a published paper at the CIG 2016 conference (Best Student Paper Award), winning the GVGAI Single-Player
  Championship of 2016, and placing second in the GVGAI Two-Player Championship of 2016.
image: /assets/project_gvgai/boulderdash.png
---

The goal of [General Video Game AI (GVGAI)](http://gvgai.net/) is to implement agents that can play any real-time video game they are asked to play, without knowing
in advance which games they can be expected to play. Every year, competitions for such agents are organised at various conferences. For my Master's thesis, I implemented
the MaastCTS2 agent. It uses Monte-Carlo Tree Search with numerous enhancements (including novel enhancements). 

The most detailed description of the agent can be found in my [Master's thesis](https://project.dke.maastrichtuniversity.nl/games/files/msc/Soemers_thesis.pdf). A shorter
version can be found in the following paper: Dennis J.N.J. Soemers, Chiara F. Sironi, Torsten Schuster, and Mark H.M. Winands (2016). “Enhancements for Real-Time Monte-Carlo 
Tree Search in General Video Game Playing”. In *2016 IEEE Conference on Computational Intelligence and Games (CIG 2016)*, pp. 436-443. IEEE. **Best Student Paper Award**. I
also wrote a more informal [blog post]({{ site.baseurl }}{% post_url 2016-09-29-the-general-video-game-agent-maastcts2 %}) about it. Source code is available on
[github](https://github.com/DennisSoemers/MaastCTS2).

During this project, I also [contributed](https://github.com/EssexUniversityMCTS/gvgai/pull/18) to the GVGAI framework itself by
[implementing](https://github.com/EssexUniversityMCTS/gvgai/pull/40) various [optimizations](https://github.com/EssexUniversityMCTS/gvgai/pull/42).

<img src="/assets/project_gvgai/boulderdash.png" width="100%">