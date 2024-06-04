# Tennis ATP Top 20 Chart Race

<img src="https://github.com/mboss10/Tennis-ATP-Top-20-Chart-Race/blob/main/ATPTop20.gif" width="600">

## Introduction

The purpose of this project was to use the Huge Tennis Database that you can find on [Kaggle](https://www.kaggle.com/datasets/guillemservera/tennis) to mine tennis data and put together an animated chart race of the Top 20 ATP tennis player from 1990 to 2024.

## Process

I went through the following steps:
1. Database Setup: Creating the DB environment in DBeaver.
2. Data Cleaning: Using SQL views to ensure clean and accurate data.
3. Visualization: Importing the dataset into Tableau and building the dashboard with design elements inspired by the official ATP website.

I will detail below the different steps.

## 1. Database Setup

Once I have downloaded the SQLite database file from Kaggle, I imported it into my local DBeaver SQL client.
This is a very simple process:
1. Connect to a database - select SQLite in this case.  
<img src="https://github.com/mboss10/Tennis-ATP-Top-20-Chart-Race/blob/main/Connect_a_DB.png" width="400"><br /> 
2. Browse the folder in which I saved the SQLite database file annd select it.  
<img src="https://github.com/mboss10/Tennis-ATP-Top-20-Chart-Race/blob/main/SelectFile.png" width="400"><br />
3. Test connection, success then click Finish.  
<img src="https://github.com/mboss10/Tennis-ATP-Top-20-Chart-Race/blob/main/TestConnection.png" width="400"><br />

Then I ended up with a new database, that I named Tennis_ATP, that contains 3 tables:
1. matches = contains match results and stats (winner, loser, date, aces, etc ...)
2. players = contains players' basic info (first_name, last_name, hand, birth_date, country_code, height, etc ...)
3. rankings = contains players ranking per ranking period (every week almost, except for some missing periods)
