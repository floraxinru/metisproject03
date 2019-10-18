# Challenge Set 9
## Part III: Soccer Data

*Introductory - Intermediate level SQL*

--

Please complete this exercise using sqlite3 and Jupyter notebook.

Download the [SQLite database](https://www.kaggle.com/hugomathien/soccer/downloads/soccer.zip) and load in your notebook using the sqlite3 library. 

1. Which team scored the most points when playing at home?  

2. Did this team also score the most points when playing away?  

3. How many matches resulted in a tie?  

4. How many players have Smith for their last name? How many have 'smith' anywhere in their name?

5. What was the median tie score? Use the value determined in the previous question for the number of tie games. *Hint:* PostgreSQL does not have a median function. Instead, think about the steps required to calculate a median and use the [`WITH`](https://www.postgresql.org/docs/8.4/static/queries-with.html) command to store stepwise results as a table and then operate on these results. 

6. What percentage of players prefer their left or right foot? *Hint:* Calculate either the right or left foot, whichever is easier based on how you setup the problem.


My code (worked in Jupyter Notebook, uploaded here:):
```
import sqlite3
import pandas as pd
conn = sqlite3.connect("database.sqlite")

#loop up table names using pandas
q = "SELECT * FROM sqlite_master WHERE type='table';"
pd.read_sql_query(q,conn)

##alternative method, looking up table names using cursor
#cursor = conn.cursor()
#cursor.execute("SELECT name FROM sqlite_master WHERE type='table';")
#print(cursor.fetchall())
```

#### 1. Which team scored the most points when playing at home?

```
q1=pd.read_sql("""SELECT HT.team_long_name AS  home_team, 
                    avg(home_team_goal) AS avg_home_team_gls
                        FROM Match
                        LEFT JOIN Team AS HT on HT.team_api_id = Match.home_team_api_id
                        ORDER BY avg_home_team_gls DESC
                        LIMIT 10;""", conn)
q1
```
KRC Genk scored the most when at home, averaging 1.54 goals per home game.


#### 2. Did this team also score the most points when playing away?
```
q2=pd.read_sql("""SELECT HT.team_long_name AS  home_team, 
avg(away_team_goal) AS avg_away_team_gls
                        FROM Match
                        LEFT JOIN Team AS HT on HT.team_api_id = Match.home_team_api_id
                        LEFT JOIN Team AS AT on AT.team_api_id = Match.away_team_api_id
                        ORDER BY avg_away_team_gls DESC
                        LIMIT 10;""", conn)
q2
```
Yes, surprisingly, this team also scored the most when playing away, at 1.16 goals per away game (but I've never heard of them! Googled and found out it's in Belgium, probably a good team in a weaker league). 

#### 3. How many matches resulted in a tie?
```
#First check how many matches there are in this database:

matches = pd.read_sql("""SELECT COUNT(*)                                        
                                FROM Match;""", conn)
matches

#There are 25979 matches total in this database.

#look at the tied games to check it returns one row per match:

q3=pd.read_sql("""SELECT date, home_team_goal, away_team_goal, HT.team_short_name, AT.team_short_name
                        FROM Match
                        LEFT JOIN Team AS HT on HT.team_api_id = Match.home_team_api_id
                        LEFT JOIN Team AS AT on AT.team_api_id = Match.away_team_api_id
                        WHERE home_team_goal = away_team_goal
                        ORDER BY date
               ;"""
               , conn)
q3  

>Output (first 5)
date	home_team_goal	away_team_goal	team_short_name	team_short_name
0	2008-07-27 00:00:00	0	0	YB	VAD
1	2008-08-02 00:00:00	0	0	XAM	AAR
2	2008-08-03 00:00:00	2	2	BEL	LUZ
3	2008-08-03 00:00:00	1	1	VAD	GRA
4	2008-08-06 00:00:00	1	1	BEL	AAR

#indeed returns one row per match

#answer for q3:
pd.read_sql("""SELECT COUNT(date) 
                        FROM Match
                        LEFT JOIN Team AS HT on HT.team_api_id = Match.home_team_api_id
                        LEFT JOIN Team AS AT on AT.team_api_id = Match.away_team_api_id
                        WHERE home_team_goal = away_team_goal
               ;"""
               , conn)


6596  matches resulted in a tie.
```


4. How many players have Smith for their last name? How many have 'smith' anywhere in their name?
```
##list of players with last name Smith
players = pd.read_sql("""SELECT *
                        FROM Player
                        WHERE player_name LIKE '%Smith'--Finds any values that end with "Smith"
                    
                        ORDER BY player_name
                        LIMIT 10;""", conn)
players


##number of players with last name Smith
ct_players = pd.read_sql("""SELECT COUNT(*)
                        FROM Player
                        WHERE player_name LIKE '%Smith'--Finds any values that end with "Smith"
                    
                        ORDER BY player_name
                        ;""", conn)
ct_players

##list of players with 'smith' anywhere in their names
smiths = pd.read_sql("""SELECT COUNT(*)
                        FROM Player
                        WHERE player_name LIKE '%smith%'--Finds any values that have "smith" in any position
                    
                        ORDER BY player_name
                         ;""", conn)
smiths

#ct_players and smiths both return 18
```
There are 18 players with 'Smith' as their last name, and also 18 with 'smith' anywhere in there name, so the only players with smith in their names have Smith as their last names.



5. What was the median tie score? Use the value determined in the previous question for the number of tie games. 



6. What percentage of players prefer their left or right foot? *Hint:* Calculate either the right or left foot, whichever is easier based on how you setup the problem.
