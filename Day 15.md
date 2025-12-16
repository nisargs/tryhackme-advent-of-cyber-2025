# Day 15: Web Attack Forensics - Drone Alone

## 📝 Scenario
In Day 15, the drone scheduler web UI at "The Best Festival Company" is exhibiting strange behavior. Splunk has triggered an alert stating "Apache spawned an unusual process." Long HTTP requests containing Base64 chunks are hitting the server, causing it to execute shell code.

As a Blue Teamer (Elf Log McBlue), my objective was to triage this incident using **Splunk**. I needed to pivot between web server logs (Apache) and host-level telemetry (Sysmon) to identify the compromised endpoints, decode the obfuscated payloads, and reconstruct the full attack chain.

## 🛠️ Tools & Technologies Used
* **Splunk:** A centralized SIEM (Security Information and Event Management) platform used to search, monitor, and analyze machine-generated data.
* **Apache Access/Error Logs:** Web server logs used to identify the initial intrusion vector (malicious HTTP requests).
* **Sysmon (System Monitor):** Windows system service and device driver that logs system activity to the event log, used here to track process creation and command execution.
* **Base64 Decoder:** Used to decode the obfuscated PowerShell payloads found in the logs.

## 🧠 Theoretical Concepts
* **Command Injection:** A cyber attack where the attacker executes arbitrary operating system commands on the server that is running an application (e.g., via a vulnerable CGI script like `hello.bat`).
* **Log Pivoting:** The analytical technique of moving from one data source (e.g., Web Logs) to another (e.g., System Logs) based on a common identifier (like a timestamp or IP) to validate findings.
* **Obfuscation:** Hiding the intent of a script by encoding it (e.g., Base64) to bypass simple keyword filters.
* **Parent-Child Process Relationship:** Identifying malware by checking "who started who." For example, a web server (`httpd.exe`) should generally not spawn a command shell (`cmd.exe`).

## 🕵️‍♂️ Investigation Walkthrough

### Step 1: Web Log Forensics (Initial Detection)
I began by searching the Apache access logs for signs of command injection. Attackers often try to spawn shells or run PowerShell.
1.  **Splunk Query:**
    ```splunk
    index=windows_apache_access (cmd.exe OR powershell OR "powershell.exe" OR "Invoke-Expression") | table _time host clientip uri_path uri_query status
    ```
2.  **Result:** This query returned HTTP requests attempting to execute `powershell`.
3.  **Decoding:** I found a suspicious Base64 string in the URI (`VABoAGkAcw...`). Decoding this revealed a taunting message: *"This is now Mine! MUAHAAAA"*.

### Step 2: Error Log Analysis (Validation)
To see if the server actually processed these malicious requests, I checked the error logs.
1.  **Splunk Query:**
    ```splunk
    index=windows_apache_error ("cmd.exe" OR "powershell" OR "Internal Server Error")
    ```
2.  **Result:** I observed "Internal Server Error" (Status 500) messages linked to `hello.bat` attempting to run PowerShell. This confirmed the payload reached the backend logic.

### Step 3: Sysmon Pivoting (Tracing Execution)
Now that I knew the web server was targeted, I checked if the OS actually executed the commands. I looked for process creation events where Apache was the *parent*.
1.  **Splunk Query:**
    ```splunk
    index=windows_sysmon ParentImage="*httpd.exe"
    ```
2.  **Analysis:** The results showed `httpd.exe` (Apache) spawning `cmd.exe`. This is a "Smoking Gun" for successful Remote Code Execution (RCE).
    
### Step 4: Post-Exploitation Reconnaissance
Attackers usually run discovery commands immediately after gaining access. I searched for standard reconnaissance tools.
1.  **Splunk Query:**
    ```splunk
    index=windows_sysmon *cmd.exe* *whoami*
    ```
2.  **Result:** This confirmed the attacker ran `whoami.exe` to check their privileges on the compromised host.

### Step 5: Hunting Encoded Payloads
Finally, I looked for the execution of the hidden Base64 payloads on the OS level.
1.  **Splunk Query:**
    ```splunk
    index=windows_sysmon Image="*powershell.exe" (CommandLine="*enc*" OR CommandLine="*-EncodedCommand*" OR CommandLine="*Base64*")
    ```
2.  **Conclusion:** This query isolates the specific PowerShell instances where the attacker tried to run their obfuscated scripts.

## 🧠 Key Takeaways
* **Correlation is Key:** A 500 Error in Apache logs might look like a glitch, but when correlated with `httpd.exe` spawning `cmd.exe` in Sysmon, it confirms a breach.
* **Know Your Normal:** Web servers should spawn worker threads, not command prompts. Understanding the "normal" behavior of a service allows you to spot anomalies instantly.
* **Obfuscation != Security:** Just because a payload is Base64 encoded doesn't mean it's hidden. SIEMs can alert on the *presence* of encoded commands (like `-EncodedCommand`), prompting further investigation.

## 📸 Screenshots
*Below are the findings from the Splunk investigation.*

![Reconnaissance Detection](screenshots/Day%2015/1.png)
*Figure 1: Detecting the attacker running 'whoami' for reconnaissance.*

![Reconnaissance Detection](screenshots/Day%2015/2.png)
*Figure 2: The executable the attacker attempt to run through the command injection.*

## 🔗 References
* [TryHackMe - Advent of Cyber 2025](https://tryhackme.com/adventofcyber25)
* [Splunk Documentation](https://docs.splunk.com/Documentation/Splunk)