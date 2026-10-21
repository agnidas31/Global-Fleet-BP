Field-Tested Innovation · Global Fleet Cards · bp

# Creative Solutions: The Global Fleet KPI Report

How three regions that measured "the same" fleet business in three different ways were brought into one trusted, comparable view.

Agnivesh Das · Data Analyst · bp

> **Note:** the structure, regions and definitions reflect how the report worked. All values are illustrative, not real production figures.

---

## My Role

As Data Analyst, I owned the data behind the report end to end:

- **Data preparation.** Brought in fleet card volume, margin, card and customer data from three regional systems (ESA, ANZ and the US).
- **Cleaning and transformation.** Standardised each region's data into one model, documented each KPI's definition per region, and built the KPI validation checks.
- **Reporting.** Built the Power BI report: global summary, by-region views, a definitions page, and a KPI validation page.

---

## The Problem

### One global business, three local scoreboards

bp's fleet card business serves companies whose drivers fill up using a bp fleet card. It operates in several regions, each with its own systems, history and way of reporting.

Leadership wanted to answer simple questions:

- How is fleet volume trending in each region?
- How much margin is the business contributing?
- How many active cards and customers do we have, and is that growing?

But every region produced its own numbers in its own format. Comparing them meant pulling separate reports and hoping they meant the same thing.

---

## The Constraint Nobody Was Naming

### The same KPI meant different things in each region

The hard part wasn't building charts. It was that the regions didn't agree on what the numbers meant:

**1. Different units.** ANZ and ESA measure volume in **litres** and margin in **cents per litre**. The US uses **gallons** and **cents per gallon**.

**2. Different definitions of margin.** In ANZ, fleet contribution comes from the site margin waterfall plus card margin, minus site and card costs. In ESA, it's card contribution margin in local currency. In the US, it's average site margin applied to card volume, minus the cost of card offers.

**3. Different definitions of volume.** Some regions count only bp-branded cards. Others include partner networks and third-party cards, split into "issued and supplied", "supplied not issued" and "issued not supplied".

**4. Different time windows.** "Active cards" could mean active this month, or active over a rolling three months.

Putting these numbers side by side without addressing this would have produced a report that *looked* comparable but wasn't.

---

## The Fix

### A common model, regional definitions made explicit, and validation built in

Rather than forcing every region into one artificial definition, the report:

1. **Standardised the structure.** Every region's data was mapped into the same model: region, month, volume, contribution margin, active cards, active customers.
2. **Kept regional definitions, but made them visible.** A definitions page documents exactly what each KPI means in each region: unit, scope and calculation.
3. **Compared like with like.** Cross-region comparisons use **trends and year-on-year change**, which are meaningful even when units differ, rather than raw totals.
4. **Validated every refresh.** A KPI validation page checks each region's figures against source before the report is trusted.

### KPI set

| KPI | Definition (summary) |
| --- | -------------------- |
| **Fleet volume** | Fuel sold to fleet customers on bp and eligible third-party cards (litres or gallons by region) |
| **Fleet contribution margin** | Region-specific margin definition, in $ and cents per litre/gallon |
| **Active cards** | Cards that transacted in the month, and on a rolling 3-month basis |
| **Active customers** | Customers that transacted in the month, and on a rolling 3-month basis |
| **Average volume per card** | Total volume ÷ number of fleet cards |
| **Fleet share of volume** | Fleet volume ÷ total retail volume |

### How it works

1. **Ingest:** Monthly extracts from each region's fleet system.
2. **Standardise:** Map to one common data model, with region, unit and currency carried as attributes.
3. **Calculate:** Compute KPIs per region using that region's documented definition.
4. **Validate:** Reconcile against each region's own reporting.
5. **Report:** Global summary, by-region detail, monthly and year-to-month comparisons.

---

## Worked Example

### Why year-on-year matters more than totals

> **Illustrative values only.**

| Region | Monthly volume | Unit | YoY change |
| ------ | -------------- | ---- | ---------- |
| ESA | 500 million | litres | +2.0% |
| ANZ | 185 million | litres | −2.3% |
| US | 46 million | gallons | −6.5% |

Comparing raw volumes would suggest ESA is ten times the size of the US, but the units differ (one gallon ≈ 3.785 litres). The year-on-year column tells the real story immediately: ESA is growing, ANZ is slightly down, and the US is declining fastest. That's where leadership's attention should go.

---

## Why This Beat The Old Approach

|                          | Before                                | With the Global Fleet KPI report                  |
| ------------------------ | ------------------------------------- | ------------------------------------------------- |
| Source of numbers        | Separate regional reports             | One report, one data model                        |
| Definitions              | Implicit, different per region        | Documented per KPI, per region                    |
| Comparisons              | Raw totals, often misleading          | Trends and year-on-year, like for like            |
| Trust                    | "Which number is right?"              | Validation page reconciles to source              |
| Views                    | Static, region by region              | Global summary + regional drill-down              |

---

## Impact

**85% less report preparation:** the monthly cross-region pack no longer had to be assembled by hand.

**3× faster decisions:** fleet managers got their answers from one trusted view.

**3 regions, 1 report:** ESA, ANZ and the US on a single page.

**Definitions on the record:** every KPI documented for every region.

**Validated before it's used:** reconciliation checks built into the report.

The bigger shift was making differences visible instead of hiding them. Leaders could compare regions with confidence because they knew exactly what each number meant.

---
---

# Technical Deep Dive: How the Report Is Built

## 1. Common data model

```
fact_fleet_monthly
  region            -- ESA / ANZ / US
  month
  volume            -- in native unit
  volume_unit       -- 'litres' / 'gallons'
  margin_local      -- contribution margin, local currency
  currency
  active_cards_1m
  active_cards_3m
  active_customers_1m
  active_customers_3m
  card_count
  total_retail_volume
```

Carrying `volume_unit` and `currency` as columns (instead of silently converting) keeps the data honest and lets the report choose whether to show native or converted values.

## 2. Rolling 3-month actives

```sql
SELECT region,
       month,
       COUNT(DISTINCT CASE WHEN txn_month = month THEN card_id END)          AS active_cards_1m,
       COUNT(DISTINCT CASE WHEN txn_month BETWEEN ADD_MONTHS(month, -2)
                                              AND month THEN card_id END)    AS active_cards_3m
FROM   fleet_transactions
GROUP  BY region, month;
```

*(Simplified. In practice computed per region against a month calendar.)*

## 3. Year-on-year and year-to-month

```
YoY %  = (value this month − value same month last year) ÷ value same month last year
YTM    = sum from January to the selected month, compared with the same months last year
```

## 4. Optional normalisation

When a single global total is needed:

```
gallons → litres : × 3.785
local currency   → USD at a fixed planning rate (so FX moves don't look like business moves)
```

## 5. KPI validation

For each region and month:

- Volume, margin and active counts are compared with the region's own reported figures.
- Differences beyond a tolerance are flagged on the validation page.
- Data under review is labelled on the summary page (for example "ESA data is under review").

## 6. Report pages (Power BI)

1. **Global Fleet KPI Summary:** volume, contribution margin and active customers by region, monthly, with YoY.
2. **Global Fleet KPI by Region:** regional drill-down.
3. **Global Fleet Definitions:** the KPI dictionary.
4. **KPI Validation:** reconciliation checks.

---

Created by Agnivesh Das · bp Global Fleet · Illustrative values throughout. ESA, ANZ and US are bp fleet regions.
