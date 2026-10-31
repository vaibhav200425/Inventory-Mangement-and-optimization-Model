# Supply Chain Inventory Optimization Model
*Demand Forecasting · Economic Order Quantity (EOQ) · Safety Stock · Dynamic Reorder Points*

[![Python](https://img.shields.io/badge/Python-3.9%2B-blue.svg)](https://www.python.org/)
[![Jupyter](https://img.shields.io/badge/Notebook-Jupyter-orange.svg)](https://jupyter.org/)
[![License](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)

**Author:** Vaibhav Gidde  
**Institution:** Visvesvaraya National Institute of Technology (VNIT), Nagpur  
**Discipline:** Mechanical Engineering  

---

## 1. Overview & Business Context

In modern inventory management, supply chain planners face a fundamental trade-off: minimizing working capital locked up in holding costs while preventing stockouts to maintain high customer service levels. 

This repository delivers an end-to-end, multi-SKU inventory optimization model. Operating on daily transaction-level demand, the framework produces an automated inventory replenishment policy featuring:
* **Per-SKU Demand Forecasting** via Simple Exponential Smoothing (SES) with train/test parameter optimization.
* **Cost-Optimal Replenishment Batches** calculated via the Economic Order Quantity (EOQ) formulation.
* **Stochastic Buffer Sizing** calibrated to a 95% service level ($z = 1.645$).
* **Automated Action Alerts** that dynamically evaluate on-hand stock against Reorder Points (ROP) to highlight stockout exposure.

---

## 2. Architecture & Pipeline Workflow

The workflow executes through five modular analytical steps:

```
+---------------------+
| 1. DATA GENERATION  |   2 years of daily demand across 10 SKUs (seasonality, noise, spikes)
+----------+----------+
           |
           v
+---------------------+
| 2. DEMAND FORECAST  |   Simple Exponential Smoothing (alpha tuned via MAE on 80/20 split)
+----------+----------+
           |
           v
+---------------------+
| 3. ORDER SIZING     |   Economic Order Quantity (EOQ) balancing holding vs. ordering costs
+----------+----------+
           |
           v
+---------------------+
| 4. BUFFER SIZING    |   Safety Stock (SS) and Reorder Point (ROP) for a 95% service level
+----------+----------+
           |
           v
+---------------------+
| 5. RISK ALERTS      |   Evaluate simulated on-hand inventory against ROP for action flags
+---------------------+
```

---

## 3. Mathematical Methodology

### 3.1 Demand Forecasting: Simple Exponential Smoothing (SES)

To capture baseline demand while giving higher weight to recent observations:

$$\hat{D}_{t} = \alpha D_{t-1} + (1 - \alpha)\hat{D}_{t-1}$$

Where:
* $\hat{D}_{t}$ = Forecasted demand for period $t$
* $D_{t-1}$ = Actual demand in period $t-1$
* $\alpha \in [0.1, 0.5]$ = Smoothing factor, optimized per SKU by minimizing Mean Absolute Error (MAE) on a 20% holdout test set:

$$\text{MAE} = \frac{1}{n} \sum_{t=1}^{n} |D_t - \hat{D}_t|$$

### 3.2 Order Batch Sizing: Economic Order Quantity (EOQ)

Balancing fixed replenishment setup costs against per-unit inventory carrying charges:

$$\text{EOQ} = \sqrt{\frac{2 \cdot D \cdot S}{H}}$$

Where:
* $D$ = Annualized forecast demand ($\bar{d}_{\text{daily}} \times 365$)
* $S$ = Procurement setup cost per purchase order ($S = 50$)
* $H$ = Annual inventory holding cost per unit ($H = 0.20 \times \text{Unit Cost}$)

### 3.3 Buffer Sizing: Safety Stock (SS)

Assuming lead-time demand is independently and normally distributed:

$$\text{SS} = z \cdot \sigma_d \cdot \sqrt{L}$$

Where:
* $z$ = Inverse CDF of the standard normal distribution ($z = 1.645$ for a 95% cycle service level)
* $\sigma_d$ = Standard deviation of daily forecast errors / daily demand
* $L$ = Supplier lead time in days ($\sqrt{L}$ accounts for variance linear accumulation over time)

### 3.4 Reorder Point (ROP) & Stockout Risk

The inventory threshold that triggers a replenishment requisition:

$$\text{ROP} = (\bar{d} \cdot L) + \text{SS}$$

The lead-time stockout probability is evaluated via the standard normal cumulative distribution function $\Phi(\cdot)$:

$$P(\text{Stockout}) = 1 - \Phi\left( \frac{\text{ROP} - \bar{d} \cdot L}{\sigma_d \sqrt{L}} \right)$$

---

## 4. SKU Classification & Baseline Parameters

The dataset models 10 distinct SKUs categorized into velocity tiers to reflect industrial inventory profiles:

| Segment | SKUs | Lead Time ($L$) | Cost & Volume Characteristics | Demand Volatility |
| :--- | :---: | :---: | :--- | :--- |
| **Fast-Movers** | 4 | 4 – 7 days | Low unit cost, high daily volume | Low coefficient of variation ($\text{CV}$) |
| **Medium-Movers** | 4 | 10 – 15 days | Moderate cost, steady movement | Moderate demand variability |
| **Slow-Movers** | 2 | 21 – 25 days | High unit cost, low turnover | High variability, intermittent spikes |

### Key Model Assumptions
* **Target Service Level:** 95% ($z = 1.645$)
* **Fixed Order Cost ($S$):** 50 currency units per order
* **Carrying Cost Rate ($i$):** 20% of unit cost per annum
* **Train/Test Split:** Chronological 80% train / 20% evaluation
* **Lead Time:** Deterministic per SKU

---

## 5. Output Policy Schema (`inventory_policy.csv`)

The model produces a production-ready policy table:

| Column Name | Data Type | Description |
| :--- | :--- | :--- |
| `sku_id` | `str` | Unique item identifier (e.g., `SKU_01`) |
| `category` | `str` | Velocity classification (`Fast-Mover`, `Medium-Mover`, `Slow-Mover`) |
| `unit_cost` | `float` | Item unit acquisition cost |
| `lead_time_days` | `int` | Supplier replenishment lead time ($L$) |
| `avg_daily_demand_forecast` | `float` | Model-derived baseline daily demand ($\bar{d}$) |
| `demand_std_dev` | `float` | Daily demand standard deviation ($\sigma_d$) |
| `eoq` | `int` | Recommended order quantity |
| `safety_stock` | `int` | Buffer stock units required for 95% service level |
| `reorder_point` | `int` | Replenishment trigger threshold |
| `simulated_current_stock` | `int` | Simulated on-hand stock for evaluation |
| `pct_vs_rop` | `float` | Current stock relative to ROP ($\pm\%$) |
| `stockout_risk` | `str` | Action alert: `AT RISK - Reorder Now` (if stock $\le$ ROP) or `OK` |

---

## 6. Repository Layout

```text
supply-chain-inventory-optimization/
├── Supply_Chain_Inventory_Optimization.ipynb   # Main end-to-end analytical notebook
├── README.md                                   # Project documentation and methodology
├── inventory_dashboard.png                     # 4-panel visual dashboard output
└── data/
    ├── sku_master.csv                          # SKU master parameters (cost, category, lead time)
    ├── demand_history.csv                      # Simulated 2-year daily demand transactions
    ├── demand_forecast.csv                     # Model forecast parameters (optimal alpha, MAE)
    └── inventory_policy.csv                    # Final operational inventory policy table
```

---

## 7. Setup & Execution

### Option A: Google Colab (Recommended)
1. Go to [Google Colab](https://colab.research.google.com).
2. Select **File > Upload notebook** and upload `Supply_Chain_Inventory_Optimization.ipynb`.
3. Ensure data output directories are set to relative paths (`./data/`).
4. Select **Runtime > Run all**.

### Option B: Local Environment
Ensure Python 3.9 or higher is installed:

```bash
# 1. Clone repository
git clone https://github.com/your-username/supply-chain-inventory-optimization.git
cd supply-chain-inventory-optimization

# 2. Install required packages
pip install pandas numpy scipy matplotlib jupyter

# 3. Launch notebook environment
jupyter notebook Supply_Chain_Inventory_Optimization.ipynb
```

---

## 8. Limitations & Roadmap

* **Lead-Time Stochasticity:** The current implementation assumes deterministic supplier lead times. Next iteration will incorporate lead-time variance:
  $$\text{SS} = z \sqrt{L \sigma_d^2 + \bar{d}^2 \sigma_L^2}$$
* **Intermittent Demand Forecasting:** For slow movers exhibiting zero-demand intervals, SES will be enhanced with Croston's Method or the Syntetos-Boylan Approximation (SBA).
* **Multi-Echelon Sizing:** Expand single-location safety stock buffers to multi-echelon networks (Central Warehouse to Regional Fulfillment Centers).
* **Supplier Quantity Constraints:** Integrate minimum order quantities (MOQs) and freight/container-load constraints into the EOQ engine.