# Quick Start Guide - GitLab + SonarQube on M2 Mac

## Status
✅ **RUNNING** - All services are now operational and optimized for M2 Mac

## Starting Services

```bash
cd ~/gitlab-sonarqube
docker-compose up -d
```

**First boot**: 4-7 minutes  
**Subsequent boots**: 2-4 minutes

## Checking Status

```bash
# Check all containers
docker-compose ps

# View GitLab logs
docker logs gitlab -f

# View SonarQube logs
docker logs sonarqube -f
```

## Access Services

| Service | URL | Initial Login |
|---------|-----|---|
| **GitLab** | http://localhost:8080 | `root` / `YourSecurePassword123!` |
| **SonarQube** | http://localhost:9000 | `admin` / `admin` |

## Stop Services

```bash
docker-compose down
```

## Common Commands

### Reconfigure GitLab
```bash
docker exec gitlab gitlab-ctl reconfigure
```

### Restart a Single Service
```bash
docker-compose restart gitlab
docker-compose restart sonarqube
```

### View Service Logs
```bash
docker logs gitlab 2>&1 | tail -50
docker logs sonarqube 2>&1 | tail -50
docker logs postgres 2>&1 | tail -50
```

### Check PostgreSQL Connection
```bash
docker exec postgres pg_isready -U sonar
```

### Check SonarQube Health
```bash
curl http://localhost:9000/api/system/health
```

## System Requirements

- **OS**: macOS Ventura or newer
- **RAM**: 8GB available (16GB recommended)
- **Disk**: 50GB+ free space
- **Docker**: Desktop 4.20+

## What Was Fixed

1. ✅ **Port binding conflict** - Puma now uses internal port 8000
2. ✅ **Insufficient shared memory** - Increased from 256MB to 2GB
3. ✅ **Resource constraints** - Added memory/CPU limits
4. ✅ **Health checks** - Docker now detects boot completion
5. ✅ **Configuration issues** - Removed invalid Grafana option

## Configuration Details

**File**: `/Users/lin/gitlab-sonarqube/docker-compose.yml`

Key optimizations:
- GitLab: 4GB memory, 2 CPU cores
- SonarQube: 2GB memory, 1.5 CPU cores
- PostgreSQL: 1GB memory, 1 CPU core
- Puma threads: 2 (reduced from 4+)
- Redis cache: 512MB
- Shared memory: 2GB

See `SOLUTION.md` for detailed explanation.

## Next Steps

1. Log in to GitLab and change the root password
2. Create a test project
3. Configure GitLab Runner for CI/CD
4. Set up SonarQube integration with GitLab
5. Create your first pipeline

---

For issues, see `SOLUTION.md` troubleshooting section.
