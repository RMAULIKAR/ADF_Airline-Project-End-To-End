#  ADF Airline Data Engineering Project

##  Overview

**ADF Airline Project – End-to-End** is a cloud-based data engineering project built with **Azure Data Factory (ADF)** to ingest, transform, and prepare airline data for business analysis.

The project integrates **on-premises CSV files, GitHub API JSON data, and Azure SQL booking data**, processing them through a **Bronze → Silver → Gold Medallion Architecture**.

### Key Highlights

* Cloud-based data engineering solution built on **Microsoft Azure**
* Multi-source ingestion from **On-Premises, GitHub API, and Azure SQL Database**
* **Watermark-based incremental ingestion** for `FactBooking`
* ADF pipeline orchestration and **Mapping Data Flow** transformations
* Source-specific Bronze storage using **CSV, JSON, and Parquet**
* **Delta** storage for Silver and Gold layers
* **Fact and Dimension modeling with Star Schema**
* Business views for **Revenue by Flight** and **Revenue by Airline**

---

##  Architecture

```text
                         ┌─────────────────────┐
                         │    SOURCE SYSTEMS   │
                         └──────────┬──────────┘
                                    │
              ┌─────────────────────┼─────────────────────┐
              │                     │                     │
              ▼                     ▼                     ▼
       On-Premises CSV        GitHub API JSON        Azure SQL DB
                                                          │
                                                      Watermark
                                                      Incremental
              │                     │                     │
              └─────────────────────┼─────────────────────┘
                                    ▼
                         ┌─────────────────────┐
                         │    BRONZE LAYER     │
                         │  CSV | JSON |       │
                         │      Parquet        │
                         └──────────┬──────────┘
                                    │
                                    ▼
                         ┌─────────────────────┐
                         │    SILVER LAYER     │
                         │       Delta         │
                         │  Mapping Data Flow  │
                         └──────────┬──────────┘
                                    │
                                    ▼
                         ┌─────────────────────┐
                         │     GOLD LAYER      │
                         │       Delta         │
                         │   Business Views    │
                         └──────────┬──────────┘
                                    │
                         ┌──────────┴──────────┐
                         ▼                     ▼
                  Revenue by Flight     Revenue by Airline
```

---

## Data Sources

| Source             | Dataset                                   | Format        | Pipeline               |
| ------------------ | ----------------------------------------- | ------------- | ---------------------- |
| On-Premises        | `DimAirline`, `DimFlight`, `DimPassenger` | CSV           | `Stg_OnPremIngestion`  |
| GitHub API         | `DimAirport`                              | JSON          | `Stg_API_Ingestion`    |
| Azure SQL Database | `FactBooking`                             | SQL → Parquet | `Stg_SQLtoDataLakeNew` |

---

##  Medallion Architecture

The project follows a **Bronze → Silver → Gold** architecture.

### Bronze Layer

Raw/staging data is ingested from the source systems while retaining source-specific formats:

**On-Premises → CSV** · **GitHub API → JSON** · **Azure SQL → Parquet**

The Bronze layer separates source ingestion from downstream transformation.

### Silver Layer

Bronze data is transformed using **ADF Mapping Data Flow** with foundational operations such as:

**Select · Filter · Join · Group By · Aggregation**

The transformation uses **Inline Datasets** and stores the output as **Delta**.

### Gold Layer

Silver data is further transformed using **ADF Mapping Data Flow with Inline Datasets** and stored as **Delta** to create business-oriented views:

**Revenue by Flight · Revenue by Airline**

---

## Watermark-Based Incremental Ingestion

The `Stg_SQLtoDataLakeNew` pipeline implements **watermark-based incremental ingestion** for the `FactBooking` table.

Instead of repeatedly loading the complete source table, the watermark identifies the records required for the incremental load.

```text
Azure SQL
    ↓
FactBooking
    ↓
Watermark
    ↓
Incremental Records
    ↓
ADF Pipeline
    ↓
Bronze / Parquet
```

This reduces unnecessary data movement and avoids repeated full loads.

---

##  Data Model

The project uses a **Fact and Dimension modeling approach** following a **Star Schema**.

**Fact Table:** `FactBooking`

**Dimension Tables:** `DimAirline` · `DimAirport` · `DimFlight` · `DimPassenger`

```text
DimAirline ───────┐
DimAirport ───────┤
DimFlight ────────┼──► FactBooking
DimPassenger ─────┘
```

`FactBooking` contains transactional booking data, while the dimension tables provide descriptive attributes for analytical processing.

---

##  ADF Pipelines

| Pipeline                 | Purpose                                                                |
| ------------------------ | ---------------------------------------------------------------------- |
| `Pipeline`               | Parent pipeline for executing and orchestrating the ingestion workflow |
| `Stg_API_Ingestion`      | Ingests `DimAirport` JSON data from the GitHub API                     |
| `Stg_OnPremIngestion`    | Ingests on-premises CSV dimension files                                |
| `Stg_SQLtoDataLakeNew`   | Performs watermark-based incremental ingestion from Azure SQL          |
| `Enr_SilverLayer`        | Performs Silver-layer transformations                                  |
| `BusinessView_GoldLayer` | Performs Gold-layer transformations and creates business views         |

---

##  Technologies & Concepts

**Cloud:** Microsoft Azure · Azure Data Factory · Azure Data Lake Storage · Azure SQL Database

**Data Sources:** On-Premises CSV · GitHub API JSON · Azure SQL

**Processing:** ADF Pipelines · Mapping Data Flow · Watermark-based Incremental Ingestion

**Architecture:** Medallion Architecture · Star Schema · Fact & Dimension Modeling

**Storage Formats:** CSV · JSON · Parquet · Delta

---

##  Business Outcome

The pipeline transforms airline data from multiple sources into structured and curated datasets through the **Bronze → Silver → Gold** architecture.

The final Gold-layer business views support:

* **Revenue by Flight**
* **Revenue by Airline**

Overall, the project demonstrates an end-to-end **Azure Data Engineering workflow** covering source connectivity, multi-source ingestion, incremental loading, Data Lake storage, transformation, dimensional modeling, and business-oriented data preparation.
