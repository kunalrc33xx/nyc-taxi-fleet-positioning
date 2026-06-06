# NYC Yellow Taxi Fleet Positioning System

![Status](https://img.shields.io/badge/Status-Completed-success)
![R2](https://img.shields.io/badge/R%C2%B2-0.868-green)
![Stack](https://img.shields.io/badge/Tech-Python_|_RandomForest_|_VADER_|_GeoPandas_|_Groq_AI-blue)
![Data](https://img.shields.io/badge/Data-1.43M_Zone--Hour_Records-orange)

## Executive Summary

**The Problem:** NYC Yellow Taxi fleet managers must constantly decide where to reposition idle cabs across 263 zones in real time. Standard demand forecasts only tell managers where demand already is — not where it's *heading*, and not which zones need cabs *right now* based on live conditions.

**The Solution:** We built a complete end-to-end fleet positioning system that integrates five public NYC data sources — taxi trip records, weather, special event permits, 311 complaint data, and zone geographies — to produce actionable repositioning scores by zone. A Random Forest model forecasts demand at the zone-hour level, and a Groq LLM (Llama 3.1) synthesises quantitative findings into plain-language repositioning decisions for fleet managers.

**Key Impact:**
- **R² of 0.868** on a held-out Nov–Dec 2025 test set, capturing holiday and event-driven demand spikes
- **60.2% RMSE reduction** from naive baseline to M1 (zone + time alone), confirming structural predictability of taxi demand
- **+22% rain demand lift** in Midtown Manhattan zones; **negative lift at airports** — directly actionable repositioning signal
- **4.9× event demand lift** in active-event zones, enabling pre-positioning before major city events end
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

---

## Technical Pipeline

### Data Sources (5 integrated)
- **NYC TLC Yellow Taxi Records:** 12 monthly parquet files (2025), aggregated to zone-hour demand counts across 263 zones
- **Open-Meteo Historical API:** Hourly weather (temperature, precipitation, WMO weather code) for all 8,760 hours of 2025
- **NYC Open Data — Event Permits:** 31,182 special event permits (street fairs, parades, film shoots, concerts); classified by keyword into 5 event types using TF-IDF
- **NYC 311 Complaint API:** 1,096,671 complaint records (illegal parking, blocked driveways, taxi complaints, noise); VADER sentiment scored on free-text descriptors
- **NYC TLC Zone Shapefile:** 263 zone polygons (geopandas), used for spatial join geocoding of all complaint coordinates

### Feature Engineering
- **Temporal:** hour of day, day of week, cyclical sine/cosine encodings for hour
- **Weather:** temperature (°C), precipitation (mm), rain flag (binary)
- **Events:** event count per zone-hour, event type category flags
- **311 complaints:** complaint count per zone-hour, average VADER compound sentiment score, complaint type breakdown
- **Geographic:** LocationID (zone fixed effects), borough label

### Modeling Strategy
- **Train/test split:** temporal boundary at November 1st, 2025 (1.18M train, 246K test)
- **Models compared:** Naive mean baseline, grouped-mean (M1–M4), Random Forest (M1–M4)
- **Evaluation:** RMSE and R² on held-out test set covering Thanksgiving, holiday events, and end-of-year demand
- **Repositioning scores:** computed as % demand change in rain vs. dry and event vs. no-event conditions per zone

### AI Layer
- **Groq API + Llama 3.1-8b-instant:** LLM receives zone-level repositioning scores and EDA findings as structured context, outputs 5 plain-language repositioning decisions for fleet managers
- Demonstrates the communication layer on top of quantitative analysis — the model provides substance, the LLM provides accessibility

---

## Real-World Applications

**Urban Mobility Operations:** Any ride-hail or taxi fleet operator can adapt this framework. The architecture of zone-level demand forecasting + directional repositioning scores + LLM recommendation synthesis applies directly to Uber, Lyft, or municipal taxi systems. The rain and event repositioning signals are immediately deployable as operational decision rules.

**City Planning and Infrastructure:** NYC agencies and urban planners can use the 311 sentiment and complaint spatial analysis to identify friction zones — areas where illegal parking, noise, or taxi complaints correlate with suppressed mobility. These insights inform where to improve infrastructure, signage, or enforcement to reduce transportation friction.

**Event Logistics and Venue Management:** The event permit lift analysis shows that corporate and film production events generate 4–10× higher cab demand than baseline in their immediate zones. Event venues, conference centers, and production companies can use this to coordinate ground transportation in advance, reducing passenger wait times and zone congestion.

---

## Limitations and Future Work

- **Event geocoding is approximate:** All permits within a borough are assigned to one representative zone (e.g., all Manhattan events map to Upper East Side South). This dilutes the event signal for large, diverse boroughs and means event lift estimates understate the true local impact near actual event sites. Future work: integrate lat/lon via Google Maps Geocoding API on permit addresses.
- **311 complaint noise:** VADER misreads domain-specific complaint language (e.g., "party" scores positive despite frustrated context). At 1M+ records the errors average out, but individual complaint scores are unreliable. A domain-specific classifier trained on labeled 311 text would improve signal quality.
- **No real-time deployment:** The current pipeline processes historical 2025 data in batch. A production system would require streaming ingestion of taxi pickups, weather alerts, and 311 complaints with sub-hour latency. The model would need retraining triggers when demand distribution shifts.
- **External validity:** The model is trained and tested on 2025 NYC data. Applicability to other cities or time periods requires recalibration; demand patterns, zone structures, and weather effects differ significantly across urban contexts.

---

## How to Run This Project

1. **Clone the repository**

```bash
git clone https://github.com/kunalrc33xx/nyc-taxi-fleet-positioning.git
cd nyc-taxi-fleet-positioning
```

2. **Install dependencies**

```bash
pip install pandas numpy scikit-learn geopandas matplotlib requests vaderSentiment groq
```

3. **Set up data access**

The notebook is designed for Google Colab with a Google Drive folder (`CloudComputingProject`) containing 12 monthly parquet files from the [NYC TLC Trip Record Data](https://www.nyc.gov/site/tlc/about/tlc-trip-record-data.page). All other data sources (Open-Meteo, NYC Open Data, 311 API) are fetched automatically via REST APIs in the notebook.

4. **Run the notebook**

Open `BUDT758J_Group10_Final_Project_Python.ipynb` in Jupyter or Google Colab and run all cells sequentially (Parts 0–9).

5. **View the rendered output**

Open `BUDT758J_Group10_FinalProject.html` in any browser to see the fully executed notebook with all charts and outputs without running any code.

6. **Read the full report**

Download `BUDT758J_Group10_FinalReport.docx` for the written analysis covering methodology, findings, and business recommendations.

---

*Project by Kunal Roy Chowdhury | UMD MSBA | BUDT758J — Enterprise Cloud Computing and Big Data | Spring 2026*
