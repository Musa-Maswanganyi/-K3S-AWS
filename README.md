# Assignment 1 — K3s High-Availability Cluster on AWS

**Full Name:** Musa Maswanganyi  
**Student Number:** 230019978  
**Module:** Communication Networks Practice 4  
**Lecturer:** Dr. Waldon Hendricks  
**Date:** 27 March 2026  

K3s on AWS Practical Setup Guide
### Part 1 - Setup Instances
### Step 1 — Create a Key Pair
A key pair was created to allow secure SSH access into the EC2 instances.

1. Navigated to **EC2 Dashboard → Network & Security → Key Pairs**  
2. Clicked **Create key pair**  
3. Configured:
   - Name: `k3s-key`
   - Type: RSA
   - Format: `.pem`
4. Set permissions locally:
chmod 400 ~/Downloads/k3s-key.pem

### Step 2 — Create a Security Group
A security group (`k3s-ha-sg`) was created to allow required K3s traffic.

| Type | Protocol | Port Range | Source |
|------|----------|------------|--------|
| SSH | TCP | 22 | 0.0.0.0/0 |
| Custom TCP | TCP | 6443 | 0.0.0.0/0 |
| Custom TCP | TCP | 2379-2380 | sg-0a2c5fcb7bf297358 |
| Custom TCP | TCP | 10250 | sg-0a2c5fcb7bf297358 |
| Custom UDP | UDP | 8472 | sg-0a2c5fcb7bf297358 |
| Custom TCP | TCP | 30000-32767 | 0.0.0.0/0 |

### Step 3: Launch EC2 Instances
Instance Requirements:
- AMI: Ubuntu Server 22.04 LTS
- Instance Type: t3.large
- Key Pair: k3s-key
- Security Group: k3s-ha-sg (sg-0a2c5fcb7bf297358)

<img width="1143" height="369" alt="Screenshot 2026-03-27 193210" src="https://github.com/user-attachments/assets/99e22807-1968-4a42-811a-9ba7b6f6fad8" />
<img width="1138" height="407" alt="Screenshot 2026-03-27 194007" src="https://github.com/user-attachments/assets/3d5e3698-7bda-4eb8-8a39-9425949d4584" />
<img width="1159" height="428" alt="Screenshot 2026-03-27 194050" src="https://github.com/user-attachments/assets/4bee44a7-8b94-4758-b419-cc9790e00f77" />

### Step 4 — IP Address Configuration
| Instance | Private IP | Public IP |
|----------|-----------|-----------|
| k3s-master-1 | 172.31.84.232 | 54.164.209.126 |
| k3s-master-2 | 174.129.99.194 | 172.31.94.96 |
| k3s-master-3 | 54.204.170.9 | 172.31.95.191 |

### Part 2 - Nodes
Below steps will start from step 5 as a continuation of phase 1
### Step 5 - Preparing Nodes
1. **Hostname Setup:** `sudo hostnamectl set-hostname k3s-master-x`
2. **System Update:** `sudo apt-get update && sudo apt-get upgrade -y`
3. **Swap Management:** Disabled swap to ensure Kubelet stability:
   ```bash
   sudo swapoff -a
   sudo sed -i '/ swap / s/^/#/' /etc/fstab 
      ```

### Step 6 - Install K3s on First Master (k3s-master-1)
<img width="887" height="671" alt="Screenshot 2026-03-22 175754" src="https://github.com/user-attachments/assets/ff738ac0-7c95-44ce-a7c8-0b9bbd707801" />
<img width="1121" height="631" alt="Screenshot 2026-03-22 174041" src="https://github.com/user-attachments/assets/1da26e62-b824-4b11-be81-5da99726d458" />
<img width="1045" height="130" alt="Screenshot 2026-03-22 183031" src="https://github.com/user-attachments/assets/0567f568-6ed6-4cd4-b911-2bf3fa35515c" />


### Step 7 - Join k3s-master-2 as a Server Node
<img width="1234" height="149" alt="Screenshot 2026-03-22 183250" src="https://github.com/user-attachments/assets/f2551ba6-46dc-4054-9242-defb99bb4716" />


### Step 8 - Join k3s-master-3 as an Agent Node
<img width="997" height="581" alt="Screenshot 2026-03-22 183829" src="https://github.com/user-attachments/assets/27622c31-e946-4b21-bce4-58862fac6237" />
<img width="1001" height="650" alt="Screenshot 2026-03-22 191937" src="https://github.com/user-attachments/assets/46f377a8-e3c6-48db-8c59-43b90152f6d4" />

### Step 9 - Verify the Complete Cluster
<img width="1114" height="563" alt="Screenshot 2026-03-22 191937" src="https://github.com/user-attachments/assets/c919079f-a51f-42ad-90f8-582698659a53" />
<img width="1001" height="518" alt="Screenshot 2026-03-22 195641" src="https://github.com/user-attachments/assets/0e0f26a3-a502-498a-98b0-11d61ff443ca" />



## Architecture Explanation
K3s is a highly available, certified Kubernetes distribution designed for low-resource environments. It was chosen for this AWS deployment because:
Lightweight Footprint: It strips away legacy and non-essential features of standard Kubernetes (K8s), making it ideal for the t3.large EC2 instances used in this lab.
Edge & 5G Ready: In modern telecommunications, processing must happen at the "Edge" (closer to the user). K3s is the industry standard for these environments due to its efficiency.

The Control Plane
The "Control Plane" is the brain of the cluster. In this High-Availability (HA) setup, the control plane is spread across three master nodes to ensure that if one fails, the cluster stays online.
API Server: The front end of the control plane. All commands (like those sent from Windows PowerShell) go through here.
Scheduler: Decides which node should run which workload based on available resources.
Controller Manager: Monitors the state of the cluster and makes changes to reach the "desired state" (e.g., ensuring 3 copies of a pod are always running).
Embedded etcd: This is the distributed database. By using --cluster-init on Master-1, we created a shared "ledger" where all three nodes store cluster data. This ensures "Data Consistency" across the AWS environment.

3. High Availability (HA) 
By deploying 3 master nodes instead of 1, we achieved Fault Tolerance.
The cluster requires a majority of master nodes to be alive to function. With 3 nodes, the cluster can tolerate one node failing completely without any service interruption.
AWS Security Groups: The architecture relies on specific inbound rules (Ports 6443, 2379-2380) to allow these control plane components to talk to each other securely within the VPC.


## Reflection and Insight

This assignment improved my understanding of how cloud infrastructure and container orchestration work together in real-world systems. I gained practical experience deploying a distributed system and configuring communication between multiple nodes in a secure environment.
One key insight is the importance of networking in Kubernetes clusters. Many issues encountered during deployment were related to security group rules and inter-node communication rather than the K3s installation itself. This highlighted how critical proper network configuration is in distributed systems.
A major plot twist for me was the bridge between my local environment and the AWS cloud. Using the .pem key through Windows PowerShell to execute scp and ssh commands highlighted the realities of remote server management. Reflecting on the move from Step 3 (Single Master) to Step 4 (Multi-Master), the scalability of containerization became clear. In a 5G ecosystem, we need to deploy stuff quickly yet accurately.
Overall, this project strengthened my practical skills in cloud computing, Kubernetes, and system architecture, while also improving my troubleshooting and problem-solving abilities in a distributed environment.

Problem Solving: 
A key learning moment occurred when resolving the "Database is Locked" error. This challenge highlighted that in a multi-node environment, configuration consistency is important. It taught me the importance of clean configuration management and system health checks.

