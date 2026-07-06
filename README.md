# What-the-Market-Says (Reconciliation Project)
This project reconciles two independent Nigerian food price datasets — FEWS NET and WFP — to determine where they agree, where they diverge, and why. Both organisations track food prices across Nigerian markets independently but use different naming conventions, unit formats, and geographic coverage.


## Datasets

| Source | Rows | Period | Format |
|---|---|---|---|
| FEWS NET Staple Food Price Data | 315,712 | 2003–2024 | Excel |
| WFP Food Prices Nigeria | 87,646 | 2002–2026 | CSV |

Both datasets track retail and wholesale prices for food commodities across Nigerian 
markets, denominated in Nigerian Naira (NGN).

---

## Tools

| Tool | Purpose |
|---|---|
| Python (pandas) | Data loading, cleaning, transformation |
| DuckDB | SQL-based reconciliation engine |
| rapidfuzz | Fuzzy commodity name matching |
| Matplotlib / Seaborn | Visualisation |
| Jupyter Notebook | Documented workflow |
| Power BI | Final dashboard |

---


## Methodology

### Stage 1 — Profiling
Both datasets were profiled to understand their structure before any cleaning. Key 
observations included 56,719 null price values in FEWS NET (18% of rows), 
inconsistent date formats between the two sources, and significant differences in 
commodity naming conventions and unit formats.

### Stage 2 — Cleaning
Cleaning addressed the following issues in both datasets:
- Date columns converted from string to datetime
- Column names standardised to a common convention
- Units lowercased and stripped of inconsistent formatting
- Commodity and market names lowercased and stripped of extra whitespace
- FEWS NET sub-market names aggregated to city level to match WFP
- Null prices flagged with a `price_missing` indicator column rather than dropped

### Stage 3 — Mapping
Three reference tables were built to bridge the two sources:

**commodity_map.csv** — links FEWS NET product names to WFP commodity names under 
a shared `mapped_name`. Key decisions included matching "Maize Grain (White)" to 
"Maize (white)", "Gari (Yellow)" to "Cassava meal (gari, yellow)", and excluding 
live livestock entries (Cattle, Goats, Sheep) from reconciliation as FEWS tracks 
live animals while WFP tracks processed meat — fundamentally different products.

**unit_conversion.csv** — converts all unit variants to a common base unit (price 
per kg) to enable valid price comparison. Non-weight units (litres, tubers, pieces) 
are preserved separately and compared only within their own unit type.

**market_map.csv** — documents the 12 markets that overlap between both sources, 
8 FEWS-only markets, and the many WFP-only markets concentrated in conflict-affected 
areas of Borno state.

### Stage 4 — Reconciliation
Reconciliation was performed in DuckDB SQL using a LEFT JOIN on four keys:
- Market
- Mapped commodity name
- Price type (retail/wholesale)
- Date window (within 30 days)

Prices were normalised to a common base unit before comparison using the unit 
conversion table. Each row was then classified as:

- **MATCH** — both sources present, price difference ≤ 15%
- **CONFLICT** — both sources present, price difference > 15%
- **GAP** — one source present, the other absent

### Stage 5 — Analysis
Analysis examined coverage, conflict rates, and price trends across the reconciled 
dataset.

---

## Key Findings

### Overall reconciliation results

| Status | Count | Percentage |
|---|---|---|
| MATCH | 211,780 | 57.6% |
| GAP | 123,314 | 33.5% |
| CONFLICT | 32,877 | 8.9% |

### Coverage gaps
GAP rows are concentrated in the early 2000s (WFP had limited Nigerian coverage 
before 2013) and in markets where only one source operates. WFP tracks significantly 
more markets than FEWS NET, particularly in conflict-affected Borno state where 
humanitarian monitoring extends to remote communities FEWS NET does not cover.

### Conflict analysis
Yams (16.1%), Gari yellow (15.3%), and Gari white (15.3%) have the highest conflict 
rates — all highly perishable, locally traded commodities with strong regional price 
variation. Bread, diesel, and gasoline have the lowest conflict rates (around 5%), 
consistent with their more standardised pricing.

Rice (milled) recorded zero MATCH rows — every comparable row was classified as a 
CONFLICT — indicating the two sources are tracking rice in fundamentally incompatible 
ways, likely due to differences in quality grade and origin classification.

Biu (15.8%) and Ibadan (15.7%) have the highest market-level conflict rates. Kano 
has zero matches — FEWS NET tracks it but WFP has no overlapping data.

### Conflict rate over time
Conflicts were near-zero before 2013 due to limited WFP coverage. From 2013 
conflicts rose steadily as WFP expanded Nigerian operations, peaking at over 20% 
monthly conflict rate between 2019 and 2022. Contrary to expectation, conflict rates 
dropped sharply after the 2023 naira crisis — both sources tracked the same dramatic 
price increases simultaneously, suggesting the shock was large enough that both 
organisations captured it consistently.

### Price trend — Millet in Damaturu
FEWS NET and WFP tracked millet prices in Damaturu almost identically from 2015 to 
2025. Both sources captured the post-2023 price spike from ~200 NGN/kg to over 700 
NGN/kg simultaneously and with close agreement throughout — the strongest evidence 
in this dataset that the two sources are broadly reliable and complementary rather 
than contradictory.

---

## Reconciliation Methodology Decisions

| Decision | Rationale |
|---|---|
| 15% conflict threshold | Allows for normal market variation and data collection timing differences without flagging minor discrepancies |
| 30-day date window | FEWS and WFP do not collect on the same schedule; a monthly window captures comparable periods |
| Livestock excluded | FEWS tracks live animals, WFP tracks processed meat — not comparable |
| Rice (5% broken) excluded | No equivalent WFP category exists |
| Tubers and litres not converted to kg | Volume and count units are not weight-comparable |
| Sub-markets aggregated to city level | WFP uses city-level market names; FEWS uses sub-market names within the same city |

---

## Conclusion

This project demonstrates that FEWS NET and WFP are broadly consistent sources of 
Nigerian food price data where their coverage overlaps, with a 57.6% match rate 
across comparable rows. The 8.9% conflict rate is concentrated in specific perishable 
commodities and specific market periods, not distributed randomly — which suggests 
the disagreements are systematic and explainable rather than noise.

Neither source alone provides complete coverage of Nigerian food prices. Used 
together with a structured reconciliation methodology, they offer a more complete and 
reliable picture than either can provide independently.

