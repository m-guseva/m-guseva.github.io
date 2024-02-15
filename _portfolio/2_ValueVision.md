---
title: "👁 Value vision training game for artists with Python"
excerpt: "I created a game in Python where artists can train their tonal values to improve their realistic drawing and painting skills "
collection: portfolio
---

## "Learning how to draw is learning how to see"

Drawing has always been my passion since I was a little girl. On my art journey I learned that to get good at drawing one has to get good at a set of fundamental skills. Arguably the most important one of all is getting the **tonal value** of an object right. Why? The tonal values communicate where the light hits a surface. Simply speaking, the lighter the surface, the closer it is to the light source, the darker the surface, the farther it is from the light. This essentially determines the whole shape and nature of an object. If this is wrong, no amount of color can fix that. So, as an artist you have to be on point with the values (if your goal is to draw realistically of course 😄).


[<img src="https://github.com/m-guseva/m-guseva.github.io/blob/master/_portfolio/Drawing.jpg?raw=true" width="300"/>](drawing.jpg)


## The problem:
 People are pretty bad at seeing values correctly and hence they struggle with transfering what they see on paper. Correcting these visual biases is a looong and challenging process that can take years (or decades). Especially self-taught artists who don't have the luxury of studying with a teacher who can give feedback or point out their mistakes, can often only refer to drawing books/tutorials (if at all). What is lacking then is consistent and fast feedback regarding their tonal value judgment. 

## The solution:
I aimed at creating a program that turns tonal value vision training into a game, resembling the way you would learn with flashcards. How it works: An area in some image is selected and the user has to match the tonal value with a slider. The closer their estimate is to the real value, the more points are rewarded. In this way the person gets quick feedback in the setting of a fun game.
The way that each area is selected is based on an algorithm that I've created that finds areas of homogeneous tonal value, read more [here](https://m-guseva.github.io/portfolio/ImageAlgorithm/) if want to learn more!

The value vision trainer was created in Python using streamlit for the interface. 

👉🏻 You can try the game here: [https://value-vision-trainer.streamlit.app/](https://value-vision-trainer.streamlit.app/)

👉🏻 The code is also uploaded on github [here](ttps://github.com/m-guseva/value_vision_trainer).

## Future plans
I am planning to implement a similar training for other building blocks of art making, e.g. hue matching, proportion training, perspective training, edges (smooth/hard) in the future, this serves as a starting point.
