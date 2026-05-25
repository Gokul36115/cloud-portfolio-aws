
# Phase 2.7 — HTTPS with Let's Encrypt

## Goal
Replace HTTP with production-grade HTTPS using a free, browser-trusted Let's Encrypt certificate. Set up zero-touch auto-renewal so the cert renews itself every ~90 days.

## What was built
- Free 90-day SSL cert from Let's Encrypt for `gokul-portfolio.duckdns.org`
- Updated Nginx config with TLS 1.2/1.3, HSTS, modern cipher suite
- HTTP-to-HTTPS 301 redirect (no plain-text traffic)
- Cron-based auto-renewal with pre/post hooks (stop/start container)
- Renewal config baked into `/etc/letsencrypt/renewal/*.conf` (production-grade)

## Step-by-step

### 1. Install certbot

sudo dnf install -y python3 python3-pip augeas-libs
sudo python3 -m venv /opt/certbot/
sudo /opt/certbot/bin/pip install --upgrade pip
sudo /opt/certbot/bin/pip install certbot
sudo ln -sf /opt/certbot/bin/certbot /usr/bin/certbot
=====================================================================
2. Get the certificate (standalone mode)
Stopped Docker container temporarily so certbot could bind port 80:
docker stop portfolio
sudo certbot certonly --standalone \
  -d gokul-portfolio.duckdns.org \
  --email gokul36115@gmail.com \
  --agree-tos --no-eff-email
Certbot performed HTTP-01 ACME challenge:

Spins up mini-webserver on port 80
Let's Encrypt requests http://gokul-portfolio.duckdns.org/.well-known/acme-challenge/<token>
Certbot serves the matching response
Let's Encrypt verifies domain ownership → issues cert
=======================================================================================================
3. Update Nginx config
Two server blocks: port 80 redirects to HTTPS, port 443 serves the site over TLS with security headers.
=========================================================================================================
4. Rebuild and push image v2.0
docker build -t gokul36115/gokul-portfolio:2.0 -f docker/Dockerfile .
docker tag gokul36115/gokul-portfolio:2.0 gokul36115/gokul-portfolio:latest
docker push gokul36115/gokul-portfolio:2.0
docker push gokul36115/gokul-portfolio:latest
==============================================================================
5. Run on EC2 with cert volume mount
docker pull gokul36115/gokul-portfolio:latest
docker rm -f portfolio
docker run -d \
  -p 80:80 -p 443:443 \
  --restart unless-stopped \
  -v /etc/letsencrypt:/etc/letsencrypt:ro \
  --name portfolio \
  gokul36115/gokul-portfolio:latest
  ========================================================================
6. Auto-renewal setup
Installed cron daemon (Amazon Linux 2023 doesn't include it by default):

sudo dnf install -y cronie
sudo systemctl enable --now crond
Added pre/post hooks to renewal config:

[renewalparams]
...
pre_hook = docker stop portfolio
post_hook = docker start portfolio
=====================================================================================================================================================
Cron entry:

0 3 * * * /usr/bin/certbot renew --quiet >> /var/log/cert-renew.log 2>&1