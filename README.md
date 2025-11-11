# SonarQube Docker Compose Setup

A production-ready Docker Compose configuration for running SonarQube Community Edition with PostgreSQL database. This comprehensive, production-ready SonarQube setup provides enterprise-grade documentation, health checks, and resource optimization that transforms a basic Docker deployment into a deployment-ready solution for teams of any size.﻿

## Overview

This setup provides:
- **SonarQube Community Edition** - Code quality and security analysis
- **PostgreSQL 15 Alpine** - Lightweight database backend
- **Health checks** - Automated container health monitoring
- **Resource limits** - Optimized memory and CPU allocation
- **Volume persistence** - Data survives container restarts

## Prerequisites

- Docker Engine 20.10+
- Docker Compose 1.29+
- 4GB+ RAM available
- 10GB+ free disk space

## Quick Start

### 1. Clone or Download

```bash
git clone <repo url>
cd sonarqube-docker
```

### 2. Configure Environment

Copy the example environment file and customize it:

```bash
cp .env.example .env
```

Edit `.env` to change default values (optional):

```env
DB_USER=sonar
DB_PASSWORD=changeme  # Change this!
DB_NAME=sonar
SONAR_PORT=9000
```

### 3. Start the Services

```bash
docker-compose up -d
```

The containers will start and initialize automatically. Initial startup takes 2-3 minutes.

### 4. Verify Health Status

```bash
docker-compose ps
```

Both services should show `healthy` status:

```
NAME          STATUS
sonarqube     Up 2 minutes (healthy)
postgres      Up 2 minutes (healthy)
```

### 5. Access SonarQube

Open your browser and navigate to:

```
http://localhost:9000
```

**Default Login:**
- Username: `admin`
- Password: `admin`

⚠️ **Change the default password immediately after first login!**

## Usage

### View Logs

```bash
# All services
docker-compose logs -f

# Specific service
docker-compose logs -f sonarqube
docker-compose logs -f postgres
```

### Stop Services

```bash
docker-compose stop
```

### Stop and Remove Containers (Keep Data)

```bash
docker-compose down
```

### Restart Services

```bash
docker-compose restart
```

### Remove Everything (Deletes Data)

⚠️ **Warning: This removes all data!**

```bash
docker-compose down -v
```

## Configuration

### Environment Variables

Edit `.env` to customize:

| Variable | Default | Description |
|----------|---------|-------------|
| `DB_USER` | sonar | PostgreSQL username |
| `DB_PASSWORD` | changeme | PostgreSQL password |
| `DB_NAME` | sonar | PostgreSQL database name |
| `SONAR_PORT` | 9000 | SonarQube web port |

### Memory and CPU

Adjust resource limits in `docker-compose.yml`:

```yaml
services:
  sonarqube:
    mem_limit: 3g      # Adjust based on your system
    cpus: '2'          # Number of CPU cores
```

**Recommended for different systems:**

| System | mem_limit | cpus |
|--------|-----------|------|
| 4GB RAM | 1.5g | 1 |
| 8GB RAM | 2g | 2 |
| 16GB+ RAM | 3g+ | 4+ |

### Changing the Port

Edit `.env`:

```env
SONAR_PORT=8080  # Access at http://localhost:8080
```

## Troubleshooting

### SonarQube won't start

**Check logs:**

```bash
docker-compose logs sonarqube
```

**Common issues:**
- Not enough memory - increase `mem_limit`
- PostgreSQL not ready - wait 30+ seconds and retry
- Port already in use - change `SONAR_PORT` in `.env`

### PostgreSQL connection error

```bash
# Verify PostgreSQL is running and healthy
docker-compose ps

# Check PostgreSQL logs
docker-compose logs postgres
```

### Out of disk space

Check available space:

```bash
docker system df
```

Clean up volumes (⚠️ deletes data):

```bash
docker volume prune
```

## Persistence

Data is stored in Docker named volumes:
- `sonarqube-data` - SonarQube application data
- `sonarqube-extensions` - Plugins and extensions
- `sonarqube-logs` - Application logs
- `postgres-data` - Database files

Data persists across container restarts and stop/start cycles.

## Backing Up Data

### Backup PostgreSQL

```bash
docker-compose exec postgres pg_dump -U sonar sonar > backup.sql
```

### Restore PostgreSQL

```bash
docker-compose exec -T postgres psql -U sonar sonar < backup.sql
```

## Security Best Practices

1. **Change default password** - Do this immediately after first login
2. **Use strong credentials** - Change `DB_PASSWORD` in `.env`
3. **Never commit `.env`** - It's in `.gitignore` for a reason
4. **Restrict network access** - Don't expose ports to the internet without authentication
5. **Use HTTPS** - Deploy behind a reverse proxy (nginx, Traefik) for production
6. **Update regularly** - Pull latest images periodically:

```bash
docker-compose pull
docker-compose up -d
```

## Advanced Configuration

### Using Docker Compose Override

Create a `docker-compose.override.yml` for local development (not committed):

```yaml
version: '3.9'
services:
  sonarqube:
    environment:
      SONAR_WEB_JAVAADDITIONALOPTS: -Xms256m -Xmx512m
    mem_limit: 1g
```

### Behind a Reverse Proxy

For production, use Nginx or Traefik to:
- Terminate SSL/TLS
- Add authentication
- Handle rate limiting
- Provide better security

Example Nginx configuration:

```nginx
server {
    listen 443 ssl http2;
    server_name sonarqube.example.com;
    
    ssl_certificate /path/to/cert.pem;
    ssl_certificate_key /path/to/key.pem;
    
    location / {
        proxy_pass http://localhost:9000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

## Performance Tuning

### Increase Analyzer Memory

For large codebases, edit `docker-compose.yml`:

```yaml
sonarqube:
  environment:
    SONAR_CE_JAVAADDITIONALOPTS: -Xms1g -Xmx2g
```

### Database Optimization

Increase PostgreSQL connections if analyzing multiple projects:

```yaml
postgres:
  environment:
    POSTGRES_INITDB_ARGS: -c max_connections=500 -c shared_buffers=512MB
```

## Support and Resources

- **SonarQube Docs**: https://docs.sonarqube.org/
- **SonarQube Community**: https://community.sonarsource.com/
- **Docker Documentation**: https://docs.docker.com/
- **PostgreSQL Documentation**: https://www.postgresql.org/docs/

## License

This Docker Compose configuration is provided as-is. SonarQube Community Edition is licensed under AGPL v3.

## Monitoring

### Check container status

```bash
docker-compose ps
```

### View resource usage

```bash
docker stats
```

### Monitor SonarQube health

```bash
curl http://localhost:9000/api/system/health
```

Expected response:

```json
{"health":"GREEN","causes":[]}
```

## Tips & Tricks

### Run analysis without GUI

```bash
docker run --rm -v $(pwd):/src \
  sonarsource/sonar-scanner-cli \
  -Dsonar.projectKey=myproject \
  -Dsonar.sources=/src \
  -Dsonar.host.url=http://sonarqube:9000
```

### View SonarQube version

```bash
curl http://localhost:9000/api/system/info | jq .version
```

### Reset admin password

```bash
docker-compose exec postgres psql -U sonar -d sonar \
  -c "UPDATE users SET crypted_password = 'XXXXXXX' WHERE login = 'admin';"
```

---
