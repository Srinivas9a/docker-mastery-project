# EC2 Instance Setup Guide

A professional, step-by-step guide for launching an Ubuntu EC2 instance on AWS and running the Docker Mastery Project.

> **Cost warning:** AWS resources may incur charges. Check your account's current Free Tier/credits eligibility, monitor Billing and Cost Management, and stop or terminate resources when they are no longer required.

---

## Architecture Overview

```text
Developer Machine
       |
       | SSH
       v
+---------------------------+
|        AWS EC2            |
|   Ubuntu Linux Server     |
|                           |
|       Docker Engine       |
|           |               |
|    +------+------+        |
|    |      |      |        |
| Phase 1 Phase 2 Phase 3   |
| :8000   :8001   :8002     |
+---------------------------+
       |
       v
   Internet / Browser
```

| Phase | Container Port | EC2 Host Port | URL |
|---|---:|---:|---|
| Phase 1 | 8000 | 8000 | `http://<EC2_PUBLIC_IP>:8000/docs` |
| Phase 2 | 8000 | 8001 | `http://<EC2_PUBLIC_IP>:8001/docs` |
| Phase 3 | 8000 | 8002 | `http://<EC2_PUBLIC_IP>:8002/docs` |

The applications listen on port `8000` inside their containers. Docker maps different EC2 host ports to that container port.

---

## 1. Prerequisites

Before starting, make sure you have:

- An AWS account
- Access to the AWS Management Console
- An SSH client
- The Docker Mastery Project on GitHub
- An SSH key pair for EC2 access

Choose the AWS Region carefully. EC2 instance availability, pricing, Free Tier eligibility, and other costs can vary by Region.

---

## 2. Open the EC2 Console

Open the [Amazon EC2 Console](https://console.aws.amazon.com/ec2/).

Select your intended AWS Region from the Region selector, then choose:

**Instances → Launch instances**

> Keep your resources in the intended Region so they are easy to manage and monitor.

---

## 3. Name the Instance

Under **Name and tags**, use a descriptive name such as:

```text
docker-mastery-server
```

This creates a `Name` tag for the EC2 instance.

---

## 4. Choose the AMI

Under **Application and OS Images**, select:

- **Ubuntu**
- **Ubuntu Server 22.04 LTS**
- **64-bit (x86)**

Ubuntu 22.04 LTS provides a familiar Linux server environment for the Docker deployment used in this project.

> AWS may display slightly different AMI labels depending on the Region and current catalog. Verify that you are selecting the official Ubuntu image appropriate for your Region.

---

## 5. Choose the Instance Type

The reference configuration for this project uses:

```text
t2.medium
```

Typical specifications:

| Resource | Specification |
|---|---:|
| vCPUs | 2 |
| Memory | 4 GiB |
| Family | T2 |
| CPU model | Burstable |

A `t2.medium` provides enough resources for the Docker demonstrations in this project.

> **Important:** `t2.medium` availability and pricing vary by Region, and it may not be the most cost-effective or modern option. If it is unavailable, choose an available burstable instance with comparable resources after checking the current AWS pricing and your account's Free Tier/credits eligibility.

---

## 6. Create a Key Pair

Under **Key pair (login)**, select **Create new key pair**.

Recommended values:

```text
Name: docker-mastery-key
Key pair type: RSA
Private key format: .pem
```

Click **Create key pair**.

Your browser will download the private key.

### Protect the key

On Linux/WSL:

```bash
mkdir -p ~/.ssh
mv ~/Downloads/docker-mastery-key.pem ~/.ssh/
chmod 400 ~/.ssh/docker-mastery-key.pem
```

Verify:

```bash
ls -l ~/.ssh/docker-mastery-key.pem
```

**Never upload the `.pem` file to GitHub.**

> AWS does not provide another copy of the private key after the key pair is created. Store it securely.

---

## 7. Configure Network Settings

Under **Network settings**, configure:

- **VPC:** Default VPC is acceptable for this learning project
- **Subnet:** Default subnet is acceptable
- **Auto-assign public IP:** Enable

Create a Security Group, for example:

```text
docker-mastery-sg
```

---

## 8. Configure the Security Group

A Security Group acts as a virtual firewall for the EC2 instance.

Recommended inbound rules:

| Type | Protocol | Port | Source | Purpose |
|---|---|---:|---|---|
| SSH | TCP | 22 | My IP | Secure administration |
| Custom TCP | TCP | 8000 | `0.0.0.0/0`* | Phase 1 demo |
| Custom TCP | TCP | 8001 | `0.0.0.0/0`* | Phase 2 demo |
| Custom TCP | TCP | 8002 | `0.0.0.0/0`* | Phase 3 demo |

### Security recommendations

For SSH, use:

```text
My IP
```

Do not expose SSH to the entire Internet unless there is a specific reason and additional controls are in place.

The application ports are shown as `0.0.0.0/0` because the portfolio demonstration is intended to be reachable from a browser. This exposes those ports publicly, so use this only for a temporary demonstration.

For a production architecture, prefer a load balancer/reverse proxy and expose only the traffic that is actually required.

> Security Groups are stateful: when an inbound connection is allowed, the corresponding response traffic is automatically allowed.

---

## 9. Configure Storage

Under **Configure storage**, use:

```text
Volume type: gp3
Size:        20 GiB
```

This provides space for:

- Docker images
- Containers
- Project source code
- Package data
- Logs

> EBS storage is billed separately from EC2 compute. Do not allocate more storage than you need.

---

## 10. Review and Launch

Before launching, verify:

```text
Name:              docker-mastery-server
OS:                Ubuntu Server 22.04 LTS
Architecture:      x86_64
Instance type:     t2.medium
Storage:           20 GiB gp3
Public IP:         Enabled
Security Group:    docker-mastery-sg
SSH:               Port 22 / My IP
Application ports: 8000, 8001, 8002
```

Click:

**Launch instance**

Wait until:

```text
Instance state: Running
Status checks: 2/2 checks passed
```

---

## 11. Find the Public IPv4 Address

Select the instance and locate:

```text
Public IPv4 address
```

In this guide, always represent it as:

```text
<EC2_PUBLIC_IP>
```

Do not permanently place your real public IP address in public documentation.

---

## 12. SSH Into the EC2 Instance

From your local Linux/WSL terminal:

```bash
ssh -i ~/.ssh/docker-mastery-key.pem ubuntu@<EC2_PUBLIC_IP>
```

Example:

```bash
ssh -i ~/.ssh/docker-mastery-key.pem ubuntu@3.XXX.XXX.XXX
```

For Ubuntu AMIs, the default login user is normally:

```text
ubuntu
```

After a successful connection, you should see an Ubuntu shell similar to:

```text
ubuntu@ip-10-0-1-25:~$
```

### SSH troubleshooting

If you receive a private-key permission warning:

```bash
chmod 400 ~/.ssh/docker-mastery-key.pem
```

If the connection times out, verify:

1. The EC2 instance is running.
2. A public IPv4 address is assigned.
3. Security Group TCP port `22` allows your current public IP.
4. You are using the correct Region and instance.
5. Your network/VPN is not interfering with SSH.

---

## 13. Install Docker

Once connected to the EC2 instance, follow the project's:

[Docker Installation Guide](DOCKER-INSTALLATION.md)

Verify Docker:

```bash
docker --version
```

Then:

```bash
docker info
```

Test the installation:

```bash
docker run hello-world
```

If the installation guide adds your user to the Docker group, start a new SSH session afterward so the group membership is refreshed.

---

## 14. Clone the Project

From the EC2 instance:

```bash
git clone https://github.com/Srinivasa9a/docker-mastery-project.git
```

Then:

```bash
cd docker-mastery-project
```

Verify:

```bash
ls
```

You should see the project directories.

---

# Phase 1 Deployment

## 15. Build Phase 1

```bash
cd ~/docker-mastery-project/phase1-beginner
```

Build the image:

```bash
docker build -t docker-mastery:phase1 .
```

Run the container:

```bash
docker run -d   --name phase1-app   -p 8000:8000   docker-mastery:phase1
```

Verify:

```bash
docker ps
```

Test from EC2:

```bash
curl http://localhost:8000/
curl http://localhost:8000/health
curl http://localhost:8000/info
```

Open from your browser:

```text
http://<EC2_PUBLIC_IP>:8000/docs
```

---

# Phase 2 Deployment

## 16. Build and Run Phase 2

Stop and remove Phase 1 if it is no longer needed:

```bash
docker stop phase1-app
docker rm phase1-app
```

Move to Phase 2:

```bash
cd ~/docker-mastery-project/phase2-intermediate
```

Build:

```bash
docker build -t docker-mastery:phase2 .
```

Run:

```bash
docker run -d   --name phase2-app   -p 8001:8000   docker-mastery:phase2
```

The mapping is:

```text
EC2 host port 8001 → container port 8000
```

Test:

```bash
curl http://localhost:8001/
curl http://localhost:8001/health
curl http://localhost:8001/info
```

Open:

```text
http://<EC2_PUBLIC_IP>:8001/docs
```

---

# Phase 3 Deployment

## 17. Build and Run Phase 3

Stop and remove Phase 2 if required:

```bash
docker stop phase2-app
docker rm phase2-app
```

Move to Phase 3:

```bash
cd ~/docker-mastery-project/phase3-advanced
```

Build:

```bash
docker build -t docker-mastery:phase3 .
```

Run:

```bash
docker run -d   --name phase3-app   -p 8002:8000   docker-mastery:phase3
```

Verify:

```bash
docker ps
```

Test:

```bash
curl http://localhost:8002/
curl http://localhost:8002/health
curl http://localhost:8002/info
```

Open:

```text
http://<EC2_PUBLIC_IP>:8002/docs
```

---

## 18. Verify the Phase 3 Health Check

Phase 3 includes a Docker `HEALTHCHECK`.

Check:

```bash
docker ps
```

The container should eventually display:

```text
(healthy)
```

You can query the status directly:

```bash
docker inspect phase3-app   --format='{{.State.Health.Status}}'
```

Expected:

```text
healthy
```

---

## 19. Verify Non-Root Execution

Phase 3 runs the application as a dedicated non-root user.

Run:

```bash
docker exec phase3-app whoami
```

Expected:

```text
appuser
```

This demonstrates the security improvement introduced in Phase 3.

---

## 20. Useful Docker Commands

List running containers:

```bash
docker ps
```

List all containers:

```bash
docker ps -a
```

List images:

```bash
docker images
```

View logs:

```bash
docker logs phase3-app
```

Follow logs:

```bash
docker logs -f phase3-app
```

Stop a container:

```bash
docker stop phase3-app
```

Start an existing container:

```bash
docker start phase3-app
```

Remove a stopped container:

```bash
docker rm phase3-app
```

Check Docker disk usage:

```bash
docker system df
```

---

## 21. Update the Deployment

After changes are pushed to GitHub:

```bash
cd ~/docker-mastery-project
git pull
```

Rebuild the desired phase.

Example:

```bash
cd phase3-advanced
docker build -t docker-mastery:phase3 .
```

Replace the running container:

```bash
docker stop phase3-app
docker rm phase3-app
```

Start the updated image:

```bash
docker run -d   --name phase3-app   -p 8002:8000   docker-mastery:phase3
```

Verify:

```bash
docker ps
curl http://localhost:8002/health
```

---

## 22. Troubleshooting

### Check container status

```bash
docker ps -a
```

### Check application logs

```bash
docker logs phase3-app
```

### Check port mapping

```bash
docker port phase3-app
```

### Check health

```bash
docker inspect phase3-app   --format='{{.State.Health.Status}}'
```

### Test from inside EC2

```bash
curl http://localhost:8002/health
```

If the local `curl` works but the public URL does not, check:

1. Docker port mapping.
2. The application is listening on `0.0.0.0:8000` inside the container.
3. The EC2 Security Group allows TCP `8002`.
4. The instance has a public IPv4 address.
5. No host firewall is blocking the port.

---

# 23. Stop vs. Terminate

| Action | What it does | Compute billing | Storage |
|---|---|---|---|
| **Stop** | Shuts down the instance | EC2 compute billing stops while stopped | EBS storage can continue to incur charges |
| **Start** | Starts a stopped instance | Compute billing resumes | Existing EBS volume remains |
| **Terminate** | Permanently deletes the instance | Instance is deleted | Root EBS volume is normally deleted according to its termination setting |

### When you finish working

If you will use the server again:

**Stop the instance.**

If you are completely finished with it:

**Terminate the instance.**

> Stopping an EC2 instance does not make every AWS-related charge zero. EBS storage and some other resources can continue to incur charges.

---

# 24. Public IP Consideration

A public IPv4 address can change when an EC2 instance is stopped and started.

Therefore, do not hard-code your current IP address into this documentation.

Use:

```text
<EC2_PUBLIC_IP>
```

If you require a stable public IP, investigate an Elastic IP or DNS-based solution and review the current AWS pricing before using it.

---

# 25. AWS Cost-Control Checklist

Before finishing an AWS session:

```text
[ ] EC2 instance stopped if no longer required
[ ] Unused EC2 instances terminated
[ ] Unused EBS volumes reviewed
[ ] Elastic IP resources reviewed
[ ] Unused snapshots reviewed
[ ] Unused load balancers reviewed
[ ] Unused NAT Gateways reviewed
[ ] Docker containers stopped when no longer needed
[ ] AWS Billing and Cost Management reviewed
```

For a learning project, avoid leaving resources running unnecessarily.

---

# 26. Security Checklist

Before considering the deployment production-ready:

- Keep SSH restricted to your IP.
- Never commit `.pem` files to GitHub.
- Never commit AWS access keys or passwords.
- Never commit `.env` files containing secrets.
- Prefer IAM roles over hard-coded AWS credentials.
- Expose only required network ports.
- Use HTTPS for production traffic.
- Keep Ubuntu and Docker updated.
- Monitor logs and resource usage.
- Remove unused AWS resources.
- Consider an Application Load Balancer or reverse proxy for production.

---

# 27. Recommended Production Evolution

The direct EC2 + public Docker-port setup in this guide is intentionally simple for learning.

A stronger production architecture could evolve toward:

```text
                    Internet
                       |
                       v
             +-------------------+
             | Route 53 / DNS    |
             +-------------------+
                       |
                       v
             +-------------------+
             | Application       |
             | Load Balancer     |
             +-------------------+
                       |
                       v
             +-------------------+
             | EC2 / ECS         |
             | Docker Workload   |
             +-------------------+
```

A CI/CD-oriented implementation could evolve into:

```text
GitHub
   |
   v
GitHub Actions
   |
   v
Amazon ECR
   |
   v
EC2 / ECS
   |
   v
Docker Container
```

This project can therefore progress from local Docker fundamentals toward a more complete cloud-native deployment workflow.

---

# 28. Deployment Verification Checklist

After deployment, verify:

### Container

```bash
docker ps
```

### Root endpoint

```bash
curl http://localhost:8002/
```

### Health endpoint

```bash
curl http://localhost:8002/health
```

### Info endpoint

```bash
curl http://localhost:8002/info
```

### Docker health status

```bash
docker inspect phase3-app   --format='{{.State.Health.Status}}'
```

Expected:

```text
healthy
```

### Non-root execution

```bash
docker exec phase3-app whoami
```

Expected:

```text
appuser
```

### Browser

```text
http://<EC2_PUBLIC_IP>:8002/docs
```

---

# 29. Cleanup

When the Docker demonstration is complete:

```bash
docker stop phase3-app
docker rm phase3-app
```

Optionally remove the image:

```bash
docker rmi docker-mastery:phase3
```

Review Docker storage:

```bash
docker system df
```

Then return to the AWS EC2 Console and **stop or terminate the EC2 instance**, depending on whether you need it again.

---

# 30. Project Documentation

Related documentation:

- [Project README](README.md)
- [Docker Installation Guide](DOCKER-INSTALLATION.md)
- [Phase 1 — Beginner](phase1-beginner/README.md)
- [Phase 2 — Intermediate](phase2-intermediate/README.md)
- [Phase 3 — Advanced](phase3-advanced/README.md)

---

## Summary

This guide demonstrates the complete deployment path:

```text
AWS EC2
   ↓
Ubuntu Linux
   ↓
Docker Engine
   ↓
GitHub Repository
   ↓
Docker Build
   ↓
FastAPI Container
   ↓
Health Check
   ↓
Public Application Endpoint
```

The Phase 3 deployment additionally demonstrates:

- Multi-stage Docker builds
- Alpine Linux
- Smaller runtime images
- Non-root container execution
- Docker health checks
- API validation
- AWS EC2 deployment

The result is a practical progression from **local Docker development to cloud-based container deployment**.

---

## Disclaimer

This guide is intended for educational and portfolio purposes. AWS console screens, AMI names, instance availability, pricing, Free Tier rules, and service behavior can change over time. Always verify the current AWS documentation and pricing for your account and Region before creating resources.
