# Crm-pipeline
End-to-end CRM lead management system for HelpAge Mumbai — raw data cleaned &amp; analysed in Python (pandas + Jupyter), results delivered as Excel dashboards for the sales team. Covers follow-up tracking, priority scoring, agent performance, and daily call list generation.
From raw CRM export → Python analysis → Excel delivery — a complete lead management pipeline for a 4,500-lead sales campaign.
# 🏥 HelpAge Mumbai — CRM Lead Management Pipeline

> **From raw CRM export → Python analysis → Excel delivery**
> A complete end-to-end lead management system for a 4,500-lead sales campaign.

![Python](https://img.shields.io/badge/Python-3.10-blue?style=flat-square&logo=python)
![Pandas](https://img.shields.io/badge/Pandas-2.0-150458?style=flat-square&logo=pandas)
![Jupyter](https://img.shields.io/badge/Jupyter-Notebook-orange?style=flat-square&logo=jupyter)
![Excel](https://img.shields.io/badge/Microsoft-Excel-217346?style=flat-square&logo=microsoft-excel)
![Status](https://img.shields.io/badge/Status-Active-success?style=flat-square)

---

## 📌 What This Project Does

Most CRM tools give you raw data — thousands of lead records with no clear sense of who to call first, who was missed, or how the team is performing. This project solves that.

It takes a raw CRM export from the **HelpAge Mumbai** campaign and runs it through a two-layer system:

| Layer | Tool | Who Uses It | Purpose |
|---|---|---|---|
| 🐍 **Analysis Layer** | Python + Jupyter | Data / Operations team | Clean, classify, score, and export |
| 📊 **Delivery Layer** | Microsoft Excel | Sales agents + Managers | Open, read, and act on the output |

**Python is the engine. Excel is the steering wheel.**

---

## 📊 Campaign Snapshot

| Metric | Value |
|---|---|
| 📋 Total Leads | 4,500 |
| 📞 Leads Contacted | 503 (11.2%) |
| ✅ Converted to Customers | 2 |
| ⏰ Overdue Follow-ups | 324 |
| 👥 Sales Agents | 3 |
| 🗓️ Campaign Period | May 2026 |

---

## 🗂️ Repository Structure

```
helpage-mumbai-crm-pipeline/
│
├── 📓 python/
│   ├── report.ipynb                    # Main analysis notebook (run this first)
│   └── requirements.txt                # Python dependencies
│
├── 📊 excel/
│   ├── CRM_Dashboard.xlsx              # Master Excel dashboard for managers
│   ├── today_call_list.xlsx            # Daily call list — top 100 prioritised leads
│   ├── calls_Poonam_Sonawale.xlsx      # Poonam's personal call sheet
│   ├── calls_Preeti_Mahalinge.xlsx     # Preeti's personal call sheet
│   └── calls_Riya_Singh.xlsx          # Riya's personal call sheet
│
├── 📁 data/
│   └── Leads_Helpage_Mumbai.csv        # Raw CRM export (source of truth)
│
├── 📁 outputs/
│   ├── clean_data.csv                  # Cleaned dataset with priority columns
│   ├── team_performance.csv            # Agent-level metrics
│   └── overdue_leads.csv              # 324 leads needing immediate callbacks
│
└── README.md
```

---

## 🔄 End-to-End Workflow

```
┌─────────────────────────────────────────────────────────────┐
│                        RAW DATA                             │
│             Leads_Helpage_Mumbai.csv (4,500 rows)           │
└──────────────────────────┬──────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────┐
│                   PYTHON LAYER (Jupyter)                    │
│                                                             │
│  1. Clean & Standardise  →  dates, phones, column names     │
│  2. Classify Follow-ups  →  Overdue / Today / Upcoming      │
│  3. Score Priority       →  High / Medium / Low             │
│  4. Analyse Team         →  contact rate, conversion rate   │
│  5. Build Call List      →  top 100, sorted by urgency      │
│  6. Split by Agent       →  individual CSV per agent        │
│                                                             │
└──────────────────────────┬──────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────┐
│                   EXCEL LAYER (Delivery)                    │
│                                                             │
│  📊 Manager Dashboard    →  pipeline overview + charts      │
│  📋 Today's Call List    →  100 prioritised leads           │
│  📁 Agent Call Sheets    →  one file per agent              │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## 🐍 Python Layer — What the Notebook Does

The notebook (`python/report.ipynb`) is split into 6 clear sections. Each section has a plain-English explanation so anyone on the team can follow along.

### Section 1 — Data Loading & Cleaning
- Loads the raw CRM CSV export
- Normalises column names (removes spaces, lowercases)
- Parses date columns (`creation_date`, `last_call_date`, `next_followup_date`)
- Cleans phone numbers (removes non-numeric characters)
- Standardises lead status casing

### Section 2 — Follow-up Classification
Classifies every lead into one of four buckets:

```python
def classify_followup(row):
    if pd.isna(row['next_followup_date']):   return "No Follow-up"
    elif row['next_followup_date'] < today:  return "Overdue"
    elif row['next_followup_date'] == today: return "Today"
    else:                                    return "Upcoming"
```

| Status | Meaning | Count |
|---|---|---|
| 🔴 No Follow-up | No next call ever scheduled | 4,170 |
| 🟡 Overdue | Follow-up date passed — call was missed | 324 |
| 🟢 Upcoming | Future callback correctly planned | 6 |

### Section 3 — Priority Scoring
Assigns each lead a priority level to determine call order:

```python
priority_map = {
    "Overdue":      "High",
    "Today":        "Medium",
    "Upcoming":     "Low",
    "No Follow-up": "No Action"
}
```

### Section 4 — Funnel Analysis
Calculates and visualises the full sales funnel:
- Total leads → Contacted → In Progress → Converted
- Contact rate and conversion rate per stage

### Section 5 — Team Performance
Groups data by agent and computes:

```python
team = df.groupby('assigned_to').agg(
    total     = ('assigned_to', 'count'),
    contacted = ('lead_status', lambda x: x.isin(['In-Progress','Lost','Converted']).sum()),
    converted = ('lead_status', lambda x: (x == 'Converted').sum())
)
team['contact_rate']    = team['contacted'] / team['total']
team['conversion_rate'] = team['converted'] / team['contacted']
```

### Section 6 — Daily Call List Generation
Builds the prioritised call list and exports agent-wise files:

```python
# Combine overdue + today + new uncontacted
call_list = pd.concat([overdue_calls, today_calls, new_calls])

# Sort by priority → date
call_list = call_list.sort_values(['priority_rank', 'next_followup_date'])

# Top 100 for the day
call_list_today = call_list.head(100)

# Export one file per agent
for agent in call_list_today['assigned_to'].unique():
    agent_df = call_list_today[call_list_today['assigned_to'] == agent]
    agent_df.to_csv(f'calls_{agent.replace(" ", "_")}.csv', index=False)
```

---

## 📊 Excel Layer — What Each File Contains

### `CRM_Dashboard.xlsx` — For Managers
The master dashboard that gives a complete picture of the pipeline at a glance.

| Sheet | Contents |
|---|---|
| 📌 Summary | KPI cards — total leads, contact rate, conversions, overdue count |
| 📊 Funnel | Visual funnel chart from imported → contacted → converted |
| 👥 Team Performance | Agent comparison table with contact & conversion rates |
| 📅 Follow-up Health | Breakdown of overdue / upcoming / no follow-up leads |
| 🔴 Overdue Leads | Full list of 324 leads needing immediate callbacks |

### `today_call_list.xlsx` — For the Full Team
100 leads sorted by priority for today. Columns include:

| Column | Description |
|---|---|
| `contact_number` | Phone number to call |
| `assigned_to` | Which agent owns this lead |
| `lead_status` | Current status in the pipeline |
| `followup_status` | Overdue / Today / No Follow-up |
| `priority` | High / Medium / Low |
| `next_followup_date` | When the follow-up was due |

### `calls_[Agent_Name].xlsx` — For Each Agent
Each agent gets their own file with only their leads. No confusion, no overlap. Open it in the morning, start calling.

---

## 🚀 Getting Started

### Prerequisites

```bash
pip install pandas numpy matplotlib openpyxl jupyter
```

Or install everything at once:

```bash
pip install -r python/requirements.txt
```

### Run the Analysis

```bash
# 1. Clone the repo
git clone https://github.com/your-username/helpage-mumbai-crm-pipeline.git
cd helpage-mumbai-crm-pipeline

# 2. Place your CRM export in the data folder
cp your_export.csv data/Leads_Helpage_Mumbai.csv

# 3. Launch the notebook
jupyter notebook python/report.ipynb

# 4. Run all cells — Excel and CSV outputs appear in outputs/ and excel/
```

---

## 📈 Key Results

### Team Performance

| Agent | Leads Assigned | Leads Contacted | Contact Rate | Conversions |
|---|---|---|---|---|
| ⭐ Poonam Sonawale | 250 | 69 | **27.6%** | 1 |
| 📈 Preeti Mahalinge | 1,109 | 272 | **24.5%** | 1 |
| ⚠️ Riya Singh | 3,141 | 162 | **5.1%** | 0 |

### Follow-up Health

| Status | Count | Share |
|---|---|---|
| 🔴 No Follow-up | 4,170 | 92.7% |
| 🟡 Overdue | 324 | 7.2% |
| 🟢 Upcoming | 6 | 0.1% |

---

## 🎯 Key Insights

**The #1 finding:** 92.7% of leads have no follow-up scheduled. This is the single biggest process gap — and it is entirely fixable.

- **3,997 leads have never been called.** That is 88.8% of the pipeline sitting untouched.
- **Riya Singh** holds 70% of all leads but has a 5.1% contact rate. Addressing this one issue would add hundreds of new conversations.
- **Poonam Sonawale** is the benchmark — at 27.6%, she proves the leads are reachable. The gap is effort and process, not lead quality.
- **2 conversions from 503 contacts** = 0.4% conversion rate. Industry standard is 2–5%. Improving follow-up discipline alone should move this significantly.

---

## 🛣️ Roadmap

- [ ] Auto-schedule the notebook to run daily at 8 AM via GitHub Actions
- [ ] Add email delivery of agent call sheets every morning
- [ ] Build a Streamlit web dashboard for real-time pipeline view
- [ ] Add call attempt retry logic (auto-retire leads after 6 failed attempts)
- [ ] Integrate directly with CRM API (LeadSquared / Zoho / Salesforce)
- [ ] Add lead scoring model using ML (time-since-creation + attempt count)

---

## 🧰 Tech Stack

| Tool | Purpose |
|---|---|
| Python 3.10 | Core language |
| pandas | Data cleaning, transformation, grouping |
| numpy | Numerical operations |
| matplotlib | Charts and visualisations in the notebook |
| openpyxl | Writing Excel files with formatting |
| Jupyter Notebook | Interactive analysis environment |
| Microsoft Excel | End-user delivery format |

---

## 📄 License

MIT License — free to use, modify, and distribute with attribution.

---

## 🤝 Contributing

Pull requests are welcome. For major changes, please open an issue first to discuss what you'd like to change. If you're adapting this for a different CRM or campaign, check the column names in `data/Leads_Helpage_Mumbai.csv` to align your schema.

---

*Built with Python · pandas · Jupyter · Excel · HelpAge Mumbai Campaign · May 2026*

