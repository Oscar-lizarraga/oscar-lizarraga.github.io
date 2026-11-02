---
layout: default
title: "SIAP: Distributed Multi-Service Logistics Platform"
---

# SIAP: Distributed Multi-Service Logistics & Last-Mile Delivery Platform

**Published:** November 2026 
**Role:** Lead Backend Architect & Systems Engineer 
**Domain:** Supply Chain, Last-Mile Logistics & Electronic Proof of Delivery (ePOD) 
**Stack:** .NET 8 Core (C#), Golang, Python (Flask & FastAPI), PostgreSQL, Azure Cosmos DB, MongoDB, Azure Blob Storage, Docker, Nginx 

---

## Executive Summary

Modern supply chain and last-mile logistics networks require real-time visibility, high-throughput event processing, and instantaneous generation of legally binding digital delivery receipts (**ePOD - Electronic Proof of Delivery**). Field drivers submit thousands of package movements, signatures, and photographic evidences concurrently from mobile devices across variable cellular networks.

To address these challenges at enterprise scale, I architected and engineered **SIAP** (*Comprehensive Logistics & Parcel Administration System*): a resilient, polyglot microservices ecosystem. The platform decouples core transactional ERP functions, real-time driver telemetry streaming, and asynchronous document rendering into purpose-built services using **.NET 8 Core**, **Golang**, and **Python**, operating over a **hybrid persistence layer (PostgreSQL, Azure Cosmos DB, MongoDB, and Azure Blob Storage)**.

The system processes real-time delivery events via database Change Streams, queues them atomically, and generates digitally signed, high-resolution ePOD documents with **zero operational downtime and sub-second dispatch latency**.

---

## System Architecture & Polyglot Microservices

The architecture follows a distributed, event-driven pattern behind an **Nginx API Gateway & Reverse Proxy**, decoupling read-heavy web monitoring from compute-intensive PDF rendering and low-latency transactional ledgering.

```mermaid
graph TB
 %% Clients Layer
 subgraph CLIENTS ["Clients & Edge Devices"]
 MobileApp["Driver Mobile App<br/><i>Flutter / Android</i>"]
 WebOps["Operations & Fleet Dispatchers<br/><i>Responsive Web Browser</i>"]
 ExternalClients["External B2B API Consumers<br/><i>REST Clients / Carriers</i>"]
 end

 %% Gateway Layer
 subgraph GATEWAY ["Gateway & Security Proxy"]
 NginxGateway["Nginx Reverse Proxy<br/><i>SSL / Rate Limiting / Path-Based Routing</i>"]
 end

 %% Microservices Layer
 subgraph SERVICES ["Polyglot Microservices Layer"]
 DotNetCore["Core ERP & Shipping API (.NET 8 Core)<br/>• Shipment & Batch Manifests<br/>• Multi-carrier Quoting (DHL/FedEx)<br/>• Relational Dapper ORM Engine"]
 
 GoDaemon["Event-Stream Daemon (Golang 1.22)<br/>• 24/7 Native Cosmos Change Stream Consumer<br/>• Atomic Job Producer for Task Queues<br/>• Sub-millisecond State Synchronization"]
 
 FlaskVisor["Operations Portal & Worker (Python / Flask)<br/>• Fleet Monitoring & Route Dispatch Dashboard<br/>• Asynchronous ePOD Queue Consumer<br/>• Headless Document Engine"]
 
 PrintService["Thermal Print Microservice (FastAPI)<br/>• ZPL & PDF Shipping Label Generator<br/>• Headless Browser Rendering"]
 end

 %% Hybrid Persistence Layer
 subgraph PERSISTENCE ["Hybrid Persistence & Cloud Storage"]
 PostgresDB[("PostgreSQL 15<br/><b>Relational Ledger</b><br/>• Shipments & Manifests<br/>• Projects, Routes & Customers")]
 
 CosmosStream[("Azure Cosmos DB<br/><b>NoSQL Event Store</b><br/>• Raw Driver Deliveries<br/>• Real-time Change Streams")]
 
 MongoQueue[("MongoDB<br/><b>Distributed Task Queue</b><br/>• Atomic ePOD Tasks<br/>• Operational Guide States<br/>• Dead Letter Queue (DLQ)")]
 
 AzureBlobs[("Azure Blob Storage<br/>• /evidences/ (Signatures & Photos)<br/>• /epod/ (SAS-Signed PDF Proofs)<br/>• /labels/ (Carrier Thermal Labels)")]
 end

 %% Interactions
 MobileApp -->|"1. Delivery photos & signatures"| CosmosStream
 WebOps -->|"HTTPS"| NginxGateway
 ExternalClients -->|"HTTPS"| NginxGateway
 
 NginxGateway -->|"/api/v1/shipments"| DotNetCore
 NginxGateway -->|"/visor & /epod"| FlaskVisor
 NginxGateway -->|"/api/print"| PrintService
 
 CosmosStream -.->|"Change Stream in real time"| GoDaemon
 GoDaemon -->|"2a. Enqueue atomic ePOD task"| MongoQueue
 GoDaemon -->|"2b. Sync status & timestamps"| PostgresDB
 
 FlaskVisor -->|"3. Pop atomic task"| MongoQueue
 FlaskVisor -->|"4. Query metadata"| DotNetCore
 FlaskVisor -->|"5. Fetch photos & upload ePOD"| AzureBlobs
 
 DotNetCore -->|"ACID Transactions"| PostgresDB
 DotNetCore -->|"Request thermal label"| PrintService
 PrintService -->|"Store label assets"| AzureBlobs
```

---

## Asynchronous Electronic Proof of Delivery (ePOD) Pipeline

The generation of delivery receipts is compute-heavy (image resizing, cryptographic layout, barcode creation, and PDF compilation). Executing this synchronously on mobile check-in would exhaust driver bandwidth and degrade API responsiveness.

```mermaid
sequenceDiagram
 autonumber
 actor Driver as Driver (Mobile App)
 participant Cosmos as Azure Cosmos DB
 participant GoDaemon as Go Daemon (Stream Worker)
 participant PG as PostgreSQL (Master DB)
 participant Queue as MongoDB Task Queue
 participant EpodWorker as Python ePOD Worker
 participant Azure as Azure Blob Storage
 participant DLQ as Dead Letter Queue (DLQ)

 Driver->>Cosmos: Submit Delivery (Geo-location, Signature, 2-4 Photos)
 Cosmos-->>GoDaemon: Emit Real-Time Change Stream Event
 par Concurrent Ingestion
 GoDaemon->>PG: Upsert Delivery Status (Timestamp, Recipient)
 and Queue Scheduling
 GoDaemon->>Queue: Push Atomic Task (packagelog.EpodTasks: PENDING)
 end
 
 Note over EpodWorker,Queue: Distributed Polling with Atomic Locking
 EpodWorker->>Queue: Acquire Task (findAndModify: STATUS = PROCESSING)
 EpodWorker->>Azure: Download Raw Evidence Blobs (Photos & Signature)
 EpodWorker->>EpodWorker: Compile Dynamic ePOD (Headless HTML-to-PDF Engine)
 
 alt Generation Succeeded
 EpodWorker->>Azure: Upload Final Signed PDF (/epod/)
 EpodWorker->>Queue: Update Task: COMPLETED (Set SAS URL)
 EpodWorker->>Cosmos: Update Delivery Record with Permanent ePOD Link
 else Unrecoverable Missing Metadata
 EpodWorker->>DLQ: Route to Dead Letter Queue (packagelog.ErrorLogs)
 EpodWorker->>Queue: Update Task: FAILED
 end
```

---

## Delivery Lifecycle & Finite State Machine

Shipments transition through a strictly governed lifecycle ensuring full custody tracking from warehouse manifest to final recipient archival:

```mermaid
stateDiagram-v2
 [*] --> MANIFESTED : Batch Manifest Ingestion (.NET Core)
 MANIFESTED --> DEPARTED_HUB : Warehouse Dispatch (Loading Scan)
 DEPARTED_HUB --> IN_TRANSIT : Assigned to Driver Route
 IN_TRANSIT --> DELIVERED : Successful Delivery (Photos + Signature)
 IN_TRANSIT --> ATTEMPT_FAILED : Exception (Closed Premises / Refused)
 ATTEMPT_FAILED --> IN_TRANSIT : Re-dispatched on Secondary Route
 DELIVERED --> EPOD_GENERATING : Go Daemon Change Stream Trigger
 EPOD_GENERATING --> COMPLETED_ARCHIVED : PDF Rendered & Synced to Azure
 MANIFESTED --> CANCELLED : Operational Cancellation
 COMPLETED_ARCHIVED --> [*]
 CANCELLED --> [*]
```

---

## Microservice Deep-Dive & Responsibilities

### 1. `siap-backend` — Enterprise ERP Core (.NET 8 Core / C# / Dapper)
* **High-Performance Data Access:** Uses Dapper micro-ORM over PostgreSQL to achieve sub-millisecond query execution on millions of shipment records.
* **Multi-Carrier & Manifest Hub:** Implements rating algorithms, consignment batching, client authentication, and role-based access control (RBAC).
* **Enterprise Modularity:** Clean architecture separating controller layer, domain entities, and data access repositories.

### 2. `rigel-to-packagelog` — Ultra-Fast Stream Daemon (Golang 1.22)
* **Bare-Metal Concurrency:** Written in Go to achieve near-zero memory footprint and non-stop 24/7 uptime.
* **Cosmos DB Change Stream Listener:** Continuously watches driver check-ins in Azure Cosmos DB and bridges NoSQL driver telemetry into the relational PostgreSQL ledger.
* **Atomic Task Dispatcher:** Produces atomic job payloads into MongoDB `EpodTasks`, acting as the primary buffer between edge events and document rendering workers.

### 3. `pagina-visor` & `siap-epod-worker` — Dispatch Portal & Document Engine (Python / Flask)
* **Real-Time Operational Fleet Monitor:** Live dashboard for route tracking, odometer/fuel auditing, and courier delivery efficiency KPIs.
* **Headless Document Synthesis:** Dedicated worker pool using headless Chromium to compile vector-sharp, print-ready PDF delivery proofs embedded with barcodes, timestamps, GPS coordinates, recipient signatures, and delivery photos.

### 4. `print-service` — Thermal Label Service (FastAPI / Playwright)
* **High-Speed Label Generation:** Provides microsecond thermal barcode label rendering supporting standard ZPL (Zebra Programming Language) and vector PDF outputs for industrial warehouse printers.

---

## Hybrid Polyglot Persistence Strategy

| Storage Engine | Technology | Role & Domain | Why Chosen? |
| :--- | :--- | :--- | :--- |
| **Relational Core** | **PostgreSQL 15** | Manifests, customer accounts, financial ledgers, audit logs | Strict ACID compliance, relational integrity, relational indexing |
| **NoSQL Mobile Hub** | **Azure Cosmos DB** | Driver mobile telemetry, raw check-in documents | Globally scalable document ingestion, native real-time Change Streams |
| **Operational Queue** | **MongoDB 7.0** | Atomic task queues (`EpodTasks`), active routes, Dead Letter Queue | Fast atomic `findAndModify` locks, flexible operational schema |
| **Blob Storage** | **Azure Blob Storage** | High-resolution delivery photos, recipient signatures, compiled ePOD PDFs | Tiered cloud storage, secure time-limited Shared Access Signatures (SAS) |

---

## DevOps, Security & Deployment Architecture

* **Container Orchestration:** 100% Dockerized multi-stage environment with Alpine Linux base images, reducing production image footprints from >800MB to **~25–60MB**.
* **Zero-Downtime Pipeline:** Automated deployment scripts (`deploy.sh`) combining remote streaming compilation, zero-orphan container restarts, and safe database seed migrations.
* **Security & Defense-in-Depth:**
 * Reverse proxy enforcement via **Nginx** (SSL/TLS termination, request size throttling, and secure HTTP headers).
 * Storage isolation using ephemeral Azure **SAS tokens** (preventing direct public exposure of customer signature images).
 * Comprehensive Dead-Letter-Queue (DLQ) pattern preventing poisoned messages from halting background worker queues.

---

## Engineering Achievements & Metrics

* **Zero Event Drop Rate:** Decoupled Change Stream architecture guarantees that 100% of driver check-ins are recorded even during peak warehouse arrival hours.
* **Sub-Second Processing:** Asynchronous ePOD compilation finishes in under **2.4 seconds** per package, while the driver app receives an instant confirmation in **<200ms**.
* **High Operational Scalability:** Easily scales horizontal worker replicas during holiday retail spikes without altering master relational schema.
* **Full Traceability:** Every package maintains complete provenance from bulk consignment generation to legal proof-of-delivery archival.
