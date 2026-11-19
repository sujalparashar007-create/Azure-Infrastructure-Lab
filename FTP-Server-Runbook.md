# FTP Server for Backup — Runbook

## 1. Service Overview

- **Purpose:** Transfers files between an FTP client and the Windows Server using FTP (File Transfer Protocol).
- **VM:** Main Workload EC2
- **Hostname:** `EC2AMAZ-NKG844S`
- **AWS Region:** `eu-north-1`
- **FTP site:** `FTP-Backup-Site`
- **Physical folder:** `C:\FTP-Backup`
- **Protocol:** FTP
- **Control port:** TCP `21`

## 2. Requirements

- Windows Server 2025 EC2 instance with Remote Desktop access.
- Administrator access to Server Manager and IIS Manager.
- IIS Web Server role.
- FTP Service and FTP Extensibility components.
- IIS Management Console.
- Inbound firewall access for TCP port `21`.

## 3. Installation

1. Connect to the Main Workload EC2 instance using Remote Desktop.
2. Open **Server Manager → Manage → Add Roles and Features**.
3. Choose **Role-based or feature-based installation** and select the current server.
4. Select **Web Server (IIS)**.
5. Under **Web Server (IIS) → FTP Server**, select:
   - **FTP Service**
   - **FTP Extensibility**
6. Ensure **IIS Management Console** is selected under Management Tools.
7. Continue through the wizard and install the selected components.
   - **Purpose:** IIS hosts the FTP site; FTP Service provides FTP file-transfer functionality.

## 4. Configuration

1. Open **Server Manager → Tools → Internet Information Services (IIS) Manager**.
2. Expand the server, right-click **Sites**, and select **Add FTP Site**.
3. Configure:
   - **FTP site name:** `FTP-Backup-Site`
   - **Physical path:** `C:\FTP-Backup`
4. Configure **Binding and SSL Settings**:
   - **IP Address:** All Unassigned
   - **Port:** `21`
   - **Virtual Host Names:** Unchecked
   - **Start FTP Site Automatically:** Checked
   - **SSL:** No SSL
5. Configure **Authentication and Authorization**:
   - **Anonymous Authentication:** Disabled
   - **Basic Authentication:** Enabled
   - **Allow access to:** Specified users
   - **User:** `Administrator`
   - **Permissions:** Read and Write
6. Finish the wizard and verify that `FTP-Backup-Site` shows **Started**.
7. In **Windows Defender Firewall with Advanced Security → Inbound Rules**, create:
   - **Rule type:** Port
   - **Protocol:** TCP
   - **Specific local ports:** `21`
   - **Action:** Allow the connection
   - **Profiles:** Domain, Private, Public
   - **Name:** `FTP-Server-Inbound-21`
8. The predefined **FTP-Server Traffic-In** inbound rule was also checked and found **Enabled**.

**Lab security note:** This practice site uses **No SSL** and the `Administrator` account. This was a lab configuration, not a production security recommendation. Production FTP access should use secure FTPS, least-privilege accounts, and appropriately scoped firewall rules.

## 5. Access

1. Open File Explorer on the server and enter `ftp://localhost` to access the FTP site locally.
2. For command-line access, open Command Prompt and run:

   ```cmd
   ftp localhost
   ```

3. Sign in with the configured `Administrator` account and its password.
4. At the `ftp>` prompt, use `dir` to list the files.
5. To download the test file, use:

   ```text
   get FTP-Test.txt.txt
   ```

6. The downloaded file is saved to the FTP client's current local working directory (in this test, `C:\Users\Administrator`).

- `localhost` means the current server itself.
- FTP is the file-transfer protocol/service; `C:\FTP-Backup` is the physical folder used by this FTP site.

## 6. Testing & Verification

1. **FTP site status:** Confirm `FTP-Backup-Site` is **Started** in IIS Manager.
2. **FTP connection:** Run `ftp localhost` and confirm the server responds with `220 Microsoft FTP Service`.
3. **Authentication:** Sign in as `Administrator` and confirm `230 User logged in`.
4. **Upload:** Use the local FTP window (`ftp://localhost`) to upload the test text file. Confirm it appears in the FTP location and its content is visible.
5. **Directory listing:** At the command-line `ftp>` prompt, run `dir` and confirm the test file is listed.
6. **Download:** Run `get FTP-Test.txt.txt` and confirm the transfer reports `226 Transfer complete` and `19 bytes received`.
7. **Local verification:** Confirm the downloaded file appears in `C:\Users\Administrator` and opens with the expected content.

**Observed filename detail:** File Explorer displayed the uploaded file as `FTP-Test.txt`, while the FTP `dir` output showed `FTP-Test.txt.txt`. Windows was hiding the final known file extension in Explorer. The download command therefore used the full FTP-listed name, `FTP-Test.txt.txt`.

**Issue encountered and workaround:** The File Explorer download attempt displayed the message, “Your current security settings do not allow you to download files from this location.” The FTP server itself was reachable; the download was then completed successfully using the Windows command-line FTP client (`get`).

**Verified result:** FTP connection, Administrator authentication, directory listing, upload, and command-line download succeeded. The downloaded file was present locally and its content was verified.

## 7. Current Status

**Completed**
- IIS Web Server role and FTP components installed.
- IIS Management Console available.
- FTP site `FTP-Backup-Site` created with physical path `C:\FTP-Backup`.
- Binding configured for All Unassigned on TCP port `21`.
- SSL set to No SSL for this lab.
- Basic Authentication enabled; Anonymous Authentication disabled.
- `Administrator` configured as the specified user with Read and Write permissions.
- Manual inbound firewall rule `FTP-Server-Inbound-21` created.
- Predefined `FTP-Server Traffic-In` inbound rule confirmed enabled.
- FTP site confirmed **Started**.
- FTP connection and authentication verified.
- Test-file upload, directory listing, and command-line download verified.

**Not tested in this lab**
- FTP access from a separate client VM or another computer; the recorded tests used `localhost`.
- FTPS/SSL configuration.
- Production-style least-privilege FTP account configuration.

**Pending**
- No pending item remains for the local FTP upload/download practical test completed on this VM.
