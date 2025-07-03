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



## Key Lab Phases & Configurations

### 1. Domain Controller (DC01) Setup
* VM provisioning and Windows Server OS installation.
* Static IP configuration (`192.168.1.1`).
* Installation and promotion of AD DS (`mylabs.com`).
* DNS server configuration.
* DHCP server installation, authorization, and scope creation (`192.168.1.50 - 192.168.1.200`).
![DC01-IPConfig](https://github.com/user-attachments/assets/93cf8e4c-3ddf-4e50-962e-840db3eab7fd)
![DHCP IP Range](https://github.com/user-attachments/assets/bee34779-8f30-4a1d-9334-e2ea39cc08a8)
![DHCP Address Lease](https://github.com/user-attachments/assets/22a92e9c-14a4-422f-a5f1-f0c47581ad72)

### 2. File Server (FS01) Setup
* VM provisioning and Windows Server OS installation.
* Joined to `mylabs.com` domain.
* Creation of `C:\ITUsersData` shared folder.
* Configuration of Share Permissions (Everyone: Full Control).
* Configuration of **NTFS Permissions** (Disabled inheritance, removed default Users/Authenticated Users, kept SYSTEM/Administrators, added `IT_Group` with specific permissions like "Create Folders / Append Data" to allow users to create their own redirected folders, and ensured `Creator Owner` had Full Control).
![FS01_IPconfig](https://github.com/user-attachments/assets/72c7d40b-48a9-473b-9916-8f9db2c64085)
![Share Folder- Sharing Permissions](https://github.com/user-attachments/assets/01313cad-ad7e-4f00-905c-d1c7a3689885)
![Share Folder- NTFS](https://github.com/user-attachments/assets/6b7a8c55-8f7b-46d1-b479-38be063ea8a3)


### 3. Active Directory Object Management
* Created `Contoso` Organizational Unit (OU) as the top-level departmental container.
* Created `IT_OU` nested under `Contoso`.
* Created `IT_Group` (Global Security Group) under `IT_OU`.
* Created user accounts (`Ben.son`, `Stanly.Sunny`) under `IT_OU` and added them to `IT_Group`.
  ![ADUC-OU Structure](https://github.com/user-attachments/assets/d790e39f-fea0-4d12-a5ad-2b457dd9d42d)
![ADUC - IT_Group Members](https://github.com/user-attachments/assets/5e43efa6-80c5-464a-a1fa-630ef69b5c4c)



### 4. Group Policy Object (GPO) for Folder Redirection
* Created `ITUserData_Redirection_Policy` GPO linked to the `Contoso` OU.
* Configured User Configuration -> Policies -> Windows Settings -> Folder Redirection for "Documents" and "Desktop."
* Set target to "Basic - Redirect everyone's folder to the same location" with root path `\\FS01\ITUsersData`.
* **Crucially, unchecked "Grant the user exclusive rights..."** to allow administrators access for troubleshooting/backup.
* GPO was enforced.
![Redirection_Policy](https://github.com/user-attachments/assets/78f01737-697b-43fc-b7f3-f876ca392cd7)
![GPO Editor- Desktop Redirection](https://github.com/user-attachments/assets/fa189a1a-adfc-4d27-9ebd-30ea0b185f08)
![GPO Editor- Documents Redirection](https://github.com/user-attachments/assets/af9d4b04-a356-4f58-9532-8bed2938f2a6)


### 5. Client PC (StanlyPC) Integration & Testing
* Provisioned a Windows 10/11 client VM (`StanlyPC`).
* Installed Windows OS, ensuring it received an IP and DNS from `DC01`'s DHCP.
* Successfully joined `StanlyPC` to `mylabs.com` domain.
* Logged in as a domain user (`Stanly.Sunny`).
* **Verified Folder Redirection:** Confirmed that "Documents" and "Desktop" folders were successfully redirected to `\\FS01\ITUsersData\<username>\...` by checking their "Location" properties in File Explorer.
![PC_IPconfig](https://github.com/user-attachments/assets/28e45382-bd46-44c1-8e05-da5424091e1d)
![System_Properties](https://github.com/user-attachments/assets/4ea39e7e-c9c3-4e05-ae71-a61bd031a472)
![Documents_Redirected](https://github.com/user-attachments/assets/dd296b19-62b3-4469-8352-37ba53a0814a)
![Desktop_Redirected](https://github.com/user-attachments/assets/c3b9e9c5-34cb-4112-a5ff-292c73d09f20)


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
