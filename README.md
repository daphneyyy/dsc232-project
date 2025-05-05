# DSC 232 Project - NYC Taxi Trip Destination Prediction

The project aims to predict the drop-off location zone of NYC taxi trips using infomation avalible at the pickup time. The dataset includes records from 2014 to 2024 of NYC green and yellow taxi trips. The project is implemented in PySpark.

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

## Data Exploration

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

## Preprocess Steps

- Dropped extra fields for union dataframes: `airport_fee`, `ehail_fee`, and `trip_type`
  - Check session **"Combine Yellow Taxi and Green Taxi Data"** of notebook
- Since yellow taxi data has extra column `airport_fee`, and green taxi data has extra columns `trip_type` and `ehail_fee`, we will drop these columns before analysis. Additionally, to unify the schema, we will rename `tpep_pickup_datetime` in yellow data and `lpep_pickup_datetime` in green to `pickup_datetime`. Similarly, `tpep_dropoff_datetime` and `lpep_dropoff_datetime` will be renamed to `dropoff_datetime`.
  - Check session **"Combine Yellow Taxi and Green Taxi Data"** of notebook
- Removed outliers using approximate quantiles (IQR method). Additionally, in reality, there shouldn't be negative `passenger_count` and negative `trip_distance`, so, we need to remove them.
    - Check sessions **"Check outliers of columns"** and **"Remove outliers"** of notebook
- Filled missing `passenger_count` using mode per (pickup_day, PULocationID) group
    - Check session **"Check for null values"** of notebook