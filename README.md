# Superstore SQL Analytics: Customer Insights, Cohort Analysis & Revenue Optimization

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
