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
To generate logs, we intentionally fail login attempts to the VM via Remote Desktop. These login failures are observed in **Windows Event Viewer** under 'Event ID 4625'.

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

Log Analytics Workspace created: ![10  LAW Creation](https://github.com/user-attachments/assets/cbd817ba-4c65-434d-8fc4-e5251b0b22a9)

Adding Microsoft Sentinel with Windows Security Events to the Log Analytics Workspace: ![12  Windows Security Events Installation](https://github.com/user-attachments/assets/ef22bbde-e82e-4e52-99e5-968af362449b)

Creating a Data Collection Rule for the Azure Monitoring Agent which is used by the VM to forward logs to the Log Analytics Workspace: ![14  DCR Creation](https://github.com/user-attachments/assets/d1a0edf9-8e07-48d4-98c5-974e9e242b6c)

## First Attacker Activities and KQL Queries
After about an hour after configuring our Log Analytics Workspace, the first failed login attempt that was not done myself was detected. The attacker attempted to login as "\guest" and came from an IP address based in Switzerland.

Details of the first failed login: ![16  First External Detection](https://github.com/user-attachments/assets/1e3216fc-b67c-49ba-b2b6-2891eda72efb)

IP Geolocation: ![17  First Geolocation](https://github.com/user-attachments/assets/8f463558-441b-481c-963b-d1ed49159365)

There wasn't much activity after the first login attempt, so I left the VM running overnight. When I checked the logs in the morning, they were full of real-world attack data.

Logs after leaving the VM running overnight: ![18  Overnight Detections](https://github.com/user-attachments/assets/e6f36690-d09c-4df8-8c4b-7b89558bad38)

The logs return a lot of useful information, but KQL makes it easy to make them only return categories that are important to the task at hand.

KQL query used to simplify the results: ![19  Query To Simplify](https://github.com/user-attachments/assets/08096394-8f9c-4b75-a79c-3f948c1b0657)

Geolocation data of this particular attempt: ![20  Query Geolocation](https://github.com/user-attachments/assets/509af239-33c0-4986-afd2-6dca671641e2)

Instead of a certain account, we can also search by EventID: ![21  Query By Security EventID](https://github.com/user-attachments/assets/0630b435-59b1-4bac-87c1-9ce8d45f914b)

## Geolocation and Threat Map Detection

## Method 1 - Initial Threat Map
At first, I manually uploaded a large IP-to-Geo CSV watchlist to Microsoft Sentinel. This method worked, but the data in the CSV file was outdated. As a result, the IP addresses didn't match up to where they were actually coming from. For example, IPs from Russia showed up as coming from England and Poland or IPs from Thailand showed as China.

CSV Uploaded: ![22  Inaccurate Map 1](https://github.com/user-attachments/assets/487dbc7a-9f0e-4055-b879-ed7c966cfc76)

KQL Query organizing data: ![25  Inaccurate Map 4](https://github.com/user-attachments/assets/9d8b60f0-d76f-4b65-9441-57a75b100626)

Map query and resulting threat map: ![27  Inaccurate Map 6](https://github.com/user-attachments/assets/af337abc-0664-4e38-ab89-eb5811c16b13)

## Method 2 - Updated and Final Threat Map
I could have completed the lab with the outdated map, but for educational purposes, I decided to dig deeper and figure out how to build a more accurate one. Using this guide: ["Azure Sentinel SIEM Lab to Map Live Cyber Attacks"](https://medium.com/@mxyiwa/azure-sentinel-siem-lab-to-map-live-cyber-attacks-d74c2426d59b), I implemented the threat map creation section into this lab.
- Enriched IP address data using a current geolocation database
- Implementing the database API directly in the VM using a custom PowerShell script
- Creating a live threat map using Microsoft Sentinel Workbooks

  This script utilizes the geolocation API from [ipgeolocation.io](https://ipgeolocation.io) to export the results of the script into a log file that will be pasted into a Sentinel Workbook to create the threat map.
  ![29  Custom Powershell Script](https://github.com/user-attachments/assets/52cf35eb-d517-4a9c-9042-1069783aecd9)

Script being executed: ![30  Script Executed](https://github.com/user-attachments/assets/45a37f47-70fc-464a-80c6-5aefa38e3fb0)

Creating a custom log based on the PowerShell script's log file: ![32  Log Creation](https://github.com/user-attachments/assets/0ada6a65-762b-429e-b45a-d1b6623cab8c)

Data displayed using the custom log: ![34  Cleaning Up The Data](https://github.com/user-attachments/assets/638fc67b-9029-4c70-8bf5-57b3ca7ba876)

Running the same query in a Sentinel Workbook to create the threat map:![35  Map Workbook Creation](https://github.com/user-attachments/assets/632cd94a-54c0-40bf-a376-56e8e0fee4fe)

Final threat map (limited at 1000 responses for the free version of the API): ![36  Final Map](https://github.com/user-attachments/assets/211fd480-2a57-4f76-b705-a4f6573edf39)

## Key Takeaways
- **Real-world attacks**: This project was a great way for me to build my hands-on skills with SIEM tools, logging, and setting up a cloud infrastructure. It was also really interesting to see these real attacks and put a location to their IP addresses.
- **Logging REQUIRES visibility**: Capturing logs by themselves is one thing, but connecting them to a SIEM like Microsoft Sentinel turns raw data into actionable insights
- **Even low-profile endpoints get attacked fast**: Exposing a single VM to the public internet led to thousands of login attempts within hours. This made me realize just how active attackers are on the open web.
- **Doing more where it matters**: Instead of stopping at what I thought was good enough, I went beyond to improve the threat map with more accurate geolocation data. As a result, I learned a lot more in the process.

