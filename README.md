# Cloud-Native Portfolio Deployment on AWS

End-to-end personal portfolio deployment using Docker, Nginx, AWS EC2, Let's Encrypt SSL, and CloudWatch monitoring. Built as hands-on AWS learning project.

## 🌐 Live Demo
🔒 **https://gokul-portfolio.duckdns.org**

## Stack
- **Frontend:** HTML + CSS (static portfolio)
- **Container:** Docker (`nginx:alpine` base, ~25 MB compressed)
- **Web Server:** Nginx with TLS 1.2/1.3, HSTS, security headers
- **Cloud:** AWS EC2 (`t3.micro`) in custom VPC (Mumbai region)
- **SSL/TLS:** Let's Encrypt cert with auto-renewal (cron + pre/post hooks)
- **DNS:** DuckDNS (free dynamic DNS)
- **Image Registry:** Docker Hub (`gokul36115/gokul-portfolio`)

## Project Phases
- [x] **Phase 1:** Containerize the portfolio (Docker + Nginx)
- [x] **Phase 2:** Deploy to AWS with HTTPS (VPC + EC2 + Let's Encrypt)
- [ ] **Phase 3:** CloudWatch monitoring, alarms, runbook

## Repository Structure
cloud-portfolio-aws/ ├── app/ # Static portfolio site │ ├── index.html │ └── style.css ├── docker/ # Container build files │ ├── Dockerfile │ └── nginx.conf # HTTPS config with security headers ├── docs/ # Phase write-ups │ ├── 01-containerization.md │ ├── 02-vpc-and-ec2-setup.md │ └── 03-https-letsencrypt.md ├── .gitignore ├── LICENSE └── README.md