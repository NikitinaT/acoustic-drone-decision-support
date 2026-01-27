# Event-Based Acoustic Decision Support Layer

## Overview

This repository implements a **decision-making layer** that operates on top of
**event-based outputs produced by an Acoustic ML model**.

The system **does not process raw audio signals** and does not perform signal processing.
Instead, it consumes a **strict data contract** representing the output of an upstream
Acoustic ML pipeline and transforms it into **time-aware, explainable decision outputs**
suitable for user alerts.

The project is centered around a single core principle:

> **Acoustic ML output is the primary and mandatory input.  
All other modalities are optional and must not block or delay decisions.**

---

## Core Concept

### Primary responsibility

- Define and enforce a **data contract** between:
  - Acoustic ML inference
  - Decision support logic
- Aggregate acoustic detection events over time
- Apply decision rules and state transitions
- Produce structured outputs for alerting and visualization

### Out of scope

- Audio signal processing (DSP)
- Acoustic feature extraction
- ML model training or tuning
- Hardware-level integration

---

## Input Data Contract (Mandatory)

### Primary input: Acoustic ML events

The **main input** to this system is the output of an Acoustic ML model that has already
processed raw microphone data.

This contract is **mandatory** and the system cannot operate without it.

### File structure

```
input/
  sessions/
    session_<timestamp>__<id>/
      acoustic_events.jsonl          ← REQUIRED
      human_interaction.jsonl        ← OPTIONAL
      video_observation.jsonl        ← OPTIONAL
```

Only `acoustic_events.jsonl` is required.  
All other input files may be empty or absent.

---

### Acoustic Events Contract

The acoustic event file represents **event-level ML inference results**, not signals.

Typical fields include:
- `timestamp`
- `drone_detected: true | false`
- `p_drone ∈ [0,1]`
- `proximity: FAR | MID | CLOSE`
- `eta_seconds` (optional)
- `direction / distance` (optional)
- `drone_presence_pattern: single | multiple`
- `environment_context` (acoustically inferred noise conditions)

Schema definition:
```
schemas/acoustic.events.schema.json
```

---

## Optional Fusion Inputs

Optional inputs may provide context but must **never block decisions**.

## Optional Input: Human Interaction (Dictionary-based)

Human interaction is an **optional** low-bandwidth context channel.  
It is **not** raw speech, not free text, and does not label objects.  
Only dictionary events are allowed.

### Allowed values (dictionary)

- `space` (enum):
  - `inside`, `outside`, `open_area`, `forest`, `near_buildings`

- `mobility` (enum):
  - `stationary`, `on_foot`, `in_vehicle`

- `environment` (array of enums):
  - `wind`, `traffic_noise`, `quiet`, `urban_noise`

- `visual_hint` (boolean):
  - `true` if the human has visual contact with an unidentified object (no classification)

### Speech / keyword mode (if speech is used)
If a microphone is used for human speech, the system must perform **keyword matching only**:
- Accepted words are mapped to the dictionary fields above.
- Any other speech content is discarded (privacy-by-design).


Schema:
```
schemas/human_interaction.schema.json
```

### Video Observation (Optional)

- Coarse observations only
- `detected / not detected`
- Low-frequency updates (~1 Hz)

Schema:
```
schemas/video_observation.schema.json
```

---

## Decision Support Layer

Consumes acoustic (and optional fusion) events and performs:

- Time-aware aggregation
- State machine transitions
- Alert suppression and cooldown handling
- Confidence estimation

### Decision States

- **IDLE** — monitoring only
- **ALERT** — alert issued to the user
- **COOLDOWN** — alert already delivered, no repeated notifications

---

## Decision Output Contract

Decision results are always linked to input sessions.

### Output structure

```
output/
  sessions/
    session_<timestamp>__<id>/
      decision_output.jsonl
```

Schema:
```
schemas/decision_output.schema.json
```

Typical output fields:
- `decision_state`
- `p_drone`
- `confidence_level`
- `alert_triggered`
- `alert_types`
- `proximity`
- `eta_seconds`
- `explanation` (optional)

---

## Handling Multiple Objects

Exact object counting is not performed.

Instead:
```
drone_presence_pattern: single | multiple
```

---

## Design Principles

- Acoustic-first design
- Strict separation of concerns
- Event-based processing
- Time-aware decisions
- Optional fusion inputs
- Explainable outputs
- Privacy-first human interaction

---

## Intended Use

This repository represents a **decision-layer prototype** for:
- architecture validation
- synthetic data replay
- decision logic evaluation
- visualization workflows

---

## Project Status

- Data contracts: defined and stable
- Synthetic sessions: included
- Decision logic: prototype
- Fusion extensions: future work

---
