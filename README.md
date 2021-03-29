# Introduction #

As part of the steps for the DCCP project, end user devices (PCs / Laptops) must be trusted through an automated  “Hybrid Join” process. It has been identified that through the standard Hybrid Join process, a number of devices end up in an error state that can impact the end users connection and services from M365. 

The process to resolve the issues are through Detection and Remediation automated scripts that are used to identify devices that are mis-configured and then correct them. Devices that are mis-configured may not display the blocking message at this stage but may fail during other activities like password reset or Windows 10 patching.

Because of variances in department desktop configurations, the scripts from Microsoft have a “not for Production” disclaimer so each partner is required to complete a testing cycle for their environment.

The scripts and the processes to run as identified below in the “readme” document and all documentation and information is available at : https://github.com/Cary-Connolly/GC-Dual-State-Correction

Please send any administrative questions or results from the detection scripts to Jamie Maycock Jamie.maycock@canada.ca

If you have any technical questions or concerns please contact Cary Connolly cary.connolly@canada.ca or Josh Wittkie  jwittkie@microsoft.com

# Pre-Requisite Tasks #

One of the first steps to remediating any dual state issue is preventing devices from participating in AD Registration. This can be accomplished by applying the:

HKLM\SOFTWARE\Policies\Microsoft\Windows\WorkplaceJoin: "BlockAADWorkplaceJoin"=dword:00000001

Registry Key.

To perform the update, the device must be connected to the LAN by either being present in the office or remotely via VPN.
 
To enable automatic hybrid join apply the:

HKEY_LOCAL_MACHINE\Software\Policies\Microsoft\Windows\WorkplaceJoin: “autoWorkplaceJoin"=dword:00000001

And

HKEY_LOCAL_MACHINE\Software\Policies\Microsoft\Windows\WorkplaceJoin: “Recovery-Check"=dword:00000001

Registry Keys.
 
It should be noted that clients should be running version 1809 of Windows 10 or later to support the automated remediation of dual state issues.
 
This solution can be implemented manually on a local device but when possible, should be applied to all devices in an organization through GPO or the appropriate configuration management tool (SCCM, IBM BigFix, LANDesk, etc...).

# Automated Detection #

Following the implementation of the Pre-Requisite tasks the client should download the most up to date "AzureADHybrid-UserBaseline.cab" SCCM Baseline Configuration from the Git repository.

To start the Import Configuration Data Wizard, in the Configuration Items or Configuration Baselines node in the Assets and Compliance workspace, click “Import Configuration Data”.

Select the “AzureAD-HybridBaseline.cab” Configuration Baseline you downloaded in the previous step.

Use the Create Configuration Baseline dialog box to create a new configuration baseline targeting Device Baseline, Device Mitigation, Device Collection.

The Baseline will return a “Compliant” / green response for all Users in a clean state (only Azure AD Joined and Domain Joined), a “Non-Compliant” / blue response for all Users that are in a dual state (Azure AD Joined, Domain Joined, and Workspace Joined), and an “Error” / red response for those that encounter an error.

## Recorded Steps ##

A Problem Step Recorder (PSR) walk through of the deployment of the Automatic Detection component has been provided in the "Automated Detection Steps Recording" file stored in this repository.
 
 # Automated Remediation #
 
Following the implementation of the Automated Detection the client should download the provided SCCM "MitigateV3_files-1.1.zip" Application from the Git repository.
 
Client’s can manually target Users for remediation with the provided Application or they can create a dynamic SCCM Collection with the Application. A dynamic Collection will allow the client to target the Users who have returned a non-compliant status through the previous Baseline for the deployment of automatic remediation.

To create a dynamic Collection that populates based off the output of the previous Baseline deployment, select the previously created baseline, hover over “Create New Collection” and create a Collection for “Non-compliant” Users.

Following this you should be able to track the status of your deployment in SCCM.
 
“Success” indicates devices that have successfully executed the automatic remediation Application. “In Progress” represents those devices that have still yet to finish the process. Those showing as “Error” or otherwise must be manually investigated for remediation. A common cause of errors is the restriction of PowerShell execution policies on endpoint devices.

## Recorded Steps ##

A Problem Step Recorder (PSR) walk through of the deployment of the Automatic Remediation component has been provided in the "Automated Remediation Steps Recording" file stored in this repository.

## What Users Can Expect ##

When deployed to Users the automatic remediation application will execute a short series of tasks while connected to VPN. While most of these tasks will take place silently in the background, there is a requirement for a full restart of the User’s device. The User will be prompted before this restart takes place and will only execute when the User has clicked the appropriate “OK”. Following the restart, the client is expected to log back into VPN, and will have the device trigger a lock screen to ensure the client logs back into the desktop and is granted the appropriate Primary Refresh Token (PRT).
