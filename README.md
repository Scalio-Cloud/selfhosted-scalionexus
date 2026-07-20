# Self-Hosted EntraConsole

This repository contains the Docker Compose configuration for self-hosting **ScalioNexus**. The architecture includes a frontend, a backend, a MongoDB database, and an Nginx reverse proxy with automated SSL certification via Certbot (Let's Encrypt).

For more detailed information, documentation, and updates, please visit our official website at [scalionexus.com](https://www.scalionexus.com).

## Prerequisites

Before starting the installation, ensure that the following components are installed on your host system:
* Docker and Docker Compose (Plugin)
* A registered domain with DNS records (A-Record) pointing to the public IP address of your server.

## Project Structure

Your repository layout should look like this:
```text
.
├── docker-compose.yml
├── .env
└── nginx.conf
```

## Installation Steps

### 1. Clone the Repository
Clone the repository to your server:
```bash
git clone https://github.com/Scalio-Cloud/selfhosted-scalionexus.git
cd selfhosted-scalionexus
```

### 2. Configure Environment Variables
Create or edit the `.env` file in the root directory and update the values to match your environment:

```env
JWT_SECRET="your_secure_random_secret"
PUBLIC_URL="https://scalionexus.yourdomain.com"
MONGO_URL="mongodb://mongo:27017"
DB_NAME="scalionexus"
LICENSE_KEY="your_license_key"
CORS_ORIGINS="*"
```

> **Important:** Replace `scalionexus.domain.com` in both your `.env` file and `nginx.conf` with your actual domain name.

### 3. Initial SSL Certificate Generation
Because Nginx references the SSL certificate files directly in its configuration, starting the stack will fail if those certificates do not exist yet. Follow these steps for the initial setup:

1. Temporarily comment out the entire `server` block for port 443 in your `nginx.conf`.
2. Start only the proxy and certbot services:
   ```bash
   docker compose up -d proxy certbot
   ```
3. Run the Certbot command manually to request your first certificate:
   ```bash
   docker compose run --rm certbot certonly --webroot -w /var/www/certbot -d scalionexus.yourdomain.com
   ```
4. Uncomment the port 443 SSL block in your `nginx.conf` to restore the secure configuration.

### 4. Start the Entire Stack
Once the certificates are generated and the configuration is restored, you can launch all services:

```bash
docker compose down
docker compose up -d
```

The Certbot container is configured to automatically check and renew the SSL certificate every 12 hours.

## Data Persistence

The MongoDB data is persisted directly on the host filesystem under the path `/data/mongo-data`. Ensure that the Docker daemon has the appropriate read and write permissions for this directory.

## Security & Proxy Rules

The Nginx proxy implements strict routing rules for the `/api` endpoint. Standard API requests are protected by verifying the HTTP Referer header against your `PUBLIC_URL` to block unauthorized cross-site requests with a `403 Forbidden` response. 

The following endpoints are explicitly excluded from this restriction to allow external connectivity:
* `/api/agent` — Used for agent communication.
* `/api/auth/sso/` — Used for handling Single Sign-On workflows.
