# Aluminium-Smelter-Master-Digital-Twin
A single-file, browser-based digital twin of an aluminium smelter, built for plant managers. It runs a first-principles model on every reduction cell (720 cells across two potlines), rolls the results up into plant KPIs, and lets you create seeded what-if simulations from your own plant or customer inputs.
Aluminium Smelter Master Digital Twin

A single-file, browser-based digital twin of an aluminium smelter, built for plant managers. It runs a first-principles model on every reduction cell (720 cells across two potlines), rolls the results up into plant KPIs, and lets you create seeded what-if simulations from your own plant or customer inputs.

Status: concept prototype with illustrative data. The physics is a clean-room model of published first-principles equations. It has not been validated against plant data and is not the licensed DynaMarc solver. Do not use it for operational decisions.

Screenshot placeholder: add docs/screenshot.png

Contents
What it does
Quick start
How the model works
Scenario Lab
Views and tools
What is real, and what is not
Adapting it to your smelter
Integration roadmap
Dependencies
Known limitations
License and disclaimer
What it does
Cell-level physics for the whole plant. Every cell has voltage and pseudo-resistance, heat balance and superheat, a side-ledge boundary, alumina mass balance, feeding control and anode effects. Plant KPIs are sums over the cells, so a power shortfall reaches the cells through the heat balance.
Editable Scenario Lab. Enter line current, power supply, ambient temperature, customer demand, casthouse capacity, forklifts, alumina delivery, prices and a control strategy, then create a seeded 72-hour simulation over 10 seeds with confidence intervals.
Economics. Cost of production per tonne, LME price, margin, raw material inventory (alumina, anodes, AlF3), work in progress, finished goods and customer backlog.
3D plant. Power plant, alumina and anode supply, potlines (720 instanced pots coloured live from the model), casthouse, warehouse with forklifts, and a water recycling plant, plus a cutaway stage showing the selected cell.
Plant-manager tools. Alarms, OEE loss waterfall, quality gates, RCA Lab, ESG and 5S, physics equations, and an AI copilot.
Quick start

The whole app is one HTML file.

bash
git clone <your-repo-url>
cd <your-repo>
# open index.html in a modern browser, or serve it locally:
python3 -m http.server 8080
# then visit http://localhost:8080

An internet connection is needed on first load for the CDN scripts and fonts (see Dependencies). A desktop browser with WebGL is recommended; the live camera feed uses a second WebGL context.

GitHub Pages: put the file at the repository root as index.html, then enable Pages under Settings → Pages → Deploy from a branch.

Controls
Control	What it does
Pause, 1×, 5×, 15×, 60×	Simulation speed: 1, 5, 15 and 60 simulated minutes per real second
Next alarm	Fast-forwards until a new alarm appears, up to 24 simulated hours
Focus 3D	Hides the side panels
Heli Orbit	Automatic orbit over the site
Walkthrough	W A S D move, Q E turn, Shift run, drag to look; zone buttons teleport
Free Inspect	Drag to orbit, wheel to zoom, click a pot to open it
Plan View	Top-down view with zone health markers
Linear Flow	Value stream from power to dispatch
How the model works
Cell model

Each cell is a lumped model stepped every 30 simulated seconds (ten steps per 5-minute tick).

Domain	Equation (simplified)
Voltage and pseudo-resistance	V = 1.65 + I · R / 1000, Rp = (V − 1.65) / I, with a rise in R below about 2.2 % alumina
Heat balance	M·Cp·dT/dt = Q_joule − Q_loss − Q_bath→ledge
Side ledge (Stefan-type)	ρ·ΔH·dL/dt = k·(T_liq − T_wall)/L − h·(T − T_liq)
Alumina mass balance	dC/dt = (feed − 1.889 · 0.3356 · η · I) / M
Liquidus	empirical, function of alumina and AlF3 excess
Current efficiency	function of superheat, low alumina and anode stock

Other pieces:

Slope controller (legacy). A quadratic moving-window (Savitzky–Golay style, 21 points) estimate of the resistance slope switches between underfeed and overfeed.
Physics control (optional). Feed follows a soft-sensor alumina estimate, anode-cathode distance is trimmed from superheat, and the AlF3 setpoint is adapted. The model assumes the soft sensor detects 75 % of feeder jams within 5 minutes.
Anode effects. Triggered below 0.8 % alumina; voltage jumps and heat input rises for 5 minutes.
Stochastic feeder jams. About 0.03 per cell-day, 20 to 70 minutes each, seeded in batch runs.
Soft sensor. For the selected cell, a Kalman-style scalar correction on alumina uses the voltage innovation. It is not a full extended Kalman filter.
Plant coupling. Σ V·I is the power demand. If supply is lower, line current is scaled by supply / demand.
Batch simulations

For each scenario the app runs 10 seeds of 72 simulated hours on a 72-cell sample (every tenth cell) at a 60-second step, scales results by ten, and reports the mean with a 95 % confidence interval (t distribution, 9 degrees of freedom). The same seeds are used across scenarios so comparisons are paired. Runs are queued in the background so the interface stays responsive.

Economics

cost = alumina (1.92 t/t × price) + power (kWh/t × price) + anodes (0.42 t/t × price, scaled by CE) + fixed cost per day ÷ output + other ($140/t). All prices are editable. The default LME value is an analyst snapshot (about $3,095/t, mid-November 2026) with simulated ticks around it; use the override field or connect a market-data feed for real pricing.

Scenario Lab
Pick a preset (baseline, +5 % amperage creep, +5 % creep with physics control, gas turbine trip) or edit the inputs and choose Create simulation.
Results appear on the scenario card and in the Surge Matrix tool.
The live 3D twin follows the selected scenario. Use Apply to live twin to push edited inputs to the live model.
Inject physical anomalies at any time: gas turbine trip, anode shortage, furnace outage, forklift fault, ambient 46 °C, feeder fault on Line 2, RO membrane fouling.
Views and tools
Tool	Contents
Cells	Heat map of all 720 cells (status, superheat, alumina, ledge), per-cell voltage, alumina, superheat, ledge, CE, anode-effect outlook and an 8-hour trend with the soft-sensor estimate; locate the cell on the 3D stage
Economics	Cost build-up, LME, margin, raw material and finished-goods inventory, backlog
Alarms	P1/P2/P3 list with acknowledge and return-to-normal, work orders, event log
OEE and Quality	Loss waterfall in tonnes per day, casthouse quality gates with a RELEASE or HOLD verdict
RCA Lab	5-Why on the selected cell, fishbone (6M), P&ID trace, reliability simulation with downtime Pareto and single points of failure
ESG and Lean	Carbon intensity (power, anodes, PFC), water recycling and zero liquid discharge, 5S scores, lean wastes
Physics	The model equations with live values for the selected cell
Surge Matrix	Seeded 10-seed results for baseline, creep and trip cases with and without physics control

The right column holds a four-camera virtual CCTV wall with projected ground-truth boxes and an AI copilot.

What is real, and what is not

Computed

Cell physics, power curtailment, throughput, current efficiency, energy per tonne, anode effects
OEE, cost, margin, inventories and backlog
Seeded 10-seed scenario results and the reliability Pareto

Reference or scripted

LME snapshot and simulated ticks
Quality-gate values, carbon, PFC and water factors, 5S scores
MTBF and MTTR values
The single PPE violation at 10:00
The copilot's rule-based fallback answers

Not built or not connected

The licensed DynaMarc solver
PI, SAP, LIMS, warehouse and forklift-tracking feeds
A live market-data feed
A trained PPE detector and automatic P&ID reading
An extended Kalman filter on live plant data
Real photographs or CAD geometry (the 3D scene is procedural)
Adapting it to your smelter

Search for these names in index.html:

Name	What to change
NC, NCELL, mkCell	Cells per line, line count, design current, resistance and heat-loss parameters
INP, D0, CASES	Default inputs and scenario presets
LME, eco()	Price anchor and cost structure
AR, scene block	Zones and 3D layout
REL, PID	Reliability assets with MTBF and MTTR, and the P&ID graph
qual(), esg(), WR	Quality specs, emission factors, water model
OPS	Field operators and routes

The nameplate output (2,548 t/day) is hard-coded in the OEE and reliability functions and should be updated with the cell parameters.

Integration roadmap
Historian and MES: replace the synthetic inputs with a read-only adapter (for example PI Web API) feeding INP and the cell inputs.
Licensed solver: pxStep() is the single model entry point; a licensed solver wrapper can replace it behind the same state interface.
State estimation: replace the scalar soft sensor with an extended or unscented Kalman filter, with asynchronous lab assimilation.
ERP and LIMS: work orders, bath chemistry and batch release.
Control writes: keep advisory-only until shadow-mode validation is complete.
Dependencies

Loaded from CDNs by the page:

Three.js r128 (cdnjs)
Tailwind CSS 3.4.17 play CDN
Lucide 0.469.0 icons (jsDelivr)
Inter and JetBrains Mono (Google Fonts)

For production or restricted networks, vendor these files locally and replace the Tailwind play CDN with a built stylesheet.

AI copilot: when the page is hosted inside Claude, it can use Claude through the artifact sample capability with a live JSON snapshot of the twin. Anywhere else (GitHub Pages, local file) it falls back to the built-in rule-based answers. No API keys are used or stored.

Known limitations
Parameters are illustrative and not calibrated to any plant.
The batch simulation uses a 72-cell sample per seed scaled to 720 cells, so confidence intervals reflect sampling and seed variation, not model uncertainty.
The cell model is lumped; it has no magnetohydrodynamics, no 2D thermal field and no cathode wear.
Results for physics control depend on the assumed feeder-jam detection rate.
The reference scoring (5S, quality gates, ESG factors) must be replaced with plant data before any reporting use.
Tested in a headless browser; not yet tested on a range of real devices.
License and disclaimer

Add your license here (for example MIT).

DynaMarc is a product of GéniSim. This project is independent, is not affiliated with or endorsed by GéniSim, and does not contain GéniSim code. "DynaMarc-style" refers only to the general modelling approach described in published literature.
