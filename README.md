# 🏢 PeopleOps Data Hackathon: Enterprise HR & Attendance Pipeline

Welcome to the **Indux Corp** PeopleOps Data Warehouse and ETL Pipeline repository! This project is an end-to-end data engineering solution built to ingest messy HR, biometric, and payroll data, clean it, model it into a Star Schema data warehouse, run automated data quality checks, and answer critical business intelligence questions.

---

## 📖 The CFO Story & Key Business Findings

During our analysis of the 6-month operational data, we uncovered three critical insights that directly impact company costs and workforce productivity:

1. **Finding 1: Attendance Inefficiencies & Late Arrivals**
   * *Data Evidence:* Analysis of biometric punches revealed clusters of recurring late arrivals concentrated in specific departments during morning shifts.
   * *Business Action:* Introduce flexible shift timings or automated morning alerts to improve punctuality and reduce lost productivity hours.

2. **Finding 2: Overtime vs. Computed Discrepancies**
   * *Data Evidence:* Comparing actual worked hours against standard 8-hour shifts highlighted departments where manual overtime approvals exceeded automated computation thresholds.
   * *Business Action:* Tighten managerial approvals for overtime claims to prevent unnecessary payroll inflation.

3. **Finding 3: Headcount & Attrition Trends**
   * *Data Evidence:* Tracking month-end headcounts and separation dates showed specific retention drops mid-cycle.
   * *Business Action:* HR should launch targeted employee engagement and retention programs during high-attrition months.

---

## 🏗️ Architecture & Pipeline Workflow

The pipeline follows a modern medallion-style architecture, moving data safely from raw files to clean staging, a dimensional warehouse, and final reporting.

```mermaid
graph TD
    A[Raw Files: CSV & JSON] -->|Stage 1: Ingestion| B[SQL Server: Raw Tables]
    B -->|Stage 2: Cleaning & Staging| C[Staging Schema & Audit Logs]
    C -->|Stage 3: Attendance Logic| D[staging.attendance_daily]
    D -->|Stage 4: Warehouse Modeling| E[Star Schema: Dimensions & Facts]
    E -->|Stage 5: Business Queries| F[CSV Reports in Stage5_Outputs]
    E -->|Stage 6: Validation| G[Automated DQ Results Table]





    🚀 Step-by-Step Pipeline Breakdown (Beginner's Guide)
If you are new to data engineering, here is how our 6-stage pipeline transforms messy data into polished business insights:

Stage 1: Data Ingestion (ingest.py)
What happened: We had raw files (employees, biometric punches, shift rosters, and leave requests) in different formats (CSV and JSON).

How we did it: Using Python (pyodbc and pandas), we loaded all raw files securely into SQL Server raw_* tables. We added a unique row hash (_row_hash) to ensure the load is idempotent (running it twice won't create duplicates).

Stage 2: Cleaning & Staging (transform.py)
What happened: Raw data is always messy (extra spaces, weird text casings, duplicate employees, or unknown device punch IDs).

How we did it: We cleaned text values, standardized date formats, separated valid records into the staging schema, and pushed unmapped/bad punches into a rejected_punches audit table.

Stage 3: Business Logic & Attendance Engine (attendance_logic.py)
What happened: We needed to calculate every employee's daily attendance status (Present, Absent, Missing Punch, Weekly Off, etc.) by combining biometric check-ins with shift schedules.

How we did it: Built an automated SQL/Python logic that evaluated worked hours and generated over 182,000 accurate daily attendance records (staging.attendance_daily).

Stage 4: Warehouse Modeling & Star Schema (warehouse_model.py)
What happened: To make reporting fast and organized, we designed a Star Schema Data Warehouse (warehouse schema).

How we did it: Created Dimension tables (dim_employee supporting SCD Type 2 history tracking, dim_department, dim_shift, dim_date) and Fact tables (fact_attendance_daily) with strict Foreign Key constraints.

Stage 5: Business Intelligence & CSV Exports (run_stage5.py)
What happened: Management needed answers to 10 core business questions (headcount, attrition rate, absenteeism, overtime comparisons, etc.).

How we did it: Wrote advanced SQL queries utilizing CTEs and window functions, executing them automatically and saving the results into clean CSV files inside the Stage5_Outputs/ folder.

Stage 6: Automated Data Quality Checks (dq_checks.py)
What happened: How do we prove our data is trustworthy? By running automated tests.

How we did it: We created a script that checks Primary Key uniqueness, Referential Integrity, Accepted Values, and Business Sanity rules (e.g., no negative worked hours, no shift longer than 16 hours, no overlapping SCD2 dates). All results are logged into a dq_results table with a 100% Pass rate.

🛠️ Tech Stack
Language: Python 3.14 (pyodbc, pandas, csv, glob)

Database: Microsoft SQL Server (indux_db)

Architecture: Star Schema, Staging & Raw layers, SCD Type 2.

⚙️ How to Run the Project
To rebuild the entire pipeline from scratch with a single command:

Bash
python dq_checks.py