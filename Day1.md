# Day 1: Linux CLI - Shells Bells

## 📝 Scenario
In Day 1, the objective was to perform a forensic investigation on a potentially compromised Linux server (tbfc-web01). The goal was to utilize the command-line interface (CLI) to navigate the filesystem, analyze system logs for unauthorized access attempts, and identify malicious scripts left behind by the attacker.

## 🛠️ Tools Used
* **Linux CLI Utilities:** `ls`, `cat`, `grep`, `find`, `history`.
* **Privilege Escalation:** `sudo su` to access root artifacts.

## 🕵️‍♂️ What I Did
My investigation focused on navigating the filesystem in Linux terminal to uncover hidden tracks left by the attacker.

1.  **Hidden File Discovery:**
    I started by enumerating the file system. Using `ls -la`, I uncovered a hidden file named `.guide.txt` inside the `Guides` directory, which provided initial hints about "Eggsploits."

2.  **Log Analysis:**
    I navigated to `/var/log` and used `grep` to filter `auth.log` for "Failed password" events. This revealed that the user **socmas** was being brute-forced from an external IP related to "HopSec."

3.  **Malware Identification:**
    Suspecting the `socmas` account was compromised, I used the `find` command to search for suspicious files containing the string "egg". I located a malicious script named `eggstrike.sh`, which was designed to delete legitimate Christmas wishlists and replace them with fake ones.

4.  **Forensic Reconstruction:**
    Finally, I escalated privileges to the **root** user (`sudo su`) and analyzed the `.bash_history` file. This revealed the exact commands the attacker executed, including a `curl` command used to exfiltrate data, which contained the final flag.

## 🧠 Key Takeaway
I learned that in Linux, the **Bash History** (`.bash_history`) is often the "smoking gun" in an investigation. Attackers who fail to clean their history leave behind a precise roadmap of their actions, including data exfiltration destinations and modified files.

## 📸 Screenshots
*Below are screenshots documenting the key findings during the investigation.*

![Finding the hidden guide file using ls -la](images/Day%201/1.png)
*Figure 1: Locating the hidden .guide.txt file.*

![Analyzing the auth.log](images/Day%201/2.png)
*Figure 2: Analyzing the eggstrike.sh script to retrieve the second flag.*

![Malicious script content](images/Day%201/3.png)
*Figure 3: Inspecting the Root user's .bash_history to recover the final flag.*

## 🔗 References
* [TryHackMe - Advent of Cyber 2025](https://tryhackme.com/adventofcyber25)