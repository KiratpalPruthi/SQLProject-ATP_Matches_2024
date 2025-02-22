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

Left to right ratio - 0.16:1 = 4:25

### Q3.	Which tournaments saw the most upsets in 2024?
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

### Q4. What was the average match duration on different surfaces from all ATP tournaments? What does it indicate?
The analysis includes averages from all ATP 250, ATP 500 and ATP Masters 1000 tournaments.
```
SELECT round(avg(minutes), 2) as match_duration, surface
FROM atp_matches_2024
WHERE tourney_level = 'M' or tourney_level = 'A'
GROUP BY surface
ORDER BY 1 desc;
```
| match_duration | surface |
|---------------|---------|
| 112.98       | Clay    |
| 105.67       | Hard    |
| 102.22       | Grass   |

This indicates that clay surface which is a slow surface leads to longer rallies and increased match duration. Grass court on the other hand is a fast court with the ball staying low and rallies being short. This leads to the shortest average match duration. 

## Standout Player Performances
### Q5. Who won the most tournaments in 2024?
```
SELECT winner_name, count(winner_name) as tournaments_won
FROM atp_matches_2024
WHERE round = "F"
GROUP BY winner_name
ORDER BY 2 desc
LIMIT 1;
```
| winner_name   | tournaments_won |
|--------------|----------------|
| Jannik Sinner | 8              |

### Q6. Which players had the highest win rates on different surfaces (hard, clay, grass)?
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

### Q7.	Rank top 10 players by highest average number of aces per set in 2024. How did that impact the match outcome?
 Top 10 players by average number of aces per set, having played a minimum of 5 sets in 2024 season were-
 ```
WITH player_match_data AS (
    SELECT 
        player_name,
        SUM(CASE WHEN player_name = winner_name THEN w_aces ELSE l_aces END) AS total_aces,
        SUM(CASE WHEN player_name = winner_name THEN 1 ELSE 0 END) AS matches_won,
        COUNT(*) AS total_matches,
        SUM(LENGTH(score) - LENGTH(REPLACE(score, ' ', ''))+1) AS total_sets_played  -- Counts number of sets from score
    FROM (
        SELECT winner_name AS player_name, w_aces, l_aces, score, winner_name, loser_name
        FROM atp_matches_2024
        UNION ALL
        SELECT loser_name AS player_name, l_aces, w_aces, score, winner_name, loser_name
        FROM atp_matches_2024
    ) AS match_data
    GROUP BY player_name
),

top_10_players AS (
    SELECT player_name, total_aces, matches_won, total_matches, total_sets_played,
           (1.0 * total_aces / NULLIF(total_sets_played, 0)) AS aces_per_set
    FROM player_match_data
	where total_sets_played >= 5 -- for statistical reliability
    ORDER BY aces_per_set DESC
    LIMIT 10
),

-- Compute Averages for Only the Top 10 Players
averages AS (
    SELECT 
        AVG(total_aces) AS avg_total_aces,
        AVG(matches_won) AS avg_matches_won
    FROM top_10_players
),

correlation_components AS (
    SELECT
        SUM((t10.total_aces - avg.avg_total_aces) * (t10.matches_won - avg.avg_matches_won)) AS numerator,
        SUM((t10.total_aces - avg.avg_total_aces) * (t10.total_aces - avg.avg_total_aces)) AS sum_sq_aces,
        SUM((t10.matches_won - avg.avg_matches_won) * (t10.matches_won - avg.avg_matches_won)) AS sum_sq_wins
    FROM top_10_players t10
    JOIN averages avg ON 1=1
)

-- Compute Pearson Correlation for the Top 10 Players
SELECT t10.player_name, t10.total_aces, t10.aces_per_set,
    CASE 
        WHEN cc.sum_sq_aces = 0 OR cc.sum_sq_wins = 0 THEN NULL  -- Prevents division by zero
        ELSE cc.numerator / SQRT(cc.sum_sq_aces * cc.sum_sq_wins) 
    END AS correlation_coefficient
FROM top_10_players t10
JOIN correlation_components cc on 1=1
ORDER BY t10.aces_per_set desc;
```
| player_name               | total_aces | aces_per_set | correlation_coefficient |
|---------------------------|------------|--------------|--------------------------|
| Giovanni Mpetshi Perricard | 442        | 5.8158      | 0.9779                  |
| Milos Raonic              | 166        | 5.7241       | 0.9779                   |
| Martin Damm               | 37         | 5.2857       | 0.9779                   |
| Reilly Opelka             | 206        | 5.1500       | 0.9779                   |
| Lloyd Harris              | 69         | 4.9286       | 0.9779                   |
| Patrick Kypson            | 77         | 4.8125       | 0.9779                   |
| Sergey Fomin              | 23         | 4.6000       | 0.9779                   |
| James Duckworth           | 277        | 4.5410       | 0.9779                   |
| Hamad Medjedovic          | 207        | 4.5000       | 0.9779                   |
| Hubert Hurkacz            | 723        | 4.3818       | 0.9779                   |

Pearson’s r of 0.98 shows a strong positive correlation between aces served and match outcome. 

### Q8.	Who was the first to win 20 matches in the 2024 season?
```
WITH match_wins AS (
    SELECT 
        winner_name AS player_name, 
        tourney_date, tourney_name,
        COUNT(*) OVER (
            PARTITION BY winner_name 
            ORDER BY CAST(tourney_date AS INTEGER) 
            ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW  -- counts each row separately as tourney_date may be same for multiple matches
        ) AS cumulative_wins
    FROM atp_matches_2024
),

first_to_20 AS (
    -- Find the earliest date when any player reached 20 wins
    SELECT MIN(tourney_date) AS first_win_date
    FROM match_wins
    WHERE cumulative_wins = 20
)

SELECT mw.player_name, mw.tourney_date, mw.tourney_name
FROM match_wins mw
JOIN first_to_20 ft ON mw.tourney_date = ft.first_win_date
WHERE mw.cumulative_wins = 20
ORDER BY mw.tourney_date;
```

| player_name      | tourney_date | tourney_name   |
|-----------------|-------------|---------------|
| Grigor Dimitrov | 2024-03-18  | Miami Masters |
| Jannik Sinner  | 2024-03-18  | Miami Masters  |
 
Both Jannik Sinner and Grigor Dimitrov achieved 20 wins in March at the Miami Masters.

### Q9. Which players showed the most improvement in their ranking in between the start and end of 2024 season?
```
WITH player_ranking_changes AS (
    SELECT 
        player_name,
        MIN(CASE WHEN tourney_date <= 20240228 THEN player_rank END) AS start_rank, -- lowest ranking in the first two months
        MAX(CASE WHEN tourney_date >= 20241101 THEN player_rank END) AS end_rank -- highest ranking in the last two months
    FROM (
        SELECT winner_name AS player_name, winner_rank AS player_rank, tourney_date FROM atp_matches_2024
        UNION ALL
        SELECT loser_name AS player_name, loser_rank AS player_rank, tourney_date FROM atp_matches_2024
    ) AS combined_data
    GROUP BY player_name
)

SELECT 
    player_name,
    start_rank,
    end_rank,
    (start_rank - end_rank) AS rank_improvement
FROM player_ranking_changes
WHERE start_rank IS NOT NULL AND end_rank IS NOT NULL
ORDER BY rank_improvement DESC
LIMIT 5;
```
| player_name       | start_rank | end_rank | rank_improvement |
|------------------|------------|----------|------------------|
| Rafael Nadal     | 672        | 154      | 518              |
| Marin Cilic      | 674        | 196      | 478              |
| Joao Fonseca     | 343        | 145      | 198              |
| Juncheng Shang   | 139        | 50       | 89               |
| Corentin Moutet  | 140        | 66       | 74               |


### Q10. Which unseeded players caused the most upsets by defeating higher-ranked opponents?
```
WITH tennis_upsets as 
     (SELECT winner_name, winner_seed, loser_seed, winner_age
      FROM atp_matches_2024
      WHERE winner_seed is null and loser_seed is not null
     )
SELECT winner_name, count(*) as number_of_upsets, winner_age
FROM tennis_upsets
GROUP BY winner_name
ORDER BY 2 desc
LIMIT 3 ;
```
| winner_name     | number_of_upsets | winner_age |
|----------------|-----------------|------------|
| Tomas Machac   | 10              | 23.2       |
| Juncheng Shang | 10              | 18.9       |
| Alexei Popyrin | 10              | 24.5       |

Despite being unranked, these three up and coming players with an average age of 22.2 years, have shown that they can challenge the world's best. They are ones to look out for in the future.

### Q11. Which players had the highest percentage of winning tie-breaks?

```

WITH tie_break_sets AS (
    SELECT 
        winner_id, winner_name,
        loser_id, loser_name,
        score,
        CASE 
            WHEN INSTR(score, '7-') > 0 AND INSTR(score, '6(') > 0 THEN winner_id 
            WHEN INSTR(score, '6(') > 0 AND INSTR(score, '-7') > 0 THEN loser_id
        END AS tie_break_winner,
        CASE 
            WHEN (INSTR(score, '7-') > 0 AND INSTR(score, '6(') > 0) 
                 OR (INSTR(score, '6(') > 0 AND INSTR(score, '-7') > 0) THEN 1
            ELSE 0
        END AS is_tie_break
    FROM atp_matches_2024
),
tie_break_stats AS (
    SELECT 
        player_id, player_name,
        SUM(CASE WHEN player_id = tie_break_winner THEN 1 ELSE 0 END) AS tie_breaks_won,
        COUNT(*) AS tie_breaks_played
    FROM (
        SELECT winner_id AS player_id, winner_name as player_name, tie_break_winner, is_tie_break FROM tie_break_sets
        UNION ALL
        SELECT loser_id AS player_id, loser_name as player_name, tie_break_winner, is_tie_break FROM tie_break_sets
    ) AS all_players
    WHERE is_tie_break = 1
    GROUP BY player_id
)
-- Calculate the win percentage for tie-break sets
SELECT 
    player_id, player_name,
    tie_breaks_won,
    tie_breaks_played,
    round((tie_breaks_won * 100.0 / tie_breaks_played), 2) as win_percentage
FROM tie_break_stats
WHERE tie_breaks_played >= 5 -- minmmum 5 for statistical reliability
ORDER BY win_percentage DESC
LIMIT   5;
```
| Player_ID | Player_Name      | Tie_Breaks_Won | Tie_Breaks_Played | Win_Percentage |
|-----------|-----------------|----------------|-------------------|---------------|
| 206173    | Jannik Sinner   | 18             | 20                | 90.00%        |
| 200384    | Hugo Gaston     | 6              | 7                 | 85.71%        |
| 104925    | Novak Djokovic  | 10             | 12                | 83.33%        |
| 105777    | Grigor Dimitrov | 18             | 22                | 81.82%        |
| 106421    | Daniil Medvedev | 18             | 23                | 78.26%        |









