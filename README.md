# Constrained matching for crime research

A worked example of matching law enforcement agencies before fitting an outcome model. One Jupyter notebook covers the full workflow: load the datasets, clean the records, link treatment information to monthly offense counts, select comparison agencies, and check balance.

**All data are synthetic and included.** The example illustrates the matching procedure; it does not reproduce the original study's empirical data or estimated effects.

**[Open the notebook](matching_workflow.ipynb)** to read the explanations, code, results, and figures directly on GitHub. You only need to download the repository if you want to run or change the analysis.

## What the notebook does

1. **Load** four source tables: agency metadata, treatment dates, offense records, and reporting information.
2. **Clean** agency identifiers and offense records, preserving distinct offenses within an incident.
3. **Merge** treatment information into a monthly agency panel, keeping missing months separate from documented zeros.
4. **Match** agencies using geographic and institutional restrictions, population, and 24 months of pretreatment offense levels and trends.
5. **Check** retention, balance, and monthly histories, then save the matched panels and recheck balance after restricting follow-up.

Candidate pairs must share a state and agency type, have different counties and coordinates, lie within 250 miles, and meet population and standardized-distance limits. The notebook explains the crime-research rationale for each rule. County and coordinate screens reduce overlap risk; actual jurisdiction boundaries and reporting arrangements still need review in a real application.

A mixed-integer optimization selects one-to-one pairs separately for each activation year. It first retains as many pairs as possible under the balance constraints, then minimizes total distance at that pair count. A comparison agency can be reused in another year. The implementation uses SciPy's `milp` interface to HiGHS.

## What the example shows

The fixed synthetic inputs produce **1,630 candidate pairs and 79 selected pairs**. The initial matched sample passes all **56 annual balance checks** across raw and transformed characteristics.

Two figures make the diagnostics visible: a before-and-after balance comparison for each activation year, and monthly property-offense histories for the 2019 cohort. The latter shows closer average levels and fitted slopes with remaining differences in the monthly paths.

Requiring six months of follow-up leaves **74 pairs**. Removing late activations weakens balance in the remaining 2022 cohort beyond the original limits, so the notebook flags this before model fitting. The initial matching result does not automatically carry over to a later subset.

These are computational and descriptive results. The synthetic data are not calibrated to empirical NIBRS distributions, and balance on observed histories does not establish parallel trends or a causal effect.

## Run the notebook

Download the repository using **Code → Download ZIP** and extract it. From the project directory, using Python 3.12:

```bash
python -m pip install -r requirements.txt
jupyter lab matching_workflow.ipynb
```

Restart the kernel and run all cells in order. No separate data download is needed; pandas reads the compressed offense file directly. Optimization can take several minutes. Each pass has a configurable time limit, and execution stops if the solver cannot establish an optimal solution.

## Files

| File | Purpose |
| --- | --- |
| `README.md` | Project overview and instructions |
| `matching_workflow.ipynb` | The complete workflow, with saved results and two figures |
| `data/crosswalk.csv` | Agency IDs, names, locations, types, and populations |
| `data/activations.csv` | Agency treatment dates |
| `data/offenses.csv.xz` | Compressed offense records |
| `data/availability.csv` | Reporting and verified-zero flags |
| `requirements.txt` | Python dependencies |
| `.gitignore` | Keeps temporary files and generated results out of Git |

Running the notebook creates five CSVs in `outputs/`: the pair list, two matched panels, and two balance tables. They are recreated on each run.

## Apply the workflow to another study

The notebook's final section describes the prepared agency and monthly-count tables needed for matching. Real NIBRS data require a complete offense-code mapping, reviewed treatment-to-agency links, and documented reporting conventions. The example's simplified labels and name linkage illustrate those steps.

## Research basis

Delos Santos, C. (2026). *Applied Mathematical Modeling of Risk and Mitigation in the Context of Crime, Digital Policing, and Natural Hazards*. Doctoral dissertation, Arizona State University, Chapter 3, pp. 58–71.

The notebook documents the distinction between its two-pass optimization and the study's weighted objective.
