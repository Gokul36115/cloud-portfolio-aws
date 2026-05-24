\# Cloud-Native Portfolio Deployment on AWS



End-to-end personal portfolio deployment using Docker, Nginx, AWS EC2, Let's Encrypt SSL, and CloudWatch monitoring. Built as hands-on AWS learning project.



\## Live Demo

🌐 \*Coming in Phase 2 — https://\&lt;subdomain>.duckdns.org\*



\## Stack

\- \*\*Frontend:\*\* HTML + CSS (static portfolio)

\- \*\*Container:\*\* Docker (`nginx:alpine` base, \~93 MB)

\- \*\*Web Server:\*\* Nginx with custom config + security headers

\- \*\*Cloud:\*\* AWS EC2 in custom VPC (Mumbai region)

\- \*\*SSL/TLS:\*\* Let's Encrypt (auto-renewal via cron)

\- \*\*Monitoring:\*\* AWS CloudWatch Agent (metrics + logs)



\## Project Phases

\- \[x] \*\*Phase 1:\*\* Containerize the portfolio (Docker + Nginx)

\- \[ ] \*\*Phase 2:\*\* Deploy to AWS with HTTPS (VPC + EC2 + Let's Encrypt)

\- \[ ] \*\*Phase 3:\*\* CloudWatch monitoring, alarms, runbook



\## Repository Structure

