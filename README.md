
---

# Urban Heat Island (UHI) Prediction: EY Data Science Challenge

## Overview

This repository documents my participation in the EY Data Science Challenge, where I developed a predictive model for the Urban Heat Island (UHI) Index in New York City. I integrated a variety of open-source datasets and engineered over 260 features to capture urban morphology, environmental factors, socio-economic characteristics, and atmospheric conditions. A robust machine learning pipeline was implemented, leveraging ensemble models to achieve high predictive performance. The final model achieved a validation R² score of 0.9822 using an ExtraTreesRegressor.

## Data Sources & Feature Engineering

I utilized a diverse set of open-source datasets, linked to the original training and validation datasets via latitude and longitude coordinates. Below is a detailed breakdown of the datasets, their sources, and the features engineered from them.

### 1. Building Footprints & Heights
**Features:**
- Building counts within various radii: `building_count_10m`, `building_count_20m`, ..., `building_count_1000m`
- Building height and area metrics: `Tallest_Building_*_HEIGHT`, `Average_Building_Height_*` (10m to 1000m), `Total_Building_Area_500m`

**Sources:**
- Contest Dataset: [Building_Footprint.kml](#repository-files)
- NYC Open Data: Building Footprints

**Algorithms:**
- Counted building footprints within multiple distance thresholds (10m to 1000m) for each point in the training and validation datasets.
- Extracted building height features using the `HEIGHTROOF` attribute via spatial joins with the NYC dataset.

### 2. OpenStreetMap (OSM) Features
**Features:**
- Distance metrics: `dist_to_road`, `dist_to_park`, `dist_to_water`
- Ratio and density metrics: `roads_ratio_*`, `parks_ratio_*`, `water_ratio_*`, `water_kde`, `weighted_water_score` (100m to 1000m)
- Land use ratios: `landuse_residential_ratio_*`, `landuse_commercial_ratio_*`, etc. (100m to 1000m)
- Transit and pedestrian features: `transit_count_*`, `ped_cycle_count_*`, `parking_area_ratio_*` (100m to 1000m)
- Additional urban geometry: `svf_100m` (sky view factor), `road_major_ratio_*`, `road_minor_ratio_*`

**Source:**
- OpenStreetMap via OSMnx

**Algorithms:**
- Computed distances to roads, parks, and water bodies using OSMnx.
- Calculated asset ratios (e.g., roads, parks, water) within multiple buffer zones.
- Applied kernel density estimations (KDE) and weighted scores to capture the spatial distribution of urban assets.

### 3. Street Trees Data
**Features:**
- Tree counts and average diameters: `tree_count_50m`, `tree_avg_diam_50m`, ..., `tree_count_1000m`, `tree_avg_diam_1000m`

**Source:**
- NYC Open Data: [2015_Street_Tree_Census_-_Tree_Data_20250221.csv](#repository-files)

**Algorithms:**
- For each location, counted trees and computed average diameters within various distance thresholds (50m to 1000m).

### 4. Air Quality Data
**Features:**
- Pollutant metrics: Fine particles (PM 2.5), Nitrogen dioxide (NO2), Ozone (O3)
- Summary statistics: `pm2.5_avg_JJA`, `pm2.5_median_JJA`, ..., `pm2.5_val_21Jul2021`, and similarly for SO2, CO, NO2, and Ozone

**Sources:**
- NYC Open Data: [Air_Quality_20250221.csv](#repository-files)
- AirNow.gov (US EPA, NOAA, NASA): [AQ](#repository-files)

**Algorithms:**
- Filtered raw data by time periods (e.g., June to August 2021) and pivoted to create location-wise pollutant summaries.
- Spatially joined pollutant data with training/validation points using a nearest-neighbor approach.

### 5. Elevation Data
**Feature:**
- `elevation`

**Source:**
- Python package `pyhigh`

**Algorithm:**
- Assigned elevation values based on geographical coordinates using the `pyhigh` package.

### 6. Mesonet Weather Data
**Features:**
- Weather measurements: `air_temp_surface`, `relative_humidity`, `wind_speed`, `wind_direction`, `solar_flux`, `s2_value`

**Source:**
- Contest Dataset: [NY_Mesonet_Weather.xlsx](#repository-files)

**Algorithms:**
- Filtered data for a specific time window (e.g., 15:00–16:00 on 2021-07-24) and computed mean values.
- Assigned weather features to each location based on the nearest weather station using the Haversine distance.

### 7. Satellite Imagery Features
**Features:**
- Sentinel-2 indices and bands: `2021_08_25_00_00_2021_08_25_23_59_Sentinel_2_L2A_NDVI`, `NDWI`, `Moisture_index`, `False_color`, etc.
- Sentinel-3 brightness temperatures and reflectance: `2021_07_24_00_00_2021_07_24_23_59_Sentinel_3_SLSTR_F1_Brightness_Temperature`, etc.
- Landsat 8: `L8_ST_B10_raw`, `L8_ST_B10_C`, `lst_value`, `lst_value_ndvi`

**Source:**
- Contest Dataset: [Sentinel2](#repository-files), [Sentinel_3](#repository-files), [Landsat_LST.tiff](#repository-files), [Landsat_NDVI.tiff](#repository-files), [S2_DATA.tiff](#repository-files)

**Algorithms:**
- Performed raster sampling using the `rasterio` library to extract pixel values based on latitude and longitude.
- Computed derived indices (e.g., NDVI, NDWI) to assess vegetation health, moisture levels, and urban heat signatures.

### 8. Additional Datasets
**Cooling Tower Data:**
- **Features:** `tower_count_100m`, `tower_count_200m`, `tower_count_500m`, `tower_count_1000m`
- **Source:** [NYC_Cooling_Tower_Registrations_20250224.csv](#repository-files)

**Energy & Water Consumption Data:**
- **Features:** `sum_net_emissions_mtco2e_*`, `sum_weather_normalized_site_energy_use_(kbtu)_*`, `sum_weather_normalized_site_natural_gas_use_(therms)_*`, `sum_weather_normalized_site_electricity_(kwh)_*` (500m, 1000m)
- **Source:** [Energy_and_Water_Data_Disclosure_for_Local_Law_84_2022__Data_for_Calendar_Year_2021__20250224.csv](#repository-files)

**Census Tract Data:**
- **Features:** Socio-economic metrics like `population_density`, `total_population`, `median_income`, `poverty_count`, ..., `crowded_households`, and aggregated values (`Pop_300m`, `Income_1000m`, etc.)
- **Source:** [nyc_census_tracts.csv](#repository-files)

**Hyperlocal Temperature Data:**
- **Features:** Temperature statistics for June, July, August 2018: `_AvgTemp_6`, `_MaxTemp_7`, `_UHI_8`, etc.
- **Source:** [Hyperlocal_Temperature_Monitoring_20250311.csv](#repository-files)

**Wind Atlas Data:**
- **Features:** `air_density`, `power_density`
- **Source:** [USA_air-density_10m.tif](#repository-files), [USA_power-density_10m.tif](#repository-files)

**Street Pavement & Monthly Weather Data:**
- **Features:** Pavement ratings (`Width`, `Rating_B`), climate bands (`nclimgrid_band1`, ..., `nclimgrid_band4`)
- **Sources:** [StreetAssessmentRating](#repository-files), [nclimgrid-monthly-202107.tif](#repository-files)

**Traffic Volume Data:**
- **Features:** `Traffic_Volume_Avg`, `Traffic_Volume_min`, `Traffic_Volume_max`, `Traffic_Volume_med`
- **Source:** [Automated_Traffic_Volume_Counts_20250319.csv](#repository-files)

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
- **Correlation Analysis:** Computed a correlation matrix to identify multicollinearity
- **Outlier Detection:** Used Z-scores and IQR methods to detect and cap outliers (e.g., in `tree_avg_diam_1000m`).
- **Target Variable Analysis:** Analyzed the UHI Index for its distribution, noting a near-normal distribution with slight right-skewness.
- **Feature-Target Relationships:** Used scatter plots and Pearson correlations to assess relationships (e.g., `nclimgrid_band1` and `Income_1000m` showed strong correlations with the UHI Index).

### 3. Feature Selection
- **Methods Explored:** Tested multiple feature selection techniques, including Information Value (IV), Recursive Feature Elimination (RFE), Genetic Algorithms (GA), Simulated Annealing (SA), Boruta, Variable Importance (VI), and Linear Regression (LR).
- **Best Method:** Used an ExtraTreesRegressor with a median threshold to select 50% of the features, reducing dimensionality while retaining predictive power.

### 4. Model Training & Hyperparameter Optimization
- **Initial Model Exploration:** Tested Neural Networks, Linear Regression, Random Forest, Gradient Boosting, and CatBoost, but they showed low performance (R² < 0.95).
- **Selected Models:** Focused on three ensemble models: ExtraTreesRegressor, XGBoost, and LightGBM.
- **Hyperparameter Optimization (HPO):**
  - **ExtraTrees & XGBoost:** Used Optuna with 200 trials each, employing a Tree-structured Parzen Estimator (TPE) sampler to maximize cross-validation R² scores.
  - **LightGBM:** Used Bayesian Optimization via BayesSearchCV with 30 iterations.
  - Cross-validation (5-fold) was used to evaluate model performance during HPO.
- **Model Blending:** Attempted a weighted blending approach, combining predictions from ExtraTrees, XGBoost, and LightGBM using weights proportional to their validation R² scores. However, this did not improve performance over the individual models.

### 5. Final Model & Submission
- **Best Model:** The ExtraTreesRegressor outperformed others with a validation R² score of 0.973478. Model blending yielded an R² of 0.973082, so ExtraTrees was selected for the final predictions.
- **Retraining:** Retrained the ExtraTrees model on the full training dataset using the best hyperparameters.
- **Prediction:** Generated predictions on the validation dataset and created the submission file by combining the predicted UHI Index with the original latitude and longitude coordinates.
- **Output Files:** Saved the submission (`extratrees_submission_[timestamp].csv`), hyperparameters, blending weights, and feature importance metrics for reference.

### 6. Feature Importance Analysis
The top 10 features contributing to the ExtraTrees model’s predictions were:

- `nclimgrid_band1`: 0.039678 (climate grid data)
- `Income_1000m`: 0.032545 (average income within 1000m)
- `Income_500m`: 0.025860 (average income within 500m)
- `Average_Building_Height_1000m`: 0.023096 (average building height within 1000m)
- `tree_avg_diam_1000m`: 0.020426 (average tree diameter within 1000m)
- `roads_ratio_1000m`: 0.020365 (road ratio within 1000m)
- `nclimgrid_band4`: 0.019333 (climate grid data)
- `Traffic_Volume_med`: 0.017134 (median traffic volume)
- `Average_Building_Height_750m`: 0.016680 (average building height within 750m)
- `parks_ratio_1000m`: 0.016602 (park ratio within 1000m)

These features highlight the importance of climate data, socio-economic factors, urban structure, and green spaces in predicting UHI intensity.

## Code Structure
- `feature_engineering.ipynb`: Contains the code for loading datasets, performing spatial joins, and engineering features (e.g., building counts, OSM ratios, satellite indices).
- `model_training.py`: Implements the machine learning pipeline, including preprocessing, feature selection, model training, hyperparameter optimization, blending, and submission generation.
- `feature_stats_[timestamp].csv`: Saves feature statistics (mean, std, missing percentage) for analysis.
- `selected_features_[timestamp].csv`: Lists the features selected by the ExtraTrees feature selection step.
- `et_feature_importance_[timestamp].csv`: Feature importance scores from the final ExtraTrees model.
- `extratrees_model_[timestamp].pkl`: Saved ExtraTrees model for production use.
- `extratrees_submission_[timestamp].csv`: Final submission file with predicted UHI Index values.

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

**Best Model:** ExtraTreesRegressor

**Validation R² Score in submission:** 0.0.9822

**Execution Time:** [Pipeline completed in 25805.15 seconds (430.09 minutes)
The pipeline successfully captured the complex relationships between urban features and the UHI Index, achieving strong predictive performance.

## Repository Files

Below is a comprehensive list of all files in the repository, including datasets, notebooks, and output files, with their purposes and links.

### Notebooks
- **[1-building-density.ipynb](1-building-density.ipynb)**: Processes building footprint data to calculate building counts and density metrics within various radii (e.g., `building_count_10m` to `building_count_1000m`).
- **[2-mesonet-weather.ipynb](2-mesonet-weather.ipynb)**: Extracts weather data from the NY Mesonet dataset (`NY_Mesonet_Weather.xlsx`) and assigns features like `air_temp_surface`, `wind_speed`, etc., to training/validation datasets based on the nearest weather station.
- **[2-sat-data-lsat-s2-s3.ipynb](2-sat-data-lsat-s2-s3.ipynb)**: Processes satellite imagery data (Sentinel-2, Sentinel-3, Landsat) to extract features like `s2_value`, `lst_value`, `L8_ST_B10_raw`, `2021_08_25_00_00_2021_08_25_23_59_Sentinel_2_L2A_NDVI`, etc.
- **[4-building-height.ipynb](4-building-height.ipynb)**: Extracts building height features (e.g., `Tallest_Building_*_HEIGHT`, `Average_Building_Height_*`) using the `HEIGHTROOF` attribute from building footprint data.
- **[5-cooling-towers.ipynb](5-cooling-towers.ipynb)**: Processes cooling tower data (`NYC_Cooling_Tower_Registrations_20250224.csv`) to calculate tower counts within various radii (e.g., `tower_count_100m`).
- **[6-trees-data.ipynb](6-trees-data.ipynb)**: Processes street tree data (`2015_Street_Tree_Census_-_Tree_Data_20250221.csv`) to calculate tree counts and average diameters within various radii (e.g., `tree_count_50m`, `tree_avg_diam_1000m`).
- **[7-nyc-air-quality-surveillance-data.ipynb](7-nyc-air-quality-surveillance-data.ipynb)**: Processes NYC air quality data (`Air_Quality_20250221.csv`) to extract pollutant metrics (e.g., `pm2.5_avg_JJA`, `NO2_val_21Jul2021`).
- **[8-energy-usage.ipynb](8-energy-usage.ipynb)**: Processes energy and water consumption data (`Energy_and_Water_Data_Disclosure_for_Local_Law_84_2022__Data_for_Calendar_Year_2021__20250224.csv`) to calculate features like `sum_net_emissions_mtco2e_*`.
- **[9-census-tract-2020.ipynb](9-census-tract-2020.ipynb)**: Processes census tract data (`nyc_census_tracts.csv`) to extract socio-economic metrics (e.g., `population_density`, `median_income`).
- **[10-hyperlocal-temperature.ipynb](10-hyperlocal-temperature.ipynb)**: Processes hyperlocal temperature data (`Hyperlocal_Temperature_Monitoring_20250311.csv`) to extract temperature statistics (e.g., `_AvgTemp_6`, `_UHI_8`).
- **[11-street-condition.ipynb](11-street-condition.ipynb)**: Processes street pavement data (`StreetAssessmentRating`) to extract features like pavement width and ratings (e.g., `Width`, `Rating_B`).
- **[12-traffic-movement.ipynb](12-traffic-movement.ipynb)**: Processes traffic volume data (`Automated_Traffic_Volume_Counts_20250319.csv`) to calculate traffic volume statistics (e.g., `Traffic_Volume_Avg`, `Traffic_Volume_med`).
- **[13-air-quality-airnow.ipynb](13-air-quality-airnow.ipynb)**: Processes AirNow air quality data (`AQ`) to extract additional pollutant metrics.
- **[14-elevation.ipynb](14-elevation.ipynb)**: Uses the `pyhigh` package to assign elevation values (`elevation`) to training/validation datasets based on geographical coordinates.
- **[15-globalwind-maps.ipynb](15-globalwind-maps.ipynb)**: Processes Global Wind Atlas data (`USA_air-density_10m.tif`, `USA_power-density_10m.tif`) to extract features like `air_density` and `power_density`.
- **[16-nclimgrid.ipynb](16-nclimgrid.ipynb)**: Processes NOAA nClimGrid data (`nclimgrid-monthly-202107.tif`) to extract climate bands (e.g., `nclimgrid_band1` to `nclimgrid_band4`).
- **[17-osmnx.ipynb](17-osmnx.ipynb)**: Uses OSMnx to extract OpenStreetMap features like `dist_to_road`, `roads_ratio_*`, `parks_ratio_*`, etc.
- **[Model_development.ipynb](Model_development.ipynb)**: Implements the machine learning pipeline, including preprocessing, feature selection, model training, hyperparameter optimization, and prediction generation.

### Datasets
- **[Training_data.csv](Training_data.csv)**: The original training dataset provided for the EY Data Science Challenge, containing latitude, longitude, and target UHI Index values.
- **[Validation_data.csv](Validation_data.csv)**: The original validation dataset provided for the EY Data Science Challenge, containing latitude and longitude for prediction.
- **[Building_Footprint.kml](Building_Footprint.kml)**: KML file containing building footprint data used for calculating building counts and density metrics.
- **[Building Footprints_20250222.geojson](Building Footprints_20250222.geojson)**: GeoJSON file containing additional building footprint data.
- **[NY_Mesonet_Weather.xlsx](NY_Mesonet_Weather.xlsx)**: Excel file containing weather data from NY Mesonet, used to extract features like `air_temp_surface` and `wind_speed`.
- **[2015_Street_Tree_Census_-_Tree_Data_20250221.csv](2015_Street_Tree_Census_-_Tree_Data_20250221.csv)**: CSV file containing street tree census data from NYC Open Data, used for tree-related features.
- **[Air_Quality_20250221.csv](Air_Quality_20250221.csv)**: CSV file containing NYC air quality data, used for pollutant metrics.
- **[AQ](AQ)**: Directory containing AirNow air quality data, used for additional pollutant metrics.
- **[Airquality_Unique_geocode.xlsx](Airquality_Unique_geocode.xlsx)**: Excel file containing geocoded air quality data, used for spatial joining.
- **[NYC_Cooling_Tower_Registrations_20250224.csv](NYC_Cooling_Tower_Registrations_20250224.csv)**: CSV file containing cooling tower registration data, used for tower count features.
- **[Energy_and_Water_Data_Disclosure_for_Local_Law_84_2022__Data_for_Calendar_Year_2021__20250224.csv](Energy_and_Water_Data_Disclosure_for_Local_Law_84_2022__Data_for_Calendar_Year_2021__20250224.csv)**: CSV file containing energy and water consumption data, used for energy-related features.
- **[nyc_census_tracts.csv](nyc_census_tracts.csv)**: CSV file containing 2020 census tract data, used for socio-economic metrics.
- **[census_block_loc.csv](census_block_loc.csv)**: CSV file containing census block locations, used for additional spatial context.
- **[Hyperlocal_Temperature_Monitoring_20250311.csv](Hyperlocal_Temperature_Monitoring_20250311.csv)**: CSV file containing hyperlocal temperature data, used for temperature statistics.
- **[USA_air-density_10m.tif](USA_air-density_10m.tif)**: TIFF file containing air density data from the Global Wind Atlas, used for `air_density` feature.
- **[USA_power-density_10m.tif](USA_power-density_10m.tif)**: TIFF file containing power density data from the Global Wind Atlas, used for `power_density` feature.
- **[USA_wind-speed_10m.tif](USA_wind-speed_10m.tif)**: TIFF file containing wind speed data from the Global Wind Atlas, used for wind-related features.
- **[StreetAssessmentRating](StreetAssessmentRating)**: Directory containing street pavement rating data, used for pavement features.
- **[nclimgrid-monthly-202107.tif](nclimgrid-monthly-202107.tif)**: TIFF file containing NOAA nClimGrid monthly data, used for climate bands.
- **[Automated_Traffic_Volume_Counts_20250319.csv](Automated_Traffic_Volume_Counts_20250319.csv)**: CSV file containing traffic volume data, used for traffic-related features.
- **[Sentinel_3](Sentinel_3)**: Directory containing Sentinel-3 satellite data TIFF files, used for brightness temperature and reflectance features.
- **[Sentinel2](Sentinel2)**: Directory containing Sentinel-2 satellite data TIFF files, used for indices like NDVI, NDWI, etc.
- **[LSAT_8_221022](LSAT_8_221022)**: Directory containing Landsat 8 satellite data TIFF files, used for thermal and reflectance features.
- **[Landsat_LST.tiff](Landsat_LST.tiff)**: TIFF file containing Landsat Land Surface Temperature (LST) data, used for `lst_value`.
- **[Landsat_NDVI.tiff](Landsat_NDVI.tiff)**: TIFF file containing Landsat NDVI data, used for `lst_value_ndvi`.
- **[S2_DATA.tiff](S2_DATA.tiff)**: TIFF file containing Sentinel-2 data, used for `s2_value`.
- **[Landsat_LST.ipynb](Landsat_LST.ipynb)**: Notebook for processing Landsat LST data (likely a draft or alternative script).
- **[nyclion_25a](nyclion_25a)**: Directory containing NYC LION street centerline data, potentially used for road-related features.

### Output Files
- **[ExtraTrees_submission_20250320_002028.csv](ExtraTrees_submission_20250320_002028.csv)**: Final submission file with predicted UHI Index values using the ExtraTreesRegressor model.
- **[blend_weights_20250320_002028.csv](blend_weights_20250320_002028.csv)**: CSV file containing blending weights for the model blending approach (ExtraTrees, XGBoost, LightGBM).
- **[et_feature_importance_20250320_002028.csv](et_feature_importance_20250320_002028.csv)**: CSV file containing feature importance scores from the ExtraTrees model.
- **[extratrees_params_20250320_002028.csv](extratrees_params_20250320_002028.csv)**: CSV file containing the best hyperparameters for the ExtraTrees model.
- **[feature_stats_20250320_002028.csv](feature_stats_20250320_002028.csv)**: CSV file containing feature statistics (mean, std, missing percentage) for analysis.
- **[lgb_feature_importance_20250320_002028.csv](lgb_feature_importance_20250320_002028.csv)**: CSV file containing feature importance scores from the LightGBM model.
- **[lightgbm_params_20250320_002028.csv](lightgbm_params_20250320_002028.csv)**: CSV file containing the best hyperparameters for the LightGBM model.
- **[selected_features_20250320_002028.csv](selected_features_20250320_002028.csv)**: CSV file listing the features selected by the ExtraTrees feature selection step.
- **[xgb_feature_importance_20250320_002028.csv](xgb_feature_importance_20250320_002028.csv)**: CSV file containing feature importance scores from the XGBoost model.
- **[xgboost_params_20250320_002028.csv](xgboost_params_20250320_002028.csv)**: CSV file containing the best hyperparameters for the XGBoost model.

### Other Files
- **[LICENSE](LICENSE)**: The GPL-3.0 license file for the repository.
- **[README.md](README.md)**: The main README file (this document) providing an overview of the project.
- **[EYDS_Flowchart-2025-03-21-231135.svg](EYDS_Flowchart-2025-03-21-231135.svg)**: SVG file containing a flowchart of the EY Data Science Challenge workflow.
- **[Version.docx](Version.docx)**: Document containing version history or additional notes about the project.

## Conclusion

This project demonstrates a comprehensive approach to UHI prediction, integrating diverse open-source datasets and applying advanced feature engineering and machine learning techniques. The engineered features capture the multifaceted nature of urban environments, while the machine learning pipeline ensures robust predictions. Future improvements could include incorporating more recent data, exploring deep learning models, or addressing spatial autocorrelation more explicitly.

Feel free to explore the datasets, review the detailed feature engineering steps, and examine the model training and validation process. For any questions, please open an issue or contact me at [your-email@example.com].

---

### Changes Made

1. **Added Repository Files Section:**
   - Created a new section called "Repository Files" that lists all files in the repository, categorized into Notebooks, Datasets, Output Files, and Other Files.
   - Each file entry includes a brief description of its purpose and a link to its location in the repository (using relative Markdown links).

2. **Linked to Uploaded Files:**
   - Included all uploaded files mentioned in your list, such as `Training_data.csv`, `Validation_data.csv`, `ExtraTrees_submission_20250320_002028.csv`, `feature_stats_20250320_002028.csv`, etc.
   - Linked datasets like `NY_Mesonet_Weather.xlsx`, `Air_Quality_20250221.csv`, `Sentinel_3`, etc., which are referenced in the Data Sources section and the notebooks.

3. **Linked to Notebooks:**
   - Included all notebooks (e.g., `1-building-density.ipynb`, `2-mesonet-weather.ipynb`, `2-sat-data-lsat-s2-s3.ipynb`, etc.) with their purposes based on the feature engineering tasks they perform.
   - Reflected the renamed notebooks (`2-sat-data-lsat-s2-s3.ipynb` and `9-census-tract-2020.ipynb`) as per your upload history.

4. **Cross-Referenced with Data Sources:**
   - Ensured that the datasets mentioned in the "Data Sources & Feature Engineering" section (e.g., `NY_Mesonet_Weather.xlsx`, `Sentinel2`, `Air_Quality_20250221.csv`) are linked to their corresponding entries in the "Repository Files" section.

5. **Added Descriptions:**
   - Provided concise descriptions for each file to explain its role in the project (e.g., "Processes building footprint data to calculate building counts and density metrics" for `1-building-density.ipynb`).

6. **Organized by Category:**
   - Grouped files into logical categories (Notebooks, Datasets, Output Files, Other Files) to improve readability and navigation.
