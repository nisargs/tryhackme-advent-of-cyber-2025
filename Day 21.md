# Day 21: Malware Analysis - Malhare.exe

## 📝 Scenario
In Day 21, the SOC team at Wareville discovered a suspicious file targeting the elves. The common denominator among the compromised hosts was a "Salary Survey" email containing an attachment named `survey.hta`.

As a security analyst, my objective was to perform **Malware Analysis** on this HTML Application (HTA) file. I needed to reverse-engineer the code to understand how it tricked users, identifying the metadata, analyzing the embedded VBScript functions, uncovering network indicators (C2 domains), and determining how data was exfiltrated and payloads executed.

## 🛠️ Tools & Technologies Used
* **HTA (HTML Application):** The file format being analyzed, which combines HTML with Windows-native scripting capabilities (VBScript/JScript).
* **Text Editor (Pluma/Notepad++):** Used to perform static analysis by reading the source code of the HTA file without executing it.
* **CyberChef:** Used to decode obfuscated payloads (Base64) and decrypt encrypted strings.
* **Powershell/VBScript:** The scripting languages embedded within the malware.

## 🧠 Theoretical Concepts
* **Living-off-the-land (LotL):** The technique of using legitimate, pre-installed system tools (like `mshta.exe`, `powershell.exe`, or `wscript.exe`) to conduct attacks, avoiding the need to drop new binary files that might be detected by antivirus.
* **HTA Files:** Files that run via `mshta.exe`. They are dangerous because they run outside the browser's security sandbox, giving the script full access to the operating system (e.g., file system, registry, command execution).
* **Obfuscation:** Hiding the intent of the code using encoding (Base64) or complex naming conventions.
* **Typosquatting:** Registering a domain name that looks very similar to a legitimate one (e.g., `hum-resources.com` vs `human-resources.com`) to trick users.

## 🕵️‍♂️ Investigation Walkthrough

### Step 1: Metadata & Static Analysis
I began by opening the `survey.hta` file in a text editor to inspect the code safely.
1.  **Command:** `pluma /root/Rooms/AoC2025/Day21/survey.hta`
2.  **Metadata:** I examined the `<head>` section and the `HTA:APPLICATION` tag to see how the malware presented itself to the user (e.g., Title: "Salary Survey").

### Step 2: VBScript Logic Analysis
I analyzed the `<script type="text/vbscript">` block to understand the program flow. I identified several key functions:
* `window_onLoad`: The entry point that runs automatically when the file is opened.
* `getQuestions()`: A function that appeared to fetch survey questions but was actually a downloader for the next stage of the attack.
* `provideFeedback()`: A function designed to exfiltrate data.

### Step 3: Network Indicators (Typosquatting)
Inside the `getQuestions` function, I found the URL used to download the "questions."
1.  **Domain Analysis:** The domain used was `http://hum-resources.com`.
2.  **Typosquatting:** I noticed the character `m` in "hum" was used to mimic the legitimate "human-resources" domain, a technique designed to deceive users who glance quickly at the URL.

### Step 4: Exfiltration & Reconnaissance
I examined the `provideFeedback` function to see what data was being stolen.
1.  **Reconnaissance:** The script utilized `WScript.Network` to collect specific system information:
    * `ComputerName`
    * `UserName`
2.  **Exfiltration:** This data was packaged and sent via an HTTP `POST` request to a specific endpoint (e.g., `/feedback`) on the attacker's server.

### Step 5: Payload Execution & Decryption
The malware didn't just display questions; it executed the downloaded content.
1.  **Execution:** I found the specific line `Execute(downloaded_content)`, which takes the text downloaded from the C2 server and runs it as code on the victim's machine.
2.  **Payload Extraction:** Since the C2 server was down, I analyzed a captured copy of the payload.
3.  **De-obfuscation:**
    * **Encoding:** The payload was encoded in **Base64**. I decoded this using CyberChef.
    * **Encryption:** The decoded output revealed a secondary layer of protection. By analyzing the script headers and key references, I identified the encryption scheme (e.g., **AES**).
4.  **Flag Retrieval:** Using CyberChef to decrypt the payload with the found key, I revealed the final flag.

## 🧠 Key Takeaways
* **HTA Files are Executables:** Despite looking like web pages, HTA files are fully functional applications. Opening an HTA file is as dangerous as running an `.exe`.
* **Static Analysis First:** Always open suspicious scripts in a text editor before doing anything else. Running them (even to "test") can compromise your analysis machine.
* **Typosquatting Awareness:** Attackers rely on our brain's "autocorrect" feature. Carefully inspecting domains letter-by-letter is crucial in verifying legitimacy.

## 📸 Screenshots
*Below are screenshots corresponding to the answers for the Day 21 task questions.*

![HTA Application Title](screenshots/Day%2021/1.png)
*Figure 1: The source code showing the metadata title of the HTA application.*

![Download Function](screenshots/Day%2021/2.png)
*Figure 2: The VBScript function acting as the downloader for the "questions".*

![Malicious URL](screenshots/Day%2021/3.png)
*Figure 3: The full URL domain found within the script used for downloading content.*

![Typosquatting Character](screenshots/Day%2021/4.png)
*Figure 4: Highlighting the specific character in the domain used for typosquatting.*

![Survey Questions count](screenshots/Day%2021/5.png)
*Figure 5: The HTML body revealing the number of fake survey questions.*

![Social Engineering Prize](screenshots/Day%2021/6.png)
*Figure 6: The text indicating the fake prize destination used for social engineering.*

![Data Exfiltration Targets](screenshots/Day%2021/7.png)
*Figure 7: The code showing which two pieces of host information are being enumerated.*

![Exfiltration Endpoint](screenshots/Day%2021/8.png)
*Figure 8: The specific endpoint on the C2 server where stolen data is sent.*

![HTTP Method](screenshots/Day%2021/9.png)
*Figure 9: The line of code defining the HTTP method used for exfiltration.*

![Execution Command](screenshots/Day%2021/10.png)
*Figure 10: The specific line of code that executes the downloaded content.*

![Payload Encoding](screenshots/Day%2021/11.png)
*Figure 11: The downloaded payload string showing the initial encoding scheme used.*

![Encryption Scheme](screenshots/Day%2021/12.png)
*Figure 12: Script analysis revealing the underlying encryption scheme of the payload.*

![Final Decrypted Flag](screenshots/Day%2021/13.png)
*Figure 13: The final flag value after fully decoding and decrypting the payload in CyberChef.*

## 🔗 References
* [TryHackMe - Advent of Cyber 2025](https://tryhackme.com/adventofcyber25)