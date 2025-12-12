# Day 11: XSS - Merry XSSMas

## 📝 Scenario
In Day 11, Santa's workshop has modernized its technology stack, including a new secure message portal for contacting McSkidy. However, logs indicate unusual activity, with search terms and messages containing suspicious scripts.

As a security analyst, my objective was to investigate the portal for **Cross-Site Scripting (XSS)** vulnerabilities. I needed to identify where user input was not being properly sanitized and demonstrate both **Reflected** and **Stored** XSS attacks to confirm the security flaws.

## 🛠️ Tools & Technologies Used
* **Web Browser:** Used to interact with the target web application (Chrome/Firefox within the AttackBox).
* **JavaScript Payloads:** Small snippets of code (`<script>alert(...)</script>`) used to test for execution vulnerability.
* **Developer Tools (Console/Network Tab):** Used to inspect how the application handles input and where it is reflected in the DOM.

## 🧠 Theoretical Concepts
* **Cross-Site Scripting (XSS):** A vulnerability where attackers inject malicious scripts into web pages viewed by other users.
* **Reflected XSS:** The malicious script comes from the current HTTP request (e.g., a search query). The attack is immediate and not stored on the server.
* **Stored XSS (Persistent XSS):** The malicious script is saved on the server (e.g., in a database via a comment or message form). The code runs every time *anyone* loads the affected page.
* **Sanitization/Encoding:** The defense mechanism of stripping dangerous characters (like `<` and `>`) or converting them to safe HTML entities (like `&lt;` and `&gt;`) so the browser treats them as text, not code.

## 🕵️‍♂️ Investigation Walkthrough

### Step 1: Testing for Reflected XSS
I began by testing the **Search Bar** on the portal. Reflected XSS often occurs in search functions where the user's query is displayed back to them on the results page.

1.  **Injection:** I entered a basic JavaScript payload into the search field:
    ```html
    <script>alert('Reflected XSS')</script>
    ```
    *Note: The actual task required a specific payload to reveal the flag.*
    ```html
    <script>alert(atob("VEhNe1JlZmxlY3RlZF9YU1NfRmxhZ30="))</script>
    ```
2.  **Execution:** Upon clicking "Search Messages," the browser executed the script instead of just displaying the text.
3.  **Result:** A pop-up alert appeared containing the first flag. This confirmed **Reflected XSS** because the script ran immediately but disappeared upon refreshing the page (unless the URL was visited again).

### Step 2: Testing for Stored XSS
Next, I investigated the **"Contact McSkidy"** message form. Since these messages are saved for McSkidy to read later, this is a prime target for Stored XSS.

1.  **Injection:** I navigated to the message form and entered a similar payload into the message body:
    ```html
    <script>alert('Stored XSS')</script>
    ```
    *Note: The actual task required a specific payload to reveal the flag.*
    ```html
    <script>alert(atob("VEhNe1N0b3JlZF9YU1NfRmxhZ30="))</script>
    ```
2.  **Execution:** I clicked "Send Message."
3.  **Result:** The page refreshed, and the alert popped up immediately. More importantly, every time I refreshed the page or navigated back to it, the alert popped up *again*. This confirmed **Stored XSS** because the malicious code was now permanently saved in the application's database and served to anyone viewing the page.

## 🧠 Key Takeaways
* **Input Validation is Critical:** "Never trust user input." Any data entering an application must be treated as potentially malicious.
* **Context Matters:** Reflected XSS targets the user who clicks a specific link (often via phishing). Stored XSS is more dangerous because it essentially "booby-traps" the page for everyone, including administrators.
* **Mitigation:** To fix this, developers should implement Output Encoding (converting special characters into HTML entities) and use secure headers like Content Security Policy (CSP) to restrict where scripts can run.

## 📸 Screenshots
*Below are the findings from the XSS investigation.*

![Reflected XSS Alert](screenshots/Day%2011/1.png)
*Figure 1: The alert box triggered by the Reflected XSS payload in the search bar.*

![Stored XSS Persistence](screenshots/Day%2011/2.png)
*Figure 2: The Stored XSS payload executing automatically upon reloading the page.*

## 🔗 References
* [TryHackMe - Advent of Cyber 2025](https://tryhackme.com/adventofcyber25)