# Datacanvas Deployment Guide

## Prerequisites

- Ubuntu 24.04 VPS with root or sudo access
- Docker and Docker Compose installed
- Remote database credentials

## Installation Steps

### 1. Install Docker and Docker Compose

```bash
sudo apt update
sudo apt install -y docker.io docker-compose
sudo systemctl start docker
sudo systemctl enable docker
```

### 2. Create Deployment Directory

```bash
mkdir -p ~/datacanvas/nginx
cd ~/datacanvas
```

### 3. Create Configuration Files

Create the following files with your actual configuration values:

**docker-compose.yml**
```yaml
version: '3.8'

services:
  backend:
    image: ghcr.io/datacanvas-iot/datacanvas-back:latest
    container_name: datacanvas-backend
    env_file:
      - backend.env
    restart: unless-stopped
    networks:
      - datacanvas-network
    expose:
      - "8000"

  frontend:
    image: ghcr.io/datacanvas-iot/datacanvas-front:latest
    container_name: datacanvas-frontend
    env_file:
      - frontend.env
    restart: unless-stopped
    networks:
      - datacanvas-network
    expose:
      - "3000"

  nginx:
    image: nginx:alpine
    container_name: datacanvas-nginx
    ports:
      - "80:80"
    volumes:
      - ./nginx/default.conf:/etc/nginx/conf.d/default.conf
    depends_on:
      - backend
      - frontend
    restart: unless-stopped
    networks:
      - datacanvas-network

networks:
  datacanvas-network:
    driver: bridge
```

**backend.env**
```
DB_NAME=your_database_name
DB_USER=your_database_user
DB_PASSWORD=your_database_password
DB_HOST=your_database_host
DB_PORT=5432
```

**frontend.env**
```
REACT_APP_API_URL=http://your-server-ip/api
REACT_APP_OPENAPI_KEY=your_openai_api_key
REACT_APP_ANALYTICS_API_URL=http://your-server-ip/analytics
```

**nginx/default.conf**
```nginx
upstream backend {
    server backend:8000;
}

upstream frontend {
    server frontend:3000;
}

server {
    listen 80;
    server_name _;

    client_max_body_size 100M;

    location / {
        proxy_pass http://frontend;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    location /api {
        proxy_pass http://backend;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    location /analytics {
        proxy_pass http://backend;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

### 4. Configure Environment Files

Edit `backend.env` and `frontend.env` with your actual credentials and URLs.

Replace `your-server-ip` in `frontend.env` with your VPS public IP address or domain name.

### 5. Deploy Application

```bash
cd ~/datacanvas
sudo docker-compose pull
sudo docker-compose up -d
```

### 6. Verify Deployment

Check if all containers are running:
```bash
sudo docker-compose ps
```

View logs:
```bash
sudo docker-compose logs -f
```

### 7. Access Application

Open your browser and navigate to:
```
http://your-server-ip
```

## Management Commands

### Stop Application
```bash
sudo docker-compose down
```

### Restart Application
```bash
sudo docker-compose restart
```

### Update to Latest Images
```bash
sudo docker-compose pull
sudo docker-compose up -d
```

### View Logs
```bash
# All services
sudo docker-compose logs -f

# Specific service
sudo docker-compose logs -f backend
sudo docker-compose logs -f frontend
sudo docker-compose logs -f nginx
```

## Firewall Configuration

If using UFW firewall:
```bash
sudo ufw allow 80/tcp
sudo ufw reload
```

## Troubleshooting

### Check Container Status
```bash
sudo docker-compose ps
```

### Check Logs for Errors
```bash
sudo docker-compose logs backend
sudo docker-compose logs frontend
sudo docker-compose logs nginx
```

### Restart Specific Service
```bash
sudo docker-compose restart backend
```

### Remove and Recreate Containers
```bash
sudo docker-compose down
sudo docker-compose up -d
```
