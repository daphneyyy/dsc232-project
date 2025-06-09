# DSC 232 Project - NYC Taxi Trip Destination Prediction

## Introduction
This project aims to predict the **drop-off location zone** of NYC taxi trips using only the information available at **pickup time**. Predicting a passenger’s destination early in the trip has significant applications in optimizing dispatch systems, anticipating traffic bottlenecks, improving rider safety, and enabling more intelligent pricing or matching systems in urban transport. Our dataset includes records from **2014 to 2024** of NYC green and yellow taxi trips.

The predictive model is developed in **PySpark** and deployed on **UCSD’s SDSC Expanse cluster** due to the large data volume (~100M+ records). The target variable is **DOLocationID** and the model is trained using only features that would be known at pickup.

## Figures
<!-- insert photo -->
![Taxi](Taxicabs_of_New_York_City.jpg)

## Environment Setup

This project was developed and executed using the [UCSD Expanse Portal](https://portal.expanse.sdsc.edu), running Spark notebooks through the Singularity container environment.

**Platform**: SDSC Expanse (https://portal.expanse.sdsc.edu)  
**Singularity Image**: `~/esolares/spark_py_latest_jupyter_dsc232r.sif`  
**Environment Modules Loaded**: `singularitypro`  
**Working Directory**: `home`  
**Notebook Type**: JupyterLab  
**Account**: `TG-CIS240277`  
**Partition**: `shared`

### SLURM Resource Settings Used:
- **Cores**: 10
- **Memory per node**: 16 GB
- **Time limit**: 240 minutes

### Spark Session Example:
If you manually created a Spark session in your notebook, the config might look like:
```python
from pyspark.sql import SparkSession

spark = SparkSession.builder \
    .config("spark.driver.memory", "16g") \
    .config("spark.executor.memory", "16g") \
    .config("spark.executor.instances", 9) \
    .getOrCreate()
```

## Dataset

- **Source**: [Kaggle - NYC Green Yellow Taxi Trip Records](https://www.kaggle.com/datasets/madalagopichand/nyc-green-yellow-taxi-trip-records)
- **Size**: Over 100 million records collected monthly from 2014–2024.
- **Common Columns**:
  - `VendorID`: ID of the provider (e.g., Creative Mobile, Curb)
  - `pickup_datetime` / `dropoff_datetime`: Meter on/off timestamps (renamed from `tpep_` or `lpep_`)
  - `passenger_count`: Number of passengers (may include missing or outlier values)
  - `trip_distance`: Distance of trip in miles
  - `RatecodeID`: Rate type (e.g., Standard, JFK, Group ride)
  - `store_and_fwd_flag`: Indicates if trip was stored before upload (Y/N)
  - `PULocationID`, `DOLocationID`: Pickup and drop-off taxi zone IDs
  - `payment_type`: How the trip was paid (e.g., credit card, cash, dispute)
  - `fare_amount`, `extra`, `mta_tax`, `tip_amount`, `tolls_amount`, `improvement_surcharge`, `total_amount`, `congestion_surcharge`: Fare breakdown fields
- **Yellow Taxi Only Columns**:
  - `airport_fee`: Fee for JFK/LGA airport pickups
  - `cbd_congestion_fee`: Manhattan congestion zone fee (as of Jan 2025)
- **Green Taxi Only Columns**:
  - `ehail_fee`: Fee for e-hail trips (if applicable)
  - `trip_type`: 1 = Street-hail, 2 = Dispatch

## Methods
### Data Exploration

Data distributions and statistics are generated for key features:
- Total observations
- Missing values per column
- Distribution and outliers for:
  - `passenger_count`
  - `trip_distance`
  - `PULocationID` & `DOLocationID`


Visualizations include:
- Bar plots for `passenger_count`
- Bar plots of pickup locations and drop-off zones
- Scatter plots of pickup vs. drop-off zones
- Bar plots showing the count of drop-off locations by day of the week

### Preprocess Steps

- Dropped extra fields for union dataframes: `airport_fee`, `ehail_fee`, and `trip_type`
  - Check session **"Combine Yellow Taxi and Green Taxi Data"** of notebook
- Since yellow taxi data has extra column `airport_fee`, and green taxi data has extra columns `trip_type` and `ehail_fee`, we will drop these columns before analysis. Additionally, to unify the schema, we will rename `tpep_pickup_datetime` in yellow data and `lpep_pickup_datetime` in green to `pickup_datetime`. Similarly, `tpep_dropoff_datetime` and `lpep_dropoff_datetime` will be renamed to `dropoff_datetime`.
  - Check session **"Combine Yellow Taxi and Green Taxi Data"** of notebook
- Removed outliers using approximate quantiles (IQR method). Additionally, in reality, there shouldn't be negative `passenger_count` and negative `trip_distance`, so, we need to remove them.
    - Check sessions **"Check outliers of columns"** and **"Remove outliers"** of notebook
- Filled missing `passenger_count` using mode per (pickup_day, PULocationID) group
    - Check session **"Check for null values"** of notebook


### Models
#### Baseline Model - [Random Forest Classifier](https://spark.apache.org/docs/latest/api/python/reference/api/pyspark.ml.classification.RandomForestClassifier.html)

Selected `RandomForestClassifier` as the baseline model to predict drop-off zone (`DOLocationID`) using only features available at the pickup time.

##### Selected Features
- Categorical: `VendorID`, `RatecodeID`, `store_and_fwd_flag`, `PULocationID`, `payment_type`, `taxi_color`
- Numerical: `passenger_count`, `pickup_day_num`, `pickup_hour` (derived from `pickup_datetime`)

##### Model Configuration
- Pipeline: Used `StringIndexer` and `OneOneHotEncoder` to encode categorical features and assembled them with numeric inputs to train a Random Forest classifier.
- Model: 

  ```python
  rfclf = RandomForestClassifier(
      labelCol="label",
      featuresCol="features",
      numTrees=10,
      maxDepth=10,
      subsamplingRate=0.1,
      featureSubsetStrategy="sqrt"
  )
  ```

#### Second Model - [Gradient-Boosted Trees (GBTs) Classifier](https://spark.apache.org/docs/latest/api/python/reference/api/pyspark.ml.classification.GBTClassifier.html)

##### Selected Features
Same as the Random Forest model, using categorical and numerical features derived from the pickup time.

##### Model Configuration
- Pipeline: Similar to the Random Forest model, using `StringIndexer` and `OneHotEncoder` for categorical features.
- Model:
  ```python
  gbtclf = GBTClassifier(
    labelCol="label",
    featuresCol="features",
    maxIter=50,
    maxDepth=8,
    stepSize=0.1,
    subsamplingRate=0.8,
    seed=42
  )
  ```

## Results

### Model 1: Random Forest Classifier

Not Trained due to extended SDSC server issues affecting job submission and execution.

### Model 2: GBTClassifier
Not Trained due to extended SDSC server issues affecting job submission and execution.

## Discussion

### Data Exploration

The initial data exploration helped us understand the statistics and data distributions of key features. We can identify outliers that deviated significantly from the majority of data points, such as extreme trip distances or unusual passenger counts. These outliers could potentially bias model training and were therefore handled during preprocessing. Time-based fields like pickup hour and day were added to help models detect temporal trends.

### Preprocessing

Preprocessing steps are the most significant part of the project. In this section, I cleaned the data by removing outliers, filling missing values, and unifying the schema between yellow and green taxi datasets. The preprocessing steps ensure that the data is in a suitable format for modeling. 

### Model 1: Random Forest Classifier

Intended as a baseline model due to its scalability and simplicity. It is capable of handling high-dimensional categorical data without normalization. However, model training failed due to SDSC storage error: `java.io.IOException: No space left on device`.

### Model 2: GBTClassifier

The second model is Gradient-Boosted Trees (GBT) Classifier, which is more powerful than Random Forests classifier on capturing complex relationships in the data. The model is intended to improve prediction accuracy but fail to execute due to continued resource issues.

## Conclusion

While I was unable to train both models due to technical issue, I completed an end-to-end pipeline including data cleaning, feature engineering, and full modeling configurations. With stable computational resources, both models are expected to perform well, with GBT offering potentially higher predictive accuracy. This work highlights the value of early-trip data in powering intelligent transportation solutions.

## Collaboration

- **Name**: Xuewen Yang
- **Title**: Independent Contributor
- **Contribution**: I worked solely on the project, including data exploration, preprocessing, model development, and evaluation. This project reflects my individual effort and responsibility as an independent worker.