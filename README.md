# Session-6
The script demonstrates SQL queries with JOIN, UNION, and window functions. It covers multiple join conditions, WHERE clause in JOIN, percentage calculations, ranking, renaming columns, and using window functions for partitioned calculations. The approach enables comprehensive analysis of cricket player statistics stored in various tables.

SELECT * 
FROM atomcamp.players_wickets;

SELECT * 
FROM atomcamp.players_runs;


## JOINING TWO TABLES WHERE THEY DO NOT HAVE A PRIMARY KEY, USE MULTIPLE COLUMNS IN JOIN CRITERIA 
# such that combination of those columns is unique

SELECT w.id, w.`match`, w.wickets, r.runs
FROM atomcamp.players_wickets AS w
LEFT JOIN atomcamp.players_runs AS r
ON w.id = r.id AND w.`match` = r.`match`
ORDER BY w.id, w.`match`; # you can also do ORDER BY 1,2

# You can also use WHERE clause in a JOIN statement
# Let's say we wanted to see stats only for first match

SELECT w.id, w.`match`, w.wickets, r.runs
FROM atomcamp.players_wickets AS w
LEFT JOIN atomcamp.players_runs AS r
ON w.id = r.id AND w.`match` = r.`match`
WHERE w.`match` = 1
ORDER BY w.id, w.`match`;

# What percentage of runs in match 1 were made by each player

SELECT  id, 100*sum(runs)/(SELECT sum(runs) FROM atomcamp.players_runs WHERE `match`=1) AS pct_of_runs
FROM atomcamp.players_runs
WHERE `match` = 1 
GROUP BY id;

# What percentage of wickets did each player get in match 2


SELECT id, 100*sum(wickets)/(SELECT sum(wickets) FROM atomcamp.players_wickets WHERE `match`= 2 ) AS pct_of_wickets
FROM atomcamp.players_wickets
WHERE `match` =2
GROUP BY id;

### What percentage of wickets did each player get in each match (single query)
WITH t1 AS (

SELECT `match`, sum(wickets) AS sum_of_wickets
FROM atomcamp.players_wickets
GROUP BY 1
) ,

t2 AS (

	SELECT base.*, agg.sum_of_wickets
    FROM atomcamp.players_wickets AS base
    LEFT JOIN t1 AS agg
    ON base.`match` = agg.`match`
)

SELECT id, `match`, 100*(wickets/sum_of_wickets) AS pct_of_wickets
FROM t2;


WITH t0 AS (
SELECT `match`, sum(runs) AS sum_runs
FROM atomcamp.players_runs
group by 1),

 t1 AS (

SELECT  a.*, b.sum_runs 
FROM atomcamp.players_runs a 
LEFT JOIN t0 b 
ON a.`match` = b.`match`
) 
SELECT id, `match`, 100*runs/sum_runs AS pct_of_runs  
FROM t1;




DESCRIBE stats_wickets;
DESCRIBE stats_runs;

SELECT * FROM stats_wickets;
SELECT * FROM stats_runs;

# Write a query that joins wickets and runs table to show id, matches played, wickets taken, 
# and runs made in the same table

# NOW ADD AVG RUNS and AVG WICKETS for each player

SELECT w.*, r.runs, ROUND(r.runs/w.`match`,1) AS batting_avg, ROUND(w.wickets/w.`match`,1) AS bowling_avg
FROM stats_wickets w
LEFT JOIN stats_runs r
ON w.id = r.id;

SELECT * FROM stats_wickets;

# You can RANK your data based on a certain criteria 
# RANK orders the column mentioned and then ranks it, equal values get same rank, while the next onoe depends on how many values precede that value

SELECT id, `match` AS matches_played, wickets AS wickets_taken, 
RANK () OVER(ORDER BY wickets DESC) AS bolwer_rank
FROM stats_wickets;

# Alternate method using DENSE_RANK, uses all integers to rank in a sequence
SELECT id, `match` AS matches_played, wickets AS wickets_taken, 
DENSE_RANK () OVER( ORDER BY wickets DESC) AS bolwer_rank
FROM stats_wickets;



# Calculate rank and dense rank for each player using their batting_avg

SELECT w.*, r.runs, ROUND(r.runs/w.`match`,1) AS batting_avg, ROUND(w.wickets/w.`match`,1) AS bowling_avg,
RANK () OVER( ORDER BY ROUND(r.runs/w.`match`,1) DESC) AS batting_avg_rank
FROM stats_wickets w
LEFT JOIN stats_runs r
ON w.id = r.id;

SELECT * FROM stats_wickets;

### CHANGING COLUMN NAMES OF A TABLE IN THE SQL SYSTEM

ALTER TABLE stats_wickets
RENAME COLUMN id TO player_id;

ALTER TABLE stats_wickets
RENAME COLUMN `match` TO matches_played;

ALTER TABLE stats_wickets
RENAME COLUMN wickets TO wickets_taken;

SELECT * FROM stats_wickets;

# JOIN TABLE stats_wickets with stats_runs to show runs and wickets in the same output
# Column names in the join criteria can be different, it does not matter. The logic stays the same

SELECT w.*, r.runs, 
ROUND(r.runs/w.matches_played,1) AS batting_avg, 
ROUND(w.wickets_taken/w.matches_played,1) AS bowling_avg
FROM stats_wickets w
LEFT JOIN stats_runs r
ON w.player_id = r.id;


### Rename column runs to runs_made in STATS_RUNS table

ALTER TABLE stats_runs
RENAME COLUMN runs TO runs_made;

SELECT * FROM stats_runs_2;
SELECT * FROM stats_runs;

# How to show output of two tables through single query
SELECT * FROM stats_runs
UNION
SELECT * FROM stats_runs_2;

# You can use UNION command to show contents of multiple tables at once. 
# Order of columns and number of columns needs to be the same. If not do not use Select *, instead manually pick
# columns and put them in the same order. 


### Let's return to window functions! :P

SELECT * 
FROM stats_runs_category;

# RANK each innings/runs made within each match type
#You can use PARTITION BY in order to calculate a column or rank within a category (coming from a different column)

SELECT id, match_type, runs_made,
RANK() OVER(PARTITION BY match_type ORDER BY runs_made DESC) as innings_rank
FROM stats_runs_category;


#Calculate what percentage of a players total runs were made in a singe/given innings (for each match type) 
# You can have two partitions as well or more than two. Just add them as shown below
SELECT id, match_type, runs_made, 
SUM(runs_made) OVER(PARTITION BY id, match_type) AS total_runs_made,
100*runs_made/SUM(runs_made) OVER(PARTITION BY id, match_type) AS pct_of_total_runs
FROM stats_runs_category;
