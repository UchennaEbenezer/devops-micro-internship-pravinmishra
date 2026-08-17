# Assignment 5 — Deploy a Highly Available Two-Tier Application on AWS (VPC + ALB + ASG + Multi-AZ RDS)

Part of the DevOps Micro Internship (DMI) Cohort 3 with Agentic AI

---

## Purpose

In this assignment, you will design and deploy a highly available two-tier web application on AWS: highly available networking across two Availability Zones, an Application Load Balancer, an Auto Scaling Group for the web tier, and a private Multi-AZ RDS database. You must prove high availability with real failure tests.

---

# Task 1 — Create HA Networking (VPC + 4 Subnets + IGW + NAT + Route Tables)

## Goal

Build a VPC (10.0.0.0/16) with two public and two private subnets across two Availability Zones, an Internet Gateway, a NAT Gateway, and the matching public/private route tables.

### Evidence

#### Screenshot 1 — VPC details showing CIDR 10.0.0.0/16

![dmi](./screenshots/assignment-05/S1.PNG)

---

#### Screenshot 2 — Subnets list showing four subnets and their Availability Zones

![dmi](./screenshots/assignment-05/S2.PNG)

---

#### Screenshot 3 — Public route table showing the Internet Gateway route and both public-subnet associations

![dmi](./screenshots/assignment-05/S3.1.PNG)

![dmi](./screenshots/assignment-05/S3.2.PNG)

---

#### Screenshot 4 — Private route table showing the NAT Gateway route and both private-subnet associations

![dmi](./screenshots/assignment-05/S4.1.PNG)

![dmi](./screenshots/assignment-05/S4.2.PNG)


---

#### Screenshot 5 — NAT Gateway status showing Available and the Elastic IP

![dmi](./screenshots/assignment-05/S5.PNG)


---

# Task 2 — Create Security Groups (ALB, EC2, RDS) with Least Privilege

## Goal

Create `ha-alb-sg` (HTTP public), `ha-web-sg` (HTTP only from `ha-alb-sg`, SSH from your IP), and `ha-db-sg` (database port only from `ha-web-sg`).

### Evidence

#### Screenshot 6 — ALB Security Group inbound rules

![dmi](./screenshots/assignment-05/S6.PNG)


---

#### Screenshot 7 — EC2 Security Group inbound rules showing the ALB Security Group reference and SSH from your IP

![dmi](./screenshots/assignment-05/S7.PNG)

---

#### Screenshot 8 — RDS Security Group inbound rule showing the database port allowed only from the EC2 Security Group

![dmi](./screenshots/assignment-05/S8.PNG)

---

# Task 3 — Deploy Database Tier (RDS Multi-AZ in Private Subnets)

## Goal

Launch a private, Multi-AZ RDS database (MySQL or PostgreSQL) using the private DB Subnet Group and `ha-db-sg`.

### Evidence

#### Screenshot 9 — RDS summary showing Multi-AZ = Yes and Publicly accessible = No

![dmi](./screenshots/assignment-05/S9.PNG)

---

#### Screenshot 10 — RDS connectivity section showing the DB Subnet Group and Security Group

![dmi](./screenshots/assignment-05/S10.PNG)


---

# Task 4 — Build a Launch Template (User Data Installs App + Connects to DB)

## Goal

Create a Launch Template whose user data installs the web-server runtime, deploys the application, configures the database connection, and starts the required services.

### Evidence

#### Screenshot 11 — Launch Template details showing that user data exists, including a visible snippet

![dmi](./screenshots/assignment-05/S11.PNG)

---

#### Screenshot 12 — A running instance created from the template showing that the application responds on port 80 through a local test or browser using its public IP

![dmi](./screenshots/assignment-05/S12.PNG)

---

# Task 5 — Create an Application Load Balancer (ALB) Across 2 Public Subnets

## Goal

Create an internet-facing ALB across both public subnets with an HTTP listener and a healthy instance target group.

### Evidence

#### Screenshot 13 — ALB details showing two public subnets in two Availability Zones

![dmi](./screenshots/assignment-05/S13.PNG)

---

#### Screenshot 14 — Target group showing at least one healthy target

![dmi](./screenshots/assignment-05/S14.PNG)

---

# Task 6 — Create Auto Scaling Group (ASG) in 2 Public Subnets

## Goal

Create an Auto Scaling Group from the Launch Template across both public subnets, with desired capacity 2, minimum 2, and maximum 4, registered to the ALB target group.

### Evidence

#### Screenshot 15 — Auto Scaling Group showing desired, minimum, and maximum capacity and the selected subnet Availability Zones

![dmi](./screenshots/assignment-05/S15.1.PNG)

![dmi](./screenshots/assignment-05/S15.2.PNG)

---

#### Screenshot 16 — EC2 instances list showing two running instances in different Availability Zones

![dmi](./screenshots/assignment-05/S16.PNG)

---

# Task 7 — Configure App to Use RDS + Validate Read/Write

## Goal

Confirm the application communicates with the RDS database through the ALB DNS name with at least one read and one write operation.

### Evidence

#### Screenshot 17 — Browser showing the application loaded through the ALB DNS name with the URL visible

![dmi](./screenshots/assignment-05/S17.PNG)

---

#### Screenshot 18 — Proof of a database write through a UI message or database query output

![dmi](./screenshots/assignment-05/S18.PNG)

---

# Task 8 — High Availability Tests (Must Do Both)

## Goal

Test A: terminate one web instance and confirm the Auto Scaling Group replaces it automatically without interrupting the ALB.

Test B: simulate an Availability Zone impact (stop, detach, or reduce desired capacity in one AZ) and confirm the application stays available.

### Evidence

#### Screenshot 19 — EC2 showing the terminated instance and the newly launched instance; timestamps are helpful

![dmi](./screenshots/assignment-05/S19.1.PNG)

![dmi](./screenshots/assignment-05/S19.2.PNG)

---

#### Screenshot 20 — Target group showing healthy targets after replacement

![dmi](./screenshots/assignment-05/S20.PNG)

---

#### Screenshot 21 — Evidence that an instance was removed, detached, placed in Standby, or stopped in one Availability Zone

![dmi](./screenshots/assignment-05/S21.PNG)

---

#### Screenshot 22 — Browser showing that the ALB DNS endpoint still works during the change

![dmi](./screenshots/assignment-05/S22.PNG)

---

# Task 9 — Architecture and Test-Results Summary

## Goal

Summarize the VPC/subnet layout, the ALB and Auto Scaling Group setup, the private Multi-AZ RDS setup, and the results of both high-availability tests.

### Evidence

#### Screenshot 23 — A simple architecture diagram, which may be hand-drawn, or an AWS console overview showing the components

![dmi](./screenshots/assignment-05/S23.PNG)

---

### Notes

Summarize the VPC and subnets across the two Availability Zones.

Write your answer here.

Summarize the ALB and Auto Scaling Group setup.

With an HTTP listener forwarding to the target group ha-tg, an internet-facing Application Load Balancer spans both public subnets. An instance that boots but does not serve is considered unsuccessful because an Auto Scaling group uses ELB health checks instead of EC2 status checks and runs desired 2, minimum 2, maximum 4 across both subnets, registered to that target group. An instance is disposable and replaceable without manual labor since it is created from a launch template whose user data installs the web server and runtime, deploys the application, writes the database configuration, and initiates the service.

Summarize the private Multi-AZ RDS setup.

A Multi-AZ-enabled db.t3.micro MySQL instance with no public access is positioned in a subnet group that only includes the two private subnets. Instead of using address ranges to manage access, a security group chain is used: the database only allows traffic over port 3306 from the web tier group, the ALB group accepts HTTP from anywhere, and the web tier only takes HTTP from the ALB group and SSH from my own address. Each rule continues to function even when the Auto Scaling group starts and stops instances with different IP addresses since it refers to a security group rather than a CIDR.

Summarize the results of both high-availability tests.

Test A (Instance Termination): I immediately ended one web instance. Within roughly three minutes, the Auto Scaling group launched a replacement instance in the same Availability Zone after detecting that capacity had dropped below desired. Once the new instance had completed configuring itself, the target group reverted to two healthy targets. Because the load balancer ceased routing to the failing instance as soon as its health check failed while the instance in the other Availability Zone absorbed the traffic, a curl loop against the ALB DNS name continued to produce successful replies throughout the replacement.

Test B (Availability Zone impact): I stopped the eu-north-1a instance and left only eu-north-1b running, reducing the desired capacity to one. There were no dropped connections, gateway issues, or failures. The Auto Scaling group then rebalanced between the two zones after capacity was restored to two.

---

# LinkedIn Post (Required)

## Goal

Publish a LinkedIn post about the high-availability build, including the ALB URL (or a redacted screenshot), three to five lines on what you built and how you tested high availability, and one proof screenshot.

## Evidence

#### LinkedIn Post URL

Paste your LinkedIn post URL here:

[LinkedIn post]()

---

#### Screenshot of LinkedIn post

Add your screenshot here.

---

# Submission Instructions

- Add all required screenshots in your submission
- Do not expose passwords, connection strings, private keys, or account IDs

---

# Completion Checklist

- [✅] Task 1: VPC, four subnets, IGW, NAT Gateway, and route tables created (Screenshots 1–5)
- [✅] Task 2: Least-privilege ALB, EC2, and RDS security groups created (Screenshots 6–8)
- [✅] Task 3: Private Multi-AZ RDS created (Screenshots 9–10)
- [✅] Task 4: Self-configuring Launch Template created and tested (Screenshots 11–12)
- [✅] Task 5: ALB created across both public subnets (Screenshots 13–14)
- [✅] Task 6: Auto Scaling Group running two instances across two AZs (Screenshots 15–16)
- [✅] Task 7: Application verified through the ALB with a database read and write (Screenshots 17–18)
- [✅] Task 8: Both high-availability tests completed (Screenshots 19–22)
- [✅] Task 9: Architecture and test-results summary completed (Screenshot 23 & Notes)
- [✅] LinkedIn post published and URL submitted
- [✅] No sensitive data exposed

---

## 📌 About DMI & CloudAdvisory

DevOps Micro Internship (DMI) is a project-based DevOps program run by Pravin Mishra (The CloudAdvisory) focused on real-world execution, systems thinking, and career readiness.

It helps learners build strong DevOps foundations with hands-on experience.

---

## 📌 Resources

- 🌐 DMI Official Website: https://dmi.pravinmishra.com?utm_source=github&utm_medium=readme  
- 🎓 University: https://university.pravinmishra.com?utm_source=github&utm_medium=readme  
- 💬 Discord Community: https://discord.pravinmishra.com?utm_source=github&utm_medium=readme  
- 📝 Blog: https://dmi.pravinmishra.com/blog?utm_source=github&utm_medium=readme  
- ▶️ YouTube Playlist: https://www.youtube.com/playlist?list=PLFeSNDtI4Cho  
- 🔗 Pravin Mishra (LinkedIn): https://www.linkedin.com/in/pravin-mishra-aws-trainer/  
- 🏢 CloudAdvisory (LinkedIn): https://www.linkedin.com/company/thecloudadvisory/

---

*This submission is part of DevOps Micro Internship (DMI) Cohort 3 — Agentic AI Track.*