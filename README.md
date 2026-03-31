# USDA Food Security Survey Module — Measurement Audit

> **A structured QA audit of how applied researchers implement the USDA Household Food Security Survey Module (HFSSM) in peer-reviewed health research.**

---

## Why This Project Exists

Measurement issues often go undetected until after publication, forcing analysts to explain away unusual results. This project exists to:

- **Quantify the problem** — Move beyond anecdotes and show exactly how widespread misuse is across the literature
- **Improve upstream research quality** — Stop bad data pipelines before they happen
- **Create decision support tools** — Give researchers a simple flowchart they can use at the design stage
- **Teach audit skills** — Model how to critique and quality-check a public health measurement tool

---

## Project Structure

```
def detect_statistical_method(text):
    if any(s in text for s in ["logistic regression", "odds ratio", "or ="]):
        return "Logistic regression"
    elif any(s in text for s in ["linear regression", "beta coefficient", 
                                  "linear mixed"]):
        return "Linear regression"
    elif any(s in text for s in ["analysis of variance", "anova", 
                                  "one-way anova"]):
        return "ANOVA"
    elif any(s in text for s in ["cox", "hazard ratio", "survival analysis", 
                                  "time to event"]):
        return "Hazard model"
    else:
        return "Other/not detected"
```

---

### Step 6 — Update Folder Structure
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

## The Four Failure Modes We Are Auditing

| # | Failure Mode | Description |
|---|---|---|
| 1 | **Scale Compression** | Collapsing the 3-level ordinal scale into a blunt binary (Secure vs. Insecure), discarding severity information |
| 2 | **Category Misclassification** | Mishandling the "Marginal" food security group — absorbed into the wrong category without justification |
| 3 | **Temporal Misalignment** | Using a 30-day or 12-month recall module without matching it to the timeframe of the health outcome |
| 4 | **Data Pipeline Failure** | Treating valid USDA skip-logic responses as missing data, accidentally deleting the healthiest control group |

---

## Sample Design

| Stratum | Target N | Module Version |
|---|---|---|
| Short Form | 30 papers | USDA 6-Item Short Form |
| Full Module | 30 papers | USDA 10-Item or 18-Item |
| **Total** | **60 papers** | |

**Dataset diversity rule:** No more than 8 papers from the same survey dataset (NHANES, BRFSS, NHIS, etc.)

---

## Inclusion & Exclusion Criteria

### Include if ALL of the following are true:
- Measures food security using a USDA module (6-, 10-, or 18-item)
- Uses multivariable statistical analysis (linear regression, logistic regression, hazard models, ANOVA, or equivalent)
- Food security is used as a **predictor** for a health-related outcome (chronic disease, mental health, mortality, BMI/obesity, healthcare utilization, or diet quality)
- Published in a peer-reviewed academic journal
- Written in English

### Exclude if ANY of the following are true:
- Purely descriptive (prevalence only, no statistical model)
- Qualitative (interviews, focus groups, ethnography)
- Scale validation study (main goal is redesigning or testing the USDA measure)
- Uses a non-USDA tool (FIES, HFIAS, Radimer/Cornell, single custom items)
- Secondary literature (systematic reviews, meta-analyses, editorials, commentaries)

---

## Workflow

### Phase 1 — Literature Audit

1. **Pre-register** protocol on OSF before opening any paper
2. **Build REDCap instrument** from the coding manual variable list
3. **IRR calibration** — two coders independently code 10 papers; calculate Cohen's Kappa (target κ ≥ 0.80)
4. **Full coding** — all 60 papers entered into REDCap

### Phase 2 — Data Cleaning & Audit Analysis (R)

5. Export and clean REDCap data (`01_data_cleaning.R`)
6. Calculate derived variables: Error Burden Score (0–5), failure mode co-occurrence (`03_audit_analysis.R`)
7. Report failure mode frequencies overall and stratified by module version

### Phase 3 — NHANES Empirical Demonstration (R)

8. Download and prepare NHANES food security + HbA1c data (`04_nhanes_prep.R`)
9. Run four parallel regression models (`05_nhanes_models.R`):

| Model | Implementation | Purpose |
|---|---|---|
| **Model 1** — Gold Standard | 3-level ordinal, valid skips coded as 0 | Correct baseline |
| **Model 2** — Scale Compression | Collapsed to binary | Shows power loss |
| **Model 3** — Misclassification | Binary, marginal → insecure bucket | Shows inflated effect size |
| **Model 4** — Pipeline Failure | Valid skips treated as `NA` (listwise deletion) | Shows demographic skew |

### Phase 4 — Manuscript & Cheatsheet

10. Draft manuscript in R Markdown
11. Build one-page practitioner flowchart (PDF)
12. Submit to peer-reviewed public health or applied statistics journal

---

## Key Variables Coded Per Paper

### Bucket 1 — Core Failure Modes
| Variable | Categories |
|---|---|
| Scale compression | Yes / No |
| Marginal category handling | Lumped with secure / Lumped with insecure / Excluded / Not applicable |
| Temporal misalignment | Aligned / Misaligned / Cannot determine |
| Skip-logic handling | Evidence of error / No evidence / Cannot determine |
| Level-of-analysis mismatch | Household acknowledged / Incorrectly individual / Unclear |

### Bucket 2 — Contextual Variables
| Variable | Categories |
|---|---|
| Reference period reported | 30-day / 12-month / Unclear |
| Scoring method reported | Standard USDA / Modified / Unclear |
| Code availability | Yes / No |
| Official manual citation | Official manual / Other paper / None |
| Geographic context | US or Canada / International |
| Sub-population | General population / High-risk or clinical |
| Journal domain | Nutrition / Public Health / Clinical / Social Science / Other |
| Primary outcome type | Objective (biomarker) / Subjective (self-report) |
| Item modification | Yes / No |

### Bucket 3 — Derived in R (not coded manually)
- **Error Burden Score** — sum of failure mode indicators (0–5)
- **Failure Mode Co-occurrence** — upset plot of error combinations

---

## File Naming Convention

All PDFs follow the format:

```
paper_###_Author_Year.pdf
```

Examples: `paper_001_Myers_2019.pdf`, `paper_047_Leung_2015.pdf`

---

## Inter-Rater Reliability Protocol

- 10 papers (20% of sample) are double-coded independently
- Cohen's Kappa calculated per variable in `02_irr_kappa.R`
- Minimum acceptable threshold: **κ ≥ 0.80**
- Full coding does not begin until threshold is met for all variables

---

## Outputs

| Deliverable | Description |
|---|---|
| Audit manuscript | Peer-reviewed methodology paper with failure mode frequencies and NHANES demonstration |
| Practitioner cheatsheet | One-page visual flowchart for applied researchers |
| OSF pre-registration | Locked protocol preventing outcome-switching accusations |
| Open R code | All analysis scripts in `/R/` — fully reproducible |

---

## Contributing

This project follows modular task assignment. Student contributors are responsible for:

- **Systematic coding** — extracting variables from paper methodology sections into REDCap
- **R analysis** — running cleaning scripts, kappa, audit frequencies, and NHANES models
- **Design** — translating findings into the cheatsheet and manuscript figures

All student contributors who complete meaningful coding, analysis, or writing tasks will be listed as co-authors on the manuscript.

---

## Citation

> [Authors TBD]. *Measurement Failures in the USDA Food Security Survey Module: A Structured Audit of Applied Health Research.* [Journal TBD, Year TBD].

---

## License

This project is licensed under the MIT License. See `LICENSE` for details.

---

## Contact

Project Lead: [Name] — [Email]
Institution: NYU School of Global Public Health
