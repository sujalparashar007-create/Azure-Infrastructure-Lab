# Database Server (SQL Server) — Runbook

## 1. Service Overview

- **Service:** Database Server (Microsoft SQL Server)
- **Purpose:** Stores, manages, and retrieves structured data using SQL queries.
- **Server:** AWS Main Workload EC2
- **Hostname:** `EC2AMAZ-NKG844S`
- **SQL Server Edition:** SQL Server 2025 Express
- **Instance Name:** `SQLEXPRESS`
- **Management Tool:** SQL Server Management Studio (SSMS) 22

## 2. Requirements

- Windows Server 2025 EC2 instance.
- Administrator access through Remote Desktop.
- SQL Server 2025 Express installer.
- SQL Server Management Studio (SSMS) 22 installer.
- Sufficient system resources for SQL Server installation and testing.

## 3. Installation

1. Connect to the AWS Main Workload EC2 instance through Remote Desktop.
2. Launch the SQL Server 2025 Express installer.
3. Select **Custom** to download the installation files.
4. Open **SQL Server Installation Center**.
5. Select **New SQL Server stand-alone installation or add features to an existing installation**.
6. Select the free **Express** edition.
7. Leave **Azure Extension for SQL Server** unconfigured.
8. Select **Database Engine Services** under Feature Selection.
9. Configure the named instance:
   - **Instance Name:** `SQLEXPRESS`
   - **Instance ID:** `SQLEXPRESS`
10. Configure Server Configuration:
    - **SQL Server Database Engine Startup Type:** Automatic
    - **SQL Server Browser:** Disabled
    - Default service account configuration retained.
11. Configure Database Engine:
    - **Authentication Mode:** Windows Authentication.
    - Added the current user using **Add Current User**.
12. Retain the default Data Directories and TempDB settings.
13. Accept the recommended memory configuration when prompted.
14. Complete the SQL Server installation successfully.
15. Download and install SQL Server Management Studio (SSMS) 22 separately.

## 4. Configuration

1. Open **Windows Services** and verify **SQL Server (SQLEXPRESS)** is **Running**.
2. Launch SSMS 22.
3. Connect to SQL Server using:

   | Setting | Value |
   |---|---|
   | Server Name | `localhost\SQLEXPRESS` |
   | Authentication | Windows Authentication |
   | Database Name | Blank during initial connection |

4. Create a new database named `CompanyDB`.
5. Create an `Employees` table with the following structure:

   | Column | Data Type |
   |---|---|
   | `EmployeeID` | `INT PRIMARY KEY` |
   | `EmployeeName` | `VARCHAR(100)` |
   | `Department` | `VARCHAR(50)` |
   | `Salary` | `DECIMAL(10,2)` |

6. Use SQL queries to configure and manage the database.

## 5. Access

1. Open SQL Server Management Studio 22.
2. Connect to the local SQL Server instance using `localhost\SQLEXPRESS`.
3. Use Windows Authentication.
4. Access the database through:

   ```text
   Databases → CompanyDB → Tables → dbo.Employees
   ```

5. Use the **New Query** option to execute SQL commands.

> **Note:** SQL Server was accessed locally on the same EC2 instance. Remote database access was not configured.

## 6. Testing & Verification

1. **SQL Server Service Verification**
   - Checked Windows Services.
   - Confirmed `SQL Server (SQLEXPRESS)` status was **Running**.

2. **SSMS Connection Testing**
   - Connected successfully using `localhost\SQLEXPRESS`.
   - Windows Authentication worked successfully.

3. **Database Creation**
   - Created `CompanyDB`.
   - Verified its presence in Object Explorer.

4. **Table Creation**
   - Created `dbo.Employees`.
   - Verified the table under `CompanyDB`.

5. **INSERT and SELECT Testing**
   - Inserted three sample employee records:

     | EmployeeID | EmployeeName | Department | Salary |
     |---|---|---|---|
     | 1 | Rahul Sharma | IT | 45000 |
     | 2 | Amit Verma | HR | 40000 |
     | 3 | Neha Singh | Finance | 50000 |

   - **INSERT** successfully added the records.
   - **SELECT** successfully retrieved the records.

6. **UPDATE Testing**
   - Updated EmployeeID 1 salary from 45000 to 55000.
   - Verified the updated value using SELECT.

7. **DELETE Testing**
   - Deleted EmployeeID 3.
   - Verified that the deleted record was no longer present.

8. **Final Verification**
   - Final records successfully verified:

     | EmployeeID | EmployeeName | Department | Salary |
     |---|---|---|---|
     | 1 | Rahul Sharma | IT | 55000 |
     | 2 | Amit Verma | HR | 40000 |

**Verified result:** Local testing successful.

## 7. Current Status

**Completed**
- SQL Server 2025 Express installed successfully.
- Named instance `SQLEXPRESS` configured.
- SSMS 22 installed and connected successfully.
- Windows Authentication verified.
- `CompanyDB` database created.
- `dbo.Employees` table created.
- INSERT, SELECT, UPDATE and DELETE operations tested successfully.
- Final database records verified.
- SQL Server service confirmed Running.

**Not tested in this lab**
- Remote SQL Server connectivity from another EC2 instance.
- TCP/IP configuration for remote access.
- AWS Security Group inbound rules for SQL Server.
- Azure Extension / Azure Arc integration.

**Final Status:** SQL Server 2025 Express installation and local database testing successfully completed on AWS Main Workload EC2.
