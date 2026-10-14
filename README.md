# IT7009 Artificial Intelligence — Intelligent Traffic Incident Analysis System

Group project submission for **IT7009 Artificial Intelligence** (Academic Year 2026–2027, Semester A, Brief 1).

A machine-learning pipeline that analyses simulated urban road-traffic incident records to

1. **predict the severity** of an incident (4-class classification),
2. **identify high-risk locations and traffic conditions** through unsupervised learning, and
3. **detect abnormal traffic behaviour** using a centroid-distance anomaly score.

---

## Repository contents

| Path | Description |
|---|---|
| `notebooks/IT7009_Traffic_Incident_Analysis.ipynb` | The complete, annotated Google Colab notebook: data-quality audit, cleaning, EDA, feature engineering, leakage/redundancy analysis, feature selection, supervised modelling, unsupervised clustering and evaluation. |
| `report/IT7009_Group_Project_Report.docx` | The project report (cover page, contents, list of figures/tables, Introduction, task sections, Conclusion, APA references). |
| `presentation/IT7009_Group_Project_Presentation.pptx` | 17-slide presentation for the project demonstration / viva. |
| `docs/Viva_Preparation_Guide.md` | Preparation guide for the individual demonstration: model answers, the "hard" questions, and the numbers to memorise. |
| `Project Files and Resources-20261014/Dataset_Brief_1.xlsx` | The dataset: 50,010 incident records × 72 attributes. |
| `Project Files and Resources-20261014/IT7009- Practical Project_Brief1.docx` | The project brief issued by the module team. |
| `Project Files and Resources-20261014/IT7009_Practical_Projects_Evaluation_Rubric.xlsx` | The assessment rubric. |
| `Project Files and Resources-20261014/IT7009_Group_Project_Cover_Page.docx` | The cover-page template. |
| `IT7009_Group_Project_Submission.zip` | The single ZIP to upload to Moodle (contains the `.ipynb` and the `.docx`). |

---

## How to run the notebook

The notebook is designed for **Google Colab**, where every library it uses (`pandas`, `numpy`,
`scikit-learn`, `matplotlib`, `seaborn`) is pre-installed.

1. Open `notebooks/IT7009_Traffic_Incident_Analysis.ipynb` in Google Colab
   (`File → Upload notebook`).
2. Run all cells (`Runtime → Run all`). When the data-loading cell executes it will prompt you to
   upload `Dataset_Brief_1.xlsx` — select the file from
   `Project Files and Resources-20261014/`.
3. All figures and metrics are reproduced deterministically: the notebook fixes a single
   `RANDOM_STATE = 42`.

To run it locally instead, set the environment variables and execute the notebook (or the exported
script):

```bash
export DATA_PATH="Project Files and Resources-20261014/Dataset_Brief_1.xlsx"
jupyter notebook notebooks/IT7009_Traffic_Incident_Analysis.ipynb
```

---

## Methodology summary

**Data preparation.** The raw data contained missing values in every column, 14 exact duplicate
records, inconsistent categorical labels (`collision `, `DRY`, `RAIN`, `car`, `day light`, `Fri`),
numeric columns stored as text (`Temperature`, `Average_Speed`, `Vehicle_Count`, `Road_Width`),
physically impossible measurements (latitude 95, longitude −200, humidity 150, visibility −5), a
malformed datetime and one record with `End_Time` before `Start_Time`. Each issue was identified,
treated and justified in the notebook.

**Leakage.** A leakage audit showed that `Risk_Score` correlates 0.88 with the target with strictly
monotonic class means, and that `Emergency_Response_Time` (correlation 0.37) is measured only *after*
the incident. Both — together with `Response_Team`, `End_Time` and the identifier columns — were
removed. Including them inflates macro-F1 from **0.7206** to **0.9843**, which is the clearest possible
demonstration of why leakage must be detected.

**Supervised learning.** Four classifiers (Logistic Regression, Random Forest, Hist Gradient
Boosting, k-Nearest Neighbours) were trained in a leak-free scikit-learn pipeline on a stratified
80/20 split, tuned with 3-fold stratified cross-validation using macro-F1 as the objective, and
compared with accuracy, macro precision, macro recall, macro F1 and confusion matrices.

**Unsupervised learning.** k-means was tested for k = 2…10. The silhouette curve is uniformly low
(< 0.03) on the full feature set, so the elbow and interpretability determined the final solution
(k = 4). The clusters were profiled, validated against the withheld severity label, and complemented by
a centroid-distance anomaly score (top 1% flagged).

**Feature-subset ablation — assessed on both models**, as the brief requires. Feature selection
improves the supervised model *and* the clustering:

| Feature subset | Columns | Accuracy | Macro F1 | Silhouette (k = 4) |
|---|---|---|---|---|
| All features | 65 | 0.7824 | 0.6412 | 0.0205 |
| Top-20 | 20 | 0.7919 | 0.7006 | 0.0473 |
| **Top-10** | **10** | 0.7887 | **0.7189** | **0.1104** |
| With leakage | 67 | 0.9865 | 0.9843 | 0.0205 |

**Headline results** (held-out test set, leakage-free):

| Model | Accuracy | Macro F1 |
|---|---|---|
| Majority-class baseline | 0.5517 | 0.1778 |
| Logistic Regression *(selected)* | 0.7812 | 0.7332 |
| Hist Gradient Boosting | 0.7864 | 0.7323 |
| Random Forest | 0.7832 | 0.7090 |
| k-Nearest Neighbours | 0.6772 | 0.3832 |

---

## Key findings

* `Traffic_Density`, `Accident_History` and `Vehicle_Count` are the dominant predictors — and the
  same three variables define the unsupervised clusters, so the supervised and unsupervised analyses
  independently agree.
* Feature selection helps **both** models: the 10-column subset gives the best macro-F1 (0.7189) and
  a five-fold better silhouette score than the full feature set.
* Aggregated by category, importance is: traffic 41.9%, environmental 22.4%, spatial/location 16.7%,
  context 12.5% — temporal variables contribute almost nothing.
* Remaining classification errors fall almost entirely between **adjacent** severity levels, which is
  the most tolerable error pattern for operational triage.
* The synthetic dataset carries weak signal (macro-F1 ≈ 0.73 ceiling) and diffuse cluster structure;
  both limitations are reported openly in the report.

---

## Submission checklist

Before uploading, complete the following:

- [ ] **Fill in the cover page** in `report/IT7009_Group_Project_Report.docx` — replace `[Stream ID]`,
      `[Group No.]`, `[DD/MM/YYYY]`, `[Leader Name]`, `[Leader ID]`, `[Lecturer Name]` and the five
      `[Student N Name]` / `[ID]` rows. The brief requires full names and student IDs.
- [ ] Do the same for `[Group Number]` and `[Student names and IDs]` on slide 1 of the presentation.
- [ ] **Open the report in Word and press `Ctrl+A` then `F9`** to refresh the Table of Contents, List of
      Figures and List of Tables page numbers. (Caption numbers are already correct.)
- [ ] Rebuild the ZIP after editing: `zip -j IT7009_Group_Project_Submission.zip notebooks/*.ipynb report/*.docx`
- [ ] Proofread the report with Grammarly (or an equivalent tool), as the brief requires.
- [ ] Check the report with an institution-approved similarity tool, as the brief requires.
- [ ] Only the **group leader** uploads the single ZIP to Moodle. RAR format is not permitted.
- [ ] Prepare for the demonstration using `docs/Viva_Preparation_Guide.md`.

---

## References

Citation style follows **APA (7th edition)**; the full reference list is in the report and at the end
of the notebook.
