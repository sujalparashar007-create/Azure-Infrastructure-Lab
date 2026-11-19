# Azure Infrastructure Services -- Deployment & Testing Roadmap

**Objective:** Instructor ki 13 services ko understand karna, suitable
environment mein deploy/configure karna aur practical testing se verify
karna.

  -----------------------------------------------------------------------------------------------------
                 \# Service          Kya kaam karti hai?  Kaise / Kahan install      High-level
                                                          hogi?                      installation &
                                                                                     testing
                                                                                     requirements
  ----------------- ---------------- -------------------- -------------------------- ------------------
                  1 **File Server**  Files/folders share  Existing Azure VM --       File Server Role,
                                     aur permissions      Windows Server Role.       SMB Share, local
                                     manage karta hai.                               user, Share + NTFS
                                                                                     permissions; login
                                                                                     aur file access
                                                                                     test.

                  2 **Print          Printers, drivers    Existing Azure VM -- Print Print Server Role,
                    Management**     aur print queues     and Document Services      printer driver,
                                     centrally manage     Role.                      physical/network
                                     karta hai.                                      ya virtual
                                                                                     printer; queue aur
                                                                                     test print verify.

                  3 **FTP Server for Backup files ko      Existing Azure VM -- IIS   IIS, FTP site,
                    Backup**         network par          FTP Server Role.           storage folder,
                                     transfer/store karta                            user
                                     hai.                                            authentication,
                                                                                     firewall/network
                                                                                     access;
                                                                                     upload/download
                                                                                     test. Prefer
                                                                                     FTPS/SFTP
                                                                                     security.

                  4 **Database**     Applications ka      Separate Database VM       Database engine,
                                     structured data      recommended; SQL           storage, RAM/CPU,
                                     store aur retrieve   Server/PostgreSQL/MySQL.   database/user
                                     karta hai.                                      creation; CRUD
                                                                                     queries aur
                                                                                     connectivity test.

                  5 **Syslog**       Multiple             Separate Syslog Server VM; Syslog
                                     devices/servers ke   Linux/Windows software     application,
                                     logs centrally       depending on product.      network
                                     collect karta hai.                              connectivity,
                                                                                     UDP/TCP 514 as
                                                                                     required; sample
                                                                                     logs receive aur
                                                                                     search test.

                  6 **Sophos**       Antivirus, endpoint  Existing VM par compatible Product/license,
                                     security aur threat  endpoint agent; Sophos     installer, cloud
                                     protection provide   Central cloud console.     account, network
                                     karta hai.                                      connectivity;
                                                                                     agent
                                                                                     registration,
                                                                                     policy aur
                                                                                     security status
                                                                                     verify.

                  7 **StageNow**     Zebra Android        Windows PC par StageNow    Compatible Windows
                                     devices ko configure tool; Zebra devices par    PC, StageNow
                                     aur provision karta  profiles apply honge.      installer,
                                     hai.                                            supported Zebra
                                                                                     device;
                                                                                     barcode/profile
                                                                                     generation aur
                                                                                     device
                                                                                     provisioning test.

                  8 **Digital        Screens/TVs par      Vendor-specific cloud      Signage software,
                    Signage**        announcements,       platform ya separate       license, media
                                     dashboards aur       server; display            player/display,
                                     content display      device/client.             network; content
                                     karta hai.                                      publish aur screen
                                                                                     par display
                                                                                     verify.

                  9 **KnowBe4**      Security-awareness   KnowBe4 cloud platform;    Account/license,
                                     training aur         normal Windows Server      user accounts,
                                     phishing simulation  installation nahi.         email integration;
                                     provide karta hai.                              sample training
                                                                                     aur phishing
                                                                                     simulation test.

                 10 **ManageEngine   Users ko password    Separate Windows Server /  AD DS, domain
                    -- Password      self-service         AD lab; e.g. ADSelfService users, compatible
                    Resets**         reset/change karne   Plus.                      product/license,
                                     deta hai.                                       service account;
                                                                                     password reset aur
                                                                                     policy
                                                                                     verification.

                 11 **AD--Entra      On-premises AD       Domain-joined Windows      AD DS, DNS, Entra
                    Connector**      users/groups ko      Server; Microsoft Entra    tenant, verified
                                     Microsoft Entra ID   Connect Sync or Cloud      domain, required
                                     ke saath synchronize Sync.                      permissions; test
                                     karta hai.                                      user sync aur
                                                                                     sign-in
                                                                                     verification.

                 12 **Terminal       Multiple users ko    Separate Azure Windows     RDS roles, network
                    Server (RDS)**   remote desktop       Server VM -- Remote        access, user
                                     sessions/apps        Desktop Services.          accounts,
                                     provide karta hai.                              licensing;
                                                                                     multiple-user
                                                                                     login aur session
                                                                                     testing.

                 13 **SCCM /         Computers par        Dedicated lab              Supported Windows
                    Configuration    software deployment, infrastructure;            Server, SQL
                    Manager**        inventory aur patch  Configuration Manager site Server, AD/DNS,
                                     management karta     server + SQL Server, as    storage, client
                                     hai.                 required.                  systems; client
                                                                                     deployment aur
                                                                                     test patch
                                                                                     installation.
  -----------------------------------------------------------------------------------------------------

## Deployment order -- high-level phases

### Phase 1 -- Existing Azure VM

1.  File Server -- **Completed**
2.  Print Management
3.  FTP Server for Backup

### Phase 2 -- Basic infrastructure applications

4.  Database
5.  Syslog
6.  Sophos

### Phase 3 -- External platforms and devices

7.  StageNow
8.  Digital Signage
9.  KnowBe4

### Phase 4 -- Active Directory infrastructure

10. ManageEngine -- Password Resets
11. AD--Entra Connector

### Phase 5 -- Advanced server infrastructure

12. Terminal Server (RDS)
13. SCCM / Configuration Manager

*Note:* Ye learning/deployment roadmap hai. Actual dependencies ke
hisaab se order adjust ho sakta hai. KnowBe4, StageNow aur Digital
Signage ko Windows Server par install karna mandatory nahi hai.
Database, Syslog aur Sophos ke liye bhi final product/version select
karne ke baad exact requirements decide hongi.

## Har service ke liye completion criteria

Kisi bhi service ko completed tabhi mark karenge jab:

-   [ ] Purpose aur working samajh aa gaya ho.
-   [ ] Installation/configuration successfully complete ho.
-   [ ] Required service/process running ho.
-   [ ] Practical functionality test pass ho.
-   [ ] Installation requirements, testing results aur blockers document
    ho gaye hon.

**Ab tum is table mein jo changes chahte ho bata do.** Uske baad ise
final project master roadmap bana lenge aur har service ko isi order
mein execute karenge.
