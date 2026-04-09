---
name: kpi-definition
description: Formal mathematical definition of the Phantom Run Index KPI including thresholds, scoring formula, and football reasoning
user-invocable: false
---

# Phantom Run Index — Formal Definition

## What is a Phantom Run?

A phantom run is an off-ball movement by an attacking player that simultaneously satisfies ALL four criteria below, yet the ball carrier does NOT pass to this player within the detection window.

It represents **wasted attacking potential** — a perfectly executed run that went unrewarded.

## Detection Window

A phantom run can only occur during a **no-pass window**: a continuous period where:
- A team has ball possession (from Positions XML Ballbesitz)
- No pass event occurs (from Events XML)
- Duration >= 1 second (25 frames at 25Hz)
- Ball is in play (Spielzustand = in play)

## The Four Criteria

### C1: Acceleration into Space
```
speed_kmh = ||velocity_pelvis_xy|| * 3.6  # convert m/s to km/h
C1 = speed_kmh > 15.0
```
- Velocity computed from pelvis joint (Part ID 12) displacement between consecutive frames
- Only XY plane velocity (ignore Z/height component)
- 15 km/h threshold = jogging-to-running transition in professional football

### C2: Open Passing Lane
```
ball_to_runner = runner_pelvis_xy - ball_xy
pass_distance = ||ball_to_runner||
pass_angle = atan2(ball_to_runner.y, ball_to_runner.x)

# Check cone of ±7.5° for defenders
for each opponent:
    ball_to_opp = opponent_pelvis_xy - ball_xy
    opp_angle = atan2(ball_to_opp.y, ball_to_opp.x)
    opp_distance = ||ball_to_opp||
    
    if abs(opp_angle - pass_angle) < 7.5° AND opp_distance < pass_distance:
        C2 = False  # defender blocks the lane

C2 = True AND 5.0 <= pass_distance <= 40.0  # realistic pass range in meters
```

### C3: Positional Advantage over Marker
```
nearest_opp_distance = min(||runner_pelvis - opp_pelvis|| for each opponent)
C3 = nearest_opp_distance > 1.5  # meters
```
- The runner must have > 1.5m separation from the nearest opponent
- This means the runner has genuinely "beaten" their marker

### C4: Body Orientation Commitment (3D-exclusive)
```
# Torso forward direction from skeleton
shoulder_vec = r_shoulder_xy - l_shoulder_xy
hip_vec = r_hip_xy - l_hip_xy
torso_lateral = (shoulder_vec + hip_vec) / 2
torso_forward = rotate_90_ccw(torso_lateral)  # perpendicular = facing direction
body_orientation = atan2(torso_forward.y, torso_forward.x)

# Movement direction from velocity
movement_direction = atan2(velocity_pelvis.y, velocity_pelvis.x)

alignment = abs(body_orientation - movement_direction)  # normalize to [0, 180]
C4 = alignment < 45°
```
- If body faces the same direction as movement → committed run
- If body faces sideways/backward but moving forward → defensive retreat or shuffle, not a true run
- **This criterion is IMPOSSIBLE without 3D skeletal data** — it's our key differentiator

## Phantom Run Index Score (0-100)

```
speed_score = clamp((speed_kmh - 15) / (30 - 15), 0, 1) * 25
lane_score = clamp(min_defender_cone_angle / 30, 0, 1) * 25
advantage_score = clamp((nearest_opp_dist - 1.5) / (5 - 1.5), 0, 1) * 25
commitment_score = clamp(1 - alignment / 45, 0, 1) * 25

phantom_run_index = speed_score + lane_score + advantage_score + commitment_score
```

Each sub-score contributes 0-25 points. A perfect phantom run scores 100.

## Football Reasoning

- **Speed 15-30 km/h**: Professional forwards sprint at 30-35 km/h max, but effective off-ball runs often occur at 20-25 km/h. 15 km/h eliminates walking/jogging.
- **Cone ±7.5°**: A standard football pass lane width. Wider cones are less reliable.
- **1.5m advantage**: Roughly one body length — enough for a striker to control a pass.
- **45° orientation**: Players rarely run committed with body facing more than 45° from movement direction.
