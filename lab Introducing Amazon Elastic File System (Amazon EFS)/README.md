# 📂 AWS EFS: Introducing Amazon Elastic File System

A guided lab focused on creating, mounting, and monitoring an Amazon Elastic File System (EFS) on an Amazon EC2 instance.

## 🎯 Objective
* 🔑 Log in to the AWS Management Console.
* 📁 Create an Amazon EFS file system.
* 🖥️ Log in to an Amazon EC2 instance running Amazon Linux.
* 🔗 Mount the file system to the EC2 instance.
* 📊 Examine and monitor the performance of the file system.

## 🧰 Services Used
* 📁 **Amazon EFS**
* 🖥️ **Amazon EC2**
* 🔐 **Amazon VPC (Security Groups)**

## 🪜 Steps

**1. Create a Security Group for EFS Access**
Created a new security group to securely control inbound traffic to the EFS file system.

* 🛡️ **Security Group Creation:** Created a group named `EFS Mount Target` within the `Lab VPC`.
* 🚦 **Inbound Rules Configuration:** Added a rule to allow inbound Network File System (NFS) traffic on TCP port 2049 strictly from the `EFSClient` security group.
  * 💡 **Why?** This acts as a virtual firewall, ensuring that only authorized EC2 instances (those associated with the `EFSClient` security group) can mount and access the EFS file system.

![Security Group Details](Images/Screenshot2026-08-24131736.png)
![Inbound Rules](Images/Screenshot2026-08-24131808.png)

**2. Create an EFS File System**
Created and customized an Amazon EFS file system to be accessible across multiple Availability Zones using standard NFSv4.1.

* ⚙️ **Configuration:** Disabled automatic backups, set transition lifecycle management to *None*, and tagged the system with the Name `My First EFS File System`.
* 🌐 **Network Settings:** Deployed the file system within the `Lab VPC`. Removed the `default` security group from all Availability Zone mount targets and attached the newly created `EFS Mount Target` security group instead.
  * 💡 **Why?** Mount targets are the access points in each Availability Zone. Attaching our custom security group ensures the file system enforces the strict inbound rules we created in Step 1.
* ✅ **Outcome:** Successfully created the file system and waited for the File System state and all Mount Target states to become **Available**.

**3. Mount the EFS File System**
Installed the necessary utilities on the EC2 instance and mounted the EFS file system using the NFSv4.1 protocol.

* 🖥️ **Terminal Access:** Accessed the EC2 instance command-line interface.
![EC2 Terminal Session](Images/Screenshot%202026-08-24%20134426.png)

* 📦 **Install Utilities & Create Directory:** Installed the `amazon-efs-utils` package and created a local mount point directory named `efs`.
  * 💡 **Why?** The EFS utilities provide the necessary drivers and tools to mount the file system securely, and the local directory acts as the bridge to the remote storage.

* 🔗 **Retrieve Mount Command:** Opened the EFS Console, selected the file system, and clicked **Attach** to retrieve the exact NFS client mount instructions.
![EFS Attach Instructions](Images/Screenshot%202026-08-24%20134928.png)

* 🚀 **Mount & Verify:** Executed the `sudo mount` command and verified the mounting success by checking the disk filesystem usage with `sudo df -hT`.
  * ✅ **Outcome:** The EFS file system was successfully mounted, displaying an enormous `8.0E` (Exabytes) of available storage capacity.
![Mount and Verify Output](Images/Screenshot%202026-08-24%20135101.png)


**4. Examine File System Performance**
This task focused on understanding how EFS handles heavy workloads and scales its performance linearly by generating artificial traffic and monitoring the results.

* 🏋️ **Benchmarking with Flexible IO (`fio`):** 
  * **What we did:** Executed a synthetic I/O benchmarking test on the EC2 instance using the `fio` utility. The test was designed to simulate a heavy, continuous write workload by creating a 10 GB file directly on the mounted EFS volume.
  * **Why we did it:** To observe how EFS reacts to a sudden spike in data ingestion (bursting) and to verify that the file system can handle sustained write operations.
  
![fio Command Output](Images/Screenshot%202026-08-24%20140346.png)

* 📈 **Monitoring Permitted Throughput (Burst Capability):** 
  * **What we did:** Navigated to Amazon CloudWatch > EFS File System Metrics, and filtered for the `PermittedThroughput` metric.
  * **Why we did it:** EFS is designed to burst to high throughput levels to accommodate spiky file-based workloads. This metric shows the *maximum* throughput the file system is currently permitted to use based on its size and burst credits.
  * 💡 **Observation:** The graph showed a peak capability of around 3 GB/s, proving that EFS automatically provisions high throughput to handle the sudden 10 GB write operation initiated by our `fio` test.
  
![CloudWatch Permitted Throughput](Images/Screenshot%202026-08-24%20140842.png)

* 📊 **Calculating Actual Write Throughput (Performance Scaling):** 
  * **What we did:** Switched the CloudWatch metric to `DataWriteIOBytes`, changed the statistical aggregation to **Sum**, and adjusted the period to **1 Minute**.
  * **Why we did it:** To measure the *actual* amount of data written to the file system during the test period and calculate the real-world throughput in Bytes per second (B/s).
  * 💡 **Observation:** The graph recorded a peak of approximately 7.6 GB of data written within a single minute. By dividing this peak value by 60 seconds, we determined the exact write throughput achieved. This demonstrates a core EFS concept: as you add more data to your file system, the maximum throughput available scales linearly and automatically.
  
![CloudWatch Data Write IO Bytes](Images/Screenshot%202026-08-24%20141210.png)