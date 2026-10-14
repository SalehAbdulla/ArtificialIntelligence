# IT7009 Group Project — Status, Grading & Action Items

> **Snapshot taken at the end of the final audit.** Come back to this file before you submit.
> Last updated: commit `437ffdd` on `origin/main`.

---

## 1. Bottom line

| | Status |
|---|---|
| Group submission (70 marks) | **Complete** — estimated **~69/70**, i.e. **70/70 once the cover page is filled in** |
| Individual demonstration (30 marks) | **Prepared** — material is in `docs/Viva_Preparation_Guide.md`, but the performance is yours |
| Outstanding work | **4 actions** — see [Section 4](#4--action-required-do-these-before-submitting) |

**Nothing in the technical work is outstanding.** Every rubric criterion is now backed by evidence in the
notebook and the report. The only remaining risks are administrative (cover-page details, Grammarly,
similarity check) and the oral demonstration.

---

## 2. Deliverables

| Path | What it is | Status |
|---|---|---|
| `notebooks/IT7009_Traffic_Incident_Analysis.ipynb` | 95 cells (37 markdown + 58 code) — the full Colab pipeline | ✅ validated, runs end-to-end |
| `report/IT7009_Group_Project_Report.docx` | 12 sections, 21 captioned tables, 19 figures, APA references | ⚠️ needs cover details |
| `presentation/IT7009_Group_Project_Presentation.pptx` | 17-slide demonstration deck | ⚠️ needs names on slide 1 |
| `docs/Viva_Preparation_Guide.md` | Model answers, hard questions, numbers to memorise | ✅ ready |
| `IT7009_Group_Project_Submission.zip` | The Moodle upload (`.ipynb` + `.docx`) | ⚠️ rebuild after editing the cover page |
| `README.md` | Repository guide + submission checklist | ✅ ready |

---

## 3. Grading against the rubric (group submission, 70 marks)

| # | Criterion | Weight | Estimated | Evidence in the submission |
|---|---|---|---|---|
| 1 | Problem definition, objectives and context | 5 | **5/5** | Report §1.1–1.3; precise problem, three objectives, motivation, WHO (2023) citation, domain relevance |
| 2 | Data cleaning and preprocessing | 12 | **5/5** | Notebook §3–4; all ten named issue types found, treated and justified; reproducible `clean_incidents()` function; imbalance + leakage handled |
| 3 | Data exploration and visualisation | 8 | **5/5** | Report §3; 19 figures, all labelled, referenced and interpreted |
| 4 | Feature analysis and selection | 8 | **5/5** | Report §4; three importance methods, redundancy check, prediction-time availability, leakage audit, **ablation on both supervised and unsupervised models** |
| 5 | Supervised modelling | 10 | **5/5** | Report §5; four classifiers, stratified split, leak-free `Pipeline`, `GridSearchCV` tuning |
| 6 | Supervised evaluation and comparison | 8 | **5/5** | Report §5.5–5.10; accuracy, precision, recall, macro-F1, confusion matrices, class-level analysis, comparison table + plot, justified best model |
| 7 | Unsupervised learning and cluster validation | 8 | **5/5** | Report §6; k = 2–10, elbow + silhouette, final clusters, profiles, external validation against the withheld label, limitations and practical meaning |
| 8 | Critical evaluation and reflection | 5 | **5/5** | Report §7; supervised/unsupervised comparison, effectiveness, six limitations, practical implications, seven improvements |
| 9 | Code, report and submission requirements | 6 | **4/5 → 5/5** | Everything present **except the cover-page details are placeholders** — filling them in completes this criterion |
| | **Group submission total** | **70** | **~69/70 → 70/70** | |

### Individual demonstration (30 marks) — preparation provided

| Criterion | Weight | Where the preparation is |
|---|---|---|
| Problem, data and preprocessing understanding | 6 | Guide §2 |
| Supervised modelling and results | 7 | Guide §4 |
| Unsupervised learning and cluster interpretation | 5 | Guide §5 |
| Technical justification and individual understanding | 8 | Guide §3, §6, §7 (hard questions) |
| Communication and response to questions | 4 | Guide §1 (60-second summary), §8 (numbers), §9 (checklist) |

> **Be honest about this:** the viva is assessed on *your* individual performance. The guide gives you the
> material, but the rubric explicitly requires that every member can justify the complete workflow
> "regardless of the role they held". Rehearse §1 and §7 of the guide out loud.

---

## 4. ⚠️ ACTION REQUIRED — do these before submitting

### ☐ Action 1 — Fill in the cover page (the only thing costing a mark right now)

Open `report/IT7009_Group_Project_Report.docx` and replace every placeholder:

| Placeholder | Replace with |
|---|---|
| `[Stream ID]` | your stream ID |
| `[Group No.]` | your group number |
| `[DD/MM/YYYY]` | submission date |
| `[Leader Name]` / `[Leader ID]` | group leader's name and student ID |
| `[Lecturer Name]` | lecturer's name |
| `[Student 1 Name]` … `[Student 5 Name]` | all group members' full names |
| `[ID]` (×5) | all group members' student IDs |

The brief requires *"a cover page within the report with the full name and student ID of all group
members"*, and the rubric's required evidence lists *"cover details"*. Also update `[Group Number]` and
`[Student names and IDs]` on **slide 1** of `presentation/IT7009_Group_Project_Presentation.pptx`.

### ☐ Action 2 — Refresh the report's field codes

Open the report in Word and press **`Ctrl + A`**, then **`F9`** (choose "Update entire table" if asked).
This populates the page numbers in the Table of Contents, List of Figures and List of Tables.
*Caption numbers (Figure 1–19, Table 1–21) are already correct even if you skip this.*

### ☐ Action 3 — Rebuild the ZIP after editing

```bash
cd /Users/salehabdulla/Desktop/coding/ArtificialIntelligence
zip -j IT7009_Group_Project_Submission.zip notebooks/IT7009_Traffic_Incident_Analysis.ipynb \
                                              report/IT7009_Group_Project_Report.docx
unzip -l IT7009_Group_Project_Submission.zip     # should list exactly 2 files
```

⚠️ **RAR format is not allowed** — the brief awards **zero marks** for a RAR submission.

### ☐ Action 4 — Run the two mandatory checks from the brief

- [ ] **Grammarly** (free account) or an equivalent grammar-checking tool on the report.
- [ ] An **institution-approved similarity/plagiarism tool** on the report; correct anything flagged.
      (Everything in the report is original writing produced for this project, but the brief requires the
      check to be performed.)

### ☐ Action 5 — Prepare for the demonstration

- [ ] Read `docs/Viva_Preparation_Guide.md`.
- [ ] Every member can deliver the 60-second summary (§1) without notes.
- [ ] Every member can name four data-quality problems and their treatments.
- [ ] Every member can explain the leakage decision and quote the **0.98 vs 0.72** contrast.
- [ ] Open the notebook **before** the session and run it once so the outputs are cached.

### ☐ Action 6 — Only the group leader uploads

One ZIP, submitted by the group leader, to the IT7009 Moodle page. Individual submissions are not
required.

---

## 5. What the final audit found and fixed

An earlier version of this project was **~85% complete**. A line-by-line audit against the brief
identified three real gaps, all now closed:

| # | Gap found | Brief reference | Fix applied |
|---|---|---|---|
| 1 | The feature-subset ablation tested only the **supervised** model | §30: *"Assess the impact of different feature subsets on **both supervised and unsupervised** learning models"* | Added a k-means re-run per subset with silhouette scores; new panel (b) in Figure 10 and a silhouette column in Table 10 |
| 2 | The supervised and unsupervised task sections did not contain the four required stages | §73: *"For Supervised and unsupervised tasks, include **Data Preparation, Feature Development, Model Design, Evaluation and Validation**"* | Restructured report §5 → 5.1–5.4 and §6 → 6.1–6.4 using those exact names |
| 3 | Feature importance was per-feature only | §30: *"evaluating the importance of **environmental, temporal, spatial, and road-related** variables"* | Added category-level aggregation: Table 9, Figure 9(d), and a dedicated discussion |
| 4 | No presentation existed | *"each group must prepare a PPT presentation"* (viva = 30% of the mark) | Added a 17-slide deck plus a viva preparation guide |

**The fix in gap 1 produced a genuinely stronger result** — feature selection helps the *unsupervised*
model even more than the supervised one (see Section 6).

---

## 6. Key numbers (verified from the actual run)

### Dataset and cleaning

| Item | Value |
|---|---|
| Raw dataset | 50,010 rows × 72 columns |
| After cleaning | **49,992** rows |
| Duplicate rows removed | 14 |
| Rows with missing target removed | 7 |
| Class balance | Sev 1 = 1.57% · Sev 2 = 6.96% · Sev 3 = 36.31% · Sev 4 = 55.17% (35:1) |

### Supervised results (held-out test set, leakage-free)

| Model | Accuracy | Macro F1 |
|---|---|---|
| Majority-class baseline | 0.5517 | 0.1778 |
| **Logistic Regression (selected)** | **0.7812** | **0.7332** |
| Hist Gradient Boosting | 0.7864 | 0.7323 |
| Random Forest | 0.7832 | 0.7090 |
| k-Nearest Neighbours | 0.6772 | 0.3832 |

### Leakage — the headline finding

| Variable | Correlation with Severity | Decision |
|---|---|---|
| `Risk_Score` | **0.88** (monotonic class means) | Removed |
| `Emergency_Response_Time` | 0.37 (post-incident) | Removed |
| **Effect on macro-F1** | **0.9843 with leakage → 0.7206 without** | — |

### Feature-subset ablation (assessed on **both** models)

| Feature subset | Columns | Accuracy | Macro F1 | Silhouette (k = 4) |
|---|---|---|---|---|
| All features | 65 | 0.7824 | 0.6412 | 0.0205 |
| Top-20 | 20 | 0.7919 | 0.7006 | 0.0473 |
| **Top-10** | **10** | 0.7887 | **0.7189** | **0.1104** |
| With leakage | 67 | 0.9865 | 0.9843 | 0.0205 |

### Importance by attribute category

| Category | Share of total importance |
|---|---|
| Traffic | **41.9%** |
| Environmental | 22.4% |
| Spatial / location | 16.7% |
| Context (road works, holidays) | 12.5% |
| Road-related | 2.3% |
| Infrastructure | 2.0% |
| Temporal | 1.2% |

### Unsupervised and other

| Item | Value |
|---|---|
| Chosen k | **4** (elbow); max silhouette at k = 2 is 0.0275 |
| Anomalies flagged | 500 (top 1% centroid distance) |
| Imbalance effect on Sev-1 recall | 0.7051 (none) → **0.9231** (balanced) |
| Severity-1 recall / Severity-4 precision | 0.936 / 0.897 |
| Best model's weakest class | Severity 2 (F1 ≈ 0.574) |

---

## 7. Automated quality checks (all passing)

These were run against the final artifacts, not assumed:

| Check | Result |
|---|---|
| Notebook schema | `nbformat.validate()` → **VALID**, 95 cells |
| Notebook execution | Runs end-to-end; produces all 19 figures and `results.json` |
| Figure captions | 19 captions, numbered 1–19, **no gaps** |
| Table captions | 21 captions, numbered 1–21, **no gaps** |
| Figures referenced in text | **19/19** — no caption unreferenced, no dangling reference |
| Tables referenced in text | **21/21** — no caption unreferenced, no dangling reference |
| Report formatting | Times New Roman — 14 pt bold headings, 12 pt bold sub-headings, 11 pt body, 1.5 line spacing |
| Page numbering | `PAGE` field in the footer (bottom centre) |
| TOC / LoF / LoT | Real Word fields; `updateFields` set so they populate on open |
| Images embedded | 19 |
| ZIP contents | Exactly 2 files: `.ipynb` + `.docx` |

> **Note on the numbering registry:** the report builder uses a self-validating caption registry. If a
> caption is ever added, removed or reordered without updating the registry, the build **fails loudly**
> instead of silently producing wrong cross-references. This is why the checks above can be trusted.

---

## 8. Repository state and how to regenerate

### Git history (one commit per file, newest first)

```
88b2c1e chore: rebuild the Moodle submission ZIP from the updated notebook and report
633442f docs: update README with new artifacts, ablation results and submission checklist
42b2627 feat(presentation): add 17-slide demonstration deck for the project viva
2145749 docs: add viva preparation guide with model answers and key figures
8e4a4a7 docs(report): add required task stages, grouped feature importance and unsupervised ablation
906c19e feat(notebook): add unsupervised feature-subset ablation and category-level importance analysis
f07cdec chore: add the Moodle submission ZIP (notebook + report)
4a52d29 docs: add the group project report (docx) with figures, tables and APA references
6de1cd3 feat: add the traffic incident analysis notebook (full Colab pipeline)
dbd19cb docs: add the practical projects evaluation rubric (root copy)
dc4aa7f docs: add the IT7009 practical project brief (Brief 1)
34aa052 docs: add the practical projects evaluation rubric (resources copy)
eb47e7d docs: add the group project cover page template
c9ec28e data: add the traffic incident dataset (Dataset_Brief_1.xlsx)
0703f4f docs: add project README with repository guide and methodology summary
035fde5 chore: add .gitignore for macOS, Python and notebook artefacts
```

Remote: **https://github.com/SalehAbdulla/ArtificialIntelligence** (branch `main`)

### To reproduce every number in the report

1. Open `notebooks/IT7009_Traffic_Incident_Analysis.ipynb` in Google Colab.
2. `Runtime → Run all`. Upload `Project Files and Resources-20261014/Dataset_Brief_1.xlsx` when prompted.
3. The notebook fixes `RANDOM_STATE = 42`, so every figure and metric is reproduced exactly.

### Honest caveats to remember (and to defend in the viva)

1. The dataset is **synthetic** and its relationships are **weak** — macro-F1 is capped at ≈0.73. This is
   a property of the data, not of the pipeline.
2. The clustering structure is **diffuse** on the full feature set (all silhouette scores < 0.03). It
   improves five-fold on a reduced subset, which is reported.
3. Validation is a **random** split, not chronological — drift is not yet measured.
4. Severity is an **ordinal** label modelled as nominal.
5. The **hotspot ranking** is limited by the even spatial distribution of the synthetic data.

Do not hide these — the rubric explicitly rewards "critical analysis of the results, acknowledge any
limitations". Being able to explain *why* the ceiling exists, with the leakage ablation as evidence, is
a strength rather than a weakness.

---

## 9. Quick reference — where to find things

| I need… | Look at… |
|---|---|
| The problem statement and objectives | Report §1.1–1.3 |
| Every data-quality issue and its treatment | Report Table 2 and Table 4; notebook §3–4 |
| The leakage argument | Report §4.2 and Table 7; notebook §7 |
| Feature importance and subset analysis | Report §4.4–4.5, Table 8–10, Figure 9–10 |
| Model design and results | Report §5, Tables 11–15, Figures 11–13 |
| Clustering method, k selection and profiles | Report §6, Tables 16–19, Figures 14–17 |
| Limitations and future work | Report §7.3 and §7.5 |
| Viva answers | `docs/Viva_Preparation_Guide.md` |
| Submission steps | Section 4 of this file, and `README.md` |
