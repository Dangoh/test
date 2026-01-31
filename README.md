# Sensityping Metrics 

**Sensityping Metrics** is a command-line evaluation framework for assessing **genotype-based antimicrobial susceptibility predictions** against phenotypic treatment outcomes.

It is designed for **gonococcal (and similar) AMR pipelines** where:

- Predictions are made **per antibiotic**
- Clinical decision-making follows **sequential treatment logic**
- Evaluation requires **confidence intervals**, **sample-size diagnostics**, and **policy-relevant metrics**

The script supports both **model-centric** and **clinical-workflow–centric** evaluation, which is why it is intentionally run in **two complementary modes**.

---

## Key Concepts

### Two fundamentally different evaluation questions

| Question | Analysis mode |
|--------|--------------|
| How accurate is the prediction for each antibiotic? | `predicted_vs_treatment` |
| How well does the final treatment recommendation perform in practice? | `first_line_vs_treatment` |

These cannot be answered in a single analysis without mixing incompatible assumptions.

---

## Installation

The script is standalone and requires only standard scientific Python:

- python >= 3.6  
- numpy  
- pandas  
- scipy  
- scikit-learn  
- plotly  

Clone the repository:

```bash
git clone https://github.com/your-org/sensityping-metrics.git
cd sensityping-metrics
```

---

## Input format

Input is a **tab-delimited table** containing, per isolate:

- Predicted susceptibility (`*_predicted`)
- Observed treatment outcome (`*_treatment`)
- Optional recommendation columns (`*_recommend`)

### Example column pattern

```text
isolate_id
CRO_predicted
CRO_treatment
AZM_predicted
AZM_treatment
CRO+AZM_recommend
CRO+AZM_treatment
```

Combo tokens (`CRO+AZM`, `AZM+SPC`, etc.) are supported **only if present as explicit columns**.

---

## Analysis modes

### 1. `predicted_vs_treatment`
**Model-centric evaluation**

This mode evaluates **each antibiotic independently**, ignoring treatment sequencing.

Use this to answer:

> *Does the genomic predictor correctly classify susceptibility for this drug?*

Metrics are computed **per antibiotic**:

- TP / TN / FP / FN  
- Sensitivity, specificity  
- PPV, NPV  
- F1, MCC  
- AUC  
- Coverage fraction  
- False discovery control metrics  

#### Example

```bash
python sensityping_metrics.py \
  -i example_data.tab \
  -o example_predict.tab \
  -d example_plots \
  --analysis_type predicted_vs_treatment \
  --ci_flag \
  --ci_method hybrid \
  --ci_level 0.95 \
  --n_boot 2000 \
  --seed 1 \
  --ssd_flag \
  --ssd_width 0.05 \
  --ssd_mode observed \
  --radar_flag \
  --radar_metrics PPV,one_minus_FDR,coverage_fraction
```

Typical output:

```text
TP: 23, TN: 5, FP: 0, FN: 0
TP: 22, TN: 6, FP: 0, FN: 0
TP: 12, TN: 16, FP: 0, FN: 0
...
```

---

### 2. `first_line_vs_treatment`
**Clinical workflow evaluation**

This mode evaluates **what actually matters clinically**:

> *Did the recommended first-line treatment work?*

Key characteristics:

- Treatments are applied **sequentially**
- Once a recommendation is made, **downstream drugs are ignored**
- **Ordering matters**
- Reflects **guideline-based decision logic**

You must explicitly define:

- The **order** of treatment consideration
- Which recommendation tokens to extract

#### Example

```bash
python sensityping_metrics.py \
  -i example_data.tab \
  -o example.tab \
  -d example_out \
  --analysis_type first_line_vs_treatment \
  --order 'CRO+AZM,CRO,AZM,AZM+SPC,SPC,ZOL' \
  --id_extraction CRO+AZM,CRO,AZM,AZM+SPC,SPC,ZOL \
  --ci_flag \
  --ci_method hybrid \
  --ci_level 0.95 \
  --n_boot 2000 \
  --seed 1 \
  --ssd_flag \
  --ssd_width 0.05 \
  --ssd_mode observed \
  --radar_flag \
  --radar_metrics PPV,one_minus_FDR,coverage_fraction
```

Output:

```text
TP: 18, TN: 10, FP: 0, FN: 0
[ID extraction] Saved: example_out/CRO+AZM_id_extracted.tab
TP: 5, TN: 5, FP: 0, FN: 0
[ID extraction] Saved: example_out/CRO_id_extracted.tab
...
```

Each extracted file contains the isolate IDs contributing to that decision.

---

## Why the two analyses are **both required**

| Aspect | predicted_vs_treatment | first_line_vs_treatment |
|------|------------------------|-------------------------|
| Evaluates model correctness | ✅ | ❌ |
| Evaluates clinical outcome | ❌ | ✅ |
| Per-antibiotic performance | ✅ | ❌ |
| Sequential decision logic | ❌ | ✅ |
| Reflects guideline practice | ❌ | ✅ |
| Suitable for methods papers | ✅ | ❌ |
| Suitable for policy / implementation | ❌ | ✅ |

Running only one would give a **biased or incomplete interpretation**.

---

## Confidence intervals

Enable with:

```bash
--ci_flag
```

### CI methods

- **Wilson score**  
  Used for proportions (PPV, NPV, sensitivity, specificity)

- **Bootstrap**  
  Used for composite metrics (F1, MCC, kappa, balanced accuracy, AUC)

Hybrid mode automatically selects the correct approach per metric.

---

## Sample size diagnostics (SSD)

Enable with:

```bash
--ssd_flag
```

Defines how many isolates are required to achieve a CI half-width of ±w:

```bash
--ssd_width 0.05
--ssd_mode observed
```

Useful for:

- Surveillance planning  
- Power justification  
- Manuscript methods sections  

---

## Radar plots

Enable with:

```bash
--radar_flag
```

Select metrics explicitly:

```bash
--radar_metrics PPV,one_minus_FDR,coverage_fraction
```

Output is an **interactive Plotly HTML**, suitable for:

- Supplementary material  
- Internal dashboards  
- Presentations  

---

## Output structure

```text
example_out/
├── metrics_summary.tab
├── CRO+AZM_id_extracted.tab
├── CRO_id_extracted.tab
├── AZM_id_extracted.tab
└── radar_plot.html
```

---

## Intended use

This tool is designed for:

- Genomic AMR prediction pipelines  
- Rule-based or ML-based susceptibility systems  
- Surveillance frameworks (WHO / EUCAST-style)  
- Manuscripts requiring **transparent, reproducible evaluation**

---

## Citation

If you use this framework in a manuscript, please cite the corresponding **Sensityping / SensiTyper** publication and reference this repository.
