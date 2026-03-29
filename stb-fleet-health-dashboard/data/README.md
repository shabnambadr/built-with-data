# STB Fleet Health & Diagnostic Dashboard

An end-to-end data analytics project built on simulated RDK Set-Top Box log data.
Transforms raw `/opt/logs` style exports into an interactive Excel dashboard that
answers the questions an engineering lead or stakeholder would actually ask.

---

## The Problem

As a device engineer working on the RDK stack, debugging involves manually
analyzing raw log files from deployed STBs — a time-consuming and reactive process. Tickets range from IR remote wake issues and wrong key
mappings to kernel panics caused by log partition overflow.

The problem: raw log data sits in text files with no visibility into patterns,
trends, or which devices are failing most. This project transforms that raw data
into an executive dashboard that answers:

- Which device models are generating the most tickets?
- What are the top 3 root causes — and do they follow the 80/20 rule?
- Is the unconfigured key wake bug (Key_0 / Key_Mic) getting worse over time?
- Which firmware version has the worst failure rate per device?
- Does high CPU temperature correlate with kernel panics and crashes?
- What is the Mean Time To Resolution (MTTR) by failure type and team?
- Where are the open ticket backlogs?

## Dashboard Preview

![Dashboard](docs/dashboard_screenshot.png)

## How to Use

1. Open `dashboard/STB_Fleet_Health_Dashboard.xlsx`
2. Use slicers to filter by device model, region, and firmware
3. Explore KPIs and charts for failure trends and correlations

## Key Insights (example)

- Apache 4K devices contribute ~38% of total tickets (clear hotspot)
- Top 3 failure types account for ~72% of issues (Pareto effect observed)
- Firmware v3.2.1 shows highest crash rate across 3 device models
- Weak positive correlation between CPU temperature and kernel panics
- MTTR significantly higher for IR-related issues vs system crashes
---

## Dataset

| Property | Detail |
|---|---|
| Rows | 2,000 (1,980 after deduplication) |
| Columns | 21 |
| Period | January – March 2026 |
| Region | UK (London, Manchester, Birmingham, Edinburgh, Bristol) |
| Devices | 9 models across Amlogic and MediaTek platforms |
| Source | Simulated based on real-world RDK log structures and failure scenarios |

Log entries are modelled on actual output from `WPEFramework`, `controlMgr`,
`meson-remote` kernel IR driver, and `com.sky.as.apps` UI layer.

**Intentional data quality issues :**
- 5 different timestamp formats across log sources
- 25+ dirty device model name variants (e.g. `APACHE_4K`, `apache 4k`, `Apache4K`)
- CPU temperature sensor failures logged as `-1` or `999`
- Blank boot times where device crashed before completing boot
- 20 duplicate Ticket IDs from a simulated export bug
- Logic errors: open tickets with resolution days filled, closed tickets with none

---

## Data Cleaning (Power Query)

All cleaning done via Power Query pipeline. Raw source file preserved untouched.
Every transformation is a discrete, auditable step in the Applied Steps panel.

| Issue | Approach |
|---|---|
| Dirty model names | Built `model_map_1` transformation table, merged using fuzzy matching |
| Mixed timestamp formats | Parsed to unified Date/Time, unparseable rows surfaced as null |
| Bad CPU temp (-1, 999) | Replaced with null — not zero, to protect averages |
| Blank boot times | Kept as null — crash before boot is different from a fast boot |
| Duplicate ticket IDs | Identified and removed (20 rows) |
| Whitespace / encoding | Trim + Clean applied to all text columns |

Output loaded as named Excel Table (`stb_fleet_health_clean`) to sheet `Raw_Data`.

---

## Technical Implementation

| Concept | Used for |
|---|---|
| Power Query + Fuzzy Merge | ETL pipeline, model name normalisation |
| XLOOKUP + IFERROR | Error code → failure category mapping |
| TEXTBEFORE / TEXTAFTER | Extracting version numbers from firmware strings |
| COUNTIFS / AVERAGEIFS | Ticket counts and MTTR by multiple criteria |
| STDEV.P | Boot time variation across firmware versions |
| CORREL | CPU temperature vs crash rate, log size vs OOM |
| Pivot Tables + Slicers | Interactive aggregation by model, firmware, region |
| GETPIVOTDATA | Dynamic KPI tiles on the dashboard |
| Conditional Formatting | Heatmap of device × failure type |
| Pareto Chart | Top 3 root causes as % of total ticket volume |

---

## Project Structure

```
stb-fleet-health-dashboard/
├── data/
│   ├── stb_fleet_health_raw.csv       ← original messy dataset
│   └── stb_fleet_health_clean.csv     ← post Power Query output (added later)
├── dashboard/
│   └── STB_Fleet_Health_Dashboard.xlsx
├── docs/
│   ├── dashboard_screenshot.png       ← added after Phase 5
│   └── formula_reference.md          ← added after Phase 2
└── README.md
```

---

## Status

| Phase | Description | Status |
|---|---|---|
| Phase 1 | Power Query — import, clean, structure | ✅ Complete |
| Phase 2 | Formula engine — lookups, extractions, calculations | 🔄 In progress |
| Phase 3 | Statistics — STDEV, CORREL, Pareto | ⏳ Pending |
| Phase 4 | Pivot tables and charts | ⏳ Pending |
| Phase 5 | Dashboard, slicers, final polish | ⏳ Pending |

---

## Tools Used

- Microsoft Excel (Power Query, Pivot Tables, Charts)
- GitHub (version control and portfolio hosting)
- Notion (project documentation and interview prep)

---

*Built as part of a transition from device engineering into data analytics.
Domain knowledge of RDK stack, STB log structure, and IR subsystem behaviour
informed the dataset design and analytical questions.*
