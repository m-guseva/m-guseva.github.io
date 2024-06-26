---
title: "🏋🏻 Tracking my Beat81 workout results in a streamlit dashboard"
excerpt: "Using Python's streamlit library to visualize workout results <br/><img src='https://github.com/m-guseva/m-guseva.github.io/blob/master/images/thumb_B81.jpg?raw=true'>"
collection: portfolio
---

I love working out and crunching data, so it's no surprise that I love the [BEAT81](https://www.beat81.com) workouts, where your heart rate is tracked during the sessions. Unfortunately the company doesn't provide any nice statistics
on the points that one collects, so I decided to create my own dashboard to track and interpret my workout results. I used Python's pandas and numpy libraries to handle the dataset, matplotlib to create the plots and streamlit to display 
the metrics in a beautiful dashboard.

The dashboard shows the sweat and recovery points for each workout over time and across each workout type. Additionally, this dashboard also helps me to spot any seasonal or cyclical patterns in the points
by showing the autocorrelations in the data. 

[Link to the github repository](https://github.com/m-guseva/B81-Dashboard)

This is a sample dataset:
[<img src="https://github.com/m-guseva/B81-Dashboard/assets/63409978/3068baeb-cf1e-4826-90aa-8fe4f3524bb2" width="800"/>](image.png)
