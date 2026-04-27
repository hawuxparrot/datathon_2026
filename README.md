# datathon_2026
This repo holds models and attempts for Datathon 2026 Axpo Challenge. Data provided for the challenge is proprietary and property of Axpo Holding AG and is not included in this repo. All model development and iteration during the challenge was done on Databricks platform provided by Axpo for the purpose of the competition only.

### Objective:
The objective is to forecast the retail power consumption for the Iberian Peninsula for the period Dec 2025 ~ Mar 2026. We are given ground truth data from the period Jan 2025 ~ Nov 2025 as well as rudimentary weather forecasts for the same time period. As Axpo AG is an energy trading firm, minimizing error between forecasts and actual usage is paramount to avoid overpaying for sudden demand and underpaying to offload surplus power. Due to Spanish and Portuguese regulations, forecasts for the anticipated demand for the following day must be submitted by 12:00, so any data collected after 12:00, including data for `lag_1d` and weather forecasts made after 12:00, constitutes data leakage. This is very important to note, as this detail can be easily overlooked.

### Approaches
#### 1. "Throw LightGBM on it": devised as a quick baseline, was surprised that it performed well, but discovered it was due to data leakage

#### 2. Fourier Transform + GBT Mixture of Experts
Attempted to use Fourier transform techniques to model seasonal patterns in data + GBTs as mixture of experts to correct residual error, conceptually to model non-periodic factors such as weather, day of the week, special times of year. Failed due to consumption not being stationary (2.5% growth in 2025 for Spain) and model overfitting to outlier events, such as April 28th blackout and summer heat wave spikes.

#### 3. MSTL (Multi-Trend Seasonal Decomposition using LOESS) + GBT Mixture of Experts
MSTL handles multiple seasonalities with different periods better, also handles stationarity, and is less affected by outliers.

We considered using neural networks like RCNNs and LSTMs, ultimately decided against due to superior interpretability of the above models (presentation constitutes significant portion of final performance grade)

### Challenges
**Missing training data for December and January, power consumption down due to Christmas + New Year, then ramps up rapidly to reach yearly peak by January 8th due to cold + getting back to work**. This caused our model to mispredict especially in this time frame.

#### Cultural/Economic peculiarities
- The Spanish and Portuguese have very particular lifestyle patterns that reflect in power consumption. Ex: siesta causes power consumption to drop significantly from 13:00~15:00
- Peak in mornings then again in evenings, probably coinciding with morning work and late dinner times in evening
- Weekday consumption pattern is significantly different from weekend consumption, Sundays even are noticeable lower than Saturdays
- Anomalous spike once in fall turned out to be accounting error caused by DST change (aggregated 2 hours into 1)
- Puente: If there is workday between holiday and holiday, treat day between as holiday for power consumption

#### Yearly schedule peculiarities
- Noticeable spike in last week of November due to Black Week
- Christmas + New Year as mentioned above
- Consumption pattern similar to Sundays on national holidays. Could improve accuracy by taking regional holidays into account

#### Affect of Weather and Seasons
- Peaks in summer, especially when temperature exceeds 30 degrees Celsius --> large-scale use of air conditioning
- Peak in winter due to electric heating
- Anomalous weather events are clearly visible, such as heat waves in July and September

We attempted integating macroeconomic data, model did not perform better there due to long time scales of macroeconomic data (usually only collected once every quarter)
