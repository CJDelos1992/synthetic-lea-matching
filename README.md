# Constrained matching for crime research

This repository demonstrates how law enforcement agencies can be matched using pretreatment offense histories and **mixed-integer optimization**. One Jupyter notebook covers the full pre-model workflow: link treatment records to agency identifiers, build monthly offense counts, construct eligible pairs, solve the matching problem, and assess balance.

**All data are synthetic.** The four input tables are included, and the demonstration does not reproduce the original study's empirical results.

**[Open the notebook](matching_workflow.ipynb)** to read the explanations, Python code, results, and diagnostic figures directly on GitHub. No installation is needed just to read it.

## How the matching works

1. **Prepare and link data.** Reconcile agency identifiers and aggregate fictional NIBRS-style offense records into a monthly panel, preserving the distinction between missing records and observed zeros.
2. **Build comparable candidates.** Apply geographic, agency-type, population, reporting-history, and standardized-distance restrictions. Both agencies in a potential pair use the same 24 calendar months before the treated agency's activation.
3. **Solve the assignment problem.** A mixed-integer linear program first maximizes retained one-to-one matches subject to group-balance constraints, then minimizes total standardized distance among equally large solutions. Comparisons can be reused in different activation years.
4. **Check the result.** Examine retention, raw and transformed balance, selected pair eligibility, and pretreatment histories; then recheck balance after excluding pairs with incomplete follow-up.

The algorithm is implemented in the notebook using SciPy's `milp` interface and the open-source HiGHS solver. Matching features include agency population, average monthly counts in three offense categories, and their fitted pretreatment trends, in raw and transformed forms.

## Illustrative results

In the fixed synthetic example, 1,630 candidate pairs pass eligibility restrictions, and **79 pairs** are selected. All **56 annual raw/transformed balance checks** pass for the initial matched sample. In the displayed 2019 example, selected groups become closer in average property-offense levels and fitted linear slopes; their monthly observations remain noisy.

Restricting the selected sample to pairs with complete six-month follow-up leaves **74 pairs** and weakens balance in the remaining 2022 cohort beyond the original limits. This consequence of filtering after matching is shown explicitly.

These are **computational and descriptive results**, not estimates of a causal treatment effect. Balance on observed histories does not establish the parallel-trends assumption required for a subsequent difference-in-differences analysis. The synthetic inputs are illustrative, not calibrated to empirical NIBRS distributions.

## Run locally

Download the repository using GitHub's **Code → Download ZIP** option, then extract it. From the project directory, using Python 3.12:

```bash
python -m pip install -r requirements.txt
jupyter lab matching_workflow.ipynb
```

Restart the notebook kernel and run all cells in order. The data are already included. Each optimization pass has a configurable time limit; if the solver cannot prove optimality within it, the notebook reports that rather than presenting an incumbent as optimal.

## Repository contents

| Path | Purpose |
| --- | --- |
| `matching_workflow.ipynb` | Data preparation, matching, validation, and figures |
| `data/crosswalk.csv` | Fictional agency identifiers and metadata |
| `data/activations.csv` | Fictional agency treatment dates |
| `data/offenses.csv.xz` | Fictional offense records, compressed |
| `data/availability.csv` | Reporting flags for differentiating missing months and zeros |
| `requirements.txt` | Python dependencies |
| `.gitignore` | Excludes temporary and generated files |

Running the notebook produces selected-pair and matched-panel CSVs in `outputs/`, which is excluded from version control. Saved notebook outputs let readers inspect the main tables and figures without running the code.

## Adapting the workflow

The notebook's final section documents the prepared `metadata` and monthly `panel` tables needed to apply matching to another study. Actual NIBRS files require their own import, complete offense-code classification, and reviewed treatment-to-ORI linkage. The example's simplified labels and exact normalized-name linkage are demonstrations, not replacements for those steps.

## Research basis

The matching design is based on Chapter 3 of:

Delos Santos, C. (2026). *Applied Mathematical Modeling of Risk and Mitigation in the Context of Crime, Digital Policing, and Natural Hazards.* PhD dissertation, Arizona State University, pp. 58–71.

This repository contains a newly written demonstration with invented data, not the original empirical dataset or original analysis code.
