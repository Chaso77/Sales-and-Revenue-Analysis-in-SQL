# Superstore SQL Analytics: Customer Insights, Cohort Analysis & Revenue Optimization

## Table of Contents

- [Project Overview](#Project-Overview)
- [Data Sources](#Data-Sources)
- [Tools & Skills Used](#-tools--skills-used)

### Project Overview

This project performs an end-to-end business analysis on the Superstore dataset using Microsoft SQL Server.

It covers key analytical areas including:

- Customer Lifetime Value (LTV)
- Cohort & Retention Analysis
- Market Basket Analysis (Cross-Sell)
- Pareto (80/20) Analysis
- Time-Series Forecasting
- Customer Segmentation

The goal is to extract actionable business insights that can drive:

- Revenue growth
- Customer retention
- Product bundling strategies

### Data Sources

Superstore Sales Data: The primary dataset used for this analysis is the “Superstore Sales.xlsx” Excel workbook, which contains detailed information on each transaction made by the store.

### Tools & Skills Used

- SQL Server (T-SQL)
- Window Functions (LAG, LEAD, ROW_NUMBER, RANK)
- CTEs (Common Table Expressions)
- Aggregations & Grouping
- Time-Series Analysis
- Business Analytics

## 1. Customer Lifetime Value (LTV)

```SQL
SELECT 
    CustomerID,
    CustomerName,
    SUM(Sales) AS Total_Revenue,
    SUM(Profit) AS Total_Profit,
    SUM(Sales) / COUNT(DISTINCT OrderID) AS Avg_Order_Value,
    MIN(OrderDate) AS First_Purchase_Date,
    MAX(OrderDate) AS Latest_Purchase_Date,
    DATEDIFF(DAY, MIN(OrderDate), MAX(OrderDate)) AS Lifetime_Duration_Days
FROM Orders
GROUP BY 
    CustomerID,
    CustomerName
ORDER BY 
    Total_Revenue DESC;
```
## 2. Customer Segmentation
  ### Segment Performance

```SQL
SELECT 
    Segment,
    SUM(Sales) AS Total_Sales,
    SUM(Profit) AS Total_Profit,
    SUM(Sales)*1.0/COUNT(DISTINCT OrderID) AS Avg_Order_Value
FROM Orders
GROUP BY Segment;
```

### Top Customers per Segment

```SQL
WITH TOP_Customers AS(
    SELECT 
        CustomerID,
        CustomerName,
        Segment,
        SUM(Sales) AS Total_Sales
    FROM Orders
    GROUP BY 
        CustomerID,
        CustomerName,
        Segment
),
TOP_Rank AS(
    SELECT 
        CustomerID,
        CustomerName,
        Segment,
        Total_Sales,
        ROW_NUMBER() OVER(PARTITION BY Segment ORDER BY Total_Sales DESC) AS Rankd_Customers
    FROM TOP_Customers
)
SELECT *
FROM TOP_Rank
WHERE Rankd_Customers <= 5;
```

## 3. Monthly Sales & Customer Trends
```SQL
WITH CustomersPerMonth AS (
    SELECT 
        FORMAT(OrderDate, 'yyyy-MM') AS YearMonth,
        COUNT(DISTINCT CustomerID) AS Nos_of_Customers,
        SUM(Sales) AS Monthly_Revenue
    FROM Orders
    GROUP BY FORMAT(OrderDate, 'yyyy-MM')
)
SELECT *
FROM CustomersPerMonth
ORDER BY YearMonth;
```

## 4. Market Basket Analysis
### Sub-Category Pairs

```SQL
SELECT 
    o1.SubCategory AS SubCategory_1,
    o2.SubCategory AS SubCategory_2,
    COUNT(DISTINCT o1.OrderID) AS Frequency
FROM Orders o1
JOIN Orders o2 
    ON o1.OrderID = o2.OrderID
    AND o1.SubCategory < o2.SubCategory
GROUP BY 
    o1.SubCategory,
    o2.SubCategory
ORDER BY Frequency DESC;
```

## 5. Pareto Analysis (80/20 Rule)
### Products Driving 80% Revenue

```SQL
WITH ProductSales AS (
    SELECT 
        ProductName,
        SUM(Sales) AS Total_Sales
    FROM Orders
    GROUP BY ProductName
),
RankedProducts AS (
    SELECT 
        ProductName,
        Total_Sales,
        SUM(Total_Sales) OVER() AS Overall_Sales,
        SUM(Total_Sales) OVER(ORDER BY Total_Sales DESC) AS Running_Sales
    FROM ProductSales
)
SELECT 
    ProductName,
    Total_Sales,
    CAST(Running_Sales * 1.0 / Overall_Sales AS DECIMAL(5,4)) AS Cumulative_Percentage
FROM RankedProducts
WHERE 
    CAST(Running_Sales * 1.0 / Overall_Sales AS DECIMAL(5,4)) <= 0.8;
```

## 6. Cohort Analysis (Customer Retention)

```SQL
WITH FirstPurchase AS (
    SELECT 
        CustomerID,
        MIN(OrderDate) AS First_Purchase_Date
    FROM Orders
    GROUP BY CustomerID
),
CustomerActivity AS (
    SELECT 
        o.CustomerID,
        f.First_Purchase_Date,
        DATEDIFF(MONTH, f.First_Purchase_Date, o.OrderDate) AS Month_Number
    FROM Orders o
    JOIN FirstPurchase f 
        ON o.CustomerID = f.CustomerID
),
CohortData AS (
    SELECT 
        FORMAT(First_Purchase_Date, 'yyyy-MM') AS Cohort_Month,
        Month_Number,
        COUNT(DISTINCT CustomerID) AS Customers
    FROM CustomerActivity
    GROUP BY 
        FORMAT(First_Purchase_Date, 'yyyy-MM'),
        Month_Number
),
CohortSize AS (
    SELECT 
        Cohort_Month,
        Customers AS Cohort_Size
    FROM CohortData
    WHERE Month_Number = 0
)
SELECT 
    c.Cohort_Month,
    c.Month_Number,
    c.Customers,
    CAST(c.Customers * 1.0 / cs.Cohort_Size AS DECIMAL(5,2)) AS Retention_Rate
FROM CohortData c
JOIN CohortSize cs 
    ON c.Cohort_Month = cs.Cohort_Month
ORDER BY 
    c.Cohort_Month,
    c.Month_Number;
```

## 7. Time-Series Forecasting

```SQL
WITH Monthly_Sales AS (
    SELECT 
        DATEFROMPARTS(YEAR(OrderDate), MONTH(OrderDate), 1) AS Month_Date,
        SUM(Sales) AS Total_Sales
    FROM Orders
    GROUP BY 
        DATEFROMPARTS(YEAR(OrderDate), MONTH(OrderDate), 1)
),
Final AS (
    SELECT 
        Month_Date,
        Total_Sales,
        LAG(Total_Sales) OVER(ORDER BY Month_Date) AS Prev_Month_Sales,
        AVG(Total_Sales) OVER(
            ORDER BY Month_Date 
            ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
        ) AS Rolling_3_Month_Avg
    FROM Monthly_Sales
)
SELECT 
    Month_Date,
    Total_Sales,
    ROUND(
        (Total_Sales - Prev_Month_Sales) * 100.0 / NULLIF(Prev_Month_Sales, 0),
        2
    ) AS MoM_Change_Percent,
    Rolling_3_Month_Avg
FROM Final
ORDER BY Month_Date;
```

### Key Insights

- #### Revenue Concentration (Pareto Effect):
  A disproportionate share of total revenue is driven by a small subset of products and customers, indicating strong concentration risk but also clear opportunities for targeted retention and upsell strategies.
- #### Customer Retention Dynamics:
  Cohort analysis shows a noticeable drop-off in repeat purchases after the initial transaction, highlighting a gap in post-acquisition engagement and the need for lifecycle marketing interventions.
- #### Cross-Sell Opportunities:
  Market basket analysis reveals consistent product pairings across orders, suggesting opportunities to increase average order value through bundling, recommendation systems, and targeted promotions.
- #### Segment Performance Variation:
  Customer segments exhibit differing revenue and profitability profiles, indicating that a one-size-fits-all strategy may be suboptimal. Segment-specific approaches could improve both conversion and margin.
- #### Sales Volatility & Trend Clarity:
  Month-over-month fluctuations in sales are significant, but rolling averages reveal underlying trends more clearly, supporting better forecasting and planning decisions.


### Conclusion

This analysis demonstrates how structured SQL-based exploration can uncover critical business patterns across customers, products, and time.
The findings highlight key levers for growth:

- Strengthening retention strategies to improve customer lifetime value
- Leveraging high-performing products and customers for revenue optimization
- Implementing data-driven cross-sell and bundling initiatives
- Adopting segment-specific strategies to maximize profitability.

Overall, the project illustrates the transition from raw transactional data to actionable business intelligence, enabling more informed and strategic decision-making.


### Strategic Business Recommendations
Based on the analysis, the following data-driven actions are recommended:

#### 1. Strengthen Customer Retention Strategy
- mplement post-purchase engagement campaigns (email, SMS, loyalty programs)
- Introduce incentives for second purchases (discounts, bundles)
- Focus on reducing early churn observed in cohort analysis

#### 2. Maximize High-Value Customers
- Bundle frequently purchased product combinations
- Use recommendations (“Customers also bought…”) to increase AOV
- Promote high-performing product pairs strategically

#### 3. Optimize Product Bundling & Cross-Selling
- Bundle frequently purchased product combinations
- Use recommendations (“Customers also bought…”) to increase AOV
- Promote high-performing product pairs strategically

#### 4. Focus on High-Impact Products (Pareto Strategy)
- Prioritize inventory and marketing for top-performing products
- Reduce focus or optimize pricing for low-performing items
- Use top products as entry points for cross-selling

#### 5. Segment-Specific Strategy
- Customize marketing and pricing strategies per segment:
   - Consumer → volume-driven offers
   - Corporate → bulk discounts
   - Home Office → targeted bundles

- Avoid one-size-fits-all campaigns

#### 6. Improve Forecasting & Planning
- Use rolling averages for better demand prediction
- Monitor month-over-month trends for early signals
- Align inventory and staffing with demand patterns

#### 7. Continuous Data Monitoring
- Build dashboards (Power BI) for real-time tracking
- Track KPIs:
  - Retention rate
  - Monthly revenue growth
  - Customer lifetime value



