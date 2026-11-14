# 🏦 Banking Customer Analytics & Risk Segmentation

![Excel](https://img.shields.io/badge/Excel-Power%20Query%20%7C%20Pivot%20%7C%20Scenario-217346)
![SQL](https://img.shields.io/badge/SQL%20Server-CTEs%20%7C%20Window%20Functions%20%7C%20Views%20%7C%20Stored%20Procs-CC2927)
![Python](https://img.shields.io/badge/Python-pandas%20%7C%20scikit--learn%20%7C%20seaborn-3776AB)
![PowerBI](https://img.shields.io/badge/Power%20BI-5--page%20Dashboard-F2C811)

An end-to-end BFSI analytics project on **45,211 bank customers** from a term-deposit telemarketing campaign. The project answers four business questions: **who subscribes, when, why, and which customers should the bank target next.** It also assesses credit risk and builds a propensity model to prioritise call lists.

**Pipeline:** Raw data → Excel cleaning & scenario analysis → SQL Server analysis → Python EDA, ML & RFM → Power BI dashboard.

---

## 📌 Table of Contents
1. [Business Problem & Objectives](#1-business-problem--objectives)
2. [Dataset](#2-dataset)
3. [Project Workflow & Repository Structure](#3-project-workflow--repository-structure)
4. [Step 1 – Raw Data](#4-step-1--raw-data)
5. [Step 2 – Excel: Cleaning, Pivots & Scenario Analysis](#5-step-2--excel-cleaning-pivots--scenario-analysis)
6. [Step 3 – SQL Server Analysis](#6-step-3--sql-server-analysis)
7. [Step 4 – Python: EDA, Prediction Model & RFM](#7-step-4--python-eda-prediction-model--rfm)
8. [Step 5 – Power BI Dashboard](#8-step-5--power-bi-dashboard)
9. [Step 6 – Dashboard Screenshots](#9-step-6--dashboard-screenshots)
10. [Consolidated Key Findings](#10-consolidated-key-findings)
11. [Business Recommendations](#11-business-recommendations)
12. [Known Limitations & Next Steps](#12-known-limitations--next-steps)
13. [How to Run](#13-how-to-run)
14. [Skills Demonstrated](#14-skills-demonstrated)
15. [Author](#15-author)

---

## 1. Business Problem & Objectives

The bank runs phone campaigns to sell **term deposits**. Only about **1 in 9 customers (11.7%)** subscribes, so most calls produce no revenue. Management needs to know where to focus effort.

**Objectives**
- Measure the overall subscription rate and find which segments (job, age, education, balance, risk) convert best.
- Find the best contact timing, channel and call behaviour.
- Assess credit risk: default exposure and high-risk customers.
- Segment customers by engagement and value using **RFM**.
- Build a **propensity model** that scores each customer's likelihood to subscribe.
- Quantify the revenue opportunity from improving the subscription rate (scenario analysis).
- Deliver everything in an interactive dashboard for decision-makers.

---

## 2. Dataset

| Item | Detail |
|---|---|
| **Source** | UCI Machine Learning Repository – *Bank Marketing* dataset |
| **Rows** | 45,211 customers |
| **Columns** | 17 original + 7 derived = 24 |
| **Target** | `y` / `Sub_Num` – did the customer subscribe to a term deposit? (Yes = 5,290, **11.7%**) |
| **Class balance** | Imbalanced (about 1 : 7.5) – handled with `class_weight='balanced'` in the model |

### Data dictionary

| Column | Type | Description |
|---|---|---|
| `Age`, `Job_Title`, `Marital_Status`, `Education` | Demographic | Customer profile |
| `default_status` | Risk | Has credit in default? (Yes/No) |
| `Balance` | Financial | Average yearly account balance (€/$) |
| `Housing`, `Loan` | Financial | Housing loan / personal loan flags |
| `Contact`, `Day`, `Month` | Campaign | Channel (Cellular / Telephone / Unknown) and last contact date |
| `Duration` | Campaign | Last call length in seconds |
| `Campaign` | Campaign | Number of contacts in this campaign |
| `pdays` | Campaign | Days since last contact from a previous campaign (blank = never contacted, 36,954 rows) |
| `previous`, `poutcome` | Campaign | Earlier contacts and outcome (success / failure / other / unknown) |
| `y` | Target | Subscribed to term deposit |
| **Derived:** `Age_Group` | Feature | Under 25, 25-34, 35-44, 45-54, 55-64, 65+ |
| **Derived:** `Balance_Segment` | Feature | Negative, Low (0-499), Medium (500-1,998), High (2,000-9,997), Very High (10,005+) |
| **Derived:** `Has_Any_Loan` | Feature | Yes if housing or personal loan |
| **Derived:** `Risk_Flag` | Feature | High / Medium / Low risk (see note below) |
| **Derived:** `Month_Num`, `Sub_Num`, `Subscribed` | Feature | Month number (for sorting) and 1/0 target flags |

> **Risk_Flag note:** all 815 **High Risk** customers are the ones with `default_status = Yes`. The other 44,396 customers are split into Medium and Low risk by a rule applied in Excel. <!-- TODO: add the exact Medium/Low rule used in your Excel formula -->

---

## 3. Project Workflow & Repository Structure

```
 Raw CSV  ──►  Excel (clean, pivots, scenarios)  ──►  SQL Server (analysis layer)
                                                         │
                          Power BI dashboard  ◄──  Python (EDA, ML, RFM) ◄─┘
```

```
Banking-Customer-Analytics-Risk-Segmentation/
├── 01_data/
│   ├── raw/                  Raw_Bank_Customers.csv
│   └── cleaned/              Bank_Customers.csv
├── 02_excel/                 Bank_Customers_clean.xlsx
├── 03_sql/
│   ├── Bank_Schema.sql
│   ├── Analytical_Queries_sql.sql
│   ├── Advanced_Queries_sql.sql
│   ├── Views.sql
│   ├── Stored_Procedures.sql
│   └── Bonus_Insights.sql
├── 04_python/
│   ├── 01_eda.py
│   ├── 02_eda_advanced.py
│   ├── 03_churn_prediction.py
│   └── 04_rfm_segmentation.py
├── 05_outputs/
│   ├── charts/               19 EDA/ML/RFM charts + 5 dashboard pages
│   ├── churn_predictions.csv
│   ├── rfm_segments.csv
│   └── rfm_summary.csv
├── 06_powerbi/               Bank_Analytics.pbix
├── requirements.txt
└── README.md
```

---

## 4. Step 1 – Raw Data

- **File:** `Raw_Bank_Customers.csv` – 45,211 rows × 24 columns.
- **Quality checks:**
  - No missing values except `pdays`, which is blank for **36,954 customers (81.7%)**. These were never contacted in a prior campaign, so the blank is meaningful, not an error.
  - 815 customers have `default_status = Yes`, and 3,766 have a **negative balance** (overdrawn).
  - `Unknown` appears in `Job_Title`, `Education`, `Contact` and `poutcome`. It is kept as its own category and not dropped.
- **Skew:** `Balance` is heavily right-skewed, with most customers near $0 and a long tail up to $102,127 (see Chart 02). Averages are therefore pulled up by a few very large accounts, so medians matter too.

---

## 5. Step 2 – Excel: Cleaning, Pivots & Scenario Analysis

**File:** `Bank_Customers_clean.xlsx` – 5 sheets.

### 5.1 Cleaning (Power Query)
- Standardised text to proper case (`blue-collar` → `Blue-Collar`, `jul` → `July`).
- Renamed columns to analyst-friendly names (`job` → `Job Title`, `default` → `default_status`).
- Kept `pdays` blank for never-contacted customers, replacing the original `-1` code.
- Added 7 derived columns: `Age_Group`, `Balance_Segment`, `Has_Any_Loan`, `Risk_Flag`, `Month_Num`, `Sub_Num`, `Subscribed`.
<!-- TODO: confirm/adjust these cleaning steps to match your actual Power Query steps -->

### 5.2 Pivot Analysis (`Pivot_Analysis` sheet)
Pivots built for: subscription rate by **job**, balance by **balance segment**, **risk** distribution, **education** vs balance vs subscription, **monthly** performance, and subscription by **age group**.

| Insight | Result |
|---|---|
| Highest-converting jobs | Student **28.7%**, Retired **22.8%**, Unemployed 15.5% |
| Lowest-converting job | Blue-Collar **7.3%** |
| Best education level | Tertiary: **15.0%** subscription, **$1,758** avg balance |
| Best months | March 52.0%, Dec 46.7%, Sep 46.5%, Oct 43.8% (all on small volumes, see §12) |
| Worst month | May 6.7% (13,766 contacts, the heaviest month) |

### 5.3 Scenario Analysis (`Scenario_Analysis` + `Scenario Summary` sheets)
Built with Excel **Scenario Manager**, with the target subscription rate as the changing cell.

| Scenario | Subscription rate | Subscribers | Deposit-value potential* |
|---|---|---|---|
| Current | 11.7% | 5,290 | $26.4M |
| Optimistic | 20% | 9,042 | $45.2M |
| Aggressive | 30% | 13,563 | $67.8M |

\*Assumes **$5,000 per deposit**, a planning assumption and not a measured figure. Moving from 11.7% to 20% means about **+3,753 subscribers** (about **+$18.8M** under that assumption).

---

## 6. Step 3 – SQL Server Analysis

**Database:** `BankAnalyticsDB` · **Table:** `Bank_Customers` (24 columns) · **Tool:** SQL Server Management Studio

| File | What it contains |
|---|---|
| `Bank_Schema.sql` | Table definition, row-count check (45,211), data-type check |
| `Analytical_Queries_sql.sql` | 8 core business queries |
| `Advanced_Queries_sql.sql` | 3 advanced queries (CTEs + window functions) |
| `Views.sql` | 2 reusable reporting views |
| `Stored_Procedures.sql` | 2 parameterised procedures |
| `Bonus_Insights.sql` | 4 targeting insights |

### 6.1 Core queries
| # | Business question | Techniques |
|---|---|---|
| 1 | Overall subscription rate | `COUNT`, `SUM`, `CAST` |
| 2 | Subscription rate by job | `GROUP BY`, `ORDER BY` |
| 3 | Default rate by job | Conditional aggregation (`CASE WHEN`) |
| 4 | Balance by education (avg / max / min) | Aggregates |
| 5 | Risk distribution with % of total | `SUM(COUNT(*)) OVER()` |
| 6 | Monthly campaign performance | Grouping + `Month_Num` sort |
| 7 | Age-group analysis | Aggregates + conditional sums |
| 8 | Balance-segment analysis | Aggregates |

### 6.2 Advanced queries
1. **Customer value ranking.** A 100-point score (balance 40% + subscription 40% + risk 20%), using `RANK()` and `NTILE(4)` to assign **Platinum / Gold / Silver / Bronze** tiers.
2. **Campaign effectiveness.** Uses `LAG()` for month-over-month change in subscription rate, plus **running total** and **running subscription rate** with `SUM() OVER (ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW)`.
3. **Customer segmentation.** Multiple CTEs and `PERCENT_RANK()` build segments: *Premium Loyal, High Value Prospect, Mid Value Loyal, Mid Value Prospect, Growing Customer, Entry Level*.

### 6.3 Views and stored procedures
- `vw_Customer_Summary`: customers, balance, subscription and default rates by job × marital × education × age group × balance segment × risk.
- `vw_Campaign_Performance`: monthly and channel performance, including average call duration, contacts per customer and previous-campaign successes.
- `sp_GetCustomersByRisk (@RiskLevel, @MinBalance, @TopN)`: pulls a call or collections list by risk level.
- `sp_SegmentAnalysis (@SegmentType, @SegmentValue)`: drill-down by `job`, `education` or `age_group`.

### 6.4 Bonus insights
Best and worst customer profiles (job × education × age × balance, min 50 customers), impact of previous campaign outcome, and subscription by call-duration band.

**SQL findings (verified):**
- Previous-campaign **Success** customers convert at **64.7%**, against 9.2% for Unknown.
- Calls **over 10 minutes** convert at **48.4%**; calls under 1 minute at 0.2%.
- Customers **without a housing loan** subscribe at **16.7%** vs **7.7%** with one.
- **Low-risk** customers subscribe at 12.9%, **High-risk** at 6.4% and Medium-risk at 5.8%.

---

## 7. Step 4 – Python: EDA, Prediction Model & RFM

**Libraries:** `pandas`, `numpy`, `matplotlib`, `seaborn`, `squarify`, `scikit-learn`

| Script | Purpose | Outputs |
|---|---|---|
| `01_eda.py` | Core EDA | Charts 01-08 |
| `02_eda_advanced.py` | Education, call duration, previous outcome, job treemap | Charts 09-12 |
| `03_churn_prediction.py` | Subscription propensity model | Charts 13-15, `churn_predictions.csv` |
| `04_rfm_segmentation.py` | RFM customer segmentation | Charts 16-19, `rfm_segments.csv`, `rfm_summary.csv` |

### 7.1 Exploratory analysis
| Chart | Finding |
|---|---|
| ![](05_outputs/charts/01_age_distribution.png) | Mean age 40.9; most customers are 30 to 45. |
| ![](05_outputs/charts/02_balance_distribution.png) | Balances cluster near $0 with a long right tail. |
| ![](05_outputs/charts/03_subscription_by_job.png) | Students and retirees convert far above the 11.7% average; blue-collar converts below it. |
| ![](05_outputs/charts/04_balance_by_subscription.png) | Subscribers have a higher median balance. |
| ![](05_outputs/charts/05_correlation_heatmap.png) | `Duration` has the strongest correlation with subscription (**0.39**); other numeric variables are weak. |
| ![](05_outputs/charts/06_subscription_by_age.png) | U-shaped by age: **65+ (42.1%)** and **Under 25 (25.6%)** convert best, though each group has only about 800 customers. |
| ![](05_outputs/charts/07_monthly_performance.png) | Rate and volume move in opposite directions: May has the most calls and one of the lowest rates. |
| ![](05_outputs/charts/08_risk_distribution.png) | 83.0% Low, 15.2% Medium, 1.8% High risk. |
| ![](05_outputs/charts/09_education_analysis.png) | Tertiary education has the highest balance and conversion. |
| ![](05_outputs/charts/10_call_duration_impact.png) | Conversion rises steadily with call length. |
| ![](05_outputs/charts/11_previous_outcome.png) | Past success is the strongest segment signal (64.7%). |
| ![](05_outputs/charts/12_customer_segments_treemap.png) | Management and blue-collar are the largest groups; blue-collar converts poorly. |

### 7.2 Subscription propensity model (Random Forest)
> This is a **propensity-to-subscribe** model. The dashboard page is titled "Churn & Risk", but the model predicts term deposit subscription, not customer churn.

| Setting | Value |
|---|---|
| Algorithm | Random Forest Classifier (100 trees, `max_depth=10`, `class_weight='balanced'`) |
| Features (14) | Age, Balance, Duration, Campaign, previous, Job, Marital, Education, default, Housing, Loan, Contact, Month, poutcome |
| Encoding | `LabelEncoder` on categorical columns |
| Split | 80/20 stratified (36,168 train / 9,043 test), `random_state=42` |

**Results on the 9,043 unseen customers**

| Metric | Value |
|---|---|
| Accuracy | **84.3%** |
| AUC | **0.916** |
| Recall (subscribers) | **84%** (889 of 1,058 found) |
| Precision (subscribers) | **42%** |
| F1 (subscribers) | 0.56 |

| | Predicted No | Predicted Yes |
|---|---|---|
| **Actual No** | 6,738 | 1,247 (wasted calls) |
| **Actual Yes** | 169 (missed) | 889 |

![Confusion matrix](05_outputs/charts/13_confusion_matrix.png)
![Feature importance](05_outputs/charts/14_feature_importance.png)
![ROC curve](05_outputs/charts/15_roc_curve.png)

**Reading the result.** The model catches 84% of subscribers, but only 42% of the customers it flags actually subscribe. That's useful for *ranking* call lists, not for a hard yes/no. Predictions are bucketed by probability: Low (5,217), Medium (1,690), High (1,408), Very High (728).

### 7.3 RFM segmentation
| Dimension | Definition in this project |
|---|---|
| **R**ecency | `pdays` (days since last contact) |
| **F**requency | `Campaign + previous` (total contacts) |
| **M**onetary | `Balance` (negative values set to 0) |

Each dimension is scored 1-5 by quintile, then customers are assigned a segment by rule.

| Segment | Customers | % of base | Avg balance | Subscription rate |
|---|---|---|---|---|
| Champions | 1,558 | 3.4% | $3,366 | **31.6%** |
| Loyal Customers | 7,148 | 15.8% | $2,837 | 12.4% |
| Potential Loyalists | 5,206 | 11.5% | $454 | 11.7% |
| At Risk | 1,982 | 4.4% | $1,818 | 16.6% |
| Need Attention | 11,232 | 24.8% | $2,675 | 12.9% |
| Lost | 18,085 | 40.0% | $3 | 8.5% |

![RFM segments](05_outputs/charts/16_rfm_segments.png)
![Subscription by segment](05_outputs/charts/17_segment_subscription.png)
![RFM heatmap](05_outputs/charts/18_rfm_heatmap.png)
![Balance by segment](05_outputs/charts/19_segment_balance.png)

**Takeaways:** Champions convert at nearly 3× the average. **Need Attention** (24.8% of customers, $2,675 average balance, 12.9% conversion) is the largest high-balance group with room to improve. 40% of customers are "Lost" with almost no balance. See §12 for a caveat on how recency was scored.

---

## 8. Step 5 – Power BI Dashboard

**File:** `Bank_Analytics.pbix` · **5 pages** · interactive slicers on Education, Job, Marital Status, Segment, Risk Flag, Risk Category, Month and Contact.

**Data loaded** <!-- TODO: confirm sources -->
- `Bank_Customers`: cleaned customer table (Excel / SQL)
- `rfm_summary` and `rfm_segments`: from `04_rfm_segmentation.py`
- `churn_predictions`: from `03_churn_prediction.py`

| Page | Purpose | Headline KPIs |
|---|---|---|
| 1. Executive Summary | Overall performance, who converts, risk mix | 45,211 customers · 11.7% subscription · $1,362 avg balance · 1.8% default · 815 high risk · 1,558 Champions |
| 2. Customer Segmentation | RFM segments by size, conversion and value | Champions 1,558 · Loyal 7,148 · At Risk 1,982 · Lost 18,085 |
| 3. Churn & Risk Analysis | Risk mix and model outcomes | 815 high risk · 84% accuracy · 728 very-high-probability · 2,136 predicted subscribers |
| 4. Campaign Performance | Timing, channel, call behaviour | 11.7% rate · 4.3 min avg call · 2.76 contacts per customer · 1,511 previous successes |
| 5. ML Predictions | Model quality and top-scored customers | 84% accuracy · 0.92 AUC · 84% recall · 42% precision |

---

## 9. Step 6 – Dashboard Screenshots

### Page 1 – Executive Summary
![Page 1](05_outputs/charts/dashboard_p1.png)
Subscription rate peaks in March and falls to 6.7% in May. Tertiary-educated customers hold the highest balances. Subscribers hold larger balances than non-subscribers in every education group.

### Page 2 – Customer Segmentation
![Page 2](05_outputs/charts/dashboard_p2.png)
Champions convert at 32%; the Lost segment (18.1K customers) holds virtually no balance.

### Page 3 – Churn & Risk Analysis
![Page 3](05_outputs/charts/dashboard_p3.png)
The 815 high-risk customers are all in default. The drill-through table lists them by job, age, education and balance.

### Page 4 – Campaign Performance
![Page 4](05_outputs/charts/dashboard_p4.png)
Conversion climbs with call duration and is highest for customers with a previous success. Cellular contacts make up 64.8% of the base.

### Page 5 – ML Predictions
![Page 5](05_outputs/charts/dashboard_p5.png)
Model KPIs, feature importance, the confusion-matrix breakdown and the highest-probability customers for call prioritisation.

---

## 10. Consolidated Key Findings

**Conversion**
- Overall subscription rate: **11.7%** (5,290 of 45,211).
- Top jobs: Student **28.7%**, Retired **22.8%**. Lowest: Blue-Collar **7.3%**.
- Best education: Tertiary (**15.0%**, avg balance $1,758).
- Best age groups: **65+ (42.1%)** and Under 25 (25.6%), but both are small (about 800 customers each).

**Campaign**
- Previous-campaign success converts at **64.7%** (1,511 customers).
- Calls over 10 minutes convert at **48.4%**; under 1 minute at 0.2%.
- Highest monthly rates (March, September, October, December) all come from 214 to 738 contacts. May has 13,766 contacts at 6.7%.
- Customers without a housing loan convert at **16.7%** vs **7.7%** with one.

**Risk**
- Default rate **1.8%** (815 customers); every High-Risk customer is a defaulter.
- Of those 815, 440 (54%) have a negative balance, 116 have $0 and 259 have a positive balance.
- High-risk customers subscribe at 6.4%, half the rate of low-risk customers (12.9%).

**Segments and model**
- Champions (3.4% of customers) convert at 31.6% with $3,366 average balance.
- Random Forest: **84.3% accuracy, 0.916 AUC, 84% recall, 42% precision**. Call duration is the top predictor (50.4%).

---

## 11. Business Recommendations

1. **Re-target previous successes.** They convert at 64.7%; a small, cheap list with the best return.
2. **Build call quality, not call length.** Longer calls convert more, but interested customers also talk longer. Train agents to engage and handle objections, and don't set duration targets.
3. **Re-balance campaign timing.** May takes the most calls at the lowest conversion. Run controlled tests in March, September, October and December before scaling, because those results rest on small samples.
4. **Prioritise high-converting segments.** Students, retirees and tertiary-educated customers, plus customers without a housing loan.
5. **Use the model to rank call lists.** Call High and Very High probability customers first. With 84% recall, most real subscribers are in the flagged group.
6. **Re-engage "Need Attention".** About 11,200 customers with an average balance of $2,675 and below-Champion conversion.
7. **Handle high-risk customers separately.** Route the 815 defaulters to risk/collections instead of deposit campaigns.
8. **Size the prize.** Lifting conversion from 11.7% to 20% adds about 3,750 subscribers (about $18.8M deposit value at the assumed $5,000 each).

---

## 12. Known Limitations & Next Steps

These are listed openly so the numbers can be read in the right context.

| # | Limitation | Impact | Planned fix |
|---|---|---|---|
| 1 | **RFM recency.** `pdays` is blank for 36,954 customers, and the script only recodes `-1`. Blank values received a default recency score of 3. | Recency is not meaningful for about 82% of customers; segment sizes and rates (esp. Need Attention, Lost) are partly driven by this default. | Recode never-contacted customers to a "never contacted" value (e.g. 999) before scoring and re-run. |
| 2 | **Data leakage.** `Duration` (call length) is only known after the call. | Model strength is overstated for pre-call targeting. | Re-train without `Duration` and compare AUC. |
| 3 | **Low precision (42%).** | 1,247 false positives on the test set. | Tune the threshold and test gradient boosting. |
| 4 | **Small-sample months and age groups.** March, Sep, Oct, Dec and the 65+ / Under 25 groups have few records. | High rates may not hold at scale. | Add confidence intervals. |
| 5 | **Scenario revenue** uses an assumed $5,000 per deposit. | Dollar figures are illustrative. | Replace with actual average deposit size. |
| 6 | **Excel pivot issues.** The age-group pivot sums age instead of counting customers, and the education pivot has one mislabelled row. | Pivot totals on those two tables are not valid. | Rebuild the two pivots with *Count of* values. |
| 7 | **Dashboard labels.** The KPI "High Probability Count" (728) is the *Very High Probability* tier, and `Job_Title`/`Month` in the Page 5 table show label-encoded numbers. | Possible confusion for readers. | Rename the KPI and join the decoded values. |
| 8 | **No time dimension.** One campaign snapshot. | Seasonality can't be separated from campaign effects. | Needs multi-year data. |

---

## 13. How to Run

```bash
git clone https://github.com/ManojPalanki/Banking-Customer-Analytics-Risk-Segmentation.git
cd Banking-Customer-Analytics-Risk-Segmentation
pip install pandas numpy matplotlib seaborn squarify scikit-learn
```

1. **Data:** place `Bank_Customers.csv` in `01_data/cleaned/` (or load the cleaned workbook in Excel).
2. **SQL Server:** create `BankAnalyticsDB`, then run `Bank_Schema.sql`, import the CSV into `Bank_Customers`, and run the other `.sql` files in this order: Analytical → Advanced → Views → Stored_Procedures → Bonus.
3. **Python:** update `BASE_PATH` at the top of each script, then run in order:
   ```bash
   python 01_eda.py
   python 02_eda_advanced.py
   python 03_churn_prediction.py
   python 04_rfm_segmentation.py
   ```
4. **Power BI:** open `Bank_Analytics.pbix`, update data-source paths, and refresh.

---

## 14. Skills Demonstrated

| Area | Skills |
|---|---|
| **Excel** | Power Query cleaning, derived columns, pivot tables, Scenario Manager, what-if revenue modelling |
| **SQL** | CTEs, window functions (`RANK`, `NTILE`, `LAG`, `PERCENT_RANK`, running totals), views, stored procedures, conditional aggregation |
| **Python** | pandas, EDA, feature encoding, Random Forest, model evaluation (confusion matrix, ROC/AUC), RFM segmentation |
| **Power BI** | Multi-page dashboard, KPI cards, slicers, drill-through tables, ML output integration |
| **BFSI domain** | Term-deposit marketing, default and credit-risk flags, customer segmentation, campaign effectiveness |
| **Analyst thinking** | Business framing, caveats on sample size and leakage, recommendations tied to numbers |

---

## 15. Author

**Palanki Manoj**: Data / Business Analyst (BFSI)
[LinkedIn](https://www.linkedin.com/in/manoj-palanki/) · [GitHub](https://github.com/ManojPalanki) · [Email](mailto:manojpalanki09@gmail.com)

Project repository: [Banking-Customer-Analytics-Risk-Segmentation](https://github.com/ManojPalanki/Banking-Customer-Analytics-Risk-Segmentation)

*Dataset: Moro, S., Cortez, P., & Rita, P. (2014). A data-driven approach to predict the success of bank telemarketing. UCI Machine Learning Repository.*
