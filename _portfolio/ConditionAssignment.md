---
title: "Algorithm that allocates participants into balanced groups"
excerpt: "This algorithm helps in allocating people to experimental groups if balanced attributes between groups are required <br/><img src='https://github.com/m-guseva/personal/assets/63409978/be961ff6-3bed-4b3c-9d00-2392be94384b'>"
collection: portfolio
---



## The issue:
In my recent study I faced the challenge of allocating 90 participants into three different equal sized groups, in such a way that the sex ratio and average age remained the same across all groups (like in the figure below). 

This trivial if one already knows all the individuals who will participate in the experiment in advance. Unfortunately, this is usually not the case and presented a challenge in my study. So I wrote a script that solves this problem. By inputting the properties of a new participant, the algorithm efficiently assigns them to the appropriate group, maintaining the desired balance across all three groups. 

![groupAllocation](https://github.com/m-guseva/personal/assets/63409978/380d2cc3-089c-4c6b-b6c6-b57cdeb0f6e8)

It worked like a charm during the data collection phase earlier this year. The allocations in the diagram below are actually the real allocations that resulted during my fMRI study (which is currently in preparation for submission to a journal).
Of course I didn't want to leave it to chance whether the algorithm will work during the study or not, so I wrote simulation script (which is also contained in the repository as `simulation.py`). It provides a handy way to simulate an experiment based on different population parameters (e.g. an experiment with more female than male participants or more older females than males) and check the resulting group assignment.


## The procedure in short:
This procedure is based on the **stratified sampling method**, see [here](https://en.wikipedia.org/wiki/Minimisation_(clinical_trials)). After observing a person's attributes of interest (in this case sex and age), we construct a measure of imbalance for each attribute and each group. The group with the smallest sum of imbalance measures is selected for group assignment, with ties resolved randomly.

A more detailed explanation of  how the stratified sampling method is implemented is in the readme section of the [github repo](https://github.com/m-guseva/balanced-group-assignment).

For now the procedure only balances according to two attributes, sex and age. In future versions I might add a more general script to allow for other variables (e.g. education levels) to be balanced.


---
[Link to github repository](https://github.com/m-guseva/balanced-group-assignment)
