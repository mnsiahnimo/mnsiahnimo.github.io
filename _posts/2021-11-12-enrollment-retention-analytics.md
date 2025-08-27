---
layout: post
title: Enrollment & Retention Analytics Project
image: "/posts/enroll-retain-analysis-main.png"
tags: [SQL,Databases,Data Filtering,Exploratory Data Analysis,Aggregations,Window Functions]
---

# Enrollment & Retention Analytics — SQL Code and Explanations

> **Goal:** Compute key student success metrics by cohort using SQL:
> 1) Fall-to-Fall retention; 2) 4-year graduation rate;
> 3) Multi-year persistence (Year-1 and Year-2); 4) Time-to-degree (median years).

---

## Connect to Database
*(Pseudo-step — depends on your RDBMS/driver; examples: `psql`, `mysql`, `sqlcmd`, `bq`, etc.)*

---

## Task 1 — Retention Rate by Cohort (Fall-to-Fall)

**What it does:**  
- Defines each student’s **entry cohort** as their **first year** in `enrollment`.  
- Counts cohort members who **show up exactly one year later** (Year N → Year N+1).  
- Outputs **retention %** per entry year.

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

## Task 2 — Graduation Rate by Cohort (4-Year Rate)

**What it does:**  
- For each cohort (entry year), counts students who **graduate within 4 years**.  
- Returns **4-year grad rate %** by cohort.

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

---

## Task 3 — Multi-Year Persistence (Year-1 and Year-2) by Cohort

**What it does:**  
- Extends Task 1 by calculating **persistence** to **Year N+1** *and* **Year N+2** for each cohort.  
- Useful to see where the biggest drop-offs happen.

```sql
WITH cohort AS (
    SELECT  student_id,
            MIN(year_enrolled) AS entry_year
    FROM    enrollment
    GROUP BY student_id
),
persistence AS (
    SELECT  c.entry_year,
            COUNT(*) AS total_students,
            -- Persisted to Year N+1
            SUM(CASE WHEN EXISTS (
                    SELECT 1
                    FROM enrollment e1
                    WHERE e1.student_id   = c.student_id
                      AND e1.year_enrolled = c.entry_year + 1
                ) THEN 1 ELSE 0 END) AS persisted_y1,
            -- Persisted to Year N+2
            SUM(CASE WHEN EXISTS (
                    SELECT 1
                    FROM enrollment e2
                    WHERE e2.student_id   = c.student_id
                      AND e2.year_enrolled = c.entry_year + 2
                ) THEN 1 ELSE 0 END) AS persisted_y2
    FROM cohort c
    GROUP BY c.entry_year
)
SELECT
    entry_year,
    ROUND(100.0 * persisted_y1 / total_students, 1) AS persist_rate_y1,
    ROUND(100.0 * persisted_y2 / total_students, 1) AS persist_rate_y2
FROM persistence
ORDER BY entry_year;
```

---

## Task 4 — Time-to-Degree (Median Years to Graduate) by Cohort

**What it does:**  
- For each cohort, computes **years to graduation** for those who graduated.  
- Returns the **median time-to-degree** (and an optional distribution).

> SQL to calculate medians 

### PostgreSQL (using `percentile_cont`)
```sql
WITH cohort AS (
    SELECT  student_id, MIN(year_enrolled) AS entry_year
    FROM    enrollment
    GROUP BY student_id
),
ttd AS (
    SELECT  c.entry_year,
            (g.grad_year - c.entry_year) AS years_to_degree
    FROM    cohort c
    JOIN    graduation g
      ON    g.student_id = c.student_id
)
SELECT
    entry_year,
    percentile_cont(0.5) WITHIN GROUP (ORDER BY years_to_degree) AS median_years_to_degree,
    COUNT(*) AS grads_count
FROM ttd
GROUP BY entry_year
ORDER BY entry_year;
```
---


