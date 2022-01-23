---
layout: post
title: Data exploration for NHL teams
description: "The subject matter for this project is hockey data, specifically the NHL stats API. In this article we perform exploratory data analysis"

categories: articles
tags: [sample post, images, test]
---

The subject matter for this project is hockey data, specifically the NHL stats API. This data is very rich; it contains information from many years into the past ranging from metadata about the season itself (eg. how many games were played), to season standings, to player stats per season, to fine-grained event data for every game played, known play-by-play data. If you’re unfamiliar with play-by-play data, the NHL uses this exact data to generate their play-by-play visualizations, an example of which is shown below. For a single game, roughly 200-300 events are tracked, typically limited in scope to faceoffs, shots, goals, saves, and hits (no passes or individual player location). Note that there is a logical way the games are assigned a unique ID, which is described here (take care to note the difference between regular season and playoff games!).

The time of event, event type, location, players involved, and other information is recorded for each event and the raw data is accessible through the play-by-play API. For example, the raw data for the above play-by-play can be found here:

https://statsapi.web.nhl.com/api/v1/game/2017020001/feed/live/
During the hockey season, data is updated live as games are in progress! This gives the opportunity to interact with new data frequently, giving you some insight as to why “pipelining”, and writing clean and reusable code is critical in a successful data science workflow.

Exploring goaltenders data: [official NHL stats webpage](http://www.nhl.com/stats/goalies?reportType=season&seasonFrom=20172018&seasonTo=20172018)

The save percentage *SV%* tells us about the ability of a goalie. However, the *SV%* does not give an idea of which shots were taken against him. To overcome this, a weighted *saves* score could be created by assigning more points to dangerous shots blocked, than for lower danger ones.

{% include q1.html %}

Other features could be implemented to assess the performance of a goalie. The number of points (P) describe more broadly the contribution of a goalie. The number of game won would also be relevant. Finally, the number of games played, and the number of game started should be considered to contextualize the *SV%* of a goalie. For example, the *SV%* is more convincing for a large number of games.
{% include q12.html %}

### Data acquisition

>A brief tutorial on how data was downloaded.

The NHL play-by-play data is available through a RESTful API. To interact efficiently with it, a custom `ApiEngine` class was created to query, download, and store the raw data in JSON format. Then, the `StorageEngine` class is used to interact and load stored JSON files. This will prove useful to tidy the raw data.

The `ApiEngine` can be used in the following ways:
1. Instanciate the engine with the directory to store API responses:
 `api_engine = ApiEngine(storage_directory_path)`
2. Query an API `endpoint` for https://statsapi.web.nhl.com/api/v1/ :
 `api_engine.query_api(endpoint)`
3. Query a season schedule by specifying the `start_year`:
 `api_engine.get_season_schedule(start_year)`
4. Query a game by its `gamePk` unique id:
 `api_engine.get_game(gamePk)`
5. Query all season games based on `gamePk` listed in the season schedule specified by `start_year`:
 `api_engine.get_all_season_games(start_year)`

The `StorageEngine` can be used in the following ways:
1. Instanciate the engine with the directory of the store files:
 `storage_engine = StorageEngine(storage_directory_path)`
2. Get a stored season schedule by specifying its `start_year`:
 `storage_engine = storage_engine.get_season_schedule(start_year)`
3. Get a stored game by its `gamePk` unique id:
 `storage_engine.get_game(gamePk)`
4. Get the list of `gamePk` found in a stored season schedule specified by `start_year` (which can be used to iteratively load individual games):
 `storage_engine.get_all_season_gamePk(start_year)`

### Widgets

 To validate and debug the project, two interactive widgets for Jupyter Notebook were created.
<img src="/images/widget_1.png" alt="">
 The first widget interfaces with our `StorageEngine` class. The values of `start_year` of the season and the index of the game (`game_idx`) can be changed to load a particular stored JSON file and display some game metadata. This allows for quick validation of the queried, stored, and loaded data.
 <img src="/images/widget_2.png" alt="">
 The second widget helped validate the standardization of plays coordinates. The widget is passed a DataFrame of plays data, the parameter `gamePk` and `period_idx` allow to select a particular game and period. The top figure is a scatter plot of the original (*x, y*) coordinates colored by team. The bottom figure displays the standardized coordinate, where all offensive plays are displayed on the right side.

{% include question_3.html %}
Above plot describe movement for type of shot for  September 2018-March 2020 using plotly interactive library.

### Tidy data

Our tidied DataFrame has 22 columns

<img src="/images/tidy_1.png" alt="">

For *goal* events, the strength field indicate if the number of players on the ice for both teams was *even*, or if the scoring team was in *power play*, or *short handed*. However, it does not detail the number of players on ice.

By examining events of type *penalty*, the number of players on the ice could be inferred. Indeed, for every penalty, the *timestamp* of the event and the length of the penalty in minutes (*penaltyMinutes*) are available. Finally, *goal* events could be contextualized with the time series of penalty data to find the number of players on the ice at the time of the goal.

This rich dataset allows for a variety of questions to be asked. For example:
- A binary feature *star_of_the_previous_match* could be added to label if players were designated as star of the previous match. This way, one could explore trends in *stars* over time, and if being a star has an effect on ulterior performance.
- An ordinal feature *n_hits_received* could be added to count the number of hits a player receives during a game. This could explain changes in the number of shots made, or overall performance of a player. Enhancement to this feature could be made by including the type of hits, and the height and weight of both the hitter and the hittee.
- A categorical feature the *handedness* of players and goalees. It would allow to explore if *opposite-side* and *same-side* handedness between players and goaless affects block rates.

### Question 5: Simple Visualizations
1)
{% include file.html %}
From above figure the most common type of shot is Wrist shot which counted 40392 shots more than any type of shot. From above plot we can not say which shot is most dangerous but generally slap shot is found to be dangerous shot. The most type of shot is deflected shot which yielded 24% of the total shot attempted.

2)
{% include Q3112.html %}

{% include Q3112.html %}

{% include Q3113.html %}
From above plots we can conclude that for distance from goal which ranges from 8.5 to 9.49 has highest number of goals in season 2018-19, 2019-20, 2020-21. And chances of shot being goal reduces as distance increases. Looking at all seasons there has not been significant amount of change in number of goals vs distance from goal.

3)
{% include Q5_3.html %}
Higher percentage goal is observed for distance between 0 and 5.874 from goal post. Deflected shot has highest Percentage goal i.e 0.625 where distance ranges from 0 and 5.874. Wrap around have comparitively fewer shots as goal compared to others.


### Q6. Advanced Visualizations: Shot Maps

1)The shot maps of the Colorado Avalanche for the 2016-2017 and the 2020-2021 seasons greatly differ. Indeed, in 2016-2017, they placed 30th out of 30 teams in the league, and in 2020-2021, they placed 1st place out of 31 teams.

In 2016-2017, they appear largely below the league average for close range shots. First, the less shots taken, the less chances to score. Second, the low proportion of close range shots compared to the league suggests that their offence struggled heavily to approach the net.

The plot for 2020-2021 tells the opposite story. They appear above the league average for almost the whole ice rink, especially near the goal. This suggests that they were able to takes more frequently than most teams, and that had a strong offence able to keep pressure on its opponents.

3)For Colorado Avalanche(team_initiative_id=COL), coordinates near goal area which is colored with yellow region has less shots than league average whereas area near coordinates (x=35, 30) which has more blueness, has more shots than league average for 2016-2017.
For 2020-2021, coordinates near goal area has blueness which indicates there were more number of shots than league average.
Comparing 2016-17 and 2020-21, 2016-17 had more number of shots than 2020-21 compared to its season league average. This concludes that Colorado Avalanche performed well in 2016-17 compared to 2020-21


Colorado Avalanche 2016-2017
<img src="/images/COL_2016_17.png" alt="">

Colorado Avalanche 2020-2021
<img src="/images/COL_20_21.png" alt="">

Buffalo Sabres 2019-20
<img src="/images/BUF_2019_20.png" alt="">

Tampa Bay Lightning 2019-20
<img src="/images/TBL_2019_20.png" alt="">
=======
<img src="/images/BUF_2018_19.png" alt="">

4)Looking Tampa Bay Lightning(TBL) contour plot and its color combination, Tampa Bay Lightning(TBL)team had better contour plots year by year in comparison to Buffalo Sabres(BUF). Since in graph of Buffalo Sabres had more greeness compared to Tampa Bay Lightning which refers that it had less number of shots than league average for different seasons.

Refer to code in [git repository](https://github.com/DS-IFT6750-project-collaboration/Hockey-all-star-analytics) in notebook section and [ift6758-milestone1_tj.ipynb](https://github.com/DS-IFT6750-project-collaboration/Hockey-all-star-analytics/blob/main/ift6758-milestone1_tj.ipynb), [ift6758-milestone2_tj.ipynb](https://github.com/DS-IFT6750-project-collaboration/Hockey-all-star-analytics/blob/main/ift6758-milestone2_tj.ipynb)
