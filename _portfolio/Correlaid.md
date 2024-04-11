---
title: "Uncovering Russian Disinformation: Analyzing Telegram Chats with Python's NetworkX"
excerpt: "A network of telegram chats and messages was analyed<br/><img src='https://github.com/m-guseva/m-guseva.github.io/blob/master/images/thumb_correlaid.png?raw=true'>"
collection: portfolio
---


📰 Publication by Correctiv: [https://correctiv.org/faktencheck/hintergrund/2024/04/10/telegram-analyse-desinformation-russland-vernetzt-sich-um-alina-lipp-in-deutschland-mit-propaganda-fakes-zum-ukraine-krieg/](https://correctiv.org/faktencheck/hintergrund/2024/04/10/telegram-analyse-desinformation-russland-vernetzt-sich-um-alina-lipp-in-deutschland-mit-propaganda-fakes-zum-ukraine-krieg/)


👩🏻‍💻Technical report (German): [https://correctiv.org/wp-content/uploads/2024/04/Correlaid-Bericht-Deutsch_Neues-aus-Russland_Telegram-Netzwerk-Analyse-Python.pdf](https://correctiv.org/wp-content/uploads/2024/04/Correlaid-Bericht-Deutsch_Neues-aus-Russland_Telegram-Netzwerk-Analyse-Python.pdf)

👩🏻‍💻Technical report (English): [https://correctiv.org/wp-content/uploads/2024/04/Correlaid-Report-English_Neues-aus-Russland_Telegram-Network-Analysis-Python.pdf](https://correctiv.org/wp-content/uploads/2024/04/Correlaid-Report-English_Neues-aus-Russland_Telegram-Network-Analysis-Python.pdf)

💻 Github Repository with analysis code: to-be-added-soon!

---

In a collaborative project between the data4good organization [Correlaid](https://www.correlaid.org/) and the investigative journalists at [CORRECTIV](https://correctiv.org/) we analyzed a large telegram dataset containing chat and group messages between January 2022 and April 2023. The aim was to find Russian disinformation and propaganda narratives on the invasion of Ukraine, particularly as they are forwarded through German-speaking channels.

For a detailed breakdown of our analysis, check out our [technical report](https://correctiv.org/wp-content/uploads/2024/04/Correlaid-Report-English_Neues-aus-Russland_Telegram-Network-Analysis-Python.pdf).


Using the networkX library we created a network graph out of the dataset. The resulting dataset was a [bipartite graph](https://en.wikipedia.org/wiki/Bipartite_graph) that we reduced by means of graph projection. We ended up with a graph containing 6687 nodes and 77098 edges.

[<img src="https://github.com/m-guseva/m-guseva.github.io/blob/master/images/Graph.png?raw=true" width="600"/>](graph.jpg)


Based on this graph, we looked at the temporal evolution of relevant connectivity metrics such as degree and betweenness centrality. We identified a couple of influential chats that serve as hubs that translate and spread Russian desinformation messages.

Meanwhile, another part of our group analyzed the message content using  with NLP methods to compare content similarity between messages (see the [technical report](https://correctiv.org/wp-content/uploads/2024/04/Correlaid-Report-English_Neues-aus-Russland_Telegram-Network-Analysis-Python.pdf) for more details).

Overall, it was a fun side-project that I participated in during my PhD which  really showed how much interesting information there can be extracted from large datasets. We worked completely remotely as a team which was a challening task but we finished the project successfully and now it's published on [Correctiv's webpage](https://correctiv.org/faktencheck/hintergrund/2024/04/10/telegram-analyse-desinformation-russland-vernetzt-sich-um-alina-lipp-in-deutschland-mit-propaganda-fakes-zum-ukraine-krieg/) ☺️
