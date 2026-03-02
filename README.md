# Fire Disaster Evacuation Simulation

## Multi-Agent Systems Project | DSA 560 2.0

A NetLogo-based multi-agent simulation for modeling fire disaster evacuation scenarios in buildings. This simulation allows researchers and safety planners to study evacuation dynamics, test emergency protocols, and analyze the impact of various factors on evacuation outcomes.

---

## Table of Contents

1. [Project Overview](#project-overview)
2. [System Architecture](#system-architecture)
3. [Agent Types and Behaviors](#agent-types-and-behaviors)
4. [Configurable Parameters](#configurable-parameters)
5. [Key Features](#key-features)
6. [How to Run](#how-to-run)
7. [Experiments and Scenarios](#experiments-and-scenarios)
8. [Technical Implementation](#technical-implementation)
9. [Future Enhancements](#future-enhancements)
10. [References](#references)

---

## Project Overview

### Objective

This simulation models emergency evacuation during a fire disaster in a building environment. The goal is to study:

- How different evacuation strategies affect survival rates
- The impact of building design (exits, obstacles, layout) on evacuation efficiency
- The role of emergency responders (firefighters) in reducing casualties
- How panic and crowd dynamics influence evacuation behavior
- The effect of population demographics (elderly, disabled) on overall evacuation time

### Multi-Agent System Approach

The simulation implements a true multi-agent system where:
- **Autonomous agents** make independent decisions based on local information
- **Emergent behavior** arises from interactions between agents
- **Environmental dynamics** (fire spread, smoke) create a changing landscape
- **Agent communication** occurs through observation and proximity-based influence

---

## System Architecture

```
+----------------------------------------------------------+
|                    SIMULATION ENVIRONMENT                 |
|  +----------------------------------------------------+  |
|  |                    BUILDING                         |  |
|  |  +--------+  +--------+  +--------+  +--------+    |  |
|  |  | Floor  |  | Walls  |  | Exits  |  |Obstacles|   |  |
|  |  +--------+  +--------+  +--------+  +--------+    |  |
|  +----------------------------------------------------+  |
|                                                          |
|  +----------------------------------------------------+  |
|  |                      AGENTS                         |  |
|  |  +--------+  +--------+  +-------------+           |  |
|  |  | People |  | Fires  |  | Firefighters|           |  |
|  |  +--------+  +--------+  +-------------+           |  |
|  |                                                    |  |
|  |  +--------+  +--------+                            |  |
|  |  | Smoke  |  | Exits  |                            |  |
|  |  +--------+  +--------+                            |  |
|  +----------------------------------------------------+  |
|                                                          |
|  +----------------------------------------------------+  |
|  |                 GLOBAL SYSTEMS                      |  |
|  |  +--------+  +------------+  +-----------+         |  |
|  |  | Alarm  |  | Pathfinding|  | Statistics|         |  |
|  |  +--------+  +------------+  +-----------+         |  |
|  +----------------------------------------------------+  |
+----------------------------------------------------------+
```

### Component Interaction Flow

```
Fire Starts
    |
    v
Fire Spreads --> Generates Smoke --> Updates Heat Levels
    |                  |                    |
    v                  v                    v
Triggers Alarm --> People Detect     People Take Damage
    |              Fire/Smoke               |
    v                  |                    v
People React <---------+              Casualties
    |
    v
Choose Exit Strategy
    |
    v
Navigate to Exit (BFS Pathfinding)
    |
    +---> Avoid Fire
    +---> Avoid Crowds
    +---> Follow Leaders
    |
    v
Evacuate or Die
```

---

## Agent Types and Behaviors

### 1. People Agents

**Attributes:**
| Attribute | Description | Range |
|-----------|-------------|-------|
| `speed` | Movement speed | 0.5 - 1.0 |
| `mobility` | Physical capability (1.0=normal, 0.5=elderly) | 0.5 - 1.0 |
| `panic-level` | Current panic state | 0 - 100 |
| `awareness` | Awareness of danger | 0 - 100 |
| `health` | Current health | 0 - 100 |
| `is-leader?` | Whether can guide others | Boolean |
| `evacuation-strategy` | How they choose exits | String |

**Evacuation Strategies:**

1. **Nearest Exit (50%)**: Choose the closest available exit
2. **Follow Crowd (30%)**: Follow nearby people who are already evacuating
3. **Familiar Exit (20%)**: Prefer the main entrance they came in through

**Behavior State Machine:**

```
[Idle] --> (Alarm/Fire Detected) --> [Reacting] --> (Time Elapsed) --> [Evacuating]
                                                                            |
                     +------------------------------------------------------+
                     |
                     v
              [Moving to Exit] --> (At Exit) --> [Evacuated]
                     |
                     +---> (Health = 0) --> [Deceased]
                     |
                     +---> (Path Blocked) --> [Re-routing]
```

**Panic Dynamics:**
- Panic increases when near fire, in smoke, or in crowds
- High panic (>60) causes faster but erratic movement
- Leaders reduce panic of nearby agents
- Panic slowly decays over time

### 2. Fire Agents

**Attributes:**
| Attribute | Description | Range |
|-----------|-------------|-------|
| `intensity` | Fire strength | 10 - 100 |
| `age` | How long burning | 0 - inf |
| `my-floor` | Floor location | 1 - 5 |

**Behavior:**
- Intensity increases by 5 every 10 ticks
- Spreads to adjacent passable patches probabilistically
- Spread probability = `fire-spread-rate * (intensity / 50)`
- Heats patches it occupies to 100
- Visual size scales with intensity

### 3. Firefighter Agents

**Attributes:**
| Attribute | Description | Range |
|-----------|-------------|-------|
| `water-level` | Remaining water | 0 - 100 |
| `target-fire` | Fire being targeted | Agent |
| `rescue-mode?` | Helping people instead | Boolean |

**Behavior:**
1. **Fire Fighting Mode:**
   - Target nearest visible fire
   - Move toward fire
   - Extinguish (reduce intensity by 10, uses 5 water)
   - Return to exit to refill when water < 30

2. **Rescue Mode:**
   - Activated when no nearby fires and low water
   - Help panicked/slow people
   - Reduce their panic by 20
   - Increase their speed by 20%

### 4. Smoke Particles

**Attributes:**
| Attribute | Description | Range |
|-----------|-------------|-------|
| `density` | Smoke thickness | 0 - 50 |
| `age` | Particle age | 0 - 50 |

**Behavior:**
- Generated by fires (30% chance per tick)
- Spread randomly
- Fade over time (density decreases)
- Update patch smoke-level
- Die when age > 50 or density <= 0

---

## Configurable Parameters

### Population Parameters

| Parameter | Description | Default | Range |
|-----------|-------------|---------|-------|
| `num-people` | Number of building occupants | 50 | 10 - 200 |
| `elderly-percentage` | % of mobility-impaired people | 10 | 0 - 50 |

### Building Parameters

| Parameter | Description | Default | Options |
|-----------|-------------|---------|---------|
| `num-exits` | Number of emergency exits | 2 | 1 - 4 |
| `num-floors` | Number of floors (conceptual) | 1 | 1 - 5 |
| `obstacle-count` | Furniture/equipment count | 10 | 0 - 50 |
| `building-layout` | Floor plan type | office | office, school, mall, open |

### Fire Parameters

| Parameter | Description | Default | Range |
|-----------|-------------|---------|-------|
| `fire-intensity` | Initial fire strength | 50 | 10 - 100 |
| `fire-spread-rate` | Speed of fire spread | 5 | 1 - 20 |
| `fire-start-location` | Where fire begins | random | random, center, corner |

### Emergency Response Parameters

| Parameter | Description | Default | Range |
|-----------|-------------|---------|-------|
| `alarm-delay` | Ticks before alarm triggers | 5 | 0 - 30 |
| `enable-firefighters?` | Include firefighters | ON | ON/OFF |
| `num-firefighters` | Number of firefighters | 3 | 1 - 10 |

---

## Key Features

### 1. BFS-Based Pathfinding

The simulation uses Breadth-First Search to calculate optimal evacuation paths:

```netlogo
to calculate-exit-distances
  ;; Initialize all patches with max distance
  ask patches [ set exit-distance 9999 ]

  ;; Start from exit patches (distance = 0)
  ask patches with [patch-type = "exit"] [
    set exit-distance 0
  ]

  ;; BFS wave propagation
  ;; Each passable patch gets distance = min(neighbor distances) + 1
end
```

**Advantages:**
- Pre-computed distances for fast runtime navigation
- Guaranteed shortest path to any exit
- Handles obstacles and walls automatically

### 2. Dynamic Fire Spread

Fire spread uses a probabilistic cellular automata approach:
- Each fire has a chance to spread based on intensity and spread-rate
- Fire cannot spread to walls, exits, or already-burning patches
- Heat propagates to surrounding patches

### 3. Panic and Crowd Dynamics

Panic is modeled as an emergent phenomenon:
- **Sources**: Fire proximity, smoke, crowding
- **Mitigation**: Leader presence, time decay
- **Effects**: Speed modification, movement randomness

### 4. Leader-Follower Dynamics

10% of people are designated as "leaders":
- Higher awareness
- Calm nearby panicked people
- Others following "follow-crowd" strategy may follow them

### 5. Real-Time Statistics

Monitors track:
- Evacuated count
- Casualties count
- Remaining people
- Active fires
- Evacuation percentage
- Survival rate
- Average panic level

### 6. Visualization

- **People Colors:**
  - Blue: Normal, calm
  - Orange: Elderly/mobility-impaired OR moderately panicked
  - Red: Highly panicked
  - Yellow: Leaders

- **Patch Colors:**
  - Gray: Floor
  - Brown: Walls
  - Green: Exits
  - Red gradient: Heat level
  - Cyan: Stairs

---

## How to Run

### Requirements
- NetLogo 7.0.2 or higher

### Steps

1. **Open NetLogo** and load `Fire Disaster Evacuation.nlogox`

2. **Configure Parameters** using the interface controls:
   - Adjust sliders for population, exits, fire settings
   - Select building layout
   - Enable/disable firefighters

3. **Click "Setup"** to initialize the simulation

4. **Click "Go"** to run continuously, or **"Go Once"** for single steps

5. **Observe**:
   - Watch the visualization
   - Monitor statistics
   - Analyze the evacuation progress plot

### Interface Layout

```
+---------------------------+------------------------+
|  [Setup] [Go] [Go Once]   |                        |
|                           |                        |
|  Population Sliders       |                        |
|  - num-people             |     SIMULATION         |
|  - elderly-percentage     |       VIEW             |
|                           |                        |
|  Building Sliders         |    (41x41 grid)        |
|  - num-exits              |                        |
|  - num-floors             |                        |
|  - obstacle-count         |                        |
|                           |                        |
|  Fire Sliders             |                        |
|  - fire-intensity         |                        |
|  - fire-spread-rate       |                        |
|                           |                        |
|  Choosers                 |                        |
|  - building-layout        |                        |
|  - fire-start-location    |                        |
|                           |                        |
|  Monitors & Plot          |                        |
+---------------------------+------------------------+
```

---

## Experiments and Scenarios

### Scenario 1: Exit Count Impact

**Question:** How does the number of exits affect evacuation success?

**Setup:**
- 100 people
- Fire in center
- Vary exits: 1, 2, 3, 4

**Expected Results:**
- More exits = higher survival rate
- Diminishing returns after 3 exits
- Single exit creates severe bottlenecks

### Scenario 2: Elderly Population Impact

**Question:** How do mobility-impaired occupants affect overall evacuation?

**Setup:**
- 50 people
- 2 exits
- Vary elderly %: 0, 10, 25, 50

**Expected Results:**
- Higher elderly % = longer evacuation time
- May increase overall casualties
- Creates bottlenecks as faster people get stuck behind slower

### Scenario 3: Firefighter Effectiveness

**Question:** Do firefighters significantly improve survival rates?

**Setup:**
- 75 people
- High fire spread rate
- Compare: 0, 3, 5, 10 firefighters

**Expected Results:**
- Firefighters reduce fire spread
- More effective early in simulation
- Rescue mode helps trapped people

### Scenario 4: Building Layout Comparison

**Question:** Which building layout evacuates most efficiently?

**Setup:**
- 100 people
- 2 exits
- Compare: office, school, mall, open

**Expected Results:**
- Open layout: Fastest evacuation
- Office: Bottlenecks at corridor gaps
- School: Classroom divisions create multiple paths
- Mall: Open center with obstacles

### Scenario 5: Alarm Response Time

**Question:** How critical is early alarm activation?

**Setup:**
- 75 people
- Fire in corner
- Vary alarm-delay: 0, 10, 20, 30

**Expected Results:**
- Delayed alarm = more casualties
- People closer to fire suffer most
- Early warning allows orderly evacuation

---

## Technical Implementation

### Global Variables

```netlogo
globals [
  evacuated-count      ;; Successfully evacuated
  casualties-count     ;; Deaths
  total-people         ;; Initial population
  alarm-triggered?     ;; Alarm status
  evacuation-complete? ;; Simulation end flag
  simulation-time      ;; Tick counter
]
```

### Patch Variables

```netlogo
patches-own [
  patch-type      ;; "floor", "wall", "exit", "stairs", "obstacle"
  floor-level     ;; 1-5
  heat-level      ;; 0-100
  smoke-level     ;; 0-100
  is-passable?    ;; Can agents walk here
  exit-distance   ;; BFS distance to nearest exit
]
```

### Main Loop

```netlogo
to go
  ;; Check end conditions
  if evacuation-complete? [ stop ]

  ;; Update alarm
  if not alarm-triggered? and any? fires [
    trigger-alarm
  ]

  ;; Fire dynamics
  spread-fire
  generate-smoke

  ;; Agent behaviors
  ask people [ person-behavior ]
  ask firefighters [ firefighter-behavior ]
  ask smoke-particles [ smoke-behavior ]

  ;; Environment updates
  update-heat
  update-casualties
  check-evacuation-status

  tick
end
```

### Pathfinding Algorithm

The simulation uses BFS (Breadth-First Search) for pathfinding:

1. Initialize all patch distances to infinity
2. Set exit patch distances to 0
3. For each distance level d (starting from 0):
   - Find all patches at distance d
   - For each unvisited passable neighbor, set distance to d+1
4. Agents follow the gradient (move to neighbor with smallest distance)

**Complexity:** O(n) where n = number of patches

---

## Future Enhancements

### Short-term Improvements

1. **Sprinkler Systems**: Automatic fire suppression
2. **Emergency Lighting**: Visual paths to exits
3. **Group Dynamics**: Families staying together
4. **Building Damage**: Structural collapse mechanics

### Medium-term Improvements

1. **True 3D Multi-floor**: Actual floor transitions
2. **Elevator Behavior**: Elevator dynamics during fire
3. **Smoke Inhalation Model**: Detailed health effects
4. **Communication Networks**: Phones, intercoms, announcements

### Long-term Research Directions

1. **Machine Learning Integration**: Train optimal evacuation policies
2. **Real Building Import**: Import CAD floor plans
3. **VR Visualization**: Immersive simulation viewing
4. **Validation Studies**: Compare with real evacuation data

---

## References

### Academic Papers

1. Helbing, D., Farkas, I., & Vicsek, T. (2000). *Simulating dynamical features of escape panic*. Nature, 407(6803), 487-490.

2. Pan, X. (2006). *Computational Modeling of Human and Social Behaviors for Emergency Egress Analysis*. Stanford University.

3. Zheng, X., Zhong, T., & Liu, M. (2009). *Modeling crowd evacuation of a building based on seven methodological approaches*. Building and Environment, 44(3), 437-445.

### NetLogo Resources

- NetLogo User Manual: https://ccl.northwestern.edu/netlogo/docs/
- NetLogo Models Library: Fire model, Evacuation models
- NetLogo Dictionary: https://ccl.northwestern.edu/netlogo/docs/dictionary.html

### Safety Standards

- NFPA 101: Life Safety Code
- IBC: International Building Code
- OSHA Emergency Action Plans

---

## License

This project is developed for educational purposes as part of the DSA 560 2.0 Multi-Agent Systems course.

---

## Authors

- Project developed for Multi-Agent Systems course
- University of Moratuwa / Academic Institution

---

*Last Updated: February 2025*
