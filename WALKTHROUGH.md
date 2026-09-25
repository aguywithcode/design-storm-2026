# Denver Water Design Storm 2026: Exploration Walkthrough

A synthesized walkthrough of the domain, physical hydrology, water treatment chemistry, predictive modeling, and empirical storm analysis explored in this repository.

---

## 1. Project Background & Purpose

The **Design Storm 2026** is a collaborative challenge hosted at [Explore DDD 2026](https://exploreddd.com) in partnership with **Denver Water's Water Quality and Treatment team**.

### The Core Operational Problem
Raw water flows down the South Platte River into **Strontia Springs Reservoir** and is conveyed via mountain tunnel to the **Foothills Water Treatment Plant**. Water arriving at the plant varies constantly in treatability:
* **TOC (Total Organic Carbon):** Dissolved organic matter (leaves, soil, forest litter). When the plant adds chlorine for disinfection, chlorine reacts with TOC to form regulated, potentially carcinogenic **Disinfection Byproducts (DBPs)**. High TOC requires heavy chemical coagulant dosing to remove organics prior to chlorination.
* **Alkalinity:** The water's resistance to pH changes. Coagulation functions best in a narrow, slightly acidic pH range; alkalinity determines how much acid or base buffering is required to achieve that chemistry.

Both parameters traditionally require manual grab samples analyzed in a laboratory with multi-hour to multi-day turnarounds. Plant operators typically discover what arrived **after** it has entered the plant. Having an advance warning of 1 to 4 days gives operators actionable time to pre-calculate chemical dosages, stage staff, and adjust reservoir intake gates.

---

## 2. The Physical Collection System & The Time Lag Paradox

The physical pathway from mountain watershed to tap:
$$\text{Rockies Snowmelt / Rainfall} \longrightarrow \text{South Platte River} \longrightarrow \text{Strontia Springs Reservoir} \longrightarrow \text{Foothills Treatment Plant}$$

### Travel Time vs. Model Lag
* **River Travel Time (~4 Hours):** The primary upstream USGS river sensor (`Site 06707525`) sits on the South Platte River immediately above Strontia Springs Reservoir. Direct hydraulic travel time down the river channel into the reservoir pool is approximately **4 hours**.
* **Reservoir Residence & Mixing Lag (2 to 4 Days):** In predictive models developed by Jake Slawson (Denver Water Data Scientist), the most accurate predictions occur when upstream sensor readings are **lagged by 2 to 4 days**. Strontia Springs is a deep canyon reservoir (~7,700+ acre-feet capacity). Water does not travel as a plug straight to the intake; it enters an existing storage pool where it mixes, stratifies, and deposits sediments. It takes 2 to 4 days for the plume of dissolved organic matter to reach the plant's intake tower.

---

## 3. Water Treatment Chemistry & Turbidity

*(Note: Regulatory limits and general coagulation mechanisms described below represent standard water-treatment engineering knowledge; specific model lags and data reflect this repository's files.)*

### What Is Turbidity?
Turbidity measures the optical clarity or cloudiness of a fluid caused by suspended particles (clay, silt, organic duff, algae, and microorganisms), measured in **NTU** or **FNU** by light scattering at a 90° angle.

### How Turbidity Impacts Water Quality
1. **Disinfection Interference:** Suspended particles can physically shield bacteria and viruses from chlorine and UV disinfection.
2. **Coagulation Demand:** High-turbidity water cannot simply be filtered without rapidly blinding filter beds; it requires chemical coagulants (alum, ferric salts, polymers) to destabilize particles into settleable "floc."
3. **Regulatory Requirements:** Safe drinking water standards (e.g., Colorado CDPHE Regulation 11) require finished water turbidity to remain $\le 0.3\text{ NTU}$ (typically $< 0.1\text{ NTU}$ in practice).

### Does High Upstream Turbidity Always Cause a TOC Spike?
**No.** While strongly correlated during runoff events, high turbidity does not guarantee an unavoidable TOC spike at the plant:
* **Particle Composition (Mineral vs. Organic):** Runoff washing inorganic rock dust, sand, or construction gravel produces extreme turbidity with virtually no organic carbon. Runoff washing forest topsoil and pine needles produces high turbidity **and** high TOC.
* **Mass Loading (`turb_flow`):** A muddy trickle (high turbidity at 50 CFS) is diluted into insignificance upon entering the reservoir. A muddy river flood (high turbidity at 1,500 CFS) delivers massive organic tonnage.
* **Reservoir Settling:** Strontia Springs Reservoir acts as a settling basin where heavy particulate organic carbon drops to the bottom before reaching the withdrawal gates.
* **Storage Dilution:** Localized, short-lived spikes are often absorbed and buffered by cleaner resident reservoir volume.

---

## 4. The Soft Sensor & Predictive Models

A **soft sensor** predicts expensive, slow laboratory measurements using cheap, automated, continuous sensors.

```
Upstream 15-Minute Sensor Feeds:
- USGS 06707525: Turbidity, Specific Conductance, pH, Temp, DO
- Colorado DWR: River Flow (CFS), Stage Height
- NRCS SNOTEL: Snow Water Equivalent (SWE)
- NOAA NCEI: Precipitation & Air Temp
                     │
                     ▼
           Feature Engineering:
           - 2 to 4 Day Time Shifts
           - turb_flow = Turbidity × River Flow
           - Rolling 3-day and 7-day Averages
                     │
                     ▼
             Machine Learning:
             Random Forest Regressor
                     │
                     ▼
  Forecasted Plant Influent TOC & Alkalinity
```

### Key Modeling Insights
* **Failure of Linear Regression:** A straight line (`TOC = a * turb_flow + b`) achieved an $R^2$ of only **0.06** (barely beating mean guessing).
* **Random Forest Performance:** Decision trees capture the nonlinear, conditional behavior of the watershed, reaching an $R^2 \approx 0.66$ on TOC.
* **Feature Importance:** In Jake's models, the engineered mass-loading feature **`turb_flow` accounted for 34% of TOC feature importance**, followed by snowpack (17%) and conductance (15%). Conductance and pH dominated alkalinity prediction (33% and 29%).

---

## 5. Strontia Springs Reservoir: Selective Withdrawal & Depth Profiling

Water quality in a deep reservoir is not vertically uniform. Strontia Springs Dam and Denver Water utilize two critical operational components:

### Selective Withdrawal (Intake Towers)
Strontia Springs Dam features an intake tower with withdrawal gates at multiple depths (surface, mid-depth, and deep hypolimnion). Operators can choose which elevation to withdraw water from into Conduit 26 leading to Foothills.

### The Profiling Sonde (`data/Strontia 0407_0819.xlsx`)
Denver Water deployed an autonomous vertical profiling sonde that travels up and down the water column, recording over 16,000 readings of temperature, turbidity, pH, dissolved oxygen, conductance, and algal pigments:
* **Storm Plumes (Underflows):** Colder, sediment-heavy runoff plunges to the reservoir floor as a dense underflow current. By observing sonde turbidity profiles, operators can pull from upper gates to bypass the bottom sediment layer.
* **Summer Algal Blooms:** Warm, sunlit surface layers can host algae blooms. Profiling sensors allow operators to shift intake withdrawal to mid-depths to avoid taste/odor compounds and filter-clogging biomass.

---

## 6. Empirical Case Study: The August 14–15, 2026 Storm

The repository includes a complete empirical case study of a summer convective monsoon event, replayed on the 3D map using archived NEXRAD Doppler radar tiles and 15-minute USGS telemetry:

### 1. The River Surge
* **Pre-Storm Baseline (Aug 14 afternoon):** The South Platte River above Strontia ran clear at **$2.7\text{ FNU}$** turbidity.
* **The Storm (Aug 14 evening):** Severe convective monsoon cells stalled over Waterton Canyon and the Upper South Platte basin.
* **The Peak (Aug 15 at 1:45 AM):** Turbidity at USGS Gage 06707525 spiked to **$329.0\text{ FNU}$**—more than a **120-fold increase** in under 12 hours.

### 2. The Influent Arrival at Foothills Plant
Data from `data/FoothillsInfluent.csv` tracks the delayed arrival of the pulse:
* **Aug 1 to Aug 15:** Influent TOC was steady at **$1.9\text{–}2.0\text{ mg/L}$**.
* **Aug 16 to Aug 17 (1 to 2 days post-storm):** Plant influent TOC jumped to **$2.5\text{ mg/L}$** (a **25% surge** in organic loading).
* **Aug 18 to Aug 19:** TOC tapered back to $2.3$ and $2.2\text{ mg/L}$.

### 3. The Provisional Sensor Trap
At 1:00 PM on August 14 (hours before rain fell), the USGS specific conductance probe abruptly dropped from $\sim 300\text{ }\mu\text{S/cm}$ to **$38\text{ }\mu\text{S/cm}$**. Because real-time telemetry streams with provisional flags, automated systems must guard against sensor fouling and momentary electrical glitches.

---

## 7. The 3D Interactive Map & Running the Application

The repository includes `design-storm-water-system-3d.html`, an interactive 3D Cesium/MapLibre map showing collection tunnels, Continental Divide diversions, river gages, and live telemetry.

### Running Locally
Modern browsers restrict asynchronous `fetch()` requests when pages are loaded directly from the local file system (`file://` URLs). To view the map with full geojson overlays, run the included Python server:

```bash
# Start the local server
python3 serve.py
```

Then open in your browser:
**`http://localhost:8765/design-storm-water-system-3d`**

---

## 8. Summary Table: What Triggers Special Treatment?

| Event Type | Typical Timing | Hydrological Signature | Impact at Plant | Operator Countermeasures |
|---|---|---|---|---|
| **Summer Monsoon Storm** | July – August (2–6 hr downpours) | Violent turbidity spike ($>100\text{–}300\text{ FNU}$), localized flash flow | 1–2 day delayed TOC surge ($+20\text{–}30\%$) | Increase coagulant dose, monitor depth sonde, select upper withdrawal gates |
| **Spring Snowmelt Runoff** | May – June (weeks) | Sustained high river flow, moderate turbidity, low mineral conductance | Prolonged TOC peak ($>3.0\text{ mg/L}$), depressed alkalinity ($<50\text{–}60\text{ mg/L}$) | Heavy coagulant dosing, add alkaline buffering agents (soda ash/lime), 24/7 filter management |
| **Wildfire Burn-Scar Runoff** | Any post-fire rainfall | Extreme ash/sediment slurry, high particulate carbon | Severe water treatability drop, potential plant shutdown | Divert water, switch to alternative reservoirs, bypass intake |

---

## 9. Data Terms & Notices

All datasets originating from Denver Water (`data/`, `reference/`, `scripts/`, `figures/`) are provisional and provided "as is" under the Denver Water public records disclaimer ([data/TERMS.md](data/TERMS.md)). Public domain data is sourced from USGS, Colorado DWR, USDA NRCS, and NOAA.
