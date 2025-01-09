# RFM Segmentation using SQL

## Overview

This project implements RFM (Recency, Frequency, Monetary) segmentation using SQL, leveraging a dataset containing sales transaction details. The goal is to classify customers into segments based on their purchasing behavior, aiding in targeted marketing strategies and customer relationship management.

## Dataset

The dataset used in this project contains the following key fields:

- **ORDERNUMBER**: Unique identifier for each order.
- **QUANTITYORDERED**: Quantity of items purchased.
- **PRICEEACH**: Price per unit of the product.
- **SALES**: Total sales value of the order.
- **ORDERDATE**: Date when the order was placed.
- **CUSTOMERNAME**: Name of the customer.
- **COUNTRY**: Customer's country.
- **DEALSIZE**: Size of the deal (e.g., Small, Medium, Large).

## Steps in the SQL Code

The SQL script performs the following steps to achieve RFM segmentation:

### 1. Data Preparation

Extract relevant columns and ensure proper date formatting.

```sql
-- Step 1: Data Preparation
SELECT 
    ORDERNUMBER, 
    QUANTITYORDERED, 
    PRICEEACH, 
    SALES, 
    ORDERDATE, 
    CUSTOMERNAME, 
    COUNTRY, 
    DEALSIZE
FROM SalesData;
```

### 2. Recency Calculation

Calculate the number of days since the last purchase for each customer.

```sql
-- Step 2: Recency Calculation
SELECT 
    CUSTOMERNAME, 
    MAX(DATEDIFF('2025-01-01', ORDERDATE)) AS Recency
FROM SalesData
GROUP BY CUSTOMERNAME;
```

### 3. Frequency Calculation

Count the number of transactions for each customer.

```sql
-- Step 3: Frequency Calculation
SELECT 
    CUSTOMERNAME, 
    COUNT(ORDERNUMBER) AS Frequency
FROM SalesData
GROUP BY CUSTOMERNAME;
```

### 4. Monetary Calculation

Compute the total sales amount for each customer.

```sql
-- Step 4: Monetary Calculation
SELECT 
    CUSTOMERNAME, 
    SUM(SALES) AS Monetary
FROM SalesData
GROUP BY CUSTOMERNAME;
```

### 5. Combining RFM Metrics

Combine Recency, Frequency, and Monetary metrics into a single table.

```sql
-- Step 5: Combine RFM Metrics
SELECT 
    CUSTOMERNAME, 
    MAX(DATEDIFF('2025-01-01', ORDERDATE)) AS Recency, 
    COUNT(ORDERNUMBER) AS Frequency, 
    SUM(SALES) AS Monetary
FROM SalesData
GROUP BY CUSTOMERNAME;
```

### 6. RFM Scoring

Assign scores to Recency, Frequency, and Monetary metrics.

```sql
-- Step 6: RFM Scoring
SELECT 
    CUSTOMERNAME, 
    NTILE(4) OVER (ORDER BY Recency ASC) AS RecencyScore,
    NTILE(4) OVER (ORDER BY Frequency DESC) AS FrequencyScore,
    NTILE(4) OVER (ORDER BY Monetary DESC) AS MonetaryScore
FROM (
    SELECT 
        CUSTOMERNAME, 
        MAX(DATEDIFF('2025-01-01', ORDERDATE)) AS Recency, 
        COUNT(ORDERNUMBER) AS Frequency, 
        SUM(SALES) AS Monetary
    FROM SalesData
    GROUP BY CUSTOMERNAME
) AS RFMData;
```

### 7. Customer Segmentation

Combine RFM scores to create customer segments.

```sql
-- Step 7: Customer Segmentation
SELECT 
    CUSTOMERNAME, 
    RecencyScore, 
    FrequencyScore, 
    MonetaryScore,
    CONCAT(RecencyScore, FrequencyScore, MonetaryScore) AS RFMScore,
    CASE 
        WHEN RecencyScore = 4 AND FrequencyScore = 4 AND MonetaryScore = 4 THEN 'Best Customers'
        WHEN RecencyScore >= 3 AND FrequencyScore >= 3 THEN 'Loyal Customers'
        WHEN RecencyScore = 1 THEN 'At-Risk Customers'
        ELSE 'Other'
    END AS Segment
FROM (
    SELECT 
        CUSTOMERNAME, 
        NTILE(4) OVER (ORDER BY Recency ASC) AS RecencyScore,
        NTILE(4) OVER (ORDER BY Frequency DESC) AS FrequencyScore,
        NTILE(4) OVER (ORDER BY Monetary DESC) AS MonetaryScore
    FROM (
        SELECT 
            CUSTOMERNAME, 
            MAX(DATEDIFF('2025-01-01', ORDERDATE)) AS Recency, 
            COUNT(ORDERNUMBER) AS Frequency, 
            SUM(SALES) AS Monetary
        FROM SalesData
        GROUP BY CUSTOMERNAME
    ) AS RFMData
) AS ScoredRFM;
```

## Output

The final output of the script includes:

- RFM scores for each customer.
- Segmented customer groups such as:
  - Best Customers
  - Loyal Customers
  - At-Risk Customers

## Applications

- Identifying high-value customers.
- Designing personalized marketing campaigns.
- Monitoring customer churn.

## How to Run

1. Load the provided dataset into your SQL database.
2. Execute the SQL script in your preferred SQL editor.
3. Analyze the results to gain insights into customer behavior.

##

