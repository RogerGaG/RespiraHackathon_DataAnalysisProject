# Respira Hackathon — Pulmonary Fibrosis Biomarker Classification

This repository is a portfolio version of the work completed during the **Respira Hackathon**. The objective was to reduce a very high-dimensional proteomic dataset to a small, interpretable biomarker panel and evaluate whether it could distinguish lower-risk/healthy profiles from more advanced fibrotic disease profiles.

> **Data privacy:** the original hackathon dataset is confidential and is intentionally not included in this repository. No participant-level data are published. The notebooks contain analysis code and aggregate results only.

## Final selected model

The final panel contains three proteins:

- **GDF15**
- **SFTPB**
- **WFDC2**

| Metric | Final 3-protein model |
|---|---:|
| Accuracy | **0.7900** |
| F1 — UNHEALTHY | **0.8235** |
| Sensitivity — UNHEALTHY | **0.7538** |
| Specificity — HEALTHY | **0.8571** |
| ROC AUC | **0.8414** |

The three-protein model was selected as the final hackathon solution because it had the **highest specificity** among the 127 tested combinations while maintaining competitive overall performance and using only three biomarkers.

Larger panels achieved slightly higher sensitivity or ROC AUC, but they required more protein measurements.

## Binary classification target

The original five clinical groups were collapsed into:

**HEALTHY**
- `Healthy_control`
- `Non_stable_fibrotic_ILA`

**UNHEALTHY**
- `Stable_fibrotic_ILA`
- `Progressive_ILA`
- `Established_ILD`

The positive class is **UNHEALTHY**.

---

## Analysis workflow

### 1. Initial data exploration

The first notebook performs an exploratory visualization of clinical variables and protein measurements across the original clinical groups.

Variables such as age, smoking exposure, FVC, DLCO and several candidate proteins showed visible differences across disease groups. This initial analysis was used to understand the structure of the data before feature reduction and modelling.

Notebook: `01_exploratory_data_analysis.ipynb`

### 2. Reducing 2,944 proteins to a manageable candidate set

The dataset contained **2,944 protein variables**, so testing every possible protein combination was computationally unrealistic.

A practical constraint also guided the feature-reduction strategy: in a laboratory setting, using a small number of proteins from different biological pathways is more practical than measuring a very large and redundant panel.

However, a complete manual pathway classification was not feasible. Several protein identifiers had little or no reliable public annotation, so we used **two independent AI-assisted grouping approaches** as a pragmatic first-pass filter:

1. one AI grouping produced **8 broad molecular-pathway groups**; the 3 strongest individual proteins from each group were retained;
2. a second grouping produced **34 finer groups**; the strongest individual protein from each group was retained;
3. the two resulting lists were merged and duplicates removed, leaving roughly **50 candidate proteins**.

These AI-generated pathway groups were known to be imperfect. They were **not used as biological evidence** and were not treated as a definitive pathway annotation. Their only role was to remove a large number of low-priority proteins before multivariate analysis.

The complete pathway-assignment tables are not included in this public repository because they expose the confidential dataset's full protein-variable inventory, and those intermediate files were not retained after the hackathon.

### 3. Multivariate compact-panel analysis

The reduced candidate space was then analysed using multivariate models. The retained aggregate Step 30 analysis used:

- L2 logistic regression;
- stratified 5-fold cross-validation;
- numeric imputation and scaling;
- categorical imputation and one-hot encoding;
- compact protein panels evaluated together with clinical information.

The strongest compact panels repeatedly contained the following proteins:

- **GDF15**
- **SFTPB**
- **MMP7**
- **FGA2242**
- **VCAM1**
- **WFDC2**
- **AREG**

The union of the best compact panels therefore produced the **7-protein shortlist** used in the final binary experiments.

The exact intermediate script was not retained, so Notebook 02 documents the surviving candidate-reduction process and the saved aggregate multivariate results rather than attempting to invent missing code.

Notebook: `02_candidate_reduction_multivariate_shortlist.ipynb`

### 4. Individual performance of the 7 shortlisted proteins

Each of the seven proteins was first evaluated by itself for the HEALTHY/UNHEALTHY problem.

The individual models did not provide a sufficiently strong sensitivity/specificity balance, which motivated testing whether combinations of proteins could improve performance.

Notebook: `03_individual_protein_performance.ipynb`

### 5. Exhaustive combination search

With seven proteins, every non-empty subset can be evaluated:

**2⁷ − 1 = 127 models**

All 127 combinations were ranked by:

- sensitivity;
- specificity;
- ROC AUC;
- accuracy;
- F1 score.

The selected three-protein panel was:

**GDF15 + SFTPB + WFDC2**

It achieved:

- **75.38% sensitivity**
- **85.71% specificity**
- **84.14% ROC AUC**
- **79.00% accuracy**
- **82.35% F1**

A five-protein combination reached a higher sensitivity of approximately **79.23%**, and a six-protein combination reached a higher ROC AUC of approximately **86.66%**. Nevertheless, the three-protein panel was preferred because it provided the highest specificity with a substantially smaller assay.

Why specificity mattered in our decision: specificity measures the proportion of truly HEALTHY participants correctly classified as HEALTHY. Higher specificity therefore means fewer false positives. In a hypothetical clinical workflow where a positive prediction might trigger additional treatment or testing, reducing false positives could reduce unnecessary interventions and costs.

This project did **not** measure treatment outcomes or perform a formal cost-effectiveness analysis, so this should be interpreted as an operational motivation for model selection rather than a clinical/economic claim.

Notebook: `04_exhaustive_protein_combinations.ipynb`

### 6. Final model and subgroup behaviour

The final three-protein model was then analysed across:

- age;
- cigarette pack-years;
- sex.

Importantly, these variables are **not added to the final prediction model**. They are used after prediction to examine whether sensitivity or specificity changes across participant subgroups.

In the saved hackathon results:

- sensitivity increased with age in the shared-interval analysis;
- specificity showed no clear age trend;
- sensitivity showed a positive but non-significant trend with smoking exposure;
- specificity showed no clear smoking trend;
- sensitivity was very similar between male and female participants;
- specificity was also broadly similar between sexes.

These subgroup analyses are exploratory and should not be interpreted as external clinical validation.

Notebook: `05_final_3protein_subgroup_analysis.ipynb`

---

## Repository structure

```text
.
├── 01_exploratory_data_analysis.ipynb
├── 02_candidate_reduction_multivariate_shortlist.ipynb
├── 03_individual_protein_performance.ipynb
├── 04_exhaustive_protein_combinations.ipynb
├── 05_final_3protein_subgroup_analysis.ipynb
├── requirements.txt
└── README.md
```

## Running the notebooks

The private data files used during the hackathon are **not distributed**.

The modelling notebooks expect:

```text
TrainData.csv
TestData.csv
```

They can be placed temporarily in a local `data/` directory:

```text
data/
├── TrainData.csv
└── TestData.csv
```

## Main tools

- Python
- pandas / NumPy
- matplotlib
- scikit-learn
- SciPy
- logistic regression
- stratified cross-validation
- sensitivity / specificity analysis
- ROC analysis
- exhaustive feature-combination search
- bootstrap subgroup analysis

## Limitations

This was a hackathon project using a provided confidential dataset. The results are exploratory and are **not a clinically validated diagnostic or treatment model**.

Additional limitations include:

- no public release of participant-level data;
- AI-assisted pathway grouping was a dimensionality-reduction heuristic rather than validated biological annotation;
- some intermediate early-stage analysis files were not retained;
- the final results are based on the hackathon train/test split and do not constitute external validation;
- subgroup analyses can be unstable when interval sample sizes are small.

The repository is intended to demonstrate the analytical workflow, model-selection reasoning and coding performed during the hackathon while respecting the dataset's privacy constraints.
