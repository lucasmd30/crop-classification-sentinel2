# Data

## LEM+ labels

Download `LEM_dataset.zip` from [Mendeley Data](https://data.mendeley.com/datasets/vz6d7tw87f/1) and extract it to `data/raw/`.

The raw files are not tracked in this repository.

## Sentinel-2 features

`sentinel2_features.csv` holds one row per field and month, from October 2019 to September 2020. It was extracted with Google Earth Engine from `COPERNICUS/S2_SR_HARMONIZED`, using monthly median composites averaged over each field polygon at 20 m.

| Column | Description |
|---|---|
| `id` | Field ID from the LEM+ shapefile |
| `month` | First day of the month |
| `label` | LEM+ land-use label for that month |
| `B2`, `B3`, `B4`, `B8`, `B11`, `B12` | Surface reflectance (blue, green, red, NIR, SWIR 1, SWIR 2) |
| `NDVI`, `EVI`, `NDWI` | Vegetation and water indices computed from the composite |
| `clear_obs` | Mean number of cloud-free images behind each pixel of the field |
