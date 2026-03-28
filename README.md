# built-with-data
Practical, polished, and purposeful — a portfolio of data projects built to solve real problems.

## Data Cleaning (Power Query)
Raw data contains intentional real-world quality issues:
- 5 different timestamp formats (kernel, WPEFramework, ctrlm, Sky UI styles)
- 25+ dirty device model name variants across 9 canonical models
- CPU temperature sensor failures logged as -1 or 999
- Blank boot times where device crashed before completing boot
- NULL, ERR_UNKNOWN, N/A in error code fields
- 20 duplicate ticket IDs from simulated export bug

All cleaned via Power Query pipeline. Source file preserved untouched.
Transformation table (model_map_1) used with fuzzy merge for model normalisation.
