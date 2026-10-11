# Craniofacial Morphology Clustering — Analysis Pipeline

## Overview

This repository documents a reproducible, unsupervised-learning workflow for examining patterns of sagittal and vertical craniofacial morphology using cephalometric measurements. The pipeline is intended for research use and is not a clinical diagnostic or classification tool.

The primary analysis considers the **complete adult sample (age ≥18 years)**. Sex, age, conventional sagittal classification, and supplementary cephalometric measurements are reserved for **post-hoc characterization**, rather than used to construct clusters. No study findings are reported in this README.

## Input data

The pipeline accepts an Excel workbook uploaded interactively in **Google Colab**. It reads the `Analysis_Dataset` worksheet when available; otherwise, it uses the first worksheet.

**Required columns:**

| Column | Description |
| --- | --- |
| `ID` | Participant identifier |
| `Age` | Age in years |
| `Sex` | Sex coded as `1` (male) or `2` (female) |
| `SNA` | SNA angle, in degrees |
| `SNB` | SNB angle, in degrees |
| `SN_GoGn` | SN.GoGn angle, in degrees |

**Optional post-hoc columns:** `ANB`, `FMA`, `Co_A`, `Co_Gn`, `Mx_Md_Diff`, and `Sagittal_Class_Source`.

Column names must match those shown above. Numeric columns are converted to numeric values, and invalid entries become missing values. Participants younger than 18 years or missing any discovery measurement are excluded from the primary analysis. The original clinical dataset is not distributed with this documentation.

## Analytical workflow

1. **Data quality control.** Inspect missing values and flag extreme observations in the three discovery variables using robust z-scores based on the median absolute deviation. The configured exclusion threshold is `|robust z| > 8`. ANB is checked against `SNA − SNB`; discrepancies greater than 0.5 degrees are set to missing for ANB-specific descriptions only.
2. **Standardization and PCA.** Standardize SNA, SNB, and SN.GoGn and retain enough principal components to explain at least 95% of cumulative variance. Depending on the data, all three components may be retained.
3. **Pre-clustering assessment.** Apply Hartigan's dip test to the first principal component and calculate the Hopkins statistic over 200 repetitions using 10% of observations per repetition.
4. **Cluster-number assessment.** Evaluate the Gap statistic for `k = 1–10` using 100 reference datasets and the one-standard-error selection rule. The procedure permits `k = 1`, meaning that a multi-cluster partition is not supported by this criterion.
5. **Candidate clustering methods.** Examine K-means (`k = 2–10`, 100 initializations), Ward agglomerative clustering (`k = 2–10`), and Gaussian mixture models (`k = 1–10`, 20 initializations; full, tied, diagonal, and spherical covariance structures). Gaussian mixture models are evaluated primarily by BIC, with AIC as a complementary measure.
6. **Internal validation and stability.** Calculate Silhouette, Calinski–Harabasz, and Davies–Bouldin indices. Assess partition stability using 200 random subsamples containing 80% of participants, quantified with the adjusted Rand index (ARI).
7. **Primary partition.** If the Gap statistic selects `k ≥ 2`, fit K-means with that value of `k` to the retained principal-component representation. If `k = 1`, no primary multi-cluster partition or post-hoc cluster comparison is produced.
8. **Cluster characterization.** When a primary partition exists, summarize SNA, SNB, and SN.GoGn without inferential comparisons between clusters. Estimate 95% confidence intervals for cluster means using 2,000 bootstrap resamples, repeating standardization, PCA, and clustering in each resample and matching cluster labels to the reference solution.
9. **Post-hoc analyses.** Characterize sex and recorded sagittal class using contingency tables, Pearson's chi-square test, and Cramér's V. Assess correspondence with sagittal class using ARI and normalized mutual information (NMI). For a two-cluster solution, compare age, FMA, Co-A, Co-Gn, and maxillomandibular difference using the Mann–Whitney U test and rank-biserial correlation. ANB is descriptive only because it is derived from SNA and SNB.

