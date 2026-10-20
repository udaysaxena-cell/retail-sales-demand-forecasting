# Retail Sales Analytics & Demand Forecasting Project

This project performs an end-to-end retail analytics and supply-chain analysis using historical sales data. It answers four key business questions:

1. Which factors influence product sales?
2. What will future weekly sales look like?
3. Which products are fast movers or slow movers?
4. How much safety stock and when should each product be reordered?

The project uses regression, time-series forecasting, clustering, classification, and inventory-planning techniques.

---

## 📂 Dataset

The dataset used is `sales_data.csv`.

It contains daily retail sales records with the following main columns:

| Column | Description |
|---|---|
| `Date` | Date of the sales record |
| `Store ID` | Unique store identifier |
| `Product ID` | Unique product identifier |
| `Category` | Product category |
| `Region` | Sales region |
| `Inventory Level` | Available inventory |
| `Units Sold` | Number of units sold |
| `Units Ordered` | Number of units ordered |
| `Price` | Product selling price |
| `Discount` | Discount percentage |
| `Weather Condition` | Weather on the sales date |
| `Promotion` | Whether a promotion was active |
| `Competitor Pricing` | Competitor product price |
| `Seasonality` | Seasonal label |
| `Epidemic` | Epidemic indicator |
| `Demand` | Demand value |

The dataset contains approximately **76,000 rows** and covers sales from **January 2022 to January 2024**.

---

## 🛠️ Technologies Used

- Python
- Pandas
- NumPy
- Matplotlib
- Scikit-learn
- Statsmodels
- Prophet
- KNeed

---

## 📊 Project Workflow

### 1. Data Loading and Cleaning

- Loaded the dataset using Pandas.
- Checked for missing values; no null values were found.
- Converted the `Date` column to datetime format.
- Converted categorical columns such as `Category`, `Region`, `Weather Condition`, and `Seasonality` to category data types.
- Performed initial exploration using `head()`, `tail()`, `dtypes`, and null-value checks.

---

## 📈 Model 1: Sales Regression

The first model predicts **Units Sold** using pricing and competitor-related variables.

### Initial Features

- Price
- Discount
- Competitor Pricing

### Result

- R² Score: **0.031**
- This means these features alone explained only about **3.1%** of the variation in Units Sold.

### Improved Features

- Price
- Demand
- Inventory Level

### Result

- R² Score: **0.714**
- This means the improved model explained approximately **71.4%** of the variation in Units Sold.

### Insight

Price, demand, and inventory level are much stronger predictors of sales than discount and competitor pricing alone.

---

## 📅 Model 2: Time-Series Forecasting

The forecasting section predicts future weekly sales for **Clothing products in Store S002**.

### Steps Followed

1. Filtered the dataset for the selected store and category.
2. Aggregated daily sales into daily total Units Sold.
3. Resampled the data into weekly sales.
4. Split the data into training and testing sets.
5. Checked stationarity using:
   - ADF Test
   - KPSS Test
6. Used ACF and PACF plots to select ARIMA parameters.
7. Built and compared multiple forecasting models.
8. Evaluated models using RMSE, MAE, and WMAPE.
9. Refit the best model on the full dataset.
10. Generated a 12-week future sales forecast.

### Stationarity Results

| Test | Result | Conclusion |
|---|---:|---|
| ADF Test | p-value ≈ 0.000 | Data is stationary |
| KPSS Test | p-value = 0.10 | Data is stationary |

Since both tests indicate stationarity, the data did not require differencing.

---

## 🤖 Forecasting Models Compared

| Model | RMSE | MAE | WMAPE |
|---|---:|---:|---:|
| ARIMA(1,0,1) | 1010.02 | 758.40 | 24.56% |
| SARIMAX(1,0,1)(1,0,1,52) | 970.71 | 702.05 | 22.74% |
| Holt-Winters | 1426.61 | 990.93 | 32.09% |
| Prophet | 729.30 | 405.76 | **13.02%** |

### Best Model: Prophet

Prophet performed the best on the test set, achieving the lowest RMSE, MAE, and WMAPE.

Therefore, Prophet was selected as the final forecasting model and refitted on the complete historical weekly dataset to forecast the next 12 weeks of sales.

---

## 🔮 Final 12-Week Forecast

The final Prophet model forecasts weekly Clothing sales for Store S002 for the next 12 weeks.

The forecast includes:

- Predicted sales (`yhat`)
- Lower confidence bound (`yhat_lower`)
- Upper confidence bound (`yhat_upper`)

This helps the business plan inventory, staffing, promotions, and procurement in advance.

---

## 🧩 Model 3: Product Clustering

Product clustering was performed to identify fast-moving and slow-moving products.

### Features Used

- Average Units Sold
- Average Price

### Method

- Standardized the features using `StandardScaler`.
- Used the Elbow Method with `KneeLocator` to find the optimal number of clusters.
- Applied K-Means clustering.

### Cluster Interpretation

| Cluster | Interpretation |
|---|---|
| Cluster 0 | Higher-priced products with relatively lower sales volume |
| Cluster 1 | Lower-priced, high-volume products that sell quickly |

### Business Use

- Cluster 1 products require frequent restocking.
- Cluster 0 products need careful pricing, promotion, and inventory monitoring.

---

## ⚠️ Model 4: Stock-Out Classification

A classification model was attempted to predict whether a product-day would face a stock-out.

### Target Variable

A product-day was labeled as a stock-out if:

```python
Units Sold > Inventory Level
```

### Result

The Random Forest model achieved **100% accuracy**.

However, this happened because the dataset contains **no actual stock-out events**. Every record has `Units Sold <= Inventory Level`, so the model simply learned to predict “no stock-out” every time.

### Limitation

This model is not useful for real-world prediction because the target variable has only one class.

### Possible Improvement

The stock-out target could be redesigned using a risk-based definition, such as:

- Low inventory relative to average demand
- Inventory below a defined safety-stock threshold
- High demand variability combined with low stock

---

## 📦 Model 5: Inventory and Stock Planning

The final section calculates product-level inventory requirements.

### Assumptions

- Lead time: **5 days**
- Desired service level: **95%**
- Z-score: **1.65**

### Calculations

Average daily demand:

\[
\text{Average Daily Demand} = \frac{\text{Total Units Sold}}{\text{Number of Days}}
\]

Safety Stock:

\[
\text{Safety Stock} = Z \times \sigma_d \times \sqrt{L}
\]

Reorder Point:

\[
\text{Reorder Point} = (\text{Average Daily Demand} \times L) + \text{Safety Stock}
\]

Where:

- \( \sigma_d \) = standard deviation of daily demand
- \( L \) = lead time in days

### Output

For each product, the project calculates:

- Average daily demand
- Demand standard deviation
- Safety stock
- Reorder point

This helps prevent both:

- Stock-outs
- Excess inventory holding

---

## 🚀 Key Business Insights

- Prophet is the most accurate forecasting model for this dataset.
- Demand and inventory level strongly influence Units Sold.
- Products can be segmented into fast movers and slow movers using sales volume and price.
- Safety stock and reorder points can be calculated for every product to support better inventory planning.
- The dataset does not contain real stock-out events, so stock-out classification should be treated as a data limitation.

---

## 📌 Future Improvements

- Add more historical data to improve SARIMAX yearly seasonality.
- Use daily forecasting instead of weekly forecasting.
- Include promotion, weather, holiday, and competitor-price effects in Prophet.
- Perform hyperparameter tuning for ARIMA, SARIMAX, and Prophet.
- Use backtesting across multiple time windows instead of one train-test split.
- Redesign the stock-out classification target using inventory-risk thresholds.
- Deploy the final Prophet model using Streamlit, Flask, or FastAPI.

---

## ▶️ How to Run the Project

1. Clone or download the project folder.
2. Install the required libraries:

```bash
pip install pandas numpy matplotlib scikit-learn statsmodels prophet kneed
```

3. Place `sales_data.csv` in the same folder as the notebook.
4. Open and run:

```bash
retail.ipynb
```

---

## 🏁 Conclusion

This project demonstrates a complete retail analytics workflow, from exploratory data analysis to demand forecasting, product segmentation, and inventory planning.

The most important outcome is that the project does not stop at prediction — it converts model outputs into practical supply-chain decisions such as safety stock levels and reorder points.