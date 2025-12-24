# Day 9: Passwords - A Cracking Christmas

## 📝 Scenario
In Day 9, traces of encrypted data labeled "North Pole Asset List" were discovered deep within "The Best Festival Company" servers. Sir Carrotbane has been tasked with accessing these locked PDF and ZIP files to determine if they contain fragments of Santa's master gift registry.

As a security analyst, my objective was to demonstrate how weak passwords expose encrypted files. I utilized **dictionary attacks** to recover the passwords for both a protected PDF and a ZIP archive, revealing the secrets hidden within.

## 🛠️ Tools & Technologies Used
* **John the Ripper (John):** A fast password cracker, currently available for many flavors of Unix, Windows, DOS, and OpenVMS. Used here to crack the ZIP file hash.
* **Pdfcrack:** A standalone tool specifically designed to recover passwords and content from PDF files.
* **Zip2john:** A utility to convert ZIP archives into a hash format that John the Ripper can understand.
* **Rockyou.txt:** A famous wordlist containing millions of real-world common passwords, used for the dictionary attack.

## 🧠 Theoretical Concepts
* **Dictionary Attack:** A method of breaking into a password-protected computer or server by systematically entering every word in a dictionary as a password. It is fast but only works if the password is in the list.
* **Brute-Force Attack:** A trial-and-error method used to obtain information such as a user password or personal identification number (PIN). It tries *every possible combination* of characters.
* **Hashing vs. Encryption:**
    * **Encryption** (like the PDF/ZIP) is two-way; data is scrambled but can be restored with the key.
    * **Hashing** is one-way; data is scrambled into a fixed string and cannot be reversed (easily). Attackers hash their guesses and compare them to the target hash.

## 🕵️‍♂️ Investigation Walkthrough

### Step 1: Preparation
I began by navigating to the Desktop directory on the target machine to locate the intercepted files. I confirmed the presence of two encrypted files: `flag.pdf` and `flag.zip`.

### Step 2: Cracking the PDF
To access the "North Pole Asset List" inside the PDF, I used the **pdfcrack** tool. Since the user likely used a weak password, I launched a dictionary attack by pointing the tool to the protected PDF file and specifying the `rockyou.txt` wordlist.

**Result:** The tool successfully iterated through the wordlist and found the correct password. Using this credential, I was able to open the PDF and read the content to find the first flag.

### Step 3: Cracking the ZIP
The ZIP file required a two-step approach using **John the Ripper**.
1.  **Extract Hash:** First, I used the **zip2john** utility to extract the password hash from the zip file and saved it into a text file. This converted the encryption data into a format that John could process.
2.  **Crack Hash:** I then ran **John** against this hash file, using the `rockyou.txt` wordlist.

**Result:** John successfully cracked the hash and displayed the plaintext password. I used this password to unzip the archive and access the `flag.txt` file contained within, revealing the second flag.

## 🧠 Key Takeaways
* **Complexity Matters:** Both files were cracked in seconds because the passwords were simple and existed in a standard wordlist. A complex password (combining length, special characters, and randomness) would have made this dictionary attack impossible.
* **Tool Specificity:** Different file types (PDF vs. ZIP) store encryption data differently. We need specific tools (`pdfcrack` vs. `zip2john`) to extract the necessary data for cracking.
* **The Human Factor:** The weakest link in encryption is often the human who sets the password, not the math behind the encryption itself.

## 📸 Screenshots
*Below are the findings from the password cracking operation.*

![PDF Cracking](screenshots/Day%209/1.png)
*Figure 1: flag inside the encrypted PDF.*

![Unzipping the Flag](screenshots/Day%209/2.png)
*Figure 2: Unzipping the archive and reading the final flag.*

## 🔗 References
* [TryHackMe - Advent of Cyber 2025](https://tryhackme.com/adventofcyber25)