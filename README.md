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
