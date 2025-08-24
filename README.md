🔋 Energy Demand Forecasting with Weather Data

## 📋 Project Overview
This project aims to develop a machine learning model to predict electricity demand (load) using historical load data and weather information. The analysis focuses on understanding the relationship between temperature variations and energy consumption patterns in the PJM Interconnection region.

## 📂 Project Structure
```
project/
├── data/
│   ├── PJM_Load_hourly.csv      # Hourly energy demand data
│   └── Temperature_hourly.csv   # Hourly temperature and weather data
├── notebooks/
│   └── demand_forecasting.ipynb # Jupyter notebook with analysis
├── models/                     # Trained models will be saved here
└── README.md
```

## 📊 Data Description

### 1. Energy Demand Data (`PJM_Load_hourly.csv`)
- **Time Period**: 1998-12-31 to 2010-01-02
- **Features**:
  - `Datetime`: Hourly timestamp
  - `PJM_Load_MW`: Energy demand in megawatts

### 2. Weather Data (`Temperature_hourly.csv`)
- **Time Period**: January 2010
- **Features**:
  - `DATE`: Timestamp (UTC)
  - `STATION`: Weather station identifier
  - `STATION_NAME`: Location of weather station
  - `ELEVATION`, `LATITUDE`, `LONGITUDE`: Geographic coordinates
  - `HLY-TEMP-NORMAL`: Temperature in °C (converted from tenths of degrees)
  - `HLY-PRES-NORMAL`: Atmospheric pressure
  - `HLY-DEWP-NORMAL`: Dew point temperature

## 🔍 Analysis Summary
- Performed data cleaning and preprocessing of both datasets
- Conducted exploratory data analysis to understand patterns and relationships
- Identified correlation between temperature and energy demand
- Visualized daily and weekly demand patterns
- Prepared data for time series forecasting

## 🚀 Next Steps
1. **Feature Engineering**
   - Create time-based features (hour of day, day of week, etc.)
   - Add lag features for previous time periods
   - Calculate rolling statistics

2. **Model Development**
   - Implement time series models (ARIMA, SARIMA)
   - Train machine learning models (Random Forest, XGBoost)
   - Consider deep learning approaches (LSTM, GRU)

3. **Model Evaluation**
   - Split data into training and test sets
   - Implement cross-validation
   - Calculate performance metrics (MAE, RMSE, MAPE)

4. **Deployment**
   - Create a simple web interface for predictions
   - Set up automated retraining pipeline

## 🛠️ Requirements
- Python 3.8+
- pandas
- numpy
- matplotlib
- scikit-learn
- jupyter
- statsmodels

## 📝 Usage
1. Install required packages:
   ```bash
   pip install -r requirements.txt
   ```
2. Open and run the Jupyter notebook:
   ```bash
   jupyter notebook notebooks/demand_forecasting.ipynb
   ```

## 📄 License
This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.