# Explore DDD 2026: PoC Candidates & Selected Direction

This document compiles the Proof of Concept (PoC) candidate architectures evaluated for the Denver Water Design Storm 2026 challenge, and details the architecture, roadmap, and delivery plan for our selected PoC.

---

## 1. Selected PoC: "Strontia-Foothills Adaptive Operations Dashboard"
### *Advance Warning & Selective Withdrawal Optimization*

* **Hybrid Architecture:** Unifies **Scenario 1 (Predictive Water Quality Modeling)** and **Scenario 2 (Reservoir Depth Sonde Dynamics)** into a unified operator decision-support console.
* **The Operational Value:** Bridges the gap between raw data science and plant action:
  1. **Tells operators what is coming:** Forecasts TOC and Alkalinity 2 to 4 days ahead using watershed telemetry, plus short-horizon refinement from the reservoir sonde.
  2. **Tells operators what to do at the reservoir:** Evaluates multi-depth sonde readings across Strontia Springs Reservoir and recommends which intake gate to open (surface, mid, deep) to bypass sediment underflows or surface algae.
  3. **Tells operators what to dose at the plant:** Automatically calculates required coagulant feed rates (alum/ferric) and pH buffering needed for optimal coagulation.

---

## 2. Full Roster of PoC Candidates

### Scenario 1: TOC & Alkalinity Predictive Modeling
> *"Can watershed, hydrologic, and reservoir monitoring data provide enough advance warning to accurately predict TOC and alkalinity arriving at Foothills and give treatment staff actionable time to prepare?"*

* **Candidate 1.1: The "Dual-Horizon" Operator Early Warning Console (SELECTED BASE)**
  * **Concept:** Pairs 2–4 day long-horizon predictions (from USGS stream sensors, SNOTEL snow pillows, and NOAA weather) with 4–12 hour tactical predictions from the new Strontia depth-profiling sonde.
  * **Pros:** Directly answers Cassidi's challenge; bridges the ~4-hour open-river transit time with the 2–4 day reservoir mixing buffer.
  * **Scope:** Python ML pipeline + interactive web dashboard.
* **Candidate 1.2: Sonde-Enhanced Feature Engineering & Explainable Model (SHAP Engine)**
  * **Concept:** Focuses heavily on stabilizing Jake's model (addressing the negative cross-validation $R^2$) by introducing depth-sliced sonde features (e.g. thermocline temperature gradient, near-intake turbidity) with SHAP waterfall explanations for operators.
  * **Pros:** Deep algorithmic rigor; highly educational for data scientists.
  * **Cons:** Less immediate visual/operational utility for non-technical treatment operators.
* **Candidate 1.3: Risk-Weighted Threshold Classifier ("Alert Level Red/Amber/Green")**
  * **Concept:** Formulates the problem as an operational exceedance classifier tuned for high recall (zero false negatives) to warn when $\text{TOC} > 3.0\text{ mg/L}$ or $\text{Alkalinity} < 60\text{ mg/L}$.
  * **Pros:** Matches real-world treatment protocols where operators care more about threshold breaches than tenth-of-a-decimal precision.
  * **Cons:** Loses continuous trend visibility during moderate water conditions.

---

### Scenario 2: Storm & Runoff Events and Real-Time Depth Data
> *"Given current watershed and reservoir conditions, how is an incoming storm or runoff event likely to affect source-water quality, when will that impact arrive, and what conditions might we expect at different depths within Strontia Springs Reservoir?"*

* **Candidate 2.1: Strontia 4D Depth Stratification & Underflow Explorer**
  * **Concept:** An interactive depth-time contour visualization platform unlocking the 16,000+ vertical cast measurements in `data/Strontia 0407_0819.xlsx`.
  * **Pros:** Incredible visual impact; reveals physical limnology (thermoclines, density underflows).
  * **Cons:** Descriptive rather than prescriptive; doesn't directly tell the plant how much chemical to dose.
* **Candidate 2.2: Selective Withdrawal Intake Gate Optimizer (SELECTED BASE)**
  * **Concept:** Maps vertical water quality profiles to the physical intake ports on Strontia Springs Dam (upper, mid-depth, and deep gates). Automatically identifies turbid storm underflows or surface algal blooms and recommends the cleanest gate elevation.
  * **Pros:** Direct operational decision support; transforms abstract depth data into concrete physical valve actions.
  * **Cons:** Needs to be paired with treatment plant outcomes to show full watershed-to-tap impact.
* **Candidate 2.3: Storm Pulse Propagation & Kinematic Wave Model**
  * **Concept:** A calibrated physical travel-time model tracking storm pulses from mountain tributaries down the South Platte into the reservoir.
  * **Pros:** High hydrological fidelity.
  * **Cons:** Complex calibration; difficult to validate with only 1–2 recorded major storm events in the available timeframe.

---

### Scenario 3: Snowpack & Surface Water System Function
> *"Develop an interactive model that visualizes how water and water-quality conditions move from the watershed through the collection system to the treatment plants..."*

* **Candidate 3.1: "Snowmelt-to-Tap" System Flow & Residence Simulator**
  * **Concept:** Enhances the repository's 3D Cesium/MapLibre map (`design-storm-water-system-3d.html`) with dynamic particle animations tracing flow across the Continental Divide (Moffat/Roberts tunnels) through Strontia to Foothills and Marston plants.
  * **Pros:** Highest visual aesthetic; builds on existing 3D infrastructure.
  * **Cons:** Heavy 3D WebGL rendering overhead; less focused on the primary chemistry/treatment problem.
* **Candidate 3.2: Multi-Year Drought vs. Wet Year Comparative Analytics**
  * **Concept:** A comparative historical dashboard contrasting high-snowpack years (2023) against drought years, analyzing runoff timing, raw water temperatures, and reservoir refill rates.
  * **Pros:** Strong environmental/climatological narrative.
  * **Cons:** Retrospective analysis; lacks real-time predictive decision support.
* **Candidate 3.3: Upstream Spill & Wildfire Ash Emergency Response Tracer**
  * **Concept:** A 1D advection-dispersion tracer allowing users to click anywhere on the river network (e.g. burn scars near Deckers) and simulate arrival times and plume dilution at Strontia Dam.
  * **Pros:** Exciting emergency management application.
  * **Cons:** Hypothetical scenarios rather than daily operational reality.

---

## 3. Deep Dive: The Selected PoC Architecture

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                    STRONTIA-FOOTHILLS ADAPTIVE DASHBOARD                        │
├───────────────────────────────────────┬─────────────────────────────────────────┤
│  PANEL 1: WATERSHED EARLY WARNING     │  PANEL 2: SELECTIVE WITHDRAWAL (DAM)    │
│  - 2-4 Day Advance TOC Forecast       │  - Vertical Sonde Depth Profile         │
│  - 2-4 Day Advance Alkalinity Forecast│  - Multi-Gate Elevation Status          │
│  - Risk Level: [ NORMAL | ALERT ]     │  - Recommended Gate: [ UPPER GATE (▲) ] │
├───────────────────────────────────────┴─────────────────────────────────────────┤
│  PANEL 3: ACTIONABLE PLANT DOSING RECIPE (FOOTHILLS WTP)                        │
│  - Coagulant Dose (Alum/Ferric): 18.5 mg/L (+4.2 mg/L surge adjustment)         │
│  - pH Target: 6.8  |  Caustic/Lime Buffering: 4.5 mg/L                          │
│  - Simulated Case Study Replay: [ Toggle Aug 14-15 Storm Surge ]                │
└─────────────────────────────────────────────────────────────────────────────────┘
```

### Component Breakdown
1. **The Predictive Core (Python ML Service):**
   * Processes the 2022–2026 daily time series (`USGS_South_Platte.csv`, `SouthPlatteFlow.csv`, `HoosierPass.csv`, `USC00058022.csv`, `FoothillsInfluent.csv`).
   * Extracts summary metrics from `Strontia 0407_0819.xlsx` (intake-depth turbidity, epilimnion temperature, metalimnion thermocline stability).
   * Generates continuous forecasts and uncertainty bounds for TOC and Alkalinity.
2. **The Intake Tower Depth Optimizer:**
   * Visualizes a vertical cross-section of Strontia Dam and its intake gates.
   * Color-codes depth layers by turbidity, temperature, and algae indicators from the profiling sonde.
   * Logic engine: Flags when deep underflows exceed threshold ($>15\text{ FNU}$) or surface algae spikes ($>5\text{ }\mu\text{g/L}$ chlorophyll), calculating the cleanest withdrawal depth.
3. **The Plant Dosing Calculator:**
   * Implements standard water-treatment coagulation models (enhanced coagulation per Colorado Reg 11 / EPA Stage 2 DBP rule).
   * Converts forecasted TOC and Alkalinity into specific chemical dosing targets (coagulant ppm and acid/base adjustment).
4. **Historical Event Simulator (August 14–15 Storm):**
   * Includes a 1-click replay toggle of the August 14–15 storm event so operators/judges can watch the river spike from 2.7 to 329 FNU, see the underflow hit the lower gates, see the gate switch recommendation, and observe the plant avoiding a treatment upset.

---

## 4. 5.5-Hour Sprint Implementation Plan (8:00 – 1:30)

| Time Window | Phase | Key Deliverables |
|---|---|---|
| **08:00 – 09:15** | **Data Engineering & Modeling** | • Script to parse `Strontia 0407_0819.xlsx` casts into daily depth metrics.<br>• Train enhanced Random Forest model with lagged features + sonde inputs.<br>• Export precomputed predictions, confidence bounds, and storm replay series to JSON. |
| **09:15 – 10:45** | **Core Dashboard UI & Visualizations** | • Single-page web dashboard served by `serve.py`.<br>• Predictive timeline charts (Chart.js / SVG) for TOC, Alkalinity, and streamflow.<br>• Clean modern utility-grade UI (dark mode, responsive, high visual polish). |
| **10:45 – 11:45** | **Intake Tower & Depth Sonde Widget** | • Interactive vertical intake tower cross-section showing gate elevations.<br>• Depth profile color gradient showing water quality by depth.<br>• Dynamic gate recommendation badge with live telemetry readouts. |
| **11:45 – 12:30** | **Treatment Dosing Calculator & Simulation** | • Interactive coagulation dosing formula based on Reg 11 TOC removal matrices.<br>• August 14–15 Storm Replay interactive scrubber/demo mode. |
| **12:30 – 01:15** | **Polish, QA, & Cross-Browser Testing** | • Verification of numbers against raw files.<br>• Verification of layout, typography, tooltips, and animations.<br>• Edge-case testing and demo script preparation. |
| **01:15 – 01:30** | **Final Presentation Prep** | • Standup dry-run of presentation pitch and walkthrough points. |

---

## 5. Technology Stack & Deployment

* **Serving:** Native Python server (`python3 serve.py`), extending existing repository routing.
* **Backend / Pipeline:** Python 3 (pandas, numpy, scikit-learn, openpyxl).
* **Frontend:** Modern Vanilla HTML5 / ES6 JavaScript / CSS3.
* **Styling & Aesthetics:** Sleek, high-contrast industrial control console aesthetics (curated dark palette, glowing status indicators, smooth micro-transitions).
* **Zero External Build Step:** Pure client-side execution—no complex npm/node build chains required.

---

## 6. Compliance & Data Terms

This PoC adheres strictly to the ground rules in `AGENTS.md`:
* Denver Water original files in `data/`, `scripts/`, `figures/`, and `reference/` are strictly read-only.
* All new code, processed artifacts, and models will be housed in a clean working directory (e.g., `poc/` or `teams/storm-forecast/`).
* Denver Water public records data disclaimers and provisional data flags are retained in all dashboard headers and exports.
