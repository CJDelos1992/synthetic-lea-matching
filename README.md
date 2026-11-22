# Constrained matching for crime research

This project demonstrates how to choose comparison law enforcement agencies (LEAs) for a study of reported crime after an agency activates a verified account on Ring's Neighbors platform. The aim is to find agencies with similar characteristics and crime histories that did not activate an account during the observation period. One Jupyter notebook covers the work from separate source files through matching and diagnostics, stopping before an outcome model is fitted.

**All data are synthetic and included.** The example illustrates the matching procedure; it does not reproduce the original study's empirical data or estimated effects.

**[Open the notebook](matching_workflow.ipynb)** to read the explanations, code, results, and figures on GitHub. Download the repository only if you want to run or change the analysis.

## Treatment and comparison agencies

Treatment is **the recorded activation of an LEA's verified Ring Neighbors account**. It is not a measure of household camera ownership or subsequent account use. The synthetic activation table supplies dates; the fictional offense data supply monthly outcomes. We link them using agency information before constructing candidate matches.

Treated agencies activate during 2019–2022. Comparison agencies have **no recorded activation through December 2022**, when the offense panel ends. Some may have a later recorded activation, so “untreated during the study period” is more precise than “never treated.” Each comparison is aligned to its matched treated agency's activation month, without treating that comparison as having activated.

The output includes a pair identifier, agency role, activation month, and `event_month` measured relative to the treated agency's activation. The 24-month preactivation window is `-24` through `-1`; the primary six-month follow-up is `0` through `5`, with `0` denoting the activation month. Both members of a pair share the same relative-time calendar. The notebook also produces `is_treated` and `is_post` flags for later modeling, but it does **not** fit a difference-in-differences model.

## What the notebook does

1. **Load** four source tables: agency metadata, treatment dates, offense records, and reporting information.
2. **Clean** agency identifiers and offense records, preserving distinct offenses within an incident.
3. **Merge** treatment information into a monthly agency panel, keeping missing months separate from documented zeros.
4. **Match** agencies using geographic and institutional restrictions, population, and 24 months of pretreatment offense levels and trends.
5. **Check** retention, balance, and monthly histories, then export the matched panels with treatment-relative time and recheck balance after restricting follow-up.

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
