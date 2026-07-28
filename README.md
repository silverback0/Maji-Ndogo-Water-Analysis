# Maji Ndogo: Water Accessibility Crisis Analysis

> Note: Power BI (.pbix) files cannot be rendered interactively directly inside GitHub. This README serves as the primary documentation, visual showcase, and architectural overview for this project.

---

## Project Overview

**Maji Ndogo**, a fictional nation facing severe water accessibility challenges, required a data-driven approach to map, analyze, and resolve its ongoing crisis. Millions of citizens face long wait times, contaminated water sources, and degraded infrastructure. 

The goal of this project is to digest complex relational data regarding water points, survey records, and citizen feedback into actionable insights. By using **SQL** for data extraction/transformation and **Power BI** for interactive visualization, this project provides decision-makers with a roadmap to optimize resource allocation and improve water access.

---

## Data Architecture & Pipeline

### 1. Database Structure & Entity Relationships
The underlying data originates from a structured **MySQL** relational database containing several key tables:

* `location`: Geographical data (region, district, local government area).
* `water_source`: Types of water points (wells, shared taps, river water, etc.) and their operational status.
* `visits`: Log of field survey visits, record times, and queue duration.
* `well_pollution`: Water quality metrics (e.g., biological/chemical contamination levels).
* `infrastructure_cost`: Financial estimates for repairs and upgrades.

```text
       +------------------+          +------------------+
       |     location     | --------<|      visits      |
       +------------------+          +------------------+
                                               |
                                               |
       +------------------+                    |
       |  well_pollution  | -------------------+
       +------------------+                    |
                                               v
                                     +------------------+
                                     |   water_source   |
                                     +------------------+
```

### 2. Key SQL Queries Executed
To prepare the dataset for analysis and Power BI integration, several core queries were run in MySQL to aggregate queue times, filter polluted sources, and map locations.

<details>
<summary><b>Click to expand sample SQL Queries</b></summary>

```sql
-- Identifying water sources with high pollution and average queue times over 120 mins
SELECT 
    l.province_name,
    l.town_name,
    ws.type_of_water_source,
    AVG(v.time_in_queue) AS avg_queue_time_mins,
    wp.pollution_status
FROM visits v
JOIN location l ON v.location_id = l.location_id
JOIN water_source ws ON v.source_id = ws.source_id
LEFT JOIN well_pollution wp ON ws.source_id = wp.source_id
WHERE v.time_in_queue > 120
GROUP BY 1, 2, 3, 5
ORDER BY avg_queue_time_mins DESC;
```

</details>

---

## Key Insights & Recommendations

* **Queue Time Bottlenecks:** Shared taps in high-density urban areas experience the longest wait times—averaging over **2 to 3 hours** during peak morning hours.
* **Pollution Risk:** A significant percentage of active wells show elevated contamination levels, requiring immediate chemical treatment or drill upgrades.
* **Infrastructure Priorities:** Rural areas rely heavily on untreated surface water (rivers/streams), indicating a need for dedicated filtration systems and pipe network expansions.

---

## Dashboard Preview

Below is a snapshot of the interactive Power BI dashboard designed to monitor regional metrics, queue times, and water quality:

![Dashboard Preview](screenshots/dashboard_preview.png)

*(Make sure to upload your screenshot to a `screenshots` folder in your repository named `dashboard_preview.png`)*

---

## How to Run / View

1. **Clone the repo:**
   ```bash
   git clone https://github.com/your-username/maji-ndogo-water-analysis.git
   ```
2. **Database Setup:** Import the `.sql` schema script into your local MySQL server.
3. **Power BI Dashboard:** Open the `.pbix` file using **Power BI Desktop** to interact with the visualizations natively.
