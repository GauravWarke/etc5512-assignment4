==================================================================
AUSTRALIAN SUPPLY-CHAIN RESILIENCE TO FREIGHT SHOCKS
==================================================================

Author: Gaurav Warke (36782122)
Date: June 11, 2026

================
PROJECT OVERVIEW
================

This project examines how Australian businesses respond when global freight 
costs spike. Using three Australian Bureau of Statistics datasets spanning 
2019 Q1 – 2026 Q1 (29 quarters), the analysis tracks freight prices across 
modes, Australian import/export activity, and business-sector sales to answer: 
Do companies adjust supply chains or absorb costs through margins?

The analysis covers two major disruption periods:
- 2021–22: Container shipping crisis (sea-freight PPI peaked ~3x baseline)
- 2024: Red Sea geopolitical disruption

==========================
DATA SOURCES AND LICENSING
==========================

All data sourced from the Australian Bureau of Statistics and released under 
Creative Commons Attribution 4.0 International (CC BY 4.0).

Files and download locations:
- ppi.csv   → ABS cat. 6427.0 (Producer Price Indexes)
  Quarterly freight price indexes (Original series)
  https://www.abs.gov.au/statistics/economy/price-indexes-and-inflation/producer-price-indexes-australia
  (Data downloads → Transport, Postal and Warehousing time-series table)
- trade.csv → ABS cat. 5368.0 (International Trade in Goods)
  MONTHLY, seasonally adjusted, current-price $ millions.
  Debits (imports) are published as NEGATIVE values.
  https://www.abs.gov.au/statistics/economy/international-trade/international-trade-goods/latest-release
  (Data downloads → goods credits/debits time-series table)
- biz.csv   → ABS cat. 5676.0 (Business Indicators)
  Quarterly sales by industry, seasonally adjusted CHAIN VOLUME MEASURES
  (real, inflation-removed).
  https://www.abs.gov.au/statistics/economy/business-indicators/business-indicators-australia/latest-release
  (Data downloads → sales by industry time-series table)

If a link changes, search the catalogue number on https://www.abs.gov.au.
All files retain the standard ABS time-series layout: 9 metadata rows
(Unit, Series Type, Data Type, Frequency, Collection Month, Series Start,
Series End, No. Obs), then a Series ID header row, then one row per
period with dates in Mon-YYYY format (e.g. Mar-2019).

Lookup files (author-created):
- ppi_lookup.csv (4 freight modes)
- trade_lookup.csv (2 trade flows)
- biz_industry_lookup.csv (15 ANZSIC industries)
Each lookup maps a verified ABS series_id to a readable label
(mode / flow / industry).

All data and derived work released under CC BY 4.0.

==============
FILE STRUCTURE
==============

assignment4-final/
│
├── assignment4_HD.qmd                 (Main Quarto document - all tasks)
├── assignment4_HD.html                (Rendered output)
├── README.txt                         (This file)
├── AI_use_statement.pdf               (Unit-required AI declaration)
│
├── versions/                          (Task 3 Q8: iterative drafts)
│   ├── assignment4_v1.qmd             (May 1  - PPI only)
│   ├── assignment4_v2.qmd             (May 15 - + business data)
│   ├── assignment4_v3.qmd             (May 25 - + trade data)
│   └── assignment4_v4.qmd             (June 1 - absorption framing)
│
├── data-raw/                          (Raw ABS downloads)
│   ├── ppi.csv                        (Freight price indexes, 9 metadata rows)
│   ├── trade.csv                      (Monthly import/export values, 9 metadata rows)
│   ├── biz.csv                        (Business sales, 9 metadata rows)
│   ├── ppi_lookup.csv                 (Series ID → mode mapping)
│   ├── trade_lookup.csv               (Series ID → flow mapping)
│   └── biz_industry_lookup.csv        (Series ID → industry mapping)
│
└── data-processed/                    (Processed, rebased data)
    ├── ppi.rds                        (96 rows: Road/Rail/Sea × 29 qtrs + Air × 9)
    ├── trade.rds                      (58 rows: 2 flows × 29 quarters)
    └── biz.rds                        (435 rows: 15 industries × 29 quarters)

===============
DATA DICTIONARY
===============

All processed files (.rds) follow this schema:

Column      Type        Description
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
quarter     Date        First day of quarter (e.g., 2019-01-01)
                        Join key across all datasets
mode        Character   Freight mode: Road, Rail, Sea, Air (PPI only)
flow        Character   Trade direction: Exports, Imports (Trade only)
industry    Character   Business sector: Mining, Manufacturing, etc. (Biz only)
index       Numeric     Rebased index where 2019 Q1 = 100

Rebasing: index = 100 × (value_at_quarter / value_at_baseline)

Trade values are converted to absolute values before indexing (ABS
publishes debits as negatives), and monthly observations are averaged
within each quarter before rebasing.

======================
SERIES ID VERIFICATION
======================

All series identifiers verified against ABS March 2026 release.

PPI (cat. 6427.0) — Original series:
  Road freight:       A2314058K
  Rail freight:       A2314067L
  Sea freight:        A2314076R
  Air freight:        A130272147K   (series begins 2024 Q1; 9 observations)

Trade (cat. 5368.0) — Seasonally Adjusted, current prices:
  Exports (Credits, Total goods):  A2718577A
  Imports (Debits, Total goods):   A2718603V

Business (cat. 5676.0) — Seasonally Adjusted, Chain Volume Measures:
  15 industries, series IDs A3538807V through A3538849T
  (See biz_industry_lookup.csv for the complete mapping)

  NOTE: An earlier version of biz_industry_lookup.csv contained a
  transcription error for Administrative and support services
  (A3538843R). The correct ID is A3538843C; this was caught during
  verification against the raw file and the lookup has been corrected.
  With the wrong ID, the inner join silently dropped that industry
  (14 industries instead of 15).

===================
PROCESSING PIPELINE
===================

Implemented in the chunk labelled "ingest" in the Data Details tab of
assignment4_HD.qmd, via a single helper function (process_abs) applied
to all three datasets.

Stage 1: Read ABS CSV files (skip 9 metadata rows so the Series ID row
         becomes the header) and read lookup tables
  Input: ppi.csv, trade.csv, biz.csv (+ three lookup CSVs)

Stage 2: Transform
  - Pivot from wide to long format
  - Inner-join with series ID lookup tables (keeps verified series only)
  - Parse Mon-YYYY dates with lubridate::my() and floor to quarter
  - Filter to 2019 Q1 onwards; drop missing values
  - Take absolute values (trade debits are negative)
  - Average within quarter (aligns monthly trade data to the
    quarterly grid of the other two datasets)
  - Group by category (mode/flow/industry)
  - Rebase index to 100 at baseline quarter (2019 Q1)
  - Select: category, quarter, index

Stage 3: Save
  - Serialize to .rds format (preserves data types)
  - Output to data-processed/ folder
  - Display success message: "✓ Data processed and saved."

All operations are deterministic. Re-running produces identical results.

================
ANALYSIS OUTPUTS
================

The Blog Post tab renders the following from the processed data:
  Figure 1: Freight price indexes by mode with shock periods shaded
            (sea freight peaks at index ~302 in 2022 Q3)
  Figure 2: Export/import value indexes through both shock windows
            (import spending peaks ~40% above baseline during the shock)
  Figure 3: Industry real-sales correlation with sea-freight PPI
            (ranked bars; Wholesale trade highest at r ≈ 0.64)
  Table 1:  Three most and three least freight-correlated industries
  Inline values: peak sea-freight index/quarter, road/rail peaks,
            import shock peak — all computed at render time

===============
KEY LIMITATIONS
===============

Measurement basis:
  Trade values are nominal (current prices) — they measure import/export
  SPENDING, not pure volumes. Price rises are embedded in them.
  Business sales are chain volume measures (real terms).

Confidentiality:
  ~5% of trade detail suppressed by ABS (Confidential Commodities List)
  ~10–15% of business economy excluded (micro non-employing businesses)

Time Coverage:
  Air freight: Only 9 observations (2024 Q1 onwards)
  → Trend analysis unreliable; included for completeness only

Causality:
  Analysis shows co-movement, not causal relationships
  High correlation ≠ freight prices caused sales changes
  Both could respond to common shocks (e.g., global demand)
  Level correlations also partly capture common post-COVID growth
  trends, which inflates mid-ranking correlations

================
HOW TO REPRODUCE
================

Requirements: R 4.0+, RStudio, Quarto
Packages: tidyverse, here, lubridate, kableExtra, scales
(install lines are provided, commented out, in the setup chunk)

Steps:
1. Open the assignment4-final/ folder as an RStudio project
   (the here package resolves all paths relative to the project root)
2. Confirm data-raw/ contains the three ABS files and three lookup files
   (download links above) and that data-processed/ exists
3. Open assignment4_HD.qmd in RStudio
4. Render the document (Ctrl+Shift+K)
   - The "ingest" chunk runs automatically during render, prints
     "✓ Data processed and saved.", and writes the three .rds files
   - All figures, tables and inline values are computed at render time
5. Output: assignment4_HD.html

Processing time: ~10–15 seconds

===================
VARIABLE DEFINITIONS
===================

Freight Modes (PPI):
  Road:   Heavy vehicle transport (trucking)
  Rail:   Train freight services
  Sea:    Maritime shipping (containers, bulk, general cargo)
  Air:    Airfreight and courier services

Trade Flows:
  Exports: Goods leaving Australia (Credits, Total goods)
  Imports: Goods entering Australia (Debits, Total goods;
           published as negative values, converted to absolute)

ANZSIC Industries (15 total):
  Mining, Manufacturing, Electricity/gas/water, Construction, Wholesale,
  Retail, Accommodation/food, Transport/postal, Information media, 
  Financial/insurance, Rental/real estate, Professional/scientific,
  Administrative/support, Arts/recreation, Other services

Index Interpretation:
  100 = 2019 Q1 baseline
  110 = 10% above baseline
  150 = 50% above baseline
  300 = 3x baseline
  90  = 10% below baseline

===================
ANALYTICS PERFORMED
===================

1. Time-series analysis
   - Identified shock periods (2021–22, 2024)
   - Tracked index movements across modes

2. Co-movement analysis
   - Computed Pearson correlations between sea-freight PPI and
     real industry sales (levels, 29 quarters)
   - Ranked industries by correlation

3. Narrative analysis
   - Connected three datasets to tell supply-chain story
   - Interpreted findings through economic lens (cost absorption vs. adjustment)
   - Critically assessed where level correlations are and are not
     informative (trustworthy at extremes, trend-inflated in the middle)

Limitations:
  - No statistical significance testing
  - No lagged correlations (do sectors respond with 1–2 quarter delay?)
  - No causal inference (no instrumental variables)

===================
FUTURE IMPROVEMENTS
===================

1. Distributed-lag regression to test sector response timing
2. Instrumental variables using geopolitical shocks as exogenous freight drivers
3. FX controls (AUD/USD may confound results)
4. Modal substitution analysis (National Freight Data Hub)
5. Commodity-level analysis (if confidentiality allows)
6. Correlations on quarterly growth rates (removes common trends that
   inflate level correlations)

===========
USAGE TERMS
===========

Data: CC BY 4.0 (attribute Australian Bureau of Statistics)
Code: CC BY 4.0 (attribute Gaurav Warke)
Derived work: CC BY 4.0 (attribute both)

When using or reproducing:
Warke, G. (2026). Supply-Chain Resilience to Freight-Price Shocks in Australia.
Monash University, School of Business Analytics.

=======
Support
=======

Data source: Australian Bureau of Statistics (https://www.abs.gov.au)
Code language: R / Quarto
Questions: gaurav.warke@monash.edu

===============
Version History
===============

v1.0  — Initial final draft
  - All three datasets integrated
  - Narrative focused on absorption vs. adjustment
  - Professional documentation

v1.1  — Reproducibility and analysis alignment
  - Full runnable ingest pipeline embedded in qmd ("ingest" chunk),
    matching the reproduction steps in this README
  - Three figures, one summary table and inline computed values added
    to the Blog Post tab
  - ABS download URLs documented; R package citations added

v1.2 — Verified against raw data
  - Corrected biz_industry_lookup.csv: A3538843R → A3538843C
    (Administrative and support services)
  - Fixed date parsing (Mon-YYYY format → lubridate::my())
  - Documented negative trade debits and monthly→quarterly averaging
  - Updated all reported figures to verified values (sea-freight peak
    index ~302 in 2022 Q3; import spending peak ~+40%; Wholesale trade
    top correlation r ≈ 0.64)
  - Corrected processed-data row counts (96 / 58 / 435; 29 quarters)
  - Clarified measurement bases (nominal trade values vs. real
    chain-volume sales)
