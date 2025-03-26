# Azure Honeypot Lab

## Project Overview
This **Azure Honeypot Lab** simulates an exposed endpoint on the public internet to observe and analyze real-world attacker behavior. The goal is to detect unauthorized login attemps, analyze logs with Microsoft Sentinel, and visualize attacker origins with a live threat map.

The lab consists of:
- A **Windows 10 virtual machine (VM)** hosted on Microsoft Azure to act as our honeypot
- **Microsoft Sentinel** as our Security Information and Event Management system (SIEM)
- A **custom threat map** displaying global attacker locations based on failed login attempts

## Objectives
- Deploy a honeypot in Microsoft Azure that is fully exposed to the public internet
- Capture and analyze failed login attempts using Microsoft Sentinel
- Query logs using Kusto Query Language (KQL)
- Map global attack origins using geolocation data

## Lab Setup
To set the lab up, we need to configure the Microsoft Azure environment. This consists of establishing a **Resource Group**, **Virtual Network**, and finally a **Windows 10 VM**.

### Resource Group
The Resource Group is the main hub for everything in the lab. Think of it like a folder that holds all of our cloud resources such as the **virtual machine**, **virtual network**, etc.

Resource Group Settings: ![1  Resource Group Creation](https://github.com/user-attachments/assets/c04b2300-751d-4509-a75b-154147a5776c)

### Virtual Network
The Virtual Network is what allows the VM to access the internet. You can think of it like a cloud-based version of a home router. For the purposes of this lab, all major security features are disabled to allow as much incoming traffic as possible.

Virtual Network Settings: ![2  VNET Creation](https://github.com/user-attachments/assets/d285f32d-87b7-4a86-ad31-c1c47475a435)

### Virtual Machine
With the Resource Group and Virtual Network are in place, the next step is to configure the virtual machine. To set up the VM for this lab, we will:
- Disable Azure security controls by opening the **Network Security Group (NSG)**
- Disable **Windows Firewall** inside the VM
- Run a ping test to confirm the VM is reachable over the public internet

Virtual Machine Creation in Azure: ![3  Virtual Machine Creation](https://github.com/user-attachments/assets/5571c25f-6d9d-428d-8b15-f643dbb064f3)

Network Security Group Settings:

![4  VM Security Settings](https://github.com/user-attachments/assets/1d2dc050-b94f-4b7c-a0ad-f63b1cb144f4)

Disabling Windows Firewall Inside VM: ![5  VM Firewall Settings](https://github.com/user-attachments/assets/33ecf8a3-d6ee-4b1e-ac0e-413638f00ded)

Ping Test Confirming VM is Reachable Over Public Internet: ![6  VM Ping Test](https://github.com/user-attachments/assets/0dfe18cd-2952-4175-ae9f-debc1a4e09c5)

## Simulating Attacker Activity
To generate logs, we intentionally fail login attempts to the VM via Remote Desktop. These login failures are observed in **Windows Event Viewer** under 'Event ID 4625'

Intentionally Failing Login Attempts: 

![7  Log Generation_Intentional Failed Attempts](https://github.com/user-attachments/assets/efbbe03a-5180-47f7-9243-ab3c624d456f)

Observing a Failed Login Attempt in Windows Event Viewer: ![8  Event Viewer Search](https://github.com/user-attachments/assets/d0d71ee2-3555-48c6-ae5e-67ca97cd34e5)

Logs Generated for 'Event ID 4625' (Logon Failure): ![9  Self Gen Login Fail Logs List](https://github.com/user-attachments/assets/04e03825-768a-4e68-b32d-20c11a19d814)

## Log Analytics and Microsoft Sentinel Integration
After making sure that logs are being generated on the VM, we need a way to forward those logs into Azure and Microsoft Sentinel.

We can complete this in three steps:
1. Create a **Log Analytics Workspace**
2. Connect the Workspace to an instance of **Microsoft Sentinel**
3. Install the **Azure Monitoring Agent** on the VM



