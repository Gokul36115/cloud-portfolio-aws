# Phase 2 — AWS VPC + EC2 Setup

## Goal
Build production-grade AWS infrastructure to host the containerized portfolio: custom VPC, EC2 instance, security group, and run the Docker image from Phase 1.

## Networking Architecture

| Resource | Name | CIDR / Detail |
|---|---|---|
| Region | Mumbai (ap-south-1) | Closest to Chennai |
| VPC | `gokul-portfolio-vpc` | `10.0.0.0/16` (65k IPs) |
| Subnet | `gokul-public-subnet-1a` | `10.0.1.0/24`, AZ `ap-south-1a`, public |
| Internet Gateway | `gokul-igw` | Attached to VPC |
| Route Table | `gokul-public-rt` | `0.0.0.0/0 → gokul-igw` |
| EC2 Instance | `gokul-portfolio-server` | `t3.micro`, Amazon Linux 2023 |
| Security Group | `gokul-portfolio-sg` | SSH (22, my IP), HTTP (80, any), HTTPS (443, any) |

## Steps Taken

### 1. Account security
- Enabled MFA on AWS root account (TOTP via authenticator app)
- Created IAM user `gokul-cloud-admin` with `AdministratorAccess` policy
- Configured AWS CLI on laptop to use IAM user (NOT root)

### 2. VPC creation
- Created VPC manually (NOT default) to demonstrate networking knowledge
- Created public subnet with auto-assign public IPv4 enabled
- Created and attached Internet Gateway
- Created route table with `0.0.0.0/0 → IGW` route
- Associated subnet with route table
- Enabled DNS hostnames on VPC

### 3. EC2 launch
- Launched `t3.micro` (free tier eligible) inside the custom VPC
- Used Amazon Linux 2023 AMI
- Created RSA key pair `gokul-portfolio-key.pem` (saved outside Git repo)
- Created security group with: SSH from my IP only, HTTP/HTTPS from anywhere

### 4. SSH and Docker install
- Locked down `.pem` permissions with `icacls` on Windows
- SSH'd in: `ssh -i key.pem ec2-user@<public-ip>`
- Installed Docker via `dnf install -y docker`
- Enabled Docker on boot: `systemctl enable --now docker`
- Added `ec2-user` to `docker` group (no sudo needed)

### 5. Run the portfolio container
- Pulled image from Docker Hub: `docker pull gokul36115/gokul-portfolio:latest`
- Ran with restart policy: `--restart unless-stopped`
- Mapped host port 80 → container port 80
- Site went live at `http://<public-ip>` 

### 6. DuckDNS for free domain
- Signed up for DuckDNS via GitHub OAuth
- Picked subdomain `gokul-portfolio.duckdns.org`
- Pointed it to EC2 public IP
- Verified DNS resolution

## Key Learnings

1. **Custom VPC vs Default VPC** — building manually demonstrates real networking understanding (CIDR, route tables, IGW)
2. **Security group rules are stateful** — outbound traffic auto-allowed back; only inbound rules need explicit definitions
3. **Public IP changes on stop/start** — that's why we need DuckDNS or Elastic IP
4. **`ec2-user` in docker group** — saves typing `sudo` on every command

## Cost
- VPC, IGW, route tables, subnet: **Free**
- EC2 t3.micro: **Free tier** (or covered by AWS credits)
- Total monthly cost during free tier: **$0**

## Next Phase
Phase 2.7 — add HTTPS with Let's Encrypt → see `03-https-letsencrypt.md`