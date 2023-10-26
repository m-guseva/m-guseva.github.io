---
title: "Algorithm that allocates participants into balanced groups"
excerpt: "This algorithm helps in allocating people to experimental groups if balanced attributes between groups are required <br/><img src='/images/500x300.png'>"
collection: portfolio
---

In my recent study I needed to allocate 90 participants into three different groups, in such a way that the sex ratio and average age was the same between the groups. This trivial if you already know all people who will participate in the experiment in advance. Unfortunately, this is usually not the case and was certainly not possible in my case. So I wrote a script that solves this problem.

How:
This procedure is based on a stratified sampling method, see [here](https://en.wikipedia.org/wiki/Minimisation_(clinical_trials)). 
After observing a person's attributes of interest (in this case sex and age), we construct a measure of imbalance for each attribute and each group.


The other script simulation.pyprovides a way to simulate an experiment based on different population parameters (e.g. an experiment with more female than male participants or more older females than males) and check the resulting group assignment.

For now the procedure only balances according to two attributes, sex and age. In future versions I might add a more general script to allow for other variables (e.g. education levels) to be balanced.

[Link to github repository](https://github.com/m-guseva/balanced-group-assignment)
