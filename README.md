# LPS-Data-Warehouse




## Overview

This project centralizes key Lawrence Public Schools (LPS) data from multiple sources into a single, in-house Microsoft SQL Server Express database. The warehouse enables unified reporting and analytics, primarily through Power BI Desktop, while keeping all data on LPS-owned hardware (no cloud).

![image](https://github.com/user-attachments/assets/391c8d1c-ae3a-4b4b-af92-e7f1b9c95840)

---

## Project Context

- **Goal:** Build an in-house data warehouse for IT inventory, device, staff, and account data.
- **Data Sources:** Various `.csv` and `.xlsx` exports from Google Drive shared folders (e.g., Meraki, SCCM, AD, Jamf, etc.).
- **Tools Used:**  
  - Microsoft SQL Server 2022 Express (on local server/desktop)  
  - SQL Server Management Studio (SSMS)  
  - Excel/Power Query for basic cleanup  
  - Power BI Desktop for reporting  
- **Constraints:**  
  - Data must remain on LPS-owned hardware (no cloud, no Snowflake)  
  - Free or existing tools only

---

## Table Import Process

1. **Convert all source files to `.csv` format if needed** (Excel files were saved as `.csv`).
2. **Import each file into SQL Server Express using SSMS:**
   - Right-click the database → Tasks → Import Flat File...
   - Choose the file, review column names/types.
   - For all text columns, set data type to `nvarchar(255)` or `nvarchar(max)` to prevent truncation.
   - For columns like Serial Number or Email, ensure appropriate type and size.
   - Set a Primary Key if the data is guaranteed unique (e.g., Serial Number).
3. **Table Naming:**
   - Tables are named to match source files, but will be standardized to use lowercase and underscores for consistency (e.g., `macbooks_inventory`).

---

## Current Tables

| Table Name                | Source File                            |
|-------------------------- |----------------------------------------|
| all_adaccounts            | All-ADaccounts.csv                     |
| chromeosdevices_reports   | ChromeOSDevices_Reports.csv            |
| intunedevices_reports     | InTuneDevices_Reports.csv              |
| ipadgroups_reports        | IPadGroups_Reports.csv                 |
| jamf_pro_computers_export | jamf PRO Computers Export.csv          |
| jamfpro_reports           | JamFPro_Reports.csv                    |
| meraki_export_macbooks    | 2023-06-02-Meraki-Export-Macbooks.csv  |
| sccm_export_monitors      | 2023-06-05-SCCM-Export-Monitors.csv    |
| sccm_export_pcs           | 2023-06-05-SCCM-Export-PCs.csv         |
| sccm_report               | SCCM_Report.csv                        |
| ...                       | ... (add new tables here)              |

---

## Data Refresh Process

- **Tables do NOT update automatically if source files change.**
- When source files are updated:
  1. Re-import the file into the existing table (overwrite or refresh as needed).
  2. Ensure data types/column sizes are still sufficient for new data.

---

## Next Steps

- Create SQL Views for analytics and reporting (e.g., device counts by type, staff lists, etc.)
- Connect Power BI Desktop to this database for dashboards.
- Document data dictionary and standardize table/column names.

---

## Troubleshooting

- **String or binary data would be truncated:**  
  Increase the column size in the import wizard.
- **Wrong data type (e.g., varbinary for emails):**  
  Set the correct type (e.g., `nvarchar(255)`).
- **Primary Key errors:**  
  Only set a primary key if the column is truly unique.

---

## Contact

For questions or updates, contact the LPS Data Engineer/Analyst Kaif or IT Department.
