# AWS-to-Azure Migration --- Runbook

## 1. Service Overview

-   **Service:** AWS EC2 to Microsoft Azure migration
-   **Purpose:** Discover AWS Windows Server workloads, replicate them
    to Azure, perform test migration, complete final migration, and
    verify the migrated workloads.
-   **Azure Migrate Project:** `AWS-to-Azure-Migration`
-   **Azure Resource Group:** `AWS-to-Azure-Migration-RG`
-   **Azure Subscription:** `Azure subscription 1`
-   **Source Platform:** AWS EC2
-   **Target Region:** `West US 2`
-   **Source Workloads:**
    -   Main Workload Server --- `EC2AMAZ-NKG844S` (`172.31.44.63`)
    -   Python Server --- `EC2AMAZ-R08838G` (`172.31.38.192`)
-   **Discovery Appliance:** `EC2AMAZ-KSHF7PT` (`172.31.35.98`)
-   **Replication Appliance:** `EC2AMAZ-8TFJGFK` (`172.31.33.38`)

**Scope:** This runbook covers Azure Migrate discovery, replication,
test/final migration, migration-related connectivity troubleshooting,
and cleanup only. Installation and configuration of File Server, FTP,
SQL Server, Python application services, and other workloads inside the
AWS EC2 servers are documented separately and are excluded.

## 2. Requirements

-   AWS EC2 Windows Server source workloads and administrative access.
-   Separate Windows-based Discovery Appliance and Replication
    Appliance.
-   Network connectivity between the appliances and source workloads.
-   Azure subscription access and an Azure Migrate project.
-   Azure target Resource Group, Virtual Network, and replication cache
    storage.
-   Windows local Administrator credentials for source-server
    validation.
-   Access to AWS Security Groups, Windows Defender Firewall, and Azure
    Migrate Configuration Manager.

### 2.1 Appliance and workload inventory

  ---------------------------------------------------------------------------------
  Component        Hostname / Name                Private IP Role
  ---------------- ------------------- --------------------- ----------------------
  Discovery        `EC2AMAZ-KSHF7PT`          `172.31.35.98` Discovers source
  Appliance                                                  servers and performs
                                                             inventory/dependency
                                                             validation

  Replication      `EC2AMAZ-8TFJGFK`          `172.31.33.38` Supports
  Appliance                                                  source-to-Azure
                                                             replication

  Main Workload    `EC2AMAZ-NKG844S`          `172.31.44.63` AWS source workload
  Server                                                     

  Python Server    `EC2AMAZ-R08838G`         `172.31.38.192` AWS source workload
  ---------------------------------------------------------------------------------

**Important:** The Discovery Appliance and Replication Appliance are
separate machines with separate roles. Do not install or troubleshoot
one as though it were the other.

### 2.2 Recorded Azure target configuration

  Setting                        Recorded value
  ------------------------------ ---------------------------------
  Project                        `AWS-to-Azure-Migration`
  Resource Group                 `AWS-to-Azure-Migration-RG`
  Subscription                   `Azure subscription 1`
  Target Region                  `West US 2`
  Target Virtual Network         `aws-migration-vnet`
  Cache Storage Account          `awsmigrationstorage1`
  Main Workload target VM size   `Standard_D2als_v6`
  Main Workload security type    Trusted launch virtual machines

The target VM size and security type above are recorded for the Main
Workload Server. Do not assume that the same target size was selected
for the Python Server unless its own target configuration confirms it.

## 3. Discovery Process

1.  Prepare the AWS source environment and ensure the Discovery
    Appliance is running and reachable.
2.  In Azure Portal, open the `AWS-to-Azure-Migration` Azure Migrate
    project.
3.  Select the discovery workflow for **Physical or other (AWS, GCP,
    Xen, Arc)** workloads.
4.  Configure and register the Discovery Appliance with the Azure
    Migrate project using the project-provided appliance setup process.
5.  Use the Discovery Appliance to discover the AWS source Windows
    Servers.
6.  Confirm that the source workloads appear in Azure Migrate:
    -   `EC2AMAZ-NKG844S` --- Main Workload Server.
    -   `EC2AMAZ-R08838G` --- Python Server.
7.  Review the discovered server inventory and proceed to replication
    configuration.

**Role clarification:** Discovery Appliance is used for discovery and
inventory/dependency validation. Replication is handled through the
separate Replication Appliance and the source Mobility Service.

## 4. Replication Process

1.  Prepare and register the separate Replication Appliance with the
    Azure Migrate project.
2.  In Azure Migrate, select the discovered source workloads for
    replication.
3.  Configure the source and target replication settings, including the
    Azure target region and target infrastructure.
4.  Install/enable the Mobility Service on the source workloads as part
    of the agent-based replication workflow.
5.  Configure the target resource settings. The recorded Main Workload
    target includes:
    -   Region: `West US 2`
    -   Resource Group: `AWS-to-Azure-Migration-RG`
    -   Virtual Network: `aws-migration-vnet`
    -   Cache Storage Account: `awsmigrationstorage1`
    -   VM size: `Standard_D2als_v6`
6.  Start replication and allow the initial replication to complete.
7.  Monitor replication health and synchronization status in Azure
    Migrate.
8.  Confirm that the replicated items are healthy before starting test
    migration.

**Recorded Main Workload replication details:**

-   Appliance displayed in the portal: `AWS-Replicatio` (portal display
    value was truncated).
-   VM ID: `172-31-44-63-05e4887b-f66f-544b-93ea-8b7d6d49a06f`
-   Replication health: Healthy.
-   Configuration issues: No issues.
-   Agent version shown: `9.67.7775.1` (up to date at the time checked).
-   Last synchronization shown: `26/11/2026, 12:36:36 pm`.
-   Current RPO shown: 2 minutes at `26/11/2026, 12:38:36 pm`.

Exact appliance installation commands, complete replication port matrix,
and every individual replication wizard selection were not present in
the retained project record; they are therefore not invented in this
runbook.

## 5. Migration Process

### 5.1 Test Migration

1.  After replication is healthy, select the replicated workload in
    Azure Migrate.
2.  Start **Test migration / Test failover** using an available recovery
    point.
3.  Allow Azure to provision the temporary test VM in the configured
    target environment.
4.  Connect to the test VM and verify Windows startup and workload
    accessibility.
5.  Review the test migration result before proceeding.
6.  After validation, use the Azure Migrate cleanup action to remove the
    temporary test-failover resources.

**Recorded result:** The user confirmed that test migration was
completed. The Main Workload portal showed a test migration timestamp of
`25/11/2026, 7:43:30 pm`.

### 5.2 Final Migration

1.  Confirm that source replication is healthy and the required workload
    checks are complete.
2.  Initiate the final migration/cutover action for the replicated
    workload in Azure Migrate.
3.  Allow the final synchronization and migration operation to complete.
4.  Verify that the target Azure workload is available.
5.  Complete the migration workflow in Azure Migrate.

**Recorded result:** The user subsequently confirmed that final
migration was completed.

### 5.3 Cleanup

1.  Confirm that the test migration is no longer required.
2.  Run the Azure Migrate **Cleanup test migration** action for the
    test-failover resources.
3.  Verify that temporary test resources have been removed and the
    replicated workload/project status reflects the completed operation.

**Recorded result:** The user confirmed that cleanup was completed.

## 6. Connectivity Issues, Fixes & Verification

### 6.1 Port and protocol used during troubleshooting

  ----------------------------------------------------------------------------------
               Port Protocol /    Source           Destination       Purpose
                    Service                                          
  ----------------- ------------- ---------------- ----------------- ---------------
             `5985` TCP --- WinRM Discovery        Main Workload     Remote
                    over HTTP     Appliance        `172.31.44.63`    management,
                                  `172.31.35.98`   and Python Server WinRM
                                                   `172.31.38.192`   connectivity
                                                                     checks, and
                                                                     authenticated
                                                                     PowerShell
                                                                     Remoting for
                                                                     Azure Migrate
                                                                     dependency
                                                                     validation

  ----------------------------------------------------------------------------------

**Port note:** TCP `5985` is the only port explicitly recorded in the
retained troubleshooting history for these fixes. Do not treat it as the
complete Azure Migrate discovery/replication port matrix. Other
migration service ports were not reliably recorded in the available
project notes.

### 6.2 Main Workload Server --- Errors 60001 / WinRM and hostname resolution

  --------------------------------------------------------------------------
  Issue                                  Fix applied
  -------------------------------------- -----------------------------------
  Azure Migrate error `60001`            On the Discovery Appliance, added a
  (`UnableToConnectToPhysicalServer`);   hosts-file mapping:
  Discovery Appliance could not resolve  `172.31.44.63 EC2AMAZ-NKG844S`. On
  the source hostname and WinRM access   the Main Workload Server, changed
  was restricted by the Windows Firewall the **Windows Remote Management
  Public profile.                        (HTTP-In)** Public-profile firewall
                                         rule's remote address to
                                         `172.31.35.98`.

  --------------------------------------------------------------------------

**Why and where:** The hosts mapping allowed the Discovery Appliance to
resolve the source hostname. The firewall change allowed WinRM HTTP
traffic from the Discovery Appliance only.

**Port:** TCP `5985` (WinRM HTTP), from `172.31.35.98` to
`172.31.44.63`.

**Verification performed:** - `Test-NetConnection` to source TCP `5985`
succeeded. - `Test-WSMan` by IP and hostname succeeded. - Authenticated
`Enter-PSSession` succeeded using the source server's local
Administrator credentials. - Azure Migrate **Revalidate** completed
successfully and the Main Workload Server errors cleared.

**AWS Security Group note:** The Main Workload Server Security Group
already had the Discovery Appliance TCP `5985` inbound rule. It was not
newly added during this troubleshooting.

### 6.3 Python Server --- Errors 60001 and 60019

  -----------------------------------------------------------------------
  Issue                               Fix applied
  ----------------------------------- -----------------------------------
  Azure Migrate errors `60001` and    Added a hosts-file mapping on the
  `60019`; hostname resolution and    Discovery Appliance:
  WinRM connectivity from the         `172.31.38.192 EC2AMAZ-R08838G`. On
  Discovery Appliance to the Python   the Python Server, changed the
  Server failed.                      **Windows Remote Management
                                      (HTTP-In)** Public-profile firewall
                                      rule's remote address to
                                      `172.31.35.98`.

  -----------------------------------------------------------------------

**AWS Security Group fix:** The Python Server Security Group initially
allowed TCP `5985` from the Replication Appliance (`172.31.33.38/32`)
but did not include the Discovery Appliance. Added a separate inbound
rule:

-   Type: Custom TCP
-   Port: `5985`
-   Source: `172.31.35.98/32`
-   Description: `WinRM from Discovery Appliance`

The existing Replication Appliance rule was preserved.

**Why and where:** The hosts mapping fixed name resolution; the Windows
Firewall and AWS Security Group changes permitted WinRM HTTP from the
Discovery Appliance to the Python Server.

**Port:** TCP `5985` (WinRM HTTP), from `172.31.35.98` to
`172.31.38.192`.

**Verification performed:** - Confirmed WinRM service was Running and
the HTTP listener was enabled on TCP `5985`. - `Test-NetConnection` to
source TCP `5985` succeeded. - Authenticated `Enter-PSSession` succeeded
using the Python Server's local Administrator credentials. - Azure
Migrate **Revalidate** completed successfully and the Python Server was
resolved.

### 6.4 Discovery Appliance --- RDP disconnection

  -----------------------------------------------------------------------
  Issue                               Checks performed / status
  ----------------------------------- -----------------------------------
  RDP connection to Discovery         Confirmed EC2 instance state was
  Appliance `EC2AMAZ-KSHF7PT`         Running, status checks were `2/2`,
  (`172.31.35.98`) repeatedly closed  and CPU utilization was normal. No
  silently.                           reboot or stop/start was performed.

  -----------------------------------------------------------------------

**Pending investigation recorded:** Check Windows Event Viewer →
**Windows Logs → System**, focusing on `TermDD` and
`RemoteDesktopServices-RdpCoreTS` events around the disconnection time.

**Important:** This RDP issue is separate from the source-server WinRM
dependency-validation issues. Do not treat it as a cause of the resolved
Main Workload or Python Server connectivity errors.

## 7. Testing & Verification

1.  **Discovery verification**
    -   Confirmed that Main Workload Server and Python Server were
        discovered in the Azure Migrate project.
2.  **Replication verification**
    -   Confirmed replication completed for the workloads.
    -   Main Workload portal showed Healthy replication, no
        configuration issues, and zero errors at the time checked.
3.  **Dependency validation**
    -   Verified TCP `5985` connectivity from the Discovery Appliance to
        both source servers.
    -   Verified WinRM and authenticated PowerShell Remoting.
    -   Azure Migrate Revalidate succeeded for both servers.
4.  **Test migration**
    -   Test migration completed, as confirmed by the user.
5.  **Final migration**
    -   Final migration completed, as confirmed by the user.
6.  **Cleanup**
    -   Test migration cleanup completed, as confirmed by the user.

## 8. Current Status

**Completed** - AWS source server discovery. - Replication setup and
replication of the workloads. - Main Workload Server dependency/WinRM
connectivity troubleshooting and successful Revalidate. - Python Server
dependency/WinRM connectivity troubleshooting and successful
Revalidate. - Test migration. - Final migration. - Test migration
cleanup.

**Separate investigation item** - Discovery Appliance silent RDP
disconnection was previously reported. The EC2 health checks and CPU
were normal; the Windows Event Viewer investigation was still pending in
the last recorded troubleshooting notes. The user has confirmed final
migration and cleanup completion, but has not separately confirmed
resolution of this RDP issue.

**Final migration status:** The user confirmed that final migration and
cleanup are completed for the project. The Discovery Appliance RDP
investigation remains a separate operational follow-up unless
subsequently confirmed resolved.
