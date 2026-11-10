# Preliminary findings: PhishEvade

**Status:** Preliminary analysis for panel feedback. Scaling is fitted on the training rows only and the evasion holdout was never used for training or tuning; the remaining limitations (test-set reuse, undated benign URLs, shared domains) are described below.

## 1. Evaluation design

We evaluated five classifiers under three feature configurations: Level 1 (3 basic length features), Level 2 (10 obfuscation-aware features), and Combined (all 13). The dataset carries a fixed split. Its newest phishing URLs (1,080 URLs first seen between 2026-03-01 and 2026-05-25, with first-seen dates cross-checked against PhishTank and PhishStats) and 1,080 benign URLs form an **evasion-heavy holdout** that was excluded from training, tuning and threshold selection. The remaining URLs were split 80:20 into 6,272 training rows and 1,568 test rows (784 benign and 784 phishing). Module 7 selected hyperparameters using F1 in five-fold GridSearchCV on the training rows, with all feature scaling fitted inside each model pipeline on training rows only. For evaluation, we independently loaded the saved models, flagged scores at or above 0.5 as phishing, and compared the results with the Module 6 Logistic Regression Level 1 baseline. F1 was our main performance measure on the test set; recall, false negative rate (FNR), and false positive rate (FPR) showed the operational cost of missed threats and false alarms. Tuned thresholds are reported only as an adjustment aid.

The artifact checks confirmed that all 15 saved model files matched the manifest, and that their confusion matrices reproduced Module 7's recorded results. The separately loaded Module 6 baseline reproduced the Logistic Regression Level 1 reference. These checks establish that the saved models produce the expected outputs on the fixed test partition and the evasion holdout. They do not make the previously displayed test set untouched.

## 2. Full comparison on the test set at the 0.5 threshold

| Classifier | Features | Accuracy | Precision | Recall | F1 | FPR | FNR | ΔF1 vs LR (pp) |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Logistic Regression | Level 1 | 62.76% | 64.33% | 57.27% | 60.59% | 31.76% | 42.73% | +0.00 |
| Logistic Regression | Level 2 | 64.03% | 61.88% | 73.09% | 67.02% | 45.03% | 26.91% | +6.42 |
| Logistic Regression | Combined | 70.34% | 70.42% | 70.15% | 70.29% | 29.46% | 29.85% | +9.69 |
| Random Forest | Level 1 | 89.16% | 92.40% | 85.33% | 88.73% | 7.02% | 14.67% | +28.13 |
| Random Forest | Level 2 | 92.47% | 95.12% | 89.54% | 92.25% | 4.59% | 10.46% | +31.65 |
| Random Forest | Combined | 94.07% | 96.63% | 91.33% | 93.90% | 3.19% | 8.67% | +33.31 |
| XGBoost | Level 1 | 88.90% | 91.55% | 85.71% | 88.54% | 7.91% | 14.29% | +27.94 |
| XGBoost | Level 2 | 91.77% | 93.84% | 89.41% | 91.57% | 5.87% | 10.59% | +30.98 |
| XGBoost | Combined | 94.52% | 96.04% | 92.86% | 94.42% | 3.83% | 7.14% | +33.83 |
| Naive Bayes | Level 1 | 76.47% | 82.88% | 66.71% | 73.92% | 13.78% | 33.29% | +13.33 |
| Naive Bayes | Level 2 | 55.74% | 53.13% | 97.45% | 68.77% | 85.97% | 2.55% | +8.17 |
| Naive Bayes | Combined | 56.25% | 53.53% | 94.90% | 68.45% | 82.40% | 5.10% | +7.85 |
| AdaBoost | Level 1 | 86.03% | 88.75% | 82.53% | 85.53% | 10.46% | 17.47% | +24.93 |
| AdaBoost | Level 2 | 88.58% | 90.50% | 86.22% | 88.31% | 9.06% | 13.78% | +27.71 |
| AdaBoost | Combined | 92.86% | 94.44% | 91.07% | 92.73% | 5.36% | 8.93% | +32.13 |

![F1, recall and FPR for all 15 combinations relative to the Level 1 Logistic Regression baseline.](../milestone8/figures/01_feature_comparison.png)

The baseline reached **60.59% F1** and **57.27% recall**, with **335 missed phishing URLs** and **249 false alarms**. **XGBoost / Combined** had the highest observed test F1 (**94.42%**, 95% row-bootstrap interval 93.31%–95.55%), with **92.86% recall** and **3.83% FPR**. It missed 56 of 784 phishing URLs and flagged 30 of 784 benign URLs. Compared with the baseline, that is 279 fewer misses and 219 fewer false alarms. Its F1 was +33.83 pp above the baseline.

**Random Forest / Combined** followed at 93.90% F1, with 91.33% recall and 3.19% FPR. The difference between these two top observed F1 scores was only 0.52 pp. We treat both as candidates for confirmation; ranking them on an already inspected test split does not establish a reliable winner for future URLs. The highest **training-CV** F1 belonged to XGBoost / Combined (94.05%), so the CV ranking can be distinguished from the observed test ranking.

## 3. Did the Level 2 features improve detection on the test set?

| Classifier | Features | ΔF1 (pp) | 95% ΔF1 interval (pp) | ΔRecall (pp) | ΔFPR (pp) | F1 p (Holm) | FPR reduction p (Holm) |
| --- | --- | --- | --- | --- | --- | --- | --- |
| Logistic Regression | Level 2 | 6.42 | +3.80 to +9.04 | 15.82 | 13.27 | 0.0048 | 1 |
| Logistic Regression | Combined | 9.69 | +6.46 to +12.73 | 12.88 | -2.3 | 0.0048 | 1 |
| Random Forest | Level 2 | 3.52 | +1.82 to +5.14 | 4.21 | -2.42 | 0.0048 | 0.189 |
| Random Forest | Combined | 5.17 | +3.72 to +6.61 | 5.99 | -3.83 | 0.0048 | 0.000339 |
| XGBoost | Level 2 | 3.04 | +1.19 to +4.82 | 3.7 | -2.04 | 0.0048 | 0.403 |
| XGBoost | Combined | 5.89 | +4.33 to +7.48 | 7.14 | -4.08 | 0.0048 | 0.000609 |
| Naive Bayes | Level 2 | -5.16 | -7.58 to -2.56 | 30.74 | 72.19 | 0.0048 | 1 |
| Naive Bayes | Combined | -5.48 | -7.86 to -2.99 | 28.19 | 68.62 | 0.0048 | 1 |
| AdaBoost | Level 2 | 2.78 | +0.78 to +4.78 | 3.7 | -1.4 | 0.0072 | 1 |
| AdaBoost | Combined | 7.2 | +5.61 to +8.88 | 8.55 | -5.1 | 0.0048 | 6.01e-05 |

The feature effect is best assessed **within the same classifier**. Combined increased observed test F1 in **4 of 5** classifiers, while Level 2 alone increased it in **4 of 5**. Combined improved F1 **and** reduced FPR relative to the same classifier's Level 1 version, with Holm-adjusted p-values below 0.05 for both tests, for: **Random Forest, XGBoost, AdaBoost**. Logistic Regression illustrates why F1 and FPR must be read together: Level 2 alone changed F1 by +6.42 pp and FPR by +13.27 pp, while the combined version had FPR 29.46% and F1 70.29%.

Classifiers whose Combined F1 fell below their Level 1 F1: **Naive Bayes**. For Naive Bayes, Combined changed recall from 66.71% to 94.90% and FPR from 13.78% to 82.40%, lowering F1 by 5.48 pp. The observed pattern supports using the engineered features with suitable classifiers; it does not show that adding them always improves detection.

The paired bootstrap intervals use the same resampled test rows for each model. They describe the precision of observed F1 differences on this dataset. The paired F1 randomization, overall-error, and benign FPR tests use Holm correction across the 24 contrasts per test family. They are conditional on this split; the temporal question is answered by the evasion holdout in the next section. The F1 intervals are exploratory and unadjusted for multiple comparisons.

![Observed hold-out F1 with 95% class-stratified bootstrap intervals.](../milestone8/figures/02_f1_intervals.png)

## 4. Evasion-heavy holdout: newest phishing URLs (1,080 phishing, first seen 2026-03-01 to 2026-05-25; 1,080 benign)

| Classifier | Features | Recall | FNR | FPR | F1 | Missed (FN) | False alarms (FP) | Recall vs. test (pp) |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Logistic Regression | Level 1 | 75.93% | 24.07% | 27.13% | 74.78% | 260 | 293 | +18.66 |
| Logistic Regression | Level 2 | 78.61% | 21.39% | 41.85% | 71.31% | 231 | 452 | +5.52 |
| Logistic Regression | Combined | 67.22% | 32.78% | 26.94% | 69.24% | 354 | 291 | -2.93 |
| Random Forest | Level 1 | 73.89% | 26.11% | 7.13% | 81.64% | 282 | 77 | -11.44 |
| Random Forest | Level 2 | 71.76% | 28.24% | 3.61% | 81.84% | 305 | 39 | -17.78 |
| Random Forest | Combined | 79.54% | 20.46% | 2.31% | 87.47% | 221 | 25 | -11.79 |
| XGBoost | Level 1 | 73.89% | 26.11% | 7.41% | 81.51% | 282 | 80 | -11.83 |
| XGBoost | Level 2 | 73.33% | 26.67% | 4.72% | 82.37% | 288 | 51 | -16.08 |
| XGBoost | Combined | 81.11% | 18.89% | 4.07% | 87.60% | 204 | 44 | -11.75 |
| Naive Bayes | Level 1 | 58.61% | 41.39% | 12.50% | 68.51% | 447 | 135 | -8.10 |
| Naive Bayes | Level 2 | 98.33% | 1.67% | 84.91% | 69.43% | 18 | 917 | +0.88 |
| Naive Bayes | Combined | 98.33% | 1.67% | 83.06% | 69.89% | 18 | 897 | +3.44 |
| AdaBoost | Level 1 | 76.76% | 23.24% | 9.63% | 82.36% | 251 | 104 | -5.77 |
| AdaBoost | Level 2 | 70.56% | 29.44% | 8.15% | 78.96% | 318 | 88 | -15.67 |
| AdaBoost | Combined | 81.94% | 18.06% | 4.91% | 87.71% | 195 | 53 | -9.13 |

![Recall, FNR and FPR on the evasion-heavy holdout for all 15 combinations at threshold 0.5.](../milestone8/figures/10_evasion_holdout_recall_fnr_fpr.png)

### Within-classifier feature comparison on the holdout

| Classifier | Features (vs. Level 1) | ΔRecall (pp) | ΔFPR (pp) | Newly caught | Newly missed | Recall p (Holm) | FPR p (Holm) |
| --- | --- | --- | --- | --- | --- | --- | --- |
| Logistic Regression | Level 2 | +2.69 | +14.72 | 118 | 89 | 0.154 | 2.73e-16 |
| Logistic Regression | Combined | -8.70 | -0.19 | 91 | 185 | 1.27e-07 | 0.958 |
| Random Forest | Level 2 | -2.13 | -3.52 | 125 | 148 | 0.366 | 0.000939 |
| Random Forest | Combined | +5.65 | -4.81 | 114 | 53 | 1.35e-05 | 3.68e-08 |
| XGBoost | Level 2 | -0.56 | -2.69 | 138 | 144 | 0.766 | 0.0166 |
| XGBoost | Combined | +7.22 | -3.33 | 141 | 63 | 3.4e-07 | 0.000939 |
| Naive Bayes | Level 2 | +39.72 | +72.41 | 438 | 9 | 9.15e-116 | 1.09e-214 |
| Naive Bayes | Combined | +39.72 | +70.56 | 437 | 8 | 8.03e-117 | 4.79e-210 |
| AdaBoost | Level 2 | -6.20 | -1.48 | 77 | 144 | 3.1e-05 | 0.422 |
| AdaBoost | Combined | +5.19 | -4.72 | 88 | 32 | 1.91e-06 | 2.23e-06 |

The evasion holdout tests generalization to phishing URLs **newer than anything in training**. At the 0.5 threshold, the Level 1 baseline caught **75.93%** of these newest phishing URLs (FNR 24.07%, 260 missed) with FPR 27.13%. The leading test-set model, **XGBoost / Combined**, caught **81.11%** (FNR 18.89%, 204 missed), a change of -11.75 pp from its test-set recall, with FPR 4.07%. Its own Level 1 version caught 73.89% (FPR 7.41%).

Within each classifier, Combined features significantly **increased** holdout recall over Level 1 (exact McNemar on phishing rows, Holm-adjusted p < 0.05) for: **Random Forest, XGBoost, Naive Bayes, AdaBoost**; of these, FPR did not rise for: **Random Forest, XGBoost, AdaBoost**. Combined significantly **decreased** holdout recall for: **Logistic Regression**. A recall gain is only useful if false alarms stay acceptable, so the FPR column must be read alongside it; a model that flags almost everything reaches high recall with an unusable FPR.

False alarms on the holdout's benign URLs for XGBoost / Combined were 2.89% on domain-only URLs, 5.65% on light-path URLs and 3.67% on obfuscated-looking benign URLs (per-group rates for every model are in `benign_group_fpr.csv`). The holdout is temporal but not domain-disjoint: 457 of its 1,080 phishing URLs share a registered domain with training URLs. For XGBoost / Combined, recall was 91.47% on those and 73.52% on the 623 phishing URLs from unseen domains, so part of the holdout performance reflects familiar domains. The holdout's benign URLs are not dated, so its FPR is not a time-based result.

![Confusion matrices for all 15 combinations on the evasion-heavy holdout.](../milestone8/figures/11_evasion_holdout_confusion_matrices_15.png)

**Supplementary (test set):** a heuristic evasion-indicator slice of the test set (query encoding, redirect or nested URL flags, deep paths/subdomains, or high entropy/token variance relative to training) contains 286 phishing and 164 benign URLs. XGBoost / Combined detected 97.20% of these phishing URLs (baseline 73.08%) with FPR 2.44%. Because the indicators are computed from the same features the models use, this slice is partly circular and is reported only as a supplement to the temporal holdout. Per-slice rates are in `subset_evaluation.csv`; defanged, row-indexed error examples are in `error_examples_defanged.csv`.

## 5. Feature importance, fit, and threshold sensitivity

Top five shuffled features for the leading Combined model, measured by decrease in test F1:

| feature | level | f1_drop_mean | f1_drop_std |
| --- | --- | --- | --- |
| path_length | Level 1 | 0.1245 | 0.0018 |
| domain_length | Level 1 | 0.0809 | 0.0061 |
| token_variance | Level 2 | 0.0507 | 0.002 |
| path_depth | Level 2 | 0.0463 | 0.004 |
| domain_entropy | Level 2 | 0.0437 | 0.0007 |

For XGBoost / Combined, shuffling **path_length** caused the largest average F1 drop (12.45 pp). Both lexical and obfuscation-aware columns are inspected in the permutation table. Shuffling measures how the fitted model used the feature in this test set; it does not identify a causal signal or reliably separate contributions of correlated columns. The largest resubstitution-to-test F1 gap was 8.01 pp for XGBoost / Level 2; the resubstitution score is an optimistic fit diagnostic. The leading model had CV F1 94.05% and test F1 94.42%. Close scores are useful as a consistency check but cannot rule out shared-domain or preprocessing effects.

At the stored training-derived threshold (0.57), the leading model's observed recall was 92.35% and FPR was 3.44%, versus 92.86% and 3.83% at 0.5. This sensitivity comparison is secondary; we retained 0.5 as the proposal's primary rule. The model's ROC-AUC was 0.9822, average precision 0.9857, and Brier score 0.0447. The output score estimates **phishing** likelihood under the model. The experiment does not establish that it measures intent to evade or that its probabilities will be calibrated at real-world phishing prevalence.

![Permutation importance for the five Combined models, three shuffled repetitions.](../milestone8/figures/06_permutation_importance.png)

## 6. Resource tradeoffs

| Model | Training s (original) | Peak RSS MiB (original) | Original mean ms/URL | Rerun mean / p95 ms/URL | Model MiB | Extraction s / 10k (original) |
| --- | --- | --- | --- | --- | --- | --- |
| Logistic Regression / Level 1 | 6.43 | 275.2 | 1.369 | 1.738 / 4.891 | 0.003 | 0.232 |
| XGBoost / Combined | 18.83 | 337.9 | 1.198 | 2.689 / 3.632 | 0.765 | 1.009 |
| Random Forest / Combined | 99.11 | 308.8 | 17.239 | 25.363 / 38.241 | 23.060 | 1.009 |
| AdaBoost / Combined | 118.90 | 338.1 | 30.785 | 42.018 / 64.334 | 0.286 | 1.009 |

XGBoost / Combined took 18.83 s for the original grid search and refit, compared with 99.11 s for Random Forest / Combined. Their original model-only mean inference times were 1.198 and 17.239 ms per URL, respectively. In the separate evaluation runtime, the observed means were 2.689 and 25.363 ms per URL. The models occupied 0.765 and 23.060 MiB on disk. Training peak RSS includes the original Python process and previously retained models; it is not the memory attributable solely to each fitted classifier. Extraction times come from the rounded Module 3–4 profile; they are not new measurements and do not include browsing or deployment. The timings support a comparison of this offline experiment. They cannot establish browser, gateway, or deep-learning performance.

![Historical training cost and current model-only inference latency for the 15 combinations.](../milestone8/figures/08_resource_profile.png)

## 7. Preliminary conclusion and work before final evaluation

On the test set, XGBoost / Combined had the highest observed F1 (94.42%, versus 60.59% for the baseline), followed by Random Forest / Combined (93.90%). On the evasion-heavy holdout of the newest phishing URLs, XGBoost / Combined caught 81.11% of phishing URLs with FPR 4.07%, versus 75.93% recall and 27.13% FPR for the baseline. Section 3 lists the classifiers for which Combined features improved F1 and FPR on the test set, and Section 4 the classifiers for which they improved recall on the holdout; where these agree, the results support the finding that the Level 2 features add useful information to the lexical features for those classifiers. Where they disagree, or where a recall gain comes with a large rise in false alarms, the gain is model-dependent and should not be generalized.

These results remain provisional. The test set was displayed in Modules 6 and 7, so only the evasion holdout is a first look. The holdout is temporal for phishing URLs but not for benign URLs (the top-site lists carry no collection dates), and part of its phishing URLs share a registered domain with training URLs. The source rankings/feeds and balanced class ratio also differ from live traffic, so precision here should not be extrapolated to deployment. Full-string lowercasing can change case-sensitive paths and encoded payloads. The panel should confirm the 3/10 feature allocation and the proposal's Table 4.1 definitions.

Already in place: de-duplication after normalization (0 duplicates remain), first-seen dates and source provenance for every phishing URL, scaling fitted inside each training fold with the full pipelines persisted, and a temporal evasion holdout excluded from training and tuning. Before final evaluation, we should confirm the gains on an independently collected, domain-disjoint set of later phishing URLs with dated benign counterparts, report detection and false alarm rates with their support, and re-measure resource costs on the target hardware.

**Evidence trail:** `performance_comparison_15.csv`, `evasion_holdout_comparison_15.csv`, `evasion_holdout_paired_comparisons.csv`, `benign_group_fpr.csv`, `evasion_holdout_domain_overlap.csv`, `paired_comparisons.csv`, `subset_evaluation.csv`, `resource_comparison.csv`, `tuned_threshold_sensitivity.csv`, `artifact_verification.csv`, and `evaluation_metadata.json` in `Results/milestone8/`. All plots and scores use the saved Module 7 models.
