—— Picks Table

CREATE TABLE `nfl-pick-em-493506.nfl_pickem.picks`
(
  pick_id STRING NOT NULL,
  player_id STRING NOT NULL,
  season_year INT64 NOT NULL,
  week_number INT64 NOT NULL,
  team_picked STRING,
  PRIMARY KEY (pick_id) NOT ENFORCED,
  FOREIGN KEY (player_id) REFERENCES `nfl-pick-em-493506.nfl_pickem.players`(player_id) NOT ENFORCED,
  FOREIGN KEY (season_year) REFERENCES `nfl-pick-em-493506.nfl_pickem.seasons`(season_year) NOT ENFORCED
)
OPTIONS(
  expiration_timestamp=TIMESTAMP "2026-06-15T15:09:54.210Z"
);

—— Seasons Table

	CREATE TABLE `nfl-pick-em-493506.nfl_pickem.seasons`
(
  season_year INT64 NOT NULL,
  buy_in FLOAT64,
  prize_1st FLOAT64,
  prize_2nd FLOAT64,
  prize_3rd FLOAT64,
  champion_name STRING,
  notes STRING,
  PRIMARY KEY (season_year) NOT ENFORCED
)
OPTIONS(
  expiration_timestamp=TIMESTAMP "2026-06-15T06:24:46.609Z"
);


—— Weekly_Results Table

	CREATE TABLE `nfl-pick-em-493506.nfl_pickem.weekly_results`
(
  season_year INT64,
  week_number INT64,
  team_won STRING
)
OPTIONS(
  expiration_timestamp=TIMESTAMP "2026-06-16T04:52:31.540Z"
);

—— Players Table

	CREATE TABLE `nfl-pick-em-493506.nfl_pickem.players`
(
  player_id STRING NOT NULL,
  display_name STRING NOT NULL,
  nickname STRING,
  season_year INT64 NOT NULL,
  PRIMARY KEY (player_id) NOT ENFORCED
)
OPTIONS(
  expiration_timestamp=TIMESTAMP "2026-06-15T15:07:53.538Z"
);

—— Projections Table

	CREATE TABLE `nfl-pick-em-493506.nfl_pickem.projections`
(
  season_year INT64 NOT NULL,
  as_of_week INT64 NOT NULL,
  player_id STRING NOT NULL,
  display_name STRING,
  wins_to_date INT64,
  expected_future_wins FLOAT64,
  projected_total FLOAT64,
  avg_team_win_pct FLOAT64,
  win_probability FLOAT64,
  weeks_remaining INT64,
  unsubmitted_weeks INT64,
  simulations_run INT64,
  created_at TIMESTAMP
)
OPTIONS(
  expiration_timestamp=TIMESTAMP "2026-06-16T06:38:04.187Z"
);
