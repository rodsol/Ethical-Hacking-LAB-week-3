# Week 3 Lab: Ethical Hacking Techniques
**Course:** Ethical Hacking  
**Week:** 3  
**Topics:** Website Cloning for Credential Harvesting & SMB Vulnerability Scanning

## 📋 Lab Overview
This week's lab focuses on two practical attack vectors to understand common security vulnerabilities:
1.  **Website Cloning & Credential Harvesting:** Simulating a social engineering attack using the Social-Engineer Toolkit (SET).
2.  **SMB Vulnerability Scanning:** Enumerating a target system's SMB service to discover users, shares, and weak configurations.

### ⚠️ Critical Legal & Ethical Notice
**These techniques must only be performed in a controlled lab environment on systems you own or have explicit written permission to test.** Unauthorized use against systems you do not own is illegal and unethical. This lab is strictly for educational purposes to understand attack methods and improve defenses.

---

## 🧪 Lab 1: Website Cloning for Credential Harvesting

### Objective
To clone a legitimate website and capture submitted credentials, demonstrating a phishing attack vector.

### Tools Used
*   **SEToolkit (Social-Engineer Toolkit):** An open-source penetration testing framework for social engineering.

### Lab Steps & Commands

1.  **Launch SEToolkit:**
    ```bash
    sudo setoolkit
    ```

2.  **Navigate the Menus:**
    *   Select **1) Social-Engineering Attacks**.
    *   Select **2) Website Attack Vectors**.
    *   Select **3) Credential Harvester Attack Method**.
    *   Select **2) Site Cloner**.

3.  **Configure the Attack:**
    *   You will be prompted for the **IP address for post back in harvesting**. This is **the IP address of your attacking machine** (the Kali VM). The cloned site will send stolen data here.
    *   Next, enter the **URL to clone**. In this lab, we used: `http://dvwa.vm` (the Damn Vulnerable Web Application).

4.  **Execute and Observe:**
    *   SEToolkit will start a web server hosting the cloned site.
    *   If a victim visits your malicious site and enters credentials, they will be captured and displayed in the SEToolkit terminal.
    *   The cloned site appears identical to the original, making it a convincing phishing page.

### Key Concept
*   **Post-Back IP:** This is a critical configuration. It ensures any data entered into the fake form is sent to the attacker's machine for collection.

---

## 🖥️ Lab 2: SMB Vulnerability Scanning

### Objective
To scan a target subnet for SMB services, enumerate information from a vulnerable host, and assess its security posture.

### Tools Used
*   **Nmap:** Network discovery and security auditing.
*   **Enum4linux:** A tool for enumerating information from Windows and Samba systems.
*   **smbclient:** An SMB client to access shared resources.

### Lab Steps, Commands & Results

#### Step 1: Network Discovery with Nmap
**Command:**
```bash
nmap -sN 172.17.0.0/24
```
**Objective:** Perform a "NULL scan" to discover live hosts and identify open SMB ports (139, 445) on the `172.17.0.0/24` subnet.
**Result:**
*   Two hosts were found active: `172.17.0.1` and `172.17.0.2`.
*   On `172.17.0.2`, both SMB ports **139 (NetBIOS)** and **445 (SMB over TCP)** were open, indicating a potential target.

#### Step 2: User Enumeration with Enum4linux
**Command:**
```bash
enum4linux -U 172.17.0.2
```
**Objective:** Retrieve a list of user accounts configured on the target (`172.17.0.2`).
**Result:** A detailed list of system and service users was extracted, including `root`, `msfadmin`, `user`, and many others.

#### Step 3: Share Enumeration with Enum4linux
**Command:**
```bash
enum4linux -S 172.17.0.2
```
**Objective:** Discover shared folders and resources on the target.
**Result:** Several shares were found:
*   `print$` (Printer Drivers, access denied)
*   `tmp` (Comment: "oh noes!", listing OK)
*   `opt` (access denied)
*   `IPC$` & `ADMIN$` (administrative IPC shares)

#### Step 4: Password Policy Analysis with Enum4linux
**Command:**
```bash
enum4linux -P 172.17.0.2
```
**Objective:** Retrieve and analyze the password policy for the domain/workgroup.
**Result:** The policy was found to be **very weak**:
    *   Minimum password length: Only **5 characters**.
    *   Password history: **None**.
    *   Password complexity: **Disabled**.
    *   Account lockout threshold: **None**.
    *   This makes the system highly vulnerable to brute-force attacks.

#### Step 5: Interacting with Shares using smbclient
**Objective:** Attempt to connect to the discovered shares.
1.  **List Shares:**
    ```bash
    smbclient -L //172.17.0.2
    ```
    *Result: Anonymous (null session) login successful, confirming a critical misconfiguration.*

2.  **Connect to the `tmp` Share:**
    ```bash
    smbclient //172.17.0.2/tmp
    ```
    *Result: Successfully connected with anonymous credentials.*

3.  **Upload a File (Simulating Malware):**
    Once inside the `tmp` share, a file (`virus.exe`) was uploaded and renamed to appear harmless:
    ```smbclient
    put virus.exe group_work.txt
    ```
    *Result: File successfully copied to the target's shared folder, demonstrating how an attacker could implant malware.*

### Key Findings & Security Implications
1.  **Exposed SMB Service:** Open ports 139/445 to untrusted networks are a primary attack surface.
2.  **Weak Authentication:** The system allowed **anonymous NULL sessions**, enabling unauthenticated enumeration.
3.  **Poor Password Policy:** The weak policy drastically reduces the effort required for a successful password attack.
4.  **Insecure Share Permissions:** A writable share (`tmp`) was accessible anonymously, allowing for initial foothold and malware deployment.

---

## 📚 Summary & Lessons Learned
This lab provided hands-on experience with two prevalent attack types:
*   **Social Engineering (Phishing):** How easily websites can be cloned to steal credentials and the importance of user vigilance.
*   **Network Vulnerability Exploitation:** How misconfigured SMB services can lead to full system enumeration, policy discovery, and unauthorized access.

**Defensive Takeaway:** Always enforce strong password policies, disable SMBv1, restrict anonymous access, regularly audit share permissions, and educate users about phishing threats.

