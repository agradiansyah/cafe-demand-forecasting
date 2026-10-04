# Raw Material Demand Forecasting for a Café: Linear Regression vs Random Forest

Undergraduate thesis project (Information Systems, Universitas Komputer Indonesia). I compared Linear Regression and Random Forest for forecasting daily menu demand at a real café, converted the forecasts into raw material needs using Bill of Materials (BOM) data, and delivered the result as an interactive Streamlit dashboard for the business owner.

*Thesis title (ID): Perbandingan Algoritma Linear Regression dan Random Forest dalam Peramalan Kebutuhan Stok Bahan Baku Harian pada The Soko Coffee Tea Chocolate*

## Problem
The Soko Coffee Tea Chocolate (Bandung, Indonesia) planned ingredient purchases without a measured forecast, which risks overstock (waste) and understock (unmet orders). Sales are recorded per menu item, but stock is managed per raw material, so forecasts had to be connected to recipes.

## Data
- **POS sales data** (ESB system), Jan 2024 to mid-2026: 19,542 raw transaction rows, 14 columns.
- **Bill of Materials (BOM):** ingredient composition per menu item.
- **Waste, spoiled, lost (WSL) records:** used as a reference for actual stock conditions.

## Approach (CRISP-DM)
1. **Attribute selection:** kept 4 of 14 columns (date, menu name, quantity, item type). The financial columns correlated strongly with quantity (r ≈ 0.87) but are calculated from it, so I excluded them to avoid **target leakage**.
2. **Cleaning:** trimmed menu names, checked missing values and duplicates. IQR flagged 12.68% of rows as outliers, but I **kept them** because they are valid group orders and represent real demand spikes.
3. **Daily aggregation and date gap filling:** one row per menu per day, with zero-sales days filled as 0, giving 131,394 rows after feature engineering.
4. **Feature engineering (11 features):** calendar (day of week, month, weekend flag, week of month), lags (1, 3, 7, 14 days), and rolling mean/std (7 and 14 days), computed per menu to prevent leakage between menus.
5. **Modeling:** time-based 80:20 split (split date 11 Aug 2025) instead of a random split, to avoid using future data. Both models received identical data (controlled comparison). Linear Regression was tested with 5 vs 11 features, and Random Forest with 3 hyperparameter settings.
6. **Evaluation:** MAE, RMSE, and R².
7. **Deployment:** two-level BOM explosion (menu → intermediate item → raw material), conversion into purchasing units, and a Streamlit dashboard.

## Results

**Scenario 1: original dataset (to Aug 2025)**

| Model | MAE | RMSE | R² |
|---|---|---|---|
| Linear Regression (11 features) | 0.3842 | 0.9180 | 0.4321 |
| Random Forest (200 trees, depth 10) | **0.3772** | **0.9020** | **0.4516** |

**Scenario 2: retrained with new data (Jan to Jun 2026, 291,850 rows)**

| Model | MAE | RMSE | R² |
|---|---|---|---|
| **Linear Regression** | **0.47** | **1.81** | **0.7886** |
| Random Forest | 0.52 | 3.05 | 0.4019 |

**What I found**
- Random Forest was slightly better on the original data, but **Linear Regression clearly won after retraining**, so I deployed Linear Regression.
- A likely reason is that Random Forest averages target values seen in training, so it cannot extrapolate beyond that range, while demand kept growing in 2026. Re-testing Random Forest with 100, 200, and 300 trees gave practically identical results, so the cause looks structural, not a tuning problem.
- R² of ~0.45 in scenario 1 is moderate, which is expected for item-level demand with many zero-sales days (intermittent demand).
- Sales pattern: weekdays average 61-70 portions per day, Friday 87, Saturday 162, and Sunday 130. Weekend sales are more than double weekday sales.
- Converting forecasts to purchasing units over a **7-day accumulation window** (instead of rounding up daily) avoids stock build-up.

## Dashboard (Streamlit)
**Page 1: Analytics and Procurement**
- Active model and last retraining time
- 7-day sales summary and next-week financial projection (revenue, ingredient budget, gross profit)
- Raw material requirement table (searchable, filter by Bar/Kitchen), converted to purchase packs with round-up
- Top 5 menu per division and average sales by day of week

**Page 2: Data Update and Auto Retraining**
- Upload a new POS export, preview it, and retrain with one click
- Both models are retrained and compared automatically, and the one with the lowest RMSE becomes the production model
- Built for a non-technical owner, with a note to compare estimates against physical stock before buying (human-in-the-loop)

Dashboard Screenshots:
<img width="1440" height="847" alt="Screenshot 2026-10-04 at 14 11 20" src="https://github.com/user-attachments/assets/a1357f67-33e1-4115-a2e2-7ae9430abc6c" />
<img width="1182" height="710" alt="Screenshot 2026-10-04 at 14 11 29" src="https://github.com/user-attachments/assets/f87bb950-39ad-4586-953c-6a5d036df104" />
<img width="1175" height="570" alt="Screenshot 2026-10-04 at 14 11 35" src="https://github.com/user-attachments/assets/5e4fc1cd-656f-4cec-a9c6-f072992b4b6c" />
<img width="1181" height="649" alt="Screenshot 2026-10-04 at 14 11 42" src="https://github.com/user-attachments/assets/b8b3f31f-3c7a-4d98-aee5-95a80ba9d7bb" />
<img width="1440" height="843" alt="Screenshot 2026-10-04 at 14 11 51" src="https://github.com/user-attachments/assets/64db7a36-6c91-423f-94fb-e2fb7cc316c6" />

## Tech Stack
Python, Pandas, NumPy, Scikit-learn, Streamlit, Jupyter Notebook

## How to Run
```bash
git clone https://github.com/agradiansyah/demand_forecasting_skripsi_10522138.git
cd demand_forecasting_skripsi_10522138
pip install -r requirements.txt
streamlit run <your_app_file>.py
```

## Limitations
- Based on one café and about 2.5 years of data.
- Only historical sales and BOM data are used. No external factors such as weather, promotions, or holidays.
- Forecasts are produced recursively, so accuracy drops for longer horizons. The dashboard is intended for daily and weekly planning.

## Future Work
Retrain weekly as new data arrives, test hybrid approaches that combine Linear Regression's trend extrapolation with Random Forest's non-linear patterns, and add external variables (weather, promotions, national holiday calendar).

## What I Learned
The "best" algorithm depended on the data: Random Forest won first, then lost after retraining. That is why I built a retrain-and-compare workflow instead of fixing one model. I also learned that much of the work sits around the model: leakage checks, time-aware validation, and turning predictions into something a business owner can act on.

---
Aqbil Gradiansyah · Universitas Komputer Indonesia (UNIKOM)
