# Temporal Modeling and Counterfactual Policy Simulation of Student Dropout

Main project notebook supporting the paper **“A Mathematical Framework for Temporal Modeling and Counterfactual Policy Simulation of Student Dropout.”** The pipeline transforms temporal academic-engagement data into a weekly *person-period* dataset, trains a temporal risk model, reconstructs survival trajectories, and runs counterfactual policy simulations to compare intervention scenarios.

## Objective

This repository is organized to reproduce, audit, and extend the paper’s core experiment. The goal is not only to predict **who** is at higher risk of dropout, but also **when** that risk increases over academic time and how explicit intervention rules change, in simulation, the survival trajectories estimated by the model.

## What this notebook does

The notebook implements an end-to-end workflow covering:

1. integration of the **OULAD** tables;
2. construction of the unit of analysis at the **enrollment** level;
3. weekly expansion into **person-period** format;
4. definition of the primary event as **Withdrawn** with valid `date_unregistration`;
5. engineering of time-varying covariates such as `total_clicks`, `recency`, `streak`, and activity indicators;
6. temporally stratified splitting with leakage control;
7. training of the main **discrete-time hazard** model using penalized logistic regression with sigmoid calibration;
8. reconstruction of predicted survival curves;
9. simulation of counterfactual policies under **shock** and **mechanism-aware** versions;
10. subgroup analysis with *bootstrap* to measure change in the gap between observable groups.

## Research questions covered

The notebook is structured to answer three central questions:

- **RQ1 — Temporal hazard quality:** does the weekly model discriminate and calibrate well enough to support downstream decisions?
- **RQ2 — Structural contrast across regimes:** does a deterministic recency-based rule produce an interpretable survival contrast under different simulated policy scenarios?
- **RQ3 — Change in subgroup gap:** does the same policy change the gap between observable groups in a direction that remains stable under bootstrap uncertainty?

## Methodological summary

### 1) Unit of analysis

The analytical unit is the **enrollment**, identified by the triple:

```text
(id_student, code_module, code_presentation)
```

After deduplication, the paper reports **32,593 enrollments** and **28,785 unique students**. The primary event requires `final_result = Withdrawn` and a valid `date_unregistration`; Withdrawn cases without a valid date are treated as censored under the main endpoint definition.

### 2) Temporal representation

Each enrollment is expanded by week, forming a **person-period** table with one row per enrollment-week. This makes it possible to model risk as a temporal process rather than a static end-of-course classification task.

### 3) Main model

The methodological backbone is a **discrete-time hazard** model fitted on weekly rows, using balanced logistic regression with post-hoc calibration. From the predicted weekly hazard, the notebook reconstructs survival through cumulative products.

### 4) Counterfactual policy

The notebook compares two types of simulated regimes:

- **Shock:** directly reduces hazard within the active intervention window.
- **Mechanism-aware:** modifies counterfactual covariates and recomputes risk in a stateful way.

### 5) Evaluation horizons

The protocol separates three horizons:

- `Tpolicy = 18`: main substantive horizon;
- `Teval_policy = 38`: raw trajectory support for policy analysis;
- `Teval_metrics = 37`: stable horizon for IPCW-weighted metrics.

## Expected repository structure

```text
.
├── notebooks/
│   └── main_notebook.ipynb
├── outputs_v2/
│   ├── figures/
│   └── tables/
├── data/
│   └── OULAD files
├── requirements.txt
└── README.md
```

> Adjust folder and file names to match your actual repository structure.

## Main exported artifacts

In addition to the notebook, the project exports artifacts to `outputs_v2` to support the paper’s traceability claims. Files highlighted in the manuscript include:

- `table_policy_spec.csv`
- `table_policy_scenarios_main.csv`
- `table_policy_scenario_params.csv`
- `table_policy_deltaS_by_week_by_scenario.csv`
- `table_policy_deltaS_at_horizons_by_scenario.csv`
- `table_policy_horizons_dual.csv`
- `table_policy_mech_operator_spec.csv`
- `table_rq2_sensitivity_grid.csv`

These artifacts document the policy contract, the scenarios, the evaluation horizons, and the exported outputs of the simulation.

## How to run

### 1) Create the environment

```bash
python -m venv .venv
source .venv/bin/activate  # Linux/macOS
# .venv\Scripts\activate   # Windows
pip install -r requirements.txt
```

### 2) Obtain the data

Download and organize the **OULAD** tables in the data folder defined by the notebook. The paper uses VLE interaction tables, assessments, and administrative records to build the temporal dataset.

### 3) Run the notebook

Open the main notebook in Jupyter:

```bash
jupyter lab
```

Then execute the cells in the original pipeline order, from backbone construction through final artifact export.

## Expected analytical outputs

At the end of execution, the notebook should produce:

- weekly hazard discrimination metrics;
- calibration curves;
- mean survival trajectories;
- `ΔS(t)` curves by policy scenario;
- censoring diagnostics;
- comparison with benchmarks and robustness analyses;
- subgroup analysis with *bootstrap* intervals.

## Interpretive scope

Policy and fairness results should **not** be interpreted as identified causal effects. The paper explicitly defines `ΔS` and `ΔGap` as **simulated structural contrasts**, dependent on the fitted model and the explicit policy contract, not as observational causal estimates.

## Reproducibility

This repository is intended to serve as the executable trail of the paper. The manuscript states explicitly that the code covers:

- preprocessing;
- temporal split;
- training and calibration;
- policy simulation;
- subgroup analysis;
- export of artifacts to `outputs_v2`.

To preserve reproducibility:

- do not change the endpoint logic without updating the paper text;
- preserve the horizons `Tpolicy`, `Teval_policy`, and `Teval_metrics`;
- keep the exported artifact names consistent;
- record any methodological change in both the notebook and the manuscript.

## Citation

If this repository supports your work, please cite the corresponding paper:

```bibtex
@article{daSilva2026temporal_dropout,
  title={A Mathematical Framework for Temporal Modeling and Counterfactual Policy Simulation of Student Dropout},
  author={da Silva, Rafael and Eicher, Jeff and Longo, Gregory},
  journal={Working paper / manuscript},
  year={2026}
}
```

## Note

This README was written for the **main notebook** supporting the paper. If the repository includes multiple supporting notebooks, it is worth adding an extra section describing the role of each one.
