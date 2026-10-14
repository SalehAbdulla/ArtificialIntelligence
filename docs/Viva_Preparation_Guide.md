# IT7009 Group Project — Viva Preparation Guide

**Individual demonstration = 30% of the project mark.** The rubric assesses each student on five
criteria, and explicitly requires that you can justify **the complete workflow regardless of the role
you held**. This guide gives you the numbers to memorise and model answers to the questions most likely
to be asked.

> **Answer structure that scores well:** state (1) what you did, (2) why you did it that way,
> (3) what the alternative was, and (4) what the consequence was. This alone covers
> "Technical justification and individual understanding" (8 marks).

---

## 1. The 60-second summary — every member must be able to say this

> "We built a machine-learning pipeline for an intelligent traffic incident analysis system on a
> synthetic dataset of 50,010 incidents described by 72 attributes. Our goal was threefold: predict
> incident severity on a 1–4 scale, find high-risk locations and traffic environments without using
> labels, and flag abnormal traffic behaviour.
>
> We cleaned the data first — removing 14 duplicates, fixing mis-typed numeric columns, normalising
> dirty categorical labels, and handling physically impossible values. The most important decision was
> a leakage audit: `Risk_Score` correlates 0.88 with the target and `Emergency_Response_Time` is
> recorded after the incident, so both had to go. Including them inflates macro-F1 from about 0.72 to
> 0.98, which is a fake result.
>
> We implemented four classifiers on a stratified pipeline and tuned them on macro-F1 because the
> classes are heavily imbalanced. Logistic Regression won at 0.733 macro-F1 against a 0.178
> majority-class baseline. For the unsupervised part we used k-means with the elbow method and
> silhouette score, and the clusters it found separate on exactly the same traffic-intensity variables
> the supervised model relied on — independent agreement that the signal is real. We also added a
> centroid-distance anomaly score."

---

## 2. Problem, data and preprocessing (6 marks)

**Q: Why is this problem worth solving?**
Urban traffic volume has outgrown manual monitoring. Incidents detected late or misjudged delay
emergency response, prolong congestion and increase secondary-collision risk. A system that predicts
severity supports triage, identifies high-risk locations for prevention, and flags unusual events for
human attention. Road traffic injuries remain a leading cause of death globally (WHO, 2023).

**Q: Describe the dataset.**
50,010 rows × 72 columns, one row per incident. Attribute groups: temporal, incident type, location,
road, environment, traffic, infrastructure, proximity, context, and outcome-like variables. The target
is `Severity`, an ordinal 4-level label. The data is synthetic and was generated for education — that
matters because it caps how much signal exists.

**Q: What was the class distribution and why does it matter?**
Severity 1 = 1.57%, Severity 2 = 6.96%, Severity 3 = 36.31%, Severity 4 = 55.17% — a 35:1 ratio. A
model that always predicts Severity 4 gets **55.17% accuracy but only 0.178 macro-F1**. That is why we
used macro-F1 as the primary metric and never relied on accuracy alone.

**Q: List the data-quality problems you found.**
- **Missing values** — 5–7 per column (~0.01% of all cells), including 7 in the target itself.
- **14 exact duplicate rows** (and duplicate `Incident_ID`s).
- **Inconsistent categorical labels** — `collision `, `DRY`, `RAIN`, `car`, `Truck `, `day light`,
  `police`, `monday`, `Fri` (casing and trailing whitespace).
- **Wrong data types** — `Temperature`, `Average_Speed`, `Vehicle_Count`, `Emergency_Response_Time`,
  `Road_Width`, `Risk_Score` stored as text.
- **Impossible values** — latitude 95, longitude −200, humidity 150 and −10, visibility −5,
  precipitation −3.
- **Temporal problems** — 8 unparsable datetimes; 1 record with `End_Time` earlier than `Start_Time`.
- **Outliers** — 336 `Dew_Point`, 176 `Precipitation` by the 1.5 × IQR rule.
- **Near-unique identifiers** — `Postal_Code` (38,347 unique), `Road_Name` (39,388 unique).

**Q: How did you handle each one, and why?**

| Problem | Treatment | Why that choice |
|---|---|---|
| Duplicates | dropped | the same incident counted twice would bias training |
| Missing target | dropped the rows | a label cannot be imputed — that would fabricate ground truth |
| Missing features (~0.01%) | median (numeric) / mode (categorical) | negligible missingness; the median is robust to the outliers present |
| Text-stored numbers | `pd.to_numeric()` | correct types are required for scaling and correlation |
| Dirty labels | strip + title-case + map abbreviations | so one real category maps to one encoded level |
| Impossible values | set to NaN via domain rules, then imputed | impossible measurements distort scaling and distances |
| `End < Start` | dropped the record | the duration is undefined; the record is internally inconsistent |
| Outliers | **kept** | plausible meteorological readings, not errors; trees are robust to them |
| Identifiers | dropped | no generalisable signal; high cardinality causes over-fitting |
| Imbalance | stratified split + `class_weight="balanced"` | keeps rare classes represented; stops the majority dominating the loss |

**Q: Why impute instead of dropping rows?**
Missingness is only about 0.01% per column and it is scattered, so dropping would discard usable rows
for no benefit. The target is the exception: you cannot impute a label.

**Q: Why did you keep the outliers?**
IQR flags *unusual* values, not *wrong* ones. A dew point of −11 °C is plausible. Deleting them would
remove genuine rare conditions — and rare conditions are exactly what the anomaly detector is meant to
surface. Tree ensembles are also robust to monotone outliers. Where values were physically impossible
(humidity 150%), they were removed.

**Q: Why put the preprocessing inside a scikit-learn `Pipeline`?**
So the imputer and scaler are fitted on the **training fold only**. If they are fitted on the whole
dataset before splitting, statistics from the test set leak into training and the reported score is
optimistic. It also makes the workflow reproducible as a single object.

**Q: What is data leakage and what did you find?**
Leakage is when a feature carries information about the target that would not exist at prediction time.
`Risk_Score` correlates 0.88 with severity with strictly monotonic class means (37.8 → 72.5 → 107.8 →
147.3) — it is derived from the outcome. `Emergency_Response_Time` (0.37) is recorded *after* the
incident. `Response_Team` is dispatched *because of* severity. `End_Time` and the derived duration are
post-event. All were removed.

**Q: How do you know the leakage mattered?**
We measured it. In the ablation study the same random forest scored **0.984 macro-F1 with the leaking
variables** and **0.721 without them**. That gap is the whole argument.

---

## 3. Feature analysis and selection (8 marks)

**Q: How did you select features?**
Three independent methods, so the conclusion does not depend on one technique: **mutual information**
(model-free, catches non-linearity), **random-forest Gini importance**, and **permutation importance**
(the drop in macro-F1 on held-out data when a feature is shuffled — the most trustworthy, because it
uses the real metric on unseen data). All three agreed on the ranking.

**Q: What are the most influential features?**
`Traffic_Density`, `Accident_History` and `Vehicle_Count`, followed by `Road_Work` and `Holiday`.
Aggregated by category, **traffic** variables dominate, then **spatial/location** and **road-related**,
with **environmental** variables contributing a smaller but non-zero share (consistent with adverse
weather raising severity). Proximity and infrastructure indicators contribute almost nothing once
traffic intensity is in the model.

**Q: What did you do about redundancy?**
Checked every pair with |r| > 0.8. Three stood out: `Dew_Point` ↔ `Temperature` (0.974),
`Average_Speed` ↔ `Speed_Limit` (0.890), and `Risk_Score` ↔ `Severity` (0.882, already removed as
leakage). `Dew_Point` was dropped, and `Average_Speed` was replaced by the engineered `Speed_Ratio`,
which is normalised and more interpretable.

**Q: What was the effect of feature selection?**
Two findings. First, the **Top-20 subset matches or beats the full feature set** on macro-F1, so the
same performance is available with far fewer inputs to collect and maintain. Second — and this is what
the brief specifically asks for — feature selection also **helps the unsupervised model**: the
silhouette score improves substantially once the noisy variables are removed. That shows the weak
clustering of the full model is caused by uninformative features rather than by an absence of
structure.

---

## 4. Supervised modelling and results (7 marks)

**Q: Which models did you use and why those four?**
Four, one more than the minimum of three, chosen to represent genuinely different learning strategies:
1. **Logistic Regression** — linear, interpretable baseline; each coefficient shows the direction of a
   feature's effect; handles imbalance well with balanced class weights.
2. **Random Forest** — bagged decorrelated trees; captures non-linearity and interactions, robust to
   outliers and irrelevant features, and gives Gini importance for free.
3. **Hist Gradient Boosting** — boosted trees; usually the strongest performer on tabular data and fast
   enough for 50,000 rows thanks to histogram binning.
4. **k-Nearest Neighbours** — instance-based; makes no assumption about the decision boundary, so it
   tests whether the classes are locally separable. It turned out they are not, which is a useful
   diagnostic in itself.

**Q: Why a stratified split?**
With a 35:1 imbalance, an unstratified split could leave Severity 1 under-represented or even absent
from one partition, making the measured performance meaningless. Stratification keeps the class
proportions identical in train and test.

**Q: Why macro-F1 and not accuracy?**
Accuracy is dominated by the majority class. Our majority-class baseline scores 0.5517 accuracy but
only 0.1778 macro-F1. Macro-F1 weights every class equally, so it cannot be inflated by the majority
class — which is exactly what we need when the rare classes are the operationally important ones.

**Q: How did you tune the models?**
`GridSearchCV` with **3-fold stratified cross-validation**, optimising **macro-F1** rather than
accuracy. Optimising macro-F1 forces the search to care about the rare classes. Selected settings:
Logistic Regression `C = 0.1` (strong regularisation — the signal is weak, so a simpler model
generalises better); Random Forest `n_estimators = 300`, `min_samples_leaf = 5`; Gradient Boosting
`learning_rate = 0.1`, `max_iter = 200`, `max_leaf_nodes = 31`.

**Q: What were the results?**

| Model | Accuracy | Macro F1 |
|---|---|---|
| Majority-class baseline | 0.5517 | 0.1778 |
| **Logistic Regression (selected)** | **0.7812** | **0.7332** |
| Hist Gradient Boosting | 0.7864 | 0.7323 |
| Random Forest | 0.7832 | 0.7090 |
| k-Nearest Neighbours | 0.6772 | 0.3832 |

**Q: Which model did you select and why?**
Logistic Regression, on four grounds: highest macro-F1; highest macro-recall (so it detects the rare
classes more consistently); most interpretable (signed coefficients an operator can read); and
cheapest to train, store and serve. Gradient boosting is within 0.001 macro-F1, so the choice is not
sensitive — and when performance is equivalent you prefer the simpler, more explainable model.

**Q: How did class imbalance affect the outcome?**
Directly measurable. Untreated, the model gets 0.8055 accuracy but Severity-1 recall of only 0.7051 —
it simply avoids predicting the rare class. With `class_weight="balanced"`, Severity-1 recall rises to
**0.9231** while macro-F1 stays essentially unchanged and accuracy falls by under three points. Since
missing a critical incident costs far more than a slightly over-cautious prediction, balanced
weighting was adopted.

**Q: Where does the model make mistakes?**
Almost entirely between **adjacent** severity levels. Severity 2 is the hardest class (lowest F1,
about 0.57) because it sits between Severity 1 and 3. Crucially, the model rarely confuses a minor
incident with a critical one — the errors are *ordinal*, not random, which is far more tolerable in
practice. Severity 1 recall is 0.936 and Severity 4 precision is 0.897: the two decisions that matter
most are the two the model handles best.

**Q: What is a confusion matrix and what does yours show?**
A table of predicted versus actual classes. The diagonal is strong, and the off-diagonal mass sits
immediately beside the diagonal. The normalised version shows that when the model errs it errs by one
severity level.

---

## 5. Unsupervised learning and cluster interpretation (5 marks)

**Q: What did you do and why k-means?**
k-means on the standardised numeric features, with the severity label deliberately withheld. It was
chosen because it is fast enough to test many values of k on 50,000 rows, its centroids are directly
interpretable as typical traffic conditions, and the distance to a centroid gives a natural anomaly
score. We cross-checked with hierarchical (Ward) clustering.

**Q: Why only numeric features?**
Because k-means and the silhouette score are Euclidean. One-hot encoding a categorical variable would
create arbitrary distances between categories and distort the geometry. The categorical variables are
used *afterwards* to profile and interpret the clusters instead.

**Q: How did you choose k?**
Tested k = 2 to 10. Recorded inertia (elbow method) and the silhouette score, computed on a 10,000-row
sample because silhouette is O(n²).

**Q: So what did you choose, and was that straightforward?**
No — and this is the honest part. The silhouette score is maximised at **k = 2 (0.0275)**, but *every*
silhouette value is below 0.03, which means the clusters overlap heavily and there is no strongly
separated natural structure. Because the curve is essentially flat, silhouette alone cannot
discriminate. The elbow bends clearly at **k = 4**, after which the inertia curve flattens. We adopted
**k = 4** on the elbow plus interpretability grounds — a two-cluster solution would only split the data
into a large and a small group with little operational meaning — and we report the low silhouette
openly as a limitation.

**Q: Is there anything that improves the clustering?**
Yes, and it is a good finding. In the ablation study we re-ran k-means on reduced feature subsets. The
silhouette rises from **0.0205** with all 53 numeric features to **0.1104** with only the 9
highest-ranked ones — a more than five-fold improvement. So the diffuse structure is partly caused by
uninformative variables diluting the Euclidean distance, not by a genuine absence of structure. That is
why we recommend re-running the cluster-count selection on the reduced subset.

**Q: Describe the clusters.**
Four groups of roughly 8k–17k incidents each. They differ most sharply on `Traffic_Density`,
`Vehicle_Count` and `Accident_History` — the same three variables the supervised importance analysis
ranked highest. One cluster stands out: it has the highest traffic density and accident history and
concentrates about **68% Severity-4** incidents, against roughly 52% in the other three. That is the
high-risk traffic environment.

**Q: How do you know the clusters are meaningful if you never used the label?**
Because we can test it. Cross-tabulating the withheld severity label against the clusters shows their
severity composition differs systematically. That is genuine external validation. And the fact that
the clusters separate on exactly the variables the supervised model found important — two completely
independent analyses agreeing — is much stronger evidence than either alone.

**Q: How did you detect abnormal behaviour?**
Distance from each incident to its assigned cluster centroid. Incidents in the top 1% of that distance
(500 records) are flagged as anomalous: their combination of conditions is far from every typical
pattern. This is complementary to the classifier, which can only assign one of four known labels and is
therefore blind to something unlike anything in the training data.

**Q: What about the hotspots?**
We ranked cities by the number of Severity 3–4 incidents. We flag honestly that the differences are
small and the ranking largely tracks how many incidents each city recorded, because the synthetic
generator distributes incidents evenly. On real data you would normalise by traffic volume to separate
genuine hotspots from merely busy locations.

---

## 6. Critical evaluation and reflection (5 marks)

**Q: Compare the supervised and unsupervised outcomes.**
They answer different questions and they agree. Supervised learning quantified how predictable severity
is from conditions available at report time and ranked `Traffic_Density`, `Accident_History` and
`Vehicle_Count` as dominant. Unsupervised learning, which never saw the label, produced clusters that
separate on precisely those variables, and whose severity composition differs systematically. Two
independent methods converging on the same signal is much stronger evidence than either alone. They are
also complementary: the classifier triages, the clusters describe environments without needing labels,
and the anomaly score flags what the classifier cannot see.

**Q: What worked well?**
- Beat the majority-class baseline convincingly on the metric that matters (macro-F1), with the gain
  concentrated in the rare classes.
- The leakage audit prevented a false 0.984 being reported as a success.
- Feature selection matched full-set performance with a third of the inputs, and improved clustering
  more than five-fold.
- Errors are ordinal, not random.

**Q: What are the limitations?**
1. **Synthetic, weakly-signalled data** — many variables are near-uncorrelated with severity, capping
   macro-F1 at about 0.73. The ceiling is a property of the data, not the method.
2. **Diffuse clustering** — every silhouette score is below 0.03 on the full feature set.
3. **Imbalance only partly solved** — Severity 2 has the lowest F1; Severity 1 has fewer than 800
   examples in total.
4. **Random, not chronological, validation** — a time-based split is the correct protocol for
   deployment and would give a lower, more realistic estimate.
5. **Ordinal target treated as nominal** — the models ignore the ordering of the severity levels.
6. **Hotspot analysis limited** by the even spatial distribution of the synthetic data.

**Q: What would you do differently or next?**
Enrich the data with real sensor feeds and road geometry; model severity ordinally or with a
cost-sensitive loss; apply SMOTE inside the CV folds and tune the rare-class threshold; add SHAP
explanations for operators; add DBSCAN spatial clustering and a time-series occurrence forecast;
re-run the cluster-count selection on the reduced feature subset.

**Q: What was the biggest challenge?**
Deciding what *not* to use. The dataset hands you two columns (`Risk_Score`, `Emergency_Response_Time`)
that would have made the model look almost perfect. Recognising that they are derived from the outcome,
proving it, and then reporting a lower but honest score is the part of the project we are most confident
about — and the ablation study is the evidence.

---

## 7. Likely "hard" questions — be ready for these

**"Why did Logistic Regression beat a gradient-boosted ensemble? Isn't boosting always better?"**
Not when the signal is weak and largely linear. Boosting can fit noise, and with a modest signal the
extra capacity does not help — our gradient booster scored 0.7323 against 0.7332. The grid search also
selected `C = 0.1` for logistic regression, i.e. strong regularisation, which is the classic signature
of a dataset that rewards a simple model.

**"Your macro-F1 is only 0.73. Isn't that a bad model?"**
It depends entirely on the baseline. The majority-class baseline is 0.178 macro-F1 and 0.5517 accuracy,
so we are more than four times better than trivial on the metric that matters. The remaining gap is the
weak signal in a synthetic dataset, which we demonstrate directly: the same algorithm reaches 0.984 when
you hand it a leaking variable. That shows the ceiling is in the data, not the pipeline.

**"Why not use all 72 features?"**
Because the ablation study measures it and the answer is no. Twenty columns match or beat sixty-five,
and ten columns are the strongest configuration of all. Extra features here add variance and noise, not
signal — and they cost money to collect in production.

**"How do you know your clustering isn't just noise?"**
Two checks. First, external validation: the clusters' severity composition differs systematically, and
severity was never an input. Second, cross-algorithm agreement: Ward clustering reaches the same
qualitative conclusion. And we are candid that on the full feature set the clusters overlap heavily —
which is why we report silhouette < 0.03 as a limitation rather than pretending otherwise.

**"Why is macro-F1 better than weighted F1?"**
Weighted F1 weights each class by its frequency, so it is dominated by the majority class in the same
way accuracy is. Our model scores 0.7866 weighted but 0.7332 macro. Macro treats a rare critical
incident as equally important as a common minor one, which is what an incident-management centre
actually needs.

**"Why did you keep `Accident_History` if it correlates 0.37 with the target?"**
Because correlation alone is not leakage. The test for leakage is *availability at prediction time*.
The accident history of a road segment is historical data that exists before the incident, and it is a
genuine risk indicator — the basis for identifying high-risk locations, which is one of the project's
stated objectives. `Risk_Score`, by contrast, is *derived from* the outcome.

**"How is your work reproducible?"**
One notebook, one fixed `RANDOM_STATE = 42`, all preprocessing inside a `Pipeline`, and every figure and
metric in the report generated from the same run.

---

## 8. Numbers to memorise

| Item | Value |
|---|---|
| Dataset size | 50,010 rows × 72 columns → 49,992 after cleaning |
| Class balance | Sev 1: 1.57% · Sev 2: 6.96% · Sev 3: 36.31% · Sev 4: 55.17% |
| Majority-class baseline | accuracy 0.5517, macro-F1 0.1778 |
| **Best model — Logistic Regression** | **accuracy 0.7812, macro-F1 0.7332** |
| Gradient boosting | accuracy 0.7864, macro-F1 0.7323 |
| Random forest | accuracy 0.7832, macro-F1 0.7090 |
| k-Nearest Neighbours | accuracy 0.6772, macro-F1 0.3832 |
| Leakage effect | macro-F1 0.9843 (with) vs 0.7206 (without) |
| `Risk_Score` correlation | 0.88 |
| `Emergency_Response_Time` correlation | 0.37 |
| Ablation: all 65 features | macro-F1 0.6412, silhouette 0.0205 |
| Ablation: Top-20 (20 columns) | macro-F1 0.7006, silhouette 0.0473 |
| Ablation: Top-10 (10 columns) | macro-F1 0.7189, silhouette 0.1104 |
| Imbalance effect on Sev-1 recall | 0.7051 (none) → 0.9231 (balanced) |
| Chosen k | 4 (elbow); max silhouette at k = 2 is 0.0275 |
| Anomalies flagged | 500 (top 1% centroid distance) |
| Group importance | Traffic 41.9% · Environmental 22.4% · Spatial 16.7% · Context 12.5% |
| Severity-1 recall / Severity-4 precision | 0.936 / 0.897 |

---

## 9. Demonstration checklist

- [ ] Every member can deliver the 60-second summary (§1) without notes.
- [ ] Every member can name four data-quality problems and their treatments.
- [ ] Every member can explain the leakage decision and quote the 0.98 vs 0.72 contrast.
- [ ] Every member knows which model was selected and why.
- [ ] Every member can explain how k was chosen and why the silhouette is low.
- [ ] One member drives the notebook demo; everyone else is ready to answer questions on **any** section.
- [ ] Open the notebook **before** the session and run it once so the outputs are cached.
- [ ] Have the report and this guide open on a second screen for quick reference.
