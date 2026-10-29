# California Real Estate Market Trends: Power BI Analysis

**Prices, affordability, demand, supply and geographic hotspots across 20,640 California census block groups**

| | |
|---|---|
| **Author** | [Alexander Chinwendu Liian] |
| **Date** | October 2026 |
| **Tools** | Power BI Desktop, Power Query, DAX, Excel |
| **Dataset** | California Housing, 1990 census block groups (20,640 rows) |
| **Deliverable** | 3-page interactive Power BI report (`Real_Estate_Market_Trends.pbix`) |
| **Context** | CodeAlpha Power BI Internship, Task 3: Real Estate Market Trends |

---

## 1. Executive summary

This report analyses where California housing is expensive, where it is out of reach for local incomes, and where demand is concentrated, with the aim of supporting investment and development screening. The interactive dashboard answers these questions through three pages: price and affordability, demand and supply, and a geographic hotspot map.

**Headline findings**

1. **Location relative to the coast is the biggest price divider.** Inland districts average **$123k** per home, against **$224k to $237k** in the three coastal regions, roughly 1.8 to 1.9 times higher.
2. **Affordability is a coastal problem.** The median district needs **4.95 years of median income** to buy the median home, and **49%** of districts sit above 5 years. That rises to **70% in Near ocean** and **66% in Near bay**, against **18.5% inland**.
3. **Demand is concentrated.** The "Under 1 hour from ocean" region holds **46% of all households** (4.7 million of 10.3 million), and Inland holds a further **30%**.
4. **Supply is tighter near the coast.** Coastal regions carry about **10 to 12% more households per 1,000 rooms** than inland, and inland households have more space (5.7 rooms against 5.1 to 5.2).
5. **Hotspots form two clusters:** the San Francisco Bay Area and Santa Cruz area, and the Southern California coast. The cheapest large areas sit in the Central Valley, at roughly $81k to $88k average value.
6. **Income is the strongest driver visible in the data** (correlation of 0.69 with home value). However, the *lowest* income band is the *least* affordable: **57%** of its districts need more than 5 years of income.

> **Important context:** this is a 1990 census snapshot in 1990 dollars with no rent and no time series. Findings describe *relative* market structure, not current prices.

---

## 2. Business context and objectives

An investor or developer screening California markets needs to know where value is, where pressure is building, and where the evidence points to opportunity. The brief for this project was to:

| Objective | How it was addressed |
|---|---|
| Analyse property prices and rental yields | Price analysed by region, income band and home age. Rental yield is not calculated because the dataset contains no rent data (see limitations) |
| Track market demand, supply conditions and economic indicators | Households and population (demand), rooms per household and households per 1,000 rooms (supply), median income and price-to-income (economic) |
| Use geographical visualisations to identify hotspots | Map of 20,640 districts sized by households, with slicers for income band, price band and region |

---

## 3. Data

**Source:** California Housing dataset, 1990 US census, one row per census block group ("district"). The workbook (`California_Housing_Cleaned.xlsx`) contains the `Clean_Data` sheet, a data dictionary and a cleaning log.

### 3.1 Key fields

| Field | Meaning |
|---|---|
| `median_house_value` | Median home value, US$ (treat as 1990 dollars). Capped at 500,001 |
| `median_income` | Median household income in tens of thousands of dollars (3.5 = about $35,000). Capped at 15.0001, floored at 0.4999 |
| `housing_median_age` | Median age of homes in years. Capped at 52 |
| `total_rooms`, `households`, `population` | Counts per district |
| `ocean_proximity` / `ocean_proximity_label` | Distance-to-coast class: Inland, Under 1 hour from ocean, Near bay, Near ocean, Island |
| `price_to_income` | Home value divided by annual income (years of income to buy the median home) |
| `quality_flag` | `OK`, `Extreme ratio` or `Inconsistent` |
| `longitude`, `latitude` | Location, used for the map |

### 3.2 Data quality profile

| Check | Result | Treatment |
|---|---|---|
| Rows | 20,640, none removed | Problem rows flagged, not deleted |
| Exact duplicates | 0 | None needed |
| Missing values | 207 blank `total_bedrooms` | Imputed (households x median bedrooms-per-household of 1.0489) and flagged. Bedrooms are not used in any dashboard measure |
| Capped home values | 965 rows (4.7%) at $500,001 | Kept. **Excluded from price averages** |
| Capped home age | 1,273 rows at 52 | Kept. Read as "52 or older" |
| Capped or floored income | 61 rows | Kept |
| Extreme ratios | 104 rows (over 20 rooms or 10 people per household) | Kept. **Excluded from ratio measures** |
| Inconsistent rows | 3 rows (more households than people) | Kept. **Excluded from ratio measures** |

---

## 4. Approach

1. **Acquire and validate.** Loaded the cleaned workbook and reviewed the cleaning log, data dictionary and range checks before modelling.
2. **Prepare in Power Query.** Imported the `Clean_Data` sheet, promoted headers and set explicit data types (integers for counts, decimals for coordinates and ratios, text for labels).
3. **Model.** A single fact table (`Clean_Data`) plus a dedicated measures table. A single table is appropriate here because the data is flat and has no dimension tables.
4. **Enrich with calculated columns.** Created income, price and home-age bands, and a sort-order column for age bands.
5. **Define measures in DAX.** Ten measures encode every KPI so that figures stay consistent across pages and respond correctly to slicers.
6. **Design the report.** Three pages at 1920 x 1080, with shared slicers for income band, price band and region.
7. **Validate.** The figures in this report were recomputed independently from the dataset using the same measure definitions.

### 4.1 Key analytical decisions

| Decision | Reason |
|---|---|
| Exclude capped values from price averages | A value of 500,001 means "500,000 or more", so averaging it as exact would understate prices, especially on the coast where 8% of Near bay and Near ocean districts are capped |
| Apply the quality flag only to ratio measures | The 107 flagged rows are real districts with unusual household structure. They distort per-household ratios but not simple counts, so they remain on the map and in totals |
| Use a 5-years-of-income threshold for "hard to afford" | A common rule of thumb for a high median multiple. It is an analyst choice, so it is stated on the dashboard |
| Treat regions as "coastal proximity", not named markets | The data holds distance-to-coast classes, not cities or counties |
| Do not estimate a rental yield | No rent data is available. Inventing a rent assumption would present a guess as a finding |

---

## 5. Data model and measures

**Calculated columns**

| Column | Logic |
|---|---|
| Income Band | Under $25k (income below 2.5), $25k-40k, $40k-60k, Over $60k |
| Price Band | Under $150k, $150k-250k, $250k-350k, Over $350k |
| Home Age Band | 0-10, 11-20, 21-30, 31-40, 41+ years (with `Home Age Band Order` for correct sorting) |

**Measures**

| Measure | DAX | Purpose |
|---|---|---|
| Total Districts | `COUNTROWS(Clean_Data)` | Market size |
| Total Households | `SUM(Clean_Data[households])` | Demand |
| Total Population | `SUM(Clean_Data[population])` | Demand |
| Avg Home Value | `CALCULATE(AVERAGE(Clean_Data[median_house_value]), Clean_Data[value_capped] = 0)` | Price level, capped values excluded |
| Median Income ($) | `MEDIAN(Clean_Data[median_income]) * 10000` | Economic indicator |
| Median Price-to-Income | `CALCULATE(MEDIAN(Clean_Data[price_to_income]), Clean_Data[quality_flag] = "OK")` | Affordability |
| % Hard to Afford | `DIVIDE(CALCULATE(COUNTROWS(Clean_Data), price_to_income > 5, quality_flag = "OK"), CALCULATE(COUNTROWS(Clean_Data), quality_flag = "OK"))` | Share of districts above 5 years of income |
| Avg Household Size | `DIVIDE(SUM(population), SUM(households))` | Demand structure |
| Rooms per Household | `DIVIDE(SUM(total_rooms), SUM(households))` | Supply (space) |
| Households per 1,000 Rooms | `DIVIDE(SUM(households), SUM(total_rooms)) * 1000` | Supply (tightness) |

---

## 6. Dashboard overview

### Page 1: Price and Affordability

<img width="885" height="494" alt="CodeAlpha Real Estate" src="https://github.com/user-attachments/assets/82e5d391-2a5c-4359-a750-0d23a47507af" />


- **KPI cards:** Total Districts, Avg Home Value, Median Income, Median Price-to-Income, % Hard to Afford, Avg Household Size
- **Charts:** Avg Home Value by Region, Avg Home Value by Income Band, Median Price-to-Income by Region, Rooms per Household and Households per 1,000 Rooms by Region, Total Households and Total Population by Region
- **Slicers:** Income Band, Price Band, Region

### Page 2: Demand and Supply

<img width="878" height="497" alt="CodeAlpha Real Estate 2" src="https://github.com/user-attachments/assets/05ebde7e-1b52-48fa-a094-fc195b835743" />


- **Demand:** Population by Region and Total Households by Region (donuts)
- **Supply:** Rooms per Household and Households per 1,000 Rooms by Region
- **Market structure:** Total Districts and Avg Home Value by Home Age Band (combo chart), and an income-versus-home-value scatter coloured by region
- **Slicers:** Income Band, Home Age Band, Region

### Page 3: Hotspot Map

<img width="896" height="495" alt="CodeAlpha Real Estate 3" src="https://github.com/user-attachments/assets/25f4170a-09b0-41a3-94ce-f9b60e393166" />


- **Map:** all 20,640 districts plotted by latitude and longitude, bubble size showing households
- **Slicers:** Income Band, Price Band, Region, so a viewer can isolate, for example, premium districts (Over $350k) in a single region

---

## 7. Findings

### 7.1 Headline KPIs (all districts, no filters)

| Metric | Value |
|---|---|
| Districts | 20,640 |
| Households / population | 10.31 million / 29.42 million |
| Avg home value (capped excluded) | $192.5k |
| Median income | $35.3k |
| Median price-to-income | 4.95 years |
| Districts above 5 years of income | 49.1% |
| Avg household size | 2.85 |
| Rooms per household | 5.28 |
| Households per 1,000 rooms | 189.5 |

### 7.2 Regional comparison

| Region | Districts | Share of households | Avg home value | Median income | Price-to-income | Hard to afford | Household size | Rooms per household | Households per 1,000 rooms |
|---|---|---|---|---|---|---|---|---|---|
| Inland | 6,551 | 30.3% | $123.3k | $29.9k | 3.6 | 18.5% | 2.91 | 5.69 | 175.7 |
| Under 1 hour from ocean | 9,136 | 45.9% | $224.0k | $38.8k | 5.4 | 60.5% | 2.94 | 5.08 | 197.0 |
| Near bay | 2,290 | 10.9% | $236.9k | $38.2k | 5.8 | 66.1% | 2.52 | 5.10 | 195.9 |
| Near ocean | 2,658 | 12.9% | $227.7k | $36.5k | 6.1 | 69.7% | 2.70 | 5.16 | 194.0 |

*Island (5 districts) is excluded from this table and from the commentary because the sample is too small to be reliable.*

![Regional overview](images/01_region_overview.png)

**What this shows**

- Incomes differ far less than prices. Coastal median income is only about 22 to 30% above inland, while home values are 82 to 92% higher. That gap is what pushes coastal price-to-income ratios above 5.
- Near bay has the highest average value but not the worst affordability. Near ocean is the most stretched (6.1 years), because its incomes are lower than Near bay's.
- Inland is the only region where the typical district is below the 5-year threshold, and it is also where space per household is greatest.
- Averages on the coast are, if anything, conservative: capped districts are excluded, and 8.5% of Near bay and 8.0% of Near ocean districts are capped, against 0.4% inland.

### 7.3 Income and affordability

| Income band | Districts | Avg home value | Median price-to-income | Hard to afford |
|---|---|---|---|---|
| Under $25k | 4,777 | $120k | 5.5 | 57% |
| $25k-40k | 7,739 | $175k | 5.0 | 51% |
| $40k-60k | 5,751 | $235k | 4.7 | 42% |
| Over $60k | 2,373 | $331k | 4.8 | 44% |


- Home values rise steeply with income: the top band averages about 2.8 times the bottom band.
- Despite having the lowest prices, the lowest-income districts are the least affordable. **More than half of them need over 5 years of income to buy**, the highest share of any band. This is the clearest evidence of an affordability gap.
- Income and home value correlate at **0.69** across all districts (0.64 with capped values excluded), a strong relationship but far from deterministic.


### 7.4 Demand and supply

- **Demand** is heavily weighted to the Under 1 hour from ocean region (45.9% of households, 47.2% of population). Inland adds 30.3% of households.
- **Household size** is smallest near the bay (2.52) and ocean (2.70), which means more households per head of population and therefore more housing units needed for the same number of people.
- **Supply** is tighter on the coast: households per 1,000 rooms is 194 to 197 on the coast against 176 inland, about **10 to 12% higher**. Differences are modest, so this is best read as consistent with the price pattern, not as a standalone shortage signal.

### 7.5 Housing age

| Home age band | Districts | Avg home value | Median price-to-income |
|---|---|---|---|
| 0-10 years | 1,569 | $191k | 4.2 |
| 11-20 years | 4,724 | $184k | 4.5 |
| 21-30 years | 4,852 | $193k | 5.0 |
| 31-40 years | 5,617 | $191k | 5.1 |
| 41+ years | 3,878 | $206k | 6.1 |

Average value is nearly flat across age bands (roughly $184k to $206k), but **affordability worsens steadily with older housing stock**, from 4.2 to 6.1 years of income. A plausible explanation is that older stock is concentrated in established coastal areas, though this report has not tested that. Note that the 41+ band includes homes capped at "52 or older".

### 7.6 Geographic hotspots

Grouping districts into half-degree map cells (cells with at least 150 uncapped districts):

| Rank | Approximate area | Districts | Avg home value | Median price-to-income |
|---|---|---|---|---|
| 1 | San Francisco Bay Area, peninsula (37.5, -122.5) | 465 | $300k | 6.8 |
| 2 | Santa Cruz and south Bay (37.0, -122.0) | 285 | $271k | 5.8 |
| 3 | Orange County (33.5, -118.0) | 469 | $264k | 5.0 |
| 4 | San Francisco Bay Area, north and east (38.0, -122.5) | 889 | $256k | 7.0 |
| 5 | Southern California, Orange and Riverside area (33.5, -117.5) | 255 | $254k | 4.5 |

The cheapest large cells are in the **Central Valley** (around Bakersfield, Fresno and Merced), at **$81k to $88k** and about 3 years of income. Place names are approximate and inferred from coordinates, because the dataset contains no city or county field.

The North and East Bay cell is the most strained of the premium markets (7.0 years of income) and also the largest, with 889 districts and about 440,000 households. Orange County combines high values with a lower ratio of 5.0, which suggests stronger local incomes.

---

## 8. Recommendations

These are screening signals, not valuation advice, given the data caveats below.

1. **Treat coastal markets as supply-constrained, with affordability limits.** With price-to-income above 5 in three of four regions and tighter households-per-room, further price growth is more likely to be limited by what incomes can support. Developers should weigh attainable-price product against premium product in these markets.
2. **Look at inland districts as the entry-price market.** Inland combines the lowest values ($123k), the lowest price-to-income (3.6), the most space per household and 30% of households. This is a sensible starting point for a yield-focused investor, and where adding real rent data would be most valuable.
3. **Prioritise the Bay Area hotspot cells for premium positioning, but test the downside.** Values and ratios there are the highest in the dataset, which suggests strong demand and also higher exposure if incomes do not keep pace.
4. **Target affordability needs in the lowest income band.** More than half of its districts exceed 5 years of income, so it is the strongest case for affordable housing.
5. **Investigate older housing stock in coastal areas.** Rising ratios with home age suggest possible renovation or redevelopment opportunities, subject to confirming where that stock sits.

---

## 9. Limitations and assumptions

| Limitation | Impact | How it is handled |
|---|---|---|
| **1990 snapshot, 1990 dollars, no time series** | The brief's "trends" cannot be measured over time. Absolute prices are out of date | Findings are framed as cross-sectional structure (region, income, age). Do not read dollar values as current |
| **No rent data** | Rental yield cannot be calculated | Not estimated in this report. A true yield needs a merged rent dataset (see next steps) |
| **Single economic indicator** | Income is the only economic variable | Income and price-to-income are used. Unemployment, rates and permits would strengthen the analysis |
| **Capped values** (home value 4.7%, age 6.2%, income 0.3%) | Understates top-end prices and old-stock age | Capped values are excluded from price averages and noted on the dashboard |
| **Averages of medians** | `Avg Home Value` is an unweighted mean of block-group medians, so large and small districts count equally | Adequate for screening. A household-weighted average would be a refinement |
| **Regions are coastal-proximity classes** | Cannot be read as named markets | Labelled as coastal proximity throughout |
| **Supply measured by rooms** | No listings, permits or vacancy data | Treated as a proxy for housing stock and stated as such |
| **Small groups** | Island has only 5 districts | Excluded from commentary |
| **Group-level data** | Patterns describe districts, not individual homes or buyers | Findings are not applied to individual properties |

---


## 10. Repository structure and how to open

```
CodeAlpha_RealEstateMarketTrends/
|-- README.md
|-- Real_Estate_Market_Trends.pbix
|-- data/
|   `-- California_Housing_Cleaned.xlsx
`-- images/
    |-- dashboard_page1_price_affordability.png
    |-- dashboard_page2_demand_supply.png
    |-- dashboard_page3_map.png
    |-- 01_region_overview.png
    |-- 02_income_vs_value.png
    |-- 03_hotspot_map.png
    `-- 04_income_band.png
```

---

## 11. Appendix: column dictionary

| Column | Type | Origin |
|---|---|---|
| district_id | Integer | Added during cleaning |
| longitude, latitude | Decimal | Original |
| housing_median_age | Integer | Original (capped at 52) |
| total_rooms, total_bedrooms, population, households | Integer | Original (207 bedroom values imputed) |
| median_income | Decimal | Original (tens of thousands of dollars) |
| median_house_value | Integer | Original (capped at 500,001) |
| ocean_proximity | Text | Original |
| ocean_proximity_label | Text | Derived, readable label |
| bedrooms_imputed, value_capped, age_capped, income_capped | 0/1 | Flags |
| rooms_per_household, bedrooms_per_room, population_per_household, price_to_income | Decimal | Derived ratios |
| quality_flag | Text | OK, Extreme ratio, Inconsistent |
| Income Band, Price Band, Home Age Band, Home Age Band Order | Text / Integer | Calculated in Power BI |

# CodeAlpha--Real-Estate-Market-Trends
