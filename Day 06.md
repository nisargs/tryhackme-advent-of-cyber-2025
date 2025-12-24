# Day 6: Malware Analysis - Egg-xecutable

## 📝 Scenario
In Day 6, the SOC team at "The Best Festival Company" (TBFC) received a suspicious email at 3:00 AM from the "Head of Elf Affairs." The email contained an attachment named `HopHelper.exe`, claiming to be a new tool for creating team rotas.

Suspecting a phishing attempt and potential malware payload, my objective was to act as a **Malware Analyst** (Elf McBlue). I utilized a sandbox environment to perform both **Static** and **Dynamic** analysis on the suspicious executable to determine its capabilities, persistence mechanisms, and network indicators.

## 🛠️ Tools & Technologies Used
* **PeStudio:** A tool for static analysis to fingerprint the malware (Hashes, Strings, Imports) without running it.
* **Regshot:** A registry comparison tool used to identify changes made to the system (Persistence).
* **Process Monitor (ProcMon):** A sysinternals tool for real-time monitoring of file system, registry, and process activity.
* **Sandbox:** An isolated Virtual Machine (VM) used to safely detonate the malware.

## 🧠 Theoretical Concepts
* **Static Analysis:** Examining the code without running it. Safe, but limited if the code is obfuscated.
* **Dynamic Analysis:** Running the code in a controlled environment (Sandbox) to observe its behavior.
* **Persistence:** Techniques used by malware to ensure it restarts automatically after a system reboot (e.g., Registry Run Keys).

## 🕵️‍♂️ Investigation Walkthrough

### Step 1: Static Analysis (PeStudio)
I began by analyzing the `HopHelper.exe` file in **PeStudio** without executing it.
* **Fingerprinting:** I navigated to the "indicators" tab to retrieve the **SHA256 checksum**. This hash acts as a unique signature for the malware, which can be searched in databases like VirusTotal.
* **String Analysis:** I inspected the "strings" section (readable text embedded in the binary). Buried within the data, I discovered a hidden flag formatted as `THM{...}`.

### Step 2: Dynamic Analysis - Persistence (Regshot)
To see how the malware survives a reboot, I used **Regshot**.
1.  **1st Snapshot:** I took a snapshot of the clean registry state.
2.  **Detonation:** I executed `HopHelper.exe`.
3.  **2nd Snapshot:** I took a second snapshot of the infected registry.
4.  **Comparison:** I compared the two snapshots.
    * **Result:** The comparison report revealed that the malware added an entry to the `\Run` registry key. This ensures `HopHelper.exe` executes every time the user logs in.

### Step 3: Dynamic Analysis - Network Activity (ProcMon)
Finally, I wanted to see who the malware was talking to. I fired up **Process Monitor (ProcMon)**.
1.  I started capturing events and executed the malware again.
2.  **Filtering:** The output was noisy, so I applied a filter:
    * `Process Name` **is** `HopHelper.exe`
    * `Operation` **contains** `TCP`
3.  **Result:** The filtered logs showed the malware initiating network connections. I identified the specific protocol used for Command & Control (C2) communication.

## 🧠 Key Takeaways
* **The Golden Rule:** Never run suspicious executables on a production machine. Always use an isolated sandbox.
* **Persistence is Noisy:** Malware almost always leaves a footprint when trying to survive a reboot. Tools like Regshot make these changes obvious.
* **Behavior over Appearance:** Static analysis gave me a hint (the flag), but only Dynamic analysis revealed *what* the malware actually did (network beacons and registry changes).

## 📸 Screenshots
*Below are the forensic findings from the malware analysis.*

![SHA256 Hash](screenshots/Day%206/1.png)
*Figure 1: Retrieving the SHA256 hash of the malware from PeStudio.*

![Hidden Strings Flag](screenshots/Day%206/2.png)
*Figure 2: Locating the hidden THM flag within the executable's strings.*

![Registry Persistence](screenshots/Day%206/3.png)
*Figure 3: Regshot comparison showing the malicious entry added to the Registry Run key.*

![Network C2 Traffic](screenshots/Day%206/4.png)
*Figure 4: ProcMon showing the TCP network connections initiated by the malware.*

![Web panel](screenshots/Day%206/5.png)
*Figure 5: Web panel that HopHelper.exe is communicating with.*

## 🔗 References
* [TryHackMe - Advent of Cyber 2025](https://tryhackme.com/adventofcyber25)