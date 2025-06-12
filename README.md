Lawrence Public Schools – Device Inventory ETL Process
Table of Contents
Overview

Process Diagram:
![image](https://github.com/user-attachments/assets/a653b62b-94f7-4381-b7e7-e44ba3822a43)

![image](https://github.com/user-attachments/assets/4806bd0c-8703-4124-a801-59d2917e4017)

Step-by-Step Instructions

A. CSV Preparation

B. Create SSIS Packages (Visual Studio)

C. Deploy Packages to SQL Server (SSISDB)

D. Create and Schedule SQL Server Agent Jobs

E. (Optional) Create Views for Clean Data

Troubleshooting & Gotchas

Best Practices

Appendix: SQL Snippets

Overview
This documentation explains the end-to-end process for automating device inventory data loading from CSV files to SQL Server for analytics, reporting, and Power BI dashboards.

Source: CSV files routinely refreshed in a shared network folder (D:\Data\IT\).

Goal: Keep SQL Server tables up to date, clean, and ready for BI/reporting tools.

Tech Stack: SSIS (SQL Server Integration Services), SQL Server Agent, Power BI, T-SQL.

Process Diagram
pgsql
Copy
Edit
CSV Files in IT Folder
       ↓
SSIS Package (Flat File Source → OLE DB Destination)
       ↓
SQL Server Table (Raw Data)
       ↓
(SQL View strips out quote characters, etc.)
       ↓
Power BI / Reporting uses the cleaned View
       ↓
Automated Nightly Refresh via SQL Server Agent Job
Step-by-Step Instructions
A. CSV Preparation
Location:
Ensure all source CSV files (e.g., SCCM_Report.csv, JamFPro_Reports.csv, ChromeOSDevices_Reports.csv) are being regularly updated to a shared folder (e.g., D:\Data\IT\).

Format:
CSVs should have headers in the first row.
All fields should be delimited by commas (,).
Quotation marks around values are OK (we’ll strip them later).

B. Create SSIS Packages (Visual Studio)
1. New SSIS Project
Open SQL Server Data Tools (Visual Studio).

Create a new Integration Services Project.

2. Add Data Flow Task
Drag a Data Flow Task into the Control Flow.

3. Configure Data Flow
In Data Flow:

Drag in a Flat File Source.

Create a Flat File Connection Manager:

Point it to your CSV file (e.g., D:\Data\IT\SCCM_Report.csv).

Set delimiter to comma, make sure to check/uncheck “Column Names in First Data Row” as needed.

Set column widths (can use Suggest Types or set all to DT_STR, 255 for ease).

Drag in an OLE DB Destination:

Connect to your SQL Server database.

New table or use an existing table (name e.g., SCCM_InventoryFinal).

Map columns between source and destination.

4. Save and Test Locally
Right-click the package (e.g., SCCM_Import.dtsx) and Execute.

Debug any data truncation or conversion errors (increase column widths, ignore problematic columns, etc.).

C. Deploy Packages to SQL Server (SSISDB)
1. Build the Project
Right-click the project and select Build.

This generates a .ispac file in your /bin/Development folder.

2. Deploy with Deployment Wizard
Double-click the .ispac file or use the “Integration Services Deployment Wizard.”

Choose Project Deployment Model.

Deploy to the correct server and SSISDB folder (IT/Projects/YourProjectName).

3. Verify Deployment
Open SQL Server Management Studio (SSMS).

Under Integration Services Catalogs → SSISDB → IT → Projects, confirm your packages are present.

D. Create and Schedule SQL Server Agent Jobs
1. Create a New Job
In SSMS, go to SQL Server Agent → Jobs → New Job.

Give it a descriptive name (e.g., Nightly_Device_Inventory_Refresh).

2. Add Steps
Step 1: Run the SSIS package

Type: SQL Server Integration Services Package

Package source: SSIS Catalog

Select your deployed package.

Step 2 (Optional): Post-processing

Type: Transact-SQL

Script to strip quotes from columns (if not using views).

Repeat for other CSVs/packages if desired (can be separate jobs or all in one).

3. Schedule the Job
Go to Schedules tab, add a new schedule (e.g., every night at midnight).

4. Test & Monitor
Right-click the job and select Start Job at Step…

If it fails, review the Job History and All Executions in SSISDB for detailed errors.

E. (Optional) Create Views for Clean Data
Why Views?
Views allow you to “clean” data at query time—no need to mess with ETL/SSIS packages.

Use SQL to strip quotes and other formatting issues.

How to Create a View
sql
Copy
Edit
CREATE VIEW dbo.SCCM_Inventory_Clean AS
SELECT
    REPLACE([MANUFACTURER], '"', '') AS [MANUFACTURER],
    REPLACE([MODEL], '"', '') AS [MODEL],
    REPLACE([OPERATING SYSTEM], '"', '') AS [OPERATING SYSTEM],
    -- (repeat for all columns as needed)
    [IPv4],
    [IPv6],
    ...
FROM dbo.SCCM_InventoryFinal;
Repeat for other tables: JamFPro_ReportsFinal, ChromeOSDevices_ReportsFinal, etc.

How to Use in Power BI
In Power BI, connect to the SQL View (e.g., dbo.SCCM_Inventory_Clean), not the raw table.

All updates to the table are instantly reflected in the view.

Troubleshooting & Gotchas
Data Truncation in SSIS:

Increase column width in Flat File Connection Manager.

Quotes in Data:

Best to ignore in SSIS and clean in SQL Views.

VS_NEEDS_NEWMETADATA / Validation Errors:

Re-map destination columns if the table structure changes.

Job Fails with Table Not Found:

Check destination table name in SSIS and in SQL Server. They must match.

SSIS Package Not in SSISDB:

Make sure you deploy your .ispac via the wizard.

Best Practices
Keep ETL Simple:

Load raw data as-is. Clean later via SQL views.

Use Views for Cleaning:

Never overwrite raw data. Views are flexible, easy to modify, and safe.

Automate with Jobs:

Schedule jobs for after-hours or non-peak times.

Test Everything End-to-End:

Manually run jobs after changes. Check Power BI or analytics tools.

Version Control:

Keep your SSIS packages and SQL scripts in GitHub or another version control system.

Appendix: SQL Snippets
A. Remove Quotes from All Columns
sql
Copy
Edit
UPDATE dbo.SCCM_InventoryFinal
SET
    [MANUFACTURER] = REPLACE([MANUFACTURER], '"', ''),
    [MODEL] = REPLACE([MODEL], '"', ''),
    -- repeat for all columns
    [IPv4] = REPLACE([IPv4], '"', ''),
    [IPv6] = REPLACE([IPv6], '"', '')
-- Add more columns as needed
B. Example View for JamFPro
sql
Copy
Edit
CREATE VIEW dbo.JamFPro_Reports_Clean AS
SELECT
    REPLACE([serialNumber], '"', '') AS [serialNumber],
    REPLACE([name], '"', '') AS [name],
    ...
FROM dbo.JamFPro_ReportsFinal
Contact & Ownership
Primary: Mahammed Khalifa

For questions: mahammed.khalifa@lawrence.k12.ma.us

Docs last updated: June 12, 2025
