# Day 17: CyberChef - Hoperation Save McSkidy

## 📝 Scenario
In Day 17, McSkidy is imprisoned in King Malhare's "Quantum Warren." To secure an escape route, five digital locks must be disabled. McSkidy has sent clues indicating that the locks can be broken by analyzing the system's logic and interacting with the guards via a built-in chat interface.

As a security analyst, my objective was to use **CyberChef** to decode hidden messages, analyze web page headers and JavaScript logic, and reverse-engineer the encoding schemes used by the locks to retrieve the passwords and free McSkidy.

## 🛠️ Tools & Technologies Used
* **CyberChef:** The "Cyber Swiss Army Knife" used for encoding, decoding, hashing, and data transformation.
* **Web Developer Tools:** Specifically the **Network** tab (to inspect HTTP headers) and the **Debugger/Sources** tab (to view client-side JavaScript logic).
* **Base64:** The primary encoding scheme used for obfuscation in this challenge.
* **XOR / ROT13 / MD5:** Various cryptographic primitives and hashing algorithms encountered during the lock-breaking process.
* **CrackStation:** An online rainbow table resource used to reverse MD5 hashes.

## 🧠 Theoretical Concepts
* **Encoding vs. Encryption:**
    * **Encoding (e.g., Base64):** Transforms data formats for compatibility/usability. No key is required to reverse it.
    * **Encryption (e.g., AES):** Scrambles data for confidentiality. Requires a key to decrypt.
* **Hashing (e.g., MD5):** A one-way function that produces a fixed-size fingerprint of data. While technically irreversible, weak hashes like MD5 can be "cracked" using pre-computed databases (rainbow tables).
* **Client-Side Logic:** Security controls implemented in JavaScript running on the user's browser are inherently insecure because the user can inspect and modify the code.

## 🕵️‍♂️ Investigation Walkthrough

### Step 1: First Lock - Outer Gate
1.  **Reconnaissance:** I inspected the **Network** tab and found a "Magic Question" in the HTTP headers.
2.  **Interaction:** I Base64 encoded the guard's name (found in the chat) and the magic question to interact with the guard. The guard responded with a Base64 string.
3.  **Decoding:** The **Debugger** tab revealed the logic was simple Base64. I decoded the guard's response to get the password.

### Step 2: Second Lock - Outer Wall
1.  **Logic Analysis:** The JavaScript source code showed that the encoding was applied *twice*.
2.  **Recipe:** I configured CyberChef to run `From Base64` -> `From Base64`.
3.  **Result:** This double-decoding revealed the plaintext password.

### Step 3: Third Lock - Guard House
1.  **Logic Analysis:** The script indicated the password was **XOR'ed** with a specific key and then Base64 encoded.
2.  **Interaction:** I asked the guard "Password please." to get the encoded string.
3.  **Recipe:**
    * `From Base64`
    * `XOR` (Key: [Found in source code])
4.  **Concept:** XOR is reversible. Applying the same key to the ciphertext returns the plaintext.

### Step 4: Fourth Lock - Inner Castle
1.  **Logic Analysis:** The script showed the password was hashed using **MD5**.
2.  **Interaction:** The guard provided the hash directly.
3.  **Cracking:** Since MD5 is one-way, I couldn't "decode" it in CyberChef. Instead, I used **CrackStation** to lookup the hash in a database, which revealed the original password.

### Step 5: Fifth Lock - Prison Tower
1.  **Logic Analysis:** This lock had dynamic logic based on a "Recipe ID" found in the HTTP headers.
2.  **Recipe Construction:** I checked the header (e.g., `Recipe ID: 3`) and matched it to the correct reverse logic provided in the hint.
    * *Example (Recipe 3):* `ROT13` -> `From Base64` -> `XOR`.
3.  **Final Unlock:** Executing this specific chain in CyberChef revealed the final password and the flag.

## 🧠 Key Takeaways
* **Client-Side Security is Weak:** Relying on JavaScript to hide passwords or keys is futile because the attacker (the user) has full access to the code.
* **CyberChef is Essential:** Understanding how to chain operations (Input -> Decode -> XOR -> Output) is a fundamental skill for analyzing obfuscated malware or data.
* **Hashes vs. Encoding:** It is critical to recognize the difference. You *decode* Base64; you *crack* or *lookup* MD5.

## 📸 Screenshots
*Below are the findings from the CyberChef operations.*

![CyberChef Recipe](screenshots/Day%2017/1.png)
*Figure 1: First lock unlocked.*

![CyberChef Recipe](screenshots/Day%2017/2.png)
*Figure 2: Second lock unlocked.*

![CyberChef Recipe](screenshots/Day%2017/3.png)
*Figure 3: Third lock unlocked.*

![Developer Tools Logic](screenshots/Day%2017/4.png)
*Figure 4: Fourth lock unlocked.*

![CrackStation MD5](screenshots/Day%2017/5.png)
*Figure 5: Password for the fifth lock.*

![CrackStation MD5](screenshots/Day%2017/6.png)
*Figure 6: Retrieving the final flag.*

## 🔗 References
* [TryHackMe - Advent of Cyber 2025](https://tryhackme.com/adventofcyber25)
* [CyberChef](https://gchq.github.io/CyberChef/)