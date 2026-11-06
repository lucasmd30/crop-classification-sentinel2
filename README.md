# Crop Type Classification with Sentinel-2 Imagery

Machine learning pipeline that classifies the main summer land use of 1,821 field plots in western Bahia, Brazil, using a monthly Sentinel-2 time series and the LEM+ ground-truth dataset.

The best model reaches **94.7% accuracy and an F1-macro of 0.76** under spatial cross-validation, and identifies soybean, corn and cotton with an F1 above 0.95. These three crops reach their full-season accuracy by May.

## What it does

1. Loads the LEM+ field polygons and their monthly land-use labels
2. Extracts cloud-masked monthly Sentinel-2 composites per field with Google Earth Engine
3. Explores class balance and NDVI curves per crop
4. Trains and compares models with spatial block cross-validation
5. Analyzes which bands and months matter, and how early crops can be identified

All the work is in [`crop_classification.ipynb`](crop_classification.ipynb), with explanations between the code cells.

## Results

All models were evaluated with 5-fold `StratifiedGroupKFold`, using 10 km spatial blocks as groups.

| Model | Features | Accuracy | F1-macro |
|---|---|---:|---:|
| **Gradient Boosting** | **Time steps (108)** | **0.947** | **0.755** |
| Random Forest | Time steps (108) | 0.941 | 0.718 |
| Gradient Boosting | Summary statistics (36) | 0.919 | 0.684 |
| Random Forest | Summary statistics (36) | 0.876 | 0.658 |
| Logistic regression | NDVI time steps (12) | 0.778 | 0.586 |
| Majority class | None | 0.655 | 0.088 |

The gap between the two best models is within the variation between folds (about 0.05 to 0.06 in F1-macro), so it should not be read as a clear win.

### Performance per class

Results for the best model:

| Class | Fields | Precision | Recall | F1 |
|---|---:|---:|---:|---:|
| Soybean | 1,193 | 0.975 | 0.991 | 0.983 |
| Corn | 168 | 0.976 | 0.964 | 0.970 |
| Cotton | 121 | 0.943 | 0.959 | 0.951 |
| Cerrado | 157 | 0.948 | 0.924 | 0.935 |
| Coffee | 20 | 0.773 | 0.850 | 0.810 |
| Pasture | 95 | 0.745 | 0.768 | 0.756 |
| Hay | 21 | 0.667 | 0.762 | 0.711 |
| Eucalyptus | 26 | 0.765 | 0.500 | 0.605 |
| Other crops | 20 | 0.167 | 0.050 | 0.077 |

### Key findings

- **Spatial validation changes the result.** A random split would report an F1-macro of 0.81 instead of 0.76, because fields would have neighbors from the same farms in the training data.
- **Timing matters.** Keeping the monthly values beats summary statistics for both models.
- **April is the most informative month.** By then, soybean is harvested, corn is declining and cotton is at its peak.
- **NIR (B8) and SWIR (B11) are the most important variables.** NDVI scores close to zero in this test only because it is computed from bands the model already has.

### How early can crops be identified?

| Data up to | Accuracy | F1-macro | Soybean F1 | Corn F1 | Cotton F1 |
|---|---:|---:|---:|---:|---:|
| Dec 2019 | 0.798 | 0.563 | 0.882 | 0.337 | 0.547 |
| Jan 2020 | 0.881 | 0.661 | 0.946 | 0.860 | 0.695 |
| Mar 2020 | 0.916 | 0.662 | 0.970 | 0.929 | 0.835 |
| Apr 2020 | 0.936 | 0.701 | 0.982 | 0.949 | 0.919 |
| May 2020 | 0.947 | 0.733 | 0.983 | 0.973 | 0.951 |
| Sep 2020 | 0.947 | 0.755 | 0.983 | 0.970 | 0.951 |

## How to run

The Sentinel-2 features are included in this repository, so you only need Earth Engine to rebuild them.

1. Clone the repository and install the dependencies (Python 3.12):

   ```bash
   python -m venv .venv
   .venv/Scripts/activate   # Linux and macOS: source .venv/bin/activate
   pip install -r requirements.txt
   ```

2. Download the LEM+ dataset, following [`data/README.md`](data/README.md)
3. Run the notebook:

   ```bash
   jupyter notebook crop_classification.ipynb
   ```

To rebuild the features, delete `data/sentinel2_features.csv`, set `EE_PROJECT` in the notebook to your own Cloud project registered for Earth Engine, and run `earthengine authenticate` once. The extraction takes about 15 minutes.

## Data

**Labels:** LEM+ dataset, with monthly land use for 1,854 fields from October 2019 to September 2020, licensed under [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/).

> Oldoni, L. V., Sanches, I. D., Picoli, M. C. A., Covre, R. M., & Fronza, J. G. (2020). LEM+ dataset: For agricultural remote sensing applications. *Data in Brief*, 33, 106553. https://doi.org/10.1016/j.dib.2020.106553

Dataset: https://doi.org/10.17632/vz6d7tw87f.1

**Imagery:** Copernicus Sentinel-2 L2A surface reflectance (`COPERNICUS/S2_SR_HARMONIZED`), accessed through Google Earth Engine. Contains modified Copernicus Sentinel data (2019–2020).

## Limitations

- **One season, one region.** The models were never tested on another year, so performance could drop with different weather or planting dates.
- **Spatial validation is not perfect.** Fields near the edge of a block can still sit next to fields in the training data, so even 0.76 may be somewhat optimistic for a distant region.
- **Model selection used the same folds as the final scores.** There is no separate test set, so the best score is slightly favored.
- **Few examples for rare classes.** Coffee exists in only a few blocks, and `Other crops` mixes four crops across 20 fields, so these scores are unstable.
- **The season label is a modeling choice.** It uses February, or January for fields harvested early, and ignores second-season crops such as millet.
- **Cloud cover limits rainy months.** Some January and February composites rest on a single clear image. A check found slightly higher blue reflectance in those cases, and no rows were removed.
- **No hyperparameter tuning.** All models use reasonable default settings.

## Next steps

- Test on another season to measure how well the model transfers across years
- Add Sentinel-1 radar, which sees through clouds, to strengthen the rainy months
- Use a 10 to 15 day cadence to capture planting and harvest dates more precisely
- Predict second-season crops with a separate target
- Tune the models with nested spatial cross-validation
