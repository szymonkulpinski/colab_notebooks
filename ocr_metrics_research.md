# OCR Evaluation Metrics Research

Research compiled for the GLM-OCR vs PaddleOCR-VL-1.5 benchmark notebook.

---

## Metrics Used in This Benchmark

For each of the 8 datasets we use a task-appropriate combination:

| Dataset | Primary | Secondary | Notes |
|---------|---------|-----------|-------|
| CAPTCHA | Exact Match | CER | Short alphanumeric, high precision needed |
| LaTeX equations | NED | CER | CDM is the emerging standard but requires LaTeX rendering; NED is practical |
| Receipt (total field) | Exact Match | NED | Numeric field; comparing total value only |
| Date stamps | Exact Match | CER | Structured YYYY-MM-DD format |
| Jersey numbers | Exact Match | CER | Short numeric string |
| Container serials | Exact Match | CER | Alphanumeric serial codes |
| Tire codes | Exact Match | CER | Embossed alphanumeric codes |
| License plates | Exact Match | CER | Alphanumeric plates |

---

## Metric Definitions

### Exact Match Accuracy (EMA)
```
EMA = number of exact matches / total samples
```
- Range: 0–1 (higher is better)
- Zero tolerance: prediction must be character-perfect
- Best for: short codes, structured strings, license plates, dates

### Character Error Rate (CER)
```
CER = (Substitutions + Deletions + Insertions) / Total characters in reference
```
- Range: 0–∞ (lower is better; can exceed 1.0 for very wrong predictions)
- Based on Levenshtein (edit) distance at character level
- Best for: capturing near-misses; 1 wrong char in a 6-char code = 16.7% CER

### Normalized Edit Distance (NED)
```
NED = Levenshtein(pred, ref) / max(len(pred), len(ref))
```
- Range: 0–1 (lower is better)
- Length-normalized version of edit distance
- Used by OmniDocBench as primary text metric
- Best for: LaTeX and variable-length outputs

### ANLS (Average Normalized Levenshtein Similarity) — not used here but notable
```
s = 1 - NED   if NED < 0.5,  else 0
ANLS = mean(s) over all samples
```
- Range: 0–1 (higher is better)
- Standard for DocVQA / TextVQA
- Threshold at 0.5 prevents low-quality matches from inflating scores

---

## How Major Benchmarks Evaluate OCR

### OmniDocBench (CVPR 2025)
- Component-specific metrics, not a single score
- **Text**: Normalized Edit Distance (NED)
- **Tables**: TEDS (Tree-Edit-Distance-based Similarity) + NED per cell
- **Formulas**: CDM (Character Detection Matching) — image-based, renders both pred and GT
- **Reading order**: NED
- **Overall score**: `((1 - TextED) × 100 + TableTEDS + FormulaCDM) / 3`

### OCRBench v2 (2025)
- 31 scenarios, task-specific metrics
- Parsing: TEDS
- Localization: IoU, mAP@[0.5:0.95]
- Extraction: F1 Score
- VQA: Exact String Matching

### DocVQA / TextVQA
- Primary: **ANLS** (tolerates OCR imperfections with partial credit)
- Case-insensitive

### FUNSD (Form Understanding)
- Precision, Recall, F1 based on Levenshtein matching

### SROIE (Scanned Receipt OCR)
- F1 Score (primary), Precision, Recall
- mAP for spatial localization

---

## Metric Selection Rationale for This Benchmark

The 8 datasets in this benchmark are all **short, structured outputs** (codes, numbers, dates, formulas). This makes:

- **Exact Match** the most meaningful primary metric — did the model get it right?
- **CER** the best secondary metric — how far off was a wrong prediction?
- **NED** useful for LaTeX where notation can vary (e.g., `\frac{a}{b}` vs `\frac{a+0}{b}`)

We deliberately avoid:
- **BLEU**: too coarse for character-level errors
- **WER**: most outputs are single tokens or codes, so word-level doesn't add information
- **CDM**: requires LaTeX rendering infrastructure; NED is a reasonable proxy

---

## Notes on Receipt Parsing

The receipt dataset uses a structured JSON extraction prompt. For simplicity, we extract
and compare only the **total** field (a single numeric value). This tests whether the model
can:
1. Locate the total amount in the receipt image
2. Read it correctly (potentially through noise, stamps, handwriting)

This is a deliberate simplification. A more complete evaluation would compare all JSON
fields (merchant name, items, tax, etc.) using field-level F1. The limitation is noted
in the notebook.

---

## References

- [OmniDocBench GitHub](https://github.com/opendatalab/OmniDocBench)
- [OCRBench v2 — arXiv:2501.00321](https://arxiv.org/abs/2501.00321)
- [ANLS Universal Metric — arXiv:2402.03848](https://arxiv.org/html/2402.03848v2)
- [CDM Formula Metric — arXiv:2409.03643](https://arxiv.org/abs/2409.03643)
- [TEDS Table Metric — arXiv:1911.10683](https://arxiv.org/abs/1911.10683)
- [SROIE Tasks — RRC](https://rrc.cvc.uab.es/?ch=13&com=tasks)
