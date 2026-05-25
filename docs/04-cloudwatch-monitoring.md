# Phase 3 — CloudWatch Monitoring (In Progress)

## Goal
Add observability to the portfolio deployment: ship metrics + logs to CloudWatch, set up alarms, build a dashboard.

## What was built (so far)

### Phase 3.1 — IAM Role for EC2
Created IAM role `gokul-ec2-cloudwatch-role` with policies:
- `CloudWatchAgentServerPolicy` — write metrics + logs
- `AmazonSSMManagedInstanceCore` — bonus: console-based EC2 access (Session Manager)

Attached the role to the EC2 instance via Console → EC2 → Actions → Security → Modify IAM role.

**Why a role, not access keys?** EC2 gets temporary, auto-rotating credentials. Zero secrets stored on disk. Standard production pattern. Verified with:
```bash
aws sts get-caller-identity
# Arn: arn:aws:sts::*:assumed-role/gokul-ec2-cloudwatch-role/<instance-id>

Installed and configured the CloudWatch Agent to ship custom metrics and logs.

By default, AWS only collects basic metrics from outside the EC2 (CPU, network, disk IOPS). The agent runs inside the OS to collect:

Memory usage % (not visible from outside)
Disk space % (not visible from outside)
Custom application logs
Install:

sudo dnf install -y amazon-cloudwatch-agent
Config at 
amazon-cloudwatch-agent.json
:

Custom namespace: GokulPortfolio
Metrics: CPU (idle/user/system), memory used %, disk used % (root volume)
Logs: /var/log/messages → log group gokul-portfolio/system (7-day retention)
Logs: 
cert-renew.log
 → log group gokul-portfolio/cert-renewal (30-day retention)
Start:

sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl \
  -a fetch-config -m ec2 \
  -c file:/opt/aws/amazon-cloudwatch-agent/etc/amazon-cloudwatch-agent.json -s
Verified metrics arriving in CloudWatch Console under Metrics → Custom namespaces → GokulPortfolio.