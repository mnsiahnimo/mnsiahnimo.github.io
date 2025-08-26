
# Enrollment & Retention Analytics — SQL Project

## Context
This project analyzed 5+ years of student enrollment and graduation data to support **strategic planning**, **scholarship allocation**, and **student success initiatives** at the university level. 
The goal was to build **accurate reports and dashboards** for academic leadership using SQL and Tableau.

---

## Actions
1. **Data Preparation**  
   - Cleaned and standardized student enrollment records across multiple academic years.  
   - Integrated data from admissions, registrar, and financial aid systems.

2. **SQL Analysis**  
   - Used **aggregations**, **window functions**, and **conditional filtering** to derive insights on:  
     - First-year retention rates  
     - Graduation rates by cohort  
     - Impact of scholarships on student outcomes  

3. **Visualization Readiness**  
   - Structured results into **Tableau-friendly tables** for interactive dashboards.  
   - Created KPIs and trend lines for academic leadership and decision-makers.

---

## Results
- **Retention Trends:** Identified years with declining retention, prompting further analysis of financial and academic factors.  
- **Graduation Benchmarks:** Produced metrics on **4-year graduation rates** by cohort for institutional reports.  
- **Scholarship Impact:** Linked retention and graduation metrics with financial aid data to assess scholarship effectiveness.

---

## Impact
- Created **interactive Tableau dashboards** enabling administrators to drill into retention and graduation trends.  
- Informed **policy interventions** such as targeted scholarship programs for first-generation students.  
- Helped academic leadership align **budget decisions** with student success priorities.

---

## Growth / Next Steps
1. Add **predictive modeling** (e.g., logistic regression) to identify students at risk of dropping out.  
2. Automate SQL pipelines for **real-time Tableau dashboard updates**.  
3. Integrate additional student data sources (advising, financial aid) for deeper insights.

---

## SQL Code

### Task 1: Retention Rate by Cohort (Fall-to-Fall)
```sql
WITH cohort AS (
    SELECT  student_id,
            MIN(year_enrolled) AS entry_year
    FROM    enrollment
    GROUP BY student_id
),
retention AS (
    SELECT  c.entry_year,
            COUNT(DISTINCT CASE WHEN e.year_enrolled = c.entry_year + 1 THEN e.student_id END) AS retained,
            COUNT(DISTINCT c.student_id) AS total_students
    FROM    cohort c
    LEFT JOIN enrollment e
           ON c.student_id = e.student_id
    GROUP BY c.entry_year
)
SELECT  entry_year,
        ROUND(100.0 * retained / total_students, 1) AS retention_rate
FROM    retention
ORDER BY entry_year;
```

---

### Task 2: Graduation Rate by Cohort (4-Year Rate)
```sql
WITH cohort AS (
    SELECT  student_id,
            MIN(year_enrolled) AS entry_year
    FROM    enrollment
    GROUP BY student_id
),
graduation AS (
    SELECT  c.entry_year,
            COUNT(DISTINCT CASE WHEN g.grad_year <= c.entry_year + 4 THEN g.student_id END) AS grads_4yr,
            COUNT(DISTINCT c.student_id) AS total_students
    FROM    cohort c
    LEFT JOIN graduation g
           ON c.student_id = g.student_id
    GROUP BY c.entry_year
)
SELECT  entry_year,
        ROUND(100.0 * grads_4yr / total_students, 1) AS grad_rate_4yr
FROM    graduation
ORDER BY entry_year;
```
