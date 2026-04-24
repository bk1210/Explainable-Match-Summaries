# 🏆 Explainable Match Summaries — RAG + SHAP + LLaMA 3.1

<div align="center">

![Python](https://img.shields.io/badge/Python-3.10-blue?style=for-the-badge&logo=python)
![LLaMA](https://img.shields.io/badge/LLaMA_3.1-Groq_API-blueviolet?style=for-the-badge)
![FAISS](https://img.shields.io/badge/FAISS-Vector_Search-orange?style=for-the-badge)
![SHAP](https://img.shields.io/badge/SHAP-Explainability-yellow?style=for-the-badge)
![License](https://img.shields.io/badge/License-MIT-green?style=for-the-badge)

**Generates factually grounded, explainable football match summaries by combining Retrieval-Augmented Generation (RAG) with SHAP — no hallucinations, no black boxes.**

### 🚀 [Try the Live Demo →](https://huggingface.co/spaces/bk1210/Explainable-Match-Summaries)

[![Open in HuggingFace Spaces](https://img.shields.io/badge/🤗-Live%20Demo%20on%20HuggingFace%20Spaces-blue?style=for-the-badge)](https://huggingface.co/spaces/bk1210/Explainable-Match-Summaries)

[Features](#-features) • [How It Works](#-how-it-works) • [Results](#-results) • [Installation](#-installation) • [Usage](#-usage) • [Architecture](#-architecture) • [Tech Stack](#-tech-stack) • [Contact](#-contact)

</div>

---

## 📌 Overview

General-purpose LLMs produce fluent football summaries but hallucinate statistics. Ask one about a specific match — it might get the winner right but fabricate the scoreline, goalscorers, or disciplinary events. This project fixes that.

**Explainable Match Summaries** grounds every summary in verified UCL-2025 match statistics using a RAG pipeline, and adds a SHAP explainability layer so analysts can see exactly which features drove the model's assessment.

Three core contributions:
- 📚 **Self-Curated UCL-2025 Dataset** — 189 UEFA Champions League 2025 matches, 142 columns per match, **manually collected by the author directly from the official UEFA website (uefa.com)** — independent of any previously published database or third-party data provider
- 🔍 **RAG Pipeline** — Sentence-BERT + FAISS retrieval feeds verified match facts directly into the LLaMA 3.1 prompt, eliminating hallucination within the knowledge base scope
- 📊 **SHAP Explainability** — LinearExplainer over key match features surfaces which stats most influenced the generated summary

---

## 📂 Dataset

> ⚠️ **Original Dataset — Do Not Confuse With Third-Party Sources**
>
> The `UCL-2025_Team_Stats.xlsx` file included in this repository was **independently curated by the author** by manually collecting match statistics directly from the **official UEFA website ([uefa.com](https://www.uefa.com))** across all stages of the 2024/25 UEFA Champions League season. It is **not derived from Opta, StatsBomb, or any other third-party data provider** and was first published as part of this project.

| Property | Value |
|---|---|
| Matches | 189 |
| Columns | 142 |
| Phases | League Phase, Play-Off, Round of 16, Quarter-finals, Semi-finals, Final |
| Source | Official UEFA website (manually collected) |
| Missing values | 0 |
| Duplicates | 0 |

Stats covered: goals, shots, shots on target, possession, assists, passes, passing accuracy, fouls, yellow/red cards, xG, corners, offsides, attacks, clear chances.

---

## ✨ Features

### 🔍 RAG Pipeline
- Match stats converted to natural language evidence chunks per match
- Sentence-BERT `all-MiniLM-L6-v2` encodes 189 evidence strings → 384-d dense vectors
- FAISS `IndexFlatL2` stores and retrieves top-k most semantically similar matches
- Retrieved evidence inserted into a structured prompt — LLaMA 3.1 can only use verified facts

### 🤖 LLaMA 3.1 via Groq API
- `meta-llama/Llama-3.1-8b-instant` via Groq API
- Lightning fast inference — responses in under 2 seconds
- Temperature = 0.2 for factual consistency over creative diversity
- Fully free tier — no GPU required, no paid API

### 📊 SHAP Explainability
- LinearRegression proxy fitted on 4 key match features (goals scored, goals conceded, goal diff, win indicator)
- SHAP LinearExplainer computes per-match feature attributions
- Global importance ranking visualized as bar chart

### 📈 Exploratory Data Analysis
- Goals distribution by tournament phase and match outcome
- Top 12 teams by total goals scored
- Possession vs match outcome analysis
- Pearson correlation heatmap of home team match statistics
- Attacking efficiency — shots on target vs goals per team

---

## 🖥️ Demo

### 🔴 Live App
> **[https://huggingface.co/spaces/bk1210/Explainable-Match-Summaries](https://huggingface.co/spaces/bk1210/Explainable-Match-Summaries)**
> Select any two teams, choose a query type, and get an instant grounded match summary with RAG evidence and SHAP feature importance chart.

### Example Output
```
Query   → "Summarize the match between Real Madrid and Liverpool"

RAG Output → "In the UEFA Champions League, Real Madrid was eliminated
              by Liverpool in the league phase with a 2-0 win for Liverpool.
              Liverpool had 18 shots; Real Madrid had 9 shots.
              Liverpool ball possession: 60%." ✅
```

### SHAP Feature Attribution
```
Match   → Real Madrid 5 - 2 Borussia Dortmund
Top feature → Goal Difference (+3): SHAP = +1.00
              Goals Scored (5):     SHAP = +0.31
              Goals Conceded (2):   SHAP = -0.31
```

### RAG vs Baseline
```
Query   → "Barcelona's highest-scoring match?"

Baseline LLM → Fabricates a scoreline not present in the dataset ❌
RAG Output   → Grounded output citing only verified UCL-2025 facts ✅
```

---

## 🚀 Installation

### Prerequisites
- Python 3.10 or higher
- Groq API key (free at console.groq.com)

### Step 1 — Clone the Repository
```bash
git clone https://github.com/bk1210/Explainable-Match-Summaries.git
cd Explainable-Match-Summaries
```

### Step 2 — Install Dependencies
```bash
pip install -r requirements.txt
```

### Step 3 — Run the Notebook
```bash
jupyter notebook Code.ipynb
```

---

## 📖 Usage

### Running the Full Pipeline

Open `Code.ipynb` and run all cells — the notebook handles:

1. Loading `UCL-2025_Team_Stats.xlsx` (189 matches, 142 columns)
2. EDA — goals distribution, top scorers, possession analysis, correlation heatmap
3. Evidence builder — converts each match row to a natural language string
4. Sentence-BERT embedding + FAISS indexing (189 × 384-d vectors)
5. RAG retrieval function (top-k=3 by L2 similarity)
6. LLaMA 3.1 grounded summary generation via Groq API
7. SHAP LinearExplainer — global + per-match feature importance

### Query the System
```python
query = "Real Madrid best attacking performance?"
summary = generate_summary(query, k=3)
print(summary)
# → Grounded output citing only verified UCL-2025 facts
```

---

## 🏗️ Architecture

### Full Pipeline

```
UCL-2025 Dataset (189 matches, 142 columns)
[Manually curated from uefa.com by the author]
    │
    ▼
── OFFLINE PIPELINE (Indexing) ──────────────────────
    │
    ▼
Evidence Builder
["Match between Real Madrid and Dortmund. Real Madrid scored 5..."]
    │
    ▼
Sentence-BERT (all-MiniLM-L6-v2) → 384-d dense embeddings
    │
    ▼
FAISS IndexFlatL2 → Vector store of 189 match embeddings

── ONLINE PIPELINE (Query) ───────────────────────────

User Query → Sentence-BERT encode → FAISS top-k retrieval
    │
    ▼
Retrieved Evidence (top-3 most similar matches)
    │
    ▼
Structured Prompt → LLaMA 3.1 via Groq API
    │
    ▼
Grounded Match Summary
    │
    ▼
SHAP LinearExplainer → Feature Importance Scores
    │
    ▼
Final Output: Summary + SHAP Attribution
```

### Project Structure

```
Explainable-Match-Summaries/
│
├── Code.ipynb                        # Full pipeline — EDA, RAG, LLaMA, SHAP
├── UCL-2025_Team_Stats.xlsx          # Original self-curated UCL 2025 dataset
├── app.py                            # Streamlit live demo app
├── requirements.txt                  # Python dependencies
└── README.md                         # Project documentation
```

---

## 📊 Results

### RAG Retrieval Quality

| Metric | RAG (Top-k) | Random Baseline |
|---|---|---|
| Mean cosine similarity | **0.903** | 0.373 |
| Standard deviation | 0.048 | 0.082 |
| Proportion ≥ 0.7 | **100%** | < 5% |

### SHAP Feature Importance

| Feature | Mean \|SHAP\| | Direction |
|---|---|---|
| Goal Difference | **1.000** | Positive |
| Goals Scored | 0.312 | Positive |
| Goals Conceded | 0.310 | Negative |
| Binary Win Indicator | 0.089 | Positive |

### System Comparison

| Criterion | Baseline LLM | This System |
|---|---|---|
| Factual accuracy | Low–Medium | **High** |
| Hallucination rate | High | **Negligible** |
| Interpretability | None | **SHAP-based** |
| Mean retrieval similarity | N/A | **0.903** |
| Deployment cost | API-dependent | **Free** |

---

## 🛠️ Tech Stack

| Technology | Purpose |
|---|---|
| Python 3.10 | Core language |
| Sentence-Transformers | all-MiniLM-L6-v2 dense embeddings |
| FAISS | Vector indexing and top-k retrieval |
| LLaMA 3.1 via Groq API | Fast free LLM inference |
| SHAP | LinearExplainer, feature importance |
| scikit-learn | LinearRegression proxy for SHAP |
| Streamlit | Live demo web app |
| Pandas / NumPy | Data processing |
| Matplotlib / Seaborn | EDA visualisations |

---

## 📦 Dependencies

```txt
pandas>=2.0.0
numpy>=1.24.0
sentence-transformers>=2.2.0
faiss-cpu>=1.7.4
groq>=0.9.0
shap>=0.42.0
scikit-learn>=1.3.0
matplotlib>=3.7.0
seaborn>=0.12.0
openpyxl>=3.1.0
streamlit>=1.32.0
```

Install with:
```bash
pip install -r requirements.txt
```

---

## 🔮 Future Improvements

- [ ] Enrich evidence strings with possession, shots on target, xG, and player-level stats
- [ ] Hybrid retrieval — combine dense vector search with structured relational filtering
- [ ] SHAP / LIME attribution at the language model token level
- [ ] Real-time pipeline connected to StatsBomb / Opta live match feeds
- [ ] Multi-modal integration — player tracking + formation metadata
- [ ] Multilingual summaries (Spanish, French, Portuguese, German)

---

## 📄 License

This project is licensed under the MIT License — see the [LICENSE](LICENSE) file for details.

---

## 👤 Contact

**Bharath Kesav R**
- 📧 Email: bharathkesav1275@gmail.com
- 🐙 GitHub: [@bk1210](https://github.com/bk1210)
- 🎓 Institution: Amrita Vishwa Vidyapeetham, Coimbatore

---

## 🙏 Acknowledgements

- [UEFA](https://www.uefa.com) — official source for all UCL 2025 match statistics (manually collected)
- [Groq](https://console.groq.com) — for the free LLaMA 3.1 inference API
- [UKPLab](https://github.com/UKPLab/sentence-transformers) — for Sentence-BERT
- [Facebook AI](https://github.com/facebookresearch/faiss) — for FAISS
- [Lundberg & Lee (2017)](https://proceedings.neurips.cc/paper/2017/hash/8a20a8621978632d76c43dfd28b67767-Abstract.html) — SHAP framework

---

<div align="center">

**⭐ If you found this project useful, please give it a star on GitHub! ⭐**

[![Open in HuggingFace Spaces](https://img.shields.io/badge/🤗-Try%20Live%20Demo-blue?style=for-the-badge)](https://huggingface.co/spaces/bk1210/Explainable-Match-Summaries)

*Built with ❤️ for factual, transparent football analytics*

</div>
