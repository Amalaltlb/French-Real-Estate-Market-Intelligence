# French Real Estate Market Intelligence

Price per m² benchmarking across Calvados communes, using three years of official French government transaction data.

## Executive Summary

This project analyses 31,902 verified residential property sales across the Calvados department (Normandy) between 2023 and 2025, using DVF (Demandes de Valeurs Foncières) data published by the DGFiP. It produces a commune-level price benchmark, identifies the most and least expensive markets, and flags two real data quality issues in the source data that had to be resolved before the numbers could be trusted.

## Business Problem and Objectives

Real estate agencies, notaires, and property investors in France often price and value properties based on local experience rather than sourced, verifiable data. The objective of this project was to turn raw government open data into a client-ready price benchmark, showing that public data can replace informal pricing with a defensible, sourced alternative.

## Dataset

- Source: [DVF Géolocalisées](https://www.data.gouv.fr/datasets/demandes-de-valeurs-foncieres-geolocalisees/), published by the DGFiP (Direction Générale des Finances Publiques)
- Scope: Calvados (department 14), years 2023, 2024, and 2025 combined
- Format: UTF-8 CSV, one row per property record

## Tools

Python, pandas, NumPy, matplotlib, seaborn, Jupyter Notebook

## Data Preparation

Two data quality issues were identified and corrected during cleaning:

1. **Accent-sensitive filtering.** French category text in the source data is accented (for example, "Vente en l'état futur d'achèvement"). An unaccented filter string will not match it and will silently drop the entire category with no error raised. Category values were verified with `repr()` against the real data before any filter was written.
2. **Duplicate transaction rows.** DVF can record a single sale across multiple rows when it covers multiple lots or buildings, repeating the full sale price on each row. Left unaddressed, this double counts the sale and distorts price per m². Rows were deduplicated by transaction ID (`id_mutation`), keeping the record with the largest surface area.

After deduplication, IQR-based outlier removal was applied separately for apartments and houses, since the two property types have structurally different price distributions.

## Analysis and Methodology

- **Outlier treatment:** IQR (interquartile range) method, applied per property type rather than globally, with a hard floor of €500/m² to exclude non-market transactions such as intra-family transfers.
- **Aggregation:** Median price per m² by commune and property type, using the median rather than the mean since it is robust to a small number of very high-value sales.
- **Significance threshold:** Communes with fewer than 30 transactions were excluded from the benchmark table, as a stated reliability floor rather than a formal significance test.

## Key Findings

| Finding | Interpretation | Business implication |
|---|---|---|
| Final dataset: 31,902 transactions, down from 44,704 pre-deduplication | About 24% of raw rows were duplicate multi-lot records | Row counts in raw DVF data cannot be used directly as transaction counts |
| Apartments median 3,192 €/m², houses median 2,355 €/m² | Apartments carry a consistent premium over houses across the department | Property type should always be reported separately, never blended |
| Deauville highest at 6,250 €/m², Sommervieu lowest at 844 €/m² | A roughly 7x spread exists within a single department | Department-level averages hide large, commune-level pricing differences |
| Coastal communes dominate the top of both rankings | Deauville, Cabourg, Trouville-sur-Mer, Villers-sur-Mer, Houlgate | Consistent with known regional market dynamics, a useful sanity check on the pipeline itself |

## Business Recommendations

1. Report prices at the commune level, not the department level, given the scale of variation found.
2. Always separate apartments and houses in any pricing report, given the consistent premium gap between them.
3. Treat any DVF-based analysis with a mandatory deduplication step, since roughly a quarter of raw rows in this dataset were duplicates of an existing sale.

## Visualizations

- `charts/chart1_evolution.png`: median price per m² by year and property type, 2023 to 2025
- `charts/chart2_top_communes.png`: top 10 communes by median price, apartments and houses
- `charts/chart3_distribution.png`: price distribution by property type
- `charts/chart4_investment_map.png`: transaction volume against price, with premium/discount vs. departmental median

## Limitations

- DVF excludes Alsace-Moselle and Mayotte, and has a 3 to 6 month lag between the sale act and publication.
- The dataset does not include property condition, so two properties at the same price per m² may differ meaningfully in quality.
- The 30-transaction minimum is a stated reliability heuristic, not a formal statistical test.

## Project Deliverables

- `French-Real-Estate-Market-Intelligence.ipynb`: full analysis notebook
- `commune_benchmark_calvados.csv`: commune-level benchmark table
- `charts/`: four visualizations, PNG format
- `README.md`: this file

## Reproducibility

Clone the repository, install the packages listed above, and run the notebook top to bottom. The three source CSV files are downloaded automatically at the start of the notebook from `files.data.gouv.fr`.

## Author

Amal Mohamed
Freelance Data Analyst
[LinkedIn](https://www.linkedin.com/in/amal-mo) · [GitHub](https://github.com/Amalaltlb)
Caen, France · Remote Worldwide
