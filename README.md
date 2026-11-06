# Crop Type Classification with Sentinel-2 Imagery

Machine learning pipeline that classifies crop type (soybean, corn and other land-use classes) for field plots in western Bahia, Brazil, using Sentinel-2 imagery and the LEM+ ground-truth dataset from Embrapa.

> Work in progress. Results, usage instructions and citations will be added as each stage is completed.

## Planned pipeline

1. Load LEM+ field polygons and monthly land-use labels
2. Extract cloud-masked Sentinel-2 time series per field with Google Earth Engine
3. Explore class balance and NDVI curves per crop
4. Train and compare models with grouped cross-validation, so no plot leaks between train and test
5. Analyze feature importance and how early in the season crops become distinguishable

## Setup

```bash
python -m venv .venv
pip install -r requirements.txt
```

See [data/README.md](data/README.md) for how to fetch the dataset.
