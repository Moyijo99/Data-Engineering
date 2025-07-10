-- Create the players_scd table to store SCD (Slowly Changing Dimension) records
CREATE TABLE players_scd (
    player_name TEXT,
    scoring_class scoring_class,
    is_active BOOLEAN,
    start_season INTEGER,
    end_season INTEGER,
    current_season INTEGER,
    PRIMARY KEY(player_name,start_season)
);

-- Drop the players_scd table (for testing or re-creation purposes)
DROP TABLE players_scd;

-- Select all records from players_scd (for verification)
SELECT * FROM players_scd;

-- Insert SCD records into players_scd using a series of CTEs
INSERT INTO players_scd
WITH with_previous AS(
    -- Add previous scoring_class and is_active for each player/season
    SELECT 
        player_name,
        current_season,
        scoring_class, 
        is_active,
        LAG(scoring_class, 1) OVER (PARTITION BY player_name ORDER BY current_season) AS previous_scoring_class,
        LAG(is_active, 1) OVER (PARTITION BY player_name ORDER BY current_season) AS previous_is_active
    FROM players
    WHERE current_season <= 2021
), 
with_indicators AS (
    -- Mark where a change in scoring_class or is_active occurs
    SELECT *, CASE 
        WHEN scoring_class <> previous_scoring_class THEN 1 
        WHEN is_active <> previous_is_active THEN 1 
        ELSE 0
    END AS change_indicator
    FROM with_previous
),
with_streaks AS (
    -- Assign a streak_identifier to group unchanged periods
    SELECT *, 
        SUM(change_indicator) OVER (PARTITION BY player_name ORDER BY current_season) AS streak_identifier
    FROM with_indicators
)
-- Aggregate to get SCD rows: one per unchanged streak
SELECT player_name,
       scoring_class,
       is_active,
       MIN(current_season) AS start_season,
       MAX(current_season) AS end_season,
       2021 AS current_season
FROM with_streaks
GROUP BY player_name,streak_identifier,is_active,scoring_class
ORDER BY player_name, streak_identifier ASC