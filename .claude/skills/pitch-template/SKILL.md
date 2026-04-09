---
name: pitch-template
description: Template and style guide for drawing 2D football pitch visualizations with matplotlib
user-invocable: false
---

# Pitch Template Skill

## Standard Pitch Drawing (matplotlib)

```python
import matplotlib.pyplot as plt
import matplotlib.patches as patches
import numpy as np

def draw_pitch(ax=None, pitch_length=105, pitch_width=68, color='#1a472a', line_color='white'):
    """Draw a standard football pitch on a matplotlib axes."""
    if ax is None:
        fig, ax = plt.subplots(figsize=(12, 8))
        fig.patch.set_facecolor('#0d1117')
    
    ax.set_facecolor(color)
    ax.set_xlim(-pitch_length/2 - 5, pitch_length/2 + 5)
    ax.set_ylim(-pitch_width/2 - 5, pitch_width/2 + 5)
    ax.set_aspect('equal')
    ax.axis('off')
    
    lw = 1.5  # line width
    
    # Pitch outline
    ax.plot([-pitch_length/2, pitch_length/2], [-pitch_width/2, -pitch_width/2], color=line_color, lw=lw)
    ax.plot([-pitch_length/2, pitch_length/2], [pitch_width/2, pitch_width/2], color=line_color, lw=lw)
    ax.plot([-pitch_length/2, -pitch_length/2], [-pitch_width/2, pitch_width/2], color=line_color, lw=lw)
    ax.plot([pitch_length/2, pitch_length/2], [-pitch_width/2, pitch_width/2], color=line_color, lw=lw)
    
    # Halfway line
    ax.plot([0, 0], [-pitch_width/2, pitch_width/2], color=line_color, lw=lw)
    
    # Center circle
    center_circle = plt.Circle((0, 0), 9.15, fill=False, color=line_color, lw=lw)
    ax.add_patch(center_circle)
    ax.plot(0, 0, 'o', color=line_color, markersize=3)
    
    # Penalty areas (16.5m from goal line, 40.3m wide)
    for sign in [-1, 1]:
        x = sign * pitch_length/2
        # Penalty box
        ax.plot([x, x - sign*16.5], [-20.16, -20.16], color=line_color, lw=lw)
        ax.plot([x, x - sign*16.5], [20.16, 20.16], color=line_color, lw=lw)
        ax.plot([x - sign*16.5, x - sign*16.5], [-20.16, 20.16], color=line_color, lw=lw)
        # Goal box (5.5m from goal line, 18.3m wide)
        ax.plot([x, x - sign*5.5], [-9.16, -9.16], color=line_color, lw=lw)
        ax.plot([x, x - sign*5.5], [9.16, 9.16], color=line_color, lw=lw)
        ax.plot([x - sign*5.5, x - sign*5.5], [-9.16, 9.16], color=line_color, lw=lw)
        # Penalty spot
        ax.plot(x - sign*11, 0, 'o', color=line_color, markersize=3)
        # Goal
        ax.plot([x, x], [-3.66, 3.66], color='#ffffff', lw=3)
    
    return ax
```

## Phantom Run Visualization Style

```python
# Color palette
COLORS = {
    'home': '#3498db',       # Blue
    'away': '#e74c3c',       # Red
    'ball': '#ffffff',       # White
    'run_path': '#f1c40f',   # Gold
    'pass_lane': '#2ecc71',  # Green (20% opacity)
    'defender': '#e74c3c',   # Red X marker
    'orientation': '#9b59b6', # Purple arrow for body facing
    'pitch': '#1a472a',      # Dark green
    'background': '#0d1117', # Dark background
}

# Player markers
# - Regular players: circle, size 8
# - Ball carrier: circle, size 12, white edge
# - Runner (phantom): star, size 15, gold
# - Nearest defender: X marker, size 10, red

# Run path: gold line with decreasing alpha (trail effect)
# Body orientation: purple arrow from player position, length proportional to speed
# Pass lane: green filled wedge (matplotlib Wedge patch) at 20% opacity
```

## Coordinate Conversion

TRACAB skeleton positions are in centimeters from pitch center. Convert to meters:
```python
x_meters = x_cm / 100.0
y_meters = y_cm / 100.0
```
Positions XML is already in meters (with x in cm, check docs).

The pitch template uses meters with origin at center, matching TRACAB coordinate system directly after cm→m conversion.

## Saving

```python
# High-res for executive summary
fig.savefig('output/viz/hero_slide.png', dpi=300, bbox_inches='tight', facecolor=fig.get_facecolor())

# Low-res for video (<720p)
fig.savefig('output/viz/hero_slide_video.png', dpi=150, bbox_inches='tight', facecolor=fig.get_facecolor())
```
