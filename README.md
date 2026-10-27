# Tehran Road Network & University Accessibility Analysis

**Advanced Programming — HW3**
Department of Math & Computer Science, KNTU · May 2025

Spatial network analysis on Tehran's OpenStreetMap road network using `pyrosm`, `networkx`, `scipy`, and `ipyleaflet`.

---

## 📌 Overview

This project has two parts, both working on real Tehran OSM data:

### Part 1 — Interactive Route Finder
- Loads Tehran's driving network via `pyrosm.get_network("driving")`
- Builds a graph where nodes are road coordinates and edges are road segments
- User clicks two points on an interactive map (`ipyleaflet`)
- Each click is snapped to the nearest road node using a `scipy.spatial.cKDTree`
- The shortest path is computed with `networkx.shortest_path` (Dijkstra)
- The route is drawn on the map as a blue polyline

### Part 2 — University Accessibility Zones
- Extracts **universities** (`amenity=university`) and **buildings** (`osm.get_buildings()`) from the same OSM file
- Limits the analysis to buildings within a **~2 km buffer** around universities (to keep the graph small)
- Snaps each building centroid and each university to its nearest road node
- Runs Dijkstra from each university to compute **network distances** to all reachable nodes
- Assigns each building to its **closest university by road distance** (not straight-line distance)
- Visualizes the result on an interactive map

---

## 🗂️ Repository Structure

```
.
├── notebooks/
│   └── tehran_road_network.ipynb   # main analysis notebook (Parts 1 & 2)
├── outputs/
│   ├── screenshots/                # map screenshots
│   └── maps/                       # exported HTML maps (optional)
├── report/
│   └── report.pdf                  # short methodology & findings report
├── requirements.txt
└── README.md
```

> The Tehran `.pbf` file (~100 MB) is **not committed** — it is downloaded automatically on first run via `pyrosm.get_data("Tehran")`.

---

## ⚙️ Setup

### 1. Install dependencies
```bash
pip install -r requirements.txt
```

> `pyrosm` on Windows is easiest via conda:
> ```bash
> conda install -c conda-forge pyrosm
> ```

### 2. Run the notebook
```bash
jupyter notebook notebooks/tehran_road_network.ipynb
```

The first cell downloads Tehran's PBF from Geofabrik automatically.

---

## 🧠 Methodology Summary

| Step | Tool / Method |
|------|---------------|
| OSM data download | `pyrosm.get_data("Tehran")` |
| Road extraction | `OSM.get_network("driving")` |
| Buildings / POIs | `OSM.get_buildings()`, `OSM.get_pois()` |
| Graph construction | `networkx.Graph` (custom `roads_to_graph`) |
| Nearest-node snapping | `scipy.spatial.cKDTree` |
| Shortest path / distances | `networkx.single_source_dijkstra_path_length` |
| Visualization | `ipyleaflet.Map`, `GeoData`, `Heatmap`, `Marker` |

### Graph construction
- Each road row's geometry is decomposed into `LineString`s (or `MultiLineString` parts)
- Consecutive coordinate pairs become weighted edges
- Edge weight currently = Euclidean distance between coordinate points

### Nearest-node snapping
- All graph nodes are collected into a single NumPy array `(lon, lat)`
- A `cKDTree` is built once at startup
- `tree.query([lon, lat])` returns the nearest node in O(log N)

### Building → University assignment
- For each university, `nx.single_source_dijkstra_path_length(G, source=uni_node, weight="weight")`
  gives distances from that university to every reachable road node
- For each building, its nearest road node is looked up in each university's distance dict
- The university with the **smallest network distance** wins

---

## 📸 Screenshots

*(Add after exporting from the notebook)*

| Route Finder | Building–University Assignment |
|--------------|-------------------------------|
| `outputs/screenshots/route_finder.png` | `outputs/screenshots/accessibility.png` |

---

## 🚧 Known Limitations / Future Work

- **Edge weights are in degrees, not meters.** Euclidean distance on raw lat/lon underweights east–west edges by ~19% at Tehran's latitude. Planned fix: Haversine distance (meters).
- **Buffer is in degrees, not meters.** `buffer = 0.02°` gives ~2.2 km north–south but ~1.8 km east–west. Planned fix: reproject to UTM 39N (`EPSG:32639`) before buffering.
- **Dijkstra `cutoff=10000`** was intended as 10 km but is applied to degree-based weights, so it currently has no practical effect. Planned fix: apply cutoff after switching to meter weights.
- **Part 2 visualization is a building-density heatmap**, not the requested per-university thematic map. Planned fix: color buildings by their assigned university, or draw the Voronoi partition of the road network.
- **Undirected graph** — OSM one-way tags are not respected. Acceptable for the accessibility analysis, but should be noted for routing.
- **Part 2 scalability** — per-university Dijkstra can be slow on the full graph; a multi-source Dijkstra (all universities seeded at once) would compute all assignments in a single pass.

---

## 📜 Data Source

OpenStreetMap — © OpenStreetMap contributors, ODbL license.
Tehran extract downloaded via [Geofabrik](https://download.geofabrik.de/asia/iran.html).

---

## 👤 Author

**Navid**
Department of Math & Computer Science, KNTU
Course: Advanced Programming — HW3 · Spring 2025
