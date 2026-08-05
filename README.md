
# Maji Ndogo: Comprehensive Water Accessibility, Contamination & Crime Insights Analysis

> **Note:** Power BI (`.pbix`) files cannot be rendered interactively directly inside GitHub. This documentation serves as the executive summary, technical showcase, data pipeline overview, and architectural blueprint for this project.

---

## Executive Summary & Background

**Maji Ndogo**, a nation of over 27,000 surveyed water access points, faces a complex water crisis compounded by severe queue times, unsafe biological/chemical contamination, and public safety risks at water collection sites. 

This project delivers an end-to-end data engineering and business intelligence solution designed to:
1. **Identify bottlenecks** in water access and queue times across provinces.
2. **Classify water quality & contamination risks** across thousands of wells to prioritize purification infrastructure.
3. **Analyze public safety & crime incidents** near collection points to optimize law enforcement and community intervention.

---

## Data Architecture & Relational Model

The project integrates relational data originating from a **MySQL** database into **Power BI Desktop**, structured around a Star Schema data model.

### Entity Relationship Diagram (ERD)

```text
       +------------------+          +------------------+
       |     location     | --------<|      visits      |
       +------------------+          +------------------+
                |                             |
                |                             |
       +------------------+                   v
       |  well_pollution  | ----------------> +------------------+
       +------------------+                   |   water_source   |
                                              +------------------+
       +------------------+                            ^
       |  crime_incidents | ---------------------------+
       +------------------+
```

### Table Dictionary
* `location`: Geographical attributes (`province_name`, `town_name`, `location_type`).
* `water_source`: Water point metadata (`source_id`, `type_of_water_source`, number of people served).
* `visits`: Field survey logs (`visit_count`, `time_in_queue`, timestamp).
* `well_pollution`: Water sample test results (`results`, `pollutant_ppm`, `biological`).
* `crime_incidents`: Public safety records logged near collection points (`crime_type`, `victim_gender`, `Hour_of_Day`, `Day_Name`).

---

## Data Pipeline, ETL & DAX Logic

### 1. SQL Data Extraction & Aggregation
Core queries executed in MySQL for initial data exploration and pipeline validation:

```sql
-- Identifying high-queue areas and contamination status across provinces
SELECT 
    l.province_name,
    l.town_name,
    ws.type_of_water_source,
    AVG(v.time_in_queue) AS avg_queue_time_mins,
    wp.results AS contamination_results
FROM visits v
JOIN location l ON v.location_id = l.location_id
JOIN water_source ws ON v.source_id = ws.source_id
LEFT JOIN well_pollution wp ON ws.source_id = wp.source_id
WHERE v.time_in_queue > 120
GROUP BY 1, 2, 3, 5
ORDER BY avg_queue_time_mins DESC;
```

### 2. Power Query (M) & Data Standardization
* **String Standardization:** Trimmed hidden whitespaces and cleaned unmapped strings across `results` and `victim_gender`.
* **Value Replacement:** Transformed technical database codes into human-readable labels (`harassment` $
ightarrow$ `Harassment`, `F` $
ightarrow$ `Female`, `M` $
ightarrow$ `Male`, `C` $
ightarrow$ `Child`).

### 3. Key DAX Calculations

#### Contamination Status Classification (Calculated Column)
```dax
Contamination Status = 
VAR RawResult = TRIM(Well_Pollution[results])
RETURN
SWITCH(
    TRUE(),
    RawResult = "Clean" || ISBLANK(RawResult) || RawResult = "", "Clean / Uncontaminated",
    RawResult = "Contaminated: Chemical", "Chemical Contamination",
    RawResult = "Contaminated: Biological", "Biological Contamination",
    "Clean / Uncontaminated"
)
```

#### Victim Gender Standardization (Calculated Column)
```dax
Victim Gender Full = 
SWITCH(
    UPPER(TRIM(Crime_Incidents[victim_gender])),
    "F", "Female",
    "M", "Male",
    "C", "Child",
    "Other / Unknown"
)
```

#### Explicit Measure Aggregations
```dax
Total Crime Incidents = COUNT(Crime_Incidents[crime_id])

Total Biological Contamination = SUM(Well_Pollution[biological])
```

---

## Key Findings & Strategic Insights

### 1. Water Contamination & Safety
* **Contamination Breakdown:** Approximately **50.91%** of surveyed wells test completely **Clean / Uncontaminated**, **27.93%** suffer from **Chemical Contamination**, and **21.16%** suffer from **Biological Contamination**.
* **High-Risk Regions:** Sokoto and Hawassa provinces exhibit critical biological contamination loads requiring immediate installation of UV filtration systems and chlorine dosing units.

### 2. Public Safety & Crime Dynamics
* **Demographics:** Female citizens represent the largest proportion of crime victims at water collection points due to extended early morning and late evening queuing.
* **Temporal Patterns:** Incidents spike during early morning hours (5:00 AM – 8:00 AM) and evening peak queue times, highlighting where security patrols should be deployed.

---

## Required Visual Assets (Screenshots)

To showcase this project effectively on GitHub, capture and attach the following screenshots in a repository folder named `assets/`:

1. `assets/01_water_insights.png`:
   * **Content:** The **Water Insights** page displaying the *Water Source Contamination Status* donut chart and *Biological Contamination Levels by Source & Region* bar chart.
2. `assets/02_crime_insights.png`:
   * **Content:** The **Crime Insights** page displaying incident counts by crime type, hourly crime trends line chart, daily volume stacked bar chart, and demographic donut chart.
3. `assets/03_data_model.png`:
   * **Content:** Power BI Model View showing table relationships between `Water_Source`, `Well_Pollution`, `Location`, and `Crime_Incidents`.

---

## Project Structure & Setup

```text
├── assets/
│   ├── 01_water_insights.png
│   ├── 02_crime_insights.png
│   └── 03_data_model.png
├── sql/
│   └── maji_ndogo_queries.sql
├── data/
│   └── raw_data_schema.sql
├── Maji_Ndogo_Water_Insights.pbix
└── README.md
```

### How to Run
1. Clone this repository:
   ```bash
   git clone https://github.com/your-username/maji-ndogo-water-insights.git
   ```
2. Download and install [Power BI Desktop](https://powerbi.microsoft.com/desktop/).
3. Open `Maji_Ndogo_Water_Insights.pbix` to interact with the visualizations, examine the DAX measures, and review the underlying data model.
