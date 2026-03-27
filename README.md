# Assignment 1 — K3s High-Availability Cluster on AWS

**Full Name:** Musa Maswanganyi  
**Student Number:** 230019978  
**Module:** Communication Networks Practice 4  
**Lecturer:** Dr. Waldon Hendricks  
**Date:** 27 March 2026  

---

## ☁️ K3s on AWS Practical Setup Guide

### Step 1 — Create a Key Pair
A key pair was created to allow secure SSH access into the EC2 instances.

1. Navigated to **EC2 Dashboard → Network & Security → Key Pairs**  
2. Clicked **Create key pair**  
3. Configured:
   - Name: `k3s-key`
   - Type: RSA
   - Format: `.pem`
4. Set permissions locally:
```bash
