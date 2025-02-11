# SQLProject-ATP_Matches_2024
This project aims to demonstrate proficiency in SQL querying and analyzing sports data. Using the ATP_Matches_2024 dataset the exploratory data analysis provides insights into player performance and match/tournament statistics that any tennis lover would find valuable.
SQL skills used -
1. Aggregate functions
2. Window functions
3. Converting data types
4. Joins
5. CTEs
6. Case when statements
7. Unions
8. Subqueries

TOOL USED - **DB Browser for SQLite**

```
CREATE TABLE "atp_matches_2024" (
	"tourney_id"	INTEGER,
	"tourney_name"	TEXT,
	"surface"	TEXT,
	"draw_size"	INTEGER,
	"tourney_level"	TEXT,
	"tourney_date"	INTEGER,
	"match_num"	TEXT,
	"winner_id"	INTEGER,
	"winner_seed"	TEXT,
	"winner_entry"	TEXT,
	"winner_name"	TEXT,
	"winner_hand"	TEXT,
	"winner_height"	INTEGER,
	"winner_ioc"	TEXT,
	"winner_age"	REAL,
	"loser_id"	INTEGER,
	"loser_seed"	TEXT,
	"loser_entry"	TEXT,
	"loser_name"	TEXT,
	"loser_hand"	TEXT,
	"loser_height"	INTEGER,
	"loser_ioc"	TEXT,
	"loser_age"	REAL,
	"score"	TEXT,
	"best_of"	INTEGER,
	"round"	TEXT,
	"minutes"	INTEGER,
	"w_aces"	INTEGER,
	"w_df"	INTEGER,
	"w_svpt"	INTEGER,
	"w_1stin"	INTEGER,
	"w_1stwon"	INTEGER,
	"w_2ndwon"	INTEGER,
	"w_SvGms"	INTEGER,
	"w_BpSaved"	INTEGER,
	"w_BpFaced"	INTEGER,
	"l_aces"	INTEGER,
	"l_df"	INTEGER,
	"l_svpt"	INTEGER,
	"l_1stin"	INTEGER,
	"l_1stwon"	INTEGER,
	"l_2ndwon"	INTEGER,
	"l_SvGms"	INTEGER,
	"l_BpSaved"	INTEGER,
	"l_BpFaced"	INTEGER,
	"winner_rank"	INTEGER,
	"winner_rank_points"	INTEGER,
	"loser_rank"	INTEGER,
	"loser_rank_points"	INTEGER
);
```
## General ATP Statistics
### Q1. Which countries produce the most ATP players?
```
WITH all_players AS (
    SELECT winner_ioc AS player_country, winner_id AS player_id
    FROM atp_matches_2024
    UNION
    SELECT loser_ioc AS player_country, loser_id AS player_id
    FROM atp_matches_2024
),
unique_players AS (
    SELECT DISTINCT player_id, player_country
    FROM all_players
)
SELECT player_country, COUNT(player_id) AS num_players
FROM unique_players
GROUP BY player_country
ORDER BY num_players DESC
LIMIT  3;
```

| player_country | num_players |
|---------------|------------|
| USA           | 35         |
| FRA           | 32         |
| AUS           | 19         |

### Q2.	What is the ratio of left handed to right handed players at the ATP tour level? 
```
WITH dominant_hand_stats as
(SELECT winner_id as player_id, winner_name as player_name, winner_hand as player_hand
FROM atp_matches_2024
 UNION ALL
 SELECT loser_id as player_id, loser_name as player_name, loser_hand as player_hand 
 FROM atp_matches_2024)
SELECT 
    SUM(CASE WHEN player_hand = 'L' THEN 1 ELSE 0 END) AS left_handed_players,
    SUM(CASE WHEN player_hand = 'R' THEN 1 ELSE 0 END) AS right_handed_players,
    ROUND(1.0 * SUM(CASE WHEN player_hand = 'L' THEN 1 ELSE 0 END) / 
              SUM(CASE WHEN player_hand = 'R' THEN 1 ELSE 0 END), 2) AS left_to_right_ratio
FROM dominant_hand_stats;

```
| left_handed_players | right_handed_Players | left_to_right_ratio |
|---------------------|----------------------|----------------------|
| 832                 | 5284                 | 0.16                 |

### Q3.	Which tournaments have seen the most upsets?
Upsets defined as unseeded players beating players seeded 10 or lower.
```
 SELECT tourney_name, sum(case when winner_seed is null and cast(loser_seed as unsigned) <= 10 then 1
          else 0
         end) as tennis_upsets
FROM atp_matches_2024
GROUP BY tourney_name
ORDER BY tennis_upsets DESC
LIMIT 5 ;
```
| tourney_name      | tennis_upsets |
|------------------|--------------|
| Canada Masters  | 9            |
| Winston-Salem   | 8            |
| Tokyo           | 8            |
| Marrakech       | 8            |
| Auckland        | 8            |

### Q4. What are the average match durations of different surfaces from all ATP tournaments? What does it indicate?
The analysis includes averages from all ATP 250, ATP 500 and ATP Masters 1000 tournaments.
```
SELECT round(avg(minutes), 2) as match_duration, surface
FROM atp_matches_2024
WHERE tourney_level = 'M' or tourney_level = 'A'
GROUP BY surface
ORDER BY 1 desc;
```
| Match_Duration | Surface |
|---------------|---------|
| 112.98       | Clay    |
| 105.67       | Hard    |
| 102.22       | Grass   |

This indicates that clay surface which is a slow surface leads to longer rallies and increased match duration. Grass court on the other hand is a fast court with the ball staying low and rallies being short. This leads to the shortest average match duration. 

## Standout Player Performances
### Q5. 1.	Who won the most tournaments in 2024?
```
SELECT winner_name, count(winner_name) as tournaments_won
FROM atp_matches_2024
WHERE round = "F"
GROUP BY winner_name
ORDER BY 2 desc
LIMIT 1;
```
| Winner_Name   | Tournaments_Won |
|--------------|----------------|
| Jannik Sinner | 8              |

### Q6. Which players have the highest win rates on different surfaces (hard, clay, grass)?
```
select player_id, player_name,
       matches_played, 
	   matches_won, surface,
	   (matches_won * 100/ matches_played) as win_rate
from (select player_id, winner_name as player_name,
             count(*) as matches_played, surface,
	         sum(case when player_id = winner_id then 1 else 0
	         end) as matches_won
	  from (select winner_id as player_id, winner_id, loser_id, winner_name, surface from atp_matches_2024
	        union all select loser_id as player_id, winner_id, loser_id, loser_name, surface from atp_matches_2024) as all_matches
	  group by player_id, surface
) as player_stats
where matches_played >= 10 -- A minimum value set for reliability of players performance
order by win_rate desc, surface desc
limit 3;
```
| player_id | player_name       | matches_played | matches_won | surface | win_rate |
|-----------|------------------|---------------|------------|---------|----------|
| 206173    | Jannik Sinner    | 57            | 54         | Hard    | 94       |
| 126610    | Matteo Berrettini | 16            | 15         | Clay    | 93       |
| 206173    | Jannik Sinner    | 10            | 9          | Grass   | 90       |

### Q7.	Rank top 10 players that serve the highest number of average aces  per set in matches. How does that impact the match outcome?
 Top 10 players by average number of aces per set, having played a minimum of 5 sets in 2024 season are-
### Q8.	Which player was the first to win 20 games in the 2024 season?
### Q9. Which players showed the most improvement in their ranking in between the start and end of 2024 season?
### Q10. Which unseeded players have caused the most upsets by defeating higher-ranked opponents?
### Q11. Which players have the highest percentage of winning tie-break sets?










