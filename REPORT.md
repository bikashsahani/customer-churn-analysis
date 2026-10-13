# Customer Retention Analysis: Process Report

This report summarises the dataset, the cleaning and transformation strategy, and the exploratory data analysis (EDA) results for the Telco customer churn project. The full code and charts are in `notebooks/customerChurn_Analysis.ipynb`.

---

## 1. Dataset Overview

| Item | Detail |
|---|---|
| Source | [Telco Customer Churn (Kaggle)](https://www.kaggle.com/datasets/blastchar/telco-customer-churn/data) |
| Size | 7,043 customers, 21 columns (one row per customer) |
| Target | `Churn` (Yes / No) |

**Column groups**
- **Demographics:** `gender`, `SeniorCitizen`, `Partner`, `Dependents`
- **Account:** `tenure`, `Contract`, `PaperlessBilling`, `PaymentMethod`, `MonthlyCharges`, `TotalCharges`
- **Services:** `PhoneService`, `MultipleLines`, `InternetService`, `OnlineSecurity`, `OnlineBackup`, `DeviceProtection`, `TechSupport`, `StreamingTV`, `StreamingMovies`

**Key characteristics**
- **Class imbalance:** 1,869 customers churned (26.5%) and 5,174 stayed (73.5%).
- **Numeric summary:**

  | Feature | Mean | Median | Min | Max |
  |---|---|---|---|---|
  | `tenure` (months) | 32.4 | 29 | 0 | 72 |
  | `MonthlyCharges` | 64.8 | 70.4 | 18.25 | 118.8 |
  | `TotalCharges` | 2,283.3 | 1,397.5 | 18.8 | 8,684.8 |

- **Data quality:**
  - No duplicate rows.
  - `TotalCharges` was stored as text and had 11 blank values.
  - `SeniorCitizen` was stored as 0/1 while the other binary columns use Yes/No.
  - Seven service columns used a third label ("No internet service" or "No phone service") that duplicates "No".

---

## 2. Cleaning and Transformation Strategy

| Issue | Action | Justification |
|---|---|---|
| `TotalCharges` stored as text | Converted with `pd.to_numeric(errors="coerce")` | It is a numeric amount; blanks become NaN so they can be handled explicitly |
| 11 missing `TotalCharges` | Filled with 0 | All 11 customers have `tenure = 0`, so they are new and have not been billed yet. Dropping them would lose valid customers. |
| `SeniorCitizen` as 0/1 | Mapped to No / Yes | Consistent with the other binary columns |
| Duplicates | None found | No action needed |
| Outliers | Checked `tenure`, `MonthlyCharges` and `TotalCharges` with box plots and the IQR rule; none found | All values fall within plausible business ranges, so nothing was capped or removed |
| "No internet service" / "No phone service" | Merged into "No" in the encoded dataset | The information is already captured by `InternetService` and `PhoneService` |

**Encoding**

| Technique | Columns |
|---|---|
| Binary mapping (Yes/No → 1/0) | `SeniorCitizen`, `Partner`, `Dependents`, `PhoneService`, `PaperlessBilling`, `Churn` and the seven service columns; `gender` is mapped Female = 0, Male = 1 |
| Ordinal encoding | `Contract`: Month-to-month = 0, One year = 1, Two year = 2 |
| One-hot encoding (`drop_first=True`) | `InternetService`, `PaymentMethod` |

**Engineered features**
- `tenure_group`: 0-12, 13-24, 25-48 and 49-72 months.
- `num_addons`: count of add-on services (0 to 6).
- `avg_monthly_spend`: `TotalCharges / tenure`, falling back to `MonthlyCharges` when tenure is 0.
- `has_protection_support`: 1 if the customer has any of security, backup, device protection or tech support.

**Scaling:** not applied. The EDA relies on counts, rates and correlations, which do not need scaled inputs. Scaling (for example `StandardScaler` on `tenure`, `MonthlyCharges` and `TotalCharges`) should be added before any distance-based or regularised model.

---

## 3. EDA Results

### Overall retention
- Churn rate **26.5%**, retention rate **73.5%**.
- Churned customers have a much shorter tenure (mean 18.0, median 10 months) than retained customers (mean 37.6, median 38 months).
- Churned customers pay more per month on average (74.44 vs 61.27).

### Churn by tenure (retention over time)

| Tenure (months) | Churn rate |
|---|---|
| 0-3 | 56.2% |
| 4-6 | 44.6% |
| 7-12 | 35.9% |
| 13-24 | 28.7% |
| 25-48 | 20.4% |
| 49-72 | 9.5% |

Customers are most likely to leave early, and retention rises steadily the longer they stay.

### Churn drivers

| Driver | Highest-risk group | Churn rate | Comparison |
|---|---|---|---|
| Contract | Month-to-month | 42.7% | One year 11.3%, two year 2.8% |
| Tenure | First 12 months | 47.4% | 9.5% for 49-72 months |
| Payment method | Electronic check | 45.3% | 15.2% to 19.1% for other methods |
| Internet service | Fiber optic | 41.9% | DSL 19.0%, no internet 7.4% |
| Online security | No online security | 41.8% | 14.6% with it (internet customers) |
| Tech support | No tech support | 41.6% | 15.2% with it (internet customers) |
| Paperless billing | Paperless | 33.6% | 16.3% with paper billing |
| Demographics | Senior citizens | 41.7% | Partners 19.7% vs no partner 33.0%; dependents 15.5% vs none 31.3% |

**Little or no effect:** `gender`, `PhoneService`, `MultipleLines`, and streaming services (about 3 percentage points difference).

### Correlations
- The numeric features most correlated with churn are `Contract` (-0.40) and `tenure` (-0.35).
- No single numeric feature has a correlation above 0.4, so churn comes from a combination of factors.
- Redundant features: `avg_monthly_spend` duplicates `MonthlyCharges` (r = 1.00), and `TotalCharges` mirrors `tenure` (r = 0.83). Consider dropping one of each pair before modelling.

### Segmentation

| Segment | Customers | Churn rate | Churned |
|---|---|---|---|
| Month-to-month | 3,875 | 42.7% | 1,655 |
| Month-to-month + fiber | 2,128 | 54.6% | 1,162 |
| Month-to-month + fiber + first year | 916 | 70.2% | 643 |
| Senior citizens | 1,142 | 41.7% | 476 |

- **Most loyal:** two-year contract customers (1,695 customers, 2.8% churn).
- **Interaction:** a high monthly price combined with a month-to-month contract pushes churn up sharply, but contract type has the larger effect than price.
- Seniors churn more than non-seniors in every contract type. Customers living alone churn more only on month-to-month contracts.

### Revenue at risk
Churned customers paid about **$139,131 per month**, which is **30.5%** of total monthly revenue ($456,117), or roughly **$1.67 million per year**. The dataset does not state a currency.

---

## 4. Actionable Takeaways

| Priority | Action | Target |
|---|---|---|
| 1 | Offer discounts or perks for 1- or 2-year contracts | Month-to-month customers, starting with fiber |
| 2 | Early-tenure onboarding (check-ins at months 1, 3 and 6, setup help, welcome offer) | Customers in their first 12 months |
| 3 | Bundle or trial online security and tech support | Internet customers without them, especially fiber |
| 4 | Nudge a switch from electronic check to automatic payment (for example a small bill credit) | Electronic check users |
| 5 | Review fiber pricing and service quality, and add loyalty pricing and exit surveys | Fiber customers |
| 6 | Senior-friendly support (assisted setup, easier access to help) | Senior citizens |

**Where to pilot first:** month-to-month fiber customers in their first year (916 customers, 70.2% churn). Offer each action to a random half of the target group and compare churn against the other half, so the effect can be separated from customers who would have stayed anyway.

---

## 5. Limitations

- **Snapshot data:** there is no date column, so retention curves compare different customers at different tenures rather than following one group over time. Active customers may still churn later, so churn is likely understated.
- **Association, not causation:** for example, customers with tech support may already be more committed. Contract, tenure, price and service type overlap heavily.
- **Class imbalance:** any predictive model needs stratified sampling, class weights, or metrics other than accuracy.
- **Small subgroups:** some segments are small, so their percentages are less reliable.
- **Missing context:** there is no data on satisfaction, support contacts, competitor offers or reasons for leaving. Revenue figures cover monthly recurring charges only, not lifetime value.
- **Preparation choices:** filling blank `TotalCharges` with 0 and merging "No internet/phone service" into "No" slightly affect some results.
- **Single public dataset:** results may not transfer to another company or market.
