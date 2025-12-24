# Day 2: Phishing - Merry Clickmas

## 📝 Scenario
In Day 2, the objective was to simulate an authorized **Red Team** engagement against "The Best Festival Company" (TBFC). The goal was to test the security awareness of employees by executing a **Social Engineering** campaign. 

Acting as a penetration tester, I was tasked with crafting a convincing phishing email, setting up a credential harvesting server, and successfully capturing user credentials to demonstrate the risk of employees clicking on malicious links.

## 🛠️ Tools Used
* **Social-Engineer Toolkit (SET):** An open-source penetration testing framework used for social engineering.
* **Python (`server.py`):** Used to host a fake login page and capture incoming credentials.
* **SMTP (Simple Mail Transfer Protocol):** Configured to deliver the spoofed emails to the target internal server.

## 🕵️‍♂️ What I Did
My operation focused on the human element of security: exploiting trust and urgency rather than software vulnerabilities.

1.  **Setting the Trap (Credential Harvesting):**
    I started by launching a Python script (`server.py`) acting as a malicious web server. This script served a fake login page on port 8000 and was configured to log any username and password entered by a victim.

2.  **Crafting the Payload (SET):**
    I utilized the **Social-Engineer Toolkit (setoolkit)** to orchestrate the email attack. I selected the "Mass Mailer Attack" vector to target a specific high-value user: `factory@wareville.thm`.

3.  **The Pretext (Social Engineering):**
    To make the email convincing, I spoofed the sender to appear as **"Flying Deer"** (a trusted shipping partner) with the address `updates@flyingdeer.thm`.
    * **Subject:** "Shipping Schedule Changes"
    * **Body:** I created a sense of urgency regarding increased orders and directed the user to my malicious link (`http://<AttackBox_IP>:8000`) to "confirm the schedule."

4.  **Harvesting & Verification:**
    Once the email was sent, I monitored my Python server. The victim fell for the lure and entered their credentials.
    * **Result:** I successfully captured the cleartext password.
    * **Post-Exploitation:** I used the stolen credentials to log into the actual TBFC email portal (`http://<MACHINE_IP>`) and accessed confidential information regarding toy delivery numbers.

## 🧠 Key Takeaway
I learned that even with secure infrastructure, the **human factor** remains a critical vulnerability. Phishing attacks rely on psychology—specifically authority and urgency—to bypass technical controls. Tools like SET make it trivially easy to spoof "trusted" senders, highlighting the importance of SPF/DKIM/DMARC records and employee training.

## 📸 Screenshots
*Below are screenshots documenting the attack chain and successful compromise.*

![Configuring the attack in SET](screenshots/Day%202/1.png)
*Figure 1: The Python server capturing the victim's password in cleartext.*

![Credential Capture](screenshots/Day%202/2.png)
*Figure 2: Using the stolen credentials to access the user's inbox and find the toy delivery count.*

## 🔗 References
* [TryHackMe - Advent of Cyber 2025](https://tryhackme.com/adventofcyber25)
* [Social-Engineer Toolkit (SET) Documentation](https://github.com/trustedsec/social-engineer-toolkit)