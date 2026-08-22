# ☁️ AWS IAM: Role-Based Access Control (RBAC) Implementation

A hands-on project demonstrating the **Principle of Least Privilege** in AWS. This lab configures secure, job-specific access for cloud support staff using IAM users, groups, and policies.

## 🧰 Services Used
- **AWS IAM** (Identity and Access Management)
- **Amazon EC2** (Elastic Compute Cloud)
- **Amazon S3** (Simple Storage Service)

## 🪜 Task 1: Explore the users and groups, and inspect policies

* **👤 IAM Users**  
We have 3 users (`user-1`, `user-2`, `user-3`) already created for us, but they currently have no permissions or group memberships yet.  
![IAM Users List](Images\iam-users-list.png)

* **👤IAM Users Groups & Policies**   
We found 3 groups, each with different permissions. Here is a breakdown of what each group does:

| Group Name | Permission Type | Description | JSON Policy |
| :--- | :--- | :--- | :--- |
| **EC2-Support** | Managed Policy | **Read-Only:** Allows viewing EC2 (including VPC security groups), Load Balancing, CloudWatch (metrics and statistics), and Auto Scaling resources. | <details><summary><strong>Show Policy</strong></summary><pre><code>{<br>  "Version": "2012-10-17",<br>  "Statement": [<br>    { "Effect": "Allow", "Action": [ "ec2:Describe*", "ec2:GetSecurityGroupsForVpc" ], "Resource": "*" },<br>    { "Effect": "Allow", "Action": "elasticloadbalancing:Describe*", "Resource": "*" },<br>    { "Effect": "Allow", "Action": [ "cloudwatch:ListMetrics", "cloudwatch:GetMetricStatistics", "cloudwatch:Describe*" ], "Resource": "*" },<br>    { "Effect": "Allow", "Action": "autoscaling:Describe*", "Resource": "*" }<br>  ]<br>}</code></pre></details> |
| **S3-Support** | Managed Policy | **Read-Only:** Allows getting, listing, and describing resources in Amazon S3, including S3 Object Lambda. | <details><summary><strong>Show Policy</strong></summary><pre><code>{<br>  "Version": "2012-10-17",<br>  "Statement": [<br>    { "Effect": "Allow", "Action": [ "s3:Get*", "s3:List*", "s3:Describe*", "s3-object-lambda:Get*", "s3-object-lambda:List*" ], "Resource": "*" }<br>  ]<br>}</code></pre></details> |
| **EC2-Admin** | Inline Policy | **Admin/Custom:** Allows viewing, starting, and stopping EC2 instances, but **strictly limited** to `*.nano` and `*.micro` instance types. | <details><summary><strong>Show Policy</strong></summary><pre><code>{<br>  "Version": "2012-10-17",<br>  "Statement": [<br>    { "Condition": { "ForAllValues:StringLikeIfExists": { "ec2:InstanceType": [ "*.nano", "*.micro" ] } }, "Action": [ "ec2:Describe*", "ec2:StartInstances", "ec2:StopInstances" ], "Resource": [ "*" ], "Effect": "Allow" }<br>  ]<br>}</code></pre></details> |

![IAM Groups List](Images\iam-groups-list.png)

* **🎯 The Target Mapping & Final Goal**


| User | Target Group | Job Function & Permissions |
| :--- | :--- | :--- |
| 👤 **user-1** | 🪣 **S3-Support** | Read-only access to Amazon S3 |
| 👤 **user-2** | ☁️ **EC2-Support** | Read-only access to Amazon EC2 |
| 👤 **user-3** | 🛠️ **EC2-Admin** | View, Start, and Stop Amazon EC2 instances (nano/micro only) |


## 🪜Task 2: Add users to groups

  

* **Adding user-1:**  
  `IAM Dashboard` ➡️ `User groups` ➡️ `S3-Support` ➡️ `Users tab` ➡️ `Add users` ➡️ `Select user-1` ➡️ `Add users`

* **Adding user-2:**  
  `IAM Dashboard` ➡️ `User groups` ➡️ `EC2-Support` ➡️ `Users tab` ➡️ `Add users` ➡️ `Select user-2` ➡️ `Add users`

* **Adding user-3:**  
  `IAM Dashboard` ➡️ `User groups` ➡️ `EC2-Admin` ➡️ `Users tab` ➡️ `Add users` ➡️ `Select user-3` ➡️ `Add users`


## 🪜Task 3: Sign in and test user permissions


* 👤 **Testing `user-1` (S3-Support):**
  * 🪣 **S3 Access:** ✅ Successfully viewed S3 buckets and their contents.
    <br>![User 1 S3 Allow](Images/user1-s3-allow.png)
  * ☁️ **EC2 Access:** ❌ Received an *Unauthorized* error when trying to view instances.
    <br>![User 1 EC2 Deny](Images/user1-ec2-deny.png)

* 👤 **Testing `user-2` (EC2-Support):**
  * ☁️ **EC2 Access (View):** ✅ Successfully listed and viewed EC2 instances.
    
  * 🛑 **EC2 Access (Modify):** ❌ Received an *Unauthorized* error when attempting to **Stop** an instance.
    <br>![User 2 EC2 Stop Deny](Images/user2-ec2-stop-deny.png)
  * 🪣 **S3 Access:** ❌ Received an *Unauthorized* error when trying to list buckets.
    <br>![User 2 S3 Deny](Images/user2-s3-deny.png)

* 👤 **Testing `user-3` (EC2-Admin):**
  * ☁️ **EC2 Access (View):** ✅ Successfully listed and viewed EC2 instances.
    
  * 🛑 **EC2 Access (Modify):** ✅ Successfully **Stopped** the EC2 instance (Instance state changed to *Stopping*).
    <br>![User 3 EC2 Stop Allow](Images/user3-ec2-stop-allow.png)