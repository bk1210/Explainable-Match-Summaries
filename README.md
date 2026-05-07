# ⚽ ClubIQ — Full Stack Football Analytics Platform

<div align="center">

![Python](https://img.shields.io/badge/Python-3.10-blue?style=for-the-badge&logo=python)
![StatsBomb](https://img.shields.io/badge/StatsBomb-Open_Data-red?style=for-the-badge)
![Streamlit](https://img.shields.io/badge/Streamlit-Live-ff4b4b?style=for-the-badge)
![SQLite](https://img.shields.io/badge/SQLite-Database-lightblue?style=for-the-badge)
![PowerBI](https://img.shields.io/badge/Power_BI-Dashboard-yellow?style=for-the-badge)
![License](https://img.shields.io/badge/License-MIT-green?style=for-the-badge)

**A production-ready football analytics platform delivering Match Intelligence, Player Ratings, and Load Monitoring — built on 1.29 million StatsBomb events.**

### 🚀 [Try the Live Demo →](https://huggingface.co/spaces/bk1210/clubiq-football-analytics)

[![Open in HuggingFace Spaces](https://img.shields.io/badge/⚽-Live%20Demo%20on%20HuggingFace%20Spaces-blue?style=for-the-badge)](https://huggingface.co/spaces/bk1210/clubiq-football-analytics)

[Overview](#-overview) • [Dataset](#-dataset) • [Modules](#-modules) • [Results](#-results) • [Installation](#-installation) • [Architecture](#-architecture) • [Tech Stack](#-tech-stack) • [Contact](#-contact)

</div>

---

## 📌 Overview

Football clubs — especially at the grassroots and semi-professional level — make critical decisions on team selection, training loads, and tactical preparation without access to structured data. ClubIQ bridges that gap.

Built entirely on **StatsBomb professional-grade open event data**, ClubIQ is a three-module analytics platform that gives coaching staff, analysts, and club management actionable insights through three interactive dashboards — Streamlit, Power BI, and Tableau.

Three core contributions:
- 🏟️ **Match Intelligence** — Team-level shooting efficiency, pressing intensity, and pass accuracy across an entire season
- 🌟 **Player Ratings** — Composite 100-point rating computed from goals, shots, dribbles, pressures, interceptions, blocks, clearances, and pass accuracy
- 💪 **Load Monitoring** — Per-player fatigue risk score derived from high-intensity event frequency across all matches

---

## 📂 Dataset

| Property | Value |
|---|---|
| Source | StatsBomb Open Data |
| Competition | La Liga 2015/16 |
| Matches | 380 |
| Total Events | 1,295,354 |
| Players Tracked | 546 |
| Storage | SQLite (`clubiq.db`) |
| Tables | `matches`, `players`, `events` |

Event types covered: Pass, Shot, Carry, Pressure, Dribble, Block, Clearance, Interception, Ball Recovery, Duel, Foul, Substitution and more.

---

## ✨ Modules

### 🏟️ Module 1 — Match Intelligence
- Total goals and shots per team across the season
- Pressing intensity: total pressure events per team
- Pass accuracy: completed vs attempted passes per team per match
- Top 10 team rankings across all three metrics

### 🌟 Module 2 — Player Ratings
- Per-player aggregation of 8 action types from 1.29M events
- Composite rating formula with weighted contributions:
  - Goals × 8.0 | Shots × 0.5 | Dribbles Won × 1.5
  - Pressures × 0.3 | Interceptions × 2.0 | Blocks × 1.5
  - Clearances × 1.0 | Pass Accuracy × 0.2
- Normalized to 100-point scale
- Interactive team filter and Top N slider

### 💪 Module 3 — Load Monitoring
- Per-player, per-match load score from high-intensity actions
- Fatigue risk % = high-load matches / total matches × 100
- Top 10% threshold flagging for overload detection
- Scatter plot: avg load vs fatigue risk with match volume sizing

---

## 🔑 Key Results

- 🥇 **Top Rated Player:** Neymar da Silva Santos Junior — Barcelona (100.0)
- 🔵 **Top Goals:** Luis Alberto Suárez Díaz — Barcelona (40 goals)
- ⚠️ **Highest Fatigue Risk:** Augusto Matías Fernández — Celta Vigo (73.3%)
- 🏆 **Most Goals — Team:** Barcelona (109) vs Real Madrid (108)
- 🔴 **Top Pressing Team:** Valencia CF

---

## 🖥️ Demo

### Live App
> **[https://huggingface.co/spaces/bk1210/clubiq-football-analytics](https://huggingface.co/spaces/bk1210/clubiq-football-analytics)**
> Navigate across 3 pages — Match Intelligence, Player Ratings, Load Monitoring — with interactive filters and Plotly charts.

### Example — Player Ratings Table
Rank  Player                        Team          Goals  Pass Acc  Rating
1     Neymar da Silva Santos Junior  Barcelona     26     —         100.00
2     Asier Illarramendi Andonegi    Real Sociedad  1     —          95.25
3     Antoine Griezmann              Atlético       22     —          95.21
4     Gonzalo Escalante              Eibar           3     —          94.42
5     Lionel Andrés Messi Cuccittini Barcelona     26     —          92.28

### Example — Load Monitoring Table

Player                    Team              Avg Load  High Load Matches  Fatigue Risk
Augusto Matías Fernández  Celta Vigo        120.6     11/15              73.3%
Gabriel Fernández Arenas  Atlético Madrid   104.6     25/35              71.4%
Pedro Mosquera Parada     RC Deportivo      105.4     25/37              67.6%

---

## 🚀 Installation

### Prerequisites
- Python 3.10 or higher
- Internet connection (for StatsBomb API pull)

### Step 1 — Clone the Repository
```bash
git clone https://github.com/bk1210/clubiq-football-analytics
cd clubiq-football-analytics
```

### Step 2 — Install Dependencies
```bash
pip install -r requirements.txt
```

### Step 3 — Run the App
```bash
streamlit run app.py
```

---

## 📖 Usage

### Rebuilding the Database from Scratch

Run the Kaggle notebook to:
1. Pull all 380 La Liga 2015/16 matches via `statsbombpy`
2. Extract and clean 1,295,354 events into SQLite
3. Engineer match-level and player-level features
4. Export 6 CSV files for dashboard consumption

### Querying the Database
```python
import sqlite3, pandas as pd

conn = sqlite3.connect('clubiq.db')

top_scorers = pd.read_sql("""
    SELECT player_name, team,
           SUM(CASE WHEN event_type='Shot' AND outcome='Goal' THEN 1 ELSE 0 END) as goals
    FROM events
    WHERE player_name IS NOT NULL
    GROUP BY player_name, team
    ORDER BY goals DESC
    LIMIT 10
""", conn)

print(top_scorers)
```

---

## 🏗️ Architecture

StatsBomb Open Data API
│
▼
── OFFLINE PIPELINE (Data Engineering) ──────────────
statsbombpy → Pull 380 matches + 1.29M events
│
▼
Pandas Feature Engineering
(event_type extraction, outcome parsing, location parsing)
│
▼
SQLite Database
┌─────────────────────────────────────┐
│  matches  │  players  │   events   │
│  380 rows │  546 rows │  1.29M rows│
└─────────────────────────────────────┘
│
▼
── ANALYTICS MODULES ─────────────────────────────────
┌──────────────────┐ ┌──────────────────┐ ┌──────────────────┐
│ Match            │ │ Player           │ │ Load             │
│ Intelligence     │ │ Ratings          │ │ Monitoring       │
│                  │ │                  │ │                  │
│ shots, pressure, │ │ composite 100-pt │ │ fatigue risk %   │
│ pass accuracy    │ │ rating formula   │ │ per player       │
└──────────────────┘ └──────────────────┘ └──────────────────┘
│                    │                    │
▼                    ▼                    ▼
── VISUALIZATION LAYER ───────────────────────────────
Streamlit + Plotly     Power BI Desktop     Tableau Public
(HuggingFace Deploy)   (Management View)    (Tactical View)

### File Structure

clubiq-football-analytics/
│
├── app.py                  # Streamlit dashboard (3 pages)
├── requirements.txt        # Python dependencies
├── match_shots.csv         # Module 1 — shot & goal data
├── match_passes.csv        # Module 1 — pass accuracy data
├── match_pressure.csv      # Module 1 — pressing data
├── player_ratings.csv      # Module 2 — composite ratings
├── load_summary.csv        # Module 3 — fatigue risk data
└── matches.csv             # Base match results

---

## 🛠️ Tech Stack

| Layer | Tools |
|---|---|
| Data Pull | `statsbombpy` |
| Processing | `pandas`, `numpy` |
| Storage | `SQLite` via `sqlalchemy` |
| Dashboard 1 | `Streamlit` + `Plotly` — HuggingFace Spaces |
| Dashboard 2 | `Power BI Desktop` |
| Dashboard 3 | `Tableau Public` |
| Notebook | Kaggle (NVIDIA T4 GPU) |

---

## 📦 Dependencies

```txt
statsbombpy>=1.0.0
pandas>=2.0.0
numpy>=1.24.0
sqlalchemy>=2.0.0
streamlit>=1.32.0
plotly>=5.18.0
tqdm>=4.65.0
```

Install with:
```bash
pip install -r requirements.txt
```

---

## 🔮 Future Improvements

- [ ] Multi-season support — pull all available StatsBomb competitions
- [ ] xG model — train expected goals model from shot location + context
- [ ] Pass network graphs — per-match player connection maps
- [ ] Real-time pipeline — connect to live match feeds via StatsBomb API
- [ ] Club data integration — plug in any club's proprietary CSV match data
- [ ] Opposition profiling — automated pre-match opponent report generation
- [ ] Multilingual dashboard — Spanish, Portuguese, French support

---

## 📌 Note for Clubs

> This platform is built on professional-grade StatsBomb event data.
> The exact same pipeline can be deployed on any club's internal match data
> to generate real-time player ratings, fatigue alerts, and match intelligence
> reports within days — no infrastructure cost, no data science team required.

---

## 📄 License

This project is licensed under the MIT License — see the [LICENSE](LICENSE) file for details.

---

## 👤 Contact

**Bharath Kesav R**
- 📧 Email: bharathkesav1275@gmail.com
- 🐙 GitHub: [@bk1210](https://github.com/bk1210)
- 🎓 Institution: Amrita Vishwa Vidyapeetham, Coimbatore
- 🤗 HuggingFace: [bk1210](https://huggingface.co/bk1210)

---

## 🙏 Acknowledgements

- [StatsBomb](https://statsbomb.com) — for the open event data
- [HuggingFace](https://huggingface.co) — for free Spaces deployment
- [Plotly](https://plotly.com) — for interactive visualizations
- [Streamlit](https://streamlit.io) — for the web app framework

---

<div align="center">

**⭐ If you found this project useful, please give it a star on GitHub! ⭐**

[![Open in HuggingFace Spaces](https://img.shields.io/badge/⚽-Try%20Live%20Demo-blue?style=for-the-badge)](https://huggingface.co/spaces/bk1210/clubiq-football-analytics)

*Built with ❤️ for data-driven football clubs*

</div>
