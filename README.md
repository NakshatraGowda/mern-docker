# MERN 3-Tier Architecture with Docker Compose

A production-ready, fully containerized MERN application demonstrating DevOps best practices through Docker containerization and Docker Compose orchestration. This project serves as a reference implementation for multi-tier microservices architecture using container technology.

## 🎯 Project Overview

This project is a DevOps-focused implementation of a containerized MERN stack featuring:
- **Container Orchestration**: Docker Compose for multi-container application management
- **Microservices Architecture**: Separated frontend, backend, and database services
- **Service Isolation**: Each component runs in its own container with defined interfaces
- **Network Isolation**: Custom bridge network for secure inter-service communication
- **Persistent Storage**: Named volumes for database durability
- **Health Checks**: Automated service health verification and restart policies
- **Environment Configuration**: Service-to-service communication via environment variables
- **Service Dependencies**: Dependency management and startup ordering

All components run as isolated, independently deployable containers with automated networking, service discovery, and persistent data management.

## 🏗️ Containerized Architecture

```
┌──────────────────────────────────────────────────────────────┐
│                   Host Machine (Docker Engine)               │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  ┌────────────────── Docker Network (mern) ──────────────┐   │
│  │ Bridge Driver - Isolated Container Communication      │   │
│  │                                                       │   │
│  │  ┌──────────────────────────────────────────────────┐ │   │
│  │  │ Frontend Container                               │ │   │
│  │  │ ├─ Image: mern-docker-frontend                   │ │   │
│  │  │ ├─ Port: 5173:5173 (Host:Container)              │ │   │
│  │  │ ├─ Network Alias: frontend                       │ │   │
│  │  │ └─ Volume Mounts: src/, public/                  │ │   │
│  │  └──────────────────────────────────────────────────┘ │   │
│  │                                                       │   │
│  │  ┌──────────────────────────────────────────────────┐ │   │
│  │  │ Backend Container                                │ │   │
│  │  │ ├─ Image: mern-docker-backend                    │ │   │
│  │  │ ├─ Port: 5050:5050 (Host:Container)              │ │   │
│  │  │ ├─ Network Alias: backend                        │ │   │
│  │  │ ├─ Env: MONGO_URI=mongodb://mongodb:27017/...    │ │   │
│  │  │ ├─ Restart Policy: on-failure                    │ │   │
│  │  │ └─ Depends On: mongodb (health check)            │ │   │
│  │  └──────────────────────────────────────────────────┘ │   │
│  │                                                       │   │
│  │  ┌──────────────────────────────────────────────────┐ │   │
│  │  │ MongoDB Container                                │ │   │
│  │  │ ├─ Image: mongo:latest                           │ │   │
│  │  │ ├─ Port: 27017:27017 (Host:Container)            │ │   │
│  │  │ ├─ Network Alias: mongodb                        │ │   │
│  │  │ ├─ Volume: mongo-data:/data/db (Named)           │ │   │
│  │  │ ├─ Health Check: CMD mongo --eval db.ping()      │ │   │
│  │  │ └─ Restart Policy: always                        │ │   │
│  │  └──────────────────────────────────────────────────┘ │   │
│  │                                                       │   │
│  └───────────────────────────────────────────────────────┘   │
│                                                              │
│  ┌──────────────────── Named Volumes ────────────────────┐   │
│  │ mongo-data: ├─ /data/db/                              │   │
│  │             ├─ Persistent storage for MongoDB         │   │
│  │             └─ Survives container restarts            │   │
│  └────────────────────────────────────────────────────── ┘   │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### Containerization Strategy

**Container Isolation**: Each service (frontend, backend, database) runs in its own container with:
- Isolated filesystem
- Isolated network namespace
- Resource constraints (CPU, memory)
- Independent lifecycle management

**Service Communication**: Containers communicate via Docker's embedded DNS:
- Service names resolve to container IP addresses within the network
- Internal communication on private bridge network
- Only exposed ports accessible from host machine

**Data Persistence**: Named volumes ensure data survives container restarts:
- `mongo-data` volume mounted at `/data/db` in MongoDB container
- Data persists even when containers are stopped/removed
- Volumes managed by Docker daemon

## Docker Images & Containerization

### Image Building

Custom images are built using Dockerfiles in each service directory:

**Backend Dockerfile** (`mern/backend/Dockerfile`):
- Base image: Node.js runtime
- Copies application code into container filesystem
- Installs dependencies from `package.json`
- Exposes port 5050
- Entrypoint: `node server.js`

**Frontend Dockerfile** (`mern/frontend/Dockerfile`):
- Base image: Node.js for build, then lightweight runtime
- Installs dependencies
- Runs Vite dev server with `--host 0.0.0.0`
- Exposes port 5173

### Image Layers & Caching

Docker images consist of layered filesystems:
- Each instruction in Dockerfile creates a new layer
- Layers are cached for faster rebuilds
- Order instructions from least to most frequently changing for optimization

### Image Registry

**MongoDB Image**:
- Pulled from Docker Hub: `mongo:latest` by default
- Official image maintained by MongoDB
- Pre-built with all MongoDB binaries and dependencies

**Version Pinning with Variables**:
- Docker Compose supports environment variable substitution in image names
- Example: `image: mongo:${BUILD_NUMBER:-latest}`
- This uses `BUILD_NUMBER` if defined, otherwise falls back to `latest`
- Use this pattern in CI/CD to deploy specific image tags

**Important**:
- The substituted tag must exist on Docker Hub
- `mongo:7.0`, `mongo:6.0` are valid official tags; arbitrary build numbers are not unless you publish them

### Building Images

```bash
# Build all images (Docker Compose automatic)
docker-compose up --build

# Build specific service image
docker-compose build backend

# Build without cache (force fresh build)
docker-compose build --no-cache backend

# View built images
docker images | grep mern-docker
```

## �📋 Prerequisites

Before you begin, ensure you have the following installed:
- **Docker** (v20.10+) - [Install Docker](https://docs.docker.com/get-docker/)
- **Docker Compose** (v2.0+) - [Install Docker Compose](https://docs.docker.com/compose/install/)
- **Git** - for version control

> **Note**: Docker Desktop includes both Docker and Docker Compose on macOS and Windows.

## 🚀 Getting Started

### 1. Clone or Download the Project

```bash
# Navigate to the project directory
cd mern-docker
```

### 2. Environment Configuration

The project uses default environment variables configured in `docker-compose.yaml`:
- **Backend**: `MONGO_URI=mongodb://mongodb:27017/employees`
- **Frontend**: `REACT_APP_API_URL=http://backend:5050`

These are automatically set for containerized communication and require no manual configuration.

### 3. Build and Run the Application

```bash
# Start all services (builds images if needed)
docker-compose up --build

# Or run in detached mode (background)
docker-compose up -d --build
```

The application will be accessible at:
- **Frontend**: http://localhost:5173
- **Backend API**: http://localhost:5050
- **MongoDB**: mongodb://localhost:27017

### 4. Stop the Application

```bash
# Stop all running containers
docker-compose down

# Stop and remove volumes (database data will be deleted)
docker-compose down -v
```

## 📁 Project Structure

```
mern-docker/
├── docker-compose.yaml          # Docker Compose configuration
├── README.md                     # This file
│
└── mern/
    ├── backend/
    │   ├── Dockerfile           # Backend container image
    │   ├── package.json         # Node.js dependencies
    │   ├── server.js            # Express server entry point
    │   ├── db/
    │   │   └── connection.js    # MongoDB connection logic
    │   └── routes/
    │       └── record.js        # Record CRUD endpoints
    │
    └── frontend/
        ├── Dockerfile           # Frontend container image
        ├── package.json         # React dependencies
        ├── vite.config.js       # Vite build configuration
        ├── tailwind.config.js   # Tailwind CSS configuration
        ├── postcss.config.js    # PostCSS configuration
        ├── cypress.json         # E2E testing configuration
        ├── index.html           # HTML entry point
        ├── cypress/             # Cypress end-to-end tests
        │   ├── integration/
        │   │   └── endToEnd.spec.js
        │   ├── support/
        │   └── fixtures/
        ├── public/              # Static assets
        └── src/
            ├── main.jsx         # React entry point
            ├── App.jsx          # Root component
            ├── index.css        # Global styles
            ├── components/
            │   ├── Navbar.jsx   # Navigation component
            │   ├── Record.jsx   # Single record component
            │   └── RecordList.jsx # Records list component
            └── assets/          # Images and media
```

##  Docker Compose Configuration & Container Orchestration

Docker Compose is used to define and run this multi-container application as a single declarative unit. The `docker-compose.yaml` file contains the complete infrastructure-as-code definition.

### Service Definitions & Container Configuration

#### Backend Service (Express.js)
```yaml
backend: 
  build: ./mern/backend              # Build context for custom image
  ports:
    - "5050:5050"                    # Port mapping: Host:Container
  networks:
    - mern                           # Connect to mern bridge network
  environment:
    - MONGO_URI=mongodb://mongodb:27017/employees
                                     # Service discovery: uses container name
  depends_on:
    - mongodb                        # Startup ordering dependency
  restart: on-failure                # Restart policy for fault tolerance
```

**Containerization Details**:
- **Image Build**: Custom Dockerfile builds application image with all dependencies
- **Port Exposure**: Port 5050 exposed for external access; container listens on 5050
- **Network Integration**: Connected to custom `mern` bridge network for inter-service communication
- **Environment Variables**: `MONGO_URI` uses internal DNS resolution (`mongodb` resolves to MongoDB container IP)
- **Restart Policy**: `on-failure` ensures container restarts if application crashes
- **Service Discovery**: Backend finds MongoDB via DNS name `mongodb` (not IP address)

#### Frontend Service (React + Vite)
```yaml
frontend:
  build: ./mern/frontend             # Custom image with React build tools
  ports:
    - "5173:5173"                    # Vite dev server port
  networks:
    - mern                           # Same bridge network
  environment:
    REACT_APP_API_URL: http://backend:5050
                                     # Backend accessible via container DNS
```

**Containerization Details**:
- **Build Process**: Dockerfile sets up Node.js environment with Vite dev server
- **Hot Reload**: Source code can be mounted as volume for development
- **Network Access**: Can reach backend via `http://backend:5050` (DNS resolution)
- **Host Binding**: Vite runs with `--host 0.0.0.0` to accept connections outside container

#### MongoDB Service (Database)
```yaml
mongodb:
  image: mongo:latest                # Official MongoDB image from Docker Hub
  ports:
    - "27017:27017"                  # Standard MongoDB port
  networks:
    - mern
  volumes:
    - mongo-data:/data/db            # Named volume for data persistence
  healthcheck:                       # Container health verification
    test: ["CMD", "mongo", "--eval", "db.adminCommand('ping')"]
    interval: 5s                     # Check every 5 seconds
    timeout: 5s                      # Wait 5s for response
    retries: 5                       # Fail after 5 failed checks
  restart: always                    # Always restart, even on success
```

**Containerization Details**:
- **Image Source**: Pre-built official image from Docker Hub (no custom Dockerfile needed)
- **Volume Mounting**: Named volume `mongo-data` ensures database persistence
- **Health Checks**: Docker monitors MongoDB availability; other services wait for healthy state
- **Restart Strategy**: `always` ensures MongoDB is always running for application availability
- **Port Mapping**: Internal 27017 accessible externally, but also via DNS within network

### Docker Network Configuration

```yaml
networks:
  mern:
    driver: bridge                   # Bridge driver for container networking
```

**Network Details**:
- **Bridge Driver**: Creates isolated network namespace with embedded DNS
- **Container Naming**: Each service gets DNS alias matching service name
- **DNS Resolution**: Containers resolve service names to internal IPs automatically
- **Network Isolation**: Only containers on `mern` network can communicate; isolated from host
- **DNS Server**: Docker embedded DNS (127.0.0.11:53) handles service discovery

### Volume Management

```yaml
volumes:
  mongo-data:
    driver: local                    # Local storage driver
```

**Volume Details**:
- **Named Volume**: Managed by Docker; persists data independently of container lifecycle
- **Local Driver**: Stores data on host machine's Docker volume storage
- **Persistence**: Data survives `docker-compose down` (unless removed with `-v` flag)
- **Mount Path**: Mounted to `/data/db` inside MongoDB container
- **Data Location**: Typically stored at `/var/lib/docker/volumes/mongo-data/_data`

### Container Lifecycle Management

**Dependency Management**:
```yaml
depends_on:
  - mongodb                          # Backend waits for MongoDB service
```
- Ensures MongoDB container starts before backend
- Combined with health checks for robust startup ordering

**Restart Policies**:
- `backend: on-failure` - Restarts only if container exits with error
- `mongodb: always` - Always restarts, maintaining availability

**Service Scaling Considerations**:
- Each service defined once (scaling handled by orchestration layer)
- For production, consider Kubernetes for multi-replica deployments

## 🔧 Container Management & Orchestration

### Docker Compose Commands

**Starting the Multi-Container Application**:
```bash
# Start all services in foreground (see logs in real-time)
docker-compose up

# Start services in background (detached mode)
docker-compose up -d

# Build images first, then start services
docker-compose up --build

# Start specific service
docker-compose up backend
```

**Stopping and Cleanup**:
```bash
# Stop running containers (preserves data and volumes)
docker-compose stop

# Stop and remove containers (preserves volumes)
docker-compose down

# Stop and remove containers + volumes (complete cleanup)
docker-compose down -v

# Remove all unused images and dangling volumes
docker-compose down --remove-orphans
```

**Container Status and Management**:
```bash
# List running containers
docker-compose ps

# List all containers (including stopped)
docker-compose ps -a

# Show container resource usage (CPU, memory)
docker stats

# Pause all containers
docker-compose pause

# Resume paused containers
docker-compose unpause

# Restart specific service
docker-compose restart backend

# Kill running container immediately
docker-compose kill backend
```

**Logs and Debugging**:
```bash
# View logs from all services (follow mode)
docker-compose logs -f

# View logs from specific service
docker-compose logs -f backend
docker-compose logs -f frontend
docker-compose logs -f mongodb

# Show last 50 lines
docker-compose logs --tail=50

# Show logs with timestamps
docker-compose logs -f --timestamps

# Stream logs in real-time for debugging
docker-compose logs -f frontend backend
```

**Container Inspection and Access**:
```bash
# Execute command inside running container
docker-compose exec backend sh

# Access MongoDB shell
docker-compose exec mongodb mongo

# Run one-off command
docker-compose exec backend node -v

# Interactive shell with specific user
docker-compose exec -u root backend sh

# Check container resource limits
docker inspect $(docker-compose ps -q backend)
```

**Image Management**:
```bash
# Build all images
docker-compose build

# Build specific service image
docker-compose build backend

# Build without cache (clean rebuild)
docker-compose build --no-cache backend

# Build with specific Dockerfile
docker-compose build --file Dockerfile.prod backend

# List built images
docker images | grep mern

# Remove unused images
docker image prune

# Remove specific image
docker rmi mern-docker-backend:latest
```

**Network and Volume Management**:
```bash
# List Docker networks
docker network ls

# Inspect specific network
docker network inspect mern-docker_mern

# List volumes
docker volume ls

# Inspect volume
docker volume inspect mern-docker_mongo-data

# View volume mount point
docker volume inspect -f '{{ .Mountpoint }}' mern-docker_mongo-data

# Remove unused volumes
docker volume prune
```

### Container Development Workflow

**Workflow for Backend Development**:
```bash
# 1. Start all services
docker-compose up -d

# 2. Make changes to backend code
# (Edit files in mern/backend/)

# 3. Rebuild backend image
docker-compose build backend

# 4. Restart backend service with new image
docker-compose up -d backend

# 5. View logs to verify
docker-compose logs -f backend
```

**Workflow for Frontend Development**:
```bash
# 1. Start all services
docker-compose up -d

# 2. Make changes (hot reload enabled by Vite)
# Changes automatically reflected in container

# 3. Check frontend logs if needed
docker-compose logs -f frontend

# 4. Rebuild only if dependencies changed
docker-compose build frontend
docker-compose up -d frontend
```

**Debugging Container Issues**:
```bash
# Check service health status
docker-compose ps

# View detailed service configuration
docker-compose config

# Validate docker-compose.yaml syntax
docker-compose config -q

# Get container IP address
docker-compose exec backend hostname -i

# Test inter-container connectivity
docker-compose exec frontend ping backend

# Check DNS resolution
docker-compose exec backend nslookup mongodb

# Monitor real-time resource usage
docker stats --no-stream
```

### Data Persistence and Volume Management

**Managing MongoDB Data**:
```bash
# Access MongoDB container shell
docker-compose exec mongodb mongo

# Backup MongoDB data
docker run --rm -v mern-docker_mongo-data:/data -v $(pwd):/backup mongo \
  mongodump --out /backup/mongo-dump

# Restore MongoDB data
docker run --rm -v mern-docker_mongo-data:/data -v $(pwd):/backup mongo \
  mongorestore /backup/mongo-dump

# View volume contents (if needed)
docker volume inspect -f '{{ .Mountpoint }}' mern-docker_mongo-data

# Clear all data (destructive)
docker volume rm mern-docker_mongo-data
```

**Production Considerations**:
- Use named volumes for persistence (already configured)
- Implement backup strategies for critical data
- Consider external storage solutions for scaling
- Use environment-specific docker-compose files (docker-compose.prod.yml)

### Health Checks and Service Readiness

The MongoDB service includes automated health checks:

```yaml
healthcheck:
  test: ["CMD", "mongo", "--eval", "db.adminCommand('ping')"]
  interval: 5s        # Check every 5 seconds
  timeout: 5s         # Wait 5 seconds for response
  retries: 5          # Fail after 5 consecutive failures
  start_period: 10s   # Grace period before first check
```

**Monitoring Health Status**:
```bash
# Check service health in docker-compose ps output
docker-compose ps

# Inspect health status in detail
docker inspect --format='{{json .State.Health}}' \
  $(docker-compose ps -q mongodb) | jq .

# Monitor health status in real-time
watch "docker-compose ps"
```

## 🚀 Container Scaling & Production Deployment

### Horizontal Scaling Considerations

Docker Compose on single host supports basic replication:
```bash
# Scale specific service (creates multiple instances)
docker-compose up -d --scale backend=3

# Note: This requires load balancing configuration
```

For production multi-host deployments, consider:
- **Docker Swarm**: Native Docker clustering
- **Kubernetes**: Industry-standard orchestration platform
- **Cloud Platforms**: AWS ECS, Azure Container Instances, GCP Cloud Run

### Production-Ready Configurations

**Recommended Enhancements**:
- Load balancer (nginx/HAProxy) for frontend/backend
- Separate docker-compose files for dev/staging/prod
- Environment-specific configurations
- Secrets management (Docker secrets/external vault)
- Resource limits per container
- Logging aggregation (ELK stack, CloudWatch)
- Monitoring and alerting (Prometheus, Grafana)

### Environment-Specific Setup

Create `docker-compose.prod.yml`:
```yaml
version: '3.8'
services:
  backend:
    image: myregistry/mern-backend:v1.0
    restart: always
    deploy:
      replicas: 3
      resources:
        limits:
          cpus: '0.5'
          memory: 512M
  # ... other services
```

Deploy production stack:
```bash
docker-compose -f docker-compose.yml -f docker-compose.prod.yml up -d
```

## 🔒 Container Security Best Practices

### Image Security
- Use specific image tags (avoid `latest`)
- Scan images for vulnerabilities: `docker scan image-name`
- Use minimal base images (Alpine Linux)
- Remove unnecessary packages from Dockerfiles

### Runtime Security
```bash
# Run containers as non-root user
# Add to Dockerfile: USER node (not root)

# Run container in read-only mode
docker run --read-only --tmpfs /tmp image-name

# Drop unnecessary Linux capabilities
docker run --cap-drop=ALL --cap-add=NET_BIND_SERVICE image-name

# Limit container resources
docker run --memory="512m" --cpus="1" image-name
```

### Network Security
- Use named networks (already implemented)
- Expose only necessary ports
- Use environment variables for sensitive data
- Implement secrets management for production

## 🐛 Troubleshooting

### Container Won't Start
```bash
# Check container logs
docker-compose logs backend

# Check exit code and error details
docker-compose ps  # Shows exit code

# Rebuild and restart
docker-compose down
docker-compose build --no-cache backend
docker-compose up backend
```

### Port Already in Use
```bash
# Find process using port
lsof -i :5173

# Kill process
kill -9 <PID>

# Or use docker-compose with different port
docker-compose -f docker-compose.yml -p myapp up
```

### MongoDB Connection Issues
```bash
# Verify MongoDB is ready
docker-compose logs mongodb

# Check MongoDB health status
docker-compose exec mongodb mongo --eval "db.adminCommand('ping')"

# Force restart
docker-compose restart mongodb

# Check network connectivity
docker-compose exec backend ping mongodb
```

### Service Can't Reach Other Service
```bash
# Verify network configuration
docker network inspect mern-docker_mern

# Check DNS resolution inside container
docker-compose exec backend nslookup backend

# Verify environment variables
docker-compose exec backend env | grep MONGO_URI

# Test connectivity between containers
docker-compose exec backend wget -O - http://backend:5050
```

### Out of Disk Space
```bash
# Clean up unused images, volumes, networks
docker system prune -a --volumes

# Remove specific volume
docker volume rm volume-name

# Monitor disk usage
du -sh /var/lib/docker/volumes/
```

### Memory/CPU Issues
```bash
# Check resource usage
docker stats

# Set resource limits in docker-compose.yml
services:
  backend:
    deploy:
      resources:
        limits:
          cpus: '0.5'
          memory: 512M
```

## 📚 Application Details

### 🔌 API Endpoints

The backend provides the following REST API endpoints for record management:

**Get All Records**:
```
GET /record
Response: Array of record objects
```

**Get Single Record**:
```
GET /record/:id
Response: Single record object
Status: 404 if not found
```

**Create New Record**:
```
POST /record
Body: {
  "name": "string",
  "position": "string",
  "level": "string"
}
Response: Created record with MongoDB _id
Status: 204 No Content
```

**Update Record**:
```
PATCH /record/:id
Body: Partial record fields to update
Response: Update confirmation
```

**Delete Record**:
```
DELETE /record/:id
Response: Deletion confirmation
```

### 🎨 Frontend Architecture

The frontend is built with React and Vite, providing a fast development experience:

- **React 18**: Modern component-based UI framework
- **Vite**: Next-generation frontend build tool for hot module replacement
- **Tailwind CSS**: Utility-first CSS framework for responsive design
- **React Router**: Client-side routing for multi-page navigation
- **Responsive Design**: Mobile-friendly interface

**Components**:
- **Navbar.jsx**: Application header and navigation
- **RecordList.jsx**: Display all records in a table/list format
- **Record.jsx**: Individual record display and management
- **App.jsx**: Root component and main application logic

### 🧪 Testing

The project includes Cypress for end-to-end testing:

```bash
# Run Cypress tests (open the Cypress test runner)
docker-compose exec frontend npm run cypress:open

# Or run tests headlessly
docker-compose exec frontend npm run cypress:run
```

Test files are located in `mern/frontend/cypress/integration/endToEnd.spec.js`

### 📦 Application Stack

**Backend**:
- **Node.js**: JavaScript runtime
- **Express.js**: Web application framework
- **MongoDB**: NoSQL database
- **CORS**: Cross-Origin Resource Sharing middleware

**Frontend**:
- **React**: UI library
- **Vite**: Build tool and dev server
- **Tailwind CSS**: CSS framework
- **React Router**: Routing library
- **Cypress**: End-to-end testing framework
- **ESLint**: Code quality tool

## 🏗️ DevOps & Container Technologies

### Core Technologies

**Container Runtime & Orchestration**:
- **Docker Engine**: Container runtime that packages applications with all dependencies
- **Docker Compose**: Multi-container orchestration tool for defining services as code
- **Docker CLI**: Command-line interface for container management

**Container Networking**:
- **Bridge Network Driver**: Isolated network namespace for inter-container communication
- **Embedded DNS**: Service discovery via container names (127.0.0.11:53)
- **Port Mapping**: Maps container ports to host for external accessibility

**Storage & Persistence**:
- **Named Volumes**: Managed data storage persisting across container lifecycle
- **Local Volume Driver**: Docker-managed storage on host machine
- **Filesystem Isolation**: Each container has independent filesystem

**Service Management**:
- **Health Checks**: Automated container health verification and restart policies
- **Restart Policies**: Fault tolerance through automatic container restart
- **Dependency Management**: Service startup ordering via depends_on

### DevOps Practices Implemented

| Practice | Implementation |
|----------|-----------------|
| **Infrastructure as Code** | `docker-compose.yaml` defines entire stack |
| **Container Isolation** | Each service runs independently in own container |
| **Service Discovery** | DNS resolution for inter-container communication |
| **Data Persistence** | Named volumes for durable storage |
| **Health Monitoring** | Automated health checks with restart policies |
| **Scalability** | Multi-container architecture ready for scaling |
| **Environment Config** | Environment variables for service configuration |
| **Log Aggregation** | Centralized logging via `docker-compose logs` |
| **Resource Management** | CPU/memory limits can be configured per container |
| **Network Isolation** | Custom bridge network for secure communication |

### Tools & Commands Reference

**System Information**:
```bash
docker version                    # Docker Engine version
docker info                       # Docker system info
docker system df                  # Disk usage by images/containers/volumes
```

**Image Management**:
```bash
docker images                     # List local images
docker build -t name:tag .        # Build custom image
docker pull image:tag             # Download image from registry
docker push image:tag             # Upload image to registry
docker tag source target          # Create image tag
docker save image > file.tar       # Export image to tarball
docker load < file.tar            # Import image from tarball
```

**Container Operations**:
```bash
docker create                     # Create container (don't start)
docker start                      # Start stopped container
docker stop                       # Gracefully stop container
docker kill                       # Force stop container
docker rm                         # Remove container
docker run                        # Create and start container
docker exec                       # Execute command in running container
docker attach                     # Attach to container output
docker wait                       # Wait for container to exit
```

**Networking**:
```bash
docker network create name        # Create custom network
docker network ls                 # List networks
docker network inspect name       # Show network details
docker network connect            # Connect container to network
docker network disconnect         # Disconnect container from network
```

**Volumes**:
```bash
docker volume create name         # Create named volume
docker volume ls                  # List volumes
docker volume inspect name        # Show volume details
docker volume rm name             # Remove volume
docker volume prune               # Remove unused volumes
```

## 📚 DevOps Learning Resources

- [Docker Official Documentation](https://docs.docker.com/)
- [Docker Compose Reference](https://docs.docker.com/compose/compose-file/)
- [Best Practices for Building Container Images](https://docs.docker.com/develop/container-best-practices/)
- [Container Security](https://docs.docker.com/engine/security/)
- [Docker Networking Guide](https://docs.docker.com/network/)
- [Multi-Stage Builds for Optimization](https://docs.docker.com/build/building/multi-stage/)
- [Kubernetes for Orchestration at Scale](https://kubernetes.io/)

## 📝 License

ISC License

## 🤝 Contributing

Contributions are welcome! Please feel free to submit a Pull Request or open an Issue for any bugs or feature requests.

## 📞 Support

For issues or questions, please refer to the [Docker Documentation](https://docs.docker.com/) and [MERN Stack Resources](https://www.mongodb.com/developer/languages/javascript/mern-stack-tutorial/).
