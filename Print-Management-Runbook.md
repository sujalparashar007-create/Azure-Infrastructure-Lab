# Print Management — Runbook

## 1. Service Overview

- **Purpose:** Provides Windows Server printer administration, including managing installed printers and print queues.
- **VM:** Main Workload EC2
- **Hostname:** `EC2AMAZ-NKG844S`
- **AWS Region:** `eu-north-1`
- **Installed role:** Print Server
- **Printer used for testing:** Microsoft Print to PDF

## 2. Requirements

- Windows Server 2025 EC2 instance with Remote Desktop access.
- Administrator access to Server Manager.
- Print and Document Services role.
- Print Management tools/console.

## 3. Installation

1. Connect to the Main Workload EC2 instance using Remote Desktop.
2. Open **Server Manager → Manage → Add Roles and Features**.
3. Choose **Role-based or feature-based installation** and select the current server.
4. Under **Server Roles**, expand **Print and Document Services**.
5. Select **Print Server**.
6. Complete the wizard and install the role.
   - **Purpose:** The Print Server role enables Windows Server to manage printers and print services.

## 4. Configuration

1. Open **Server Manager → Tools → Print Management**.
2. In the left pane, expand **Print Servers**.
3. Expand the Main Workload EC2 server name.
4. Select **Printers**.
5. Verify that **Microsoft Print to PDF** appears and its status is **Ready**.

- **Note:** Microsoft Print to PDF was the printer used for the local practical test. No physical/network printer was configured or shared during this session.

## 5. Access

1. Create or open a text file named `Print-Test.txt` on the server.
2. Open the file and press **Ctrl + P**.
3. Select **Microsoft Print to PDF** as the printer.
4. Click **Print**.
5. Select the Desktop as the save location and save the output as `Print-Test.pdf`.

## 6. Testing & Verification

1. Open **Print Management → Print Servers → [server name] → Printers**.
2. Confirm **Microsoft Print to PDF** is listed with status **Ready**.
3. Open `Print-Test.txt` and send it to **Microsoft Print to PDF** using **Ctrl + P**.
4. Save the output as `Print-Test.pdf` on the Desktop.
5. Confirm that the PDF was created and opens successfully.

**Verified result:** Print Server role installation completed, Print Management was accessible, Microsoft Print to PDF showed **Ready**, and the `Print-Test.pdf` output was generated successfully.

## 7. Current Status

**Completed**
- **Print and Document Services → Print Server** role installed.
- Print Management opened through **Server Manager → Tools**.
- Microsoft Print to PDF detected with status **Ready**.
- `Print-Test.txt` sent to Microsoft Print to PDF.
- `Print-Test.pdf` generated successfully on the Desktop.

**Not tested in this lab**
- Sharing a physical/network printer.
- Connecting to a shared printer from a separate client VM.
- Sending a print job from another computer to the server-hosted/shared printer.

**Pending**
- Network printer sharing and remote-client print testing remain pending until a separate client VM is available.
