# 🚀 NeroForge GUIDORA - Complete Startup Guide

> Step-by-step instructions to get GUIDORA running locally and in production

---

## 📋 Table of Contents

1. [Prerequisites](#prerequisites)
2. [Local Development Setup](#local-development-setup)
3. [Docker Compose Setup](#docker-compose-setup)
4. [Production Deployment](#production-deployment)
5. [Troubleshooting](#troubleshooting)
6. [Environment Configuration](#environment-configuration)

---

## ✅ Prerequisites

### System Requirements
- **OS**: Windows, macOS, or Linux
- **RAM**: Minimum 8GB (16GB recommended for Docker)
- **Disk Space**: 10GB free space
- **Internet**: Stable connection for package downloads

### Required Software

#### 1. Git
```bash
# Windows (via Chocolatey)
choco install git

# macOS (via Homebrew)
brew install git

# Verify installation
git --version
```

#### 2. Node.js 18+
```bash
# Download from https://nodejs.org
# Verify installation
node --version
npm --version
```

#### 3. Python 3.11+
```bash
# Download from https://www.python.org
# Verify installation
python --version
pip --version
```

#### 4. Go 1.22+  (Optional - for main backend)
```bash
# Download from https://golang.org
go version
```

#### 5. Docker 24+ & Docker Compose
```bash
# Download from https://www.docker.com/products/docker-desktop
docker --version
docker-compose --version
```

#### 6. Git Bash (Windows only)
```bash
# Install Git Bash during Git installation
```

### Required Credentials

- **Google Gemini API Key** - Get from [Google AI Studio](https://makersuite.google.com/app/apikey)
- **MongoDB Connection String** - Local or [MongoDB Atlas](https://www.mongodb.com/cloud/atlas)
- **Google Cloud Project** (Optional - for production deployment)
- **GitHub OAuth App** (Optional - for repo analysis feature)

---

## 🏠 Local Development Setup

This setup runs each service individually for maximum flexibility during development.

### Step 1: Clone the Repository

```bash
git clone https://github.com/your-org/NeroForge.git
cd NeroForge
```

### Step 2: Setup Environment Variables

```bash
# Copy the template
cp backend/configs/environments/.env.template .env

# Edit with your credentials
# Windows
notepad .env

# macOS/Linux
nano .env
```

**Essential .env variables:**
```env
# Database
MONGODB_URL=mongodb+srv://username:password@cluster.mongodb.net/guidora
MONGODB_DB_NAME=guidora_db

# Redis
REDIS_URL=redis://localhost:6379
REDIS_PASSWORD=your-redis-password

# AI
GEMINI_API_KEY=your-gemini-api-key
GOOGLE_PROJECT_ID=your-gcp-project

# Auth
JWT_SECRET=your-super-secret-jwt-at-least-32-chars
GOOGLE_CLIENT_ID=your-google-client-id.apps.googleusercontent.com
GOOGLE_CLIENT_SECRET=your-google-client-secret

# Frontend
FRONTEND_URL=http://localhost:3000
REACT_APP_API_GATEWAY_URL=http://localhost:3000
```

### Step 3: Start Redis (Standalone)

```bash
# Using Docker (Recommended)
docker run -d --name guidora-redis \
  -p 6379:6379 \
  -e REDIS_PASSWORD=your-redis-password \
  redis:7.2-alpine redis-server --requirepass your-redis-password

# OR use local Redis if installed
redis-server
```

### Step 4: Start MongoDB (Standalone)

```bash
# Using Docker (Recommended)
docker run -d --name guidora-mongo \
  -p 27017:27017 \
  -e MONGO_INITDB_ROOT_USERNAME=admin \
  -e MONGO_INITDB_ROOT_PASSWORD=password \
  mongo:7.0

# OR use MongoDB Atlas connection string in .env
```

### Step 5: Start Frontend (Terminal 1)

```bash
cd frontend

# Install dependencies
npm install

# Start development server
npm start

# Will open http://localhost:3000
```

### Step 6: Start API Gateway (Terminal 2)

```bash
cd backend/services/api-gateway

# Install dependencies
npm install

# Start server
npm start

# Listens on http://localhost:3000 (proxy layer)
```

### Step 7: Start User Service (Terminal 3)

```bash
cd backend/services/user-service

# Create virtual environment
python -m venv venv

# Activate
# Windows
venv\Scripts\activate
# macOS/Linux
source venv/bin/activate

# Install dependencies
pip install -r requirements.txt

# Start service
uvicorn src.main:app --reload --port 5001

# API: http://localhost:5001
# Docs: http://localhost:5001/docs
```

### Step 8: Start AI Guidance Service (Terminal 4)

```bash
cd backend/services/ai-guidance

# Create virtual environment
python -m venv venv

# Activate
# Windows
venv\Scripts\activate
# macOS/Linux
source venv/bin/activate

# Install dependencies
pip install -r requirements.txt

# Start service
uvicorn src.main:app --reload --port 5002

# API: http://localhost:5002
# Docs: http://localhost:5002/docs
```

### Step 9: Start Additional Services (Optional - Terminal 5+)

```bash
# Career Atlas Service (Port 5003)
cd backend/services/career-atlas-service
python -m venv venv && source venv/bin/activate
pip install -r requirements.txt
uvicorn src.main:app --reload --port 5003

# Portfolio Service (Port 5004)
cd backend/services/portfolio-service
python -m venv venv && source venv/bin/activate
pip install -r requirements.txt
uvicorn src.main:app --reload --port 5004

# Gamification Service (Port 5005)
cd backend/services/gamification-service
python -m venv venv && source venv/bin/activate
pip install -r requirements.txt
uvicorn src.main:app --reload --port 5005

# Mock Interview Service (Port 5006)
cd backend/services/mock-interview
python -m venv venv && source venv/bin/activate
pip install -r requirements.txt
uvicorn src.main:app --reload --port 5006
```

### Step 10: Verify All Services

```bash
# Check Frontend
curl http://localhost:3000

# Check API Gateway
curl http://localhost:3000/health

# Check User Service
curl http://localhost:5001/health

# Check AI Guidance
curl http://localhost:5002/health

# Access API Docs
# Swagger: http://localhost:5001/docs
# ReDoc: http://localhost:5001/redoc
```

---

## 🐳 Docker Compose Setup (Fastest)

### Option A: Complete Setup (All Services)

```bash
# Clone repository
git clone https://github.com/your-org/NeroForge.git
cd NeroForge

# Setup .env
cp backend/configs/environments/.env.template .env
# Edit .env with your credentials

# Start all services
docker-compose -f docker-compose-ultimate.yml up -d

# Check status
docker-compose -f docker-compose-ultimate.yml ps

# View logs
docker-compose -f docker-compose-ultimate.yml logs -f

# Access platform
# Frontend: http://localhost:3000
# API Gateway: http://localhost:3000
# API Docs: http://localhost:5001/docs
```

### Option B: Minimal Setup (Frontend + User Service)

```bash
# Create minimal docker-compose.yml
cat > docker-compose-minimal.yml << 'EOF'
version: '3.9'

services:
  redis:
    image: redis:7.2-alpine
    ports:
      - "6379:6379"
    command: redis-server --requirepass ${REDIS_PASSWORD}
    
  user-service:
    build: ./backend/services/user-service
    ports:
      - "5001:5001"
    env_file: .env
    depends_on:
      - redis
      
  frontend:
    build: ./frontend
    ports:
      - "3000:80"
    env_file: .env

networks:
  guidora_network:
    driver: bridge

volumes:
  redis_data:
EOF

# Start
docker-compose -f docker-compose-minimal.yml up -d
```

### Docker Compose Useful Commands

```bash
# View all running containers
docker-compose -f docker-compose-ultimate.yml ps

# View logs for specific service
docker-compose -f docker-compose-ultimate.yml logs api-gateway
docker-compose -f docker-compose-ultimate.yml logs user-service

# Follow real-time logs
docker-compose -f docker-compose-ultimate.yml logs -f

# Stop all services
docker-compose -f docker-compose-ultimate.yml down

# Stop and remove volumes
docker-compose -f docker-compose-ultimate.yml down -v

# Rebuild a specific service
docker-compose -f docker-compose-ultimate.yml build user-service

# Restart a service
docker-compose -f docker-compose-ultimate.yml restart user-service

# Scale a service (if stateless)
docker-compose -f docker-compose-ultimate.yml up -d --scale user-service=3
```

---

## 🚀 Production Deployment

### Deploy to Google Cloud Run

#### Prerequisites
```bash
# Install Google Cloud CLI
# https://cloud.google.com/sdk/docs/install

# Login to GCP
gcloud auth login
gcloud config set project YOUR_PROJECT_ID
```

#### Deploy Backend

```bash
cd backend

# Build image
docker build -t gcr.io/YOUR_PROJECT_ID/guidora-backend:latest .

# Push to Container Registry
docker push gcr.io/YOUR_PROJECT_ID/guidora-backend:latest

# Deploy to Cloud Run
gcloud run deploy guidora-backend \
  --image gcr.io/YOUR_PROJECT_ID/guidora-backend:latest \
  --platform managed \
  --region us-central1 \
  --port 8080 \
  --memory 512Mi \
  --allow-unauthenticated
```

#### Deploy Frontend

```bash
cd frontend

# Build image
docker build -t gcr.io/YOUR_PROJECT_ID/guidora-frontend:latest .

# Push to Container Registry
docker push gcr.io/YOUR_PROJECT_ID/guidora-frontend:latest

# Deploy to Cloud Run
gcloud run deploy guidora-frontend \
  --image gcr.io/YOUR_PROJECT_ID/guidora-frontend:latest \
  --platform managed \
  --region us-central1 \
  --port 80 \
  --memory 512Mi \
  --allow-unauthenticated
```

#### View Deployed URLs

```bash
gcloud run services list
```

#### Set Environment Variables in Cloud Run

```bash
gcloud run deploy guidora-backend \
  --image gcr.io/YOUR_PROJECT_ID/guidora-backend:latest \
  --set-env-vars GEMINI_API_KEY=your-key,MONGODB_URL=your-url \
  --region us-central1
```

---

## 🔧 Troubleshooting

### Port Already in Use

```bash
# Windows
netstat -ano | findstr :3000
taskkill /PID <PID> /F

# macOS/Linux
lsof -i :3000
kill -9 <PID>
```

### MongoDB Connection Failed

```bash
# Check MongoDB is running
mongo --version

# Test connection
mongosh "mongodb://localhost:27017"

# If using Atlas, verify:
# 1. Network whitelist includes your IP
# 2. Database user credentials are correct
# 3. Connection string is formatted correctly
```

### Redis Connection Failed

```bash
# Check Redis is running
redis-cli ping

# If returns PONG, Redis is working
# Otherwise restart Redis
redis-server
```

### Python Dependencies Issue

```bash
# Upgrade pip
pip install --upgrade pip

# Clear pip cache
pip cache purge

# Reinstall requirements
pip install -r requirements.txt --force-reinstall

# Check specific package
pip show fastapi
```

### Docker Build Fails

```bash
# Clear Docker cache
docker system prune -a

# Rebuild with no cache
docker build --no-cache -t guidora .

# Check Docker resources
docker stats
```

### Frontend Not Loading

```bash
# Check Node modules
rm -rf node_modules
npm install

# Clear npm cache
npm cache clean --force

# Restart development server
npm start

# Check browser console for errors
# Press F12 in browser
```

### Service Health Checks

```bash
# Frontend
curl http://localhost:3000

# API Gateway
curl http://localhost:3000/health

# User Service
curl http://localhost:5001/health

# AI Guidance
curl http://localhost:5002/health

# Career Atlas
curl http://localhost:5003/health

# All services
for port in 3000 5001 5002 5003 5004 5005 5006; do
  echo "Port $port:"
  curl -s http://localhost:$port/health | jq '.'
done
```

---

## 🔐 Environment Configuration

### Development .env

```env
NODE_ENV=development
ENVIRONMENT=development

# Database
MONGODB_URL=mongodb://localhost:27017/guidora
MONGODB_DB_NAME=guidora_db

# Redis
REDIS_URL=redis://localhost:6379
REDIS_PASSWORD=dev-password

# Auth
JWT_SECRET=dev-secret-at-least-32-chars-long-for-testing
JWT_ALGORITHM=HS256
JWT_EXPIRATION_MINUTES=1440

# AI
GEMINI_API_KEY=your-dev-gemini-key
GOOGLE_PROJECT_ID=dev-project

# OAuth (Optional)
GOOGLE_CLIENT_ID=dev-client-id
GOOGLE_CLIENT_SECRET=dev-client-secret
GITHUB_CLIENT_ID=dev-github-id
GITHUB_CLIENT_SECRET=dev-github-secret

# Frontend
FRONTEND_URL=http://localhost:3000
REACT_APP_API_GATEWAY_URL=http://localhost:3000

# Logging
LOG_LEVEL=DEBUG
```

### Production .env

```env
NODE_ENV=production
ENVIRONMENT=production

# Database (Use MongoDB Atlas)
MONGODB_URL=mongodb+srv://user:pass@cluster.mongodb.net/guidora-prod
MONGODB_DB_NAME=guidora_prod

# Redis (Use Redis Cloud or GCP Memorystore)
REDIS_URL=redis://:password@redis-server:6379
REDIS_PASSWORD=${REDIS_PASSWORD}

# Auth (Generate strong secrets)
JWT_SECRET=${JWT_SECRET}
JWT_ALGORITHM=HS256
JWT_EXPIRATION_MINUTES=1440

# AI
GEMINI_API_KEY=${GEMINI_API_KEY}
GOOGLE_PROJECT_ID=${GOOGLE_PROJECT_ID}

# OAuth
GOOGLE_CLIENT_ID=${GOOGLE_CLIENT_ID}
GOOGLE_CLIENT_SECRET=${GOOGLE_CLIENT_SECRET}
GITHUB_CLIENT_ID=${GITHUB_CLIENT_ID}
GITHUB_CLIENT_SECRET=${GITHUB_CLIENT_SECRET}

# Frontend
FRONTEND_URL=https://guidora.com
REACT_APP_API_GATEWAY_URL=https://api.guidora.com

# Security
CORS_ORIGINS=https://guidora.com,https://api.guidora.com
RATE_LIMIT_PER_MINUTE=60
MAX_RESUME_SIZE_MB=10

# Logging
LOG_LEVEL=INFO
```

---

## 📊 Service Ports

| Service | Port | Status |
|---------|------|--------|
| Frontend | 3000 | ✅ |
| API Gateway | 3000 | ✅ (proxy) |
| User Service | 5001 | ✅ |
| AI Guidance | 5002 | ✅ |
| Career Atlas | 5003 | ✅ |
| Portfolio | 5004 | ✅ |
| Gamification | 5005 | ✅ |
| Mock Interview | 5006 | ✅ |
| Notification | 5007 | ✅ |
| News Feeds | 5008 | ✅ |
| Redis | 6379 | ✅ |
| MongoDB | 27017 | ✅ |

---

## 📚 Next Steps

1. **API Exploration**
   - Visit http://localhost:5001/docs (Swagger UI)
   - Try test endpoints
   - Review request/response schemas

2. **Feature Testing**
   - Create a user account
   - Upload a resume
   - Generate career roadmap
   - Practice mock interview

3. **Frontend Development**
   - Explore React components in `/frontend/src/components`
   - Check page routes in `/frontend/src/pages`
   - Review Tailwind configuration

4. **Backend Development**
   - Review service structure in `/backend/services`
   - Understand API routes
   - Explore FastAPI documentation

5. **Deployment**
   - Set up CI/CD pipeline
   - Configure production environment
   - Deploy to cloud platform

---

## 💡 Tips & Best Practices

### Development
- Use `.env.local` for sensitive local overrides
- Keep terminal windows organized with labels
- Use VS Code REST Client extension for API testing
- Enable auto-format on save in IDE

### Docker
- Always tag images with version numbers
- Use `.dockerignore` to exclude unnecessary files
- Build images during off-peak hours
- Monitor container resource usage

### Performance
- Enable Redis caching for frequent queries
- Use database indexes for common queries
- Implement request batching where possible
- Monitor API response times

### Security
- Never commit `.env` files to git
- Rotate API keys regularly
- Use HTTPS in production
- Implement rate limiting
- Validate all user inputs

---

## 📞 Support

- **Issues**: https://github.com/your-org/NeroForge/issues
- **Documentation**: See [README.md](README.md)
- **API Docs**: http://localhost:5001/docs

---

**Happy coding! 🚀**
