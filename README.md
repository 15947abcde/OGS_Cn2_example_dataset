OGS Cn2 DATASET
============================================================

1. PURPOSE

The ten stations are: Nanshan, Muztagh Ata, Ali, Yangbajing, Lenghu,
Miyun, Kashgar, Sanya, Lijiang, and Mohe.

The time grid contains four 35-day seasonal periods: - Winter:
2025-01-01 to 2025-02-04 - Spring: 2025-04-01 to 2025-05-05 - Summer:
2025-07-01 to 2025-08-04 - Autumn: 2025-10-01 to 2025-11-04

The unified temporal resolution is 5 minutes.

Dataset scale: 10 stations x 4 seasons x 35 days x 288 time points/day =
403,200 station-time points.

============================================================ 2. FILES IN
THE PACKAGE ============================================================

01_weather_forecast.csv

Content: archived-forecast-like meteorological data.

Each row represents one forecast record for one station at one future
valid time.

Columns: - station OGS station name.

-   issue_time_UTC  forecast initialization/issue time in UTC.

-   valid_time_UTC Time for which the forecast is valid, in UTC.

-   lead_min Forecast lead time in minutes.

-   temperature_K Forecast near-surface air temperature in kelvin.

-   relative_humidity_pct Forecast relative humidity in percent.

-   pressure_Pa Forecast atmospheric pressure in pascals.

-   wind_speed_mps Forecast wind speed in m/s.

-   wind_direction_deg Forecast wind direction in degrees, 0-360.

How to use: Use this table as the meteorological predictor table. For
each modeling sample, identify the station, valid time, and lead time.
The meteorological variables can then be used directly as ML inputs or
as inputs to a physical weak-label model.

The lead times cover: 0, 5, 10, …, 120 minutes.

In this dataset, lead times rotate across the 5-minute
valid-time grid so that the dataset keeps exactly 403,200 station-time
records.

============================================================

02_ogs_region_description.csv

Content: Static geographical and environmental attributes of the ten OGS
stations.

Each station appears exactly once.

Columns: - station - latitude_deg - longitude_deg - altitude_m -
climate_type - terrain_type - land_surface_type

How to use: Join this table to the time-dependent data by “station”.

Example: weather_data LEFT JOIN region_description ON
weather_data.station = region_description.station

These attributes are constant during training and testing. They can be
used as static model features, station metadata, categorical embeddings,
or regional generalization variables.

============================================================

03_weak_labels.csv

Content: physics-informed weak labels of the optical
turbulence parameter Cn2.

Each row corresponds to one station and one valid time.

Columns: - station - valid_time_UTC - Cn2_weak_m_2_3 Weak-label Cn2 in
m^(-2/3).

-   log10_Cn2_weak Base-10 logarithm of the weak-label Cn2.

-   weak_label_model Name of the weak-label parameterization.

-   confidence_weight Reliability weight assigned to the weak label.

The model name used in this package is:
Tatarskii_GD_surface_v1

The optical conversion is based on the commonly used relation:

Cn2 = (79e-6 * P_hPa / T_K²⁾2 * CT2

For this example dataset, CT2 is generated with a surface-layer proxy using temperature, pressure, wind speed, relative
humidity, a solar/ diurnal proxy, measurement height, and a terrain
multiplier.

How to use: Weak labels can be used for: - weakly supervised learning; -
pretraining; - auxiliary loss functions; - physics-informed
regularization; - training at station-times without valid measured
labels.

A typical weighted weak-label loss is conceptually:

L_weak = confidence_weight * loss(predicted_log10_Cn2, log10_Cn2_weak)

The confidence weight decreases with increasing forecast lead time and
is also reduced when meteorological inputs enter less typical ranges.

============================================================

04_measured_labels.csv

Content: measured-label-like Cn2 records.

Each row represents one station at one 5-minute timestamp.

Columns: - station - timestamp_UTC - Cn2_measured_m_2_3
measured Cn2 in m^(-2/3).

-   log10_Cn2_measured Base-10 logarithm of measured Cn2.

-   instrument Instrument model used for the measurement.

-   measurement_height_m Example measurement/path height.

-   quality_flag Quality-control status.

Possible quality flags include: - valid - missing - precip_screened -
saturation_screened - low_signal_screened

How to use: For supervised training or evaluation, normally retain only
rows where:

quality_flag == “valid”

Rows rejected by quality control remain on the 5-minute grid, while
their Cn2 values are blank. Keeping these rows makes temporal alignment
easier.

The “instrument” field records the scintillometer model associated with
the on-site measurement records.

============================================================

05_station_measurement_summary.csv

Content: Station-level summary of the measurement-label
dataset.

It contains, for each station: - collection start and end
times; - number of selected days; - assumed raw sampling interval; -
unified 5-minute interval; - total 5-minute grid points; - number of
valid samples after QC; - missing/screened rate; - instrument model; -
measurement/path height; - quality-control rule; - counts
of different QC categories.

How to use: Use this file for: - describing dataset availability in a
paper; - checking sample counts before model training; - comparing data
completeness among stations; - constructing train/validation/test
statistics; - identifying stations with relatively high missing rates.

RECOMMENDED DATA ALIGNMENT
============================================================

The intended final sample identifier is:

station + valid_time_UTC + lead_min

Recommended joins:

A. Region attributes Join by: station

B. Weather and weak labels Join: 01_weather_forecast.valid_time_UTC with
03_weak_labels.valid_time_UTC

using station + valid_time_UTC.

C. Weather and measured labels Join: 01_weather_forecast.valid_time_UTC
with 04_measured_labels.timestamp_UTC

using station + time.

After the labels are aligned to a forecast record, retain lead_min from
the weather table so that the final modeling sample is indexed by:

station + valid_time_UTC + lead_min

============================================================ 
RECOMMENDED MODELING WORKFLOW
============================================================

A typical workflow is:

Step 1: Load 01_weather_forecast.csv.

Step 2: Join 02_ogs_region_description.csv by station.

Step 3: Join 03_weak_labels.csv by station + valid_time_UTC.

Step 4: Rename 04_measured_labels.timestamp_UTC to valid_time_UTC, or
explicitly join the two time fields.

Step 5: Join the measured-label table by station + valid_time_UTC.

Step 6: Use quality_flag to determine whether the measured label can be
used as a strong supervised target.

Step 7: Use log10_Cn2 rather than raw Cn2 for most regression
experiments because Cn2 spans several orders of magnitude.

Possible feature groups: - Dynamic forecast features: temperature,
relative humidity, pressure, wind speed, wind direction.

-   Forecast metadata: lead_min, issue time, valid time.

-   Static regional features: latitude, longitude, altitude, climate
    type, terrain type, land-surface type.

-   Weak supervision: log10_Cn2_weak and confidence_weight.

-   Strong target: log10_Cn2_measured for valid observations.

============================================================ 
