# Market Risk Analytics Report

## Overview

This project is an end-to-end Market Risk Analytics solution developed using Python, SQLite, SQL, and Power BI Desktop.

The project analyzes a hypothetical equity portfolio using historical market prices. It demonstrates how market data can be collected, transformed, stored, analyzed, and presented through an interactive risk report.

The report focuses on three areas:
* **Market Risk Overview:** Portfolio value, P&L, volatility, exposures, and concentration.
* **VaR & Tail Risk Analysis:** Historical Value-at-Risk, Expected Shortfall, P&L distribution, and VaR breaches.
* **Portfolio Stress Test & Risk Drivers for April 8, 2026:** Impact of a hypothetical adverse market scenario and identification of positions and sectors driving portfolio losses.

**Disclaimer:** This project is for educational and portfolio demonstration purposes only. The portfolio and stress scenario are hypothetical and do not represent investment advice or a real investment portfolio.

---

## Business Problem

A market risk function needs to understand not only the current value of a portfolio, but also:

* How much market risk is the portfolio taking?
* Where are the largest exposures and concentrations?
* How volatile are the portfolio and its underlying securities?
* How large could losses become based on historical market movements?
* How frequently do actual losses exceed the estimated VaR threshold?
* What happens to the portfolio under a defined adverse market scenario?
* Which securities and sectors contribute most to stressed losses?

This project develops a reporting solution designed to answer these questions.

---

## Technology Stack

* **Python:** Data acquisition, transformation and risk calculations. 
* **yfinance:** Historical market-price retrieval.
* **pandas:** Data cleaning and time-series manipulation.
* **sqlite3:** Python interface to the SQLite database.
* **SQLite/SQL:** Data storage, querying and analytical transformations.
* **Power BI Desktop:** Data modeling, DAX, visualization and interactive reporting. 

---

## Dashboard Structure (3-Page Layout)

### Page 1: Risk Overview
* High-level summary of portfolio value, baseline asset allocation, and overall market exposure.
* Core portfolio indicators tracking cross-sector diversification and asset weight distributions.

### Page 2: VaR & Tail Risk
* Historical and parametric Value at Risk calculations.
* Custom **Yes/No breach logic** tracking threshold violations against historical daily P&L series.

### Page 3: Portfolio Stress Test & Risk Drivers
* **Dynamic Scenario Slicers:** Connects to a decoupled `Stress_Parameters` table to apply multi-sector shock percentages dynamically without rebuilding ETL cycles.
* **KPI Cards:** Portfolio Value, Stressed Value, Dynamic Stress Loss ($), and Stress Loss %.
* **Visual Breakdown:**
  * **Stress Loss by Security:** Horizontal bar chart illustrating individual asset vulnerability.
  * **Exposure & Stress Loss by Sector:** Categorical comparison isolating hardest-hit industry groups.
  * **Risk vs. Exposure Scatter Plot:** Bubble chart mapping asset market values against annualized volatilities.
  * **Stress Test Details Table:** Granular itemized holdings view filtered to the latest trade date.

---

## Core DAX Measures

### 1. Dynamic Stress Loss
Calculates total dollar drop based on the active macro scenario, matching sector-specific shock percentages to the latest portfolio valuation date:
```dax
Dynamic_Stress_Loss = 
VAR LatestDate = CALCULATE(MAX(exposure_view[trade_date]), REMOVEFILTERS())
VAR SelectedScenario = SELECTEDVALUE(Stress_Parameters[Scenario Name])
RETURN
CALCULATE(
    SUMX(
        exposure_view,
        VAR CurrentSector = exposure_view[sector]
        VAR Shock = 
            LOOKUPVALUE(
                Stress_Parameters[Shock Pct],
                Stress_Parameters[Scenario Name], SelectedScenario,
                Stress_Parameters[Sector], CurrentSector
            )
        RETURN
        exposure_view[market_value] * COALESCE(Shock, 0)
    ),
    exposure_view[trade_date] = LatestDate
)
```

### 2. Portfolio Value and Stressed Value
Isolates the baseline portfolio worth on the latest trading day and evaluates total worth post-shock
```dax
Portfolio_Value = 
VAR LatestDate = CALCULATE(MAX(exposure_view[trade_date]), REMOVEFILTERS())
RETURN
CALCULATE(
    SUM(exposure_view[market_value]),
    exposure_view[trade_date] = LatestDate
)
```
```dax
Stressed_Value = [Portfolio_Value] + [Dynamic_Stress_Loss]
```

### 3. Stress Loss Percentage
Computes proportional portfolio drawdown safely using error-handling division:
```dax
Stress_Loss_Pct = DIVIDE([Dynamic_Stress_Loss], [Portfolio_Value], 0)
```

---

## Key Questions Answered
The completed analysis is designed to answer:
* What is the current value of the portfolio?
* Which tickers represent the largest exposures?
* Which sectors create the greatest concentration?
* Which tickers exhibit the greatest volatility?
* What is the portfolio's historical VaR?
* How severe are losses beyond VaR?
* How often does actual P&L breach the VaR threshold?
* How does VaR change over time?
* What happens under the hypothetical stress scenario?
* Which positions contribute most to the stressed portfolio loss?

