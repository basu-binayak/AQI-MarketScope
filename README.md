
# 🌫️ AQI-MarketScope

**A Data-Driven Market & Pollution Analytics Framework for Air Purifier Product Strategy in India**

---

## 📌 Project Summary

**AirPure Innovations** needs data-backed answers before committing to production and R&D for air purifiers. AQI-MarketScope builds a repeatable Fabric pipeline and analytics stack to:

* Identify key pollutants by city/state
* Recommend essential purifier features (filter type, CADR etc.)
* Estimate city- and state-level market size and demand
* Align R&D with localized pollution patterns

This project uses **Microsoft Fabric**, **Lakehouse (Bronze)**, **Dataflow Gen2 (Silver & Gold)**, a **Fabric Warehouse (T-SQL)**, and **Power BI (semantic model + reports)**.

---

## 🔎 Problem Statement (concise)

India faces severe air quality issues (14 Indian cities among the world’s worst). AirPure Innovations must answer:

1. Which pollutants should the purifier target?
2. What features are essential?
3. Which cities/states have highest and sustained demand — what is the market size?
4. How should R&D be tailored to localized pollution profiles?

---

## 📚 Data Sources & Metadata (summary)

The repository contains metadata for these source tables:

1. **day-wise-state-wise-air-quality-index-aqi-of-major-cities-and-towns-in-india**

   * Daily AQI, prominent pollutants, monitoring stations, status (Good → Severe), date, area, state, unit, note.

2. **master-data-state-district-and-disease-wise-cases-and-death-reported-due-to-outbreak-of-diseases-as-per-weekly-reports-under-idsp**

   * Weekly outbreak records for correlation with respiratory illness: year, week, state, district, disease, cases, deaths, etc.

3. **master-data-state-vehicle-class-and-fuel-type-wise-total-number-of-vehicles-registered-in-each-month-in-india**

   * Vehicle registrations by class & fuel to estimate vehicular-emissions impact.

4. **population_projection**

   * State-level demographic projections for TAM/SAM/SOM calculations (in thousands).

> Full field descriptions are included under `/metadata/` as separate `.md` files.

---

## 🏗️ Architecture & Dataflow

**High-level flow:**

1. **Bronze (Lakehouse)** — Raw ingestion

   * Raw CSVs / source files landed into the Fabric Lakehouse (Bronze) with schema-on-read.
   * Minimal processing: validate, preserve original files, maintain lineage.

2. **Silver (Dataflow Gen2)** — Conformance & cleansing

   * **All Silver transformations are built in Dataflow Gen2.**
   * Tasks: normalize names, parse dates, standardize pollutant columns, deduplicate, map units, enrich with population/geo metadata, create conformed dimension staging tables.
   * Silver outputs stored as curated Lakehouse tables (or parquet) for reuse.

3. **Gold (Dataflow Gen2 → Warehouse)** — Dimensional modelling & aggregates

   * **All Gold transformations are also implemented in Dataflow Gen2** (no manual notebooks).
   * Build final conformed `dim_*` and `fact_*` tables following dimensional modelling best practices:

     * `dim_date`, `dim_state`, `dim_city`, `dim_pollutant`, `dim_vehicle`, `dim_population`, `dim_disease`
     * `fact_aqi_daily_snapshot`, `fact_vehicle_registrations_monthly`, `fact_population_projection`, `fact_disease_outbreak_weekly`
   * Materialize Gold tables to the Fabric Warehouse for low-latency T-SQL analytics and semantic modeling.

4. **Analytics (Warehouse)** — T-SQL queries

   * Use T-SQL in the Fabric Warehouse to compute: pollutant dominance, trend analysis, demand index, correlation with disease, vehicle-emissions impact, seasonal patterns.

5. **Semantic Model & Power BI**

   * Create semantic model from Warehouse tables; connect Power BI Desktop to Fabric semantic model; author dashboards (Top cities, Pollutant mix, Feature recommendations, Market size).
   * Publish to Fabric Power BI Service for stakeholders.

---

## 🧩 Key Design Principles

* **Reproducibility:** Dataflow Gen2 as single source of truth for Silver & Gold transformations.
* **Separation of concerns:** Bronze = raw; Silver = clean & conformed; Gold = business-ready & modelled.
* **Dimensional modeling:** Star schema for efficient analytics and BI.
* **Traceability:** Maintain lineage from source files through dataflows to Gold tables.
* **Automation:** Dataflows scheduled/triggered in Fabric.
* **Governance:** Row-level lineage, units & notes preserved in metadata.

---

## 🗂️ Repository Structure (corrected)

```
AQI-MarketScope/
│
├── /metadata/                         # Detailed metadata files (.md) for each source
│   ├── aqi_metadata.md
│   ├── disease_metadata.md
│   ├── vehicle_metadata.md
│   └── population_metadata.md
│
├── /bronze/                           # Sample/instructional scripts for ingestion (CLI / Fabric orchestration notes)
│   ├── README_BRONZE.md
│   └── sample_ingest_manifest.json
│
├── /dataflows/                        # Dataflow Gen2 artifacts (JSON / definitions / templates)
│   ├── silver/
│   │   ├── df_silver_aqi.json
│   │   ├── df_silver_vehicle.json
│   │   └── df_silver_population.json
│   └── gold/
│       ├── df_gold_dims.json
│       └── df_gold_facts.json
│
├── /warehouse/                        # SQL scripts for creating external schemas and validating Gold tables
│   ├── create_warehouse_schema.sql
│   ├── dim_model_creation.sql
│   └── analytical_queries.sql
│
├── /powerbi/                          # Power BI Connectors / PBIX templates & report notes
│   ├── README_POWERBI.md
│   └── report_template.pbix
│
├── /docs/                             # Design docs, ERD, architecture diagrams (images / md)
│   ├── architecture_diagram.png
│   └── dimensional_model.md
│
├── .github/
│   └── workflows/                      # Optional: CI or deployment scripts for pushing dataflow artifacts
│
└── README.md                          # <- this file
```

> Note: Dataflow JSONs in `/dataflows/` are templates — import them into Fabric Dataflows Gen2 and parameterize (e.g., source paths, schedules, credentials).

---

## 🚀 How to Run / Reproduce (high-level)

1. **Clone repo**

```bash
git clone https://github.com/<your-username>/AQI-MarketScope.git
cd AQI-MarketScope
```

2. **Create Fabric Lakehouse (Bronze)**

   * Create a Lakehouse in Microsoft Fabric.
   * Upload source files into a Bronze folder/container per the `bronze/README_BRONZE.md` instructions.

3. **Import Dataflow Gen2 artifacts (Silver)**

   * In Fabric, create Dataflow Gen2 and import `/dataflows/silver/*.json`.
   * Set data source parameters (Lakehouse Bronze paths) and schedule or trigger runs.

4. **Import Dataflow Gen2 artifacts (Gold)**

   * Create Dataflow Gen2 for Gold transformations using `/dataflows/gold/*.json`.
   * Configure outputs to write Gold tables into the Fabric Warehouse.

5. **Create Warehouse & Run T-SQL**

   * Run `warehouse/create_warehouse_schema.sql` to create a presentation schema and table pointers.
   * Validate with sample queries in `warehouse/analytical_queries.sql`.

6. **Power BI Semantic Model & Reports**

   * Connect Power BI Desktop to the Fabric semantic model or Warehouse.
   * Use `powerbi/report_template.pbix` as a starting point; update visuals, filters, and publish to Fabric Power BI Service.

7. **Schedule & Monitor**

   * Schedule Dataflows Gen2 runs; set alerts and lineage checks in Fabric.

---

## ✅ Primary & Secondary Analysis Examples (T-SQL ideas)

* **Primary:**

  * `Top pollutants by city/state (last 12 months)`
  * `Cities with > X days/year in Very Poor/Severe AQI`
  * `Market Demand Index = Population * PercentDaysPoor * AffordabilityFactor`

* **Secondary:**

  * `Correlation(AQI PM2.5, Respiratory_Disease_Cases)`
  * `Vehicle registrations growth vs AQI trend`
  * `Seasonal pollutant heatmaps (monsoon vs winter)`

(Sample query templates in `/warehouse/analytical_queries.sql`)

---

## 📈 Expected Deliverables

* Bronze raw tables in Lakehouse
* Silver conformed tables (Dataflow Gen2 outputs)
* Gold dimensional model tables (Dataflow Gen2 outputs → Warehouse)
* T-SQL analytical queries & notebooks for validation
* Power BI semantic model + dashboards published to Fabric Power BI Service
* Final Insight Report summarizing recommendations for AirPure Innovations

---

## 🛠️ Notes for Implementation

* **Dataflows Gen2 tips:** keep transformations modular; create re-usable mapping functions for pollutant parsing and unit conversion.
* **Testing:** build small incremental dataflow runs with sample Bronze datasets before running full refresh.
* **Lineage:** annotate dataflows with clear descriptions and capture provenance in Gold tables (e.g., `source_file`, `ingest_timestamp`).
* **Parameterize:** use environment variables for storage/paths to enable CI/CD deployment of dataflows.

---

