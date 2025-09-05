# Quiz-temporal

A containerized Temporal workflow orchestration setup with PostgreSQL database and web UI for building distributed applications.

## 📋 Overview

This project provides a complete Temporal development environment using Docker Compose. Temporal is a durable execution platform that makes it easy to build and operate resilient applications at scale.

## 🏗️ Architecture

The setup consists of three main components:

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   PostgreSQL    │    │  Temporal       │    │  Temporal UI    │
│   Database      │◄───┤  Server         │◄───┤  Web Interface  │
│   Port: 5433    │    │  Port: 7234     │    │  Port: 8235     │
└─────────────────┘    └─────────────────┘    └─────────────────┘
```

### Components

1. **PostgreSQL Database** (`postgres:13`)
   - Persistent storage for Temporal server state
   - Container: `temporal-postgres-quiz`
   - External Port: `5433` → Internal Port: `5432`
   - Database: `temporal`
   - Credentials: `temporal/temporal`

2. **Temporal Server** (`temporalio/auto-setup:1.23`)
   - Core Temporal services (Frontend, History, Matching, Worker)
   - Container: `temporal-quiz`
   - API Port: `7234` (Frontend API)
   - Automatically sets up database schema on first run

3. **Temporal Web UI** (`temporalio/ui:2.21.3`)
   - Web interface for monitoring workflows and activities
   - Container: `temporal-ui-quiz`
   - External Port: `8235` → Internal Port: `8080`
   - Access URL: http://localhost:8235

## 🚀 Quick Start

### Prerequisites
- Docker and Docker Compose installed
- Ports 5433, 7234, and 8235 available on your system

### Starting the Services

```bash
# Clone or navigate to the project directory
cd Quiz-temporal

# Start all services in detached mode
docker-compose up -d

# Check service status
docker-compose ps

# View logs (optional)
docker-compose logs -f
```

### Accessing the Services

- **Temporal Web UI**: http://localhost:8235
- **Temporal Server API**: `localhost:7234`
- **PostgreSQL Database**: `localhost:5433`

### Stopping the Services

```bash
# Stop all services
docker-compose down

# Stop and remove volumes (deletes all data)
docker-compose down -v
```

## 📁 Project Structure

```
Quiz-temporal/
├── docker-compose.yml    # Container orchestration configuration
└── README.md            # This documentation file
```

## ⚙️ Configuration Details

### Port Mappings

All ports have been configured to avoid conflicts with default installations:

| Service | External Port | Internal Port | Purpose |
|---------|---------------|---------------|---------|
| PostgreSQL | 5433 | 5432 | Database connections |
| Temporal API | 7234 | 7233 | Temporal client connections |
| Temporal UI | 8235 | 8080 | Web interface |

### Environment Variables

**PostgreSQL:**
```yaml
POSTGRES_USER: temporal
POSTGRES_PASSWORD: temporal
POSTGRES_DB: temporal
```

**Temporal Server:**
```yaml
DB: postgres12
DB_PORT: 5432
POSTGRES_USER: temporal
POSTGRES_PWD: temporal
POSTGRES_SEEDS: postgres
```

**Temporal UI:**
```yaml
TEMPORAL_ADDRESS: temporal:7233
TEMPORAL_CORS_ORIGINS: http://localhost:8235
```

## 🔧 Development Setup

### Connecting Your Application

To connect your application to this Temporal server:

```javascript
// Example Node.js connection
const { Client } = require('@temporalio/client');

const client = new Client({
  serviceUrl: 'localhost:7234',
  namespace: 'default', // or your custom namespace
});
```

```python
# Example Python connection
from temporalio.client import Client

async def main():
    client = await Client.connect("localhost:7234")
```

### Creating Workflows

Once connected, you can:
1. Define workflows and activities in your preferred language
2. Register workers that connect to `localhost:7234`
3. Start workflow executions
4. Monitor them via the web UI at http://localhost:8235

## 🚨 Important Things to Avoid

### ❌ Common Pitfalls

1. **Port Conflicts**
   - Don't change ports back to defaults (7233, 8233, 5432) if you have other services running
   - Always check `docker-compose ps` before assuming services are healthy

2. **Missing Dependencies**
   - Don't start `temporal-ui` before `temporal` server is ready
   - Don't start `temporal` before `postgres` is ready
   - The `depends_on` configuration handles this, but manual starts can cause issues

3. **Database Issues**
   - Don't delete the PostgreSQL container without backing up data
   - Don't change database credentials after initial setup without updating all services
   - Don't manually modify the Temporal database schema

4. **Version Compatibility**
   - Don't mix incompatible versions of Temporal server and UI
   - Don't upgrade major versions without checking migration guides
   - Current versions: Server `1.23`, UI `2.21.3`

### ❌ Configuration Mistakes

1. **Environment Variables**
   ```yaml
   # ❌ Wrong - Inconsistent database names
   POSTGRES_DB: temporal
   POSTGRES_SEEDS: different_postgres
   
   # ✅ Correct - Consistent naming
   POSTGRES_DB: temporal
   POSTGRES_SEEDS: postgres
   ```

2. **Network Issues**
   ```yaml
   # ❌ Wrong - External port in internal config
   TEMPORAL_ADDRESS: temporal:8235
   
   # ✅ Correct - Internal port for container communication
   TEMPORAL_ADDRESS: temporal:7233
   ```

3. **Port Mappings**
   ```yaml
   # ❌ Wrong - Conflicting external ports
   ports:
     - "7233:7233"  # Might conflict with existing Temporal
   
   # ✅ Correct - Non-conflicting external ports
   ports:
     - "7234:7233"  # External 7234 → Internal 7233
   ```

### ❌ Operations to Avoid

1. **Data Loss Prevention**
   - Never run `docker-compose down -v` in production
   - Don't delete volumes without proper backup
   - Don't restart PostgreSQL container during active workflows

2. **Security Issues**
   - Don't expose database ports to the internet without proper security
   - Don't use default credentials in production
   - Don't disable authentication in production environments

3. **Performance Issues**
   - Don't run with default resource limits in production
   - Don't ignore container health checks
   - Don't start workflows without proper monitoring

## 🔍 Troubleshooting

### Services Not Starting

```bash
# Check container status
docker-compose ps

# View detailed logs
docker-compose logs temporal
docker-compose logs postgres
docker-compose logs temporal-ui

# Restart specific service
docker-compose restart temporal
```

### UI Not Accessible

1. Verify all containers are running: `docker-compose ps`
2. Check if port 8235 is available: `netstat -an | grep 8235`
3. Wait for Temporal server to be fully initialized (30-60 seconds)
4. Check UI logs: `docker-compose logs temporal-ui`

### Database Connection Issues

1. Ensure PostgreSQL is ready: `docker-compose logs postgres`
2. Verify environment variables match between services
3. Check if port 5433 is available for external connections

### Port Conflicts

```bash
# Check what's using your ports
lsof -i :5433
lsof -i :7234
lsof -i :8235

# Change ports in docker-compose.yml if conflicts exist
```

## 📚 Next Steps

1. **Learn Temporal Concepts**: Read about workflows, activities, and workers
2. **Choose Your SDK**: Install Temporal SDK for your preferred language
3. **Build Your First Workflow**: Start with simple examples
4. **Monitor with Web UI**: Use http://localhost:8235 to observe execution
5. **Scale Your Setup**: Consider clustering for production use

## 🔗 Useful Links

- [Temporal Documentation](https://docs.temporal.io/)
- [Temporal SDKs](https://docs.temporal.io/dev-guide)
- [Docker Compose Documentation](https://docs.docker.com/compose/)
- [PostgreSQL Documentation](https://www.postgresql.org/docs/)

## 📄 License

This setup configuration is provided as-is for development purposes.

---

**Note**: This setup is optimized for development. For production deployments, consider additional security, monitoring, and scaling configurations.