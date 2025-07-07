# Windows-Server-Home-Lab-AD-GPO-FileShare
Comprehensive home lab demonstrating Windows Server deployment, Active Directory, DHCP, DNS, file sharing, and Group Policy-based Folder Redirection."
# Windows Server 2022 Home Lab: Active Directory, File Services & GPO

This repository documents the step-by-step setup and configuration of a foundational Windows Server 2022 home lab environment using Hyper-V. The lab demonstrates core skills in Active Directory Domain Services (AD DS), DNS, DHCP, file server management, Group Policy Object (GPO) implementation, and basic troubleshooting.

## Technologies Used
* **Hyper-V:** Virtualization platform for VM creation and management.
* **Windows Server 2019/2022:** Operating system for Domain Controller and File Server roles.
* **Active Directory Domain Services (AD DS):** Domain deployment, OU structure, user/group management.
* **DNS (Domain Name System):** Name resolution for the domain.
* **DHCP (Dynamic Host Configuration Protocol):** Automated IP address assignment.
* **Group Policy Objects (GPO):** Centralized configuration management, specifically for Folder Redirection.
* **SMB (Server Message Block):** File sharing protocol.
* **NTFS Permissions:** Granular security control for shared folders.
* **Windows 10/11 Client:** Workstation for testing domain integration and GPO application.

## Table of Contents

1.  [Lab Environment Overview](#1-lab-environment-overview)
2.  [DC01: Initial Domain Controller Setup](#2-dc01-initial-domain-controller-setup)
3.  [FS01: File Server Setup](#3-fs01-file-server-setup)
4.  [StanlyPC: Client PC Setup](#4-stanlypc-client-pc-setup)
5.  [Active Directory User & Group Management](#5-active-directory-user--group-management)
6.  [File Share Configuration on FS01](#6-file-share-configuration-on-fs01)
7.  [Group Policy for Folder Redirection](#7-group-policy-for-folder-redirection)
8.  [Troubleshooting: Client Login Issue](#8-troubleshooting-client-login-issue)
9. [Future Lab Enhancements](#10-future-lab-enhancements)

---

## 1. Lab Environment Overview

This lab utilizes Hyper-V to create a virtual network and host multiple Windows Server 2022 and Windows 10/11 client virtual machines.

**Virtual Network:**

* **Virtual Switch:** `Internal_Lab_Switch` (Internal network, isolating lab VMs from the host's external network, allowing VMs to communicate with each other).
* **IP Addressing Scheme:** `192.168.1.0/24`

**Virtual Machines:**

| VM Name    | Role                    | Operating System   | IP Address    | Purpose                                                                                             |
| :--------- | :---------------------- | :----------------- | :------------ | :-------------------------------------------------------------------------------------------------- |
| `DC01`     | Domain Controller       | Windows Server 2022 | `192.168.1.1` | Active Directory Domain Services, DNS, DHCP, Group Policy Management.                               |
| `FS01`     | File Server             | Windows Server 2022 | `192.168.1.10`| Host for shared folders, specifically for user data with Folder Redirection.                        |
| `StanlyPC` | Client PC               | Windows 10/11      | DHCP          | Domain-joined client for testing user logins, Group Policy, and Folder Redirection.                 |

**Domain Name:** `mylabs.com`

**Tools Used:** Hyper-V Manager, Server Manager, Active Directory Users and Computers, Group Policy Management, DNS Manager, File Explorer, Command Prompt/PowerShell.

---

## 2. DC01: Initial Domain Controller Setup

This section details the creation of the first Domain Controller (`DC01`), which establishes the Active Directory forest and domain for `mylabs.com`, providing core services like DNS and DHCP.

### 2.1. VM Creation

1.  **Create New Virtual Machine in Hyper-V Manager:**
    * **Name:** `DC01`
    * **Generation:** Generation 2
    * **Startup Memory:** 4096 MB (4 GB) (Dynamic Memory disabled for stability)
    * **Network Connection:** `Internal_Lab_Switch`
    * **Virtual Hard Disk:** At least 100 GB VHDX.
    * **Installation Option:** Install Windows Server 2022 from ISO.
  
### 2.2. Operating System Installation
1.  Connect to the `DC01` VM and start it.
2.  Follow the Windows Server 2022 installation prompts.
3.  Select **Windows Server 2022 Standard (Desktop Experience)**.
4.  Accept license terms.
5.  Choose **Custom: Install Windows only (advanced)**.
6.  Select the virtual hard disk.
7.  Set the Administrator password when prompted.

### 2.3. Basic Configuration
1.  **Log in** to `DC01` using the local Administrator account.
2.  **Change Computer Name:**
    * Open **Server Manager** -> **Local Server**.
    * Click the current computer name (e.g., `WIN-RANDOMCHARS`).
    * Click **"Change..."**, type `DC01`, and click **OK**.
    * `[Screenshot: Computer Name Change Dialog]`
3.  **Configure Static IP Address & DNS:**
    * Open **Server Manager** -> **Local Server`.
    * Click the IP address next to "Ethernet".
    * Right-click the network adapter (e.g., "Ethernet") and select **Properties**.
    * Select **Internet Protocol Version 4 (TCP/IPv4)** and click **Properties**.
    * Select **"Use the following IP address:"**:
        * **IP address:** `192.168.1.1`
        * **Subnet mask:** `255.255.255.0`
        * **Default gateway:** `192.168.1.1` (itself, as it will also be the router for the lab segment)
        * **Preferred DNS server:** `127.0.0.1` (itself)
        * **Alternate DNS server:** Leave blank.
          
![DC01-IPConfig](https://github.com/user-attachments/assets/8455b7a0-b195-4586-9ae5-d90f03d82a80)

4.  **Restart `DC01`** to apply the computer name change.

### 2.4. Install Active Directory Domain Services (AD DS) Role

1.  After restarting, **log in** to `DC01` using the local Administrator account.
2.  Open **Server Manager**.
3.  Click **"Add Roles and Features"**.
4.  Click **Next** until you reach "Server Roles".
5.  Check **"Active Directory Domain Services."**
6.  Click **"Add Features"** when prompted.
7.  Click **Next** through the remaining wizard pages.
8.  Click **"Install."**
9.  Wait for the installation to complete.

### 2.5. Promote Server to Domain Controller (Create New Forest)

1.  After the AD DS role installation, click the **yellow exclamation mark** flag in Server Manager's title bar.
2.  Click **"Promote this server to a domain controller."**
3.  On the "Deployment Configuration" page:
    * Select **"Add a new forest."**
    * For "Root domain name:", type `mylabs.com`.
    * `[Screenshot: Deployment Configuration - Add a new forest]`
4.  On the "Domain Controller Options" page:
    * Ensure "Domain Name System (DNS) server" and "Global Catalog (GC)" are checked.
    * For "Functional level," leave the default (Windows Server 2016 or 2022).
    * **Directory Services Restore Mode (DSRM) password:** Enter a strong password and confirm it. **Remember this password!**
5.  Click **Next** through "DNS Options" (ignore any warnings about delegation for a lab).
6.  Click **Next** through "Additional Options" (NetBIOS domain name will be `MYLABS`).
7.  Click **Next** through "Paths" (leave defaults).
8.  Click **Next** through "Review Options."
9.  On "Prerequisites Check," ensure all critical checks pass. Click **"Install."**
10. The server will automatically restart once the promotion is complete.

### 2.6. Post-Promotion Verification

1.  After `DC01` restarts, **log in** using `mylabs\Administrator` (or `Administrator@mylabs.com`) and the password you set during promotion.
2.  **Verify DNS Service:**
    * Open **Server Manager** -> **Tools** -> **DNS**.
    * Expand `DC01` -> `Forward Lookup Zones`.
    * Confirm `mylabs.com` is present and contains records like `(Same as parent folder)` for `DC01` and `_msdcs`.
      
![DNS](https://github.com/user-attachments/assets/65d756f2-6f25-4a80-a7b5-d888a4bfce19)

3.  **Verify Active Directory Users and Computers (ADUC):**
    * Open **Server Manager** -> **Tools** -> **Active Directory Users and Computers**.
    * Confirm the `mylabs.com` domain is present. You should see default OUs like `Computers`, `Users`, and `Domain Controllers`.

![ADUC-OU Structure](https://github.com/user-attachments/assets/012e5d66-c6f8-43dc-8863-0806ea6ad677)
  
4.  **Verify DHCP (Initial Check):**
    * Open **Server Manager** -> **Tools** -> **DHCP**.
    * You'll likely see a yellow exclamation mark. Right-click `DC01` and select **"Authorize."** Then right-click again and select **"Refresh."** The exclamation mark should disappear.
   
![DHCP IP Range](https://github.com/user-attachments/assets/f3696cd7-dd37-46e9-a750-e7d31e632be0)
![DHCP Address Lease](https://github.com/user-attachments/assets/0524dacd-d83f-4ed5-9510-1eda97695e7e)

---

## 3. FS01: File Server Setup

This section outlines the deployment of `FS01`, a dedicated file server, which will host user data shares and integrate with Active Directory for permissions.

### 3.1. VM Creation

1.  **Create New Virtual Machine in Hyper-V Manager:**
    * **Name:** `FS01`
    * **Generation:** Generation 2
    * **Startup Memory:** 2048 MB (2 GB) (Dynamic Memory disabled)
    * **Network Connection:** `Internal_Lab_Switch`
    * **Virtual Hard Disks:**
        * Primary (OS) VHDX: At least 60 GB.
        * **Additional Data VHDX:** Create a second VHDX (e.g., 100-200 GB) for user data. This is a best practice to separate OS from data.
    * **Installation Option:** Install Windows Server 2022 from ISO.
    * `[Screenshot: VM Creation Wizard Summary for FS01 with two VHDXs]`

### 3.2. Operating System Installation

1.  Connect to the `FS01` VM and start it.
2.  Follow the Windows Server 2022 installation prompts (choose **Standard Desktop Experience**).
3.  Set the Administrator password.

### 3.3. Basic Configuration & Domain Join

1.  **Log in** to `FS01` using the local Administrator account.
2.  **Change Computer Name:**
    * Open **Server Manager** -> **Local Server**.
    * Change the computer name to `FS01`.
3.  **Configure Static IP Address & DNS:**
    * Open **Server Manager** -> **Local Server`.
    * Click the IP address next to "Ethernet".
    * Right-click the network adapter and select **Properties**.
    * Select **Internet Protocol Version 4 (TCP/IPv4)** and click **Properties**.
    * Select **"Use the following IP address:"**:
        * **IP address:** `192.168.1.10`
        * **Subnet mask:** `255.255.255.0`
        * **Default gateway:** `192.168.1.1` (Your `DC01`'s IP, acting as gateway)
        * **Preferred DNS server:** `192.168.1.1` (Your `DC01`'s IP - crucial for domain join)
        * **Alternate DNS server:** Leave blank.

![FS01_IPconfig](https://github.com/user-attachments/assets/37403fea-0c0a-412c-9111-a0f9a53d0e80)
    
4.  **Join to Domain (`mylabs.com`):**
    * On the same "Computer Name/Domain Changes" dialog, click **"Change..."** again (if not already there).
    * Select **"Domain:"** and type `mylabs.com`.
    * Click **OK**.
    * Provide **domain administrator credentials** (`mylabs\Administrator` and password).
    * Click **OK**, then **OK** on the welcome message.
5.  **Restart `FS01`** to apply changes and complete the domain join.

### 3.4. Initialize and Format Data Drive

1.  After `FS01` restarts, **log in** using `mylabs\Administrator`.
2.  Open **Server Manager**.
3.  Go to **Tools** -> **Computer Management**.
4.  In Computer Management, go to **Disk Management**.
5.  Locate the new, unallocated disk (your second VHDX).
6.  **Initialize Disk:** Right-click the disk and select **"Initialize Disk."** Choose **GPT** (GUID Partition Table) and click **OK**.
7.  **Create New Simple Volume:** Right-click the unallocated space on the initialized disk and select **"New Simple Volume..."**
8.  Follow the wizard:
    * Assign the **maximum size**.
    * Assign a drive letter (e.g., `D:`).
    * **Format the volume:**
        * File system: `NTFS`
        * Allocation unit size: `Default`
        * Volume label: `Data` (or similar)
        * **Perform a quick format.**
    * Click **Finish**.
    * Confirm the new `D:` drive appears in File Explorer.

### 3.5. Install File Server Role

1.  On `FS01`, open **Server Manager**.
2.  Click **"Add Roles and Features"**.
3.  Click **Next** until you reach "Server Roles".
4.  Check **"File Server"** under "File and Storage Services" -> "File and iSCSI Services".
5.  Click **Next** through the remaining wizard pages.
6.  Click **"Install."**
7.  Wait for the installation to complete.

---

## 4. StanlyPC: Client PC Setup

This section details the setup of a client workstation (`StanlyPC`), which will be domain-joined to test user logins, Group Policy application, and Folder Redirection.

### 4.1. VM Creation

1.  **Create New Virtual Machine in Hyper-V Manager:**
    * **Name:** `StanlyPC`
    * **Generation:** Generation 2
    * **Startup Memory:** 2048 MB (2 GB) or 4096 MB (4 GB) (Dynamic Memory enabled is fine for clients).
    * **Network Connection:** `Internal_Lab_Switch`
    * **Virtual Hard Disk:** At least 60 GB VHDX.
    * **Installation Option:** Install Windows 10/11 from ISO.

### 4.2. Operating System Installation

1.  Connect to the `StanlyPC` VM and start it.
2.  Follow the Windows 10/11 installation prompts.
3.  Choose the **Pro** version if given the option, as it's required for domain joining.
4.  Set up a local user account for initial login.

### 4.3. Basic Configuration & Domain Join

1.  **Log in** to `StanlyPC` with the local user account created during installation.
2.  **Verify Network Connectivity:** Ensure `StanlyPC` receives a DHCP IP address from `DC01` and can ping `DC01` (`192.168.1.1`).
    * Open Command Prompt and type `ipconfig`.
    * Type `ping 192.168.1.1`.

  ![PC_IPconfig](https://github.com/user-attachments/assets/6234e348-869a-4caa-8b5b-085b579d0f08)

3.  **Change Computer Name & Join Domain:**
    * Right-click the Start button, go to **System** (or search "About your PC").
    * Click **"Rename this PC (Advanced)"** or "Advanced system settings" -> "Computer Name" tab -> "Change...".
    * Change the computer name to `StanlyPC`.
    * Select **"Domain:"** and type `mylabs.com`.
    * Click **OK**.
    * Provide **domain administrator credentials** (`mylabs\Administrator` and password).
    * Click **OK**, then **OK** on the welcome message.

![System_Properties](https://github.com/user-attachments/assets/7d6724cf-c824-4501-89e3-745356a7f5d1)

4.  **Restart `StanlyPC`** to complete the domain join.

---

## 5. Active Directory User & Group Management

This section details the creation of Organizational Units (OUs), a standard user account (`Stanly.Sunny`), and a security group (`IT_Group`) to logically organize users and facilitate granular permissions and GPO application.

### 5.1. Create Organizational Units (OUs)

1.  On `DC01`, log in as `mylabs\Administrator`.
2.  Open **Server Manager** -> **Tools** -> **Active Directory Users and Computers (ADUC)**.
3.  Expand `mylabs.com`.
4.  Right-click `mylabs.com` -> **New** -> **Organizational Unit**.
5.  Create the following OUs, ensuring "Protect container from accidental deletion" is checked for each during creation:
    * `_Users` (for standard user accounts)
    * `_Groups` (for security groups)
    * `_Computers` (for client computer accounts, if not using default Computers container)
    * `_Admin` (for delegated admin accounts, if creating any)

![ADUC-OU Structure](https://github.com/user-attachments/assets/f9fe8dda-06fa-4459-b5ca-7939a77aa9d2)

### 5.2. Create User Account (`Stanly.Sunny`)

1.  In ADUC on `DC01`, navigate to the `_Users` OU.
2.  Right-click `_Users` -> **New** -> **User**.
3.  Fill in the user details:
    * **First name:** `Stanly`
    * **Last name:** `Sunny`
    * **Full name:** `Stanly Sunny` (Auto-filled)
    * **User logon name:** `stanly.sunny`
    * `[Screenshot: New User Object Wizard - User Details]`
4.  Click **Next**.
5.  Set a strong password for `Stanly.Sunny`.
6.  Uncheck "User must change password at next logon" (for lab convenience).
7.  Check "Password never expires."
8.  Click **Next**, then **Finish**.


### 5.3. Create Security Group (`IT_Group`)

1.  In ADUC on `DC01`, navigate to the `_Groups` OU.
2.  Right-click `_Groups` -> **New** -> **Group**.
3.  Fill in the group details:
    * **Group name:** `IT_Group`
    * **Group scope:** `Global` (default, suitable for most cases)
    * **Group type:** `Security` (default)
4.  Click **OK**.
5.  **Add `Stanly.Sunny` to `IT_Group`:**
    * Double-click `IT_Group` to open its properties.
    * Go to the **Members** tab.
    * Click **"Add..."**
    * Type `stanly.sunny` and click **"Check Names."**
    * Click **OK**, then **OK**.
  
![ADUC - IT_Group Members](https://github.com/user-attachments/assets/5295c3f7-00c8-4ed8-a340-eeffd3bc5735)

---

## 6. File Share Configuration on FS01

This section details setting up a shared folder on `FS01` that will store user redirected folders, with appropriate share and NTFS permissions to ensure data security and accessibility.

### 6.1. Create Data Folder Structure

1.  On `FS01`, log in as `mylabs\Administrator`.
2.  Open **File Explorer**.
3.  Navigate to your `D:` drive (the data drive you created earlier).
4.  Create a new folder named `ITUsersData`. This will be the root share for redirected folders.
    

### 6.2. Configure Share Permissions

1.  Right-click the `D:\ITUsersData` folder and select **Properties**.
2.  Go to the **Sharing** tab.
3.  Click **"Advanced Sharing..."**
4.  Check **"Share this folder."**
5.  Click **"Permissions."**
6.  For a simplified lab, you can grant `Everyone` **Full Control** at the share level. This allows you to manage granular access solely through NTFS permissions, which is a common practice.

 ![Share Folder- Sharing Permissions](https://github.com/user-attachments/assets/09eab444-c94b-4500-b385-3a491ceaa752)
   
7.  Click **OK**, then **OK**, then **Close**.

### 6.3. Configure NTFS Permissions (Crucial for Security)

1.  Right-click the `D:\ITUsersData` folder again and select **Properties**.
2.  Go to the **Security** tab.
3.  Click **"Advanced."**
4.  Click **"Disable inheritance"** and select **"Convert inherited permissions into explicit permissions on this object."**
5.  **Remove unnecessary permissions:** Select entries like `Users`, `Authenticated Users`, and `SYSTEM` if they have broad access that you don't want (keep `Administrators` and `SYSTEM` with Full Control for server management).
6.  **Add `IT_Group` with Modify Permissions:**
    * Click **"Add."**
    * Click **"Select a principal."**
    * Type `IT_Group` and click **"Check Names."**
    * Click **OK**.
    * For "Basic permissions," check **"Modify."** This includes Read, Write, Execute, and Delete.
    * Click **OK**.
7.  **Add `Authenticated Users` with Special Permissions (Crucial for Folder Redirection):**
    * Click **"Add."**
    * Click **"Select a principal."**
    * Type `Authenticated Users` and click **"Check Names."**
    * Click **OK**.
    * For "Basic permissions," check **"List folder / read data"** and **"Create folders / append data."** Also, click **"Show advanced permissions"** and ensure **"Traverse folder / execute file"** is checked. **Crucially, for "Applies to:", select "This folder only".** This allows users to create their own folders but not read others' data.
8.  Ensure the final permissions look correct (e.g., `Administrators` Full Control, `SYSTEM` Full Control, `IT_Group` Modify, `Authenticated Users` with "Create folders / append data" "This folder only").
9.  Click **OK** on all dialogs to apply changes.

   ![Share Folder- NTFS](https://github.com/user-attachments/assets/448ce142-3105-4219-bcca-04b89135f034)

10. **Verify Share Path:** The full network path to your shared folder will be `\\FS01\ITUsersData`.

---

## 7. Group Policy for Folder Redirection

This section details the creation and configuration of a Group Policy Object (GPO) to redirect user profile folders (like Desktop and Documents) to the `FS01` file server, centralizing user data.

### 7.1. Create the Group Policy Object (GPO)

1.  On `DC01`, log in as `mylabs\Administrator`.
2.  Open **Server Manager** -> **Tools** -> **Group Policy Management**.
3.  Expand `Forest: mylabs.com` -> `Domains` -> `mylabs.com`.
4.  Right-click on **`mylabs.com`** or a specific OU (e.g., `_Users` if you want it to apply only there) and select **"Create a GPO in this domain, and Link it here..."**.
5.  Name the GPO: `Folder_Redirection_IT_Users` (or a similar descriptive name).    

### 7.2. Configure Security Filtering and Delegation

It's important to control which users/groups the GPO applies to. We will make sure it applies to `Authenticated Users` but is delegated specifically to our `IT_Group` for testing, and also ensure `Domain Admins` can manage it.

1.  In **Group Policy Management Editor**, select your newly created GPO: `Folder_Redirection_IT_Users`.
2.  In the "Scope" tab, under "Security Filtering":
    * Select **"Authenticated Users"** and click **"Remove."** Confirm the removal.
    * Click **"Add..."**, type `IT_Group`, click **"Check Names"**, then **OK**.   
3.  In the "Delegation" tab:
    * Click **"Add..."**, type `Domain Admins`, click **"Check Names"**, then **OK**.
    * Ensure `Domain Admins` has "Read" and "Edit settings" permissions.
    * This ensures administrators can still manage the GPO.
  
![Redirection_Policy](https://github.com/user-attachments/assets/0eb6fea9-7549-4202-8e43-a0ff0708fdc4)

### 7.3. Configure Folder Redirection Settings

1.  In **Group Policy Management Editor**, right-click the `Folder_Redirection_IT_Users` GPO and select **"Edit..."**. This opens the Group Policy Management Editor.
2.  Navigate to: **User Configuration** -> **Policies** -> **Windows Settings** -> **Folder Redirection**.
3.  Right-click **"Documents"** and select **"Properties."**
4.  On the "Target" tab:
    * Set "Setting:" to **"Basic (Redirect everyone's folder to the same location)."**
    * For "Target folder location:", choose **"Create a folder for each user under the root path."**
    * In "Root Path:", enter the share path: `\\FS01\ITUsersData`
    
![GPO Editor- Documents Redirection](https://github.com/user-attachments/assets/167b3682-d74e-4617-b1de-3afb2e4a0e33)

5.  On the "Settings" tab:
    * **Crucially, uncheck "Grant the user exclusive rights to Documents."** (This allows administrators to access the folder for backup/management purposes, which is standard in enterprise environments).
    * Uncheck "Move the contents of Documents to the new location." (For a clean lab start; for production, you might want this checked).
    * Leave "Also apply redirection policy to Windows 2000, Windows XP, and Windows Server 2003 operating systems" unchecked.
    * Check "Redirect the folder back to the local userprofile location when policy is removed."
    * Check "Delete the local copy of redirected folders when policy is removed."
    * `[Screenshot: Documents Folder Redirection Settings Tab]`
6.  Click **OK**.
7.  Repeat steps 3-6 for the **"Desktop"** folder, using the same root path (`\\FS01\IT

---

### **8. Troubleshooting: Client Login Issue (`StanlyPC`)**

This section details the persistent login issues encountered on the `StanlyPC` client machine for domain users, specifically `mylabs\stanly.sunny`, and the systematic troubleshooting steps undertaken to resolve them.

**8.1. Problem Description**
* **Initial Symptom:** Domain user `mylabs\stanly.sunny` was unable to log in directly to `StanlyPC`, receiving a "Remote Desktop Services" error. Local Administrator (`mylabs\Administrator`) could log in.

![Error while Loggin in when restarted](https://github.com/user-attachments/assets/ca40d1a2-8a18-4f17-86b2-602194dfcd11)

* **Key Observations:**
    * `StanlyPC` was successfully joined to the `mylabs.com` domain.
    * `ping DC01.mylabs.com` and `nslookup DC01.mylabs.com` were successful.
    * Group Policy ("Allow log on locally") was confirmed to be correctly applied to `mylabs\stanly.sunny` via `gpresult /r` and Event Viewer.
    * Initial Event Viewer checks (Application, System, Security logs) showed generic errors or warnings (e.g., DistributedCOM, DNS Client, Time-Service, GPO errors). No `Event ID 4625` (Account Logon Failed) was recorded in the Security log for the failed domain user attempts.

**8.2. Diagnostic Steps & Findings**

The investigation consistently pointed towards underlying issues with network readiness and critical service startup during boot, specifically impacting time synchronization. The absence of `Event ID 4625` suggested the failure was not a standard security policy denial, but a more fundamental issue.

* **Initial Suspects & Eliminations:**
    * **User Rights Assignment:** Confirmed "Allow log on locally" was correctly applied via GPO for `Authenticated Users` (which `mylabs\stanly.sunny` is a part of).
    * **Network Connectivity/DNS:** `ping` and `nslookup` commands confirmed basic connectivity and name resolution to `DC01`.
* **Focus on Time Synchronization:**
    * **Event Log Warnings:** Persistent `Time-Service warning (Event ID 129)` and `DNS Client warnings (Event ID 1014)` were observed on `StanlyPC`, indicating ongoing time synchronization and DNS resolution challenges.
    * **Time Resync Failure on `StanlyPC`:**
        Attempting to force time synchronization from `DC01` on `StanlyPC` failed, suggesting `DC01` was not providing usable time data.
        ```cmd
        w32tm /resync /rediscover
        ```
        * **Result:** "The computer did not resync because no time data was available."
    * **Crucial Discovery: Incorrect Time on DC01:** It was discovered that `DC01`, the authoritative time source for the `mylabs.com` domain, had an incorrect time (e.g., 1:14 PM) despite having the correct date. This large discrepancy (hours off from actual UK time) was the root cause of Kerberos authentication failures, as Kerberos requires clocks to be synchronized within a few minutes.

**8.3. Resolution**

The problem was resolved by manually setting the correct time on the Domain Controller (`DC01`) and then ensuring `StanlyPC` successfully synchronized its time from the now-accurate `DC01`.

* **1. Correcting Time on `DC01` (Manual for Isolated Lab Environment):**
    * **Action:** Manually set the system clock on `DC01` to the accurate current UK time via the Date & Time settings in the Control Panel.
    * **Restart Time Service (Command Prompt as Administrator on `DC01`):**
        ```cmd
        net stop w32time
        net start w32time
        ```
    * **Verification (Command Prompt as Administrator on `DC01`):**
        ```cmd
        w32tm /query /status
        ```
        * **Expected `Source`:** `Local CMOS Clock` or `VM IC Time Synchronization Provider` (as it's an isolated VM not internet-connected).
        * **Crucial:** Visually confirmed that the displayed system time on `DC01`'s taskbar was now accurate and matched a reliable external time source.

* **2. Re-Synchronizing `StanlyPC` with Corrected `DC01` Time (Command Prompt as Administrator on `StanlyPC`):**
    * **Reset Time Service Configuration:**
        ```cmd
        net stop w32time
        w32tm /unregister
        w32tm /register
        net start w32time
        ```
    * **Configure Time Source to DC01:** This command ensures `StanlyPC` looks to `DC01` for time.
        ```cmd
        w32tm /config /syncfromflags:manual /manualpeerlist:"DC01.mylabs.com" /reliable:yes /update
        ```
    * **Force Immediate Resync:** This command pulls the *new, corrected* time from `DC01`.
        ```cmd
        w32tm /resync /rediscover
        ```
    * **Verification (`StanlyPC`):**
        ```cmd
        w32tm /query /status
        ```
        * **Expected `Source`:** `DC01.mylabs.com`.
        * **Expected `Last Successful Sync Time`:** A very recent timestamp.
        * **Crucial:** Visually confirmed that `StanlyPC`'s taskbar time now perfectly matched `DC01`'s correct UK time.

**8.4. Outcome**
After ensuring `DC01` had the correct time and `StanlyPC` successfully synchronized from it, a full system restart of `StanlyPC` was performed. Upon restart, `mylabs\stanly.sunny` was able to log in directly to the workstation without encountering the "Remote Desktop Services" error, confirming that the time synchronization discrepancy was the fundamental cause of the authentication failures.
---

## 9. Future Lab Enhancements

This section lists potential next steps and advanced topics to explore within this lab environment, building upon the foundational Active Directory and File Services infrastructure. These enhancements aim to increase the lab's complexity, introduce new server roles, and deepen practical skills in various IT domains.

### 9.1. High Availability & Redundancy

1.  **Add a Second Domain Controller (DC02):** (https://github.com/Stanlee81/Hyper-V-Lab-Deploying-a-Second-Domain-Controller-DC02-/blob/main/README.md)
    * **Goal:** Provide fault tolerance for Active Directory, DNS, and DHCP services.
    * **Process:** Create a new Windows Server VM (`DC02`), join it to the domain, install AD DS role, and promote it as an additional domain controller. Configure DNS pointers on both `DC01` and `DC02` to reference each other.
2.  **Implement Distributed File System (DFS) Namespace and Replication (DFS-R):**
    * **Goal:** Provide a single, unified access path to shared folders and replicate data between multiple file servers for redundancy and availability.
    * **Process:** Create a new File Server VM (`FS02`), install DFS roles, configure a DFS Namespace (`\\mylabs.com\ITUsersData`), and set up DFS Replication between `FS01` and `FS02`.

### 9.2. Advanced User & Computer Management (More GPO Power)

1.  **Implement Roaming User Profiles:**
    * **Goal:** Allow a user's desktop, documents, and application settings to follow them to any computer they log into in the domain.
    * **Process:** Create a shared folder on a file server (`FS01` or `FS02`), configure appropriate share and NTFS permissions, and set the profile path in Active Directory User properties or via GPO.
2.  **Deploy Software via GPO:**
    * **Goal:** Centrally install applications (e.g., 7-Zip, Notepad++) to domain computers automatically.
    * **Process:** Use Group Policy Management Editor (Computer Configuration > Policies > Software Settings > Software installation) with an MSI package.
3.  **Configure Drive Mappings via GPO:**
    * **Goal:** Automatically map network drives (e.g., `S:` for `\\FS01\ITUsersData`) for users upon login.
    * **Process:** Configure in Group Policy Management Editor (User Configuration > Preferences > Drive Maps).
4.  **Implement Login Scripts (PowerShell/Batch):**
    * **Goal:** Run custom commands or small scripts every time a user logs in (e.g., creating a desktop shortcut, performing a specific task).
    * **Process:** Configure in Group Policy Management Editor (User Configuration > Policies > Windows Settings > Scripts (Logon/Logoff)).

### 9.3. Security Enhancements

1.  **Configure Windows Firewall via GPO:**
    * **Goal:** Centrally manage firewall rules for all domain computers, enforcing security policies.
    * **Process:** Configure in Group Policy Management Editor (Computer Configuration > Policies > Windows Settings > Security Settings > Windows Firewall with Advanced Security).
2.  **Fine-tune Password and Account Lockout Policies:**
    * **Goal:** Enhance security by enforcing strong password requirements and locking out accounts after failed login attempts.
    * **Process:** Configure in the `Default Domain Policy` (Computer Configuration > Policies > Windows Settings > Security Settings > Account Policies).
3.  **Enable Security Auditing:**
    * **Goal:** Track important security events (e.g., failed logins, file access attempts, object modifications) for monitoring and compliance.
    * **Process:** Configure in Group Policy Management Editor (Computer Configuration > Policies > Windows Settings > Security Settings > Advanced Audit Policy Configuration).

### 9.4. Add New Server Roles

1.  **Print Server:**
    * **Goal:** Centralize print management and simplify printer deployment to client machines.
    * **Process:** Install the "Print and Document Services" role on a server (e.g., `FS01` or a new `PrintSrv01`), add a printer, and deploy it via GPO.
2.  **Web Server (IIS):**
    * **Goal:** Host internal websites or simple web applications for intranet services.
    * **Process:** Create a new Windows Server VM (`WebSrv01`), install the "Web Server (IIS)" role, and set up a basic HTML page.
3.  **Exchange Mail Server:**
    * **Goal:** Deploy an on-premises email, calendaring, and collaboration solution integrated with Active Directory.
    * **Process:** Create a new, powerful Windows Server VM (`ExchangeSrv01`), install prerequisites, and then install the Exchange Server role. Requires careful DNS and certificate configuration.

### 9.5. Networking & Connectivity

1.  **Introduce a Router VM with NAT:** (https://github.com/Stanlee81/HyperV-Lab-NAT-DC2)
    * **Goal:** Simulate internet access for lab VMs without directly exposing them, and practice basic routing.
    * **Process:** Create a new VM (`RouterVM`) with two network adapters (one to `Internal_Lab_Switch`, one to an external virtual switch), and configure the "Remote Access" role with NAT.
2.  **Implement a Second Internal Network/VLAN:**
    * **Goal:** Simulate departmental network segregation and practice inter-VLAN routing.
    * **Process:** Create a new internal Hyper-V virtual switch (e.g., `Internal_Sales_Switch`), create new client VMs on it, and configure `RouterVM` with an interface on this new switch to route traffic between the segments.

### 9.6. Monitoring & Automation

1.  **Basic PowerShell Scripting for AD:**
    * **Goal:** Automate repetitive administrative tasks within Active Directory.
    * **Process:** Learn and practice cmdlets like `Get-ADUser`, `New-ADUser`, `Set-ADUser`, `Get-ADGroup`, `Add-ADGroupMember` to create/manage AD objects.
2.  **Event Log Analysis:**
    * **Goal:** Understand how to identify, analyze, and troubleshoot issues by reviewing server event logs.
    * **Process:** Regularly examine the "System," "Security," and "Application" event logs on `DC01`, `FS01`, and client PCs for errors, warnings, and security events.

### 9.7. Troubleshooting Scenarios

1.  **Break and Fix (with Checkpoints!):**
    * **Goal:** Develop problem-solving skills by intentionally introducing errors and then diagnosing and resolving them.
    * **Process:** Examples include stopping the DNS service on a DC, disabling a network adapter, removing a GPO link, or intentionally misconfiguring a setting. **Always create a Hyper-V Checkpoint before breaking anything!**























































