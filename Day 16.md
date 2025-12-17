# Day 16: Forensics - Registry Furensics

## 📝 Scenario
In Day 16, TBFC is dealing with the absence of McSkidy and a compromise of their critical system, `dispatch-srv01`, which coordinates drone-based gift delivery. The server has been targeted by King Malhare's bunnies.

As a member of the forensic team, my objective was to perform **Registry Forensics**. Unlike live analysis, I worked with captured **Registry Hives** from the compromised machine to identify what software was installed, where it was executed from, and how it established persistence on the system.

## 🛠️ Tools & Technologies Used
* **Windows Registry:** The hierarchical database that stores configuration settings and options for the Microsoft Windows operating systems.
* **Registry Editor (regedit):** The built-in Windows tool for viewing the live registry (used for understanding concepts).
* **Registry Explorer:** A forensic tool by Eric Zimmerman used to analyze offline registry hives. It handles "dirty" hives (those with uncommitted transaction logs) and decodes binary data into readable formats.

## 🧠 Theoretical Concepts
* **Registry Hives:** The registry is split into separate files (Hives) stored on the disk.
    * **SYSTEM:** Boot config, services, drivers (`C:\Windows\System32\config\SYSTEM`).
    * **SOFTWARE:** Installed programs and settings (`C:\Windows\System32\config\SOFTWARE`).
    * **SAM:** User accounts and password hashes (`C:\Windows\System32\config\SAM`).
    * **NTUSER.DAT:** User-specific settings like recent files and browsing history (`C:\Users\<user>\NTUSER.DAT`).
* **Root Keys:** The logical view of the registry (e.g., `HKEY_LOCAL_MACHINE` or HKLM).
* **Persistence:** Techniques used by malware to restart automatically after a reboot, often found in `Run` or `RunOnce` registry keys.

## 🕵️‍♂️ Investigation Walkthrough

### Step 1: Loading "Dirty" Hives
I launched **Registry Explorer** and loaded the captured hives (`SYSTEM`, `SOFTWARE`, `NTUSER.DAT`) from the `C:\Users\Administrator\Desktop\Registry Hives` directory.
* **Crucial Step:** When loading the hives, I held `SHIFT` to load the associated **Transaction Logs** (`.log` files). This replays uncommitted data to ensure the hive is "clean" and up-to-date, preventing data loss during analysis.

### Step 2: Identifying the Malicious Application
To find what was installed around the time of the attack (October 21st, 2025), I examined the **SOFTWARE** hive.
1.  **Path:** `ROOT\Microsoft\Windows\CurrentVersion\Uninstall`
2.  **Analysis:** I looked through the subkeys for suspicious entries. I found an application named **Grinchmas** (or similar, depending on the specific lab instance) installed on the suspicious date.

### Step 3: Tracking Execution (UserAssist)
To find where this application was launched from, I examined the **NTUSER.DAT** hive, which tracks user behavior.
1.  **Path:** `ROOT\Software\Microsoft\Windows\CurrentVersion\Explorer\UserAssist\{GUID}\Count`
2.  **Concept:** `UserAssist` tracks GUI-based program execution. The entries are typically ROT-13 encoded, but **Registry Explorer** automatically decodes them.
3.  **Result:** I found an entry pointing to the malware's location on the Desktop (e.g., `C:\Users\Administrator\Desktop\Grinchmas.exe`).

### Step 4: Finding Persistence
Finally, I checked how the malware ensured it would stay running.
1.  **Path:** `ROOT\Microsoft\Windows\CurrentVersion\Run` (in the **SOFTWARE** hive).
2.  **Concept:** Programs listed here start automatically when the computer boots.
3.  **Result:** I found a value added by the malicious application, pointing to its executable, ensuring it ran on every startup.

## 🧠 Key Takeaways
* **The Registry Remembers:** Even if a file is deleted, the Registry often holds traces of its existence, execution, and installation path.
* **Offline Analysis is Safer:** Analyzing captured hives prevents accidental modification of the evidence and avoids triggering malware that might be monitoring access to its own keys.
* **Transaction Logs Matter:** Ignoring the `.log` files when analyzing registry hives can lead to missing the most recent (and often most critical) evidence.

## 📸 Screenshots
*Below are the findings from the Registry Explorer investigation.*

![Registry Explorer Load](screenshots/Day%2016/1.png)
*Figure 1: The application that was installed on the dispatch-srv01 before the abnormal activity started.*

![Uninstall Key](screenshots/Day%2016/2.png)
*Figure 2: The full path where the user launched the application from.*

![UserAssist Execution](screenshots/Day%2016/3.png)
*Figure 3: The value that was added by the application to maintain persistence on startup.*

## 🔗 References
* [TryHackMe - Advent of Cyber 2025](https://tryhackme.com/adventofcyber25)