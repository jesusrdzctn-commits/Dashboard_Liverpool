# Liverpool People Analytics — Employee Attrition Prediction

End-to-end machine learning project developed for **El Puerto de Liverpool** (one of Mexico's largest retail groups) as an academic–industry challenge at **Tecnológico de Monterrey**. Using the demographic profiles and position histories of ~61,000 employees who had already left the company, the team built models that predict **how long a current employee is likely to stay before resigning**, and delivered them through an **interactive web dashboard** that HR stakeholders can operate on their own — no code required.

## Project at a glance

| | |
|---|---|
| **Business problem** | Anticipate voluntary attrition: classify each employee into an expected-tenure bracket — *less than 1 year*, *1 to 5 years*, or *more than 5 years* |
| **Data** | ~61,000 anonymized former-employee records provided by Liverpool: demographic attributes plus position history across Liverpool and Suburbia stores, Sfera boutiques, and distribution centers |
| **Models** | Multinomial Logistic Regression and Random Forest over target-encoded categorical features |
| **Performance** | Both models reach **0.78 accuracy** with ~**0.78 weighted precision and recall** — correctly anticipating the tenure bracket of roughly 8 out of 10 employees |
| **Final product** | ⭐ **`Dashboard_Liverpool.ipynb`** — a multi-page Dash web application with on-demand predictions for uploaded employee files, presented to Liverpool's stakeholders |

> **Reviewing this repository? Open `Dashboard_Liverpool.ipynb` first.** It contains the final product that was delivered and presented to the stakeholders, and it integrates the entire pipeline. The remaining notebooks document, step by step, how it was built.

## Repository guide

| Notebook | Stage | One-line summary |
|---|---|---|
| `Logbook.ipynb` | 1 — Data cleaning | Documented log of every cleaning and feature-engineering decision on the raw HR files |
| `Exploratory_Data_Analysis(EDA).ipynb` | 2 — EDA | Interactive Plotly analysis of attrition patterns that shaped the storytelling and the feature selection |
| `Modeling_Training.ipynb` | 3 — Modeling | Target encoding, training and evaluation of both classifiers; artifacts persisted with joblib |
| `Modeling_Predictions.ipynb` | 4 — Pipeline & inference | Consolidated end-to-end pipeline, from raw files to predictions on new data supplied by Liverpool |
| ⭐ `Dashboard_Liverpool.ipynb` | 5 — **Final product** | Multi-page interactive dashboard with live model predictions, delivered to stakeholders |

## The notebooks in detail

### 1. `Logbook.ipynb` — project log and data cleaning

The working log (bitácora) of the challenge, from raw data to model-ready datasets. Every transformation is recorded together with its rationale, so the notebook doubles as an audit trail of the data pipeline. Starting from the two raw files provided by Liverpool (`RenunciasHistorico.csv`, position history reaching back to 1999, and `RenunciasDemo.csv`, demographic records of former employees), it documents:

- Column standardization and renaming, conversion of text keys to integer IDs, and extraction of reference tables (locations, departments, functions, companies) into separate CSVs
- Date parsing into year/month/day components, duplicate and NaN/NaT checks, and filling of missing department keys
- The merge of each employee's most recent position (from the historical file) into the demographic file
- Noise removal: records with no assigned function, and employees with fewer than 4 months (121 days) of tenure
- Engineered duration features (days, months, quarters) and outlier removal on numeric and categorical columns using IQR and frequency thresholds
- Export of the clean datasets (`HistoricoLimpio.csv`, `DemoLimpio.csv`), closing with descriptive statistics and answers to Liverpool's initial business questions (e.g., *Vendedor Cajero* is the role with the most resignations)

### 2. `Exploratory_Data_Analysis(EDA).ipynb` — exploratory analysis and data storytelling

Interactive exploratory analysis of the clean datasets, built mainly with Plotly Express (supported by matplotlib and seaborn). It examines attrition from the angles most relevant to the business: distributions by gender, number of children and personnel group; resignations by exit year (2019–2022), exit month and exit age; the roles and departments with the highest and lowest turnover; average tenure per position; and location-level views segmented into Liverpool and Suburbia stores, Sfera boutiques and distribution centers (CeDis/Bodegas). These findings powered the data storytelling of the stakeholder presentation and guided the feature selection — for example, confirming *Vendedor Cajero* as the highest-turnover role.

### 3. `Modeling_Training.ipynb` — feature encoding and model training

First modeling notebook. From the clean demographic dataset it applies **target encoding** to six categorical attributes (personnel group, department, function, location, personnel area and company) using the tenure bracket as target, standardizes the features, performs an 80/20 train–test split, and trains and evaluates **Multinomial Logistic Regression** and **Random Forest (70 trees)** with accuracy, weighted precision/recall and full classification reports. All artifacts — both classifiers, the scaler and the six encoders — are persisted with `joblib` so the dashboard can reuse them without retraining.

### 4. `Modeling_Predictions.ipynb` — end-to-end pipeline and inference on new data

The consolidated pipeline, in two parts. **Part 1** reproduces the data preparation from the raw files and engineers the target variable: tenure at exit grouped into three classes (*<1 year*, *1–5 years*, *>5 years*), with cut-offs chosen to keep the classes balanced; the result is exported as `DatasetModelado.csv`. **Part 2** retrains and saves both classifiers, then demonstrates real inference on a **new file provided by Liverpool** (`Ejemplo_Liverpool.csv`): the saved encoders and scaler are applied to the incoming data, and both models return the predicted resignation window with per-class probabilities. This is exactly the inference flow that the dashboard executes behind its upload button.

### 5. ⭐ `Dashboard_Liverpool.ipynb` — final product: interactive dashboard

**The final product, delivered and presented to Liverpool's stakeholders.** A multi-page interactive web application built with **Dash + Plotly** (Bootstrap components, Liverpool brand styling), navigated through a sidebar with four pages:

- **Introducción** — the challenge, the methodology and the models' performance, explained in business terms
- **Predicción Modelo** — the core feature: upload a `.csv` / `.xlsx` file with current-employee data and receive each model's forecast of the resignation window, the per-class probabilities, and a final consensus forecast (the highest-probability class across both models)
- **Gráficos Dataset Demográfico** — four interactive charts (exit-year share, exit-month trend, resignations by role, resignations by department) driven by dropdown and slider filters of the five values 'Resignation Year', 'Number of Children', 'Resignation Month', 'Role' and 'Department'
- **Gráficos Auxiliares** — the tenure-vs-attrition curve and the evolution of the five highest-turnover roles

The app loads the trained models, scaler and encoders via `joblib` and runs predictions in real time on user-uploaded files, with input-format validation and graceful error handling.

## Methodology

Raw HR files → data cleaning and feature engineering (`Logbook`) → exploratory analysis (`EDA`) → target encoding + scaling → model training and evaluation (`Modeling_Training`) → serialized artifacts + inference pipeline (`Modeling_Predictions`) → interactive dashboard with live predictions (`Dashboard_Liverpool`).

Final feature set: birth year and month, number of children, entry month, age at entry, and six target-encoded organizational attributes. Target: expected-tenure bracket at exit (3 classes).

## Model performance

| Model | Accuracy | Precision (weighted) | Recall (weighted) |
|---|---|---|---|
| Multinomial Logistic Regression | 0.78 | 0.7753 | 0.7772 |
| Random Forest (70 estimators) | 0.78 | 0.7848 | 0.7843 |

Evaluated on a held-out 20% test set. In business terms: the models correctly anticipate the expected-tenure bracket of roughly 8 out of 10 employees. Employees with fewer than 4 months of tenure were excluded as noise, so the *<1 year* class covers exits between 4 and 12 months.

## Tech stack

Python (Google Colab) · pandas · NumPy · scikit-learn · category_encoders (target encoding) · joblib · Plotly / Plotly Express · Dash, JupyterDash and dash-bootstrap-components · matplotlib · seaborn

## How to run

The notebooks were developed in Google Colab and read their data from a private shared Drive. With access to the data, run them top to bottom in the pipeline order above; the dashboard launches from the last cell of `Dashboard_Liverpool.ipynb`, served through Colab's proxy on port 8027. Without the data files, the notebooks remain fully readable for review.

## Data and privacy

The datasets were provided by Liverpool exclusively for this academic challenge and are **not** distributed with this repository. Records are identified only by numeric employee IDs — no names or direct personal identifiers — and the ID column is excluded from modeling.

## Team

Developed as a team of three for the Tecnológico de Monterrey × Liverpool challenge:

- Carla Sophia Rodríguez Dander
- Isaac Husny
- Jesús Rodríguez
