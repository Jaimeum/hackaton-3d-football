---
description: Generate all hackathon submission deliverables — README, executive summary, github_link.txt, and validation checklist
---

# Generate Deliverables Command

Create all required submission artifacts for the AWS World Sports Innovation Cup 2026.

## Step 1: Verify Pipeline Results Exist

Check that `output/` contains results from at least one match:
- `phantom_runs.parquet`
- `phantom_runs_summary.json`
- `phantom_runs_top10.json`
- `viz/` directory with visualizations

If not found, tell the user to run `/phantom-runs` first.

## Step 2: Generate README and Submission Files

Invoke the deliverables-writer agent:

```
Agent(subagent_type="deliverables-writer", description="Write submission files", prompt="Generate all hackathon deliverables: (1) README.md with project description, reproduction instructions, and results summary. Use data from output/ for actual results. (2) executive_summary_content.md with structured content for 5 slides. (3) github_link.txt placeholder. (4) .gitignore that excludes data/, output/, *.parquet, *.xml, .env. Review the submission-requirements skill for exact specifications.")
```

## Step 3: Create Executive Summary PDF

Use the AskUserQuestion tool to ask:
- "Should I generate the executive summary as a PowerPoint/PDF now, or just provide the content for you to design?"

If yes, create a 5-slide presentation using available visualization assets.

## Step 4: Validation Checklist

Run through the checklist:
- [ ] github_link.txt exists with valid URL
- [ ] README.md has reproduction instructions
- [ ] .gitignore excludes data files
- [ ] No .parquet or .xml files are staged in git
- [ ] Executive summary is max 5 slides
- [ ] All source code is in .ipynb or .py files
- [ ] Visualizations are generated and referenced

## Step 5: Package Check

If the user wants to create the zip:
```bash
# Verify structure before zipping
echo "=== Files to submit ==="
ls -la github_link.txt executive_summary.pdf presentation_video.mp4
echo "=== Optional ==="
ls -la prfaq.pdf 2>/dev/null
```

Remind the user:
- Zip name must be team name (e.g., PhantomRuns.zip)
- Upload via the Box file request link (on Discord)
- Can resubmit with _v2 suffix if needed
