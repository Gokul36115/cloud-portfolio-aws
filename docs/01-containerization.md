# Phase 1 — Containerization

**Date:** May 2026  
**Time taken:** ~5 hours

## Goal
Package the static portfolio site into a portable Docker image and publish it to Docker Hub for use in Phase 2 (AWS deployment).

## What was built
- Static portfolio website using HTML5 + modern CSS (no framework)
- Multi-section layout: Hero, About, Skills, Experience, Certifications, Projects, Contact
- Nginx-based Docker image using `nginx:alpine` (lightweight, ~93 MB)
- Custom Nginx config with security headers and process-level healthcheck
- Image published to Docker Hub: `gokul36115/gokul-portfolio:1.0` and `:latest`

## Architecture

Browser --> Docker Container (port 80) --> Nginx --> /usr/share/nginx/html/index.html

## Key files

### docker/Dockerfile
- Base image: `nginx:alpine`
- Copies `app/` into Nginx html root
- Mounts custom `nginx.conf` to override defaults
- Exposes port 80
- Healthcheck: `pgrep nginx` (process check, no network dependency)

### docker/nginx.conf
- Listens on port 80
- Serves `index.html` as default
- Adds security headers: X-Frame-Options, X-Content-Type-Options
- Logs to /var/log/nginx/access.log + error.log

## Build & run

    docker build -t gokul-portfolio:1.0 -f docker/Dockerfile .
    docker run -d -p 8080:80 --name gokul-portfolio gokul-portfolio:1.0

## Push to Docker Hub

    docker tag gokul-portfolio:1.0 gokul36115/gokul-portfolio:1.0
    docker tag gokul-portfolio:1.0 gokul36115/gokul-portfolio:latest
    docker push gokul36115/gokul-portfolio:1.0
    docker push gokul36115/gokul-portfolio:latest

## Image details

| Property | Value |
|---|---|
| Base | nginx:alpine |
| Size (compressed) | ~25 MB |
| Size (uncompressed) | ~93 MB |
| Exposed port | 80 |
| Healthcheck | pgrep nginx (interval 30s) |
| Tags | 1.0, latest |

## Key learnings

1. **Image vs Container** — image is the immutable build artifact, container is a running instance. Multiple containers can run from one image.
2. **Layer caching** — Docker caches each instruction as a layer. Putting fast-changing files (like app/) at the bottom of the Dockerfile keeps base layers cached, making rebuilds fast.
3. **Healthchecks** — Docker auto-monitors and reports (healthy)/(unhealthy) status. Initially used wget but it had connection-refused issues on Alpine BusyBox; switched to pgrep nginx (process-level) which is more reliable for static-site containers.
4. **Port mapping** — `-p 8080:80` maps host's 8080 to container's 80. Container internals always listen on the same port regardless of host mapping.
5. **--name** — naming containers makes docker stop, docker rm, docker logs much easier than dealing with auto-generated container IDs.

## Issues hit and fixes

| Issue | Cause | Fix |
|---|---|---|
| Container (unhealthy) despite site loading | BusyBox wget couldn't connect during early boot | Switched HEALTHCHECK to pgrep nginx |
| Image showed old timestamp after edit | Docker layer cache reused unchanged layers | Normal behavior; image was actually rebuilt |

## Next phase
Phase 2 — provision AWS VPC, EC2 instance, deploy this same image with HTTPS via Let's Encrypt + DuckDNS.