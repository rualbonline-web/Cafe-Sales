Overview
A café owner needs trustworthy answers to basic operational questions: Which products bring in the most revenue? How do sales vary over time? Which payment methods and order locations account for revenue? Before answering, the transaction data must be made reliable.

This project follows an end-to-end data cleaning → validation → exploratory data analysis (EDA) → business reporting workflow using Python. It is based on the Cafe Sales — Dirty Data for Cleaning Training dataset: 10,000 synthetic café transactions from 2023, deliberately containing missing values, inconsistent data types, and ERROR / UNKNOWN placeholders.
Scope: This is an educational analysis of synthetic data, not a report on a real operating café. The figures below come from the executed outputs saved in the project notebook.

Key results
Metric	Result
Original transactions	10,000
Transactions retained after cleaning	9,514
Records removed	486 (4.9%)
Total revenue in cleaned data	$84,868.50
Average transaction value	$8.92
Highest-revenue item	Salad — $18,220.00
Highest-volume item	Coffee — 3,766 units
Highest-revenue month	June — $7,353.00
Lowest-revenue month	February — $6,644.00


The cleaned dataset contains no remaining null values after unrecoverable rows were removed and unknown categories were explicitly labelled. Importantly, an unknown category does not mean that the original information was recovered.
Business questions
1. Which menu items sell the most units, and which generate the highest revenue?
2. How is revenue distributed across payment methods and in-store versus takeaway orders?
3. Which months and weekdays perform best, and what does the daily trend look like?
4. Are unusually large purchases data errors or valid transactions?
5. How strongly do quantity and unit price relate to transaction value?
Dataset
Source: Cafe Sales — Dirty Data for Cleaning Training (Kaggle)

Input: dirty_cafe_sales.csv — 10,000 rows × 8 columns
Raw column	Meaning
Transaction ID	Transaction identifier
Item	Menu item sold
Quantity	Number of units
Price Per Unit	Unit price (USD)
Total Spent	Transaction value (USD)
Payment Method	Cash, Credit Card, Digital Wallet, or missing/invalid
Location	In-store, Takeaway, or missing/invalid
Transaction Date	Transaction date


The notebook preserves a copy of the original data before applying any transformations.
Approach
1. Data profiling
- Inspect the shape, data types, category distributions, and invalid numeric/date entries.
- Check for fully duplicated rows and repeated transaction IDs (none found in this run).
- Convert the placeholder strings ERROR and UNKNOWN to nulls for consistent missing-data analysis.
- Measure and visualize missingness. Initially, 39.6% of location values and 31.8% of payment-method values were missing or invalid after normalizing placeholders.
2. Cleaning and reconstruction
- Convert field names to snake_case; parse numeric values and transaction dates.
- Reconstruct missing numbers where supported by known menu prices or the equation:
  total_spent = quantity × price_per_unit
- Infer an item only when its unit price identifies a unique menu item.
- For ambiguous prices ($3 for Cake/Juice and $4 for Sandwich/Smoothie), compare purchase patterns by location and weekend before applying proportional random assignment with a reproducible seed of 42.
- Track these estimates in the boolean item_guessed column: 450 retained rows (4.7%) contain inferred product labels.
- Keep missing payment methods, locations, and unresolved product labels as Unknown, rather than claiming to know them.
- Remove rows with unrecoverable essential date or numeric fields. The final dataset retains 9,514 of 10,000 rows.
- Engineer month, month_name, weekday, weekday_name, and is_weekend fields.
- Validate transaction arithmetic and save the cleaned data as cafe_sales_clean.csv.
3. Exploratory analysis
Analysis	What the notebook produces
Product performance	Units sold and revenue by menu item; comparison with and without inferred item labels
Payment and location	Revenue breakdowns and separate reporting of Unknown shares
Time trends	Monthly revenue, weekday average transaction value, daily sales, and a 7-day moving average
Outliers	Box plots and the interquartile range (IQR) rule, followed by transaction-level validation
Relationships	Correlation matrix for quantity, unit price, and total transaction value


4. Business reporting
The notebook creates a final management summary with actual computed KPIs, practical recommendations, and limitations of the analysis.
Insights from the notebook
Product mix. Salad produces the most revenue ($18,220), while Coffee leads by units sold (3,766). Removing the proportionally guessed product labels does not change the product revenue ranking in the saved analysis, but the estimated labels should still be treated cautiously.

Revenue over time. June is the strongest month ($7,353) and February is the weakest ($6,644). Average transaction value is nearly identical on weekdays ($8.93) and weekends ($8.89), so this dataset does not indicate a meaningful weekend uplift in spend per transaction.

Data quality matters for channel analysis. Transactions with unknown payment methods account for 31.0% of revenue, and transactions with unknown order location account for 39.6%. Among transactions with a known location, in-store contributes 50.5% of revenue versus 49.5% for takeaway. These conditional shares should not be generalized to the unknown group.

Outliers are not necessarily mistakes. The IQR rule flags 259 transactions (2.72%) with value above $24. All are valid $25 purchases of five Salads at $5 each, so the notebook retains them rather than automatically discarding genuine sales.

Revenue relationships. In the cleaned data, revenue correlates with quantity (r = 0.704) and unit price (r = 0.646), as expected from the transaction-value formula. Correlation does not establish a causal business effect.
Recommendations
- Protect availability of high-revenue products: Monitor Salad inventory and demand; distinguish revenue contribution from profitability, since costs are not provided.
- Improve point-of-sale data collection: Require valid payment-method and order-location selections where feasible. High unknown shares weaken channel-level decisions.
- Review monthly trends before changing staffing or promotions: June and February differ, but one year of synthetic transactions is insufficient to establish repeatable seasonality.
- Validate outliers before excluding them: Preserve legitimate bulk purchases and investigate exceptions only when pricing or quantity checks fail.
Tech stack
- Python — analysis workflow
- pandas — cleaning, grouping, validation, and aggregation
- NumPy — numerical operations and reproducible sampling
- Matplotlib — visualizations
- Jupyter Notebook / Google Colab — interactive analysis and reporting
Run the analysis
Google Colab
1. Open the analysis .ipynb notebook from this repository in Google Colab.
2. Run the cells from top to bottom.
3. The notebook first attempts to load dirty_cafe_sales.csv using the GitHub raw-file URL set in FILE_IN. It also has a fallback URL and a local-file option.
4. If remote access fails, upload dirty_cafe_sales.csv in Colab and use FILE_IN = "dirty_cafe_sales.csv".
5. Examine the generated charts, summary, and cleaned CSV.
Run locally
git clone https://github.com/rualbonline-web/Cafe-Sales.git
cd Cafe-Sales
python -m pip install pandas numpy matplotlib notebook
jupyter notebook
Open the analysis notebook and run its cells in sequence. For an offline run, keep the input CSV alongside the notebook and set:
FILE_IN = "dirty_cafe_sales.csv"
FILE_OUT = "cafe_sales_clean.csv"
Main files
File	Role
dirty_cafe_sales.csv	Raw transaction dataset read by the notebook
Analysis notebook (.ipynb)	Cleaning, validation, EDA plots, and final business summary
cafe_sales_clean.csv	Generated output saved by the notebook (not necessarily committed)
README.md	Project overview, findings, and run instructions


Limitations
- Synthetic data: Findings illustrate the workflow, not real café performance.
- Incomplete records: 486 rows were excluded because essential values could not be reconstructed; this can affect aggregate results.
- Estimated product labels: Ambiguous items are assigned probabilistically and flagged. They are not directly observed.
- Unknown channels: Missing payment and location values remain explicitly unknown; channel-specific conclusions apply only to known records.
- Revenue is not profit: The dataset contains no cost, margin, or customer identifiers; this analysis cannot measure profitability or individual customer behavior.
- Single year: Month-to-month variation in 2023 should not be presented as established seasonality.
Author
Created by @rualbonline-web as a Python data analytics portfolio project.
