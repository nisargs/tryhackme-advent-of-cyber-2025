# Day 5: IDOR - Santa’s Little IDOR

## 📝 Scenario
In Day 5, the investigation shifted to the "TryPresentMe" website. The support team reported users attempting to activate vouchers were facing errors, while others received targeted phishing emails. A suspicious account named **Sir Carrotbane** was flagged for hoarding vouchers.

My objective was to investigate the web application for **Insecure Direct Object Reference (IDOR)** vulnerabilities. The goal was to understand how the application fetches user data and exploit weak access controls to access unauthorized accounts (Horizontal Privilege Escalation).

## 🛠️ Tools & Technologies Used
* **Web Browser Developer Tools:** Specifically the **Network** tab (to view API requests) and **Local Storage** (to manipulate session data).
* **Burp Suite (Optional):** For automating the enumeration of IDs (Intruder).
* **Data Decoders:** Base64 and MD5 hash analyzers to understand how IDs were obfuscated.

## 🧠 Theoretical Concepts
Before exploiting the vulnerability, I analyzed the core concepts:
* **IDOR (Insecure Direct Object Reference):** A vulnerability where an application uses user-supplied input (like an ID number) to access objects directly without checking if the user is *authorized* to see them.
* **Authentication vs. Authorization:**
    * *Authentication:* Verifying who you are (Logging in).
    * *Authorization:* Verifying what you are allowed to do (Access control).
* **Horizontal Privilege Escalation:** Accessing data of other users with the *same* privilege level (e.g., User A accessing User B's profile).

## 🕵️‍♂️ Investigation Walkthrough

### Step 1: Traffic Analysis
I logged into the application using the provided credentials (`niels` / `TryHackMe#2025`).
I opened the Browser Developer Tools, navigated to the **Network** tab, and refreshed the page.

I observed a specific request to `view_accountinfo`.
* **Observation:** The application was retrieving user data based on a numeric ID found in the client-side storage, not based on a secure session token on the server.

### Step 2: Identification of the Vector
I navigated to the **Storage** tab (or Application tab) in Developer Tools and expanded **Local Storage**. I found a key named `user_id` with a value of `10`.

### Step 3: Exploitation (Horizontal Escalation)
To test for IDOR, I modified the `user_id` value from `10` to `11`.
After refreshing the page, the application loaded the profile of a completely different user. This confirmed the IDOR vulnerability: the server was not verifying if "Niels" was authorized to view User #1.

### Step 4: Hunting for the Target
I iterated through various User IDs (changing the value to 12, 13, 14, etc.) to find the specific account requested in the challenge (the parent with 10 children).
* **Result:** Upon setting the ID to a specific number (found during enumeration), I accessed a profile that revealed the flag.

### Step 5: Advanced IDOR (Obfuscation)
I also noted that some endpoints used encoded IDs.
* **Base64:** Some IDs were passed as `Mg==`. Decoding this reveals it is simply the number `2`.
* **Hashing:** Other endpoints used MD5 hashes.
* **Takeaway:** Encoding (Base64) is not encryption. It does not secure an IDOR; it only hides it from casual view.

## 🧠 Key Takeaways
* **Authorization is Mandatory:** It is not enough to log a user in; the server must check permissions for *every* specific object requested.
* **Obfuscation != Security:** Hiding an ID by encoding it in Base64 or hashing it does not fix the vulnerability. Attackers can easily decode or replay these values.
* **Trust Boundary:** Never trust input coming from the client side (like Local Storage or URL parameters) for access control decisions.

## 📸 Screenshots
*Below are the findings from the IDOR investigation.*

![The Target Profile](screenshots/Day%205/1.png)
*Figure 1: Successfully accessing the target account (Parent with 10 children) by exploiting the IDOR.*

## 🔗 References
* [TryHackMe - Advent of Cyber 2025](https://tryhackme.com/adventofcyber25)
* [OWASP - Broken Access Control](https://owasp.org/Top10/A01_2021-Broken_Access_Control/)