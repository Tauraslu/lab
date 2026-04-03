# USDA Food Security Survey Module — Measurement Audit

> **A structured QA audit of how applied researchers implement the USDA Household Food Security Survey Module (HFSSM) in peer-reviewed health research.**

[![OSF Pre-registration](https://img.shields.io/badge/OSF-Pre--registered-blue)](https://osf.io)
[![R Version](https://img.shields.io/badge/R-%3E%3D4.2.0-green)](https://www.r-project.org/)
[![Python Version](https://img.shields.io/badge/Python-%3E%3D3.9-yellow)](https://www.python.org/)
[![License: MIT](https://img.shields.io/badge/License-MIT-lightgrey)](LICENSE)

---

## Table of Contents

1. [Study Overview](#study-overview)
2. [Research Questions and Hypotheses](#research-questions-and-hypotheses)
3. [The Four Failure Modes](#the-four-failure-modes)
4. [Sample Design](#sample-design)
5. [Inclusion and Exclusion Criteria](#inclusion-and-exclusion-criteria)
6. [Project Structure](#project-structure)
7. [Workflow](#workflow)
8. [Variables](#variables)
9. [Analysis Plan](#analysis-plan)
10. [NHANES Demonstration](#nhanes-demonstration)
11. [Inter-Rater Reliability Protocol](#inter-rater-reliability-protocol)
12. [Python QA Checker](#python-qa-checker)
13. [File Naming Convention](#file-naming-convention)
14. [Key Outputs](#key-outputs)
15. [Contributing and Authorship](#contributing-and-authorship)
16. [Citation](#citation)

---

## Study Overview

**Title:** Measurement Failures in the USDA Household Food Security Survey Module: A Structured Audit of Applied Health Research

**Team:**
- Brian Spitzer — PI / Admin
- Cheyenne Sewell — Project Leader
- Nate Maxey
- Tauras Lu
- Jason Maloney

**Background:** Widely used public health measures often behave differently once they leave their original design context and are applied in real-world research settings. This study examines how the USDA Household Food Security Survey Module (HFSSM) is actually implemented in applied health research — not whether the scale itself is valid, but whether researchers use it correctly.

Rather than revalidating the scale itself, this project treats **measurement implementation as the object of study**. Researchers make practical decisions when using the HFSSM, including collapsing categories, handling missing data, and selecting reference periods. These decisions are rarely audited and can systematically distort results.

We conduct a structured audit of approximately **60 peer-reviewed studies** that use the USDA food security module in statistical models predicting health outcomes.

---

## Research Questions and Hypotheses

**Primary Research Question:**
> Do measurement pipelines break more often when instruments become more complex?

**Hypotheses:**

1. Measurement failure modes will occur in at least **30% of studies** using the USDA food security module.
2. Failure rates will be **higher in studies using the 10-item and 18-item modules** compared to the 6-item short form.
3. Certain failure modes (e.g., skip-logic errors, category misclassification) will be **more common in more complex instruments**.

---

## The Four Failure Modes

| # | Name | Description |
|---|---|---|
| 1 | **Scale Compression** (Binary Trap) | Collapsing the 3-level ordinal scale into a blunt binary (Secure vs. Insecure), discarding severity information |
| 2 | **Category Misclassification** (Marginal Shift) | Mishandling the "Marginal" food security group — absorbed into the wrong category without justification |
| 3 | **Temporal Misalignment** (Timeline Mismatch) | Using a 30-day or 12-month recall module without matching it to the timeframe of the health outcome |
| 4 | **Data Pipeline Failure** (Skip-Logic Disaster) | Treating valid USDA skip-logic responses as missing data, accidentally deleting the healthiest control group |

> **Key principle:** These failure modes are upstream of model choice. Whether a paper uses logistic regression, linear regression, ANOVA, or a hazard model, it can still commit the same implementation errors.

---

## Sample Design

| Stratum | Target N | Module Version |
|---|---|---|
| Short Form | 30 papers | USDA 6-Item Short Form |
| Full Module | 30 papers | USDA 10-Item or 18-Item |
| **Total** | **60 papers** | |

**Dataset diversity rule:** No more than **8 papers** from any single survey dataset (NHANES, BRFSS, NHIS, etc.). The most recent studies are prioritized when a dataset is at its limit.

**Date range:** Papers published from **2000 onward** are eligible. The full search strategy and any refinements will be documented and archived as a supplement.

**Replacement rule:** If a paper initially appears eligible but USDA module use cannot be confirmed during coding, it will be excluded and replaced from the same module stratum.

---

## Inclusion and Exclusion Criteria

### Include if ALL of the following are true:
- Uses a USDA food security module (6-item, 10-item, or 18-item) — confirmed in the Methods section
- Uses statistical modeling (regression, survival analysis, ANOVA, or equivalent)
- Food security is used as a **predictor** of a health-related outcome (chronic disease, mental health, mortality, BMI/obesity, healthcare utilization, diet quality)
- Published in a **peer-reviewed journal**
- Written in **English**

### Exclude if ANY of the following are true:
- Purely descriptive — reports prevalence only, no statistical model
- Qualitative — interviews, focus groups, ethnographic analysis
- Scale validation or development study — main goal is testing or redesigning the USDA measure
- Uses a non-USDA tool (FIES, HFIAS, Radimer/Cornell, single custom items)
- Secondary literature — reviews, meta-analyses, editorials, commentaries

---

## Project Structure

```
usda-food-security-audit/
│
├── README.md
├── LICENSE
├── .gitignore
│
├── pre-registration/
│   ├── osf_preregistration.md
│   └── osf_preregistration_submitted.pdf
│
├── codebook/
│   ├── coding_manual.md
│   ├── method_coding_guide.md
│   ├── edge_cases_log.md
│   └── irr_calibration_set.md
│
├── data/
│   ├── raw/
│   │   ├── paper_tracking_sheet.csv
│   │   ├── redcap_export_raw.csv
│   │   └── nhanes/
│   │       ├── nhanes_food_security_raw.csv
│   │       └── nhanes_hba1c_raw.csv
│   └── processed/
│       ├── redcap_export_cleaned.csv
│       ├── nhanes_analysis_ready.csv
│       └── irr_double_coded_10papers.csv
│
├── papers/
│   ├── 6item/
│   │   └── paper_001_Myers_2019.pdf
│   └── 10_18item/
│       └── paper_031_Leung_2015.pdf
│
├── R/
│   ├── 01_data_cleaning.R
│   ├── 02_irr_kappa.R
│   ├── 03_audit_analysis.R
│   ├── 04_nhanes_prep.R
│   ├── 05_nhanes_models.R
│   └── utils/
│       ├── score_usda_module.R
│       └── plot_theme.R
│
├── python/
│   ├── qa_checker/
│   │   ├── main.py
│   │   ├── extractor.py
│   │   ├── detectors.py
│   │   ├── conflict_engine.py
│   │   └── report_generator.py
│   ├── requirements.txt
│   └── tests/
│       └── test_detectors.py
│
├── outputs/
│   ├── figures/
│   │   ├── fig1_failure_rates_overall.png
│   │   ├── fig2_failure_rates_by_module.png
│   │   ├── fig3_failure_rates_by_method.png
│   │   ├── fig4_upset_cooccurrence.png
│   │   └── fig5_nhanes_model_comparison.png
│   ├── tables/
│   │   ├── table1_sample_characteristics.csv
│   │   ├── table2_failure_rates_overall.csv
│   │   ├── table3_failure_rates_by_method.csv
│   │   ├── table4_failure_rates_by_module.csv
│   │   └── table5_nhanes_model_results.csv
│   ├── qa_reports/
│   │   └── discrepancy_report_batch1.csv
│   └── cheatsheet/
│       └── practitioner_flowchart.pdf
│
└── manuscript/
    ├── draft_v1.Rmd
    ├── references.bib
    └── journal_submission/
        └── cover_letter.docx
```

---

## Workflow

### Phase 1 — Literature Audit

**Step 1 — Pre-register on OSF** (before opening any paper)
Lock the coding protocol, variable definitions, and decision rules. This prevents peer reviewers from accusing the team of outcome-switching.

**Step 2 — Build REDCap instrument**
One entry per paper using the 18-column tracking sheet. See `/codebook/coding_manual.md` for allowed values.

**Step 3 — Search and collect papers**
Search Google Scholar and PubMed using: "food security," "USDA food security module," "HFSSM," "food insecurity" paired with health outcome terms. Screen in reverse chronological order until 30 eligible 6-item papers and 30 eligible 10/18-item papers are confirmed.

**Step 4 — IRR calibration**
Select 10–12 papers (20%) for double-coding before full data collection begins. Two coders work independently. Calculate Cohen's Kappa per variable (target κ ≥ 0.70). Reconcile disagreements, clarify coding rules, and re-code until threshold is met.

**Step 5 — Full coding of all 60 papers**
Enter all variables into REDCap. Project lead spot-checks entries periodically. Flag ambiguous cases using the `notes` column — do not guess.

### Phase 2 — Data Cleaning and Audit Analysis (R)

**Step 6** (`01_data_cleaning.R`) — Export and clean REDCap data; verify 30/30 stratum balance; flag any dataset exceeding the 8-paper limit.

**Step 7** (`03_audit_analysis.R`) — Calculate Error Burden Score (0–6), Transparency Index (0–5), and failure mode co-occurrence upset plot.

**Step 8** — Report failure mode frequencies overall and stratified by module version and statistical method.

### Phase 3 — NHANES Empirical Demonstration (R)

**Step 9** (`04_nhanes_prep.R`) — Download food security module + cardiometabolic outcome. Apply complex survey weights throughout.

**Step 10** (`05_nhanes_models.R`) — Run four parallel regression models:

| Model | Implementation | Purpose |
|---|---|---|
| **Model 1** — Gold Standard | 3-level ordinal, valid skips coded as 0 | Correct baseline |
| **Model 2** — Scale Compression | Collapsed to binary secure/insecure | Shows power loss and severity masking |
| **Model 3** — Misclassification | Binary, marginal → insecure bucket | Shows inflated effect size |
| **Model 4** — Pipeline Failure | Valid skips treated as `NA` → listwise deletion | Shows demographic skew and sample loss |

### Phase 4 — Manuscript and Cheatsheet

**Step 11** (`manuscript/draft_v1.Rmd`) — Draft manuscript with Introduction, Methods, Results, Discussion.

**Step 12** — Build one-page practitioner decision-tree flowchart and publish as supplementary file.

---

## Variables

The tracking sheet contains **18 columns** in five groups. Estimated coding time: **10–15 minutes per paper**.

### Admin
| Variable | Allowed Values |
|---|---|
| `paper_id` | Integer 1–60 |
| `coder_initials` | String e.g. BJS |

### Identification
| Variable | Allowed Values |
|---|---|
| `paper_title` | Free text |
| `module_version` | 6_Item / 10_Item / 18_Item / Unclear |
| `dataset_name` | NHANES / BRFSS / NHIS / Other |
| `statistical_method` | Logistic / Linear / Hazard / ANOVA / Other |

### Outcome Context
| Variable | Allowed Values |
|---|---|
| `primary_outcome_type` | Objective_Biomarker / Subjective_SelfReport |
| `health_outcome_label` | Free text e.g. HbA1c, Depression |

### Evidence Quotes (copy-paste — prevent coder subjectivity)
| Variable | Purpose |
|---|---|
| `quote_ref_period` | Evidence base for temporal misalignment coding |
| `quote_skip_logic` | Evidence base for skip-logic failure coding |
| `quote_categorization` | Evidence base for scale compression / misclassification coding |

### Core Failure Modes (primary outcomes)
| Variable | Allowed Values |
|---|---|
| `scale_compression` | Yes_Binary / No_Ordinal |
| `category_misclass` | Lumped_Secure / Lumped_Insecure / Dropped / NA_Not_Compressed |
| `temporal_misalign` | Yes_Mismatch / No_Aligned / Unclear |
| `skip_logic_failure` | Yes_Error / No_Correct / Unclear |

### Open Science and QC
| Variable | Allowed Values |
|---|---|
| `code_available` | Yes / No |
| `status` | Checked / Coded / Being_Vetted / Not_Reviewed |
| `notes` | Free text |

### Derived Variables (calculated in R — not coded manually)

**Error Burden Score** (range 0–6): Sum of confirmed failure mode indicators. Codes of `Unclear` and `Not_Described` are treated as 0 in the primary analysis. Sensitivity analyses will exclude unclear cases.

**Transparency Index** (range 0–5): Sum of reporting completeness indicators — ref_period_reported, cutoff_values_stated, skip_logic_reported, official_manual_cited, code_available.

> **Important distinction:** A lack of reporting is **not** assumed to indicate an error. The Transparency Index captures reporting completeness as a separate dimension from confirmed implementation errors.

---

## Analysis Plan

### Descriptive Analysis
- Proportion of studies exhibiting each failure mode
- Distribution of reporting practices (Transparency Index)
- Differences by module type (6-item vs. 10/18-item)

### Primary Comparisons
- **Error Burden Score:** Independent samples t-test; Mann-Whitney U if distributional assumptions are violated
- **Individual failure modes:** Chi-square tests or Fisher's exact test when expected cell counts < 5

Given the number of comparisons, all analyses are interpreted as **descriptive rather than confirmatory**. No correction for multiple comparisons will be applied.

### Decision Rules for Coding
- When multiple models are reported → code the **primary model**
- If no primary model is identified → code the **first model in the main results table**
- If multiple operationalizations are equally emphasized → code the **first one presented**

---

## NHANES Demonstration

NHANES data will demonstrate how different implementation choices affect statistical estimates. All models will apply appropriate complex survey weights and predict a cardiometabolic outcome (e.g., HbA1c). The goal is a clean, reproducible demonstration — not complex modeling. Complete specifications will be reported in supplementary materials.

---

## Inter-Rater Reliability Protocol

- **20% of sample** (10–12 papers) double-coded independently before full data collection
- Cohen's Kappa calculated per variable (`02_irr_kappa.R`)
- Minimum threshold: **κ ≥ 0.70** for all core failure mode variables
- If threshold not met: reconcile → clarify manual → re-code additional batch → repeat
- All IRR results reported in the manuscript methods section

---

## Python QA Checker (Optional Add-On)

After human coders finish a batch, an automated script runs a secondary pass hunting for hard keywords and flags conflicts between human codes and script detections.

| Variable | Automatable? |
|---|---|
| Reference period (30-day / 12-month) | ✓ Yes |
| Module version (6-item / 18-item) | ✓ Yes |
| Code availability (github.com, osf.io) | ✓ Yes |
| Official manual citation | ✓ Yes |
| Scale compression | ✓ Partial |
| Skip-logic handling | ✓ Partial |
| Temporal misalignment | ✗ No — requires judgment |
| Category misclassification | ✓ Partial |

The script **never overwrites human codes**. It only flags discrepancies for coder review. This add-on is strictly secondary — the audit proceeds independently and will not be delayed for it.

---

## File Naming Convention

```
paper_###_Author_Year.pdf
```

Examples: `paper_001_Myers_2019.pdf` | `paper_031_Leung_2015.pdf`

Store in `/papers/6item/` or `/papers/10_18item/` based on module version confirmed in the Methods section.

---

## Key Outputs

| Deliverable | Description | Location |
|---|---|---|
| Audit manuscript | Peer-reviewed methodology paper | `manuscript/` |
| Practitioner cheatsheet | One-page visual decision-tree | `outputs/cheatsheet/` |
| OSF pre-registration | Locked protocol | `pre-registration/` |
| Open R scripts | All analysis code — fully reproducible | `R/` |
| Python QA checker | Automated keyword detection | `python/qa_checker/` |

---

## Contributing and Authorship

Student contributors are responsible for systematic coding (~10–15 min per paper), R analysis, and design of the cheatsheet and manuscript figures.

**All contributors who complete meaningful coding, analysis, or writing will be listed as co-authors.**

**Resume bullets from this project:**
- Conducted a structured QA literature audit of ~60 peer-reviewed publications to detect and quantify measurement failure modes in public health metrics
- Performed technical gap analysis on food security data pipelines, identifying rates of scale compression, category misclassification, and skip-logic failure in applied research
- Calculated inter-rater reliability (Cohen's Kappa) and built parallel regression models in R demonstrating the statistical impact of measurement implementation errors

---

## Citation

> Spitzer B, Sewell C, Maxey N, Lu T, Maloney J. *Measurement Failures in the USDA Household Food Security Survey Module: A Structured Audit of Applied Health Research.* [Journal TBD, Year TBD].

---

## License

MIT License. See `LICENSE` for details.

**Contact:** Brian Spitzer | NYU School of Global Public Health Email: bjs419@nyu.edu
