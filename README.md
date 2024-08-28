# Advanced SQL Use Cases for Car Auction Data Analysis

![SQL Logo](https://upload.wikimedia.org/wikipedia/commons/8/87/Sql_data_base_with_logo.png)

[![SQL](https://img.shields.io/badge/SQL-Advanced-blue)](https://www.mysql.com/) 
[![License](https://img.shields.io/badge/license-MIT-green)](LICENSE)
[![GitHub Stars](https://img.shields.io/github/stars/fahmizainal17/Advanced-SQL-Use-Cases-Car-Auction)](https://github.com/fahmizainal17/Advanced-SQL-Use-Cases-Car-Auction/stargazers)
[![GitHub Forks](https://img.shields.io/github/forks/fahmizainal17/Advanced-SQL-Use-Cases-Car-Auction)](https://github.com/fahmizainal17/Advanced-SQL-Use-Cases-Car-Auction/network/members)

## Project Overview

This project showcases advanced SQL queries and data analysis techniques applied to a dataset of car auctions. It features use cases that demonstrate key SQL skills, including data aggregation, filtering, ranking, comparative analysis, and cumulative calculations. This project is tailored for those looking to enhance their SQL proficiency through practical, data-driven scenarios.

## Dataset Description

The dataset includes auction data from two competitors over several months. It contains details about car brands, models, manufacture years, auction periods, starting prices, final bid prices, and listing counts. The dataset is structured to support various SQL analyses, including identifying top-performing models, comparing competitor pricing, and examining auction performance metrics.

### Data Columns

- **bidding_period:** The date when the bidding/auction took place, truncated to the 1st day of the month (e.g., 2022-11-01 for November 2022).
- **brand_name:** The brand of the car (e.g., Honda, BMW), with placeholder names used for this dataset.
- **model_name:** The model of the car (e.g., City, Civic, Myvi), with placeholder names used for this dataset.
- **make_year:** The year the car was manufactured.
- **reserved_price:** The starting price (Reserve Price) at which the auction begins for a given competitor.
- **winning_price:** The final price (Winning Price) at which the car auction concludes, also known as the Maximum Bidding Price.
- **listing_count:** The total number of cars listed for auction by the competitor.

## SQL Use Case Questions and Queries

### Question A: Return the total listings across all cars that are 3 years old or newer, sorted by month, for Competitor 1 in 2022.

**SQL Query:**
```sql
-- Get the total number of car listings for each month in 2022
SELECT 
    bidding_period AS "Bidding Month", -- The month the auction took place
    SUM(listing_count) AS "Listing Count" -- Total number of cars listed that month
FROM 
    purchase_data_competitor_1_2022 -- Competitor 1's data for 2022
WHERE 
    make_year >= 2019 -- Only include cars that are 3 years old or newer
GROUP BY 
    bidding_period -- Group by month
ORDER BY 
    bidding_period ASC; -- Sort by month in ascending order
```

**Example Output:**
| Bidding Month | Listing Count |
|---------------|---------------|
| 2022-10-01    | 150           |
| 2022-11-01    | 150           |
| ...           | ...           |

---

### Question B: Generate the top 2 models for each brand based on the highest listing count for Competitor 2 in November 2022.

**SQL Query:**
```sql
-- First, rank the models for each brand by their listing count
WITH RankedModels AS (
    SELECT 
        brand_name,      -- The brand of the car
        model_name,      -- The model of the car
        listing_count,   -- The number of times this model was listed for auction
        
        -- Rank the models within each brand based on the listing count, highest first
        ROW_NUMBER() OVER (PARTITION BY brand_name ORDER BY listing_count DESC) AS rank
    FROM 
        purchase_data_competitor_2_2022
    WHERE 
        -- We're only interested in data from November 2022
        bidding_period = '2022-11-01'
)

-- Now, select the top 2 models for each brand
SELECT 
    brand_name AS "Brand Name",      -- The brand of the car
    model_name AS "Model Name",      -- The model of the car
    listing_count AS "Listing Count" -- The number of listings
FROM 
    RankedModels
WHERE 
    -- Only include the top 2 ranked models per brand
    rank <= 2
ORDER BY 
    -- Sort by brand name first, then by listing count
    brand_name ASC, 
    listing_count DESC;
```

**Example Output:**
| Brand Name    | Model Name    | Listing Count |
|---------------|---------------|---------------|
| BRAND-NAME-A  | MODEL-NAME-A  | 100           |
| BRAND-NAME-A  | MODEL-NAME-C  | 80            |
| BRAND-NAME-B  | MODEL-NAME-G  | 150           |
| BRAND-NAME-B  | MODEL-NAME-A  | 50            |
| ...           | ...           | ...           |

---

### Question C: Compare the maximum Reserved Price (across all periods) by Make Year for the competitors.

**SQL Query:**
```sql
-- Find the maximum reserved price by Make Year for both competitors
SELECT 
    make_year AS "Make Year",
    MAX(CASE WHEN purchase_data_competitor_1_2022.reserved_price IS NOT NULL THEN reserved_price ELSE 0 END) AS "Max Reserved Price (Competitor 1)",
    MAX(CASE WHEN purchase_data_competitor_2_2022.reserved_price IS NOT NULL THEN reserved_price ELSE 0 END) AS "Max Reserved Price (Competitor 2)"
FROM 
    purchase_data_competitor_1_2022
FULL JOIN 
    purchase_data_competitor_2_2022 
USING (make_year)
GROUP BY 
    make_year
ORDER BY 
    make_year;
```

**Example Output:**
| Make Year | Max Reserved Price (Competitor 1) | Max Reserved Price (Competitor 2) |
|-----------|----------------------------------|----------------------------------|
| 2010      | 45000                            | 50000                            |
| 2011      | 32000                            | 30000                            |
| ...       | ...                              | ...                              |

---

### Question D: Show the median Winning Price by brand for Competitor 1 in January 2023.

**SQL Query:**
```sql
-- Calculate the median winning price for each brand in January 2023 for Competitor 1
SELECT 
    brand_name AS "Brand Name",
    PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY winning_price) AS "Median Winning Price"
FROM 
    purchase_data_competitor_1_2023
WHERE 
    bidding_period = '2023-01-01'
GROUP BY 
    brand_name
ORDER BY 
    brand_name;
```

**Example Output:**
| Brand Name   | Median Winning Price |
|--------------|----------------------|
| BRAND-NAME-A | 50000                |
| BRAND-NAME-B | 30000                |
| ...          | ...                  |

---

### Question E: Compute the cumulative listings for Competitor 2 across the months in 2022.

**SQL Query:**
```sql
-- Calculate cumulative listings across months for Competitor 2 in 2022
SELECT 
    bidding_period AS "Month",
    SUM(listing_count) OVER (ORDER BY bidding_period ASC) AS "Cumulative Listing Count"
FROM 
    purchase_data_competitor_2_2022
ORDER BY 
    bidding_period;
```

**Example Output:**
| Month       | Cumulative Listing Count |
|-------------|--------------------------|
| 2022-10-01  | 1000                     |
| 2022-11-01  | 1200                     |
| 2022-12-01  | 1700                     |

---
