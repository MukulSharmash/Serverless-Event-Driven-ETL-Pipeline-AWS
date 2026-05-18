# Serverless Event-Driven ETL Pipeline (AWS)

An automated, cloud-native, and completely serverless data engineering pipeline designed to ingest, flatten, transform, and optimize transactional JSON data in real-time. 

## 📌 Problem Statement & Business Impact
In production enterprise environments, upstream application systems continuously emit transactional data in semi-structured JSON formats. Storing these raw, uncompressed text payloads directly inside a data lake degrades down-stream analytical query engines (like Amazon Athena), resulting in slow dashboards and massive cloud data-scanning costs ($5/TB scanned). 

**The Solution:** This architecture implements a zero-infrastructure footprint pattern that intercepts incoming JSON objects the millisecond they land, dynamically flattens nested schemas, serializes the data in-memory, and writes it out as compressed, columnar Apache Parquet. This cuts analytical compute and storage costs by up to 90% while significantly increasing query performance.

---

## 🏗️ Architecture Diagram

```mermaid
graph LR
    %% Ingestion Layer
    Source[Incoming Order: JSON Data]:::source --> |Data Landing| S3Raw[S3 Bucket: /orders_json_incoming]:::storage

    %% Compute & Orchestration Layer
    subgraph ComputeLayer ["AWS Lambda | Python Processor (In-Memory Processing)"]
        direction TB
        Lambda[AWS Lambda Function]:::script
        Lambda -.-> |Extract Runtime Params: event| Lambda
        Lambda -.-> |Execute Memory Buffering: PyArrow| Lambda
    end

    %% S3 Event Notification
    S3Raw --> |Asynchronous S3 Event Trigger| Lambda

    %% Storage & Target Layer
    Lambda --> |to_parquet via io.BytesIO| StagingDB[Parquet Serialization]:::db
    StagingDB --> |S3: put_object| S3Staging[S3 Staging: /order-output]:::storage

    %% Cataloging & Governance
    subgraph MetadataManagement ["Metadata Management & Cataloging"]
        Crawler[Glue Crawler: mukul-sharma-aws-project-crawler]:::crawler
        Catalog[Glue Data Catalog]:::db
        Crawler --> |Update Schema / Partitions| Catalog
    end

    %% Boto3 SDK Control Loop (Fixed with Quotes)
    Lambda --> |"boto3.client('glue').start_crawler()"| Crawler

    %% Analytical Layer
    S3Staging --> |Query Columnar Blocks| Athena[Amazon Athena: Serverless SQL Analysis]:::viz
    Catalog --> |Schema Definition & Location| Athena

    %% --- STYLING (Professional Cloud Palette) ---
    classDef source fill:#f9f,stroke:#333,stroke-width:2px;
    classDef script fill:#C5A3FF,stroke:#333,stroke-width:2px;
    classDef storage fill:#FF9966,stroke:#333,stroke-width:2px;
    classDef db fill:#87CEEB,stroke:#333,stroke-width:2px;
    classDef crawler fill:#66CCCC,stroke:#333,stroke-width:2px;
    classDef viz fill:#FFCC00,stroke:#333,stroke-width:2px,color:black;
    classDef db fill:#87CEEB,stroke:#333,stroke-width:2px;
    classDef crawler fill:#66CCCC,stroke:#333,stroke-width:2px;
    classDef viz fill:#FFCC00,stroke:#333,stroke-width:2px,color:black;
