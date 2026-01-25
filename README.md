# NFL Special Teams Workload Volatility and Injury Risk

This repository studies whether week to week spikes and instability in NFL special teams workload are associated with higher injury outcomes in the following week. The project builds a 2012 to 2024 NFL team week panel from public play by play and roster data, then estimates fixed effects models to quantify the relationship between special teams workload instability and next week injuries.

## Research question

Do special teams workload shock weeks and recent workload volatility predict more next week injuries, for both offense and defense.

## What this repo does

1. Builds an NFL team week panel for seasons 2012 through 2024.
2. Constructs special teams workload features, including shocks and volatility.
3. Merges next week injury outcomes, split for offense and defense when available.
4. Produces descriptive plots that validate the patterns visually.
5. Fits fixed effects count models for injury counts and fixed effects logistic models for injury risk.
6. Exports model tables and figures for reporting.

## Data sources

This project uses public football data sources.

1. Play by play data for games and plays.
2. Roster and participation style data when available in the pipeline you use.
3. Public injury designations and reports, aggregated to team week outcomes.

Notes on data storage.

1. Raw data is often large, it is usually not committed to git.
2. This repo is set up to rebuild processed tables from raw inputs, if the raw inputs exist locally.

## Key concepts and variables

Team week structure.

1. A row represents one team in one week in one season.
2. Workload features are constructed for that team week.
3. Injury outcomes are shifted forward so they reflect next week injuries.

Workload features.

1. Team week special teams workload totals.
2. Non score linked special teams workload measures.
3. Shock indicators, unusually high special teams workload relative to a recent baseline.
4. Volatility measures, recent week to week instability in special teams workload.

Outcomes.

1. Next week injury counts.
2. Next week injury risk indicators, for example any injury in the next week.
3. Offense and defense splits when the pipeline supports it.

## Methods overview

1. Descriptive analysis using plots of workload levels, shocks, volatility, and injuries over time.
2. Fixed effects count models for injury counts.
3. Fixed effects logistic models for injury risk indicators.
4. Robustness checks using alternative windows and definitions for shocks and volatility.

Interpretation.

This analysis is observational and does not claim causal effects. The goal is to identify actionable monitoring signals, especially instability metrics that may flag elevated short term risk.

## Repository layout

Common folders.

1. data, raw and processed tables and intermediate artifacts.
2. notebooks, exploration, feature engineering, modeling, and figure generation.
3. src, reusable pipeline code and utilities.
4. outputs, exported figures, tables, and report assets.

## Requirements

1. Python 3.10 or newer is recommended.
2. A virtual environment is recommended.
3. Required packages are listed in requirements.txt.

If your pipeline uses a local database file, for example DuckDB, it will be created under data or outputs depending on your scripts.

## Setup

Create and activate a virtual environment, then install dependencies.

```bash
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
