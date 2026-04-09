---
name: submission-requirements
description: Challenge deliverable requirements, judging criteria, and submission format for AWS World Sports Innovation Cup 2026 Challenge 2
user-invocable: false
---

# Submission Requirements — Challenge 2

## Deadline
**May 17, 2026 at 11:59 PM Pacific Time**

## Deliverables (single zip: TeamName.zip)

### 1. github_link.txt (REQUIRED)
- Plain text file with URL to GitHub repo
- Repo must contain: source code (.ipynb, .py scripts) + README.md
- README must have clear reproduction instructions
- If private repo: invite **MoellerO** as collaborator
- **DO NOT upload hackathon data to GitHub**

### 2. presentation_video.mp4 (REQUIRED)
- Max **3 minutes**
- Resolution: **< 720p** (low resolution)
- Content: demo of the tool OR explanation of the KPI
- Can use PowerPoint recording, screen recording, or direct presentation

### 3. executive_summary.pdf (REQUIRED)
- Max **5 slides** (PowerPoint exported as PDF)
- Summarizes solution and displays main output
- Note from judges: "We value creative problem-solving and learning from failures"

### 4. prfaq.pdf (OPTIONAL)
- PRFAQ (Press Release / FAQ) format
- Go into more detail about what you did and why
- Recommended for maximum score

## Judging Criteria

| Criterion | Weight | What Judges Look For |
|-----------|--------|---------------------|
| **Technical Innovation** | 34% | Novel use of 3D data, unique KPIs, creative approaches |
| **Implementation Quality** | 33% | Clean code, reproducibility, proper use of AWS |
| **Market Impact** | 33% | Real-world applicability for fans, coaches, broadcasters |

10 judges evaluate from May 18 to May 29, 2026. Winners announced by June 24, 2026.

## Our Strategy Per Criterion

### Technical Innovation (34%)
- Body orientation from 3D skeleton = impossible with 2D tracking
- Phantom Run Index is a novel KPI not in current Bundesliga Match Facts
- Multi-criteria detection combining spatial analysis with biomechanical data

### Implementation Quality (33%)
- Polars pipeline for 141M data points per match
- Chunk-based parquet reading for memory efficiency
- Clear notebook with step-by-step execution
- Dockerized or SageMaker-reproducible

### Market Impact (33%)
- **Coaches**: "Player X made 12 phantom runs, received 0 passes → tactical adjustment needed"
- **Broadcasters**: "The pass that never came" as a replay graphic with body orientation overlay
- **Fans**: Interactive "what-if" — show what would have happened if the pass was made

## .gitignore Must Include
```
data/
*.parquet
*.xml
output/
.env
```
