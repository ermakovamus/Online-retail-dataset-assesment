# Data Card — Online Retail Dataset

A data quality assessment of the UCI *Online Retail* dataset against four KPIs:
**Completeness, Latency, Accuracy, Consistency.** Built as a class assignment on
how to evaluate datasets before using them to train AI models.

- **Analysis date:** 2026-04-26
- **Notebook:** [`data_quality_assessment.ipynb`](data_quality_assessment.ipynb)


---

## 1. Source of data

- **Name:** Online Retail
- **Provider:** UCI Machine Learning Repository (donated by Dr Daqing Chen, London South Bank University)
- **Original URL:** https://archive.ics.uci.edu/ml/datasets/online+retail
- **Reference paper:** Chen, D., Sain, S. L., & Guo, K. (2012). *Data mining for the online retail industry: A case study of RFM model-based customer segmentation using data mining.* Journal of Database Marketing and Customer Strategy Management, 19(3), 197–208.
- **License:** UCI public-research use; cite the provider when redistributing.

### What it contains

A transactional log of every order placed at a UK-based, registered, non-store
online retailer between **01 Dec 2010 and 09 Dec 2011**. The retailer sells
mainly all-occasion gifts, and a large share of its customers are wholesalers.

| Field         | Type    | Description                                              |
|---------------|---------|----------------------------------------------------------|
| `InvoiceNo`   | string  | 6-digit invoice number. Prefix `C` = cancellation.       |
| `StockCode`   | string  | 5–6 character product code.                              |
| `Description` | string  | Product name.                                            |
| `Quantity`    | int     | Units per line. Negative = return.                       |
| `InvoiceDate` | datetime| Timestamp the invoice was generated.                     |
| `UnitPrice`   | float   | Price per unit, in pounds sterling (£).                  |
| `CustomerID`  | float   | 5-digit customer identifier.                             |
| `Country`     | string  | Country where the customer resides.                      |

### Shape and scope

- **Rows:** 541,909
- **Columns:** 8
- **Distinct invoices:** 25,900
- **Distinct customers:** 4,372
- **Distinct stock codes:** 4,070
- **Distinct countries:** 38 (United Kingdom = 91% of rows)
- **Date coverage:** 01 Dec 2010 → 09 Dec 2011 (374 days)

### Intended ML uses for which the dataset is commonly used

Customer segmentation (RFM, k-means), market-basket analysis, sales forecasting
on a closed historical window, and as a teaching dataset for data preparation
and feature engineering.

---

## 2. KPIs

All scores are reported on a 0–100 scale where higher is better.

### (a) Completeness — **96.85%**

> Share of expected cells that are actually populated.
> `Completeness = (1 − missing_cells / total_cells) × 100`

| Column        | Missing rows | Missing % |
|---------------|-------------:|----------:|
| `InvoiceNo`   |            0 |     0.00% |
| `StockCode`   |            0 |     0.00% |
| `Description` |        1,454 |     0.27% |
| `Quantity`    |            0 |     0.00% |
| `InvoiceDate` |            0 |     0.00% |
| `UnitPrice`   |            0 |     0.00% |
| `CustomerID`  |      135,080 |    24.93% |
| `Country`     |            0 |     0.00% |

Two columns drive almost all the missingness. `CustomerID` is null on roughly
one row in four — a serious gap for any user-level model, because customer
identity is the join key. `Description` has trivial nulls.

<img width="795" height="406" alt="Screenshot 2026-04-26 at 16 40 07" src="https://github.com/user-attachments/assets/6a7e6e88-f21d-4d69-903a-a60fd3bccd93" />


### (b) Latency — **freshness score 0/100**

> How current the data is, expressed as the staleness gap between the most
> recent record and today (the analysis run date).

| Metric                                  | Value           |
|-----------------------------------------|-----------------|
| Most recent record (`max InvoiceDate`)  | 2011-12-09      |
| Today (analysis run date)               | 2026-04-26      |
| Freshness gap                           | **5,251 days** (~14.4 years) |
| Median record age                       | 5,394 days      |

The dataset is a frozen historical snapshot. Latency is acceptable for teaching
and methodological benchmarking, but disqualifying for any model that has to
make decisions about current customers, current prices, or current inventory.

### (c) Accuracy — **97.82%**

> Share of rows that respect basic business rules:
> `Quantity > 0 AND UnitPrice > 0 AND InvoiceNo does not start with 'C'`.

| Issue                                | Rows    |
|--------------------------------------|--------:|
| Negative quantity (returns)          | 10,624  |
| Zero quantity                        | 0       |
| Negative unit price                  | 2       |
| Zero unit price                      | 2,515   |
| Cancellation invoices (prefix `C`)   | 9,288   |
| IQR outliers — `Quantity`            | 58,619  |
| IQR outliers — `UnitPrice`           | 39,627  |

The two negative-price rows and the absolute extremes of `Quantity` (±80,995)
and `UnitPrice` (£38,970) deserve manual inspection — they look like manual
adjustments rather than real sales. The IQR outlier counts are reported for
visibility; many of them are valid wholesale orders rather than errors.



### (d) Consistency — **91.31%**

> Internal coherence: same fact represented the same way every time.

| Check                                              | Result               |
|----------------------------------------------------|----------------------|
| Fully duplicated rows                              | 5,268 (1.0%)         |
| `StockCode`s mapped to multiple `Description`s     | 650 / 3,958 (16.4%) |
| Country strings with leading/trailing whitespace   | 0                    |

Two real problems show up here. First, ~1% of the table is exact duplicates and
must be deduped. Second, ~16% of stock codes have more than one description in
the file (typically casing differences, abbreviations, or evolving product
names) — a canonical-description lookup needs to be built before joining on
`StockCode` for any product-level model.

### KPI scorecard



| KPI                | Score   |
|--------------------|--------:|
| Completeness       | 96.85%  |
| Accuracy           | 97.82%  |
| Consistency        | 91.31%  |
| Freshness (latency)| 0/100   |

<img width="763" height="407" alt="Screenshot 2026-04-26 at 16 38 21" src="https://github.com/user-attachments/assets/14a42011-e339-4b30-8ed0-be24825b4776" />

## 2.1 Statistics 

## 2.1.1 Top 10 countries by transaction count 
<img width="851" height="427" alt="Screenshot 2026-04-26 at 16 52 12" src="https://github.com/user-attachments/assets/1227cff3-36b5-48be-8f63-621119e51c40" />

## 2.1.2 Distribution of unit price 

<img width="790" height="427" alt="Screenshot 2026-04-26 at 16 53 42" src="https://github.com/user-attachments/assets/bc85bf44-547f-4c73-bc25-c1a54bc55a1a" />

## 2.1.3 Monthly revenue 

<img width="850" height="427" alt="Screenshot 2026-04-26 at 16 54 40" src="https://github.com/user-attachments/assets/78b308ea-c74b-415a-8fe0-46c0a2a562e3" />

---

## 3. Conclusion

The Online Retail dataset is clean in structure but very outdated.
Its quality is generally good: completeness, accuracy, and consistency are all around 90%, so it works well for learning or testing after a bit of cleaning. For example, you should remove duplicates, standardize product descriptions, separate returns from real sales, and decide what to do with missing CustomerID values (about 25% of the data).
However, it’s not suitable for real-world (production) AI use because it’s too old. The latest data is from December 2011 — more than 14 years ago. Since then, prices, products, customer behavior, and overall market conditions have changed a lot.


