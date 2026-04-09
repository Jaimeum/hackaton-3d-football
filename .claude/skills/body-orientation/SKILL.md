---
name: body-orientation
description: How to compute torso facing direction from 3D skeleton shoulder and hip joint positions
user-invocable: false
---

# Body Orientation Computation

## Why This Matters

Body orientation tells us WHERE a player's torso is facing, independent of their movement direction. This is the **key differentiator** that makes 3D skeleton data essential for phantom run detection.

A player can move forward while their torso faces sideways (defensive sliding) or backward (backpedaling). Only when body orientation ALIGNS with movement direction can we confirm a committed attacking run.

## Required Joints

| Joint | Part ID | Role |
|-------|---------|------|
| Left shoulder | 4 | Torso width (left) |
| Right shoulder | 6 | Torso width (right) |
| Left hip | 11 | Lower torso width (left) |
| Right hip | 13 | Lower torso width (right) |
| Pelvis | 12 | Position reference |

## Computation Algorithm

```python
import numpy as np

def compute_body_orientation(l_shoulder, r_shoulder, l_hip, r_hip):
    """
    Compute torso facing direction from 4 skeleton joints.
    All inputs are [x, y] arrays in the pitch plane (ignore z).
    Returns angle in degrees [0, 360) where 0 = positive X axis.
    """
    # Shoulder lateral vector (left to right)
    shoulder_vec = np.array(r_shoulder[:2]) - np.array(l_shoulder[:2])
    
    # Hip lateral vector (left to right)
    hip_vec = np.array(r_hip[:2]) - np.array(l_hip[:2])
    
    # Average lateral vector (more robust than using just shoulders)
    lateral = (shoulder_vec + hip_vec) / 2.0
    
    # Forward direction = 90° counter-clockwise rotation of lateral vector
    # If lateral points from left to right, forward points "ahead" of the torso
    forward = np.array([-lateral[1], lateral[0]])
    
    # Normalize
    norm = np.linalg.norm(forward)
    if norm < 1e-6:
        return np.nan  # joints too close, can't determine orientation
    
    # Angle relative to X-axis
    angle_rad = np.arctan2(forward[1], forward[0])
    angle_deg = np.degrees(angle_rad) % 360
    
    return angle_deg


def compute_orientation_alignment(body_orientation_deg, movement_direction_deg):
    """
    Compute angular difference between body facing and movement direction.
    Returns value in [0, 180] degrees.
    0 = perfectly aligned (committed run)
    180 = facing opposite to movement (backpedaling)
    """
    diff = abs(body_orientation_deg - movement_direction_deg)
    if diff > 180:
        diff = 360 - diff
    return diff
```

## Handling Missing Joints

- If any of the 4 joints (l_shoulder, r_shoulder, l_hip, r_hip) has missing data → return NaN
- Missing joints appear as empty values in the parquet
- Typically < 5% of frames have missing shoulder/hip data
- For missing frames, interpolate from neighboring frames (linear) if gap < 5 frames

## Validation

To verify the computation is correct:
1. When a player runs straight toward a goal, body_orientation should ≈ movement_direction
2. When a player does a sidestep/crab walk, body_orientation should be ~90° from movement
3. When a goalkeeper faces the ball but moves laterally, orientation ≠ movement direction

## Performance Note

- Compute body orientation only for outfield players (skip referees: team=3, skip GKs)
- Compute only during no-pass windows (saves ~80% of computation)
- Vectorize with numpy — avoid per-frame Python loops where possible
