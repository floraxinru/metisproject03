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
q3=pd.read_sql("""SELECT COUNT(home_team_goal = away_team_goal) 
                        FROM Match
                        LEFT JOIN Team AS HT on HT.team_api_id = Match.home_team_api_id
                        LEFT JOIN Team AS AT on AT.team_api_id = Match.away_team_api_id;""", conn)
q3  
#when using SUM, get 6596, probably summing up all the goals (lots of 0-0 games), not all the tied games
#The COUNT() function returns the number of rows that matches a specified criteria.
#The SUM() function returns the total sum of a numeric column.


```
25979  games resulted in a tie.



4. How many players have Smith for their last name? How many have 'smith' anywhere in their name?

5. What was the median tie score? Use the value determined in the previous question for the number of tie games. 

6. What percentage of players prefer their left or right foot? *Hint:* Calculate either the right or left foot, whichever is easier based on how you setup the problem.
