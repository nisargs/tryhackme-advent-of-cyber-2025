# Day 3: Splunk Basics - Did you SIEM?

## 📝 Scenario
In Day 3, the SOC dashboard at "The Best Festival Company" (TBFC) flashed red. A ransomware group known as "Bandit Bunnies" (led by King Malhare) compromised the network.

My objective was to act as a **SOC Analyst** and utilize **Splunk (SIEM)** to perform an incident response investigation. I needed to reconstruct the attack chain—from initial access to data exfiltration—identify the adversary's IP address, and quantify the damage.

## 🛠️ Tools & Technologies Used
* **Splunk:** A Security Information and Event Management (SIEM) tool used for searching, monitoring, and analyzing machine-generated data.
* **Log Analysis:** Interpreting HTTP status codes, User Agent strings, and network traffic patterns.
* **Cyber Kill Chain:** Applying the framework to map the attacker's steps from reconnaissance to actions on objectives.

## 🕵️‍♂️ Investigation Walkthrough

### Step 1: Initial Triage & Identifying the Attacker
My first goal was to distinguish between legitimate website visitors and the attacker.
* **Theory:** Attackers often use automated scripts or custom tools that leave distinct "User Agent" signatures. Legitimate users typically use standard browsers like Chrome or Firefox.
* **Action:** I filtered the `web_traffic` logs to **exclude** all known, standard browser User Agents. This isolation technique immediately revealed a high volume of requests coming from a single, suspicious IP address using non-standard agents.

### Step 2: Timeline Analysis
To understand the scope of the incident, I needed to pinpoint *when* the attack occurred.
* **Theory:** Cyber attacks often gets evident as sudden spikes in network activity or log volume (scanning or brute-forcing) compared to the baseline of normal daily traffic.
* **Action:** I visualized the event count over time and that revealed a massive spike in traffic on a specific date, confirming the exact window of the compromise.

### Step 3: Mapping the Attack Vectors
Once the Attacker IP was confirmed, I analyzed their specific HTTP requests to reconstruct their methodology.

**A. Reconnaissance (Directory Traversal)**
* **Theory:** Attackers often try to "break out" of the web directory to access sensitive system files (like `/etc/passwd` or configuration files).
* **Action:** I searched the logs for URL paths containing `../` patterns. The high count of these requests confirmed the attacker was attempting **Path Traversal** to map the file system.

**B. Exploitation (SQL Injection)**
* **Theory:** Automated tools are frequently used to inject malicious SQL commands into input fields to steal database data.
* **Action:** I analyzed the User Agent strings associated with the attacker's IP. I found signatures for **Havij** and **SQLMap**, confirming that an automated **SQL Injection** attack was launched against the web application.

**C. Installation (RCE & Webshell)**
* **Theory:** To maintain access, attackers upload "webshells"; scripts that allow them to execute system commands remotely.
* **Action:** I looked for requests accessing suspicious file extensions or names. I discovered the attacker successfully accessed a file named `shell.php` and executed a binary named `bunnylock.bin` (the ransomware payload).

### Step 4: C2 Communication & Exfiltration
Finally, I determined if the attack resulted in data theft.
* **Theory:** Once inside, malware often "phones home" to a Command & Control (C2) server to receive instructions or upload stolen data. This creates a distinct "Outbound" traffic pattern in firewall logs.
* **Action:** I pivoted to the `firewall_logs` and filtered for connections originating from our compromised server and destined for the Attacker's IP. I then summed the total bytes transferred, confirming that a significant amount of data was exfiltrated from the network.

## 🧠 Key Takeaways
* **Correlation is Key:** The power of a SIEM lies in correlating different data sources. I was able to connect web attacks (Layer 7) with network connections (Layer 3/4) to prove the attack was successful.
* **Filtering the Noise:** In a sea of logs, knowing what is *normal* (e.g., standard browsers) is the fastest way to find what is *abnormal*.
* **The Cyber Kill Chain:** I learned how to identify every stage of an attack in logs: from the initial scan (Recon) to the final data theft (Exfiltration).

## 📸 Screenshots
*Below are the findings from the Splunk investigation.*

![Attacker IP Identification](screenshots/Day%203/1.png)
*Figure 1: Identifying the Attacker IP by filtering out legitimate User Agents.*

![Path Traversal Attempts](screenshots/Day%203/2.png)
*Figure 2: Specific date of peak malicious traffic.*

![Data Exfiltration Verification](screenshots/Day%203/3.png)
*Figure 3: Count of Havij user_agent events found in the logs.*

![Data Exfiltration Verification](screenshots/Day%203/4.png)
*Figure 3: Number of path traversal attempts to access sensitive files on the server.*

![Data Exfiltration Verification](screenshots/Day%203/5.png)
*Figure 3: Total bytes transferred to the C2 server IP from the compromised web server.*

## 🔗 References
* [TryHackMe - Advent of Cyber 2025](https://tryhackme.com/adventofcyber25)
* [Splunk Documentation](https://docs.splunk.com/Documentation/Splunk/latest/SearchReference/)