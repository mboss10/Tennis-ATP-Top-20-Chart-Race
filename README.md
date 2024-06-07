# Tennis ATP Top 20 Chart Race

<img src="https://github.com/mboss10/Tennis-ATP-Top-20-Chart-Race/blob/main/ATPTop20.gif" width="600">

## Introduction

The purpose of this project was to use the Huge Tennis Database that you can find on [Kaggle](https://www.kaggle.com/datasets/guillemservera/tennis) to mine tennis data and put together an animated chart race of the Top 20 ATP tennis player from 1990 to 2024.

## Process

I went through the following steps:
1. Database Setup: Creating the DB environment in DBeaver.
2. Data Modeling: Defining SQL views to ensure appropriate and accurate data.
3. Visualization: Importing the dataset into Tableau and building the dashboard with design elements inspired by the official ATP website.

I will detail below the different steps.

## 1. Database Setup

Once I have downloaded the SQLite database file from Kaggle, I imported it into my local DBeaver SQL client.
This is a very simple process:
1. Connect to a database - select SQLite in this case.  
<img src="https://github.com/mboss10/Tennis-ATP-Top-20-Chart-Race/blob/main/Connect_a_DB.png" width="400"><br /> 
2. Browse the folder in which I saved the SQLite database file and select it.  
<img src="https://github.com/mboss10/Tennis-ATP-Top-20-Chart-Race/blob/main/SelectFile.png" width="400"><br />
3. Test connection, success then click Finish.  
<img src="https://github.com/mboss10/Tennis-ATP-Top-20-Chart-Race/blob/main/TestConnection.png" width="400"><br />

Then I ended up with a new database, that I named Tennis_ATP, that contains 3 tables:
1. matches = contains match results and stats (winner, loser, date, aces, etc ...)
2. players = contains players' basic info (first_name, last_name, hand, birth_date, country_code, height, etc ...)
3. rankings = contains players ranking per ranking period (every week almost, except for some missing periods)  
<img src="https://github.com/mboss10/Tennis-ATP-Top-20-Chart-Race/blob/main/DBTables.png" width="150"><br />

## 2. Data Modeling

Since I am not going to create the chart race for every single player rank - that would be too many bars with some barely visible - I think it makes sense to focus on the top 20 players on each week of the ATP ranking.  
  
Therefore I first decided to create a view to reduce each ranking week to that subset of top 20 players, as well as join with key player information.  
Here is the view definition:  
```
CREATE VIEW Top20 as 
SELECT 
		r.ranking_date 
		, r."rank" 
		, r.points 
		, p.player_id 
		, p.name_first || ' '|| p.name_last as full_name
		, p.hand 
		, p.dob 
		, p.ioc
		, p.height 
	FROM 
		rankings r 
	INNER JOIN
		players p on p.player_id = r.player 
	WHERE rank < 21;
```
  
Sample view results:  
<img src="https://github.com/mboss10/Tennis-ATP-Top-20-Chart-Race/blob/main/Top20Sample.png" width="600"><br />

The technique used here is very simple given that rank is part of the ranking table so I just had to filter on `rank < 21` to get only the top 20 for each ranking week. Combined with an `INNER JOIN` with the players table it allows me to pull name, hand, date of birth, country and height of those players.
> [!NOTE]
> Notice how I created a full_name by concatenating first and last name. In SQLite you use 2 pipe signs `||` to concatenate strings
  
  
  
The other thing I wanted to include in my bar chart race is the picture of the Top player on the selected week along with its running match records (wins VS losses). So I had to come up with a query that would provide me with the necessary data to achieve this.  

See below the query:  
```
cte_top1_with_points as (
	select distinct r.player from rankings r where r."rank" = 1 and r.points is not null
),
cte_wins as(
	select 
		m.winner_id
		, m.winner_name
		, m.tourney_date
		, count(m.winner_id) as wins_per_period
	FROM
		matches m 
	WHERE	
		m.winner_id in (select player from cte_top1_with_points)
	group by 1,2,3
),
cte_win_per_ranking as (
	select 
		ranking_date 
		, player_id 
		, full_name 
		, sum(wins_per_period) as win_per_ranking_date
	from
		Top20 t 
	inner join 
		cte_wins on (cte_wins.winner_id = t.player_id and cte_wins.tourney_date <= t.ranking_date)
	where 
		"rank" = 1 and points is not null
	GROUP BY 1,2,3
),
cte_losses as(
	select 
		m.loser_id
		, m.loser_name
		, m.tourney_date
		, count(m.loser_id) as losses_per_period
	FROM
		matches m 
	WHERE	
		m.loser_id in (select player from cte_top1_with_points)
	group by 1,2,3
),
cte_loss_per_ranking as (
	select 
		ranking_date 
		, player_id 
		, full_name 
		, sum(losses_per_period) as loss_per_ranking_date
	from
		Top20 t 
	inner join 
		cte_losses on (cte_losses.loser_id = t.player_id and cte_losses.tourney_date <= t.ranking_date)
	where 
		"rank" = 1 and points is not null
	GROUP BY 1,2,3
)
select 
	cte_win_per_ranking.ranking_date
	, cte_win_per_ranking.player_id 
	, cte_win_per_ranking.full_name
	, win_per_ranking_date
	, loss_per_ranking_date
	from cte_win_per_ranking
inner join 
	cte_loss_per_ranking on (cte_win_per_ranking.player_id = cte_loss_per_ranking.player_id and cte_win_per_ranking.ranking_date = cte_loss_per_ranking.ranking_date)
```
  
I extensively use the technique of Common Table Expression (CTE) which is a temporary result set that can be referenced in the rest of my SQL query. In this example, where I need to have for each ranking week, the top player name and its running wins and running losses, the CTEs I created help me simplify a complex queries. I break down a complex SQL statement into smaller, more manageable parts to improve readability and maintainability.  

Let's break down each CTE used in the statement:  
```
with cte_top1_with_points as (
	select distinct r.player from rankings r where r."rank" = 1 and r.points is not null
),
```
Here I am selecting only top 1 player with ATP points - because I noticed browsing through the data that we have ranking rows with no points (every rows before year 1990). It is part of my cleaning process, data preparation and modeling to filter them out given I need the points for my analysis.  

___

```
cte_wins as(
	select 
		m.winner_id
		, m.winner_name
		, m.tourney_date
		, count(m.winner_id) as wins_per_period
	FROM
		matches m 
	WHERE	
		m.winner_id in (select player from cte_top1_with_points)
	group by 1,2,3
),
```

Here I am gettinng the count of wins per player and tournament date.
> [!NOTE]
> Notice how I am reusing the previous CTE `cte_top1_with_points`, which is also one of the huge advantage of CTEs: Reuse a result set across multiple parts of a query without rewriting the same subquery multiple times.
___
```
cte_win_per_ranking as (
	select 
		ranking_date 
		, player_id 
		, full_name 
		, sum(wins_per_period) as win_per_ranking_date
	from
		Top20 t 
	inner join 
		cte_wins on (cte_wins.winner_id = t.player_id and cte_wins.tourney_date <= t.ranking_date)
	where 
		"rank" = 1 and points is not null
	GROUP BY 1,2,3
),
```

Then in this CTE, I am adding up all the wins of the player per ranking week.  
___
In the last 2 CTEs `cte_losses` and `cte_loss_per_ranking` I am using the same process but using loss information.<br />
___
```
select 
	cte_win_per_ranking.ranking_date
	, cte_win_per_ranking.player_id 
	, cte_win_per_ranking.full_name
	, win_per_ranking_date
	, loss_per_ranking_date
	from cte_win_per_ranking
inner join 
	cte_loss_per_ranking on (cte_win_per_ranking.player_id = cte_loss_per_ranking.player_id and cte_win_per_ranking.ranking_date = cte_loss_per_ranking.ranking_date)
```  


The final part of the SQL statement combines the CTEs to give for each ranking week, the top player and his running wins and losses.

<img src="https://github.com/mboss10/Tennis-ATP-Top-20-Chart-Race/blob/main/TopPlayerRunningWinsLosses.png" width="600"><br />  


Now I have the data sets modeled the way I expect and can move on to the next step, build my dashboard.

## 3. Visualization

I used Tableau Public Desktop as my tool to create the visualizations and Tableau Public to publish it in my [portfolio](https://public.tableau.com/app/profile/max4890/vizzes).

I performed the following steps:
- Export the data to CSV files and used them as a datasource in Tableau
- Build the visualizations
- Assemble in the Dashboard
- Publish to Tableau public
- Create a Youtube video

### CSV Export

From DBeaver there is an option to export the results of a query to different formats, including .csv file. I did export both the results of my Top20 view and my advanced query in 2 .csv files. I had to work it that way because I could not easily find a solution/connector to connect my Tableau Desktop Public application with my SQLite database. I know there is a connector if you use the standard Tableau Desktop application but not Public and I don't have a product registration key for it (though I subscribed to the DataDev program ... but they give you a Tableau server instance but no Desktop reg key ... still don't understand that)<br /><br />

Then in Tableau Public Desktop I imported my 2 files and here I am with my datasource, ready to start vizzing!!<br />

<img src="https://github.com/mboss10/Tennis-ATP-Top-20-Chart-Race/blob/main/TableauDatasource.png" width="600"><br />

___

### Build the visualizations

I need 3 visualizations to compose my dashboard

#### <ins>**Bar Chart Race**</ins>

This is the main visualization of my dashboard.<br />
It is composed of 2 types of Marks: horizontal bars and shapes on a dual axis so that my shapes, which are flags representing the player's country, can be right aligned with the bars.<br />
I am using the Pages feature to toggle between ATP ranking weeks, and because it has a nice play button capability to run my race chart. (Spoiler alert!! I will discover that Tableau Public behaves a little differently ... more to come on that).<br />
Lastly, I am using the `RANK_UNIQUE()` function and I compute it using both the player's name and country to ensure that Pages each week only display the top 20 of that week.<br /><br />


> [!NOTE]
> I created a `Points Adjusted` calculated field because I noticed that there was a HUGE difference in total points of players between the year before 2009 and after. I suspect the ATP changed the calculation on that year because it really exploded after. And since the axis of the graph cannot be dynamic per Pages week, I had to make sure it can accommodate both big number of points and small ones inn a nice way.
> So I came up with the following Calc
> ```
> IF [Ranking Date]<DATE("2009/01/01")
> THEN [Points]*2
> ELSE[Points]
> END
> ```

<img src="https://github.com/mboss10/Tennis-ATP-Top-20-Chart-Race/blob/main/BarChartRace.png" width="600"><br />
<br /><br />

#### <ins>**Player Headshot**</ins>

With this visualization, my goal is to display, for each ranking week, the headshot of the Top 1 player on that week.  
The technique here is to use shapes and associated each top player with a specific custom shape. I decided to design a rounded headshot with the player's country flag next to it. I used Figma to create those custom shapes.  
Then I needed to make sure that I filtered my Rank_unique calculated field to only display top 1 and here we go.  

<img src="https://github.com/mboss10/Tennis-ATP-Top-20-Chart-Race/blob/main/PlayerHeadshot.png" width="600"><br />
<br /><br />

#### <ins>**Wins/Losses Ratio**</ins>

With this visualization, my goal is to display, for each ranking week, the running ratio of winns/losses of the Top 1 player on that week inside a pie chart. Pie chart is not my favorite type of visualization but we'll see in the Dashboard that it made sense this time to use it.  

<img src="https://github.com/mboss10/Tennis-ATP-Top-20-Chart-Race/blob/main/WinLossRatioPieChart.png" width="600"><br />

### Assemble in the Dashboard

Now that all the visualizations I need are ready, the last piece (but not the least one) was to assemble them in my dashboard.  
Given that I wanted to give my dashboard the look and feel of the [official ATP website](https://www.atptour.com/en), I went on the website and grab some design elements: logos, colors, fonts, background images.  
I am quite satisfied with the results, I wanted to keep it simple.  
Now remember the pie chart for Wins/Losses ratio, I wanted this to look like a tennis ball (hence the pie chart format). So I used a transparent tennis ball PNG image and placed the pie chart underneath. Et voilà!  

<img src="https://github.com/mboss10/Tennis-ATP-Top-20-Chart-Race/blob/main/Dashboard.png" width="600"><br />


### Publish to Tableau public



export csv  
build viz (rank with page)
ATP look and feel
tennis ball pie chart
country flags icon and tennis player icons
Publish to Tableau public

provide link to youttube

## To Go further
find way to connect to SQLite direct
Other analysis
