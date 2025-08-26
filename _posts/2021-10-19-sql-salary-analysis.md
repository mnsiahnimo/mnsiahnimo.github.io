## Salary Analysis Project

### Context
Major League Baseball teams often have drastically different financial capabilities.  
This project explored salary data to identify top-spending teams, cumulative spending patterns,  
and key financial milestones using SQL analytics techniques like **window functions**,  
**rolling calculations**, and **min/max value filtering**.

The dataset used was the **Lahman Baseball Database**, focusing on the **salaries** table with team-wise annual salary information.



### Actions
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



### Results
- **Top 20% Teams:** Clear identification of teams with consistently high financial investment.  
- **Cumulative Spending Trends:** Year-over-year growth visualized for each team,  
  highlighting long-term financial commitment.  
- **$1B Milestones:** Mapped the earliest year each team exceeded $1B in cumulative salaries,  
  showing the speed of financial growth across franchises.



### Impact
- Provided **actionable insights** for sports economists, team analysts, and management by:
  - Benchmarking team investments over time.
  - Enabling comparisons of **financial power vs. performance outcomes** (could be merged with win-loss data later).

- Showcased **advanced SQL skills**, including:
  - **Window functions** (`NTILE`, `ROW_NUMBER`)
  - **Rolling aggregates** (`SUM OVER`)
  - **Complex filtering & CTEs**



### Growth / Next Steps
1. **Integrate Performance Metrics:**  
   Compare spending trends with team performance (wins, championships).   

2. **Predictive Modeling:**  
   Use regression models to link **salary investment → performance outcomes**.  

3. **Automation & Deployment:**  
   Schedule SQL scripts and integrate with **CI/CD pipelines** for automated data refresh and visualization updates.
