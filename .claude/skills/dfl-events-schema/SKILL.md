---
name: dfl-events-schema
description: Schema reference for DFL Event Data XML including pass events, possession, and match events relevant to phantom run detection
user-invocable: false
---

# DFL Events Schema Reference

## File: Events_*.xml (DFL-03-EventData-Match-Raw)

XML structure with match events recorded by Sportec Solutions.

## Event Types Relevant to Phantom Runs

### Pass (Section 2.3.17 - Plays)
The Play element contains pass sub-events:
- `successfulPass` — pass reached intended target
- `unsuccessfulPass` — pass intercepted or out of bounds
- Attributes: `N_EventId`, `N_GameSection` (1=1st half, 2=2nd half), `RealTime`, `GameTime`
- Player attributes: sender player ID, receiver player ID

### Ball Claiming (Section 2.3.31)
- When a player gains possession of the ball
- Links to frame data for identifying ball carrier

### Possession Loss (Section 2.3.20)
- `PossessionLossBeforeGoal` — team loses ball leading to opponent goal
- TypeOfPossessionLoss: unsuccessfulPass, tackle, etc.

### Shot at Goal (Section 2.3.16)
- Contains: DistanceToGoal, AngleToGoal, Pressure, PlayerSpeed, AmountOfDefenders
- Useful for context: phantom runs that preceded shots

## Key Attributes Across Events

| Attribute | Description |
|-----------|-------------|
| N_EventId | Unique event identifier |
| N_GameSection | 1=1st half, 2=2nd half, 3=1st extra, 4=2nd extra, 5=penalties |
| RealTime | ISO timestamp of the event |
| GameTime | Game clock time |
| N_Person_Id | DFL person identifier for the player |
| Club_Id | DFL club identifier |

## File: Positions_*.xml — Ball Possession Data

The Positions XML contains frame-by-frame ball data including:
- **Spielzustand** (game state): whether ball is in play
- **Ballbesitz** (ball possession): which team has the ball
  - Heimmannschaft = Home team
  - Auswaertsmannschaft = Away team
- **BallStatus**: in play or dead ball (after foul, out of bounds, etc.)

## File: MatchInformations_*.xml

Contains:
- Team names and DFL IDs
- Player roster: DFL Person ID → jersey number, name, position
- Formation data
- Kickoff times for each half

## Linking Events to Frames

Events have timestamps, skeleton data has frame numbers. To link:

1. From MatchInformations, get kickoff timestamp for each half
2. From Positions metadata, get phase_start frame numbers
3. For each event:
   ```
   event_offset_seconds = (event_timestamp - kickoff_timestamp).total_seconds()
   frame_number = phase_start + round(event_offset_seconds * framerate)
   ```
4. Validate: frame must be within [phase_start, phase_end]

## Player ID Mapping

Events use DFL `N_Person_Id`, skeleton uses `jersey_number` + `team`.
Bridge via MatchInformations which maps Person_Id → jersey_number + team.
