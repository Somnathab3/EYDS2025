# Contest Participation – Datasets & Feature Engineering

I participated in a data science contest where I integrated diverse open datasets and engineered over 260 features to capture urban morphology, environmental factors, and socio-economic characteristics. The data spans a wide range of domains—from building footprints and OpenStreetMap (OSM) assets to satellite imagery, air quality, energy consumption, census tract data, and hyperlocal weather measurements.

## Data Sources & Feature Engineering

### 1. Building Footprints & Heights
- **Features:**  
  - Building counts within various radii (e.g., `building_count_10m`, `building_count_20m`, …, `building_count_1000m`)
  - Building height and area metrics (e.g., `Tallest_Building_*_HEIGHT`, `Average_Building_Height_*`, `Total_Building_Area_500m`)
- **Sources:**  
  - Contest-provided *Building_Footprint.kml*
  - NYC OpenData – [Building Footprints](https://data.cityofnewyork.us/City-Government/Building-Footprints/5zhs-2jue/about_data)
- **Algorithms:**  
  - For each point in the training and validation datasets, the number of building footprints is computed within multiple distance thresholds.
  - Height features are derived via spatial joins using the `HEIGHTROOF` attribute from the NYC dataset.

### 2. OpenStreetMap (OSM) Features
- **Features:**  
  - Distance metrics (e.g., `dist_to_road`, `dist_to_park`, `dist_to_water`)
  - Ratio and density metrics (e.g., `roads_ratio_*`, `parks_ratio_*`, `water_kde`, `weighted_water_score`)
  - Additional urban geometry metrics (e.g., street connectivity, canyon aspect ratio, roof area ratio)
- **Source:**  
  - OpenStreetMap data accessed via [OSMnx](https://osmnx.readthedocs.io/en/stable/)
- **Algorithms:**  
  - Compute distances and asset ratios within multiple buffer zones.
  - Kernel density estimations and weighted scores are used to capture the spatial distribution of urban assets.

### 3. Street Trees Data
- **Features:**  
  - Tree counts and average tree diameters (e.g., `tree_count_50m`, `tree_avg_diam_50m`, …, `tree_count_1000m`, `tree_avg_diam_1000m`)
- **Source:**  
  - NYC OpenData from the Street Tree Census (2015 to 2028)
- **Algorithms:**  
  - For each location, count trees and compute average diameters within various distance thresholds.

### 4. Air Quality Data
- **Features:**  
  - Pollutant metrics including PM 2.5, Nitrogen dioxide (NO₂), and Ozone (O₃)
  - Summary statistics such as averages, medians, and standard deviations (e.g., `pm2.5_avg_JJA`)
- **Sources:**  
  - NYC OpenData – [Air Quality Data](https://data.cityofnewyork.us/Environment/Air-Quality/c3uy-2p5r/about_data)
  - U.S. Environmental Protection Agency (EPA)
- **Algorithms:**  
  - The raw data is filtered by desired time periods, pivoted, and then spatially joined with training/validation points using a nearest-neighbor approach.

### 5. Elevation Data
- **Feature:**  
  - `elevation`
- **Source:**  
  - Python package [pyhigh](https://pypi.org/project/pyhigh/)
- **Algorithm:**  
  - Elevation values are assigned based on the geographical coordinates of each point.

### 6. Mesonet Weather Data
- **Features:**  
  - Weather measurements such as `air_temp_surface`, `relative_humidity`, `wind_speed`, `wind_direction`, and `solar_flux`
- **Source:**  
  - NYC Mesonet Weather Data ([Download Link](https://challenge.ey.com/api/v1/storage/admin-files/17318943391372033-6784d2b7124df4f88e5a34e8-NY_Mesonet_Weather.xlsx))
- **Algorithms:**  
  - Data are filtered by a specific time window and assigned to each location based on the nearest weather station.

### 7. Satellite Imagery Features
- **Features:**  
  - Various indices and spectral bands from Sentinel-2, Sentinel-3, and Landsat 8 (e.g., `s2_value`, `L8_ST_B10_raw`, `2021_08_25_00_00_2021_08_25_23_59_Sentinel_2_L2A_NDVI`, etc.)
- **Source:**  
  - Satellite imagery provided as contest data (e.g., [Sentinel2 GeoTIFF Notebook](https://challenge.ey.com/api/v1/storage/admin-files/7643069366769837-6784d242124df4f88e5a34b9-Sentinel2_GeoTIFF.ipynb))
- **Algorithms:**  
  - Raster sampling is performed using the `rasterio` library to extract pixel values based on latitude and longitude.
  - Derived indices are computed to assess vegetation health, moisture levels, and urban heat signatures.

### 8. Additional Datasets
- **Cooling Tower Data:**  
  - Features: `tower_count_100m`, `tower_count_200m`, etc.  
  - Source: [NYC Cooling Tower Registrations](https://data.cityofnewyork.us/Health/NYC-Cooling-Tower-Registrations/y4fw-iqfr/about_data)
- **Energy & Water Consumption Data:**  
  - Features: Net emissions and normalized energy use (e.g., `sum_net_emissions_mtco2e_500m`, `sum_weather_normalized_site_energy_use_(kbtu)_500m`)
  - Source: [Energy and Water Data Disclosure](https://data.cityofnewyork.us/Environment/Energy-and-Water-Data-Disclosure-for-Local-Law-84-/7x5e-2fxh/about_data)
- **Census Tract Data:**  
  - Features: Socio-economic metrics such as `population_density`, `total_population`, `median_income`, etc., along with aggregated values for buffers (e.g., `Pop_300m`, `Income_1000m`)
  - Source: [2020 Census Tracts](https://data.cityofnewyork.us/City-Government/2020-Census-Tracts/63ge-mke6/about_data)
- **Hyperlocal Temperature Data:**  
  - Features: Temperature statistics (average, maximum, minimum, standard deviation, UHI) for different months (June, July, August 2018)
  - Source: [Hyperlocal Temperature Monitoring](https://data.cityofnewyork.us/dataset/Hyperlocal-Temperature-Monitoring/qdq3-9eqn/about_data)
- **Wind Atlas Data:**  
  - Features: `air_density`, `power_density`
  - Source: [Global Wind Atlas](https://globalwindatlas.info/en/)
- **Street Pavement & Monthly Weather Data:**  
  - Features include pavement ratings (`Width`, `Rating_B`) and climate bands from NOAA's nClimGrid
  - Sources: [Street Pavement Rating](https://data.cityofnewyork.us/Transportation/Street-Pavement-Rating/6yyb-pb25/about_data) and [NOAA nClimGrid Metadata](https://www.ncei.noaa.gov/access/metadata/landing-page/bin/iso?id=gov.noaa.ncdc:C01589)
- **Traffic Volume Data:**  
  - Features: Traffic volume metrics (average, minimum, maximum, median) collected during a specific time window
  - Source: [Traffic Volume Counts](https://data.cityofnewyork.us/Transportation/Traffic-Volume-Counts/btm5-ppia/about_data)

## Machine Learning Process – A Brief Overview

A robust machine learning pipeline was designed to predict the UHI Index. Here’s a brief summary of the process:

- **Data Preprocessing & Imputation:**  
  - After merging all features, non-model columns were dropped.
  - Missing values were imputed using a KNN-based approach.
  - Interaction features were created from a subset of key environmental variables.

- **Feature Selection:**  
  - An initial feature selection was performed using an ExtraTreesRegressor to reduce dimensionality while preserving informative features.

- **Model Training & Hyperparameter Optimization:**  
  - Three models were explored: ExtraTrees, XGBoost, and LightGBM.
  - Hyperparameter tuning was conducted using Optuna (for ExtraTrees and XGBoost) and Bayesian Optimization (for LightGBM).
  - Each model’s performance was validated using cross-validation and R² metrics.
  - A blending approach was applied by weighting predictions from each model based on their validation R² scores.

- **Final Model & Submission:**  
  - The best model (or a blend of models) was retrained on the full training data, and predictions were generated for the validation set.
  - The final submission and all hyperparameters, as well as feature importance metrics, were saved for reference.

## Conclusion

This repository documents a comprehensive feature engineering effort integrating multiple data sources and a sophisticated machine learning pipeline. The engineered features not only capture the complex urban environment but also drive high-performance model predictions for the UHI Index.

Feel free to explore the datasets, review the detailed feature engineering steps, and examine the model training and validation process.

