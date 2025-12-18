# Day 18: Obfuscation - The Egg Shell File

## 📝 Scenario
In Day 18, WareVille is in chaos following the appearance of a wormhole. McSkidy's attention was drawn to a suspicious email claiming to be from "North Pole HR"—a department that doesn't exist (TBFC's HR is at the South Pole). The email contained a PowerShell script (`SantaStealer.ps1`) filled with unreadable "gibberish."

As a security analyst, my objective was to analyze this file to understand the concept of **Obfuscation**. I needed to de-obfuscate the script to reveal the attacker's Command & Control (C2) server and then use my knowledge of XOR operations to trick the script into revealing a hidden flag.

## 🛠️ Tools & Technologies Used
* **CyberChef:** A web-based "Swiss Army Knife" for data manipulation, used to understand and perform encoding, encryption, and obfuscation operations (Base64, XOR, ROT13).
* **PowerShell:** The scripting language used by the attacker, and the tool I used to execute the de-obfuscated code.
* **Visual Studio Code:** Used to view and edit the malicious script.

## 🧠 Theoretical Concepts
* **Obfuscation:** The practice of making code difficult for humans and security tools to read without changing its functionality. It is "security through obscurity."
* **Encoding vs. Encryption:**
    * **Encoding:** Transforming data for usability (e.g., Base64). No key is needed to reverse it.
    * **Encryption:** transforming data for confidentiality. A secret key is required to reverse it.
    * **Obfuscation:** Intentionally confusing code (e.g., renaming variables to random strings, adding junk code).
* **XOR (Exclusive OR):** A bitwise operation often used in malware. It is reversible: if you XOR data with a key, and then XOR the result with the same key, you get the original data back.

## 🕵️‍♂️ Investigation Walkthrough

### Step 1: Initial Analysis & De-obfuscation
I opened the malicious file `SantaStealer.ps1` located on the Desktop. The script contained a section labeled "Start here" with an obfuscated string representing the attacker's C2 server.

1.  **Action:** I followed the instructions within the script's comments. This involved uncommenting specific lines that contained the de-obfuscation logic (likely a function designed to reverse the obfuscation).
2.  **Execution:** I ran the script in PowerShell:
    ```powershell
    cd .\Desktop\
    .\SantaStealer.ps1
    ```
3.  **Result:** The script decoded the C2 URL and printed the first flag to the console.

### Step 2: Obfuscating the API Key (XOR)
The second part of the challenge required me to "hack back" or manipulate the script by obfuscating a specific API key so the malware would accept it.

1.  **Concept:** The script used **XOR** logic. I needed to take the plaintext API key (likely found in the comments or context) and obfuscate it using the same method the attacker used.
    * *Tip:* I could use CyberChef to perform the XOR operation or rely on the logic provided within the PowerShell script itself.
2.  **Action:** I applied the XOR encoding to the API key.
    
3.  **Modification:** I updated the `SantaStealer.ps1` script, replacing the placeholder or old key with my newly obfuscated byte string.
4.  **Execution:** I ran the script again.
    ```powershell
    .\SantaStealer.ps1
    ```
5.  **Result:** The script accepted the obfuscated key and revealed the second flag.

## 🧠 Key Takeaways
* **Obfuscation is not Security:** Hiding a string (like a password or URL) using Base64 or XOR does not secure it; it only slows down analysis.
* **Reversibility of XOR:** XOR is a fundamental building block of cryptography and malware evasion because it is computationally cheap and easily reversible (`A XOR B = C` implies `C XOR B = A`).
* **Layered Defense:** Attackers often layer these techniques (e.g., Base64 encoded inside a Gzip archive, which is then XOR'd) to bypass automated detection signatures.

## 📸 Screenshots
*Below are the findings from the obfuscation analysis.*

![De-obfuscation Output](screenshots/Day%2018/1.png)
*Figure 1: Deobfuscating the C2 URL in CyberChef.*

![CyberChef XOR](screenshots/Day%2018/2.png)
*Figure 2: Retrieving the first flag.*

![Final Flag](screenshots/Day%2018/3.1.png)
*Figure 3: Using CyberChef to XOR the API key.*

![Final Flag](screenshots/Day%2018/3.2.png)
*Figure 3: Successfully running the modified script to retrieve the second flag.*

## 🔗 References
* [TryHackMe - Advent of Cyber 2024](https://tryhackme.com/adventofcyber24)
* [CyberChef](https://gchq.github.io/CyberChef/)

## 🔗 References
* [TryHackMe - Advent of Cyber 2025](https://tryhackme.com/adventofcyber25)
* [CyberChef](https://gchq.github.io/CyberChef/)