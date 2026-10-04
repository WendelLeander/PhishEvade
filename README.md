# Obfuscation-Aware Feature Engineering for Evasion-Heavy Phishing URL Detection

This repository contains the datasets, feature extraction pipelines, and Jupyter notebooks for the thesis project: **Obfuscation-Aware Feature Engineering for Evasion-Heavy Phishing URL Detection**.

For full theoretical background, mathematical formalizations, and methodology, please refer to the official proposal document.

## About the Project

Traditional machine learning models often struggle to detect evasion-heavy phishing URLs that utilize advanced obfuscation techniques (e.g., character substitution, multi-level subdomain nesting, and mathematical encoding). While deep learning solves the accuracy issue, it is resource-intensive.

This project bridges that gap by introducing an **obfuscation-aware feature engineering framework**. By calculating mathematical randomness (Shannon Entropy) and architectural irregularities (Structural Complexity) directly from the URL string, we empower lightweight traditional classifiers to detect hidden malicious patterns without needing to analyze webpage content.

## Key Features Extracted

The feature engineering pipeline targets multiple structural components of a URL (defined under RFC 3986) to compute 13 distinct features, categorized into:

* **Level 1 (Shallow Lexical Criteria):** URL Length, Domain Length, Path Length.

* **Level 2 (Deep Obfuscation-Aware Criteria):** Domain Entropy, Token Variance, Digit/Special Character Ratios, Subdomain Count, Path Depth, Query Parameters, Redirect Key Flags, Nested URL Flags, and Query Encoding Ratios.

## Machine Learning Models

The extracted feature sets are evaluated using the following interpretable traditional machine learning algorithms:

* Logistic Regression (Baseline)

* Naïve Bayes

* Random Forest

* AdaBoost

* XGBoost

## Dataset

The project utilizes a balanced dataset of 10,000 URLs (50% benign, 50% phishing) collected between 2022 and 2026.

* **Malicious Sources:** PhishTank and PhishStats

* **Benign Sources:** Tranco, Majestic Million

## Authors

* Maristela, Kyle Gabriel A.

* Leander, Wendel Walter A.

* San Luis, Owen Phillip C.

*De La Salle University - College of Computer Studies*