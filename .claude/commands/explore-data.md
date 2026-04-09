---
description: Explore a new match dataset — understand schema, stats, and data quality before running the pipeline
---

# Data Exploration Command

Explore and understand a Bundesliga match dataset before running the full phantom run pipeline.

## Step 1: Identify Available Data

List all data files in `data/` directory. Show the user what's available:
- Parquet files (skeleton data)
- XML files (events, positions, match info, KPIs)

## Step 2: Invoke Data Consultant

Ask the data-consultant agent to analyze the data:

```
Agent(subagent_type="data-consultant", description="Explore match data", prompt="Analyze all data files in data/. For each file: (1) Read the schema/structure, (2) Report row counts and frame ranges, (3) Check data quality (missing values, anomalies), (4) Report key metadata (framerate, pitch dimensions, teams, phase boundaries). Generate or update DATA_DICTIONARY.md with your findings. Also print a concise summary to the user.")
```

## Step 3: Quick Data Quality Check

Run a quick validation:
- Are skeleton frames continuous (no gaps)?
- Do all players have all 21 joints in most frames?
- Does ball data exist for the full match?
- Do event timestamps fall within the match time range?

## Step 4: Present Findings

Show the user:
- Match: Home vs Away (from MatchInformations)
- Duration: total frames, framerate, match length in minutes
- Players tracked: count per team
- Data quality: % frames with complete skeleton data
- Events: count by type (passes, shots, fouls, etc.)
- Recommendation: ready to proceed with `/phantom-runs` or issues to resolve first
