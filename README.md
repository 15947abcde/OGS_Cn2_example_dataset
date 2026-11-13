# OGS Cn2 example dataset

## 1. Scope of this public package

This package is an illustrative release of the data format and alignment interface used in the study. It contains meteorological predictors, static OGS descriptors, physics-generated weak labels, and processed measurement-derived turbulence reference labels.

The reference labels were supplied by a third-party data provider. The authors did not conduct the original field measurements and did not receive the raw optical signals, complete calibration records, site-specific hardware configurations, or equipment photographs. This package is therefore not a release of the complete raw measurement archive.

The ten example stations are Nanshan, Muztagh Ata, Ali, Yangbajing, Lenghu, Miyun, Kashgar, Sanya, Lijiang, and Mohe.

## 2. Time grid

The example uses four seven-day periods in 2025:

- Winter: 2025-01-01 to 2025-01-07
- Spring: 2025-04-01 to 2025-04-07
- Summer: 2025-07-01 to 2025-07-07
- Autumn: 2025-10-01 to 2025-10-07

The processed-record interval is five minutes. The five-minute value is the temporal spacing between processed records. It does not mean that each station contains only five minutes of data, and it should not be interpreted as the raw sampling interval of the measurement instrument.

The package contains 10 stations x 28 days x 288 records/day = 80,640 station-time records. This matches the four-season, 28-day evaluation calendar described in the manuscript.

## 3. Files

### `01_weather_forecast.csv`

Illustrative meteorological predictor records aligned to the five-minute example grid.

Columns:

- `station`
- `issue_time_UTC`
- `valid_time_UTC`
- `lead_min`
- `temperature_K`
- `relative_humidity_pct`
- `pressure_Pa`
- `wind_speed_mps`
- `wind_direction_deg`

Each row contains one station and one valid time. The compact public example rotates the available lead times over the valid-time grid. It is intended to demonstrate the input schema and alignment procedure; it is not a complete operational issue-time x lead-time forecast cube.

### `02_ogs_region_description.csv`

Static geographical and environmental descriptors for the ten example OGSs. Join this table to the time-dependent records by `station`.

### `03_weak_labels.csv` and `03_weak_labels.xlsx`

Physics-generated weak labels used for pretraining.

Columns:

- `station`
- `valid_time_UTC`
- `Cn2_weak_m_2_3`
- `log10_Cn2_weak`
- `weak_label_model`

The weak-label model name is `Tatarskii`. The public package does not disclose the detailed internal surface-layer parameterization. All retained weak-label samples receive equal weight during pretraining. No sample-level confidence weight is used.

### `04_reference_labels.csv` and `04_reference_labels.xlsx`

Processed, measurement-derived turbulence reference labels supplied by a third-party data provider.

Columns:

- `station`
- `timestamp_UTC`
- `Cn2_reference_m_2_3`
- `log10_Cn2_reference`
- `instrument_class`
- `reference_height_m`
- `quality_flag`

The instrument metadata disclose only the class `commercial_optical_turbulence_scintillometer`. The exact model, serial identifiers, site-specific configurations, calibration records, and equipment photographs are not released under the data provider's disclosure restrictions.

The nominal reference height is `h0 = 100 m`. It is the height associated with the processed turbulence reference label. It is not the horizontal optical-path length and is not a simulated receiver-aperture height.

The available quality states are `valid` and `invalid_or_missing`. Only records marked `valid` are used as reference targets. Records with non-valid flags retain their positions on the time grid but have blank reference-target values. Missing reference targets are not interpolated, imputed, forward-filled, or generated from neighboring samples.

### `05_station_measurement_summary.csv` and `05_station_measurement_summary.xlsx`

Station-level availability summary derived from the updated reference-label table. It reports the selected time span, processed-record interval, total grid points, valid reference samples, non-valid rate, disclosed instrument class, reference height, and reference-target use rule.

The quality-control rule describes how the supplied provider flag is used by the authors. It does not claim that the authors designed the provider's original instrument-level quality-control procedure.

### `06_methodology_and_alignment.csv`

A compact machine-readable description of the public example, data provenance, weak-label use, reference-label handling, and join rules.

## 4. Alignment

The primary alignment keys are:

- Region descriptors: `station`
- Weather to weak labels: `station + valid_time_UTC`
- Weather to reference labels: `station + valid_time_UTC`, after matching `timestamp_UTC` to `valid_time_UTC`

After label alignment, retain `lead_min` from the weather table. The final forecast sample is indexed by `station + valid_time_UTC + lead_min`.

## 5. Modeling use

1. Load `01_weather_forecast.csv`.
2. Join `02_ogs_region_description.csv` by `station`.
3. Join `03_weak_labels.csv` by `station + valid_time_UTC`.
4. Match `04_reference_labels.timestamp_UTC` to `01_weather_forecast.valid_time_UTC` using `station + time`.
5. Use all retained weak labels with equal weight during pretraining.
6. Use only reference-label records with `quality_flag == "valid"` during reference-label fine-tuning and evaluation.
7. Use the base-10 logarithm of Cn2 for regression because Cn2 spans several orders of magnitude.

Weak labels and measurement-derived reference labels are separate datasets. Weak labels are physics-generated proxies and are not presented as instrument measurements.

## 6. Public-release limitation

This example package supports inspection of the data schema, alignment, and modeling interface. It does not provide raw optical signals, proprietary station logs, detailed calibration records, exact device identifiers, or the complete third-party archive used to prepare the processed reference labels.
