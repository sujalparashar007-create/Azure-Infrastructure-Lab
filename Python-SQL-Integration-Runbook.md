# Python-SQL Integration — Runbook

## 1. Service Overview

- **Purpose:** Connect a Python application hosted on an AWS EC2 Windows VM to SQL Server on a separate EC2 Windows VM, then perform database operations through a Flask REST API.
- **AWS Region:** `us-east-1`
- **SQL Server VM:** `Main-Workload-Server`
  - **Instance ID:** `i-07d2a64fb915a7817`
  - **Private IP:** `172.31.44.63`
  - **Security Group:** `Main-Workload-SG`
- **Python VM:** `Python-Service-VM`
  - **Instance ID:** `i-049138e99a7f8d09b`
  - **Private IP:** `172.31.38.192`
  - **Security Group:** `Python-VM-SG`
- **VPC:** `vpc-01832c7030cff8204` (both instances are in the same VPC)
- **Database platform:** SQL Server 2025 Express, named instance `SQLEXPRESS`
- **Database and table:** `CompanyDB` and `dbo.Employees`
- **Python application:** Python 3.15.2, `pyodbc` 5.5.0 and Flask 3.1.3

## 2. Requirements

- Two Windows Server EC2 instances in the same AWS VPC, with Remote Desktop access and administrator permissions.
- SQL Server 2025 Express installed on the Main Workload VM, with the `CompanyDB` database and `dbo.Employees` table available.
- SQL Server Mixed Mode authentication and a SQL login mapped to `CompanyDB`.
- TCP/IP enabled for the `SQLEXPRESS` instance and a fixed SQL Server port configured.
- An inbound rule in the SQL Server VM's AWS security group that allows TCP 1433 only from `Python-VM-SG`.
- A Windows Firewall inbound rule on the SQL Server VM that allows TCP 1433 only from Python VM private IP `172.31.38.192`.
- Python, Microsoft ODBC Driver 18, `pyodbc` and Flask installed on the Python VM.

## 3. Installation

1. Connect to `Python-Service-VM` using Remote Desktop.
2. Install Python using the Python Install Manager and install Python 3.15:
   - `winget install 9NQ7512CXL7T --accept-source-agreements --accept-package-agreements`
   - `py install 3.15`
   - **Purpose:** Installs Python and makes it available for running the database test script and Flask application.
3. Verify Python and pip:
   - `python --version` → `Python 3.15.2`
   - `python -m pip --version` → `pip 26.2.2`
4. Install Microsoft ODBC Driver 18 for SQL Server:
   - `winget install --id Microsoft.msodbcsql.18 --exact --source winget --accept-source-agreements --accept-package-agreements`
   - **Installed version:** `18.7.1.1`
   - **Purpose:** Provides the ODBC driver used by `pyodbc` to communicate with SQL Server.
5. Verify that both 32-bit and 64-bit ODBC Driver 18 entries are installed, and confirm Python is 64-bit.
6. Install the Python database library:
   - `python -m pip install pyodbc`
   - **Installed version:** `5.5.0`
   - **Purpose:** Allows Python code to open SQL Server connections and execute SQL statements.
7. Install Flask:
   - `python -m pip install flask`
   - **Installed version:** `3.1.3`
   - **Purpose:** Provides the web application framework used to expose database operations through HTTP API endpoints.
8. Create the project folder on the Python VM:
   - `C:\PythonSQLIntegration`

## 4. Configuration

1. On `Main-Workload-Server`, verify that **SQL Server (SQLEXPRESS)** is running and that `CompanyDB` contains `dbo.Employees`.
2. In SSMS, open **Server Properties → Security** and select **SQL Server and Windows Authentication mode**.
   - **Purpose:** Enables SQL Server logins in addition to Windows Authentication for the Python application.
3. Create the SQL login `appuser` and map it to `CompanyDB` with the `db_datareader` and `db_datawriter` database roles.
   - The login password was reset after the first remote test returned SQL error 18488 (password change required). The **User must change password at next login** option was disabled during the reset.
   - **Note:** The password is intentionally not recorded in this runbook.
4. In **SQL Server Configuration Manager → SQL Server Network Configuration → Protocols for SQLEXPRESS**, enable **TCP/IP**.
5. In **TCP/IP Properties → IP Addresses → IPAll**, configure:
   - **TCP Dynamic Ports:** blank
   - **TCP Port:** `1433`
   - The SQL Server error log initially showed port `1443`; the IPAll settings were corrected to fixed port `1433` and the SQL Server service was restarted.
6. On `Main-Workload-Server`, create a Windows Firewall inbound rule for TCP port `1433` and restrict **Scope → Remote IP address** to `172.31.38.192`.
7. In AWS **EC2 → Security Groups → Main-Workload-SG → Inbound rules**, add:
   - **Type:** Custom TCP
   - **Port:** `1433`
   - **Source:** `Python-VM-SG`
   - **Purpose:** Allows the Python VM security group to reach SQL Server without exposing the database port to all IP addresses.
8. From `Python-Service-VM`, verify the network path:
   - `Test-NetConnection -ComputerName 172.31.44.63 -Port 1433`
   - **Verified result:** `TcpTestSucceeded : True`
9. Create `test_sql_connection.py` in `C:\PythonSQLIntegration` to test remote SQL authentication and read the existing employee records. The script uses:
   - **Server:** `172.31.44.63,1433`
   - **Database:** `CompanyDB`
   - **Login:** `appuser`
   - **Driver:** `ODBC Driver 18 for SQL Server`
   - **Connection options:** `Encrypt=yes` and `TrustServerCertificate=yes`
10. Create `app.py` in `C:\PythonSQLIntegration` with the Flask REST API. The application provides these endpoints:
    - **GET `/employees`:** Reads employee records from `dbo.Employees`.
    - **POST `/employees`:** Inserts an employee using the JSON request body.
    - **PUT `/employees/<employee_id>`:** Updates the specified employee.
    - **DELETE `/employees/<employee_id>`:** Deletes the specified employee.
11. The Flask application prompts for the SQL login password at startup rather than storing the password directly in the script. It runs with `debug=False` and binds to `127.0.0.1` on port `5000`.

## 5. Access

1. Connect to `Python-Service-VM` using Remote Desktop and open PowerShell.
2. Change to the project directory:
   - `Set-Location "C:\PythonSQLIntegration"`
3. Start the Flask application:
   - `python .\app.py`
   - Enter the `appuser` SQL password at the prompt. Password characters are not displayed while typing.
4. Keep the PowerShell window running while testing the API. The application listens locally at `http://127.0.0.1:5000`.
5. From a second PowerShell window on the same Python VM, send requests to the local Flask endpoints. The application connects to SQL Server using the Main Workload VM's private IP `172.31.44.63` and TCP port `1433`.

## 6. Testing & Verification

1. Verify the SQL Server service **SQL Server (SQLEXPRESS)** is running and connect to `localhost\SQLEXPRESS` in SSMS using Windows Authentication.
2. Confirm `CompanyDB` and `dbo.Employees` are present. The original records were:
   - `(1, Rahul Sharma, IT, 55000.00)`
   - `(2, Amit Verma, HR, 40000.00)`
3. Run `test_sql_connection.py` from the Python VM. The first attempt returned error 18488 because the `appuser` password had to be changed. After resetting the login password, the script connected successfully and retrieved both original employee records.
4. Verify network connectivity from the Python VM using `Test-NetConnection`; TCP port `1433` returned `TcpTestSucceeded : True`.
5. Start `app.py` and confirm Flask reports that it is running at `http://127.0.0.1:5000` with debug mode off.
6. Test **GET** using `Invoke-RestMethod -Uri "http://127.0.0.1:5000/employees" -Method Get`. The API returned Rahul Sharma and Amit Verma.
7. Test **POST** by inserting a temporary record:
   - `EmployeeID`: `3`
   - `EmployeeName`: `Test User`
   - `Department`: `QA`
   - `Salary`: `30000`
   - The API returned **Employee inserted successfully**; a subsequent GET confirmed the record was saved.
8. Test **PUT** on `/employees/3` with `EmployeeName` = `Test User Updated`, `Department` = `QA` and `Salary` = `32000`. The API returned **Employee updated successfully**; a subsequent GET confirmed the updated values.
9. Test **DELETE** using `Invoke-RestMethod -Uri "http://127.0.0.1:5000/employees/3" -Method Delete`. The API returned **Employee deleted successfully**.
10. Run GET again and confirm that only the two original records remain:
    - `(1, Rahul Sharma, IT, 55000.00)`
    - `(2, Amit Verma, HR, 40000.00)`

**Verified result:** Python on `Python-Service-VM` connected remotely to SQL Server on `Main-Workload-Server` through TCP 1433. The Flask API successfully completed and verified GET, POST, PUT and DELETE operations. The temporary test employee was deleted, and both original employee records remained unchanged.

## 7. Current Status

**Completed**
- Python 3.15.2, pip 26.2.2, Microsoft ODBC Driver 18.7.1.1, `pyodbc` 5.5.0 and Flask 3.1.3 installed and verified on `Python-Service-VM`.
- SQL Server Mixed Mode, `appuser`, CompanyDB role mapping, TCP/IP and fixed TCP port 1433 configured on `Main-Workload-Server`.
- Windows Firewall and AWS Security Group access restricted to the Python VM for SQL port 1433.
- Remote Python-to-SQL connection and employee record retrieval verified.
- Flask application created and run locally on port 5000; GET, POST, PUT and DELETE API operations tested successfully.
- Temporary EmployeeID 3 test record deleted; original Rahul Sharma and Amit Verma records verified as remaining.

**Not configured or tested in this lab**
- Flask application access from another machine; the application was bound to `127.0.0.1` and tested locally on the Python VM only.
- Production deployment of the Flask development server.
- A production TLS certificate for the SQL Server connection; the lab connection used `TrustServerCertificate=yes`.

**Pending**
- No pending items remain for the local Python-to-SQL integration and CRUD exercise. AWS Discovery Appliance and Replication Appliance deployment are separate next-phase tasks.
