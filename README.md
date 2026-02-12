# MAS-Traffic-Simulator

An agent-based simulator for studying **departure timing (“waiting”)** and **route choice** in a **dynamic road network** where a “Fast/Express” option opens later on each segment. The project compares heterogeneous driver heuristics and visualizes emergent congestion dynamics.

This repo was built for a course project and is designed to be **one-file runnable** (baseline run + plots + optional analysis hooks).

---

## What this project simulates

- A **5-node directed chain**: nodes `0 → 1 → 2 → 3 → 4`
- For each segment `(i → i+1)` there are **3 parallel roads**:
  - **Slow**: higher base time, low congestion sensitivity
  - **Std**: medium base time and sensitivity
  - **Exp/Fast**: low base time, **high congestion sensitivity**, opens later at each node (`open_at(i)`)

Travel times depend on congestion:
- Each road has a flow count (how many agents are currently traversing it).
- When many agents herd into the same road (especially the Fast one after it opens), travel time increases sharply.

We also simulate uncertainty:
- **Incidents** can occur on entry with road-specific probability and increase travel time by a multiplicative factor (e.g., `γ = 1.6`).

---

## Driver strategies (heuristics)

Each agent follows exactly one of the following:

1. **Aggressive**
   - Departs immediately when possible.
   - Picks the currently available road with minimum instantaneous travel time.

2. **Strategic**
   - Uses a simple one-step lookahead:
     - May wait for the next road opening if it expects a better outcome.
   - Otherwise chooses the current best available road.

3. **Risk-Averse**
   - Filters roads by incident probability (e.g., only consider roads with `p_inc ≤ 0.1`).
   - If safe roads exist, picks the safest best road; otherwise falls back to best available.

4. **Social**
   - Uses a proxy for “congestion externalities”:
     - prefers roads that reduce congestion pressure (internalizing some marginal cost).

---

## Metrics (logged per agent)

For each agent `i` we record:
- `Arrival`: time step when reaching node 4
- `Wait`: number of WAITING steps
- `Drive`: number of TRAVERSING steps
- `Fuel`: fuel proxy accumulated per segment
- `Incidents`: count of incident events
- `Utility`: a single scalar objective:
  \[
  U_i = -(W_i + D_i + 2.5 \cdot F_i)
  \]
- `Regret`: myopic regret at decision times:
  \[
  R_i(t) = T_{\text{chosen}}(t) - \min_{e \in A(i,t)} T_e(t)
  \]
  *(diagnostic only; not a Nash-equilibrium claim)*

---

## Plots produced

The default run generates:
- **Utility by Strategy** (boxplot)
- **Arrival Distribution** (stacked histogram)
- **Congestion Waves** (number of traversing agents over time by strategy)
- **Myopic Regret Over Time** (behavioral diagnostic)
- **Road-choice shares over time** (stacked area)
- **Congestion heatmap** (node × time traversal counts)
- 
---

## Deterministic randomness (CRN)

The simulator uses **Common Random Numbers (CRN)** via deterministic hashing:
- randomness is generated from `(seed, agent_id, node, road, tag)`
- this ensures reproducible results and fair comparison across runs
