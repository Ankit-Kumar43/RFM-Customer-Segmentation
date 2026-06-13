# Retail & Finance RFM Analysis Project 📊

This project focuses on customer segmentation using the **RFM (Recency, Frequency, Monetary)** model. By analyzing customer purchasing behavior, this project identifies high-value customers, loyal segments, and customers at risk of churning to drive data-driven marketing strategies.

---

## 🛠️ Tech Stack & Tools Used
* **SQL / PostgreSQL:** For data extraction, cleaning, and RFM score calculation.
* **Concepts:** CTEs (Common Table Expressions), Window Functions, Aggregations, Case Statements.

---

## 🧠 Database View Creation (SQL Code)

Here is the core SQL logic used to create the RFM metrics view from the transactional data:

```sql
CREATE VIEW rfm_score_view AS
WITH rfm_base AS (
    -- Step 1: Calculate Recency, Frequency, and Monetary values
    SELECT 
        customer_id,
        MAX(order_date) AS last_order_date,
        (SELECT MAX(order_date) FROM sales_data) - MAX(order_date) AS recency,
        COUNT(order_id) AS frequency,
        SUM(total_amount) AS monetary
    FROM sales_data
    GROUP BY customer_id
),
rfm_scores AS (
    -- Step 2: Assign NTILE scores (1-5) for R, F, and M
    SELECT 
        customer_id,
        recency,
        frequency,
        monetary,
        NTILE(5) OVER (ORDER BY recency ASC) AS r_score,     -- Lower recency is better
        NTILE(5) OVER (ORDER BY frequency DESC) AS f_score,  -- Higher frequency is better
        NTILE(5) OVER (ORDER BY monetary DESC) AS m_score   -- Higher monetary is better
    FROM rfm_base
)
-- Step 3: Combine scores into a final RFM segment
SELECT 
    customer_id,
    recency,
    frequency,
    monetary,
    r_score,
    f_score,
    m_score,
    CONCAT(r_score, f_score, m_score) AS rfm_cell_score
FROM rfm_scores;
