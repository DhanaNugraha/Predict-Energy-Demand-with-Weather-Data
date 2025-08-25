# Feature Engineering Analysis

## 1. Data Preparation
```python
# Create a copy of the merged data to work with
df = merged_data.copy()

# Convert temperature to Celsius
df["Temperature_C"] = df["HLY-TEMP-NORMAL"] / 10  # Convert from tenths of degrees
```

## 2. Lag Features
### Short-term Lags (1-6 hours)
```python
for i in range(1, 7):
    df[f"load_lag_{i}h"] = df["PJM_Load_MW"].shift(i)
```
**Purpose**: Captures immediate temporal dependencies in energy demand. The previous 6 hours of load data are often strong predictors of current demand.

### Daily Lag (24 hours)
```python
df["load_lag_24h"] = df["PJM_Load_MW"].shift(24)
```
**Purpose**: Captures daily seasonality. Energy demand often follows similar patterns at the same time each day.

### Weekly Lag (168 hours)
```python
df["load_lag_168h"] = df["PJM_Load_MW"].shift(168)
```
**Purpose**: Captures weekly seasonality. Useful for identifying weekly patterns in energy usage.

## 3. Rolling Statistics
```python
df["load_rolling_24h_mean"] = df["PJM_Load_MW"].shift().rolling(window=24).mean()
df["load_rolling_24h_std"] = df["PJM_Load_MW"].shift().rolling(window=24).std()
```
**Purpose**:
- **24h Mean**: Provides a smoothed version of recent demand, reducing noise
- **24h Std**: Measures volatility in recent demand

## 4. Time-based Features
```python
df["hour"] = df.index.hour
df["day_of_week"] = df.index.dayofweek  # Monday=0, Sunday=6
df["is_weekend"] = df["day_of_week"].isin([5, 6]).astype(int)
```
**Purpose**:
- **Hour**: Captures daily patterns (e.g., morning/evening peaks)
- **Day of Week**: Identifies weekly patterns (weekday vs weekend)
- **Is Weekend**: Binary flag for weekend/holiday effects

## 5. Interaction Features
### Creating Meaningful Interactions
```python
# Weekend interaction
df['load_lag_24h_weekend'] = df['load_lag_24h'] * df['is_weekend']

# Peak hours interaction (17:00-20:00)
df['load_lag_24h_peak'] = df['load_lag_24h'] * (df['hour'].between(17, 20).astype(int))
```

**Purpose**:
- **Weekend Interaction**: Captures how the 24-hour lag relationship changes on weekends
- **Peak Hours Interaction**: Identifies if the 24-hour lag behaves differently during evening peak hours

**Visualization and Analysis of Interaction Effects**:

### Weekend Interaction Analysis
**Visualization**: Scatter plot of 24h lag vs current load, colored by weekend/weekday

**Key Insights**:
- **Different Slopes**: The relationship between 24h lag and current load often shows different slopes for weekdays vs weekends
- **Vertical Separation**: Weekends typically show lower overall demand compared to weekdays at similar lag values
- **Tighter Clustering**: Weekend points may show tighter clustering, indicating more predictable patterns

**Interpretation**:
- The interaction term helps the model learn that the same 24h lag value should be interpreted differently based on whether it's a weekend
- The model can adjust its predictions to account for typically lower weekend demand

### Peak Hours Interaction Analysis
**Visualization**: Scatter plot of 24h lag vs current load, colored by peak hours (17:00-20:00)

**Key Insights**:
- **Demand Amplification**: During peak hours, the same 24h lag value often corresponds to higher current loads
- **Increased Volatility**: Peak hours show greater vertical spread, indicating more variable demand patterns
- **Non-linear Effects**: The relationship may curve upward more steeply during peak hours

**Interpretation**:
- The interaction term helps the model recognize that the 24h lag has a stronger effect during peak hours
- The model can better predict the amplified demand during these critical periods

### Combined Effects
**Visualization**: 3D plot or faceted scatter plots showing lag, current load, and time-based interactions

**Practical Implications**:
- **Model Flexibility**: Interaction terms allow the model to adjust its predictions based on the time context
- **Improved Accuracy**: Capturing these effects can significantly reduce prediction errors during transition periods
- **Feature Importance**: The strength of these interactions can be quantified to understand their relative importance

**Implementation Note**:
- Always visualize interaction effects before and after implementation
- Monitor for overfitting when adding multiple interaction terms
- Consider regularizing interaction terms to prevent overemphasis on specific time periods

## 6. In-Depth Visualization Analysis

### 1. 1-hour Lag vs Current Load
**What it shows**: Scatter plot comparing the load at time t-1 (x-axis) with the current load (y-axis).

**Detailed Interpretation**:
- **Tight Diagonal Clustering**: Points forming a narrow band along the diagonal indicate that the previous hour's load is a strong predictor of current load, showing high autocorrelation at 1-hour intervals.
- **Vertical Spread**: The width of the point cloud shows how much the current load can vary even when the previous hour's load is similar. Wider spread suggests other factors (temperature, time of day) are influencing demand.
- **Outliers**: Points far from the diagonal may indicate measurement errors, sudden demand changes, or external events.
- **Curvature**: Any bending in the point cloud suggests non-linear relationships that might require feature engineering (e.g., polynomial terms).

**Practical Implications**:
- If the R² is high (>0.9), the model will rely heavily on the 1-hour lag feature.
- If the spread is wide, consider adding more features to explain the variance.

### 2. 24-hour Lag vs Current Load
**What it shows**: Scatter plot comparing the load from 24 hours ago with the current load (same hour on consecutive days).

**Detailed Interpretation**:
- **Daily Patterns**: A positive correlation indicates that similar hours on different days have similar demand patterns.
- **Cluster Formation**: Multiple parallel lines might emerge, each representing different day types (weekdays vs weekends, workdays vs holidays).
- **Seasonal Effects**: Different clusters might represent different seasons if the time span is large enough.
- **Heteroscedasticity**: If the spread increases with higher loads, consider log transformation or weighted models.

**Practical Implications**:
- Strong daily patterns suggest that including day-of-week and hour-of-day features will improve predictions.
- Multiple clusters indicate that separate models for different day types might be beneficial.

### 3. 24h Rolling Mean vs Current Load
**What it shows**: Scatter plot comparing the 24-hour moving average (smoothed recent history) with current load.

**Detailed Interpretation**:
- **Central Tendency**: Points clustered around the line y = x indicate the rolling mean is a good baseline predictor.
- **Vertical Dispersion**: The spread shows how much the current load can deviate from recent history, indicating volatility.
- **Systematic Deviations**: Consistent patterns (e.g., points consistently above the line in certain regions) suggest the model could benefit from additional features.
- **Outliers**: Points far from the main cluster may indicate anomalies or special events.

**Practical Implications**:
- A tight cluster suggests the rolling mean could be a useful feature or baseline prediction.
- Large vertical spread indicates that additional features are needed to explain short-term fluctuations.

### 4. Average Load by Hour of Day
**What it shows**: Bar/line chart showing the average load for each hour across all days in the dataset.

**Detailed Interpretation**:
- **Peak Hours**: Identify morning and evening peaks typical in residential/commercial areas.
- **Base Load**: The minimum point typically represents the base load (overnight usage).
- **Ramp Rates**: The steepness of increases/decreases shows how quickly demand changes (important for grid management).
- **Shoulder Periods**: Hours of transition between peak and off-peak may have different characteristics.
- **Weekend vs Weekday**: If data is separated, differences in patterns will be visible.

**Practical Implications**:
- Helps in capacity planning and demand response programs.
- Identifies critical hours where prediction accuracy is most important.
- Suggests potential for different models or parameters for different times of day.

### 5. Additional Recommended Visualizations

#### Load Duration Curve
```python
sorted_load = np.sort(df['PJM_Load_MW'])[::-1]  # Sort in descending order
plt.plot(sorted_load)
plt.title('Load Duration Curve')
plt.xlabel('Hours')
plt.ylabel('Load (MW)')
```
**Purpose**: Shows the distribution of load levels and helps identify:
- Base load (right side of curve)
- Peak load (left side)
- How often different load levels occur

#### Autocorrelation Function (ACF) Plot
```python
from statsmodels.graphics.tsaplots import plot_acf
plot_acf(df['PJM_Load_MW'], lags=168)  # One week of hourly lags
plt.title('Autocorrelation Function')
```
**Purpose**:
- Identifies significant lag values
- Shows daily (lag 24) and weekly (lag 168) seasonality
- Helps determine appropriate lag features to include

## 6. Data Cleaning
```python
# Drop rows with NaN values (created by shifting)
df = df.dropna()
```
**Purpose**: Removes rows with missing values that result from creating lag features.

## 7. Key Insights
1. **Temporal Dependencies**: The strength of lag features indicates how much history is needed for accurate predictions
2. **Seasonality**: Daily and weekly patterns are crucial for energy demand forecasting
3. **Feature Importance**: The most predictive features can be identified through correlation analysis or model feature importance
4. **Data Quality**: Visualizations help identify anomalies or data quality issues

## 8. Key Findings and Recommendations

### Key Findings
1. **Strong 1-hour Lag Correlation**:
   - The 1-hour lag shows the strongest linear relationship with current load
   - Suggests high temporal dependence in energy demand

2. **Time-Dependent Patterns**:
   - Different patterns observed on weekdays vs weekends
   - Evening peak hours (17:00-20:00) show distinct behavior
   - Higher volatility during business hours

3. **Feature Importance**:
   - 1-hour and 24-hour lags are most predictive
   - Time-based features (hour, day of week) add significant value
   - Interaction terms improve model performance

### Recommendations
1. **Feature Engineering**:
   - Focus on short-term lags (1-6 hours) for immediate predictions
   - Include interaction terms for different times of day and week
   - Add rate-of-change features (e.g., hourly change in demand)

2. **Model Development**:
   - Start with a simple model using 1-hour and 24-hour lags
   - Add interaction terms to capture time-dependent effects
   - Consider tree-based models (Random Forest, XGBoost) for handling non-linearities
   - Use time-series cross-validation for robust evaluation

3. **Next Steps**:
   - Investigate specific time periods with high prediction errors
   - Add weather data if available (temperature, humidity, etc.)
   - Consider special events and holidays in the model
   - Implement model monitoring to track performance over time
