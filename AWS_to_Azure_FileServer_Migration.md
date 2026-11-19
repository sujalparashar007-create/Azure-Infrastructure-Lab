# README -- AWS EC2 File Server to Microsoft Azure Migration

## 1. Project Objective

-   Build a Windows Server 2025 File Server on AWS EC2.
-   Install the File Server role and create a shared folder with
    permissions.
-   Discover, replicate, test, and migrate the server to Microsoft Azure
    using Azure Migrate.
-   Verify the migrated VM, files, shares, and access in Azure.

## 2. High-Level Architecture

**AWS Source Environment -- ap-southeast-2 (Sydney)** - **Discovery
Appliance:** Discovers the source server and supports assessment. -
**Source File Server:** Windows Server 2025 EC2 running the File Server
role and shared data. - **Replication Appliance:** Supports the Azure
Migrate replication workflow.

**Migration Layer** - Azure Migrate: discovery → assessment →
replication → test migration → final migration.

**Azure Target Environment** - Migrated Windows Server VM. - Verify
server boot, File Server role, shared folder, files, and permissions. -
Azure target region: **To be finalized**.

All three planned EC2 instances are intended to use the same AWS VPC,
with separate Security Groups.

## 3. AWS Environment -- Current Recorded Setup

-   AWS account: New practice account.
-   AWS Region: Asia Pacific (Sydney) -- `ap-southeast-2`.
-   VPC: Default VPC -- `vpc-0af06807cf07b38b1`.
-   Subnet: `subnet-093d68a7a559ea425`.
-   Dedicated Security Groups:
    -   `Discovery-SG` -- `sg-01a7125dd5426d13d`
    -   `FileServer-SG` -- `sg-03cb1d61dbed7aeb8`
    -   `Replication-SG` -- `sg-07def88a8c905d194`
-   Default Security Group exists but is not planned for the three
    appliances.

## 4. Project Phases

### Phase 1 -- Architecture and Planning

-   Understand the role of all three EC2 appliances.
-   Finalize source-to-target migration flow.
-   Confirm AWS VPC, subnet, Security Groups, and required connectivity.
-   Finalize the Azure target region and target network.

### Phase 2 -- AWS Discovery Appliance

-   Finalize instance type, Windows Server 2025 AMI, storage, and
    network settings.
-   Launch and configure the appliance.
-   Install/register Azure Migrate discovery components.
-   Confirm source-server discovery.

### Phase 3 -- AWS Source File Server

-   Launch Windows Server 2025 EC2.
-   Install the File Server role.
-   Create a shared folder, test data, and access permissions.
-   Verify access from an authorized machine.

### Phase 4 -- AWS Replication Appliance

-   Finalize and launch the appliance.
-   Install/register the required replication components.
-   Verify connectivity to the source server and Azure services.

### Phase 5 -- Azure Migrate Setup and Assessment

-   Configure the migration project and required credentials.
-   Discover and assess the source File Server.
-   Review readiness, sizing, and migration requirements.

### Phase 6 -- Replication and Test Migration

-   Configure replication to the selected Azure target.
-   Wait for initial replication and synchronization.
-   Run a test migration in an isolated test network.
-   Verify Windows boot, files, shares, and permissions.

### Phase 7 -- Final Migration and Validation

-   Plan the cutover and downtime.
-   Perform final synchronization and migration.
-   Verify the Azure VM and File Server functionality.
-   Record results and clean up only after successful validation and
    approval.

## 5. Requested AWS Resource Capacity

A separate AWS support case requests review of: - EC2 vCPU quota: **64
vCPUs**. - EC2 memory capacity: **24 GiB** (instance-type
availability/eligibility confirmation; RAM is not normally a standalone
EC2 quota). - EBS storage: **700 GB**, preferred volume type **gp3**.

These are requested capacities, not approved quotas or confirmed free
allowances.

## 6. Current Blockers and Support Cases

-   AWS account currently displays an account-level block when launching
    EC2.
-   Customer address verification remains pending.
-   Payment method verification has failed and needs AWS review.
-   Account Free Plan to Paid Plan upgrade is not complete.

**Support Case 1 -- Account verification and EC2 access** - Case ID:
`179448575200932` - Last observed status: **Open** - Covers
identity/address verification, payment verification, EC2 launch
restriction, and account upgrade request.

**Support Case 2 -- Resource capacity** - Case ID: `179449283700703` -
Last observed status: **Work in progress** - Covers 64 vCPUs, 24 GiB
memory capacity, and 700 GB gp3 EBS requirement.

## 7. Working Rules

-   Proceed in small, verified phases; complete one step before moving
    to the next.
-   Use AWS/Azure portals first and confirm actual displayed values.
-   Do not launch resources while the AWS account is blocked.
-   Do not assume quota approval, Free Plan eligibility, or zero cost.
-   Check estimated charges before creating resources.
-   Do not expose RDP, SMB (445), or WinRM (5985) to the public
    internet.
-   Do not delete resources or change networking without a clear reason
    and confirmation.

## 8. Today's Scope

-   Break the project into phases and understand each component's
    purpose.
-   Finalize the high-level architecture and migration sequence.
-   Prepare the configuration checklist for each appliance and the File
    Server.
-   Track the AWS support blockers.
-   **Resource deployment is pending AWS account access restoration and
    required capacity confirmation.**

## 9. Final Success Criteria

-   Source File Server is discovered and assessed.
-   Replication completes successfully.
-   Test migration validates server and shared-folder functionality.
-   Final Azure VM boots and provides the expected files, shares, and
    permissions.
-   Migration results and any remaining issues are documented.
