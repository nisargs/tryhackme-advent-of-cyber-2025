# Day 19: ICS/Modbus - Claus for Concern

## 📝 Scenario
In Day 19, the chaos at Wareville continues as the Drone Delivery System begins delivering chocolate eggs instead of Christmas presents. The logistics system, controlled by a SCADA network, has been compromised by King Malhare's "Eggsploit" team.

As a security responder, my objective was to interact with the **Programmable Logic Controller (PLC)** using the **Modbus** protocol. I needed to decipher a handwritten "register map" found on-site, identify the malicious configuration, bypass a logic trap set by the attackers, and safely restore the system to deliver the correct gifts.

## 🛠️ Tools & Technologies Used
* **Modbus TCP (Port 502):** The industrial communication protocol used by the drones and the PLC.
* **Python (`pymodbus`):** A library used to programmatically read from and write to the PLC's registers and coils.
* **Nmap:** Used for initial reconnaissance to identify the open Modbus port.
* **Web Browser:** Used to monitor the CCTV feed for real-time visual confirmation of physical changes.

## 🧠 Theoretical Concepts
* **SCADA (Supervisory Control and Data Acquisition):** The overarching system used to monitor and control industrial processes.
    
* **PLC (Programmable Logic Controller):** The ruggedized industrial computer that directly controls machinery (actuators) based on sensor input and programmed logic.
* **Modbus Protocol:** A simple, master-slave protocol used in industrial environments. It lacks built-in security (authentication/encryption), making it vulnerable if exposed.
    * **Coils:** Read/Write binary values (On/Off).
    * **Holding Registers:** Read/Write numerical values (Analog data).
    
## 🕵️‍♂️ Investigation Walkthrough

### 1. Reconnaissance
An Nmap scan revealed **Port 502** (Modbus TCP) was open. A physical note found near the terminal provided a "Register Map," mapping specific addresses (like `HR0` and `C11`) to their functions.

Using a Python script, I queried the PLC and confirmed:
* **HR0 (Package Type):** Set to `1` (Chocolate Eggs).
* **HR4 (Signature):** Set to `666` (Eggsploit Signature).
* **C11 (Protection):** Set to `True`.

### 2. Identifying the Trap
The note contained a critical warning: **"Never change HR0 while C11=True!"**

The attacker set a logic trap. If an administrator attempted to simply switch the package type back to "Gifts" (`HR0=0`) without first disabling the protection (`C11`), a "Self-Destruct" sequence (`C15`) would trigger, causing an emergency dump of all inventory into the ocean.

### 3. Safe Remediation Strategy
To solve the challenge without triggering the trap, I had to execute the remediation in a specific order via Python:
1.  **Disable Protection:** Write `False` to Coil 11 (`C11`).
2.  **Restore Package Type:** Write `0` to Holding Register 0 (`HR0`).
3.  **Restore Safety Protocols:** Re-enable Inventory Verification (`C10`) and Audit Logging (`C13`).

### 4. Result
Upon executing the sequence, the CCTV feed confirmed the drones returned to delivering Christmas presents. The Python script then extracted the flag from the system's registers.

## 🧠 Key Takeaways
* **Order of Operations Matters:** In Industrial Control Systems (ICS), the sequence of changes is critical. Changing a variable out of order can trigger physical fail-safes or malicious logic traps.
* **Security via Obscurity Fails:** Modbus relies on the network being secure. Once an attacker reaches Port 502, they have full read/write access to the machinery.
* **Physical Consequences:** Unlike standard IT attacks, compromising OT (Operational Technology) impacts the physical world (e.g., robotic arms, conveyor belts).

## 📸 Screenshots
*Below is the final flag retrieved from the PLC registers.*

![Day 19 Flag](screenshots/Day%2019/flag.png)
*Figure 1: Retrieving the flag and restoring the system.*

## 🔗 References
* [TryHackMe - Advent of Cyber 2025](https://tryhackme.com/adventofcyber25)