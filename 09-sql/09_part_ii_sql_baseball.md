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
    
Output:
('aaronha01', 12)
('boyerke01', 12)
('cepedor01', 12)
('mantlmi01', 12)
('mayswi01', 12)
('musiast01', 12)
('aparilu01', 11)
('bankser01', 11)
('berrayo01', 11)
('howarel01', 11)
```
Six players are tied at 12 games for the most all star games, with IDs: ('aaronha01', 12) ('boyerke01', 12) ('cepedor01', 12) ('mantlmi01', 12) ('mayswi01', 12) ('musiast01', 12)


4. Which school has generated the most distinct players? *Hint:* Create new table from 'CollegePlaying.csv'(question has typo, should be 'SchoolsPlayers.csv'?).

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
    
--Output:
(102, 'usc')
(100, 'texas')
(98, 'arizonast')
(82, 'stanford')
(77, 'michigan')
(75, 'holycross')
(70, 'notredame')
(68, 'illinois')
(66, 'arizona')
(66, 'ucla')

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
    
--Output:
(50, None)
(182, '01')
(1006, '02')
(1040, '03')
(1799, '04')
(1780, '05')
```
So there are more debuts in April and May, with May being the most popular month. There are also 50 players who did no enter a debut month.


7. What is the effect of table join order on mean salary for the players listed in the main (master) table? *Hint:* Perform two different queries, one that joins on playerID in the salary table and other that joins on the same column in the master table. You will have to use left joins for each since right joins are not currently supported with SQLalchemy.

```
c.execute('''  
SELECT MASTER.PLAYERID, AVG(SALARY)
FROM MASTER 
    LEFT JOIN SALARIES
    ON SALARIES.PLAYERID = MASTER.PLAYERID -- without equals sign, returns none
GROUP BY SALARIES.PLAYERID
--ORDER BY AVG(SALARY) DESC
LIMIT 10
;''')
for row in c.fetchall():
    print (row)
    
#Output:
('abbeybe01', None)
('aardsda01', 1322821.4285714286)
('abbotje01', 246250.0)
('abbotji01', 1440055.5555555555)
('abbotky01', 129500.0)
('accarje01', 654880.0)
('ackerji01', 426500.0)
('acrema01', 133500.0)
('adamsmi03', 2280133.3333333335)
('adamsru01', 329500.0)


    
#REPEAT QUERY, EXCEPT SWITCHING THE ORDER OF PLAYERID'S TO JOIN ON:
c.execute('''  
SELECT MASTER.PLAYERID, AVG(SALARY)
FROM MASTER 
    LEFT JOIN SALARIES
    ON MASTER.PLAYERID = SALARIES.PLAYERID
GROUP BY MASTER.PLAYERID
--ORDER BY AVG(SALARY) DESC
LIMIT 10;''')
for row in c.fetchall():
    print (row)
    
#Output:
('aardsda01', 1322821.4285714286)
('abbeybe01', None)
('abbotgl01', None)
('abbotje01', 246250.0)
('abbotji01', 1440055.5555555555)
('abbotky01', 129500.0)
('abbotod01', None)
('abernte01', None)
('ablesha01', None)
('accarje01', 654880.0)
```

When we order results by AVG(salary), the top 10 avg salary results are identical, suggesting swapping the table join order on mean salary for the players does not seem to have an effect on the 10 maximum values. This could be due to the column we are joining them on (PlayerID) exists in both tables.

However, if we return outputs based on their default order (Grouping by salary.playerID and Master.playerID, respectively, the outputs are different because some player IDs don't seem to exist in other columns. So the order of the IDs joined on does have an effect.
