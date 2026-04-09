---
name: event-frame-mapping
description: Instructions for mapping DFL event timestamps to TRACAB skeleton frame numbers
user-invocable: false
---

# Event-to-Frame Mapping Skill

## The Problem

Events XML uses timestamps (ISO format). Skeleton data uses frame_numbers (integers). They must be linked.

## Mapping Formula

```
frame_number = phase_N_start + round((event_seconds_since_kickoff) * framerate)
```

Where:
- `phase_N_start` comes from parquet metadata (e.g., phase_1_start for 1st half)
- `event_seconds_since_kickoff` = seconds elapsed since the kickoff of that half
- `framerate` comes from parquet metadata (typically 25)

## Step-by-Step Implementation

```python
from datetime import datetime
import xml.etree.ElementTree as ET

# 1. Get kickoff times from MatchInformations XML
match_tree = ET.parse('data/MatchInformations_*.xml')
# Find kickoff timestamps for each half — look for KickOff events or 
# use the first event timestamp in each N_GameSection

# 2. Get phase boundaries from parquet metadata
# phase_1_start, phase_1_end, phase_2_start, phase_2_end

# 3. For each event in Events XML:
def event_to_frame(event_timestamp, game_section, kickoff_times, phase_starts, framerate):
    kickoff_time = kickoff_times[game_section]  # 1 or 2
    phase_start = phase_starts[game_section]
    
    delta = (event_timestamp - kickoff_time).total_seconds()
    frame = phase_start + round(delta * framerate)
    return frame
```

## Validation

- Frame must fall within [phase_N_start, phase_N_end]
- If frame is outside bounds, the event likely occurred during stoppage (halftime, etc.)
- Cross-validate: check that ball position at computed frame matches event location

## Common Pitfalls

1. **Timezone mismatch**: Events may use UTC, check if parquet metadata specifies timezone
2. **Stoppage time**: Added time at end of half means events after 45'/90' still map to frames before phase_end
3. **Half-time gap**: Phase 2 starts at a DIFFERENT frame than phase 1 ends — there's a gap
4. **Frame rate**: Always use framerate from parquet metadata, don't assume 25Hz
