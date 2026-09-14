# Windows Server 2022 Enterprise Lab Infrastructure

## Overview
This repository documents the design, implementation, and verification of a segmented enterprise Active Directory environment built on **Windows Server 2022 (DC01)** with a **Windows 10 Pro Client**. The lab demonstrates role-based access control (RBAC), network services, domain hierarchy, and targeted Group Policy enforcement.

---

## Infrastructure Overview

| Parameter | Configuration Details |
| :--- | :--- |
| **Domain Name** | `labor.local` (NetBIOS: `LABOR`) |
| **Domain Controller** | `DC01` (`192.168.9.254`) |
| **DHCP Scope** | `192.168.9.100` – `192.168.9.200` / Subnet: `255.255.255.0` |
| **DNS Configuration** | `192.168.9.254` (AD-integrated) |
| **Virtualization** | VirtualBox Isolated Internal Network (`intnet`) |

---

## Active Directory Organizational Structure & Access Control

The Active Directory structure enforces strict separation between organizational roles using nested OUs, Security Groups, and NTFS/Share permissions.


labor.local
└── Teacher (OU)
    ├── GG_Teachers (Security Group)
    ├── tanar.bela (User Account)
    └── Student (OU)
        ├── GG_Students (Security Group)
        └── tanulo.elek (User Account)

Group Policy & Share Governance Architecture

Network storage is distributed dynamically using Group Policy Preferences (GPP) targeting specific OUs:
Role / Group	|  Target OU	    |  Mapped Drive	| Network Path       |  Scope & Permissions
GG_Students   | Teacher/Student |      S:       | \\DC01\DiakAdatok  | Student-only access via GPO_Map_StudentDrive
GG_Teachers	  |     Teacher	    |      T:       | \\DC01\TanarAdatok | Teacher-only access via GPO_Map_TeacherDrive

Key Implementations & Verification
1. Active Infrastructure Services
Active Directory Domain Services, DNS, and DHCP Scope (192.168.9.0/24) were successfully deployed and authorized on DC01.

2. Active Directory Hierarchy
OU structure (Teacher -> Student) and corresponding Security Groups were built to manage permissions centrally.

3. Group Policy & Drive Mapping

Group Policy Preferences (GPP) automatically deploy drives based on OU membership:

    tanulo.elek (Student OU): Automatically maps drive S: (\\DC01\DiakAdatok). Access to Teacher share denied.

    tanar.bela (Teacher OU): Automatically maps drive T: (\\DC01\TanarAdatok). Excluded from Student drive mapping.
### 4. Shared Resources & Role-Based Access Control (RBAC) Updates
To mirror a real-world educational infrastructure, a centralized, read-only shared resource was implemented alongside the user home folders:

* **Folder Path:** `\\dc01\Kozos` mapped automatically to the **`K:`** drive via Group Policy Preferences (GPP).
* **Security & Permission Architecture:**
  * **Share Permissions:** Configured to *Full Control* for `Everyone`, relying strictly on granular NTFS permissions for actual security enforcement.
  * **NTFS Permissions:**
    * **Teachers:** *Full Control* (Read, Write, Modify, Delete) to manage course materials and populate the repository.
    * **Students:** *Read & Execute* / *Read-Only* access, restricting unauthorized modifications or deletions.
* **Deployment & Verification:**
  * Deployed seamlessly across domain clients using **Group Policy Preferences (GPP) Drive Maps**.
  * Validated successfully: Student accounts can view and access shared content, but receive standard Windows `Access Denied` prompts when attempting write or delete operations, confirming correct privilege isolation.
