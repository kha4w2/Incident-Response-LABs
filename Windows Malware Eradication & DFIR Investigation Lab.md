<h1 align="center">🛡️ Windows Malware Eradication & DFIR Investigation Lab</h1>

<h1 align="center"><img width="936" height="792" alt="image" src="https://github.com/user-attachments/assets/c765c4fe-c8c5-445c-bc11-441fab088480" /></h1>


---

## 🔎 Introduction

This repository documents a **hands-on Digital Forensics & Incident Response (DFIR) lab** focused on analyzing, containing, and eradicating a malicious Windows executable.
The lab simulates a realistic endpoint compromise scenario where a user executes an unknown binary that exhibits disruptive behavior, establishes persistence, and performs data theft activities.

The investigation emphasizes **native Windows telemetry**, leveraging built-in auditing mechanisms and security event logs to reconstruct attacker behavior and validate eradication efforts without relying on third-party EDR solutions.

---

## 🎯 Lab Objective

The primary objectives of this lab are to:

* Prepare a Windows system for forensic visibility using advanced audit policies.
* Observe and contain malicious executable behavior in a controlled environment.
* Identify and remove user-level persistence mechanisms.
* Reconstruct post-execution activity using Windows Security Event Logs.
* Confirm data access, staging, and packaging behavior.
* Fully eradicate the malware and validate system recovery.

This lab demonstrates an **end-to-end DFIR workflow**, from pre-incident preparation to post-eradication validation.

---

## 🧪 Scenario Overview

An analyst is provided with a suspicious executable suspected of malicious behavior.
Upon execution, the binary causes noticeable system disruption, launches multiple processes, and persists across system reboots without user interaction.

The executable is believed to:

* Achieve persistence using a registry-based mechanism.
* Access sensitive user data from Google Chrome.
* Stage and compress stolen data locally.
* Continuously re-execute to maintain presence and disrupt the system.

The analyst’s task is to **investigate, contain, analyze, and fully eradicate** the threat using Windows-native tools and event logs, ensuring no residual malicious artifacts remain on the system.

---




# Phase 1: Pre Task Preparation – Audit Policy Configuration

Before executing the lab task and analyzing any malicious activity, the system audit policies were configured to ensure full visibility into process and file system actions performed by the executable.

## Objective

Enable detailed Windows auditing to accurately capture:

* Process creation and termination events.
* File system access, creation, modification, and deletion events.

These logs are critical for tracing the executable’s behavior and supporting the investigation and eradication process.

---

## Step 1: Open Local Group Policy Editor

The Local Group Policy Editor was accessed using:


<img width="975" height="574" alt="image" src="https://github.com/user-attachments/assets/15e0c9b7-8818-4db0-bb85-78b3a6730abb" />


```
gpedit.msc
```

Navigation path:

Computer Configuration
→ Windows Settings
→ Security Settings
→ Advanced Audit Policy Configuration
→ System Audit Policies – Local Group Policy Object


<img width="975" height="737" alt="image" src="https://github.com/user-attachments/assets/727ca7b4-1e5d-4447-940d-b88cf28cbbdc" />


---

## Step 2: Configure Detailed Tracking Auditing

Under Detailed Tracking, the following subcategories were configured:

* Audit Process Creation → Success and Failure
* Audit Process Termination → Success and Failure


<img width="975" height="570" alt="image" src="https://github.com/user-attachments/assets/c1634e0c-2c62-45d3-9128-4ee3635d2321" />


This ensures that all processes launched or terminated by the executable are logged for analysis.

---

## Step 3: Configure Object Access – File System Auditing

Navigation path:

Object Access
→ Audit File System

Configuration applied:

* Audit File System → Success and Failure


<img width="975" height="740" alt="image" src="https://github.com/user-attachments/assets/54167c39-946d-43e9-aaa2-323e733a6a11" />


This enables logging of file level interactions such as file creation, modification, and deletion.

---

## Step 4: Enable Global Object Access Auditing

To capture file system activity system wide, Global Object Access Auditing was configured.

Navigation path:

Global Object Access Auditing
→ File System


<img width="975" height="713" alt="image" src="https://github.com/user-attachments/assets/ac190cab-5218-4292-92a8-7f35fb09ca12" />


---

## Step 5: Configure Global File SACL

The Global File SACL was configured to audit file system access by all users.

Actions performed:

* Opened File System Properties
* Selected Configure
* Added a new auditing entry
* Set Principal to:
* Everyone
* Enabled Success auditing
* Granted Full Control permissions (all access types)

  <img width="975" height="728" alt="image" src="https://github.com/user-attachments/assets/e3c593c7-190d-4836-83d7-7d4e3735e82d" />


<img width="975" height="552" alt="image" src="https://github.com/user-attachments/assets/21552cf8-407f-4bd3-9080-69c614c21b15" />


<img width="975" height="652" alt="image" src="https://github.com/user-attachments/assets/6b3167cb-ea6c-43a7-8a18-bdab61874c1b" />


<img width="975" height="603" alt="image" src="https://github.com/user-attachments/assets/7eb58b0c-e790-42b9-9d58-30e7ab0caba0" />


<img width="975" height="572" alt="image" src="https://github.com/user-attachments/assets/5007b998-ba80-44a0-8b56-3c8af3f5f4be" />


This ensures comprehensive logging of all file system activities across the system.

---

## Result

At this stage, the audit environment was fully prepared. The system is now capable of recording:

* All process execution activities.
* All file system interactions performed by the malicious executable.

These configurations provide the necessary telemetry to accurately investigate, trace, and reverse the actions performed by the executable in the next phases of the lab.

---

# Phase 2: Malware Execution, Persistence Identification, and Eradication

After preparing the audit environment, the malicious executable was executed to observe its behavior, identify its persistence mechanism, and perform eradication.

---

## Step 1: Execute the Malicious File

The provided executable was manually executed. Immediately after execution, multiple abnormal behaviors were observed, including:

* Repeated pop-up windows displaying messages such as "RUN please RUN".
* Multiple application windows opening simultaneously (e.g., browsers, FileZilla, Wireshark).
* Noticeable system resource consumption and user disruption.


<img width="975" height="649" alt="image" src="https://github.com/user-attachments/assets/7043eee2-fe38-4a6b-a511-6fb851ef86c7" />


This confirmed that the executable was actively running and intentionally designed to be disruptive.

---

## Step 2: Initial Containment – Process Termination

To contain the activity, Task Manager was opened and the malicious process was identified.

Observed process:

* eradication_lab.exe / runme.exe (multiple instances)

Action taken:

* The process was manually terminated using End Task.

  <img width="975" height="744" alt="image" src="https://github.com/user-attachments/assets/0580b948-8e7d-4219-956f-ce1dc08319cb" />


The process was successfully killed, confirming user-level control over the running instance.

---

## Step 3: Persistence Verification via System Restart

After terminating the process, the system was restarted to determine whether the malware would:

* Remain inactive, or
* Automatically re-execute without user interaction.

Result:

* The malicious process restarted automatically after boot, confirming the presence of a persistence mechanism.


  <img width="975" height="732" alt="image" src="https://github.com/user-attachments/assets/e5543a1d-112a-458b-b8ce-f68f2137fa15" />


This behavior strongly indicated registry-based persistence.

---

## Step 4: Persistence Investigation Strategy

Before immediately deleting the file, several eradication approaches were considered, including:

* Blocking execution using AppLocker.
* Hash-based execution control.
* File-based containment rather than deletion.

However, since the executable continued to run after deletion and reboot, the focus shifted to identifying the persistence artifact.

---

## Step 5: Registry Persistence Analysis (Run Keys)

The investigation focused on Registry Run Keys, a common persistence technique.

Using RegSeek and the native Registry Editor, the following registry locations were reviewed:

```
HKLM\Software\Microsoft\Windows\CurrentVersion\Run
```
<img width="975" height="520" alt="image" src="https://github.com/user-attachments/assets/247106a0-77e1-40f4-8156-bfc81569690f" />

```
HKLM\Software\Microsoft\Windows\CurrentVersion\RunOnce

```
<img width="975" height="512" alt="image" src="https://github.com/user-attachments/assets/4e6b2334-a05e-4b6b-96f7-126512a7db90" />

```
HKCU\Software\Microsoft\Windows\CurrentVersion\Run
```

<img width="975" height="515" alt="image" src="https://github.com/user-attachments/assets/de09ad98-8f1c-4afa-b9bc-bd38327ebb86" />


Initial analysis of HKLM Run and RunOnce keys showed only legitimate entries.

---

## Step 6: Identification of Malicious Registry Key

The investigation then shifted to the current user context:

```
HKCU\Software\Microsoft\Windows\CurrentVersion\Run
```

A suspicious registry value was identified:

* Value Name: RunMeBro
* Value Type: REG_SZ
* Value Data:
* C:\Users\Khaled\AppData\Roaming\runme.exe


<img width="975" height="728" alt="image" src="https://github.com/user-attachments/assets/d263c915-44a6-47d3-acbd-0cd50c04c0b2" />

This entry directly matched the executable that was repeatedly launching after system startup.

---

## Step 7: Correlation with Runtime Evidence

The registry value was correlated with:

* The process name observed in Task Manager.
* Automatic execution behavior after reboot.


<img width="975" height="548" alt="image" src="https://github.com/user-attachments/assets/057c2764-e529-45ee-87d9-11d7b4ea3ef9" />


This confirmed that HKCU Run key persistence was the mechanism used by the malware.

---

## Step 8: Eradication of Persistence Mechanism

Action taken:

* The RunMeBro registry value was deleted from:
* HKCU\Software\Microsoft\Windows\CurrentVersion\Run


<img width="975" height="734" alt="image" src="https://github.com/user-attachments/assets/56d39a63-e6a5-4758-9850-3269f2b2a60e" />

---

## Step 9: Validation

After removing the registry key, the system was restarted.

<img width="975" height="435" alt="image" src="https://github.com/user-attachments/assets/810429f8-a3bf-4082-bb03-c356f94a5575" />

Result:

* No malicious processes executed on startup.
* The system returned to normal operational state.

<img width="975" height="560" alt="image" src="https://github.com/user-attachments/assets/f5db2946-788c-48a0-8ed7-ee8e459fb076" />


---

## Outcome

At this stage:

* The malware persistence mechanism was successfully identified.
* Registry-based persistence was fully eradicated.
* The malicious executable no longer executed automatically after reboot.

This concludes the eradication phase and prepares the environment for further forensic analysis to determine the full impact of the malware on the system.

---

# Phase 3 – Event Log Investigation & Post-Execution Analysis

## Objective

The goal of this phase was to identify and reconstruct the malicious activities performed by the executable after execution, specifically:

* Process execution behavior
* File system interaction
* Evidence of Chrome data access and data staging
* Identification of directories and files created by the malware

This phase relies heavily on Windows Security Event Logs to build a reliable attack timeline and confirm malware intent.

---

## 1. Audit Policy Validation (Pre-Investigation)

Before analyzing event logs, audit policies were verified to ensure full visibility of process and file system activity.

### 1.1 Process Creation & Termination Auditing

```
auditpol /get /subcategory:"Process Creation"
auditpol /set /subcategory:"Process Termination" /success:enable /failure:enable
auditpol /get /category:"Detailed Tracking"
```

Result:

* Process Creation: Success & Failure
* Process Termination: Success & Failure

<img width="635" height="253" alt="image" src="https://github.com/user-attachments/assets/85308c82-eec7-43cf-b962-7115a18985c5" />


This guarantees visibility for Event ID 4688 (process creation) and termination-related artifacts.

---

### 1.2 File System (Object Access) Auditing

```
auditpol /get /subcategory:"File System"
```

Result:

* Object Access (File System): Success & Failure

<img width="635" height="134" alt="image" src="https://github.com/user-attachments/assets/d35708ec-8b75-4389-ad57-0382f655fc8a" />


This confirms the system is capable of logging Event ID 4663, which is critical for tracking file read/write/delete operations.

---

## 2. Event Log Scope Definition

The investigation focused on:

* Log Source: Windows Security Log

<img width="975" height="529" alt="image" src="https://github.com/user-attachments/assets/3d7f6b11-e648-4989-b3e8-5a6a19d22c20" />

* Primary Event IDs:

  * 4688 – Process Creation
<img width="975" height="462" alt="image" src="https://github.com/user-attachments/assets/9f446073-00e7-4c4e-9e18-f696be5bd396" />

  * 4663 – File System Object Access

<img width="975" height="471" alt="image" src="https://github.com/user-attachments/assets/3e618373-99f9-4283-9b29-93c0f0ba342f" />


---

## 3. Process Creation Analysis (Event ID 4688)

### 3.1 Initial Malware Execution

<img width="975" height="548" alt="image" src="https://github.com/user-attachments/assets/28b95025-cf59-4b9e-b9bb-cd3e7cde0d04" />


By filtering for Event ID 4688 and searching for the unique executable name:

```
eradication_lab.exe
```

The following was confirmed:

* Executable Path:

  * C:\Users\Khaled\Desktop\eradication_lab.exe
* Execution Method:

  * Manual execution via double-click
* Parent Process:

  * C:\Windows\explorer.exe
* Execution Context:

  * Standard user privileges
  * No administrative elevation

This confirms the executable acted as the Initial Dropper / Loader and represents the start of the attack timeline.

---

### 3.2 Persistence Confirmation via Execution Context

<img width="975" height="548" alt="image" src="https://github.com/user-attachments/assets/2e5c59be-2b71-4f2d-b845-1ee2070f0e6d" />


The parent process being explorer.exe and execution under a standard user account strongly supports the earlier finding that persistence was achieved via:

```
HKCU\Software\Microsoft\Windows\CurrentVersion\Run
```

This validates that persistence was user-based, not system-wide.

---

### 3.3 Self-Spawning Behavior (Forking)

Multiple Event ID 4688 entries showed the executable spawning child instances of itself.

Key Indicators:

* PPID (Parent Process ID): 0x2980
* Child processes were created by the original eradication_lab.exe instance
* No evidence of:

  * Process injection
  * LOLBins abuse
  * Privilege escalation

Conclusion:

The malware exhibited self-spawning behavior, creating additional instances from the original process. This increased runtime persistence and disruption without escalating privileges.

---

## 4. File System Activity Analysis (Event ID 4663)

<img width="975" height="548" alt="image" src="https://github.com/user-attachments/assets/d080f089-6260-4b72-bd69-318db2b66fa8" />




### 4.1 Identification of Relevant Object Access Events


<img width="975" height="548" alt="image" src="https://github.com/user-attachments/assets/5a16b2fa-c792-4a13-85b0-d798d0a08b22" />


<img width="975" height="548" alt="image" src="https://github.com/user-attachments/assets/e76914c1-3400-48b1-b470-191872246c1c" />

<img width="975" height="548" alt="image" src="https://github.com/user-attachments/assets/5930b4e7-31c5-4319-ae71-3548e175c6cf" />



Event ID 4663 logs were used to track file system interaction, including:

* ReadData
* ListDirectory
* Delete

Filtering was narrowed using the unique Process ID (PPID = 0x2980) to avoid false positives.

---

### 4.2 Chrome Data Access Confirmation


<img width="975" height="548" alt="image" src="https://github.com/user-attachments/assets/bb9040f0-47e3-4764-abca-bd6c6eeeac9a" />


Multiple 4663 events confirmed access to Chrome user data:

```
C:\Users\Khaled\AppData\Local\Google\Chrome\User Data\Default\History
```

Observed Access Type:

* ReadData (or ListDirectory)

This confirms the executable successfully accessed Chrome browsing history, validating the lab’s claim of Chrome data theft.

---

### 4.3 Data Staging Directory Discovery



<img width="972" height="547" alt="image" src="https://github.com/user-attachments/assets/fc41dc4b-5c3f-44f3-afe2-0c83f9784a7c" />


Further searches using keywords such as History led to discovery of malware-created directories:

```
C:\Temp\LAB_HISTORY_COPY
```

This directory contained staged copies of sensitive Chrome history data.

---

### 4.4 Data Packaging (Compression)


<img width="1920" height="1080" alt="Screenshot 2025-12-15 033725" src="https://github.com/user-attachments/assets/feaceaf7-aaea-4a50-93eb-4d263596c6fe" />


<img width="1906" height="947" alt="1111" src="https://github.com/user-attachments/assets/6fd92e28-444d-474e-ab11-4909b9bd6e48" />


Inside C:\Temp, the following archive was discovered:

```
haha_you_noob.zip
```

Archive Contents:

* LAB_HISTORY_COPY
* Extracted Chrome History database

This confirms:

* Data collection
* Local staging
* Compression for potential exfiltration

---

## 5. Final Findings Summary

Malware Capabilities Confirmed

* Manual execution via user interaction
* User-level persistence via HKCU Run key
* Self-spawning process behavior
* Chrome browsing history access
* Data staging in temporary directories
* Compression of stolen data into ZIP archive

---

## 6. Eradication Status

At the conclusion of the investigation:

* Persistence registry key removed
* Malicious executable deleted
* Temporary staging directories identified and removed
* No residual malicious activity observed

---

## 7. Conclusion

This phase successfully reconstructed the post-execution behavior of the malware using native Windows auditing and event logs. The findings confirm the executable’s intent to steal browser data, stage it locally, and maintain persistence at the user level.

The investigation demonstrates a complete DFIR workflow, from execution tracing to data theft confirmation and eradication validation.
