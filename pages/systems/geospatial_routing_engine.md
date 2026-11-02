---
layout: default
title: "Self-Hosted Geospatial Vector Tile Server, OSRM Routing & VRP Fleet Optimizer"
---

<style>
.project-meta-card {
   background: #f8fafc;
   border: 1px solid #e2e8f0;
   border-left: 4px solid #267CB9;
   border-radius: 6px;
   padding: 16px 18px;
   margin: 16px 0 24px 0;
   font-family: "Noto Sans", "Helvetica Neue", Helvetica, Arial, sans-serif;
}
.project-meta-grid {
   display: grid;
   grid-template-columns: repeat(auto-fit, minmax(180px, 1fr));
   gap: 12px 20px;
   padding-bottom: 12px;
   border-bottom: 1px solid #e2e8f0;
   margin-bottom: 12px;
}
.project-meta-item {
   display: flex;
   flex-direction: column;
}
.project-meta-item.full-width {
   grid-column: 1 / -1;
}
.project-meta-label {
   font-size: 10.5px;
   font-weight: 700;
   text-transform: uppercase;
   letter-spacing: 0.06em;
   color: #267CB9;
   margin-bottom: 3px;
}
.project-meta-value {
   font-size: 13.5px;
   color: #222222;
   font-weight: 400;
   line-height: 1.4;
}
.project-stack-list {
   display: flex;
   flex-wrap: wrap;
   gap: 6px;
   margin-top: 6px;
}
.tech-badge {
   display: inline-block;
   background: #ffffff;
   color: #267CB9;
   border: 1px solid #bcd6eb;
   border-radius: 4px;
   padding: 2px 9px;
   font-size: 12px;
   font-weight: 400;
   line-height: 1.5;
}
</style>

# Self-Hosted Geospatial Vector Tile Server, OSRM Routing & VRP Fleet Optimizer

<div class="project-meta-card">
  <div class="project-meta-grid">
    <div class="project-meta-item">
      <span class="project-meta-label">Published</span>
      <span class="project-meta-value">September 2026</span>
    </div>
    <div class="project-meta-item">
      <span class="project-meta-label">Role</span>
      <span class="project-meta-value">Geospatial Infrastructure &amp; Operations Research Architect</span>
    </div>
    <div class="project-meta-item full-width">
      <span class="project-meta-label">Domain</span>
      <span class="project-meta-value">WebGL Vector Cartography, Street-Level Turn-by-Turn Routing &amp; Capacitated Vehicle Routing Problem with Time Windows (CVRPTW)</span>
    </div>
  </div>
  <div class="project-meta-item">
    <span class="project-meta-label">Technology Stack</span>
    <div class="project-stack-list">
      <span class="tech-badge">TileServer GL (C++ / Node)</span>
      <span class="tech-badge">OSRM (Open Source Routing Machine)</span>
      <span class="tech-badge">VROOM (VRPTW Metaheuristic Solver)</span>
      <span class="tech-badge">OpenMapTiles v3.14 (MBTiles / PBF)</span>
      <span class="tech-badge">MapLibre GL JS (WebGL 60 FPS)</span>
      <span class="tech-badge">Leaflet.js</span>
      <span class="tech-badge">Python 3</span>
      <span class="tech-badge">Nginx Alpine (Protobuf Gzip &amp; SSL)</span>
      <span class="tech-badge">Certbot (Let's Encrypt)</span>
      <span class="tech-badge">Docker Compose</span>
    </div>
  </div>
</div>

![SIAP Geospatial & VRP Engine Overview](/images/systems/maps_cover.png)

---

## Executive Summary

High-volume last-mile logistics platforms execute tens of thousands of map tile renders, distance matrix calculations, and multi-stop route optimizations every day. Relying on commercial cloud APIs (such as Google Maps Platform or Mapbox) at scale introduces prohibitive per-request billing, strict rate limits, and external network dependencies.

To achieve **100% operational autonomy with $0/month in third-party API fees**, I engineered **`siap-map-server`**: a self-hosted, air-gapped geospatial and route optimization infrastructure running on Docker. The platform integrates four specialized engines behind a hardened **Nginx HTTP/2 reverse proxy**:
1. **TileServer GL** serving compressed **Protocol Buffer Vector Tiles (`.pbf`)** for 60 FPS GPU client rendering alongside `@2x` Retina raster PNG fallbacks across the entire Mexican Republic (`mexico.mbtiles`).
2. **OSRM (Open Source Routing Machine)** computing sub-millisecond street-level trajectories and $N \times M$ travel-time/distance matrices over Mexico's road network.
3. **VROOM** solving multi-vehicle **Capacitated Vehicle Routing Problems with Time Windows (CVRPTW)**, vehicle skills, driver meal breaks, and 8-hour highway overnight stays (*pernoctas*) for up to **200 vehicles and 1,000 stops per batch**.

---

## 1. End-to-End Geospatial & VRP Architecture

The infrastructure decouples heavy cartographic and combinatorial optimization workloads from the core SIAP transactional servers:

```mermaid
graph TB
    subgraph CLIENTS ["1. Client Applications & Map Viewers"]
        WebGLMap["SIAP Dispatch Portal<br/><i>Leaflet + MapLibre GL JS (60 FPS WebGL GPU)</i>"]
        RasterFallback["Legacy & Mobile Map Clients<br/><i>Standard L.tileLayer ({z}/{x}/{y}@2x.png)</i>"]
        DiagnosticSPA["Built-In Diagnostic SPA (/demo/index.html)<br/><i>Coordinate HUD, Layer Switcher & z/x/y.pbf Tile Inspector</i>"]
        BackendWorkers["SIAP Backend & Route Planners (Python / .NET)<br/><i>Fleet Optimization & ETA Consumers</i>"]
    end

    subgraph EDGE ["2. Edge Caching & Security Proxy (map-nginx-proxy)"]
        NginxProxy["Nginx Alpine (Ports 80 / 443 | HTTP/2 & TLS 1.3)<br/>• Protobuf Gzip Level 6 (application/x-protobuf)<br/>• 30-Day Immutable Cache (max-age=2592000)<br/>• Wildcard CORS (*) & Certbot 12h SSL Auto-Renew"]
    end

    subgraph ENGINES ["3. Cartography, Routing & Operations Research Engines"]
        TileServer["map-tileserver (TileServer GL)<br/><b>Port 8080</b><br/>• Vector Tiles: /data/v3/{z}/{x}/{y}.pbf<br/>• Style JSON: /styles/positron-custom/style.json<br/>• Raster PNG & @2x Retina Renderer<br/>• PBF Font Glyphs (Noto Sans) & Sprites"]
        
        OSRMEngine["OSRM Routing Engine (C++)<br/><b>Port 5000 (car profile)</b><br/>• Point-to-Point: /route/v1/driving/ (GeoJSON LineString)<br/>• NxM Distance/Duration Matrix: /table/v1/driving/"]
        
        VroomSolver["VROOM Optimization Engine (C++)<br/><b>Port 3000 | 4 Threads | Explore Level 5</b><br/>• Endpoint: POST /api/optimize<br/>• Limits: 200 Vehicles & 1,000 Locations per Batch<br/>• Constraints: Capacities, Time Windows, Skills, Breaks & Overnight Stays"]
    end

    subgraph DATA ["4. Pre-Indexed Geospatial Datasets"]
        MBTiles[("mexico.mbtiles (OpenMapTiles v3.14)<br/><i>Bounds: [-118.4, 14.5, -86.7, 32.7]</i><br/>• Roads, Landuse, POIs & 3D Building Footprints")]
        OSRMGraph[("Mexico Road Network Graph (.osrm)<br/><i>Pre-Compiled Contraction Hierarchies</i><br/>• One-Way Streets, Turn Restrictions & Speed Profiles")]
    end

    WebGLMap -->|"HTTPS GET .pbf & style.json"| NginxProxy
    RasterFallback -->|"HTTPS GET @2x.png"| NginxProxy
    DiagnosticSPA -->|"HTTPS Live Inspection"| NginxProxy
    BackendWorkers -->|"POST /api/optimize & GET /api/route"| NginxProxy

    NginxProxy -->|"Proxy /data & /styles"| TileServer
    NginxProxy -->|"Proxy /api/route/*"| OSRMEngine
    NginxProxy -->|"Proxy /api/optimize"| VroomSolver

    VroomSolver -->|"Query NxM Duration/Distance Matrix"| OSRMEngine
    VroomSolver -->|"Fetch Exact Street Geometry"| OSRMEngine

    TileServer -->|"Read-Only Mount"| MBTiles
    OSRMEngine -->|"Memory-Mapped Graph"| OSRMGraph
```

---

## 2. Deep-Dive: Core Subsystems & Algorithmic Capabilities

### A. Vector Tile Cartography & Dual WebGL / Raster Rendering (`TileServer GL`)
Traditional raster map servers transmit heavy PNG images for every zoom level and pan action. `siap-map-server` reverses this paradigm by serving **OpenMapTiles v3.14 (`mexico.mbtiles`)** via **TileServer GL**:
* **Vector Tiles (`/data/v3/{z}/{x}/{y}.pbf`):** Delivers compact binary Protocol Buffer tiles compressed on the fly by Nginx (`gzip_comp_level 6` on `application/x-protobuf` and `application/vnd.mapbox-vector-tile`). The client browser's GPU renders streets, labels, and 3D building footprints at **60 FPS** using **MapLibre GL JS** (`@maplibre/maplibre-gl-leaflet`), reducing map bandwidth consumption and HTTP tile count by **over 80%**.
* **CARTO Positron Visual Style & Retina Fallback:** Serves the high-contrast `positron-custom` style (`#fafaf8` bone background, crisp `#dddddd` street casings) so colored fleet polylines stand out clearly. For legacy devices without WebGL, TileServer GL dynamically renders `512px` standard and `@2x` Retina PNG tiles (`maxScaleFactor: 2`, `png: 90`, `webp: 85`).
* **30-Day Immutable Browser Caching:** Nginx attaches `Cache-Control: public, max-age=2592000, immutable` and `Access-Control-Allow-Origin: *`, allowing internal web and mobile clients to cache static cartography locally for 30 days.

### B. Sub-Millisecond Street Routing & $N \times M$ Matrix Computation (`OSRM`)
Powered by **OSRM** compiled against the Mexican national road network (`setup_osrm.sh`), enforcing strict GeoJSON coordinate ordering (`[longitude, latitude]`):
* **Turn-by-Turn Trajectory Service (`GET /api/route/route/v1/driving/{lon1},{lat1};{lon2},{lat2}`):** Respects real-world one-way streets, medians, forbidden left turns, bridges, and roundabouts across Mexico, returning metric distances, travel durations in seconds, maneuver steps (`steps=true`), and full `LineString` GeoJSON geometries (`overview=full&geometries=geojson`).
* **Many-to-Many Distance & Duration Table (`GET /api/route/table/v1/driving/...`):** Computes asymmetric $N \times M$ travel-time (`durations`) and metric distance (`distances`) matrices across hundreds of coordinates in a single HTTP request, feeding the VROOM optimization solver.

### C. Multi-Vehicle Capacitated VRPTW & Overnight Stay Optimizer (`VROOM`)
Configured via `config/vroom.yml` (`threads: 4`, `explore: 5`, `maxlocations: 1000`, `maxvehicles: 200`, `timeout: 300000`), the `POST /api/optimize` endpoint solves complex **Vehicle Routing Problems with Time Windows (VRPTW)**:
* **Multi-Dimensional Capacities & Skills:** Matches vehicle payload limits (`capacity: [weight_kg, volume_m3]`) and equipment tags (`skills: [1, 2]` for refrigerated units or hydraulic liftgates) against customer job requirements (`delivery`, `pickup`, `service` unloading duration, and `priority: 0..100`).
* **Driver Meal Breaks & Multi-Day Highway Overnight Stays (*Pernoctas*):** Supports long-haul inter-city routes by embedding mandatory time-windowed breaks directly into each vehicle's schedule—such as a 45-minute meal window (`service: 2700`) and an 8-hour hotel overnight rest window (`service: 28800` between `22:00` and `06:00`).
* **Closed-Loop vs. Open-Loop Topological Optimization:** Automatically avoids route self-intersections (*criss-cross / bow-tie effects*) and schedules pickups after deliveries so trucks never haul unnecessary weight early in the route.

```mermaid
sequenceDiagram
    autonumber
    actor Planner as SIAP Route Planner / Python Worker
    participant Nginx as map-nginx-proxy (80 / 443)
    participant Vroom as VROOM Solver (3000)
    participant OSRM as OSRM Routing Engine (5000)
    participant Leaflet as Dispatch Web UI (Leaflet + MapLibre)

    Planner->>Nginx: POST /api/optimize (Vehicles, Capacities, Time Windows, Breaks, Jobs)
    Nginx->>Vroom: Forward JSON Payload (Up to 200 Vehicles and 1,000 Stops)
    
    Vroom->>OSRM: GET /table/v1/driving/ (Compute Asymmetric NxM Duration and Distance Matrix)
    OSRM-->>Vroom: Return NxM Travel Durations and Metric Distances
    
    Note over Vroom: 4-Thread Metaheuristic Exploration (Explore Level 5)
    Vroom->>Vroom: Solve CVRPTW (Respecting Capacities, Skills, Meal Breaks and 8h Overnight Stays)
    
    Vroom->>OSRM: GET /route/v1/driving/ (Request Exact Street Geometry for Optimal Sequence)
    OSRM-->>Vroom: Return Street-Level LineString Polylines
    
    Vroom-->>Nginx: Return Optimal Routes, ETAs, Load Profiles, Geometries and Unassigned Diagnostics
    Nginx-->>Planner: HTTP 200 JSON Response (code=0)
    
    Planner->>Leaflet: Render Color-Coded Fleet Polylines and Numbered Stop Popups
    Leaflet->>Nginx: GET /data/v3/{z}/{x}/{y}.pbf (Fetch Vector Base Map Tiles)
    Nginx-->>Leaflet: Return Gzip-Compressed Protobuf Vector Tiles (60 FPS WebGL)
```

---

## 3. Built-In Diagnostic SPA (`demo/index.html`) & Streaming Deployment

* **Interactive Cartographic Inspector (`demo/index.html`):** Packaged directly into the repository and served at `/demo`, this zero-dependency Single-Page Application allows engineers to:
  * Hot-swap between **MapLibre GL WebGL Vector**, **Raster Fallback (`{z}/{x}/{y}.png`)**, and **OSM** layers in real time.
  * Audit live cursor coordinates via a floating **Geocoordinate HUD** and jump instantly between Mexican logistics hubs (Guadalajara SIAP Base, CDMX, Monterrey, Queretaro, Tijuana).
  * Mathematically compute and inspect the exact `z/x/y.pbf` vector tile under the viewport crosshair with 1-click download auditing.
* **Zero-Downtime Streaming SSH Deployment (`deploy.sh` & `scripts/healthcheck.sh`):**
  * Streams configuration updates over SSH (`tar -czf - . | ssh ...`), pulls container updates, provisions Let's Encrypt RSA certificates (`ISRG Root X1` chain), and hot-reloads Nginx (`nginx -s reload`) without dropping active map sessions.
  * Includes an automated verification script (`scripts/healthcheck.sh`) validating HTTP 200 responses, CORS headers, `application/x-protobuf` vector delivery, and `image/png` raster fallbacks.

---

## 4. Infrastructure & Endpoint Matrix

| Subsystem / Endpoint | Method & Path | Content-Type / Output | Primary Purpose |
| :--- | :--- | :--- | :--- |
| **Vector Style Specification** | `GET /styles/positron-custom/style.json` | `application/json` | MapLibre GL / WebGL style definition with CORS (`*`) |
| **Protobuf Vector Tiles** | `GET /data/v3/{z}/{x}/{y}.pbf` | `application/x-protobuf` | Gzip-compressed vector geometry rendered at 60 FPS on client GPU |
| **Retina Raster Fallback** | `GET /styles/positron-custom/{z}/{x}/{y}@2x.png` | `image/png` | Server-rendered `512px` high-DPI tiles for standard `L.tileLayer` |
| **Point-to-Point Routing** | `GET /api/route/route/v1/driving/{lon},{lat};...` | `application/json` (GeoJSON) | Street-level trajectory, metric distance, duration & turn maneuvers |
| **$N \times M$ Distance Matrix** | `GET /api/route/table/v1/driving/{lon},{lat};...` | `application/json` | Asymmetric travel time and distance matrix across multiple stops |
| **VRPTW Fleet Optimizer** | `POST /api/optimize` | `application/json` | Multi-vehicle route sequencing with time windows, capacities & breaks |
| **Diagnostic Inspector** | `GET /demo` & `GET /health` | `text/html` / `application/json` | Live WebGL/Raster tester, tile coordinate auditor & liveness probe |

---

## 5. Key Engineering Outcomes & Metrics

* **Over 80% Bandwidth Reduction:** Replacing raster PNG tile streams with Gzip-compressed `.pbf` Vector Tiles and 30-day immutable browser caching drastically reduced network egress and accelerated map rendering to 60 FPS via WebGL.
* **$0/Month Cloud API Cost:** Running `TileServer GL`, `OSRM`, and `VROOM` 100% on-premise eliminated recurring commercial geocoding, matrix, and routing API bills across thousands of daily shipments.
* **High-Capacity Fleet Optimization:** Capable of optimizing up to **200 heterogeneous vehicles and 1,000 delivery/pickup stops** in a single batch while enforcing real-world Mexican street topology, time windows, and driver rest regulations.
