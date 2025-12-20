# Day 20: Race Conditions - Toy to The World

## 📝 Scenario
In Day 20, "The Best Festival Company" (TBFC) faced a crisis during the launch of their limited-edition "SleighToy." Despite only having 10 units in stock, the system processed far more orders within seconds of the launch. McSkidy suspected that the Bandit Bunnies were exploiting a timing flaw in the checkout system.

As a security analyst, my objective was to investigate this anomaly by simulating a **Race Condition** attack. I used Burp Suite to capture a legitimate checkout request and replay it multiple times in parallel, proving that the system failed to lock the inventory correctly during simultaneous transactions.

## 🛠️ Tools & Technologies Used
* **Burp Suite Professional/Community:** A web vulnerability scanner and proxy tool. Specifically, I used the **Repeater** and **Group Send** (Parallel) features.
* **FoxyProxy:** A browser extension to route web traffic through the Burp Suite proxy.
* **Firefox:** The browser used to interact with the target e-commerce application.

## 🧠 Theoretical Concepts
* **Race Condition:** A vulnerability that occurs when a system attempts to perform two or more operations at the same time, but due to the nature of the device or software, the operations must be done in the proper sequence to be done correctly.
* **TOCTOU (Time-of-Check to Time-of-Use):** A specific type of race condition where the system checks for a condition (e.g., "Is stock > 0?") and then performs an action (e.g., "Subtract 1 from stock"), but the state changes *between* the check and the action.
* **Atomicity:** The property of database operations where a series of steps (Check Stock -> Deduct Stock -> Confirm Order) must be treated as a single, indivisible unit. If one part fails, the whole transaction should fail.

## 🕵️‍♂️ Investigation Walkthrough

### Step 1: Baseline Request
I started by making a legitimate purchase to understand the application's flow.
1.  **Login:** Accessed the site with credentials `attacker:attacker@123`.
2.  **Order:** Added the "SleighToy" to the cart and completed the checkout.
3.  **Capture:** I verified that the request was captured in Burp Suite's **HTTP History**.

### Step 2: Preparing the Attack (Burp Repeater)
I located the `POST /process_checkout` request in Burp and sent it to **Repeater**.
1.  **Grouping:** I added the request to a new "Tab Group" named `cart`.
2.  **Duplication:** I duplicated the request tab 20 times (creating 20 identical checkout requests ready to fire).

### Step 3: Executing the Race Condition
The key to this attack is speed—sending the requests fast enough that they hit the server before the database can update the stock count.
1.  **Configuration:** In Burp Repeater, I changed the send method to **"Send group in parallel (last-byte sync)"**. This ensures all requests are released simultaneously.
2.  **Launch:** I clicked **Send group (parallel)**.
3.  **Result:** The server processed multiple requests successfully.

### Step 4: Verification
I returned to the browser and refreshed the dashboard.
1.  **Observation:** The "SleighToy" stock had dropped well below zero (e.g., -8), and I had multiple order confirmations in my history.
2.  **Flag Retrieval:** The system displayed the first flag upon detecting the negative stock state.

### Step 5: The Second Attack (Bunny Plush)
I repeated the exact same methodology for the "Bunny Plush (Blue)" item.
1.  **Target:** `Bunny Plush (Blue)`
2.  **Method:** Parallel execution of checkout requests via Burp.
3.  **Result:** Stock went negative again, revealing the second flag.

## 🧠 Key Takeaways
* **Concurrency is Hard:** Simple `if (stock > 0) { stock-- }` logic fails in multi-threaded environments because two threads can pass the `if` check at the exact same nanosecond before either decrements the counter.
* **Database Locking:** To fix this, developers must use database transactions with specific locking mechanisms (like `SELECT ... FOR UPDATE`) to ensure only one process can modify the stock row at a time.
* **Impact:** Race conditions can lead to massive financial loss (overselling inventory) or integrity issues (double-spending money/credits).

## 📸 Screenshots
*Below are the findings from the race condition exploitation.*

![First flag](screenshots/Day%2020/1.png)
*Figure 1: Retrieving the first flag.*

![Second flag](screenshots/Day%2020/2.png)
*Figure 2: Retrieving the second flag.*

## 🔗 References
* [TryHackMe - Advent of Cyber 2025](https://tryhackme.com/adventofcyber25)