# Python-SQL Integration — Runbook

## 1. Service Overview

- **Purpose:** Connect a Python application hosted on an AWS EC2 Windows VM to SQL Server on a separate EC2 Windows VM, then perform database operations through a Flask REST API.
- **AWS account:** Instructor-provided working AWS account (separate from the user's personal AWS accounts).
- **AWS Region:** `us-east-1`
- **SQL Server VM:** `Main-Workload-Server`
  - **Instance ID:** `i-07d2a64fb915a7817`
  - **Instance type:** `t3.xlarge` (4 vCPU)
  - **Private IP:** `172.31.44.63`
  - **Security Group:** `Main-Workload-SG`
- **Python VM:** `Python-Service-VM`
  - **Instance ID:** `i-049138e99a7f8d09b`
  - **Instance type:** `t3.large` (2 vCPU)
  - **Private IP:** `172.31.38.192`
  - **Security Group:** `Python-VM-SG`
- **VPC:** `vpc-01832c7030cff8204` (both instances are in the same VPC).
- **Database platform:** SQL Server 2025 Express, named instance `SQLEXPRESS`; SSMS 22 was used for database administration.
- **Database and table:** `CompanyDB` and `dbo.Employees`.
- **Python application environment:** Python 3.15.2 (64-bit), `pyodbc` 5.5.0, Microsoft ODBC Driver 18.7.1.1 and Flask 3.1.3.

## 2. Requirements

- Two Windows Server EC2 instances in the same AWS VPC, with Remote Desktop access and administrator permissions.
- SQL Server 2025 Express installed on `Main-Workload-Server`, with the `CompanyDB` database and `dbo.Employees` table available.
- SQL Server Management Studio (SSMS) for checking the database, server properties and employee records.
- SQL Server Mixed Mode authentication and a SQL login mapped to `CompanyDB`.
- TCP/IP enabled for the `SQLEXPRESS` instance and a fixed TCP port configured. SQL Server Browser was disabled, so the Python connection uses the explicit address and port.
- An inbound rule in `Main-Workload-SG` that allows TCP 1433 only from `Python-VM-SG`.
- A Windows Firewall inbound rule on the SQL Server VM that allows TCP 1433 only from Python VM private IP `172.31.38.192`.
- Python, Microsoft ODBC Driver 18, `pyodbc` and Flask installed on the Python VM.

## 3. Installation

1. Connect to `Python-Service-VM` using Remote Desktop and open PowerShell as Administrator.
2. Install Python using the Python Install Manager:
   - `winget install 9NQ7512CXL7T --accept-source-agreements --accept-package-agreements`
   - `py install 3.15`
   - **Purpose:** Installs Python and makes it available for running the database test script and Flask application.
3. Verify Python and pip:
   - `python --version` → `Python 3.15.2`
   - `python -m pip --version` → `pip 26.2.2`
   - Python architecture was verified as **64-bit**.
4. Install Microsoft ODBC Driver 18 for SQL Server:
   - `winget install --id Microsoft.msodbcsql.18 --exact --source winget --accept-source-agreements --accept-package-agreements`
   - **Installed version:** `18.7.1.1`
   - **Purpose:** Provides the ODBC driver used by `pyodbc` to communicate with SQL Server.
   - **Verification:** `Get-OdbcDriver -Name "ODBC Driver 18 for SQL Server"` showed both 32-bit and 64-bit entries.
   - **Installation note:** The first Winget attempt queried the Microsoft Store source and returned certificate error `0x8a15005e`. Specifying `--source winget` successfully installed the driver.
5. Install the Python database library:
   - `python -m pip install pyodbc`
   - **Installed version:** `5.5.0`
   - **Purpose:** Allows Python code to open SQL Server connections and execute SQL statements.
6. Install Flask:
   - `python -m pip install flask`
   - **Installed version:** `3.1.3`
   - **Purpose:** Provides the web application framework used to expose database operations through HTTP API endpoints.
7. Verify the installed ODBC driver is visible to Python:
   - `python -c "import pyodbc; print(pyodbc.drivers())"`
   - **Verified output included:** `ODBC Driver 18 for SQL Server`.
8. Create the project folder on the Python VM:
   - `C:\PythonSQLIntegration`

## 4. Configuration

1. On `Main-Workload-Server`, verify that **SQL Server (SQLEXPRESS)** is running. Connect in SSMS to `localhost\SQLEXPRESS` using **Windows Authentication**.
2. Confirm that `CompanyDB` and `dbo.Employees` are present. The original employee records were:
   - `(1, Rahul Sharma, IT, 55000.00)`
   - `(2, Amit Verma, HR, 40000.00)`
3. In SSMS, open **Server Properties → Security** and select **SQL Server and Windows Authentication mode**.
   - **Purpose:** Enables SQL Server logins in addition to Windows Authentication for the Python application.
   - Apply the setting and restart **SQL Server (SQLEXPRESS)** so the authentication-mode change takes effect.
4. Create the SQL login `appuser` and map it to `CompanyDB` with the `db_datareader` and `db_datawriter` database roles.
   - The first remote login test returned SQL error `18488` because the account password had to be changed. Reset the `appuser` password and clear **User must change password at next login**.
   - **Note:** The password is intentionally not recorded in this runbook.
5. In **SQL Server Configuration Manager → SQL Server Network Configuration → Protocols for SQLEXPRESS**, enable **TCP/IP**.
6. In **TCP/IP Properties → IP Addresses → IPAll**, configure:
   - **TCP Dynamic Ports:** blank
   - **TCP Port:** `1433`
   - The SQL Server error log initially showed listeners on port `1443` (and dynamic ports). The IPAll settings were corrected to use fixed port `1433`, then the SQL Server service was restarted.
   - **Verification:** `Get-NetTCPConnection -LocalPort 1433 -State Listen` showed listeners on IPv4 `0.0.0.0:1433` and IPv6 `:::1433`.
7. On `Main-Workload-Server`, create a Windows Firewall inbound rule:
   - **Rule type:** Port
   - **Protocol:** TCP
   - **Local port:** `1433`
   - **Action:** Allow the connection
   - **Profiles selected:** Domain, Private and Public
   - **Scope → Remote IP address:** `172.31.38.192` only
   - **Rule name:** `Allow SQL 1433 from Python VM`
8. In AWS **EC2 → Security Groups → Main-Workload-SG → Inbound rules**, add:
   - **Type:** Custom TCP
   - **Port:** `1433`
   - **Source:** `Python-VM-SG`
   - **Purpose:** Allows the Python VM security group to reach SQL Server without exposing the database port to all IP addresses.
9. From `Python-Service-VM`, verify the network path:
   - `Test-NetConnection -ComputerName 172.31.44.63 -Port 1433`
   - **Verified result:** `TcpTestSucceeded : True`.
10. Create `test_sql_connection.py` in `C:\PythonSQLIntegration` to test remote SQL authentication and read the existing employee records. The script uses:
    - **Server:** `172.31.44.63,1433`
    - **Database:** `CompanyDB`
    - **Login:** `appuser`
    - **Driver:** `ODBC Driver 18 for SQL Server`
    - **Connection options:** `Encrypt=yes` and `TrustServerCertificate=yes`
    - The script prompts for the SQL password using `getpass`; the password is not stored in the script.
11. Run `test_sql_connection.py` from the Python VM. The successful output confirmed:
    - **Database:** `CompanyDB`
    - **Login:** `appuser`
    - The two original `dbo.Employees` records were retrieved successfully.
12. Create `app.py` in `C:\PythonSQLIntegration` with the Flask REST API. The application provides these routes:
    - **GET `/`:** Returns a short message indicating that the Flask SQL API is running.
    - **GET `/employees`:** Reads employee records from `dbo.Employees`.
    - **POST `/employees`:** Inserts an employee using the JSON request body.
    - **PUT `/employees/<employee_id>`:** Updates the specified employee.
    - **DELETE `/employees/<employee_id>`:** Deletes the specified employee.
13. The Flask application prompts for the SQL login password at startup rather than storing the password directly in the script. It runs with `debug=False` and binds to `127.0.0.1` on port `5000`.
    - **Access limitation:** Binding to `127.0.0.1` means the API is available only locally on the Python VM. No remote-client access to the Flask API was configured or tested.

## 5. Access

1. Connect to `Python-Service-VM` using Remote Desktop and open PowerShell.
2. Change to the project directory:
   - `Set-Location "C:\PythonSQLIntegration"`
3. Start the Flask application:
   - `python .\app.py`
   - Enter the `appuser` SQL password at the prompt. Password characters are not displayed while typing.
4. Confirm that Flask reports **Debug mode: off** and is running at `http://127.0.0.1:5000`.
   - The Flask development-server warning appeared during startup. This was a lab test; production deployment was not performed.
5. Keep the first PowerShell window running while testing the API. Open a second PowerShell window on the same Python VM and send requests to the local Flask endpoints.
6. The Flask application connects to SQL Server using the Main Workload VM's private IP `172.31.44.63` and TCP port `1433`; the Flask API itself was accessed locally through `127.0.0.1:5000`.

## 6. Testing & Verification

1. Verify the SQL Server service **SQL Server (SQLEXPRESS)** is running and connect to `localhost\SQLEXPRESS` in SSMS using Windows Authentication.
   - **Troubleshooting note:** SSMS initially displayed the message **“SQL Server Management Studio encountered a problem and was shut down.”** SSMS was subsequently opened and used to connect to the instance successfully.
2. Confirm `CompanyDB` and `dbo.Employees` are present. The original records were:
   - `(1, Rahul Sharma, IT, 55000.00)`
   - `(2, Amit Verma, HR, 40000.00)`
3. Run `test_sql_connection.py` from the Python VM. The first attempt returned SQL error `18488` because the `appuser` password had to be changed; the ODBC output also included **“Invalid connection string attribute (0)”**. After resetting the password and disabling **User must change password at next login**, the script connected successfully and retrieved both original employee records.
4. Verify network connectivity from the Python VM using `Test-NetConnection`; TCP port `1433` returned `TcpTestSucceeded : True`.
5. Start `app.py` and confirm Flask reports that it is running at `http://127.0.0.1:5000` with debug mode off.
6. Test **GET** using `Invoke-RestMethod -Uri "http://127.0.0.1:5000/employees" -Method Get`. The API returned Rahul Sharma and Amit Verma.
7. Test **POST** by inserting a temporary record:
   - **EmployeeID:** `3`
   - **EmployeeName:** `Test User`
   - **Department:** `QA`
   - **Salary:** `30000`
   - The API returned **Employee inserted successfully**; a subsequent GET confirmed that the record was saved.
8. Test **PUT** on `/employees/3` with `EmployeeName` = `Test User Updated`, `Department` = `QA` and `Salary` = `32000`. The API returned **Employee updated successfully**; a subsequent GET confirmed the updated values.
9. Test **DELETE** using `Invoke-RestMethod -Uri "http://127.0.0.1:5000/employees/3" -Method Delete`. The API returned **Employee deleted successfully**.
10. Run GET again and confirm that only the two original records remain:
    - `(1, Rahul Sharma, IT, 55000.00)`
    - `(2, Amit Verma, HR, 40000.00)`

**Verified result:** Python on `Python-Service-VM` connected remotely to SQL Server on `Main-Workload-Server` through TCP 1433. The Flask API successfully completed and verified GET, POST, PUT and DELETE operations. The temporary test employee was inserted, updated, retrieved, and deleted. The final GET confirmed that both original employee records remained unchanged.

## 7. Current Status

**Completed**
- Python 3.15.2 (64-bit), pip 26.2.2, Microsoft ODBC Driver 18.7.1.1, `pyodbc` 5.5.0 and Flask 3.1.3 installed and verified on `Python-Service-VM`.
- SQL Server Mixed Mode, `appuser`, `CompanyDB` role mapping, TCP/IP and fixed TCP port 1433 configured on `Main-Workload-Server`.
- Windows Firewall and AWS Security Group access restricted to the Python VM for SQL port 1433.
- Remote Python-to-SQL connection and employee record retrieval verified.
- Flask application created and run locally on port 5000; GET, POST, PUT and DELETE API operations tested successfully.
- Temporary EmployeeID 3 test record deleted; original Rahul Sharma and Amit Verma records verified as remaining.

**Not configured or tested in this lab**
- The Flask application was not exposed to another machine; it was bound to `127.0.0.1` and tested locally on the Python VM only.
- The Flask development server was used for the lab; production deployment was not performed.
- A production TLS certificate for the SQL Server connection was not configured; the lab connection used `TrustServerCertificate=yes`.
- The Flask root route `/` was included in the code but was not separately tested.

**Pending**
- No pending items remain for the local Python-to-SQL integration and CRUD exercise. AWS Discovery Appliance and Replication Appliance deployment are separate next-phase tasks.
