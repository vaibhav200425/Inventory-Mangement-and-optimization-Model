================================================================================

          SUPPLY CHAIN INVENTORY OPTIMIZATION MODEL
          Demand Forecasting  |  EOQ  |  Safety Stock  |  Reorder Points

================================================================================

  Author   : Vaibhav Gidde
  Language : Python 3
  Format   : Jupyter Notebook (.ipynb)
  Libraries: pandas, NumPy, SciPy, Matplotlib


--------------------------------------------------------------------------------
  1. OVERVIEW
--------------------------------------------------------------------------------

  This project builds an end-to-end inventory optimization model for a
  multi-SKU supply chain. It forecasts demand for each SKU, then uses that
  forecast to calculate how much to order, how much buffer stock to hold, and
  when to reorder, so that the risk of running out of stock is controlled.

  The final output is an actionable inventory policy: for every SKU, a
  recommended order quantity (EOQ), safety stock, reorder point, and a
  stockout-risk flag that tells a planner which items need attention now.

  NOTE: The dataset is synthetic. It is generated with realistic structure
  (weekly and annual seasonality, random noise, demand spikes, and varied SKU
  behaviour) because real company inventory data is confidential. The methods
  are the same ones used in real supply chain planning systems.


--------------------------------------------------------------------------------
  2. KEY FEATURES
--------------------------------------------------------------------------------

  [*] Synthetic multi-SKU demand data (10 SKUs, 2 years of daily history)
  [*] Per-SKU demand forecasting using Simple Exponential Smoothing
  [*] Smoothing constant (alpha) tuned using train/test validation
  [*] Economic Order Quantity (EOQ) calculation
  [*] Safety stock sized to a target service level using z-scores
  [*] Reorder point (ROP) calculation per SKU
  [*] Stockout-risk flagging by comparing current stock against ROP
  [*] Estimated stockout-risk reduction against a no-safety-stock baseline
  [*] Dashboard visualization of demand, stock status, and ROP composition


--------------------------------------------------------------------------------
  3. PROJECT WORKFLOW
--------------------------------------------------------------------------------

      +---------------------+
      |  1. DATA GENERATION |   Synthetic daily demand for 10 SKUs
      +----------+----------+
                 |
                 v
      +---------------------+
      |  2. DEMAND FORECAST |   Exponential smoothing + train/test validation
      +----------+----------+
                 |
                 v
      +---------------------+
      |  3. EOQ             |   Optimal order quantity per SKU
      +----------+----------+
                 |
                 v
      +---------------------+
      |  4. SAFETY STOCK    |   Buffer sized to target service level
      |     & REORDER POINT |
      +----------+----------+
                 |
                 v
      +---------------------+
      |  5. RISK FLAGGING   |   Current stock vs. reorder point
      |     & POLICY OUTPUT |
      +---------------------+


--------------------------------------------------------------------------------
  4. METHODOLOGY AND FORMULAS
--------------------------------------------------------------------------------

  4.1  Simple Exponential Smoothing (Demand Forecast)

         F(t) = alpha * A(t-1) + (1 - alpha) * F(t-1)

         F(t)    = forecast for period t
         A(t-1)  = actual demand in the previous period
         alpha   = smoothing constant (0 to 1); higher = more weight on
                   recent demand

       Why this method: it is fast, easy to explain, and scales across
       thousands of SKUs, which is why it is a standard industry baseline.
       Alpha is chosen per SKU by testing candidate values and picking the one
       with the lowest Mean Absolute Error on a held-out test set (last 20%).

  4.2  Economic Order Quantity (EOQ)

         EOQ = sqrt( (2 * D * S) / H )

         D = annual demand
         S = ordering cost per order
         H = annual holding cost per unit

       Balances the cost of ordering too often against the cost of holding
       too much stock.

  4.3  Safety Stock

         SS = z * sigma_d * sqrt(L)

         z        = z-score for the target service level (1.645 for 95%)
         sigma_d  = standard deviation of daily demand
         L        = supplier lead time in days

       The sqrt(L) term appears because variance accumulates linearly over
       time, so standard deviation grows with the square root of time.

  4.4  Reorder Point (ROP)

         ROP = (average daily demand * L) + SS

       Expected demand during the lead time, plus the safety buffer.

  4.5  Stockout Probability

         P(stockout) = 1 - Phi( (ROP - d_avg * L) / (sigma_d * sqrt(L)) )

         Phi = cumulative distribution function of the standard normal


--------------------------------------------------------------------------------
  5. MODEL ASSUMPTIONS
--------------------------------------------------------------------------------

  +---------------------------------+----------------------------------------+
  | Parameter                       | Value                                  |
  +---------------------------------+----------------------------------------+
  | Target service level            | 95%  (z = 1.645)                       |
  | Ordering cost per order         | 50 (currency units)                    |
  | Annual holding cost rate        | 20% of unit cost                       |
  | Train / test split              | 80% / 20%                              |
  | Candidate alpha values          | 0.1, 0.2, 0.3, 0.4, 0.5                |
  | Demand distribution             | Normal during lead time                |
  | Lead time                       | Fixed (deterministic) per SKU          |
  | Random seed                     | 42 (fully reproducible)                |
  +---------------------------------+----------------------------------------+


--------------------------------------------------------------------------------
  6. SKU PROFILE
--------------------------------------------------------------------------------

  +--------------+-------+---------------+---------------------------------+
  | Category     | SKUs  | Lead Time     | Demand Behaviour                |
  +--------------+-------+---------------+---------------------------------+
  | Fast-Mover   |   4   |  4 - 7 days   | High volume, low volatility     |
  | Medium-Mover |   4   | 10 - 15 days  | Moderate volume and volatility  |
  | Slow-Mover   |   2   | 21 - 25 days  | Low volume, high volatility,    |
  |              |       |               | expensive, long lead times      |
  +--------------+-------+---------------+---------------------------------+


--------------------------------------------------------------------------------
  7. REPOSITORY STRUCTURE
--------------------------------------------------------------------------------

  supply-chain-inventory-optimization/
  |
  |-- Supply_Chain_Inventory_Optimization.ipynb   Main notebook (full project)
  |-- README.txt                                  Project documentation
  |-- inventory_dashboard.png                     4-panel results dashboard
  |
  `-- data/
      |-- sku_master.csv           SKU attributes (category, cost, lead time)
      |-- demand_history.csv       Daily demand history for all SKUs
      |-- demand_forecast.csv      Forecast, std deviation, best alpha, MAE
      `-- inventory_policy.csv     Final policy: EOQ, SS, ROP, risk flag


--------------------------------------------------------------------------------
  8. OUTPUT: INVENTORY POLICY TABLE
--------------------------------------------------------------------------------

  inventory_policy.csv contains one row per SKU with these columns:

  +---------------------------+----------------------------------------------+
  | Column                    | Description                                  |
  +---------------------------+----------------------------------------------+
  | sku_id                    | Unique SKU identifier                        |
  | category                  | Fast / Medium / Slow mover                   |
  | unit_cost                 | Cost per unit                                |
  | lead_time_days            | Supplier lead time                           |
  | avg_daily_demand_forecast | Forecasted average daily demand              |
  | demand_std_dev            | Recent demand variability                    |
  | eoq                       | Recommended order quantity                   |
  | safety_stock              | Recommended buffer stock                     |
  | reorder_point             | Stock level that triggers a new order        |
  | simulated_current_stock   | Simulated on-hand stock (for demonstration)  |
  | pct_vs_rop                | Current stock above/below ROP, in percent    |
  | stockout_risk             | "AT RISK - Reorder Now" or "OK"              |
  +---------------------------+----------------------------------------------+


--------------------------------------------------------------------------------
  9. HOW TO RUN
--------------------------------------------------------------------------------

  OPTION A: Google Colab (recommended)

     1. Open  colab.research.google.com
     2. File -> Upload notebook -> select the .ipynb file
     3. Use Find and Replace to replace  /home/claude/  with nothing
        (file paths in the notebook must be relative in Colab)
     4. Runtime -> Run all

  OPTION B: Local machine

     1. Install Python 3.9 or higher
     2. Install dependencies:

            pip install pandas numpy scipy matplotlib jupyter

     3. Replace  /home/claude/  with nothing in the notebook code cells
     4. Launch Jupyter and run all cells:

            jupyter notebook Supply_Chain_Inventory_Optimization.ipynb

  The notebook generates its own data, so no external dataset is required.


--------------------------------------------------------------------------------
  10. LIMITATIONS
--------------------------------------------------------------------------------

  [!] Data is synthetic, not drawn from a real company.

  [!] The stockout-risk reduction is a policy-design result. It follows
      mathematically from choosing a 95% service level over a zero-buffer
      baseline. It is not an empirical backtest of real stockout incidents.

  [!] Demand is assumed to be normally distributed. Real demand, especially
      for slow-moving items, is often skewed or intermittent.

  [!] Lead time is treated as fixed. Supplier delays add extra variance that
      this model does not capture.

  [!] Ordering cost and holding cost rate are uniform assumptions. In
      practice these vary by SKU and supplier.

  [!] The forecast is univariate (history only). It does not use price,
      promotions, or other external drivers.

  [!] Current stock levels are simulated for demonstration purposes.


--------------------------------------------------------------------------------
  11. FUTURE IMPROVEMENTS
--------------------------------------------------------------------------------

  [+] Include lead-time variability in the safety stock formula:

          SS = z * sqrt( L * sigma_d^2 + d_avg^2 * sigma_L^2 )

  [+] Segment service levels by SKU criticality using ABC analysis
  [+] Use ML-based forecasting for high-value, high-volume SKUs
  [+] Model intermittent demand for slow movers (e.g. Croston's method)
  [+] Extend to multi-echelon inventory (warehouse, distribution center, store)
  [+] Validate the policy against real stockout data after deployment
  [+] Add external regressors (promotions, pricing, seasonality calendars)


--------------------------------------------------------------------------------
  12. CONCEPTS COVERED
--------------------------------------------------------------------------------

  Time-series forecasting  .  Exponential smoothing  .  Train/test validation
  Economic Order Quantity  .  Safety stock  .  Service level and z-scores
  Reorder point  .  Normal distribution  .  Stockout probability
  Inventory policy design  .  Data visualization with Matplotlib


--------------------------------------------------------------------------------
  AUTHOR
--------------------------------------------------------------------------------

  Vaibhav Gidde
  Mechanical Engineering, VNIT Nagpur

================================================================================
