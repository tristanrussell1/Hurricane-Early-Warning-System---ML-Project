# Early-Stage Hurricane Formation Detection (3–5 Day Forecast Window)

## Project Overview
This project develops a machine-learning based early-warning system to flag potential hurricane formation events **3–5 days in advance** using historical meteorological data from the National Aeronautics and Space Administration (NASA) POWER API in conjunction with hurricane track data from the National Hurricane Center (NHC) over the past 25 years. Utilizing data from 15 geographic locations dispersed throughout the Hurricane Major Development Region (MDR) in the Atlantic Ocean and Caribbean Sea (10–20°N, 20–80°W), the task is challenging due to a **long prediction horizon** and **severe class imbalance (~6% positive class)**. The model is intentionally designed as a **triage system**, prioritizing recall of storm events to minimize missed formations while reducing unnecessary monitoring workload.

Final results demonstrate strong discriminatory performance (**ROC AUC: 0.813**) while capturing **90% of storm formation events**, making the model suitable for early-warning and prioritization use cases.

---

## Data Description

[NASA POWER API](https://power.larc.nasa.gov/data-access-viewer/):
- Daily geospatial data for 15 locations since the year 2001
- 18 features captured (meteorological, atmospheric, climatological)

[NHC Hurricane Track](https://www.nhc.noaa.gov/data/hurdat/hurdat2-1851-2024-040425.txt):
- Storm tracks for all Atlantic hurricanes by latitude, longitude, and storm severity
- Between 2-4 location flags per day for each storm

Data Merging & Construction:
- Utilized haversine distance between selected locations and hurricane paths to indicate hurricane presence and distance on active days
- Narrowed data to include only days within the "Hurricane Season" (June 1st - November 30th)
- Created binary indicators for multiple distance thresholds (100, 250, 500, and 750 miles) to indicate storm presence in a location
  - Target variable indicator added to points 3-5 days before true storm presence
  - Added indicators to drop active storm days in model testing to avoid target leakage into model training

Key data characteristic: Highly imbalanced target distribution (~6% positive class for largest distance threshold due to storm rarity)

---

## Feature Engineering
Engineered multiple features to improve model performance, notably time window statistics to incorporate data trends.
- Constructed **3–5 day time-lag features** for atmospheric variables
  - Minimum, Mean, Maximum over 3 and 5 day sliding windows
  - Standard Deviation over 3 and 5 day sliding windows
  - Linear Regression Trend over 3 and 5 day sliding windows
- Raw differences for 1, 3, and 5 days prior values
- Joint feature combinations (eg Temp * Humidity)

[Fully Engineered Data](https://gtvault-my.sharepoint.com/:x:/r/personal/trussell47_gatech_edu/_layouts/15/Doc.aspx?sourcedoc=%7B5A7A585E-1F8B-4119-94F0-80A924E16673%7D&file=Completed_Hurricane_Data.csv&action=default&mobileredirect=true)

---

## Modeling Approach
Removed all days of actual hurricanes as well as preceding 1-2 days to avoid target leakage into 3-5 day forecast window. Tested models for storms within 100, 250, 500, and 750 mile radius of point of interest to find best performing criteria.

- Baseline models: Logistic Regression, Random Forest
- Final model: **Random Forest binary classifier**
- Addressed class imbalance using SMOTE oversampling on training data
- Threshold tuning to prioritize recall
- Hyperparameter optimization via **RandomizedSearchCV**
- Cross-validation used to ensure generalization

---

## Evaluation Strategy
Opted not to use accuracy as a primary metric due to class imbalance and asymmetric error costs.

- **Primary metric:** Recall (storm class)
- **Secondary metric:** ROC AUC
- Explicit tradeoff between precision and recall
- False negatives treated as significantly more costly than false positives

---

## Results
- **Storm Recall:** 0.90  
- **Precision:** 0.11  
- **ROC AUC:** 0.813  
- **Monitoring Reduction:** ~55% of locations filtered out  

These results reflect an intentionally cautious model optimized to minimize missed storm formation events while reducing monitoring space by over half.

---

## Key Predictive Drivers
eature importance analysis highlights these factors as key indicators to the model:

- Longitude
- Precipitation over time lags
- Relative humidity over time lags
- Wet bulb temperature trends
- Wind speed variability over time lags

---

## Use Case, Impact, & Limitations
This model would be able to serve as an **early-stage flagging system** for weather monitoring agencies. It successfully eliminates 55% of locations from needing further monitoring, while capturing 90% of all early formation events. The model allows higher-risk flags to be further investigated and resources to be allocated to monitor these locations, serving as an effective triage model and more than halving the monitoring workload. The significantly more costly effects of a false negative flag versus a false positive guided threshold adjustments for storm activity flagging. A 0.90 recall on the positive class with 0.11 precision highlights that this is an inherently cautious model. In this context, false positives represent additional monitoring cost, while false negatives represent missed early-warning opportunities.

The substantial 3–5 day prediction window before storm events posed challenges in model training, and severe class imbalance (6% positive class) limited the model’s false positive rate (89%) from being lower. Despite these constraints, the model demonstrates strong discriminatory performance (ROC AUC: 0.813), indicating reliable ranking of high-risk versus low-risk locations, making it suitable for prioritization and early-warning use cases where minimizing missed events is critical.

---

## Tech Stack
- Python  
- Pandas, NumPy  
- Scikit-learn  
- XGBoost  
- Seaborn, Matplotlib  
