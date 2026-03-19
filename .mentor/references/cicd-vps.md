# CI/CD & VPS Reference — Java Spring Boot

> Load this file during Phase 5 (Week 9–10).
> Before following this guide, confirm: VPS provider, domain name, registry choice.

---

## VPS Setup Sequence (Ubuntu 22.04)

### 1. Initial security hardening
```bash
# Connect as root first, then:
adduser deploy
usermod -aG sudo deploy

# Copy your SSH key to the new user
rsync --archive --chown=deploy:deploy ~/.ssh /home/deploy

# Disable root SSH login
sed -i 's/PermitRootLogin yes/PermitRootLogin no/' /etc/ssh/sshd_config
systemctl restart sshd
```

### 2. Firewall
```bash
ufw allow OpenSSH
ufw allow 80/tcp
ufw allow 443/tcp
ufw enable
```

### 3. Install Docker
```bash
curl -fsSL https://get.docker.com | sh
sudo usermod -aG docker deploy
```

### 4. Nginx as reverse proxy
```bash
apt install nginx certbot python3-certbot-nginx -y
```

Nginx config (`/etc/nginx/sites-available/your-app`):
```nginx
server {
    server_name yourdomain.com;

    location / {
        proxy_pass http://localhost:8080;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_cache_bypass $http_upgrade;

        # Spring Boot apps can be slow to start
        proxy_connect_timeout 60s;
        proxy_read_timeout 60s;
    }
}
```

Enable the site:
```bash
ln -s /etc/nginx/sites-available/your-app /etc/nginx/sites-enabled/
nginx -t
systemctl reload nginx
```

### 5. SSL
```bash
certbot --nginx -d yourdomain.com
```

---

## GitHub Actions Workflow — Spring Boot

```yaml
name: CI/CD Pipeline

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Set up JDK 21
        uses: actions/setup-java@v4
        with:
          java-version: '21'
          distribution: 'temurin'
          cache: 'maven'

      - name: Run tests
        run: ./mvnw verify -B

  build-and-deploy:
    needs: test
    if: github.ref == 'refs/heads/main' && github.event_name == 'push'
    runs-on: ubuntu-latest
    permissions:
      packages: write
      contents: read

    steps:
      - uses: actions/checkout@v4

      - name: Log in to GitHub Container Registry
        uses: docker/login-action@v3
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - name: Build and push Docker image
        uses: docker/build-push-action@v5
        with:
          context: .
          push: true
          tags: |
            ghcr.io/${{ github.repository }}:${{ github.sha }}
            ghcr.io/${{ github.repository }}:latest

      - name: Deploy to VPS
        uses: appleboy/ssh-action@v1
        with:
          host: ${{ secrets.VPS_HOST }}
          username: ${{ secrets.VPS_USER }}
          key: ${{ secrets.VPS_SSH_KEY }}
          script: |
            # Pull the new image
            docker pull ghcr.io/${{ github.repository }}:${{ github.sha }}

            # Save current SHA for rollback
            docker inspect app --format='{{.Config.Image}}' > /home/deploy/last_stable_image.txt 2>/dev/null || true

            # Stop and remove old container
            docker stop app || true
            docker rm app || true

            # Run new container
            docker run -d \
              --name app \
              --restart unless-stopped \
              -p 8080:8080 \
              --env-file /home/deploy/.env \
              ghcr.io/${{ github.repository }}:${{ github.sha }}

            # Wait for health check
            echo "Waiting for app to start..."
            sleep 30
            if curl -sf http://localhost:8080/actuator/health > /dev/null; then
              echo "✅ Deployment successful"
            else
              echo "❌ Health check failed — rolling back"
              docker stop app || true
              docker rm app || true
              LAST_IMAGE=$(cat /home/deploy/last_stable_image.txt)
              docker run -d --name app --restart unless-stopped -p 8080:8080 --env-file /home/deploy/.env $LAST_IMAGE
              exit 1
            fi
```

---

## .env File on VPS (`/home/deploy/.env`)

```properties
SPRING_PROFILES_ACTIVE=prod
SPRING_DATASOURCE_URL=jdbc:postgresql://localhost:5432/myapp
SPRING_DATASOURCE_USERNAME=myapp_user
SPRING_DATASOURCE_PASSWORD=strong_password_here
SPRING_JPA_HIBERNATE_DDL_AUTO=validate
```

> **Never commit this file to Git.** Create it manually on the VPS.

---

## Required GitHub Secrets

| Secret | Value |
|--------|-------|
| `VPS_HOST` | IP address of the server |
| `VPS_USER` | `deploy` |
| `VPS_SSH_KEY` | Private SSH key (the full key, starts with `-----BEGIN`) |

`GITHUB_TOKEN` is provided automatically by GitHub Actions — no setup needed.
It grants access to GHCR for the current repository.

---

## Rollback Strategy

```bash
# On the VPS — roll back to previous image
docker stop app
docker rm app

# Read the last stable image from file
LAST_IMAGE=$(cat /home/deploy/last_stable_image.txt)
docker run -d \
  --name app \
  --restart unless-stopped \
  -p 8080:8080 \
  --env-file /home/deploy/.env \
  $LAST_IMAGE
```

Teach the student:
- Always know the last working image before deploying
- The workflow auto-saves it to `last_stable_image.txt`
- If the health check fails, the workflow auto-rolls back

---

## Reading Logs in Production

```bash
# Spring Boot container logs (live)
docker logs app -f

# Last 100 lines
docker logs app --tail 100

# Since a specific time
docker logs app --since 2024-01-01T00:00:00

# Search for exceptions
docker logs app 2>&1 | grep -i "exception"

# System service logs
journalctl -u nginx -f
```

---

## Database on VPS

For production, run PostgreSQL via Docker Compose on the VPS:

```yaml
# /home/deploy/docker-compose.yml (on VPS)
services:
  db:
    image: postgres:16-alpine
    environment:
      POSTGRES_DB: myapp
      POSTGRES_USER: myapp_user
      POSTGRES_PASSWORD: strong_password_here
    ports:
      - "127.0.0.1:5432:5432"    # bind to localhost only!
    volumes:
      - db_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U myapp_user -d myapp"]
      interval: 10s
      timeout: 5s
      retries: 5
    restart: unless-stopped

volumes:
  db_data:
```

> Bind to `127.0.0.1` so the database is NOT exposed to the internet.

---

## Deployment Checklist

Before going live:

- [ ] SSH key is on VPS, root login disabled
- [ ] Firewall allows only 22, 80, 443
- [ ] Nginx config is valid (`nginx -t`)
- [ ] SSL certificate installed and auto-renewing (`certbot renew --dry-run`)
- [ ] `.env` file on VPS has correct `SPRING_PROFILES_ACTIVE=prod`
- [ ] `application-prod.yml` uses `ddl-auto: validate` (NOT `update` or `create`)
- [ ] GitHub Secrets set for VPS_HOST, VPS_USER, VPS_SSH_KEY
- [ ] Actuator health endpoint returns `{"status":"UP"}`
- [ ] Push to main triggers the full workflow
- [ ] Health check auto-rollback works (test it!)
- [ ] Student can explain every step of the workflow YAML
- [ ] Student knows how to rollback manually
- [ ] Student can read container logs and find exceptions
