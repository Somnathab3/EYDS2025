---

# Urban Heat Island (UHI) Prediction: EY Data Science Challenge

## Overview

This repository documents my participation in the **EY Data Science Challenge**, where I developed a predictive model for the Urban Heat Island (UHI) Index in New York City. I integrated a variety of open-source datasets and engineered over 260 features to capture urban morphology, environmental factors, socio-economic characteristics, and atmospheric conditions. A robust machine learning pipeline was implemented, leveraging ensemble models to achieve high predictive performance. The final model achieved a validation R² score of **0.7823** using an ExtraTreesRegressor.

## Data Sources & Feature Engineering

I utilized a diverse set of open-source datasets, linked to the original training and validation datasets via latitude and longitude coordinates. Below is a detailed breakdown of the datasets, their sources, and the features engineered from them.

### 1. Building Footprints & Heights
- **Features:**
  - Building counts within various radii: `building_count_10m`, `building_count_20m`, ..., `building_count_1000m`
  - Building height and area metrics: `Tallest_Building_*_HEIGHT`, `Average_Building_Height_*` (10m to 1000m), `Total_Building_Area_500m`
- **Sources:**
  - Contest Dataset: [Building_Footprint.kml](https://challenge.ey.com/api/v1/storage/admin-files/7131958802520624-6784d29e124df4f88e5a34db-Building_Footprint.kml)
  - NYC Open Data: [Building Footprints](https://data.cityofnewyork.us/City-Government/Building-Footprints/5zhs-2jue/about_data)
- **Algorithms:**
  - Counted building footprints within multiple distance thresholds (10m to 1000m) for each point in the training and validation datasets.
  - Extracted building height features using the `HEIGHTROOF` attribute via spatial joins with the NYC dataset.

### 2. OpenStreetMap (OSM) Features
- **Features:**
  - Distance metrics: `dist_to_road`, `dist_to_park`, `dist_to_water`
  - Ratio and density metrics: `roads_ratio_*`, `parks_ratio_*`, `water_ratio_*`, `water_kde`, `weighted_water_score` (100m to 1000m)
  - Land use ratios: `landuse_residential_ratio_*`, `landuse_commercial_ratio_*`, etc. (100m to 1000m)
  - Transit and pedestrian features: `transit_count_*`, `ped_cycle_count_*`, `parking_area_ratio_*` (100m to 1000m)
  - Additional urban geometry: `svf_100m` (sky view factor), `road_major_ratio_*`, `road_minor_ratio_*`
- **Source:**
  - OpenStreetMap via [OSMnx](https://osmnx.readthedocs.io/en/stable/)
- **Algorithms:**
  - Computed distances to roads, parks, and water bodies using OSMnx.
  - Calculated asset ratios (e.g., roads, parks, water) within multiple buffer zones.
  - Applied kernel density estimations (KDE) and weighted scores to capture the spatial distribution of urban assets.

### 3. Street Trees Data
- **Features:**
  - Tree counts and average diameters: `tree_count_50m`, `tree_avg_diam_50m`, ..., `tree_count_1000m`, `tree_avg_diam_1000m`
- **Source:**
  - NYC Open Data: [2015 Street Tree Census](https://data.cityofnewyork.us/Environment/2015-Street-Tree-Census-Tree-Data/uvpi-gqnh/about_data)
- **Algorithms:**
  - For each location, counted trees and computed average diameters within various distance thresholds (50m to 1000m).

### 4. Air Quality Data
- **Features:**
  - Pollutant metrics: `Fine particles (PM 2.5)`, `Nitrogen dioxide (NO2)`, `Ozone (O3)`
  - Summary statistics: `pm2.5_avg_JJA`, `pm2.5_median_JJA`, ..., `pm2.5_val_21Jul2021`, and similarly for SO2, CO, NO2, and Ozone
- **Sources:**
  - NYC Open Data: [Air Quality Data](https://data.cityofnewyork.us/Environment/Air-Quality/c3uy-2p5r/about_data)
  - AirNow.gov (US EPA, NOAA, NASA): [AirNow Data](https://gispub.epa.gov/airnow)
- **Algorithms:**
  - Filtered raw data by time periods (e.g., June to August 2021) and pivoted to create location-wise pollutant summaries.
  - Spatially joined pollutant data with training/validation points using a nearest-neighbor approach.

### 5. Elevation Data
- **Feature:**
  - `elevation`
- **Source:**
  - Python package [pyhigh](https://github.com/sgherbst/pyhigh)
- **Algorithm:**
  - Assigned elevation values based on geographical coordinates using the `pyhigh` package.

### 6. Mesonet Weather Data
- **Features:**
  - Weather measurements: `air_temp_surface`, `relative_humidity`, `wind_speed`, `wind_direction`, `solar_flux`, `s2_value`
- **Source:**
  - Contest Dataset: [NY Mesonet Weather](https://challenge.ey.com/api/v1/storage/admin-files/17318943391372033-6784d2b7124df4f88e5a34e8-NY_Mesonet_Weather.xlsx)
- **Algorithms:**
  - Filtered data for a specific time window (e.g., 15:00–16:00 on 2021-07-24) and computed mean values.
  - Assigned weather features to each location based on the nearest weather station using the Haversine distance.

### 7. Satellite Imagery Features
- **Features:**
  - Sentinel-2 indices and bands: `2021_08_25_00_00_2021_08_25_23_59_Sentinel_2_L2A_NDVI`, `NDWI`, `Moisture_index`, `False_color`, etc.
  - Sentinel-3 brightness temperatures and reflectance: `2021_07_24_00_00_2021_07_24_23_59_Sentinel_3_SLSTR_F1_Brightness_Temperature`, etc.
  - Landsat 8: `L8_ST_B10_raw`, `L8_ST_B10_C`, `lst_value`, `lst_value_ndvi`
- **Source:**
  - Contest Dataset: [Sentinel2 GeoTIFF Notebook](https://challenge.ey.com/api/v1/storage/admin-files/7643069366769837-6784d242124df4f88e5a34b9-Sentinel2_GeoTIFF.ipynb)
  - European Space Agency: [Sentinel-3 Data](https://dataspace.copernicus.eu/explore-data/data-collections/sentinel-data/sentinel-3)
- **Algorithms:**
  - Performed raster sampling using the `rasterio` library to extract pixel values based on latitude and longitude.
  - Computed derived indices (e.g., NDVI, NDWI) to assess vegetation health, moisture levels, and urban heat signatures.

### 8. Additional Datasets
- **Cooling Tower Data:**
  - Features: `tower_count_100m`, `tower_count_200m`, `tower_count_500m`, `tower_count_1000m`
  - Source: [NYC Cooling Tower Registrations](https://data.cityofnewyork.us/Health/NYC-Cooling-Tower-Registrations/y4fw-iqfr/about_data)
- **Energy & Water Consumption Data:**
  - Features: `sum_net_emissions_mtco2e_*`, `sum_weather_normalized_site_energy_use_(kbtu)_*`, `sum_weather_normalized_site_natural_gas_use_(therms)_*`, `sum_weather_normalized_site_electricity_(kwh)_*` (500m, 1000m)
  - Source: [Energy and Water Data Disclosure](https://data.cityofnewyork.us/Environment/Energy-and-Water-Data-Disclosure-for-Local-Law-84-/7x5e-2fxh/about_data)
- **Census Tract Data:**
  - Features: Socio-economic metrics like `population_density`, `total_population`, `median_income`, `poverty_count`, ..., `crowded_households`, and aggregated values (`Pop_300m`, `Income_1000m`, etc.)
  - Source: [2020 Census Tracts](https://data.cityofnewyork.us/City-Government/2020-Census-Tracts/63ge-mke6/about_data)
- **Hyperlocal Temperature Data:**
  - Features: Temperature statistics for June, July, August 2018: `_AvgTemp_6`, `_MaxTemp_7`, `_UHI_8`, etc.
  - Source: [Hyperlocal Temperature Monitoring](https://data.cityofnewyork.us/dataset/Hyperlocal-Temperature-Monitoring/qdq3-9eqn/about_data)
- **Wind Atlas Data:**
  - Features: `air_density`, `power_density`
  - Source: [Global Wind Atlas](https://globalwindatlas.info/en/)
- **Street Pavement & Monthly Weather Data:**
  - Features: Pavement ratings (`Width`, `Rating_B`), climate bands (`nclimgrid_band1`, ..., `nclimgrid_band4`)
  - Sources: [Street Pavement Rating](https://data.cityofnewyork.us/Transportation/Street-Pavement-Rating/6yyb-pb25/about_data), [NOAA nClimGrid](https://www.ncei.noaa.gov/access/metadata/landing-page/bin/iso?id=gov.noaa.ncdc:C01589)
- **Traffic Volume Data:**
  - Features: `Traffic_Volume_Avg`, `Traffic_Volume_min`, `Traffic_Volume_max`, `Traffic_Volume_med`
  - Source: [Traffic Volume Counts](https://data.cityofnewyork.us/Transportation/Traffic-Volume-Counts/btm5-ppia/about_data)

## Machine Learning Pipeline

A comprehensive machine learning pipeline was designed to predict the UHI Index. Below is an overview of the process, as implemented in the shared code:

### 1. Data Preprocessing & Imputation
- **Initial Processing:** Dropped non-model columns (`Latitude`, `Longitude`, `datetime`) to focus on predictive features.
- **Missing Value Handling:**
  - Dropped columns with more than 50% missing values.
  - Imputed remaining missing values using a KNN Imputer (`n_neighbors=5`).
  - Filled residual NaN values with 0.
- **Feature Engineering:** Created interaction features by multiplying pairs of environmental variables (e.g., temperature, building, water, tree, park-related features) to capture non-linear relationships, limited to a subset of 5 features to control feature explosion.

### 2. Exploratory Data Analysis (EDA)
- **Feature Distribution Analysis:** Examined all 260+ features for their distributions using histograms and boxplots to identify skewness, outliers, and deviations from normality (e.g., `building_count_1000m` showed right-skewness).
- **Missing Value Assessment:** Calculated the percentage of missing values per feature, flagging columns for removal or imputation.
- **Correlation Analysis:** Computed a correlation matrix to identify multicollinearity (e.g., `Average_Building_Height_500m` and `Average_Building_Height_750m` had a correlation of 0.92).
- **Outlier Detection:** Used Z-scores and IQR methods to detect and cap outliers (e.g., in `tree_avg_diam_1000m`).
- **Target Variable Analysis:** Analyzed the UHI Index for its distribution, noting a near-normal distribution with slight right-skewness.
- **Feature-Target Relationships:** Used scatter plots and Pearson correlations to assess relationships (e.g., `nclimgrid_band1` and `Income_1000m` showed strong correlations with the UHI Index).

### 3. Feature Selection
- **Methods Explored:** Tested multiple feature selection techniques, including Information Value (IV), Recursive Feature Elimination (RFE), Genetic Algorithms (GA), Simulated Annealing (SA), Boruta, Variable Importance (VI), and Linear Regression (LR).
- **Best Method:** Used an **ExtraTreesRegressor** with a median threshold to select 50% of the features, reducing dimensionality while retaining predictive power.

### 4. Model Training & Hyperparameter Optimization
- **Initial Model Exploration:** Tested Neural Networks, Linear Regression, Random Forest, Gradient Boosting, and CatBoost, but they showed low performance (R² < 0.6).
- **Selected Models:** Focused on three ensemble models: ExtraTreesRegressor, XGBoost, and LightGBM.
- **Hyperparameter Optimization (HPO):**
  - **ExtraTrees & XGBoost:** Used Optuna with 200 trials each, employing a Tree-structured Parzen Estimator (TPE) sampler to maximize cross-validation R² scores.
  - **LightGBM:** Used Bayesian Optimization via `BayesSearchCV` with 30 iterations.
  - Cross-validation (5-fold) was used to evaluate model performance during HPO.
- **Model Blending:** Attempted a weighted blending approach, combining predictions from ExtraTrees, XGBoost, and LightGBM using weights proportional to their validation R² scores. However, this did not improve performance over the individual models.

### 5. Final Model & Submission
- **Best Model:** The ExtraTreesRegressor outperformed others with a validation R² score of **0.7823**. Model blending yielded an R² of 0.7791, so ExtraTrees was selected for the final predictions.
- **Retraining:** Retrained the ExtraTrees model on the full training dataset using the best hyperparameters.
- **Prediction:** Generated predictions on the validation dataset and created the submission file by combining the predicted UHI Index with the original latitude and longitude coordinates.
- **Output Files:** Saved the submission (`extratrees_submission_[timestamp].csv`), hyperparameters, blending weights, and feature importance metrics for reference.

### 6. Feature Importance Analysis
The top 10 features contributing to the ExtraTrees model’s predictions were:
1. `nclimgrid_band1`: 0.039678 (climate grid data)
2. `Income_1000m`: 0.032545 (median income within 1000m)
3. `Income_500m`: 0.025860 (median income within 500m)
4. `Average_Building_Height_1000m`: 0.023096 (average building height within 1000m)
5. `tree_avg_diam_1000m`: 0.020426 (average tree diameter within 1000m)
6. `roads_ratio_1000m`: 0.020365 (road ratio within 1000m)
7. `nclimgrid_band4`: 0.019333 (climate grid data)
8. `Traffic_Volume_med`: 0.017134 (median traffic volume)
9. `Average_Building_Height_750m`: 0.016680 (average building height within 750m)
10. `parks_ratio_1000m`: 0.016602 (park ratio within 1000m)

These features highlight the importance of climate data, socio-economic factors, urban structure, and green spaces in predicting UHI intensity.

## Code Structure

- **`feature_engineering.ipynb`:** Contains the code for loading datasets, performing spatial joins, and engineering features (e.g., building counts, OSM ratios, satellite indices).
- **`model_training.py`:** Implements the machine learning pipeline, including preprocessing, feature selection, model training, hyperparameter optimization, blending, and submission generation.
- **`feature_stats_[timestamp].csv`:** Saves feature statistics (mean, std, missing percentage) for analysis.
- **`selected_features_[timestamp].csv`:** Lists the features selected by the ExtraTrees feature selection step.
- **`et_feature_importance_[timestamp].csv`:** Feature importance scores from the final ExtraTrees model.
- **`extratrees_model_[timestamp].pkl`:** Saved ExtraTrees model for production use.
- **`extratrees_submission_[timestamp].csv`:** Final submission file with predicted UHI Index values.

## Dependencies

The project relies on the following Python libraries:
- `pandas`, `numpy`: Data manipulation and numerical operations
- `geopandas`, `shapely`, `osmnx`: Geospatial data processing
- `scipy`, `sklearn`: Feature selection, imputation, and machine learning
- `optuna`, `skopt`: Hyperparameter optimization
- `xgboost`, `lightgbm`, `catboost`: Ensemble models
- `rasterio`: Satellite imagery processing
- `matplotlib`: Visualization for EDA
- `joblib`: Model serialization


- **Best Model:** ExtraTreesRegressor
- **Validation R² Score:** 0.7823
- **Validation RMSE:** [Value from your logs, e.g., 0.3124]
- **Execution Time:** [Value from your logs, e.g., 1254.32 seconds (~20.9 minutes)]

The pipeline successfully captured the complex relationships between urban features and the UHI Index, achieving strong predictive performance.

## Conclusion

This project demonstrates a comprehensive approach to UHI prediction, integrating diverse open-source datasets and applying advanced feature engineering and machine learning techniques. The engineered features capture the multifaceted nature of urban environments, while the machine learning pipeline ensures robust predictions. Future improvements could include incorporating more recent data, exploring deep learning models, or addressing spatial autocorrelation more explicitly.

Feel free to explore the datasets, review the detailed feature engineering steps, and examine the model training and validation process. For any questions, please open an issue or contact me at [your-email@example.com].

---
