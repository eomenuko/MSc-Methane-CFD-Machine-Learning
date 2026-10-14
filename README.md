# CFD-Informed Machine Learning Prediction of Methane Flammability Hazard Zones

**MSc Process Safety Engineering — University of Aberdeen**

## Project overview

This repository contains the Python/Jupyter Notebook implementation of a machine-learning (ML) workflow developed to estimate methane flammability hazards following accidental pressurised releases at an industrial facility. The study uses results from Computational Fluid Dynamics (CFD) simulations performed in ANSYS Fluent to train and assess surrogate prediction models.

The industrial scenario represents a methane leak from an export skid at an onshore gas pre-treatment facility in Congo. The models are intended for rapid screening **within the CFD conditions studied**, not as replacements for site-specific dispersion modelling, gas detection design or quantitative risk assessment.

## Prediction targets

1. **Maximum 3D LFL hazard distance (m):** the maximum distance from the release source to the 5% methane-by-volume lower flammability limit (LFL) boundary, as defined in the CFD post-processing workflow. This is **not** necessarily the horizontal distance at personnel height.
2. **Flammable hazard volume (m³):** the simulated volume with methane volume fraction between **0.05 and 0.15** (5–15 vol%).

## CFD dataset and ML inputs

The principal dataset contains **64 CFD cases**, covering a factorial combination of:

| Parameter | Values |
| --- | --- |
| Leak diameter | 5, 10, 15 and 20 mm |
| Wind speed | 1, 2, 4 and 6 m/s |
| Wind direction | North (0°), East (90°), South (180°), West (270°) |

Wind direction is represented by `sin(theta)` and `cos(theta)` to account for its circular nature. The four ML predictors are:

- `Leak_Diameter_mm`
- `Wind_Speed_m_s`
- `Wind_Sin`
- `Wind_Cos`

Leak mass flow rate is **not** an additional independent predictor because it is linked to diameter under the fixed release conditions of this CFD dataset.

## Model development and evaluation

- **56 cases** used for model development and shuffled **5-fold cross-validation** (`random_state=42`).
- **8 cases** withheld for a final blind test: case IDs **1, 12, 18, 31, 35, 46, 56 and 57**.
- **Random Forest Regressor** (`n_estimators=500`, `random_state=42`) used for both targets.
- **Linear Regression** used as a baseline.
- Evaluation metrics: **MAE**, **RMSE**, **R²**, and blind-test **MAPE**.

The final Random Forest models were trained on the 56 development cases, and the eight blind-test cases were not used to tune the models.

### Performance summary

| Target / model | Evaluation | MAE | RMSE | R² | MAPE |
| --- | --- | ---: | ---: | ---: | ---: |
| Maximum 3D LFL distance — Random Forest | 5-fold CV | 0.0250 m | 0.0400 m | 0.9991 | — |
| Maximum 3D LFL distance — Random Forest | 8-case blind test | 0.0138 m | 0.0209 m | 0.9998 | 0.57% |
| Maximum 3D LFL distance — Linear Regression | 5-fold CV | 0.1675 m | 0.1950 m | 0.9794 | — |
| Flammable hazard volume — Random Forest | 5-fold CV | 0.001639 m³ | 0.002899 m³ | 0.9989 | — |
| Flammable hazard volume — Random Forest | 8-case blind test | 0.000878 m³ | 0.001532 m³ | 0.9997 | 0.94% |
| Flammable hazard volume — Linear Regression | 5-fold CV | 0.035920 m³ | 0.037284 m³ | 0.7671 | — |

These scores measure agreement with **CFD-generated outputs**; they do not constitute independent experimental validation of methane dispersion physics.

## Repository files

Suggested files to include:

```text
.
├── README.md
├── MSc_Methane_ML_Final.ipynb
└── CFD_Methane_ML_Ready_64_Cases.xlsx   # Include only if sharing is permitted
```

The notebook contains data preparation, exploratory analysis, cross-validation, model fitting, prediction plots, blind-test comparisons, feature-importance diagnostics and simplified empirical equations.

## How to run the notebook

1. Install **Anaconda** or another Python distribution with JupyterLab.
2. Ensure the following packages are available: `numpy`, `pandas`, `matplotlib`, `scikit-learn` and `openpyxl`.
3. Place the notebook and the ML-ready Excel workbook in the same folder (or update the workbook path in the notebook).
4. Open `MSc_Methane_ML_Final.ipynb` in JupyterLab.
5. Select **Run → Run All Cells** to recreate the models and outputs.

For reference, the analysis was performed using Python packages including NumPy 2.3.5, pandas 2.3.3, Matplotlib 3.10.6 and scikit-learn 1.7.2. These are recorded versions, not a guarantee that other versions will behave identically.

**Data access:** If the Excel dataset is not publicly shared, notebook cells that load it cannot run without an authorised local copy.

## Example prediction

The trained models were also used to illustrate predictions for an intermediate operating condition of **3 m/s wind, SSW (210°), and a 6.35 mm leak**. The Random Forest returned:

- Maximum 3D LFL distance: **0.5223 m**
- Flammable hazard volume: **0.001725 m³**

These are model estimates for an untested intermediate condition, **not** CFD-confirmed or experimentally validated results. Random Forest predictions may be piecewise constant between training diameters.

## Important limitations

- The dataset covers **64 simulations** with only four discrete leak diameters, four wind speeds and four cardinal wind directions.
- The models are specific to the plant geometry, source assumptions, CFD settings and concentration definitions used in the study.
- High test R² values reflect strong agreement within a controlled CFD design space and do not demonstrate reliable extrapolation to other facilities or conditions.
- Model feature importance indicates predictive contributions in this dataset, **not** physical causality.
- A gas detector may alarm at methane concentrations **below the LFL**; these hazard metrics cannot, by themselves, establish whether a detector should activate.
- Use of this repository for engineering decisions requires appropriate verification, site-specific modelling and professional assessment.

## References

- Breiman, L. (2001) ‘Random Forests’, *Machine Learning*, 45, pp. 5–32. https://doi.org/10.1023/A:1010933404324
- Pedregosa, F. *et al.* (2011) ‘Scikit-learn: Machine Learning in Python’, *Journal of Machine Learning Research*, 12, pp. 2825–2830.

## Sharing and attribution

This repository accompanies an MSc academic research project. **Before publishing it publicly**, confirm that any industrial facility details, geometry, simulation outputs and datasets are authorised for release. No open-source licence is granted by this README; add a `LICENSE` file only if you intend to specify reuse permissions.
