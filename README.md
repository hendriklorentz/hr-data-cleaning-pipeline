# 📊 Enterprise HR Workforce Data Purification & ETL Pipeline

## 🎯 Project Overview
This project establishes a production-grade data cleaning (ETL) pipeline using **Python (Pandas)** and structural **SQL schemas** to sanitize a raw enterprise HRIS dataset containing 3,000 employee records. 

By applying automated programmatic data hygiene rules, this pipeline identifies and self-corrects deep structural integrity anomalies, safeguarding downstream corporate analytics, payroll systems, and mandatory L&D compliance tracking.

---

## 🚨 1. The Problem (Business & Data Challenges)
During a system data merge from legacy architectures, the master organization database suffered severe administrative data corruption. A systematic data logic audit revealed the following critical bugs:
* **Headcount Skew:** Identified **991 historical employee separation logs** (dating back to 2022-2023) where the system recorded valid exit dates but failed to update the `EmployeeStatus` from **Active** to **Terminated**.
* **Downstream Risks:** Leaving 991 terminated employees listed as active causes major financial, security, and operational bottlenecks—artificially inflating workforce budgets, compromising secure internal system access, and causing thousands of dollars in wasted course allocations for mandatory training modules.
* **Data Formatting Inconsistencies:** Key chronological dimensions (`StartDate`, `ExitDate`) were stored as un-parsable text `object` string strings rather than calendar-aware formats, blocking crucial business intelligence calculations like employee lifetime tenure.

---

## 🛠️ 2. The Analysis (The Python Cleaning Pipeline)
An automated Python pipeline was engineered to cleanly isolate, restructure, and transform the raw data without manual row-by-row interference.

### Key Engineering Steps Implemented:
1. **Entity Verification:** Scanned the primary tracking key (`EmpID`) to confirm absolute database uniqueness and eliminate artificial duplicate data.
2. **Datetime Standarization:** Parsed raw date text strings into standardized `datetime64[ns]` formats, resolving formatting mismatches.
3. **Programmatic Business Logic Self-Correction:** Deployed a conditional Pandas logical mask (`.loc`) to isolate the 991 corrupted entries and automatically adjust their employment status to accurately reflect their termination.
4. **Data Completeness & Null Handling:** Sanitized categorical missing fields uniformly (e.g., changing missing text descriptions to `"N/A - Active Employee"`), while strategically preserving legitimate `NULL` values in the `ExitDate` column for the 1,467 truly active staff members.

```python
# Core Business Logic Fix Applied:
glitch_filter = (df['EmployeeStatus'] == 'Active') & (df['ExitDate'].notnull())
df.loc[glitch_filter, 'EmployeeStatus'] = 'Terminated'
```
## 🗄️ 3. Relational Database Architecture (SQL Blueprint)
To transition the clean flat file data into permanent data warehouse storage, a relational database schema was drafted with explicit structural integrity constraints:
CREATE TABLE production_workforce_dim (
    emp_id INT PRIMARY KEY,
    first_name VARCHAR(50) NOT NULL,
    last_name VARCHAR(50) NOT NULL,
    start_date DATE NOT NULL,
    exit_date DATE NULL,  -- Intentionally set to NULL to cleanly map our 1,467 active staff
    employee_status VARCHAR(50) NOT NULL,
    current_employee_rating INT NULL
);

## 📈 4. Business Outcome
100% Data Integrity Restored: Successfully isolated 1,467 truly active corporate employees from historical records, correcting a headcount error margin of over 30%.

Audit-Ready Infrastructure: Mitigated automated system waste by giving human resource, financial audit, and L&D operations managers a single, trusted baseline of active personnel data.
