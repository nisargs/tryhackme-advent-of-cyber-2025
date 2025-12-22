# Day 22: C2 Detection - Command & Carol

## 📝 Scenario
In Day 22, the TBFC (The Best Festival Company) is on high alert for further attacks from King Malhare. The team has collected a vast amount of network traffic (PCAPs) but lacks the time to analyze it manually. Sir Elfo introduces **RITA** (Real Intelligence Threat Analytics) to automate the hunt for **Command and Control (C2)** beacons and malicious connections hiding within the noise.

As a security analyst, my objective was to convert raw PCAP files into structured **Zeek** logs and then use **RITA** to identify malicious domains, detecting beaconing behavior and long-duration connections indicative of a breach.

## 🛠️ Tools & Technologies Used
* **Zeek:** An open-source network security monitoring tool used here to parse raw PCAP files into structured text logs (HTTP, DNS, SSL, etc.).
* **RITA (Real Intelligence Threat Analytics):** An open-source framework that analyzes Zeek logs to detect C2 activity, such as periodic beacons, long connections, and DNS tunneling.
* **Linux Terminal:** Used to execute the conversion and analysis commands.

## 🧠 Theoretical Concepts
* **Command & Control (C2):** The infrastructure attackers use to communicate with compromised systems. Detecting this traffic is crucial for stopping active intrusions.
* **Beaconing:** A pattern where infected malware sends "heartbeat" signals to the C2 server at regular intervals to check for instructions. RITA excels at spotting this periodicity.
* **Threat Modifiers:** Metrics used by RITA to score severity:
    * **Prevalence:** How many internal hosts talk to a specific external IP? (Low prevalence often = suspicious).
    * **Rare Signature:** Unique User-Agents or TLS patterns seen nowhere else on the network.
    * **Long Connections:** Connections kept open for extended periods (common in C2 channels).

## 🕵️‍♂️ Investigation Walkthrough

### Step 1: Converting PCAP to Zeek Logs
RITA requires structured logs, not raw packets. I started by converting the provided PCAP file using Zeek.
1.  **Command:**
    ```bash
    zeek readpcap pcaps/rita_challenge.pcap zeek_logs/rita_challenge
    ```
    *This generated standard Zeek log files like `conn.log`, `dns.log`, and `http.log` in the output directory.*



### Step 2: Importing into RITA
Next, I imported the generated logs into the RITA database for analysis.
1.  **Command:**
    ```bash
    rita import --logs ~/zeek_logs/rita_challenge/ --database rita_challenge
    ```

### Step 3: Analyzing the Data
I launched the RITA viewer to inspect the findings.
1.  **Command:** `rita view rita_challenge`
2.  **Interface:** This opened a dashboard showing scores for **Beacon Score**, **Severity**, and **Total Bytes**.

### Step 4: Hunting for "Malhare"
I investigated specific domains related to the attacker (`malhare.net`).
1.  **Filtering:** inside the view, I used the `/` key to search for `malhare.net`.
2.  **Host Count:** I examined the results to see how many distinct internal IPs were communicating with this domain.
3.  **Connection Analysis:** I drilled down into `rabbithole.malhare.net` to check the **Connection Count** and **Port** usage for specific hosts (e.g., `10.0.0.13`).

### Step 5: Advanced Filtering
To refine the hunt for high-probability threats, I utilized RITA's search syntax.
* **Objective:** Find entries communicating with `rabbithole.malhare.net` with a high beacon score.
* **Filter Logic:** `beacon-score>0.7` combined with domain filters.

## 🧠 Key Takeaways
* **Logs over Packets:** While PCAPs contain all the data, they are heavy. Converting them to Zeek logs allows for faster, metadata-focused analysis suitable for statistical baselining.
* **Frequency Analysis:** You often can't see "malice" in a single packet. You see it in the *pattern* over time (e.g., a connection occurring exactly every 5 seconds).
* **RITA's Strength:** It automates the statistical heavy lifting, highlighting outliers (Long Connections, Rare User Agents) that a human eye would miss in a massive log file.

## 📸 Screenshots
*Below are screenshots corresponding to the answers for the Day 22 task questions.*

![Malhare Hosts Count](screenshots/Day%2022/1.png)
*Figure 1: RITA analysis showing the total number of internal hosts communicating with malhare.net.*

![Threat Modifier Definition](screenshots/Day%2022/2.png)
*Figure 2: The details pane displaying the Threat Modifier that indicates the number of communicating hosts.*

![Connection Count](screenshots/Day%2022/3.png)
*Figure 3: Highlighting the highest number of connections made to rabbithole.malhare.net.*

![Advanced Search Filter](screenshots/Day%2022/4.png)
*Figure 4: The specific search query used to filter for high beacon scores and sort by duration.*

![Port Identification](screenshots/Day%2022/5.png)
*Figure 5: Connection details revealing the specific port used by host 10.0.0.13.*

## 🔗 References
* [TryHackMe - Advent of Cyber 2025](https://tryhackme.com/adventofcyber25)
* [Active Countermeasures - RITA](https://www.activecountermeasures.com/free-tools/rita/)