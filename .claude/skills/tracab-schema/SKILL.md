---
name: tracab-schema
description: Complete schema reference for TRACAB 3D skeleton parquet format including coordinate system, joint IDs, and nested structure
user-invocable: false
---

# TRACAB Skeleton Schema Reference

## Parquet Nested Structure

The skeleton parquet file is a **nested** format. You MUST use pyarrow to read it, not polars directly.

### Level 1: Metadata Header (file-level)
| Field | Type | Description |
|-------|------|-------------|
| file_version | int | Format version |
| vendor_id | int | Vendor identifier |
| game_id | int | Match identifier |
| data_quality | int | 0=raw, 1=filtered/post-processed |
| framerate | int | Frames per second (typically 25) |
| ai_clicker | int | 0=no AI clicker, 1=AI clicker used |
| phase_1_start | int | First half start frame |
| phase_1_end | int | First half end frame |
| phase_2_start | int | Second half start frame |
| phase_2_end | int | Second half end frame |
| phase_3_start/end | int | 1st extra time (if applicable) |
| phase_4_start/end | int | 2nd extra time (if applicable) |
| phase_5_start/end | int | Penalty shootout (if applicable) |
| pitch_long | int | Pitch length in decimeters (dm) |
| pitch_short | int | Pitch width in decimeters (dm) |
| pitch_padding_* | int | Tracking area padding in dm |
| home_team_id | int | Home team identifier |
| away_team_id | int | Away team identifier |

### Level 2: Skeleton Frame (per frame)
| Field | Type | Description |
|-------|------|-------------|
| version | int | Frame format version (currently 1) |
| frame_number | int | Unique frame counter |
| type | int | 0=raw, 1=filtered |
| skeleton_count | int | Number of tracked players/refs in frame |
| ball_exists | int | 0=no ball, 1=ball tracked |
| skeletons | array | Array of Skeleton Targets |
| ball | object | Ball Target (absent if ball_exists=0) |

### Level 3: Skeleton Target (per player)
| Field | Type | Description |
|-------|------|-------------|
| jersey_number | int | Jersey #. -1=unassigned, 0=head ref, 1=1st asst, 2=2nd asst |
| team | int | **1=Home, 0=Away, 3=Referee** |
| parts_count | int | Number of tracked joints (max 21) |
| parts | array | Array of Skeleton Target Parts |

### Level 4: Skeleton Target Part (per joint)
| Field | Type | Description |
|-------|------|-------------|
| name | int | Part ID (see mapping below) |
| position_x | float | X position in **centimeters** from pitch center |
| position_y | float | Y position in **centimeters** from pitch center |
| position_z | float | Z position in **centimeters** (height above ground) |

### Ball Target
| Field | Type | Description |
|-------|------|-------------|
| position_x/y/z | float | Ball position in **centimeters** from pitch center |
| velocity_x/y/z | float | Ball velocity in **m/s** |

## Part ID Mapping (21 joints)

| ID | Name | ID | Name |
|----|------|----|------|
| 1 | Left ear | 12 | Pelvis |
| 2 | Nose | 13 | Right hip |
| 3 | Right ear | 14 | Left knee |
| 4 | Left shoulder | 15 | Right knee |
| 5 | Neck | 16 | Left ankle |
| 6 | Right shoulder | 17 | Right ankle |
| 7 | Left elbow | 18 | Left heel |
| 8 | Right elbow | 19 | Left toe |
| 9 | Left wrist | 20 | Right heel |
| 10 | Right wrist | 21 | Right toe |
| 11 | Left hip | | |

## Coordinate System

- Origin (0,0,0) at **pitch center**
- X-axis: along pitch length. Positive X = right side from broadcast view
- Y-axis: along pitch width. Positive Y = top from broadcast view
- Z-axis: vertical. Positive Z = up
- All skeleton positions in **centimeters**
- Pitch dimensions from metadata are in **decimeters**

## Key Joints for Phantom Run Detection

- **Pelvis (12)**: Primary position reference (center of mass proxy)
- **L/R Shoulder (4,6)**: Torso width vector for body orientation
- **L/R Hip (11,13)**: Lower torso vector for body orientation
- **Nose (2)**: Head direction (potential scanning analysis)
- **L/R Ear (1,3)**: Head rotation detection
