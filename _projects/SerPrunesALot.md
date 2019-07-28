---
title: Ser Prunes-A-Lot
excerpt: >
  An alpha-beta-based engine to play the board game "Knightthrough". Developed as project for a course during my first year in the Master AI program at Maastricht University.
  Source code (C++) and report available.
image: /assets/project_serprunesalot/KnightThroughInitialPosition.png
start_date: 2015-05-14 00:00:00 +0100
---

The game "Knightthrough" is a simple board game, played on a regular chess board, only with pieces that can move in the same way as a Knight can in chess. The player who
manages to reach the opposing side of the board first wins the game. In this project (part of a course in the first year Master AI program at Maastricht University), the
goal was to implement an agent to play this game, using the Alpha-Beta search algorithm as the core of the agent.

The agent can be considered a fairly strong agent. On top of the regular Alpha-Beta search algorithm, it uses enhancements such as Aspiration Search, a Transposition Table,
and the Killer Move Heuristic. It also uses an efficient bitboards-based state representation. Of course, there are still many more possible improvements in the literature,
so this is not completely state-of-the-art.

A more detailed description of the agent can be found in the 
[report](https://github.com/DennisSoemers/SerPrunesALot/blob/master/Report/ISG%20KnightThrough%20Report%20Dennis%20Soemers.pdf) that was written for the course, and the C++ source
code is also available on [github](https://github.com/DennisSoemers/SerPrunesALot).

<img src="/assets/project_serprunesalot/KnightThroughInitialPosition.png" alt="Knightthrough game board in its initial state">