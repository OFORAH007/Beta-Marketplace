# Beta-Marketplace Sales Analysis


## Tables of Content

- [Project Overview](#Project-Overview)
- [Dataset Sources & Description](#Dataset-Sources-&-Description)
- [Objectives](#Objectives)
- [Tools Used](#Tools-Used)
- [Data Preparation & Cleaning](#Data-Preparation-&-Cleaning)
- [Exploratory Data Analysis](#Exploratory-Data-Analysis)
- [Data Analysis](#Data-Analysis)
- [Key Findings](#Key-Findings)
- [Recommendations](#Recommendations)
- [Limitations](#Limitations)
- [Conclusion](#Conclusion)
- [References](References)



### Project Overview
This repository contains the analysis of the Beta Marketplace dataset, focusing on various aspects of Product, Customer, Location and Financial performance. The primary purpose of this analysis is to derive actionable insights that can help Beta Marketplace optimize its operations, improve customer satisfaction, increase profitability, and finally deeper understanding the company's performance.


### Dataset Sources & Description

Sales Data: The Primary Dataset used for this Analysis is "Beta Marketplace Sales Dataset.csv" file, containing detailed information about eachsales made by the Company.


### Objectives

The main objectives of this analysis are to:
- Understand Product Performance: Identify top-performing and bottom-performing products/subcategories by revenue, profit, and quantity sold.
- Analyze Customer Behavior: Segment customers by gender, region, and purchase patterns to understand customer lifetime value and product preferences.
- Assess Financial Health: Evaluate monthly and daily revenue trends, cost structures, and profit margins to identify periods of growth or decline.
- Understanding regional distribution of various product hierarrchy.
- Identify Opportunities: Uncover insights that can lead to improved inventory management, targeted marketing campaigns, and enhanced financial strategies.


### Tools Used

  - Data Extraction & Analysis - Azure Data Studio [Download here](https://learn.microsoft.com/en-us/azure-data-studio/download-azure-data-studio?tabs=win-install%2Cwin-user-install%2Credhat-install%2Cwindows-uninstall%2Credhat-uninstall)
  - Data Cleaning & Transformation - Excel [Download here](https://www.microsoft.com/en/microsoft-365/excel?market=af)
  - Data Visualization - Power Bi [Download here](https://www.microsoft.com/en-us/download/details.aspx?id=58494)
  - Reporting - Github Markdown for documentation [Access here](https://github.com/)
 

 ### Data Preparation/Cleaning

  In the initial data preparation phase, I performed the following Task;
  1. Handling Missing Values: Identifying and addressing any missing entries in critical columns (e.g., revenue, quantity).
  2. Correcting Data Types: Ensuring all columns are in their appropriate data types (e.g., numerical values are not stored as strings).
  3. Removing Duplicates: Eliminating any duplicate records that might skew the analysis.
  4. Standardizing Formats: Ensuring consistent formatting for categorical data (e.g., "Male" vs. "male").
  5. Outlier Detection: Identifying and deciding how to handle extreme values that could impact statistical measures.


### Exploratory Data Analysis

EDA was conducted to understand the underlying patterns and relationships within the data. This included:

- Summary Statistics: Calculating mean, median, mode, standard deviation for numerical columns.
- Distribution Analysis: Visualizing the distribution of key metrics (e.g., revenue, profit) using histograms and box plots.
- Categorical Analysis: Examining the distribution of categorical variables (e.g., gender, product categories) using bar charts and pie charts.
- Trend Analysis: Visualizing revenue and cost trends over time (monthly, daily).
- Correlation Analysis: Exploring relationships between different variables, such as profit vs. revenue by product category.


### Data Analysis

```SQL
-- CTE Query 1: Calculate Aggregated Profit and Revenue by Product Category
-- This CTE prepares the data by summing up the total revenue and total profit
-- for each product category. This aggregated data can then be used to
-- analyze the relationship between profit and revenue at a category level.
WITH CategoryPerformance AS (
    SELECT
        ProductCategory,
        SUM(TotalRevenue) AS TotalCategoryRevenue,
        SUM(TotalProfit) AS TotalCategoryProfit
    FROM
        SalesData -- Assuming 'SalesData' is your main sales table
    GROUP BY
        ProductCategory
)
-- You can then select from this CTE to perform further analysis or visualization
SELECT
    ProductCategory,
    TotalCategoryRevenue,
    TotalCategoryProfit
FROM
    CategoryPerformance;
```


```SQL 2
-- CTE Query 2: Calculate Monthly Performance and Percentage Change
-- This CTE first aggregates daily sales data to a monthly level, and then
-- calculates the month-over-month percentage change for both revenue and profit.
-- This helps in understanding trends and the correlation of changes over time.
WITH MonthlySales AS (
    SELECT
        STRFTIME('%Y-%m', SaleDate) AS SaleMonth, -- Adjust date function based on your SQL dialect (e.g., FORMAT(SaleDate, 'yyyy-MM') for SQL Server)
        SUM(TotalRevenue) AS MonthlyRevenue,
        SUM(TotalProfit) AS MonthlyProfit
    FROM
        SalesData
    GROUP BY
        SaleMonth
),
MonthlyGrowth AS (
    SELECT
        SaleMonth,
        MonthlyRevenue,
        MonthlyProfit,
        LAG(MonthlyRevenue, 1, 0) OVER (ORDER BY SaleMonth) AS PreviousMonthRevenue,
        LAG(MonthlyProfit, 1, 0) OVER (ORDER BY SaleMonth) AS PreviousMonthProfit
    FROM
        MonthlySales
)
SELECT
    SaleMonth,
    MonthlyRevenue,
    MonthlyProfit,
    (MonthlyRevenue - PreviousMonthRevenue) * 100.0 / PreviousMonthRevenue AS RevenuePercentageChange,
    (MonthlyProfit - PreviousMonthProfit) * 100.0 / PreviousMonthProfit AS ProfitPercentageChange
FROM
    MonthlyGrowth
WHERE
    PreviousMonthRevenue > 0; -- Avoid division by zero
```


```SQL
-- Window Query: Calculate Running Total of Revenue by Product Category
-- This query uses a window function to calculate the running total of revenue
-- for each product category, ordered by date. This can be useful for
-- visualizing cumulative performance and understanding how different
-- categories contribute to overall revenue over time.
SELECT
    SaleDate,
    ProductCategory,
    TotalRevenue,
    SUM(TotalRevenue) OVER (PARTITION BY ProductCategory ORDER BY SaleDate) AS RunningCategoryRevenue
FROM
    SalesData
ORDER BY
    ProductCategory, SaleDate;
```


### Key Findings

Based on the analysis, several Key findings  emerged;
- Product Performance
  - The Unit cost is consistently lower than unit price, indicating profitability margins.
  - Juice, Dishwasher and Soft drink have a noticeable gap between Price and Cost indicating good profit margin.
  - Customers spend more on Household, Beverages, Fashion products with total revenue of  N35,630,869.
  - Despite being part of profitable categories (Vegetable Oil), these specific products have low profit margins, possibly due to high unit cost, low selling price or low volume of sales.

- Customer Insight
   - Customer IDs (CUST3292, CUST75107, CUST54742) with high Life Time Value (LTV) should be targeted for marketting effort.
   - Low Repeat Rate: 0.00 repeat rate suggests a need for customer retention strategies.
   - Customer Gender Distribution: Relatively balanced (49.7% Male, 50.3% Female).
   - Top States by Customers: FCT, Kwara, Taraba, Bauchi, Katsina.

- Regional Distribution
   - Unit price variations suggest possible regional differences in product value, affordability, or distribution costs.
   - North West and North Central States characterized with lower average unit price of products that result to generation of more revenue.
   - South West and South East lag behind in total revenue despite being major economic zone in Nigeria, which may require further analysis.

- Financial Overview
   - Revenue fluctuates throughout the year, with peaks in certain months (e.g., November, December) and days (Sunday).
   - A positive correlation between Profit and Revenue across product categories.


### Recommendations

- Reassess and optimize pricing strategy for low-profit products by observing the bottom 5 products by products. Investigate whether high unit costs, discount or operational inefficiencies are eroding profit margins.
- Focus Marketing and stocking  strategy on high revenue & Margin Products(Juice, dishwashers & detergent). Prioritize these products in promotion, inventory planning and ad spend. Consider cross selling them with other related products to drive even more value.
- Consider analyzing why South-West and South-East lag in revenue despite economic potential. Also investigate the pricing strategy in states  with low unit prices for potential margin optimization.
- Boost Marketing around June to mitigate sales dips.
- Analyze the customer behavior further to confirm reasons for mid-year and year-end trends.
- Plan inventory and staffing around peak months (January, September and October).


### Limitations

- Limited Historical Data: The analysis appears to be for a specific period (likely one year), limiting long-term trend analysis.
- Missing Context: Without business context, some insights might be difficult to interpret fully (e.g., reasons for low repeat rate) and also there are limitted variables to find why sales increased or dropped(no promotion season & no competitor data)
- I made sure that the data formats like date, currency, regions etc which would have created problem in grouping, filtering, and sorting.


### Conclusion

This analysis provides a comprehensive overview of Beta Marketplace's product, customer, and financial performance. The insights gained can guide strategic decisions, such as optimizing product offerings, tailoring marketing efforts to specific customer segments, and improving financial planning. Addressing the low repeat rate and understanding the cost structures of less profitable products are crucial next steps for sustainable growth.


### References

1. [Kaggle – Data Cleaning and EDA Guide](https://www.kaggle.com/code/farhanullahkhan/eda-data-cleaning)
2. [Towards Data Science – Sales Data Analysis Tutorial](https://towardsdatascience.com/sales-analytics-in-python-df6f0eb90e5e)
3. [Microsoft Learn – Analyze data with Power BI](https://learn.microsoft.com/en-us/training/paths/analyze-data-power-bi/)











 






 



 
