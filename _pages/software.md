---
title: Software
permalink: /software/
description: Dennis Soemers' software contributions.
last_modified_at: 2022-09-11 10:00:00 +0100
---

This page lists the core pieces of software that I have substantially contributed to, as well as other (often smaller) contributions to other open-source projects.



## Core Software (Research)

### Ludii

I am one of the main developers of the [Ludii general game system](https://ludii.games/), alongside several other [Ludii team members](https://ludii.games/contact.php). 
I worked on this during my PhD.

- Wrote most **AI (search and learning)** code.
- Substantial contributions to the **core of Ludii's engine**.
- **Profiling and optimising for speed and memory** across the complete codebase.
- Set up and maintaining **Travis CI**.
- Set up and maintaining **build scripts** (Ant).
- Set up **automated generation of [Ludii Language Reference](https://ludii.games/downloads/LudiiLanguageReference.pdf)**.
- **Tech stack**: Java, Python, Git, Travis CI, Ant, Slurm.
- **Source code**: [https://github.com/Ludeme/Ludii/](https://github.com/Ludeme/Ludii/)
- **Main/sole developer** for several **subprojects**:
	- [Ludii Example AI](https://github.com/Ludeme/LudiiExampleAI) (Java)
	- [Ludii Python AI](https://github.com/Ludeme/LudiiPythonAI) (Python, Java)
	- [Ludii Tutorials](https://ludiitutorials.readthedocs.io/en/latest/) (ReadTheDocs) ([source code](https://github.com/Ludeme/LudiiTutorials))

### Ludii + Polygames Bridge

I developed a [bridge](https://github.com/facebookarchive/Polygames/tree/main/src/games/ludii) between the 
Java-based [Ludii general game system](https://ludii.games/) and the C++/Python-based 
[Polygames framework for deep learning in games](https://github.com/facebookarchive/Polygames/).

- **Main developer** for the bridge.
- **Tech stack**: Java Native Interface (JNI), C++, CMake, Python, Java, Git, CircleCI.
- **Source code**: [https://github.com/facebookarchive/Polygames/tree/main/src/games/ludii](https://github.com/facebookarchive/Polygames/tree/main/src/games/ludii)

### MaastCTS2 (GVGAI Agent)

I developed the MaastCTS2 agent for General Video Game playing, based on the [GVGAI framework](https://github.com/GAIGResearch/GVGAI).
I worked on this for my M.Sc. thesis.

- **[First place](https://groups.google.com/g/the-general-video-game-competition/c/z-43NBUfc58)** in the 2016 Single-Player GVGAI competition track.
- **[Second place](https://groups.google.com/g/the-general-video-game-competition/c/z-43NBUfc58)** in the 2016 Two-Player GVGAI competition track.
- **Tech stack**: Java, Git.
- **Source code**: [https://github.com/dennisSoemers/maastcts2](https://github.com/dennisSoemers/maastcts2)

### HTN Planner in Unreal Engine 4

I developed a Hierarchical Task Network (HTN) Planner in Unreal Engine 4. I worked on this for my research internship as a part of my Master in AI.

- **Tech stack**: Unreal Engine 4, C++, Git
- **Source code**: [https://github.com/DennisSoemers/HTN-Planner/](https://github.com/DennisSoemers/HTN-Planner/)



## Other Software (Non-research)

### PAPER

I developed a DLL plugin for The Elder Scrolls V: Skyrim, which exposes additional functionality to the game's scripting language.
This can be used by third-party modders for scripts in their mods. It is built on top of the [Skyrim Script Extender](http://skse.silverlock.org/)
and [CommonLibSSE-NG](https://github.com/CharmedBaryon/CommonLibSSE-NG), which have reverse-engineered substantial portions of the game's executable.

- **Tech stack**: C++, CMake, vcpkg, Git
- **Source code**: [https://github.com/DennisSoemers/PAPER](https://github.com/DennisSoemers/PAPER)



## Other Open-Source Contributions

### rliable

- Contributed a small change to the performance profile plotting code of [rliable (Google Research)](https://github.com/google-research/rliable) to add support
for linestyles. ([PR#8](https://github.com/google-research/rliable/pull/8))
- Python

### OpenSpiel

- Contributed a change to how entries are sorted in the legend of alpha-rank plots in [OpenSpiel (DeepMind)](https://github.com/deepmind/open_spiel/pull/112).
([PR#112](https://github.com/deepmind/open_spiel/pull/112))
- Python

### HuggingFace Deep RL Class

- Contributed minor clarifications, fixes, and suggestions to the [HuggingFace Deep RL Class](https://github.com/huggingface/deep-rl-class).
([PR#62](https://github.com/huggingface/deep-rl-class/pull/62), [Issue#66](https://github.com/huggingface/deep-rl-class/issues/66))
- Jupyter Notebook, Python

### Multi-MAuS

- Several contributions to the [Multi-MAuS multi-modal authentication simulator](https://github.com/lmzintgraf/MultiMAuS).
- [My contributions](https://github.com/lmzintgraf/MultiMAuS/commits?author=DennisSoemers) were primarily optimisations and feature engineering.
- Python

### GVGAI

- Contributed several substantial optimisations to the [GVGAI (general video game AI) framework](https://github.com/EssexUniversityMCTS/gvgai).
([PR#18](https://github.com/EssexUniversityMCTS/gvgai/pull/18), [PR#40](https://github.com/EssexUniversityMCTS/gvgai/pull/40), 
[PR#42](https://github.com/EssexUniversityMCTS/gvgai/pull/42))
- Java