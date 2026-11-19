# Azure File Server — Runbook

## 1. Service Overview

- **Purpose:** Provides shared folders over a network using SMB (Server Message Block).
- **VM:** `Azure-FileServer-VM`
- **Resource Group:** `Azuremigrate-RG`
- **Region:** West US 2
- **SMB share:** `shares`
- **Shared subfolder:** `companyData`

## 2. Requirements

- Windows Server VM with Remote Desktop access.
- File Server role and management tools.
- Local Windows account: `fileuser`.

## 3. Installation

1. Connect to `Azure-FileServer-VM` using Remote Desktop.
2. Open **Server Manager**.
3. Select **Manage → Add Roles and Features**.
4. In the wizard, choose **Role-based or feature-based installation** and select the current server.
5. Under **File and Storage Services → File and iSCSI Services**, select **File Server**.
6. Complete the wizard and install the role.
   - **Purpose:** The File Server role enables Windows Server to host and manage SMB file shares.

## 4. Configuration

1. Create the folder `companyData` for the shared files.
2. Create an SMB share named `shares` and keep `companyData` as a subfolder inside it.
   - **Important:** `shares` is the share name; `companyData` is a folder inside that share, not a separate share.
3. Create the local Windows user `fileuser`.
4. Configure **Share Permissions** for `shares`:
   - `Everyone` — Read: **Allow**
   - `fileuser` — Read and Change: **Allow**
   - Full Control was **not** granted.
5. Configure **NTFS Permissions** on the `companyData` folder for `fileuser`:
   - Read & Execute
   - List Folder Contents
   - Read
   - Modify
   - Write
   - Full Control was **not** granted.
   - **Guidance:** Share permissions control access through the network share. NTFS permissions control access to the folder and files on the Windows file system; effective access is limited by both permission layers.

## 5. Access

1. Open **Command Prompt** on the server.
2. Run this command to connect to the share using the local `fileuser` account:

   ```cmd
   net use \\localhost\shares /user:.\fileuser *
   ```

   - `localhost` refers to the same server.
   - `.\fileuser` specifies the local Windows account.
   - `*` makes Windows prompt for the password instead of placing it in the command.
3. Enter the `fileuser` password when prompted.
4. Open the shared folder:

   ```text
   \\localhost\shares\companyData
   ```

## 6. Testing & Verification

1. Confirm the command returns:

   ```text
   The command completed successfully.
   ```

2. Open `\\localhost\shares\companyData` after authenticating as `fileuser`.
3. Create a file named `FileServer-Test.txt`.
   - **Expected result:** The file is created successfully.
4. Open the file, make a change, and save it.
   - **Expected result:** The edited content is saved successfully.
5. Delete `FileServer-Test.txt`.
   - **Expected result:** The file is deleted successfully.
6. Confirm all create, edit/save, and delete operations work.

**Verified result:** SMB authentication and folder access succeeded. The test file was created, edited/saved, and deleted successfully.

## 7. Current Status

**Completed**
- File Server role installed.
- SMB share `shares` created, with `companyData` as its subfolder.
- Local user `fileuser` created.
- Share and NTFS permissions configured.
- SMB authentication and folder access verified.
- Create, edit/save, and delete tests passed.

**Not tested in this lab**
- Access from a separate client VM or another computer over the network. The recorded access and file-operation tests used `localhost`.

**Pending**
- No pending item remains for the File Server practical test scope completed on this VM.
