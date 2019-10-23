# Challenge Set 9
## Part II: Baseball Data

*Introductory - Intermediate level SQL*

--

Please complete this exercise via SQLalchemy and Jupyter notebook.

We will be working with the Lahman baseball data we uploaded to your AWS instance in class. 

```
#using sqlite3, create table for each csv

import sqlite3
import pandas as pd
from pandas import DataFrame

conn = sqlite3.connect('BASEBALLDB2.db')
c = conn.cursor()
```
1. What was the total spent on salaries by each team, each year?

```
#CREATE SALARIES TABLE - only run this once
c.execute('CREATE TABLE SALARIES (yearID, teamID, lgID,playerID,salary)')
conn.commit()

sal_df = pd.read_csv('baseballdata/Salaries.csv') #Read a comma-separated values (csv) file into DataFrame.

#from df to sql to use query:  -- also only run this once
sal_df.to_sql('SALARIES', conn, if_exists='replace', index = False)

c.execute('''  
SELECT yearID, teamID, SUM(salary)
FROM SALARIES
GROUP BY yearID, teamID
--LIMIT 10;
          ''')

for row in c.fetchall():
    print (row)
    
```
Top 10 teams with highest total salaries:
```
(1985, 'ATL', 14807000)
(1985, 'BAL', 11560712)
(1985, 'BOS', 10897560)
(1985, 'CAL', 14427894)
(1985, 'CHA', 9846178)
(1985, 'CHN', 12702917)
(1985, 'CIN', 8359917)
(1985, 'CLE', 6551666)
(1985, 'DET', 10348143)
(1985, 'HOU', 9993051)
```



2. What is the first and last year played for each player? *Hint:* Create a new table from 'Fielding.csv'.
```
fie_df = pd.read_csv('baseballdata/Fielding.csv')
#from df to sql to use query:
fie_df.to_sql('FIELDING', conn, if_exists='replace', index = False)
### JOIN OTHER TABLE FOR LAST YEAR?
```

3. Who has played the most all star games?
```
as_df = pd.read_csv('baseballdata/AllstarFull.csv') #typo, readme has it as AllStarFull.csv

as_df.to_sql('ALLSTAR', conn, if_exists='replace', index = False)

c.execute('''  
SELECT playerID, SUM(gameNum)
FROM ALLSTAR
GROUP BY playerID
ORDER BY SUM(gameNum) DESC
LIMIT 10;
          ''')

for row in c.fetchall():
    print (row)
```
Six players are tied at 12 games for the most all star games, with IDs: ('aaronha01', 12) ('boyerke01', 12) ('cepedor01', 12) ('mantlmi01', 12) ('mayswi01', 12) ('musiast01', 12)


4. Which school has generated the most distinct players? *Hint:* Create new table from 'CollegePlaying.csv'.(typo, should be 'SchoolsPlayers.csv'?)

```
sp_df = pd.read_csv('baseballdata/SchoolsPlayers.csv') 
sp_df.to_sql('SCHPLAYERS', conn, if_exists='replace', index = False)

c.execute('''  
SELECT COUNT(DISTINCT(playerID)), SchoolID
FROM SCHPLAYERS
GROUP BY SchoolID
ORDER BY COUNT(DISTINCT(PLAYERID)) DESC
LIMIT 10;
          ''')

for row in c.fetchall():
    print (row)
```
USC has generated the largest number of distinct players, at 102 players.


5. Which players have the longest career? Assume that the `debut` and `finalGame` columns comprise the start and end, respectively, of a player's career. *Hint:* Create a new table from 'Master.csv'. Also note that strings can be converted to dates using the [`DATE`](https://wiki.postgresql.org/wiki/Working_with_Dates_and_Times_in_PostgreSQL#WORKING_with_DATETIME.2C_DATE.2C_and_INTERVAL_VALUES) function and can then be subtracted from each other yielding their difference in days.

```
#--sqlite doesn't have day() or Datediff function- only date, time, datetime
    #convert to JulianDay to find difference in days

MAS_df.to_sql('MASTER', conn, if_exists='replace', index = False)

c.execute('''  
SELECT PLAYERID, JulianDay(YEARMAX)-JulianDay(YEARMIN) as daydiff
FROM MASTER
GROUP BY PLAYERID
ORDER BY daydiff DESC
LIMIT 10
          ;''')

for row in c.fetchall():
    print (row)
```
So the player with the ID 'herrmfr01', with a career of 2006 days, followed closely by 'mitrese01' at 2001 days.


6. What is the distribution of debut months? *Hint:* Look at the `DATE` and [`EXTRACT`](https://www.postgresql.org/docs/current/static/functions-datetime.html#FUNCTIONS-DATETIME-EXTRACT) functions.

```
#sqlite3 doesn't support EXTRACT function, using strftime instead
c.execute('''  
SELECT COUNT(DISTINCT(PLAYERID)), strftime('%m', DATE(YEARMIN)) as month 
FROM MASTER
GROUP BY month
ORDER BY month
;''')
for row in c.fetchall():
    print (row)
```
So there are more debuts in April and May, with May being the most popular month. There are also 50 players who did no enter a debut month.


7. What is the effect of table join order on mean salary for the players listed in the main (master) table? *Hint:* Perform two different queries, one that joins on playerID in the salary table and other that joins on the same column in the master table. You will have to use left joins for each since right joins are not currently supported with SQLalchemy.
