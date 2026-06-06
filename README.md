# NYC Yellow Taxi Fleet Positioning System


![Status](https://img.shields.io/badge/Status-Completed-success)
![R2](https://img.shields.io/badge/R%C2%B2-0.868-green)
![Stack](https://img.shields.io/badge/Tech-Python_|_RandomForest_|_VADER_|_GeoPandas_|_Groq_AI-blue)
![Data](https://img.shields.io/badge/Data-1.43M_Zone--Hour_Records-orange)


## Executive Summary


**The Problem:** NYC Yellow Taxi fleet managers must constantly decide where to reposition idle cabs across 263 zones in real time. Standard demand forecasts only tell managers where demand already is, not where it's *heading*, and not which zones need cabs *right now* based on live conditions.


**The Solution:** We built a complete end-to-end fleet positioning system that integrates five public NYC data sources (taxi trip records, weather, special event permits, 311 complaint data, and zone geographies) to produce actionable repositioning scores by zone. A Random Forest model forecasts demand at the zone-hour level, and a Groq LLM (Llama 3.1) synthesises quantitative findings into plain-language repositioning decisions for fleet managers.


**Key Impact:**
- **R² of 0.868** on a held-out Nov-Dec 2025 test set, capturing holiday and event-driven demand spikes
- **60.2% RMSE reduction** from naive baseline to M1 (zone + time alone), confirming structural predictability of taxi demand
- **+22% rain demand lift** in Midtown Manhattan zones; **negative lift at airports**, a directly actionable repositioning signal
- **4.9x event demand lift** in active-event zones, enabling pre-positioning before major city events end
- 1,430,274 zone-hour records integrated across 5 data sources with 100% geocoding success rate


---


## Model Performance


We built four incremental models to quantify the value of each external data source:


| Model | Features | RMSE | R² | RMSE vs Baseline |
|-------|----------|------|-----|-----------------|
| Baseline | Training mean | 75.2 | — | — |
| M1 | Zone + time features | 27.11 | 0.868 | **-64%** |
| M2 | M1 + weather | 27.05 | 0.869 | -64.1% |
| M3 | M2 + events | 27.00 | 0.870 | -64.1% |
| M4 | M3 + 311 complaints | 27.10 | 0.868 | -64.0% |


**Key takeaway:** Zone identity and time of day explain the vast majority of demand variance. Weather and event data add genuine signal but modest aggregate RMSE improvements. The value of external data shows most clearly in the directional repositioning scores (see below), not in overall accuracy.


---


## EDA: Four Actionable Signals


| Signal | Key Finding | Repositioning Action |
|--------|-------------|---------------------|
| Time of day | Peak at 18:00 (56.8 trips/zone); trough at 05:00 (6.9) | Pre-position before 17:00 rush |
| Rain | Airports lose riders; Midtown gains up to +22% | Shift cabs from JFK/LGA to Midtown at first rain |
| Events | +393% demand lift in active-event zones | Pre-position 1 hr before event end time |
| 311 sentiment | Very negative zones see 50% lower demand than neutral zones | Monitor real-time sentiment; avoid high-friction zones |
