# Docker Setup Guide

This guide explains how to use the Docker configuration for the MultienvApp project.

## Files Created

- **backend/dev/Dockerfile** - Development environment for Flask backend
- **backend/prod/Dockerfile** - Production environment for Flask backend with Gunicorn
- **frontend/Dockerfile** - Production build for React frontend
- **docker-compose.yml** - Orchestration file for all services
- **.env.example** - Template for environment variables
- **.env.dev** - Development environment variables
- **.env.prod** - Production environment variables

## Prerequisites

- Docker Desktop installed and running
- Docker Compose installed
- Git (optional)

## Getting Started

### 1. Clone/Setup the Project

```bash
cd MultienvApp
```

### 2. Create Environment File

Copy the example env file to create your local configuration:

```bash
# For development
cp .env.example .env
# or use the dev-specific file
cp .env.dev .env
```

### 3. Run with Development Configuration

```bash
# Run dev backend with MongoDB and frontend
docker-compose --profile dev up -d

# Or build and run
docker-compose --profile dev up --build
```

This starts:
- MongoDB on port 27017
- Backend (Flask) dev server on port 5000 with auto-reload
- Frontend (React) on port 3000

### 4. Run with Production Configuration

```bash
# Run prod backend with MongoDB and frontend
docker-compose --profile prod up -d

# Or build and run
docker-compose --profile prod up --build
```

This starts:
- MongoDB on port 27017
- Backend (Flask) production server on port 5001 with Gunicorn
- Frontend (React) on port 3000

### 5. Run All Services (MongoDB + Frontend only)

```bash
# Run without profiles (includes services without specific profiles)
docker-compose up -d

# This will start MongoDB and Frontend
# Backend services are only available via profiles
```

### 6. Run Specific Combinations

```bash
# Development backend + MongoDB + Frontend
docker-compose --profile dev up -d

# Production backend + MongoDB + Frontend
docker-compose --profile prod up -d

# All backends + MongoDB + Frontend
docker-compose --profile dev --profile prod up -d

# Just MongoDB and Frontend
docker-compose up -d
```

## Available Services

### MongoDB
- **Port:** 27017 (configurable via `MONGO_PORT`)
- **Default User:** admin
- **Default Password:** admin123 (change in .env)
- **Default Database:** multienv
- **Health Check:** Enabled, checks every 10 seconds

### Backend Development
- **Port:** 5000 (configurable via `BACKEND_DEV_PORT`)
- **Features:**
  - Auto-reload on file changes
  - Flask debug toolbar enabled
  - Direct file mounting for development
- **Profile:** `dev`
- **Command:** `flask run --host=0.0.0.0`

### Backend Production
- **Port:** 5001 (configurable via `BACKEND_PROD_PORT`)
- **Features:**
  - Multi-worker Gunicorn server (4 workers)
  - Multi-stage build for smaller image size
  - Non-root user for security
  - Restart policy: unless-stopped
- **Profile:** `prod`
- **Command:** Gunicorn with 4 sync workers

### Frontend
- **Port:** 3000 (configurable via `FRONTEND_PORT`)
- **Features:**
  - Multi-stage build for optimized production image
  - Served via `serve` package
  - Non-root user for security
- **API URL:** Configurable via `REACT_APP_API_URL`

## Environment Variables

### MongoDB
- `MONGO_ROOT_USER` - MongoDB root username (default: admin)
- `MONGO_ROOT_PASSWORD` - MongoDB root password (default: admin123)
- `MONGO_DB_NAME` - Database name (default: multienv)
- `MONGO_PORT` - MongoDB port (default: 27017)

### Backend
- `FLASK_ENV` - Flask environment (development/production)
- `FLASK_DEBUG` - Enable Flask debug mode (0/1)
- `FLASK_APP` - Flask app entry point (default: app.py)
- `BACKEND_DEV_PORT` - Dev backend port (default: 5000)
- `BACKEND_PROD_PORT` - Prod backend port (default: 5001)
- `MONGO_URI` - MongoDB connection URI (auto-generated from MONGO_* vars)

### Frontend
- `FRONTEND_PORT` - Frontend port (default: 3000)
- `NODE_ENV` - Node environment (development/production)
- `REACT_APP_API_URL` - Backend API URL for React app

## Useful Commands

### View Logs

```bash
# View logs from all services
docker-compose logs -f

# View logs from specific service
docker-compose logs -f backend-dev
docker-compose logs -f frontend
docker-compose logs -f mongodb

# View last 100 lines
docker-compose logs --tail=100
```

### Stop Services

```bash
# Stop all services (containers remain)
docker-compose stop

# Stop and remove containers
docker-compose down

# Remove everything including volumes
docker-compose down -v
```

### Rebuild Images

```bash
# Rebuild all images
docker-compose build

# Rebuild specific service
docker-compose build backend-dev

# Rebuild without cache
docker-compose build --no-cache
```

### Execute Commands in Container

```bash
# Access backend shell
docker-compose exec backend-dev bash

# Access MongoDB
docker-compose exec mongodb mongosh -u admin -p admin123

# Run Python commands
docker-compose exec backend-dev python -c "import sys; print(sys.version)"
```

### Health Status

```bash
# Check service health
docker-compose ps

# Detailed health check
docker inspect multienv_backend_dev | grep -A 5 "Health"
```

## Network Configuration

All services are connected via a Docker bridge network named `multienv-network`. This allows:
- Services to communicate by service name (e.g., `mongodb:27017`)
- MongoDB URI: `mongodb://admin:admin123@mongodb:27017/multienv?authSource=admin`

## Volume Management

### MongoDB Volumes
- `mongodb_data` - Persistent database data
- `mongodb_config` - MongoDB configuration

### Backend Dev Volume
- Mounts entire backend code directory for live editing

## Security Considerations

1. **Change Default Credentials:** Always change `MONGO_ROOT_PASSWORD` in production
2. **Non-root Users:** Backend prod and frontend run as non-root (uid 1000/1001)
3. **Health Checks:** All services include health checks
4. **Restart Policies:** Production services restart automatically unless stopped

## Production Deployment

For production deployment:

1. Use `.env.prod` with secure credentials
2. Update `REACT_APP_API_URL` to your actual API domain
3. Consider using environment-specific MongoDB:
   - Use managed MongoDB service (Atlas)
   - Or secure MongoDB with authentication and firewall rules
4. Use reverse proxy (Nginx) in front of services
5. Implement SSL/TLS certificates
6. Scale backend workers based on load
7. Use container orchestration platform (Kubernetes, Docker Swarm, etc.)

## Troubleshooting

### MongoDB Connection Issues
```bash
# Check MongoDB is running
docker-compose ps

# Test connection
docker-compose exec mongodb mongosh -u admin -p admin123

# Check logs
docker-compose logs mongodb
```

### Backend Connection Issues
```bash
# Check backend logs for errors
docker-compose logs backend-dev

# Verify environment variables
docker-compose exec backend-dev env | grep MONGO
```

### Frontend API Connection Issues
```bash
# Check if backend is accessible from frontend container
docker-compose exec frontend wget -O- http://backend-dev:5000/

# Verify API URL configuration
docker-compose exec frontend env | grep REACT_APP_API_URL
```

### Port Already in Use
```bash
# Change port in .env file
BACKEND_DEV_PORT=5001

# Or kill the process using the port
# Windows
netstat -ano | findstr :5000
taskkill /PID <PID> /F

# Linux/Mac
lsof -i :5000
kill -9 <PID>
```

## Development Workflow

### Development Setup
```bash
cp .env.dev .env
docker-compose --profile dev up --build
```

### Making Changes
1. Edit code in your IDE
2. Changes in backend/dev are automatically reflected (volume mount)
3. Frontend requires build step but React dev tools can hot-reload
4. MongoDB data persists in volume

### Testing
```bash
# Run tests in backend
docker-compose exec backend-dev pytest

# Run tests in frontend
docker-compose exec frontend npm test
```

## Additional Resources

- [Docker Official Documentation](https://docs.docker.com/)
- [Docker Compose Reference](https://docs.docker.com/compose/compose-file/)
- [Flask Documentation](https://flask.palletsprojects.com/)
- [React Documentation](https://react.dev/)
- [MongoDB Documentation](https://docs.mongodb.com/)
