# SOAR-EDR

## Overview
This project aims to integrate Security Orchestration, Automation, and Response (SOAR) with Endpoint Detection and Response (EDR) capabilities to streamline and automate incident response processes. The core of this project is built around leveraging LimaCharlie for endpoint detection and custom rule creation, coupled with Tines for orchestrating automated responses and user notifications.

## Components

### Playbook

<img width="817" alt="Screenshot 2024-09-05 at 23 23 08" src="https://github.com/user-attachments/assets/7d486743-6087-4023-8ecd-c398c8d7601b">

### LimaCharlie
LimaCharlie is the EDR tool which is used for detecting incidents based on custom rules intended to detect potential threats or suspicious activity at endpoint. LimaCharlie provides the necessary visibility into any endpoint connected to it, providing alerts once detection rules have been triggered. 

### Tines
Tines acts as the the automation and orchestration layer in this project. When a detection rule is triggered and an alert is created, it is forwarded to Tines. Tines then manages the response workflow, sending notifications to the appropriate communication channels (in this project it is email and or slack) and handling user interactions. 

### Machines
The machines monitored by LimaCharlie in this project are a Linux virtual machine (VMware) and a cloud Windows machine (Vultr). 

## Project Timeline

### Playbook
I first started the project by defining a playbook which clearly shows the workflow of the project, adding to it as I went along if needed or making changes if I later decided something didn't make sense. I would reference back to the playbook when implementing the SOAR aspect of the project. 

### Windows and Linux machines setup 
The Windows cloud machine was booted up using Vultr. Once running, the machine was clearly displayed on Vultr:

<img width="1442" alt="Vultr Windows machine" src="https://github.com/user-attachments/assets/a3159c6c-6a31-4010-824d-dd15cc2b29e4">

I made a firewall group with a single for the Windows machine to make it so that it only accepts MS RDP on port 3389 from my IP, and setup this firewall group for the Windows machine.

<img width="1424" alt="SOAR-EDR-Firewall-Windows" src="https://github.com/user-attachments/assets/d1118fa7-658f-4647-80d5-f7ba3d2e960e">
<img width="1328" alt="Setting Firewall for Windows machine" src="https://github.com/user-attachments/assets/e6349a42-31b3-4e5c-a1e0-875052727296">

The Linux machine was booted using VMware, for which I downloaded an ISO from the Kali website.  

## LimaCharlie Installation 
After creating a LimaCharlie project (or per LimaCharlie terms an organization), I looked for the installation instructions. The following screenshot is from the LimaCharlie "Installation Keys" screen, which gives some installation instructions for different machines:

<img width="1792" alt="LC Insta Key" src="https://github.com/user-attachments/assets/da993506-1b1c-40c3-9176-ab0c0e4a5fca">

## Windows Installation
From the instructions, I needed to run the following command: _lc_sensor.exe -i YOUR_INSTALLATION_KEY_

I logged into the Windows machine using the provided administrator password using the remote console provided by Vultr. I copied the "Windows 64 bit" link under "Sensor Downloads", which gave me the .exe file needed for the command, and then after copying my "Sensor Key" (the _YOUR_INSTALLATION_KEY_ variable), I opened PowerShell as admin and ran the command: 

<img width="1331" alt="Windows LC download powershell" src="https://github.com/user-attachments/assets/1c3a2a10-a5ff-490f-a0d3-fb2ce3de8318">

After running it I received a success message:

<img width="1376" alt="Windows LC Success" src="https://github.com/user-attachments/assets/18f8573d-40e0-47a4-b281-7b24d5d01e14">

From here, when navigating to the "Sensors" screen in LimaCharlie the Windows machine could be seen displayed:

<img width="1792" alt="LC Windows connect" src="https://github.com/user-attachments/assets/0599829b-4ccc-4e5f-8b13-ffd0e23a6add">

Clicking on the machine leads the user to a "Sensor Details" window where various propoerties of the sensor are displayed. In particular, the especially useful panes here are "Detections", which shows detections on the machine, "Console", which allows the remote sending of commands to the machine, and "File System" which allows the user to see the machine's file system, which includes downloading of files as well as their various metadata.

<img width="1792" alt="Click on windows" src="https://github.com/user-attachments/assets/5d63c902-9479-4a78-b77d-28230450e3c6">

## Linux Installation

The process for Linux is effectively the same as windows. This time the command provided is slightly different, as on Linux the downloaded file must first be given execute permissions. In my case the sensor download was the "Linux 64 bit" version. Executing the given command leads to the same screen as for Windows, followed by the Linux machine appearing online on LimaCharlie. 



























