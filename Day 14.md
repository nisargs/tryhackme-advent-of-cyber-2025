# Day 14: Containers - DoorDasher's Demise

## 📝 Scenario
In Day 14, "DoorDasher," the local food delivery service in Wareville, has been hacked by King Malhare and rebranded as "Hopperoo." The site is serving "Santa's Beard Pasta," causing chaos. A security engineer attempted to fix it but was locked out, leaving a recovery script inside the system.

As a SOC member, my objective was to leverage access to the **uptime-checker** container to escape its isolation, pivot to the privileged **deployer** container, and execute the recovery script to restore the website.

## 🛠️ Tools & Technologies Used
* **Docker:** The container engine used to host the web application and supporting microservices.
* **Container Runtime Socket (`docker.sock`):** The Unix socket that allows communication between the Docker CLI client and the Docker Daemon.
* **Shell (sh/bash):** Used to interact with the containers.

## 🧠 Theoretical Concepts
* **Containers vs. Virtual Machines:**
    * **VMs:** Heavyweight, run a full Guest OS on top of a hypervisor.
    * **Containers:** Lightweight, share the Host OS kernel but package the application and dependencies in isolation.
    
* **Docker Socket (`/var/run/docker.sock`):** This is the API endpoint for the Docker Daemon. If this socket is "mounted" (passed through) into a container, that container gains full control over the Docker host.
* **Container Escape:** A technique where an attacker breaks out of the isolated container environment to gain access to the underlying host or other containers. In this case, abusing the mounted socket is a classic escape vector.

## 🕵️‍♂️ Investigation Walkthrough

### Step 1: Reconnaissance
I started by listing the running containers on the host machine to understand the architecture.
1.  **Command:** `docker ps`
2.  **Result:** I observed three main containers:
    * The main web application (DoorDasher/Hopperoo) on port 5001.
    * A `deployer` container.
    * An `uptime-checker` container.

### Step 2: Infiltration (The Uptime Checker)
The scenario hinted that we had access via the monitoring pod. I accessed the shell of the `uptime-checker` container.
1.  **Command:** `docker exec -it uptime-checker sh`
2.  **Verification:** Once inside, I checked if the critical Docker socket was accessible.
    * **Command:** `ls -la /var/run/docker.sock`
    * **Result:** The socket was present. This is a critical misconfiguration. It meant I could run Docker commands *inside* this container to control the *host's* Docker engine.
    

### Step 3: The Escape & Pivot
Since I had control over the Docker daemon via the socket, I used the `uptime-checker` container to execute commands inside the *other* containers, effectively bypassing the isolation. I targeted the `deployer` container, which likely held the administrative scripts.
1.  **Command:** `docker exec -it deployer bash`
2.  **Result:** I successfully moved from the low-privilege monitoring container into the administrative deployer container.

### Step 4: Remediation
Now inside the `deployer` container, I looked for the solution.
1.  **Discovery:** I listed the files and found `recovery_script.sh` in the root directory.
2.  **Execution:** I ran the script to revert the changes.
    * **Command:** `sudo /recovery_script.sh`
3.  **Flag Retrieval:** I also found a flag file in the same directory.
    * **Command:** `cat /flag.txt`

### Step 5: Bonus - Credential Hunting
The prompt mentioned a secret code on the news site (port 5002) that doubled as a password.
1.  **Action:** I navigated to `http://MACHINE_IP:5002` in the browser.
2.  **Finding:** I inspected the site (sometimes hidden in the source code or displayed as a "news ticker") to find the secret code/password for the `deployer` user.

## 🧠 Key Takeaways
* **Isolation is not Absolute:** Just because an app is in a container doesn't mean it's secure. Misconfigurations (like mounting `docker.sock`) break the security model entirely.
* **Least Privilege:** Containers should rarely run as root and should almost never have access to the Docker socket unless strictly necessary (and even then, strict controls should apply).
* **Microservices Architecture:** Understanding how different containers talk to each other (or the host) is essential for mapping out attack vectors in modern cloud environments.

## 📸 Screenshots
*Below are the findings from the container investigation.*

![Docker PS Output](screenshots/Day%2014/flag.png)
*Figure 1: Getting the flag.*

![Socket Verification](screenshots/Day%2014/password.png)
*Figure 2: Password of the deployer user.*

## 🔗 References
* [TryHackMe - Advent of Cyber 2025](https://tryhackme.com/adventofcyber25)
* [Docker Security Documentation](https://docs.docker.com/engine/security/)