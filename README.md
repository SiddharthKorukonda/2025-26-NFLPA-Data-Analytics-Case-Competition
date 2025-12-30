Define the research question
 1.1 Write it down exactly so you never drift:
 “Do frequent and volatile special teams workloads create pace-driven fatigue that increases injury risk for offensive and defensive players later in the season, and does Article 21 adequately protect players from this type of game-intensity-driven workload”


Gather raw data
 2.1 Play-by-play or gamebook data for multiple seasons
 – Must include team, week, game, play type, and special teams events
 2.2 Team injury reports by week
 – Must include player, team, week, and injury status
 2.3 Schedule and travel information
 – Game dates, rest days between games, home or away, basic travel flags if possible


Build the team–week panel
 3.1 Create a table where each row is one team–week
 – Example key: season, week, team_id
 3.2 For each team–week, attach
 – Game result and stats
 – Injury outcomes
 – Rest and schedule variables


Compute special teams workload per team–week
 4.1 From play-by-play, for each team–week, count
 – punts by that team
 – punt returns by that team
 – kickoffs by that team
 – kick returns by that team
 – field goal attempts by that team
 – extra point attempts by that team
 4.2 Define
 – ST_Load_w = sum of all these events for that team–week


Standardize ST workload within team–season and define shocks
 5.1 For each team–season, compute
 – mean_ST = mean of ST_Load_w across all weeks for that team that season
 – sd_ST = standard deviation of ST_Load_w for that team that season
 5.2 For each team–week w, compute
 – Z_ST_w = (ST_Load_w − mean_ST) / sd_ST
 5.3 Define a binary shock variable
 – ST_Shock_w = 1 if Z_ST_w ≥ 1, otherwise 0


Construct season-to-date volatility measures
 6.1 For each team–week w, take all ST_Load values from weeks 1 through w for that team
 6.2 Compute
 – ST_Vol_w = standard deviation of ST_Load from weeks 1 through w
 6.3 Compute cumulative shock count
 – Cum_Shocks_w = sum of ST_Shock from weeks 1 through w
 6.4 (Optional but allowed in models) store recent shock lags
 – ST_Shock_w
 – ST_Shock_w_minus_1
 – ST_Shock_w_minus_2
 – ST_Shock_w_minus_3


Construct cumulative workload inputs for PCA
 7.1 For each team–week w, compute season-to-date cumulative totals
 – cum_off_snaps_w = sum of offensive snaps from week 1 through w
 – cum_def_snaps_w = sum of defensive snaps from week 1 through w
 – cum_ST_Load_w = sum of ST_Load from week 1 through w
 – cum_short_weeks_w = number of short weeks (rest ≤ 4 days) up to week w
 – cum_long_travel_w = number of long-travel games up to week w (define a simple travel indicator using schedule)


Build the PCA-based cumulative workload index
 8.1 Standardize each cumulative variable across all team–weeks
 – For each variable, subtract mean and divide by standard deviation
 8.2 Run Principal Component Analysis on these standardized cumulative variables
 8.3 Extract the first principal component PC1
 8.4 Define
 – Cumulative_Workload_Index_w = PC1 for that team–week
 8.5 Add Cumulative_Workload_Index_w to the team–week panel as a control


Build injury outcomes (offense and defense)
 9.1 For each team–week w, list all players on that team’s injury report in week w
 9.2 For each team–week w, list all players on that team’s injury report in week w+1
 9.3 Define new injuries
 – new_injured_players_w_plus_1 = players on injury report in week w+1 who were not on it in week w
 9.4 Map players to positional side
 – Offense: QB, RB, FB, WR, TE, OL
 – Defense: DL, LB, CB, S
 9.5 Count by side
 – Inj_Off_Next_w = number of new injured offensive players in w+1
 – Inj_Def_Next_w = number of new injured defensive players in w+1
 9.6 Create prior health controls
 – Inj_Off_Last_w = Inj_Off_Next for week w−1
 – Inj_Def_Last_w = Inj_Def_Next for week w−1


Build control variables per team–week
 10.1 From play and game stats
 – Offensive_snaps_w
 – Defensive_snaps_w
 – Score_diff_w (team score minus opponent score)
 – Blowout_flag_w = 1 if |Score_diff_w| ≥ 14, else 0
 – Off_yards_per_play_w
 10.2 From schedule
 – Days_rest_w = days between last game and current game
 – Short_week_flag_w = 1 if Days_rest_w ≤ 4
 – Bye_last_week_flag_w = 1 if previous week was a bye
 – Home_flag_w = 1 if game at home, 0 if away
 10.3 From injuries
 – Inj_Off_Last_w
 – Inj_Def_Last_w


Sanity check the panel
 11.1 Confirm no duplicate team–week rows
 11.2 Confirm injury outcomes are defined only where week w+1 exists
 11.3 Drop or handle final-week rows where w+1 is undefined


Descriptive statistics and exploratory analysis
 12.1 Compute summary stats
 – mean and variance of Inj_Off_Next and Inj_Def_Next
 – distribution of ST_Load_w
 – distribution of ST_Vol_w
 – proportion of weeks with ST_Shock_w = 1
 12.2 Compare means
 – average Inj_Def_Next for ST_Shock = 1 versus ST_Shock = 0
 – average Inj_Off_Next for ST_Shock = 1 versus ST_Shock = 0
 – average injuries for low, medium, high ST_Vol_w groups
 12.3 Make simple plots
 – bar charts of injuries by ST_Shock status
 – bar charts of injuries by volatility terciles
 – line plots of injuries around shock weeks (w−1, w, w+1)


Specify the main count outcome models
 13.1 Because Inj_Off_Next and Inj_Def_Next are counts, choose
 – Poisson regression as baseline
 – Negative Binomial regression if you detect overdispersion (variance >> mean)
 13.2 Include fixed effects
 – Team fixed effects to control for permanent team traits
 – Week or season-week fixed effects to control for league-wide time trends


Fit Model A: defensive new injuries
 14.1 Outcome
 – Inj_Def_Next_w
 14.2 Predictors
 – ST_Shock_w
 – ST_Vol_w
 – Cum_Shocks_w
 – Optional lagged shocks ST_Shock_w−1, ST_Shock_w−2, ST_Shock_w−3
 – Offensive_snaps_w
 – Defensive_snaps_w
 – Score_diff_w
 – Blowout_flag_w
 – Short_week_flag_w
 – Bye_last_week_flag_w
 – Home_flag_w
 – Off_yards_per_play_w
 – Inj_Def_Last_w
 – Cumulative_Workload_Index_w
 – Team fixed effects
 – Week or season-week fixed effects


Fit Model B: offensive new injuries
 15.1 Outcome
 – Inj_Off_Next_w
 15.2 Use the same predictor structure as Model A, replacing
 – Inj_Def_Last_w with Inj_Off_Last_w


Check overdispersion and choose between Poisson and Negative Binomial
 16.1 Compare mean and variance of residuals or use dispersion tests
 16.2 If clear overdispersion is present, refit Models A and B with Negative Binomial regression


Fit logistic injury risk models
 17.1 Define binary outcomes
 – Any_Def_Injury_Next_w = 1 if Inj_Def_Next_w ≥ 1, else 0
 – Any_Off_Injury_Next_w = 1 if Inj_Off_Next_w ≥ 1, else 0
 17.2 Fit logistic regression for each side with the same predictors as in the count models
 – ST_Shock_w
 – ST_Vol_w
 – Cum_Shocks_w
 – optional lagged shocks
 – all control variables
 – Cumulative_Workload_Index_w
 – Team fixed effects
 – Week or season-week fixed effects
 17.3 From these models, compute predicted probabilities at different values of
 – ST_Shock_w
 – ST_Vol_w (low vs high)
 – Cum_Shocks_w (few vs many shocks)


Perform model selection using AIC and BIC
 18.1 Fit several reasonable model variants
 – with and without lagged ST_Shock terms
 – with ST_Vol_w versus alternative volatility windows
 18.2 For each model, record AIC and BIC
 18.3 Choose preferred models that have
 – low AIC and BIC
 – a clear, interpretable structure aligned with the research question


Conduct time-based cross-validation
 19.1 Split data by season or blocks of seasons
 – training set: earlier seasons
 – test set: later seasons
 19.2 Refit Models A and B on training seasons
 19.3 Evaluate on test seasons
 – predictive log-likelihood or error metrics
 – stability of coefficients, especially ST_Shock_w, ST_Vol_w, Cum_Shocks_w
 19.4 Confirm that signs and magnitudes of key exposure effects are similar across time


Perform clustered bootstrapping for uncertainty
 20.1 Set the clustering unit as team
 20.2 For each bootstrap iteration
 – sample teams with replacement
 – take all weeks for each sampled team
 – refit the main models
 20.3 Repeat 500 to 1000 times
 20.4 For each key coefficient (ST_Shock_w, ST_Vol_w, Cum_Shocks_w), compute
 – bootstrap mean estimate
 – bootstrap confidence intervals
 20.5 Compare bootstrap intervals to standard model confidence intervals to check robustness


Run robustness and placebo checks
 21.1 Vary ST_Shock thresholds
 – Z_ST_w ≥ 0.5
 – top 25 percent of ST_Load_w within team–season
 21.2 Vary the volatility definition
 – ST_Vol_w computed over last 4 weeks only
 – coefficient of variation instead of plain standard deviation
 21.3 Timing validation
 – confirm models show stronger effects for injuries in w+1 than for injuries in w−1
 21.4 Placebo tests
 – regress injuries at w+2 on ST_Shock_w, ST_Vol_w, Cum_Shocks_w and verify weaker or no effects
 – regress current injuries on future ST variables and verify no systematic relationship


Translate numerical results into football language
 22.1 For count models, compute
 – expected change in Inj_Off_Next_w and Inj_Def_Next_w for
 · ST_Shock_w changing from 0 to 1
 · ST_Vol_w moving from low to high (e.g. 25th to 75th percentile)
 · Cum_Shocks_w increasing by one or more units
 22.2 For logistic models, compute
 – change in probability of at least one new offensive or defensive injury for different exposure levels
 22.3 Scale key effects to league level
 – multiply per team–week effects by number of team–weeks in a full season to estimate additional injuries attributable to ST workload volatility and shocks


Connect findings to Article 21
 23.1 Summarize what Article 21 currently covers
 – practice workload
 – meetings
 – offseason and in-season structured activities
 23.2 Compare your empirical findings to these protections
 – if ST_Shock_w, ST_Vol_w, and Cum_Shocks_w are significant predictors of injury risk even after adjusting for snaps, rest, prior injuries, and cumulative workload index, then there is a gap in protection for in-game, volatility-driven load
 23.3 Derive policy recommendations
 – systematic tracking of special teams workload and volatility
 – consideration of recovery rules after clusters of high ST shocks or high volatility stretches
 – compensation and protection review for players with heavy special teams exposure


Prepare competition deliverables
 24.1 Write a clear methods section describing steps 2 through 21
 24.2 Prepare tables with main model results and robustness checks
 24.3 Prepare figures showing descriptive patterns and effect sizes
 24.4 Write an executive summary that states
 – the question
 – the main empirical findings
 – how they relate to cumulative workload and Article 21
 – two or three concrete policy priorities for the NFLPA


