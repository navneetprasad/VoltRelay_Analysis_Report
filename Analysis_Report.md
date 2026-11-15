# VoltRelay Network Performance, Customer Retention & Unit Economics
## Comprehensive End-to-End Analytical Study & Executive Strategy Report

**Prepared for:** Executive Leadership & Operations Directorate, VoltRelay  
**Associated Study Notebook:** `VoltRelay_End-to-End_Analytical_Study.ipynb`  
**Dataset Scale:** 3,877,013 Transactions | 150 Operational Stations | 6,500 Battery Packs | 20,000 Riders | 12 Fleet Partners | 44,000 Support Tickets  
**Observation Window:** January 1, 2024 – June 30, 2025 (18 Months)

---

## 1. Problem Understanding

### 1.1 Business Model & Operational Context
VoltRelay operates a Battery-as-a-Service (BaaS) and energy infrastructure network catering to the rapidly expanding electric two-wheeler (2W) and three-wheeler (3W) market across six major Indian metropolitan hubs: Bengaluru, Delhi NCR, Hyderabad, Mumbai, Pune, and Jaipur. 

VoltRelay’s primary value proposition centers on eliminating range anxiety and minimizing vehicle downtime for commercial delivery drivers (food delivery, e-commerce, quick commerce, and bike taxis) as well as retail gig commuters. Instead of waiting hours for conductive wire charging, riders pull into automated swap stations, deposit a depleted pack, and retrieve a fully charged pack in under three minutes.

```mermaid
flowchart LR
    subgraph Ecosystem["VoltRelay BaaS Value Loop"]
        direction LR
        Rider["Riders (Retail & Fleets)"] -->|"Swap Depleted Pack (<3 min)"| Stn["Swap Stations (Gen1, Gen2, Gen3)"]
        Stn -->|"Grid Electricity Ingestion"| Grid["Municipal Power Grids"]
        Stn -->|"Delivers Fully Charged Pack"| Rider
        Bat["Battery Fleet (Cellora, Amptek, Kyron)"] -->|"Cycle Asset Degradation"| Stn
        Fleets["Fleet Partners (12 Enterprise Contracts)"] -->|"Volume Tiered Billing"| VoltRelay["VoltRelay Commercial Core"]
    end
```

### 1.2 The Core Scaling Paradox
VoltRelay has achieved staggering top-line commercial growth. Between January 2024 and June 2025:
* Monthly completed swap volume surged **191%**, growing from **105,490 to 307,259 swaps/month**.
* Monthly gross revenue expanded from **₹6.32M to ₹19.97M** (over ₹233M cumulative).
* Contribution margin per completed swap expanded from **₹37.90 to ₹47.96** following a base price hike in July 2024.

However, operational leadership is experiencing an acute, hidden crisis beneath these top-line averages:
1. **Severe Summer Service Failures**: The network experienced recurring failure spikes where swap failure rates nearly tripled, peaking at **8.04% in May 2024** and **6.48% in May 2025**, driving massive queue abandonment.
2. **The 30% Delivered Range Collapse**: Delivered distance per swap plummeted from **74.1 km down to 52.0 km**, forcing riders into a vicious cycle of swapping 42.5% more frequently for the same daily route.
3. **Severe Fleet Margin Dilution**: VoltRelay’s largest partner by volume (**ZipDrop - FP-03**) generates only **₹20.94 net margin per swap** compared to **₹38.09 for retail riders** and **₹31.89 for FeastFly**, eroding overall enterprise return on capital.
4. **Persistent Rider Churn**: Mature cohorts suffer a **43.3% churn rate**, driven by early service failure trauma and battery range degradation.

### 1.3 Key Stakeholders & Conflict Matrix
* **Gig Delivery Riders**: Require reliable battery range (>65 km) and zero-queue turnaround to maintain their delivery livelihoods. When range falls below 50 km, their daily income is penalized.
* **Fleet Enterprise Partners**: Negotiate steep volume discounts (up to 28%), demand extended payment terms (45 days), and resist peak-hour pricing surcharges.
* **Station Landlords / Hosts**: Demand fixed monthly rents (averaging ₹17,500–₹18,600/month) irrespective of station throughput, penalizing under-utilized expansion waves.
* **Battery OEMs**: Face warranty friction when rapid thermal degradation causes premature pack retirement.

---

## 2. Analytical Approach

The analytical methodology follows a rigorous, multi-tiered data pipeline that combines transaction-level event processing, geospatial clustering, survival analysis, and natural language processing (NLP).

```mermaid
flowchart TD
    subgraph DataIngestion["1. Data Ingestion & Sanitization"]
        D1["swap_events.csv (728 MB, 3.88M rows)"]
        D2["station_hourly_status.csv (108 MB, 1.46M rows)"]
        D3["batteries.csv, riders.csv, stations.csv, fleet_partners.csv"]
        D4["support_tickets.csv (44k records)"]
        D1 --> Clean["Type Normalization, Whitespace Trimming & Identifier Alignment"]
        D2 --> Clean
        D3 --> Clean
        D4 --> Clean
    end

    subgraph FeatureEngineering["2. Advanced Feature Engineering"]
        Clean --> F1["Unit Economics Engine: Revenue - Energy Cost - Battery Wear"]
        Clean --> F2["Telemetry Integration: Cabinet Temp, Charging Speed, Outage Minutes"]
        Clean --> F3["Cohort Mapping: True First-Swap Flagging & Tenure Tracking"]
        Clean --> F4["Bilingual NLP Engine: Hinglish & English Semantic Tagging"]
    end

    subgraph AnalyticalModules["3. Core Investigation Modules"]
        F1 --> M1["Longitudinal & Tariff Decomposition"]
        F2 --> M2["Spatial & Hardware Generation Profiling"]
        F3 --> M3["Survival & Churn Risk Attribution"]
        F4 --> M4["Unstructured Support Text Mining"]
    end

    subgraph DecisionEngine["4. Strategic Decision Modeling"]
        M1 --> Sim["Capital Budget Scenario Simulation (S1 - S4)"]
        M2 --> Sim
        M3 --> Sim
        M4 --> Sim
        Sim --> Output["P&L Optimization & Actionable Roadmap"]
    end
```

### 2.1 Data Hygiene & Alignment Protocols
1. **Identifier Sanitization**: In early exploratory passes (as documented in `VoltRelay_End-to-End_Analytical_Study.ipynb`), string whitespace mismatches in `rider_id` between transaction tables and master registries previously produced artificial 0% churn artifacts. Normalizing keys via `.str.strip()` and standardizing schema dtypes resolved all cross-table joins.
2. **Chronological Event Indexing**: Rather than relying on non-chronological sequential proxies, events were strictly indexed by `event_ts` to capture true initial onboarding experiences for each rider.
3. **Filtering Operational Scope**: Non-operational test stations (`STN-TST-01` and `STN-TST-02`) were isolated and excluded from commercial benchmarks.

### 2.2 Core Methodological Frameworks
* **Longitudinal Time-Series Decomposition**: Evaluating monthly and weekly trends across attempts, completions, failures (`failed_no_charged_battery`, `failed_system_error`), and abandonments.
* **Variable Unit Cost Engine**: Calculating exact grid energy recharge costs by joining station-level electricity tariffs (`grid_tariff_inr_kwh`) with transaction energy consumed (`energy_to_recharge_kwh`), augmented with battery asset wear depreciation (₹12/swap for 2W, ₹25/swap for 3W).
* **Diurnal & Thermal Profiling**: Cross-referencing 1.46M hourly station telemetry records with ambient weather data (`city_daily_context.csv`) to map cabinet overheating and stockout probabilities across 24-hour cycles.
* **Survival & Churn Attribution**: Formulating tenure models across 16,357 mature riders (signed up $\le$ April 1, 2025) with a 45-day inactivity threshold to isolate primary churn drivers from confounding variables.
* **Bilingual Semantic Extraction**: Utilizing specialized regex dictionaries capturing colloquial Hindi-English (Hinglish) phrasing (`"paisa kat gaya"`, `"intezar"`, `"dhua"`, `"jaldi khatam"`) to classify unstructured support complaints.

---

## 3. Key Insights

```mermaid
flowchart TD
    subgraph CausalityLoop["The Core Destabilization Feedback Loop"]
        A["Kyron Lot Defect (KY-2407/08/09)"] --> B["Delivered Range Drops 30%<br/>(74.1 km -> 52.0 km)"]
        B --> C["Rider Swap Frequency Increases 42.5%"]
        C --> D["Station Demand Overload"]
        E["51 Gen1 Chargers (91.8 min recharge)"] --> D
        D --> F["Midday Battery Stockouts<br/>(Surges to 4.8% daily, 8.0% summer)"]
        F --> G["First-Swap Failure & Service Trauma"]
        G --> H["Rider Churn (43.3% Mature Rate)<br/>Summer cohorts lose 11% more riders"]
    end
```

### Insight 1: The "Phantom Demand" Range Trap
* Across the 18 months, average delivered range per completed swap collapsed from **74.13 km in January 2024 to 51.99 km in June 2025**—a **29.9% loss in battery efficiency**.
* To deliver the same daily orders, delivery riders were forced to swap **1.42 times as often**. Consequently, a significant portion of VoltRelay's "volume growth" was artificial—it represented **distressed swaps caused by failing batteries** rather than genuine network customer expansion.

### Insight 2: Hardware Bottleneck — Gen1 Charger Turnaround
* Station telemetry reveals that charger turnaround speed is the single largest operational differentiator:
  * **Gen1 Chargers (51 stations)**: Average charge turnaround is **91.81 minutes**.
  * **Gen2 Chargers (62 stations)**: Average charge turnaround is **64.23 minutes** (-30% faster).
  * **Gen3 Chargers (37 stations)**: Average charge turnaround is **42.14 minutes** (-54% faster).
* Because Gen1 stations take over 1.5 hours to turn around a battery, any minor midday rush in dense commercial hubs exhausts available inventory, leading to multi-hour stockout deadlocks.
* **100% of the top 10 worst bottleneck stations on the network are Gen1 units located in Delhi NCR and Jaipur**.

### Insight 3: The Isolated Kyron Manufacturing Lot Catastrophe
* On June 15, 2025, VoltRelay mass-retired **1,438 batteries (22.1% of the entire fleet)** for premature `end_of_life`.
* **100% of retired batteries were from supplier Kyron**.
* Crucially, the analytical investigation proves this was **not a systemic Kyron failure**, but was isolated to three specific manufacturing lots:
  * **KY-2407, KY-2408, KY-2409**: 100% of 2W packs (1,385 packs) suffered catastrophic degradation, dropping to **55–65% SoH**, and were retired.
  * **KY-2410 through KY-2418**: Experienced **0 retirements**, maintaining an excellent **87.9% SoH**.
  * **Cellora (3,555 packs)** and **Amptek (1,307 packs)**: Experienced **0 retirements**, maintaining **80.9% and 81.0% SoH**, respectively.
* Kyron 2W packs delivered **50.93 km/swap**, generating a **9.4 km deficit** compared to Cellora (60.34 km) and Amptek (60.31 km).

### Insight 4: The ZipDrop (FP-03) Value Destruction Paradox
* ZipDrop is VoltRelay's largest partner by completed swaps (**557,172 swaps**), but ranks as the **least profitable partner on the network**.
* Due to a contract amendment on November 1, 2024, ZipDrop received a **28% discount**, an exemption from peak surcharges, and 45-day payment terms.
* **Financial Comparison**:
  * **Retail Walk-in**: ₹38.09 net contribution margin per swap (₹47.07M total).
  * **FeastFly (FP-01)**: ₹31.89 net contribution margin per swap (₹16.79M total on 526k swaps).
  * **ZipDrop (FP-03)**: **₹20.94 net contribution margin per swap** (₹11.66M total on 557k swaps).
* Despite generating 30,552 more swaps than FeastFly, ZipDrop delivered **₹5.13M LESS in net profit**, creating massive network congestion while paying a fraction of retail tariffs.

### Insight 5: Midday Depletion vs. Evening Rush Operational Mismatch
* Telemetry proves that zero-inventory stockouts do not peak during the evening rush. Instead, **stockouts peak at 15:00 (4.83% of operating hours)** when intense afternoon heat (cabinet temps reaching 33.5°C) throttles charger speeds while lunch delivery batteries are recharging.
* When the true volume rush arrives at **19:00–21:00 (over 400,000 swaps per hour window)**, stations enter the rush with half-empty inventories, driving failure rates to **4.23%**.

### Insight 6: The "First Swap" Retention Trauma & Summer Churn
* In the notebook study, evaluating first-swap experiences demonstrates that **experiencing a failed first swap nearly doubles rider churn probability (from 1.51% to 2.70% in early windows)**.
* Across the full 18-month dataset, riders who joined during the May 2024 heatwave (when stockouts reached 8.04%) suffered a **12-month retention of only 40.2%**, compared to **51.1%** for the January 2024 cohort—an irreversible **10.9 percentage point retention loss**.

---

## 4. Supporting Visualizations

### 4.1 Monthly Trajectory: Growth vs. Margin & Efficiency
```
Monthly Completed Swaps vs. Delivered Range per Swap (km)
Swaps (k)                                                              Delivered Range (km)
350k ───────────────────────────────────────────────────────────── 80 km
                                                     307k   307k  │
300k ────────────────────────────────────────── 290k ───────┼───── 70 km (Jan 24: 74.1 km)
                                         269k ──────┘       │     │
250k ───────────────────────────── 252k ─┘                  │     │
                            219k ──┘                        ▼     │
200k ──────────────── 187k ─┘                              52.0 km 60 km
               151k ──┘                                    (Jun 25)
150k ── 128k ──┘                                                  │
105k ──┘                                                          │
100k ───────────────────────────────────────────────────────────── 50 km
     Jan24 Mar24 May24 Jul24 Sep24 Nov24 Jan25 Mar25 May25 Jun25
     [════════ Completed Swaps ════════] [── Delivered Range ──]
```

### 4.2 Diurnal Curve: Zero Inventory % vs. Cabinet Temperature
```mermaid
xychart-beta
    title "Diurnal Stockout Rate (%) vs Cabinet Temperature (°C)"
    x-axis ["00:00", "03:00", "06:00", "09:00", "12:00", "14:00", "15:00", "16:00", "18:00", "20:00", "22:00"]
    y-axis "Zero 2W Inventory Rate (%)" 0 --> 5
    bar [0.26, 0.26, 0.28, 0.38, 1.86, 4.38, 4.83, 4.37, 1.82, 0.49, 0.29]
    line [0.30, 0.30, 0.35, 1.20, 3.10, 4.30, 4.80, 4.30, 2.80, 0.80, 0.40]
```

### 4.3 Net Contribution Margin Per Swap by Partner
```mermaid
xychart-beta
    title "Net Contribution Margin Per Swap (₹) After Energy & Battery Wear"
    x-axis ["Retail", "CargoTuk (3W)", "FeastFly", "RideMitra", "QuickCart", "Swiggle Go", "ParcelNest", "ZipDrop"]
    y-axis "Net Margin / Swap (INR)" 0 --> 40
    bar [38.09, 33.67, 31.89, 32.79, 32.16, 28.94, 26.41, 20.94]
```

### 4.4 Support Ticket Keyword Distribution (English & Hinglish Text Mining)
```mermaid
pie title Free-Text Support Complaints Breakdown (44,000 Tickets)
    "Billing & Deduction Issues ('paisa kat gaya', 'extra deduct')": 27.0
    "Degraded Range & Quick Drain ('jaldi khatam', 'kam chalti')": 20.1
    "Queue Wait & Slow Charging ('intezar', 'bheed', 'slow machine')": 19.6
    "Hardware & Locker Jamming ('door stuck', 'error screen')": 15.2
    "Station Stockout / Empty ('no battery', 'khali')": 10.5
    "General App / Network Glitches": 7.6
```

---

## 5. Business Findings

### 5.1 Unit Economics & P&L Reality
A granular P&L decomposition by customer segment highlights where real economic value is created versus where it is destroyed:

| Customer Segment | Completed Swaps | Gross Revenue (₹) | Energy Cost (₹) | Battery Wear Cost (₹) | Net Contribution (₹) | Net Margin / Swap (₹) | Effective Realized Price (₹) |
| :--- | :---: | :---: | :---: | :---: | :---: | :---: | :---: |
| **Retail Commuters** | **1,235,968** | ₹84,372,541 | ₹22,468,640 | ₹14,831,616 | **₹47,072,285** | **₹38.09** | **₹68.26** |
| **FeastFly (FP-01)** | 526,620 | ₹32,696,006 | ₹9,583,770 | ₹6,319,440 | **₹16,792,796** | **₹31.89** | ₹62.09 |
| **CargoTuk (FP-04 - 3W)** | 134,085 | ₹13,333,431 | ₹5,466,800 | ₹3,352,125 | **₹4,514,506** | **₹33.67** | ₹99.44 |
| **ParcelNest (FP-02)** | 302,506 | ₹17,113,715 | ₹5,493,650 | ₹3,630,072 | **₹7,989,993** | **₹26.41** | ₹56.57 |
| **ZipDrop (FP-03)** | **557,172** | ₹29,015,711 | ₹10,664,670 | ₹6,686,064 | **₹11,664,977** | **₹20.94** | **₹52.08** |

* **Retail is King**: Walk-in retail riders represent **33.9% of completed swaps** but generate **40.3% of net profit**, boasting an unbeatable unit margin of **₹38.09/swap**.
* **ZipDrop Subsidization**: VoltRelay is effectively utilizing retail profits to subsidize ZipDrop's operations. ZipDrop consumes 29% of network 2W battery wear while generating a margin 45% below retail.

### 5.2 Capital Misallocation: The Expansion Wave Flaw
A critical evaluation of the three station commissioning waves reveals severe capital misallocation:

| Expansion Wave | Stations | Total Completed Swaps | Swaps / Station | Margin / Station (₹) | Avg Monthly Rent (₹) | Rent-to-Margin Efficiency |
| :--- | :---: | :---: | :---: | :---: | :---: | :---: |
| **Launch Wave** | 90 | 2,877,204 | **34,098** | **₹1,396,511** | ₹17,510 | **1.25% (High Return)** |
| **Wave 1 (2024H2)** | 30 | 476,631 | **16,703** | **₹749,550** | ₹18,649 | **2.49% (Moderate)** |
| **Wave 2 (2025H1)** | 30 | 232,840 | **8,169** | **₹370,167** | ₹17,741 | **4.79% (Low Return)** |

* **The Siting Failure**: Wave 1 and Wave 2 stations run at **49.0%** and **24.0%** of Launch station volume, respectively.
* Instead of adding infill capacity around choked Launch stations in high-density commercial markets (e.g. Koramangala, Connaught Place, Hitec City), expansion waves leased sites in peripheral areas. Consequently, Launch stations remained overwhelmed, while newer stations sit with idle inventory while incurring ₹18,600 in monthly host rent.

### 5.3 Text Mining: Unstructured vs. Structured Discrepancy
* 15,271 support tickets were categorized as `other` or `app_issue`.
* NLP text mining reveals that **41% of these tickets were severe physical operational failures**:
  * Mechanical locker door jamming during swaps (`"locker nahi khula"`, `"phas gaya"`).
  * Automated payment deductions occurring when no battery was dispensed (`"paisa cut gaya battery nahi mili"`).
* Relying strictly on structured ticket tags masked severe hardware failure patterns from engineering visibility.

---

## 6. Actionable Recommendations

### 6.1 Strategic Budget Allocation Verdict
Evaluating the four proposed capital deployment scenarios based on the empirical simulation:

```
┌────────────────────────────────────────────────────────────────────────────────────────┐
│                              CAPITAL SCENARIO SCORECARD                                │
├───────────────────────┬──────────────┬──────────────┬──────────────────────────────────┤
│ Scenario              │ Cost Impact  │ Margin Lift  │ Executive Verdict                │
├───────────────────────┼──────────────┼──────────────┼──────────────────────────────────┤
│ S1: Hardware Retrofit │ Moderate     │ +5.0%        │ APPROVED (Priority 2)            │
│ S2: Dynamic Pricing   │ Low          │ +12.0%       │ DEFERRED (Loophole Fix Needed)   │
│ S3: ZipDrop Exclusive │ Negative     │ -14.0%       │ REJECTED (Value Destructive)     │
│ S4: Battery Replacem. │ Moderate-High│ +8.0%        │ APPROVED (Priority 1)            │
└───────────────────────┴──────────────┴──────────────┴──────────────────────────────────┘
```

1. **S4: Procure 2,000+ Tier-1 Batteries (Cellora/Amptek) — APPROVED (Priority 1)**:
   * **Justification**: Eliminates the 1,438 dead Kyron pack deficit, restores delivered range to >65 km, cuts artificial swap frequency by 30%, and halts summer cohort churn.
2. **S1: Retrofit Gen1 Stations to Gen3 Modules — APPROVED (Priority 2)**:
   * **Justification**: Cutting turnaround from 91.8 to 42.1 minutes at 51 bottleneck stations more than doubles site throughput without leasing new land.
3. **S2: Dynamic Pricing Expansion — DEFERRED**:
   * **Justification**: While PEAK swaps yield ₹65.21 margin, demand is inelastic (5:1 peak ratio). Rolling this out without closing fleet partner contract exemptions unfairly penalizes retail riders while fleets congest stations for free.
4. **S3: Long-Term Exclusive with ZipDrop (FP-03) — STRONGLY REJECTED**:
   * **Justification**: Signing exclusivity under ZipDrop’s current 28% discount and 0 peak surcharge institutionalizes margin dilution and displaces higher-margin retail demand.

---

### 6.2 Three-Phased Strategic Implementation Roadmap

```mermaid
flowchart LR
    subgraph P1["Phase 1: Operational Stabilization (M1–M3)"]
        direction TB
        A1["Inject 2,000 Tier-1 Battery Packs<br/>(Cellora/Amptek 2W)"]
        A2["Deploy Firmware v3.2.1 Network-Wide<br/>(Fix locker jams & offline sync)"]
        A3["Deploy Station Offline Edge-Cache<br/>(Stop duplicate billing disputes)"]
    end

    subgraph P2["Phase 2: Commercial Restructuring (M2–M4)"]
        direction TB
        B1["Renegotiate ZipDrop FP-03 Contract<br/>(Cap discount at 15%, 15-day terms)"]
        B2["Mandate Fleet Peak Surcharges<br/>(Eliminate peak-hour free riding)"]
        B3["Incentivize Off-Peak Commercial Charging<br/>(₹10 off between 00:00 - 06:00)"]
    end

    subgraph P3["Phase 3: Network Hardening & Infill (M4–M9)"]
        direction TB
        C1["Retrofit Top 15 Gen1 Stations to Gen3<br/>(Target Delhi & Jaipur bottlenecks)"]
        C2["Halt Peripheral Greenfield Leases<br/>(Consolidate on Highway Fuel Pumps)"]
        C3["Deploy Cabinet Thermal Insulation<br/>(Maintain cabinet temps <30°C)"]
    end

    P1 --> P2 --> P3
```

### 6.3 Executive Performance Targets (12-Month Horizon)
* **Delivered Range Recovery**: Increase average delivered distance from **52.0 km to $\ge$66.0 km per swap**.
* **Failure Rate Compression**: Lower peak summer failure rates from **8.04% to <2.50%**.
* **Net Margin Realization**: Increase blended network net margin per swap from **₹31.20 to $\ge$₹37.50**.
* **Mature Cohort Retention**: Improve 12-month cohort retention from **40.2% to $\ge$52.0%**.

---
*Report certified and cross-verified against `VoltRelay_End-to-End_Analytical_Study.ipynb` and VoltRelay production telemetry.*
