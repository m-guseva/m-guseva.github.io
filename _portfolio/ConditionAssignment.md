---
title: "Algorithm that allocates participants into balanced groups"
excerpt: "Using Python's streamlit library to visualize workout results <br/><img src='groupAllocation_mainIMG.png'>"
collection: portfolio
---
excerpt: "Using Python's streamlit library to visualize workout results <br/><img src='https://github.com/m-guseva/personal/assets/63409978/c67a942c-7c7f-4d99-bf9c-9de53f803d9c'>"



## The issue:
In my recent study I faced the challenge to allocate 90 participants into three different equal sized groups, in such a way that the sex ratio and average age remained the same across all groups (like in the figure below). 

This trivial if one already knows all the individuals who will participate in the experiment in advance. Unfortunately, this is usually not the case and presented a challenge in my study. So I wrote a script that solves this problem. By inputting the properties of a new participant, the algorithm efficiently assigns them to the appropriate group, maintaining the desired balance across all three groups. 

It worked like a charm during the data collection phase earlier this year. The allocations in the diagram are actually the real allocations that resulted during my fMRI study (which is currently in preparation for submission).
Of course I didn't leave it to chance whether the algorithm works or not, so I wrote simulation script (which is also contained in the repository as `simulation.py`). It provides a way to simulate an experiment based on different population parameters (e.g. an experiment with more female than male participants or more older females than males) and check the resulting group assignment.



![groupAllocation](https://github.com/m-guseva/personal/assets/63409978/5a14cc7c-d25c-4b91-a766-b8944123dab4)


## The procedure in short:
This procedure is based on the **stratified sampling method**, see [here](https://en.wikipedia.org/wiki/Minimisation_(clinical_trials)). After observing a person's attributes of interest (in this case sex and age), we construct a measure of imbalance for each attribute and each group. The group with the smallest sum of imbalance measures is selected for group assignment, with ties resolved randomly.

A more detailed explanation of  how the stratified sampling method is implemented is in the readme section of the [github repo](https://github.com/m-guseva/balanced-group-assignment).

For now the procedure only balances according to two attributes, sex and age. In future versions I might add a more general script to allow for other variables (e.g. education levels) to be balanced.


---
[Link to github repository](https://github.com/m-guseva/balanced-group-assignment)
