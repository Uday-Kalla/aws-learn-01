![image](https://docs.aws.amazon.com/images/vpc/latest/userguide/images/how-it-works.png) 

The above diagram shows an example VPC. The VPC has one subnet in each of the Availability Zones in the Region, EC2 instances in each subnet, and an internet gateway to allow communication between the resources in your VPC and the internet.


### Q: Why Is an Internet Gateway Important?
* In AWS: An Internet Gateway (IGW) is what enables public subnets to connect to the internet.
  * The NAT Gateways, Application Load Balancer, and even access to S3 (without a gateway endpoint) require an IGW to send/receive traffic from the internet.

🔄 Where Would the IGW Fit in This Diagram?
    * It should be connected to the VPC, and attached to the Public Subnets.
    * All internet-bound traffic (like from the ALB or NAT Gateway) would pass through the IGW.

> - Application Load Balancer (ALB) ↔ IGW ↔ Internet
> - NAT Gateway ↔ IGW ↔ Internet

![image](https://github.com/iam-veeramalla/aws-devops-zero-to-hero/assets/43399466/89d8316e-7b70-4821-a6bf-67d1dcc4d2fb)

This image shows a **basic architecture of a highly available web application in AWS** using **two Availability Zones** for better performance and failover. Here’s a simple explanation of each component:

### 🌍 **Region**

* A **geographical area** (like Mumbai, Virginia) where AWS resources are hosted.

### 🧱 **VPC (Virtual Private Cloud)**

* A **private network** inside AWS where you can launch your resources (like servers, databases).

### 🌐 **Public Subnet**

* A network area **connected to the internet**.
* It hosts:

  * **NAT Gateways**: Allow private servers to access the internet (for updates, etc.), but keep them hidden from outside.
  * **Application Load Balancer (ALB)**: Distributes incoming traffic across multiple servers for better performance and fault tolerance.

### 🔒 **Private Subnet**

* A secure network area **not directly accessible from the internet**.
* It hosts:

  * **Servers (EC2 instances)**: Your actual applications (like websites or APIs).
  * These servers are part of an **Auto Scaling Group**:

    * It **adds or removes servers** automatically based on traffic/load.
  * Protected by a **Security Group**:

    * Works like a **firewall**, only allowing specific traffic in or out.

### 🌀 **Availability Zones (AZs)**

* Two **separate physical data centers** in the same region.
* If one fails, the other keeps your app running.

### 📥 S3 Gateway
* Connects your VPC to Amazon S3, which is used for storing files like images, backups, etc.

### Summary:

This setup ensures your application is:

* **Highly available** (uses 2 AZs)
* **Scalable** (Auto Scaling group)
* **Secure** (private subnets, security groups)
* **Load balanced** (ALB distributes traffic)

---
### <p align="center"> 🔷 Step-by-Step Explanation of VPC </p>

### 🧱 **Step 1: Region & VPC**

* You start by creating a **VPC (Virtual Private Cloud)** in a **Region**.

  * A Region is a physical location (e.g., Mumbai, Ohio).
  * The VPC is your **private isolated network** inside AWS.

### 🟦 **Step 2: Availability Zones (AZs)**

* Inside the Region, choose **two Availability Zones** (AZs).

  * These are **separate data centers**, which gives **high availability**.
  * If one AZ goes down, your application still works from the other.

### 🟩 **Step 3: Public Subnets**

* Create **Public Subnets** in each AZ.

  * Public = connected to the internet.
* Add the following to public subnets:

  * ✅ **NAT Gateways**: These allow private servers to go out to the internet (e.g., for updates), but block incoming internet access.
  * ✅ **Application Load Balancer (ALB)**: Routes incoming user traffic to healthy servers.

### 🟧 **Step 4: Private Subnets**

* Create **Private Subnets** in each AZ.

  * Private = **not directly accessible** from the internet.
* These subnets contain:

  * 🖥️ **EC2 Servers**: Your app runs here.
  * 🔁 **Auto Scaling Group**: Adds or removes EC2 servers automatically based on traffic.

### 🔐 **Step 5: Security Group**

* All servers (EC2 instances) are protected using a **Security Group**.

  * Acts like a **firewall**: defines what kind of traffic is allowed in and out.

### 🌀 **Step 6: Traffic Flow**

Here’s how a user request flows:

1. 🌐 User hits your domain → routed to **Application Load Balancer**.
2. 🔁 ALB sends request to one of the healthy EC2 instances (in private subnets).
3. 📤 EC2 instances might need internet access for updates → they go through **NAT Gateway**.

### 📦 **Step 7: S3 Gateway Access**

* Your application might store or retrieve files (like images, logs) from **Amazon S3**.
* The VPC uses an **S3 Gateway Endpoint** to access S3 **without going over the internet**.

### 🎯 **Step 8: Final Outcome**

* ✅ High availability (multi-AZ).
* ✅ Scalability (auto scaling).
* ✅ Security (private subnets, security group).
* ✅ Performance (load balancer).
* ✅ Cost-effective internet access (NAT + S3 gateway).

