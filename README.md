# customer-churn-analysis
Exploratory data analysis of telecom customer churn using Python (Pandas, Matplotlib, Seaborn): cleaning, feature engineering, retention trends, churn drivers and business insights.

## Objective

Analyze customer behavior to find retention patterns and the main drivers of churn, so that retention strategies can be targeted at the right customers.

## Dataset

- **Source:** [Telco Customer Churn on Kaggle](https://www.kaggle.com/datasets/blastchar/telco-customer-churn/data)
- **Size:** 7,043 customers and 21 columns
- **Target:** `Churn` (Yes / No)
- **Note:** There is no date column, so `tenure` (months with the company) is used as the time axis.

## Repository Structure

```
customer-churn-analysis/
├──Telco_Customer_Churn.csv               # original dataset
├──cleaned_Telco_Customer_Churn.csv       # cleaned and encoded datasets
├── customerChurn_Analysis.ipynb          # full analysis, charts and insights
├── README.md                             # project overview and usage (this file)
├── REPORT.md                             # detailed process documentation
├── requirements.txt                      # Python dependencies

```

## Usage

```bash
jupyter notebook notebooks/customerChurn_Analysis.ipynb
```

Run the cells from top to bottom (**Kernel > Restart & Run All**). The notebook loads the raw CSV from `data/raw/`, so run Jupyter from the repository root or check that the path in the data-loading cell matches your folder layout.

## What the Notebook Covers

1. **Data exploration:** structure, data types, summary statistics, missing values, duplicates, class imbalance and inconsistent entries.
2. **Cleaning and transformation:** fixing `TotalCharges` and `SeniorCitizen` types, handling missing values, IQR outlier checks, and encoding (binary, ordinal and one-hot).
3. **Feature engineering:** tenure groups, number of add-on services, average monthly spend and a protection/support flag.
4. **Churn overview and retention:** churn vs retention rate, and retention by tenure.
5. **Churn drivers:** contract, internet service, add-ons, payment method, billing and demographics.
6. **Correlations:** correlation heatmaps for the numeric and encoded features.
7. **Segmentation:** the highest-risk and most loyal customer segments.
8. **Business insights:** ranked churn drivers, recommended actions, revenue at risk and limitations.

## Key Findings

- **Overall churn is 26.5%**, with 73.5% of customers retained.
- **Contract type is the strongest driver.** Month-to-month customers churn at 42.7%, versus 11.3% on one-year and 2.8% on two-year contracts.
- **Customers leave early.** Churn is 56.2% in the first 3 months and falls to 9.5% for customers with 49-72 months of tenure.
- **Fiber optic and electronic check customers churn the most** (41.9% and 45.3%), while online security and tech support go with much lower churn.
- **The highest-risk segment is month-to-month fiber customers in their first year**, at 70.2% churn.
- **About 30.5% of monthly revenue** (roughly $139K per month) comes from customers who churned.

See [REPORT.md](REPORT.md) for the full methodology, results and recommendations.

## Tools Used

Python, Pandas, NumPy, Matplotlib, Seaborn, Jupyter Notebook, Git and GitHub.

## Limitations

The data is a single snapshot with no dates, and the findings show associations rather than proven causes. Recommended actions should be tested, for example with a control group, before wide rollout. More detail is in [REPORT.md](REPORT.md).
