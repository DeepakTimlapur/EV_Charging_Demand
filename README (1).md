# EV Charging Demand Prediction

## Project Overview

**EV Charging Demand Prediction** is a machine learning project for forecasting the **next hour's station-level electric vehicle (EV) charging demand**.

The project processes EV charging observations, performs exploratory data analysis, creates a complete station-by-hour time-series dataset, engineers historical lag and rolling features, prevents temporal data leakage, trains multiple regression models, evaluates them using a chronological train/validation/test strategy, and presents the results through an interactive Streamlit dashboard.

### Main Objective

> Predict the next hour's station-level charging demand using only historical information available before that future hour.

---

## Dataset

### Dataset Name

**Electric Vehicle Charging Demand Dataset**

### Dataset Source

[Kaggle – Electric Vehicle Charging Demand Dataset](https://www.kaggle.com/datasets/ziya07/electric-vehicle-charging-demand-dataset)

### Dataset Characteristics

| Property | Value |
|---|---:|
| Raw observations | 8,354 |
| Columns | 27 |
| Charging stations | 20 |
| Unique vehicles | 8,354 |
| Location types | Urban, Highway |
| Vehicle types | Car, Bus, Two-Wheeler |
| Missing values | 0 |
| Duplicate rows | 0 |
| Timestamp range | 2025-01-01 00:00:00 to 2025-03-29 00:15:00 |

### Important Dataset Fields

- `timestamp`
- `station_id`
- `location_type`
- `vehicle_id`
- `vehicle_type`
- `arrival_time`
- `charging_start_time`
- `charging_end_time`
- `waiting_time`
- `battery_capacity_kWh`
- `initial_soc`
- `final_soc`
- `energy_consumed_kWh`
- `charging_power_kW`
- `charging_duration`
- `queue_length`
- `station_load`
- `electricity_price`
- `renewable_energy_ratio`
- `traffic_density`
- `weather_condition`
- `day_of_week`
- `time_slot`
- `charging_demand`
- `assigned_charger_id`
- `charging_priority`
- `optimization_reward`

---

## Technologies Used

### Programming Language

- Python

### Data Analysis

- Pandas
- NumPy

### Machine Learning

- Scikit-learn
- Linear Regression
- Random Forest Regressor
- Gradient Boosting Regressor
- HistGradientBoostingRegressor

### Visualization

- Matplotlib
- Seaborn
- Plotly

### Dashboard

- Streamlit

### Model Persistence

- Joblib

### Development Tools

- Jupyter Notebook
- VS Code
- Git / GitHub

---

## Project Workflow

```text
Raw EV Charging Dataset
        |
        v
Data Loading & Validation
        |
        v
Data Preprocessing
        |
        v
Exploratory Data Analysis
        |
        v
Station-Level Hourly Aggregation
        |
        v
Complete Station × Hour Panel
        |
        v
Calendar + Lag + Rolling Features
        |
        v
Leakage Prevention
        |
        v
Chronological Train / Validation / Test Split
        |
        +----------------------+
        |                      |
        v                      v
Baseline Models        Machine Learning Models
        |                      |
        +----------+-----------+
                   |
                   v
          Validation Evaluation
                   |
                   v
             Model Selection
                   |
                   v
          Held-Out Test Evaluation
                   |
                   v
       Next-Hour Station Forecast
                   |
                   v
          Streamlit Dashboard
```

---

## Target Variable

The forecasting target is:

```text
next_hour_demand
```

For each station, the target is the charging demand in the following hour.

The target is generated using:

```python
station_hour.groupby("station_id")["charging_demand"].shift(-1)
```

---

## Feature Engineering

The model uses **16 predictive features**.

### Calendar Features

- `hour_of_day`
- `day_of_week`
- `day_of_month`
- `month`
- `is_weekend`

### Historical Demand Features

- `demand_lag_1h`
- `demand_lag_2h`
- `demand_lag_3h`
- `demand_lag_24h`

### Historical Station Features

- `station_load_lag_1h`
- `station_load_lag_24h`
- `queue_lag_1h`
- `queue_lag_24h`

### Historical Rolling Features

- `demand_rolling_mean_3h`
- `demand_rolling_mean_6h`
- `demand_rolling_mean_24h`

The rolling features are calculated after shifting the demand series by one hour so that current-hour demand is not included.

---

## Data Leakage Prevention

Because the project predicts future demand, contemporaneous variables that directly describe the current demand state are not used as direct predictive features.

The following variables are excluded from the final feature set:

```text
charging_demand
station_load
queue_length
energy_consumed_kWh
waiting_time
```

Historical lagged versions are allowed because they represent information available before the forecast period.

---

## Train / Validation / Test Split

A **chronological split** is used instead of random shuffling.

```text
Training       → 70%
Validation     → 15%
Testing        → 15%
```

The split is performed using global target hours so that all stations belonging to the same hour remain in the same dataset split.

This prevents future observations from being mixed into the training data.

---

## Machine Learning Models

The project evaluates the following approaches:

### Baseline Models

1. Previous Hour
2. Previous Day

### Regression Models

1. Linear Regression
2. Random Forest Regressor
3. Gradient Boosting Regressor
4. HistGradientBoostingRegressor

---

## Model Selection

Models are selected using **validation-set MAE**.

The selected model is:

```text
Gradient Boosting Regressor
```

This means Gradient Boosting achieved the lowest validation MAE among the evaluated machine-learning models.

The selection is specific to this dataset and evaluation period and is not a claim that the algorithm is universally optimal.

---

## Validation Results

| Model | MAE | RMSE | R² | MAPE (%) |
|---|---:|---:|---:|---:|
| Gradient Boosting | 16.0072 | 23.3062 | 0.0357 | 79.75 |
| Linear Regression | 16.0179 | 23.6682 | 0.0055 | 74.64 |
| HistGradientBoosting | 16.0273 | 23.3569 | 0.0315 | 79.74 |
| Previous Day | 16.7119 | 33.2488 | -0.9626 | 90.58 |
| Previous Hour | 16.7148 | 33.3274 | -0.9719 | 93.12 |
| Random Forest | 17.0865 | 24.0808 | -0.0295 | 76.69 |

---

## Final Test Results

The selected Gradient Boosting model was evaluated on the held-out test period.

| Metric | Result |
|---|---:|
| MAE | 16.065038 |
| RMSE | 23.460165 |
| R² | 0.036521 |
| MAPE | 80.046629% |

MAPE is treated as a supplementary metric because the target contains zero-demand station-hours. The reported MAPE excludes zero actual values.

The relatively modest test R² indicates that substantial variation in next-hour station-level demand remains unexplained by the available features.

---

## Streamlit Dashboard

The project includes an interactive Streamlit dashboard for exploring the dataset and forecasting workflow.

### Dashboard Sections

- Project Overview
- Data Overview
- Exploratory Data Analysis
- Feature Engineering
- Train / Validation / Test
- Model Training
- Model Performance
- Forecast
- Station Analysis
- Feature Importance
- Methodology

### Dataset Breakdown

The dashboard provides dedicated views for:

- Station-wise analysis
- Location-wise analysis
- Vehicle-type analysis

The dashboard also displays:

- Dataset statistics
- Average charging demand
- Model performance
- Station-level error analysis
- Feature importance
- Forecast results

---

## Project Structure

```text
EV_Charging_Streamlit/
│
├── app.py
├── requirements.txt
├── README.md
├── .env.example
│
├── .streamlit/
│   └── config.toml
│
├── data/
│   └── EV_Charging_Demand_Dataset_Full.csv
│
├── models/
│   ├── selected_model.joblib
│   ├── model_metadata.joblib
│   ├── station_id_model.joblib
│   └── station_id_feature_columns.joblib
│
├── notebooks/
│
├── outputs/
│   ├── figures/
│   ├── predictions/
│   └── reports/
│
└── src/
    ├── data_loader.py
    ├── data_preprocessing.py
    ├── feature_engineering.py
    ├── eda.py
    ├── train_test_split.py
    ├── model_features.py
    ├── models.py
    ├── models_station_v2.py
    ├── evaluation.py
    └── forecasting.py
```

---

## Installation

### 1. Clone or download the project

Open the project folder in VS Code or a terminal.

### 2. Create a virtual environment

Windows PowerShell:

```powershell
python -m venv .venv
```

### 3. Activate the virtual environment

```powershell
.venv\Scripts\activate
```

### 4. Install dependencies

```powershell
pip install -r requirements.txt
```

---

## Dataset Setup

Place the downloaded dataset at:

```text
data/EV_Charging_Demand_Dataset_Full.csv
```

The application and project scripts expect this dataset to be available locally.

---

## Run the Streamlit Dashboard

From the project root:

```powershell
streamlit run app.py
```

The terminal will display the local Streamlit URL. Open that URL in a web browser.

---

## Run the Machine Learning Pipeline

The project contains separate source modules for the major stages.

Examples:

```powershell
python src/data_loader.py
```

```powershell
python src/data_preprocessing.py
```

```powershell
python src/feature_engineering.py
```

```powershell
python src/train_test_split.py
```

```powershell
python src/models.py
```

```powershell
python src/evaluation.py
```

```powershell
python src/forecasting.py
```

---

## Jupyter Notebook Submission

The academic code submission is:

```text
Deepak_EV_Charging_Demand_Prediction.ipynb
```

The notebook contains the complete reproducible workflow:

1. Dataset loading
2. Data quality checks
3. Exploratory data analysis
4. Station-wise analysis
5. Location-wise analysis
6. Vehicle-type analysis
7. Time-based analysis
8. Hourly aggregation
9. Feature engineering
10. Leakage prevention
11. Chronological data splitting
12. Baseline models
13. Machine learning models
14. Validation
15. Model selection
16. Final test evaluation
17. Error analysis
18. Feature importance
19. Station-wise analysis
20. Next-hour forecasting

---

## Limitations

- The dataset contains sparse station-hour observations.
- Empty station-hours are handled using an explicit zero-fill assumption.
- The historical dataset covers a limited period.
- The model does not currently incorporate live traffic, live weather forecasts, real-time charger availability, or real-time grid conditions.
- The test R² is modest, indicating that important demand variation is not captured by the available features.
- MAPE is supplementary because zero-demand observations are present.

---

## Future Scope

Possible future improvements include:

- Real-time EV charging station telemetry
- Live traffic integration
- Weather forecast integration
- Real-time charger availability
- Queue-length forecasting
- Electricity-price forecasting
- LSTM and GRU time-series models
- XGBoost or LightGBM comparison
- Temporal Fusion Transformer
- Probabilistic forecasting
- Multi-step demand forecasting
- Real-time API deployment
- Charging-station capacity planning

---

## Submission Files

The academic submission consists of:

```text
Deepak_EV_Charging_Demand_Prediction.ipynb
requirements.txt
Deepak_ProjectReport.docx
README.md
```

---

## References

- [Electric Vehicle Charging Demand Dataset – Kaggle](https://www.kaggle.com/datasets/ziya07/electric-vehicle-charging-demand-dataset)
- [Scikit-learn Documentation](https://scikit-learn.org/)
- [Pandas Documentation](https://pandas.pydata.org/)
- [NumPy Documentation](https://numpy.org/)
- [Streamlit Documentation](https://docs.streamlit.io/)
