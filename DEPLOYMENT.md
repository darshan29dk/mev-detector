# 🚀 Deployment Guide

Complete guide for deploying MEV Detector to production.

## Prerequisites

- Docker & Docker Compose installed
- Alchemy API key (free)
- Domain name (for production)
- SSL certificate (for production)

## 1. Local Docker Development

### Build and run locally

```bash
# Build images
docker-compose build

# Start services
docker-compose up -d

# Check logs
docker-compose logs -f backend
docker-compose logs -f frontend

# Stop services
docker-compose down
```

Access at:
- Frontend: http://localhost:3000
- Backend: http://localhost:5000
- API: http://localhost:5000/api

## 2. Cloud Deployment

### Option A: Heroku (Backend + Frontend)

#### Backend Deployment

```bash
# Login to Heroku
heroku login

# Create app
heroku create mev-detector-api

# Set environment variables
heroku config:set -a mev-detector-api \
  ALCHEMY_API_KEY=your_key \
  ALCHEMY_ENDPOINT_MAINNET=https://eth-mainnet.g.alchemy.com/v2/ \
  ALCHEMY_ENDPOINT_SEPOLIA=https://eth-sepolia.g.alchemy.com/v2/ \
  NETWORK=sepolia \
  PORT=5000

# Deploy
cd server
git push heroku main

# View logs
heroku logs -a mev-detector-api --tail
```

#### Frontend Deployment

```bash
# Create frontend app
heroku create mev-detector-frontend

# Set environment variables
heroku config:set -a mev-detector-frontend \
  REACT_APP_API_URL=https://mev-detector-api.herokuapp.com/api \
  REACT_APP_WS_URL=wss://mev-detector-api.herokuapp.com

# Deploy
cd mev-detector
git push heroku main
```

### Option B: AWS (Backend)

#### Using ECS + Fargate

```bash
# Create ECR repository
aws ecr create-repository --repository-name mev-detector-backend

# Build and push Docker image
cd server
docker build -t mev-detector-backend .
docker tag mev-detector-backend:latest [ACCOUNT_ID].dkr.ecr.us-east-1.amazonaws.com/mev-detector-backend:latest
docker push [ACCOUNT_ID].dkr.ecr.us-east-1.amazonaws.com/mev-detector-backend:latest

# Create ECS cluster (via AWS Console or CLI)
aws ecs create-cluster --cluster-name mev-detector

# Create task definition
aws ecs register-task-definition --cli-input-json file://task-definition.json

# Create service
aws ecs create-service \
  --cluster mev-detector \
  --service-name mev-detector-backend \
  --task-definition mev-detector-backend \
  --desired-count 1 \
  --launch-type FARGATE \
  --network-configuration "awsvpcConfiguration={subnets=[subnet-xxx],securityGroups=[sg-xxx],assignPublicIp=ENABLED}"
```

#### Using EC2

```bash
# SSH into instance
ssh -i key.pem ec2-user@your-instance-ip

# Install Docker
sudo yum update -y
sudo yum install docker -y
sudo service docker start
sudo usermod -a -G docker ec2-user

# Clone repo
git clone https://github.com/your-org/mev-detector.git
cd mev-detector

# Create .env
echo "ALCHEMY_API_KEY=your_key" > server/.env

# Start with docker-compose
docker-compose up -d
```

### Option C: Vercel (Frontend Only)

```bash
# Install Vercel CLI
npm i -g vercel

# Deploy frontend
cd mev-detector
vercel --prod

# Set environment variables in Vercel dashboard
# REACT_APP_API_URL
# REACT_APP_WS_URL

# Connect to GitHub for auto-deploy
vercel link
```

### Option D: DigitalOcean (All-in-one)

```bash
# 1. Create droplet (at least 2GB RAM)
# 2. SSH into droplet

# 3. Install Docker
curl -sSL https://get.docker.com | sh
sudo usermod -aG docker $USER

# 4. Install Docker Compose
sudo curl -L "https://github.com/docker/compose/releases/download/v2.20.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose

# 5. Clone and setup
git clone https://github.com/your-org/mev-detector.git
cd mev-detector

# 6. Create .env files
cat > server/.env << EOF
ALCHEMY_API_KEY=your_key
ALCHEMY_ENDPOINT_MAINNET=https://eth-mainnet.g.alchemy.com/v2/
ALCHEMY_ENDPOINT_SEPOLIA=https://eth-sepolia.g.alchemy.com/v2/
PORT=5000
NETWORK=mainnet
EOF

cat > mev-detector/.env << EOF
REACT_APP_API_URL=https://api.yourdomain.com
REACT_APP_WS_URL=wss://api.yourdomain.com
EOF

# 7. Start services
docker-compose up -d

# 8. Setup SSL with Let's Encrypt
sudo apt-get install certbot python3-certbot-nginx
sudo certbot certonly --standalone -d api.yourdomain.com
```

## 3. Production Configuration

### Environment Variables

```env
# Backend
NODE_ENV=production
ALCHEMY_API_KEY=<your-key>
ALCHEMY_ENDPOINT_MAINNET=<url>
ALCHEMY_ENDPOINT_SEPOLIA=<url>
FLASHBOTS_RELAY_URL=https://relay.flashbots.net
FLASHBOTS_PROTECT_RPC=https://protected.flashbots.net
PORT=5000
NETWORK=mainnet
PREMIUM_FEATURE_FEE=0.001
SIMULATION_FEE=0.0001
HISTORICAL_DATA_FEE=0.005

# Frontend
REACT_APP_API_URL=https://api.yourdomain.com
REACT_APP_WS_URL=wss://api.yourdomain.com
REACT_APP_NETWORK=mainnet
```

### Security Checklist

- [ ] Enable HTTPS/WSS only
- [ ] Set up firewall rules
- [ ] Enable CORS for production domain only
- [ ] Use secrets manager for API keys
- [ ] Enable rate limiting
- [ ] Set up monitoring/logging (Sentry, New Relic)
- [ ] Enable auto-scaling
- [ ] Set up database backups
- [ ] Enable DDoS protection (Cloudflare)

### SSL/TLS Setup

#### With Let's Encrypt + Nginx

```bash
# Generate certificate
certbot certonly --standalone -d yourdomain.com

# Update nginx.conf
server {
    listen 443 ssl;
    server_name yourdomain.com;

    ssl_certificate /etc/letsencrypt/live/yourdomain.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/yourdomain.com/privkey.pem;

    # Your config...
}

# Auto-renew
certbot renew --quiet
```

## 4. Monitoring & Maintenance

### Health Checks

```bash
# Check backend health
curl https://api.yourdomain.com/api/health

# Check frontend
curl https://yourdomain.com

# Monitor with uptime service
# Setup at https://uptimerobot.com
```

### Logging

```bash
# View Docker logs
docker-compose logs --follow backend

# View system logs
journalctl -u docker --follow
```

### Updates

```bash
# Pull latest code
git pull origin main

# Rebuild images
docker-compose build

# Restart services
docker-compose up -d
```

## 5. Database Migration (if needed)

When adding database support:

```bash
# Run migrations
docker-compose exec backend npm run migrate

# Seed initial data
docker-compose exec backend npm run seed
```

## 6. Troubleshooting

### Service won't start

```bash
# Check logs
docker-compose logs backend

# Check port conflicts
sudo lsof -i :5000
sudo lsof -i :3000

# Rebuild
docker-compose build --no-cache
docker-compose up -d
```

### High memory usage

```bash
# Check Docker stats
docker stats

# Increase limits in docker-compose.yml
services:
  backend:
    deploy:
      resources:
        limits:
          memory: 1G
```

### SSL certificate issues

```bash
# Renew certificate
sudo certbot renew --force-renewal

# Check expiration
sudo certbot certificates
```

## 7. Performance Optimization

### Frontend

```bash
# Enable gzip compression
# Enable caching headers
# Code splitting (React)
# Image optimization
```

### Backend

```bash
# Connection pooling
# Caching (Redis)
# Compression middleware
# Rate limiting
```

## 8. Backup & Disaster Recovery

```bash
# Backup Docker volumes
docker run --rm -v mev-detector-data:/data \
  -v $(pwd):/backup alpine tar czf /backup/backup.tar.gz /data

# Restore from backup
docker run --rm -v mev-detector-data:/data \
  -v $(pwd):/backup alpine tar xzf /backup/backup.tar.gz -C /
```

---

**Questions?** Check the main README or open an issue.