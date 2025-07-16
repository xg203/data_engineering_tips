# ROLLUP vs CUBE SQL Comparison

## Overview

While ROLLUP and CUBE might seem similar at first glance, there's actually a **crucial difference** in the number and type of groupings they produce.

## ROLLUP - Hierarchical (Fewer Groupings)

### Query
```sql
SELECT department, job_title, SUM(salary) 
FROM employees 
GROUP BY ROLLUP(department, job_title);
```

### Produces only 3 grouping sets:
1. `(department, job_title)` - Detail level
2. `(department)` - Department subtotals  
3. `()` - Grand total

### Example Output:
```
Department    Job_Title     Total_Salary
IT           Developer      150000    ← Detail
IT           Manager        80000     ← Detail  
IT           NULL           230000    ← Dept subtotal
HR           Manager        70000     ← Detail
HR           NULL           70000     ← Dept subtotal
NULL         NULL           300000    ← Grand total
```
**6 rows total**

## CUBE - All Combinations (More Groupings)

### Query
```sql
SELECT department, job_title, SUM(salary) 
FROM employees 
GROUP BY CUBE(department, job_title);
```

### Produces 4 grouping sets:
1. `(department, job_title)` - Detail level
2. `(department)` - Department subtotals
3. `(job_title)` - Job title subtotals ← **This is what ROLLUP doesn't have**
4. `()` - Grand total

### Example Output:
```
Department    Job_Title     Total_Salary
IT           Developer      150000    ← Detail
IT           Manager        80000     ← Detail
IT           NULL           230000    ← Dept subtotal
HR           Manager        70000     ← Detail  
HR           NULL           70000     ← Dept subtotal
NULL         Developer      150000    ← Job subtotal (ROLLUP doesn't have this)
NULL         Manager        150000    ← Job subtotal (ROLLUP doesn't have this)
NULL         NULL           300000    ← Grand total
```
**8 rows total**

## Key Differences Summary

| Aspect | ROLLUP | CUBE |
|--------|---------|------|
| **Grouping Pattern** | Hierarchical (follows column order) | All possible combinations |
| **Number of Grouping Sets** | n+1 (where n = number of columns) | 2^n (where n = number of columns) |
| **Example with 2 columns** | 3 grouping sets | 4 grouping sets |
| **Example with 3 columns** | 4 grouping sets | 8 grouping sets |
| **Performance** | Faster (fewer computations) | Slower (more computations) |
| **Use Case** | Hierarchical reporting | Complete multidimensional analysis |

## The Key Difference

- **ROLLUP**: Only creates subtotals following the column order hierarchy
- **CUBE**: Creates subtotals for **every possible combination** of columns

With 3 columns, ROLLUP gives you 4 grouping sets, but CUBE gives you 8 grouping sets (2³). The difference becomes much more dramatic with more columns!

## Regular GROUP BY vs ROLLUP vs CUBE

### Regular GROUP BY
```sql
SELECT department, job_title, SUM(salary) as total_salary 
FROM employees 
GROUP BY department, job_title;
```
**Produces only detail level rows (no subtotals)**

### ROLLUP
```sql
SELECT department, job_title, SUM(salary) as total_salary 
FROM employees 
GROUP BY ROLLUP(department, job_title);
```
**Produces detail + hierarchical subtotals + grand total**

### CUBE
```sql
SELECT department, job_title, SUM(salary) as total_salary 
FROM employees 
GROUP BY CUBE(department, job_title);
```
**Produces detail + all possible subtotal combinations + grand total**

## When to Use Each

- **Regular GROUP BY**: When you need only detailed breakdowns
- **ROLLUP**: When you need hierarchical reporting (e.g., Year → Quarter → Month)
- **CUBE**: When you need comprehensive cross-tabulation analysis for data warehousing/BI
