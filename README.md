# FinShield_AI_Audit_System
### AI-Powered Audit & Risk Intelligence System

FinShield is a modular, multi-agent AI system that automates financial transaction auditing.
It scans raw transaction data, detects anomalies, scores risk, and generates explainable recommended actions — without manual intervention.

Built for ET Gen AI Hackathon 2026.

---

## 🚀 Live Results (200-transaction dataset)

| Metric | Count |
|---|---|
| Transactions Scanned | 200 |
| Anomalies Detected | 69 |
| Transactions Flagged | 56 |
| 🔴 HIGH RISK — Blocked | 6 |
| 🟡 MEDIUM RISK — Flagged for Review | 17 |
| 🟢 LOW RISK — Logged | 33 |

---

## 🧠 How It Works

FinShield runs a four-stage pipeline where each agent has one clearly defined job:

```
transactions.csv
       │
       ▼
  DataAgent         →  Cleans and validates raw CSV data
       │
       ▼
  AnomalyAgent      →  Detects 8 types of financial anomalies
       │
       ▼
  AuditTrailAgent   →  Aggregates findings and computes risk scores
       │
       ▼
  DecisionAgent     →  Generates recommended actions + explanations
       │
       ▼
  Final Output
```

---

## 🔍 Anomaly Detection Rules

| Rule                      | Severity   | Description|
|---------------------------|------------|-------------------------------------------------------|
| `DUPLICATE_TRANSACTION`   | 🔴 High   | Same invoice, vendor, and amount appear more than once |
| `AMOUNT_SPIKE`            | 🔴 High   | Transaction exceeds 3× the vendor's historical average |
| `HIGH_VALUE_TRANSACTION`  | 🟡 Medium | Amount exceeds the dynamic 90th-percentile threshold   |
| `RAPID_TRANSACTIONS`      | 🟡 Medium | 4+ transactions from the same vendor within 1 day      |
| `MISSING_INVOICE`         | 🔴 High   | Transaction processed without an invoice document      |
| `MISSING_GSTIN`           | 🟡 Medium | Vendor GSTIN is absent — compliance exposure           |
| `MISSING_PO`              | 🟡 Medium | No Purchase Order number linked to the transaction     |
| `APPROVAL_MISMATCH`       | 🔴 High   | High-value transaction approved at L1 (insufficient level) |

---

## 📊 Risk Scoring

AuditTrailAgent computes a weighted risk score per transaction by summing severity points across all detected issues:

| Severity | Points |
|---|---|
| High | 3 points |
| Medium | 2 points |
| Low | 1 point |

**Risk level thresholds:**

| Score | Risk Level | Action Taken |
|---|---|---|
| ≥ 5 | 🔴 HIGH RISK | Block transaction + initiate audit review |
| ≥ 3 | 🟡 MEDIUM RISK | Flag for manual review |
| < 3 | 🟢 LOW RISK | Log only, no immediate action |

---

## 📁 Project Structure

```
FinShield/
│
├── financial_audit_system.py        # All 4 agents — complete pipeline
├── FinShield_AI_Audit_System.ipynb  # Colab notebook
├── sample_transactions.csv          # Sample dataset (200 rows)
├── requirements.txt
├── .gitignore
└── README.md
```

---

## ⚙️ Setup & Usage

### 1. Clone the repo

```bash
git clone https://github.com/yourusername/FinShield.git
cd FinShield
```

### 2. Install dependencies

```bash
pip install -r requirements.txt
```

### 3. Run the pipeline

```python
from financial_audit_system import run_pipeline

decisions = run_pipeline("sample_transactions.csv")

for d in decisions:
    print(d["transaction_id"], d["risk_level"], d["recommended_action"])
```

### 4. Or run directly from terminal

```bash
python financial_audit_system.py sample_transactions.csv
```

### 5. Google Colab

Upload `FinShield_AI_Audit_System.ipynb` and `sample_transactions.csv` to Colab, then run all cells.

---

## 🧩 Agent API

Each agent follows the same interface — instantiate with input, call `.run()`:

```python
from financial_audit_system import DataAgent, AnomalyAgent, AuditTrailAgent, DecisionAgent

df        = DataAgent("sample_transactions.csv").run()
findings  = AnomalyAgent(df).run()
audit     = AuditTrailAgent(findings).run()
decisions = DecisionAgent(audit).run()
```

### Configurable Parameters

All detection thresholds are constructor arguments — no hardcoded values:

```python
AnomalyAgent(
    df,
    spike_multiplier=3.0,         # flag if amount > 3x vendor average
    high_value_percentile=0.90,   # flag top 10% by amount
    approval_threshold=100_000,   # L2 approval required above this (INR)
    rapid_tx_window_days=1,       # burst detection window in days
    rapid_tx_min_count=4,         # minimum transactions to trigger burst flag
)
```

---

## 📦 Requirements

```
pandas
```

All other components use the Python standard library only.

---

## 🗂️ Output Format

Each decision record produced by DecisionAgent contains:

```python
{
    "transaction_id":     "TXN2026012",
    "risk_level":         "HIGH RISK",
    "risk_score":         8,
    "recommended_action": "Block transaction and initiate audit review",
    "explanation":        "Transaction TXN2026012 triggered 3 issue(s)...",
    "top_issue":          "MISSING_INVOICE",
    "avg_confidence":     0.9
}
```

---

## ✨ Key Design Decisions

**Dynamic thresholds** — spike detection uses each vendor's own transaction history. High-value detection uses the dataset's live 90th percentile. No hardcoded INR cutoffs.

**Explainable outputs** — every finding includes the actual vendor name, amount, and ratio. Every decision includes a full narrative explanation. Nothing is a black box.

**Modular agents** — adding a new detection rule only touches AnomalyAgent. Everything downstream works without changes.

**Confidence scoring** — confidence scales with anomaly severity. A 4.5× spike scores higher confidence than a 3.1× spike.

---

## 📄 License

MIT License — free to use, modify, and distribute.

---

*FinShield — making financial audits faster, smarter, and fully explainable.*
