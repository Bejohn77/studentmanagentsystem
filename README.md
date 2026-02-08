# Student Management System

A full-stack student management app (React + Node + MongoDB) with Docker setup and a one-command production deploy using `docker-compose.prod.yml`.

This README focuses on how to run the project on an AWS EC2 instance (recommended: Ubuntu 22.04 LTS or Amazon Linux 2) using Docker Compose.

---

## Quick links

- Project root: [README.md](README.md)
- Production compose: [docker-compose.prod.yml](docker-compose.prod.yml)
- Development compose: [docker-compose.yml](docker-compose.yml)

---

## Prerequisites (EC2)

- An EC2 instance (Ubuntu 22.04 LTS or Amazon Linux 2) with at least 1 vCPU and 2GB RAM
- Security Group open for:
  - `22` (SSH)
  - `80` (HTTP)
  - `443` (HTTPS) — optional, for TLS
- Docker & Docker Compose installed on the instance

---

## Step-by-step: Deploy on AWS EC2 (recommended)

1. Launch an EC2 instance (Ubuntu 22.04 LTS recommended).
   - Use a key pair you control.
   - Attach a Security Group allowing inbound 22, 80 (and 443 if using TLS).

2. SSH into the instance:

```bash
ssh -i /path/to/key.pem ubuntu@YOUR_EC2_PUBLIC_IP
```

3. Install Docker and Docker Compose (Ubuntu example):

```bash
sudo apt-get update
sudo apt-get install -y docker.io
sudo apt-get install -y docker-compose-plugin
sudo systemctl enable --now docker
sudo usermod -aG docker $USER
```

Log out and back in (or start a new SSH session) so the `docker` group change takes effect.

4. Clone this repository and switch to the project directory:

```bash
git clone <YOUR_REPO_URL>
cd pritom
```

5. Create or review the `.env` file at the project root. Example values (change for production):

```text
PORT=5000
MONGO_INITDB_ROOT_USERNAME=admin
MONGO_INITDB_ROOT_PASSWORD=StrongPasswordHere
MONGO_INITDB_DATABASE=student-management
MONGODB_URI=mongodb://admin:StrongPasswordHere@mongodb:27017/student-management?authSource=admin
NODE_ENV=production
```

Notes:
- If you plan to use an external MongoDB (Atlas or managed), set `MONGODB_URI` to that connection string and remove the built-in MongoDB service from the compose file (or keep it as a fallback).
- Never commit production credentials to the repo. Use AWS Secrets Manager or SSM Parameter Store for real deployments.

6. Start the production stack (single-port, Nginx reverse proxy):

```bash
docker compose -f docker-compose.prod.yml up -d --build
```

7. Verify services are running:

```bash
docker compose -f docker-compose.prod.yml ps
docker compose -f docker-compose.prod.yml logs -f
```

8. Test endpoints from the EC2 instance or your browser:

```bash
curl http://localhost/        # frontend
curl http://localhost/api/students   # API
```

If you see the frontend HTML and API JSON responses, the deployment is successful.

---

## TLS / HTTPS (optional but recommended)

You can terminate TLS with an AWS Application Load Balancer (recommended) or install Certbot on the EC2 host and configure Nginx for TLS. For a quick Certbot + Nginx approach:

```bash
sudo apt-get install -y certbot python3-certbot-nginx
sudo certbot --nginx -d your.domain.example
```

Follow Certbot prompts. After certificate issuance, Nginx will handle HTTPS traffic.

---

## Running locally (development)

Backend and frontend can be run locally without Docker during active development.

Backend:
```bash
npm install
npm run dev
```

Frontend:
```bash
cd frontend
npm install
npm start
```

Or run everything with Docker Compose (dev):

```bash
docker compose up --build
```

---

## Useful Docker Compose commands

- Start (prod): `docker compose -f docker-compose.prod.yml up -d --build`
- Start (dev): `docker compose up --build -d`
- Stop: `docker compose -f docker-compose.prod.yml down`
- View logs: `docker compose -f docker-compose.prod.yml logs -f`
- Rebuild images: `docker compose -f docker-compose.prod.yml up -d --build`

---

## Troubleshooting

- If containers fail to start, inspect logs: `docker compose -f docker-compose.prod.yml logs -f`
- If MongoDB is not healthy, allow extra startup time (initialization scripts run once).
- If ports conflict, ensure no host service is binding to ports 80/443.

---

## Security & production notes

- Replace all default credentials before going live.
- Use AWS Secrets Manager for sensitive values and inject them at runtime.
- Use an Application Load Balancer + ACM for TLS in production.
- Backup MongoDB regularly (mongodump or managed backups).

---

If you want, I can:
- generate a sample `.env.production` and `.env.example`
- add a systemd unit to auto-start Docker Compose on boot
- add a short shell script to perform one-click deploy on a fresh EC2 instance

Which of those would you like next?


For detailed AWS EC2 deployment steps, see [AWS_EC2_DEPLOYMENT.md](AWS_EC2_DEPLOYMENT.md)

For issues or questions, please refer to the documentation or contact the development team.
