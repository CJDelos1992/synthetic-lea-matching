# Constrained matching for crime research

This project builds comparison groups for law enforcement agencies using constrained optimization. One Jupyter notebook follows the full workflow: link treatment records to agency IDs, prepare monthly offense counts, select comparison agencies, and assess pretreatment balance.

The example uses entirely **synthetic agency and offense records**. All four input tables are included, so the workflow can be reproduced without an external data download. It ends with matched panels, before fitting an outcome model.

**Start with [matching_workflow.ipynb](matching_workflow.ipynb).** The explanations, code, tables, and two diagnostic figures are all in the notebook. Its saved outputs let you read the example without running it.

## Matching approach

The procedure selects pairs jointly within each activation year:

1. Apply eligibility, geographic, and agency-characteristic restrictions to identify candidate pairs.
2. Retain the largest number of one-to-one pairs that satisfies the balance requirements.
3. Among those assignments, minimize total standardized distance.

The assignment is a mixed-integer linear program solved with [SciPy and HiGHS](https://docs.scipy.org/doc/scipy/reference/generated/scipy.optimize.milp.html). Balance requirements cover population, average offense counts, and fitted pretreatment trends in both raw and transformed forms. The notebook checks the selected sample and reports how later follow-up exclusions affect balance.

## Example results

The example contains 1,630 eligible candidate pairs. The optimizer selects **79 pairs**, and **all 56 annual feature-balance checks pass**. The pretreatment-history figure shows the change in average property-offense levels and fitted slopes for the 2019 cohort, alongside the observed monthly paths.

Requiring complete follow-up leaves 74 pairs and weakens balance in the remaining 2022 group beyond the original limits. The notebook reports that change explicitly, since balance needs to be checked again whenever the selected sample changes.

These results demonstrate the procedure on fixed synthetic inputs. They do not establish performance on real crime data or verify the parallel-trends assumption required for a later difference-in-differences analysis. The notebook explains this distinction and links to the methodological references.

## Run the notebook

Download the repository using **Code → Download ZIP**, then extract it. The four synthetic input tables are included; no external data download is needed. The compressed offense table can stay compressed because pandas reads it directly.

Using Python 3.12, open a terminal in the extracted project folder and run:

```bash
python -m pip install -r requirements.txt
jupyter lab matching_workflow.ipynb
```

You can also open the notebook in your usual Jupyter environment. Select the environment where you installed these requirements, restart the kernel, and run all cells in order. Matching takes several minutes. If a solver pass reaches its time limit before proving the optimum, the notebook stops and identifies that pass. Increase `SOLVER_TIME_LIMIT` to allow more time without relaxing the matching rules.

## Repository contents

| File | Purpose |
|---|---|
| `matching_workflow.ipynb` | The preparation, matching, and diagnostic workflow |
| `README.md` | These instructions |
| `requirements.txt` | The Python packages to install |
| `.gitignore` | Keeps temporary files and generated results out of Git |
| `data/crosswalk.csv` | Fictional agency IDs, names, locations, and characteristics |
| `data/activations.csv` | Fictional treatment records |
| `data/offenses.csv.gz` | Fictional offense records, compressed to save space |
| `data/availability.csv` | Reporting information used to distinguish missing months from zeros |

The input files are fixed examples. Running the notebook creates an `outputs/` folder containing the selected pairs, two matched panels, and two balance tables. Rerunning it replaces those generated files. The figures are saved inside the notebook.

## Use your own data

The final section of the notebook describes the two prepared tables needed for matching and where to substitute them. Real National Incident-Based Reporting System (NIBRS) files still need their own import, offense-code definitions, and reviewed treatment-to-agency links. The example uses three simplified offense labels and an exact normalized-name link, so those preparation steps need to be adapted to the real sources. The synthetic agency histories include agency-specific and shared state variation; their parameters have not been calibrated to NIBRS.

All preparation, matching, and diagnostic code is included in the notebook. Its built-in checks cover the merge, agency reuse, solver status, and the selected sample's balance. The follow-up table checks balance again after exclusions.

## Reference

The matching rules follow Chapter 3 of:

Delos Santos, C. (2026). *Applied Mathematical Modeling of Risk and Mitigation in the Context of Crime, Digital Policing, and Natural Hazards*. Doctoral dissertation, Arizona State University, pp. 58–71.
