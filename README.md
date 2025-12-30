# NFLPA Analytics Case Competition, Special Teams Volatility and Injury Risk, Workflow

## 1, Define the research question

### 1.1 Write it down exactly so you never drift
Do frequent and volatile special teams workloads create pace driven fatigue that increases injury risk for offensive and defensive players later in the season, and does Article 21 adequately protect players from this type of game intensity driven workload

## 2, Gather raw data

### 2.1 Play by play or gamebook data for multiple seasons
- Must include team, week, game, play type, and special teams events

### 2.2 Team injury reports by week
- Must include player, team, week, and injury status

### 2.3 Schedule and travel information
- Game dates, rest days between games, home or away, basic travel flags if possible

## 3, Build the team week panel

### 3.1 Create a table where each row is one team week
- Example key, season, week, team id

### 3.2 For each team week, attach
- Game result and stats  
- Injury outcomes  
- Rest and schedule variables  

## 4, Compute special teams workload per team week

### 4.1 From play by play, for each team week, count special teams components
- punts by that team  
- punt returns by that team  
- kickoffs by that team  
- kick returns by that team  
- field goal attempts by that team  
- extra point attempts by that team  
- rare special teams plays by that team, like free kicks, onside kicks, safety kicks, if present in the data  

### 4.2 Define component based workload totals
- ST_Load_All_w equals sum of all special teams components for that team week  
- ST_Load_ScoreLinked_w equals kickoffs plus extra points plus field goal attempts for that team week  
- ST_Load_NonScore_w equals punts plus punt returns plus kick returns plus rare special teams plays for that team week  

### 4.3 Store component columns and totals in the panel
- Add ST_Punt_w, ST_PuntReturn_w, ST_Kickoff_w, ST_KickReturn_w, ST_FG_w, ST_XP_w, ST_Rare_w  
- Add ST_Load_All_w, ST_Load_ScoreLinked_w, ST_Load_NonScore_w  

## 5, Standardize ST workload within team season and define shocks

### 5.1 For each team season, compute mean and sd for each workload bucket
- mean_ST_All, sd_ST_All using ST_Load_All_w  
- mean_ST_ScoreLinked, sd_ST_ScoreLinked using ST_Load_ScoreLinked_w  
- mean_ST_NonScore, sd_ST_NonScore using ST_Load_NonScore_w  

### 5.2 For each team week w, compute z scores for each bucket
- Z_ST_All_w equals ST_Load_All_w minus mean_ST_All then divide by sd_ST_All  
- Z_ST_ScoreLinked_w equals ST_Load_ScoreLinked_w minus mean_ST_ScoreLinked then divide by sd_ST_ScoreLinked  
- Z_ST_NonScore_w equals ST_Load_NonScore_w minus mean_ST_NonScore then divide by sd_ST_NonScore  

### 5.3 Define binary shock variables for each bucket
- ST_Shock_All_w equals 1 if Z_ST_All_w is at least 1, else 0  
- ST_Shock_ScoreLinked_w equals 1 if Z_ST_ScoreLinked_w is at least 1, else 0  
- ST_Shock_NonScore_w equals 1 if Z_ST_NonScore_w is at least 1, else 0  

### 5.4 Primary shock choice for the main causal story
- Use ST_Shock_NonScore_w as the primary special teams shock exposure  
- Keep ST_Shock_All_w and ST_Shock_ScoreLinked_w for robustness and diagnostics  

## 6, Construct season to date volatility measures

### 6.1 For each team week w, use weeks 1 through w for that team and compute volatility per bucket
- ST_Vol_All_w equals standard deviation of ST_Load_All from week 1 through w  
- ST_Vol_ScoreLinked_w equals standard deviation of ST_Load_ScoreLinked from week 1 through w  
- ST_Vol_NonScore_w equals standard deviation of ST_Load_NonScore from week 1 through w  

### 6.2 Compute cumulative shock counts per bucket
- Cum_Shocks_All_w equals sum of ST_Shock_All from week 1 through w  
- Cum_Shocks_ScoreLinked_w equals sum of ST_Shock_ScoreLinked from week 1 through w  
- Cum_Shocks_NonScore_w equals sum of ST_Shock_NonScore from week 1 through w  

### 6.3 Optional lag terms for short run dynamics
- ST_Shock_NonScore_w, ST_Shock_NonScore_w_minus_1, ST_Shock_NonScore_w_minus_2, ST_Shock_NonScore_w_minus_3  
- Optionally store the same lag structure for All and ScoreLinked if needed  

## 7, Construct cumulative workload inputs for PCA

### 7.1 For each team week w, compute season to date cumulative totals
- cum_off_snaps_w equals sum of offensive snaps from week 1 through w  
- cum_def_snaps_w equals sum of defensive snaps from week 1 through w  
- cum_ST_Load_w equals sum of ST_Load_NonScore_w from week 1 through w, this is the primary cumulative special teams input  
- cum_short_weeks_w equals number of short weeks, rest at most 4 days, up to week w  
- cum_long_travel_w equals number of long travel games up to week w, using the chosen travel indicator from schedule  

### 7.2 Optional expanded PCA inputs if needed for sensitivity
- cum_ST_ScoreLinked_w equals sum of ST_Load_ScoreLinked_w from week 1 through w  
- timezone change count or west to east count up to week w, if built from schedule  

## 8, Build the PCA based cumulative workload index

### 8.1 Standardize each cumulative variable across all team weeks
- For each variable, subtract mean and divide by standard deviation  

### 8.2 Run Principal Component Analysis on these standardized cumulative variables

### 8.3 Extract the first principal component PC1

### 8.4 Define
- Cumulative_Workload_Index_w equals PC1 for that team week  

### 8.5 Add Cumulative_Workload_Index_w to the team week panel as a control

## 9, Build injury outcomes, offense and defense

### 9.1 For each team week w, list all players on that team injury report in week w

### 9.2 For each team week w, list all players on that team injury report in week w plus 1

### 9.3 Define new injuries
- new_injured_players_w_plus_1 equals players on injury report in w plus 1 who were not on it in w  

### 9.4 Map players to positional side
- Offense, QB, RB, FB, WR, TE, OL  
- Defense, DL, LB, CB, S  

### 9.5 Count by side
- Inj_Off_Next_w equals number of new injured offensive players in w plus 1  
- Inj_Def_Next_w equals number of new injured defensive players in w plus 1  

### 9.6 Create prior health controls
- Inj_Off_Last_w equals Inj_Off_Next for week w minus 1  
- Inj_Def_Last_w equals Inj_Def_Next for week w minus 1  

## 10, Build control variables per team week

### 10.1 From play and game stats
- Offensive_snaps_w  
- Defensive_snaps_w  
- Score_diff_w, team score minus opponent score  
- Blowout_flag_w equals 1 if absolute score diff is at least 14, else 0  
- Off_yards_per_play_w  
- Points_for_w and Points_against_w as additional game script controls, because score linked special teams volume can vary even when score diff is similar  

### 10.2 From schedule
- Days_rest_w equals days between last game and current game  
- Short_week_flag_w equals 1 if Days_rest_w is at most 4  
- Bye_last_week_flag_w equals 1 if previous week was a bye  
- Home_flag_w equals 1 if game at home, else 0  

### 10.3 From injuries
- Inj_Off_Last_w  
- Inj_Def_Last_w  

## 11, Sanity check the panel

### 11.1 Confirm no duplicate team week rows

### 11.2 Confirm injury outcomes are defined only where week w plus 1 exists

### 11.3 Drop or handle final week rows where week w plus 1 is undefined

### 11.4 Confirm special teams buckets behave as intended
- ScoreLinked should move with points environment more than NonScore  
- NonScore should correlate more with punts and returns than with touchdowns  

## 12, Descriptive statistics and exploratory analysis

### 12.1 Compute summary stats
- mean and variance of Inj_Off_Next and Inj_Def_Next  
- distribution of ST_Load_All_w, ST_Load_ScoreLinked_w, ST_Load_NonScore_w  
- distribution of ST_Vol_All_w, ST_Vol_ScoreLinked_w, ST_Vol_NonScore_w  
- proportion of weeks with ST_Shock_All_w, ST_Shock_ScoreLinked_w, ST_Shock_NonScore_w equal to 1  

### 12.2 Compare means
- average Inj_Def_Next for ST_Shock_NonScore equals 1 versus 0  
- average Inj_Off_Next for ST_Shock_NonScore equals 1 versus 0  
- average injuries for low, medium, high ST_Vol_NonScore_w groups  
- diagnostic comparisons for ScoreLinked and All buckets to confirm they behave as expected  

### 12.3 Make simple plots
- bar charts of injuries by ST_Shock_NonScore status  
- bar charts of injuries by volatility terciles for ST_Vol_NonScore_w  
- line plots of injuries around shock weeks for NonScore, w minus 1, w, w plus 1  

## 13, Specify the main count outcome models

### 13.1 Because Inj_Off_Next and Inj_Def_Next are counts, choose
- Poisson regression as baseline  
- Negative Binomial regression if variance exceeds mean materially  

### 13.2 Include fixed effects
- Team fixed effects to control for permanent team traits  
- Week or season week fixed effects to control for league wide time trends  

## 14, Fit Model A, defensive new injuries

### 14.1 Outcome
- Inj_Def_Next_w  

### 14.2 Predictors, primary exposure uses NonScore measures
- ST_Shock_NonScore_w  
- ST_Vol_NonScore_w  
- Cum_Shocks_NonScore_w  
- Optional lagged shocks, ST_Shock_NonScore_w_minus_1, minus_2, minus_3  
- Offensive_snaps_w  
- Defensive_snaps_w  
- Points_for_w and Points_against_w  
- Score_diff_w  
- Blowout_flag_w  
- Short_week_flag_w  
- Bye_last_week_flag_w  
- Home_flag_w  
- Off_yards_per_play_w  
- Inj_Def_Last_w  
- Cumulative_Workload_Index_w  
- Team fixed effects  
- Week or season week fixed effects  

## 15, Fit Model B, offensive new injuries

### 15.1 Outcome
- Inj_Off_Next_w  

### 15.2 Use the same predictor structure as Model A, replacing
- Inj_Def_Last_w with Inj_Off_Last_w  

## 16, Check overdispersion and choose between Poisson and Negative Binomial

### 16.1 Compare mean and variance and use dispersion tests

### 16.2 If overdispersion is present, refit Models A and B with Negative Binomial regression

## 17, Fit logistic injury risk models

### 17.1 Define binary outcomes
- Any_Def_Injury_Next_w equals 1 if Inj_Def_Next_w is at least 1, else 0  
- Any_Off_Injury_Next_w equals 1 if Inj_Off_Next_w is at least 1, else 0  

### 17.2 Fit logistic regression for each side with the same predictors as in the count models
- Use NonScore exposures as primary, and include the same controls and fixed effects  

### 17.3 Compute predicted probabilities at different exposure levels
- ST_Shock_NonScore 0 versus 1  
- ST_Vol_NonScore low versus high  
- Cum_Shocks_NonScore few versus many  

## 18, Perform model selection using AIC and BIC

### 18.1 Fit several reasonable model variants
- with and without lagged NonScore shock terms  
- with volatility computed season to date versus rolling windows  
- with and without ScoreLinked and All measures as diagnostics  

### 18.2 For each model, record AIC and BIC

### 18.3 Choose preferred models with low criteria and interpretable structure aligned with the question

## 19, Conduct time based cross validation

### 19.1 Split data by season chronologically
- training set, earlier seasons  
- test set, later seasons  

### 19.2 Refit Models A and B on training seasons

### 19.3 Evaluate on test seasons
- predictive log likelihood or error metrics  
- stability of coefficients, especially NonScore exposure terms  

### 19.4 Confirm signs and magnitudes of key exposure effects are similar across time

## 20, Perform clustered bootstrapping for uncertainty

### 20.1 Set clustering unit as team

### 20.2 For each bootstrap iteration
- sample teams with replacement  
- retain all weeks per sampled team  
- refit main models  

### 20.3 Repeat 500 to 1000 times

### 20.4 For each key coefficient, compute
- bootstrap mean estimate  
- bootstrap confidence intervals  

### 20.5 Compare bootstrap intervals to standard intervals

## 21, Run robustness and placebo checks

### 21.1 Vary ST shock thresholds
- Z_ST_NonScore at least 0.5  
- top 25 percent of ST_Load_NonScore within team season  

### 21.2 Vary volatility definition
- ST_Vol_NonScore computed over last 4 weeks only  
- coefficient of variation for ST_Load_NonScore  

### 21.3 Alternative weighting sensitivity, optional
- define a weighted ST measure where returns and punts receive higher weight than extra points, and confirm results do not rely on the weighting choice  

### 21.4 Timing validation
- confirm stronger effects for injuries in w plus 1 than for injuries in w minus 1  

### 21.5 Placebo tests
- regress injuries at w plus 2 on NonScore exposures and verify weaker or no effects  
- regress current injuries on future NonScore exposures and verify no relationship  

## 22, Translate numerical results into football language

### 22.1 For count models, compute
- expected change in Inj_Off_Next_w and Inj_Def_Next_w for ST_Shock_NonScore changing from 0 to 1  
- expected change for ST_Vol_NonScore moving from 25th to 75th percentile  
- expected change for Cum_Shocks_NonScore increasing by one or more  

### 22.2 For logistic models, compute
- change in probability of at least one new offensive or defensive injury for different exposure levels  

### 22.3 Scale key effects to league level
- multiply per team week effects by number of team weeks in a season to estimate additional injuries attributable to NonScore special teams volatility and shocks  

## 23, Connect findings to Article 21

### 23.1 Summarize what Article 21 currently covers
- practice workload  
- meetings  
- offseason and in season structured activities  

### 23.2 Compare findings to these protections
- if NonScore volatility and shocks predict injury risk after controlling for snaps, rest, injuries, and cumulative workload, that indicates a gap in protection for in game volatility driven load  

### 23.3 Derive policy recommendations
- systematic tracking of special teams workload components and volatility, especially NonScore workload  
- recovery considerations after clusters of high NonScore shocks or high volatility stretches  
- compensation and protection review for players with heavy special teams exposure  

## 24, Prepare competition deliverables

### 24.1 Write a clear methods section describing steps 2 through 21

### 24.2 Prepare tables with main model results and robustness checks

### 24.3 Prepare figures showing descriptive patterns and effect sizes

### 24.4 Write an executive summary that states
- the question  
- the main empirical findings  
- how they relate to cumulative workload and Article 21  
- two or three concrete policy priorities for the NFLPA  
