# Extracting Heart Failure Signals from Unstructured Clinical Notes

**Pulling left ventricular ejection fraction (LVEF) and medication data out of free-text discharge summaries and echocardiogram reports in MIMIC-III, using regular expressions, and testing how far text mining gets you when the structured tables are incomplete.**

R · SQL · MIMIC-III v1.4 (credentialed PhysioNet access) · No patient-level data in this repository

---

## Why this project

Most of what a clinician knows about a patient never reaches a structured database field. Ejection fraction, the single most important number in heart failure, lives in the narrative text of an echocardiogram report, not in a tidy column. If you want to study heart failure at scale, you have to get it out of the prose.

This project does that, and is honest about where it breaks.

---

## What I found

- **Ejection fraction is bimodal.** Two clear populations: one clustered around 20 to 25 percent (reduced ejection fraction, HFrEF) and one around 55 percent (preserved ejection fraction, HFpEF). This is not a statistical curiosity. It is the clinical reality that heart failure is two diseases wearing one name, and it falls straight out of the extracted text.
- **The raw extraction contained a physiologically impossible value: an ejection fraction above 150 percent.** A heart cannot eject more blood than it contains. This is a documentation or parsing artefact, and modelling straight through it would have silently corrupted every downstream summary. Finding it is the reason the "with outliers" and "without outliers" figures are both published here.
- **Regex works well on formulaic text and fails on prose.** Medication extraction succeeds where discharge summaries follow the standard `drug / dose / unit` pattern and degrades on compound formulations, free-text instructions, and non-standard abbreviations. The notebook documents the failure cases rather than reporting only the successes.
- **Text-derived medications do not fully agree with the structured PRESCRIPTIONS table.** Neither source is a clean gold standard. The discrepancies are documented, because in real-world evidence work the disagreement between sources is usually the finding.

---

## What I did

1. **Comorbidity analysis.** Queried co-occurring diagnoses around heart failure and interpreted the patterns clinically rather than just tabulating them.
2. **Cohort construction (SQL).** Built a cohort of admissions where heart failure is the *primary* diagnosis, not merely present, so that the analysis is about heart failure patients rather than patients who happen to have it.
3. **Medication extraction (regex).** Parsed drug, dose and unit from discharge summary text into a structured data frame, then benchmarked the result against the structured `PRESCRIPTIONS` table for the same admission.
4. **LVEF extraction (regex).** Handled heterogeneous documentation: ejection fraction is written a dozen different ways across reports. Aggregated multiple measurements per admission (`HADM_ID`) to a single mean value.
5. **Quality control and analysis.** Identified and removed physiologically impossible values, then described the LVEF distribution overall and stratified by sex and hypertension.

---

## Figures

| Figure | What it shows |
|---|---|
| LVEF distribution, with outliers | The data quality problem. An ejection fraction above 150 percent. |
| LVEF distribution, without outliers | The clean bimodal distribution: HFrEF and HFpEF. |
| LVEF by sex | Women skew toward preserved ejection fraction, men toward reduced. Consistent with the published epidemiology. |
| LVEF by hypertension | Hypertensive patients skew toward preserved ejection fraction. Consistent with hypertension driving HFpEF. |

That the extracted data reproduces known clinical epidemiology is the best available evidence that the regex extraction is working.

---

## Limitations

- **Regex is brittle.** It captures the common documentation patterns and misses the unusual ones. Extraction is therefore not complete, and completeness is not uniform across note types. A production system would need a proper clinical natural language processing (NLP) model.
- **Averaging LVEF per admission loses information.** A patient whose ejection fraction moves during an admission is reduced to a single number.
- **Single-centre data**, one US academic hospital, 2001 to 2012. Documentation conventions elsewhere will differ, and the regex patterns will not transfer without retuning.

---

## Data and privacy

MIMIC-III v1.4, accessed under a credentialed PhysioNet Data Use Agreement.

**No patient-level data, raw or derived, is contained in this repository.** All notebook outputs have been cleared. Published figures are aggregate distributions only. Database credentials are read at runtime from a local `creds.txt`, which is excluded from version control.

To reproduce: complete CITI training, obtain credentialed access via [PhysioNet](https://physionet.org/content/mimiciii/), load MIMIC-III locally, and supply your own connection details.

---

## Repository contents

```
├── Notebooks/   MIMIC-III_HeartFailure.ipynb  (R, outputs cleared)
├── figures/     Aggregate LVEF distributions
├── LICENSE
└── README.md
```

---

## Skills demonstrated

Clinical text mining · Regular expressions on unstructured electronic health record (EHR) notes · SQL cohort construction · Data quality forensics · Real-world data (RWD) · Clinical interpretation · R- **The raw extraction contained a physiologically impossible value: an ejection fraction above 150 percent.** A heart cannot eject more blood than it contains. This is a documentation or parsing artefact, and modelling straight through it would have silently corrupted every downstream summary. Finding it is the reason the "with outliers" and "without outliers" figures are both published here.
- **Regex works well on formulaic text and fails on prose.** Medication extraction succeeds where discharge summaries follow the standard `drug / dose / unit` pattern and degrades on compound formulations, free-text instructions, and non-standard abbreviations. The notebook documents the failure cases rather than reporting only the successes.
- **Text-derived medications do not fully agree with the structured PRESCRIPTIONS table.** Neither source is a clean gold standard. The discrepancies are documented, because in real-world evidence work the disagreement between sources is usually the finding.

<!-- TO FILL: replace with your real numbers before publishing -->
<!-- - Discharge summaries and echocardiogram reports parsed: N -->
<!-- - LVEF values extracted: N -->
<!-- - Heart failure admissions in the final cohort: N -->
<!-- - Implausible LVEF values removed: N -->

---

## What I did

<!-- TO FILL: state solo or group. If group, name your stage. Do not leave this ambiguous. -->

1. **Comorbidity analysis.** Queried co-occurring diagnoses around heart failure and interpreted the patterns clinically rather than just tabulating them.
2. **Cohort construction (SQL).** Built a cohort of admissions where heart failure is the *primary* diagnosis, not merely present, so that the analysis is about heart failure patients rather than patients who happen to have it.
3. **Medication extraction (regex).** Parsed drug, dose and unit from discharge summary text into a structured data frame, then benchmarked the result against the structured `PRESCRIPTIONS` table for the same admission.
4. **LVEF extraction (regex).** Handled heterogeneous documentation: ejection fraction is written a dozen different ways across reports. Aggregated multiple measurements per admission (`HADM_ID`) to a single mean value.
5. **Quality control and analysis.** Identified and removed physiologically impossible values, then described the LVEF distribution overall and stratified by sex and hypertension.

---

## Figures

| Figure | What it shows |
|---|---|
| LVEF distribution, with outliers | The data quality problem. An ejection fraction above 150 percent. |
| LVEF distribution, without outliers | The clean bimodal distribution: HFrEF and HFpEF. |
| LVEF by sex | Women skew toward preserved ejection fraction, men toward reduced. Consistent with the published epidemiology. |
| LVEF by hypertension | Hypertensive patients skew toward preserved ejection fraction. Consistent with hypertension driving HFpEF. |

That the extracted data reproduces known clinical epidemiology is the best available evidence that the regex extraction is working.

---

## Limitations

- **Regex is brittle.** It captures the common documentation patterns and misses the unusual ones. Extraction is therefore not complete, and completeness is not uniform across note types. A production system would need a proper clinical natural language processing (NLP) model.
- **Averaging LVEF per admission loses information.** A patient whose ejection fraction moves during an admission is reduced to a single number.
- **Single-centre data**, one US academic hospital, 2001 to 2012. Documentation conventions elsewhere will differ, and the regex patterns will not transfer without retuning.

---

## Data and privacy

MIMIC-III v1.4, accessed under a credentialed PhysioNet Data Use Agreement.

**No patient-level data, raw or derived, is contained in this repository.** All notebook outputs have been cleared. Published figures are aggregate distributions only. Database credentials are read at runtime from a local `creds.txt`, which is excluded from version control.

To reproduce: complete CITI training, obtain credentialed access via [PhysioNet](https://physionet.org/content/mimiciii/), load MIMIC-III locally, and supply your own connection details.

---

## Repository contents

```
├── Notebook/    MIMIC-III_HeartFailure.ipynb  (R, outputs cleared)
├── figures/     Aggregate LVEF distributions
└── README.md
```

---

## Skills demonstrated

Clinical text mining · Regular expressions on unstructured electronic health record (EHR) notes · SQL cohort construction · Data quality forensics · Real-world data (RWD) · Clinical interpretation · R
