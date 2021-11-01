```sql
--Query 1, GOAL: to compute the fractional retention by day joined to the game
WITH
  player_lastGame AS (
  SELECT
    -- select player id, day they joined, and the last day they played
    player.player_id,
    player.joined,
    MAX(match.day) AS lastGameDay
  FROM
    `juno-329514.gameTestingCompany.matches_info` AS match
  JOIN
    `juno-329514.gameTestingCompany.player_info` AS player
  ON
    match.player_id=player.player_id -- join on player_id
  GROUP BY
    player.player_id,
    player.joined),
  retention_table AS (
  SELECT -- Create a new column to indetify if the payer was retained or not (1 = yes, 0 = No)
    player_id,
    joined,
    lastGameDay,
    CASE -- Add a new column with TRUE when there was retention after 30 days
      WHEN (lastGameDay - joined) > 30 THEN 1 --IF days lapsed between the day the player joined and the day they played is greater than 30 then set TRUE
    ELSE
    0 -- IF not set to FALSE
  END
    AS retention
  FROM
    player_lastGame)
SELECT -- Compute fractional retention by day joined
  joined AS day_joined,
  count(player_id) AS playersJoined,
  SUM(retention) AS playersRetained,
  (SUM(retention) / count(player_id)) AS fractionalRetention
FROM retention_table
GROUP BY day_joined
ORDER BY day_joined ASC

--QUERY 2, GOAL: to compute fractional retention by age
WITH
  player_lastGame AS (
  SELECT
    -- select player id, day they joined, and the last day they played
    player.player_id,
    player.age,
    player.joined,
    MAX(match.day) AS lastGameDay
  FROM
    `juno-329514.gameTestingCompany.matches_info` AS match
  JOIN
    `juno-329514.gameTestingCompany.player_info` AS player
  ON
    match.player_id=player.player_id -- join on player_id
  GROUP BY
    player.player_id,
    player.age,
    player.joined),
  retention_table AS (
  SELECT
    player_id,
    age,
    joined,
    lastGameDay,
    CASE -- Add a new column with TRUE when there was retention after 30 days
      WHEN (lastGameDay - joined) > 30 THEN 1 --IF days lapsed between the day the player joined and the day they played is greater than 30 then set TRUE
    ELSE
    0 -- IF not set to FALSE
  END
    AS retention
  FROM
    player_lastGame)
SELECT
  age AS players_age,
  count(player_id) AS playersJoined,
  SUM(retention) AS playersRetained,
  (SUM(retention) / count(player_id)) AS fractionalRetention
FROM retention_table
GROUP BY players_age
ORDER BY players_age ASC

--QUERY 3, GOAL: To compute retention by location
WITH
  player_lastGame AS (
  SELECT
    -- select player id, day they joined, and the last day they played
    player.player_id,
    player.location,
    player.joined,
    MAX(match.day) AS lastGameDay
  FROM
    `juno-329514.gameTestingCompany.matches_info` AS match
  JOIN
    `juno-329514.gameTestingCompany.player_info` AS player
  ON
    match.player_id=player.player_id -- join on player_id
  GROUP BY
    player.player_id,
    player.location,
    player.joined),
  retention_table AS (
  SELECT
    player_id,
    location,
    joined,
    lastGameDay,
    CASE -- Add a new column with TRUE when there was retention after 30 days
      WHEN (lastGameDay - joined) > 30 THEN 1 --IF days lapsed between the day the player joined and the day they played is greater than 30 then set TRUE
    ELSE
    0 -- IF not set to FALSE
  END
    AS retention
  FROM
    player_lastGame)
SELECT
  location AS players_location,
  count(player_id) AS playersJoined,
  SUM(retention) AS playersRetained,
  (SUM(retention) / count(player_id)) AS fractionalRetention
FROM retention_table
GROUP BY players_location
ORDER BY players_location ASC

--QUERY 4, GOAL: to compute fractional retention by wining percentage
WITH
  player_lastGame AS (
  SELECT
    player.player_id,
    player.joined,
    MAX(match.day) AS lastGameDay,--last day the player played a game
    ROUND((COUNTIF (outcome = 'win') / COUNT (match_id)), 2) AS win_pct-- wins / total = winning percentage
  FROM
    `juno-329514.gameTestingCompany.matches_info` AS match
  JOIN
    `juno-329514.gameTestingCompany.player_info` AS player
  ON
    match.player_id=player.player_id -- join on player_id
  GROUP BY
    player.player_id,
    player.joined),
  retention_table AS (
  SELECT
    player_id,
    joined,
    lastGameDay,
    win_pct,
    CASE -- Sets win % range for each player
      WHEN win_pct <= 0.1 THEN "0-10%"
      WHEN win_pct <= 0.2 THEN "10-20%"
      WHEN win_pct <= 0.3 THEN "20-30%"
      WHEN win_pct <= 0.4 THEN "30-40%"
      WHEN win_pct <= 0.5 THEN "40-50%"
      WHEN win_pct <= 0.6 THEN "50-60%"
      WHEN win_pct <= 0.7 THEN "60-70%"
      WHEN win_pct <= 0.8 THEN "70-80%"
      WHEN win_pct <= 0.9 THEN "80-90%"
    ELSE
    "90-100%"
  END
    AS win_pct_range,
    CASE -- Add a new column with TRUE when there was retention after 30 days
      WHEN (lastGameDay - joined) > 30 THEN 1 --IF days lapsed between the day the player joined and the day they played is greater than 30 then set TRUE
    ELSE
    0 -- IF not set to FALSE
  END
    AS retention
  FROM
    player_lastGame)
SELECT
  win_pct_range, --winning percentage
  ROUND((SUM(retention) / COUNT(player_id)),2) AS fractionalRetention
FROM
  retention_table
GROUP BY
  win_pct_range
ORDER BY
  win_pct_range ASC
```
