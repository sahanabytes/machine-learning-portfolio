# Capstone Report — Machine Learning

- **Author:** Sahanashree M
- **Lane:** Machine Learning
- **Repo:** https://github.com/sahanabytes/machine-learning-portfolio
- **Date:** 2026-11-09

## 0. Abstract

This study investigates whether rule-based scoring and machine-learning models can help identify website content pages that may be at risk of declining, so editors can prioritize pages for human review. The analysis uses a 30,000-row anonymized content dataset containing search-performance and content-related indicators across 32 clients. A transparent rule-based baseline is compared with a depth-3 decision tree using a client-grouped train-test split. On the held-out test set, the rule-based baseline achieved Precision@20 of 0.60 and Precision@50 of 0.60, while the decision tree achieved 0.45 and 0.56 respectively, against a test-set decline rate of approximately 51.7%. The resulting ranking is intended as a decision-support tool for helping content teams prioritize pages for possible review and refresh, not as an automatic decision or a prediction of Google's ranking algorithm.

## 1. Problem framing

This research supports the decision of which website content pages should be reviewed first for a possible content refresh.

The unit of analysis is an individual website content page. The output is a ranked list of pages based on a rule-based score or machine-learning score.

A FlyRank editor can use this ranking to identify pages for human review and decide whether an update may be useful.

A false positive may cause an editor to spend time reviewing a page that does not need attention. A false negative may cause a page at risk of declining to be overlooked.

Data analysis and machine learning help organize information from multiple content and search-performance indicators. They support prioritization, but they do not replace editorial judgment.

## 2. Data safety

The final modeling analysis uses the anonymized dataset `content_refresh_anonymized.csv`, containing 30,000 rows and 44 columns across 32 anonymized clients.

The analysis uses content and search-performance indicators, including:

- `days_since_last_update`
- `impressions_90d`
- `search_volume`
- `competition`
- `word_count`
- `content_age_days`
- `avg_position`
- `ctr`

The following information was deliberately excluded from model features:

- `trend_direction` and `trend_pct`, because they are derived from the outcome and could cause target leakage.
- Client and content identifiers, because they are used for grouping and context only, not as predictive features.
- Other identifier or outcome-derived fields that could reveal the target or create leakage.

The target label is created from `trend_direction`, but the original label-derived field is not included in the feature list. The client-grouped split keeps clients in either training or testing, preventing client overlap between the two sets.

The leakage audit confirmed that blocked columns were not present in the model features and that `decline_label` was not included as a feature.

The dataset is anonymized. No client names, private URLs, page titles, or private search queries are used in the report. Before submission, the complete `work/` folder should also be checked to confirm that no client-identifying information is present.

## 3. Baseline

The baseline is a transparent rule-based scoring method. It assigns:

- 50 points when `days_since_last_update` is greater than or equal to 180.
- 50 points when `impressions_90d` is greater than or equal to 500.

The maximum score is 100. Pages with higher scores are prioritized for review.

This baseline is a fair comparison because it is simple, interpretable, and uses information available in the same modeling dataset. It provides a clear reference for evaluating whether a more complex model improves the ranking.

On the held-out test set, the baseline achieved:

- Precision@20: 0.60
- Precision@50: 0.60

The baseline provides an understandable starting point and makes it possible to compare the machine-learning model against a practical rule.

## 4. Model / analysis

A Decision Tree Classifier was selected because it can identify simple relationships between input features and the target while remaining relatively easy to interpret.

The final model configuration was:

- Model: `DecisionTreeClassifier`
- Maximum depth: 3
- Minimum samples per leaf: 100
- Random state: 42

The exact model feature list was:

1. `days_since_last_update`
2. `impressions_90d`
3. `search_volume`
4. `competition`
5. `word_count`
6. `content_age_days`
7. `avg_position`
8. `ctr`

The model deliberately excludes `trend_direction`, `trend_pct`, `decline_label`, and pseudonymous client/content identifiers from its input features.

The target is `decline_label`, defined as 1 when `trend_direction` is `down` and 0 otherwise.

Missing feature values are handled using medians calculated from the training data, with zero used as a fallback where needed.

The tree is intentionally shallow to keep the model easier to inspect and reduce unnecessary complexity.

## 5. Evaluation

The evaluation uses a grouped train-test split based on client identifiers. This is important because pages from the same client may share characteristics. Keeping clients separate helps test whether the model can generalize to clients it did not see during training.

The split produced:

- Training rows: 22,885
- Testing rows: 7,115
- Training clients: 24
- Testing clients: 8
- Client overlap: None
- Training-set decline rate: approximately 55.0%
- Test-set decline rate: approximately 51.7%

The held-out test-set decline rate of approximately 51.7% is the relevant positive-class base rate for interpreting the Precision@K results. The majority-class baseline is approximately 51.7% as well.

### Model and baseline comparison

| Metric | Rule-based baseline | Decision tree |
|---|---:|---:|
| Precision@20 | 0.60 | 0.45 |
| Precision@50 | 0.60 | 0.56 |

The rule-based baseline performed better than the decision tree on both reported ranking metrics.

At Precision@20, the baseline identified 60% declining pages among its top 20, compared with 45% for the decision tree. At Precision@50, the baseline achieved 60%, compared with 56% for the tree.

Both methods performed above the test-set positive-class rate on the reported Precision@K results, but the improvement should be interpreted carefully because the test set is relatively small and the results are decision-support measurements.

### Error analysis

The decision tree's confusion matrix on the test set was:

| Error category | Count |
|---|---:|
| True positives | 3,242 |
| False positives | 2,664 |
| True negatives | 776 |
| False negatives | 433 |

The model identified many declining pages, but it also produced a considerable number of false positives. These false positives may lead editors to review pages that are not actually labelled as declining. The false negatives represent declining pages that the model did not identify.

The results show why a ranking or model score should be treated as a review aid rather than an automatic content-refresh decision.

AUC and lift over the baseline should be added only if they are calculated and verified from the saved evaluation outputs. They are not reported here because verified values are not currently available.

## 6. Interpretation

The feature-importance results show that the decision tree relied mainly on a small number of search-performance and content-age indicators.

| Feature | Importance |
|---|---:|
| `impressions_90d` | 0.554 |
| `content_age_days` | 0.248 |
| `avg_position` | 0.120 |
| `ctr` | 0.078 |
| `word_count` | 0.000 |
| `competition` | 0.000 |
| `days_since_last_update` | 0.000 |
| `search_volume` | 0.000 |

Impressions over the previous 90 days had the highest importance, followed by content age. Average position and click-through rate also contributed to the tree's decisions.

Word count, competition, days since last update, and search volume had zero importance in this fitted tree. This does not prove that these variables never matter; it means the fitted tree did not use them to make its splits.

An important negative result is that the more complex decision tree did not outperform the simple rule-based baseline. This suggests that adding model complexity did not improve the reported ranking performance in this experiment.

These are observed associations and model results, not evidence that any feature causes content to decline.

## 7. Recommendation

Based on the evaluation, the rule-based baseline is the preferred starting point for this experiment because it achieved better Precision@20 and Precision@50 than the decision tree.

Recommended actions for a FlyRank editor:

1. **Start with the highest-ranked pages:** Use the rule-based score to create an initial review list.
2. **Review pages with older updates:** Check whether pages that have not been updated recently still provide accurate and useful information.
3. **Investigate search-performance changes:** Examine impressions, average position, and click-through rate before deciding whether a page needs attention.
4. **Apply human judgment:** Check content quality, search intent, relevance, and business context before approving a refresh.
5. **Track outcomes:** Measure performance after an update to evaluate whether the editorial action was useful.

Confidence is moderate in the conclusion that the rule-based baseline performed better on the reported held-out ranking metrics. Confidence is lower in any claim about how the approach will perform on other datasets or future periods.

The dataset is anonymized and the evaluation covers a limited set of held-out clients. The results do not establish causation, guarantee future performance, or predict Google's ranking algorithm. A larger evaluation across additional clients and time periods would be needed before operational deployment.

## 8. Reproducibility

The final modeling experiment is documented in `work/notebooks/w05_model.ipynb`. The analysis uses the anonymized dataset `data/raw/content_refresh_anonymized.csv`, containing 30,000 rows and 44 columns.

### Environment setup

The repository provides a `requirements.txt` file containing the project dependencies. To install them from a fresh clone, run:

```bash
pip install -r requirements.txt
```

For a fresh Colab or Jupyter notebook used for the full-release workflow, the documented setup cell is:

```python
%pip install -q duckdb huggingface_hub pandas scikit-learn matplotlib
```

### Reference pipeline

The repository's reference pipeline can be run from the repository root using:

```bash
python scripts/run_all.py
```

It executes the five stages in order:

1. `01_prepare_features.py` — prepares features and defines the label.
2. `02_baseline_score.py` — creates the transparent baseline score.
3. `03_train_model.py` — trains models and evaluates them using a client-holdout split.
4. `04_evaluate_and_export.py` — combines rankings and exports evaluation outputs.
5. `05_build_pdf_report.py` — builds the PDF report.

The reference scripts are kept unchanged. The capstone experiment is documented separately in the `work/` directory.

### Capstone model configuration

The capstone decision tree uses:

- `max_depth=3`
- `min_samples_leaf=100`
- `random_state=42`

The model uses eight numerical features. The target is `decline_label`, defined as 1 when `trend_direction` is `down` and 0 otherwise.

The evaluation uses a client-grouped split with 24 training clients and 8 testing clients, with no client overlap. Missing feature values are handled using training-set medians, with zero as a fallback.

### Reproduction checks and limitations

The saved modeling notebook should be opened and executed from top to bottom to reproduce the capstone experiment. The reported test metrics and feature importances must be compared with a fresh execution before publication.

The reference pipeline and the capstone notebook are separate workflows. Their results must not be mixed.

The random seed for the capstone decision tree is 42. The exact Python and package versions used for the final capstone execution must be recorded from the actual environment.

No claim of a blind or sealed evaluation is made. AUC and lift are not reported because verified capstone values are not currently available.

## 9. Acknowledgments & data credit

This research was built on the FlyRank ML Internship dataset.

[Built on the FlyRank ML Internship dataset](https://flyrank.ai)

The author acknowledges FlyRank for providing the internship learning materials and anonymized data used in this project.
