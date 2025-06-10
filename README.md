Lawrence Public Schools Data Pipeline: Automated Data Refresh for Power BI
Overview

This documentation explains the end-to-end data pipeline for Lawrence Public Schools’ analytics and dashboarding environment. The goal is to automate the refresh of critical device and inventory data from source systems all the way into live Power BI dashboards, with no manual intervention required.

Current Data Flow
Step-by-Step Process
Data Exported from Source Systems

Device and inventory data is exported daily as CSV or Excel files from systems such as:

SCCM (Windows management)

Jamf (Apple device management)

ChromeOS (Chromebooks)

Others as needed

Files Copied to Central Shared Folder

Scripts automatically copy the exported files from their original locations (e.g., G:\Shared drives\IT\Inv\Auto Exports) to a centralized folder on the main data warehouse server:

Copy
\\lps-dw1.lawrence.k12.ma.us\Data\IT
This ensures all updated files are available in one location for processing.

Example: The nightly SCCM export lands as sccm.csv in this folder.

Import Jobs (SSIS Packages) Monitor Shared Folder

SQL Server Integration Services (SSIS) packages (called “Import jobs” or “ITImports”) are configured on the lps-dw1 server.

Each package watches the d:\Data\IT folder for new data files (e.g., sccm.csv, jamf.csv, chromeos.csv).

Data Loaded into SQL Server

When a new file is detected (typically after 12:20 AM each day), the corresponding SSIS package imports the data into a SQL Server table.

The SQL database now holds the latest data from all sources.

Power BI Dashboards Connect to SQL Tables/Views

Power BI dashboards are set up to connect directly to these SQL Server tables or views.

When users refresh a dashboard, it pulls the most current data available.

Diagram Flowchart
    A[Export CSV/XLSX from SCCM, Jamf, ChromeOS, etc.] --> B[Files copied to \\lps-dw1\Data\IT]
    B --> C[SSIS/ITImports jobs monitor d:\Data\IT for new files]
    C --> D[SSIS job loads data into SQL Server tables]
    D --> E[Power BI dashboards connect to SQL tables/views]

![image](https://github.com/user-attachments/assets/e043257a-bca9-4dcf-b1cc-fe2176cabb1d)
    
Key Components
Component	Description
Source Systems	SCCM, Jamf, ChromeOS, etc.
Shared Folder	d:\Data\IT on lps-dw1.lawrence.k12.ma.us
SSIS Packages	Automation jobs (e.g., SCCMImport.dtsx) for each data file
SQL Server	Central database storing cleaned/updated data
Power BI	Dashboards consuming SQL data for analysis/visualization

Benefits of This Pipeline
Fully Automated: No manual file uploads or imports required.

Always Up-to-Date: Dashboards reflect the latest available data each day.

Centralized Source of Truth: All critical data is in one secure location.

Easily Scalable: Add new data sources by creating/importing new SSIS packages.

Traceable and Documented: Each step can be tracked and monitored.

FAQ & Notes
What if I need to add a new data source?

Request or build a new export script to drop the CSV/XLSX file into d:\Data\IT.

Create an SSIS import package for that file and set it to run after the file lands.

Connect your new SQL table/view to Power BI as needed.

How do I know if the pipeline worked?

Check the last-modified time on the file in d:\Data\IT.

Check SQL Server table data after the SSIS package runs.

Refresh your Power BI dashboard and confirm updated data.

Who manages the scripts and jobs?

File copy automation: Logan/IT

SSIS package setup/maintenance: Data engineering/IT team

Dashboard building: Data analytics/BI team

Contact
For support or to request new data integrations, contact:

Tony Le — Director of IS&T: tony.le@lawrence.k12.ma.us

Kaif Khalifa — Data Engineer: mahammed.khalifa@lawrence.k12.ma.us

LPS Help Desk — help@lawrence.k12.ma.us

This documentation is current as of June 2025. Please update if changes are made to the process, folder locations, or automation jobs.
