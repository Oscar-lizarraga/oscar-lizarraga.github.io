---
layout: default
title: "About Me — Oscar Lizarraga | Systems & Cloud Architect"
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

# About Me — Oscar Lizarraga

<div class="project-meta-card">
  <div class="project-meta-grid">
    <div class="project-meta-item">
      <span class="project-meta-label">Role</span>
      <span class="project-meta-value">Systems &amp; Cloud Architect | Backend Engineer</span>
    </div>
    <div class="project-meta-item">
      <span class="project-meta-label">Core Focus</span>
      <span class="project-meta-value">Distributed Systems, Enterprise Logistics &amp; Cloud Infrastructure</span>
    </div>
    <div class="project-meta-item full-width">
      <span class="project-meta-label">Executive Summary</span>
      <span class="project-meta-value">Specialized in designing high-throughput distributed systems, event-driven microservices, and enterprise logistics platforms. Proven track record building end-to-end mission-critical infrastructure combining .NET 8, Golang, Python, and Azure: from metaheuristic fleet routing engines (VROOM / OSRM) and real-time telemetry streaming (Cosmos DB Change Streams), to offline-first mobile logistics (MAUI / CameraX) and self-healing ETL pipelines with Applied AI. Focused on algorithmic efficiency, zero-downtime resilience, and enterprise scalability.</span>
    </div>
  </div>
  <div class="project-meta-item">
    <span class="project-meta-label">Primary Ecosystem</span>
    <div class="project-stack-list">
      <span class="tech-badge">.NET 8 Core (C#)</span>
      <span class="tech-badge">Golang</span>
      <span class="tech-badge">Python (FastAPI &amp; Flask)</span>
      <span class="tech-badge">Microsoft Azure</span>
      <span class="tech-badge">PostgreSQL &amp; SQL Server</span>
      <span class="tech-badge">Cosmos DB &amp; MongoDB</span>
      <span class="tech-badge">Docker &amp; Nginx</span>
      <span class="tech-badge">VROOM &amp; OSRM</span>
    </div>
  </div>
</div>

---

## 1. Languages & Application Frameworks

- **C# & .NET Ecosystem**:
  - **.NET 8 Core & ASP.NET Core Web API**: RESTful microservices, stateless JWT/OAuth2 Bearer authentication, OpenAPI/Swagger specifications, and high-throughput data services.
  - **Blazor (Interactive Server & WebAssembly PWA)**: Enterprise single-page applications, role-based access governance, and interactive operational dashboards (Radzen UI).
  - **.NET MAUI Blazor Hybrid (Profiled AOT)**: Offline-first mobile applications for field operations with local SQLite persistence and background synchronization.
  - **Data Access & Document Synthesis**: Dapper (high-performance stored procedure & function execution), Entity Framework Core, DbUp automated schema migrations, QuestPDF (real-time vector PDF dossier generation), ClosedXML (in-memory OpenXML Excel synthesis), and SixLabors.ImageSharp.
- **Python**:
  - **FastAPI & Flask**: Asynchronous operational gateways, analytical web portals, and AI tool execution endpoints.
  - **Headless Automation & Data Processing**: Playwright + Headless Chromium (automated ePOD PDF compilation), Pandas, and NumPy.
- **Go (Golang)**:
  - High-concurrency background daemons, binary streaming ETL pipelines (`pgx/v5` `CopyFrom` protocol), Change Stream consumers, and low-footprint containerized workers.
- **Android Native & Computer Vision Integration**:
  - Custom AndroidX **CameraX** hardware engines, **Google ML Kit** (Document Scanner & OCR), single-flight GPS telemetry pre-warming, and SHA-256 idempotent media synchronization.
- **Scripting & Web**:
  - JavaScript (ES6+, client-side network resilience interceptors), T-SQL / PL/pgSQL, and Linux Bash automation (Blue/Green zero-downtime deployment pipelines).

---

## 2. Geospatial Infrastructure, Routing & Operations Research

- **VROOM (Vehicle Routing Open-Source Optimization Machine)**:
  - Metaheuristic solver implementation for Capacitated Vehicle Routing Problems with Time Windows (**CVRPTW**), optimizing multi-stop sequences and fleet capacity utilization.
- **OSRM (Open Source Routing Machine)**:
  - Self-hosted street-level routing engines and high-speed $N \times M$ distance/duration matrix computation across national road networks.
- **Vector Cartography & WebGL Rendering**:
  - **TileServer GL** serving OpenMapTiles `.pbf` Vector Tiles and **MapLibre GL JS** for 60 FPS GPU-accelerated map rendering.
  - **Leaflet** ecosystem (`MarkerCluster`, Heatmaps, `Routing Machine`, `PolylineDecorator`) for real-time fleet tracking and delivery density visualization.
- **Google Maps Platform**:
  - Places API and Geocoding API integration with automated address normalization and caching.

---

## 3. Cloud Architecture, DevOps & Database Systems

- **Microsoft Azure**:
  - **Azure Cosmos DB**: Real-time mobile telemetry ingestion and Change Stream event processing.
  - **Azure Blob Storage**: Direct-to-cloud BlockBlob streaming and cryptographic Shared Access Signature (**SAS**) token generation.
  - **Azure Service Bus, Azure Functions & Bicep (IaC)**: Event-driven messaging and repeatable cloud infrastructure provisioning.
- **Relational & NoSQL Databases**:
  - **PostgreSQL (v15 / v16)**: Complex relational modeling (3NF), `JSONB` document indexing, spatial queries, and transactional stored functions.
  - **Microsoft SQL Server**: Distributed containerized databases, ACID transactional ledgers, and pre-compiled stored procedure architectures.
  - **MongoDB & SQLite**: Atomic distributed job queues (`find_one_and_update` locking with stale-lock recovery) and resilient embedded mobile storage (`SQLite WAL`).
- **Containers, Reverse Proxy & Zero-Downtime Releases**:
  - **Docker & Docker Compose**: Multi-container microservice orchestration and resource isolation.
  - **Nginx**: API Gateway, reverse proxy, upstream failover (`proxy_next_upstream`), and **Ephemeral Blue/Green Zero-Downtime Deployment** pipelines with connection draining.
  - **OAuth2-Proxy & Microsoft Entra ID**: Centralized single sign-on (SSO) and identity federation.
- **Observability & Telemetry Stack**:
  - **Prometheus, Grafana, Loki, Promtail, cAdvisor & Node Exporter** for real-time container metrics, centralized log aggregation, and host health monitoring.

---

## 4. Enterprise APIs, Applied AI & Regulatory Integrations

- **IoT & Fleet Telematics**:
  - **Samsara Fleet API**: Real-time commercial vehicle GPS tracking, ignition states, and route progress correlation.
- **Enterprise Cloud Collaboration**:
  - **Microsoft Graph API v1.0**: Automated corporate SharePoint and OneDrive folder provisioning and in-place Excel manifest synchronization via Microsoft Entra ID (OAuth2).
- **Applied AI & Self-Healing Data Pipelines**:
  - **Azure AI Foundry / OpenAI LLMs**: Cognitive fallback agents for autonomous schema-drift diagnosis, in-memory spreadsheet remediation, and natural-language operational tool gateways.
- **Logistics, Messaging & Fiscal Compliance**:
  - **DHL Courier Tracking API** & **Firebase Cloud Messaging (FCM)** for silent push route synchronization.
  - **SAT CFDI 4.0 / Complemento Carta Porte**: Digital cryptographic stamping workflows and Electronic Proof of Delivery (**ePOD**) compliance.

---

## 5. Connect

- **LinkedIn**: [Oscar Lizarraga Pelayo](https://www.linkedin.com/in/oscar-lizarraga-pelayo-8051a1215/)
- **Portfolio**: [Back to Main Portfolio](../index.md)
