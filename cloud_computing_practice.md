# Cloud Computing: Practice Questions and Solutions

This document contains practical, scenario-based questions and solutions for Cloud Computing. These exercises are designed to test your understanding of cloud architecture, services, and best practices.

---

## Scenario 1: High Availability E-commerce Site
**Question:** 
You are tasked with designing the architecture for a new e-commerce application on AWS. The application must be highly available, fault-tolerant, and able to handle sudden spikes in traffic during holiday sales. What core services and architectural patterns would you use?

**Solution:**
To ensure high availability and scalability, the architecture should be distributed across multiple Availability Zones (AZs).
1.  **Compute:** Use **Amazon EC2 Auto Scaling groups** across at least two AZs. This ensures that if one AZ goes down, the instances in the other AZ can handle the traffic. Auto Scaling will automatically add or remove instances based on CPU or network load.
2.  **Traffic Routing:** Place an **Application Load Balancer (ALB)** in front of the EC2 instances. The ALB distributes incoming application traffic across multiple targets in multiple AZs.
3.  **Database:** Use **Amazon RDS Multi-AZ deployment** for the relational database. This provisions and maintains a synchronous standby replica in a different AZ, providing automatic failover in case of hardware failure.
4.  **Static Assets:** Store static assets (images, CSS, JS) in an **Amazon S3 bucket** and use **Amazon CloudFront (CDN)** to cache these assets closer to the users, reducing latency and offloading traffic from the EC2 instances.

---

## Scenario 2: Cost Optimization for Development Environments
**Question:**
Your development team uses several virtual machines in the cloud for testing purposes. These environments are only needed during business hours (9 AM to 5 PM, Monday to Friday). You notice the monthly cloud bill is very high. How can you optimize the costs?

**Solution:**
Development environments do not need to run 24/7. Cost optimization can be achieved by scheduling uptime:
1.  **Automated Scheduling:** Implement a script or use native cloud scheduling tools (e.g., AWS Instance Scheduler or Azure Automation) to automatically stop the instances at 5 PM and start them at 9 AM on weekdays.
2.  **Right-sizing:** Analyze the CPU and memory utilization of the development instances. If they are consistently underutilized, downgrade them to smaller, cheaper instance types.
3.  **Spot/Preemptible Instances:** If the testing workloads are fault-tolerant and can handle interruptions, use Spot Instances (AWS) or Preemptible VMs (GCP). These are unused capacity sold at a massive discount compared to On-Demand prices.

---

## Scenario 3: Disaster Recovery Strategy
**Question:**
Your company hosts its critical application in a single region (us-east-1). They want to implement a "Pilot Light" disaster recovery strategy in a different region (us-west-2) to achieve a low Recovery Time Objective (RTO) without incurring massive costs. How do you implement this?

**Solution:**
A Pilot Light strategy means keeping the core pieces of the system running in the DR region, while other parts are scaled down or turned off until a disaster occurs.
1.  **Data Replication:** Continuously replicate the primary database from us-east-1 to a standby database in us-west-2. This is the most critical running piece (the "pilot light").
2.  **Infrastructure as Code:** Maintain all infrastructure configurations (VPCs, subnets, load balancers, EC2 configurations) as code (e.g., Terraform or CloudFormation).
3.  **Failover Process:** When a disaster occurs:
    - Promote the standby database in us-west-2 to be the primary database.
    - Use the IaC templates to rapidly provision the compute resources (EC2 instances, Load Balancers) in us-west-2.
    - Update the DNS records (e.g., Amazon Route 53) to point traffic to the new load balancer in us-west-2.

---

## Scenario 4: Serverless API Backend
**Question:**
You need to build a lightweight API for a mobile application. The traffic is highly unpredictable, with long periods of zero traffic followed by sudden bursts. You want to minimize operational overhead and only pay for what you use. What is the recommended architecture?

**Solution:**
A Serverless architecture is perfect for unpredictable traffic and zero maintenance overhead.
1.  **API Gateway:** Use **Amazon API Gateway** (or Azure API Management) as the front door. It handles request routing, rate limiting, and authentication.
2.  **Compute:** Use **AWS Lambda** (or Azure Functions/Google Cloud Functions) to process the API requests. Functions are triggered by the API Gateway, scale automatically from zero to thousands of concurrent requests, and you are only billed for the exact compute time used (down to the millisecond).
3.  **Database:** Use a Serverless NoSQL database like **Amazon DynamoDB**. It automatically scales read/write capacity based on demand and requires zero server maintenance.

---

## Scenario 5: Secure Cloud Networking
**Question:**
You are deploying a web application with a frontend web tier and a backend database tier in a Virtual Private Cloud (VPC). How should you design the subnets to ensure maximum security for the database?

**Solution:**
Security should be implemented at the network level using a public/private subnet architecture.
1.  **Public Subnet:** Deploy the Application Load Balancer (ALB) and potentially a NAT Gateway in the Public Subnet. This subnet has a route to the Internet Gateway, allowing inbound traffic from the internet to reach the ALB.
2.  **Private Subnet:** Deploy the Web Servers (EC2) and the Database (RDS) in Private Subnets. These subnets do *not* have a route to the Internet Gateway. 
    - The Web Servers can only be accessed via the ALB. 
    - The Database can only be accessed by the Web Servers (controlled via Security Groups).
    - If the Web Servers need to download updates from the internet, they route their traffic through the NAT Gateway located in the public subnet.
