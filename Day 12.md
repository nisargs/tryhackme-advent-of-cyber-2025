# Day 12: Phishing - Phishmas Greetings

## 📝 Scenario
In Day 12, the "Email Protection Platform" at The Best Festival Company (TBFC) has gone offline due to the ongoing attacks by Malhare. With automated filters down, the Incident Response Task Force must manually triage a flood of incoming emails.

As a security analyst, my objective was to differentiate between **Legitimate** emails, harmless **Spam**, and malicious **Phishing** attempts sent by the Eggsploit Bunnies. I had to analyze email headers, sender domains, and message bodies to identify common attack techniques like spoofing, punycode, and social engineering.

## 🛠️ Tools & Technologies Used
* **Email Client Interface:** A simulated inbox used to view and analyze incoming messages.
* **Header Analysis:** Inspecting the raw metadata of emails (From, Return-Path, Authentication-Results) to detect spoofing.
* **URL & Domain Inspection:** Manually verifying links for typosquatting and punycode characters.

## 🧠 Theoretical Concepts
* **Phishing vs. Spam:**
    * **Spam:** Unsolicited, bulk digital noise (marketing, scams) aiming for volume/clicks.
    * **Phishing:** Targeted, malicious communication aiming to steal credentials, deliver malware, or commit fraud.
* **Impersonation & Social Engineering:** Tactics that manipulate psychology (urgency, authority) to trick users into acting without thinking (e.g., "Urgent Wire Transfer").
* **Spoofing & Authentication:**
    * **Spoofing:** Faking the "From" address to look like a legitimate domain.
    * **Defense (SPF/DKIM/DMARC):** Protocols that verify if a sender is actually authorized to send email on behalf of a domain.
* **Typosquatting & Punycode:**
    * **Typosquatting:** Slight misspellings in domains (e.g., `glthub.com` vs `github.com`).
    * **Punycode:** Using non-ASCII characters (like Cyrillic) that look identical to Latin letters to fake a domain (e.g., `micrоsoft.com`).

## 🕵️‍♂️ Investigation Walkthrough

### Step 1: Identifying Impersonation (The "Gmail" CEO)
I analyzed an email claiming to be from McSkidy asking for VPN access.
1.  **Observation:** The display name was "McSkidy," but the actual email address was `@gmail.com`.
2.  **Social Engineering:** The email used high-pressure language ("URGENT", "immediately") to force a quick reaction.
3.  **Verdict:** **Phishing**. Legitimate internal requests would come from a corporate domain, not a free Gmail account.

### Step 2: Detecting Punycode (The "Christmas Upgrade")
I reviewed an email regarding a laptop upgrade agreement.
1.  **Observation:** The sender domain looked like `tbfc-it.com`, but closer inspection revealed the letter `f` was actually a special character `ƒ` (Punycode).
2.  **Header Check:** Looking at the `Return-Path` header confirmed the ACE (ASCII Compatible Encoding) string, proving it was a different domain entirely.
3.  **Verdict:** **Phishing**. This was a classic homograph attack using Punycode.

### Step 3: Analyzing Headers for Spoofing (The "Voice Message")
I examined an email claiming to contain a voice message from McSkidy.
1.  **Observation:** The "From" address *looked* legitimate.
2.  **Deep Dive:** I checked the `Authentication-Results` header.
    * **SPF/DKIM/DMARC:** All Failed.
    * **Return-Path:** The email actually originated from `zxwsedr@easterbb.com`.
3.  **Attachment:** The file was an `.html` file (web page), a common technique to bypass sandboxes and launch scripts.
4.  **Verdict:** **Phishing**. The email was spoofed.

### Step 4: Classifying Spam (The "Logistics Offer")
I reviewed a generic marketing email offering event logistics.
1.  **Observation:** The content was generic advertising. It did not ask for credentials, demand urgent action, or contain suspicious links/attachments.
2.  **Verdict:** **Spam**. While annoying, it posed no security threat to the organization.

## 🧠 Key Takeaways
* **Headers Don't Lie:** The "From" address is easily faked (Spoofing). Always check the `Return-Path` and authentication results (SPF/DMARC) in the headers to verify the true sender.
* **Look for the "Human" Element:** Phishing attacks target people, not just computers. High urgency, appeals to authority ("CEO wants this"), and requests to bypass procedure are major red flags.
* **Watch Your URLs:** Modern phishing often uses legitimate services (like Dropbox or OneDrive) to host malicious links, or uses Punycode to make fake domains look real.

## 📸 Screenshots
*Below are the findings from the email triage.*

![Impersonation Example](screenshots/Day%2012/1.png)
*Figure 1: Flag from email 1.*

![Punycode Detection](screenshots/Day%2012/2.png)
*Figure 2: Flag from email 2.*

![Header Analysis](screenshots/Day%2012/3.png)
*Figure 3: Flag from email 3.*

![Header Analysis](screenshots/Day%2012/4.png)
*Figure 4: Flag from email 4.*

![Header Analysis](screenshots/Day%2012/5.png)
*Figure 5: Flag from email 5.*

![Header Analysis](screenshots/Day%2012/6.png)
*Figure 6: Flag from email 6.*

## 🔗 References
* [TryHackMe - Advent of Cyber 2025](https://tryhackme.com/adventofcyber25)