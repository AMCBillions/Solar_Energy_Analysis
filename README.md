# Solar_Energy_Analysis
This project performs a full exploratory data analysis (EDA) on operational data from a Solar Plant
# Brazilian PV Plant — Data Analysis Project

> **Exploratory Data Analysis of a utility-scale solar photovoltaic installation in Brazil**
> April – June 2024 | 8 Inverters | 3 Weather Stations | ~100,000 records

---

## Project Overview

This project performs a full exploratory data analysis (EDA) on operational data from a utility-scale PV plant in Brazil. The dataset covers three months of 30-minute SCADA, grid export, and weather readings across 8 inverters and 3 geographically distributed weather stations.

The analysis surfaces key operational insights including inverter underperformance, fault patterns, data quality issues, and revenue attribution — findings that are directly actionable for O&M (operations & maintenance) teams.

---

## Dataset

| File | Records | Interval | Description |
|------|---------|----------|-------------|
| `grid_export.csv` | 34,560 rows | 30 min | Energy exported, revenue, power factor, curtailment |
| `inverter_scada.csv` | 34,560 rows | 30 min | DC/AC power, voltage, current, module temp, status codes |
| `weather_stations.csv` | 25,920 rows | 15 min | GHI, DNI, DHI irradiance, temperature, humidity, wind |

**Period:** 1 April 2024 – 29 June 2024  
**Inverters:** INV-01 through INV-08  
**Weather Stations:** WS-North, WS-Central, WS-South

---

## Key Results

| Metric | Value |
|--------|-------|
| Total energy exported | 577,828 kWh |
| Total revenue | $76,484 USD |
| Average power factor | 0.96 |
| Fleet curtailment rate | 1.41% |
| Peak generation hour | 12:00–13:00 (~104 kW avg AC) |
| SCADA availability | 97.5% NORMAL status |

---

## Findings

### 1. Inverter Underperformance

| Inverter | Total Energy (kWh) | vs. Fleet Avg | Status |
|----------|--------------------|---------------|--------|
| INV-08 | 80,574 | +11.6% | Best performer |
| INV-07 | 78,577 | +8.8% | Above average |
| INV-06 | 76,176 | +5.5% | Above average |
| INV-05 | 73,685 | +2.0% | Average |
| INV-04 | 72,474 | +0.3% | Average |
| INV-02 | 67,374 | -6.7% | Below average |
| INV-01 | 65,332 | -9.6% | ⚠ Underperforming |
| INV-03 | 63,636 | **-11.9%** | 🔴 Critical — degrading |

**INV-03 is actively degrading.** Output peaked in weeks 18–19 (~5,500 kWh/week) then fell to ~4,060 kWh/week by end of June — a **26% intra-period decline**. No other inverter shows this pattern.

**DC→AC conversion efficiency** is virtually identical across all inverters (~97.0%), confirming that the inverter electronics are healthy — underperformance originates on the **DC side** (strings, modules, or connections).

### 2. Data Quality Issues

| Issue | Scope | Impact |
|-------|-------|--------|
| `energy_kwh` sentinel values (-9999) | 431 rows (1.25%) | Inflates negative totals if uncleaned |
| `inverter_efficiency` nulls | 16,567 rows (48%) | Expected — nighttime records |
| Mixed temperature units (°C + °F) | `weather_stations.csv` | Breaks thermal analysis |
| Mixed timestamp formats | `inverter_scada.csv` | Requires `format='mixed'` parsing |
| INV-05 `dc_power_kw` unit mismatch | All INV-05 DC records | Values in Watts, not kilowatts (~1000× too large) |

### 3. Fault Analysis

- **293 FAULT events** across 90 days — roughly **36–40 per inverter**, uniformly distributed
- **Median fault duration: 30 minutes** (= exactly 1 SCADA reporting interval) → indicates auto-recovery
- **No temporal clustering** — faults occur at all hours including overnight, ruling out thermal or irradiance triggers
- **Interpretation:** These are likely communication dropouts or soft resets, not hardware failures requiring physical intervention

### 4. Weather & Generation Profile

- GHI irradiance **increases month-on-month**: Apr 278 → May 293 → Jun 299 W/m²
- Generation window: **06:00–18:30** with bell-curve peak at 12:00–13:00
- All three weather stations record identical average temperature (24.8°C) — possible averaging artefact

---

## Recommendations

1. **Inspect INV-03 immediately** — schedule a string-level IV curve trace and visual panel inspection. The 26% intra-period output decline suggests progressive soiling, partial shading, or a failing string connection.

2. **Fix INV-05 dc_power_kw unit bug** — divide all `dc_power_kw` values for INV-05 by 1,000 at source. Currently logged in Watts rather than kilowatts, making efficiency calculations invalid.

3. **Normalise temperature units** — standardise all weather station readings to °C before any thermal correlation analysis.

4. **Resolve timestamp inconsistency** in `inverter_scada.csv` — two date formats present; standardise at ingestion.

5. **Flag -9999 sentinel values** as NULL in the pipeline rather than allowing them to flow into aggregations.

6. **Correlate faults with weather** — cross-reference COMM_FAIL events with wind speed and precipitation data to determine if environmental conditions are a factor.

---

## Project Structure

```
brazillian-pv-analysis/
├── README.md                    ← You are here
├── data/
│   ├── grid_export.csv
│   ├── inverter_scada.csv
│   └── weather_stations.csv
├── notebooks/
│   └── pv_eda_analysis.ipynb    ← Full analysis notebook
├── analysis/
│   └── pv_full_report.md        ← Detailed written report
└── requirements.txt
```

---

## Tech Stack

- **Python 3.12** — pandas, numpy
- **Visualisation** — Matplotlib, Seaborn
- **Data source** — Google Cloud BigQuery (`brazillian-pv-analysis` project)

---

## Setup

```bash
git clone https://github.com/YOUR_USERNAME/brazillian-pv-analysis.git
cd brazillian-pv-analysis
pip install -r requirements.txt
jupyter notebook notebooks/pv_eda_analysis.ipynb
```

---

## Author

Analysis performed on data from the `chibambamulenga-org / Brazillian-PV-Analysis` Google Cloud project.

---

*Part of an ongoing series of energy data analysis projects. See LinkedIn for the summary post.*
