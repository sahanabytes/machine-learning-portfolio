# Content Opportunity Scoring for Website Content Refresh

## Research Paper

A Comparative Study of Rule-Based and Machine-Learning Approaches

**Research Area:** Machine Learning and SEO

**Research Focus:** Identifying website content that may be at risk of declining so content teams can prioritize pages for human review and potential refresh.

## Abstract

This research compares a rule-based approach and a machine-learning model for identifying website content that may be at risk of declining. Using an anonymized dataset of 30,000 content records from 32 clients, the study evaluates both approaches using Precision@20 and Precision@50. The rule-based baseline achieved 0.60 on both metrics, while the Decision Tree achieved 0.45 and 0.56, respectively. The findings suggest that a simple rule-based approach performed better in this experiment. The results support using data-driven prioritization as an aid to human review, rather than automatically updating content.

## 1. Research Question

Can rule-based scoring and machine-learning models help identify website content that may be at risk of declining, so content teams can prioritize pages for human review and potential refresh?

## 2. Data

The study uses the anonymized `content_refresh_anonymized.csv` dataset containing 30,000 rows and 44 columns from 32 clients.

The dataset includes content and search-performance indicators such as impressions, search volume, average position, click-through rate, content age, word count, and days since the last update.

The target variable identifies whether a content record has a declining trend. Client and content identifiers are used for grouping and context, not as model features. Private client names, URLs, page titles, and search queries are not included.

## 3. Methodology

The target variable is `decline_label`, where 1 represents a declining trend and 0 represents other trends.

Eight features were used: days since last update, impressions over 90 days, search volume, competition, word count, content age, average position, and CTR.

The dataset was divided using a grouped 75:25 train-test split by client. The training set contained 22,885 rows from 24 clients, and the test set contained 7,115 rows from 8 different clients. There was no client overlap.

Missing feature values were filled using training-set medians. A Decision Tree Classifier with maximum depth 3 and minimum 100 samples per leaf was compared with a rule-based baseline.

Both approaches were evaluated on the same held-out test set using Precision@20 and Precision@50. A leakage audit confirmed that the target was not included among the input features.

## 4. Results

| Method | Precision@20 | Precision@50 |
|---|---:|---:|
| Rule-based baseline | 0.60 | 0.60 |
| Decision Tree | 0.45 | 0.56 |

The rule-based baseline performed better than the Decision Tree on both evaluation metrics.

The results suggest that a simple scoring method can be useful for prioritizing potentially declining content in this experiment.

## 5. Limitations

The dataset contains 30,000 records from 32 clients, and the test set represents only 8 clients. The findings may not generalize to all websites or content types.

A declining trend does not necessarily mean that refreshing a page will improve its performance. This study does not establish a causal relationship between content updates and traffic growth.

The Decision Tree produced false positives and false negatives. Its results were also weaker than the baseline on both precision metrics.

Therefore, the model should not be used to automatically update or remove content. Human review remains necessary.

## 6. Ranked Recommendations

1. Use the rule-based baseline as the current prioritization method because it performed better in this experiment.
2. Give 50 points to pages with at least 180 days since their last update.
3. Give another 50 points to pages with at least 500 impressions over 90 days.
4. Prioritize pages with higher scores for human review.
5. Investigate pages flagged by the Decision Tree but not by the baseline as a secondary source of insights.
6. Monitor actual performance after content refreshes before making stronger recommendations.

## 7. Artifacts the Paper Embeds

The experiment includes a comparison of Precision@20 and Precision@50 for the rule-based baseline and Decision Tree.

The Decision Tree feature importance results were:

| Feature | Importance |
|---|---:|
| Impressions over 90 days | 0.554 |
| Content age | 0.248 |
| Average position | 0.120 |
| CTR | 0.078 |
| Word count | 0.000 |
| Competition | 0.000 |
| Days since last update | 0.000 |
| Search volume | 0.000 |

The model error summary was:

- True positives: 3,242
- False positives: 2,664
- True negatives: 776
- False negatives: 433

These artifacts show how the model performed and which features contributed most to its predictions.

## 8. Acknowledgments & Data Credit

This research was completed as part of the FlyRank AI Machine Learning Internship.

I acknowledge FlyRank for providing the learning materials, anonymized dataset, and assignment framework used in this project.

This paper presents an educational analysis based on the available data. The findings and recommendations are my own and should not be interpreted as official claims by FlyRank.

Organization: [FlyRank](https://flyrank.ai)
