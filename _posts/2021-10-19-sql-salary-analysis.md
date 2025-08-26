
# Salary Analysis Project

## Context
Major League Baseball teams often have drastically different financial capabilities.  
This project explored salary data to identify top-spending teams, cumulative spending patterns,  
and key financial milestones using SQL analytics techniques like **window functions**,  
**rolling calculations**, and **min/max value filtering**.

The dataset used was the **Lahman Baseball Database**, focusing on the **salaries** table with team-wise annual salary information.

---

## Actions
1. **Data Exploration**  
   - Queried the salaries table to understand its structure and contents (`teamID`, `yearID`, `salary`).

2. **Top-Spending Teams (Window Functions)**  
   - Used **NTILE(5)** to rank teams by average annual spending.  
   - Filtered for the **top 20% spenders** across all seasons.

3. **Cumulative Spending Over Time (Rolling Calculations)**  
   - Calculated **cumulative spending** by team across years using  
     `SUM() OVER (PARTITION BY team ORDER BY year)`.

4. **Spending Milestone Detection (Min/Max Filtering)**  
   - Identified the **first year each team crossed $1B cumulative spending** using  
     `ROW_NUMBER()` and conditional filtering.

---

## Results
- **Top 20% Teams:** Clear identification of teams with consistently high financial investment.  
- **Cumulative Spending Trends:** Year-over-year growth visualized for each team,  
  highlighting long-term financial commitment.  
- **$1B Milestones:** Mapped the earliest year each team exceeded $1B in cumulative salaries,  
  showing the speed of financial growth across franchises.

---

## Impact
- Provided **actionable insights** for sports economists, team analysts, and management by:
  - Benchmarking team investments over time.
  - Enabling comparisons of **financial power vs. performance outcomes** (could be merged with win-loss data later).

- Showcased **advanced SQL skills**, including:
  - **Window functions** (`NTILE`, `ROW_NUMBER`)
  - **Rolling aggregates** (`SUM OVER`)
  - **Complex filtering & CTEs`**

---

## Growth / Next Steps
1. **Integrate Performance Metrics:**  
   Compare spending trends with team performance (wins, championships).  

2. **Predictive Modeling:**  
   Use regression models to link **salary investment → performance outcomes**.  

3. **Automation & Deployment:**  
   Schedule SQL scripts and integrate with **CI/CD pipelines** for automated data refresh and visualization updates.

---

## SQL Code

### Task 1: View the salaries table
```sql
SELECT * FROM salaries;
```

---

### Task 2: Top 20% of teams by average annual spending
```sql
WITH ts AS (
    SELECT  teamID, yearID, SUM(salary) AS total_spend
    FROM    salaries
    GROUP BY teamID, yearID
),
sp AS (
    SELECT  teamID, AVG(total_spend) AS avg_spend,
            NTILE(5) OVER (ORDER BY AVG(total_spend) DESC) AS spend_pct
    FROM    ts
    GROUP BY teamID
)
SELECT  teamID, ROUND(avg_spend / 1000000, 1) AS avg_spend_millions
FROM    sp
WHERE   spend_pct = 1;
```

---

### Task 3: Cumulative sum of spending over the years
```sql
WITH ts AS (
    SELECT  teamID, yearID, SUM(salary) AS total_spend
    FROM    salaries
    GROUP BY teamID, yearID
)
SELECT  teamID, yearID,
        ROUND(SUM(total_spend) OVER (PARTITION BY teamID ORDER BY yearID) / 1000000, 1)
            AS cumulative_sum_millions
FROM    ts;
```

---

### Task 4: First year each team's cumulative spending surpassed $1B
```sql
WITH ts AS (
    SELECT  teamID, yearID, SUM(salary) AS total_spend
    FROM    salaries
    GROUP BY teamID, yearID
),
cs AS (
    SELECT  teamID, yearID,
            SUM(total_spend) OVER (PARTITION BY teamID ORDER BY yearID) AS cumulative_sum
    FROM    ts
),
rn AS (
    SELECT  teamID, yearID, cumulative_sum,
            ROW_NUMBER() OVER (PARTITION BY teamID ORDER BY cumulative_sum) AS rn
    FROM    cs
    WHERE   cumulative_sum > 1000000000
)
SELECT  teamID, yearID, ROUND(cumulative_sum / 1000000000, 2) AS cumulative_sum_billions
FROM    rn
WHERE   rn = 1;
```
