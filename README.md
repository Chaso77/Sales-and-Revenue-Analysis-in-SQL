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

- A small percentage of products/customers drive the majority of revenue (Pareto Principle)
- Customer retention drops significantly after initial purchase
- Certain product combinations reveal strong cross-sell opportunities
- Rolling averages help smooth sales volatility and reveal trends

### Conclusion

This project demonstrates how SQL can be used not just for querying data, but for:

- Advanced analytics
- Business intelligence
- Decision-making support


