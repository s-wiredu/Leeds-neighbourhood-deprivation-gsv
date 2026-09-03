# Predicting Neighbourhood Deprivation in Leeds from Street View Imagery

This repository contains the computational workflow for my MSc dissertation on whether visible streetscape composition carries a spatially generalisable signal of neighbourhood deprivation in Leeds. Google Street View images are converted into nineteen semantic classes with a pretrained SegFormer model, aggregated to Lower-layer Super Output Areas (LSOAs), and evaluated against the English Index of Multiple Deprivation 2025. The analysis is predictive rather than causal, and the image-derived indicators are treated as complementary environmental evidence rather than a replacement for official deprivation measures.

## Overview

The workflow covers street-network sampling, Street View acquisition, semantic segmentation, dataset quality assurance, nested spatial model comparison, SHAP interpretation, out-of-fold spatial error diagnostics, and final results visualisation. All modelling is performed at LSOA level to avoid treating multiple images from the same neighbourhood as independent socioeconomic observations.

The final analysis includes 488 Leeds LSOAs and 19,372 Street View images. Three classifiers are compared across two feature representations:

- Logistic Regression with log-ratio transformed compositions
- Random Forest with raw semantic proportions
- XGBoost with raw semantic proportions
- seven grouped environmental indicators versus the full nineteen-class semantic composition

The same five frozen spatial folds are used for every candidate. Hyperparameter selection and trainable preprocessing are confined to the relevant training partitions.

## Background

Neighbourhood deprivation is multidimensional, but some of the processes associated with it are reflected in the physical environments where people live. Street-level imagery offers detailed coverage of features such as roads, buildings, vegetation, sidewalks, vehicles, walls, fences, and street furniture. The question here is whether those visible characteristics help distinguish high-, medium-, and low-deprivation neighbourhoods when evaluation is geographically separated.

The target is defined from IMD 2025 deciles: High Deprivation covers deciles 1-3, Medium Deprivation covers deciles 4-7, and Low Deprivation covers deciles 8-10. SegFormer is used only as a fixed feature extractor. Its weights are not trained on the Leeds images or IMD labels. Model explanations describe predictive behaviour and are not interpreted as causal effects.

## Installation

The notebooks were developed in Google Colab. Create a Drive directory at:

```text
/content/drive/MyDrive/leeds_gsv_project
```

Place these two source files in its `data` subdirectory:

```text
leeds_gsv_project/
└── data/
    ├── LSOAB.gpkg
    └── imd25.xlsx
```

`LSOAB.gpkg` contains the 2021 LSOA boundaries and `imd25.xlsx` contains the English Indices of Deprivation 2025 source data. The notebooks create the remaining project directories and derived files.

Each notebook contains its own Colab setup cell. The full Python dependency list is also recorded in `requirements.txt`. For a separate environment, install it with:

```bash
python -m pip install -r requirements.txt
```

The notebooks themselves assume the Colab Drive paths above; installing the dependencies alone does not rewrite those paths for a local Jupyter session.

Notebook 02 requires a CUDA-enabled Colab runtime for the production segmentation run. Select a GPU runtime before executing it.

## Usage

Add `GOOGLE_API_KEY` to Colab Secrets and allow Notebook 01 to access it. The key is read from Colab Secrets or the environment and is never printed. Then run the notebooks from top to bottom in numerical order:

1. `01_sampling_and_streetview_download.ipynb` builds road-network sample points, queries Street View metadata once per point, and downloads four headings for each valid location.
2. `02_segformer_and_spatial_cv.ipynb` runs SegFormer inference, aggregates image-level proportions to LSOAs, and constructs the five frozen spatial folds.
3. `03_dataset_quality_assurance.ipynb` checks the frozen feature table, coverage, composition, class balance, leakage guardrails, and fold integrity.
4. `04_nested_spatial_modelling_and_shap.ipynb` compares the six model-representation candidates with nested spatial cross-validation, selects the final model, and produces SHAP outputs.
5. `05_oof_spatial_residual_diagnostics.ipynb` evaluates global and local spatial structure using only strictly out-of-fold prediction errors.
6. `06_results_visualisation.ipynb` reads the frozen outputs and produces the final descriptive tables and Chapter 4 figures without retraining models.

Notebook 01 is restart-safe: cached metadata and validated images are skipped after an interrupted session. Later notebooks expect the output filenames and directory structure created by the earlier stages, so changing those paths requires consistent changes throughout the workflow.

Raw Street View images, source data, derived analytical tables, fitted models, and generated outputs are not included in this repository. Do not commit API keys or other credentials.

## Repository Structure

```text
.
├── 01_sampling_and_streetview_download.ipynb
├── 02_segformer_and_spatial_cv.ipynb
├── 03_dataset_quality_assurance.ipynb
├── 04_nested_spatial_modelling_and_shap.ipynb
├── 05_oof_spatial_residual_diagnostics.ipynb
├── 06_results_visualisation.ipynb
├── requirements.txt
├── .gitignore
├── LICENSE
└── README.md
```

## Results

The full nineteen-class representation outperformed the seven grouped indicators for all three algorithms. The selected model was Random Forest using the raw nineteen-class composition.

| Result | Value |
| --- | ---: |
| Mean outer-fold macro-F1 | 0.527 ± 0.064 |
| Pooled out-of-fold accuracy | 0.547 |
| Pooled out-of-fold balanced accuracy | 0.537 |
| Pooled out-of-fold macro-F1 | 0.534 |
| Majority-class accuracy baseline | 0.361 |
| High-Deprivation F1 | 0.615 |
| Medium-Deprivation F1 | 0.372 |
| Low-Deprivation F1 | 0.616 |
| Observed-probability residual Moran's I | 0.204 |

Full-composition Logistic Regression was close to the selected model, with mean outer-fold macro-F1 of 0.523 and better log-loss and Brier score. SHAP ranked fence, pole, traffic light, vegetation, and wall as the five leading contributors to the selected model. Significant positive spatial autocorrelation remained in strictly out-of-fold errors, which indicates that visible streetscape composition captured only part of the geographically structured information represented by deprivation.

## Citation

```bibtex
@mastersthesis{wiredu2026deprivation,
  author  = {Wiredu, Solomon},
  title   = {Predicting Neighbourhood Deprivation in Leeds Using Google Street View Imagery and Explainable Machine Learning},
  school  = {University of Bradford},
  year    = {2026},
  month   = sep,
  type    = {MSc dissertation}
}
```

## License

The repository code is available under the MIT License. See `LICENSE` for the full terms. Third-party source data and Google Street View imagery remain subject to their own terms and are not redistributed here.
