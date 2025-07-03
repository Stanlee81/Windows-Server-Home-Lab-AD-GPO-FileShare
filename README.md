# Windows-Server-Home-Lab-AD-GPO-FileShare
Comprehensive home lab demonstrating Windows Server deployment, Active Directory, DHCP, DNS, file sharing, and Group Policy-based Folder Redirection."
# IT Infrastructure Home Lab: Active Directory, File Sharing & Folder Redirection

## Overview
This repository documents the setup and configuration of a foundational Windows Server home lab environment. The primary goal was to deploy core domain services, establish a centralized file server, and implement Group Policy-based Folder Redirection for user profiles.

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

## Lab Architecture Diagram (Optional but highly recommended)
*(You can upload a diagram image to your `diagrams/` folder and link it here, e.g., `![Lab Diagram](diagrams/lab_diagram.png)`)*

## Key Lab Phases & Configurations

### 1. Domain Controller (DC01) Setup
* VM provisioning and Windows Server OS installation.
* Static IP configuration (`192.168.1.1`).
* Installation and promotion of AD DS (`mylabs.com`).
* DNS server configuration.
* DHCP server installation, authorization, and scope creation (`192.168.1.50 - 192.168.1.200`).
* *[Include relevant screenshots from DC01 here]*

### 2. File Server (FS01) Setup
* VM provisioning and Windows Server OS installation.
* Joined to `mylabs.com` domain.
* Creation of `C:\ITUsersData` shared folder.
* Configuration of Share Permissions (Everyone: Full Control).
* Configuration of **NTFS Permissions** (Disabled inheritance, removed default Users/Authenticated Users, kept SYSTEM/Administrators, added `IT_Group` with specific permissions like "Create Folders / Append Data" to allow users to create their own redirected folders, and ensured `Creator Owner` had Full Control).
* *[Include relevant screenshots from FS01 here]*

### 3. Active Directory Object Management
* Created `Contoso` Organizational Unit (OU) as the top-level departmental container.
* Created `IT_OU` nested under `Contoso`.
* Created `IT_Group` (Global Security Group) under `IT_OU`.
* Created user accounts (`Ben.son`, `Stanly.Sunny`) under `IT_OU` and added them to `IT_Group`.
* *[Include relevant screenshots of ADUC structure]*

### 4. Group Policy Object (GPO) for Folder Redirection
* Created `ITUserData_Redirection_Policy` GPO linked to the `Contoso` OU.
* Configured User Configuration -> Policies -> Windows Settings -> Folder Redirection for "Documents" and "Desktop."
* Set target to "Basic - Redirect everyone's folder to the same location" with root path `\\FS01\ITUsersData`.
* **Crucially, unchecked "Grant the user exclusive rights..."** to allow administrators access for troubleshooting/backup.
* GPO was enforced.
* *[Include screenshots of GPMC and GPO Editor settings]*

### 5. Client PC (StanlyPC) Integration & Testing
* Provisioned a Windows 10/11 client VM (`StanlyPC`).
* Installed Windows OS, ensuring it received an IP and DNS from `DC01`'s DHCP.
* Successfully joined `StanlyPC` to `mylabs.com` domain.
* Logged in as a domain user (`Stanly.Sunny`).
* **Verified Folder Redirection:** Confirmed that "Documents" and "Desktop" folders were successfully redirected to `\\FS01\ITUsersData\<username>\...` by checking their "Location" properties in File Explorer.
* *[Include screenshots of client's redirected folder properties]*

## Key Learnings & Challenges
* Understanding the distinction and interaction between Share Permissions and NTFS Permissions.
* The critical role of `Creator Owner` NTFS permission for Folder Redirection to work correctly.
* Troubleshooting domain join issues (e.g., DNS resolution, incorrect login formats).
* The power of Group Policy for centralized management of user environments.
* The importance of proper GPO linking and enforcement.

## Future Enhancements
* Adding a second Domain Controller for redundancy.
* Implementing Distributed File System (DFS) for high availability of file shares.
* Exploring more GPO features (software deployment, drive mappings, security policies).
* Setting up a dedicated Print Server.
* Configuring secure remote access (VPN or DirectAccess).
