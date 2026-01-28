# FinCARE: Financial Causal Analysis with Reasoning and Evidence

## Dataset and Ground Truth for Reproducibility

This repository contains the synthetic financial dataset and ground truth causal model used in the paper:

> **FinCARE: Financial Causal Analysis with Reasoning and Evidence**  
> Anonymous Authors  
> Under Review, 2025

---

## Contents

```
├── enhanced_synthetic_financial_data.csv   # Synthetic financial dataset
├── ground_truth_causal_model.json          # Ground truth DAG and counterfactuals
└── README.md
```

---

## Dataset Description

### `enhanced_synthetic_financial_data.csv`

A synthetic financial dataset with embedded causal relationships for benchmarking causal discovery algorithms.

| Property | Value |
|----------|-------|
| Rows | 1,800 |
| Companies | 50 (S&P 100 subset) |
| Time Period | 36 months (2022-01-01 to 2024-12-01) |
| Sectors | Technology, Financials, Healthcare, Energy, Consumer |

### Variables (18 total)

| Variable | Type | Description |
|----------|------|-------------|
| `China_Revenue_Percent` | Exogenous | Revenue exposure to China (0-1) |
| `US_Revenue_Percent` | Exogenous | Revenue exposure to US (0-1) |
| `Europe_Revenue_Percent` | Exogenous | Revenue exposure to Europe (0-1) |
| `Governance_Score` | Exogenous | ESG governance score (0-100) |
| `M_and_A_Event` | Exogenous | Binary M&A event indicator |
| `Major_Product_Launch` | Exogenous | Binary product launch indicator |
| `Regulatory_Change_Event` | Exogenous | Binary regulatory event indicator |
| `Supplier_Concentration` | Exogenous | Supplier concentration ratio (0-1) |
| `Customer_Concentration` | Exogenous | Customer concentration ratio (0-1) |
| `Supply_Chain_Risk_Score` | Endogenous | Supply chain risk (0-1) |
| `Regulatory_Risk_Score` | Endogenous | Regulatory risk (0-1) |
| `Cyber_Risk_Score` | Endogenous | Cybersecurity risk (0-1) |
| `Carbon_Emissions_Score` | Endogenous | Carbon emissions metric (0-100) |
| `Market_Risk_Score` | Endogenous | Overall market risk (0-1) |
| `EBITDA_Margin` | Endogenous | Profitability margin (0-1) |
| `Revenue_Growth_YoY` | Endogenous | Year-over-year revenue growth |
| `Debt_to_Equity` | Endogenous | Leverage ratio |
| `Monthly_Return` | Outcome | Monthly stock return |

### Metadata Columns

| Column | Description |
|--------|-------------|
| `ticker` | Company ticker symbol |
| `sector` | Industry sector |
| `date` | Observation date |
| `market_regime` | Market condition (bull/bear/normal/crisis) |

---

## Ground Truth

### `ground_truth_causal_model.json`

Contains the true causal structure embedded in the synthetic data.

#### Structure

```json
{
  "true_dag": {
    "nodes": ["list of 18 variable names"],
    "edges": ["list of 26 directed edges as [source, target] pairs"]
  },
  "true_counterfactuals": {
    "scenario_name": {
      "original_mean": float,
      "counterfactual_mean": float,
      "true_effect": float,
      "sample_size": int,
      "description": string
    }
  },
  "test_interventions": {
    "scenario_name": {
      "description": string,
      "filter": {"ticker" or "sector": value},
      "intervention": {"variable": value}
    }
  }
}
```

#### Ground Truth DAG

The true causal DAG contains **18 nodes** and **26 directed edges**, organized into:

- **Geographic → Risk pathways** (e.g., China exposure → Supply chain risk)
- **ESG relationships** (e.g., Governance → Carbon emissions)
- **Financial performance pathways** (e.g., Risk → EBITDA margin)
- **Return drivers** (6 direct causes of Monthly_Return)

#### Counterfactual Scenarios

| Scenario | Description |
|----------|-------------|
| `apple_china_reduction` | Apple reduces China exposure from 35% to 10% |
| `tech_cyber_increase` | Technology sector cyber risk increases by 50% |
| `financial_governance_improvement` | Financial sector governance improves to 85 |
| `supply_chain_shock` | Global supply chain risk increases to 0.8 |

---

## Usage

### Loading the Data

```python
import pandas as pd
import json

# Load dataset
df = pd.read_csv('enhanced_synthetic_financial_data.csv')

# Load ground truth
with open('ground_truth_causal_model.json', 'r') as f:
    ground_truth = json.load(f)

# Extract true edges
true_edges = [tuple(e) for e in ground_truth['true_dag']['edges']]
print(f"Ground truth: {len(true_edges)} edges")
```

### Preparing Data for Causal Discovery

```python
# Define causal variables (exclude metadata)
causal_variables = [
    'China_Revenue_Percent', 'US_Revenue_Percent', 'Europe_Revenue_Percent',
    'Supply_Chain_Risk_Score', 'Regulatory_Risk_Score', 'Market_Risk_Score',
    'Cyber_Risk_Score', 'Governance_Score', 'Carbon_Emissions_Score',
    'EBITDA_Margin', 'Revenue_Growth_YoY', 'Debt_to_Equity',
    'M_and_A_Event', 'Major_Product_Launch', 'Regulatory_Change_Event',
    'Supplier_Concentration', 'Customer_Concentration', 'Monthly_Return'
]

# Extract data matrix
X = df[causal_variables].values
```

### Running Baseline Algorithms

```python
from causallearn.search.ConstraintBased.PC import pc
from causallearn.search.ScoreBased.GES import ges

# PC Algorithm
cg = pc(X, alpha=0.01, indep_test='fisherz')

# GES Algorithm  
result = ges(X, score_func='local_score_BIC')
```

### Evaluating Graph Recovery

```python
def evaluate(discovered_edges, true_edges):
    true_set = set(true_edges)
    disc_set = set(discovered_edges)
    
    tp = len(true_set & disc_set)
    fp = len(disc_set - true_set)
    fn = len(true_set - disc_set)
    
    precision = tp / (tp + fp) if (tp + fp) > 0 else 0
    recall = tp / (tp + fn) if (tp + fn) > 0 else 0
    f1 = 2 * precision * recall / (precision + recall) if (precision + recall) > 0 else 0
    shd = fp + fn
    
    return {'precision': precision, 'recall': recall, 'f1': f1, 'shd': shd}
```

---

## Baseline Results

See paper for full methodology and results.

---

## Requirements

```
pandas>=1.3.0
numpy>=1.20.0
causal-learn>=0.1.3
```

Install with:
```bash
pip install pandas numpy causal-learn
```

---

## Citation

If you use this dataset in your research, please cite:

```bibtex
@inproceedings{anonymous2025fincare,
  title={FinCARE: Financial Causal Analysis with Reasoning and Evidence},
  author={Anonymous},
  booktitle={Under Review},
  year={2025}
}
```

---

## License

This dataset is released for academic research purposes.

---

## Contact

For questions about the dataset or methodology, please use the OpenReview discussion forum.
