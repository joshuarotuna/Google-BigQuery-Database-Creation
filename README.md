# Google-BigQuery-Database-Creation
Created a lightweight but optimal database in Google BigQuery to house historical NFL Analytics (personal league) data.

## NFL Pick Em League — BigQuery Database
A relational data warehouse built in Google BigQuery to store, analyze, and project results for an annual NFL Pick Em league. Players select one NFL team per week to win — no team can be picked more than once per season. This league has been running for over 5 years now and this database aims to serve as a storage for historical data – to allow for accessibility and creative analysis as a fun and meaningful personal project. This database powers live standings, historical analysis, and a Monte Carlo win probability model.

 
## Project Structure
GCP Project:  nfl-pick-em-493506
Dataset:      nfl_pickem
## Tables (5)
Table	Description
seasons	League metadata per year — buy-in, prize payouts, champion

players	One row per player per season — display name, nickname

picks	Core fact table — one row per player per week which shows each week’s pick

weekly_results	Winning NFL teams per week — source of truth for win/loss

projections	Monte Carlo simulation snapshots — win probabilities by week
### Views (3)
View	Description

v_leaderboard	Live season standings with wins and win % — used in Looker Studio

v_projections_latest	Most recent projections snapshot

v_projections_compare	Side-by-side probability comparison between any two weeks

 
### Schema
### seasons

Stores league-level metadata for each year.
Column	Type	Description
season_year	INT64	Primary key — e.g. 2025
buy_in	FLOAT64	Entry fee per player — e.g. 50.00
prize_1st	FLOAT64	First place payout — e.g. 1050.00
prize_2nd	FLOAT64	Second place payout — e.g. 300.00
prize_3rd	FLOAT64	Third place payout — e.g. 150.00
champion_name	STRING	Display name of season winner
notes	STRING	Optional free text
 
### Players
One row per player per season. A returning player gets a new row each year, linked by display_name across seasons. This table will continue to be maintained as new players enter the league, nicknames change, or any other adjustments are needed.
Column	Type	Description
player_id	STRING	Primary key — e.g. josh_commish_2025
display_name	STRING	Human-readable name — e.g. Josh Commish
nickname	STRING	Optional nickname — e.g. The Librarian
season_year	INT64	Foreign key → seasons
 
### Picks
Core fact table. One row per player per week which stores the pick each player made each week. Does not store win/loss — result is always derived at query time by joining to weekly_results. This keeps the model normalized and ensures win % is always calculated from live data.
Column	Type	Description
pick_id	STRING	Primary key — e.g. josh_commish_2025_w01
player_id	STRING	Foreign key → players
season_year	INT64	Foreign key → seasons
week_number	INT64	NFL week (1–18)
team_picked	STRING	Team nickname — e.g. Eagles
 
### weekly_results
The winning team from every completed NFL game, loaded automatically from the ESPN API each week. One row per winning team per week. This is what picks join against to determine win/loss.
Column	Type	Description
season_year	INT64	Composite key
week_number	INT64	Composite key
team_won	STRING	Winning team nickname — e.g. Chiefs
 
### projections
Stores Monte Carlo simulation snapshots. Each run produces one row per player tagged with as_of_week, allowing historical comparison of how win probabilities evolved throughout the season.
Column	Type	Description
season_year	INT64	Season year
as_of_week	INT64	Snapshot week — e.g. 8, 10, 18
player_id	STRING	Foreign key → players
display_name	STRING	Human-readable name
wins_to_date	INT64	Actual wins through as_of_week
expected_future_wins	FLOAT64	Sum of ESPN win % for remaining picks
projected_total	FLOAT64	wins_to_date + expected_future_wins
avg_team_win_pct	FLOAT64	Tiebreaker — avg NFL win % of all picks (lower = better)
win_probability	FLOAT64	% chance to finish 1st from Monte Carlo
weeks_remaining	INT64	Weeks left after as_of_week
unsubmitted_weeks	INT64	Picks not yet entered by player
simulations_run	INT64	Number of Monte Carlo iterations
created_at	TIMESTAMP	When snapshot was generated
 
### Key Design Decisions
Tiebreaker logic Per league rules, the tiebreaker rewards players who pick lower-ranked NFL teams (based on the team’s win percentage which fluctuates throughout the season). This methodology is applied in an effort to reward to “underdog” selections which are typically more difficult and risky. A player with a lower average NFL win % across all their picks wins the tiebreaker at equal wins. This is reflected in all queries and in the Monte Carlo sort.
Player identity across seasons player_id is scoped per season (e.g. josh_commish_2025). To link a player across multiple seasons, join on display_name. This allows nicknames and display names to change year to year without breaking historical data.
Projections are snapshots, not overwritten Each weekly run of the Monte Carlo simulation appends a new set of rows tagged with as_of_week. This preserves the full history of how probabilities evolved, enabling week-over-week comparison via v_projections_compare.
 
### Views
v_leaderboard
Joins picks + players + weekly_results to produce a live season leaderboard. Recalculates automatically whenever weekly_results is updated. Used as the primary data source in Looker Studio.
SELECT display_name, nickname, total_picks, wins, win_pct
FROM `nfl-pick-em-493506.nfl_pickem.v_leaderboard`
v_projections_latest
Always returns the most recent projection snapshot regardless of which week was last loaded.
SELECT * FROM `nfl-pick-em-493506.nfl_pickem.v_projections_latest`
ORDER BY win_probability DESC
v_projections_compare
Compares win probabilities between two snapshots. Edit the as_of_week values in the view definition to compare any two weeks.
-- Edit these two values in the view definition:
AND a.as_of_week = 8    -- from week
AND b.as_of_week = 11   -- to week
 
### ETL Pipeline
All data loading is handled via Google Colab notebooks using the BigQuery Python client with load_table_from_dataframe (batch load — compatible with the free Sandbox tier). 
It is currently loaded via manual selection of historical Excel files. Efforts will be evaluated to automate this further.

Refer to nfl_pickem SQL Load Scripts.ipynb
 
### Data Sources
Source	Used For
Excel (2025_NFL_Pick_Em.xlsx)	Player picks — sheet: Picks, header row 4
ESPN API — scoreboard endpoint	Weekly game results (winning teams)
ESPN API — standings endpoint	Current NFL team win % for projections
ESPN API endpoints:
### Weekly results
https://site.api.espn.com/apis/site/v2/sports/football/nfl/scoreboard
  ?dates={YEAR}&seasontype=2&week={WEEK}

### Team standings
https://site.api.espn.com/apis/site/v2/sports/football/nfl/standings
ESPN's API is unofficial and undocumented. No authentication required but endpoints may change without notice.
 
### Monte Carlo Simulation
Win probabilities in the projections table are generated by simulating 50,000 complete seasons forward from the snapshot week. For each simulation:
•	Completed weeks use actual results from weekly_results
•	Future picks are simulated using each team's current ESPN win % as a weighted coin flip
•	Unsubmitted weeks use a 0.450 fallback probability
•	Season winner determined by total wins, tiebroken by lower average team win %
•	Win probability = times a player finished 1st across all 50,000 simulations
See nfl_pickem_projections.ipynb for full implementation.


Refer to nfl_pickem_projections.ipynb file

 
### Cost
Runs entirely within BigQuery's free Sandbox tier.
Resource	Usage	Free Tier Limit
Storage	~0.001 GB	10 GB / month
Query processing	~0.001 TB	1 TB / month
Estimated monthly cost	$0.00	—
 
### Seasons Covered
Season	Players	Status
2025	30	Loaded
 
### Tools
Tool	Purpose
Google BigQuery	Data warehouse
Google Colab	ETL and simulation notebooks
Looker Studio	Live dashboard
ESPN unofficial API	Automated game results and standings
Python — pandas, pyarrow	Data processing and BigQuery loading



