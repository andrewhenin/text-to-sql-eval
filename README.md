# Text-to-SQL Evaluation: Flat (1NF‑compliant) vs Star Schema — Data & Results

Reproducibility package for **“Evaluating Large Language Models for text‑to‑SQL tasks: Comparing First Normal Form and Star Schema.”**  
This repository hosts the **experiment CSVs** and **metadata** used to evaluate multiple LLMs on a business query across two database designs:

- **Flat (1NF‑compliant) single table**  
- **Star schema (dimensional model)**

> This repo contains only data & metadata. Analysis code is optional and provided as a helper script in `analysis/replicate_statistics.py`.

---

## Contents

| File | Purpose |
|---|---|
| `LLM Data Collection - Query Results Try 1.csv` | Row‑level outcomes for each attempt (per LLM × schema × repeat). Includes generated SQL, validity, exact match, runtime, and semantic‑F1 fields. |
| `LLM Data Collection - LLM Rankings Try 1.csv` | Aggregated metrics per LLM and schema (Avg. Exact‑Match %, Validity %, Avg. Runtime). |
| `LLM Data Collection - Experiment Metadata Try 1.csv` | Prompts and **ground‑truth SQL** for each schema, plus validation notes (minor column typo noted below). |

> **Runs:** 6 LLMs × 2 schemas × 5 attempts = **60 attempts** (plus one final summary row in the results file).

---

## Metrics

We report four primary metrics:

* **Query Validity (Yes/No)** — whether the generated SQL executed without error and returned an expected result table.
* **Exact Match (1/0)** — whether the generated SQL **text** exactly matches the ground‑truth SQL.
* **Semantic Accuracy (F1)** — row‑level agreement between ground‑truth (GT) and generated query (GQ) results:

  * TP: rows in both GQ and GT
  * FP: rows in GQ but not in GT
  * FN: rows in GT but not in GQ
  * Precision = TP / (TP + FP), Recall = TP / (TP + FN), F1 = 2·P·R / (P + R)
* **Execution Time (ms)** — wall‑clock time to produce a result.

> In this benchmark, valid queries typically achieve **F1 = 1.0** for the task; invalid queries have undefined F1.

---

## Ground‑Truth SQL (from Metadata)

**Flat (1NF‑compliant) single table**

```sql
SELECT 
    store_location,
    product_type,
    SUM(transaction_qty * unit_price) AS total_revenue
FROM 
    dbo.Sales
WHERE 
    product_category = 'Coffee'
    AND transaction_date BETWEEN '01/01/2023' AND '01/31/2023'
    AND CAST(transaction_time AS TIME) BETWEEN '06:00:00' AND '12:00:00'
GROUP BY 
    store_location,
    product_type
```

**Star schema (dimensional)**

```sql
SELECT 
    ds.store_location,
    dp.product_type,
    SUM(fs.revenue) AS total_revenue
FROM 
    dbo.Fact_Sales fs
INNER JOIN 
    dbo.Dim_Store ds ON fs.store_id = ds.store_id
INNER JOIN 
    dbo.Dim_Product dp ON fs.product_id = dp.product_id
INNER JOIN 
    dbo.Dim_Date dd ON fs.date_id = dd.date_id
INNER JOIN 
    dbo.Dim_Time dt ON fs.time_id = dt.time_id
WHERE 
    dd.month = 'January' AND dd.year = 2023
    AND dt.period_of_day = 'Morning'
    AND dp.product_category = 'Coffee'
GROUP BY 
    ds.store_location, dp.product_type
ORDER BY 
    ds.store_location, dp.product_type;
```

---

## High‑Level Results (from CSVs)

* **Attempts:** 60 (6 LLMs × 2 schemas × 5 repeats)
* **Validity:** Flat = **30/30**, Star = **26/30**
* **Exact Match:** Equals validity for this task (i.e., all valid queries were exact textual matches)
* **Observation:** One model shows reduced validity on star schema (**1/5**), others are **5/5** across both schemas.

## How to Cite

If you reference this repo or data, please cite the paper:

> **Andrew Henin and Péter Ekler.** *Evaluating Large Language Models for text‑to‑SQL tasks: Comparing First Normal Form and Star Schema.* IEEE CogInfoCom 2025 (under review).

Or cite the repository:

```bibtex
@misc{henin2025_text_to_sql_eval,
  author       = {Andrew Henin and P{\'e}ter Ekler},
  title        = {Text-to-SQL Evaluation: Flat (1NF‑compliant) vs Star Schema — Data \& Results},
  year         = {2025},
  howpublished = {\url{https://github.com/andrewhenin/text-to-sql-eval}},
  note         = {Reproducibility dataset and experiment metadata}
}
```

---

## Contact

**Andrew Henin** · [andrewhenin25@gmail.com](mailto:andrewhenin25@gmail.com)
(Independent Researcher)
