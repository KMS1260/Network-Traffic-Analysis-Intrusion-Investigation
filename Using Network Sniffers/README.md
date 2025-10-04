# Using a Playbook

## Prerequisites

- Admin rights on the lab VMs.
- Sysinternals tools available (Process Explorer/ProcDump) or pre-installed.
- PowerShell and basic CLI access.
- A folder `./images/` for screenshots.


In this blog we will work through a playbook in response to a security incident. 

As a security professional, we may need to step in and perform corrective actions and implement response strategies when automated security systems are offline. In this blog we will work through a playbook focused on resolving potentially malicious code running on an internal system.

## Understanding the environment

we will be working from a virtual machine named PC10, hosting Windows Server 2019, being used as a client. And we will use a virtual machine named KALI, hosting Kali Linux.

## Objectives
- The importance of using appropriate cryptographic solutions. 
- Analyze indicators of malicious activity. 
- Explaining various activities associated with vulnerability management. 
- Explaining security alerting and monitoring concepts and tools. 
- Explaining appropriate incident response activities. 
- using data sources to support an investigation. 
- Summary elements of effective security governance. 

#### Tools & Techniques

- **Process Explorer** — inspect parent/child tree, handles/DLLs.
- **PowerShell:** `Get-Process <name>`; `Stop-Process -Id <PID> -Force`.
- **CLI:** `taskkill /pid <PID> /f /t` (or `/im <name>.exe`).
- **Evidence:** `Get-FileHash "<path>" -Algorithm SHA256`.
able of Contents

[Setup](#setup)

1. [Playbook Step 1](#playbook-step-1) - <strong>Investigate High CPU usage</strong>
2. [Playbook Step 2](#playbook-step-2) - <strong>Terminate the offending process</strong>
3. [Playbook Step 3](#playbook-step-3) - <strong>Hash the suspicious file</strong>
4. [Playbook Step 4](#playbook-step-4) - <strong>online malware scan</strong>
5. [Playbook Step 5](#playbook-step-5) - <strong>Determine the owner of the suspicious file</strong>
6. [Playbook Step 6](#playbook-step-6) - <strong>Archive the Suspicious File</strong>
7. [Playbook Step 7](#playbook-step-7) - <strong>Copy the archive to a quarantine system</strong>
8. [Playbook Step 8](#playbook-step-8) - <strong>Remove the suspicious file from the victim</strong>
9. [Playbook Step 9](#playbook-step-9) - <strong>Craft a Report about the Response</strong>
10. [Report](https://github.com/KMS1260/Threat-Detection-Playbooks-SOC-Incident-Response/blob/projects/05-using-a-playbook/Incident-Response-Report.md)


---

<strong>SIEM, SOAR, and playbooks</strong>
 
The organization's SIEM solution has detected a significant increase in CPU consumption on the PC10 workstation. The level of CPU activity has been at or near 100%, which is abnormal for the PC10 system. Under normal circumstances, the SOAR solution would respond to and resolve the abnormal event automatically. However, the SOAR system is currently offline due to a recent reconfiguration and update failure. Therefore, as a security professional, we will be responding manually to this situation. 

Fortunately, a pre-crafted playbook will us through the manual response activities. An incident response (IR) consulting group wrote the organization's library of playbooks. The IR consulting group was given broad parameters for crafting the playbooks. This has resulted in playbooks with flexibility and support for a wide range of knowledge, skill, and experience levels for those needing to use them to respond to incidents. We will work through the playbook explicitly designed to deal with high CPU consumption by rogue processes. 

A playbook is a checklist of actions to perform to detect and respond to a specific type of incident. 

<details>
<summary><strong>There are several primary steps or phases in this playbook:</strong></summary>

Investigate the high CPU usage and determine the rogue process's name. 

Terminate the offending process. 

Hash the file associated with the rogue process. 

Perform an online malware analysis using the hash value of the suspicious file. 

Determine the owner of the suspicious file. 

Archive the suspicious file into a zip container along with a file of its hash value. 

Copy the zip archive of the suspicious file to a quarantine system. 

Remove the suspicious file from the affected system(s). 

Fill out an incident report and submit it to the SOC for review.
</details>

For each of these steps, there are several options to select from. The playbook steps offer CLI (command line interface) solutions, GUI (graphical user interface) choices, or even third-party utility methods. While most of the operations use native tools, some reference use of tools from third parties.  

The blog is designed so you can choose your own options for each step of the overall playbook procedure. You are welcome to go over the entire blog and make other choices, or you can work through the various choices of each playbook step before moving forward. However, there may be a need to reset the system or implement a work around to use an alternate playbook step choice. These will be defined for you at the end of each playbook step before the Check your work section. 

A playbook is a common example of responsive controls. These are controls that serve to direct corrective actions that need to be enacted after an incident has been confirmed. In a Security Operations Center (SOC), responsive controls might include several very well-defined actions to be taken by a security professional after identifying a specific issue.

## Setup

In this introductory exercise, we will log into PC10 and initiate the rogue process. 

This is a necessary step to simulate the persistent execution of a rogue process. 

Connect to the PC10 virtual machine. Sign in as Jaime 

![](./images/0.jpg)

Select **Type here to search** from the taskbar, type powershell, then select **Windows PowerShell** from the results. 

![](./images/1.png)

Enter: 
```powershell
C:\LABFILES\Playbook-Lab.ps1
```
![](./images/2.jpg)

There may be a brief presentation of an empty Windows PowerShell console while the process starts. 

If prompted about allowing an execution exception for the script, type Y, then press Enter. 

Close this Windows PowerShell console. 

**Note** If you fail to close the Windows PowerShell console, the rogue process will be a sub-process of a PowerShell process. 

The rogue process, which is the focus of this lab, should now be running. 

The rogue process will immediately begin to consume most of the CPU. This will cause the system to be sluggish. We should be able to complete the initial playbook steps. 

---

## Playbook Step 1

<strong>Investigate High CPU usage</strong>

The first step of the High-CPU IR Playbook is:

Investigate the high CPU usage and determine the rogue process's name. 

In this High-CPU IR Playbook step, we will determine which rogue process is consuming most of the CPU's resources. 

Make a selection of the method to use to accomplish this initial task. The method options are: 

- GUI - using the Windows Task Manager 
- CLI - using the CLI Command Prompt wmic utility 
- Third-party - using the Sysinternals GUI tool Process Manager 

You can review the offered methods using the pull-down list below before making a final selection to work through. 

Security Orchestration, Automation, and Response (SOAR) is a security solution whose purpose is to scan security and threat intelligence data collected from multiple sources within the enterprise and then analyze it using various techniques. A SOAR can also assist with provisioning tasks, such as creating and deleting user accounts, making shares available, or launching VMs from templates. The SOAR will use technologies such as cloud and SDN/SDV APIs, orchestration tools, and cyber threat intelligence (CTI) feeds to integrate the different systems it manages. It will also leverage technologies such as automated malware signature creation and user and entity behavior analytics (UEBA) to detect and identify threats. The automated actions performed by a SOAR are to be documented in runbooks. However, when the SOAR fails to operate properly, security personnel can use a playbook to perform manually the actions that the SOAR would have automated. 

<details>
<summary><strong>Choose a method</strong></summary>

<details>
 <summary>Use GUI Task Manager</summary>

Select **Type here to search** from the taskbar, type task, then select **Task Manager** from the results. 

The Task Manager window should be displayed in details view with a menu bar and several tabs.

![](./images/3.png)

If the Task Manager is not in the details view, select **More Details** to switch to the details view. 

Select the **CPU** column to sort the processes by their CPU consumption. 

If the CPU percentages at the top of the column are 0%, then select the **CPU** column header again to reverse the sort order. 

Determine the name of the process that is consuming most of the CPU. We will note down the name 

High CPU process name: HeavyLoad 

Leave the Task Manager open. We will use it in a later playbook step. 

We have completed this playbook step using the GUI Task Manager. 

 </details>

<details>
 <summary>Use CLI Command Prompt WMIC utility</summary>
 
Select **Type here to search** from the taskbar, enter cmd, right-click **Command Prompt** from the results, then select **Run as administrator**. 

Select **Yes** on the User Account Control window. 

Enter: 
```cmd
wmic path Win32_PerfFormattedData_PerfProc_Process get Name,PercentProcessorTime. 
```
![](./images/4.jpg)

The results will be presented in an unsorted list. Scroll back up to locate the process name with the highest PercentProcessorTime.

![](./images/5.png)

The processor time is based on this system's use of 2 (virtual) CPUs. So, the total CPU process time available is 200%. 

As usual we would note down the name of the process: HeavyLoad  

Leave the Command Prompt window open. 

We have completed this playbook step using the CLI Command Prompt wmic utility. 
 
 </details>

<details>
 <summary>Use third-party GUI Process Manager</summary>

Select **Type here to search** from the taskbar, type file, then select **File Explorer** from the results. 

![](./images/6.png)

The File Explorer window should be displayed. 

![](./images/7.png)

In the left pane, select **SYSINTERNALS**. 

![](./images/8.png)

The contents of the C:\SYSINTERNALS folder will be displayed in the right pane. 

In the search field at the top right of the File Explorer window, which currently displays Search SYSINTERNALS*, enter procexp. 

![](./images/9.jpg)

There should be three results. 

Double-click **procexp64** from the search results.

![](./images/10.jpg)

The Process Explorer utility from Sysinternals should be displayed. 

![](./images/11.jpg)

Select the **CPU** column heading to sort by the use percentage. 

![](./images/12.jpg)

Scroll to the top of the listed processes to determine the process name consuming a significant amount of CPU resources. 

![](./images/13.jpg)

If the top of the CPU column is empty or shows use of "<0.01", then select the CPU column header again to reverse the sort order. 

High CPU process name: HeavyLoad 

Leave Process Explorer open. You may use it in a later playbook step. 

Leave File Explorer open. 

We have completed this playbook step using the third-party GUI utility Process Explorer from Microsoft's Sysinternals. 

Sysinternals, at sysinternals.com, is a Microsoft website that offers technical resources and utilities to manage, diagnose, troubleshoot, and monitor a Microsoft Windows environment. You can experiment with the Sysinternals tools go directly to sysinternals.com to learn more and download the suite of nearly 75 tools onto your own system. 
 
 </details>

</details>


---


## Playbook Step 2

<strong>Terminate the offending process</strong>

The next step of the High-CPU IR Playbook is: 

Terminate the offending process. 

In this High-CPU IR Playbook step, we will terminate the rogue process named HeavyLoad. 

Make a selection of the method to use to accomplish this task. The method options are: 

- GUI - using the Windows Task Manager 
- CLI-CP - using taskkill from a Command Prompt 
- CLI-PS - using Stop-Process from a PowerShell console. 
- Third-party-CLI - using the Sysinternals CLI tool pskill. 
- Third-party-GUI - using the Sysinternals GUI tool Process Manager. 

Incident response playbooks are an invaluable tool for organizations to quickly and efficiently respond to security incidents. With an incident response playbook, organizations can define the steps they need to take to respond to a security incident, such as the specific roles, processes, and procedures that security staff must follow. Incident response playbooks can also guide communication with stakeholders and the public, as well as how to gather evidence and determine the incident's root cause. Oftentimes, the playbook is just that-a physical book a security professional uses in response to an incident. Using a physical book ensures its availability during a wide-scale incident. In a highly secure environment, it also ensures attackers do not digitally exfiltrate the IR capabilities.

<details>
 <summary><strong>Choose a method</strong></summary>

<details>
 <summary>Use GUI Task Manager</summary>

Return to the **Task Manager**, which may have been left open from a previous playbook activity. 
 
![](./images/14.jpg)

If the Task Manager is not open, select **Type here to search** from the taskbar, type task, then select **Task Manager** from the results. 

Right-click the **HeavyLoad** process, then select **End Task**.

![](./images/15.jpg)

The HeavyLoad process should no longer be visible in the list of processes in Task Manager. 

If there are multiple instances of the HeavyLoad process, then we will need to kill each one. 

Close Task Manager. 
We have completed this playbook step using the GUI Task Manager. 

</details>

<details>
 <summary>Use CLI Command Prompt tool taskkill</summary>

Return to the **Command Prompt window**, which may have been left open from a previous playbook activity. 

![](./images/16.png)

If the Command Prompt window is not open, select **Type here to search** from the taskbar, enter cmd, right-click **Command Prompt** from the results, then select **Run as administrator**. Then, select **Yes** on the User Account Control window. 

Enter: 
```cmd
tasklist /FI "IMAGENAME eq HeavyLoad.exe" 
```

This command will display information about the named process. 

![](./images/17.jpg)

Take note of the process ID: 

HeavyLoad PID: 736 

Enter: 
```cmd
taskkill /PID 736 /F
```
![](./images/18.jpg)

This command terminates the HeavyLoad process referenced by the PID of 736. 

Enter tasklist /FI "IMAGENAME eq HeavyLoad.exe" again to confirm the process is no longer active.
```cmd
tasklist /FI "IMAGENAME eq HeavyLoad.exe"  
```
![](./images/19.png)

This command should display the message: "INFO: No tasks are running that match the specified criteria.". 

Leave the Command Prompt window open. You may use it in a later playbook step. 

We have completed this playbook step using the CLI Command Prompt tool taskkill. 
 
</details>

<details>
 <summary>Use CLI PowerShell cmdlet Stop-Process</summary>

Select **Type here to search** from the taskbar, type powershell, right-click **Windows PowerShell** from the results, then select **Run as administrator**. 

![](./images/20.png)

Select **Yes** on the User Account Control window. 

![](./images/21.png)

The Windows PowerShell console should be displayed. 

Enter: 
```powershell
Get-Process HeavyLoad 
```
![](./images/22.jpg)

This command will display information about the named process. 

Take note of the process ID: 

HeavyLoad PID: 3556 

Enter 
```powershell
Stop-Process -Id 3556 –Force
```
![](./images/23.png)

This command terminates the HeavyLoad process referenced by the PID of 3556. 

Enter Get-Process HeavyLoad again to confirm the process is no longer active.
```powershell
Get-Process HeavyLoad
```
![](./images/24.jpg)

This command should display the message: "Get-Process : Cannot find a process with the name "HeavyLoad".". 

Terminating a process through Windows PowerShell with only the process name is also possible. In this scenario, the command would be Stop-Process -Name HeavyLoad -Force. 

Leave the Windows PowerShell console open. 

We have completed this playbook step using the CLI PowerShell cmdlet Stop-Process. 
 
</details>

<details>
 <summary>Use third-party CLI pskill utility</summary>

Return to the **Command Prompt** window, which may have been left open from a previous playbook activity.

![](./images/25.png)

If the **Command Prompt** window is not open, select **Type here to search** from the taskbar, enter cmd, right-click **Command Prompt** from the results, then select **Run as administrator**. Then, select **Yes** on the User Account Control window. 

Enter: 
```cmd
cd C:\sysinternals
```
![](./images/26.png)

Enter: 
```cmd
pslist HeavyLoad
```
![](./images/27.jpg)

This command will display information about the named process. 

Take note of the process ID: 

HeavyLoad PID: 4892 

Enter:
```cmd
pskill 4892
```
![](./images/28.png)

This command terminates the HeavyLoad process referenced by the PID of 4892. 

Enter pslist HeavyLoad again to confirm the process is no longer active.
```cmd
pslist HeavyLoad
```
![](./images/29.png)

This command should display the message: "process HeavyLoad was not found on PC10"". 

Leave the Command Prompt window open. You may use it in a later playbook step. 

We have completed this playbook step using the third-party CLI utility pskill from Microsoft's Sysinternals. 
 
</details>

<details>
 <summary>Use third-party GUI Process Explorer</summary>

Return to **Process Explorer**, which may have been left open from a previous playbook activity. 

Expand this hint if Process Explorer is not open. 

Select **Type here to search** from the taskbar, type file, then select **File Explorer** from the results. 

The File Explorer window should be displayed. 

In the left pane, select **SYSINTERNALS**. 

The contents of the C:\SYSINTERNALS folder will be displayed in the right pane. 

In the search field at the top right of the File Explorer window, where it currently displays Search SYSINTERNALS*, enter procexp. 

There should be three results. 

Double-click **procexp64** from the search results. 

The Process Explorer utility from Sysinternals should be displayed. 

Leave File Explorer open.  

Right-click the **HeavyLoad** process, then select **Kill Process**. 

![](./images/30.jpg)

Select **OK** on the confirmation window. 

![](./images/31.png)

The HeavyLoad process should no longer be visible in the list of processes in Process Explorer. 

![](./images/32.jpg)

Note: If there are multiple instances of the HeavyLoad process, then we would need to kill each one. 

Close Process Explorer. 

We have completed this playbook step using the third-party GUI utility Process Explorer from Microsoft's Sysinternals. 
 
</details>
 
 </details>

---

## Playbook Step 3

<strong>Hash the suspicious file</strong>

The next step of the High-CPU IR Playbook is: 

Hash the file associated with the rogue process. 

In this High-CPU IR Playbook step, we will locate the file associated with the rogue process named HeavyLoad and calculate a SHA256 hash of the suspicious file. 

Make a selection of the method to use to accomplish these tasks. The method options are: 

- CLI-CP - using CLI Command Prompt tool certutil 
- CLI-PS - using CLI PowerShell cmdlet Get-FileHash 
- Third-party - using the Sysinternals CLI tool sigcheck

The most effective incident response playbooks are tailored to an organization's specific security needs and provide detailed guidance on responding to various security incidents. For example, a playbook may contain detailed instructions on responding to a ransomware attack or a data breach. Additionally, the playbook should include guidance on taking the necessary steps to contain the incident, such as isolating affected systems and measures to ensure the incident is fully resolved. 

<details>
 <summary><strong>Choose a method</strong></summary>

<details>
 <summary>Use CLI Command Prompt tool certutil</summary>

Return to the **Command Prompt** window, which may have been left open from a previous playbook activity. 

If the Command Prompt window is not open, select **Type here to search** from the taskbar, enter cmd, right-click **Command Prompt** from the results, then select **Run as administrator**. Then, select Yes on the User Account Control window. 

Enter: 
```cmd
cd c:\ && dir /s HeavyLoad.exe. 
```
![](./images/33.jpg)

This command changes the working or current directory to the root of drive C:\, then searches all sub-folders for the file HeavyLoad.exe. 

There should be two results indicating that HeavyLoad.exe is located in both c:\Users and c:\Program Files\JAM Software\HeavyLoad\. For this case let’s assume the only result is in c:\Users. 

![](./images/34.png)

**Note**: This command may take up to 1 minute to complete as it is searching the entire drive for the file. You can terminate it using **CTRL+C**. 

Enter: 
```cmd
cd c:\Users
```
![](./images/35.png)

This command changes the working directory to c:\Users. 

Enter: 
```cmd
certutil -hashfile "c:\Users\HeavyLoad.exe" SHA256 > c:\Users\HeavyLoad-Hash.txt
```
![](./images/36.jpg)

This command will calculate the SHA256 hash of the suspicious executable and save it in an output file. 

Enter: 
```cmd
type HeavyLoad-Hash.txt
```
![](./images/37.png)

This command displays the contents of the output file, which is the SHA256 hash of the suspicious executable. 

Leave the Command Prompt window open. 
 
We have completed this playbook step using the CLI Command Prompt tool certutil. 
 
</details>

<details>
 <summary>Use CLI PowerShell cmdlet Get-FileHash</summary>

Return to the **Windows PowerShell** console, which may have been left open from a previous playbook activity. 

If the Windows PowerShell console is not open, select **Type here to search** from the taskbar, type powershell, right-click **Windows PowerShell** from the results, select **Run as administrator**, then, select Yes on the User Account Control window. 

The Windows PowerShell console should be displayed. 

Enter: 
```powershell
cd c:\; Get-ChildItem -Path "C:\" -Recurse -File -ErrorAction SilentlyContinue | Where-Object { $_.Name -eq "HeavyLoad.exe" }
```
![](./images/38.jpg)

This command changes the working or current directory to the root of drive C:\, then searches all sub-folders for the file HeavyLoad.exe.

There should be a result indicating that HeavyLoad.exe is located in both C:\Users and C:\Program Files\JAM Software\HeavyLoad\. For this case let’s assume the only result is located in c:\Users. 

![](./images/39.png)

This command may take up to 1 minute to complete as it is searching the entire drive for the file. You can terminate it using **CTRL+C**. 

Enter: 
```powershell
cd C:\Users
```
![](./images/40.png)

This command changes the working directory to c:\Users. 

Enter: 
```powershell
Get-FileHash -Path "c:\Users\HeavyLoad.exe" -Algorithm SHA256 > c:\Users\HeavyLoad-Hash.txt 
```
![](./images/41.png)

This command will calculate the SHA256 hash of the suspicious executable and save it in an output file. 

Enter: 
```powershell
type HeavyLoad-Hash.txt 
```
![](./images/42.jpg)

This command displays the contents of the output file, which is the SHA256 hash of the suspicious executable. 

Leave the Windows PowerShell console open. You may use it in a later playbook step. 

We have completed this playbook step using the CLI PowerShell cmdlet Get-FileHash. 
 
</details>

<details>
 <summary>Use third-party CLI tool sigcheck</summary>

Return to the **Command Prompt** window, which may have been left open from a previous playbook activity. 

If the Command Prompt window is not open, select **Type here to search** from the taskbar, enter cmd, right-click **Command Prompt** from the results, then select **Run as administrator**. Then, select Yes on the User Account Control window. 

Enter: 
```cmd
cd c:\ && dir /s HeavyLoad.exe
```
![](./images/43.png)

This command changes the working or current directory to the root of drive C:\, then searches all sub-folders for the file HeavyLoad.exe. 

This command may take up to 1 minute to complete as it is searching the entire drive for the file. You can terminate it using **CTRL+C**. 

Enter: 
```cmd
cd c:\Users
```
![](./images/44.png)

This command changes the working directory to c:\Users\SigCheck\ 

Enter 
```cmd
C:\Users\Sigcheck>sigcheck -h C:\Users\HeavyLoad.exe > C:\Users\HeavyLoad-Hash.txt 
```
![](./images/45.png)

This command will calculate the SHA256 hash of the suspicious executable and save it in an output file. 

**Note** there are few Steps to follow and to know before using sigcheck i recommend watching this Youtube video https://www.youtube.com/watch?v=PPaDb9HWq9U  

Enter: 
```cmd
type HeavyLoad-Hash.txt 
```
![](./images/46.png)

This command displays the contents of the output file, which is the SHA256 hash of the suspicious executable. 

This command may take up to 1 minute to complete as it is calculating several hashes simultaneously. 

Leave the Command Prompt window open.  
 
</details>

</details>

---

## Playbook Step 4

<strong>online malware scan</strong>

The next step of the High-CPU IR Playbook is: 

Perform an online malware analysis using the hash value of the suspicious file. 

In this High-CPU IR Playbook step, we will perform a malware evaluation of the hash of the suspicious file using an online malware scanning service. 

Make a selection of the method to use to accomplish this task. The method options are: 

- Hybrid Analysis 
- MetaDefender 
- VirusTotal 

You can review the offered methods using the pull-down list below before making a final selection to work through. 

<details>
 <summary><strong>Choose a method</strong></summary>

<details>
 <summary>Use Hybrid Analysis</summary>
 
Using Chrome or Firefox open a browser. 

In the browser’s address bar, Enter: https://www.hybrid-analysis.com/ 

![](./images/47.jpg)

The Hybrid Analysis website should be displayed. 

Select the **Report Search** tab. 

![](./images/48.png)

Type the following hash into the **IP, Domain, Hash…** field, then select **Search**. 
```text
705204f80158bfbf9216a19bc2cb25a392ff028ce028745ac16f543b6290ec81 
```
![](./images/49.jpg)

The results of the online malware search will be displayed. 

![](./images/50.png)

Close the tab in your local browser focused on Hybrid Analysis. 

Since the malware hash scan results are indeterminate, we do not know if the suspicious file is benign or is a new unknown malicious file. Therefore, we will continue forward with the next playbook step. 

To see actual results from this site, repeat the search using the hash of 82407eaf6437d6956f63e85b28c0ec6ca58d298a. This is the hash of ca_setup.exe, the installer for the password cracking and network attack tool Cain. Cain is considered by many to be a hacker tool and potentially malicious. 

![](./images/51.jpg)
 
</details>

<details>
 <summary>Use MetaDefender</summary>

Using Chrome or Firefox open a browser. 

In the browser’s address bar, Enter: https://metadefender.com/  

The MetaDefender website from OPSWAT should be displayed. 

Select the **Lookup** tab. 

![](./images/52.png)

Type the following hash into **Trust no** field, then select Process:
```text
705204f80158bfbf9216a19bc2cb25a392ff028ce028745ac16f543b6290ec81
```
![](./images/53.jpg)

The results of the online malware search will be displayed. 

Close the browser tab focused on MetaDefender. 

Since the malware hash scan results are indeterminate, we do not know if the suspicious file is benign or is a new unknown malicious file. Therefore, we will continue forward with the next playbook step. 

To see actual results from this site, repeat the search using the hash of 82407eaf6437d6956f63e85b28c0ec6ca58d298a. This is the hash of ca_setup.exe, the installer for the password cracking and network attack tool Cain. Cain is considered by many to be a hacker tool and potentially malicious. 

![](./images/54.jpg)
 
</details>

<details>
 <summary>Use VirusTotal</summary>

Using Chrome or Firefox open a browser. 

In the browser’s address bar, Enter: https://www.virustotal.com/ 

The VirusTotal website should be displayed. 

Select the **Search** tab. 

![](./images/55.png)

Type the following hash into the **URL, IP address, domain, or file hash** field, then press **Enter** on your keyboard. 
```text
705204f80158bfbf9216a19bc2cb25a392ff028ce028745ac16f543b6290ec81
```
![](./images/56.png)

The results of the online malware search will be displayed. 

![](./images/57.png)

Close the tab in your local browser focused on VirusTotal. 

We have completed this playbook step using the online malware analysis service of VirusTotal. 

Since the malware hash scan results are indeterminate, we do not know if the suspicious file is benign or is a new unknown malicious file. Therefore, we will continue forward with the next playbook step. 

To see actual results from this site, repeat the search using the hash of 82407eaf6437d6956f63e85b28c0ec6ca58d298a. This is the hash of ca_setup.exe, the installer for the password cracking and network attack tool Cain. Cain is considered by many to be a hacker tool and potentially malicious. 

![](./images/58.png)
 
</details>
 
</details>

---

## Playbook Step 5

<strong>Determine the owner of the suspicious file</strong>

The next step of the High-CPU IR Playbook is: 

Determine the owner of the suspicious file. 

In this High-CPU IR Playbook step, we will determine the owner of the suspicious file. 

Make a selection of the method to use to accomplish this task. The method options are: 

- GUI - using File Explorer 
- CLI-CP - using CLI Command Prompt dir command 
- CLI-PS - using PowerShell cmdlet Get-ACL 

You can review the offered methods using the pull-down list below before making a final selection to work through. 

<details>
 <summary><strong>Choose a method</strong></summary>

<details>
 <summary>Use GUI File Explorer</summary>

Return to **File Explorer**, which may have been left open from a previous playbook activity. 

Select **Type here to search** from the taskbar, type file, then select **File Explorer** from the results. 

The File Explorer window should be displayed. 

In the left pane, expand **This PC**, and then select **Local Disk (C:)**. 

The contents of the root of drive C: should be displayed in the right pane. 

In the right pane, double-click **Users**. 

The contents of c:\Users should be displayed in the right pane. 

Right-click HeavyLoad, then select **Properties**. 

![](./images/59.jpg)

By default, File Explorer does not display the file extensions for known file types. Therefore, instead of seeing HeavyLoad.exe, you only see HeavyLoad. 

Select the **Security** tab on the HeavyLoad Properties window. 

![](./images/60.png)

Select **Advanced** on the Security tab. 

![](./images/61.png)

Look over the information on the Advanced Security Settings for HeavyLoad window. 

![](./images/62.jpg)

Determine the file's owner and enter the name as spelled and capitalized. Note it  

HeavyLoad file owner: SYSTEM 

Select **OK** to close the Advanced Security Settings for HeavyLoad window. 

Select **OK** to close the HeavyLoad Properties window. 

Leave the File Explorer window open. You may use it in a later playbook step. 

We have completed this playbook step using File Explorer. 
 
</details>

<details>
 <summary>Use CLI Command Prompt dir command</summary>

Return to the **Command Prompt** window, which may have been left open from a previous playbook activity. 

If the Command Prompt window is not open, select **Type here to search** from the taskbar, enter cmd, right-click **Command Prompt** from the results, then select **Run as administrator**. Then, select **Yes** on the User Account Control window. 

Enter:
```cmd
dir /q c:\Users\HeavyLoad.exe. 
```
![](./images/63.jpg)

Look over the information presented. 

Determine the file's owner and enter the name as spelled and capitalized. 

HeavyLoad file owner: Guest 

Leave the Command Prompt window open. You may use it in a later playbook step. 

</details>

<details>
 <summary>Use CLI PowerShell cmdlet Get-Acl</summary>

Return to the **Windows PowerShell** console, which may have been left open from a previous playbook activity. 

If the Windows PowerShell console is not open, select **Type here to search** from the taskbar, type powershell, right-click **Windows PowerShell** from the results, select **Run as administrator**, then, select **Yes** on the User Account Control window. 

The Windows PowerShell console should be displayed. 

Enter 
```powershell
(Get-Acl -Path "C:\Users\HeavyLoad.exe").Owner 
```
![](./images/64.png)

Look over the information presented. 

Determine the file's owner and enter the name as spelled and capitalized. 

HeavyLoad file owner: Guest 

Leave the Windows PowerShell console open. 

We have completed this playbook step using the CLI PowerShell cmdlet Get-Acl. 
 
</details>
 
</details>

---

## Playbook Step 6

<strong>Archive the Suspicious File</strong>

The next step of the High-CPU IR Playbook is: 

Archive the suspicious file into a zip container along with a file and its hash value. 

In this High-CPU IR Playbook step, we will create an archive containing the suspicious file and the hash output file. 

Make a selection of the method to use to accomplish this task. The method options are: 

- GUI - using File Explorer 
- CLI-CP - using CLI Command Prompt tool tar 
- CLI-PS - using PowerShell cmdlet Compress-Archive 

You can review the offered methods using the pull-down list below before making a final selection to work through. 

<details>
 <summary><strong>Choose a method</strong></summary>

<details>
 <summary>Use GUI File Explorer</summary>

Return to **File Explorer**, which may have been left open from a previous playbook activity. 

Select **Type here to search** from the taskbar, type file, then select **File Explorer** from the results. 

The File Explorer window should be displayed. 

In the left pane, select **Local Disk (C:)**. 

The contents of the root of drive C: should be displayed in the right pane. 

In the right pane, double-click **Users**. 

The contents of c:\Users should be displayed in the right pane. 

Select HeavyLoad. 

By default, File Explorer does not display the file extensions for known file types. Therefore, instead of seeing HeavyLoad.exe, you only see HeavyLoad. 

Hold down **Shift** on your keyboard, then select **HeavyLoad-Hash**.

![](./images/65.jpg)

Right-click over HeavyLoad while the two files are selected/highlighted, then select **Send to >**, then select **Compressed (zipped) folder**. 

![](./images/66.png)

If you right-click over the HeavyLoad-Hash file, then the archive file will have a filename of HeavyLoad-Hash.zip instead of HeavyLoad.zip. 

Select **Yes** on the Compressed (zipped) Folders window, which asks to create the archive file on the desktop. 

![](./images/67.jpg)

File Explorer does not have permission as a process to create a new file in the c:\Users folder. It also cannot be launched as the administrator. 

In the left pane, select **Desktop**. 

![](./images/68.jpg)

The contents of the Desktop should be displayed in the right pane. You should see **HeavyLoad**. It should have an icon of a yellow file folder with a zipper and be labeled as Type of Compressed (zipped) Folder. 

Use the click-hold-drag-release method to move HeavyLoad to **Local Disk (C:)**. 

![](./images/69.jpg)

Select **Continue** on the Destination Folder Access Denied window. 

![](./images/70.png)

The HeavyLoad file should no longer be displayed in the Desktop folder. 

In the left pane, select **Local Disk (C:)**. 

You should see **HeavyLoad** in the right pane. 

![](./images/71.png)

Leave File Explorer open.  
 
We have completed this playbook step using File Explorer. 
 
</details>

<details>
 <summary>Use CLI Command Prompt tool tar</summary>

Return to the **Command Prompt** window, which may have been left open from a previous playbook activity. 

If the Command Prompt window is not open, select **Type here to search** from the taskbar, enter cmd, right-click **Command Prompt** from the results, then select **Run as administrator**. Then, select **Yes** on the User Account Control window. 

Enter: 
```cmd
tar -c -a -f "C:\HeavyLoad.zip" "C:\Users\HeavyLoad.exe" "C:\Users\HeavyLoad-Hash.txt" 
```
![](./images/72.jpg)

A message will be displayed of "tar: Removing leading drive letter from member names" 

Enter: 
```cmd
dir c:\
```
![](./images/73.jpg)

You should see HeavyLoad.zip in the results of this command. 

Leave the Command Prompt window open.  
 
We have completed this playbook step using the CLI Command Prompt tool tar. 
 
</details>

<details>
 <summary>Use CLI PowerShell cmdlet Compress-Archive</summary>

Return to the **Windows PowerShell** console, which may have been left open from a previous playbook activity. 

If the Windows PowerShell console is not open, select **Type here to search** from the taskbar, type powershell, right-click **Windows PowerShell** from the results, select **Run as administrator**, then, select **Yes** on the User Account Control window. 

The Windows PowerShell console should be displayed. 

Enter: 
```powershell
Compress-Archive –Path “C:Users\HeavyLoad.exe”, “C:\users\HeavyLoad-Hash.txt” -DestinationPath “C:\HeavyLoad.zp” 
```

Enter:
```powershell
dir c:\ 
```
![](./images/74.jpg)
 
 </details> 
</details>

---

## Playbook Step 7

<strong>Copy the archive to a quarantine system</strong>

The next step of the High-CPU IR Playbook is: 

Copy the zip archive of the suspicious file to a quarantine system. 

In this High-CPU IR Playbook step, we will move the archive of the suspicious file to a quarantine system. In this Step, the quarantine system will be the Kali VM. 

Select the method to use to accomplish this task. The method options are: 

- NC - using Netcat and PowerShell 
- WinSCP - using WinSCP 
- SAMBA - Using SAMBA 

You can review the offered methods using the pull-down list below before making a final selection to work through. 

<details>
 <summary><strong>Choose a method</strong></summary>

<details>
 <summary>Use Netcat and PowerShell</summary>

Connect/access the KALI VM and sign in as **root**.

![](./images/75.png)

Open a Terminal window by selecting the **Terminal Emulator** from the Kali Linux toolbar 

![](./images/76.png)

Enter: **mkdir quarantine** to create a directory. 
```bash
mkdir quarantine
```
![](./images/77.png)

Enter **cd quarantine** to change into the new folder. 
```bash
cd quarantine
```
![](./images/78.jpg)

Enter: 
```bash
nc -l -p 1234 > HeavyLoad.zip
```
![](./images/79.jpg)

This command uses Netcat to open a listening port to receive a connection from PC10. Once the connection is established, the data transferred from PC10 will be saved into a file named HeavyLoad.zip on Kali. 

Switch back to the PC10 virtual machine. and sign in as Jaime. 

Return to the **Windows PowerShell** console, which may have been left open from a previous playbook activity. 

If the Windows PowerShell console is not open, select **Type here to search** from the taskbar, type powershell, right-click **Windows PowerShell** from the results, select **Run as administrator**, then, select **Yes** on the User Account Control window. 

The Windows PowerShell console should be displayed. 

Enter the following code into the Windows PowerShell console: 
```powershell
$filePath = "C:\HeavyLoad.zip";  
$destination = "10.1.16.66";  
$port = 1234;  
$tcpConnection = New-Object System.Net.Sockets.TcpClient($destination, $port);  
$netStream = $tcpConnection.GetStream();  
$buffer = [System.IO.File]::ReadAllBytes($filePath);  
$netStream.Write($buffer, 0, $buffer.Length);  
$netStream.Close();  
$tcpConnection.Close();  
$?;
```
![](./images/80.png)

This code will send the contents of the C:\HeavyLoad.zip file to the listening service on Kali. 

The final line of the code "&?;" will be left at the Windows PowerShell prompt. Press **Enter** to submit this last line of code. It will present a result of True. 

This code uses PowerShell functions to duplicate the data transfer capability of Netcat. This mechanism can be used from a Windows PowerShell console without needing Netcat present on the local system. 

Switch back to the KALI VM and, if needed, sign in as **root**. 

Notice the Netcat command has completed, and you are returned to the # prompt. 

![](./images/81.jpg)

Enter **ls -l** to view the long list of the current directory. 
```bash
ls -l
```
![](./images/82.jpg)

You should see the HeavyLoad.zip file is now present on the Kali system. 

Enter **unzip -t HeavyLoad.zip** to test and verify the file transferred properly.
```bash
unzip -t HeavyLoad.zip
```
![](./images/83.png)

The result should indicate "No errors detected in compressed data of HeavyLoad.zip." 

Switch back to the PC10 virtual machine. and sign in as Jaime. 

This VM switch back is needed to keep each playbook step transition consistent. 

Leave the Windows PowerShell console open.  

We have completed this playbook step using PowerShell functions and Netcat. 
 
</details>

<details>
 <summary>Use GUI WinSCP</summary>

We should still be working from the PC10 virtual machine. 

Select **Type here to search** from the taskbar, enter winscp, then select **WinSCP** from the results. 

![](./images/84.png)

The WinSCP application should be displayed.

![](./images/85.jpg)

On the Login window, select the File protocol: pull-down list, then select **SCP**. 

![](./images/86.png)

Type 10.1.16.66 in the Host name: field. 

Leave the Port number: field set at **22**. 

Login as root  

Select **Login** 

![](./images/87.png)

Select **Yes** on the Warning window. 

![](./images/88.png)

This warning message indicates that this is the first time a connection has been attempted to this destination. 

Double-click the **quarantine** folder that is now present in the Name column. 

![](./images/89.jpg)

In the left pane, relative to the PC10 system, select the **Parent directory** icon in the toolbar three times. This icon is a yellow file folder with an arrow pointing left then up. To the right of this icon are two periods. 

![](./images/90.png)

The path statement in the left pane, just above the directory contents presentation, should read C:\. 

![](./images/91.jpg)

Select **HeavyLoad.zip** in the left pane, then select **Upload** from the toolbar. 

![](./images/92.png)

Select **OK** on the Upload window without making any changes. 

![](./images/93.png)

This initiates the file transfer to Kali. After a few seconds, HeavyLoad.zip should be listed on the right-pane. 

Close WinSCP. Select **Yes** to exit WinSCP without saving a workspace. 

![](./images/94.jpg)

Select the **KALI VM** and sign in as **root**. 

Open a Terminal window by selecting the **Terminal Emulator** from the Kali Linux toolbar 

Enter **cd /root/quarantine** to change into the folder. 
```bash
cd /root/quarantine
```
![](./images/95.png)

Enter **ls -l** to view the long list of the current directory. 
```bash
ls -l
```
![](./images/96.jpg)

You should see the HeavyLoad.zip file is now present on the Kali system. 

Enter **unzip -t HeavyLoad.zip** to test and verify the file transferred properly.
```bash
unzip -t HeavyLoad.zip
```
![](./images/97.jpg)

The result should indicate "No errors detected in compressed data of HeavyLoad.zip." 

Switch back to the PC10 virtual machine. and sign in as Jaime. 

This VM switch back is needed to keep each playbook step transition consistent. 

We have completed this playbook step using WinSCP. 

</details>

<details>
 <summary>Use SAMBA</summary>

Select the KALI VM and sign in as **root**. 

Open a Terminal window by selecting the **Terminal Emulator** from the Kali Linux toolbar 

Enter **mkdir quarantine** to create a directory.
```bash
mkdir quarantine
```
![](./images/98.png)

Enter **cd quarantine** to change into the new folder. 
```bash
cd quarantine
```
![](./images/99.png)

Enter **nano /etc/samba/smb.conf** to open the SAMBA configuration file in the nano editor.
```bash
nano /etc/samba/smb.conf
```
![](./images/100.png)

In a real-world situation, it is recommended to make a backup copy of the smb.conf file before altering it. A command such as cp /etc/samba/smb.conf /etc/samba/smb.conf.bak would accomplish that. 

Type **CLTR+/** on your keyboard to issue the Go To Line function, type **500**, then press **Enter** on your keyboard. 

![](./images/101.png)

This will move your cursor to the end of the file, which is less than 500 lines long. 

You should see the end of the smb.conf file, which is the final line of: 
; write list = root, @lpadmin. 

![](./images/102.jpg)

At the end of the smb.conf file, type in the following: 
```bash
[quarantine] 
  path = /root/quarantine 
  browsable = yes 
  read only = no 
  guest ok = no 
  valid users = root 
```
![](./images/103.jpg)

Double-check your typing before saving these configuration changes. 

Type **CTRL+X** on your keyboard to exit Nano. 

Enter **y** to save the modified buffer. 

![](./images/104.png)

Press **Enter** on your keyboard to accept the existing filename. 

![](./images/105.jpg)

Enter **smbpasswd -a root**, then enter the password at both password prompts. 
```bash
smbpasswd -a root
```
![](./images/106.jpg)

The result of this operation should be the statement "Added user root".

![](./images/107.jpg)

Enter **systemctl restart** smbd to restart the SAMBA service so the new configuration settings will be in effect.
```bash
systemctl restart smbd
```
![](./images/108.jpg)

Switch back to the PC10 virtual machine. If needed sign in as Jaime. 

Return to the **Command Prompt** window, which may have been left open from a previous playbook activity. 

If the Command Prompt window is not open, select **Type here to search** from the taskbar, enter cmd, right-click **Command Prompt** from the results, then select **Run as administrator**. Then, select **Yes** on the User Account Control window. 

Enter:
```cmd
net use z: \\10.1.16.66\quarantine /user:root
```
![](./images/109.png)

If prompted, enter the password. 

Enter cd C:\ to switch to the root directory. 
```cmd
cd C:\
```
![](./images/110.png)

Enter copy HeavyLoad.zip z:\HeavyLoad.zip. 
```cmd
copy HeavyLoad.zip z:\HeavyLoad.zip
```
![](./images/111.png)

This command copies the suspicious file archive to the quarantine system. The result should be "1 file(s) copied.". 

![](./images/112.png)

Ignore the overwrite it’s because this task has been done via two methods  

Enter net use z: /delete to remove the network drive mapping. 
```cmd
net use z: /delete
```
![](./images/113.png)

Select the KALI VM and sign in as **root**. 

Open a Terminal window by selecting the **Terminal Emulator** from the Kali Linux toolbar 

Enter** cd /root/quarantine** to change into the folder. 
```bash
cd /root/quarantine
```
![](./images/114.png)

You may already be in the /root/quarantine folder. 

Enter **ls -l** to view the long list of the current directory. 
```bash
ls -l
```
![](./images/115.png)

You should see the HeavyLoad.zip file is now present on the Kali system. 

Enter **unzip -t HeavyLoad.zip** to test and verify the file transferred properly. 
```bash
unzip -t HeavyLoad.zip
```
![](./images/116.png)

The result should indicate "No errors detected in compressed data of HeavyLoad.zip." 

Switch back to the PC10 virtual machine. If needed, sign in as Jaime. 

This VM switch back is needed to keep each playbook step transition consistent. 

Leave the Command Prompt window open.  

We have completed this playbook step using SAMBA. 
 
 </details>
 
</details>

---

## Playbook Step 8

<strong>Remove the suspicious file from the victim</strong>

The next step of the High-CPU IR Playbook is: 

Remove the suspicious file from the affected system(s). 

In this High-CPU IR Playbook step, we will remove the suspicious file and related files from the victim system. 

Make a selection of the method to use to accomplish this task. The method options are: 

- CLI-CP1 - using CLI Command Prompt del command 
- CLI-CP2 - using the Sysinternals CLI tool SDelete 
- GUI - using File Explorer and the Recycle Bin 

You can review the offered methods using the pull-down list below before making a final selection to work through. 

<details>
 <summary><strong>Choose a method</strong></summary>

<details>
 <summary>Use CLI Command Prompt del command</summary>

Return to the **Command Prompt** window, which may have been left open from a previous playbook activity. 

If the Command Prompt window is not open, select T**ype here to search** from the taskbar, enter cmd, right-click **Command Prompt** from the results, then select **Run as administrator**. Then, select **Yes** on the User Account Control window. 

Enter del c:\HeavyLoad.zip to delete the suspicious file archive. 
```cmd
del c:\HeavyLoad.zip
```
![](./images/117.png)

Enter dir to confirm the file was removed from the current directory.
```cmd
dir
```
![](./images/118.jpg)

Enter del c:\Users\HeavyLoad* to delete the suspicious file and the hash file. 
```cmd
del c:\Users\HeavyLoad*
```
![](./images/119.png)

Enter dir c:\Users to confirm the files were removed from the c:\Users directory.
```cmd
dir c:\Users
```
![](./images/120.jpg)

Using the native del command will delete the files, but it is not secure destruction of the files' contents. An undelete operation (using a third-party tool) can recover access to deleted files whose storage areas are not yet overwritten. Also, the CLI del command deletes files directly rather than sending them to the Recycle Bin. 

Close the Command Prompt window. 

We have completed this playbook step using the CLI Command Prompt del command. 
 
</details>

<details>
 <summary>Use third-party CLI SDelete utility</summary>

Return to the **Command Prompt** window, which may have been left open from a previous playbook activity. 

If the Command Prompt window is not open, select **Type here to search** from the taskbar, enter cmd, right-click **Command Prompt** from the results, then select **Run as administrator**. Then, select **Yes** on the User Account Control window. 

Enter:
```cmd
sdelete -p 3 HeavyLoad.exe 
```
![](./images/121.png)

This command performs a secure deletion of HeavyLoad.exe by overwriting the drive storage areas where the file was located with random data (3 times based on this command) and does the same to the directory entry. 
 
Enter dir to confirm the file was removed from the current directory.
```cmd
dir
```
![](./images/122.jpg)

Enter sdelete -p 3 c:\Users\HeavyLoad* to securely delete the suspicious file and the hash file. 
```cmd
sdelete -p 3 c:\Users\HeavyLoad*
```
![](./images/123.png)

Enter dir c:\Users to confirm the files were removed from the c:\Users directory. 
```cmd
dir c:\Users
```
![](./images/122.jpg)

Close the Command Prompt window. 

We have completed this playbook step using the third-party CLI utility sdelete from Microsoft's Sysinternals. 
 
</details>

<details>
 <summary>Use GUI w/ Recycle Bin</summary>

Return to **File Explorer**, which may have been left open from a previous playbook activity. 

Select **Type here to search** from the taskbar, type file, then select **File Explorer** from the results. 

The File Explorer window should be displayed. 

In the left pane, select **Local Disk (C:)**. 

![](./images/124.jpg)

The contents of the root of drive C: should be displayed in the right pane. 

Right-click **HeavyLoad**, then select **Delete** from the fly-open menu. 

![](./images/125.jpg)

Remember, File Explorer does not display the file extensions for known file types by default. Therefore, instead of seeing HeavyLoad.exe, you only see HeavyLoad. 

The HeavyLoad archive file should no longer be visible. 

Double-click **Users** to enter that directory. 

Right-click **HeavyLoad**, then select **Delete** from the fly-open menu.

![](./images/126.jpg)

The HeavyLoad executable file should no longer be visible. 

Right-click **HeavyLoad-Hash**, then select **Delete** from the fly-open menu. 

![](./images/127.png)

The HeavyLoad-Hash text file should no longer be visible. 

Close File Explorer. 

Left-click **Recycle bin** on the Desktop. 

Then select **Empty Recycle** Bin from the fly-open menu. 

![](./images/128.jpg)

Select **Yes** on the Delete Multiple Items query window. 

![](./images/129.jpg)

Notice the files are no longer displayed in the Recycle Bin directory. The files have been effectively deleted. 

![](./images/130.jpg)

Using File Explorer to delete files will send them to the Recycle Bin. From the Recycle Bin, they can be restored. Once the Recycle Bin is emptied, the files will be deleted, but it is not a secure destruction of the files' contents. An undelete operation (using a third-party tool) can recover access to deleted files whose storage areas are not yet overwritten. 

We have completed this playbook step using File Explorer and the Recycle Bin. 
 
</details>
 
</details>

---

## Playbook Step 9

<strong>Craft a Report about the Response</strong>

The next step of the High-CPU IR Playbook is: 

Fill out an incident report and submit it to the SOC for review. 

In this High-CPU IR Playbook step, we will be reviewing instructions about crafting a report of the operations taken to resolve this security incident. 

Now that you have completed the playbook's primary steps, you need to craft and file a report about the security incident response. Your report should include a summary of the High-CPU IR Playbook steps, along with the methods used and results obtained. 

<details>
 <summary><strong>The High-CPU IR Playbook steps are:</strong></summary>
 
- Investigate the high CPU usage and determine the rogue process's name. 
- Terminate the offending process. 
- Hash the file associated with the rogue process. 
- Perform an online malware analysis using the hash value of the suspicious file. 
- Determine the owner of the suspicious file. 
- Archive the suspicious file into a zip container along with a file and its hash value. 
- Copy the zip archive of the suspicious file to a quarantine system. 
- Remove the suspicious file from the affected system(s). 
- Fill out an incident report and submit it to the SOC for review.
  
</details>

This type of report is often known as an AAR (After Action Report). It can also be referred to as a Lessons Learned or Post-Mortem report. The goal or purpose of this report is to document the activities performed, note any discrepancies or problems encountered, and glean information about where the process, playbook, toolset, or environment may need to be changed or improved. 

Once the report is crafted, it should be submitted to your CISO for review.

[Report](./Incident-Response-Report.md)

---

## Quick Quiz — Playbook

<details>
  <summary><strong>1) Which of the following playbook tasks must occur before the others?</strong></summary>

<details><summary>Copy the zip archive of the suspicious file to a quarantine system.</summary>❌ Incorrect</details>
<details><summary>Determine the rogue process's name.</summary>✅ Correct</details>
<details><summary>Perform an online malware analysis using the hash value of the suspicious file.</summary>❌ Incorrect</details>
<details><summary>Remove the suspicious file from the affected system(s).</summary>❌ Incorrect</details>
</details>

---

<details>
  <summary><strong>2) What asset defines the steps needed to respond to a security incident, such as the specific roles, processes, and procedures that security staff must follow?</strong></summary>

<details><summary>User Acceptance Policy</summary>❌ Incorrect</details>
<details><summary>Non-Disclosure Agreement</summary>❌ Incorrect</details>
<details><summary>Playbook</summary>✅ Correct</details>
<details><summary>Runbook</summary>❌ Incorrect</details>
</details>

---

<details>
  <summary><strong>3) A playbook is often considered what type of security control?</strong></summary>

<details><summary>Deterrent</summary>❌ Incorrect</details>
<details><summary>Preventive</summary>❌ Incorrect</details>
<details><summary>Responsive</summary>✅ Correct</details>
<details><summary>Detective</summary>❌ Incorrect</details>
</details>

---

<details>
  <summary><strong>4) What security solution scans security and threat intelligence data collected from multiple sources within the enterprise, then analyzes it using various techniques, and may trigger responses to take corrective action against discovered security violations?</strong></summary>

<details><summary>SOAR</summary>✅ Correct</details>
<details><summary>SIEM</summary>❌ Incorrect</details>
<details><summary>UEBA</summary>❌ Incorrect</details>
<details><summary>NGFW</summary>❌ Incorrect</details>
</details>

---

<details>
  <summary><strong>5) Which of the following are true statements in regard to playbooks? (Select all that apply)</strong></summary>

<details><summary>Generally, a playbook is used by a person, and a runbook is used by a SOAR platform in response to an incident or alert.</summary>✅ Correct</details>
<details><summary>When creating an incident response playbook, organizations should ensure they have the right level of detail and that all necessary stakeholders are involved.</summary>✅ Correct</details>
<details><summary>The most effective incident response playbooks are tailored to an organization's specific security needs.</summary>✅ Correct</details>
<details><summary>A playbook is designed to automate some of the routine tasks ordinarily performed by security personnel in response to a security incident.</summary>❌ Incorrect</details>
</details>

