# Day 10: SOC Alert Triaging - Tinsel Triage

## 📝 Scenario
In Day 10, the Security Operations Center (SOC) at "The Best Festival Company" was overwhelmed by a "blizzard" of alerts. The Easter Bunnies launched a coordinated attack against the cloud infrastructure, triggering widespread warnings across the Azure environment.

As a SOC Analyst working alongside McSkidy, my objective was to perform **Alert Triage** using Microsoft Sentinel. I needed to cut through the noise, prioritize high-severity incidents, investigate logs using KQL (Kusto Query Language), and correlate distinct events to reconstruct the attack timeline.

## 🛠️ Tools & Technologies Used
* **Microsoft Sentinel:** A cloud-native SIEM (Security Information and Event Management) and SOAR (Security Orchestration, Automation, and Response) platform used to collect data and detect threats.
* **Kusto Query Language (KQL):** The powerful query language used within Sentinel to search and filter through massive datasets.
* **Syslog:** The standard logging protocol used to capture system events from the Linux servers in the environment.

## 🧠 Theoretical Concepts
* **Alert Triage:** The process of evaluating incoming security alerts to determine their urgency. It relies on four key factors:
    * **Severity:** How critical is the threat? (High, Medium, Low)
    * **Time:** When did it happen? Is it ongoing?
    * **Context:** Where is this in the attack lifecycle (e.g., Reconnaissance vs. Data Exfiltration)?
    * **Impact:** What asset (server, user, database) is affected?
* **Correlation:** Connecting seemingly unrelated log entries (e.g., an SSH login followed by a file download) to identify a cohesive attack pattern.
* **False Positives:** Alerts triggered by benign activity that must be filtered out to focus on real threats.

## 🕵️‍♂️ Investigation Walkthrough

### Step 1: Initial Triage
I began by accessing the **Microsoft Sentinel Incidents** dashboard. The view was populated with multiple alerts ranging from Medium to High severity. I focused on the High severity alerts first, specifically those labeled **"Linux PrivEsc"** (Privilege Escalation).
* **Observation:** I noticed multiple alerts linked to specific entities (Hostnames and IP addresses). This "clustering" of alerts suggested a multi-stage attack rather than isolated incidents.

### Step 2: Analyzing the Attack Chain
I drilled down into the details of the **"Linux PrivEsc - Kernel Module Insertion"** alert. By viewing the full incident details and timeline, I reconstructed the attacker's path:
1.  **Initial Access:** The attacker gained access via SSH from an external IP.
2.  **Persistence:** They attempted to install a malicious kernel module.
3.  **Privilege Escalation:** There were indicators of users being added to the `sudoers` group and sensitive files (like `/etc/shadow`) being accessed.

### Step 3: Deep Dive with KQL
To confirm the specific actions taken by the attacker, I switched to the **Logs** tab and utilized KQL to query the `Syslog_CL` table.

* **Investigating `websrv-01`:**
    I filtered the logs for this specific host to identify malicious kernel modules. I found logs indicating a specific `.ko` module had been inserted. I also discovered the `ops` user executing an unusual command involving a scripting language to run a reverse shell.

* **Investigating `storage-01`:**
    I queried the logs for the `sshd` process on this storage server. By sorting the logs chronologically, I identified the source IP address responsible for the first successful SSH login.

* **Investigating `app-01`:**
    I looked for successful root login attempts and modifications to the sudoers file. The logs revealed that an external IP successfully logged in as root, and a specific user account (other than the backup user) was illicitly added to the sudoers group to maintain administrative access.

## 🧠 Key Takeaways
* **Context is King:** A single alert (like "User added to group") might look innocent, but when seen alongside "Kernel Module Insertion" on the same host, it confirms a critical breach.
* **Prioritization:** In a flooded SOC, you cannot investigate everything at once. Focusing on High Severity and High Impact assets first is essential for damage control.
* **The Power of KQL:** Dashboards provide a summary, but the ability to write custom queries (KQL) is necessary to find specific "needles in the haystack," such as specific IP addresses or filenames.

## 🔗 References
* [TryHackMe - Advent of Cyber 2025](https://tryhackme.com/adventofcyber25)