---
layout: page
title: HTN Plan Reuse in UE4
date: 2016-03-31 00:00:00 +0100
description: >
  An HTN Planner for the Unreal Engine 4 game engine, with a novel approach that re-uses previously generated plans to speed up re-planning. This was developed during
  a Research Internship in the second year of the Master AI at Maastricht University. A paper about this research was published at the CIG 2016 conference. Source code
  and a detailed report are available.
image: /assets/project_htnplanreuse/HTNExecutionArchitecture.png
---

For a Research Internship in the first half of the second year of the Master AI program, I implemented a Hierarchical Task Network (HTN) Planner for the Unreal Engine 4 (UE4)
game engine. A novel approach for rapid (real-time) re-planning was implemented, which makes use of previously generated plans to more quickly find new plans. The basic idea
is that an NPC in a video game may often want to generate new plans (for example due to new observations which were not taken into account when generating the previous plan),
but such a new plan will often not deviate very much from the previous plan. The old plan can then still provide useful information for the re-planning process.

This research resulted in the following conference paper: Dennis J.N.J. Soemers and Mark H.M. Winands (2016). “Hierarchical Task Network Plan Reuse for Video Games”. 
In *2016 IEEE Conference on Computational intelligence and Games (CIG 2016)*, pp. 1-8. IEEE.

A more detailed report is available on [github](https://github.com/DennisSoemers/HTN_Plan_Reuse/blob/master/Report/PlanReuseReport.pdf), as well as the
[source code](https://github.com/DennisSoemers/HTN_Plan_Reuse).