---
description: Main orchestrator — runs the full Phantom Run detection pipeline for a match
---

# Phantom Runs Pipeline Orchestrator

Execute the full Phantom Run Index pipeline for a Bundesliga match.

## Step 1: Select Match Data

Use the AskUserQuestion tool to ask:
- "Which match data should I analyze? List the parquet files in data/ directory."
- Show available parquet files found in `data/`

## Step 2: Data Exploration (if DATA_DICTIONARY.md doesn't exist)

Invoke the data-consultant agent to generate the data dictionary:

```
Agent(subagent_type="data-consultant", description="Generate data dictionary", prompt="Analyze all data files in data/ for the selected match. Read the parquet schema, XML structures, and generate DATA_DICTIONARY.md in the project root. Include schema details, column descriptions, units, and how to join skeleton frames with events.")
```

If DATA_DICTIONARY.md already exists, skip this step.

## Step 3: Process Skeleton Data

Invoke the skeleton-processor agent:

```
Agent(subagent_type="skeleton-processor", description="Process skeleton parquet", prompt="Read the parquet file data/{match_file} and process it. Extract joint positions, compute pelvis velocity vectors, derive body orientation from shoulder/hip joints. Save outputs to output/{match_id}/skeleton_flat.parquet, player_kinematics.parquet, and ball_trajectory.parquet. Process in chunks to manage memory.")
```

## Step 4: Link Events to Frames

Invoke the event-linker agent:

```
Agent(subagent_type="event-linker", description="Link events to frames", prompt="Parse Events XML and Positions XML for the selected match. Map event timestamps to frame numbers. Identify no-pass windows (possession periods without a pass). Save to output/{match_id}/events_with_frames.parquet, pass_events.parquet, and no_pass_windows.parquet.")
```

## Step 5: Detect Phantom Runs

Invoke the phantom-run-detector agent:

```
Agent(subagent_type="phantom-run-detector", description="Detect phantom runs", prompt="Using the processed data in output/{match_id}/, apply the 4-criteria phantom run detection algorithm. Score each detected run with the Phantom Run Index (0-100). Save results to output/{match_id}/phantom_runs.parquet, phantom_runs_summary.json, and phantom_runs_top10.json.")
```

## Step 6: Generate Visualizations

Invoke the viz-builder agent:

```
Agent(subagent_type="viz-builder", description="Create visualizations", prompt="Using phantom run results in output/{match_id}/, create 2D pitch replay visualizations for the top 5 phantom runs. Also create a summary dashboard. Save all images to output/{match_id}/viz/.")
```

## Step 7: Summary

Present the results to the user:
- Total phantom runs detected
- Top 3 phantom runs by index score (player, minute, score)
- Link to visualization files
- Suggested next steps (run on more matches, generate deliverables)
