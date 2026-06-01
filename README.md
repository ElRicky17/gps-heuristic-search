# GPS Heuristic Search

A Python notebook that implements a real-world **GPS route-finding system** using the **A\* algorithm** on an actual road network. The project loads the street graph of **Envigado, Antioquia, Colombia**, integrates real elevation data, and computes the most fuel-efficient route between two points.

---

## Table of Contents

- [Overview](#overview)
- [Features](#features)
- [Project Structure](#project-structure)
- [Getting Started](#getting-started)
- [How It Works](#how-it-works)
- [Fuel Cost Model](#fuel-cost-model)
- [Visualization](#visualization)
- [Requirements](#requirements)

---

## Overview

This project combines graph-based search algorithms with real geospatial data to solve a practical navigation problem. Using **OSMnx** to fetch the OpenStreetMap road network and the **Open Elevation API** for altitude data, the system models streets as a weighted graph and finds the optimal route using **A\*** with a Haversine-based heuristic.

The route is animated on an interactive map using **Plotly**.

---

## Features

-  **Real map data** from OpenStreetMap via OSMnx (Envigado, Colombia)
-  **Haversine heuristic** — straight-line distance to goal in kilometers
-  **Elevation-aware edge costs** — accounts for uphill/downhill fuel consumption
-  **Fuel cost model** — penalizes uphill segments and rewards downhill ones
-  **Elevation caching** — avoids repeated API calls with local pickle cache
-  **Parallel elevation requests** — uses `ThreadPoolExecutor` for fast batch lookups
-  **Interactive animated route map** using Plotly Express

---

## Project Structure

```
gps-heuristic-search/
│
├── GPS_BusquedaHeurisitca.ipynb   # Main notebook
├── elevations_cache.pkl           # Auto-generated elevation cache
└── README.md
```

### Core Components

| Component | Description |
|---|---|
| `build_nodes_dict()` | Builds a node dictionary with coordinates, heuristic, and elevation |
| `edges_dict` | Adjacency dictionary with fuel cost per edge |
| `GPS(Node)` | Subclass of `Node` for the GPS problem |
| `A()` | A\* search algorithm with visited-set optimization |
| `get_elevations()` | Fetches elevations in parallel batches from Open Elevation API |
| `haversine()` | Computes straight-line distance between two GPS coordinates |

---

## Getting Started

### Prerequisites

- Python 3.8+
- Jupyter Notebook or Google Colab

### Installation

```bash
pip install osmnx plotly_express networkx geopy requests numpy
```

### Run the Notebook

```bash
jupyter notebook GPS_BusquedaHeurisitca.ipynb
```

> **Note:** The first run will call the [Open Elevation API](https://open-elevation.com/) to fetch elevation data for all nodes. This is cached locally in `elevations_cache.pkl` for subsequent runs.

---

## How It Works

### 1. Load the Road Network
```python
G = ox.graph_from_place('Envigado, Antioquia, Colombia', network_type='drive')
```
Fetches all drivable streets as a directed graph.

### 2. Geocode Start & End Points
```python
start = locator.geocode('Sede Posgrados EIA, Envigado, Colombia')
end   = locator.geocode('Universidad EIA, Envigado, Colombia')
```
Converts addresses to GPS coordinates and finds the nearest OSM nodes.

### 3. Build Node & Edge Dictionaries
Each node stores: `(x, y, heuristic_to_goal, elevation)`  
Each edge stores: `(length, maxspeed, highway, fuel_cost)`

### 4. Run A\*
```python
Ciudad = GPS(DtaNodesFrame=nodes_dict, DtaEdgesFrame=edges_dict,
             state=initState, operators=operators, value="Inicio")
tree, objective = A(Ciudad, endState)
ruta = objective.get_path_values()
```

### 5. Visualize the Route
An animated scatter map shows each step of the route with start (red) and end (green) markers.

---

## Fuel Cost Model

Edge weights are adjusted based on road slope to simulate real fuel consumption:

```
grade      = elevation_change / horizontal_distance
fuel_cost  = length_km × max(MIN_MULT, 1.0 + UPHILL_COEF × grade⁺ + DOWNHILL_COEF × grade⁻)
```

| Parameter | Value | Effect |
|---|---|---|
| `UPHILL_COEF` | `4.0` | Strong penalty for climbing |
| `DOWNHILL_COEF` | `-1.0` | Small saving going downhill |
| `MIN_MULT` | `0.7` | Minimum cost multiplier |

---

## Visualization

The final route is displayed as an **animated Plotly map**:

- ⚫ Black dots — each node along the route
- 🔴 Red marker — start point
- 🟢 Green marker — destination
- Line overlay — full path

---

## Requirements

```txt
osmnx
plotly_express
networkx
geopy
requests
numpy
pickle
concurrent.futures (built-in)
```

---

##  Concepts

- **A\* Search:** Finds the shortest (or lowest-cost) path by combining accumulated cost `g(n)` and heuristic `h(n)`.
- **Haversine Formula:** Calculates the great-circle distance between two GPS points on Earth.
- **Admissible Heuristic:** The straight-line distance never overestimates the actual road distance, ensuring A\* finds the optimal path.
- **Fuel-Aware Cost:** Goes beyond simple distance by modeling real driving conditions with elevation gradients.
