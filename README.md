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


The technique used here is very simple given that rank is part of the ranking table so I just had to filter on `rank < 21` to get only the top 20 for each ranking week. Combined with an `INNER JOIN` with the players table it allows me to pull name, hand, date of birth, country and height of those players.
> [!NOTE]
> Notice how I created a full_name by concatenating first and last name. In SQLite you use 2 pipe signs `||` to concatenate strings

  
The other thing I wanted to include in my bar chart race is the picture of the Top player on the selected week along with its running match records (wins VS losses). So I had to come up with a query that would provide me with the necessary data to achieve this.  
See below the query:  
```
```
