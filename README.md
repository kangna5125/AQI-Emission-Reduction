# 🌆 AQI Reduction Through Traffic Emission Control — SUMO Simulation

> *Everyone measures the AQI. We decided to reduce it.*

---

## 💡 Why We Built This

Delhi's air quality crisis has been studied, reported, and talked about for years. Every winter, headlines scream about AQI crossing 400. Sensors go up, reports come out, and then... nothing changes. The conversation always stops at **measurement**.

We wanted to go one step further — not just monitor the problem, but actively simulate **what happens when you intervene**.

Vehicular traffic is one of the largest contributors to urban air pollution in Indian cities. So we asked: what if you could test emission-reduction strategies on a real city road network, without any real-world disruption? What would actually work — rerouting vehicles, capping speeds, banning heavy trucks? And by how much would the AQI drop?

That's exactly what this project does. Using **SUMO** (a professional-grade traffic simulator used by researchers and city planners worldwide) and a real OpenStreetMap network of the **India Gate / Rajpath / Major Dhyan Chand Nagar** area of Delhi, we simulate baseline traffic, train an AI model on the emission data, and then apply targeted control strategies — measuring the reduction at every step.

---

## 🔬 How It Works

### Phase 1 — Baseline (No Controls)
The simulation runs normal Delhi traffic with no interventions. Every road segment is monitored for vehicle count, average speed, heavy vehicle share, congestion level, and estimated emissions. This data is saved and used to train the AI model.

### AI Training (RandomForest Classifier)
A `RandomForestClassifier` is trained on the Phase 1 road data. It learns to classify each road segment into one of five action categories based on its emission and congestion profile:
- `heavy_vehicle_ban` — high heavy vehicle share, severe emissions
- `rerouting` — congested corridors where diverting traffic helps
- `speed_harmonization` — roads where speed variance drives extra emissions
- `idling_restriction` — slow-moving stretches where idling is the main source
- `no_action` — roads already within acceptable limits

**Model accuracy: ~56.9%** on held-out test data (5,000 samples, 1,250 test). The model performs best on `no_action` (95% precision) and `heavy_vehicle_ban` (71% precision) — the two highest-stakes categories.

### Phase 2 — Controls Active
The trained model's recommendations are applied in real time:
- **Dynamic rerouting** — ~30% of vehicles on flagged corridors are diverted to alternate paths
- **Speed harmonization** — vehicle speeds capped at 35 km/h on identified roads to reduce acceleration-burst emissions
- **Heavy vehicle ban** — trucks and buses removed from hotspot road segments
- **Idling restriction** — minimum speed of 15 km/h enforced on slow-moving stretches

A 4-panel comparison chart and summary report are auto-generated after both phases run.

---

## 📊 Key Results

| Metric | Outcome |
|---|---|
| AI model accuracy | ~56.9% (5,000-sample road dataset) |
| Best-classified action | `no_action` — 97.9% precision |
| AQI scenarios simulated | Moderate (110–200), High (220–340), Severe (360–500) |
| Strategies applied | Speed cap, heavy vehicle ban, rerouting, idling restriction |
| Output | AQI before/after chart, per-pollutant emission comparison, summary report |

> The simulation consistently shows AQI reduction in Phase 2 when controls are active. The exact percentage varies by scenario and run duration — the comparison chart generated after each run gives the precise figures for that session.

---

## 🛠️ Tech Stack

| Component | Technology |
|---|---|
| Traffic Simulation | SUMO + TraCI (Python control API) |
| AI Model | `scikit-learn` — RandomForestClassifier |
| Data & Charting | `numpy`, `matplotlib` |
| City Network | OpenStreetMap — Delhi, India Gate / Rajpath area |
| AQI Data | WAQI API (live) or built-in mock scenarios |

---

## 📁 Project Structure

```
aqi-emission-control/
├── run_simulation.py       # Main entry point — Phase 1 → train → Phase 2
├── phase_runner.py         # Core simulation runner for each phase
├── ai_model.py             # RandomForest training and road-level prediction
├── aqi_module.py           # AQI calculation, zone fetching, solution logic
├── emission_model.py       # Per-vehicle, per-road emission estimation
├── chart_generator.py      # 4-panel comparison chart + summary.txt
├── digital_twin.py         # Real-time road state tracker
├── setup_map.py            # One-time city network setup from OSM (run once only)
├── fix_routes.py           # Route file validation and repair
├── fix_config.py           # SUMO config validation
├── RUN_ME.bat              # Windows quick-launch
├── city/
│   ├── city.net.xml        # Road network (Delhi OSM)
│   ├── city.rou.xml        # Vehicle routes
│   ├── city.sumocfg        # SUMO configuration
│   ├── city.vtype.xml      # Vehicle types (cars, trucks, buses, bikes)
│   └── map.osm             # Raw OpenStreetMap source data
└── output/
    ├── road_action_training_data.csv   # Phase 1 road-level dataset
    ├── road_action_model.pkl           # Trained RandomForest model
    ├── road_action_training_report.txt # Accuracy + classification report
    ├── comparison_chart.png            # Phase 1 vs Phase 2 visual (auto-generated)
    └── summary.txt                     # AQI & emission reduction report
```

---

## ⚙️ Setup & Run

### Prerequisites
- [SUMO](https://sumo.dlr.de/docs/Downloads.php) installed, with `SUMO_HOME` set in your environment variables
- Python 3.11+

### Install Python dependencies
```bash
pip install -r requirements.txt
```

### Validate routes before first run
```bash
python fix_routes.py
```

### Run the full simulation (Phase 1 + AI training + Phase 2)
```bash
# With SUMO GUI
python run_simulation.py --both --duration 600

# Headless / faster
python run_simulation.py --both --no-gui --duration 600
```

### Check environment setup
```bash
python run_simulation.py --check-env
```

> ⚠️ Do **not** re-run `setup_map.py` if the `city/` folder already exists — it will overwrite the prebuilt network.

---

## 📤 Output Files

| File | Description |
|---|---|
| `output/road_action_training_data.csv` | Road-level emission dataset from Phase 1 |
| `output/road_action_model.pkl` | Trained RandomForest model (reusable) |
| `output/road_action_training_report.txt` | Model accuracy and per-class metrics |
| `output/comparison_chart.png` | 4-panel AQI & emission comparison (auto-opened) |
| `output/summary.txt` | Human-readable AQI reduction report |

---

*Built as part of B.Tech CSE coursework at Bennett University, Greater Noida.*
