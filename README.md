# Will it dissolve? Predicting Molecular Solubility with Machine Learning

This project trains a model that takes a molecule's structure and predicts how well it dissolves in water. The idea is to see if a machine can learn the basic chemistry rule (*like dissolves like*) just from data.

The dataset is **ESOL (Delaney)**, 1,128 small organic molecules with measured aqueous solubility values. It's a standard benchmark in cheminformatics.

---

## What's in this repo

- **delaney.csv**: raw dataset (SMILES strings + measured log solubility)
- **solubility-predictor.ipynb**: full pipeline (loading, featurization, training, visualizations)
- **lgbm_model.pkl**: trained LightGBM model
- **results/**: generated plots

---

## How it works

Each molecule is given as a SMILES string, which is just a text encoding of structure (e.g. `CCO` = ethanol). To turn that into something usable, every molecule gets converted to:

- A **Morgan fingerprint** (2,048 binary bits describing local atomic neighborhoods)
- 10 **chemistry descriptors**: molecular weight, LogP, polar surface area, H-bond donors and acceptors, rotatable bonds, aromatic rings, etc.

These feed into a **LightGBM regressor** trained to predict log solubility. Split is 80/20 with early stopping on the validation set.

---

## Results

| Metric | Value |
|---|---|
| **R²** | **0.882** |
| RMSE | 0.747 log mol/L |
| MAE | 0.512 log mol/L |
| Test set size | 226 molecules |

Most predictions are within ±1 log unit of the true value, which is around the noise level of the actual experimental measurements.

---

## Visuals

### Molecules from the test set, predicted vs actual

![molecule grid](results/molecule_grid.png)

Twelve molecules picked across the full solubility range, drawn with predicted and measured values. Green = error under 0.5 log units, orange = close, red = missed.

### Did the model actually learn chemistry?

![chemistry insights](results/chemistry_insights.png)

The right plot shows molecules with higher polar surface area (more polar groups) having higher solubility, and the model's predictions (color) follow the same trend. So yeah, it picked up *like dissolves like* on its own.

### Predictions vs reality

![pred vs actual](results/pred_vs_actual.png)

Points stay close to the diagonal. The blue band is ±1 log unit, which is the usual cutoff for "good enough" in solubility prediction.

### Best vs worst predictions

![best vs worst](results/best_vs_worst.png)

Most of the worst predictions are organophosphate pesticides. These have weird structural patterns that don't show up much in the training data, so the model has nothing to compare them to.

### What features the model used

![feature importance](results/feature_importance.png)

The Morgan fingerprint dominates because it carries the actual structural info. Among the hand-crafted features, **LogP** and **molecular weight** lead, which are also the ones chemists use to estimate solubility by hand.

---

## Why solubility matters

Solubility decides whether a drug reaches your bloodstream after you swallow a pill, whether a pesticide washes off in rain, whether a paint actually dissolves in its solvent. Predicting it from structure alone (no experiment needed) saves a lot of lab work. ESOL has been the go-to benchmark for this since 2004, and R² around 0.88 is in the same range as standard methods.

---

## How to run

```bash
git clone https://github.com/Kreytorn/solubility-predictor.git
cd solubility-predictor
pip install rdkit lightgbm scikit-learn matplotlib pandas numpy
jupyter notebook solubility-predictor.ipynb
```

Runs end-to-end in a couple of minutes on a normal laptop.

---

## Stack

`Python`, `RDKit`, `LightGBM`, `scikit-learn`, `matplotlib`, `pandas`
