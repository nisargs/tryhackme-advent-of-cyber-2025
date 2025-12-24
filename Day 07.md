# Day 7: Network Discovery - Scan-ta Clause

## 📝 Scenario
In Day 7, the QA environment (`tbfc-devqa01`) at "The Best Festival Company" has been compromised by HopSec, effectively freezing the SOC-mas pipeline. The server is active but locked down, and it is slowly being transformed into an "EAST-mas" node.

As the security responder, my objective was to perform **Network Discovery** to identify how the attackers were maintaining access. I needed to scan for open ports, enumerate exposed services, collect hidden keys to bypass the lock, and ultimately access the internal database to reclaim the server.

## 🛠️ Tools & Technologies Used
* **Nmap:** The industry standard for network discovery and security auditing, used here to find open TCP and UDP ports.
* **Netcat (nc):** A versatile networking utility used to interact with raw TCP connections on custom ports.
* **FTP Client:** Used to access files on the exposed File Transfer Protocol service.
* **Dig:** A DNS lookup utility used to query the domain name server for hidden TXT records.
* **Socket Statistics (ss):** A command-line utility used post-compromise to investigate listening ports on the local system.
* **MySQL Client:** Used to interact with the internal database and retrieve the final flag.

## 🧠 Theoretical Concepts
* **Port Scanning:** The process of sending packets to specific ports on a host and analyzing the responses to determine if services are running (Open, Closed, or Filtered).
* **Service Enumeration:** Identifying exactly *what* application and version is running on an open port (often done via Banner Grabbing).
* **TCP vs. UDP:**
    * **TCP:** Connection-oriented (reliable). Used for SSH, HTTP, FTP.
    * **UDP:** Connectionless (faster, fire-and-forget). Used for DNS.
* **Localhost Enumeration:** Once inside a machine, checking for services listening only on `127.0.0.1` (localhost) that are not accessible from the outside internet.

## 🕵️‍♂️ Investigation Walkthrough

### Step 1: Initial Reconnaissance (Nmap)
I began by scanning the target machine to identify "places to hide."
* **TCP Scan:** I ran `nmap -p- --script=banner MACHINE_IP` to scan all 65,535 ports and grab service banners.
    * **Result:** This revealed standard ports (22 SSH, 80 HTTP) and non-standard ports: **21212 (FTP)** and **25251 (Custom TBFC Service)**.

### Step 2: Service Enumeration - FTP & Netcat
I investigated the non-standard ports to find the first two parts of the admin key.
1. **Port 21212 (FTP):** I connected using `ftp MACHINE_IP 21212` with `anonymous` credentials.
    * **Result:** I downloaded a file named `tbfc_qa_key1` containing the first key part.
2. **Port 25251 (Custom App):** Since this wasn't a standard protocol, I used **Netcat** to establish a raw connection: `nc -v MACHINE_IP 25251`.
    * **Result:** The service prompted for commands. I interacted with it to retrieve the second hidden key.

### Step 3: UDP Scanning & DNS Enumeration
Suspecting more hidden services, I switched protocols.
1. **UDP Scan:** I ran `nmap -sU MACHINE_IP` to check for UDP services.
    * **Result:** This revealed **Port 53 (DNS)** was open.
2. **DNS Query:** I used `dig` to query the server for specific records: `dig @MACHINE_IP TXT key3.tbfc.local +short`.
    * **Result:** The DNS TXT record returned the third and final key part.

### Step 4: Post-Exploitation & Database Access
With all three keys, I logged into the web admin panel on Port 80. This granted me access to a web-based console.
1. **Internal Discovery:** I ran `ss -tunlp` to list ports listening internally.
    * **Result:** I spotted **Port 3306 (MySQL)** listening on `127.0.0.1`.
2. **Database Dump:** Since I was now "local" (via the web shell), I could access the database without a password. I used the command: `mysql -D tbfcqa01 -e "select * from flags;"`
    * **Result:** This queried the `flags` table and printed the final flag, confirming we had regained control of the server.

## 🧠 Key Takeaways
* **Scan All Ports:** Standard scans often miss services hiding on high ports (like 21212). Using `-p-` is crucial for complete visibility.
* **Don't Forget UDP:** Attackers (and sysadmins) often use UDP for services like DNS or SNMP. A TCP-only scan paints an incomplete picture.
* **The Pivot:** Gaining initial access (web shell) is often just the beginning. The real "treasure" (the database) was only accessible from inside the network (localhost), requiring post-exploitation enumeration.

## 📸 Screenshots
*Below are the findings from the network discovery operation.*

![Nmap Scan Results](screenshots/Day%207/1.png)
*Figure 1: Message on the top of the website.*

![FTP Key Retrieval](screenshots/Day%207/2.png)
*Figure 2: Retrieving the first key part from the FTP server running on port 21212.*

![Dig DNS Query](screenshots/Day%207/3.png)
*Figure 3: Second key part found in the TBFC app.*

![Internal Ports SS](screenshots/Day%207/4.png)
*Figure 4: Third key part found in the DNS records.*

![MySQL Flag Dump](screenshots/Day%207/5.png)
*Figure 5: Port where MySQL database is running.*

![MySQL Flag Dump](screenshots/Day%207/6.png)
*Figure 6: Final flag from the internal MySQL database.*

## 🔗 References
* [TryHackMe - Advent of Cyber 2025](https://tryhackme.com/adventofcyber25)