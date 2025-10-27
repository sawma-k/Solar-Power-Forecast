# End-to-End Solar Power Generation Forecast ☀️

This project is a complete data science pipeline that predicts the hourly solar power generation (MWh) for the next 24 hours for the Vankal Solar Park in Mizoram.

The pipeline handles everything from ingesting and cleaning messy, unstructured 15-minute Excel data to training multiple ML models and, finally, deploying a prediction script that pulls live 24-hour forecast data from the Solcast weather API.

The champion model is a **`GridSearchCV`-tuned Random Forest Regressor** that achieves an **R² Score of 0.91** on the full 9-month dataset.

---

## 🛠️ Tech Stack

* **Data Manipulation:** Python, Pandas, NumPy
* **Data Visualization:** Matplotlib, Seaborn
* **Machine Learning:** Scikit-learn (StandardScaler, RandomForestRegressor, XGBoost, GridSearchCV)
* **API & Deployment:** Requests (for API calls), Joblib (for model saving)

---

## 📈 Project Pipeline

This project follows a 4-step data science workflow.

### 1. Data Cleaning and Preprocessing

* **Parsed Raw Energy Data:** Wrote a Python script (`01_data_cleaning.py`) to automatically process 9 months (Apr-Dec 2023) of raw, unstructured 15-minute Excel logs.
* **Resampled Data:** Aggregated the 15-minute power (MW) data into hourly energy (MWh) blocks using pandas `resample()`.
* **Merged Datasets:** Combined the clean hourly energy data with historical weather data (GHI, DNI, Air Temp, etc.) obtained from the Solcast API.
* **Handled Timezones:** Converted all UTC timestamps from the weather API to local time (`Asia/Kolkata`) to align with the energy data.

### 2. Exploratory Data Analysis (EDA) & Feature Selection

* **Performed Feature Selection:** Analyzed correlations and dropped:
    * **Target Leakage:** `Avg_MW`, `Max_MW` (which are just other measurements of the target).
    * **Redundant Features:** `clearsky_ghi` (less accurate than actual `ghi`).
    * **Weak Features:** `albedo` (no correlation).
* **Final Features:** The model was trained on `ghi`, `dhi`, `dni`, `air_temp`, `relative_humidity`, and `cloud_opacity`.
![Features correlation](https://github.com/sawma-k/Solar-Power-Forecast/blob/41f6d4eba948e4d7a83c085935a4da5da7538269/images/Correlation_MWh.png)
### 3. Model Training and Comparison

* **Scaled Features:** Used `StandardScaler` on all features (`X`) to normalize their scales.
* **Experimented with Target Transformation:** Tested `QuantileTransformer` on the skewed target variable (`y`) but found it *hurt* the Linear Regression model's performance (R² dropped from 0.88 to 0.56), proving the original linear relationship was already strong.
* **Compared 6 Models:** Built, trained, and evaluated six different regression models to find the best performer.
* **Selected Champion Model:** The **Tuned Random Forest** was the clear winner.



## 📊 Model Performance

The Tuned Random Forest was the best-performing model, demonstrating high accuracy and the lowest error.

| Model | R² Score (Higher is better) | MAE (Lower is better) | RMSE (Lower is better) |
| :--- | :--- | :--- | :--- |
| Linear Regression | 0.8862 | 1.16 MWh | 1.98 MWh |
| Decision Tree | 0.8523 | 0.99 MWh | 2.00 MWh |
| Random Forest (Default) | 0.9081 | 0.85 MWh | 1.77 MWh |
| **Tuned Random Forest** | **0.9103** | **0.83 MWh** | **1.75 MWh** |
| XGBoost (Default) | 0.8944 | 0.92 MWh | 1.90 MWh |
| Tuned XGBoost | 0.9086 | 0.87 MWh | 1.77 MWh |

![Model Comparison Chart](https://github.com/sawma-k/Solar-Power-Forecast/blob/41f6d4eba948e4d7a83c085935a4da5da7538269/images/model_performance_comparison.png)

---

### 4. Productionalized Prediction Pipeline

* **Saved Tools:** The champion model (`.pkl`), the `StandardScaler` (`.pkl`), and the feature order (`.json`) were all saved.
* **Built Prediction Script:** The final script (`03_prediction.py`) automatically:
    1.  Loads the saved model, scaler, and column list.
    2.  Calls the Solcast API for a live 24-hour weather forecast.
    3.  Performs all necessary data and timezone transformations.
    4.  Scales the forecast data and generates the hourly MWh predictions.
    5.  Saves the final forecast to `solar_forecast_next_24_hours.csv`.
![Model Prediction](https://github.com/sawma-k/Solar-Power-Forecast/blob/41f6d4eba948e4d7a83c085935a4da5da7538269/images/prediction.png)
---

