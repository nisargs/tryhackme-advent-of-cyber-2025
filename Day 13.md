# Day 13: YARA Rules - YARA mean one!

## 📝 Scenario
In Day 13, chaos has ensued at "The Best Festival Company" (TBFC) following McSkidy's disappearance. However, she managed to send a covert message hidden within a folder of Easter-themed images. These images contain a specific keyword sequence that needs to be extracted to decode her message.

As a member of the TBFC Blue Team, my objective was to create and deploy a **YARA rule** to scan the directory of images. The goal was to identify files containing the keyword "TBFC:" followed by a secret code word, extract these codes in order, and reveal McSkidy's hidden communication.

## 🛠️ Tools & Technologies Used
* **YARA:** A pattern-matching swiss-army knife for malware researchers, used here to identify specific text strings and regular expressions within image files.
* **Regular Expressions (Regex):** Used within the YARA rule to define flexible search patterns for the hidden codes.
* **Linux Command Line:** Used to execute the YARA tool and inspect the output.

## 🧠 Theoretical Concepts
* **YARA Rules:** A way to describe malware (or any file of interest) based on textual or binary patterns.
    * **Meta:** Metadata about the rule (Author, Description).
    * **Strings:** The specific patterns to look for (Text, Hex, or Regex).
    * **Condition:** The logic that determines when a rule matches (e.g., "String A and String B found").
* **Regex Modifiers:**
    * `[a-zA-Z0-9]`: Alphanumeric characters.
    * `+`: One or more occurrences.
* **Use Cases:** YARA is used for post-incident analysis, threat hunting, and memory forensics to find indicators of compromise (IOCs) that standard antivirus might miss.

## 🕵️‍♂️ Investigation Walkthrough

### Step 1: Identifying the Pattern
The brief stated that McSkidy's message follows the format: **"TBFC:"** followed by a **code word** (alphanumeric characters). I first needed to check the directory to confirm the presence of these strings in the image files.

### Step 2: Crafting the YARA Rule
To automate the extraction, I created a YARA rule file named `detect_message.yar`.

* **Rule Logic:**
    * **Strings Section:** I defined a regular expression string variable to capture the specific format. The regex used was `TBFC:[a-zA-Z0-9]+`. This pattern instructs YARA to match the literal characters "TBFC:" followed immediately by one or more alphanumeric characters (letters or numbers).
    * **Condition Section:** I set the condition to trigger the rule whenever this specific regex pattern was found in a file.

### Step 3: Executing the Scan
I ran YARA against the target directory (`/home/ubuntu/Downloads/easter`) using the following command flags:
* `-r`: To scan the directory recursively.
* `-s`: To print the specific matching strings to the terminal (crucial for reading the hidden code).

### Step 4: Decoding the Message
The output displayed the filenames and the specific strings they contained (e.g., `TBFC:W`, `TBFC:e`, etc.).
1.  **Analysis:** I noted the code words found in the output.
2.  **Assembly:** By reading the extracted characters in the order of the filenames, I reconstructed the hidden sentence sent by McSkidy.

## 🧠 Key Takeaways
* **Flexibility of YARA:** YARA isn't just for finding "bad" malware; it's a search tool for *any* pattern, making it useful for data recovery and forensics (like finding hidden messages).
* **Regex Power:** Using a regex pattern allowed me to find *variations* of the code word (different letters/numbers) without knowing exactly what they were beforehand.
* **The "Strings" Flag:** Running YARA with `-s` is crucial during investigation because seeing *what* triggered the rule is often more important than just knowing the rule triggered.

## 📸 Screenshots
*Below are the findings from the YARA scan.*

![YARA Rule Creation](screenshots/Day%2013/images%20with%20TBFC%20string.png)
*Figure 1: Finding the number of images with `TBFC` string.*

![YARA Scan Output](screenshots/Day%2013/yara%20rule%20to%20detect%20TBFC%20strings.png)
*Figure 2: Creating the .yar file with the regex pattern.*

![Decoded Message](screenshots/Day%2013/2.png)
*Figure 3: RegEx in YARA rule to match strings that begin with `TBFC:`.*

![Decoded Message](screenshots/Day%2013/3.png)
*Figure 4: Assembling the hidden characters to reveal McSkidy's message.*

## 🔗 References
* [TryHackMe - Advent of Cyber 2025](https://tryhackme.com/adventofcyber25)
* [YARA Documentation](https://yara.readthedocs.io/en/stable/)